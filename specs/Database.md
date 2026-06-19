# Database — Esquema PostgreSQL

> Esquema derivado de `Models.md` y `Userstories.md`. Motor: **PostgreSQL 16**. Encoding: UTF-8. Timezone app: UTC; display: `America/La_Paz`.

---

## 1. Convenciones

| Convención | Valor |
|------------|-------|
| PK | `UUID DEFAULT gen_random_uuid()` |
| Timestamps | `TIMESTAMPTZ NOT NULL DEFAULT now()` |
| Soft delete | No en MVP (usar `is_active`) |
| Naming | snake_case, plural para tablas |
| Multi-tenant | `school_id` en tablas de tenant + índices compuestos |
| Migraciones | Flyway o Prisma Migrate |

---

## 2. Extensiones

```sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";  -- gen_random_uuid()
CREATE EXTENSION IF NOT EXISTS "citext";    -- emails case-insensitive
```

---

## 3. DDL — Tablas core

### 3.1 schools

```sql
CREATE TABLE schools (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug          VARCHAR(64) NOT NULL UNIQUE,
  name          VARCHAR(255) NOT NULL,
  timezone      VARCHAR(64) NOT NULL DEFAULT 'America/La_Paz',
  entry_time    TIME NOT NULL DEFAULT '08:00:00',
  logo_url      TEXT,
  settings      JSONB NOT NULL DEFAULT '{}',
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 3.2 users

```sql
CREATE TYPE user_role AS ENUM ('admin', 'teacher', 'parent', 'treasurer');

CREATE TABLE users (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id     UUID NOT NULL REFERENCES schools(id),
  email         CITEXT NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name    VARCHAR(100) NOT NULL,
  last_name     VARCHAR(100) NOT NULL,
  role          user_role NOT NULL,
  phone         VARCHAR(20),
  fcm_token     TEXT,
  is_active     BOOLEAN NOT NULL DEFAULT true,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (school_id, email)
);

CREATE INDEX idx_users_school_role ON users (school_id, role);
```

### 3.3 courses

```sql
CREATE TABLE courses (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id     UUID NOT NULL REFERENCES schools(id),
  name          VARCHAR(100) NOT NULL,
  grade_level   VARCHAR(10) NOT NULL,
  section       VARCHAR(10) NOT NULL,
  school_year   VARCHAR(9) NOT NULL,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (school_id, name, school_year)
);
```

### 3.4 course_teachers

```sql
CREATE TABLE course_teachers (
  course_id     UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
  teacher_id    UUID NOT NULL REFERENCES users(id),
  subject       VARCHAR(100),
  PRIMARY KEY (course_id, teacher_id)
);
```

### 3.5 students

```sql
CREATE TABLE students (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id       UUID NOT NULL REFERENCES schools(id),
  course_id       UUID NOT NULL REFERENCES courses(id),
  first_name      VARCHAR(100) NOT NULL,
  last_name       VARCHAR(100) NOT NULL,
  enrollment_code VARCHAR(32) NOT NULL,
  is_active       BOOLEAN NOT NULL DEFAULT true,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (school_id, enrollment_code)
);

CREATE INDEX idx_students_course ON students (course_id);
```

### 3.6 guardians (US-021)

```sql
CREATE TYPE guardian_relationship AS ENUM ('padre', 'madre', 'tutor');

CREATE TABLE guardians (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  parent_id     UUID NOT NULL REFERENCES users(id),
  student_id    UUID NOT NULL REFERENCES students(id) ON DELETE CASCADE,
  relationship  guardian_relationship NOT NULL,
  is_primary    BOOLEAN NOT NULL DEFAULT false,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (parent_id, student_id)
);

CREATE INDEX idx_guardians_student ON guardians (student_id);
```

---

## 4. DDL — Asistencia (E1, E9)

```sql
CREATE TYPE attendance_status AS ENUM ('presente', 'tarde', 'ausente');

CREATE TABLE attendance_records (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id    UUID NOT NULL REFERENCES students(id),
  course_id     UUID NOT NULL REFERENCES courses(id),
  recorded_by   UUID NOT NULL REFERENCES users(id),
  date          DATE NOT NULL,
  check_in_at   TIMESTAMPTZ,
  check_out_at  TIMESTAMPTZ,
  status        attendance_status NOT NULL,
  notes         TEXT,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (student_id, date)
);

