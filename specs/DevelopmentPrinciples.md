# Development Principles — Clean Code & Best Practices

> Guía de ingeniería para el monorepo Colegio (NestJS + Next.js + React Native).
> Aplica a **todo** el código: backend, web, móvil y packages compartidos.

---

## 1. Filosofía core

| Principio | Significado | En este proyecto |
|-----------|-------------|------------------|
| **KISS** | Keep It Simple, Stupid | Preferir solución directa sobre abstracción prematura |
| **DRY** | Don't Repeat Yourself | Una fuente de verdad: specs, tipos OpenAPI, validadores shared |
| **SOLID** | Cinco principios OOP | Especialmente SRP e DIP en módulos NestJS |
| **YAGNI** | You Aren't Gonna Need It | MVP first; no microservicios ni features especulativas |
| **Clean Code** | Código legible y mantenible | Nombres claros, funciones pequeñas, tests significativos |

### Regla de oro

> Si la spec (`Userstories.md`) no lo pide, **no lo construyas** (YAGNI).
> Si ya existe en `packages/shared` o `Openapi.yml`, **no lo dupliques** (DRY).

---

## 2. KISS — Keep It Simple

### Hacer

- Un módulo NestJS por dominio (`assignments`, `attendance`, `auth`)
- DTOs planos con class-validator; sin capas extra innecesarias
- Componentes React con una responsabilidad visible
- Flujos lineales: controller → service → repository

### Evitar

- Generic factories para casos con una sola implementación
- Patrón Strategy cuando solo hay una estrategia
- Wrappers de API que solo reenvían sin lógica
- Over-engineering de estado global (Redux) cuando React Query basta

```typescript
// ❌ Complejo sin necesidad
class AssignmentSubmissionStrategyFactory {
  create(type: string): ISubmissionStrategy { ... }
}

// ✅ KISS — lógica directa en service
async submitAssignment(dto: SubmitDto, studentId: string) {
  const assignment = await this.repo.findOrFail(dto.assignmentId);
  this.validateDueDate(assignment, dto.submittedAt);
  return this.repo.save({ ...dto, studentId, status: this.resolveStatus(assignment) });
}
```

---

## 3. DRY — Don't Repeat Yourself

### Fuentes únicas de verdad

| Qué | Dónde vive | No duplicar en |
|-----|------------|----------------|
| Contrato API | `specs/Openapi.yml` | Controllers, clientes web/móvil |
| Tipos/enums | `packages/shared` | Copiar strings mágicos en apps |
| Reglas de negocio | NestJS services | Web, móvil, controllers |
| Esquema BD | `specs/Database.md` + migraciones | SQL ad-hoc en repos |
| User stories | `specs/Userstories.md` | Lógica no trazable |

### DRY entre web y móvil

```typescript
// packages/shared/src/roles.ts
export const ROLES = ['admin', 'director', 'teacher', 'student', 'parent'] as const;

// packages/shared/src/validators/submission.ts
export const submissionStatusSchema = z.enum([
  'pendiente', 'entregada', 'entregada_tarde', 'calificada', 'devuelta',
]);
```

### Cuándo NO aplicar DRY ciegamente

- Web y móvil tienen UX distinta → componentes UI separados, lógica compartida en hooks/shared
- Dos dominios similares hoy pero evolucionan distinto → duplicar temporalmente (YAGNI) hasta ver patrón real

---

## 4. SOLID — Aplicado al stack

### S — Single Responsibility Principle

Cada clase/archivo = **una razón para cambiar**.

```
assignments/
├── assignments.controller.ts   # HTTP only
├── assignments.service.ts      # business rules
├── assignments.repository.ts   # DB access
└── dto/
```

```typescript
// ❌ Controller con lógica de negocio
@Post()
create(@Body() dto) {
  if (dto.due_at < new Date()) throw new BadRequestException();
  // 50 lines más...
}

// ✅ Controller delgado
@Post()
create(@Body() dto: CreateAssignmentDto, @CurrentUser() user: AuthUser) {
  return this.assignmentsService.create(dto, user);
}
```

### O — Open/Closed Principle

Abierto a extensión, cerrado a modificación.

```typescript
// ✅ Nuevo proveedor OAuth sin tocar login local
interface OAuthProvider {
  verifyToken(token: string): Promise<OAuthProfile>;
}

@Injectable()
class GoogleOAuthProvider implements OAuthProvider { ... }

// Registrar en auth module; LocalAuthService no cambia
```

