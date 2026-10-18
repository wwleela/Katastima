# Katastima — Roadmap, Metrics and Release Gates

**Current stage:** Discovery/design. No supplier integrations or customer outcomes are yet verified in this repository.

## Roadmap principles

- Start with one procurement loop, not a broad marketplace.
- Separate observed operational facts from forecasts or AI-generated hypotheses.
- Build around current retailer/distributor workflows where possible.
- Validate data freshness and product equivalence before recommending a price.
- No automation may move money or submit an order without explicit retailer approval.

## Stage 0 — Problem and data validation (week 1–2)

**Work**
- Interview 5–10 kirana owners and 3–5 distributors in Hyderabad (targets for discovery, not claimed participants).
- Collect anonymised examples of purchase invoices, price lists, trade schemes, and order/receipt records with permission.
- Select one category with frequent replenishment and sufficient supplier overlap.
- Map how pack sizes, GST, discounts, freight, credit terms, MOQ, delivery cut-offs and returns are represented.
- Measure current time from “need identified” to a submitted order; record stockout, partial-fill and discrepancy examples.

**Exit criteria**
- A specific user/job and measurable current-workflow baseline.
- At least two suppliers offering comparable data for a small common SKU set.
- A clear way to confirm availability and final order acceptance.
- Defined permission to store and use supplier commercial data.

**Stop condition:** if no supplier can provide sufficiently fresh data, test an order-assist/WhatsApp draft workflow rather than claiming live comparison.

## Stage 1 — Procurement decision MVP (week 3–5)

**Work**
- Import/normalise one category's catalogue and pack units.
- Show stock/reorder suggestions using transparent deterministic rules.
- Compare verified offers and explicit terms.
- Generate an editable purchase-order draft and calculate totals deterministically.
- Require retailer confirmation; start with manual supplier confirmation or generated order sheet.
- Log all offer sources/timestamps, edits, submitted order snapshots and outcomes.

**Exit criteria**
- Price/pack comparison tests pass for representative cases.
- No purchase is submitted without explicit confirmation.
- Duplicate submissions are blocked by idempotency.
- Unknown/stale price and unknown availability states are distinct and visible.
- Cross-retailer and cross-supplier access tests pass.

## Stage 2 — Reconcile, learn, improve (week 6–8)

**Work**
- Record accepted quantities, partial fills, substitutions, delivery dates and final invoice amounts.
- Compare recommendation against fulfilment outcome.
- Build simple supplier reliability indicators only where observation count is visible.
- Add invoice/bill parsing with human review for low-confidence line items.
- Run usability tests with low-end devices and constrained network conditions.

**Exit criteria**
- Each recommendation has evidence, source freshness and a clear edit/override path.
- No AI-parsed price/quantity is used without confidence validation or human correction.
- We can compare pilot baselines against outcomes without assuming causation.

## Stage 3 — Integrations and expansion (only after proof)

Potential candidates: distributor-specific APIs, partner catalogues, ONDC-compatible participants, scheme feeds, automated invoice reconciliation. Choose based on actual partner readiness and integration cost, not on technology trend alone.

Expand one category/area at a time. Do not launch wide catalogues with unreliable identity or availability data.

## Metrics

| Metric | Definition | Why it matters |
|---|---|---|
| Order preparation time | Median minutes from starting replenishment task to confirmed order draft | Direct friction reduction |
| Draft acceptance rate | Suggested orders/items accepted with no or limited edit / suggestions shown | Recommendation usefulness |
| Stockout rate | Stockout SKU-days / active SKU-days, with defined observation window | Availability impact |
| Fill rate | Units received / units confirmed by supplier | Supplier fulfilment reliability |
| Price variance | Difference between displayed confirmed offer and final invoice for comparable lines | Data and commercial trust |
| Effective unit cost | Total comparable spend / saleable base units received, with tax treatment explicit | Avoid headline-price traps |
| Excess stock / days of cover | Stock position divided by transparent sales-rate estimate | Working-capital control |
| Discrepancy rate | Orders with price, quantity, substitution or delivery discrepancy / received orders | Operational friction |
| Freshness coverage | Offers within their agreed freshness window / offers shown | Trust in comparisons |
| Order duplicate rate | Duplicate submissions observed per 100 order attempts | Reliability guardrail |

### Measurement rules

- Measure baseline first; only then set numeric improvement targets.
- Split results by category, store size, supplier and data-quality class where sample allows.
- Show sample size and observation period for supplier metrics.
- Do not claim sales/margin lift from usage correlation alone.
- Track recommendation overrides as product research; do not treat every override as user error.

## Risk register

| Risk | Failure mode | Control |
|---|---|---|
| Stale supplier price | Bad purchase recommendation | Timestamp, TTL, expired state and no false “best offer” |
| Wrong pack mapping | Misleading unit price | Pack-level canonical matching and human review |
| Scheme misinterpretation | Inflated savings | Encode eligibility and threshold; keep assumptions visible |
| Overstock | Cash trapped in inventory | Show order-total and days-of-cover; retailer confirms |
| False availability | Order cannot be filled | Supplier confirmation status, not silence-as-stock |
| Supplier data leakage | Confidential commercial terms exposed | RLS, role boundaries, partner/private offer separation |
| AI extraction error | Incorrect quantity/price | Source trace, confidence, deterministic checks, manual confirmation |
| Duplicate order | Double supply/cost | Idempotency key, request status, replay test |
| Feature bloat | Low adoption | One workflow, few exceptions, test with shopkeepers |
| Market competition | Generic listing app is replaceable | Differentiate on retailer-specific replenishment + outcome reconciliation |

## First execution sequence

1. Get one real retailer's last two invoices and stock list (with permission).
2. Get two supplier lists for overlapping SKUs.
3. Normalise 20–50 products and compare packs manually.
4. Prototype a single “recommended order” screen or WhatsApp-ready order draft.
5. Test whether the user changes or accepts the suggested quantity/supplier.
6. Only after the comparison is trusted, automate submission/status capture.
