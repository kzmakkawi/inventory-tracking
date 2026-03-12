---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-01b-continue', 'step-05-domain', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish', 'step-12-complete']
inputDocuments:
  - _bmad-output/brainstorming/brainstorming-session-2026-03-05-1200.md
  - _bmad-output/brainstorming/brainstorming-session-2026-03-12-0000.md
  - _bmad-output/brainstorming/stakeholder-discovery-questions.md
  - _bmad-output/interviews/consolidated-findings.md
  - _bmad-output/interviews/engineer-interview.md
  - _bmad-output/interviews/pm-interview.md
workflowType: 'prd'
classification:
  projectType: 'web_app'
  domain: 'Physical asset tracking with document-based verification'
  complexity: 'low-medium'
  projectContext: 'greenfield-app-brownfield-platform'
  deployment: 'Separate URL, separate Railway app, separate repo — shared Supabase project'
  dataEntry: 'Two modes: document-driven (OCR) + manual in-app entry'
  buildApproach: 'Inherit pipeline from accounting; build movement domain only'
  criticalPath: 'Catalog + cost code matching must be populated before go-live'
  coreJob: 'Turn paper into movement records; make records visible across roles'
briefCount: 0
researchCount: 0
brainstormingCount: 2
projectDocsCount: 0
interviewCount: 3
---

# Product Requirements Document - inventory-tracking

**Author:** Khalilmakkawi
**Date:** 2026-03-12

## Executive Summary

D-NAMO Inventory is an internal web application for Group DOUMMAR Holding (GDH) that makes every material movement a permanent, structured, and visible record. Materials flow through three tiers — In-Transit (supplier → country warehouse), In-Country (country warehouse), and On-Site (site warehouses) — tracked today in personal Excel sheets, WhatsApp threads, and storekeeper notebooks. The absence of a centralized system directly causes duplicate orders, delayed invoicing, and cash flow pressure. D-NAMO Inventory eliminates the manual handoffs between the people doing the physical work and the financial system that depends on them: every approved delivery receipt triggers a QuickBooks "receive items" signal automatically, shortening the delivery → invoice → cash collection cycle.

**Target users:** Storekeeper, Logistics Officer, Engineer, Project Manager, Controller/Group Controller (GDH internal only).

**The problem being solved:** No centralized inventory visibility forces GDH to operate reactively — materials are re-ordered because no one knows the warehouse has stock, invoices are delayed because accounting waits for a phone call from logistics, and management has no real-time view of what's arriving when.

### What Makes This Special

D-NAMO Inventory is not a standalone product — it is the second module on the D-NAMO platform, built as a delta on top of the accounting app's proven infrastructure. The OCR pipeline (Mistral Pixtral), SharePoint intake, approval queue, QBO connector, RBAC, and audit trail are inherited, not rebuilt. What's net-new is purely the inventory domain: materials catalog (5-level WBS cost code tree), locations hierarchy (country warehouse → site warehouses), append-only movements ledger, and stock level views.

Two data entry modes serve distinct workflows without overlap: document-driven (delivery receipt scanned → OCR extracts fields → logistics officer confirms → engineer approves → stock updated + QBO signal sent) and manual in-app entry (storekeeper or logistics officer logs a movement directly — "100m cable trays issued to Site A" — no invoice required). Dual-control is preserved by design: logistics owns the physical record, accounting receives the financial output automatically.

The critical path to user adoption has two components. First, catalog quality: cost code matching and catalog fuzzy search at the point of receipt confirmation are the difference between a form the storekeeper completes and one they abandon — catalog + cost code data must be migrated from existing Excel sheets before go-live. Second, functional completeness: the app must deliver a working inventory management experience on day one — real-time stock levels, movement logging (receipt, issue, return, transfer, adjustment), delivery schedule view, and low-stock alerts. Users will not migrate from Excel to a system that shows them less than Excel does.

## Project Classification

