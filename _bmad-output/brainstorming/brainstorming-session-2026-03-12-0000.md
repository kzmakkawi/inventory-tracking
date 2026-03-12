---
stepsCompleted: [1, 2, 3]
inputDocuments:
  - /Users/khalilmakkawi/D-NAMO.Master/supabase/migrations/
  - /Users/khalilmakkawi/D-NAMO.Master/_bmad-output/planning-artifacts/architecture.md
  - /Users/khalilmakkawi/D-NAMO.Master/_bmad-output/planning-artifacts/prd/
  - /Users/khalilmakkawi/D-NAMO.Accounting/_bmad-output/planning-artifacts/architecture/
  - /Users/khalilmakkawi/D-NAMO.Accounting/_bmad-output/planning-artifacts/epics.md
  - /Users/khalilmakkawi/D-NAMO.Accounting/_bmad-output/planning-artifacts/ux/ux-design-specification-documents-queue.md
  - /Users/khalilmakkawi/D-NAMO.Accounting/_bmad-output/implementation-artifacts/tech-spec-permission-matrix-multi-module.md
  - /Users/khalilmakkawi/Projects/inventory-tracking/_bmad-output/interviews/consolidated-findings.md
session_topic: 'D-NAMO platform nucleus architecture — how all child apps share data, documents, users, OCR, and workflows without duplicating work'
session_goals: |
  1. Design the Data Taxi / centralized document processing pipeline as a shared platform service
  2. Define cross-module data sharing patterns (nucleus vs app-specific boundaries)
  3. Design the triage queue across modules (role-based, entity-scoped)
  4. Solve the permissions matrix across inventory, accounting, pre-sales, procore
  5. Define inventory <-> accounting data flow (delivery receipt → invoice readiness)
  6. Decide repo architecture (mono vs multi, shared packages)
selected_approach: 'ai-recommended'
techniques_used: ['Assumption Reversal', 'Morphological Analysis', 'Solution Matrix']
ideas_generated: []
context_file: ''
---

# Brainstorming Session Results

**Facilitator:** Khalilmakkawi
**Date:** 2026-03-12

## Session Overview

**Topic:** D-NAMO platform nucleus architecture — how all child apps (accounting, inventory, pre-sales, procore, HR, and future) share data, documents, users, OCR processing, and workflows without duplicating work.

**Goals:**
1. Design the "Data Taxi" / centralized document processing pipeline as a shared platform service
2. Define cross-module data sharing patterns (what belongs in the nucleus vs what stays app-specific)
3. Design the triage queue across modules — role-based visibility, entity-scoped, future WhatsApp/Inbox entry
4. Solve the permissions matrix cleanly across all modules (inventory, accounting, pre-sales, procore)
5. Define the inventory ↔ accounting data flow (delivery receipt → invoice readiness → QBO signal)
6. Decide repo architecture (mono vs multi-repo, shared packages strategy)

### Context Loaded

**D-NAMO.Master (Platform Nucleus)**
- Single Supabase project, shared Postgres DB with RLS for entity isolation
- Core tables: `users`, `entities`, `permissions`, `audit_log`, `file_registry`, `jobs`, `notifications`, `tool_registry`
- RBAC: two-level (tool access + within-tool scope), enforced at prompt, backend, and DB levels
- Plugin architecture: new module = new web app + tool registration
- Two-vendor discipline: Railway + Supabase only (no Redis, no Elasticsearch)

**D-NAMO.Accounting (Most Mature Child App)**
- Full document processing pipeline ALREADY BUILT: email/SharePoint intake → OCR (Mistral Pixtral) → confidence scoring → duplicate detection → review queue → approve/reject → CSV/QBO/Procore export
- Key tables: `acct_documents`, `acct_extracted_data`, `acct_duplicate_flags`, `acct_review_actions`, `acct_export_records`, `acct_procore_commitments`
- Multi-entity, multi-module RBAC (accounting + pettycash roles per user)
- Built Python/FastAPI backend, agent-compatible REST API design
- V1.5 adding contact field extraction (6 new fields); V2.0 = full QBO; V2.5 = Procore push
- Entity-first triage queue UX (Linear-inspired grouping with confidence heat strips)

