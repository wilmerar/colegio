# US-029 — Panel de control del padre

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E-P | **Must** | **Móvil** + Web |

## Story

**Como** padre/madre  
**Quiero** un panel con resumen de asistencia, tareas, calificaciones y disciplina  
**Para** controlar el progreso de mi hijo en un solo lugar

---

## Acceptance criteria (Gherkin)

```gherkin
Feature: Panel de control parental
  Scenario: Dashboard del hijo
    Given padre vinculado a "Juan Pérez"
    When GET /parents/dashboard/{studentId}
    Then ve resumen asistencia, tareas, calificaciones, disciplina
    And puede navegar al detalle de cada sección

  Scenario: Solo hijos vinculados
    When consulta dashboard de alumno ajeno
    Then 403
```

---

## API

- `GET /parents/dashboard/{studentId}`

---

## Tests

| Tipo | Archivo |
|------|---------|
| Unit | `parent-dashboard.service.spec.ts` |
| BDD/E2E | `features/parent/dashboard.feature` |

---

## DoD

- [ ] US-T01 + US-T02
