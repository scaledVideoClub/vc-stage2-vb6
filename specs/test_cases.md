# Test Cases — Stage 2: Visual Basic 6
# VCM-v2 — Video Club Manager

> Manual test scripts only. No test framework. No automation.
> Steps must be executable by a human inside the Windows XP VM using the VB6 application.
> Execute all precondition setup steps using the application's own flows unless noted.
> "Pass / Fail" fields are left blank — to be filled during execution.
>
> Cases marked **"Tested along TC-XXX"** share the same execution run.
> Run the primary case, verify all listed conditions, and mark both as pass/fail together.

---

## Coverage Map

| Area | IDs |
|------|-----|
| Flow 1 — Rent a Movie | TC-001 – TC-009 |
| Flow 2 — Search / Browse | TC-010 – TC-015 |
| Flow 3 — Return a Movie | TC-016 – TC-022 |
| Flow 4 — Manage Movies | TC-023 – TC-030 |
| Flow 5 — Manage Copies | TC-031 – TC-038 |
| Flow 6 — Manage Customers | TC-039 – TC-046 |
| Business Rules | TC-047 – TC-057 |
| Edge Cases (EC) | TC-058 – TC-065 |
| Invariants (INV) | TC-066 – TC-074 |
| Startup / Config | TC-075 – TC-077 |

---

## Flow 1 — Rent a Movie

---

### TC-001
**Title:** Rent a movie — happy path
**References:** Flow 1, BR-1, BR-3, INV-1, INV-3
**Also tests:** INV-1 (TC-066), INV-3 partial (TC-069)

**Preconditions:**
- At least one movie exists with at least one Available copy.
- At least one customer is registered with no active rentals and no overdue rentals.
- Note the copy code (e.g. 101) and customer name.

**Steps:**
1. On the main screen, type the copy code in the copy code field.
2. Click "Rent".
3. In the rental form, type the customer name in the search field.
4. Click "Find". Confirm the customer appears in the list.
5. Select the customer.
6. Click "Confirm Rental".

**Expected result:**
- No warnings appear during step 5.
- Confirmation message appears after step 6. Rental form closes.
- Main list refreshes. Movie shows one fewer available copy.
- **[INV-1 check]** Query DB: `SELECT COUNT(*) FROM Rentals WHERE copy_id = [copy_id] AND status = 'Active'` → Count = 1.
- Copy status in DB = Rented.

**Pass / Fail:** ___

---

### TC-002
**Title:** Rent — copy code not found
**References:** Flow 1, EC-1

**Preconditions:**
- No copy with code 99999 exists.

**Steps:**
1. Type 99999 in the copy code field. Click "Rent".

**Expected result:**
- Error: "Copy not found."
- No rental form opens. No DB changes.

**Pass / Fail:** ___

---

### TC-003
**Title:** Rent — copy is already Rented
**References:** Flow 1, EC-2, VAL-1, INV-3
**Also tests:** EC-2 (TC-059), INV-3 partial (TC-069)

**Preconditions:**
- A copy with a known code is currently Rented. (Complete TC-001 first without returning.)

**Steps:**
1. Type the Rented copy code in the copy code field. Click "Rent".

**Expected result:**
- Error: "This copy is already rented."
- No rental form opens. No DB changes.

**Pass / Fail:** ___

---

### TC-004
**Title:** Rent — copy is Unavailable
**References:** Flow 1, EC-1, VAL-1, INV-3
**Also tests:** EC-1 (TC-058), INV-3 partial (TC-069)

**Preconditions:**
- A copy is marked Unavailable. (Mark it via Flow 5 — TC-034.)

**Steps:**
1. Type the Unavailable copy code in the copy code field. Click "Rent".

**Expected result:**
- Error: "This copy is marked as unavailable."
- No rental form opens. No DB changes.

**Pass / Fail:** ___

---

### TC-005
**Title:** Rent — customer has overdue rental — employee cancels
**References:** Flow 1, BR-4, EC-4, VAL-3

**Preconditions:**
- Customer A has an active rental whose expected_return is in the past. (Adjust in DB.)
- An available copy exists.

**Steps:**
1. Enter available copy code. Click "Rent".
2. Search for and select Customer A.
3. Observe warning. Click "Cancel".

**Expected result:**
- Warning: "Warning: this customer has overdue rentals. Proceed?"
- No new rental created. Copy remains Available.

**Pass / Fail:** ___

