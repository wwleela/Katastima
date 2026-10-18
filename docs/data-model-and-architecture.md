# Katastima — Wholesale Data Model and Architecture

**Status:** Conceptual design only. Review against user research, supplier data examples and database constraints before applying migrations.

## 1. Existing reference baseline

The current [BharatShop reference schema](https://github.com/wwleela/bharatshop-architecture/blob/main/supabase/schema.sql) defines four tables: `stores`, `products`, `sales` and `scan_logs`. Its `products` table holds a store's product identity, cost_price, sell_price, stock, low-stock threshold, unit and tags. That supports a simple single-purchase-cost inventory model, but not multi-supplier procurement.

**Do not overload `products.cost_price` with wholesale offers.** A store product is what the retailer sells/holds. A supplier offer is a time-dependent listing for a specific pack under specific commercial terms. Multiple suppliers can offer the same retailer product at different prices, MOQ, scheme, delivery and validity.

## 2. Recommended entities

| Entity | Purpose | Critical fields |
|---|---|---|
| `retail_products` (or existing tenant-owned products) | Store's internal catalogue and on-hand stock | store_id, canonical_product_id, name, base_unit, stock_on_hand, reorder policy |
| `canonical_products` | Cross-supplier identity/pack reference | brand, variant, GTIN/barcode if present, pack_size, base_unit, category, mapping confidence |
| `suppliers` | Wholesaler/distributor profile and capabilities | name, service area, order channel, active state, contact controlled by role |
| `supplier_products` | Maps a supplier SKU to a canonical product | supplier_id, supplier_sku, canonical_product_id, supplier_pack_size, units_per_pack, mapping_status |
| `supplier_offers` | Time-bound price/availability offer | supplier_product_id, price, currency, tax treatment, MOQ, order_multiple, stock state, updated_at, valid_until |
| `trade_schemes` | Conditional promo/discount/bonus | supplier_offer_id, scheme type, threshold, dates, eligibility, terms |
| `supplier_terms` | Delivery, freight, payment, return and service conditions | supplier_id, minimum_order_value, delivery window, freight rule, payment terms, returns policy |
| `purchase_orders` | Retailer-controlled order lifecycle | store_id, supplier_id, status, request_id/idempotency_key, submitted_at, supplier_reference, totals snapshot |
| `purchase_order_lines` | Ordered pack/qty and submitted price snapshot | purchase_order_id, supplier_product_id, quantity_packs, unit price snapshot, discount snapshot, tax snapshot, line total |
| `purchase_order_events` | Append-only state and discrepancy history | purchase_order_id, event_type, occurred_at, actor, payload |
| `goods_receipts` and `goods_receipt_lines` | What actually arrived | purchase_order_id, received_qty, damaged/short qty, actual invoice price, expiry/batch if relevant |
| `supplier_performance_snapshots` | Derived outcomes from observed transactions | fill rate, on-time rate, discrepancy rate, observation window, sample size |

Entity names are provisional. Use migrations once the first pilot category and identity model are agreed.

## 3. Data separation

**Product identity vs supplier listing vs retailer stock**

- Product identity answers “what is this?” and includes the exact pack.
- Supplier listing answers “what does this supplier offer, and under what terms right now?”
- Store inventory answers “what does this particular retailer currently hold and sell?”
- Purchase-order snapshots answer “what price, units, conditions and quantities were agreed at submission?”
- Goods receipt answers “what actually arrived and was billed?”

These must not collapse into one row. Historical orders must remain auditable after a supplier changes its price list.

## 4. Effective cost calculation

Use `NUMERIC/DECIMAL` for monetary values. Do not use floating-point numbers for money.

For a chosen offer, show:
- Supplier's stated unit/pack price and currency.
- Exact pack-to-base-unit conversion.
- Trade-scheme benefit and eligibility conditions.
- Freight/handling and order-level costs.
- GST/tax amount and whether the comparison is before or after tax; validate the chosen treatment with an accounting expert.
- Net comparable price per base unit and total purchase commitment.
- Any unallocated costs or missing fields.

Do not claim that all schemes are earned until their thresholds/eligibility are met. If costs cannot be compared safely, show “comparison incomplete” rather than a false winner.

## 5. Freshness contracts

Every price/availability record needs:
- source channel (supplier portal, file import, manual confirmation, API, etc.);
- source record/document identifier;
- observed_at and valid_until;
- confirmation state (unverified, supplier-published, retailer-confirmed, expired);
- import batch and correction history.

Initial proposed behavior:
- Expired offers cannot be labeled “best available.”
- A price list with no stock signal can inform price comparison, but availability must say unknown.
- If the supplier does not provide validity dates, use an explicit configured TTL agreed in pilot and show the timestamp.
- Never translate “no response” into “in stock.”

## 6. Order state machine

`draft → awaiting_retailer_confirmation → submitted → supplier_accepted | supplier_rejected`

After acceptance, delivery can progress to:
`partially_fulfilled → delivered`, with separate events for substitutions, cancellations, returns and invoice discrepancies.

Not every supplier can return machine-readable statuses. In that case, the system should show “awaiting supplier confirmation” until an operator/retailer records the response. It must never show “ordered” as equivalent to “accepted.”

## 7. Architecture

1. **Mobile/web retailer interface:** draft order, evidence, totals, confirmation and state.
2. **Supplier ingestion:** CSV/Excel/manual import first; future API/ONDC/partner connectors behind one adapter contract.
3. **Canonical product mapper:** deterministic identifier/pack matching with human review for ambiguity.
4. **Rules engine:** reorder maths, MOQ, price normalisation, scheme conditions and ranking.
5. **Postgres data layer:** tenant-scoped retailer data, supplier-access-controlled listings, immutable commercial snapshots and append-only events.
6. **AI parsing adapter:** extracts structured values from bills/messages, retaining raw input, confidence and provenance; all money and order decisions validated by deterministic code.
7. **Audit/observability:** import errors, stale data, unmatched products, order retries, duplicates and discrepancies.

## 8. Security model

- Retailer users can access only their store inventory, orders, receipts and negotiated/private price context.
- Supplier users can access only their own catalogues and their relevant orders.
- Katastima operators require explicitly scoped support roles with audit logging.
- Public/partner catalogue visibility must be separate from private negotiated offers.
- Use RLS policies and test with at least two retailer accounts and two supplier accounts.
- Use idempotency keys for import batches and order submission.
- Secrets/API keys belong server-side, never in the app bundle or repository.

## 9. Required tests before connecting a real supplier

- Same SKU at two suppliers with different pack sizes is not incorrectly compared.
- Scheme eligibility condition is not silently applied when threshold is not met.
- Stale price and unknown availability are displayed honestly.
- Retry with the same idempotency key does not create a duplicate order.
- Supplier accepting less than requested results in partial-fill status, not a false full fulfilment.
- Final invoice price differs from quote: discrepancy remains visible.
- Retailer A cannot read retailer B's order or private offer.
- LLM extraction mistakes in quantity, pack or price are caught/confirmed before submission.
- Offline/retry does not lose a draft or silently repeat a submission.
