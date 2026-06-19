# Architecture — Aplicación de Colegios

## 1. Visión del producto

Plataforma **web + móvil** centrada en:

1. **Control del alumno por padres de familia** — dashboard unificado (asistencia, tareas, notas, disciplina).
2. **Gestión de tareas estilo [Moodle](https://moodle.org/)** — publicar, entregar, calificar, feedback.
3. **Jerarquía escolar clara** — Administrador → Director → Profesor → Alumno/Padre.

### Clientes oficiales

| Cliente | Stack | Roles |
|---------|-------|-------|
| **Portal Web** | Next.js 14 + TypeScript | Admin, Director, Profesor, Padre, Alumno |
| **App Móvil** | React Native + TypeScript (Expo) | Profesor, Padre, Alumno |

> Una sola **API REST** (`Openapi.yml`) sirve a web y móvil. TypeScript en las tres capas (backend, web, móvil).

### Distribución web vs móvil por rol

| Rol | Web | Móvil | Razón |
|-----|:---:|:-----:|-------|
| Administrador | ✓ | — | Gestión compleja; solo escritorio |
| Director | ✓ | — | Reportes, tablas, export PDF |
| Profesor | ✓ | ✓ | Web: panel entregas; móvil: asistencia rápida en aula |
| Alumno | ✓ | ✓ | Móvil: entregar tareas con cámara; web: vista cómoda |
| Padre | ✓ | ✓ | **Móvil principal** — push y monitoreo diario |

---

## 2. Jerarquía de roles

```mermaid
flowchart TB
  ADMIN[Administrador<br/>Usuarios + Módulos + Todo]
  DIRECTOR[Director<br/>Profesores + Reportes]
  TEACHER[Profesor<br/>Tareas + Control alumnos]
  STUDENT[Alumno<br/>Entregas + Consulta]
  PARENT[Padre<br/>Monitoreo del hijo]

  ADMIN --> DIRECTOR
  ADMIN --> TEACHER
  DIRECTOR --> TEACHER
  TEACHER --> STUDENT
  TEACHER --> PARENT
  STUDENT -.->|vinculado| PARENT
```

| Rol | Código | Acceso |
|-----|--------|--------|
| Administrador | `admin` | Total del tenant |
| Director | `director` | Profesores, reportes, comunicados |
| Profesor | `teacher` | Sus cursos: tareas, asistencia, disciplina, calificaciones |
| Alumno | `student` | Propio: tareas, entregas, asistencia, notas |
| Padre | `parent` | Hijo(s) vinculados: monitoreo, sin entregas |

---

## 3. Principios arquitectónicos

1. **SDD:** User stories Gherkin → Models → Database → OpenAPI → código.
2. **Multi-tenant:** Aislamiento por `school_id`.
3. **API-first, multi-cliente:** Next.js (web) + React Native (móvil) → misma API NestJS.
4. **TypeScript end-to-end:** Tipos generados desde `Openapi.yml` compartidos en web y móvil.
4. **LMS módulo central:** Tareas con ciclo Moodle (publish → submit → grade → notify).
5. **RBAC estricto:** Middleware de rol + ownership (padre→hijo, profesor→curso).
6. **Event-driven:** Notificaciones push/email vía cola async.

---

## 4. Diagrama de contexto

```mermaid
C4Context
  title App Colegios — Web + Móvil

  Person(admin, "Administrador", "Usuarios y módulos")
  Person(director, "Director", "Profesores y reportes")
  Person(teacher, "Profesor", "Tareas Moodle, control alumnos")
  Person(student, "Alumno", "Entrega tareas")
  Person(parent, "Padre", "Monitorea hijo")

  System(app, "App Colegios", "API + Web + Móvil")
  System_Ext(fcm, "Firebase FCM", "Push")
  System_Ext(storage, "Object Storage", "Entregas y adjuntos")

  Rel(admin, app, "Web")
  Rel(director, app, "Web")
  Rel(teacher, app, "Web + Móvil")
  Rel(student, app, "Web + Móvil")
  Rel(parent, app, "Móvil + Web")
  Rel(app, fcm, "Push")
  Rel(app, storage, "Archivos")
```

---

## 5. Diagrama de contenedores

```mermaid
flowchart TB
  subgraph clients [Clientes]
    WEB[Portal Web<br/>Next.js + TypeScript]
    MOBILE[App Móvil<br/>React Native + Expo]
  end

  subgraph backend [Backend Node.js]
    API[API REST<br/>NestJS + TypeScript]
    WORKER[Notification Worker]
  end

  subgraph data [Datos]
    PG[(PostgreSQL)]
    REDIS[(Redis)]
    S3[(MinIO/S3)]
  end

  WEB --> API
  MOBILE --> API
  API --> PG
  API --> REDIS
  API --> S3
  REDIS --> WORKER
  WORKER --> PG
```

---

## 6. Módulo LMS (estilo Moodle)

```
assignments/
├── create          # Profesor publica tarea (US-003)
├── list            # Alumno/padre/profesor listan
├── submissions/
│   ├── submit      # Alumno entrega (US-025)
│   ├── grade       # Profesor califica (US-026)
│   └── panel       # Panel entregas (US-027)
└── notifications   # Push a alumno y padre
```

### Estados de entrega (Submission)

```
pendiente → entregada | entregada_tarde → calificada
                                      ↘ devuelta (rehacer)
```

---

## 7. Módulos de dominio

```
src/
├── auth/              # US-020 Login web/móvil
├── admin/             # US-022 Usuarios y módulos
├── director/          # US-023, US-024 Profesores y reportes
├── users/
├── students/
├── guardians/         # US-021 Vinculación padre-alumno
├── assignments/       # US-003–028 LMS Moodle
├── submissions/       # Entregas de alumnos
├── parent-dashboard/  # US-029 Panel control parental
├── attendance/        # US-001, US-002
├── discipline/        # US-005, US-006
├── grades/            # US-013
├── announcements/
├── signatures/
└── notifications/
```

---

## 8. Stack tecnológico oficial

| Capa | Tecnología | Notas |
|------|------------|-------|
| **Backend** | Node.js + **NestJS** + TypeScript | API REST, RBAC, multi-tenant |
| **Web** | **Next.js 14** (App Router) + TypeScript | Un portal con layouts por rol |
| **Móvil** | **React Native** + TypeScript + **Expo** | iOS + Android, un codebase |
| DB | PostgreSQL 16 | |
| Cache/Colas | Redis + BullMQ | Notificaciones async |
| Storage | MinIO / S3 | Entregas, adjuntos, firmas |
| Auth | JWT + refresh token | Secure storage en móvil |
| Push | Firebase FCM | `@react-native-firebase/messaging` o Expo Notifications |
| API Spec | OpenAPI 3.1 | Generación de cliente con `openapi-typescript` |
| Monorepo (opcional) | Turborepo o pnpm workspaces | `apps/api`, `apps/web`, `apps/mobile`, `packages/shared` |

### Por qué React Native (decisión registrada)

- Un solo codebase para Android e iOS.
- Mismo lenguaje (TypeScript) que NestJS y Next.js.
- Funcionalidades clave cubiertas: push, cámara, adjuntos, firma en canvas.
- MVP más rápido que Kotlin + Swift por separado.

### Estructura de repositorio propuesta

```
colegio/
├── apps/
│   ├── api/                 # NestJS (Node.js)
│   ├── web/                 # Next.js — portal único, rutas por rol
│   └── mobile/              # React Native + Expo
├── packages/
│   ├── shared/              # Tipos, validadores, constantes
│   └── api-client/          # Cliente generado desde Openapi.yml
└── specs/                   # SDD (este proyecto)
```

### Web — Next.js: layouts por rol

```
app/
├── (auth)/login/
├── (admin)/          # Solo rol admin
│   ├── users/
│   └── modules/
├── (director)/       # Solo rol director
│   ├── teachers/
│   └── reports/
├── (teacher)/        # Rol teacher
│   ├── assignments/
│   ├── submissions/  # Panel entregas Moodle
│   └── attendance/
└── (family)/         # Padre + alumno
    ├── dashboard/    # Panel control (padre)
    ├── assignments/  # Lista y detalle tareas
    └── submit/       # Entrega (alumno)
```

### Móvil — React Native: navegación por rol

Tras login, redirigir según `user.role`:

```
MobileNavigator
├── TeacherStack     # Profesor
├── StudentStack     # Alumno
└── ParentStack      # Padre
```

Admin y director **no tienen app móvil** — se les muestra mensaje para usar el portal web.

---

## 9. Matriz de pantallas por rol y plataforma

Leyenda: **W** = Web (Next.js) · **M** = Móvil (React Native) · **—** = no disponible

### Administrador (solo Web)

| Pantalla | W | M | User Story |
|----------|:-:|:-:|------------|
| Login | ✓ | — | US-020 |
| Gestión de usuarios (CRUD) | ✓ | — | US-022 |
| Activar/desactivar módulos | ✓ | — | US-022 |
| Vincular padre ↔ alumno | ✓ | — | US-021 |
| Configuración del colegio | ✓ | — | US-022 |
| Vista de todos los módulos | ✓ | — | — |

### Director (solo Web)

| Pantalla | W | M | User Story |
|----------|:-:|:-:|------------|
| Login | ✓ | — | US-020 |
| Listado de profesores | ✓ | — | US-023 |
| Asignar profesor → curso/materia | ✓ | — | US-023 |
| Reportes por profesor | ✓ | — | US-024 |
| Exportar reporte PDF | ✓ | — | US-024 |
| Comunicados oficiales | ✓ | — | US-007 |
| Dashboard resumen colegio | ✓ | — | — |

### Profesor (Web + Móvil)

| Pantalla | W | M | User Story | Notas |
|----------|:-:|:-:|------------|-------|
| Login | ✓ | ✓ | US-020 | |
| Publicar tarea (Moodle) | ✓ | ✓ | US-003 | Web: formulario completo; móvil: versión simplificada |
| Panel de entregas | ✓ | M* | US-027 | *Móvil: resumen; web: tabla completa |
| Calificar entrega + feedback | ✓ | ✓ | US-026 | |
| Marcar asistencia | ✓ | ✓ | US-001 | **Móvil prioritario** en aula |
| Historial asistencia curso | ✓ | — | US-002 | |
| Cuaderno disciplina | ✓ | ✓ | US-005, US-006 | |
| Publicar calificaciones | ✓ | — | US-013 | |
| Solicitar firma digital | ✓ | ✓ | US-018 | |
| Chat con padres | — | ✓ | US-010 | Fase v1.2; móvil first |

### Alumno (Web + Móvil)

| Pantalla | W | M | User Story | Notas |
|----------|:-:|:-:|------------|-------|
| Login | ✓ | ✓ | US-020 | |
| Mis tareas (pendientes/entregadas/calificadas) | ✓ | ✓ | US-028 | |
| Detalle de tarea + instrucciones | ✓ | ✓ | US-028 | |
| **Entregar tarea** (archivo/foto) | ✓ | ✓ | US-025 | **Móvil prioritario** — cámara |
| Ver calificación y feedback | ✓ | ✓ | US-026 | |
| Mi asistencia | ✓ | ✓ | US-002 | |
| Mis calificaciones | ✓ | ✓ | US-013 | |
| Comunicados | ✓ | ✓ | US-008 | |
| Calendario escolar | ✓ | ✓ | US-009 | |

### Padre de familia (Web + Móvil)

| Pantalla | W | M | User Story | Notas |
|----------|:-:|:-:|------------|-------|
| Login | ✓ | ✓ | US-020 | |
| **Panel de control del hijo** | ✓ | ✓ | US-029 | **Móvil = pantalla principal** |
| Tareas del hijo (solo lectura) | ✓ | ✓ | US-004 | No puede entregar |
| Asistencia del hijo | ✓ | ✓ | US-002 | Push en móvil |
| Calificaciones del hijo | ✓ | ✓ | US-013 | |
| Disciplina del hijo | ✓ | ✓ | US-005 | |
| **Firma digital** (autorizaciones) | ✓ | ✓ | US-018, US-019 | Canvas táctil en móvil |
| Comunicados | ✓ | ✓ | US-008 | |
| Chat con profesor | — | ✓ | US-010 | Fase v1.2 |
| Pagos / QR pensión | ✓ | ✓ | US-012 | Fase v1.3 |

### Pantallas exclusivas de móvil

| Pantalla | Rol | Motivo |
|----------|-----|--------|
| Notificaciones push (centro) | Todos en móvil | FCM nativo |
| Marcar asistencia rápida en aula | Profesor | Uso en pie, sin laptop |
| Entregar tarea con cámara | Alumno | Foto directa del cuaderno |
| Firma en canvas táctil | Padre | Experiencia natural |
| Salida segura (check-out) | Profesor | Portería / salida del colegio |

### Pantallas exclusivas de web

| Pantalla | Rol | Motivo |
|----------|-----|--------|
| CRUD usuarios | Admin | Tablas complejas, muchos campos |
| Configuración módulos | Admin | Solo administración |
| Reportes director + export PDF | Director | Pantalla grande, impresión |
| Panel entregas completo (tabla 30+ alumnos) | Profesor | Calificar legacy desktop |
| Asignación masiva profesor-curso | Director | Gestión batch |

---

## 10. Código compartido (packages/shared)

Tipos y utilidades reutilizables entre web, móvil y API:

```typescript
// packages/shared/src/roles.ts
export const ROLES = ['admin', 'director', 'teacher', 'student', 'parent'] as const;

// packages/shared/src/submission-status.ts
export type SubmissionStatus = 'pendiente' | 'entregada' | 'entregada_tarde' | 'calificada' | 'devuelta';

// packages/api-client — generado desde specs/Openapi.yml
```

Generar cliente:

```bash
npx openapi-typescript specs/Openapi.yml -o packages/api-client/src/schema.ts
```

---

## 11. Seguridad RBAC

| Endpoint pattern | admin | director | teacher | student | parent |
|-----------------|:-----:|:--------:|:-------:|:-------:|:------:|
| `/admin/*` | ✓ | — | — | — | — |
| `/director/*` | ✓ | ✓ | — | — | — |
| `/assignments POST` | ✓ | — | ✓* | — | — |
| `/submissions POST` | — | — | — | ✓* | — |
| `/parents/dashboard/*` | ✓ | — | — | — | ✓* |
| `/students/{id}/*` | ✓ | ✓ | ✓* | ✓† | ✓‡ |

\* Solo recursos propios/asignados  
† Solo `{id}` = propio  
‡ Solo `{id}` = hijo vinculado

---

## 12. Roadmap frontend

| Fase | Web (Next.js) | Móvil (React Native) |
|------|---------------|----------------------|
| **MVP** | Login, admin usuarios, profesor tareas+panel, padre dashboard, alumno entregas | Login padre/profesor/alumno, dashboard padre, entrega tareas, asistencia profesor, push |
| **v1.1** | Reportes director, calificaciones | Calificaciones, disciplina (consulta padre) |
| **v1.2** | Comunicados, chat web profesor | Chat, firma digital, comunicados |
| **v1.3** | Pagos, encuestas | Pagos QR, offline cache tareas |

---

## 13. Roadmap backend + producto

| Fase | Alcance | Stories |
|------|---------|---------|
| **MVP** | Auth 5 roles, LMS Moodle, panel padre, asistencia, admin usuarios | US-020–029, US-001–006 |
| **v1.1** | Reportes director, calificaciones, disciplina avanzada | US-023–024, US-013 |
| **v1.2** | Comunicados, chat, firma digital | US-007+, US-018 |
| **v1.3** | Pagos, encuestas, offline móvil | Resto |

---

## 14. Referencias

- [Moodle](https://moodle.org/) — referencia UX para tareas y entregas
- Specs: `Userstories.md`, `Models.md`, `Database.md`, `Openapi.yml`
