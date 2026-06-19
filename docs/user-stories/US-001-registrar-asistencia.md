# US-001 — Registrar asistencia

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E1 | **Must** | **Móvil** (aula) + Web |

## Story

**Como** profesor  
**Quiero** marcar la asistencia de mis alumnos  
**Para** que quede registro y el padre sea notificado

---

## Acceptance criteria (Gherkin)

```gherkin
Feature: Registro de asistencia
  Scenario: Entrada a tiempo
    When marca entrada a las "07:55"
    Then status "presente"
    And padre recibe push de ingreso

  Scenario: Entrada tardía
    When marca entrada a las "08:15" y horario es "08:00"
    Then status "tarde"
```

---

## API

- `POST /attendance`

---

## Tests

| Tipo | Archivo |
|------|---------|
| Unit | `attendance.service.spec.ts` |
| BDD | `features/attendance/entry.feature` |

---

## DoD

- [ ] US-T01 + US-T02
