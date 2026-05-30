# PRD — Stage 2: Visual Basic 6
# VCM-v2 — Video Club Manager

> This document defines what the system does, for whom, and under what rules.
> It does not define how it is built. For that, see tech.md.

---

## 1. Overview

VCM-v2 (Video Club Manager) is a desktop application for managing the
day-to-day operations of a physical video rental store.

It replaces the command-line interface of Stage 1 with a graphical Windows
application. The system runs on a single PC at the store counter, used
exclusively by store staff. No authentication or user management is required.

The application centers on a movie list as its main view. All other operations
— renting, returning, managing copies, managing customers — are initiated from
there or from the main menu, and open as secondary forms.

Each movie has a daily rental price. When a copy is returned, the system
calculates the total charge based on rental duration and displays it to the
employee. Late returns incur an additional fee. Payment recording is
informational only — no payment processing occurs in this stage.

The system is transactional and operational. It does not produce reports,
statistics, or recommendations. It records what happens and enforces the
rules of the rental business.

---

## 2. Actors

### Store Employee
The only actor in the system. No authentication is required.
The employee operates the application from a single PC at the store counter.

Responsibilities:
- Register rentals and returns
- Calculate and display rental charges and late fees at return time
- Search for movies by title, code, or genre
- Inform customers about availability
- Manage the movie catalog (add, edit)
- Manage physical copies (add, mark as unavailable)
- Manage customer records (add, edit, view rental history)

There is no distinction between roles (e.g. manager vs. cashier).
Access control is out of scope for this stage.

---

## 3. Core Flows

### Main Screen

The main screen displays the movie catalog as a list. It is the entry point
for all operations. Above the list, filter controls allow searching by genre
and title.

When a movie is selected from the list, the following actions become available:

- **Rent** — opens the rental form for that movie (Flow 1)
- **Copies** — opens the copy management form for that movie (Flow 5)
- **Edit** — opens the movie edit form (Flow 4)

---

### Flow 1: Rent a Movie

**Trigger:** A customer brings a physical copy to the counter.

**Steps:**
1. The employee enters the copy code in the search field on the main screen.
   The copy code identifies both the movie and the specific copy
   (e.g. 99901 = movie 999, copy 01).
2. The system displays the movie details and confirms the copy is available.
3. The employee clicks "Rent".
4. A form opens to select the customer.
   The employee searches by name or customer code and selects the customer.
5. The system checks the customer's rental status:
   - If the customer has overdue returns, a warning is displayed.
   - If the customer has reached the maximum number of active rentals,
     a warning is displayed.
   In both cases, the employee may choose to cancel or proceed anyway.
   This override exists by design — small stores often serve known customers
   and the employee has discretion.
6. The employee confirms. The rental is registered with the current date
   and expected return date.

**Outcome:** The rental is recorded. The copy status changes to Rented.

**Error case:** If the copy code is not found or the copy is marked as
unavailable, the system shows an error and does not proceed.
A copy in unavailable state should not be on the shelf — this indicates
a data inconsistency that the employee must resolve manually.

---

### Flow 2: Search / Browse Movies

**Trigger:** A customer asks about a movie by title or genre,
or the employee needs to check availability.

**Steps:**
1. On the main screen, the employee uses the filter controls above the movie list:
   - A genre dropdown to filter by category (optional).
   - A free-text field to filter by title or movie code (optional).
2. The employee clicks "Search". The list updates to show matching movies.
3. The employee selects a movie from the list to view its details:
   available copies, genre, and rental price.
4. If at least one copy is available, the employee informs the customer
   and the customer retrieves the physical VHS box from the shelf.
5. The employee may then initiate a rental directly from the movie detail view
   by entering or confirming the specific copy code brought to the counter.

**Outcome:** The employee has located the movie and confirmed availability.
If the customer retrieves a copy, the flow continues into Flow 1 (Rent a Movie).

**Notes:**
- Both filters are optional. Submitting with no filters resets the list
  to show all movies.
- Availability shown in the list reflects current data.
  If a copy is marked unavailable, it does not count as available
  even if its box is physically on the shelf.

### Flow 3: Return a Movie

**Trigger:** A customer comes to the counter to return a rented copy.

