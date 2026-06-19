# US-003 — Publicar tarea (Moodle)

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E2 | **Must** | Web (+ móvil simplificado) |

## Story

**Como** profesor  
**Quiero** crear una tarea con instrucciones, adjuntos y fecha límite  
**Para** que los alumnos sepan qué entregar y cuándo

---

## Acceptance criteria (Gherkin)

```gherkin
Feature: Publicación de tarea estilo Moodle
  Background:
    Given el profesor "María López" imparte "Matemáticas" en "3ro A"

  Scenario: Crear tarea con adjunto
    When publica tarea "Ejercicios de fracciones" due_at "2026-06-25 23:59"
    Then la tarea aparece en el aula virtual de "3ro A"
    And alumnos y padres reciben notificación

  Scenario: Fecha límite obligatoria
    When intenta publicar sin due_at
    Then error "La fecha límite es obligatoria"
```

---

## API

- `POST /courses/{courseId}/assignments`

---

## Tests

| Tipo | Archivo |
|------|---------|
| Unit | `assignments.service.spec.ts` |
| BDD | `features/assignments/create.feature` |

---

## DoD

- [ ] US-T01 + US-T02