---

### TC-006
**Title:** Rent — customer has overdue rental — employee proceeds
**References:** Flow 1, BR-4, EC-4, VAL-3
**Also tests:** EC-4 (TC-062), BR-4 partial (TC-054)

**Preconditions:**
- Customer A has an active rental with expected_return in the past. (Adjust in DB.)
- An available copy exists.

**Steps:**
1. Enter available copy code. Click "Rent".
2. Search for and select Customer A.
3. Observe warning. Click "Proceed".

**Expected result:**
- Warning appears before confirmation.
- Rental registered. Copy status = Rented. New Active rental exists for Customer A.

**Pass / Fail:** ___

---

### TC-007
**Title:** Rent — customer at maximum active rental limit — employee cancels
**References:** Flow 1, BR-4, EC-5, VAL-2

**Preconditions:**
- Customer B has exactly `max_active_rentals` active rentals (default: 5).
- An available copy exists.

**Steps:**
1. Enter available copy code. Click "Rent".
2. Search for and select Customer B.
3. Observe warning. Click "Cancel".

**Expected result:**
- Warning: "Warning: this customer has reached the rental limit. Proceed?"
- No new rental created. Copy remains Available.

**Pass / Fail:** ___

---

### TC-008
**Title:** Rent — customer at maximum active rental limit — employee proceeds
**References:** Flow 1, BR-4, EC-5, VAL-2
**Also tests:** EC-5 (TC-063)

**Preconditions:**
- Customer B has exactly `max_active_rentals` active rentals.
- An available copy exists.

**Steps:**
1. Enter available copy code. Click "Rent".
2. Search for and select Customer B. Observe warning. Click "Proceed".

**Expected result:**
- Rental created. Customer B now has `max_active_rentals + 1` active rentals.
- Copy status = Rented.

**Pass / Fail:** ___

---

### TC-009
**Title:** Rent — both warnings active at the same time
**References:** Flow 1, BR-4, EC-4, EC-5

**Preconditions:**
- Customer C has `max_active_rentals` active rentals AND at least one is overdue.
- An available copy exists.

**Steps:**
1. Enter available copy code. Click "Rent".
2. Search for and select Customer C.
3. Observe all warnings. Click "Proceed".

**Expected result:**
- Both warnings appear (overdue and rental limit).
- Rental created if employee proceeds.

**Pass / Fail:** ___

---

## Flow 2 — Search / Browse Movies

---

### TC-010
**Title:** Search — no filters returns all movies
**References:** Flow 2

**Preconditions:**
- At least 3 movies exist.

**Steps:**
1. Clear genre dropdown (blank/all) and text field.
2. Click "Search".

**Expected result:**
- All movies shown. None excluded.

**Pass / Fail:** ___

---

### TC-011
**Title:** Search — filter by genre
**References:** Flow 2, BR-6, VAL-6
**Also tests:** BR-6 (TC-053 partial), VAL-6, INV-9 partial (TC-074)

**Preconditions:**
- At least one "Action" movie and at least one other-genre movie exist.

**Steps:**
1. Set genre dropdown to "Action". Leave text field empty. Click "Search".

**Expected result:**
- Only "Action" movies shown.
- Genre dropdown contains only values from config.ini (no free-text entry possible).

**Pass / Fail:** ___

---

### TC-012
**Title:** Search — filter by title (partial match)
**References:** Flow 2

**Preconditions:**
- A movie titled "The Matrix" exists. At least one other movie exists.

**Steps:**
1. Leave genre blank. Type "Matrix" in text field. Click "Search".

**Expected result:**
- "The Matrix" appears. Other movies excluded.

**Pass / Fail:** ___

---

### TC-013
**Title:** Search — filter by movie ID (exact match)
**References:** Flow 2

**Preconditions:**
- A movie with ID 5 exists.

**Steps:**
1. Leave genre blank. Type "5" in text field. Click "Search".

**Expected result:**
- Only movie ID 5 appears.

**Pass / Fail:** ___

---

### TC-014
**Title:** Search — filter returns no results
**References:** Flow 2

**Preconditions:**
- No movie with "ZZZNOTEXIST" in the title exists.

**Steps:**
1. Type "ZZZNOTEXIST" in text field. Click "Search".

**Expected result:**
- Movie list empty. No error.

**Pass / Fail:** ___

---

