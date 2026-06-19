# US-008 — Leer comunicados

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E4 | **Should** | Web + Móvil |

## Story

**Como** padre/alumno/profesor  
**Quiero** leer comunicados del colegio  
**Para** no perderme avisos oficiales

## Acceptance criteria (Gherkin)

```gherkin
Feature: Lectura de comunicados
  Scenario: Marcar como leído
    When abre el comunicado
    Then se marca como leído
```

## API — `GET /announcements`, `POST /announcements/{id}/read`
