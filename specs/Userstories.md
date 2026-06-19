# User Stories — Aplicación de Colegios (El Alto)

> **Producto:** App móvil y portal web para comunicación escuela–familia, enfocada en uso diario por padres, docentes y dirección.
>
> **Referencia competitiva:** MyEncore, QuickSchools, Classter, Phidias, DocCF, Clickschool, iSAMS. Este producto se diferencia por simplicidad, chat seguro sin exponer teléfonos, firma digital local y notificaciones en tiempo real de asistencia/salida.

---

## Épicas

| ID | Épica | Prioridad |
|----|-------|-----------|
| E1 | Asistencia y puntualidad | Must |
| E2 | Agenda digital de tareas | Must |
| E3 | Cuaderno de disciplina | Must |
| E4 | Comunicados y circulares | Must |
| E5 | Calendario escolar | Must |
| E6 | Mensajería directa (chat seguro) | Must |
| E7 | Control de pagos | Should |
| E8 | Boletín de calificaciones | Should |
| E9 | Salida segura | Should |
| E10 | Galería de actividades | Could |
| E11 | Directorio escolar | Could |
| E12 | Encuestas y votaciones | Could |
| E13 | Firma digital (El Alto) | Must |

---

## E1 — Asistencia y Puntualidad

### US-001: Registrar entrada del alumno

**Como** docente  
**Quiero** marcar la entrada de cada alumno al inicio de clase  
**Para que** quede constancia oficial y el padre sea notificado automáticamente

**Prioridad:** Must | **Épica:** E1

```gherkin
Feature: Registro de entrada de alumno
  Como docente
  Quiero marcar la asistencia de entrada de mis alumnos
  Para que el sistema notifique al padre en tiempo real

  Background:
    Given el docente "María López" está autenticada
    And tiene asignado el curso "3ro A" del colegio "San Miguel"
    And el alumno "Juan Pérez" está matriculado en "3ro A"
    And el padre "Carlos Pérez" tiene la app instalada y notificaciones activas

  Scenario: Entrada a tiempo
    When la docente marca entrada de "Juan Pérez" a las "07:55"
    Then el registro queda con estado "presente"
    And el padre recibe notificación push "Juan ingresó al colegio a las 07:55"

  Scenario: Entrada tardía
    When la docente marca entrada de "Juan Pérez" a las "08:15"
    And el horario de entrada del colegio es "08:00"
    Then el registro queda con estado "tarde"
    And el padre recibe notificación push "Juan llegó tarde a las 08:15"

  Scenario: Falta sin justificación
    When la docente marca "Juan Pérez" como ausente
    Then el registro queda con estado "ausente"
    And el padre recibe notificación push "Juan no asistió hoy"
```

---

### US-002: Consultar historial de asistencia (padre)

**Como** padre/madre  
**Quiero** ver el historial de asistencia de mi hijo  
**Para que** pueda hacer seguimiento de puntualidad y faltas

**Prioridad:** Must | **Épica:** E1

```gherkin
Feature: Historial de asistencia para padres
  Como padre
  Quiero consultar la asistencia de mi hijo
  Para hacer seguimiento de su puntualidad

  Background:
    Given el padre "Carlos Pérez" está autenticado
    And está vinculado al alumno "Juan Pérez" del colegio "San Miguel"

  Scenario: Ver resumen mensual
    When el padre consulta asistencia del mes actual
    Then ve un resumen con días presentes, tardanzas y ausencias
    And puede filtrar por rango de fechas

  Scenario: Detalle de un día específico
    Given existe un registro de "tarde" el "2026-06-10"
    When el padre abre el detalle del "2026-06-10"
    Then ve la hora de marcado y el estado "tarde"
```

---

## E2 — Agenda Digital de Tareas

### US-003: Publicar tarea con adjuntos

**Como** docente  
**Quiero** publicar tareas con fecha de entrega y archivos adjuntos  
**Para que** los padres y alumnos tengan claro qué deben hacer

**Prioridad:** Must | **Épica:** E2