### TC-015
**Title:** Browse — Rent button disabled when no copies available
**References:** Flow 2, EC-8
**Also tests:** EC-8 (TC-065)

**Preconditions:**
- A movie exists with zero Available copies (all Rented or Unavailable).

**Steps:**
1. Find the movie in the list. Select it. Observe the "Rent" button.

**Expected result:**
- Rent button is disabled.
- Employee can still view details and access Copies and Edit.

**Pass / Fail:** ___

---

## Flow 3 — Return a Movie

---

### TC-016
**Title:** Return — happy path, on time
**References:** Flow 3, BR-1, BR-2
**Also tests:** INV-6 partial (TC-072), TC-022 (no charge stored)

**Preconditions:**
- A copy is Rented. Rental date is today or within max_rental_days.

**Steps:**
1. Enter the copy code in the return field. Click "Return".
2. Verify displayed info (movie title, customer name, dates, charge).
3. Click "Confirm Return".
4. Query DB: `SELECT status, return_date FROM Rentals WHERE copy_id = [copy_id] ORDER BY rental_id DESC` — check top row.
5. Query DB: verify no monetary/charge column exists in the Rentals table schema.

**Expected result:**
- Return form shows correct details. Charge = days_rented × daily_rate (no late fee).
- After confirmation: rental status = Returned, return_date = today (non-null).
- Copy status = Available. Main list refreshes.
- **[INV-6 check]** Returned rental has non-null return_date.
- **[TC-022 check]** Rentals table has no charge/amount column.

**Pass / Fail:** ___

---

### TC-017
**Title:** Return — late return with fee
**References:** Flow 3, BR-1, BR-2
**Also tests:** BR-1 multi-day (TC-049)

**Preconditions:**
- A copy is Rented with rental_date 5 days ago. (Adjust in DB.)
- max_rental_days = 3, daily_rate = 2.50, late_daily_rate = 4.00.

**Steps:**
1. Enter copy code. Click "Return". Observe charge breakdown. Click "Confirm Return".

**Expected result:**
- Days rented = 5.
- Standard charge = 3 × 2.50 = 7.50.
- Late fee = 2 × 4.00 = 8.00.
- Total = 15.50.
- Return confirmed. Copy = Available. Rental = Returned.

**Pass / Fail:** ___

---

### TC-018
**Title:** Return — same-day return counts as 1 day (BR-1 minimum)
**References:** Flow 3, BR-1
**Also tests:** BR-1 (TC-047), BR-1 next-day (TC-048)

**Preconditions:**
- Copy A is Rented today (rental_date = today).
- Copy B is Rented yesterday (rental_date = yesterday). Adjust in DB if needed.

**Steps:**
1. Return Copy A. Observe days and charge.
2. Return Copy B. Observe days and charge.

**Expected result:**
- Copy A: days = 1, charge = 1 × daily_rate.
- Copy B: days = 1, charge = 1 × daily_rate.

**Pass / Fail:** ___

---

### TC-019
**Title:** Return — copy code not found
**References:** Flow 3, VAL-9

**Preconditions:**
- No copy with code 88888 exists.

**Steps:**
1. Enter "88888" in the return field. Click "Return".

**Expected result:**
- Error: "Copy not found." No DB changes.

**Pass / Fail:** ___

---

### TC-020
**Title:** Return — copy exists but no active rental (copy is Available)
**References:** Flow 3, EC-3, VAL-9

**Preconditions:**
- A copy is in Available status.

**Steps:**
1. Enter the copy code in the return field. Click "Return".

**Expected result:**
- System reports no active rental found.
- Copy remains Available. No crash.

**Pass / Fail:** ___

---

### TC-021
**Title:** Return — copy is Unavailable (lost copy found, EC-3/EC-7)
**References:** Flow 3, EC-3, EC-7
**Also tests:** EC-3/EC-7 (TC-061)

**Preconditions:**
- A copy exists with status Unavailable. No active rental for it.

**Steps:**
1. Enter the copy code in the return field. Click "Return".

**Expected result:**
- Copy status changes to Available. Confirmation shown.
- No rental record created or modified.

**Pass / Fail:** ___

---

### TC-022
**Title:** Return — charge is informational only (no payment stored)
**References:** Flow 3, BR-2, PRD constraint 4
**Tested along TC-016** — verify in step 5 of TC-016.

**Pass / Fail:** ___ *(mark same as TC-016)*

---

## Flow 4 — Manage Movies

---

