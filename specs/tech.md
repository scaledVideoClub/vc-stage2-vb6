# Tech Spec — Stage 2: Visual Basic 6
# VCM-v2 — Video Club Manager

> This document defines how the system is built.
> It is derived from prd.md and domain.md. When in conflict, prd.md wins.
> When in conflict with master-context.md, that wins.

---

## 1. Architecture Style

VCM-v2 is a **single-workstation Windows desktop application** built in
Visual Basic 6. It follows the event-driven paradigm native to VB6:
user interface events on forms directly invoke data access code.

There are **no architectural layers**. Forms contain both UI logic and
database operations. There are no classes acting as services, repositories,
or data transfer objects. No interfaces are defined. There is no separation
between presentation and data logic — this is deliberate and era-correct.

The application starts directly with `frmMain`. The main form opens the
database connection on load and closes it on unload. A single shared module
(`modApp.bas`) holds config variables, utility functions, and the global
connection object.

### What This Means in Practice

- A button click handler may open a Recordset, read or write data, and update
  a control on the same form — all in the same procedure.
- There are no modules named `CustomerService`, `RentalRepository`, or similar.
- Code is organized by form and by one shared module (`modApp.bas`).
- Config values are read at startup from `config.ini` and stored in
  module-level variables in `modApp.bas`. They are not objects or classes.
- Data-bound controls (grids, text boxes) are used for display and simple edits.
  Explicit `gConn.Execute` calls are used only for transactional writes that
  involve business logic (rental registration, return, copy status changes).

### Forbidden Patterns (Era Constraint)

- No layered architecture (services, repositories, DAOs, DTOs)
- No ORM or query builder
- No dependency injection or interface-based design
- No classes used as data containers (use module-level variables or Recordsets)
- No stored procedures
- No networking, HTTP, or web references
- No authentication or session management
- No unit test framework

---

## 2. Data Storage

### Database Engine

**SQL Server 2000 Personal Edition — build 8.00.194**
Installed in the Windows XP SP3 VM. Confirmed in `docs/environment.md`.

**Database name:** `vcm2`

The database and schema are created by running `src/sql/setup.sql`
inside the VM using Query Analyzer or `osql`. No migration system exists.

### Database Creation Script

`src/sql/setup.sql` — run once during environment setup.

```sql
-- Create database
IF NOT EXISTS (
    SELECT name FROM master.dbo.sysdatabases WHERE name = 'vcm2'
)
BEGIN
    CREATE DATABASE vcm2
END
GO

USE vcm2
GO

-- Movies
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name = 'Movies' AND xtype = 'U')
BEGIN
    CREATE TABLE Movies (
        movie_id   INT IDENTITY(1,1) PRIMARY KEY,
        title      VARCHAR(100) NOT NULL,
        genre      VARCHAR(50)  NOT NULL,
        summary    VARCHAR(500) NULL
    )
END
GO

-- Copies
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name = 'Copies' AND xtype = 'U')
BEGIN
    CREATE TABLE Copies (
        copy_id    INT PRIMARY KEY,
        movie_id   INT NOT NULL REFERENCES Movies(movie_id),
        status     VARCHAR(15) NOT NULL
                   CONSTRAINT chk_copy_status
                   CHECK (status IN ('Available', 'Rented', 'Unavailable')),
        comment    VARCHAR(30) NULL
    )
END
GO

-- Customers
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name = 'Customers' AND xtype = 'U')
BEGIN
    CREATE TABLE Customers (
        customer_id INT IDENTITY(1,1) PRIMARY KEY,
        name        VARCHAR(100) NOT NULL,
        phone       VARCHAR(30)  NULL,
        address     VARCHAR(200) NULL
    )
END
GO

-- Rentals
IF NOT EXISTS (SELECT * FROM sysobjects WHERE name = 'Rentals' AND xtype = 'U')
BEGIN
    CREATE TABLE Rentals (
        rental_id       INT IDENTITY(1,1) PRIMARY KEY,
        copy_id         INT NOT NULL REFERENCES Copies(copy_id),
        customer_id     INT NOT NULL REFERENCES Customers(customer_id),
        rental_date     DATETIME NOT NULL,
        expected_return DATETIME NOT NULL,
        return_date     DATETIME NULL
    )
END
GO
```

