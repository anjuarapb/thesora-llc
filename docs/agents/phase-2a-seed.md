# Phase 2A — Seed Script
**Assigned to:** Agent B  
**Branch:** `phase/2a-seed`  
**Create from:** `main` after `phase/1-fsm-services` merges  
**Blocks:** Phases 3A, 3B, 3C, 4 — all pages depend on seed data to render correctly.

---

## Objective

Write the seed script that populates the database with realistic, fully-legal data covering all statuses, all invariants, and all edge cases the UI pages need to display. The seed must be idempotent — running it twice must not duplicate rows.

---

## Files You Own

```
app/seed.py     (create)
```

---

## Files You Must NOT Touch

```
app/models.py
app/db.py
app/fsm.py
app/services.py
app/storage.py
app/__init__.py
docker-compose.yml
requirements.txt
tests/
```

---

## `app/seed.py` — Full Specification

Run as: `python -m app.seed`

### Idempotency rule
Check if data already exists before inserting. If `clients` table has rows, exit cleanly with a message. Never duplicate seed data.

### Admin user
- If `users` table is empty: generate a random 16-character alphanumeric password
- Hash with `bcrypt`
- Insert `User(username='admin', password_hash=hashed, name='Admin')`
- Print to console: `[SEED] Admin password: <password>` — print once only
- If `users` table already has rows: skip, do not reprint password

### 2 Clients
```
Client 1: name="DMG Pro",  portal_type="DMG",   payment_terms_days=30
Client 2: name="MCS360",   portal_type="MCS",   payment_terms_days=45
```

### 5 Crews
Include varied trades, zip codes, COI expiry dates. One crew must have COI expiring in 5 days from seed run date.

```
Crew 1:
  name="Marcus Webb", company_name="Webb Property Services LLC"
  phone="555-201-0001", whatsapp="555-201-0001"
  states_served=["FL","GA"], zips_served=["32801","32802","30301"]
  trades=["grass_cut","trash_out","board_up"]
  rate_card={"grass_cut": 45, "trash_out": 200, "board_up": 75}
  coi_expires=<today + 5 days>   ← the expiring-soon crew
  w9_on_file=True, rating=4.20, jobs_completed=34, on_time_pct=94.10

Crew 2:
  name="Darnell Price", company_name="Price Field Solutions"
  phone="555-202-0002", whatsapp="555-202-0002"
  states_served=["TX","OK"], zips_served=["75201","73101"]
  trades=["winterization","plumbing","rekey"]
  rate_card={"winterization": 95, "rekey": 65}
  coi_expires=<today + 120 days>
  w9_on_file=True, rating=4.50, jobs_completed=61, on_time_pct=98.20

Crew 3:
  name="Rosa Fuentes", company_name="Fuentes Maintenance Co"
  phone="555-203-0003", whatsapp="555-203-0003"
  states_served=["AZ","NM"], zips_served=["85001","87101"]
  trades=["grass_cut","trash_out","handyman"]
  rate_card={"grass_cut": 50, "trash_out": 180, "handyman": 85}
  coi_expires=<today + 200 days>
  w9_on_file=False, rating=3.80, jobs_completed=18, on_time_pct=88.90

Crew 4:
  name="James Okafor", company_name="Okafor Property Care"
  phone="555-204-0004", whatsapp="555-204-0004"
  states_served=["OH","MI","IN"], zips_served=["43201","48201","46201"]
  trades=["board_up","rekey","winterization","grass_cut"]
  rate_card={"board_up": 80, "rekey": 70, "winterization": 100, "grass_cut": 40}
  coi_expires=<today + 365 days>
  w9_on_file=True, rating=4.70, jobs_completed=92, on_time_pct=97.80

Crew 5:
  name="Tanya Simmons", company_name="Simmons Contracting LLC"
  phone="555-205-0005", whatsapp="555-205-0005"
  states_served=["NC","SC","VA"], zips_served=["27601","29201","23219"]
  trades=["trash_out","handyman","plumbing"]
  rate_card={"trash_out": 220, "handyman": 90, "plumbing": 130}
  coi_expires=<today - 10 days>   ← already expired, for UI red-flag testing
  w9_on_file=True, rating=3.20, jobs_completed=9, on_time_pct=77.80
```

### 10 Work Orders — one per status

Each WO must have a **full legal-transition history chain** built by calling `services.py` functions and `fsm.transition()` — never by directly setting `status`. Timestamps must be spaced realistically (hours apart, not identical).

Use `datetime.now(timezone.utc) - timedelta(days=N, hours=H)` to backdate.

