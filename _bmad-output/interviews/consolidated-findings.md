# Consolidated Interview Findings — D-NAMO Inventory Tracking

**Date:** 2026-03-09
**Interviews Completed:** Engineer, Project Manager
**Interviews Pending:** Logistics Officer, Storekeeper, Warehouse/Logistics Coordinator, Procurement (dedicated)

---

## Top-Level Problem Statement

D-NAMO has **no centralized inventory visibility**. Materials are tracked in personal Excel sheets, storekeeper notebooks, and email threads. This directly causes duplicate orders, wasted budget, warehouse leftovers, and cash flow pressure. The PM explicitly confirmed: "no visibility for the moment in the absence of a proper monitoring tool."

---

## Validated Pain Points (ranked by severity)

### 1. No Inventory Visibility → Duplicate Orders → Cash Flow Hit
- **Source:** PM (Visibility Q6, Q8)
- **Impact:** Financial — unnecessary purchases, excess warehouse stock, negative cash flow effect
- **Severity:** Critical — this is the single biggest cost/time problem cited

### 2. Consumables Are Untrackable
- **Source:** Engineer (Q4, Q7, Storekeeper Q7)
- **Impact:** Operational — cable ties, lugs, screws, anchor nuts burn through fast with no reliable tracking
- **Severity:** High — most frequently mentioned frustration across both interviews

### 3. Fragile, Siloed Record-Keeping
- **Source:** Engineer (Storekeeper Q8), PM (Procurement Q3)
- **Impact:** Data loss risk — if Excel/notebook disappears, all records are gone
- **Severity:** High — no backup, no system of record, no shared access

### 4. Manual Quantity Takeoff for Consumables
- **Source:** Engineer (Storekeeper Q9)
- **Impact:** Time — most time-consuming weekly task, calculating consumable needs per item from drawings/BOQ
- **Severity:** Medium-High — automation opportunity, potential differentiator

### 5. No Delivery/Shipment Dashboard
- **Source:** Engineer (Storekeeper Q10), PM (Visibility Q3, Procurement Q1)
- **Impact:** Operational — no centralized view of what's ordered, in transit, or arriving when
- **Severity:** Medium-High — PM's ideal report is simply "updated delivery schedule"

### 6. Reactive Reporting Only
- **Source:** Engineer (Storekeeper Q6), PM (Visibility Q7)
- **Impact:** Management blind spots — stock levels reported only when asked, no proactive alerts
- **Severity:** Medium

---

## Confirmed Workflow Patterns

### Two-Tier Material Model
| | Critical/Project Items | Consumables |
|---|---|---|
| **Sourcing** | 2–3 supplier quotes → PO based on specs | Bulk order, replenish as needed |
| **Tracking** | Packing list, PO matching, item-level | Storekeeper notebook/memory |
| **Visibility** | Known (ordered per drawing/BOQ) | Low (fast-burn, loosely tracked) |
| **Pain level** | Moderate (process exists, just manual) | High (no reliable process) |

### Procurement Flow
1. Engineer gets 2–3 quotes from suppliers
2. PM approves all POs (company policy)
3. Lead time agreed with supplier
4. Follow-up via email until shipment
5. D-NAMO often handles own shipping; supplier notifies when goods are packed
6. BOL obtained when supplier ships
7. Shipping company notifies of delays
8. Logistics manager/storekeeper verifies receipt against PO, reports deviations

### Material-to-Invoice Flow
1. Storekeeper + site engineer track materials installed
2. Monthly usage table prepared
3. Handed to engineer for invoicing
4. Management watches delivery → invoice → cash flow pipeline

---

## Emerging Product Requirements

### Must-Have (MVP)
| Requirement | Business Case | Source |
|---|---|---|
| Real-time stock/warehouse visibility | Prevents duplicate orders, saves money | PM Q6, Q8 |
| Delivery schedule dashboard | PM's #1 ask, replaces Excel tracking | PM Vis-Q3, Proc-Q3 |
| PO-to-receipt matching | Currently manual comparison by storekeeper | PM Proc-Q5 |
| PO approval workflow | PM must approve all POs per policy | PM Vis-Q5 |
| Consumables stock tracking with low-stock alerts | #1 operational pain point | Eng Q4, Q7 |

### Should-Have
| Requirement | Business Case | Source |
|---|---|---|
| Partial delivery tracking | Happens regularly, risk of items lost between shipments | PM Proc-Q6 |
| Shipment status tracking (self-shipped + supplier-shipped) | D-NAMO often handles own shipping | PM Proc-Q2, Q4 |
| Material usage reporting (monthly, per activity/location) | Feeds invoicing workflow | Eng Q5 |
| Port/shipping ETA tracking | International shipments need customs visibility | Eng SK-Q10 |

### Could-Have (Differentiators)
| Requirement | Business Case | Source |
|---|---|---|
| BOM-based consumable estimation from drawings/BOQ | Automates the most time-consuming weekly task | Eng SK-Q9 |
| Link material arrival to invoicing readiness | Management's core concern is cash flow | PM Vis-Q7 |
| Proactive stock level reporting / dashboards | Replace reactive "report when asked" model | Eng SK-Q6 |

---

## Unanswered Questions (Follow-Up Needed)

### From PM Interview
- **Visibility Q1:** How do you currently track ordered vs. arrived vs. used?
- **Visibility Q2:** Where does that information live?
- **Procurement Q8:** Biggest information gap between ordering and site having material?

### From Engineer Interview
- **General questions** not covered: tools used, disputes, data loss experiences

### Uninterviewed Roles
- **Logistics Officer** — delivery receiving, document handling, on-site logging
- **Storekeeper (dedicated)** — issuance process, request handling, returns, reconciliation
- **Warehouse/Logistics Coordinator** — customs, warehouse management, dispatch to site
- **Procurement (dedicated)** — if separate from PM

---

## Key Quotes

> "No visibility for the moment in the absence of a proper monitoring tool." — PM

> "Having no visibility on the inventories could result in additional orders that generate additional cost on the project and left over in the warehouse." — PM

> "Cable ties, cable lugs, screws, anchor nuts — these are consumed very fast on site." — Engineer

> "All information on the Excel/notebook, unless there are hardcopies." — Engineer (on what's lost if records disappear)

> "Quantity takeoff, and calculating for each item the needed consumables." — Engineer (most time-consuming weekly task)
