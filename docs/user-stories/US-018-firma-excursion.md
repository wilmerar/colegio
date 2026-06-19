# US-018 — Firma autorización de excursión

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E13 | **Should** | **Móvil** + Web |

## Story

**Como** padre  
**Quiero** firmar digitalmente una autorización en pantalla  
**Para** no ir físicamente al colegio

## Acceptance criteria (Gherkin)

```gherkin
Feature: Firma digital de autorizaciones
  Scenario: Firmar excursión
    When dibuja firma y confirma
    Then documento estado "firmado"
    And timestamp e imagen guardados
```

## API — `POST /signatures/requests/{id}/respond`