```gherkin
Feature: Publicación de tareas
  Como docente
  Quiero crear tareas con descripción, fecha límite y adjuntos
  Para que las familias las consulten desde la app

  Background:
    Given el docente "María López" está autenticada en el curso "3ro A"

  Scenario: Crear tarea con foto adjunta
    When publica la tarea "Ejercicios de fracciones" con entrega "2026-06-25"
    And adjunta una imagen "hoja_ejercicios.jpg"
    Then la tarea aparece en la agenda del curso "3ro A"
    And los padres del curso reciben notificación de nueva tarea

  Scenario: Validar fecha de entrega obligatoria
    When intenta publicar una tarea sin fecha de entrega
    Then el sistema muestra error "La fecha de entrega es obligatoria"
    And la tarea no se publica
```

---

### US-004: Consultar agenda de tareas (padre)

**Como** padre/madre  
**Quiero** ver las tareas pendientes de mi hijo ordenadas por fecha  
**Para que** pueda apoyarlo a cumplir con los deberes

**Prioridad:** Must | **Épica:** E2

```gherkin
Feature: Consulta de agenda por padre
  Como padre
  Quiero ver las tareas pendientes de mi hijo
  Para apoyarlo en cumplir con los deberes

  Scenario: Listar tareas pendientes
    Given existen 2 tareas pendientes y 1 vencida para "Juan Pérez"
    When el padre abre la agenda
    Then ve las tareas ordenadas por fecha de entrega ascendente
    And las tareas vencidas se muestran destacadas

  Scenario: Descargar adjunto de tarea
    Given la tarea "Ejercicios de fracciones" tiene adjunto
    When el padre abre el detalle de la tarea
    Then puede descargar o visualizar el archivo adjunto
```

---

## E3 — Cuaderno de Disciplina

### US-005: Registrar llamada de atención

**Como** docente  
**Quiero** registrar incidentes conductuales con descripción y gravedad  
**Para que** quede trazabilidad y el padre sea informado

**Prioridad:** Must | **Épica:** E3

```gherkin
Feature: Registro de llamadas de atención
  Como docente
  Quiero registrar incidentes conductuales
  Para informar al padre y mantener historial disciplinario

  Scenario: Crear llamada de atención
    When la docente registra incidente "Interrumpe clase repetidamente" con gravedad "media" para "Juan Pérez"
    Then el incidente queda en el cuaderno de disciplina
    And el padre recibe notificación con resumen del incidente

  Scenario: Gravedad inválida
    When intenta registrar con gravedad "extrema"
    Then el sistema rechaza con error "Gravedad debe ser: leve, media o grave"
```

---

### US-006: Registrar felicitación (refuerzo positivo)

**Como** docente  
**Quiero** registrar felicitaciones por buen comportamiento  
**Para que** el padre también vea el refuerzo positivo del hijo

**Prioridad:** Must | **Épica:** E3

```gherkin
Feature: Felicitaciones en cuaderno de disciplina
  Como docente
  Quiero registrar felicitaciones
  Para reconocer conductas positivas ante los padres

  Scenario: Registrar felicitación
    When la docente registra felicitación "Ayudó a un compañero en matemáticas" para "Juan Pérez"
    Then el registro aparece como tipo "felicitación" en el cuaderno
    And el padre recibe notificación positiva
```

---

## E4 — Comunicados y Circulares

### US-007: Enviar comunicado masivo (dirección)

**Como** director/a o administrador/a  
**Quiero** enviar comunicados oficiales a padres, docentes o todo el colegio  
**Para que** informar suspensiones, feriados u otros avisos importantes

**Prioridad:** Must | **Épica:** E4

```gherkin
Feature: Comunicados masivos
  Como director
  Quiero enviar circulares oficiales
  Para informar a toda la comunidad escolar

  Scenario: Comunicado a todo el colegio
    When publica comunicado "Suspensión de clases por feriado" para audiencia "todo_el_colegio"
    Then todos los padres y docentes activos reciben notificación
    And el comunicado queda archivado en el historial

  Scenario: Comunicado solo a un curso
    When publica comunicado "Reunión de padres 3ro A" para audiencia "curso:3ro A"
    Then solo los padres de "3ro A" reciben la notificación
```

---

### US-008: Leer comunicados (padre)

