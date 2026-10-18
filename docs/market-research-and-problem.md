# Katastima — Market Evidence and Problem Framing

**Research cut-off:** 18 October 2026  
**Status:** Secondary research and product hypothesis. Not primary validation.

## What the evidence supports

### 1. Indian general trade is a relevant, large distribution surface

A Ministry of Commerce & Industry release dated 12 June 2026 describes India's general-trade ecosystem as more than **1.4 crore kirana stores**, accounting for approximately **75–80% of FMCG sales**. The same release identifies fragmented ordering systems, limited inventory visibility and manual sales processes as inefficiencies for retailers, distributors and brands. It describes DigiDukaan as an ONDC initiative to digitise B2B procurement and improve offer visibility, fill rates and working-capital management.

Source: [PIB — DPIIT and ONDC convene FMCG leaders to digitise procurement for India's kirana stores (12 June 2026)](https://www.pib.gov.in/newsite/erelcontent.aspx?lang=2&reg=48&relid=289960)

### 2. Hyderabad is not an unserved blank slate

The same PIB release says DigiDukaan had early Hyderabad traction with more than **10,000 retailers and 35+ brands** onboarded through Qwipo. The ONDC programme page also describes 10K+ kiranas, 1K+ distributors, 35+ brands, unified ordering, improved offer visibility and smarter replenishment.

Sources:
- [ONDC — DigiDukaan for kirana B2B procurement](https://www.ondc.org/pages/digidukaan.html)
- [PIB release](https://www.pib.gov.in/newsite/erelcontent.aspx?lang=2&reg=48&relid=289960)

**Product implication:** Do not attempt to win solely by making another distributor listing app. Existing initiatives validate category activity and competition. Katastima needs a sharper wedge: connect the retailer's own inventory and sales to a trustworthy purchase recommendation, compare offers after pack-size normalisation and scheme/term visibility, and learn from actual fulfilment outcomes.

### 3. Wholesale price alone is an incomplete decision

A low unit price can be offset by an MOQ that ties up cash, freight, ineligible schemes, short expiry, late delivery or poor fill rate. Katastima should display verified landed economics and explicit constraints, not rank offers by headline price alone.

This is a product-design inference to validate with buyers and suppliers; the cited pages do not quantify the relative impact of each factor.

## Problem statement

A kirana owner needs to replenish fast-moving items without spending unnecessary time reconciling stock, sales, pack units, supplier quotes, promotions, delivery and payment terms. Today, procurement can be fragmented across calls, sales visits, WhatsApp, separate apps and paper records. The owner must decide what to order and from whom while protecting availability, cash flow and margin.

**Job to be done:** “Help me create the right order from suppliers I trust, at a comparable effective cost, without tying up more cash than I can afford or running out of stock.”

## Initial hypotheses—not established facts

| Hypothesis | Why it matters | How to test |
|---|---|---|
| Retailers lose time consolidating orders across suppliers and channels. | If true, a unified order draft has immediate utility. | Time 5–10 real replenishment tasks; compare with the retailer's current method. |
| Retailers cannot easily compare effective prices and schemes. | Headline prices are not always comparable. | Ask for actual price lists/bills and calculate the comparable cost for the same pack. |
| Stock signals can improve purchase timing. | Replenishment must balance stockout risk and cash lock-up. | Compare recommendations with actual sales/stock and retailer judgement. |
| Suppliers will share price, scheme and availability data in a repeatable format. | Without refreshed data, recommendations are unsafe. | Request sample CSV/Excel/WhatsApp offers from at least 3–5 distributors. |
| A draft-and-confirm workflow is adoptable without supplier API integration. | Reduces build time while preserving retailer control. | Pilot with manually confirmed orders before automating submission. |

## Discovery script

Interview both retailer and distributor sides. Ask about the most recent real order, not a hypothetical ideal process.

**Retailer**
1. Walk me through the last order you placed: how did you decide SKU, quantity and supplier?
2. How do you check stock, expected sales, schemes, MOQ and delivery?
3. What changes between the price on the list and the amount finally paid?
4. Which items cause the most frequent stockouts or excess stock?
5. What information would you trust enough to use for an order recommendation?
6. Would a system preparing an order for your confirmation fit your current process? What would block it?

**Distributor**
1. How are catalogues, price changes, schemes, stock and delivery cut-offs shared today?
2. How often can availability/price be refreshed reliably?
3. Which order formats can your team accept without re-entering everything?
4. How do you handle partial fills, substitutions, returns, expiries and payment terms?
5. What data can be exposed to a retailer, and what is contractually restricted?

## Research references

- [PIB, Ministry of Commerce & Industry, 12 June 2026](https://www.pib.gov.in/newsite/erelcontent.aspx?lang=2&reg=48&relid=289960)
- [ONDC — DigiDukaan](https://www.ondc.org/pages/digidukaan.html)
- [ONDC — publications and the ONDC–McKinsey digital commerce report](https://www.ondc.org/pages/resources-publications.html)

## What we must not claim yet

- That Katastima has retailer/distributor partnerships.
- That it has live price or availability feeds.
- That users will save a particular percentage or time.
- That the AI predicts demand accurately.
- That wholesale is “booming” in a way that proves this specific product will win.

Those are tests, not inputs to a pitch deck.