### Schema Notes

- `copy_id` is not `IDENTITY` — it is computed by the application using
  `movie_id * 100 + sequence`. The application assigns it before INSERT.
- `genre` is a `VARCHAR` on Movies. No Genres table exists. Valid values
  come from `config.ini` and are enforced by the application.
- Dates are stored as `DATETIME`. Only the date portion is meaningful;
  the time component is always midnight (`00:00:00`).
- No monetary values are stored. Charges are computed at runtime from config.

### Configuration File

**Location:** `config.ini` in the application directory (same folder as the `.exe`).

```ini
[Rental]
daily_rate=2.50
late_daily_rate=4.00
max_rental_days=3
max_active_rentals=5

[Genres]
list=Action,Comedy,Drama,Horror,Sci-Fi,Documentary,Animation
```

Read at application startup using the Windows API (`GetPrivateProfileString`,
`GetPrivateProfileInt`). Stored in module-level variables in `modApp.bas`.
Not editable via the UI.

---

## 3. Data Access

**ADO 2.6** (ActiveX Data Objects), referenced as
`Microsoft ActiveX Data Objects 2.6 Library` in the VB6 project references.
ADO 2.6 ships with SQL Server 2000 / MDAC 2.6 and is appropriate for this environment.

### Connection

A single `ADODB.Connection` object (`gConn`) is declared in `modApp.bas`.
It is opened in `frmMain_Load` and closed in `frmMain_QueryUnload`.

**Connection string:**
```
Provider=SQLOLEDB;Data Source=(local);Initial Catalog=vcm2;Integrated Security=SSPI;
```

### Recordset Strategy

- **Data-bound controls** (MSFlexGrid, DataGrid, bound TextBoxes) are used
  for display and simple edits where no business logic is involved.
  These controls connect to a Recordset and reflect changes through the ADO
  cursor without manual SQL for each field update.
- **Explicit `gConn.Execute`** is used for transactional writes with business
  logic: registering a rental, confirming a return, changing copy status.
  These operations require invariant checks and cannot be delegated to a
  bound control's automatic update.
- No `ADODB.Command`. No stored procedures. No parameterized queries.
  String concatenation is used for inline SQL — acceptable for a
  closed single-workstation system.

---

## 4. Application Structure

### Project File

`VCM2.vbp` — the single VB6 project file, located in `src/`.

### Startup

VB6 project startup is set to `frmMain` (not `Sub Main`).
`frmMain_Load` opens the DB connection and loads config.
`frmMain_QueryUnload` closes the connection.

### Forms

| Form file          | Description                                               |
|--------------------|-----------------------------------------------------------|
| `frmMain.frm`      | Main screen — movie list, filters, action buttons         |
| `frmRent.frm`      | Rental form — customer search and rental confirmation     |
| `frmReturn.frm`    | Return form — copy code entry, charge display, confirm    |
| `frmMovieEdit.frm` | Add / Edit movie form                                     |
| `frmCopies.frm`    | Copy management modal — copies list for one movie         |
| `frmCustomer.frm`  | Customer form — master/detail with rental history toggle  |

### Module

| Module file   | Description                                                        |
|---------------|--------------------------------------------------------------------|
| `modApp.bas`  | Global connection (`gConn`), config vars, utility functions        |
|               | (copy code generation, fee calc, Windows API declares, LoadConfig) |

Everything shared lives in `modApp.bas`. No other `.bas` modules.

### Repository Structure

```
/src/
  VCM2.vbp
  frmMain.frm
  frmRent.frm
  frmReturn.frm
  frmMovieEdit.frm
  frmCopies.frm
  frmCustomer.frm
  modApp.bas
  /sql/
    setup.sql
/specs/
  prd.md
  domain.md
  tech.md
  test_cases.md
/docs/
  setup.md
  run.md
  decisions.md
  environment.md
  retrospective.md
  cr-checklist.md
config.ini
CLAUDE.md
README.md
```

---

## 5. Key Flows — Step by Step

### Flow 1: Rent a Movie

**Entry point:** `frmMain` — employee enters copy code and clicks "Rent".

