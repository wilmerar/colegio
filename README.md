# Colegio — Control Escolar Web + Móvil

Plataforma para colegios en El Alto, centrada en **control parental** y **tareas estilo [Moodle](https://moodle.org/)**.

## Stack oficial

| Capa | Tecnología |
|------|------------|
| Backend | **Node.js + NestJS** + TypeScript |
| Web | **Next.js 14** + TypeScript |
| Móvil | **React Native + Expo** + TypeScript |
| API | OpenAPI → tipos compartidos |

## Roles y plataforma

| Rol | Web | Móvil |
|-----|:---:|:-----:|
| Administrador | ✓ | — |
| Director | ✓ | — |
| Profesor | ✓ | ✓ |
| Alumno | ✓ | ✓ |
| Padre | ✓ | ✓ (principal) |

Detalle de pantallas por rol: [Architecture.md §9](specs/Architecture.md#9-matriz-de-pantallas-por-rol-y-plataforma)

## Documentación SDD

| Archivo | Contenido |
|---------|-----------|
| [Userstories.md](specs/Userstories.md) | User stories + Gherkin |
| [Architecture.md](specs/Architecture.md) | Stack, pantallas web/móvil, monorepo |
| [Models.md](specs/Models.md) | Entidades y DTOs |
| [Database.md](specs/Database.md) | PostgreSQL DDL |
| [Openapi.yml](specs/Openapi.yml) | API REST v0.2 |
| [Testing.md](specs/Testing.md) | BDD y CI |
| [DefinitionOfDone.md](specs/DefinitionOfDone.md) | Criterios de done |
| [DevelopmentPrinciples.md](specs/DevelopmentPrinciples.md) | KISS, DRY, SOLID, YAGNI, clean code, design patterns |
| [pull_request_template.md](.github/pull_request_template.md) | Checklist Pre-PR para GitHub |

## Estructura de repo (propuesta)

```
colegio/
├── apps/api/       # NestJS
├── apps/web/       # Next.js
├── apps/mobile/    # React Native + Expo
├── packages/shared/
└── specs/
```

## MVP

- Auth 5 roles
- LMS Moodle (publicar → entregar → calificar)
- Panel de control parental (móvil first)
- Asistencia con push
- Admin usuarios (solo web)
