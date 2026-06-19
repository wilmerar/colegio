# US-T01 — Unit tests para código nuevo

| Épica | Prioridad | Rol |
|-------|-----------|-----|
| Ingeniería | **Must** | Desarrollador |

## Story

**Como** desarrollador del proyecto Colegio  
**Quiero** añadir unit tests al introducir lógica de negocio nueva  
**Para que** el código sea mantenible, seguro y trazable a las user stories

---

## Acceptance criteria

```gherkin
Feature: Unit tests obligatorios en PRs con lógica nueva

  Scenario: Nuevo método en service
    Given añado un método con regla de negocio en AssignmentsService
    When abro un pull request
    Then existe archivo colocado assignments.service.spec.ts
    And hay al menos un test happy path y un test de error o borde
    And los tests pasan con pnpm test:unit

  Scenario: Gherkin cubierto
    Given la story US-025 define entrega tardía
    When implemento submit()
    Then existe test should set entregada_tarde when submitted after due_at

  Scenario: Sin tests triviales
    Given un test que solo assert toBeDefined()
    When se revisa el PR
    Then el revisor rechaza por anti-patrón (Testing.md §2.10)

  Scenario: Mock en frontera correcta
    Given un unit test de SubmissionsService
    When se ejecuta
    Then el repository está mockeado
    And no se conecta a PostgreSQL real
```

---

## Reglas (resumen)

| Cambio | Unit test | Archivo |
|--------|-----------|---------|
| Service / regla negocio | **Sí** | `*.service.spec.ts` |
| Guard RBAC | **Sí** | `*.guard.spec.ts` |
| Validator shared | **Sí** | `packages/shared/*.spec.ts` |
| Controller solo delega | No* | Integration test |

\*El service debe estar cubierto.

## Estructura AAA

```typescript
it('should set status entregada_tarde when submitted after due_at', async () => {
  // Arrange
  const assignment = fixture.assignment({ dueAt: '2026-06-25T23:59:00Z' });
  // Act
  const result = await service.submit(dto, studentUser);
  // Assert
  expect(result.status).toBe('entregada_tarde');
});
```

## Cobertura

| Área | Objetivo |
|------|----------|
| Services | ≥ 80% |
| Guards | 100% branches permisos |

---

## Referencias

- Guía completa: [07-testing.md §2](../07-testing.md)
- Principios: [03-development-principles.md](../03-development-principles.md)
- DoD: [08-definition-of-done.md](../08-definition-of-done.md)

## Definition of Done

- [ ] Cada método nuevo de service/guard tiene tests
- [ ] Happy path + error/borde por método
- [ ] Escenario Gherkin de la story cubierto
- [ ] CI verde; cobertura no baja
