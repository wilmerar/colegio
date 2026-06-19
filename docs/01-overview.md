# 01 — Overview

## Producto

Plataforma **web + móvil** para colegios (El Alto, Bolivia):

1. **Control del alumno por padres** — asistencia, tareas, notas, disciplina.
2. **Tareas estilo [Moodle](https://moodle.org/)** — publicar → entregar → calificar → feedback.

## Stack

| Capa | Tecnología |
|------|------------|
| Backend | Node.js + NestJS + TypeScript |
| Web | Next.js 14 + React |
| Móvil | React Native + Expo |
| BD | **PostgreSQL 16** + Redis + S3 |
| Auth | JWT + Google OAuth 2.0 |

## Roles (4 niveles)

| Rol | Web | Móvil |
|-----|:---:|:-----:|
| Administrador | ✓ | — |
| Director | ✓ | — |
| Profesor | ✓ | ✓ |
| Alumno / Padre | ✓ | ✓ |

## Épicas

| ID | Épica | Prioridad |
|----|-------|-----------|
| E0 | Auth, usuarios | Must |
| E1 | Asistencia | Must |
| E2 | LMS Moodle | Must |
| E-P | Panel parental | Must |
| E-D | Director / reportes | Must |
| E3 | Disciplina | Must |
| E4–E13 | Comunicados, pagos, firma… | Should / Could |

Ver índice completo: [user-stories/README.md](user-stories/README.md)

## Documentación relacionada

- [02-architecture.md](02-architecture.md)
- [03-development-principles.md](03-development-principles.md)
- [07-testing.md](07-testing.md)