### TC-023
**Title:** Add Movie — happy path
**References:** Flow 4, VAL-4, VAL-5
**Also tests:** BR-6/INV-9 partial (genre from dropdown only), TC-027 (sequential ID)

**Preconditions:**
- Note the current max movie_id in DB before running.

**Steps:**
1. Open Add Movie form from main menu.
2. Enter title: "Blade Runner". Select genre: "Sci-Fi". Enter summary: "A neo-noir science fiction film."
3. Click "Save".
4. Query DB for the new movie record.

**Expected result:**
- Confirmation shown. Form closes. "Blade Runner" appears in main list.
- New movie_id = previous max + 1 (not zero-padded).
- Genre stored = "Sci-Fi" (exactly as selected from dropdown).

**Pass / Fail:** ___

---

### TC-024
**Title:** Add Movie — title is required
**References:** Flow 4, VAL-4

**Preconditions:**
- Add Movie form is open.

**Steps:**
1. Leave Title empty. Select a genre. Click "Save".

**Expected result:**
- Error: "Title is required." No movie saved. Form stays open.

**Pass / Fail:** ___

---

### TC-025
**Title:** Add Movie — genre is required
**References:** Flow 4, VAL-5

**Preconditions:**
- Add Movie form is open.

**Steps:**
1. Enter a title. Leave genre unselected. Click "Save".

**Expected result:**
- Error: "Genre is required." No movie saved. Form stays open.

**Pass / Fail:** ___

---

### TC-026
**Title:** Add Movie — summary is optional
**References:** Flow 4

**Preconditions:**
- Add Movie form is open.

**Steps:**
1. Enter title: "Casablanca". Select genre: "Drama". Leave summary empty. Click "Save".

**Expected result:**
- Movie saved. Summary is null/empty in DB. No error.

**Pass / Fail:** ___

---

### TC-027
**Title:** Add Movie — system assigns sequential ID
**References:** Flow 4, domain.md Movie entity
**Tested along TC-023** — verify in step 4 of TC-023.

**Pass / Fail:** ___ *(mark same as TC-023)*

---

### TC-028
**Title:** Edit Movie — update title and genre
**References:** Flow 4
**Also tests:** TC-029 (title required on edit) — run TC-029 steps within the same edit session before saving

**Preconditions:**
- A movie exists with title "Old Title" and genre "Action".

**Steps:**
1. Select the movie. Click "Edit".
2. **[TC-029 check]** Clear the title field. Click "Save" → expect error "Title is required." Do not close form.
3. Re-enter title: "New Title". Change genre to "Comedy". Click "Save".

**Expected result:**
- Step 2: error shown, no DB change, form stays open.
- Step 3: confirmation shown. Main list shows "New Title" / "Comedy". DB updated.

**Pass / Fail:** ___

---

### TC-029
**Title:** Edit Movie — title cannot be cleared
**References:** Flow 4, VAL-4
**Tested along TC-028** — verified in step 2 of TC-028.

**Pass / Fail:** ___ *(mark same as TC-028)*

---

### TC-030
**Title:** Movies cannot be deleted
**References:** PRD Flow 4 notes, domain.md Movie rules
**Also tests:** Copies cannot be deleted (TC-038), Customers cannot be deleted (TC-046)

**Preconditions:**
- At least one movie, one copy, and one customer exist.

**Steps:**
1. Select a movie. Look for a "Delete" button on main screen and in Edit form. Note result.
2. Open Copies for a movie. Select a copy. Look for "Delete" button. Note result.
3. Open Customer form with a customer loaded. Look for "Delete" button. Note result.

**Expected result:**
- No delete option exists for movies, copies, or customers anywhere in the UI.

**Pass / Fail:** ___

---

## Flow 5 — Manage Copies

---

### TC-031
**Title:** Add Copy — first and subsequent copies; copy code formula
**References:** Flow 5, BR-5, INV-7, INV-8
**Also tests:** TC-032, TC-033, BR-5 (TC-056), INV-7 (TC-073 partial), INV-8

**Preconditions:**
- Movie ID 7 exists with no copies. Movie ID 10 exists with no copies.

**Steps:**
1. Open Copies for movie ID 7. Add three copies, one at a time. Note each copy_id.
2. Open Copies for movie ID 10. Add one copy. Note copy_id.
3. Query DB: `SELECT copy_id, COUNT(*) FROM Copies GROUP BY copy_id HAVING COUNT(*) > 1`.