| Dimension | Value |
|---|---|
| **Project Type** | Web app — internal SPA, desktop-first |
| **Domain** | Physical asset tracking with document-based verification |
| **Complexity** | Low-Medium |
| **Project Context** | Greenfield app, brownfield platform |
| **Deployment** | Separate URL, Railway app, and repo — shared Supabase project with accounting |
| **Build Approach** | Inherit platform pipeline; build inventory movement domain only |
| **UI Framework** | Vue 3 + Reka UI (headless) + Tailwind CSS — same as accounting app |

## Success Criteria

### User Success

- **Storekeeper** can log a material movement (issue, return, transfer) in under 2 minutes with no training beyond a 10-minute walkthrough
- **Logistics Officer** can process a delivery receipt — upload, confirm OCR fields, cost code match, submit for approval — in under 5 minutes
- **Project Manager** can see the live delivery schedule and current stock levels for their project without asking anyone
- **Controller** can pull a materials status report for any entity on demand, replacing the manual compilation process
- All five roles are **active** within **30 days of go-live** on the Cyprus project — "active" defined per role:
  - Storekeeper: ≥3 movements logged in last 7 days
  - Logistics Officer: ≥1 receipt processed in last 7 days
  - PM: delivery schedule checked at least once in last 7 days
  - Engineer: at least 1 approval action in last 7 days
  - Controller: at least 1 report or stock view in last 7 days

### Business Success

- Zero duplicate orders attributable to stock blindness within 60 days of go-live
  - *Pre-launch baseline required: PM provides estimated duplicate order count + cost for last 6 months before go-live*
- Delivery → invoice cycle shortened: accounting receives QBO "receive items" signals automatically on every approved receipt — no manual notification from logistics required
- Monthly usage table for invoicing generated automatically from the movements ledger — matches existing Excel format exactly, exportable as Excel/CSV
- GDH management can access a real-time materials status view at any time
- Data loss risk eliminated: all inventory records in Supabase, no single-point-of-failure Excel files

### Technical Success

