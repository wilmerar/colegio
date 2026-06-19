# US-027 — Panel de entregas (profesor)

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E2 | **Must** | Web (tabla completa) |

## Story

**Como** profesor  
**Quiero** ver el estado de todas las entregas de una tarea  
**Para** identificar alumnos pendientes

## Acceptance criteria (Gherkin)

```gherkin
Feature: Panel de entregas
  Scenario: Resumen contadores
    Then ve entregadas, tardías, pendientes, calificadas
    And puede filtrar por estado
```

## API — `GET /assignments/{assignmentId}/submissions`