### L — Liskov Substitution Principle

Subtipos sustituibles sin romper contrato.

```typescript
// ✅ Cualquier NotificationSender envía sin que el caller sepa cuál es
interface NotificationSender {
  send(payload: NotificationPayload): Promise<void>;
}
// FcmSender, EmailSender — misma interfaz
```

### I — Interface Segregation Principle

Interfaces pequeñas y específicas.

```typescript
// ❌ Interface gigante
interface UserRepository {
  findById, findByEmail, save, delete, findTeachers, findParents, ...
}

// ✅ Interfaces segregadas o repository por agregado
interface AssignmentRepository {
  findById(id: string): Promise<Assignment>;
  save(assignment: Assignment): Promise<Assignment>;
}
```

### D — Dependency Inversion Principle

Depender de abstracciones, no de implementaciones.

```typescript
// ✅ Service depende de interfaz/token de inyección
constructor(
  @Inject(ASSIGNMENT_REPOSITORY) private readonly repo: AssignmentRepository,
  private readonly eventBus: EventBus,
) {}
```

---

## 5. YAGNI — You Aren't Gonna Need It

### Construir ahora (MVP)

- Auth local + Google OAuth
- LMS: publicar, entregar, calificar
- Panel padre, asistencia, admin usuarios
- PostgreSQL + Redis + S3

### No construir hasta que la spec lo pida

| Tentación | Por qué esperar |
|-----------|-----------------|
| Microservicios | Un monolito modular NestJS basta |
| GraphQL | REST + OpenAPI ya definido |
| Event sourcing | CRUD + cola BullMQ es suficiente |
| Multi-idioma i18n completo | MVP en español |
| Offline sync móvil | Fase v1.3 en roadmap |
| Microsoft/Apple OAuth | Solo Google en MVP; enum extensible |
| CQRS | YAGNI para este tamaño |

---

## 6. Clean Code — Principios prácticos

### 6.1 Nombres significativos

```typescript
// ❌
const d = await repo.get(id);
const flag = u.r === 'parent';

// ✅
const assignment = await assignmentRepository.findById(assignmentId);
const isParent = user.role === 'parent';
```

Convenciones del proyecto:

| Contexto | Convención | Ejemplo |
|----------|------------|---------|
| Tablas BD | snake_case plural | `assignment_submissions` |
| TypeScript | camelCase | `submissionStatus` |
| Clases | PascalCase | `AssignmentsService` |
| Archivos | kebab-case o dot.notation | `assignments.service.ts` |
| Constantes | SCREAMING_SNAKE | `MAX_ATTACHMENT_SIZE` |
| Endpoints | kebab-case REST | `/auth/link/google` |

### 6.2 Funciones pequeñas

- Máximo **~30 líneas** por función; si crece, extraer privados
- Un nivel de abstracción por función
- Máximo **3–4 parámetros**; agrupar en objeto/DTO si más

### 6.3 Comentarios

- El código explica el **qué**; comentarios el **por qué** no obvio
- No comentar código obvio
- Gherkin en specs, no duplicado como comentarios en código

```typescript
// ✅ Por qué de negocio
// Padres no pueden entregar tareas — US-004, solo lectura
if (user.role === 'parent') {
  throw new ForbiddenException('PARENT_CANNOT_SUBMIT');
}
```

### 6.4 Manejo de errores

```typescript
// ✅ Errores de dominio explícitos
throw new ForbiddenException({
  error: {
    code: 'USER_NOT_INVITED',
    message: 'El email no está registrado en este colegio',
  },
});

// ❌ Tragar errores
try { await save(); } catch (e) { return null; }

// ❌ throw genérico
throw new Error('something went wrong');
```

### 6.5 No dejar código muerto

- Sin `console.log` en commits (usar Logger de NestJS)
- Sin imports no usados
- Sin features comentadas — git guarda el historial

### 6.6 Tests limpios

- Patrón Arrange – Act – Assert
- Un assert principal por test
- Nombres: `should mark submission as late when submitted after due_at`

---

## 7. Design patterns — Cuándo usarlos

Patrones **recomendados** para este proyecto:

