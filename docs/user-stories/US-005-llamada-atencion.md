# US-005 — Llamada de atención

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E3 | **Must** | Web + Móvil |

## Story

**Como** profesor  
**Quiero** registrar incidentes conductuales  
**Para** informar al padre y mantener historial

## Acceptance criteria (Gherkin)

```gherkin
Feature: Registro de incidentes
  Scenario: Crear llamada de atención
    When registra incidente gravedad "media"
    Then padre recibe notificación
    And aparece en panel de control
```

## API — `POST /discipline` · Tests — `discipline/incident.feature`
