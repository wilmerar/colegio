## Summary

<!-- 1–2 sentences: what and why -->

**User Story:** US-XXX — <!-- title -->

---

## Pre-PR checklist

### Spec & traceability
- [ ] Linked to `specs/Userstories.md` (US-XXX)
- [ ] `Openapi.yml` updated (if API changed)
- [ ] `Database.md` / migration added (if schema changed)

### Code quality
- [ ] KISS / DRY / YAGNI — no unnecessary abstractions or duplication
- [ ] Lint + type-check pass
- [ ] No secrets, `console.log`, or dead code

### Security & RBAC
- [ ] `school_id` scoped from JWT (multi-tenant)
- [ ] Role permissions verified (403 cases covered)
- [ ] Parent cannot submit assignments / cross-tenant blocked

### Tests
- [ ] Unit tests for new business rules
- [ ] Integration test for new/changed endpoints
- [ ] CI green

---

## Test plan

<!-- How you verified it -->

- [ ] Manual: <!-- steps -->
- [ ] Automated: <!-- test files -->

---

## Screenshots / evidence

<!-- UI changes only — optional -->
