# US-020 — Login local y Google OAuth

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E0 | **Must** | Web + Móvil |

## Story

**Como** usuario del colegio (admin, director, profesor, alumno o padre)  
**Quiero** iniciar sesión con email/contraseña o Google  
**Para** acceder solo a las funciones de mi rol

---

## Acceptance criteria (Gherkin)

```gherkin
Feature: Autenticación multi-rol web y móvil
  Scenario: Login exitoso de padre en móvil
    Given el padre tiene cuenta activa en colegio "San Miguel"
    When inicia sesión con credenciales válidas
    Then recibe token JWT con rol "parent"
    And ve el panel de control de sus hijos

  Scenario: Login con Google ID token
    Given google_enabled en el colegio
    When POST /auth/google con id_token válido
    Then recibe JWT y user_oauth_identities vinculado

  Scenario: Credenciales inválidas
    When inicia sesión con contraseña incorrecta
    Then recibe error 401 "Credenciales inválidas"

  Scenario: Alumno no accede a admin
    When intenta GET /admin/users
    Then recibe 403
```

---

## API

- `POST /auth/login`
- `POST /auth/google`
- `POST /auth/refresh`
- `POST /auth/logout`
- `GET /auth/me`
- `POST/DELETE /auth/link/google`

Ver [06-api-openapi.yml](../06-api-openapi.yml)

---

## Tests

| Tipo | Archivo |
|------|---------|
| Unit | `auth.service.spec.ts`, `google-oauth.provider.spec.ts` |
| Integration | `auth/login.feature`, `auth/google.feature` |

---

## DoD

- [ ] [US-T01](US-T01-unit-tests-codigo-nuevo.md) + [US-T02](US-T02-pre-pr-checklist.md)
