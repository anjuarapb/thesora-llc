# Phase 1 — FSM + Services + Tests
**Assigned to:** Agent A  
**Branch:** `phase/1-fsm-services`  
**Create from:** `main` after `phase/0-scaffold` merges  
**Blocks:** Phase 2A (seed needs services). Phase 3B (WO detail calls services).

---

## Objective

Implement the work-order state machine, the service layer that enforces all business invariants, and the full test suite. **This is the gate for the entire project.** No UI work starts until all tests in this phase pass.

---

## Files You Own

```
app/fsm.py          (create)
app/services.py     (create)
tests/test_fsm.py   (create)
tests/test_services.py  (create)
tests/test_payments.py  (create)
```

---

## Files You Must NOT Touch

```
app/models.py
app/db.py
app/storage.py
app/__init__.py
docker-compose.yml
requirements.txt
alembic/
```

---

## `app/fsm.py` — Full Specification

### Transition map
```python
TRANSITIONS: dict[str, list[str]] = {
    "created":          ["assigned", "cancelled"],
    "assigned":         ["in_progress", "cancelled"],
    "in_progress":      ["results_received", "cancelled"],
    "results_received": ["submitted"],
    "submitted":        ["invoiced", "chargeback"],
    "invoiced":         ["paid", "chargeback"],
    "paid":             [],
    "cancelled":        [],
    "chargeback":       [],
}

TERMINAL_STATUSES = {"paid", "cancelled", "chargeback"}
```

