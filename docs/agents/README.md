# Thesora Ops Portal — Agent Phase Index

Master reference for the lead. One row per phase. Update status as phases progress.

---

## Phase Status Board

| Phase | Agent | Branch | Instruction File | Depends On | Status |
|---|---|---|---|---|---|
| 0 — Scaffold | Lead | `phase/0-scaffold` | `phase-0-scaffold.md` | — | ⬜ Not started |
| 1 — FSM + Services | Agent A | `phase/1-fsm-services` | `phase-1-fsm-services.md` | Phase 0 | ⬜ Not started |
| 2A — Seed | Agent B | `phase/2a-seed` | `phase-2a-seed.md` | Phase 1 | ⬜ Not started |
| 2B — Auth + Shell | Agent C | `phase/2b-auth-shell` | `phase-2b-auth-shell.md` | Phase 0 | ⬜ Not started |
| 3A — Dashboard + WO List | Agent D | `phase/3a-dashboard-workorders` | `phase-3a-dashboard-workorders.md` | 2A + 2B | ⬜ Not started |
| 3B — WO Detail | Agent E | `phase/3b-wo-detail` | `phase-3b-wo-detail.md` | 2A + 2B | ⬜ Not started |
| 3C — Crews + Deadlines | Agent F | `phase/3c-crews-deadlines` | `phase-3c-crews-deadlines.md` | 2A + 2B | ⬜ Not started |
| 4 — Payments | Agent G | `phase/4-payments` | `phase-4-payments.md` | 3B | ⬜ Not started |
| 5 — Integration + README | Lead | `phase/5-integration` | `phase-5-integration.md` | 3A+3B+3C+4 | ⬜ Not started |

Status key: ⬜ Not started · 🔄 In progress · 👀 In review · ✅ Merged

---

## Dependency Graph

```
Phase 0 (Lead)
    │
    ├──────────────────────────────────────────┐
    │                                          │
Phase 1 — Agent A                     Phase 2B — Agent C
    │                                          │
Phase 2A — Agent B                             │
    │                                          │
    └──────────────┬────────────────────────────┘
                   │
       ┌───────────┼───────────┐
       │           │           │
  Phase 3A      Phase 3B    Phase 3C
  Agent D       Agent E     Agent F
                   │
               Phase 4
               Agent G
                   │
               Phase 5
               Lead
```

---

## Lead Checklist — Before Unblocking Each Phase

### Before Phase 1 starts
- [ ] Phase 0 merged to `main`
- [ ] `alembic upgrade head` creates all 10 tables on clean DB
- [ ] `from app.models import WorkOrder` imports cleanly
- [ ] Notify Agent A: "Pull main, create branch phase/1-fsm-services, read phase-1-fsm-services.md"

### Before Phase 2A starts
- [ ] Phase 1 merged to `main`
- [ ] `pytest tests/` — all tests green
- [ ] Notify Agent B: "Pull main, create branch phase/2a-seed, read phase-2a-seed.md"

### Before Phase 2B starts (can start same time as Phase 1)
- [ ] Phase 0 merged to `main`
- [ ] Notify Agent C: "Pull main, create branch phase/2b-auth-shell, read phase-2b-auth-shell.md"

### Before Phases 3A, 3B, 3C start (all three can start in parallel)
- [ ] Phase 2A merged to `main`
- [ ] Phase 2B merged to `main`
- [ ] `python -m app.seed` runs clean, all 10 WOs present
- [ ] Login page works, session guard works
- [ ] Notify Agents D, E, F simultaneously

### Before Phase 4 starts
- [ ] Phase 3B merged to `main`
- [ ] WO detail page fully functional (walk a WO to invoiced manually)
- [ ] Notify Agent G

### Before Phase 5 starts
- [ ] Phases 3A, 3B, 3C, 4 all merged to `main`
- [ ] `pytest tests/` still green after all merges
- [ ] Lead does Phase 5 personally

---

## Shared Contracts (enforce on every PR review)

These rules apply to every agent. Reject any PR that violates them:

| Rule | Check |
|---|---|
| `wo.status` never set directly | `git diff` must not contain `wo.status =` outside `fsm.py` |
| Money never float | No `= 45.0` or similar — must be `Decimal('45.00')` |
| Timestamps UTC | No `datetime.now()` without `timezone.utc` |
| Timezone display | `ZoneInfo('Asia/Dhaka')` — no hardcoded `+6` |
| No new dependencies | `requirements.txt` unchanged unless lead approved |
| No models.py changes | `git diff --name-only` must not include `app/models.py` |
| No db.py changes | Same |
| Tests pass | `pytest tests/ -v` output attached or confirmed in PR |

---

## Files Ownership Map

| File | Owner | Locked after |
|---|---|---|
| `docker-compose.yml` | Lead | Phase 0 |
| `requirements.txt` | Lead | Phase 0 |
| `app/db.py` | Lead | Phase 0 |
| `app/models.py` | Lead | Phase 0 |
| `app/storage.py` | Lead | Phase 0 |
| `alembic/` | Lead | Phase 0 |
| `app/fsm.py` | Agent A | Phase 1 |
| `app/services.py` | Agent A | Phase 1 |
| `tests/` | Agent A (Phase 1), Lead (Phase 5) | — |
| `app/seed.py` | Agent B | Phase 2A |
| `app/main.py` | Agent C (shell) → page agents (route bodies) → Lead (final wiring) | Phase 5 |
| `app/reset_admin.py` | Agent C | Phase 2B |
| `app/pages/dashboard.py` | Agent D | Phase 3A |
| `app/pages/workorders.py` | Agent D | Phase 3A |
| `app/pages/wo_detail.py` | Agent E | Phase 3B |
| `app/pages/crews.py` | Agent F | Phase 3C |
| `app/pages/deadlines.py` | Agent F | Phase 3C |
| `app/pages/payments.py` | Agent G | Phase 4 |
| `README.md` | Lead | Phase 5 |

---

## Blocker Protocol

If an agent is blocked by something outside their file scope:
1. They open a GitHub Issue titled `[BLOCKER] Phase X — <description>`
2. They do NOT modify files outside their scope to work around it
3. Lead patches the blocking file and notifies the agent
4. Agent rebases their branch from `main` and continues

Common blockers to anticipate:
- Agent A needs a model field that doesn't exist → Lead patches `models.py`
- Agent E needs a service function that Agent A didn't implement → Lead patches `services.py`
- Agent D/E/F need a nav helper that Agent C didn't export → Lead patches `main.py`
