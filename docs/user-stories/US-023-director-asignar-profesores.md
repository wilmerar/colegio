# US-023 — Director asigna profesores

| Épica | Prioridad | Plataforma |
|-------|-----------|------------|
| E-D | **Must** | Web |

## Story

**Como** director  
**Quiero** asignar profesores a cursos y materias  
**Para** organizar la carga académica

## Acceptance criteria (Gherkin)

```gherkin
Feature: Gestión de profesores por director
  Scenario: Asignar profesor a curso
    When asigna "María López" a "3ro A" materia "Matemáticas"
    Then puede publicar tareas de esa materia
  Scenario: Director no crea admins
    Then 403
```

## API — `POST /director/teachers/assignments`
