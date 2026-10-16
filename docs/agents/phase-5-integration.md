# Phase 5 — Integration, README & Final QA
**Assigned to:** Lead  
**Branch:** `phase/5-integration`  
**Create from:** `main` after ALL of phases 3A, 3B, 3C, and 4 merge  
**This is the final phase before v0 is live.**

---

## Objective

Wire all pages into the app shell, run the full test suite, do a complete manual walkthrough, write the production README, and confirm the app is ready to deploy on the VPS.

---

## Files You Own (this phase)

```
app/main.py         ← final wiring of all page imports
README.md           ← full production README (replace placeholder)
```

---

## Step 1 — Final main.py Wiring

Confirm all 6 page routes are wired to their implementations (not placeholders). The final `app/main.py` route section should look like:

```python
from app.pages.dashboard import render_dashboard
from app.pages.workorders import render_workorders
from app.pages.wo_detail import render_wo_detail
from app.pages.crews import render_crews
from app.pages.deadlines import render_deadlines
from app.pages.payments import render_payments

@ui.page('/')
def dashboard(): require_auth(); render_nav(); render_dashboard()

@ui.page('/workorders')
def workorders(): require_auth(); render_nav(); render_workorders()

@ui.page('/workorders/{id}')
def wo_detail(id: int): require_auth(); render_nav(); render_wo_detail(id)

@ui.page('/crews')
def crews(): require_auth(); render_nav(); render_crews()

@ui.page('/crews/{id}')
def crew_detail(id: int): require_auth(); render_nav(); render_crew_detail(id)

@ui.page('/deadlines')
def deadlines(): require_auth(); render_nav(); render_deadlines()

@ui.page('/payments')
def payments(): require_auth(); render_nav(); render_payments()
```

Check that `render_nav()` is not being called twice (once in main.py and once inside the page function). Standardize: call it in main.py only, remove from page functions if duplicated.

---

## Step 2 — Run Full Test Suite

```bash
docker compose up -d postgres
export DATABASE_URL=postgresql://thesora:password@localhost:5432/thesora_test
pytest tests/ -v --tb=short
```

All tests must pass:
- [ ] `tests/test_fsm.py` — full transition matrix
- [ ] `tests/test_services.py` — tests (a) and (b)
- [ ] `tests/test_payments.py` — test (c) concurrent batch
- [ ] End-to-end lifecycle test

If any test fails: fix before proceeding. Do not ship with failing tests.

---

## Step 3 — Fresh Database Smoke Test

```bash
docker compose down -v          # wipe all volumes
docker compose up -d
docker compose exec app alembic upgrade head
docker compose exec app python -m app.seed
```

- [ ] Seed runs without errors
- [ ] Admin password printed to console
- [ ] App accessible at http://localhost:8080
- [ ] Login with printed password works

---

## Step 4 — Manual Walkthrough Checklist

Work through this sequence on the seeded database:

### Dashboard
- [ ] Stat cards show correct numbers (verify against seed data counts)
- [ ] "Needs attention" table shows WO1 (unassigned), WO5 (overdue)
- [ ] Recent activity feed shows entries
- [ ] All timestamps in Asia/Dhaka timezone (UTC+6 — verify: a UTC midnight event shows as 6:00 AM BDT)

### Work Orders List
- [ ] All 10 seed WOs visible
- [ ] Status filter: select "assigned" → shows WO2 and WO10 only
- [ ] Search "Elm" → shows WO1 (1402 Elm St)
- [ ] Create new WO via dialog → appears in table
- [ ] Row click → navigates to WO detail

### WO Detail
- [ ] Open WO2 (assigned) — fields display correctly
- [ ] Status buttons show only "Mark In Progress" and "Cancel WO"
- [ ] Click "Mark In Progress" → status changes, history timeline updates
- [ ] Open WO1 (created) — "Assign Crew" button visible
- [ ] Assign Crew 4 (Okafor) with rate $70 → WO transitions to assigned
- [ ] Open WO3 (in_progress) — "Mark Results Received" button visible
- [ ] Click it → WO transitions, assignment ended
- [ ] Job-cost strip: WO4 shows $140.00 (bid $320 - rate $180)
- [ ] Job-cost strip: WO1 (no bid/allowable) shows "Estimate unavailable"
- [ ] Add a note → appears in thread
- [ ] Upload a photo → thumbnail appears
- [ ] Upload >10MB file → error shown, no upload
- [ ] Upload .pdf → rejected

### Crews
- [ ] All 5 crews visible
- [ ] Webb (COI in 5 days) → red COI cell
- [ ] Simmons (COI expired) → red COI cell
- [ ] Okafor (COI 365 days) → green COI cell
- [ ] Add new crew → appears in table
- [ ] Edit crew → changes saved
- [ ] Click "View WOs" for Okafor → shows their WO history

### Deadlines Board
- [ ] WO5 (overdue) → red row
- [ ] WOs due within 24h → orange row
- [ ] WOs due within 48h → yellow row
- [ ] Row click → navigates to WO detail

