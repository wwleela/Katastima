# Katastima — Wholesale Procurement MVP PRD

**Status:** Product hypothesis for discovery and pilot. No live procurement integrations are claimed.  
**Product principle:** recommend and prepare; the retailer decides and confirms.

## 1. Objective

Help an independent retailer decide **what to replenish, which verified supplier offer to use, and what order to place**. Reduce avoidable stockouts, procurement effort and working-capital lock-up without adding a complex marketplace workflow.

## 2. Primary users

- **Retailer / kirana owner:** manages stock and cash; needs a fast, reliable purchase decision.
- **Store operator:** may prepare an order but does not necessarily control payment/approval.
- **Distributor / wholesaler:** provides a catalogue, price list, scheme terms, availability and fulfilment updates.
- **Katastima operator (internal):** maps supplier SKUs to common products, checks data quality and resolves exceptions during the pilot.

Do not assume a consumer marketplace workflow. A store's procurement and a supplier's catalogue have different permissions and commercial terms.

## 3. The first workflow

1. **Observe stock position.** Use store stock, recent sales, pending purchase orders and a retailer-confirmed lead-time assumption. If sales history is insufficient, mark the recommendation as rule-based/low confidence.
2. **Suggest replenishment.** Show a short list of products at or below the reorder threshold with suggested quantity and a plain-language reason.
3. **Compare available offers.** Only compare matched products/packs. Show supplier, pack size, confirmed price, scheme terms, MOQ, estimated delivery, availability update time, freight and taxes separately.
4. **Optimise for usable economics.** Explain the recommended option using effective cost, MOQ/cash impact, delivery fit and observed supplier reliability. The cheapest headline price is not always the recommended offer.
5. **Build a draft purchase order.** Normalise quantities and pack units, show line and total arithmetic, terms and data freshness.
6. **Require retailer confirmation.** Submit through an available supplier workflow. If no integration exists, export/share a clear order request for manual confirmation. Do not claim the supplier accepted it until acceptance is recorded.
7. **Track fulfilment.** Record accepted, rejected, partially filled, substituted, delivered, returned and invoice-reconciled states.
8. **Learn from outcome.** Compare requested quantities/prices with received quantities and final bill; use these observations to improve future stock and supplier recommendations.

## 4. MVP scope

### In scope

- One pilot geography (start with Hyderabad as a discovery/pilot hypothesis).
- One category or narrow SKU group chosen after interviews.
- Supplier price-list import by CSV/Excel or operator-assisted entry.
- Common product/pack mapping and unit conversion.
- Price and scheme validity dates, MOQ, delivery terms, availability timestamp and source.
- Rule-based reorder suggestions with retailer-editable quantities.
- Purchase-order draft, confirmation and status history.
- Bill/receipt capture to reconcile expected versus actual item, quantity and price.
- Basic supplier performance indicators from observed orders only.
- Low-connectivity handling: persist drafts locally or allow export/retry; show pending sync clearly.
- Role-based access, tenant isolation, audit events and idempotent order submission.

### Not in the first MVP

- A nationwide open marketplace.
- Unverified “live” prices or stock availability.
- Automatic purchase placement or payment.
- AI-negotiated prices, autonomous substitutions or credit underwriting.
- Complex financing, lending, supplier payouts or a universal logistics layer.
- A chatbot as the main interface.
- Savings or margin guarantees.

## 5. Product requirements

