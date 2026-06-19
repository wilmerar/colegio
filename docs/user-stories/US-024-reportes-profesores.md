# US-024 — Reportes de profesores

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E-D | **Must** | Web |

## Story

**Como** director  
**Quiero** ver reportes de actividad docente  
**Para** supervisar cumplimiento

## Acceptance criteria (Gherkin)

```gherkin
Feature: Reportes de profesores
  Scenario: Reporte de tareas
    Then ve publicadas, entregas, calificadas, pendientes
  Scenario: Export PDF
    Then descarga resumen del periodo
```

## API — `GET /director/reports/teachers/{teacherId}?format=pdf`
