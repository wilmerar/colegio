# US-019 — Firma acuse disciplinario

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E13 | **Should** | Móvil + Web |

## Story

**Como** padre  
**Quiero** firmar el acuse de una llamada de atención  
**Para** confirmar que fui informado

## Acceptance criteria (Gherkin)

```gherkin
Feature: Firma de acuse disciplinario
  Scenario: Firmar acuse
    Then incidente marcado "acuse firmado"
```

## API — `POST /signatures/requests/{id}/respond` (type=acuse_disciplina)
