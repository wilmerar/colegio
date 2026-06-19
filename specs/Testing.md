# Testing — Estrategia BDD y QA

> Los escenarios Gherkin viven en `Userstories.md`. Este documento define **cómo** convertirlos en pruebas ejecutables y qué cubrir en cada capa.

---

## 1. Pirámide de pruebas

```
        ┌─────────────┐
        │  E2E / BDD  │  ← Cucumber + escenarios de Userstories.md
        ├─────────────┤
        │ Integración │  ← API + DB (Supertest / Testcontainers)
        ├─────────────┤
        │   Unitarias │  ← Servicios, reglas de negocio
        └─────────────┘
```

| Capa | Herramienta | Cobertura objetivo |
|------|-------------|-------------------|
| Unitarias | Jest / Vitest | ≥ 80% en services |
| Integración | Supertest + PostgreSQL test | Todos los endpoints OpenAPI |
| E2E/BDD | Cucumber.js + Playwright (web) / Detox (móvil) | Must-have user stories |
| Contrato | OpenAPI validator | Request/response schema |

---

## 2. Unit tests — adding tests for new code

> **Rule:** Every PR with new business logic must include unit tests. No unit test → no merge (except trivial config/docs-only changes).

### 2.1 When unit tests are required

| Change type | Unit test required? | Where |
|-------------|---------------------|--------|
| New/changed **service** method (business rule) | **Yes** | `*.service.spec.ts` |
| New **guard** / RBAC check | **Yes** | `*.guard.spec.ts` |
| New **validator** / pure function in `packages/shared` | **Yes** | `*.spec.ts` next to file |
| New **custom hook** with logic (not just fetch wrapper) | **Yes** | `*.test.ts` |
| New **controller** endpoint only (delegates to service) | No* | Cover via integration test |
| UI component layout/styling only | No* | Optional snapshot; E2E if critical |
| DTO / type-only change | No | Contract test if OpenAPI changed |

\*Service behind the endpoint must already be unit-tested.

### 2.2 What to unit test (and what to skip)

**Do test:**
- Business rules from Gherkin (`Given/When/Then` → test cases)
- Branching logic (late submission, role denied, invalid state)
- Error codes returned (`USER_NOT_INVITED`, `PARENT_CANNOT_SUBMIT`)
- Edge cases (empty list, boundary dates, max score)

**Do not unit test:**
- Framework behaviour (NestJS routing, React rendering internals)
- Database queries in isolation — use integration tests with test DB
- Third-party SDKs (Google OAuth, FCM) — mock the adapter boundary

### 2.3 File layout & naming

```
apps/api/src/assignments/
├── assignments.service.ts
├── assignments.service.spec.ts      ← colocated
├── guards/
│   └── submission.guard.ts
│   └── submission.guard.spec.ts

packages/shared/src/validators/
├── submission.ts
└── submission.spec.ts

apps/web/src/hooks/
├── useStudentAssignments.ts
└── useStudentAssignments.test.ts
```

Naming: `describe('AssignmentsService')` → `describe('submitAssignment')` → `it('should mark as entregada_tarde when after due_at')`.

### 2.4 Structure — Arrange, Act, Assert

```typescript
describe('SubmissionsService', () => {
  describe('submit', () => {
    it('should set status entregada_tarde when submitted after due_at', async () => {
      // Arrange
      const assignment = fixture.assignment({ dueAt: '2026-06-25T23:59:00Z' });
      const dto = { assignmentId: assignment.id, submittedAt: '2026-06-26T10:00:00Z' };
      repo.findById.mockResolvedValue(assignment);

      // Act
      const result = await service.submit(dto, studentUser);

      // Assert
      expect(result.status).toBe('entregada_tarde');
      expect(eventBus.publish).toHaveBeenCalledWith('SubmissionReceived', expect.any(Object));
    });

    it('should throw ForbiddenException when parent tries to submit', async () => {
      await expect(service.submit(dto, parentUser)).rejects.toMatchObject({
        response: { error: { code: 'PARENT_CANNOT_SUBMIT' } },
      });
    });
  });
});
```

