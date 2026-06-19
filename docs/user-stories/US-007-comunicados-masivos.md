# US-007 — Comunicados masivos

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E4 | **Should** | Web |

## Story

**Como** director/admin  
**Quiero** enviar comunicados oficiales  
**Para** informar suspensiones, feriados u avisos importantes

## Acceptance criteria (Gherkin)

```gherkin
Feature: Comunicados masivos
  Scenario: Circular a todo el colegio
    When publica audiencia "todo_el_colegio"
    Then todos los roles reciben notificación
```

## API — `POST /announcements`