**Como** padre/madre  
**Quiero** ver los comunicados oficiales del colegio  
**Para que** no me pierda avisos importantes

**Prioridad:** Must | **Épica:** E4

```gherkin
Feature: Lectura de comunicados
  Como padre
  Quiero ver comunicados del colegio
  Para estar informado de avisos oficiales

  Scenario: Marcar comunicado como leído
    Given existe un comunicado no leído para el padre
    When abre el detalle del comunicado
    Then el comunicado se marca como leído
    And desaparece del contador de no leídos
```

---

## E5 — Calendario de Exámenes y Actividades

### US-009: Publicar evento en calendario escolar

**Como** docente o administrador  
**Quiero** agregar exámenes, ferias y reuniones al calendario  
**Para que** las familias planifiquen con anticipación

**Prioridad:** Must | **Épica:** E5

```gherkin
Feature: Calendario escolar
  Como docente
  Quiero publicar eventos en el calendario
  Para que padres y alumnos conozcan fechas importantes

  Scenario: Crear examen en calendario
    When crea evento tipo "examen" titulo "Examen de Ciencias" fecha "2026-07-02" curso "3ro A"
    Then el evento aparece en el calendario del curso
    And los padres de "3ro A" reciben recordatorio 24 horas antes

  Scenario: Vista mensual del calendario
    When el padre abre el calendario de junio 2026
    Then ve todos los eventos del hijo en vista mensual
```

---

## E6 — Mensajería Directa (Chat Seguro)

### US-010: Iniciar chat docente–padre

**Como** padre o docente  
**Quiero** enviar mensajes directos sin compartir mi número personal  
**Para que** la comunicación sea privada y trazable dentro de la app

**Prioridad:** Must | **Épica:** E6

```gherkin
Feature: Chat seguro docente-padre
  Como padre
  Quiero chatear con el docente de mi hijo
  Para resolver dudas sin dar mi número de celular

  Background:
    Given el padre "Carlos Pérez" está vinculado al alumno en curso "3ro A"
    And la docente "María López" es docente de "3ro A"

  Scenario: Enviar mensaje de texto
    When el padre envía "¿Habrá tarea para el viernes?"
    Then la docente recibe el mensaje en la app
    And el número personal de ninguna parte queda expuesto

  Scenario: Bloquear contacto fuera de relación escolar
    Given un padre no vinculado al curso del docente
    When intenta iniciar chat con la docente
    Then el sistema deniega con error "No tiene permiso para contactar a este docente"
```

---

### US-011: Moderación y archivo de conversaciones

**Como** administrador del colegio  
**Quiero** que las conversaciones queden registradas y sean auditables  
**Para que** haya respaldo en caso de conflictos

**Prioridad:** Should | **Épica:** E6

```gherkin
Feature: Auditoría de mensajes
  Como administrador
  Quiero acceder al historial de chats bajo política de privacidad
  Para resolver disputas cuando sea necesario

  Scenario: Consulta auditada por administrador
    Given existe una conversación entre padre y docente
    When el administrador solicita auditoría con motivo "reporte formal #123"
    Then puede ver el historial completo
    And queda registro de quién accedió y cuándo
```

---

## E7 — Control de Pagos

### US-012: Generar recordatorio de pensión con QR

**Como** tesorero/a o administrador/a  
**Quiero** generar recordatorios de mensualidad con código QR de pago  
**Para que** los padres paguen rápido desde la app

**Prioridad:** Should | **Épica:** E7

```gherkin
Feature: Recordatorio de pagos con QR
  Como tesorero
  Quiero enviar recordatorios de pensión con QR
  Para facilitar el cobro de cuotas escolares

  Scenario: Generar QR de pago
    Given el alumno "Juan Pérez" tiene pensión pendiente de "350 BOB" mes "Junio 2026"
    When el tesorero genera recordatorio de pago
    Then el padre recibe notificación con monto y QR escaneable
    And el QR contiene referencia única de pago "PAY-2026-06-JP001"

  Scenario: Marcar pago como recibido
    When el tesorero confirma pago recibido para referencia "PAY-2026-06-JP001"
    Then el estado del cargo cambia a "pagado"
    And el padre ve el comprobante en historial de pagos
```