```
WO 1  — status: created
  client=DMG Pro, address="1402 Elm St, Orlando FL 32801"
  task_type="grass_cut", deadline=<now + 72h>
  History: NULL→created

WO 2  — status: assigned
  client=DMG Pro, address="887 Pine Ave, Atlanta GA 30301"
  task_type="rekey", deadline=<now + 48h>
  crew=Crew 4 (Okafor), quoted_rate=70.00
  History: NULL→created → created→assigned
  Assignment: active (ended_at=NULL)

WO 3  — status: in_progress
  client=MCS360, address="334 Oak Blvd, Dallas TX 75201"
  task_type="winterization", deadline=<now + 36h>
  crew=Crew 2 (Price), quoted_rate=95.00
  History: NULL→created → created→assigned → assigned→in_progress
  Assignment: active (ended_at=NULL)

WO 4  — status: results_received
  client=MCS360, address="91 Maple Dr, Phoenix AZ 85001"
  task_type="trash_out", deadline=<now + 24h>
  bid_amount=320.00
  crew=Crew 3 (Fuentes), quoted_rate=180.00
  History: NULL→created → ... → in_progress→results_received
  Assignment: ended (ended_reason='completed')

WO 5  — status: submitted
  client=DMG Pro, address="2201 Cedar Ln, Columbus OH 43201"
  task_type="board_up", deadline=<now - 2h>   ← past deadline, for overdue testing
  allowable_amount=250.00
  crew=Crew 4 (Okafor), quoted_rate=80.00
  History: NULL→created → ... → results_received→submitted
  Assignment: ended (ended_reason='completed')

WO 6  — status: invoiced
  client=MCS360, address="560 Birch Ct, Charlotte NC 27601"
  task_type="grass_cut", deadline=<now - 48h>
  allowable_amount=150.00
  crew=Crew 1 (Webb), quoted_rate=45.00
  History: NULL→created → ... → submitted→invoiced
  Assignment: ended (ended_reason='completed')

WO 7  — status: paid
  client=DMG Pro, address="77 Walnut St, Detroit MI 48201"
  task_type="rekey", deadline=<now - 96h>
  allowable_amount=180.00
  crew=Crew 4 (Okafor), quoted_rate=70.00
  History: NULL→created → ... → invoiced→paid
  Assignment: ended (ended_reason='completed')
  Payment: payments(status='sent') + payment_items row

WO 8  — status: cancelled
  client=DMG Pro, address="403 Spruce Way, Indianapolis IN 46201"
  task_type="handyman", deadline=<now + 12h>
  History: NULL→created → created→cancelled

WO 9  — status: chargeback
  client=MCS360, address="1100 Ash Rd, Albuquerque NM 87101"
  task_type="winterization", deadline=<now - 120h>
  allowable_amount=200.00
  crew=Crew 2 (Price), quoted_rate=95.00
  chargeback_flag=True
  History: NULL→created → ... → invoiced→chargeback
  Assignment: ended (ended_reason='completed')

WO 10 — status: assigned (second assigned WO for dashboard unassigned/assigned count testing)
  client=MCS360, address="299 Poplar Ave, Richmond VA 23219"
  task_type="trash_out", deadline=<now + 96h>
  bid_amount=400.00, bid_status='pending'
  crew=Crew 5 (Simmons), quoted_rate=220.00
  History: NULL→created → created→assigned
  Assignment: active (ended_at=NULL)
```

### Payment rows for WO 7
After WO 7 reaches `paid` via `batch_pay()`:
- `payments`: `crew_id=Crew4.id, batch_date=<WO7 paid date>, amount=70.00, status='sent'`
- `payment_items`: `payment_id=<above>, wo_id=WO7.id, amount=70.00`

---

## Gate Condition

```bash
python -m app.seed
```

- [ ] Runs without errors on a fresh database
- [ ] Running a second time exits cleanly with "already seeded" message, no duplicate rows
- [ ] `SELECT status, count(*) FROM work_orders GROUP BY status` shows all 9 statuses covered
- [ ] `SELECT * FROM assignments WHERE ended_at IS NULL` shows exactly 3 rows (WO2, WO3, WO10)
- [ ] `SELECT * FROM payments` shows 1 row with `status='sent'`
- [ ] Crew 1 (Webb) has `coi_expires` within 5 days of today
- [ ] Crew 5 (Simmons) has `coi_expires` in the past

---

## Contracts to Respect

- Never set `work_orders.status` directly — always use `services.create_wo()`, `services.assign_crew()`, `services.complete_results()`, or `fsm.transition()`
- All timestamps UTC, spaced at least 1 hour apart per transition
- Commit after each WO is fully built (not one giant commit at the end) — easier to debug if seed fails midway
- Money as `Decimal`, never float literals — use `Decimal('45.00')` not `45.0`