One main behaviour per `it`. Name tests as: **`should [expected] when [condition]`**.

### 2.5 Mocking rules (backend)

| Dependency | Mock strategy |
|------------|---------------|
| Repository / DB | `jest.mock` or manual mock object injected via NestJS TestingModule |
| External APIs (Google, FCM, S3) | Mock adapter class; never call real APIs in unit tests |
| Event bus / queue | `jest.fn()`; assert `publish` called with correct event |
| Other services | Mock only if not under test; prefer testing real service + mocked repo |

```typescript
const module = await Test.createTestingModule({
  providers: [
    SubmissionsService,
    { provide: ASSIGNMENT_REPOSITORY, useValue: mockRepo },
    { provide: EventBus, useValue: { publish: jest.fn() } },
  ],
}).compile();
```

### 2.6 Map Gherkin scenario → unit tests

Each **Scenario** in `Userstories.md` should produce at least one unit or integration test.

| Gherkin step | Unit test assertion |
|--------------|---------------------|
| `Then el registro queda con estado "tarde"` | `expect(result.status).toBe('tarde')` |
| `Then recibe error 403` | `rejects.toMatchObject({ status: 403 })` |
| `Then el padre recibe notificación` | `expect(eventBus.publish).toHaveBeenCalledWith(...)` |
| `Then no puede cambiar el voto` | second call throws / returns 409 |

### 2.7 Coverage expectations

| Area | Target |
|------|--------|
| `apps/api/src/**/*.service.ts` | ≥ **80%** lines/branches |
| `apps/api/src/**/guards/*.ts` | **100%** of permission branches |
| `packages/shared/src/**/*.ts` | ≥ **90%** |
| New file in PR | Must not **decrease** overall service coverage |

Run locally before PR:

```bash
pnpm --filter api test:unit -- --coverage
pnpm --filter web test -- --coverage
```

### 2.8 Frontend unit tests (web & mobile)

| Test | Tool | Example |
|------|------|---------|
| Pure utils / formatters | Vitest/Jest | `formatDueDate.spec.ts` |
| Custom hooks | React Testing Library + Vitest | `useStudentAssignments.test.ts` |
| Role-gated UI logic | RTL | render as parent → submit button absent |
| Screens | Optional | Prefer integration/E2E for flows |

```typescript
it('does not show submit button for parent role', () => {
  render(<AssignmentDetail assignment={mock} user={parentUser} />);
  expect(screen.queryByRole('button', { name: /entregar/i })).not.toBeInTheDocument();
});
```

### 2.9 PR checklist — unit tests (copy into PR)

```markdown
### Unit tests added
- [ ] New/changed service methods have tests in `*.service.spec.ts`
- [ ] Happy path + at least one error/edge case per method
- [ ] Gherkin scenario US-XXX covered
- [ ] Mocks at repository/adapter boundary (no real DB/API)
- [ ] `pnpm test:unit` passes locally
- [ ] Coverage not decreased (services ≥ 80%)
```

### 2.10 Anti-patterns (do not merge)

- Test that only asserts `toBeDefined()` or mock was called with no behaviour check
- Copy-paste tests with renamed strings only
- Testing private methods directly — test via public API
- Skipping tests with `it.skip` without linked issue
- Unit test that requires running PostgreSQL (move to integration)

---

## 3. Mapeo User Story → Suite de pruebas