---

## E8 — Boletín de Calificaciones

### US-013: Publicar notas parciales

**Como** docente  
**Quiero** publicar calificaciones parciales por materia  
**Para que** el padre haga seguimiento antes del cierre de trimestre

**Prioridad:** Should | **Épica:** E8

```gherkin
Feature: Boletín de calificaciones
  Como docente
  Quiero publicar notas parciales
  Para que los padres vean el avance académico

  Scenario: Publicar nota de una materia
    When publica calificación "Matemáticas" nota "78" periodo "Parcial 1" para "Juan Pérez"
    Then el padre puede ver la nota en el boletín parcial
    And recibe notificación de nueva calificación publicada

  Scenario: Padre consulta boletín completo
    Given existen notas publicadas en 5 materias
    When el padre abre el boletín del periodo "Parcial 1"
    Then ve todas las materias con sus notas y promedio
```

---

## E9 — Salida Segura

### US-014: Marcar salida del alumno

**Como** docente o personal de portería  
**Quiero** marcar cuando el alumno sale del colegio  
**Para que** el padre esté alerta de la hora de salida

**Prioridad:** Should | **Épica:** E9

```gherkin
Feature: Salida segura
  Como docente
  Quiero marcar la salida del alumno
  Para notificar al padre que ya salió del colegio

  Scenario: Salida normal
    When marca salida de "Juan Pérez" a las "12:30"
    Then el padre recibe push "Juan salió del colegio a las 12:30"
    And el registro queda vinculado al registro de entrada del mismo día

  Scenario: Salida sin entrada previa
    When intenta marcar salida sin entrada registrada
    Then el sistema advierte "No hay entrada registrada hoy" y permite confirmar salida
```

---

## E10 — Galería de Actividades

### US-015: Subir fotos de actividades del curso

**Como** docente  
**Quiero** subir fotos de proyectos y eventos del aula  
**Para que** solo los padres de ese curso las vean de forma privada

**Prioridad:** Could | **Épica:** E10

```gherkin
Feature: Galería privada por curso
  Como docente
  Quiero publicar fotos de actividades
  Para compartir momentos del aula con las familias

  Scenario: Subir álbum de actividad
    When sube álbum "Feria de Ciencias 2026" con 5 fotos al curso "3ro A"
    Then solo padres de "3ro A" pueden ver el álbum
    And padres de otros cursos no tienen acceso

  Scenario: Denegar acceso a usuario no autorizado
    Given un padre de "4to B"
    When intenta ver álbum de "3ro A"
    Then recibe error 403 "Acceso denegado"
```

---

## E11 — Directorio Escolar

### US-016: Consultar directorio de emergencias

**Como** padre  
**Quiero** acceder rápido a contactos de emergencia del colegio  
**Para que** sepa a quién llamar en una urgencia

**Prioridad:** Could | **Épica:** E11

```gherkin
Feature: Directorio escolar
  Como padre
  Quiero ver contactos de emergencia
  Para actuar rápido ante una urgencia

  Scenario: Ver directorio del colegio
    Given el colegio tiene contactos de enfermería, psicopedagogía y centro de salud cercano
    When el padre abre el directorio
    Then ve nombre, teléfono y horario de atención de cada contacto
    And puede iniciar llamada desde la app (tel: link)
```

---

## E12 — Encuestas y Votaciones

### US-017: Crear encuesta para junta escolar

**Como** administrador o delegado de junta  
**Quiero** crear encuestas con opciones de voto  
**Para que** la comunidad decida temas como uniforme de promoción o menú de kermesse

**Prioridad:** Could | **Épica:** E12

```gherkin
Feature: Encuestas y votaciones
  Como administrador
  Quiero crear encuestas con plazo de votación
  Para tomar decisiones participativas

  Scenario: Votar en encuesta activa
    Given existe encuesta "Color uniforme promoción" con opciones "Azul" y "Verde"
    And el padre aún no ha votado
    When vota por "Azul"
    Then su voto queda registrado
    And no puede cambiar el voto

  Scenario: Encuesta cerrada
    Given la encuesta cerró ayer
    When el padre intenta votar
    Then recibe mensaje "La encuesta ha finalizado"
    And ve resultados agregados si la encuesta es pública
```

