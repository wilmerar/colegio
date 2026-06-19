# Colegio — App de Comunicación Escuela–Familia

Proyecto **Spec-Driven Development (SDD)** para una aplicación móvil de gestión escolar orientada a colegios en **El Alto, Bolivia** y Latinoamérica.

## Documentación (SDD)

Todos los artefactos de especificación viven en `specs/`:

| Archivo | Descripción |
|---------|-------------|
| [Userstories.md](specs/Userstories.md) | 21 user stories con acceptance criteria en **Gherkin (BDD)** |
| [Architecture.md](specs/Architecture.md) | Arquitectura, stack, módulos y roadmap |
| [Models.md](specs/Models.md) | Entidades de dominio y DTOs |
| [Database.md](specs/Database.md) | Esquema PostgreSQL (DDL) |
| [Openapi.yml](specs/Openapi.yml) | Contrato REST API derivado de user stories |
| [Testing.md](specs/Testing.md) | Estrategia BDD, pirámide de tests, CI |
| [DefinitionOfDone.md](specs/DefinitionOfDone.md) | Criterios de done por story/sprint/release |

## Flujo SDD

```
Userstories.md (Gherkin)
       ↓
Models.md + Database.md + Openapi.yml
       ↓
Implementación + Testing.md
       ↓
DefinitionOfDone.md ✓
```

## Funcionalidades (desde spec PDF)

**Principales (Must):** Asistencia con notificación, agenda de tareas, cuaderno de disciplina, comunicados, calendario, chat seguro, firma digital.

**Secundarias (Should/Could):** Pagos QR, calificaciones, salida segura, galería, directorio, encuestas.

## Posicionamiento vs. competencia

Analizadas: MyEncore, QuickSchools, Classter, Phidias, DocCF, Clickschool, iSAMS.

Este producto **no compite como ERP completo** sino como app de **comunicación diaria** con:
- Chat in-app sin exponer teléfonos personales
- Firma digital simple (adaptada a El Alto)
- Notificaciones push de asistencia/salida en tiempo real
- Modelo SaaS accesible por alumno (objetivo: plan competitivo vs. $1–15 USD/alumno/mes del mercado)

## Próximos pasos

1. Validar user stories Must con stakeholders del colegio piloto
2. Implementar MVP (fase 1 en Architecture.md): auth, asistencia, tareas, disciplina, comunicados, chat, firma
3. Generar proyecto NestJS desde `Openapi.yml` (openapi-generator)
4. Configurar Cucumber con features de `Userstories.md`