| User Story | Tipo test | Archivo feature |
|------------|-----------|-----------------|
| US-020 | BDD + Integration | `features/auth/login.feature` |
| US-021 | Integration | `features/students/guardians.feature` |
| US-022 | Integration | `features/admin/users.feature` |
| US-023 | Integration | `features/director/teachers.feature` |
| US-024 | BDD + Integration | `features/director/reports.feature` |
| US-003 | BDD | `features/assignments/create.feature` |
| US-025 | BDD + Integration | `features/submissions/submit.feature` |
| US-026 | BDD | `features/submissions/grade.feature` |
| US-027 | Integration | `features/submissions/panel.feature` |
| US-004, US-028 | BDD | `features/assignments/student_view.feature` |
| US-029 | BDD + E2E | `features/parent/dashboard.feature` |
| US-001, US-002 | BDD | `features/attendance/*.feature` |
| US-005, US-006 | BDD | `features/discipline/*.feature` |

---

## 4. Ejemplo: feature ejecutable desde US-001

Archivo: `tests/features/attendance/entry.feature`

```gherkin
@attendance @us-001
Feature: Registro de entrada de alumno
  Como docente
  Quiero marcar la asistencia de entrada de mis alumnos
  Para que el sistema notifique al padre en tiempo real

  Background:
    Given el colegio "San Miguel" existe con horario de entrada "08:00"
    And la docente "María López" está autenticada en curso "3ro A"
    And el alumno "Juan Pérez" está matriculado en "3ro A"
    And el padre "Carlos Pérez" está vinculado a "Juan Pérez" con FCM token activo

  Scenario: Entrada a tiempo
    When la docente registra entrada de "Juan Pérez" a las "07:55"
    Then la respuesta API es 201
    And el registro tiene estado "presente"
    And se encola notificación push al padre con texto que contiene "ingresó al colegio"

  Scenario: Entrada tardía
    When la docente registra entrada de "Juan Pérez" a las "08:15"
    Then el registro tiene estado "tarde"
    And se encola notificación push al padre con texto que contiene "llegó tarde"

  Scenario: Falta
    When la docente registra ausencia de "Juan Pérez"
    Then el registro tiene estado "ausente"
    And se encola notificación push al padre con texto que contiene "no asistió"
```

### Step definitions (TypeScript + Cucumber)

```typescript
// tests/steps/attendance.steps.ts
When('la docente registra entrada de {string} a las {string}', async (student, time) => {
  response = await api.post('/attendance').send({
    student_id: context.students[student].id,
    check_in_at: `${today}T${time}:00-04:00`,
    status: 'presente',
  });
});
```

---

## 5. Pruebas de integración API (por endpoint)

Cada endpoint en `Openapi.yml` debe tener al menos:

1. **Happy path** — 2xx con schema válido
2. **Auth** — 401 sin token, 403 rol incorrecto
3. **Validación** — 400 campos inválidos
4. **Multi-tenant** — 403 acceso cross-school
5. **Not found** — 404 recurso inexistente

### Ejemplo: POST /attendance

```typescript
describe('POST /attendance', () => {
  it('201 - registra entrada (US-001)', async () => { /* ... */ });
  it('403 - docente de otro curso', async () => { /* ... */ });
  it('401 - sin token (US-020)', async () => { /* ... */ });
  it('400 - student_id inválido', async () => { /* ... */ });
});
```

---

## 6. Pruebas de reglas de negocio (unitarias)

| Regla | User Story | Test |
|-------|------------|------|
| Solo alumno puede entregar tarea | US-025 | `submissions.guard.spec.ts` |
| Padre no puede entregar por el hijo | US-004 | `submissions.guard.spec.ts` |
| Entrega tardía si `submitted_at > due_at` | US-025 | `submissions.service.spec.ts` |
| Director no crea admins | US-023 | `director.guard.spec.ts` |
| Panel padre solo hijos vinculados | US-029 | `parent-dashboard.guard.spec.ts` |
| Entrada después de `entry_time` → `tarde` | US-001 | `attendance.service.spec.ts` |
| `due_at` obligatorio en tarea | US-003 | `assignments.service.spec.ts` |
| Padre solo ve hijos vinculados | US-021 | `guardian.guard.spec.ts` |

---

