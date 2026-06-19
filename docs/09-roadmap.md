# 09 — Roadmap

Extraído de [02-architecture.md](02-architecture.md).

## Backend + producto

| Fase | Alcance | User stories |
|------|---------|--------------|
| **MVP** | Auth local + Google, LMS, panel padre, asistencia, admin | US-001–006, US-020–029, US-T01 |
| **v1.1** | Reportes director, calificaciones | US-013, US-023–024 |
| **v1.2** | Comunicados, chat, firma digital | US-007–008, US-018–019 |
| **v1.3** | Pagos, encuestas, offline móvil | E7, E12+ |

## Frontend

| Fase | Web | Móvil |
|------|-----|-------|
| **MVP** | Admin, profesor tareas, padre dashboard | Padre/profesor/alumno, push, entregas |
| **v1.1** | Reportes director | Calificaciones padre |
| **v1.2** | Comunicados, chat | Firma digital, chat |
| **v1.3** | Pagos | QR, cache offline |

## Prioridad de implementación sugerida

```
E0 (auth) → E2 (LMS) → E-P (dashboard) → E1 (asistencia) → E3 (disciplina)
         → E-D (director) → E4+ (comunicados, firma, pagos…)
```

Guías transversales desde el inicio: **US-T01** (unit tests), **US-T02** (Pre-PR).