**D-MEP Inventory Tracker (Greenfield)**
- Tracks material movements: warehouse → site → subcontractor
- Two-tier material model: project items (PO-tracked) vs consumables (bulk, replenish)
- Procore as data backbone (POs, commitments, requisitions)
- Key pain: no real-time visibility, no delivery dashboard, manual consumable takeoffs from BOQs
- No OCR today — all manual entry; but BOQ parsing from PDFs is the differentiator opportunity

**Key Constraints:**
- Two-vendor discipline (Railway + Supabase — no Redis, no external queues)
- Entity isolation is non-negotiable (RLS at DB layer)
- Platform enhances, never gates child apps (resilience-first)
- 3-person dev team
- WhatsApp-first for field workers; web SPA second

### Session Setup

**Approach:** AI-Recommended Techniques
**Analysis Context:** Platform nucleus architecture + inventory app design (mirroring accounting app)

**Recommended Technique Sequence:**
- **Phase 1 — Assumption Reversal** (deep): Challenge the 4 big decisions made today before locking them into PRD. Expected outcome: reinforced conviction or discovered risks.
- **Phase 2 — Morphological Analysis** (deep): Systematically map every open architectural dimension (OCR location, queue UI, QBO sync, repo structure, schema design) → decision matrix resolving the 6 open questions.
- **Phase 3 — Solution Matrix** (structured): Feature-vs-role grid for inventory-specific needs → draft feature scope + data model skeleton for PRD.

**Status: Phase 1 (Assumption Reversal) and Phase 2 (Morphological Analysis) complete. Phase 3 (Solution Matrix) in progress.**

---

## Key Architectural Direction — Established in Session

### Decision: Elevate Accounting OCR to Platform-Level Pipeline

**Direction:** The accounting app's OCR pipeline should be lifted to the D-NAMO platform nucleus. Each module (accounting, inventory, pre-sales, procore) becomes a consumer of a shared document processing service — not an owner of its own.

**Platform owns:**
- Document intake (email polling, SharePoint monitoring, WhatsApp photo upload)
- OCR extraction (Mistral Pixtral) → confidence scoring → duplicate detection
- `platform_documents` + `platform_extracted_data` tables (renamed from `acct_*`)
- Centralized triage queue (entity-first + module-scoped views)
- Audit trail for all review actions

**Each module registers:**
- Its document types (e.g., accounting: invoice/receipt; inventory: delivery_receipt/BOQ; pre-sales: RFQ)
- Its approval logic and downstream export destinations
- Its role-based view of the queue (accountant sees invoices; storekeeper sees delivery receipts)

### Steelman: Why This Is the Right Call

1. **Compounding rework avoided** — OCR pipe-plumbing (SharePoint auth, email polling, Mistral API, confidence scoring, duplicate detection, retry logic, audit trail) solved once, not three times across three codebases.

2. **Accounting OCR is the hardest part and it's already working** — Mistral Pixtral integration, per-field confidence scoring with penalty system, duplicate detection, 16+ field extraction schema. Lift it, don't replicate it. Rename `acct_documents` → `platform_documents`, add `doc_module` as a first-class field, let modules register document types.

3. **The triage queue UX already generalizes** — Entity-first grouping with confidence heat strips works for any document type. Adding a module axis (accounting/inventory/pre-sales) doesn't require a redesign — it requires parameterization. A Group Controller opening the queue sees all document types across all entities, filtered by authorization.

4. **The only path to WhatsApp triage** — A field engineer photos a delivery receipt on WhatsApp → platform pipeline classifies it as `delivery_receipt` → routes to inventory module's queue → storekeeper reviews. This only works with one centralized pipeline. Per-app OCR requires the sender to know which module to route to before sending — defeating the purpose.

