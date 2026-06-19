# User Stories — Índice

> Una user story = un archivo `.md` (como el proyecto Pomodoro de referencia).

## Leyenda

| Prioridad | Significado |
|-----------|-------------|
| **Must** | MVP |
| **Should** | v1.1–v1.2 |
| **Could** | v1.3+ |

## E0 — Auth y usuarios

| ID | Archivo | Título |
|----|---------|--------|
| US-020 | [US-020-login-local-google.md](US-020-login-local-google.md) | Login local + Google OAuth |
| US-021 | [US-021-vincular-padre-alumno.md](US-021-vincular-padre-alumno.md) | Vincular padre ↔ alumno |
| US-022 | [US-022-admin-usuarios-modulos.md](US-022-admin-usuarios-modulos.md) | Admin: usuarios y módulos |

## E-D — Director

| ID | Archivo | Título |
|----|---------|--------|
| US-023 | [US-023-director-asignar-profesores.md](US-023-director-asignar-profesores.md) | Asignar profesores |
| US-024 | [US-024-reportes-profesores.md](US-024-reportes-profesores.md) | Reportes de profesores |

## E2 — LMS Moodle

| ID | Archivo | Título |
|----|---------|--------|
| US-003 | [US-003-publicar-tarea.md](US-003-publicar-tarea.md) | Publicar tarea |
| US-025 | [US-025-entregar-tarea.md](US-025-entregar-tarea.md) | Alumno entrega tarea |
| US-026 | [US-026-calificar-tarea.md](US-026-calificar-tarea.md) | Calificar + feedback |
| US-027 | [US-027-panel-entregas.md](US-027-panel-entregas.md) | Panel entregas profesor |
| US-004 | [US-004-monitoreo-tareas-padre.md](US-004-monitoreo-tareas-padre.md) | Padre monitorea tareas |
| US-028 | [US-028-tareas-alumno.md](US-028-tareas-alumno.md) | Vista tareas alumno |

## E-P — Panel parental

| ID | Archivo | Título |
|----|---------|--------|
| US-029 | [US-029-panel-control-padre.md](US-029-panel-control-padre.md) | Dashboard del hijo |

## E1 — Asistencia

| ID | Archivo | Título |
|----|---------|--------|
| US-001 | [US-001-registrar-asistencia.md](US-001-registrar-asistencia.md) | Registrar asistencia |
| US-002 | [US-002-consultar-asistencia.md](US-002-consultar-asistencia.md) | Consultar asistencia |

## E3 — Disciplina

| ID | Archivo | Título |
|----|---------|--------|
| US-005 | [US-005-llamada-atencion.md](US-005-llamada-atencion.md) | Llamada de atención |
| US-006 | [US-006-felicitacion.md](US-006-felicitacion.md) | Felicitación |

## Should / Could (fases posteriores)

| ID | Archivo | Título |
|----|---------|--------|
| US-007 | [US-007-comunicados-masivos.md](US-007-comunicados-masivos.md) | Comunicados masivos |
| US-008 | [US-008-leer-comunicados.md](US-008-leer-comunicados.md) | Leer comunicados |
| US-013 | [US-013-boletin-calificaciones.md](US-013-boletin-calificaciones.md) | Boletín calificaciones |
| US-018 | [US-018-firma-excursion.md](US-018-firma-excursion.md) | Firma excursión |
| US-019 | [US-019-firma-disciplina.md](US-019-firma-disciplina.md) | Firma acuse disciplina |

## Ingeniería / proceso (T)

| ID | Archivo | Título |
|----|---------|--------|
| US-T01 | [US-T01-unit-tests-codigo-nuevo.md](US-T01-unit-tests-codigo-nuevo.md) | Unit tests para código nuevo |
| US-T02 | [US-T02-pre-pr-checklist.md](US-T02-pre-pr-checklist.md) | Checklist Pre-PR |

## Trazabilidad API

Ver tabla completa en [specs/Userstories.md](../specs/Userstories.md) (legacy) o en cada archivo US-XXX.

## Plantilla para nuevas stories

```markdown
# US-XXX — Título

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|

## Story
Como / Quiero / Para

## Acceptance criteria (Gherkin)

## API endpoints

## Tests
- Unit: ...
- Integration: ...

## Definition of Done
- [ ] ...
```
