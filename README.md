# vc-stage2-vb6

**Scaled Video Club — Stage 2: Visual Basic 6**

Part of the [Scaled Video Club](https://github.com/scaledVideoClub) project.
A video rental system built across 7 stages, each reflecting the tools and practices of its era.

---

## Stage Overview

| Field | Value |
|-------|-------|
| Stage | 2 of 7 |
| Stack | Visual Basic 6 |
| Paradigm | Event-driven + GUI |
| Constraint | Stock (physical copies) |
| Environment | Windows native (VM) |

---

## What Is New in This Stage

- Event-driven programming model (form events, control events)
- GUI forms replacing DOS screen navigation
- Genre / category classification for movies
- Customer rental history
- Multiple copies per movie (copy management)
- Basic search by name or code

---

## Development Strategy

This stage reflects mid-1990s Windows desktop development practices.
Written requirements existed; formal process did not yet dominate.

### Spec-Driven Development
A written spec exists before any UI or code is built: screens, events, and data
structures are defined first. Spec lives in `/specs/`. This maps to what a
mid-sized software shop would have called a "functional design document."

### Testing
No TDD. A written test plan is prepared before coding begins (`test_cases.md`),
but tests are executed manually after implementation. No automated test framework.
VB6 had no standard unit test tooling.

### Code Review
Informal self-review via a lightweight checklist (see `/docs/cr-checklist.md`).
No external reviewer. Review happens before merging to `main`, but is not PR-gated.

### Version Control
Git is used as a modern concession — SourceSafe existed but branching was painful
and rarely used in solo projects of this era.
All commits go directly to `main`. No branches, no PRs.
`main` must always run. Commit only working states.

### Docs & Jira
Jira stories defined before starting each feature. Commits reference the story ID.
No PR required — commits go directly to `main`.
`decisions.md` updated continuously during implementation.

---

## Repository Structure

```
/specs/
  prd.md
  domain.md
  tech.md
  test_cases.md
/src/
  [VB6 project files]
/docs/
  setup.md
  run.md
  decisions.md
  retrospective.md
  cr-checklist.md
CLAUDE.md
README.md
```

---

## Forbidden Concepts

The following must not appear in this stage, even if they seem natural:

- No separation of UI from business logic into distinct layers (Stage 3)
- No manual memory management (Stage 4)
- No service/repository pattern (Stage 5)
- No web interfaces, REST APIs, or HTTP (Stage 6+)
- No automated tests or CI pipelines (Stage 4+)

---

## Definition of Done

- [ ] All PRD flows work end-to-end (demonstrable in the VB6 environment)
- [ ] All test cases in `test_cases.md` pass (executed manually)
- [ ] All commits are on `main` and the app runs cleanly
- [ ] `docs/setup.md`, `run.md`, `decisions.md` are complete
- [ ] `docs/retrospective.md` is written

---

*See [vc-project](https://github.com/scaledVideoClub/vc-project) for the master context and full stage map.*