**Steps:**
1. The employee enters the copy code.
2. The system retrieves the active rental for that copy and displays:
   - Movie title and copy code.
   - Customer name.
   - Rental date and expected return date.
   - Calculated charge (see Business Rules for day-counting logic).
   - Late fee surcharge, if applicable.
   - Total amount due.
3. The employee informs the customer of the total amount due.
4. The employee confirms the return. The rental is closed.

**Outcome:** The rental is marked as returned. The copy status
changes back to Available.

**Notes:**
- The system displays the amount due for informational purposes only.
  No payment is recorded. There is no cash register concept in this stage.
- If no active rental exists for the copy code, the system shows an error.

### Flow 4: Manage Movies

**Trigger:** The employee needs to add a new movie to the catalog
or update an existing movie's information.

#### Add Movie
1. The employee opens the Add Movie form from the main menu.
2. The employee enters:
   - Title (required)
   - Genre (required, selected from a predefined list)
   - Summary (optional)
3. The employee confirms. The system assigns a sequential numeric ID
   and saves the movie.
4. The new movie appears in the main list.

#### Edit Movie
1. The employee selects a movie from the main list and opens the Edit form.
2. The employee may update Title, Genre, and/or Summary.
3. The employee confirms. Changes are saved.

**Notes:**
- The movie ID is assigned by the system. It is a sequential number,
  not zero-padded (e.g. 1, 2, 10, 100).
- Movies cannot be deleted. A movie with no active copies
  is effectively inactive but remains in the catalog.
- Price is not set per movie. A single daily rental rate applies
  to all movies and is configured at the system level.

### Flow 5: Manage Copies

**Trigger:** The employee needs to add a new copy to a movie
or update the status of an existing one.

**Access:** From the main screen, select a movie and click "Copies".
A modal form opens showing all copies for that movie and their current status.

#### Add Copy
1. The employee clicks "Add Copy" in the copies form.
2. The system assigns the next sequential copy code for that movie
   (e.g. for movie 10: 1001, 1002, 1003...).
3. The new copy is saved with status Available and no comment.
4. It appears immediately in the copies list.

#### Mark Copy as Unavailable
1. The employee selects a copy from the list.
2. The employee clicks "Mark Unavailable".
3. The employee may enter an optional short comment (e.g. "lost", "damaged").
4. The copy status changes to Unavailable.

**Copy fields:**
- Code (system-assigned: movie ID + 2-digit zero-padded sequence)
- Status: Available | Rented | Unavailable
- Comment (optional free text, e.g. "lost", "damaged")

**Notes:**
- Copies cannot be deleted.
- A copy with status Rented can be marked Unavailable directly.
  The system will display a warning indicating that the active rental
  must be resolved and that an additional damage or loss fee should be
  charged to the customer. The fee amount and recovery process are
  handled outside the system.
- The Rented status is set by the system, not by the employee directly.

### Flow 6: Manage Customers

**Trigger:** The employee needs to register a new customer, update
their information, or review their rental activity.

**Access:** From the main menu. Opens a single customer form.

The form is a master/detail layout:
- **Top section:** customer data fields.
- **Bottom section:** rental grid, toggled between Active Rentals
  and Full History via a selector.

#### Find and Edit Customer
1. The employee enters a customer ID or name in the search field and confirms.
2. The system loads the customer's data into the form:
   - Customer ID (read-only, system-assigned)
   - Name
   - Phone (optional)
   - Address (optional)
3. The rental grid loads with active rentals by default (movie title,
   copy code, rental date, expected return date).
4. The employee may toggle the selector to view full history
   (same columns, actual return date instead of expected).
5. The employee may edit customer fields and click Save.

#### Add New Customer
1. The employee clicks "New".
2. The form clears. A new system-assigned customer ID appears (read-only).
3. The employee enters Name, and optionally Phone and Address.
4. The employee clicks Save. The customer is registered.

**Notes:**
- Customers cannot be deleted.
- Email is out of scope for this stage.
- The customer ID is assigned by the system, sequential, not zero-padded.

---

## 4. Business Rules

### BR-1: Rental Day Counting
Rental duration is counted in calendar days, with a minimum of 1 day.
- Returning on the same day as rental = 1 day.
- Returning the following day = 1 day.
- Returning two days later = 2 days.
- Formula: `max(1, return_date − rental_date)` in calendar days.

