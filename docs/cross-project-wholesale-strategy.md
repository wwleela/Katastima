# Cross-Project Wholesale Strategy — Katastima + TripWise

**Decision date:** 18 October 2026  
**Purpose:** Treat wholesale as two related but separate vertical problems. Do not force one generic feature into both products.

## Executive decision

| Project | Right interpretation of wholesale | Recommended priority | Why |
|---|---|---:|---|
| **Katastima** | A kirana buys stock from wholesalers/distributors and must optimise replenishment, effective cost, cash, availability and fulfilment. | **P0 — discover and prototype first** | This directly fits Katastima's stated independent-retail operating-system thesis. Existing BharatShop reference schema has stock and a single store-level cost price, but no multi-supplier offer or purchase-order model. |
| **TripWise** | A travel agent/tour operator acquires service inventory at supplier net rates and assembles customer-facing quotes with margin, availability and contractual terms. | **P1 — separate B2B discovery track** | Current TripWise is a consumer, budget-aware itinerary planner. Wholesale would introduce a different user, rate model, supplier relationship and operational risk. Validate it as a potential Partner Desk, not as a tab in the existing capstone app. |

Market evidence supports investigating both categories, not assuming success. The Government of India and ONDC identify digital procurement as an active focus for general trade and report early DigiDukaan traction in Hyderabad. TBO describes a large B2B travel distribution network connecting travel buyers and suppliers. These are category signals; they do not prove that either proposed Katastima or TripWise feature has product-market fit.

Sources:
- [PIB — Kirana procurement digitisation and Hyderabad traction, 12 June 2026](https://www.pib.gov.in/newsite/erelcontent.aspx?lang=2&reg=48&relid=289960)
- [ONDC — DigiDukaan for B2B procurement](https://www.ondc.org/pages/digidukaan.html)
- [TBO — Company profile and travel distribution network](https://www.tbo.com/engagement/investors/)

## Shared principles worth reusing

Both workflows involve fragmented supplier data and decisions where incorrect terms can create financial loss. Reusable technical patterns:
- Source provenance, timestamps and explicit data freshness.
- Exact product/variant/units and commercial-condition normalisation.
- Clear unknown/stale/unconfirmed states instead of hallucinated confidence.
- Immutable snapshots of prices and terms at order/quote submission.
- Human confirmation before any commercial commitment.
- Idempotency and auditable status history.
- Role-scoped permissions; supplier-only and buyer-specific commercial terms must not leak across tenants.
- Outcome reconciliation: compare what was offered/ordered against what was confirmed, delivered and charged.

**Do not share domain tables just because the pattern is similar.** Grocery cartons and travel experiences have different price bases, availability rules, tax/cancellation terms and transaction states.

## Katastima first: the minimum usable loop

1. Obtain one retailer's recent stock/purchase history plus two distributor offer lists for overlapping SKUs (with permission).
2. Normalise 20–50 SKUs and their pack sizes, supplier prices, schemes, MOQ, availability timestamps and delivery terms.
3. Prepare a reorder list from stock and observed sales; show the reason for each suggested quantity.
4. Compare only verified, comparable offers. Display cash/MOQ impact and missing fields.
5. Produce an editable order draft; require explicit retailer confirmation.
6. Initially send/share the order for manual supplier confirmation; record acceptance and actual receipt/invoice.
7. Evaluate order preparation time, acceptance/override, stockouts, fill rate, price variance and discrepancy rate against the baseline.

Do not start with a nationwide wholesale marketplace, AI purchasing agent, credit feature or automated payments.

## TripWise second: prove agent pain before code

1. Interview 5–8 independent travel agents and 3–5 suppliers.
2. Examine real quote requests to capture net rates, date validity, availability, inclusions, taxes/fees, cancellation terms and markup workflow.
3. Test one destination and one supplier category with structured supplier lists and manual confirmation.
4. Only proceed if agents share a repeated problem, suppliers can provide current terms, and quote accuracy/response can be measured.
5. Keep agent net rates confidential; only an approved sell price and customer-relevant terms appear in the customer quote. A booking is not confirmed until the supplier confirms it.

Current TripWise capstone success metrics and scope should remain consumer-focused unless this separate track is approved.

## Cross-project efficiency objective

The common aim is not to “add AI” or “add wholesale.” It is to remove manual reconciliation **without trading manual work for inaccurate automation**.

For each feature, the release gate is:
1. One validated user problem.
2. One narrow workflow that can be completed end-to-end.
3. Data sources and freshness rules documented.
4. Deterministic financial arithmetic and explicit confirmation.
5. Idempotency, privacy, and failure-path tests.
6. A baseline and observed outcome metric.
7. A go/no-go decision based on evidence, not feature count.

## Current work products

### Katastima
- [Root MVP-1 baseline and scope guardrails](root-mvp1-baseline.md)
- [Market evidence and problem framing](market-research-and-problem.md)
- [Wholesale procurement PRD](wholesale-procurement-prd.md)
- [Data model and architecture](data-model-and-architecture.md)
- [Roadmap, metrics and release gates](roadmap-and-metrics.md)
- [Discovery tracking issue](https://github.com/wwleela/Katastima/issues/1)

### TripWise
- [B2B travel distribution / wholesale fit assessment](https://github.com/wwleela/TripWise.io/blob/main/projects/tripwise-capstone/docs/b2b-travel-distribution-fit.md)
- [Discovery tracking issue](https://github.com/wwleela/TripWise.io/issues/1)
