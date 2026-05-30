# Environment Documentation — Stage 2: Visual Basic 6

> Fase 0 — Environment Validation
> Status: **CLOSED**
> Validated by: manual inspection and smoke test
> Date: 2025

---

## Summary

This document records the validated development environment for Stage 2.
It serves as the formal closure of Fase 0 and the baseline for all subsequent work in this stage.

---

## Host Machine

| Item | Value |
|------|-------|
| OS | Microsoft Windows 11 Pro (10.0.26200) — Lenovo ThinkPad E14, Intel Core i7-10510U |
| Editor | Visual Studio Code |
| Version control | Git (host-side) |
| GitHub CLI | `gh` (host-side) |
| Project path (host) | Shared folder mapped into VM |

---

## Virtual Machine

| Item | Value |
|------|-------|
| Hypervisor | VMware |
| Guest OS | Windows XP Professional SP3 (32-bit) — 5.1.2600 |
| VB6 IDE | Visual Basic 6.0 |
| Database engine | SQL Server 2000 Personal Edition |
| SQL Server build | 8.00.194 |

---

## Shared Folders

| Mount point (VM) | Purpose |
|-----------------|---------|
| `z:\vc-stage2-vb6` | Project folder — Git-initialized. Contains all source, specs, and docs. |
| `z:\shfldr1` | General-purpose shared folder (scratch, file transfer) |

---

## Workflow

### Development
1. VB6 forms and visual components are designed inside the VM using the VB6 IDE (RAD workflow — drag, drop, property sheets).
2. Non-visual code (modules, logic) may be edited from the host using VSCode when practical.
3. All compilation and execution happens inside the VM.

### Version Control
- Git is managed from the host side.
- Commits are made directly to `main` (no branches — era-appropriate for Stage 2).
- All project files are in `z:\vc-stage2-vb6`, which maps to the host Git repo.

### Code Review
- Code review sessions happen from the host using Claude Desktop App.
- No automated review tooling (era-appropriate).
- Review artifact: self-review checklist (see `docs/cr-checklist.md`).

### Testing
- All testing is manual, executed inside the VM.
- Test evidence: screenshots taken during execution.
- Test cases are derived from `specs/test_cases.md`.
- No unit testing framework (era-appropriate for Stage 2).

---

## Database Engine Decision

**Chosen engine:** SQL Server 2000 Personal Edition (build 8.00.194)

**Rationale:**
- Already installed and validated in the VM.
- SQL Server 2000 was a legitimate and common choice for desktop VB6 applications in the late 1990s / early 2000s.
- Personal Edition is appropriate for a single-user desktop application.
- No licensing or setup overhead for this learning context.

**Rejected alternatives:**
- Access/JET — valid for the era but less instructive for SQL discipline.
- SQLite — anachronistic (not available / not common in this era for VB6).

---

## Known Constraints

- The VM has no internet access. All dependencies must be available locally.
- VB6 IDE cannot be driven from the host — all form design is VM-only.
- Screenshots are the sole testing artifact for UI flows.
- No CI pipeline (era-appropriate).

---

## Fase 0 Checklist

- [x] VM boots and is stable
- [x] VB6 IDE launches successfully
- [x] SQL Server 2000 is installed and accessible
- [x] Shared folder `z:\vc-stage2-vb6` is mounted and writable from the VM
- [x] Git repo exists and is accessible from the host
- [x] VSCode on host can open and edit files in the project folder
- [x] Database engine selected and documented
- [x] Smoke test: VB6 project created, connected to SQL Server, basic form runs

---

## Next Step

With Fase 0 formally closed, the project proceeds to spec work:

1. `specs/prd.md`
2. `specs/domain.md`
3. `specs/tech.md`
4. `specs/test_cases.md`
5. `CLAUDE.md`