### BR-2: Rental Fee Calculation
- Days within the allowed rental period: `days × daily_rate`.
- Days beyond the allowed period: `allowed_days × daily_rate`
  + `late_days × late_daily_rate`.
- Late rate is higher than the standard daily rate.
- The total is displayed at return time. It is informational only —
  no payment is recorded in this stage.

### BR-3: Configurable Parameters
The following values are defined in a configuration file (INI or DB).
They are not editable through the application UI.

| Parameter | Description |
|-----------|-------------|
| `daily_rate` | Standard rental price per day (applies to all movies) |
| `late_daily_rate` | Price per day for each day beyond the allowed period |
| `max_rental_days` | Maximum number of days before a rental is considered late |
| `max_active_rentals` | Maximum number of simultaneous active rentals per customer |

### BR-4: Customer Rental Warnings
When registering a rental, the system checks the customer's status
and displays a warning if either condition is met:
- The customer has one or more overdue active rentals.
- The customer has reached the `max_active_rentals` limit.

Warnings do not block the transaction. The employee may override
and proceed. This is by design — small stores operate on trust
and employee discretion.

### BR-5: Copy Code Format
A copy code is composed of the movie ID followed by a 2-digit
zero-padded copy sequence number, concatenated with no separator.

Examples:
- Movie 1, first copy → 101
- Movie 1, twelfth copy → 112
- Movie 10, first copy → 1001
- Movie 100, first copy → 10001

Copy sequence numbers are assigned by the system at creation time
and are never reused within the same movie.

### BR-6: Genre List
The list of available genres is predefined and stored in the database
or configuration file. It is not hardcoded in the application.
There is no UI to add or modify genres in this stage.

---

## 5. Edge Cases

### EC-1: Renting a copy with status Unavailable
The employee enters a copy code that exists but is marked Unavailable.
The system displays an error. The rental cannot proceed.
This indicates a process error — the physical box should not be on the shelf.

### EC-2: Renting a copy already Rented
The employee enters a copy code that is currently active in another rental.
The system displays an error. The rental cannot proceed.

### EC-3: Returning a copy with no active rental
The employee enters a copy code at return time and no active rental exists.
- If the copy exists and is marked Unavailable: the system marks it
  Available and confirms. This covers the case of a previously lost
  or damaged copy being found and returned.
- If the copy code does not exist: the system displays an error.

### EC-4: Customer has overdue rentals — employee proceeds
The system displays a warning. The employee may choose to cancel or
proceed. If the employee proceeds, the rental is registered normally.
No additional restriction is applied.

### EC-5: Customer has reached the active rental limit — employee proceeds
Same behavior as EC-4. Warning displayed, employee has full discretion
to override and proceed.

### EC-6: Marking a Rented copy as Unavailable
The employee marks a copy as Unavailable while it has an active rental
(damage or loss scenario). The system displays a warning indicating
that an additional fee must be charged to the customer. The rental is
closed and the copy is marked Unavailable. Fee handling is done
outside the system.

### EC-7: Attempting to return a copy marked Unavailable
Same behavior as EC-3: if the copy exists and is Unavailable, it is
marked Available. If the copy does not exist, an error is shown.

### EC-8: Movie with no available copies
The main list displays the number of available copies per movie.
If a movie has zero available copies, the Rent action is disabled.
The employee can still view details and manage copies.
In normal operation this edge case should not arise —
if no copies are available, no physical box would be on the shelf.

---

## 6. Constraints

1. **Stock constraint** — access to a movie is limited by the number of
   available physical copies. A rental cannot proceed if no copies
   are available.

2. **Single workstation** — the application runs on a single PC with no
   network connectivity. Multi-user and multi-terminal scenarios are
   out of scope.

3. **No authentication** — there is no login, session management, or
   role-based access control. Any person at the workstation has full
   access to all functions.

4. **No payment processing** — rental charges and late fees are calculated
   and displayed for informational purposes only. No payment is recorded
   or processed by the system.

5. **No reporting** — the system has no reporting, statistics, or analytics
   features. It is strictly transactional.

6. **Genre list not editable via UI** — genres are predefined and stored
   in the database or configuration. No UI exists to add or modify them.

7. **Configuration not editable via UI** — rental rates, late fees, and
   limits are defined in a configuration file or database table.
   They cannot be changed through the application interface.
