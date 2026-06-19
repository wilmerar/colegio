## Summary

<!-- 1–2 sentences: what and why -->

**User Story:** US-XXX — <!-- title -->

---

## Pre-PR checklist

### Spec & traceability
- [ ] Linked to `docs/user-stories/US-XXX-*.md`
- [ ] [US-T02](docs/user-stories/US-T02-pre-pr-checklist.md) Pre-PR checklist
- [ ] `docs/06-api-openapi.yml` updated (if API changed)
- [ ] `docs/04-database.md` / migration (if schema changed)

### Code quality
- [ ] KISS / DRY / YAGNI — no unnecessary abstractions or duplication
- [ ] Lint + type-check pass
- [ ] No secrets, `console.log`, or dead code

### Security & RBAC
- [ ] `school_id` scoped from JWT (multi-tenant)
- [ ] Role permissions verified (403 cases covered)
- [ ] Parent cannot submit assignments / cross-tenant blocked

### Tests
- [ ] [US-T01](docs/user-stories/US-T01-unit-tests-codigo-nuevo.md) — unit tests for new logic
- [ ] Happy path + error/edge case; Gherkin US-XXX covered
- [ ] Integration test for new/changed endpoints
- [ ] CI green; coverage ≥ 80% services

<!-- List new test files:
- apps/api/src/.../foo.service.spec.ts — describes what is covered
-->

---

## Test plan

<!-- How you verified it -->

- [ ] Manual: <!-- steps -->
- [ ] Automated: <!-- test files -->

---

## Screenshots / evidence

<!-- UI changes only — optional -->