## 7. Pruebas de notificaciones

Mock de FCM en tests; verificar cola de eventos:

```typescript
it('publica evento AttendanceRecorded al marcar entrada', async () => {
  await attendanceService.recordEntry(dto);
  expect(eventBus.publish).toHaveBeenCalledWith(
    'AttendanceRecorded',
    expect.objectContaining({ student_id: dto.student_id })
  );
});
```

**Casos de notificación:**

| Evento | US | Assert |
|--------|-----|--------|
| AttendanceRecorded | US-001 | Push al padre correcto |
| AssignmentCreated | US-003 | Push a padres del curso |
| AnnouncementPublished | US-007 | Push según audiencia |
| SignatureRequested | US-018 | Push al padre asignado |

---

## 8. Pruebas de seguridad

| Caso | Expected |
|------|----------|
| Padre accede alumno ajeno | 403 |
| Docente marca asistencia curso no asignado | 403 |
| Padre ve álbum de otro curso | 403 (US-015) |
| Admin audita chat sin motivo | 400 |
| JWT expirado | 401 |
| Rate limit excedido | 429 |

---

## 9. Pruebas E2E (flujos críticos MVP)

### Flujo 1: Día escolar completo
1. Docente marca entrada → padre recibe push (simulado)
2. Docente publica tarea → padre la ve en agenda
3. Docente registra incidente → padre firma acuse (US-019)
4. Docente marca salida → padre recibe push

### Flujo 2: Comunicación
1. Director publica circular
2. Padre la lee → contador baja
3. Padre chatea con docente

### Flujo 3: Firma excursión
1. Docente solicita firma
2. Padre firma en canvas
3. Docente ve confirmación

---

## 10. CI/CD — Pipeline de tests

```yaml
# .github/workflows/test.yml
jobs:
  test:
    steps:
      - run: npm run test:unit
      - run: npm run test:integration  # levanta PG con docker
      - run: npm run test:bdd           # cucumber features/
      - run: npm run test:openapi       # valida contrato
      - run: npm run test:e2e           # solo en main/nightly
```

**Gates:**
- PR: unit + integration + BDD Must stories
- Main: + E2E smoke
- Release: + prueba manual checklist (ver abajo)

---

## 11. Checklist manual pre-release

- [ ] Login padre/docente/admin en dispositivo real
- [ ] Push notification en Android e iOS
- [ ] Subida de adjunto en tarea (foto desde cámara)
- [ ] Firma digital en pantalla táctil
- [ ] Chat sin filtración de teléfono
- [ ] QR de pago escaneable
- [ ] Tiempo de respuesta API < 500ms p95

---

## 12. Datos de prueba (fixtures)

| Entidad | Fixture |
|---------|---------|
| Colegio | `san-miguel`, entry 08:00 |
| Curso | `3ro A`, año 2026 |
| Docente | `maria.lopez@san-miguel.edu` |
| Padre | `carlos.perez@email.com` |
| Alumno | `Juan Pérez`, enrollment `JP-2026-001` |

Fixtures en `tests/fixtures/` — recreados por transacción rollback en integration tests.

---

## 13. Métricas de calidad

| Métrica | Objetivo MVP |
|---------|--------------|
| Cobertura unitaria (services) | ≥ 80% |
| Endpoints con test integración | 100% |
| User Stories Must con escenario BDD ejecutable | 100% |
| Bugs críticos abiertos en release | 0 |
| p95 latencia API (staging) | < 500ms |

---

## 14. Herramientas recomendadas

| Propósito | Herramienta |
|-----------|-------------|
| BDD runner | `@cucumber/cucumber` |
| API testing | `supertest` + `openapi-validator` |
| DB test | `@testcontainers/postgresql` |
| Mock FCM | `nock` o mock in-process |
| E2E móvil | Detox (React Native) o Patrol (Flutter) |
| E2E web | Playwright |
| Coverage | Istanbul / c8 |