**Expected result:**
- Movie 7 copies: 701, 702, 703 (sequence never reused).
- Movie 10 first copy: 1001.
- 701 ≠ 1001 — codes are distinct across movies.
- All new copies: status = Available, comment = empty.
- **[INV-7 check]** Uniqueness query returns zero rows.

**Pass / Fail:** ___

---

### TC-032
**Title:** Add Copy — second and subsequent copies
**References:** Flow 5, BR-5, INV-8
**Tested along TC-031** — verified in step 1 of TC-031.

**Pass / Fail:** ___ *(mark same as TC-031)*

---

### TC-033
**Title:** Add Copy — copy codes unique across movies
**References:** BR-5, INV-7
**Tested along TC-031** — verified in steps 2–3 of TC-031.

**Pass / Fail:** ___ *(mark same as TC-031)*

---

### TC-034
**Title:** Mark Copy as Unavailable — from Available; comment optional
**References:** Flow 5, EC-6
**Also tests:** TC-037 (comment is optional)

**Preconditions:**
- A copy exists with status Available.

**Steps:**
1. Open Copies for that movie. Select the Available copy. Click "Mark Unavailable".
2. Enter comment: "scratched". Confirm.
3. Verify: copy status = Unavailable, comment = "scratched" in DB.
4. Open Copies for another Available copy. Click "Mark Unavailable". Leave comment empty. Confirm.
5. Verify: copy status = Unavailable, comment = null/empty in DB.

**Expected result:**
- Step 2–3: status = Unavailable, comment stored correctly. No warning about active rental.
- Step 4–5: status = Unavailable, no error despite empty comment.

**Pass / Fail:** ___

---

### TC-035
**Title:** Mark Copy as Unavailable — from Rented status (EC-6)
**References:** Flow 5, EC-6, domain.md Copy state transitions
**Also tests:** EC-6 (TC-064), INV-2 (TC-068)

**Preconditions:**
- A copy is currently Rented (active rental exists). Note the copy_id and rental_id.

**Steps:**
1. Open Copies for that movie. Select the Rented copy. Click "Mark Unavailable".
2. Observe the warning. Click "Confirm".
3. Query DB: `SELECT COUNT(*) FROM Rentals WHERE copy_id = [copy_id] AND status = 'Active'`.

**Expected result:**
- Warning: "This copy has an active rental. The rental will be closed. A damage/loss fee must be charged to the customer outside the system. Proceed?"
- After confirm: active rental closed (status = Returned, return_date = today).
- Copy status = Unavailable.
- **[INV-2 check]** Active rental count = 0.

**Pass / Fail:** ___

---

### TC-036
**Title:** Mark Copy as Unavailable — employee cancels the warning
**References:** Flow 5, EC-6

**Preconditions:**
- A copy is currently Rented.

**Steps:**
1. Open Copies. Select the Rented copy. Click "Mark Unavailable".
2. Observe warning. Click "Cancel".

**Expected result:**
- No changes. Copy remains Rented. Active rental remains Active.

**Pass / Fail:** ___

---

### TC-037
**Title:** Comment is optional when marking Unavailable
**References:** Flow 5
**Tested along TC-034** — verified in steps 4–5 of TC-034.

**Pass / Fail:** ___ *(mark same as TC-034)*

---

### TC-038
**Title:** Copies cannot be deleted
**References:** domain.md Copy rules
**Tested along TC-030** — verified in step 2 of TC-030.

**Pass / Fail:** ___ *(mark same as TC-030)*

---

## Flow 6 — Manage Customers

---

### TC-039
**Title:** Add Customer — happy path; phone and address optional
**References:** Flow 6, VAL-7
**Also tests:** TC-041 (optional fields)

**Preconditions:**
- Customer form accessible from main menu.

**Steps:**
1. Open Customer form. Click "New".
2. Enter name: "Maria García", phone: "555-1234", address: "Calle Mayor 10". Click "Save".
3. Verify: customer loaded with system-assigned ID (sequential, not zero-padded).
4. Click "New" again. Enter name: "Juan Pérez". Leave phone and address empty. Click "Save".
5. Verify: customer saved, phone and address = null in DB.

**Expected result:**
- Step 3: ID assigned, all fields match.
- Step 5: saved without error. Optional fields are null.

**Pass / Fail:** ___

---

### TC-040
**Title:** Add Customer — name is required
**References:** Flow 6, VAL-7