1. Read copy code from text box.
2. Open Recordset: `SELECT copy_id, movie_id, status FROM Copies WHERE copy_id = [code]`
   - Not found → error "Copy not found." Stop.
   - `status = 'Unavailable'` → error "This copy is marked as unavailable." Stop.
   - `status = 'Rented'` → error "This copy is already rented." Stop.
3. Read movie title: `SELECT title FROM Movies WHERE movie_id = [movie_id]`.
4. Set `frmRent.CopyId` and `frmRent.MovieTitle`. Open `frmRent` as modal.
5. In `frmRent`, employee types name or customer ID. Recordset filters
   `Customers` by `name LIKE '%[input]%'` or exact `customer_id`. Results
   shown in a list. Employee selects one.
6. On selection, run warning checks:
   - Overdue: `SELECT COUNT(*) FROM Rentals WHERE customer_id = [id] AND return_date IS NULL AND expected_return < GETDATE()` — if > 0, warn.
   - Limit: `SELECT COUNT(*) FROM Rentals WHERE customer_id = [id] AND return_date IS NULL` — if >= `gMaxActiveRentals`, warn.
   Both warnings are non-blocking. Employee may cancel or proceed.
7. Employee clicks "Confirm Rental":
   - `gConn.Execute "INSERT INTO Rentals (copy_id, customer_id, rental_date, expected_return, return_date) VALUES ([copy_id], [cust_id], GETDATE(), DATEADD(d, [gMaxRentalDays], GETDATE()), NULL)"` 
   - `gConn.Execute "UPDATE Copies SET status = 'Rented' WHERE copy_id = [copy_id]"`
8. Confirmation message. Close `frmRent`. Refresh movie list on `frmMain`.

---

### Flow 2: Search / Browse Movies

**Entry point:** `frmMain` — filter controls above the movie list.

1. Employee selects genre from dropdown (optional) and/or types in free-text field (optional).
2. Clicks "Search". Build SQL:

```sql
SELECT m.movie_id, m.title, m.genre,
       SUM(CASE WHEN c.status = 'Available' THEN 1 ELSE 0 END) AS available_copies
FROM Movies m
LEFT JOIN Copies c ON c.movie_id = m.movie_id
WHERE 1=1
  [AND m.genre = '[genre]']
  [AND (m.title LIKE '%[text]%' OR CAST(m.movie_id AS VARCHAR) = '[text]')]
GROUP BY m.movie_id, m.title, m.genre
```

3. Recordset bound to the main list control. List refreshes.
4. Employee selects a row. Detail area shows genre, summary, available copies.
   "Rent" button disabled if `available_copies = 0`.

---

### Flow 3: Return a Movie

**Entry point:** `frmMain` — employee enters copy code and clicks "Return".

1. Read copy code.
2. Open Recordset:
   `SELECT r.rental_id, r.customer_id, r.rental_date, r.expected_return, c.movie_id FROM Rentals r JOIN Copies c ON c.copy_id = r.copy_id WHERE r.copy_id = [code] AND r.return_date IS NULL`
   - No active rental found:
     - Check `Copies WHERE copy_id = [code]`.
     - If exists and `status = 'Unavailable'`: `UPDATE Copies SET status = 'Available'`. Message: "Copy marked as Available." Stop.
     - If not found: error "Copy not found." Stop.
3. Load `frmReturn` with rental data. Display:
   - Movie title, customer name, rental date, expected return date.
   - `days_rented = max(1, TODAY - rental_date)` (calendar days via `DateDiff("d", ...)`)
   - Fee breakdown (see Section 9).
4. Employee clicks "Confirm Return":
   - `gConn.Execute "UPDATE Rentals SET return_date = GETDATE() WHERE rental_id = [id]"`
   - `gConn.Execute "UPDATE Copies SET status = 'Available' WHERE copy_id = [code]"`
5. Confirmation. Refresh main list.

---

### Flow 4: Manage Movies

**Add Movie:** Main menu → "Add Movie" → `frmMovieEdit` opens blank.

