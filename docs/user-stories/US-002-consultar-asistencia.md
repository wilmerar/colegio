# US-002 — Consultar asistencia

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E1 | **Must** | Web + Móvil |

## Story

**Como** padre o alumno  
**Quiero** ver el historial de asistencia  
**Para** hacer seguimiento (padre del hijo, alumno de sí mismo)

## Acceptance criteria (Gherkin)

```gherkin
Feature: Consulta de asistencia
  Scenario: Padre ve historial del hijo
    When consulta asistencia del mes actual
    Then ve resumen presentes, tardanzas, ausencias
  Scenario: Alumno ve su propia asistencia
    Then ve solo sus registros
```

## API — `GET /attendance` · Tests — `attendance/history.feature`