**Preconditions:**
- Customer form in "New" mode.

**Steps:**
1. Leave name empty. Enter phone and address. Click "Save".

**Expected result:**
- Error: "Customer name is required." No customer saved.

**Pass / Fail:** ___

---

### TC-041
**Title:** Add Customer — phone and address optional
**References:** Flow 6, domain.md Customer entity
**Tested along TC-039** — verified in steps 4–5 of TC-039.

**Pass / Fail:** ___ *(mark same as TC-039)*

---

### TC-042
**Title:** Find Customer — by name and by ID
**References:** Flow 6
**Also tests:** TC-043 (find by ID)

**Preconditions:**
- "Maria García" exists. A customer with ID 3 exists.

**Steps:**
1. Open Customer form. Type "Maria" in search field. Click "Find".
2. Verify record loads (or selection list appears with correct entry).
3. Clear search field. Type "3". Click "Find".
4. Verify customer ID 3 loads directly.

**Expected result:**
- Step 2: Maria García record found. Top section shows correct ID, name, phone, address.
- Step 4: Customer with ID 3 loads.

**Pass / Fail:** ___

---

### TC-043
**Title:** Find Customer — by customer ID
**References:** Flow 6
**Tested along TC-042** — verified in steps 3–4 of TC-042.

**Pass / Fail:** ___ *(mark same as TC-042)*

---

### TC-044
**Title:** View Active Rentals and Full History in Customer form
**References:** Flow 6, INV-6
**Also tests:** TC-045 (full history toggle), INV-6 partial (TC-072)

**Preconditions:**
- Customer has 2 active rentals and at least 1 returned rental.

**Steps:**
1. Open Customer form. Find the customer.
2. Verify rental grid is in "Active Rentals" mode (default). Read grid.
3. Toggle selector to "Full History". Read grid.

**Expected result:**
- Step 2: 2 rows. Columns: movie title, copy code, rental date, expected return. Return date column empty.
- Step 3: all rentals shown (active + returned). Returned rentals show actual return_date. Active rentals show null/blank. Ordered by rental date descending.

**Pass / Fail:** ___

---

### TC-045
**Title:** Toggle to Full Rental History
**References:** Flow 6, INV-6
**Tested along TC-044** — verified in step 3 of TC-044.

**Pass / Fail:** ___ *(mark same as TC-044)*

---

### TC-046
**Title:** Customers cannot be deleted
**References:** domain.md Customer rules
**Tested along TC-030** — verified in step 3 of TC-030.

**Pass / Fail:** ___ *(mark same as TC-030)*

---

## Business Rules

---

### TC-047
**Title:** BR-1 — Same-day return = 1 day minimum
**References:** BR-1
**Tested along TC-018** — verified in step 1 of TC-018.

**Pass / Fail:** ___ *(mark same as TC-018)*

---

### TC-048
**Title:** BR-1 — Next-day return = 1 day
**References:** BR-1
**Tested along TC-018** — verified in step 2 of TC-018.

**Pass / Fail:** ___ *(mark same as TC-018)*

---

### TC-049
**Title:** BR-1 — Two-day-later return = 2 days
**References:** BR-1
**Tested along TC-017** — use the same Rented copy with rental_date 2 days ago as an intermediate check before running the 5-day scenario, or run separately with rental_date adjusted to 2 days ago.

**Preconditions:**
- rental_date is 2 days ago. Adjust in DB.

**Steps:**
1. Enter copy code. Click "Return". Observe days shown.

**Expected result:**
- Days = 2. Charge = 2 × daily_rate.

**Pass / Fail:** ___

---

### TC-050
**Title:** BR-2 — On-time return: no late fee (boundary)
**References:** BR-2

**Preconditions:**
- A copy was rented exactly max_rental_days ago (default: 3 days). Adjust in DB.
- daily_rate = 2.50.

**Steps:**
1. Enter copy code. Click "Return". Read charge breakdown.

**Expected result:**
- Days = 3. Standard charge = 7.50. Late fee = 0. Total = 7.50.

**Pass / Fail:** ___

---

### TC-051
**Title:** BR-2 — Late return: correct fee calculation
**References:** BR-2
**Tested along TC-017** — verified in TC-017.

**Pass / Fail:** ___ *(mark same as TC-017)*

---

### TC-052
**Title:** BR-3 — Config values reloaded on application restart
**References:** BR-3, tech.md Section 2 / 10