5. **Timing is the argument** — Accounting is mid-V1.5. Inventory is greenfield. Pre-sales and procore haven't started. This is the last cheap moment to make this call. In six months, three apps with their own OCR services make migration enormously costly.

6. **Consistent with platform philosophy** — Plugin architecture (new module = new tool registration), two-vendor discipline, resilience-first. Per-app OCR contradicts all three principles.

### Open Questions (To Brainstorm Further)
- What does the migration path look like for accounting? (rename tables vs. new platform tables + accounting reads from platform)
- Does `platform_extracted_data` use a generic field schema or per-doc-type schemas?
- How does the module-scoped queue view work in the UI? One app with a module switcher, or each app has its own queue view that reads from the platform?
- Where does the platform OCR service live? In the D-NAMO.Master repo or a new dedicated service?
- How do module-specific export destinations (QBO for accounting, Procore for inventory) plug in without coupling to the platform?

---

## Accountant Interview — Chadi Abboud (2026-03-12)

### The Actual Procore + QuickBooks Inventory Workflow (As It Exists Today)

**The core architecture Chadi uses:**
> "There's no inventory module on Procore. We're using the Commitment as a follow-up tool for inventory."

Two Procore projects per entity:
1. **Inventory Project** (e.g., "Cyprus Inventory") — the purchase center, tracks what's been ordered from suppliers
2. **Site/Construction Project** (e.g., "Demo for Cyprus") — the project consuming materials from inventory

**The full end-to-end flow:**

```
1. PM/Logistics creates Commitment in Procore (Inventory Project)
   → Line items: material, qty, unit price, cost code (WBS)
   → Attached to supplier company in Procore directory

2. CEO/COO approves Commitment in Procore
   → Status: Executed

3. SmoothX syncs Commitment → QuickBooks as Purchase Order
   → Now visible in QBO as "goods in transit" balance with supplier

4. [Payment happens separately — down payment, etc.]

5. Goods arrive → Logistics notifies Accounting
   → Accountant does "Receive Items" in QBO (manually matches invoice to PO)
   → Item now in QBO inventory with qty + cost

6. To deploy to a project:
   → PM creates Commitment in Site Project to "the company itself" (internal)
   → QBO: internal Sales Order at zero margin (inventory → project cost)
   → Inventory balance decreases, project cost increases (perpetual inventory)

7. SmoothX handles the Procore commitment → QBO sales order sync (TBC with SmoothX team)
```

### Key Insights from Chadi

**1. Dual-control is intentional and important**
> "The logistics will work through Procore and the accounting will follow up through the accounting software. That way you have double control layer... if the logistics guy is doing Procore AND manual inventory, you have a conflict of interest."

- Logistics → Procore (physical movements)
- Accounting → QuickBooks (financial record)
- D-NAMO app should PRESERVE this separation, not collapse it

**2. QuickBooks is the inventory system of record (not Procore)**
> "To have a report where I take all the items item by item with quantity with the price — like a legit inventory report — I don't believe Procore can do it. This will be by QuickBooks."

- QBO inventory module tracks: item, qty, cost, movement history
- QBO classes/projects used for cost distribution across projects
- Procore is the commitment/approval layer; QBO is the financial/inventory layer

**3. Procore's irreplaceable value is budget tracking per project**
> "If we have the BOQ and the budget in Procore, I'm using it as a cost center by project. All the cost of the project — I'm using it to do budget versus actual buy item."
> "If we use the same energy to do our own thing, maybe it's easier."

- Chadi acknowledged Procore could be replaced IF the app does: commitments, approvals, cost codes, budget vs actual
- His Excel budget format: Materials / PM team cost / Labor team / Project OPEX → compared monthly vs QBO actuals
- QuickBooks doesn't have a budget module natively → currently an Excel workaround

**4. The manual touchpoints D-NAMO can eliminate:**
- Item receipt in QBO (currently manual — logistics tells accountant, accountant enters)
- Creating cost code items in Procore WBS (manual, tedious, item by item)
- Payment applications inside Procore (manual)
- Matching delivery invoice to PO (currently done by accountant in QBO)
- Budget vs actual tracking (currently Excel + QBO manual reconciliation)

