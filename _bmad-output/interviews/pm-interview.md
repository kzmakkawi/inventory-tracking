# Project Manager Interview — D-NAMO Inventory Tracking

**Role:** Project Manager (Visibility & Control + Procurement/Supply Chain)
**Date:** 2026-03-09
**Source Questions:** stakeholder-discovery-questions.md — Role 4 (PM) + Role 6 (Procurement)

---

## Raw Answers

### Procurement Questions (Role 6)

**Q1. When you place an order with a supplier, how do you track its status until it arrives?**
We agree first on the lead time then we do follow-ups via email.

**Q2. How do you know when something has shipped — does the supplier tell you, or do you have to chase?**
In most of the cases we ship the materials. We will be notified by the supplier when the materials are ready and packed for shipping. In case the shipping is in the supplier scope, the follow-up will continue until BOL is provided.

**Q3. Do you track expected delivery dates? Where? How often are they accurate?**
I have an Excel sheet that I use to track my shipments.

**Q4. When a shipment is delayed, who finds out first and how does the information flow to the site?**
We are notified by the shipping companies whenever there is a delay in the arrival of the vessels.

**Q5. How do you match what was ordered against what actually arrived and was invoiced?**
The logistics manager / storekeeper will verify and report. Normally they compare the PO against the goods received and report any deviations.

**Q6. Do you ever have items partially delivered across multiple shipments? How do you track that?**
Yes, sometimes we receive partial deliveries. They are tracked and received on site the same way a full shipment is treated.

**Q7. What documents come with a shipment that you need to verify and file?**
Packing list and invoice.

**Q8. What's the information gap that causes the most problems between ordering and the site actually having the material?**
*(No answer provided)*

### Visibility Questions (Role 4 — PM)

**Q1. Right now, how do you track what's been ordered vs. what's arrived vs. what's been used on a project?**
*(Not answered)*

**Q2. Where does that information live — one person's head, an Excel file, WhatsApp threads?**
*(Not answered)*

**Q3. What's on your ideal weekly materials status report — what do you need to know?**
Updated delivery schedule.

**Q4. When something goes wrong (delayed shipment, wrong material delivered, shortage on site), how do you find out?**
Sometimes we are notified by the supplier or we discover it ourselves from follow-up.

**Q5. Who approves material purchases or purchase orders on your projects?**
As per company policy, I must approve all POs.

**Q6. Do you have visibility on materials sitting in the warehouse that haven't been sent to site yet?**
Honestly no visibility for the moment in the absence of a proper monitoring tool.

**Q7. What do you report to GDH management about materials, and how painful is it to compile?**
Management is concerned only about materials delivery because this will allow generation of invoices and therefore keep a positive cash flow.

**Q8. What's the single biggest inventory or materials problem that has cost a project money or time?**
Having no visibility on the inventories could result in additional orders that generate additional cost on the project and left over in the warehouse. This will also affect the company cash flow.

---

## Analysis

### Key Findings

1. **Zero warehouse visibility** — PM explicitly states "no visibility for the moment in the absence of a proper monitoring tool" (Visibility Q6). This is a direct product mandate.

2. **Duplicate ordering is costing real money** — the single biggest problem cited: no inventory visibility leads to re-ordering materials already in the warehouse, creating excess cost and leftover stock (Visibility Q8). This is the strongest business case for the product.

3. **Cash flow is the management metric** — management cares about materials delivery because it unlocks invoicing (Visibility Q7). The system must connect material arrival to invoicing readiness.

4. **Shipping is often self-managed** — D-NAMO frequently handles its own shipping rather than relying on supplier logistics (Procurement Q2). The system needs to track both supplier-shipped and self-shipped goods.

5. **Tracking is Excel-based and email-driven** — shipment tracking lives in a personal Excel sheet (Procurement Q3), status updates come via email follow-ups (Procurement Q1). No centralized system.

6. **PO-to-receipt matching is delegated** — logistics manager/storekeeper compares PO against goods received and reports deviations (Procurement Q5). This is a manual process ripe for automation.

7. **Partial deliveries happen** — tracked the same way as full shipments (Procurement Q6), but with no mention of how remaining quantities are monitored. Risk of items falling through the cracks.

8. **PM is the PO approval bottleneck** — all POs must go through PM approval (Visibility Q5). Approval workflow is a must-have feature.

9. **Ideal report is simple** — PM just wants an updated delivery schedule (Visibility Q3). The MVP dashboard should prioritize this.

### Critical Product Requirements Emerging

| Requirement | Source |
|---|---|
| Real-time warehouse/stock visibility | Visibility Q6, Q8 |
| Delivery schedule dashboard | Visibility Q3 |
| PO approval workflow | Visibility Q5 |
| PO-to-receipt matching | Procurement Q5 |
| Partial delivery tracking | Procurement Q6 |
| Shipment status tracking (self-shipped + supplier-shipped) | Procurement Q2, Q4 |
| Link material arrival to invoicing readiness | Visibility Q7 |

### Gaps — Not Yet Answered
- Procurement Q8 (biggest information gap between ordering and site) — no answer
- Visibility Q1–Q2 (current tracking method, where info lives) — not answered
- General questions (WhatsApp vs Excel, disputes, data loss) — not covered