- OCR field accuracy validated on 10–20 real GDH delivery receipts in technical spike (Epic 1) before build begins — field-by-field results for: material description, qty, unit, unit price, supplier, invoice #, delivery date
- OCR spike decision gate: if qty or unit price accuracy < 75% on sample → launch with manual-entry-only first; OCR added in follow-up sprint
- High-consequence fields (qty, unit price) surfaced with elevated confidence warnings in the review UI
- Catalog search returns results in under 300ms; movement entry form applies smart defaults (today's date, last-used location, last-used material)
- QBO integration: inventory sends `ItemReceipt` signal (not `Bill` — different QBO entity from accounting app); OAuth + `python-quickbooks` shared with accounting; business logic net-new
- QBO `ItemReceipt` endpoint validated separately in technical spike
- QBO "receive items" signal fires on every PM-approved receipt — zero manual reconciliation required
- Duplicate detection fires before approval: same supplier + same invoice # + same delivery date flagged
- Supabase Realtime delivers stock level updates within 2 seconds of a movement being approved
- SharePoint second-folder polling operates independently of accounting app's folder
- OCR service unavailability results in "pending OCR" state, not a broken flow
- RBAC role boundaries validated with logistics officer + PM walk-through before build begins

### Measurable Outcomes

| Outcome | Target | Timeframe |
|---|---|---|
| Full role adoption (Cyprus project) | All 5 roles active (per-role definition) | 30 days post-launch |
| Duplicate order incidents | 0 attributable to stock blindness | 60 days post-launch |
| QBO auto-signal coverage | 100% of approved receipts | From day 1 |
| OCR accuracy baseline | Field-by-field, per spike results | Validated pre-build |
| Stock level freshness | ≤2s lag via Realtime | From day 1 |
| Movement entry time | ≤2 min (storekeeper) / ≤5 min (logistics) | From day 1 |

## Product Scope

### MVP

All features below ship together as a single release. Launch does not occur until all are complete.

**Pre-launch tasks (not sprint stories):**
- PM provides duplicate order baseline (count + estimated cost, last 6 months)
- Cost Code List + item master imported from Excel into catalog
- Logistics officer + PM walk-through to validate RBAC role boundaries and location scopes
- Technical spike: OCR field accuracy on sample GDH delivery receipts + QBO `ItemReceipt` endpoint validation

**Application features:**
- Materials catalog: WBS cost code tree (5-level, seeded from Excel import, editable), fuzzy search at point of entry, sub-300ms response
- Locations hierarchy: country warehouse + site warehouses, linked to entity/project
- Movements ledger (append-only): receipt, issue, return, transfer, adjustment
- Document-driven entry: upload delivery receipt → OCR → field confirmation → engineer approval → stock updated + QBO `ItemReceipt` signal
- Manual entry: in-app movement logging form with smart defaults, inline catalog search
- Partial delivery tracking: log received qty against expected, leave remainder open
- Stock level views: per material per location, real-time via Supabase Realtime
- Live delivery schedule: in-transit shipments with expected arrival dates
- Low-stock alerts: configurable threshold per material per location
- Approval chain: Logistics Officer logs → Engineer approves → PM approves requests; notification routing
- QBO integration: `ItemReceipt` signal on PM-approved receipts (OAuth + `python-quickbooks`, net-new business logic)
- RBAC: 5 roles (storekeeper, logistics officer, engineer, PM, controller) with module-scoped permissions
- Duplicate detection: same supplier + invoice # + delivery date flagged before approval
- Monthly usage report: auto-generated per project/date range, matches existing Excel format, exportable as Excel/CSV

### Growth Features (Post-MVP)

- Budget vs. actual dashboard: committed spend vs. received vs. deployed, per cost code (PM and controller)
- Shipment status tracking: BOL upload, shipping company ETA, customs visibility
- Multi-entity rollout: extend beyond Cyprus project to other GDH entities
- Goods Return Note (GRN) document type: OCR intake
- Material issue slip document type: OCR intake

### Vision (Future)

- BOQ parsing: extract materials list from PDF drawings to seed catalog and estimate consumable needs
- WhatsApp intake: field worker photos delivery receipt → platform pipeline routes to inventory queue
- Platform OCR migration: accounting and inventory OCR lifted to D-NAMO.Master as a shared service

## User Journeys

### Journey 1: Tariq — Logistics Officer (Document-Driven Receipt)

*Primary user, happy path.*

It's Tuesday morning. A truck from the freight company pulls into the Cyprus warehouse. Tariq has been tracking this shipment for two weeks — 500m of galvanized cable trays, ordered for the Demo project. He's been following up by email with the shipping company, noting ETAs in his personal Excel sheet.

The driver hands him a delivery note and invoice. Tariq counts the items — 480m arrived, 20m short. He signs the delivery note, takes both documents, and heads back to the office.

At his desk, he opens D-NAMO Inventory, goes to **New Receipt**, and uploads the invoice PDF. Within 30 seconds, the OCR has extracted: supplier name, invoice number, date, and 6 line items with quantities and unit prices. Three fields come back with high confidence; two line items have yellow confidence indicators — the item descriptions are ambiguous. Tariq glances at the original document on the left, confirms the descriptions, and fixes one unit price the OCR misread.

He notes the partial delivery: expected 500m, received 480m. The app logs 480m received and leaves 20m as an open outstanding quantity. He submits for approval.

Fadi, the site engineer, gets a notification. He reviews the receipt, confirms the quantities match what arrived, and approves. The stock level for cable trays in the Cyprus Country Warehouse updates in real time. Chadi in accounting gets a clean QBO `ItemReceipt` signal automatically — no phone call, no email.

**Capabilities revealed:** Upload flow, OCR field confirmation UI, partial delivery tracking, approval notification, QBO signal trigger, real-time stock update.

---

### Journey 2: Karim — Storekeeper (Manual Movement Logging)

*Primary user, daily workflow.*

End of the workday on site. Karim has issued materials to three different crews today. He sits down at the site office computer and opens D-NAMO Inventory.

He clicks **Log Movement → Issue**. The form pre-fills today's date and his location (Site Warehouse A). He types "cable" in the material search — the catalog returns "Galvanized Cable Tray 100x50mm" in under a second. He selects it, enters 25m issued to the electrical crew for Zone B activity, and hits save. 12 seconds.

He repeats for two more items: 50 cable ties (consumable, bulk) and 10 cable lugs. For the cable ties, the system shows a low-stock warning — current level is 120 units, threshold is 100. Karim glances at it and adds a note: "reorder needed."

At the end of the week, his site engineer Fadi pulls up the stock view for Site Warehouse A — every movement Karim logged is there. No notebook to transcribe, no Excel to update. When management asks for the monthly usage table at the end of the month, the system generates it in one click — same format as the Excel they used before.

**Capabilities revealed:** Manual entry form, smart defaults, inline catalog search, consumable tracking, low-stock alerts, stock level view, monthly usage report generation.

---

### Journey 3: Rami — Project Manager (Delivery Schedule + Approval)

*Secondary user, visibility and control.*

It's Thursday. Rami gets a call from the site engineer: "The cable trays aren't here yet and we need to start installation Monday." Rami used to handle this with a frantic round of WhatsApp messages and an Excel sheet he sometimes forgot to update.

Today, he opens D-NAMO Inventory and goes to the **Delivery Schedule** view. He can see every in-transit shipment for the Cyprus project — supplier, expected arrival date, items, quantities. The cable tray shipment is there: ETA Friday, 480m received last week, 20m still outstanding from the same order. He calls the freight company, confirms Friday delivery, and updates the ETA in the app.

He also sees a pending material request from Tariq: 200m of conduit pipe needed from the country warehouse. He reviews the request, checks the country warehouse stock level — 350m available — and approves. Tariq gets a notification to proceed with the dispatch.

At month-end, his GDH management report takes 30 seconds: filter by entity, date range, export. Done.

**Capabilities revealed:** Delivery schedule view, in-transit tracking, ETA update, material request approval, warehouse stock visibility, management report export.

---

### Journey 4: Karim — Edge Case (Damaged Delivery)

*Primary user, error recovery.*

A delivery arrives at the site with 10 units of a junction box visibly damaged. The driver wants Karim to sign for all 40 units. Karim refuses to sign for the damaged 10.

He opens D-NAMO Inventory and creates a receipt for 30 units (the undamaged ones). In the notes field, he logs: "10 units damaged on arrival — refused. Supplier notified." He submits for approval.

Fadi approves the 30-unit receipt. Stock updates for 30 units. The QBO signal fires for 30 units — not 40. Chadi's QBO record matches what was physically accepted.

The outstanding 10 units remain open on the original order. Tariq follows up with the supplier. When the replacement 10 units arrive two weeks later, Tariq creates a second receipt to close out the outstanding quantity.

**Capabilities revealed:** Notes field on receipts, deliberate partial acceptance, outstanding quantity tracking across multiple receipts, audit trail per movement.

---

### Journey 5: Chadi — Controller (Zero-Touch Financial Output)

*Integration user — Chadi never logs into D-NAMO Inventory.*

Chadi's relationship with D-NAMO Inventory is entirely passive. He never opens the app. He never logs a movement. His interaction is: QBO shows him an `ItemReceipt` entry, the quantities and costs match what was physically received, and his inventory asset balance is correct.

On the first day of the month, Chadi opens QBO. The previous month's deliveries are all there — automatically received, matched to purchase orders, quantities and costs correct. His monthly inventory reconciliation, which used to take an afternoon of cross-referencing emails and Excel sheets, takes 10 minutes of spot-checking.

The one time the system matters to him actively: a duplicate delivery receipt slips through from the logistics team (same invoice number, same supplier, same date submitted twice). D-NAMO flags it before approval. The logistics officer sees the duplicate warning, confirms it's a mistake, and cancels the second submission. QBO never sees the duplicate entry.

**Capabilities revealed:** QBO `ItemReceipt` auto-signal, duplicate detection before approval, clean audit trail, zero manual step for accounting.

---

### Journey 6: Khalilmakkawi — Admin (Onboarding a New Entity)

*Admin/operations user.*

GDH is expanding — a new entity, "Lebanon MEP," needs to be onboarded to D-NAMO Inventory. Khalilmakkawi opens the admin panel, creates the new entity, assigns its SharePoint folder path, and sets up the QBO company connection. He invites 3 users: a storekeeper, a logistics officer, and a PM. Each gets a Microsoft SSO invitation.

He then runs the catalog import for Lebanon MEP — uploads the Cost Code List Excel file, maps columns to the catalog schema, and imports. 87 items loaded, 3 flagged for review (duplicate cost codes). He resolves the duplicates and confirms the import.

Lebanon MEP is live.

**Capabilities revealed:** Entity management, user invitation (Microsoft SSO), SharePoint folder configuration, QBO company connection per entity, catalog import tooling, bulk import validation.

---

### Journey Requirements Summary

| Journey | Capabilities Revealed |
|---|---|
| Tariq — Receipt (OCR) | Upload flow, OCR confirmation UI, partial delivery, approval chain, QBO signal, Realtime stock update |
| Karim — Manual movement | Entry form, smart defaults, catalog search, low-stock alerts, stock view, monthly report |
| Rami — PM visibility | Delivery schedule, in-transit tracking, request approval, warehouse visibility, report export |
| Karim — Damaged delivery | Notes on receipts, deliberate partial acceptance, outstanding qty tracking, audit trail |
| Chadi — Controller | QBO auto-signal, duplicate detection, zero-touch accounting output |
| Admin — Entity onboarding | Entity management, user invitation, SharePoint config, QBO per entity, catalog import |

## Web App Specific Requirements

### Project-Type Overview

D-NAMO Inventory is a desktop-first internal SPA built on Vue 3 + Vite. All users are GDH employees on corporate machines — no public-facing surface, no SEO, no mobile requirement.

### Browser Matrix

| Browser | Support |
|---|---|
| Chrome (latest) | ✅ Primary |
| Edge (latest) | ✅ Primary |
| Firefox, Safari | Not required |

### Responsive Design

Desktop-first. Target viewport: 1280px+ (standard office monitor). No mobile breakpoints required for MVP. Tablet support not required.

### SEO Strategy

Not applicable — internal application, no public indexing required. Authentication gates all routes.

### Accessibility Level

Standard usability. No WCAG compliance target. Design for clarity and low digital-literacy users (storekeepers): large click targets, clear labels, minimal cognitive load on entry forms, inline validation feedback.

## Project Scoping & Phased Development

### MVP Strategy & Philosophy

**MVP Approach:** Problem-solving MVP — full functional baseline ships as a single release. Partial launches are not viable: users will not migrate from Excel to a system that shows them less than Excel does. This mirrors the accounting app launch model: complete, working, trusted on day one.

**Resource Requirements:** 3-person dev team. Inherited platform infrastructure (OCR pipeline, auth, RBAC, QBO connector, SharePoint intake) from accounting app eliminates the majority of greenfield risk — build scope is the inventory domain only.

**Core User Journeys Supported:** All 6 — Tariq (OCR receipt), Karim (manual movement), Rami (PM visibility), Karim (damaged delivery edge case), Chadi (zero-touch QBO output), Admin (entity onboarding). See Product Scope for full feature list.

### Post-MVP Features

**Phase 2 (Growth):**
- Budget vs actual dashboard: committed spend vs received vs deployed, per cost code (PM + controller)
- Shipment status tracking: BOL upload, shipping company ETA, customs/port visibility
- Multi-entity rollout: extend beyond Cyprus to other GDH entities
- GRN (Goods Return Note) OCR document type
- Material issue slip OCR document type

**Phase 3 (Vision):**
- BOQ parsing: extract materials list from PDF drawings to seed catalog and estimate consumable needs
- WhatsApp intake: field worker photos delivery receipt → platform pipeline routes to inventory queue
- Platform OCR migration: accounting + inventory OCR lifted to D-NAMO.Master as shared service

### Risk Mitigation Strategy

**Technical risks:** OCR accuracy and QBO `ItemReceipt` both validated in pre-build spikes with explicit go/no-go gates. Inherited accounting app infrastructure eliminates the highest-risk platform components.

**Market risks:** Internal product — adoption risk is behavioral, not market. Mitigated by: catalog pre-populated before go-live (eliminates the #1 dropout point), per-role success criteria tracked at 30 days.

**Resource risks:** 3-person team is viable because domain scope is narrow (inventory movement only) and platform scope is zero (all inherited). If timeline pressure emerges, Phase 2 features absorb it — MVP scope is non-negotiable.

## Functional Requirements

### Document Intake & OCR Processing

- **FR1:** Logistics Officer can upload a delivery receipt document (PDF or image) for OCR processing
- **FR2:** System can extract structured fields from an uploaded delivery receipt (supplier, invoice #, date, line items, qty, unit, unit price)
- **FR3:** Logistics Officer can review OCR-extracted fields alongside the original document and confirm or override each field
- **FR4:** System can display per-field confidence indicators for OCR-extracted data
- **FR5:** System can detect potential duplicate submissions (same supplier + invoice # + delivery date) and warn the submitter before approval
- **FR6:** System can receive documents from a designated SharePoint folder and queue them for review
- **FR7:** Logistics Officer can match OCR-extracted line items to catalog entries via fuzzy search suggestions at point of confirmation

### Materials Catalog Management

- **FR8:** Admin can import materials and cost codes from an Excel file into the catalog
- **FR9:** Admin can create, edit, and deactivate materials in the catalog
- **FR10:** Admin can manage the 5-level WBS cost code hierarchy (create, edit, reorder)
- **FR11:** Any authorized user can search the materials catalog by name, SKU, or cost code during movement entry
- **FR12:** System can suggest catalog matches for OCR-extracted item descriptions using fuzzy search

### Inventory Movement Logging

- **FR13:** Storekeeper can log a material issue (site warehouse → consumed on site)
- **FR14:** Storekeeper can log a material return (on-site → site warehouse)
- **FR15:** Logistics Officer can log a material transfer (country warehouse → site warehouse)
- **FR16:** Logistics Officer can log a manual receipt (in-transit → country warehouse) without an uploaded document
- **FR17:** Admin or Logistics Officer can log a stock adjustment with a mandatory reason
- **FR18:** Logistics Officer can record a partial delivery — log received quantity against expected, leaving the remainder open as outstanding
- **FR19:** Any submitter can add notes to a receipt or movement record
- **FR20:** System can pre-fill movement entry forms with smart defaults (today's date, last-used location, last-used material)
- **FR21:** System can track outstanding quantities across multiple partial deliveries of the same order

### Stock Visibility & Reporting

- **FR22:** Storekeeper can view current stock levels for their assigned site warehouse
- **FR23:** Logistics Officer can view current stock levels across the country warehouse and all site warehouses
- **FR24:** PM can view current stock levels for locations within their project scope
- **FR25:** Controller can view current stock levels across all locations and entities
- **FR26:** System can deliver real-time stock level updates within seconds of a movement being approved
- **FR27:** Logistics Officer and PM can view a delivery schedule of all in-transit shipments with expected arrival dates
- **FR28:** Logistics Officer or Admin can update the expected arrival date for an in-transit shipment
- **FR29:** System can trigger a low-stock alert when material quantity at a location falls below a configurable threshold per material
- **FR30:** PM and Controller can generate a monthly usage report filtered by project, location, and date range
- **FR31:** PM and Controller can export the monthly usage report in Excel/CSV format
- **FR32:** Logistics Officer and PM can create an in-transit shipment record (supplier, items, expected qty, ETA) linked to a purchase order, seeding the delivery schedule

### Approval Workflow

- **FR33:** Logistics Officer can submit a receipt or movement for engineer approval
- **FR34:** Engineer can approve or reject submitted receipts and movements, with an optional comment
- **FR35:** Logistics Officer can submit a material transfer request (country → site) for PM approval
- **FR36:** PM can approve or reject material transfer requests
- **FR37:** PM can override any pending movement or request
- **FR38:** System can send in-app notifications to the appropriate approver when an action is pending
- **FR39:** System can determine the final approver for each movement type based on movement category, then update stock levels and trigger the QBO signal upon that final approval
- **FR40:** Engineer can view a queue of all movements and receipts pending their approval across their assigned locations
- **FR41:** System can display a notification feed showing pending and historical action items for the logged-in user
- **FR42:** Logistics Officer and PM can receive low-stock alerts for locations within their scope
- **FR43:** Storekeeper can submit a material request to the Logistics Officer for materials to be transferred from the country warehouse to their site warehouse

### QBO Integration

- **FR44:** System can send a QBO `ItemReceipt` signal to the connected QuickBooks company upon PM-approved receipt
- **FR45:** Admin can configure the QBO company connection per entity
- **FR46:** Controller can view the status of QBO export records (sent, pending, failed)
- **FR47:** System can send a QBO signal on approved stock adjustments to reflect inventory write-offs or corrections in QuickBooks

### Administration & Access Control

- **FR48:** Admin can create and configure a new entity in the system
- **FR49:** Admin can assign a SharePoint folder path per entity for passive document intake
- **FR50:** Admin can invite users via Microsoft SSO and assign them roles per entity
- **FR51:** System can enforce role-based access — each role can only view and act within their assigned location and feature scope
- **FR52:** System can maintain an append-only audit trail of all movement, approval, and export actions
- **FR53:** Controller and Admin can view the full audit trail for any movement, document, or approval action

### Locations Management

- **FR54:** Admin can create and manage locations (country warehouse, site warehouses) linked to an entity and project
- **FR55:** Admin can deactivate a location without deleting its movement history

## Non-Functional Requirements

### Performance

All response time targets measured at p95 under normal operating load with a fully populated catalog.

- Catalog fuzzy search returns results in < 300ms from user input
- Stock level updates delivered to all connected clients within ≤ 2 seconds of movement approval via Supabase Realtime
- OCR field extraction completes within 30 seconds of document upload
- Initial SPA bundle loads in < 3 seconds on corporate LAN
- Movement entry form renders within < 500ms
- QBO signal fires within 10 seconds of final approval (asynchronous — does not block the user's confirmation screen)

### Security

- All data encrypted in transit (HTTPS/TLS) and at rest (Supabase default encryption)
- Row-Level Security enforced at the database layer — no query can return data outside the user's entity scope, regardless of application logic
- RLS policies must be validated in a dedicated security test pass before go-live — not assumed correct from code review alone
- Authentication exclusively via Supabase Auth + Microsoft SSO — no custom credential storage
- QBO OAuth tokens stored server-side only, never exposed to the frontend
- All movement, approval, and export actions written to an append-only audit log — no record can be modified or deleted

### Integration Reliability

- OCR service (Mistral Pixtral) unavailability must not block document submission — document enters "pending OCR" state and can be manually completed; no error state shown to user
- QBO signal failure must not block movement approval — signal retried asynchronously; failure status visible to Controller via FR41
- SharePoint polling failure must not affect the active upload path — the two intake paths are independent
- Supabase Realtime disconnection must fall back gracefully — UI shows last-known state with a staleness indicator, not incorrect data
- Daily automated backups with point-in-time recovery (PITR) enabled on the shared Supabase project — recovery point objective (RPO) ≤ 24 hours

### Data Integrity

- The movements ledger is append-only — no hard deletes on movement records; corrections are made via adjustment entries with mandatory reason
- Stock level updates and movement ledger appends must execute within the same database transaction — no application-layer split; enforced via Supabase database function or trigger
- Stock levels must always be consistent with the movements ledger — no stock level can be set directly outside a transaction-bound movement record
- QBO `ItemReceipt` signals must be idempotent — duplicate signals for the same approved receipt must not create duplicate QBO entries
