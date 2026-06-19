# US-028 — Vista tareas del alumno

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E2 | **Must** | Web + Móvil |

## Story

**Como** alumno  
**Quiero** ver mis tareas por estado  
**Para** gestionar mis entregas

## Acceptance criteria (Gherkin)

```gherkin
Feature: Vista de tareas del alumno
  Scenario: Pestañas por estado
    Then ve pendientes, entregadas, calificadas
    And cuenta regresiva en pendientes
```

## API — `GET /students/{studentId}/assignments`
