# US-T02 — Checklist Pre-PR

| Épica | Prioridad | Rol |
|-------|-----------|-----|
| Ingeniería | **Must** | Desarrollador |

## Story

**Como** desarrollador  
**Quiero** seguir un checklist antes de abrir un pull request  
**Para que** cada cambio cumpla SDD, calidad y seguridad

---

## Acceptance criteria

```gherkin
Feature: Pre-PR checklist en GitHub

  Scenario: PR con story trazada
    Given implementé US-025 entregar tarea
    When creo el PR en GitHub
    Then la plantilla .github/pull_request_template.md está rellena
    And el checklist Spec, Code quality, RBAC y Tests está marcado

  Scenario: API actualizada
    Given añadí POST /assignments/{id}/submissions
    When reviso el PR
    Then Openapi.yml (docs/06-api-openapi.yml) está actualizado
    And hay test de integración 201, 401, 403

  Scenario: Principios de desarrollo
    Given el PR introduce abstracción para un solo caso
    When se aplica US-T02
    Then el revisor pide simplificar (KISS/YAGNI)
```

---

## Checklist (copiar al PR)

### Spec & traceability
- [ ] Linked to `docs/user-stories/US-XXX-*.md`
- [ ] `docs/06-api-openapi.yml` updated (if API changed)
- [ ] `docs/04-database.md` / migration (if schema changed)

### Code quality
- [ ] KISS / DRY / YAGNI ([03-development-principles.md](../03-development-principles.md))
- [ ] Lint + type-check pass
- [ ] No secrets, `console.log`, dead code

### Security & RBAC
- [ ] `school_id` from JWT (multi-tenant)
- [ ] Role permissions verified (403 cases)
- [ ] Parent cannot submit assignments

### Tests ([US-T01](US-T01-unit-tests-codigo-nuevo.md))
- [ ] Unit tests for new business rules
- [ ] Integration test for new/changed endpoints
- [ ] CI green; coverage ≥ 80% services

---

## Referencias

- Plantilla GitHub: [`.github/pull_request_template.md`](../../.github/pull_request_template.md)
- DoD completo: [08-definition-of-done.md](../08-definition-of-done.md)

## Definition of Done

- [ ] PR abierto con checklist completo
- [ ] Revisor aprueba DoD + US-T01 tests