### Payments
- [ ] WO6 (invoiced) appears in batch preview under Webb
- [ ] Exceptions section shows "None" (or any legitimate exceptions)
- [ ] Click "Mark Batch Paid" for Webb → WO6 moves to paid, payment row appears in history
- [ ] Payment history shows WO7's seed payment (status: sent)

### Auth
- [ ] Logout → redirected to login
- [ ] Accessing `/` without login → redirected to login
- [ ] `python -m app.reset_admin` → password reset works

---

## Step 5 — Write README.md

Replace the placeholder README with the full production README. Must include all 4 required sections:

### 1. Run Instructions
```markdown
## Running the App

**Prerequisites:** Docker + Docker Compose

```bash
# 1. Clone the repo
git clone <repo-url>
cd thesora-llc

# 2. Copy and configure environment
cp .env.example .env
# Edit .env: set POSTGRES_PASSWORD and SECRET_KEY

# 3. Start services
docker compose up -d

# 4. Run database migrations
docker compose exec app alembic upgrade head

# 5. Seed initial data (first run only)
docker compose exec app python -m app.seed
# Note the admin password printed to console

# 6. Open the app
# http://localhost:8080
```

**Reset admin password:**
```bash
docker compose exec app python -m app.reset_admin
```
```

### 2. Security Note
```markdown
## Security

This is an internal tool. It must run behind a VPN, SSH tunnel, or firewall.
**Never expose port 8080 to the public internet without HTTPS.**

HTTPS (via Caddy reverse proxy) is planned for v1. Until then:
- Access via SSH tunnel: `ssh -L 8080:localhost:8080 user@your-vps`
- Or restrict port 8080 to VPN IP range in your firewall rules

Change `SECRET_KEY` in `.env` before first use. The default value is insecure.
```

### 3. Backup & Restore
```markdown
## Backup & Restore

### Nightly backup (add to cron)
```bash
docker compose exec -T postgres pg_dump -U thesora thesora \
  | gzip > backups/thesora-$(date +%F).sql.gz

rsync -az ./data/photos/ user@offsite:/backups/thesora-photos/
```

### Weekly restore verification
```bash
# Restore into a scratch database to verify backup integrity
gunzip -c backups/thesora-YYYY-MM-DD.sql.gz \
  | docker compose exec -T postgres psql -U thesora thesora_verify
```

### Monthly full restore drill
Run before going live and monthly thereafter:
```bash
docker compose down -v
docker compose up -d postgres
docker compose exec app alembic upgrade head
gunzip -c backups/thesora-YYYY-MM-DD.sql.gz \
  | docker compose exec -T postgres psql -U thesora thesora
docker compose up -d app
# Verify app loads and data is correct
```
```

### 4. Operational Notes
```markdown
## Operational Notes

- **"Paid" status** on a work order means the crew payment batch was sent.
  The payment row's own status ("sent" → "cleared") tracks settlement confirmation.
  Update payment status to "cleared" manually once bank transfer confirms.

- **Job-cost strip** is an estimate. Materials purchased for a job are not tracked in v0.
  Chargebacks are not deducted. Use it for margin awareness, not accounting.

- **State/trade licensing:** The app does not verify that Thesora LLC holds the required
  contractor license for a work order's state and trade type. Staff must verify this
  manually before accepting any work order.

- **Photo retention:** Photos are legal evidence against chargebacks. Never delete the
  `data/photos/` directory. The nightly rsync to offsite storage is not optional.

- **Scrapers and integrations** (Pruvan API, portal scrapers, SMS/Telegram) are v1 features.
  In v0, all work orders are entered manually.
```

---

## Step 6 — Create .env.example

```bash
DATABASE_URL=postgresql://thesora:changeme@postgres:5432/thesora
POSTGRES_PASSWORD=changeme
SECRET_KEY=change-this-to-a-random-string-in-production
```

---

## Gate Condition (ship checklist)

- [ ] All tests pass (`pytest tests/ -v` — zero failures)
- [ ] Fresh DB smoke test passes
- [ ] Full manual walkthrough checklist above — every item checked
- [ ] README.md complete with all 4 required sections
- [ ] `.env.example` committed
- [ ] No `print()` debug statements left in any file
- [ ] No hardcoded passwords, secrets, or credentials in any file
- [ ] `docker compose up` on a clean machine with only `.env` configured brings up a working app

---

## Post-Ship: Handoff Notes for v1 Planning

Document these as GitHub issues before closing out v0:

1. `[v1]` Replace `storage.py` filesystem with MinIO S3-compatible storage
2. `[v1]` Add Caddy reverse proxy + HTTPS
3. `[v1]` Pruvan API integration for photo pull
4. `[v1]` Portal scrapers (Playwright) with three-layer fallback
5. `[v1]` TaskIQ background jobs: deadline alerts, COI expiry scan, AR dunning
6. `[v1]` Telegram bot alerts for new WOs and deadline breaches
7. `[v1]` SMS/WhatsApp dispatch via Twilio or similar
8. `[v1]` Client CRUD page
9. `[v1]` PDF report generation for WO summaries
10. `[v1]` EXIF metadata extraction on photo upload