---

## E13 — Firma Digital (El Alto)

### US-018: Firmar autorización de excursión

**Como** padre/madre  
**Quiero** firmar digitalmente una autorización desde la pantalla  
**Para que** no tenga que ir físicamente al colegio

**Prioridad:** Must | **Épica:** E13

```gherkin
Feature: Firma digital de autorizaciones
  Como padre
  Quiero firmar autorizaciones en pantalla
  Para agilizar permisos escolares

  Scenario: Firmar autorización de excursión
    Given existe solicitud de firma "Autorización visita museo" para "Juan Pérez"
    When el padre dibuja su firma en el canvas y confirma
    Then el documento queda estado "firmado"
    And se guarda imagen de firma, timestamp e IP/dispositivo
    And la docente recibe confirmación de firma completada

  Scenario: Rechazar firma
    When el padre rechaza la autorización con motivo "Conflicto de horario"
    Then el documento queda estado "rechazado"
    And la docente es notificada del rechazo
```

---

### US-019: Firmar acuse de llamada de atención

**Como** padre/madre  
**Quiero** firmar el acuse de recibo de una llamada de atención  
**Para que** quede constancia de que fui informado

**Prioridad:** Must | **Épica:** E13

```gherkin
Feature: Firma de acuse disciplinario
  Como padre
  Quiero firmar el acuse de una llamada de atención
  Para confirmar que fui informado del incidente

  Scenario: Firmar acuse de incidente
    Given existe llamada de atención "Interrumpe clase" pendiente de firma
    When el padre firma el acuse digitalmente
    Then el incidente queda marcado como "acuse firmado"
    And queda registro auditable de la firma
```

---

## E0 — Autenticación y Multi-tenant

### US-020: Inicio de sesión por rol

**Como** usuario (padre, docente, administrador)  
**Quiero** iniciar sesión de forma segura  
**Para que** acceda solo a la información de mi colegio y rol

**Prioridad:** Must | **Épica:** E0

```gherkin
Feature: Autenticación multi-rol
  Como usuario del colegio
  Quiero autenticarme con email y contraseña
  Para acceder según mi rol

  Scenario: Login exitoso de padre
    Given el padre tiene cuenta activa en colegio "San Miguel"
    When inicia sesión con credenciales válidas
    Then recibe token JWT con rol "parent" y tenant "san-miguel"
    And solo ve datos de sus hijos vinculados

  Scenario: Credenciales inválidas
    When inicia sesión con contraseña incorrecta
    Then recibe error 401 "Credenciales inválidas"
    And no se emite token
```

---

### US-021: Vincular padre con alumno

**Como** administrador del colegio  
**Quiero** vincular cuentas de padres con alumnos  
**Para que** cada padre vea únicamente a sus hijos

**Prioridad:** Must | **Épica:** E0

```gherkin
Feature: Vinculación padre-alumno
  Como administrador
  Quiero vincular padres con alumnos
  Para controlar el acceso familiar

  Scenario: Vincular padre existente
    When vincula padre "Carlos Pérez" con alumno "Juan Pérez" relación "padre"
    Then el padre puede ver asistencia, tareas y calificaciones de Juan
    And no puede ver datos de otros alumnos
```

---

## Matriz de trazabilidad (User Story → API)

| User Story | Endpoints principales |
|------------|----------------------|
| US-001, US-002 | `POST/GET /attendance` |
| US-003, US-004 | `POST/GET /assignments` |
| US-005, US-006 | `POST/GET /discipline` |
| US-007, US-008 | `POST/GET /announcements` |
| US-009 | `POST/GET /calendar/events` |
| US-010, US-011 | `POST/GET /messages` |
| US-012 | `POST/GET /payments` |
| US-013 | `POST/GET /grades` |
| US-014 | `POST /attendance/checkout` |
| US-015 | `POST/GET /galleries` |
| US-016 | `GET /directory` |
| US-017 | `POST/GET /polls` |
| US-018, US-019 | `POST/GET /signatures` |
| US-020 | `POST /auth/login` |
| US-021 | `POST /students/{id}/guardians` |
