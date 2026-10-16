# Phase 3A — Dashboard + Work Orders List
**Assigned to:** Agent D  
**Branch:** `phase/3a-dashboard-workorders`  
**Create from:** `main` after `phase/2a-seed` AND `phase/2b-auth-shell` both merge  
**Can run parallel to:** Phase 3B, Phase 3C  
**Blocks:** Phase 5 (integration)

---

## Objective

Build the Dashboard page and the Work Orders list page. These are read-heavy pages with filtering and search. The dashboard is the first thing staff see every day — it must show the right numbers from the database, not hardcoded values.

---

## Files You Own

```
app/pages/dashboard.py      (create)
app/pages/workorders.py     (create)
```

---

## Files You Must NOT Touch

```
app/main.py         ← you will edit the route bodies inside it, but do not change
                      require_auth(), render_nav(), ui.run(), or first-boot logic
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
Replace only the body of the two placeholder functions in `app/main.py`:
```python
@ui.page('/')
def dashboard(): ...   # replace body only

@ui.page('/workorders')
def workorders(): ...  # replace body only
```
Import your page modules and call them:
```python
from app.pages.dashboard import render_dashboard
from app.pages.workorders import render_workorders

@ui.page('/')
def dashboard(): require_auth(); render_dashboard()

@ui.page('/workorders')
def workorders(): require_auth(); render_workorders()
```

---

## `app/pages/dashboard.py` — Full Specification

### Stat cards row
Query live from DB. Display as a horizontal row of cards:

| Card | Query |
|---|---|
| Total open | `COUNT(*) WHERE status NOT IN ('paid','cancelled','chargeback')` |
| Overdue | `COUNT(*) WHERE deadline_at < now() AND status NOT IN terminal` |
| Due in 48h | `COUNT(*) WHERE deadline_at BETWEEN now() AND now()+48h AND status NOT IN terminal` |
| Due this week | `COUNT(*) WHERE deadline_at BETWEEN now() AND now()+7d AND status NOT IN terminal` |
| Unassigned | `COUNT(*) WHERE status='created'` |

Terminal statuses to exclude: `paid`, `cancelled`, `chargeback`

### "Needs attention" table
Rows: WOs that are overdue OR unassigned (`status='created'`) OR due within 48h — excluding terminal statuses.

Columns: WO#, Address, Task Type, Status, Deadline (formatted in Asia/Dhaka timezone), Assigned Crew (or "—" if none), Client.

Each row is clickable — clicking navigates to `/workorders/{id}`.

Color coding on deadline cell:
- Overdue: red text
- < 24h: orange text  
- < 48h: yellow/amber text

### Recent activity feed
Last 20 rows from `wo_status_history` JOIN `work_orders`, plus last 10 rows from `wo_notes`, merged and sorted by timestamp descending.

Display each item as a single line:
- Status change: `[WO #123] 1402 Elm St → assigned (by admin) — 2h ago`
- Note: `[WO #123] Note by admin — 3h ago`

"X ago" format: use relative time (e.g. "5 min ago", "2h ago", "3 days ago").

### Timezone
All displayed timestamps must be converted to `Asia/Dhaka` (UTC+6). Use `pytz` or `zoneinfo` (stdlib Python 3.9+).

---

## `app/pages/workorders.py` — Full Specification

### Table
Columns: WO#, External WO#, Client, Address, State, Task Type, Status, Deadline, Assigned Crew, Quoted Rate.

- Deadline cell: color-coded same as dashboard (overdue=red, <24h=orange, <48h=yellow)
- Status cell: display as a colored badge (created=grey, assigned=blue, in_progress=orange, results_received=purple, submitted=teal, invoiced=yellow, paid=green, cancelled=grey, chargeback=red)
- All timestamps displayed in Asia/Dhaka timezone
- Default sort: `deadline_at ASC NULLS LAST`
- Row click → navigate to `/workorders/{id}`

### Filters (above the table)
All filters are additive (AND logic):
- Status: dropdown multi-select (all statuses + "All")
- Client: dropdown (all clients + "All")
- State: text input (partial match)
- Crew: dropdown (all crews + "All")
- Deadline window: dropdown ("All", "Overdue", "Due today", "Due in 48h", "Due this week")

### Search
Text input: searches `property_address` (ILIKE) and `external_wo_no` (ILIKE). Clears other filters when used (or applies on top — your choice, but be consistent).

### "New Work Order" button
Opens a dialog with fields:
- Client (dropdown — all clients)
- External WO# (text, optional)
- Property Address (text, required)
- State (text)
- Zip (text)
- Task Type (text, required) — or dropdown: `grass_cut`, `rekey`, `trash_out`, `winterization`, `board_up`, `handyman`, `plumbing`, `occupancy_inspection`, `other`
- Scope (textarea, optional)
- Allowable Amount (number, optional)
- Bid Amount (number, optional)
- Bid Status (dropdown: `none`, `pending`, `approved`, `rejected`)
- Deadline (datetime picker, optional)

On submit: call `services.create_wo()`, close dialog, refresh table.

Validation: Address and Task Type are required. Show inline error if missing.

---

## Gate Condition

- [ ] Dashboard loads with correct stat card numbers matching seed data
- [ ] "Needs attention" table shows WO5 (overdue, submitted) and WO2/WO10 (assigned but check deadline) and WO1 (unassigned/created)
- [ ] Recent activity feed shows entries from seed data
- [ ] Work orders table shows all 10 seed WOs
- [ ] Status filter works — selecting "assigned" shows only WO2 and WO10
- [ ] Search by partial address works
- [ ] "New Work Order" dialog creates a WO and it appears in the table
- [ ] Row click navigates to `/workorders/{id}` (placeholder page is fine — just must navigate)
- [ ] All timestamps display in Asia/Dhaka timezone

---

## Contracts to Respect

- All DB queries go through `get_session()` from `app.db`
- Never hardcode counts or data — everything from DB
- Terminal statuses constant: import `TERMINAL_STATUSES` from `app.fsm`
- Timezone: `zoneinfo.ZoneInfo('Asia/Dhaka')` — no hardcoded UTC+6 offset
- Money displayed with 2 decimal places and `$` prefix
- `require_auth()` is the first call, `render_nav()` is the second
