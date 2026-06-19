# US-025 — Entregar tarea (alumno)

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E2 | **Must** | Web + Móvil (cámara) |

## Story

**Como** alumno  
**Quiero** subir mi entrega antes o después de la fecha límite  
**Para** que el profesor pueda revisarla y calificarla

---

## Acceptance criteria (Gherkin)

```gherkin
Feature: Entrega de tarea por alumno
  Scenario: Entrega a tiempo
    When entrega archivo antes de due_at
    Then status "entregada"
    And profesor recibe notificación

  Scenario: Entrega tardía
    When entrega después de due_at
    Then status "entregada_tarde"

  Scenario: Padre no puede entregar
    Given usuario rol parent
    When POST /assignments/{id}/submissions
    Then 403 PARENT_CANNOT_SUBMIT
```

---

## API

- `POST /assignments/{assignmentId}/submissions`

---

## Tests

| Tipo | Archivo |
|------|---------|
| Unit | `submissions.service.spec.ts`, `submissions.guard.spec.ts` |
| Integration | `features/submissions/submit.feature` |

---

## DoD

- [ ] US-T01 + US-T02
