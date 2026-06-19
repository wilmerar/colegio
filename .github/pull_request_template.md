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
- [ ] Unit tests for new business rules (see below)
- [ ] Integration test for new/changed endpoints
- [ ] CI green

#### Unit tests (required for new logic)
- [ ] Added/updated `*.service.spec.ts` (or `*.guard.spec.ts` / `packages/shared/*.spec.ts`)
- [ ] Happy path **+** at least one error or edge case per new method
- [ ] Maps to Gherkin scenario **US-XXX**
- [ ] Mocks at repo/adapter boundary — no real DB or external APIs
- [ ] `pnpm test:unit` passes; service coverage ≥ 80%

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