| ID | Requirement | Acceptance condition |
|---|---|---|
| P1 | Common product identity | Same supplier item is matched by brand/GTIN or manually reviewed equivalence, pack size and base unit. Uncertain matches are not silently merged. |
| P2 | Offer freshness | Every offer has source and updated/valid-through timestamps. Expired or stale offers are flagged and excluded from automatic “best offer” ranking. |
| P3 | Comparable cost | UI shows supplier pack, quantity, MOQ, discounts/schemes, freight, taxes and a normalised per-unit figure with assumptions. No arithmetic uses binary floating point for currency. |
| P4 | Scheme eligibility | Scheme conditions, eligibility, threshold, benefit type and validity are explicit. Do not assume a benefit applies until eligibility can be checked. |
| P5 | Reorder suggestion | Recommendation explains stock on hand, sales window, pending quantity, lead time and safety buffer. Retailer can change qty. |
| P6 | Purchase order | Draft has unique ID, immutable submitted-price snapshots and editable-before-confirmation status. Submit only after explicit confirmation. |
| P7 | Idempotency | Repeated click/network retry with the same request ID cannot create multiple supplier orders. |
| P8 | Fulfilment reconciliation | Received quantities and final billed prices can differ from requested values; discrepancy is recorded, never overwritten. |
| P9 | Failure safety | Stale data, unmapped units, unavailable product or missing supplier confirmation causes a visible exception—not an invented answer. |
| P10 | Tenant privacy | Retailer A cannot read another retailer's inventory, order or negotiated price. Supplier sees only orders addressed to that supplier and fields needed to fulfil them. |

## 6. Recommendation logic: start deterministic

**Reorder quantity (baseline):**

`target stock = expected demand during lead time + safety buffer`

`suggested quantity = max(0, target stock - on-hand stock - confirmed inbound quantity)`

Round to the supplier's order multiple, then flag MOQ and cash impact. Demand during lead time should initially use a transparent trailing-sales average if sufficient data exists, otherwise use a retailer-entered target. Do not label a heuristic as machine-learning forecasting.

**Offer ranking should consider:**
- Exact pack/variant match and offer freshness (hard gates).
- Verified effective unit cost and scheme eligibility.
- MOQ and order-total/cash effect.
- Delivery time/cut-off and stated availability.
- Actual observed fill rate, discrepancy rate and on-time delivery once enough outcomes exist.

The UI should tell the retailer why an offer is ranked first and when the data is insufficient to rank.

## 7. AI boundaries

Good initial uses:
- Parse a supplier invoice or price-list image into structured line candidates.
- Suggest SKU matches with a confidence score and require review for ambiguous variants/units.
- Convert a retailer's natural-language request into a draft reorder list.
- Summarise a confirmed order in the retailer's preferred language.

Do not let an LLM:
- Set a wholesale price, invent stock, infer an unlisted scheme or claim current availability.
- Compute financial totals without deterministic verification.
- Send orders, accept substitutions or approve credit autonomously.
- Treat a low-confidence pack-size match as the same product.

For any LLM extraction, keep source document reference, raw value, parsed value, confidence and correction history.

## 8. Pilot hypothesis

A manageable discovery/pilot target (not existing traction): **5–10 retailers, 3–5 willing distributors, one category, 100–300 mapped SKUs**. Reduce these targets if onboarding or data freshness cannot be managed. Prefer a small, high-quality catalogue over a broad inaccurate one.

## 9. UX principle

Primary interaction should be one actionable question: **“What should I order today?”** The answer should list only a few high-value exceptions, with one-tap inspect/edit actions. Defer full analytics dashboards until interviews show they are necessary.

## 10. Dependencies and risks

- Supplier catalogues may use inconsistent names, units, pack sizes and promotional language.
- Scheme economics may depend on category, retailer status, volume, payment and time.
- Availability can be stale even if a price list is recent.
- Distributors may prefer existing fulfilment and credit systems; Katastima must fit rather than require wholesale replacement.
- Product recommendations can harm margin if cost and tax treatment are inconsistent.
- The market already includes digital procurement initiatives, including DigiDukaan; differentiation must be demonstrated with usage and outcome evidence.

## 11. Open questions before implementation

- Which first category has repeat orders and a willing distributor?
- Will the retailer provide sales/stock data, or must first version use bill capture and manual stock?
- Which supplier fields can be refreshed daily?
- Which supplier will accept a digital draft/order format?
- How should returned, damaged, expired or short-delivered products be reconciled?
- What exact tax and scheme fields are necessary for a correct comparison, validated with a local accounting/tax expert?
