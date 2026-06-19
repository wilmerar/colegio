# US-026 — Calificar tarea + feedback

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E2 | **Must** | Web + Móvil |

## Story

**Como** profesor  
**Quiero** calificar entregas y dar retroalimentación  
**Para** cerrar el ciclo de la tarea

## Acceptance criteria (Gherkin)

```gherkin
Feature: Calificación estilo Moodle
  Scenario: Calificar con nota y comentario
    When asigna nota "85" y feedback
    Then status "calificada"
    And padre recibe notificación
```

## API — `PATCH /submissions/{submissionId}/grade`