CREATE INDEX idx_attendance_student_date ON attendance_records (student_id, date DESC);
CREATE INDEX idx_attendance_course_date ON attendance_records (course_id, date);
```

**Trigger sugerido:** Al insertar/actualizar `check_in_at`, comparar con `schools.entry_time` y setear `status = 'tarde'` si aplica (US-001).

---

## 5. DDL — Tareas (E2)

```sql
CREATE TABLE assignments (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  course_id     UUID NOT NULL REFERENCES courses(id),
  teacher_id    UUID NOT NULL REFERENCES users(id),
  title         VARCHAR(255) NOT NULL,
  description   TEXT,
  due_date      DATE NOT NULL,
  attachments   JSONB NOT NULL DEFAULT '[]',
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_assignments_course_due ON assignments (course_id, due_date);
```

---

## 6. DDL — Disciplina (E3)

```sql
CREATE TYPE discipline_type AS ENUM ('incidente', 'felicitacion');
CREATE TYPE discipline_severity AS ENUM ('leve', 'media', 'grave');

CREATE TABLE discipline_records (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id        UUID NOT NULL REFERENCES students(id),
  teacher_id        UUID NOT NULL REFERENCES users(id),
  type              discipline_type NOT NULL,
  severity          discipline_severity,
  description       TEXT NOT NULL,
  signature_status  VARCHAR(20),
  created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
  CHECK (
    (type = 'incidente' AND severity IS NOT NULL) OR
    (type = 'felicitacion' AND severity IS NULL)
  )
);

CREATE INDEX idx_discipline_student ON discipline_records (student_id, created_at DESC);
```

---

## 7. DDL — Comunicados (E4)

```sql
CREATE TYPE announcement_audience AS ENUM (
  'todo_el_colegio', 'docentes', 'padres', 'curso'
);

CREATE TABLE announcements (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id     UUID NOT NULL REFERENCES schools(id),
  author_id     UUID NOT NULL REFERENCES users(id),
  title         VARCHAR(255) NOT NULL,
  body          TEXT NOT NULL,
  audience      announcement_audience NOT NULL,
  course_id     UUID REFERENCES courses(id),
  published_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  CHECK (
    (audience = 'curso' AND course_id IS NOT NULL) OR
    (audience != 'curso')
  )
);

CREATE TABLE announcement_reads (
  announcement_id UUID NOT NULL REFERENCES announcements(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES users(id),
  read_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
  PRIMARY KEY (announcement_id, user_id)
);
```

---

## 8. DDL — Calendario (E5)

```sql
CREATE TYPE calendar_event_type AS ENUM ('examen', 'actividad', 'reunion', 'feriado');

CREATE TABLE calendar_events (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id       UUID NOT NULL REFERENCES schools(id),
  course_id       UUID REFERENCES courses(id),
  type            calendar_event_type NOT NULL,
  title           VARCHAR(255) NOT NULL,
  description     TEXT,
  starts_at       TIMESTAMPTZ NOT NULL,
  ends_at         TIMESTAMPTZ,
  reminder_sent   BOOLEAN NOT NULL DEFAULT false,
  created_by      UUID NOT NULL REFERENCES users(id),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_calendar_school_starts ON calendar_events (school_id, starts_at);
```

---

## 9. DDL — Mensajería (E6)

```sql
CREATE TABLE message_threads (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id     UUID NOT NULL REFERENCES schools(id),
  course_id     UUID NOT NULL REFERENCES courses(id),
  parent_id     UUID NOT NULL REFERENCES users(id),
  teacher_id    UUID NOT NULL REFERENCES users(id),
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (course_id, parent_id, teacher_id)
);

CREATE TABLE messages (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  thread_id     UUID NOT NULL REFERENCES message_threads(id) ON DELETE CASCADE,
  sender_id     UUID NOT NULL REFERENCES users(id),
  body          TEXT NOT NULL,
  sent_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
  read_at       TIMESTAMPTZ
);

CREATE INDEX idx_messages_thread_sent ON messages (thread_id, sent_at);
```

---

## 10. DDL — Pagos (E7)

```sql
CREATE TYPE payment_status AS ENUM ('pendiente', 'pagado', 'vencido', 'cancelado');

CREATE TABLE payments (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id    UUID NOT NULL REFERENCES students(id),
  reference     VARCHAR(50) NOT NULL UNIQUE,
  concept       VARCHAR(255) NOT NULL,
  amount_cents  INTEGER NOT NULL CHECK (amount_cents > 0),
  due_date      DATE NOT NULL,
  status        payment_status NOT NULL DEFAULT 'pendiente',
  qr_payload    TEXT,
  paid_at       TIMESTAMPTZ,
  confirmed_by  UUID REFERENCES users(id),
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_payments_student_status ON payments (student_id, status);
```

---

## 11. DDL — Calificaciones (E8)

```sql
CREATE TABLE grades (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id    UUID NOT NULL REFERENCES students(id),
  course_id     UUID NOT NULL REFERENCES courses(id),
  subject       VARCHAR(100) NOT NULL,
  period        VARCHAR(50) NOT NULL,
  score         NUMERIC(5,2) NOT NULL CHECK (score >= 0 AND score <= 100),
  teacher_id    UUID NOT NULL REFERENCES users(id),
  published_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (student_id, subject, period)
);
```

---

## 12. DDL — Galería (E10)

```sql
CREATE TABLE gallery_albums (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  course_id     UUID NOT NULL REFERENCES courses(id),
  title         VARCHAR(255) NOT NULL,
  description   TEXT,
  created_by    UUID NOT NULL REFERENCES users(id),
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE gallery_photos (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  album_id      UUID NOT NULL REFERENCES gallery_albums(id) ON DELETE CASCADE,
  url           TEXT NOT NULL,
  caption       VARCHAR(255),
  uploaded_at   TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 13. DDL — Directorio (E11)

```sql
CREATE TABLE directory_contacts (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id     UUID NOT NULL REFERENCES schools(id),
  category      VARCHAR(50) NOT NULL,
  name          VARCHAR(150) NOT NULL,
  phone         VARCHAR(20) NOT NULL,
  schedule      VARCHAR(100),
  sort_order    INTEGER NOT NULL DEFAULT 0
);
```

---

## 14. DDL — Encuestas (E12)

```sql
CREATE TABLE polls (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id           UUID NOT NULL REFERENCES schools(id),
  title               VARCHAR(255) NOT NULL,
  description         TEXT,
  options             JSONB NOT NULL,
  starts_at           TIMESTAMPTZ NOT NULL,
  ends_at             TIMESTAMPTZ NOT NULL,
  is_public_results   BOOLEAN NOT NULL DEFAULT true,
  created_by          UUID NOT NULL REFERENCES users(id),
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE poll_votes (
  poll_id       UUID NOT NULL REFERENCES polls(id) ON DELETE CASCADE,
  user_id       UUID NOT NULL REFERENCES users(id),
  option_id     VARCHAR(50) NOT NULL,
  voted_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  PRIMARY KEY (poll_id, user_id)
);
```

---

## 15. DDL — Firmas digitales (E13)

```sql
CREATE TYPE signature_status AS ENUM ('pendiente', 'firmado', 'rechazado');
CREATE TYPE signature_document_type AS ENUM (
  'autorizacion_excursion', 'acuse_disciplina', 'personalizado'
);

CREATE TABLE signature_requests (
  id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id           UUID NOT NULL REFERENCES schools(id),
  student_id          UUID NOT NULL REFERENCES students(id),
  parent_id           UUID NOT NULL REFERENCES users(id),
  document_type       signature_document_type NOT NULL,
  title               VARCHAR(255) NOT NULL,
  body                TEXT NOT NULL,
  related_id          UUID,
  status              signature_status NOT NULL DEFAULT 'pendiente',
  signature_image_url TEXT,
  signed_at           TIMESTAMPTZ,
  rejected_reason     TEXT,
  device_info         JSONB,
  created_by          UUID NOT NULL REFERENCES users(id),
  created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_signatures_parent_status ON signature_requests (parent_id, status);
```

---

## 16. DDL — Auditoría (US-011)

```sql
CREATE TABLE audit_logs (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id     UUID NOT NULL REFERENCES schools(id),
  actor_id      UUID NOT NULL REFERENCES users(id),
  action        VARCHAR(100) NOT NULL,
  resource_type VARCHAR(50) NOT NULL,
  resource_id   UUID NOT NULL,
  reason        TEXT,
  metadata      JSONB,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_audit_school_created ON audit_logs (school_id, created_at DESC);
```

---

## 17. Row-Level Security (multi-tenant)

```sql
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_users ON users
  USING (school_id = current_setting('app.current_school_id')::UUID);
```

**Nota:** La API setea `SET app.current_school_id = '<uuid>'` por request tras validar JWT. Aplicar política similar a tablas con `school_id`.

---

## 18. Índices de rendimiento adicionales

```sql
-- Resumen asistencia mensual (US-002)
CREATE INDEX idx_attendance_monthly
  ON attendance_records (student_id, date)
  WHERE status IN ('presente', 'tarde', 'ausente');

-- Comunicados no leídos (US-008)
CREATE INDEX idx_announcements_school_published
  ON announcements (school_id, published_at DESC);

-- Tareas pendientes por curso (US-004)
CREATE INDEX idx_assignments_pending
  ON assignments (course_id, due_date)
  WHERE due_date >= CURRENT_DATE;
```

---

## 19. Datos semilla (dev)

```sql
-- Colegio demo El Alto
INSERT INTO schools (slug, name, entry_time) VALUES
  ('san-miguel', 'Colegio San Miguel', '08:00:00');

-- Ver scripts/seed.sql en implementación para usuarios, cursos y alumnos demo
```

---

## 20. Backup y retención

| Aspecto | Política MVP |
|---------|--------------|
| Backup PG | Diario automático, retención 30 días |
| Archivos S3 | Versionado habilitado |
| Audit logs | Retención 2 años |
| Mensajes chat | Retención mientras alumno activo + 1 año |
| Firmas | Retención 5 años (valor legal) |
