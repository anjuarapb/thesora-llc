# Phase 3B — Work Order Detail Page
**Assigned to:** Agent E  
**Branch:** `phase/3b-wo-detail`  
**Create from:** `main` after `phase/2a-seed` AND `phase/2b-auth-shell` both merge  
**Can run parallel to:** Phase 3A, Phase 3C  
**Blocks:** Phase 4 (payments page reuses job-cost logic)

---

## Objective

Build the Work Order detail page — the most complex page in the app. It is the primary workspace for the back-office team: they read WO details, trigger status transitions, assign crews, upload photos, add notes, and check job cost here.

---

## Files You Own

```
app/pages/wo_detail.py      (create)
```

---

## Files You Must NOT Touch

```
app/main.py         ← replace only the body of the wo_detail route
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
from app.pages.wo_detail import render_wo_detail

@ui.page('/workorders/{id}')
def wo_detail(id: int): require_auth(); render_wo_detail(id)
```

---

## `app/pages/wo_detail.py` — Full Specification

### Layout
Two-column layout:
- Left (wider): WO fields, status controls, job-cost strip, status history timeline
- Right (narrower): Notes thread, photo upload + thumbnail grid

### WO Fields Section
Display all fields. Make editable fields inline-editable (click to edit) or use an "Edit" button that opens a dialog. Editable fields:
- `external_wo_no`, `property_address`, `state`, `zip`, `task_type`, `scope_text`
- `allowable_amount`, `bid_amount`, `bid_status`
- `deadline_at`
- `chargeback_flag` (checkbox)

Non-editable display fields: `id`, `client name`, `status`, `created_at`

On save: update the WO row, refresh the page section. No status change on field edit.

### Status Transition Controls
- Query `TRANSITIONS[wo.status]` from `app.fsm` to get legal next states
- Render one button per legal next state
- Button labels:
  - `assigned` → "Assign Crew" (opens crew picker dialog)
  - `in_progress` → "Mark In Progress"
  - `results_received` → "Mark Results Received" (calls `services.complete_results()`)
  - `submitted` → "Mark Submitted"
  - `invoiced` → "Mark Invoiced"
  - `paid` → disabled here — paid only via Payments page batch
  - `cancelled` → "Cancel WO" (confirm dialog before executing)
  - `chargeback` → "Mark Chargeback" (confirm dialog)

#### Assign Crew dialog
Opens when "Assign Crew" is clicked:
- Crew dropdown: all active crews
- Quoted Rate field (numeric, required)
- On confirm: call `services.assign_crew(session, wo, crew_id, quoted_rate, user)`
- On reassignment (WO already `assigned`): same dialog, same call — services handles the reassignment logic

#### Transition execution
For all non-assign transitions:
- Call `fsm.transition(wo, to_status, user, session)` then `session.commit()`
- Exception: `results_received` calls `services.complete_results()` instead
- On `ValueError` from FSM: show error notification, do not navigate away
- On success: refresh the page

### Job-Cost Strip
Display below the status controls. Always visible.

Formula: `COALESCE(bid_amount, allowable_amount) − latest_assignment.quoted_rate`

"Latest assignment" = highest `id` in `assignments` for this WO, regardless of `ended_at`.

Display rules:
- If both `bid_amount` and `allowable_amount` are NULL: show `"Estimate unavailable"` — never show $0
- If `latest_assignment` is None or `quoted_rate` is NULL: show `"Estimate unavailable"`
- Otherwise: show `$X.XX estimated — materials not tracked; chargebacks excluded`
- Positive margin: green text. Negative margin: red text.

### Status History Timeline
Ordered list of `wo_status_history` rows for this WO, newest first.

Each row: `NULL → created (by system) — Jan 15, 2025 14:32 BDT`

All timestamps in Asia/Dhaka timezone.

### Notes Thread
Right column, top section.

Display all `wo_notes` for this WO, oldest first (chronological reading order).

Each note:
```
[author] — Jan 15, 2025 14:45 BDT
Note body text here
```

Add-note form at the bottom:
- Textarea (required)
- "Add Note" button
- Author: use `app.storage.user['username']`
- On submit: insert `WoNote`, refresh notes section

### Photo Upload
Right column, below notes.

Upload control:
- Accept: `.jpg`, `.jpeg`, `.png` only
- Max size: 10 MB — reject with error message if exceeded
- Kind dropdown (required): `before`, `after`, `address`, `closeup`, `other`
- "Upload" button

On upload:
1. Validate file type and size
2. Call `storage.save_photo(wo_id, file_bytes, kind, original_filename)`
3. Insert `WoPhoto(wo_id, kind, file_path, uploaded_at=now())`
4. Refresh thumbnail grid

Thumbnail grid:
- Show all photos for this WO from `wo_photos` table
- Each thumbnail: image preview + kind label + upload timestamp
- Clicking a thumbnail opens the full image in a new tab (use presigned-style path or direct file serve)
- No delete button — v0 has no photo deletion

File serving: NiceGUI's `app.add_static_files('/photos', './data/photos')` — serve the photos directory statically. Thumbnails link to `/photos/{wo_id}/{filename}`.

---

## Gate Condition

- [ ] Navigate to `/workorders/2` (assigned WO) — all fields display correctly
- [ ] Status transition buttons show only legal next states for each WO status
- [ ] "Assign Crew" dialog opens, selecting crew + rate and confirming transitions WO to `assigned`
- [ ] Reassignment works: on an already-assigned WO, "Assign Crew" replaces the assignment
- [ ] "Mark Results Received" on an in-progress WO succeeds and ends the assignment
- [ ] Attempting an illegal transition (e.g. calling transition directly) shows error, WO unchanged
- [ ] Job-cost strip shows correct value for WO4 (`$320 - $180 = $140`)
- [ ] Job-cost strip shows "Estimate unavailable" for WO1 (no bid/allowable)
- [ ] Notes: add a note, it appears in the thread
- [ ] Photo upload: upload a jpg, thumbnail appears in grid
- [ ] Photo upload: attempt >10MB file, error shown, no upload
- [ ] Photo upload: attempt .pdf file, rejected
- [ ] Status history timeline shows full chain for WO7 (paid)
- [ ] All timestamps in Asia/Dhaka timezone

---

## Contracts to Respect

- All status changes go through `fsm.transition()` or `services.*` — never set `wo.status` directly
- `storage.save_photo()` is the only way to write photo files — no direct filesystem calls
- `require_auth()` first, `render_nav()` second
- Money displayed as `$X.XX`, never as float with more than 2 decimal places
- Timezone: `zoneinfo.ZoneInfo('Asia/Dhaka')`
- `paid` transition is NOT available from this page — it only happens via the Payments batch
