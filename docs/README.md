# Documentación — App Colegios

> Estructura SDD inspirada en proyectos con `docs/` numerados + **user stories individuales**.

## Índice principal

| # | Documento | Descripción |
|---|-----------|-------------|
| — | [user-stories/](user-stories/README.md) | **User stories** (una por archivo, Gherkin) |
| 01 | [01-overview.md](01-overview.md) | Visión, roles, stack |
| 02 | [02-architecture.md](02-architecture.md) | Arquitectura web + móvil, integraciones |
| 03 | [03-development-principles.md](03-development-principles.md) | KISS, DRY, SOLID, YAGNI, clean code |
| 04 | [04-database.md](04-database.md) | PostgreSQL, OAuth, DDL |
| 05 | [05-data-model.md](05-data-model.md) | Entidades y DTOs |
| 06 | [06-api-openapi.yml](06-api-openapi.yml) | Contrato REST API |
| 07 | [07-testing.md](07-testing.md) | BDD, unit tests, integración |
| 08 | [08-definition-of-done.md](08-definition-of-done.md) | DoD + plantilla PR |
| 09 | [09-roadmap.md](09-roadmap.md) | Fases MVP → v1.3 |

## Otros

| Recurso | Ubicación |
|---------|-----------|
| Plantilla PR | [`.github/pull_request_template.md`](../.github/pull_request_template.md) |
| Cursor rules | [`.cursor/rules/`](../.cursor/rules/) |
| Legacy specs | [`specs/`](../specs/) (redirige aquí) |

## Convención user stories

```
docs/user-stories/
  README.md              ← índice por épica
  US-001-registrar-asistencia.md
  US-020-login-local-google.md
  US-T01-unit-tests-codigo-nuevo.md   ← guías de ingeniería
```

Formato por archivo: **Story → Gherkin → API → Tests → DoD**.

## Flujo de trabajo

1. Crear o editar story en `docs/user-stories/US-XXX-*.md`
2. Actualizar docs técnicos (`04-database`, `06-api`, …) si aplica
3. Implementar + tests según `07-testing.md`
4. Cerrar con checklist `08-definition-of-done.md`
