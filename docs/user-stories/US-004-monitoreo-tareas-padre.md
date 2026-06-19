# US-004 — Padre monitorea tareas del hijo

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E2 / E-P | **Must** | Web + Móvil |

## Story

**Como** padre  
**Quiero** ver el estado de las tareas de mi hijo  
**Para** apoyarlo sin entregar por él

## Acceptance criteria (Gherkin)

```gherkin
Feature: Monitoreo de tareas por padre
  Scenario: Ver estados Moodle
    Then ve pendiente, entregada, entregada_tarde, calificada
    And no puede subir entregas
  Scenario: Recordatorio 24h antes de vencer
    Then padre recibe push de tarea pendiente
```

## API — `GET /students/{id}/assignments` · Tests — `assignments/student_view.feature`
