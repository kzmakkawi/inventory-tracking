# Engineer Interview — D-NAMO Inventory Tracking

**Role:** Engineer (Consumer/Requester + partial Procurement)
**Date:** 2026-03-09
**Source Questions:** stakeholder-discovery-questions.md — Role 3 (Engineer) + Role 2 (Storekeeper, partial)

---

## Raw Answers

### Engineer Questions (Role 3)

**Q1. When you need materials for a task, how do you get them?**
We get two to three offers from different suppliers, and I choose the best price based on the proposed materials that is compliant with the project specs, then I place a PO.

**Q2. Do you know what's available in the store before you ask, or do you have to go check?**
In most of the times we bring most of the materials from outside, so we have to be precise with the order. So we have a packing list of items received and we track the items being installed. For small consumables items like screws, washers, nuts, cable ties, etc. I usually check.

**Q3. When something you need isn't on site yet, how do you find out when it will arrive?**
Suppliers give us lead time for delivery.

**Q4. Have you ever had work stopped or delayed because materials weren't there when you needed them? How often?**
It is very rare. The small consumables are the trickiest, because they get finished fast.

**Q5. Do you track what materials you've used for a specific activity or location? Or is that someone else's job?**
In Absa it was the storekeeper and site engineer's job. They prepare a table for me and hand it over at the end of each month so that we can invoice.

**Q6. Does anyone ever ask you to account for what materials you used? In what form?**
The critical items that we get from outside — we order it beforehand based on the drawings and BOQ. The materials that we need on a daily basis (consumables) are tracked by storekeeper, site engineer, and supervisor on site.

**Q7. What's the most frustrating materials-related problem you deal with?**
Cable ties, cable lugs, screws, anchor nuts, etc. These are consumed very fast on site.

### Storekeeper Questions (Role 2, answered by same engineer)

**Q1. How do you know what materials are physically in the store right now?**
The storekeeper has his records, and I ask him to give a count of materials I need.

**Q6. Do you report your stock levels to anyone? How often and in what format?**
I report when being asked by management.

**Q7. What are the most common problems you run into?**
Most common problems are again the consumables — tracking them is quite tricky.

**Q8. If your notebook or Excel disappeared tomorrow, what information would be gone forever?**
All information on the Excel/notebook, unless there are hardcopies.

**Q9. What's the most time-consuming thing you do every week that you wish you didn't have to?**
Quantity takeoff, and calculating for each item the needed consumables.

**Q10. If one thing could be automatically tracked for you, what would it be?**
Material stock, delivery status of material on site, and if we have the shipment from outside — the time it needs to reach the port.

---

## Analysis

### Key Findings

1. **Two-tier material model confirmed:** Critical/project-specific items (ordered per PO with packing lists) vs. consumables (high-volume, fast-burn, loosely tracked). The system must handle both distinctly.

2. **Consumables are the #1 pain point** — mentioned repeatedly (Q4, Q7, Storekeeper Q7). Cable ties, lugs, screws, anchor nuts burn through fast and are the hardest to track.

3. **No real-time stock visibility** — engineer relies on asking the storekeeper for a count. No self-service view.

4. **Quantity takeoff is a major time sink** — manually calculating consumables needed per item from drawings/BOQ is the most time-consuming weekly task. BOM-based estimation could be a differentiator.

5. **Delivery tracking is manual** — knows lead times from suppliers but has no centralized view of what's in transit, when it arrives, or port clearance timelines.

6. **Fragile record-keeping** — if the Excel/notebook is lost, everything is gone. No backup, no system of record.

7. **Reporting is reactive** — only reports stock levels "when asked by management." No proactive dashboards or alerts.

8. **Invoicing workflow matters** — storekeeper + site engineer prepare monthly usage tables for the engineer to invoice. This is a downstream use case the system should support.

### Automation Wishlist (from Q10)
- Material stock levels
- Delivery status of materials to site
- Shipping/port ETA for international shipments

### Gaps — Not Yet Answered
- General questions (WhatsApp vs Excel, disputes, data loss) not covered
- Storekeeper Q2–Q5 (material request, issuance, returns, reconciliation) not covered from storekeeper's perspective
- Logistics Officer and PM roles still uninterviewed