1. Employee fills Title (required), selects Genre from dropdown (populated from `gGenreList`), enters Summary (optional).
2. Clicks "Save". Validate: Title not empty, Genre selected.
3. `gConn.Execute "INSERT INTO Movies (title, genre, summary) VALUES ('[title]', '[genre]', '[summary]')"`
4. Confirmation. Form closes. Main list refreshes.

**Edit Movie:** `frmMain` → select movie → "Edit" → `frmMovieEdit` opens pre-populated.

1. Recordset loads movie data into bound text boxes and dropdown.
2. Employee edits fields. Clicks "Save". Validate Title not empty.
3. `gConn.Execute "UPDATE Movies SET title = '[title]', genre = '[genre]', summary = '[summary]' WHERE movie_id = [id]"`
4. Confirmation. Form closes. Main list refreshes.

---

### Flow 5: Manage Copies

**Entry point:** `frmMain` → select movie → "Copies" → `frmCopies` opens as modal.

**Initial load:** Recordset on `Copies WHERE movie_id = [id] ORDER BY copy_id`.
Bound to a grid. Columns: Copy Code, Status, Comment.

**Add Copy:**
1. Employee clicks "Add Copy".
2. Compute next copy ID (see Section 8).
3. `gConn.Execute "INSERT INTO Copies (copy_id, movie_id, status) VALUES ([new_id], [movie_id], 'Available')"`
4. Grid refreshes.

**Mark Copy as Unavailable:**
1. Employee selects a row. Clicks "Mark Unavailable".
2. If `status = 'Rented'`: warn "This copy has an active rental. The rental will be closed. A damage/loss fee must be charged to the customer outside the system. Proceed?" Employee may cancel.
   If confirmed: `gConn.Execute "UPDATE Rentals SET return_date = GETDATE() WHERE copy_id = [id] AND return_date IS NULL"`
3. Prompt for optional comment.
4. `gConn.Execute "UPDATE Copies SET status = 'Unavailable', comment = '[comment]' WHERE copy_id = [id]"`
5. Grid refreshes.

---

### Flow 6: Manage Customers

**Entry point:** Main menu → "Customers" → `frmCustomer`.

**Find Customer:**
1. Employee types name or ID, clicks "Find".
2. Recordset: `SELECT customer_id, name, phone, address FROM Customers WHERE name LIKE '%[input]%' OR customer_id = [input]`
   - One result: load directly. Multiple: show selection list.
3. Top section (bound text boxes): Customer ID (read-only), Name, Phone, Address.
4. Bottom grid: Active Rentals by default.

```sql
SELECT m.title, c.copy_id, r.rental_date, r.expected_return
FROM Rentals r
JOIN Copies c ON c.copy_id = r.copy_id
JOIN Movies m ON m.movie_id = c.movie_id
WHERE r.customer_id = [id] AND r.return_date IS NULL
```

Toggle to Full History:

```sql
SELECT m.title, c.copy_id, r.rental_date, r.expected_return, r.return_date
FROM Rentals r
JOIN Copies c ON c.copy_id = r.copy_id
JOIN Movies m ON m.movie_id = c.movie_id
WHERE r.customer_id = [id]
ORDER BY r.rental_date DESC
```

5. Employee edits Name/Phone/Address. Clicks "Save":
   `gConn.Execute "UPDATE Customers SET name = '[name]', phone = '[phone]', address = '[address]' WHERE customer_id = [id]"`

**Add New Customer:**
1. Clicks "New". Form clears. Customer ID shows "(auto)".
2. Employee fills Name (required), Phone, Address. Clicks "Save".
3. `gConn.Execute "INSERT INTO Customers (name, phone, address) VALUES ('[name]', '[phone]', '[address]')"`
4. Form reloads with new customer (re-query by MAX(customer_id) to get assigned ID).

---

## 6. Validation Rules

