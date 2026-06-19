# US-021 — Vincular padre con alumno

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E0 | **Must** | Web (admin) |

## Story

**Como** administrador  
**Quiero** vincular cuentas de padres con alumnos  
**Para** que cada padre monitoree únicamente a sus hijos

## Acceptance criteria (Gherkin)

```gherkin
Feature: Vinculación padre-alumno
  Scenario: Vincular padre existente
    When vincula padre "Carlos Pérez" con alumno "Juan Pérez"
    Then padre ve panel de Juan solamente
```

## API — `POST /students/{studentId}/guardians`
