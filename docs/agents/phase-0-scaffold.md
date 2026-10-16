# Phase 0 — Scaffold & Contracts
**Assigned to:** Lead  
**Branch:** `phase/0-scaffold`  
**Create from:** `main`  
**Blocks:** Everything. No other agent starts until this merges.

---

## Objective

Establish the shared foundation — folder structure, Docker setup, all 10 SQLAlchemy models, Alembic wiring, and stub interfaces. Every other agent imports from these files. They must not be changed after this merges without a lead-approved patch.

---

## Files to Create

```
thesora-llc/
├── docker-compose.yml
├── requirements.txt
├── alembic.ini
├── alembic/
│   ├── env.py
│   └── versions/          (empty, Alembic populates this)
├── app/
│   ├── __init__.py        (empty)
│   ├── db.py
│   ├── models.py
│   └── storage.py
├── tests/
│   └── conftest.py
├── data/
│   └── photos/            (empty dir, gitignored)
├── .github/
│   └── pull_request_template.md
├── .gitignore
└── README.md              (placeholder only — full README written in Phase 5)
```

---

## Exact Specifications

### `docker-compose.yml`
- Two services only: `app` and `postgres`
- `postgres`: image `postgres:16`, db name `thesora`, user `thesora`, password via env, volume-mounted data dir
- `app`: build from `Dockerfile`, port `8080:8080`, depends on `postgres`, env vars for DB URL
- Named volume for postgres data
- Mount `./data/photos` into app container at `/app/data/photos`

### `Dockerfile`
- Base: `python:3.11-slim`
- Install requirements, copy app, run `python -m app.main`

### `requirements.txt`
```
nicegui>=1.4
sqlalchemy>=2.0
alembic>=1.13
psycopg2-binary>=2.9
pillow>=10.0
bcrypt>=4.0
python-dotenv>=1.0
pytest>=8.0
pytest-asyncio>=0.23
```

### `app/db.py`
- Load `.env` at module level using `python-dotenv`: `from dotenv import load_dotenv; load_dotenv()`
- SQLAlchemy `create_engine()` reading `DATABASE_URL` from env (`os.environ['DATABASE_URL']`)
- `SessionLocal` sessionmaker
- `get_session()` context manager yielding a session
- `Base = declarative_base()` exported for models and Alembic

### `app/models.py`
Implement all 10 tables exactly as specified. Key rules:
- All money columns: `Numeric(12, 2)`
- All timestamps: `TIMESTAMP(timezone=True)`, `server_default=func.now()`
- Array columns (`states_served`, `zips_served`, `trades`): `ARRAY(Text)`, `server_default=text("'{}'")` 
- `rate_card`: `JSONB`, `server_default=text("'{}'")` 
- `work_orders.status` CHECK constraint: `('created','assigned','in_progress','results_received','submitted','invoiced','paid','cancelled','chargeback')`
- `assignments.ended_reason` CHECK constraint: `('completed','reassigned')`
- `payments.status` CHECK constraint: `('pending','sent','cleared')`
- Partial unique index on `assignments`: `CREATE UNIQUE INDEX idx_one_active_assignment ON assignments(wo_id) WHERE ended_at IS NULL`
- All relationships defined with `relationship()` for ORM navigation

Tables:
1. `clients`
2. `crews`
3. `users`
4. `work_orders`
5. `wo_status_history`
6. `assignments`
7. `wo_photos`
8. `wo_notes`
9. `payments`
10. `payment_items`

### `app/storage.py`
Stub interface only — filesystem implementation:
```python
def save_photo(wo_id: int, file_bytes: bytes, kind: str, original_filename: str) -> str:
    # saves to data/photos/{wo_id}/{uuid}_{kind}.{ext}
    # returns relative file path
    ...

def list_photos(wo_id: int) -> list[dict]:
    # returns list of dicts with keys: file_path, kind, uploaded_at
    ...
```
Store files at `./data/photos/{wo_id}/`. Create directory if not exists. Filename: `{uuid}_{kind}.{ext}` preserving original extension. This interface must be swappable for S3/MinIO in v1 — no caller should reference the filesystem directly.

### `tests/conftest.py`
- Call `load_dotenv()` at the top
- Pytest fixture `db_session` — creates all tables against `TEST_DATABASE_URL` from `.env`
- Fixture scope: `function` (fresh state per test — drop and recreate all tables each time)
- Export `db_session` for all test files to import

### `alembic/env.py`
- Call `load_dotenv()` at the top before reading any env vars
- Import `Base` from `app.models`
- `target_metadata = Base.metadata`
- Read `DATABASE_URL` from `os.environ['DATABASE_URL']`

### `.gitignore`
```
__pycache__/
*.pyc
.env
data/photos/
backups/
*.egg-info/
.pytest_cache/
```

### `.github/pull_request_template.md`
```markdown
## Phase
<!-- e.g. Phase 1 — FSM + Services -->

## Gate condition
- [ ] All assigned tests pass (`pytest tests/`)
- [ ] App starts without errors (`docker compose up`)

## Scope check
- [ ] I only modified files listed in my instruction file
- [ ] I did not change models.py, db.py, or docker-compose.yml

## Notes for lead
<!-- anything the next agent needs to know -->
```

---

## Gate Condition (must be true before merging to main)

- [ ] `docker compose up` starts both containers without errors
- [ ] `docker compose exec app alembic upgrade head` creates all 10 tables
- [ ] `psql` inspection confirms partial unique index `idx_one_active_assignment` exists
- [ ] `from app.models import WorkOrder, Crew, Client` imports cleanly
- [ ] `from app.storage import save_photo, list_photos` imports cleanly
- [ ] No test files yet — conftest.py imports without error

---

## Do NOT Build

- No FSM logic
- No seed data
- No UI pages
- No authentication logic
- No business logic of any kind

---

## Notes for Dependent Agents

- `app/db.py` exports: `engine`, `SessionLocal`, `get_session`, `Base`
- `app/models.py` exports one class per table, named: `Client`, `Crew`, `User`, `WorkOrder`, `WoStatusHistory`, `Assignment`, `WoPhoto`, `WoNote`, `Payment`, `PaymentItem`
- `app/storage.py` exports: `save_photo`, `list_photos`
- All credentials come from `.env` via `python-dotenv` — never hardcoded
- `.env` is in `.gitignore` — never commit it
- `DATABASE_URL` format: `postgresql://thesora:<POSTGRES_PASSWORD>@postgres:5432/thesora`
- Test DB uses `TEST_DATABASE_URL` from `.env` pointing to `thesora_test` database
- Agent configures git using `GH_USER`, `GH_EMAIL`, `GH_TOKEN` from `.env`