| Rule  | Trigger                    | Check                                              | Response                          |
|-------|----------------------------|----------------------------------------------------|-----------------------------------|
| VAL-1 | Rent — copy code entered   | `status = 'Available'`                             | Error if Rented / Unavailable / not found |
| VAL-2 | Rent — customer selected   | Active rentals < `gMaxActiveRentals`               | Non-blocking warning              |
| VAL-3 | Rent — customer selected   | No active rental with `expected_return < TODAY`    | Non-blocking warning              |
| VAL-4 | Add/Edit Movie — Save      | `title` not empty                                  | Error: "Title is required."       |
| VAL-5 | Add/Edit Movie — Save      | Genre selected (not blank)                         | Error: "Genre is required."       |
| VAL-6 | Genre dropdown — form load | Values from `gGenreList`                           | Dropdown populated; no free entry |
| VAL-7 | Add Customer — Save        | `name` not empty                                   | Error: "Customer name is required." |
| VAL-8 | Mark Copy Unavailable      | Status check before UPDATE                         | Warning if Rented; confirm required |
| VAL-9 | Return — copy code entered | Active rental exists                               | Handle per EC-3/EC-7 if not found |

---

## 7. Error Handling

Every procedure that performs a database operation uses `On Error GoTo`.
No silent failures. If an operation fails, the user is informed.

**Standard pattern:**

```vb
Sub SomeProcedure()
    On Error GoTo ErrHandler
    ' ... normal code ...
    Exit Sub
ErrHandler:
    MsgBox "An error occurred: " & Err.Description, vbExclamation, "Error"
End Sub
```

### User-Facing Messages

| Situation                              | Message                                                                      | Type    |
|----------------------------------------|------------------------------------------------------------------------------|---------|
| Copy code not found                    | "Copy not found."                                                            | Error   |
| Copy is Rented                         | "This copy is already rented."                                               | Error   |
| Copy is Unavailable                    | "This copy is marked as unavailable."                                        | Error   |
| No active rental at return             | Handled per EC-3/EC-7 (see Flow 3)                                           | —       |
| Customer has overdue rentals           | "Warning: this customer has overdue rentals. Proceed?"                       | Warning |
| Customer at rental limit               | "Warning: this customer has reached the rental limit. Proceed?"              | Warning |
| Marking a Rented copy as Unavailable   | "This copy has an active rental. The rental will be closed. A damage/loss fee must be charged to the customer outside the system. Proceed?" | Warning |
| Title missing on movie save            | "Title is required."                                                         | Error   |
| Genre not selected on movie save       | "Genre is required."                                                         | Error   |
| Customer name missing                  | "Customer name is required."                                                 | Error   |
| DB connection failure at startup       | "Cannot connect to database. Check configuration and try again."             | Fatal   |
| config.ini not found at startup        | "config.ini not found. Application cannot start."                           | Fatal   |

Fatal errors display a message box and terminate the application.

---

## 8. Copy Code Generation

`copy_id` is not database-assigned. The application computes it before INSERT.

**Algorithm (in `modApp.bas`):**

```vb
Public Function NextCopyId(movie_id As Integer) As Long
    Dim rs As ADODB.Recordset
    Dim maxCopyId As Long
    Dim sequence As Integer

    Set rs = New ADODB.Recordset
    rs.Open "SELECT MAX(copy_id) FROM Copies WHERE movie_id = " & movie_id, _
            gConn, adOpenStatic, adLockReadOnly

    If IsNull(rs.Fields(0).Value) Then
        sequence = 1
    Else
        maxCopyId = CLng(rs.Fields(0).Value)
        sequence = CInt(maxCopyId Mod 100) + 1
    End If

    rs.Close
    Set rs = Nothing

    NextCopyId = CLng(movie_id) * 100 + sequence
End Function
```

Copy codes are globally unique as long as no movie exceeds 99 copies.
This limit is a known boundary of the encoding scheme and is not enforced
in this stage.

---

## 9. Fee Calculation

**Algorithm (in `modApp.bas`):**

```vb
Public Function CalcRentalFee( _
    rental_date As Date, _
    return_date As Date) As Currency

    Dim days As Integer

    days = DateDiff("d", rental_date, return_date)
    If days < 1 Then days = 1

    If days <= gMaxRentalDays Then
        CalcRentalFee = days * gDailyRate
    Else
        CalcRentalFee = (gMaxRentalDays * gDailyRate) + _
                        ((days - gMaxRentalDays) * gLateDailyRate)
    End If
End Function
```

`frmReturn` calls this function and displays the breakdown:
standard charge, late fee (if any), and total. Informational only — no amount is stored.

