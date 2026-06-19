# US-022 — Admin gestiona usuarios y módulos

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E0 | **Must** | Web |

## Story

**Como** administrador  
**Quiero** CRUD usuarios y activar/desactivar módulos  
**Para** controlar acceso al sistema

## Acceptance criteria (Gherkin)

```gherkin
Feature: Gestión de usuarios por administrador
  Scenario: Crear profesor
    When crea rol "teacher"
    Then puede iniciar sesión y ver cursos
  Scenario: Desactivar módulo pagos
    Then endpoints payments retornan 403
```

## API — `CRUD /admin/users`, `PATCH /admin/modules`
