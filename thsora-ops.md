# Thesora Contracting LLC — Operating Model, Automation & Systems Manual

*Companion volume to the Consultant Briefing — expanded coverage of the operating model, automation, crew management, resources, vocabulary, and the backbone portal.*

---

## Table of Contents
1. [The Bangladesh/India Operating Model](#1-the-bangladeshiindia-operating-model)
2. [Automation Map & Operating Role](#2-automation-map--operating-role)
3. [Crew Database & Voice-Call Workflow](#3-crew-database--voice-call-workflow)
4. [Hidden Leverage Points & Resource Directory](#4-hidden-leverage-points--resource-directory)
5. [Industry Vocabulary (Full Set)](#5-industry-vocabulary-full-set)
6. [Operational Backbone Portal — Feature List](#6-operational-backbone-portal--feature-list)

---

## 1. The Bangladesh/India Operating Model

### 1.1 Why this model exists
Property preservation and maintenance processing is a mature, recognized outsourcing category in South Asia. Dedicated BPO firms in Dhaka (e.g., ASL BPO — operating since 2014 with 30+ US clients) and India run entire work-order back offices for US preservation companies. New PP-focused BPOs continue to be founded and publicly recruit in Dhaka. The conclusion: the modality is proven, the talent pool is hireable off the shelf, and there is nothing experimental about running this back office from Bangladesh.

### 1.2 What the BD/India team actually does (the standard service package)

| Task | Detail |
| :--- | :--- |
| **Work-order processing** | Receive WOs, update statuses, manage the full lifecycle in client portals |
| **Photo management** | Download from field apps, label (before/during/after, address verification, close-ups), upload to portals per client requirements |
| **Damage reporting** | Document property damage from photos into structured reports |
| **Bid creation** | Price repair bids using RepairBase/XactPRM with standard bid verbiage — directly drives approval rates |
| **Invoicing** | Generate invoices against client price sheets; track to payment |
| **Crew recruiting** | Source, screen, and onboard field contractors in target states |
| **Order assignment** | Dispatch WOs to crews; monitor completion |
| **Client support** | Answer client queries, chase approvals, manage escalations |

### 1.3 How the operating day runs
* **Night-shift alignment:** The team works an evening/night shift (≈6 PM–3 AM BST) to overlap with US business hours — live calls with crews and clients happen during US daytime.
* **Field apps, not portals:** Crews capture photos on Pruvan or PPW; the back-office team pulls the photos, labels them, and uploads results. Pruvan/PPW integrations are therefore first-class automation targets, not afterthoughts.
* **Role separation (the quality structure):**
  * **Processors** — work orders and bids
  * **QC** — photo compliance checking (where chargebacks are prevented)
  * **Vendor manager** — crew calls, recruiting, onboarding
  * **Collections/billing** — invoicing and AR follow-up
  
*Note: Small teams blur these roles; better operations separate them because QC and bidding quality determine the business's survival.*

### 1.4 Hiring & benchmarking
* Experienced PP processors are openly recruited via BDJobs, Fiverr, Upwork — hire people who already know RepairBase bid language and photo requirements rather than training from scratch.
* Benchmark the team against the BPO standard: work-order turnaround accuracy, bid approval rate, invoice error rate. Any gap vs. what dedicated BPOs deliver for their US clients is the "lack of grasp" to close.

### 1.5 Security & resilience practices
* Portal credentials live in a managed vault — never on individual employees' machines.
* Every process documented — no single employee is a walking single point of failure (the known failure mode of the freelancer model: VAs disappear with data and portal access).
* Bench capacity during volume spikes: contract Fiverr/Upwork PP processors rather than over-hiring.

### 1.6 Optional revenue line
Once the internal system is built, property-preservation processing for other US preservation companies is itself a proven Bangladesh export business — the internal tooling can become a service line.

---

## 2. Automation Map & Operating Role

### 2.1 Task-by-task automation potential

| Process | Potential | Design |
| :--- | :--- | :--- |
| **Bid/opportunity discovery** | High | Portal watchers + SAM.gov saved searches → Telegram/Slack alerts within minutes of posting; speed-to-bid is a real win-rate factor |
| **Bid pricing** | Medium | Pricing engine over the historical job DB (scope + zip + crew rate + actual cost + outcome) suggests a range; human approves. Margin lives here — never fully automatic |
| **Pipeline / work-win tracking** | High | Central database: every bid → status → won/lost reason. This dataset is what fixes operational blindness |
| **Client communication** | Medium | Templated auto-updates at each status change; humans handle exceptions and negotiations |
| **Crew searching / onboarding** | Med-High | Sourcing filters + auto-sent onboarding packet (W-9, COI request, rate sheet, portal instructions) |
| **Crew vetting calls** | Manual + assist | The screening call stays human (10–15 min, structured script). Everything around it is automatable: transcript → auto-filled scored profile → decision |
| **Work monitoring & photo compliance** | High | #1 loss-prevention automation: deadline tracker per WO, auto-reminders at T-48h/24h/4h, photo-checklist validation per scope, flag non-compliant submissions before the portal rejects them |
| **Purchasing (Amazon rectification)** | Medium | Request → auto price-check → one-click approval → order → cost auto-assigned to the WO. Prevents untracked spend |
| **Crew payment settlement** | High | Auto job-cost sheet per WO (payout − crew rate − materials), weekly payment batch, auto confirmation SMS. Removes spreadsheet pain and disputes |
| **Client collection** | High | AR aging tracker, auto statements + escalating dunning at day 30/45/60, humans call only hard cases |

### 2.2 The governing pattern
Automate capture, tracking, reminders, and money math. Keep humans for judgment, voice, and relationships.

### 2.3 The operating role (head of back office)
* Instrument everything — data before automation.
* Automate the repetitive 80% — bids matching, tracking, comms.
* Keep humans on the judgment 20% — crew voice screening, negotiation, escalations.

### 2.4 Build order (strict sequence)
1. **Weeks 1–4:** Central work-order database (Postgres or Airtable to start). No automation is possible before this exists.
2. **Weeks 4–8:** Deadline & photo-compliance tracker — attacks chargeback losses directly.
3. **Weeks 8–12:** Payment settlement engine — auto job-cost sheets, weekly batches.
4. **Then:** Portal scrapers/alerts, pricing engine, AR collections automation.

*Pre-build rule: spend 1–2 weeks observing the current team and documenting their exact workflow plus the last 12 months of loss events. Every failed automation project starts by building what the programmer assumed, not what the process needs.*

---

## 3. Crew Database & Voice-Call Workflow

### 3.1 Crew database schema (the second most valuable asset)
* **Coverage & identity:** Name, phone (WhatsApp), company name (verify on opencorporates + state SoS), City + zip codes served, radius willing to travel.
* **Trades & compliance:** Skills (plumbing / carpentry / lawn / trash-out / winterization / handyman / board-up), Licenses held; COI on file + expiry date; W-9 on file.
* **Rates (flat per service — not hourly):**

| Service | Typical crew rate |
| :--- | :--- |
| Grass cut | $40–60 |
| Lock change / rekey | $50–90 |
| Trash-out (by yardage) | $150–500 |
| Winterization | $75–125 |
| Minor plumbing repair | $75–150 |

* **Performance:** Jobs completed, on-time %, photo compliance %, chargeback history, dispute history, rating, response time.

### 3.2 Why the database matters
With ~30–50 crews scored across the licensed states, crew assignment becomes a lookup query instead of a phone marathon — and the worst 20% of crews (where the losses hide) get identified and dropped.

### 3.3 Voice-call workflow (hybrid design)
1. **Auto-pipeline:** Sourced lead → filter → onboarding packet emailed/texted automatically.
2. **Human (10–15 min):** Structured screening call — verify license/insurance, confirm rates and coverage area, availability, 2 references. One employee can handle 10–15 calls/day.
3. **Post-call automation:** Recording → transcription → auto-fill crew profile → score → approve/decline.
4. **Probation:** First 2 jobs under extra-strict photo and deadline monitoring; relax monitoring for proven crews.

*Why the call stays manual: a crew you have never spoken to is the biggest reliability risk in the business. No automation replaces it — but everything around it can be automated.*

---

## 4. Hidden Leverage Points & Resource Directory

### 4.1 Field-tech leverage (the integration layer is the moat)
* **Pruvan API + free tier:** Geo-coded, timestamped, tamper-proof "certified" photos upload in real time — QC can call the crew while still on site, eliminating return trips. Cheaper than PPW ($20/device/mo with volume discounts). Includes Dropbox/Drive auto-archive, PDF report generation, check-ins, and map of the service area.
* **PPW auto-pushback:** Auto-push results (photos/forms) to certain national clients for ~$0.25/work order — eliminates manual portal uploads.
* **XactPRM:** Localized preservation pricing data; pairs with Pruvan (proof) — having this pricing data is the bidding moat.

### 4.2 Industry-insider resources
* **PropertyVendors.com** — operator blogs (including honest chargeback post-mortems) + crew/vendor-wanted ads per state; doubles as a recruiting board.
* **NAMFS (namfs.org)** — the National Association of Mortgage Field Services: directories, standards, industry contacts. Unknown outside the industry.
* **NFR industry-links page (nfronline.com)** — curated list of nationals and servicers to register with.
* **National portals to register with:** DMG Pro, MCS, Safeguard Properties, ServiceLink (AssetShield), Five Brothers, Altisource, M&M, Cyprexx, United Field Services, PKMG.

### 4.3 Government opportunity tools
* **SAM.gov** — federal contracts (free registration; UEI + CAGE assigned).
* **GovBidPortals.com** — directory of all 50 state procurement portals; state/local thresholds are $5k–10k (vs. federal $25k).
* **USAspending.gov** — who holds facility-maintenance contracts near you and when they recompete; subcontracting channel.
* **Agency procurement forecasts** — published 6–18 months before RFPs; free first-mover intelligence.
* **Cooperative purchasing (OMNIA Partners, Sourcewell, NASPO ValuePoint)** — one contract reaches thousands of state/local/school buyers.
* **Micro-purchases (under ~$10k, via Government Purchase Cards)** — never hit SAM.gov — capture via agency vendor days and OSDBU (small business) office relationships.

### 4.4 Demand forecasting (free, underused)
* **data.gov** — HUD & Census datasets: vacancy rates, foreclosure/REO counts by county/zip = prediction of preservation volume in target states. Almost no contractor does this.
* **OpenFEMA (fema.gov):** Disaster declarations → post-disaster securing/repair demand spikes.

### 4.5 Company vetting (crew due diligence)
* **opencorporates.com** — verify a crew's LLC exists, owners, good standing before onboarding.
* **State Secretary-of-State business search** — official entity status and license verification.

### 4.6 Hiring (Bangladesh back office)
* **BDJobs.com, Fiverr, Upwork** — experienced PP processors and team leads; use as bench capacity during volume spikes.

### 4.7 The genuinely hidden dataset
Each national's vendor price sheet (allowables) — obtained only by registering as a vendor. Comparing the same task's allowable across multiple clients serving the same zip is unpublished arbitrage intelligence. Over time, your own job-history database becomes the proprietary pricing moat. Protect it.

---

## 5. Industry Vocabulary (Full Set)

### 5.1 Work-order lifecycle

| Term | Definition |
| :--- | :--- |
| **Work Order (WO)** | The atomic unit of business: a client's instruction to perform a task at a property. Flows: received → assigned → completed → results submitted → invoiced → paid |
| **Bid / Estimate** | A priced proposal sent to the client BEFORE performing repair work |
| **Approval** | Client accepts the bid. Rule #1: never do unapproved work — it will not be paid |
| **Allowable** | The pre-approved maximum a client will pay for a standard task without a bid. Knowing each client's allowables table is a core skill |
| **Chargeback / Backcharge** | The client deducts or withholds payment after the fact for failed QC, missing photos, missed deadlines, or rework. The #1 killer in this industry |
| **Cut-off / Deadline** | Date/time results must be submitted; missing it = automatic chargeback or reassignment |
| **Conveyance Condition** | The HUD/FHA standard a foreclosed property must meet before transfer; drives many recurring orders |
| **Initial Secure** | First visit to a newly vacant property: rekey, board-up, debris assessment, photos |
| **Recut** | A recurring grass cut (typically every 2–4 weeks in season) |
| **Winterization** | Draining/protecting plumbing for cold season; seasonal high volume, strict photo rules |
| **Trash-out / Debris Removal** | Removing interior/exterior debris, priced by cubic yardage |
| **Board-up** | Securing broken windows/doors, priced per opening |
| **Rekey / Lock change** | Installing lender-specific lockbox/codes; standard on initial secures |
| **MRR (Monthly Recurring Route)** | The predictable income core: recurring grass cuts, secure checks, inspections per property per month |
| **Occupancy Inspection** | Verifying occupied/vacant status with photo proof — low-skill, high-volume |

### 5.2 Pricing & bidding

| Term | Definition |
| :--- | :--- |
| **RepairBase** | Industry-standard cost estimator for preservation line items; clients expect bids in its structure and verbiage (subscription) |
| **XactPRM** | Verisk/Xactware's preservation pricing database; pairs with Pruvan |
| **The Bluebook** | Construction/repair unit-cost database for larger repair bids |
| **HUD/FHA price guidelines** | Public ceilings for conveyance-related reimbursement |
| **Vendor Price Sheet / Rate Card** | Each national's payout table by state/zip — semi-hidden, obtained by registering |
| **Allowable arbitrage** | Comparing the same task's allowable across clients serving the same zip |

### 5.3 Crew / vendor management

| Term | Definition |
| :--- | :--- |
| **1099 subcontractor** | Standard crew relationship: independent contractor, paid per job, gives W-9; you issue 1099-NEC at year end |
| **W-9** | IRS form collecting crew's tax ID — collect before first assignment, always |
| **COI (Certificate of Insurance)** | Proof of crew's General Liability; track expiry dates — expired COI = uncovered liability |
| **Route density** | Jobs per crew per day within a tight radius; the biggest crew-profitability variable |
| **Coverage map / Service area** | Zips/cities a crew serves — the core crew-database field |
| **Probation period** | First 2–3 jobs for a new crew under extra-strict monitoring |
| **E&O exposure** | Your bidding/QC errors are your liability, not the crew's |

### 5.4 Field technology

| Term | Definition |
| :--- | :--- |
| **Pruvan** | Field photo/mobile app: geo-coded, timestamped "certified" photos; API + free tier; the proof layer against chargebacks |
| **PPW (Property Preservation Wizard)** | SaaS work-order management + mobile app; auto-pushback to certain clients (~$0.25 / WO) |
| **EXIF / GPS stamping** | Metadata in field photos proving when/where taken; required by most clients |
| **CSR (Certified Service Record)** | Pruvan's tamper-proof record: check-ins, photos, forms, service-area map |

---

## 6. Operational Backbone Portal — Feature List

**Stack:** NiceGUI (UI) + PostgreSQL (single source of truth) + TaskIQ (background jobs) + Playwright (scraping) + MinIO (self-hosted S3-compatible document/photo storage). Single VPS, Docker Compose.

### 6.1 Operations Tracker (the real-time dashboard that saves the team from chaos)
* **Live work-order table:** Status, client, property, task type, deadline countdown, assigned crew, quoted rate, expected payout.
* **Deadline board:** Color-coded T-48h / T-24h / T-4h / overdue; auto-escalation visible to everyone.
* **Red-flag column:** Chargeback-risk, missing photos, unapproved-work risk, stale bids awaiting client response.
* **Daily metrics strip:** Orders in queue, submitted today, overdue, crew utilization.
* **Filters:** By state, client, task type, crew, status.
* **Search:** By address, WO number, property.
* *Everything renders from Postgres views — the tracker is a window on the database, not a separate system.*

### 6.2 Work-order ingest (scraper layer)
* Scheduled Playwright scrapers per portal (randomized 30–60 min intervals).
* Three ingest layers per portal: official integration (Pruvan/PPW auto-import) → Playwright scrape → IMAP email-parsing fallback.
* `scrape_runs` observability: every run logged; page-structure hash detects portal redesigns before silent data loss.
* Session vault: portal credentials encrypted, one persistent browser context per portal.
* Anti-detection: human-like delays, realistic viewport, retry with jitter.
* New WO → row in DB with status `new` + Telegram alert → human reviews and bids. *Scrapers detect; humans decide; never auto-accept work.*

### 6.3 Crew database module
* Full crew CRUD with all schema fields (coverage, trades, rate cards, COI/license expiries, performance scores).
* Crew lookup by zip + trade + availability — assignment becomes a query.
* COI / W-9 / license expiry dashboard with 14/7/1-day alerts.
* Onboarding packet generation (auto-filled W-9 request, COI request, rate sheet, portal instructions).

### 6.4 Dispatch & communications log
* Assign crew → auto-send SMS/WhatsApp dispatch with WO details.
* Every message and call logged against the WO (channel, direction, body, timestamp).
* Message templates: dispatch offer, deadline reminders (T-48/24/4h), photo-fix request, payment confirmation.
* Inbound SMS webhook → stored → alert to assigned team member.
* Post-call summaries typed into the WO thread — the human voice layer stays, everything else is logged.

### 6.5 Photo QC gate
* WO cannot move to "submitted" until the required photo checklist for its task type validates (before/after, address verification, work-item close-ups).
* Photos pulled via Pruvan API or crew upload → stored in MinIO with EXIF verification.
* Non-compliant submission flagged back to crew with the specific missing shots — *before* the portal rejects it.
* QC history per crew feeds the crew performance score.

### 6.6 Internal chat & work threads
* Per-WO comment threads + general channels (replaces scattered WhatsApp groups).
* Full history: six months later, show why a decision was made.
* **Culture rule:** If it happened in a personal DM, it didn't happen.

### 6.7 Invoicing & payments
* Auto job-cost sheet per completed WO (payout − crew rate − materials).
* Weekly crew payment batch → review screen → execute → auto payment-confirmation SMS.
* AR aging per client; auto statements + escalating dunning at day 30/45/60; hard cases flagged to humans.

### 6.8 Document vault (MinIO)
* **Buckets:** 
  * `photos/{wo_id}/` (never delete — chargeback evidence)
  * `documents/crews/`
  * `documents/clients/` (versioned)
  * `exports/`
* Presigned URLs served through NiceGUI; MinIO never exposed to the internet.
* Expiry tracking on all compliance documents (COIs, licenses, insurance certificates).
* Nightly `mc mirror` to offsite storage; monthly restore drills.

### 6.9 Alerts engine (TaskIQ)

| Job | Schedule | Purpose |
| :--- | :--- | :--- |
| `scrape_portals` | every 30–60 min, randomized | Pull new WOs / bid approvals |
| `check_deadlines` | every 30 min | T-48h / T-24h / T-4h reminders; escalate breaches |
| `qc_photos` | on event | Validate photo checklist per task type |
| `coi_expiry_scan` | daily | Alert on expiring crew COIs/licenses (14/7/1 days) |
| `payment_batch` | weekly | Generate crew payments → review → execute |
| `ar_dunning` | daily | Escalating statements at day 30/45/60 |
| `nightly_snapshot` | daily 11 PM BST | Metrics snapshot for dashboard trends |

**Delivery channels:** Telegram bot (team), SMS/WhatsApp (crew).

### 6.10 Build principles
1. **Data before automation** — manual entry first; automating an unrun process automates your own misunderstanding.
2. **Alerts over dashboards early** — a Telegram ping beats a beautiful chart nobody watches in the first months.
3. **Everything logged, nothing in DMs.**
4. **Scrapers will break** — budget for it. The three-layer fallback is not optional.
5. **Photos are legal evidence** — retention is a compliance obligation, not storage hygiene.

---
*End of manual. Companion documents: the Consultant Briefing (business overview & compliance) and the Playbook canvas (Part 1 + Appendix A/B full detail).*