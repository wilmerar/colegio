# US-013 — Boletín de calificaciones

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E8 | **Should** | Web + Móvil |

## Story

**Como** profesor/padre  
**Quiero** publicar y consultar notas parciales  
**Para** seguimiento académico antes del cierre de trimestre

## Acceptance criteria (Gherkin)

```gherkin
Feature: Boletín de calificaciones
  Scenario: Padre consulta notas
    When abre calificaciones periodo "Parcial 1"
    Then ve notas por materia y promedio
```

## API — `GET /students/{id}/grades`, `POST /grades`
