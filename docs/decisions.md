# Stage 2 — Decisions

This document records technical and design decisions made during Stage 2 development.
Each decision is documented with context: the question asked, the choice made, and why.

---

## D-001: Genre Storage — INI, Not Database

**Decision:** Genre values are stored in `config.ini [Genres]` section, not in a database table.

**Context:**
- Genres are system configuration, not data that changes at runtime
- Stage 2 forbids introducing the Genres entity (reserved for later stages)
- Single workstation, single employee, no need for centralized genre management

**Rationale:**
- Simpler: no Genres table, no FK on Movies, no JOIN
- Era-correct: 1999 style would hardcode or config-file categories
- Spec-compliant: domain.md lists Genre as VARCHAR on Movies, not a separate entity

**Implementation:**
- `gGenreList` loaded from `config.ini` at startup in `frmMain_Load`
- When displaying movie genre in forms, use the VARCHAR value directly
- When validating genre in movie creation/edit, check against `gGenreList`

---

## D-002: Configuration Via config.ini, Not Database

**Decision:** All system configuration (genres, fees, rental duration, etc.) lives in `config.ini`, not in database tables.

**Context:**
- Stage 2 is single-workstation, single employee
- No need for runtime configuration changes via UI
- Keeps data schema simple

**Rationale:**
- Aligns with VB6 era practices (INI files were standard)
- Reduces schema complexity
- Configuration is not domain data — it's environmental

**Implementation:**
- `config.ini` in the project root
- Windows API calls (`GetPrivateProfileString`, `GetPrivateProfileInt`) to read values
- Values cached in module-level variables at startup
- No UI for configuration in this stage

---

## D-003: Single Global Connection (gConn)

**Decision:** One `ADODB.Connection` object, opened at startup, shared across all forms.

**Context:**
- Single workstation, single employee at a time
- No connection pooling or concurrency concerns
- VB6 pattern: keep it simple

**Rationale:**
- Reduces state management complexity
- Era-correct for 1999
- Easier error recovery (one place to close on shutdown)

**Implementation:**
- Opened in `frmMain_Load`
- Closed in `frmMain_QueryUnload`
- Passed implicitly via module-level variable `gConn` (visible to all forms)

---

## D-004: Copy ID Computation — Application, Not Database

**Decision:** Copy ID is computed by the application as `movie_id * 100 + sequence`, not assigned by database IDENTITY.

**Context:**
- Tech spec defines the formula explicitly
- Multiple copies per movie need unique IDs within a predictable scheme
- Sequence per movie (1, 2, 3...) never reused

**Rationale:**
- Application controls the ID allocation logic — easier to audit and understand
- No IDENTITY column — keeps SQL simpler
- Era-correct: VB6 apps often managed their own ID sequences

**Implementation:**
- `NextCopyId()` function in `modApp.bas` queries max sequence for a movie, increments, computes copy_id
- Call before INSERT in movie copy creation flow

---

## D-005: Warnings Are Non-Blocking

**Decision:** Customer warnings (overdue, rental limit) are displayed but do not prevent rental.

**Context:**
- Tech spec BR-4: warnings "encourage" but do not block
- Employee at counter has judgment to override
- Real-world scenario: urgent customer needs access despite warnings

**Rationale:**
- Matches business rules intent: it's guidance, not a lock
- User experience: dialog warns, buttons are OK / Cancel, not just OK
- Stage 2 does not enforce business logic at the DB level

**Implementation:**
- Warning checks in `frmRent` before confirmation
- If warnings exist, show message box with details
- Button: "Continue Anyway" or "Cancel"
- No modification to rental status on warning — proceed as normal if employee chooses

---

## D-006: No Payment Recording

**Decision:** Rental fees are calculated and displayed, but never stored in the database.

**Context:**
- Stage 2 has no payment processing or business rules for payment
- Fee display is for employee information only
- Actual payment (cash, card) is outside system scope

**Rationale:**
- Keeps schema and flows simple
- Matches Stage 2 scope: rental lifecycle, not financial
- Payment recording deferred to later stages

**Implementation:**
- `CalcRentalFee()` in `modApp.bas` returns a currency value
- Displayed in rental confirmation dialog
- No Payments table, no payment INSERT
- No accounting ledger or balance tracking

---

## D-007: Single Module — modApp.bas Only

**Decision:** All shared code lives in `modApp.bas`. No other `.bas` modules.

**Context:**
- Forbidden concept: multiple modules fragment the codebase unnecessarily in VB6 single-workstation app
- One module keeps all utilities, globals, and helper functions visible and findable

**Rationale:**
- Era-correct: VB6 projects with one module were standard
- Easier to review and maintain for a small app
- Aligns with CLAUDE.md forbidden list

**Implementation:**
- All `Public` variables, functions, and Subs are in `modApp.bas`
- Forms reference `modApp` functions as needed
- No form-to-form calls; all communication through globals or form parameters

---

*Last updated: [during Fase 0]*
*Next: add decisions as development progresses*