| Patrón | Dónde | Propósito |
|--------|-------|-----------|
| **Repository** | NestJS data layer | Aislar PostgreSQL de services |
| **DTO** | API boundary | Validación entrada/salida |
| **Guard / Middleware** | NestJS, Next.js | RBAC, tenant, auth |
| **Factory** | Crear entidades complejas | `SubmissionFactory.createFromDto()` |
| **Strategy** | OAuth providers, notification senders | Google vs local; FCM vs email |
| **Observer / Events** | Domain events → BullMQ | `AssignmentGraded` → push padre |
| **Adapter** | Integraciones externas | Google API, FCM, S3 wrapper |
| **Facade** | `ParentDashboardService` | Agrega asistencia+tareas+notas en una llamada |
| **Decorator** | `@Roles()`, `@CurrentUser()` | Cross-cutting auth |
| **Module** | NestJS modules | Bounded contexts |

Patrones **evitar por ahora** (YAGNI):

| Patrón | Por qué no |
|--------|------------|
| Singleton manual | NestJS DI ya lo gestiona |
| Abstract Factory complejo | Pocos productos familiares |
| Flyweight | No hay problema de memoria a escala |
| Mediator custom | Event bus / BullMQ suficiente |

### Ejemplo: Repository + Service (backend)

```typescript
// repository — solo datos
@Injectable()
export class AssignmentRepository {
  constructor(private readonly db: PrismaService) {}

  findPublishedByCourse(courseId: string) {
    return this.db.assignment.findMany({
      where: { courseId, status: 'publicada' },
      orderBy: { dueAt: 'asc' },
    });
  }
}

// service — reglas de negocio
@Injectable()
export class AssignmentsService {
  constructor(private readonly repo: AssignmentRepository) {}

  async listForStudent(studentId: string, user: AuthUser) {
    await this.guardianGuard.assertStudentAccess(user, studentId);
    const courseId = await this.studentsService.getCourseId(studentId);
    return this.repo.findPublishedByCourse(courseId);
  }
}
```

### Ejemplo: Custom hook (frontend)

```typescript
// ✅ DRY — lógica compartida web/móvil via api-client
export function useStudentAssignments(studentId: string) {
  return useQuery({
    queryKey: ['assignments', studentId],
    queryFn: () => api.GET('/students/{studentId}/assignments', {
      params: { path: { studentId } },
    }),
    enabled: !!studentId,
  });
}
```

---

## 8. Principios por capa

### Backend (NestJS)

1. Controllers sin lógica de negocio
2. Services sin SQL directo
3. Siempre filtrar por `school_id` del JWT
4. DTOs = contrato OpenAPI
5. Transacciones para operaciones multi-tabla
6. Domain events async para side effects (push, email)

### Web (Next.js)

1. Server Components por defecto
2. `'use client'` solo con interactividad
3. Layouts por rol — no condicionales gigantes en una página
4. Fetch en server cuando sea posible
5. No duplicar validación — Zod schemas from shared

### Mobile (React Native)

1. Screens delgadas; lógica en hooks
2. Una responsabilidad por screen
3. Manejar loading/error/empty en toda pantalla
4. SecureStore para tokens; nunca AsyncStorage para secrets

### Shared packages

1. Sin dependencias de framework
2. Solo tipos, constantes, validadores puros
3. Sin lógica que pertenezca al backend

---

## 9. Checklist pre-PR

- [ ] ¿Resuelve una user story trazable en `Userstories.md`?
- [ ] ¿KISS — es la solución más simple que funciona?
- [ ] ¿DRY — tipos/reglas en shared o service, no copiados?
- [ ] ¿YAGNI — sin features fuera de scope?
- [ ] ¿SOLID — controller delgado, service con reglas, repo con datos?
- [ ] ¿Clean — nombres claros, sin console.log, errores tipados?
- [ ] ¿OpenAPI actualizado si cambió la API?
- [ ] ¿Tests para reglas de negocio nuevas?
- [ ] ¿RBAC verificado (padre no entrega, admin cross-tenant bloqueado)?

---

## 10. Referencias

- Specs SDD: `Userstories.md`, `DefinitionOfDone.md`, `Testing.md`
- Stack: `Architecture.md`
- Cursor rules: `.cursor/rules/`
- Libros: *Clean Code* (Robert C. Martin), *Clean Architecture* (Robert C. Martin)