### `transition(wo, to_status, user, session)` function
- `wo`: a `WorkOrder` ORM instance (already loaded in the caller's session)
- `to_status`: string
- `user`: string (username, for history log)
- `session`: the active SQLAlchemy session

Behavior:
1. Raise `ValueError` if `to_status` not in `TRANSITIONS[wo.status]`
2. Set `wo.status = to_status`
3. Append a `WoStatusHistory` row: `wo_id=wo.id, from_status=old_status, to_status=to_status, changed_by=user, changed_at=now()`
4. Do NOT commit — caller commits. This function only mutates within the session.

**This function must never be bypassed. All status changes in the entire codebase go through it.**

---

## `app/services.py` — Full Specification

### `create_wo(session, client_id, property_address, task_type, **kwargs) -> WorkOrder`
- Creates `WorkOrder` with `status='created'`
- In the same flush: creates `WoStatusHistory(from_status=None, to_status='created', changed_by='system')`
- Returns the WO (not yet committed — caller commits)

### `assign_crew(session, wo, crew_id, quoted_rate, user) -> Assignment`
Invariants:
- WO status must be `created` or `assigned`; raise `ValueError` otherwise
- If `assigned` (reassignment): find active assignment (`ended_at IS NULL`), set `ended_at=now()`, `ended_reason='reassigned'`
- Create new `Assignment(wo_id, crew_id, quoted_rate)`
- Call `transition(wo, 'assigned', user, session)` — only if WO was `created`; if already `assigned` (reassignment), do NOT re-transition status
- All mutations in same session, caller commits

### `complete_results(session, wo, user)`
- Requires active assignment (`ended_at IS NULL`); raise `ValueError` if none
- End active assignment: `ended_at=now()`, `ended_reason='completed'`
- Call `transition(wo, 'results_received', user, session)`
- Caller commits

### `batch_pay(session, wo_ids: list[int], user: str) -> Payment`
Concurrency-safe implementation:
1. `SELECT ... FOR UPDATE` all WO rows for `wo_ids` (lock order: sorted by id to prevent deadlock)
2. Re-verify every locked WO has `status='invoiced'`; if any does not, raise `ValueError` — abort entire batch, no partial state
3. For each WO: call `transition(wo, 'paid', user, session)`
4. Find latest assignment per WO (highest `id`, regardless of `ended_at`) — this is the crew and rate source
5. Group by crew: create one `Payment(crew_id, batch_date=today, amount=sum(quoted_rates), status='sent')` per crew
6. Create `PaymentItem(payment_id, wo_id, amount=quoted_rate)` per WO
7. Return list of created Payment rows
8. Caller commits

---

## `tests/test_fsm.py` — Full Specification

Test every cell of the transition matrix:

```python
# Legal transitions — must NOT raise
@pytest.mark.parametrize("from_s,to_s", [
    ("created", "assigned"),
    ("created", "cancelled"),
    ("assigned", "in_progress"),
    ("assigned", "cancelled"),
    ("in_progress", "results_received"),
    ("in_progress", "cancelled"),
    ("results_received", "submitted"),
    ("submitted", "invoiced"),
    ("submitted", "chargeback"),
    ("invoiced", "paid"),
    ("invoiced", "chargeback"),
])
def test_legal_transition(from_s, to_s): ...

# Illegal transitions — must raise ValueError
@pytest.mark.parametrize("from_s,to_s", [
    ("created", "in_progress"),
    ("created", "paid"),
    ("assigned", "submitted"),
    ("results_received", "assigned"),
    ("paid", "invoiced"),
    ("cancelled", "created"),
    ("chargeback", "paid"),
])
def test_illegal_transition(from_s, to_s): ...
```

Also test:
- `WoStatusHistory` row is written on every successful transition
- `transition()` does not commit — session must be explicitly committed by caller

---

## `tests/test_services.py` — Full Specification

### Test (a) — `created → assigned` atomicity
```
GIVEN a WO in 'created' status
WHEN transition('assigned') is called WITHOUT creating an assignment first
THEN ValueError is raised and WO remains 'created'

WHEN assign_crew() is called (creates assignment + transitions atomically)
THEN WO status is 'assigned' AND assignment row exists in same transaction
```

### Test (b) — `in_progress → results_received` atomicity
```
GIVEN a WO in 'in_progress' with NO active assignment
WHEN complete_results() is called
THEN ValueError is raised

GIVEN a WO in 'in_progress' WITH an active assignment
WHEN complete_results() is called
THEN WO status is 'results_received'
AND assignment.ended_at is not null
AND assignment.ended_reason == 'completed'
AND both changes are in the same committed transaction
```

### End-to-end test
Walk a single WO through the full lifecycle:
```
create_wo() → assign_crew() → transition('in_progress') →
complete_results() → transition('submitted') →
transition('invoiced') → batch_pay()
```
Assert status at each step. Assert final payment row exists with `status='sent'`.

---

## `tests/test_payments.py` — Full Specification

### Test (c) — Concurrent batch payment
```
GIVEN a WO with status 'invoiced'
WHEN two separate DB sessions both call batch_pay() on the same WO_id
  (simulate with two sessions, first commits before second attempts)
THEN exactly one succeeds
AND the other raises (due to FOR UPDATE + status re-verification)
AND no partial payment rows exist from the failed attempt
AND the WO ends in 'paid' status exactly once
```

Implementation note: use two separate `SessionLocal()` instances. Open both, have session 1 lock + commit first, then session 2 attempts and must fail.

---

## Gate Condition (must be true before opening PR)

```bash
pytest tests/ -v
```

- [ ] Zero failures
- [ ] Zero errors
- [ ] All 3 spec tests (a, b, c) explicitly present and passing
- [ ] Full FSM matrix tests present and passing
- [ ] End-to-end lifecycle test present and passing

---

## Contracts to Respect

- `transition()` is the only place `wo.status` is ever set — no exceptions
- Money values: always `Decimal`, never `float`
- Timestamps: always UTC (`datetime.now(timezone.utc)`)
- `batch_pay()` locks rows in ascending `id` order to prevent deadlock
- Never call `session.commit()` inside `fsm.py` — callers own the transaction boundary

---

## Notes for Dependent Agents

After this merges, the following are available:
- `from app.fsm import transition, TRANSITIONS, TERMINAL_STATUSES`
- `from app.services import create_wo, assign_crew, complete_results, batch_pay`

Phase 2A (seed) will call `create_wo`, `assign_crew`, `complete_results`, and `transition` directly to build legal history chains.  
Phase 3B (WO detail) will call `assign_crew` and `transition` from UI button handlers.  
Phase 4 (payments) will call `batch_pay` from the UI.
