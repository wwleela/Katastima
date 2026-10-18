# Katastima — Wholesale Intelligence for Independent Retail

**Thesis:** Help independent retailers make better replenishment decisions—what to buy, from whom, how much, at what effective cost, and when—without adding another complicated dashboard.

Katastima is intended as an AI-enabled operating layer for India's independent retailers and kiranas. Its first wedge is not a general wholesale marketplace. It is a **procurement decision and workflow layer** that joins store inventory, sales velocity, supplier offers, trade schemes and purchase outcomes.

## Why this problem

Retailers often coordinate procurement across supplier visits, phone calls, WhatsApp, separate apps and handwritten records. This creates fragmented price/scheme visibility, manual ordering, stock availability uncertainty and working-capital lock-up. ONDC's DigiDukaan initiative describes these as current general-trade digitisation challenges, and reports early Hyderabad traction: 10,000+ kiranas and 35+ brands onboarded through Qwipo as of the 12 June 2026 government release. Sources are recorded in [market research](docs/market-research-and-problem.md).

The market signal validates that procurement digitisation is active; it does **not** by itself prove that Katastima has product-market fit. We must validate a specific workflow with retailers and distributors.

## Product direction

1. **Know what needs replenishing.** Begin with clear rules using stock on hand, recent sales, supplier lead time, minimum order quantities and outstanding purchase orders.
2. **Compare actual supplier offers.** Compare like-for-like product packs, confirmed price, scheme eligibility, freight, tax treatment, availability freshness and delivery terms.
3. **Make ordering easy.** Build a draft order, get the retailer's confirmation, then submit through an agreed partner workflow (initially CSV/WhatsApp/manual confirmation if APIs are unavailable).
4. **Close the loop.** Record what was ordered, fulfilled, substituted, returned and actually paid, then improve future recommendations from observed outcomes.

## Operating principles

- Invisible AI: one useful next action, not a dashboard the owner must study.
- Deterministic money math; LLMs may help parse bills/messages, never invent prices or silently place orders.
- Supplier price and stock must be sourced, timestamped and freshness-labelled.
- No purchase is sent without retailer confirmation.
- No credit terms, availability, savings or fulfilment performance are fabricated.
- Build for low-end devices, intermittent connectivity and familiar workflows.
- Tenant isolation and row-level security are release blockers, not later polish.

## Current status

This repository is a **product and architecture baseline**, not a working wholesale marketplace. No supplier network, live price feed or purchase-order integration is claimed.

The prior [BharatShop architecture](https://github.com/wwleela/bharatshop-architecture) is a reference prototype. Its checked SQL schema defines `stores`, `products`, `sales` and `scan_logs`; it does not yet provide a normalized supplier catalogue, time-bound wholesale offers, schemes or purchase orders. Katastima will design these explicitly instead of assuming the existing schema already supports them.

## Documents

- [Market evidence and problem framing](docs/market-research-and-problem.md)
- [Wholesale procurement PRD](docs/wholesale-procurement-prd.md)
- [Data model and architecture](docs/data-model-and-architecture.md)
- [Roadmap, metrics and release gates](docs/roadmap-and-metrics.md)

## MVP success criteria

Measure outcomes, not AI usage: time to prepare an order, stockout rate, order fill rate, rate of price/availability mismatches, effective purchase cost versus comparable quotes, retailer acceptance of reorder recommendations, and inventory/cash tied up. Baselines must be measured during a pilot before targets are represented as results.
