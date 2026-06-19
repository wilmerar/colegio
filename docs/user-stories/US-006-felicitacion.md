# US-006 — Felicitación

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E3 | **Must** | Web + Móvil |

## Story

**Como** profesor  
**Quiero** registrar felicitaciones  
**Para** que el padre vea refuerzo positivo

## Acceptance criteria (Gherkin)

```gherkin
Feature: Felicitaciones
  Scenario: Registrar felicitación
    Then aparece como tipo "felicitacion"
    And padre recibe notificación positiva
```

## API — `POST /discipline` (type=felicitacion)