**5. Petty Cash as the model**
> "Like the thing we did for the petty cash — why give everyone [access to QuickBooks]? We're making our own thing. It's just an approval process and someone to fill the items."

- Petty cash module = built outside QBO, feeds QBO via CSV/API
- Same pattern applies to inventory: D-NAMO app handles approvals + data entry, QBO gets the clean output

### What D-NAMO Inventory App Should Do (From This Interview)

**The app sits BETWEEN Procore and QBO** — it's the bridge that automates the manual handoffs:

| Today (Manual) | D-NAMO App (Automated) |
|---|---|
| Logistics tells accountant goods arrived | Delivery receipt photo → OCR → auto-signal to accounting |
| Accountant manually receives items in QBO | OCR-matched receipt → auto "receive items" signal to QBO |
| PM/Logistics manually creates cost codes in Procore | Item database in D-NAMO maps to Procore cost codes |
| Budget vs actual tracked in Excel | D-NAMO surfaces QBO actuals vs committed budgets in real-time |
| Invoice doesn't match PO → manual reconciliation | OCR extracts invoice → auto-diff against Procore commitment |
| Logistics controls both physical + financial (conflict of interest) | D-NAMO enforces separation: logistics reviews delivery, accounting approves financial |

### New Open Questions Surfaced
- Does D-NAMO need a full budget module? Or just surface QBO actuals + Procore commitments in one view?
- Should the inventory app write back to QBO (auto-receive items) or just notify the accountant to do it?
- How does the internal transfer (Inventory Project → Site Project) get triggered? Via D-NAMO or stays in Procore?
- SmoothX handles Procore → QBO sync today. Does D-NAMO replace SmoothX eventually, or do they coexist?
- Cost codes (WBS) in Procore are painful to create. Should D-NAMO maintain its own item catalog that maps to Procore codes?

---

## Core Product Decision (2026-03-12)

### Drop Procore. Mirror the Accounting App.

**Decision:** The inventory app will use the accounting app as its architectural blueprint. Procore is dropped as a dependency — D-NAMO Inventory becomes the system of record for physical inventory. QBO remains the financial system of record.

**Copy from accounting app (same infrastructure):**
- Auth + sign-in (Supabase + Azure SSO)
- Permissions / RBAC (same two-level model, `inventory` module roles)
- OCR pipeline (Mistral Pixtral → confidence scoring → duplicate detection)
- Triage queue (entity-first, role-scoped views, confidence heat strips)
- Approval workflow (reviewer → approve/reject → export)
- QBO connection (same integration pattern — feeds QBO like petty cash does)
- Audit trail (all actions logged)
- Email + SharePoint document intake

**Invent for inventory (domain-specific):**

| Accounting | Inventory equivalent |
|---|---|
| Document types: invoice, receipt, PO | Document types: delivery receipt, BOQ, material issue slip, GRN |
| `acct_extracted_data` (vendor, amount, date, tax…) | `inv_extracted_data` (material, qty, unit, supplier, delivery date…) |
| Roles: admin, reviewer, viewer | Roles: admin, storekeeper, logistics officer, PM, controller |
| Approve invoice → signal QBO as Bill | Approve delivery receipt → update stock + signal QBO "receive items" |
| Export: QBO Bill / CSV | Export: QBO inventory receipt / stock movement ledger |
| Duplicate detection: same vendor + doc number | Duplicate detection: same delivery + same items |
| Review queue: accountant sees invoices | Review queue: storekeeper sees delivery receipts; PM sees issue slips |

**New data models to design:**
- Materials catalog (item name, SKU, unit, category, cost code)
- Locations (warehouse, site storehouses, with-subcontractor)
- Stock levels (per material, per location, per entity)
- Movements ledger (issue, return, transfer, receipt — append-only)
- Two-tier material classification: project items (PO-tracked) vs consumables (bulk/replenish)
- Delivery dashboard + low-stock alerts
- PO-to-receipt matching (commitment → delivery receipt diff)