---

## 10. Config Loading

**In `modApp.bas`:**

```vb
' Windows API declares
Private Declare Function GetPrivateProfileString Lib "kernel32" _
    Alias "GetPrivateProfileStringA" ( _
    ByVal lpAppName As String, ByVal lpKeyName As String, _
    ByVal lpDefault As String, ByVal lpReturnedString As String, _
    ByVal nSize As Long, ByVal lpFileName As String) As Long

Private Declare Function GetPrivateProfileInt Lib "kernel32" _
    Alias "GetPrivateProfileIntA" ( _
    ByVal lpAppName As String, ByVal lpKeyName As String, _
    ByVal nDefault As Long, ByVal lpFileName As String) As Long

' Global config vars
Public gConn             As ADODB.Connection
Public gDailyRate        As Currency
Public gLateDailyRate    As Currency
Public gMaxRentalDays    As Integer
Public gMaxActiveRentals As Integer
Public gGenreList()      As String

Public Sub LoadConfig()
    Dim iniPath As String
    Dim buf     As String * 512
    Dim genreStr As String

    iniPath = App.Path & "\config.ini"

    If Dir(iniPath) = "" Then
        MsgBox "config.ini not found. Application cannot start.", _
               vbCritical, "Startup Error"
        End
    End If

    Dim dRate As String * 32
    GetPrivateProfileString "Rental", "daily_rate", "2.50", dRate, Len(dRate), iniPath
    gDailyRate = CCur(Left(dRate, InStr(dRate, Chr(0)) - 1))

    Dim lRate As String * 32
    GetPrivateProfileString "Rental", "late_daily_rate", "4.00", lRate, Len(lRate), iniPath
    gLateDailyRate = CCur(Left(lRate, InStr(lRate, Chr(0)) - 1))

    gMaxRentalDays    = GetPrivateProfileInt("Rental", "max_rental_days", 3, iniPath)
    gMaxActiveRentals = GetPrivateProfileInt("Rental", "max_active_rentals", 5, iniPath)

    GetPrivateProfileString "Genres", "list", "", buf, Len(buf), iniPath
    genreStr = Left(buf, InStr(buf, Chr(0)) - 1)
    gGenreList = Split(genreStr, ",")
End Sub
```

`LoadConfig` is called from `frmMain_Load` before opening the connection.

---

## 11. Form Interaction Model

Forms do not call methods on other forms. Data is passed via `Public`
form-level variables set before `frmX.Show vbModal`.

**Example — opening the rental form:**

```vb
' In frmMain:
frmRent.CopyId = lngCopyId
frmRent.MovieTitle = strTitle
frmRent.Show vbModal
' execution resumes here after frmRent closes
```

`frmCopies` and `frmRent` and `frmReturn` are always modal.
`frmMovieEdit` and `frmCustomer` are modal.
`frmMain` is the only non-modal (top-level) form.

After a modal form closes, the calling form refreshes its data if needed
(e.g. re-run the movie list query after a rental or edit).

---

## 12. Era Constraints Summary

| Forbidden                             | Reason                                          |
|---------------------------------------|-------------------------------------------------|
| `Sub Main` as startup                 | Form-based startup is natural for simple VB6 apps |
| Classes acting as services            | Not the VB6 event-driven paradigm               |
| Interfaces or abstract classes        | Future-stage concept                            |
| ORM or query builder                  | Future-stage concept                            |
| `ADODB.Command` / parameterized queries | Unnecessary for this stage and environment    |
| Stored procedures                     | Keep logic in the application for this stage    |
| Multiple `.bas` modules               | One `modApp.bas` is sufficient; avoid over-engineering |
| Networking or HTTP                    | Single-workstation constraint                   |
| Authentication / login forms          | Out of scope for this stage                     |
| Reporting or analytics features       | Out of scope for this stage                     |
| Unit test framework                   | Not introduced until Stage 4                    |
| Genre as a database table             | Genre is a string on Movie; list comes from INI |

---

*Tech spec version: 1.1 — Stage 2 VB6*
*Status: Approved — ready for implementation*
*Environment confirmed: SQL Server 2000 Personal Edition build 8.00.194 (docs/environment.md)*