**Preconditions:**
- config.ini in application directory. Note current daily_rate.

**Steps:**
1. Close application. Edit config.ini: change daily_rate to 3.00. Restart application.
2. Perform a return. Observe charge.
3. Restore config.ini to original value.

**Expected result:**
- Charge reflects 3.00/day, not the old rate.

**Pass / Fail:** ___

---

### TC-053
**Title:** BR-3/BR-6 — Genre list read from config.ini; no free-text entry
**References:** BR-3, BR-6, VAL-6, INV-9
**Also tests:** INV-9 (TC-074), VAL-6

**Preconditions:**
- config.ini [Genres] list = Action,Comedy,Drama,Horror,Sci-Fi,Documentary,Animation.

**Steps:**
1. Open Add Movie form. Click genre dropdown. Count and read all options.
2. Attempt to type directly into the genre field (if combo box — verify no free-text accepted).

**Expected result:**
- Dropdown shows exactly the 7 genres from config.ini. No others. No free-text entry.

**Pass / Fail:** ___

---

### TC-054
**Title:** BR-4 — Overdue warning; no warning when on time
**References:** BR-4, Flow 1, VAL-3
**Also tests:** TC-055 (no warning when on time)

**Preconditions:**
- Customer X has an active rental with expected_return = yesterday. (Adjust in DB.)
- Customer Y has active rentals but none overdue, and is below the limit.
- Two available copies exist.

**Steps:**
1. Rent a copy to Customer X. Observe warning at customer selection.
2. Cancel that rental (do not proceed).
3. Rent a copy to Customer Y. Observe (or non-observe) warnings at customer selection.

**Expected result:**
- Step 1: warning "Warning: this customer has overdue rentals. Proceed?"
- Step 3: no warnings appear.

**Pass / Fail:** ___

---

### TC-055
**Title:** BR-4 — No overdue warning when all rentals are on time
**References:** BR-4
**Tested along TC-054** — verified in step 3 of TC-054.

**Pass / Fail:** ___ *(mark same as TC-054)*

---

### TC-056
**Title:** BR-5 — Copy code formula: movie_id × 100 + sequence
**References:** BR-5, INV-7, INV-8
**Tested along TC-031** — verified in step 1 of TC-031.

**Pass / Fail:** ___ *(mark same as TC-031)*

---

### TC-057
**Title:** BR-6 — Genre values in DB match config.ini
**References:** BR-6, INV-9
**Tested along TC-053** — after adding movies through the app, run the DB query below.

**Steps:**
1. Query DB: `SELECT DISTINCT genre FROM Movies`. Compare against config.ini list.

**Expected result:**
- All genre values in DB are on the config.ini list. No unlisted value exists.

**Pass / Fail:** ___

---

## Edge Cases

---

### TC-058
**Title:** EC-1 — Renting an Unavailable copy
**References:** EC-1, VAL-1
**Tested along TC-004.**

**Pass / Fail:** ___ *(mark same as TC-004)*

---

### TC-059
**Title:** EC-2 — Renting an already Rented copy
**References:** EC-2, VAL-1
**Tested along TC-003.**

**Pass / Fail:** ___ *(mark same as TC-003)*

---

### TC-060
**Title:** EC-3 — Return of a copy with no active rental (copy is Available)
**References:** EC-3, Flow 3
**Tested along TC-020.**

**Pass / Fail:** ___ *(mark same as TC-020)*

---

### TC-061
**Title:** EC-3/EC-7 — Returning a copy that is Unavailable (found after loss)
**References:** EC-3, EC-7, Flow 3
**Tested along TC-021.**

**Pass / Fail:** ___ *(mark same as TC-021)*

---

### TC-062
**Title:** EC-4 — Employee proceeds despite overdue warning
**References:** EC-4, BR-4
**Tested along TC-006.**

**Pass / Fail:** ___ *(mark same as TC-006)*

---

### TC-063
**Title:** EC-5 — Employee proceeds despite rental limit warning
**References:** EC-5, BR-4
**Tested along TC-008.**

**Pass / Fail:** ___ *(mark same as TC-008)*

---

### TC-064
**Title:** EC-6 — Marking a Rented copy Unavailable closes the active rental
**References:** EC-6, Flow 5
**Tested along TC-035.**

**Pass / Fail:** ___ *(mark same as TC-035)*

---