**Dual-control preserved:**
- Logistics/Storekeeper: reviews and approves physical delivery receipts in D-NAMO
- Accounting (Chadi): gets the clean output to QBO — same as today but automated
- No single person controls both physical and financial records

---

## Phase 1 — Assumption Reversal (2026-03-12)

### Assumption #1: "The OCR pipeline belongs at the platform level"
**Reversal tested:** Each module should own its own OCR pipeline.
**Verdict: ✅ Confirmed — platform OCR is correct.**

**[Reversal #1]**: The Platform Already Exists
_Concept:_ The accounting app is not a child of the platform — it IS the platform, v1. Inventory doesn't build its own OCR; it onboards to an existing working system. The "lift to platform" is really just a rename + parameterization of what's already running. Zero greenfield platform risk.
_Novelty:_ Reframes the migration cost from "big architectural effort" to "rename a table, add a column, update the intake router."

**Key insight from Excel sheet review:**
- The Reception sheet (goods-in log) maps 1:1 to what OCR needs to extract: supplier, invoice #, item description, unit, qty, unit price
- The Cost Code List (125-row 5-level WBS taxonomy) is the materials catalog backbone
- Balance sheet = computed from movements ledger in real-time
- Dispatched by Site = per-project deployment view

**[Reversal #2]**: The Excel Sheet Is Already the Schema
_Concept:_ The 5-sheet Excel model is the exact relational schema D-NAMO needs — Reception = documents + extracted_data, Disbursement = movements, Balance = stock view, Dispatched by Site = project view. The data model is already designed; the job is to automate the inputs.
_Novelty:_ No schema design from scratch — reverse-engineer the Excel into normalized tables and the architecture is done.

**[Reversal #3]**: Catalog-First, Not OCR-First
_Concept:_ The materials catalog is the prerequisite that makes OCR useful. OCR extracts item description → fuzzy match against catalog → cost code suggestion auto-populated → storekeeper one-click confirms or overrides. Without a populated catalog, cost code suggestions are impossible.
_Novelty:_ Inverts the build order — catalog population (importing Cost Code List + item master from Excel) is Day 1 work, not a later feature.

**[Reversal #4]**: The Cost Code Tree Is a First-Class Data Model
_Concept:_ The 5-level WBS isn't a static lookup table — it's a live, editable hierarchy that users extend over time. Needs proper tree CRUD, not just a flat import. Items get added organically as first delivery receipts are reviewed; cost code taxonomy imported on day one.
_Novelty:_ Dynamic catalog grows with the business; cost code changes don't require a developer.

**Intake path confirmed:** SharePoint folder monitoring (separate folder per app) → OCR pipeline. Active upload (drag/drop in-app) is the primary path for storekeepers at point-of-receipt. SharePoint is the secondary/bulk path.

---

### Assumption #2: "Procore should be dropped — D-NAMO becomes the inventory system of record"
**Reversal tested:** Procore should stay as the backbone.
**Verdict: ✅ Confirmed — drop Procore.**

**[Reversal #2 — Confirmed]**: Drop Procore While Adoption Is Still Low
_Concept:_ The team is in early Procore adoption — habits haven't formed. The window to redirect to D-NAMO without organizational pain is right now. Once PMs build workflows around Procore's commitment UI, migration becomes a political and behavioral problem.
_Novelty:_ The decision to drop Procore isn't just architectural — it's a change management opportunity with a closing window.

**Budget vs actual:** D-NAMO surfaces budget vs actual at a high level, role-gated:

| Role | Queue Access | Budget View |
|---|---|---|
| Storekeeper | Delivery receipts for their location | No |
| Logistics Officer | Material issue slips, dispatch | No |
| PM | Issue slips for their project | Yes — their project only |
| Controller / Group Controller | All documents, all entities | Yes — all projects |
| Accounting (Chadi) | Financial exports, QBO signals | Yes — cost view |

**[Idea #5]**: Role-Gated Budget Dashboard
_Concept:_ Budget vs actual view is a first-class PM feature — committed spend vs received materials vs deployed to project, per cost code. Replaces both Procore's budget module AND Chadi's Excel workaround, with data already in D-NAMO.
_Novelty:_ Single source of truth for budget vs actual, no Excel reconciliation.

---

### Assumption #3: "The inventory app should be a structural mirror of the accounting app"
**Reversal tested:** Inventory is fundamentally different and mirroring accounting forces the wrong UX.
**Verdict: ❌ Reversed — different UX for inventory.**

**[Reversal #3 — Partial]**: Active Upload Complements the Queue — Does Not Replace It
_Concept:_ The inventory app mirrors the accounting queue UX (entity-first grouping, confidence heat strips, approve/reject, same role-scoped views) AND adds an active upload UI for point-of-receipt processing. Both paths feed the same pipeline. The queue handles SharePoint-dropped documents in bulk; active upload handles real-time dock processing.
_Novelty:_ Two entry points, one pipeline. The storekeeper at the dock doesn't need to wait for the queue cycle — they upload and confirm in real-time. The bulk path remains for documents that arrive ahead of or after the physical delivery.

**Two entry points:**
1. **Active (primary):** Storekeeper/Logistics opens app → drag/upload receipt → real-time OCR → immediate confirm
2. **Passive (queue):** SharePoint folder → documents accumulate → same queue UX as accounting → bulk review

---

### Assumption #4: "Two-vendor discipline (Railway + Supabase only) is the right constraint"
**Reversal tested:** The two-vendor rule is artificial and will force bad architectural decisions.
**Verdict: ✅ Confirmed — constraint holds.**
Supabase covers pgvector (fuzzy matching), Realtime (live stock levels), pg_cron (background jobs). Railway handles FastAPI + synchronous OCR on upload. No gap identified.

---

## Phase 2 — Morphological Analysis (2026-03-12)

### Architectural Decision Matrix

| Dimension | Decision | Rationale |
|---|---|---|
| **Accounting migration** | New `inv_*` tables in same Supabase project; `acct_*` tables untouched | Accounting is in production — zero disruption. Inventory builds clean from day one. Merge later. |
| **Extracted data schema** | Separate `inv_extracted_data` table (mirrors `acct_extracted_data` structure, not shared base yet) | Clean copy of accounting pattern. Merge into shared base table later when there's bandwidth. |
| **Queue UI** | Each app has its own queue + active upload, reading from its own tables | Queue mirrors accounting UX. Active upload added on top for point-of-receipt processing. Both paths feed the same pipeline. |
| **OCR service location** | Stays in accounting repo; inventory calls it as an internal API | Already working. No premature extraction. Move to D-NAMO.Master when platform solidifies. |
| **QBO connector** | Inventory builds its own | No coupling to accounting. Inventory sends "receive items"; accounting sends "bill". Different signals. |
| **Repo architecture** | Separate repo, shared Supabase project | New Railway app + new repo. Same Supabase project — shared users, entities, RBAC, auth. |

### Key Constraint Confirmed
Same Supabase project = shared auth. One login for users who access both accounting and inventory (e.g. Chadi). Entity isolation maintained via RLS. Inventory gets its own schema/tables — `acct_*` tables completely untouched.

---

## Phase 3 — Solution Matrix (2026-03-12)

### Location Hierarchy

```
In-Transit
    ↓  (Logistics Officer logs delivery receipt + OCR)
Country Warehouse  ←→  managed by Logistics Officer
    ↓  (PM approves request, Logistics Officer executes)
Site Warehouse A / B / C  ←→  managed by Storekeeper (site-specific)
    ↓  (Storekeeper logs usage)
Used on site
    ↑  (Storekeeper logs returns)
```

### Roles

| Role | Scope | Primary Responsibility |
|---|---|---|
| **Storekeeper** | Site Warehouse (their site only) | Logs daily material usage and returns on-site |
| **Logistics Officer** | Country Warehouse + all Site Warehouses | Manages deliveries, country → site movements, in-transit tracking |
| **Engineer** | Country Warehouse + their project's Site Warehouse | Approves logistics inputs; confirms on-site stock after delivery |
| **PM** | Country Warehouse + all Site Warehouses (own project) | Approves material requests; full override; budget view |
| **Controller / Group Controller** | All locations, all projects, all entities | Read-only oversight; budgeting and reports (Chadi is here) |

**Note:** Accounting (QBO) is not a role in the app — they receive data automatically via QBO integration after PM-approved movements.

### Approval Chain

```
Logistics Officer logs movement → Engineer approves → stock updated
Storekeeper logs usage/return → Engineer approves → stock updated
Logistics Officer requests country → site → PM approves → Logistics executes
```

### Feature × Role Matrix

| Feature | Storekeeper | Logistics | Engineer | PM | Controller |
|---|---|---|---|---|---|
| Log material usage (on-site) | ✅ | — | — | — | — |
| Log material return (site → site warehouse) | ✅ | — | — | — | — |
| Request material (country → site warehouse) | — | ✅ | — | — | — |
| Process delivery receipt (in-transit → country) | — | ✅ | — | — | — |
| Execute country → site movement | — | ✅ | — | — | — |
| Track in-transit shipments | — | ✅ | ✅ | ✅ | — |
| Approve logistics inputs & movements | — | — | ✅ | ✅ | — |
| Approve material requests | — | — | — | ✅ | — |
| Confirm on-site stock after delivery | — | — | ✅ | — | — |
| Override any movement | — | — | — | ✅ | — |
| View stock — site warehouse (own) | ✅ | ✅ | ✅ | ✅ | ✅ |
| View stock — country warehouse | — | ✅ | ✅ | ✅ | ✅ |
| View stock — all sites | — | ✅ | — | ✅ (own project) | ✅ |
| Budget vs actual dashboard | — | — | — | ✅ (own project) | ✅ (all) |
| Reports & exports | — | — | — | ✅ | ✅ |

### Data Model Skeleton

**Core tables (new, in shared Supabase project):**

```
inv_documents           — delivery receipts, BOQs, material issue slips
inv_extracted_data      — OCR output: supplier, item description, qty, unit, unit price, invoice #
inv_materials_catalog   — item master: name, SKU, unit, category, cost_code (5-level WBS tree)
inv_cost_codes          — 5-level WBS hierarchy (seeded from Excel Cost Code List, editable)
inv_locations           — country warehouse + site warehouses, linked to entity + project
inv_stock_levels        — qty on hand per material per location (computed from movements)
inv_movements           — append-only ledger: receipt, issue, return, transfer, adjustment
inv_movement_requests   — logistics requests pending PM approval
inv_review_actions      — engineer/PM approval log
inv_export_records      — QBO "receive items" signals sent
```

**Material states:**
```
in_transit → in_country → on_site → consumed | returned
```

**Two-tier material classification (from original design):**
- **Project items** — PO-tracked, tied to specific project commitments
- **Consumables** — bulk stock, replenished by threshold

### Document Types (OCR intake)
- Delivery receipt (primary — in-transit → country warehouse)
- BOQ / material schedule (future — bulk catalog seeding)
- Material issue slip (country → site)
- Goods Return Note (site → country)

### OCR Flow (Active Upload — Primary Path)
```
Storekeeper / Logistics opens app
→ Drag/upload delivery receipt PDF
→ Real-time OCR (Mistral Pixtral via accounting repo API)
→ Fields populated live: supplier, invoice #, items, qty, unit price
→ Catalog fuzzy match → cost code suggestion per line item
→ User confirms / overrides
→ Engineer approves
→ Stock updated + QBO "receive items" signal sent
```

### OCR Flow (Passive — SharePoint Fallback)
```
Documents dropped into SharePoint folder (separate folder from accounting)
→ SharePoint polling detects new files
→ OCR pipeline runs in background
→ Documents land in review queue
→ Bulk review session: confirm fields, cost codes, approve
```

