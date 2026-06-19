# Definition of Done (DoD)

> Criterios que debe cumplir **cada user story** antes de considerarse terminada. Aplica a desarrollo, QA y despliegue.

---

## 1. DoD por User Story

Una user story está **Done** cuando se cumplen **todos** los ítems:

### 1.1 Especificación
- [ ] La user story existe en `Userstories.md` con formato *Como / Quiero / Para*.
- [ ] Tiene al menos un escenario Gherkin con `Given / When / Then`.
- [ ] Está trazada a endpoints en `Openapi.yml` y entidades en `Models.md` / `Database.md`.

### 1.2 Implementación
- [ ] Código implementado siguiendo módulos de `Architecture.md`.
- [ ] Endpoint(s) documentados en OpenAPI coinciden con la implementación.
- [ ] Validaciones de negocio de los escenarios Gherkin están en el service layer.
- [ ] RBAC aplicado: padre/docente/admin solo acceden lo permitido.
- [ ] `school_id` (multi-tenant) verificado en todas las queries.

### 1.3 Pruebas
- [ ] Tests unitarios para reglas de negocio (ver `Testing.md`).
- [ ] Test de integración por endpoint (happy path + 401/403/400).
- [ ] Escenario(s) Gherkin ejecutables en verde (Cucumber).
- [ ] Sin regresiones: suite CI completa en verde.

### 1.4 Calidad
- [ ] Sin errores de linter ni type-check.
- [ ] Sin secretos hardcodeados.
- [ ] Logs estructurados en operaciones críticas (sin PII sensible).
- [ ] Revisión de código aprobada (1 reviewer mínimo).

### 1.5 UX / Producto (si aplica UI)
- [ ] Flujo probado en dispositivo móvil (Android mínimo).
- [ ] Mensajes de error en español, claros para el usuario.
- [ ] Notificaciones push funcionando en staging (si la story las dispara).

### 1.6 Despliegue
- [ ] Migración de BD incluida y probada (up/down).
- [ ] Desplegado en entorno staging.
- [ ] Product Owner acepta el escenario Gherkin en demo.

---

## 2. DoD por Sprint / Incremento

Un incremento (sprint) está **Done** cuando:

- [ ] Todas las stories comprometidas cumplen DoD individual.
- [ ] `Openapi.yml` actualizado y validado (Spectral / openapi-cli).
- [ ] Documentación de API publicada (Swagger UI o Redoc).
- [ ] Changelog de sprint registrado.
- [ ] Demo realizada con datos de colegio demo (`san-miguel`).
- [ ] Deuda técnica documentada en backlog (si se difirió algo).

---

## 3. DoD por Release (MVP / v1.x)

Una release está **Done** cuando:

### Funcional
- [ ] Todas las stories de la fase (ver `Architecture.md` roadmap) completadas.
- [ ] Flujos E2E críticos pasan (ver `Testing.md` sección 8).
- [ ] Checklist manual pre-release completado.

### No funcional
- [ ] HTTPS en todos los entornos productivos.
- [ ] Backup automático de PostgreSQL configurado.
- [ ] Rate limiting activo en API.
- [ ] p95 latencia < 500ms en staging con carga simulada (100 usuarios concurrentes).
- [ ] Política de privacidad y términos publicados (datos de menores).

### Operaciones
- [ ] Monitoreo básico (uptime + errores 5xx).
- [ ] Runbook de incidentes documentado.
- [ ] Plan de rollback definido.

---

## 4. DoD específico por épica

### E1 — Asistencia
- [ ] Cálculo automático tarde vs. horario colegio.
- [ ] Push al padre en < 30 segundos desde marcado.
- [ ] Resumen mensual correcto (US-002).

### E6 — Chat
- [ ] Ningún endpoint expone teléfono de usuarios.
- [ ] Auditoría admin requiere campo `reason` (US-011).

### E13 — Firma digital
- [ ] Imagen de firma almacenada con timestamp e IP/dispositivo.
- [ ] Estados: pendiente → firmado | rechazado.
- [ ] Retención 5 años configurada en storage policy.

### E7 — Pagos
- [ ] Referencia única por pago.
- [ ] QR generado es escaneable (validado manualmente).
- [ ] Conciliación manual por tesorero funcional en MVP.

---

## 5. Definición de "No Done" (anti-patrones)

Estos casos **no** cumplen DoD aunque "funcione en dev":

- Endpoint sin test de integración.
- Escenario Gherkin sin step definitions ejecutables.
- Padre puede ver datos de alumno no vinculado.
- Notificación push no probada en dispositivo real.
- Migración SQL sin probar en BD limpia.
- OpenAPI desactualizado respecto al código.

---

## 6. Roles y responsabilidades

| Rol | Responsabilidad en DoD |
|-----|------------------------|
| Developer | Implementación + tests unit/integration |
| QA | BDD scenarios en verde + checklist manual |
| Product Owner | Aceptación Gherkin en demo |
| DevOps | Despliegue staging/prod + backups |
| Tech Lead | Revisión arquitectura + OpenAPI sync |

---

## 7. Métricas de cumplimiento

| Métrica | Meta |
|---------|------|
| Stories con DoD completo al cierre de sprint | 100% |
| Escenarios Gherkin automatizados (Must) | 100% |
| Endpoints con contrato OpenAPI | 100% |
| Bugs críticos post-release (7 días) | 0 |

---

## 8. Plantilla de cierre de story (PR description)

```markdown
## User Story
US-XXX: [Título]

## Gherkin verificado
- [ ] Escenario 1: [nombre]
- [ ] Escenario 2: [nombre]

## Checklist DoD
- [ ] OpenAPI actualizado
- [ ] Tests unit + integration
- [ ] Cucumber feature en verde
- [ ] RBAC verificado
- [ ] Demo en staging

## Screenshots / evidencia
(adjuntar si hay UI)
```
