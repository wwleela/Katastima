# Katastima — Root MVP-1 Baseline (Source-of-Truth Guardrail)

**Purpose:** Preserve the original Katastima MVP thesis while evaluating wholesale procurement as an adjacent feature track. This document is a concise baseline derived from the prior `Katastima-MVP1-PRD.md` and the later MVP-1 hardening report in the working library. It is not a substitute for either full source artifact.

## 1. Product thesis — unchanged

Katastima turns messy, low-friction retail data (photographed supplier bills and POS sales records) into structured margin and inventory intelligence for India's independent retailers, starting with kirana and small/mid-sized marts.

The MVP-1 question is narrow: **can Katastima turn a retailer's existing data into a reorder or margin decision that the retailer actually acts on?** It is not initially a marketplace, delivery-speed product, private-label business, credit product or autonomous purchasing agent.

Primary user: an owner-operator of an independent kirana or small/mid-size mart. Multi-location chains and other audiences were not validated as the original primary target.

## 2. Original MVP-1 job and workflow

The central job-to-be-done in the PRD is:

> When I look at my stock, I want to know what to reorder and what's actually profitable, so I don't run out of fast-movers or tie up cash in slow ones I can't see clearly.

The root workflow is:

1. Capture an invoice/bill (photo to structured line-item candidates).
2. Validate extracted data, show confidence and let the retailer correct uncertain fields before inventory is updated.
3. Match products conservatively; do not silently merge ambiguous SKUs or pack sizes.
4. Keep inventory visible and update it only through a controlled, confirmed commit.
5. Make margin visible from captured sale-price and cost-price snapshots.
6. Compute a transparent trailing 14-day sales-velocity baseline and classical reorder-point signal when data supports it.
7. Show actionable, low-friction alerts; label sparse or insufficient data instead of inventing a confident recommendation.

**Invisible AI principle:** the owner should receive a clear next action, not be forced to study another analytics dashboard. Financial calculations and reorder logic must be deterministic; use AI only where it adds a measurable capability, such as parsing bills that people would otherwise key in manually.

## 3. Original MVP scope from the PRD

- Invoice/bill ingestion with the existing Gemini OCR cascade.
- Product/SKU normalization, starting with exact or near-exact matching; fuzzy AI matching remains an open question until failure rates justify it.
- Inventory visibility.
- Deterministic margin calculation using recorded sale-item snapshots.
- Trailing 14-day sales velocity.
- Classical reorder point: average daily sales × lead time + safety stock, with explicit insufficient-data handling.
- Inventory-aging/dead-stock identification and simple actionable alerts were listed as proposed / not fully implemented in the original PRD.
- Human review/correction before invoice data commits to stock.
- Manual entry must remain available when OCR fails or confidence is low.

## 4. Original MVP-1 non-goals

The original PRD explicitly defers broad supplier marketplace functionality, kirana-to-kirana fulfilment, private label, a conversational retail assistant, advanced demand forecasting/ML, space optimization and product recommendations. Deferral is not a permanent rejection; those ideas require separate evidence and a staged decision.

Wholesale procurement can be explored as an **adjacent extension**, but it must not erase or redefine the root inventory/margin/reorder job.

## 5. Status ledger — do not silently reconcile dated evidence

The source materials record different development snapshots. Keep their status distinction visible until the repository and live environment are reconciled.

| Area | Earlier MVP-1 PRD snapshot (16 Sep 2026) | Later local hardening report (21 Sep 2026) | Safe current statement |
|---|---|---|---|
| Invoice OCR / cascade | OCR pipeline and cascade described as built; evaluation not run | Local code hardening/tests reported | Do not claim accuracy until evaluated on hand-verified real bills |
| Margin and reorder SQL | SQL written, not run against real data | Migration files and views described as created; explicitly not executed against live database | Live calculations remain unverified until run and hand-checked against real records |
| Manual invoice correction UI | Marked not yet built and required | Later report describes correction persistence/service behavior but does not establish a full live pilot | Verify current code and live flow before claiming end-to-end readiness |
| Alerts / insights | Some components marked proposed/not built in earlier PRD | Later report describes local services/UI and reports 38/38 tests, lint/typecheck and Expo export passing | Local tests/build do not establish live database deployment or retailer usefulness |
| Database / deployment | Live status not established | Explicitly: live Supabase not verified; migrations remain local specifications, zero live DDL executed | Do not call database changes deployed or production-ready without direct verification |
| Retailer value | Product hypothesis | No live retailer outcome proven by the source artifacts | Must observe an actual retailer using the workflow and record the decision/value |

The later local hardening report states 38/38 local tests, clean TypeScript/lint and successful Expo exports across web/iOS/Android. It also explicitly states live Supabase was **not verified** and migrations were **not executed**. Preserve both facts.

## 6. Extension boundary for wholesale

Wholesale may add a procurement decision layer on top of the root MVP outputs:

- Current stock and recent sales → reorder candidate.
- Verified supplier offers → comparable pack-level options and commercial terms.
- Retailer-editable quantity → draft purchase order.
- Explicit retailer confirmation → supplier submission or manual/WhatsApp order handoff.
- Supplier response, goods receipt and invoice reconciliation → compare requested with accepted, delivered and billed outcomes.

This is a staged candidate workflow, not evidence that procurement is implemented. It must not replace invoice capture, inventory visibility, margin intelligence or reorder logic. Do not add automatic ordering/payments, credit, a national marketplace or AI-set financial values in the first pilot.

## 7. Non-negotiable product and engineering rules

- Deterministic calculations for money, margins, units and reorder quantities.
- Source provenance, timestamps and explicit unknown/stale/unverified states.
- Do not invent supplier prices, stock, schemes, delivery, savings or demand.
- Treat SKU, variant, pack size and base-unit matching as critical; uncertain matches require review.
- Human confirmation before any commercial commitment.
- Immutable price/term snapshots at order submission; preserve later discrepancies.
- Tenant/RLS boundaries and idempotency are launch blockers.
- Do not expose one retailer's inventory, orders or negotiated offers to another retailer or unauthorised supplier.
- Measure a baseline before setting improvement targets.

## 8. Decision rule

Proceed from discovery to a narrow procurement prototype only after a repeated retailer problem, comparable real supplier offers, a freshness/availability contract and an order-confirmation path are demonstrated. If fresh supplier data is unavailable, test an order-preparation assistant with manual confirmation rather than presenting offers as live.

**Root MVP guardrail:** every wholesale feature decision must explain how it strengthens the existing “make an accurate margin or replenishment decision from messy retail data” thesis without changing the primary user or silently expanding the MVP.
