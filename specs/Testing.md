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

## 2. Mapeo User Story → Suite de pruebas

| User Story | Tipo test | Archivo feature (propuesto) |
|------------|-----------|----------------------------|
| US-001 | BDD + Integration | `features/attendance/entry.feature` |
| US-002 | BDD + Integration | `features/attendance/history.feature` |
| US-003 | BDD + Integration | `features/assignments/create.feature` |
| US-004 | BDD + Integration | `features/assignments/list.feature` |
| US-005 | BDD | `features/discipline/incident.feature` |
| US-006 | BDD | `features/discipline/compliment.feature` |
| US-007 | BDD + Integration | `features/announcements/broadcast.feature` |
| US-008 | BDD | `features/announcements/read.feature` |
| US-009 | BDD | `features/calendar/events.feature` |
| US-010 | BDD + Integration | `features/messaging/chat.feature` |
| US-011 | Integration | `features/messaging/audit.feature` |
| US-012 | BDD | `features/payments/qr.feature` |
| US-013 | BDD | `features/grades/publish.feature` |
| US-014 | BDD | `features/attendance/checkout.feature` |
| US-015 | BDD + Integration | `features/galleries/album.feature` |
| US-016 | Integration | `features/directory/contacts.feature` |
| US-017 | BDD | `features/polls/vote.feature` |
| US-018 | BDD + E2E | `features/signatures/excursion.feature` |
| US-019 | BDD | `features/signatures/discipline_ack.feature` |
| US-020 | Integration | `features/auth/login.feature` |
| US-021 | Integration | `features/students/guardians.feature` |

---

## 3. Ejemplo: feature ejecutable desde US-001

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

## 4. Pruebas de integración API (por endpoint)

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

## 5. Pruebas de reglas de negocio (unitarias)

| Regla | User Story | Test |
|-------|------------|------|
| Entrada después de `entry_time` → `tarde` | US-001 | `attendance.service.spec.ts` |
| `due_date` obligatorio en tarea | US-003 | `assignments.service.spec.ts` |
| Gravedad solo en incidentes | US-005 | `discipline.service.spec.ts` |
| Padre solo ve hijos vinculados | US-021 | `guardian.guard.spec.ts` |
| Un voto por encuesta | US-017 | `polls.service.spec.ts` |
| Chat solo entre padre-docente del mismo curso | US-010 | `messaging.service.spec.ts` |
| Firma guarda timestamp + device_info | US-018 | `signatures.service.spec.ts` |

---

## 6. Pruebas de notificaciones

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

## 7. Pruebas de seguridad

| Caso | Expected |
|------|----------|
| Padre accede alumno ajeno | 403 |
| Docente marca asistencia curso no asignado | 403 |
| Padre ve álbum de otro curso | 403 (US-015) |
| Admin audita chat sin motivo | 400 |
| JWT expirado | 401 |
| Rate limit excedido | 429 |

---

## 8. Pruebas E2E (flujos críticos MVP)

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

## 9. CI/CD — Pipeline de tests

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

## 10. Checklist manual pre-release

- [ ] Login padre/docente/admin en dispositivo real
- [ ] Push notification en Android e iOS
- [ ] Subida de adjunto en tarea (foto desde cámara)
- [ ] Firma digital en pantalla táctil
- [ ] Chat sin filtración de teléfono
- [ ] QR de pago escaneable
- [ ] Tiempo de respuesta API < 500ms p95

---

## 11. Datos de prueba (fixtures)

| Entidad | Fixture |
|---------|---------|
| Colegio | `san-miguel`, entry 08:00 |
| Curso | `3ro A`, año 2026 |
| Docente | `maria.lopez@san-miguel.edu` |
| Padre | `carlos.perez@email.com` |
| Alumno | `Juan Pérez`, enrollment `JP-2026-001` |

Fixtures en `tests/fixtures/` — recreados por transacción rollback en integration tests.

---

## 12. Métricas de calidad

| Métrica | Objetivo MVP |
|---------|--------------|
| Cobertura unitaria (services) | ≥ 80% |
| Endpoints con test integración | 100% |
| User Stories Must con escenario BDD ejecutable | 100% |
| Bugs críticos abiertos en release | 0 |
| p95 latencia API (staging) | < 500ms |

---

## 13. Herramientas recomendadas

| Propósito | Herramienta |
|-----------|-------------|
| BDD runner | `@cucumber/cucumber` |
| API testing | `supertest` + `openapi-validator` |
| DB test | `@testcontainers/postgresql` |
| Mock FCM | `nock` o mock in-process |
| E2E móvil | Detox (React Native) o Patrol (Flutter) |
| E2E web | Playwright |
| Coverage | Istanbul / c8 |