### TC-065
**Title:** EC-8 — Rent button disabled when available_copies = 0
**References:** EC-8, Flow 2
**Tested along TC-015.**

**Pass / Fail:** ___ *(mark same as TC-015)*

---

## Invariants

---

### TC-066
**Title:** INV-1 — Rented copy has exactly one Active Rental
**References:** INV-1
**Tested along TC-001** — verified in DB check at end of TC-001.

**Pass / Fail:** ___ *(mark same as TC-001)*

---

### TC-067
**Title:** INV-2 — Available copy has no Active Rental
**References:** INV-2

**Preconditions:**
- A copy is in Available status (e.g. after a return).

**Steps:**
1. Query DB: `SELECT COUNT(*) FROM Rentals WHERE copy_id = [copy_id] AND status = 'Active'`.

**Expected result:**
- Count = 0.

**Pass / Fail:** ___

---

### TC-068
**Title:** INV-2 — Unavailable copy has no Active Rental (after EC-6)
**References:** INV-2
**Tested along TC-035** — verified in DB check at end of TC-035.

**Pass / Fail:** ___ *(mark same as TC-035)*

---

### TC-069
**Title:** INV-3 — Cannot rent a Rented or Unavailable copy
**References:** INV-3
**Tested along TC-003 and TC-004** — both error conditions verified there.

**Pass / Fail:** ___ *(mark same as TC-003 and TC-004)*

---

### TC-070
**Title:** INV-4 — Every Copy references a valid Movie
**References:** INV-4

**Preconditions:**
- Several copies added through the application.

**Steps:**
1. Query DB: `SELECT c.copy_id FROM Copies c LEFT JOIN Movies m ON m.movie_id = c.movie_id WHERE m.movie_id IS NULL`.

**Expected result:**
- Zero rows returned.

**Pass / Fail:** ___

---

### TC-071
**Title:** INV-5 — expected_return = rental_date + max_rental_days
**References:** INV-5

**Preconditions:**
- max_rental_days = 3. A rental was just created (TC-001 can supply this).

**Steps:**
1. Query DB: `SELECT rental_date, expected_return FROM Rentals WHERE rental_id = [id]`.

**Expected result:**
- expected_return = rental_date + 3 days.

**Pass / Fail:** ___

---

### TC-072
**Title:** INV-6 — Active = null return_date; Returned = non-null return_date
**References:** INV-6
**Tested along TC-016 and TC-044** — verified in DB checks there.

**Pass / Fail:** ___ *(mark same as TC-016)*

---

### TC-073
**Title:** INV-7 — Copy codes unique across the system
**References:** INV-7
**Tested along TC-031** — verified in step 3 of TC-031.

**Pass / Fail:** ___ *(mark same as TC-031)*

---

### TC-074
**Title:** INV-9 — Movie genre values match config.ini
**References:** INV-9
**Tested along TC-053 / TC-057** — verified by DB query in TC-057.

**Pass / Fail:** ___ *(mark same as TC-057)*

---

## Startup / Config

---

### TC-075
**Title:** Application starts successfully with valid config.ini
**References:** tech.md Section 10, BR-3

**Preconditions:**
- config.ini present with valid values. DB vcm2 accessible.

**Steps:**
1. Start the application (double-click exe or run from VB6 IDE).

**Expected result:**
- Main screen opens. Movie list visible. No errors at startup.

**Pass / Fail:** ___

---

### TC-076
**Title:** Fatal error when config.ini is missing
**References:** tech.md Section 10, Section 7

**Preconditions:**
- config.ini exists. Make a backup copy.

**Steps:**
1. Rename config.ini to config.ini.bak. Start the application.

**Expected result:**
- Fatal error: "config.ini not found. Application cannot start." Main form does not open.

**Cleanup:** Rename back to config.ini.

**Pass / Fail:** ___

---

### TC-077
**Title:** Fatal error when database is unreachable
**References:** tech.md Section 7

**Preconditions:**
- SQL Server service is running.

**Steps:**
1. Stop the SQL Server service (via Services in Windows XP). Start the application.

**Expected result:**
- Fatal error: "Cannot connect to database. Check configuration and try again." Main form does not open.

**Cleanup:** Restart SQL Server service.

**Pass / Fail:** ___

---

*Test cases version: 1.1 — Stage 2 VB6*
*Total cases: 77 (runnable executions: ~45)*
*Status: Ready for execution*
*Era constraint: manual execution only — no test framework, no automation*
