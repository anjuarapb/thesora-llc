# Phase 3C — Crews + Deadlines Board
**Assigned to:** Agent F  
**Branch:** `phase/3c-crews-deadlines`  
**Create from:** `main` after `phase/2a-seed` AND `phase/2b-auth-shell` both merge  
**Can run parallel to:** Phase 3A, Phase 3B  
**Blocks:** Phase 5 (integration)

---

## Objective

Build the Crews management page (full CRUD) and the Deadlines Board (read-only, color-coded urgency view). These are independent of each other and of the WO detail page — build both in this phase.

---

## Files You Own

```
app/pages/crews.py          (create)
app/pages/deadlines.py      (create)
```

---

## Files You Must NOT Touch

```
app/main.py         ← replace only the bodies of the crews and deadlines route functions
app/models.py
app/db.py
app/fsm.py
app/services.py
app/storage.py
tests/
docker-compose.yml
requirements.txt
```

### How to plug into main.py
```python
from app.pages.crews import render_crews
from app.pages.deadlines import render_deadlines

@ui.page('/crews')
def crews(): require_auth(); render_crews()

@ui.page('/deadlines')
def deadlines(): require_auth(); render_deadlines()
```

---

## `app/pages/crews.py` — Full Specification

### Crews table
Columns: Name, Company, Phone, States Served, Trades, COI Expires, W-9, Rating, Jobs, On-Time %, Active, Actions.

- COI Expires cell: red background (or red text) if expired OR less than 14 days from today. Green if >14 days. Grey/dash if NULL.
- W-9 cell: green checkmark if `True`, red X if `False`
- Active cell: green "Active" badge or grey "Inactive" badge
- Rating: display as `4.20 ★`
- Actions column: "Edit" button + "View WOs" link per row

Default sort: `name ASC`

### "Add Crew" button
Opens a dialog with all crew fields:
- Name (text, required)
- Company Name (text)
- Phone (text)
- WhatsApp (text)
- States Served (comma-separated text input → stored as array)
- Zips Served (comma-separated text input → stored as array)
- Trades (multi-select or comma-separated: `grass_cut`, `rekey`, `trash_out`, `winterization`, `board_up`, `handyman`, `plumbing`, `occupancy_inspection`)
- Rate Card (key-value pairs — simplest approach: textarea accepting JSON, or dynamic add-row UI)
- COI Expires (date picker)
- W-9 On File (checkbox)
- Rating (number 0.00–5.00)
- Active (checkbox, default true)

On submit: insert `Crew` row, close dialog, refresh table.

### Edit dialog
Same fields as Add, pre-populated with existing values. On save: update row, refresh table.

### Crew detail sub-page (`/crews/{id}`)
Register this route in `main.py` as well (add a placeholder if not already there).

Display:
- All crew fields (read-only, with an "Edit" button that opens the edit dialog)
- WO history table: all `work_orders` where the crew has an `assignment` row
  - Columns: WO#, Address, Task Type, Status, Quoted Rate, Assigned At, Ended At, Ended Reason
  - Sort: `assigned_at DESC`
- Performance summary: jobs completed, on-time %, rating, chargeback count (count of WOs with `chargeback_flag=True` assigned to this crew)

---

## `app/pages/deadlines.py` — Full Specification

### Table
Query: all `work_orders` where:
- `status NOT IN ('paid', 'cancelled', 'chargeback')`
- `deadline_at IS NOT NULL`

Sort: `deadline_at ASC` (most urgent first)

Columns: WO#, Address, Task Type, Status, Client, Assigned Crew (or "Unassigned"), Deadline (Asia/Dhaka timezone), Time Remaining.

### Row color coding
Apply to the entire row background:

| Condition | Color |
|---|---|
| `deadline_at < now()` | Red (`bg-red-100` or similar) |
| `deadline_at < now() + 24h` | Orange (`bg-orange-100`) |
| `deadline_at < now() + 48h` | Yellow (`bg-yellow-100`) |
| Otherwise | No color (default) |

### Time Remaining column
- Overdue: `"OVERDUE (Xh Ym ago)"`
- Future: `"Xh Ym remaining"`
- Calculate from `now()` in UTC, display in hours and minutes

### Row click
Clicking any row navigates to `/workorders/{id}`.

### Auto-refresh
Refresh the table every 5 minutes automatically (NiceGUI `ui.timer`). Display last-refreshed timestamp at top right.

---

## Gate Condition

- [ ] Crews table shows all 5 seed crews
- [ ] Crew 1 (Webb, COI expiring in 5 days) shows red COI cell
- [ ] Crew 5 (Simmons, COI expired) shows red COI cell
- [ ] Crew 4 (Okafor, COI 365 days) shows green COI cell
- [ ] "Add Crew" dialog creates a new crew and it appears in the table
- [ ] "Edit" dialog pre-populates fields and saves changes correctly
- [ ] `/crews/{id}` shows crew detail and their WO history
- [ ] Deadlines board shows all non-terminal WOs with deadlines from seed data
- [ ] WO5 (overdue, submitted) shows red row
- [ ] WO2 (due in 48h) shows yellow row (verify against seed deadline)
- [ ] Row click on deadlines board navigates to correct WO detail page
- [ ] All timestamps in Asia/Dhaka timezone

---

## Contracts to Respect

- `require_auth()` first, `render_nav()` second on every page
- COI expiry threshold: 14 days — import as a constant, do not hardcode `14` in multiple places
- Timezone: `zoneinfo.ZoneInfo('Asia/Dhaka')`
- Array fields (states, zips, trades): stored as PostgreSQL arrays — when reading from a comma-separated input, split on `,` and strip whitespace before saving
- Rate card: stored as JSONB — validate it's valid JSON before saving, show error if not
- Never query `work_orders` directly for crew WO history — join through `assignments` table
