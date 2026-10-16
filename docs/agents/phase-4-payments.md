# Phase 4 — Payments Page
**Assigned to:** Agent G  
**Branch:** `phase/4-payments`  
**Create from:** `main` after `phase/3b-wo-detail` merges  
**Blocks:** Phase 5 (integration)

---

## Objective

Build the Payments page: weekly batch preview grouped by crew, exceptions list, concurrency-safe "Mark batch paid" action, and payment history. The `batch_pay()` service function already exists and is tested — this phase is purely the UI layer over it.

---

## Files You Own

```
app/pages/payments.py       (create)
```

---

## Files You Must NOT Touch

```
app/main.py         ← replace only the body of the payments route
app/models.py
app/db.py
app/fsm.py
app/services.py     ← batch_pay() is already implemented and tested — do not modify
app/storage.py
tests/
docker-compose.yml
requirements.txt
```

### How to plug into main.py
```python
from app.pages.payments import render_payments

@ui.page('/payments')
def payments(): require_auth(); render_payments()
```

---

## `app/pages/payments.py` — Full Specification

### Page layout
Three sections stacked vertically:
1. Weekly Batch Preview (batchable WOs)
2. Exceptions List (invoiced WOs that cannot be batched)
3. Payment History

---

### Section 1 — Weekly Batch Preview

**Query:** All `work_orders` with `status = 'invoiced'` that are NOT in the exceptions list (see Section 2 for exception criteria).

**Grouping:** Group by the crew from the WO's latest assignment (highest `assignment.id` for that WO, regardless of `ended_at`).

**Display:** One expandable group per crew:

```
▼ James Okafor (Okafor Property Care)                    Total: $150.00
  WO #6  | 560 Birch Ct, Charlotte NC  | grass_cut  | $45.00
  WO #X  | ...                          | ...        | $XX.00
  [Mark Batch Paid for this crew]
```

Per-crew columns: WO#, Address, Task Type, Quoted Rate.
Per-crew footer: sum of quoted rates.

**"Mark Batch Paid" button** (one per crew group):
- Collects all `wo_id`s in that crew's group
- Calls `services.batch_pay(session, wo_ids, user)`
- On success: show notification `"Payment batch sent for [crew name] — $X.XX"`, refresh page
- On `ValueError` (concurrent modification caught): show error `"Batch failed: one or more WOs were modified. Refresh and retry."` — do not crash
- Button is disabled if the crew group has zero batchable WOs

**"Mark All Paid" button** (top of section, above all groups):
- Collects ALL batchable WO ids across all crews
- Calls `services.batch_pay()` once per crew group (separate payment per crew)
- Same error handling as per-crew button

---

### Section 2 — Exceptions List

**Title:** "Invoiced WOs that cannot be batched" — always visible, even if empty (show "None" if empty).

**Query:** All `work_orders` with `status = 'invoiced'` that meet ANY of these conditions:

| Exception | Condition | Display label |
|---|---|---|
| (a) No assignment | No row in `assignments` for this WO | "No assignment on record" |
| (b) NULL quoted rate | Latest assignment exists but `quoted_rate IS NULL` | "Quoted rate missing" |
| (c) NULL payout | Both `bid_amount` and `allowable_amount` are NULL | "No bid or allowable amount" |

A single WO can match multiple exception conditions — list all that apply.

**Columns:** WO#, Address, Task Type, Client, Reason(s).

**No action buttons** on exceptions — staff must fix the underlying data first (via WO detail page), then the WO will move to the batchable section on refresh.

---

### Section 3 — Payment History

**Query:** All rows from `payments` table, newest first, with their `payment_items` and associated WOs.

**Display:** Expandable list:

```
▼ Batch — Jan 15, 2025  |  James Okafor  |  $70.00  |  Status: sent
  WO #7  | 77 Walnut St, Detroit MI  | rekey  | $70.00
```

Columns per payment: Batch Date, Crew Name, Total Amount, Status badge.
Status badge colors: `pending`=grey, `sent`=blue, `cleared`=green.

Per-payment items: WO#, Address, Task Type, Amount.

---

## Gate Condition

- [ ] Page loads with seed data — WO6 (invoiced) appears in batch preview under Crew 1 (Webb)
- [ ] Exceptions section shows "None" (all seed invoiced WOs should be batchable — verify against seed)
- [ ] "Mark Batch Paid" for Webb's group: WO6 transitions to `paid`, payment row created with `status='sent'`
- [ ] After marking paid, WO6 disappears from batch preview and appears in payment history
- [ ] Payment history shows the seed payment for WO7 (status: sent)
- [ ] Concurrent click simulation: clicking "Mark All Paid" twice rapidly does not create duplicate payment rows
- [ ] Exceptions list correctly identifies a WO with no assignment (manually insert one to test)
- [ ] Exceptions list correctly identifies a WO with NULL quoted_rate (manually insert one to test)
- [ ] All timestamps in Asia/Dhaka timezone

---

## Contracts to Respect

- `require_auth()` first, `render_nav()` second
- Never call `fsm.transition()` directly from this page — always go through `services.batch_pay()`
- `batch_pay()` is already concurrency-safe via `SELECT FOR UPDATE` — do not add extra locking in the UI layer
- Money displayed as `$X.XX`
- Timezone: `zoneinfo.ZoneInfo('Asia/Dhaka')`
- "Latest assignment" = highest `id` in `assignments` for that WO — this is the same rule used in `services.batch_pay()` and the job-cost strip on WO detail
- Exceptions list must never be hidden or collapsed by default — staff must see it every time they open the page
