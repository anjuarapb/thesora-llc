# Phase 2B — Auth + App Shell
**Assigned to:** Agent C  
**Branch:** `phase/2b-auth-shell`  
**Create from:** `main` after `phase/0-scaffold` merges  
**Can run parallel to:** Phase 1 and Phase 2A  
**Blocks:** All page phases (3A, 3B, 3C, 4) — they all need the app shell and session state.

---

## Objective

Build the NiceGUI application entry point, the login wall, session management, and the admin CLI reset tool. Pages are NOT built here — only the shell that all pages will plug into.

---

## Files You Own

```
app/main.py         (create)
app/reset_admin.py  (create)
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
alembic/
```

---

## `app/main.py` — Full Specification

### Framework
NiceGUI. Entry point: `python -m app.main` (or `uvicorn app.main:app` — follow NiceGUI's standard pattern).

### First-boot admin creation
On startup, before the app serves any request:
1. Open a DB session
2. Query `users` table — if zero rows exist:
   - Generate a random 16-character alphanumeric password (`secrets.token_urlsafe(12)`)
   - Hash with `bcrypt`
   - Insert `User(username='admin', password_hash=hashed, name='Admin')`
   - Commit
   - Print to console: `[BOOT] Admin created. Password: <password>` — this is the only time it prints
3. If users already exist: do nothing, print nothing

### Login page (`/login`)
- NiceGUI page at route `/login`
- Username input + password input (type=password) + "Sign in" button
- On submit: query DB for user by username, verify password with `bcrypt.checkpw()`
- On success: store in `app.storage.user['username']` and `app.storage.user['name']`, redirect to `/`
- On failure: show inline error "Invalid credentials" — no redirect
- No rate limiting in v0

### Session guard
Write a helper `require_auth()` that:
- Checks `app.storage.user.get('username')`
- If not set: redirects to `/login`
- Call this at the top of every page handler

### Routing shell
Register placeholder routes for all pages. Each placeholder just calls `require_auth()` and renders a `ui.label('Coming soon')`. Page agents will replace these:

```python
@ui.page('/')
def dashboard(): require_auth(); ui.label('Dashboard — coming soon')

@ui.page('/workorders')
def workorders(): require_auth(); ui.label('Work Orders — coming soon')

@ui.page('/workorders/{id}')
def wo_detail(id: int): require_auth(); ui.label(f'WO {id} — coming soon')

@ui.page('/crews')
def crews(): require_auth(); ui.label('Crews — coming soon')

@ui.page('/deadlines')
def deadlines(): require_auth(); ui.label('Deadlines — coming soon')

@ui.page('/payments')
def payments(): require_auth(); ui.label('Payments — coming soon')
```

### Navigation bar
Render a persistent top nav on every authenticated page with links:
- Dashboard (`/`)
- Work Orders (`/workorders`)
- Deadlines (`/deadlines`)
- Crews (`/crews`)
- Payments (`/payments`)
- Logout button: clears `app.storage.user`, redirects to `/login`

Implement nav as a reusable function `render_nav()` that page agents can call at the top of their page functions.

### Logout
- Clear `app.storage.user`
- Redirect to `/login`

### App config
```python
ui.run(
    host='0.0.0.0',
    port=8080,
    title='Thesora Ops',
    storage_secret=os.environ.get('SECRET_KEY', 'dev-secret-change-in-prod'),
    reload=False,
)
```

---

## `app/reset_admin.py` — Full Specification

Run as: `python -m app.reset_admin`

Behavior:
1. Prompt: `New password for admin: ` (use `getpass.getpass()` — input hidden)
2. Prompt again: `Confirm password: `
3. If they don't match: print error, exit 1
4. If match: hash with bcrypt, update `users` table where `username='admin'`
5. If no admin user exists: create one
6. Print: `[OK] Admin password updated.`

---

## Gate Condition

- [ ] `docker compose up` starts the app, port 8080 responds
- [ ] Navigating to `/` without login redirects to `/login`
- [ ] Login with wrong credentials shows "Invalid credentials", stays on `/login`
- [ ] Login with correct admin credentials (printed on first boot) reaches `/` with nav bar visible
- [ ] All 6 placeholder routes render without errors when authenticated
- [ ] Logout clears session and redirects to `/login`
- [ ] `python -m app.reset_admin` updates the password and new password works on login
- [ ] Running app a second time does NOT regenerate or reprint the admin password

---

## Contracts to Respect

- Use `bcrypt` for all password hashing — no `hashlib`, no `sha256`
- Session state lives only in `app.storage.user` — no cookies, no JWT, no custom session table
- `require_auth()` must be the first call in every page function — page agents will follow this pattern
- `render_nav()` must be exported so page agents can import and call it
- `SECRET_KEY` env var must be documented in a comment — operators must change it in production

---

## Notes for Page Agents (3A, 3B, 3C, 4)

After this merges, page agents must:
1. Import `require_auth` and `render_nav` from `app.main`
2. Replace the placeholder route function body with their full implementation
3. Call `require_auth()` as the first line
4. Call `render_nav()` immediately after

Page agents do NOT modify the `ui.run()` call or the first-boot logic.
