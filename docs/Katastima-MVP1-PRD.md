# KATASTIMA — MVP-1 PRD & Measurable AI Evaluation Specification
### (formerly BharatShop) — Senior/Principal AI PM Deliverable

**Status legend used throughout:** KNOWN · [Assumption] · [Open Question] · [Proposed — requires validation]

---

## 1. Executive Summary

Katastima turns messy, low-friction retail data (photographed invoices, POS sales records) into structured margin and inventory intelligence for India's independent retailers, starting with kirana and mid-sized marts. MVP-1 tests one narrow question: **can Katastima turn a retailer's existing (messy) data into a reorder or margin decision they actually act on** — not delivery speed, not marketplace, not private label. Those are long-term thesis, explicitly out of MVP-1 scope (Section 8).

## 2. Problem Statement

Independent retailers operate with margin and inventory blindness: they generally do not have a structured view of which SKUs are profitable, which are slow-moving, or when to reorder. **[Assumption, partially supported]** — a Bain-sourced figure shows over 30% of Tier 1/2 kirana had *some* digital payment/ordering by 2024 (KNOWN, sourced in prior project phase), but this measures payment/ordering adoption, not inventory-intelligence adoption specifically — the latter remains unmeasured.

Qualitative signal exists that even computerized mid-size stores lack reorder/forecasting tooling. **[Open Question — unresolved]**: exact count, dates, and content of the interviews this claim is based on have not yet been logged. This must be resolved before this claim can move from anecdote to evidence (see Post-MVP action list).

**[Proposed — requires validation]** An additional or alternative path to close this gap: rather than relying only on formal interviews, run a small pilot with the subset of existing Katastima/BharatShop OS users who already have transaction history, and test whether their real sales data can predict reorder timing accurately enough to prevent stockouts during known seasonal/festival demand spikes. This would supplement, not replace, the interview log — it produces usage-based evidence instead of stated-preference evidence, and it connects directly to the pooled cross-store seasonal forecasting already scoped as Post-MVP (Section 28) rather than requiring new infrastructure. It does not resolve the [Open Question] on its own — a pilot still needs a defined sample size, a real seasonal window to test against, and a specific "stockout prevented" or "reorder timed correctly" success criterion before its results count as evidence rather than another anecdote.

## 3. Target Users / ICP

**Primary:** Independent kirana or small/mid-size mart owner-operator in Tier 1/2 Indian cities. **[Assumption]** — single-location, owner-operated; multi-location chains not validated as in-scope.

**Secondary (not MVP-1 target):** Artisans/producers, end customers — both explicitly deferred (see Section 8).

## 4. Jobs-to-be-Done

"When I look at my stock, I want to know what to reorder and what's actually profitable, so I don't run out of fast-movers or tie up cash in slow ones I can't see clearly."

## 5. User Pain Points

| Pain point | Evidence status |
|---|---|
| Manual/slow inventory entry from handwritten/printed bills | KNOWN — existing GeminiService.ts OCR pipeline built to address this |
| No reorder/forecasting system, even with a computer | [Open Question] — real but unlogged interview evidence |
| Margin blindness across branded vs. unbranded stock | [Assumption, secondary-evidence supported] — industry data shows 15-30% margin on loose/unbranded vs 2-8% branded; not yet confirmed at individual-retailer level |

## 6. Product Hypothesis

*"If Katastima converts a retailer's existing invoice/sales data into a clear reorder or margin action, retailers will continue using it beyond a single session."* **[Hypothesis — not established]**. MVP-1 exists to test this, not to assume it.

## 7. MVP-1 Scope

1. Invoice/bill ingestion (photo → structured data) — KNOWN, existing capability (`GeminiService.ts`, live model `gemini-2.5-flash`)
2. Product/SKU normalization — [Open Question]: fuzzy-match against existing catalog, or exact-match only for MVP-1? Recommend exact/near-exact match only for MVP-1; defer fuzzy SKU matching (Section 14).
3. Inventory visibility — KNOWN, `products.current_stock` exists in schema
4. Margin calculation — KNOWN, SQL written (`BharatShop-Phase0-Phase1.sql`), **not yet run against real data**
5. Sales velocity — KNOWN, computed in the same SQL file (14-day trailing window)
6. Reorder-point logic — KNOWN, SQL view written, **not yet run against real data**, explicitly flags `insufficient_data` rather than presenting a false-confidence number
7. Inventory-aging/dead-stock identification — NOT built. Proposed for MVP-1 as a simple deterministic rule (e.g., zero sales in 30 days). **[Proposed — requires validation of the 30-day threshold]**
8. Simple actionable alerts — NOT built. Depends on 4-7 above being live first.

## 8. Explicit Non-Goals (MVP-1)

Out of scope, regardless of enthusiasm elsewhere in the project history: AI invoice extraction *beyond* the existing tier-1/tier-2 cascade (no new model work), demand forecasting beyond classical reorder math, conversational retail assistant, space optimization, product recommendations, hyperlocal demand graph, supplier marketplace, "Katastima Standard" certification, private-label/Made-in-India goods line, kirana-to-kirana fulfillment network. All of these were explored in prior project phases (BharatShop v2/v3 PRDs) and are preserved as Post-MVP roadmap (Section 28), not discarded — just gated behind MVP-1 evidence.

## 9. User Journeys

**Journey A — Invoice to inventory:** Owner photographs supplier bill → Tier 1 model extracts line items → if confidence low, escalates to Tier 2 (`CascadeService.ts`, built, **evaluation not yet run**) → owner reviews/confirms extracted items (Section 19) → inventory updates.

**Journey B — Margin visibility:** Owner records a sale via POS → `sales.items` snapshot captures sell_price and cost_price at time of sale (KNOWN, confirmed in schema) → margin query (Section 13) aggregates by category/brand type on demand or on a schedule.

**Journey C — Reorder decision:** System evaluates `product_reorder_status` view nightly or on-demand → owner sees a simple flag ("Reorder X soon") rather than raw statistics (Invisible AI principle, Section 4 of source doc).

## 10. Functional Requirements

- Photograph → structured invoice data, with visible confidence indicator per line item (schema already supports a confidence field — KNOWN)
- Manual correction UI for extracted invoice data (NOT yet built — required for Section 19)
- Margin report, filterable by category and branded/unbranded (SQL exists, UI does not)
- Reorder status view, filterable by "needs reorder now" (SQL exists, UI does not)
- Dead-stock/aging flag (not built, Section 7 item 7)

## 11. Data Model Requirements

KNOWN (confirmed in `schema.sql`): `products` table with `cost_price`, `sell_price`, `current_stock`, `category`. `sales` table with JSONB `items` snapshot preserving price-at-sale.

**[Open Question — must be checked directly against live schema before Phase 1 SQL runs]:**
- Does `products.is_branded` exist?
- Does `products.lead_time_days` exist?
- Does `products.safety_stock_units` exist?

If any are missing, that's a small migration, not a redesign — but do not run the margin/reorder SQL and interpret its output until this is confirmed, since the SQL currently defaults these values, and defaults are not measurements.

## 12. System Architecture

Reuses existing infrastructure rather than introducing new systems for MVP-1: Supabase (Postgres + PostGIS, RLS via `auth.uid()` — KNOWN, confirmed), Expo/React Native client, Gemini API called directly at `gemini-2.5-flash` (KNOWN, confirmed live in `bharatshop-architecture` repo — note the separate `bharatshop-cloud-run` repo uses `gemini-1.5-flash` and is not what's deployed; do not conflate the two in any architecture diagram). Full layer-by-layer stack detail, including gaps not previously specified, is in Section 33.

## 13. Deterministic Baseline (build this before any new AI)

Per source doc's "deterministic before ML/GenAI" principle:
- Margin analytics query — written, **STATUS: not yet run against real data**
- Reorder-point view — written, classical formula (ROP = avg daily sales × lead time + safety stock), explicitly flags low-confidence rows as `insufficient_data`, **STATUS: not yet run against real data**
- Dead-stock rule (Section 7 item 7) — not yet written

No AI/ML is required for any of the above. This is intentional and should stay that way for MVP-1.

## 14. AI Components and Justification

| Component | Justification | Status |
|---|---|---|
| Bill/invoice OCR extraction | Directly replaces the highest-friction manual task (bill-by-bill inventory entry); no deterministic alternative exists for reading handwritten/printed bills | KNOWN, built |
| Model cascading (Tier 1 → Tier 2 escalation) | Cost control on the above, only escalates on defined failure conditions | Built, **not yet evaluated** |
| SKU fuzzy-matching via AI | **[Open Question]** — not yet justified. Exact-match may suffice for MVP-1 catalog sizes; introduce fuzzy AI matching only if exact-match failure rate is measured and shown to matter. | Not built, not justified yet |
| Demand forecasting (ML) | Not justified for MVP-1 — insufficient per-store data volume (see Katastima PRD's own inherited BharatShop v3 analysis on why enterprise-style forecasting doesn't transfer to single-store data). Classical ROP math substitutes. | Deferred, Post-MVP |
| Conversational assistant | No measurable reason identified for MVP-1 | Deferred, Post-MVP |

## 15. AI Evaluation Methodology

For bill/invoice OCR (Tier 1 + Tier 2 cascade):
- **Field-level accuracy** — correct value extracted per field (item name, quantity, unit price, cost price, total)
- **Line-item extraction accuracy** — correct number of line items identified (neither missed nor hallucinated)
- **Quantity accuracy** — exact match against hand-verified ground truth
- **Unit-price / cost-price accuracy** — exact match, tolerance **[Proposed — requires validation]**, e.g. within ₹0.50 or 1%, to be decided based on what a human reviewer would consider "close enough to not need correction"
- **Total-value accuracy** — exact match against hand-verified ground truth

No target values are stated as fixed thresholds here. Any number quoted as a pass/fail bar must be labeled **[Proposed threshold — requires validation]** and selected only after running the harness on a real ground-truth set (Section 16) and reviewing the actual error distribution — not chosen in advance to sound rigorous.

## 16. Ground Truth Strategy

Minimum ground-truth set for MVP-1 evaluation: **[Proposed — requires validation]** 20-30 real bills, spanning printed/handwritten, Telugu/Hindi/English, clean/damaged condition — matching the design already specified in `eval-cascade.ts` and the Discovery Guide. **STATUS: not yet collected.** This is the single largest evidence gap standing between "designed" and "evaluated" for the AI components of Katastima.

Dataset should also capture variation across: thermal-paper quality (common for POS-printed receipts, prone to fading/low contrast), regional distributor invoice formats (layout varies by wholesaler, not just by language), and abbreviation/spelling variation in product names across suppliers — three real failure modes worth designing for from the first collection pass rather than discovering after the harness is already built.

## 17. Metrics

**The single question to answer before any vanity metric:** does Katastima produce a decision a retailer considers useful enough to repeat? Everything below is in service of answering that, not a checklist to fill independently of it.

**Product metrics (adoption/retention signal):** % of owners who complete a first invoice scan; % who return to check margin/reorder view within 7 days; % of reorder alerts acted on (requires instrumentation, not yet built).

**Technical metrics:** extraction field accuracy, Tier 2 escalation rate, cost per invoice, end-to-end latency. All currently unmeasured — `eval-cascade.ts` exists to produce these but has not been run.

## 18. Error Taxonomy

- **Extraction errors:** missed line item, hallucinated line item, wrong quantity, wrong unit price, wrong unit-conversion (e.g. "1 bora" misread), language misidentification
- **SKU-matching errors** (if introduced Post-MVP): false match to wrong product, no match found requiring manual creation
- **System errors:** upload failure, API timeout, malformed JSON response, unsupported invoice format
- **Duplicate line item:** same product extracted more than once from a single invoice, inflating quantity/cost

## 19. Human-in-the-Loop Workflow

**Required for MVP-1**, given no accuracy threshold has been validated yet: owner reviews and can correct extracted invoice data before it commits to inventory. Each correction is a ground-truth data point — this doubles as informal, ongoing evaluation data collection alongside the formal 20-30 bill harness. **Not yet built.**

## 20. Fallback UX

Manual entry must remain available at all times — OCR failure or low-confidence extraction should never block inventory update, only flag for review. The existing `confidence` field (KNOWN, present in schema) should drive this flagging directly.

## 21. Latency Requirements

The original BharatShop report claimed "~8.5 seconds" for bill-scan processing. **[Open Question — inherited, unverified]**: this number was never independently confirmed anywhere in this project's evidence trail (no log, no benchmark). Do not carry it forward as fact. A real latency target should be **[Proposed — requires validation]** after the eval harness runs and actual Tier 1/Tier 2 timings (already captured in `CascadeService.ts`'s `tier1_latency_ms`/`tier2_latency_ms` fields) are reviewed.

## 22. Inference-Cost Measurement

`eval-cascade.ts` is built to compute this, gated on real Gemini pricing pulled from `ai.google.dev/gemini-api/docs/pricing` at run time — third-party pricing trackers disagreed with each other when checked earlier in this project, so no fixed number should be hardcoded here. **STATUS: not yet run.**

## 23. Privacy / Security Considerations

KNOWN: Supabase RLS policies gate data access via `auth.uid()`. Bill images may contain supplier identity/business terms — treat as sensitive by default. Any margin data shared externally (investor materials, case studies) must be aggregated by category/region, never shown per-individual-store, per the standard already set earlier in this project.

## 24. Observability / Telemetry

Not yet implemented. Mixpanel or PostHog proposed in prior project phase, not wired to the live app. Required before Section 17's product metrics can be measured for real.

## 25. Acceptance Criteria

Reuses Engineering SOP v3 (Section 3) directly — margin query hand-verified against 5 real transactions, reorder view hand-verified against 5 SKUs with zero calculation mismatch, eval harness run on 20+ real bills with logged, non-placeholder output.

## 26. Experiment Plan

Pilot with **[Open Question — count not yet determined]** real stores, ideally overlapping with existing BharatShop OS users where a relationship already exists. Collect real invoices for the ground-truth set (Section 16) during this same pilot rather than as a separate exercise.

## 27. MVP Launch Gates

No component may be described as "shipped" or "live" unless it clears Engineering SOP Section 2 (Definition of Done) in full — code understood and explainable, acceptance criterion checked against real data, AI evaluation run and logged if applicable, PRD status updated with a date.

Gates by category, all required together — clearing one does not substitute for another:

| Gate | Requirement |
|---|---|
| **Technical** | Core workflows (ingest → normalize → margin → reorder) execute reliably against real, not seed, data |
| **Data** | Input/output transformations are auditable — a reviewer can trace any margin or reorder number back to the source transaction |
| **AI** | Evaluation results (Section 15/25) exist for the defined ground-truth set — not skipped because the deterministic baseline works |
| **Product** | At least one real retailer has actually used the relevant workflow — not a demo, not seed data |
| **Value** | Observable evidence the output is useful to that retailer — a repeated action, a stated preference, or a decision made differently because of it — not just "the feature ran" |

## 28. Post-MVP Roadmap

Dead-stock/aging alerts (beyond the simple MVP-1 rule), pooled cross-store demand signal (requires enough stores live to pool — genuine network effect, not available at MVP-1 scale), private-label validation research and pilot (BharatShop v3 Phase 2 — accessories/handcrafted art category), hyperlocal discovery screen (PostGIS function already exists at DB level, unwired), conversational assistant, kirana-to-kirana fulfillment network (fully parked, no evidence base yet).

## 29. Open Questions (consolidated)

1. Real count/content of shopkeeper interviews supporting the reorder-pain claim — **still outstanding from prior project phase**
2. Does `products` table have `is_branded`, `lead_time_days`, `safety_stock_units`? — **needs a direct schema check**
3. What tolerance defines "accurate enough" for price fields in OCR evaluation? (Section 15)
4. Is "Katastima" as a name validated with the actual ICP, or primarily investor-facing? (flagged in the preamble to this document)
5. How many pilot stores, and over what real timeframe, for the ground-truth collection (Section 26)?
6. Is SKU fuzzy-matching actually needed for MVP-1, or does exact-match suffice at current catalog sizes?

## 30. Technical Risks

Sparse per-SKU sales data may make velocity/reorder-point calculations low-confidence for long-tail products (already handled defensively — the view labels these `insufficient_data` rather than presenting a false number, but this may mean the majority of SKUs are unable to get a real reorder recommendation until more transaction history accumulates). Gemini model/pricing drift (already observed once — `1.5-flash` vs `2.5-flash` mismatch between two repos). Schema gaps (Section 29, item 2) could block the SQL from running as written.

## 31. Product Risks

Retailers may not act on alerts even if accurate (adoption risk, unmeasured). The margin difference between branded and unbranded stock may not be large enough at the individual-retailer level to change stocking behavior, even though industry-level secondary data suggests it should be (Section 5). Human-correction burden (Section 19) could be high enough to erase the time-savings value proposition if OCR accuracy turns out to be poor — this is exactly why Section 16's ground-truth evaluation needs to happen before claiming the feature works.

## 32. Strategic Principle — the sequence Katastima actually needs to prove

Katastima should not try to prove "AI can transform kiranas" as one big claim — that's unfalsifiable and it's exactly the kind of thing that leads back to invented metrics. Instead, prove a sequence of small, independently checkable claims, in order, and don't skip ahead:

1. Retail data can be captured (Section 7.1 — ingestion works on real bills)
2. Retail data can be normalized (Section 7.2 — SKU/product mapping holds up)
3. Retail economics can be made visible (Section 7.3 — margin numbers are correct and trusted)
4. Inventory decisions can be improved (Section 7.5 — reorder alerts are accurate and acted on)
5. Retailers repeatedly use the system (Section 17 — the one real product-market-fit signal)
6. ML/AI improves specific decisions beyond the deterministic baseline (Section 28 — only pursued once step 5 is real)
7. The resulting infrastructure enables differentiated retail (Section 28 — private label, marketplace, global expansion — everything currently parked)

Each step is the evidence gate for the next. Claiming step 4, 5, or 7 without steps 1-3 actually being true — which is close to what happened earlier in this project with the compression figure, the cascading-cost claim, and the "hyperlocal discovery is live" framing — is the specific failure this sequence exists to prevent. If you're ever unsure what to build or claim next, the answer is: whichever numbered step isn't checked off yet.

---

## 33. Technology Stack — Detailed, Free-Tier-First

**Convention:** KNOWN = already built and verified in the repo audits earlier in this project. [Proposed] = not previously specified, recommended now. [Open Question] = genuine fork, needs your decision before Codex builds against it.

| Layer | Recommendation | Status | Free-tier reality check |
|---|---|---|---|
| Mobile app framework | Expo (React Native) | KNOWN, built | Free, no usage cap |
| Backend / database | Supabase (Postgres 15 + PostGIS 3.4) | KNOWN, built | Free tier ~500MB DB. Auto-pauses after ~7 days of inactivity — **mitigation, not migration:** add a free GitHub Actions scheduled job pinging the project every 3-4 days to keep it warm. Self-hosting was evaluated as an alternative (see note below) and explicitly not recommended for MVP-1. |
| Auth | Supabase Auth (JWT + RLS) | KNOWN, built | Free, included |
| File/image storage (bill photos) | Supabase Storage | [Proposed] — not previously confirmed as the storage layer | Free tier ~1GB. The existing 95%+ compression pipeline directly extends this tier's runway — worth stating that connection explicitly in any pitch. |
| AI / invoice OCR | Gemini API (`gemini-2.5-flash` → `gemini-2.5-pro` cascade) | KNOWN, built | Free tier has per-minute/per-day rate limits. Fine for eval-harness runs and early dev; **may throttle a real multi-store pilot** — check current limits at ai.google.dev before committing to a pilot store count, don't assume. |
| Edge/serverless functions | Supabase Edge Functions (Deno) | KNOWN, mentioned in original report | Free tier included |
| Payments | Direct UPI deep-link | KNOWN, scoped | Free, no PSP fees. **[Confirmed]** Payment reconciliation (automated matching against bank/PSP settlement files) is explicitly Post-MVP — MVP-1 covers core payment processing and basic transaction status tracking only. Reconciliation introduces heavy operational/edge-case complexity that would slow initial validation of the core product flows. |

**Infrastructure philosophy [Confirmed]:** managed free tiers wherever generous, self-hosted lightweight infrastructure only where a service's free tier genuinely isn't. Applied concretely: Supabase's free tier is generous enough to stay managed for MVP-1 despite the auto-pause quirk (mitigated above, not migrated away from) — self-hosting it on Oracle Cloud's "Always Free" tier was evaluated and set aside for now, because that tier's own limits have been reported as shrinking (4 OCPU/24GB → 2 OCPU/12GB per multiple June 2026 sources, though other recent sources still list the older spec) and it carries its own idle-instance reclamation risk — trading one dormancy problem for a less predictable one, plus real ops overhead. Revisit only if the Supabase pause mitigation proves insufficient in practice.
| Product analytics/telemetry | PostHog | [Proposed] — replaces the earlier undecided "Mixpanel/PostHog" placeholder | PostHog's free tier is more generous for an early-stage build than Mixpanel's, and it's self-hostable later if you outgrow it. Recommend committing to this rather than leaving it open. |
| Error tracking / crash reporting | Sentry | [Proposed] — **not previously in any version of this PRD** | Free tier covers early-stage volume. Catches production failures the AI eval harness won't (crashes, network failures, RLS permission errors). |
| CI/CD | GitHub Actions | [Proposed] — **not previously in any version of this PRD** | Free minutes tier is sufficient at this scale. Use it to run `eval-cascade.ts` automatically on push, not just when someone remembers to run it locally. |
| Version control | GitHub | KNOWN implicitly (existing repos live there) | Free for private repos |
| Maps/tiles for the discovery screen | **[Confirmed] Mapbox free tier** (~50K map loads/month), via `react-native-maps` | Confirmed | Use Mapbox specifically, not the public OpenStreetMap tile server directly — OSM's public tiles carry a usage policy that prohibits heavy production/commercial use, which a real pilot would likely violate. Mapbox's free tier is generous enough for MVP-1 and requires no billing account. Migrate to Google Maps later only if a specific feature (e.g. advanced geocoding) requires it. |
| Push notifications (reorder alerts) | Expo push notifications | [Proposed] | Free, built into Expo already — no separate service needed |
| Case-study/landing page hosting | GitHub Pages or Vercel free tier | [Proposed] | For the eventual public case study (workflow log Stage 6) |
| Secrets management | Supabase Vault + untracked `.env` | [Proposed] | Free. **Add to Definition of Done:** verify no API keys were ever committed to the public repo — worth a direct check given this repo has already been audited multiple times in this project. |

### Where this PRD was missing things before you asked

Direct answer to "where did I miss things" — these are real gaps, not stack choices:

1. **No error tracking anywhere in any prior version** — the AI eval harness catches extraction errors, but nothing catches app crashes, network failures, or RLS permission errors in production. Sentry closes this.
2. **No CI/CD** — `eval-cascade.ts` currently only runs if someone remembers to run it. Without automation, "evaluated" quietly drifts back into "was evaluated once" the same way the compression figure and cascading claim did.
3. **Free-tier operational risks were never flagged** — Supabase's auto-pause and Gemini's rate limits are the kind of thing that breaks a pilot silently, not loudly. Now documented above, with the auto-pause specifically mitigated via a scheduled keep-alive rather than left as an open risk.
4. **No cost-monitoring/alerting** — nothing in this PRD tracks spend before it happens. Even a manual weekly check of Gemini/Supabase usage against free-tier limits is better than finding out via a surprise bill.
5. **No backup/disaster-recovery plan for the database** — Supabase's free tier does not include point-in-time recovery. If this pilot's data becomes the investor-facing evidence base (Section 26), losing it has real cost. Worth a manual export cadence at minimum until this graduates off free tier.
6. **No testing framework for the app code itself**, separate from the AI eval harness — no mention of Jest/unit tests or Detox/integration tests for the Expo app. The eval harness tests the AI; nothing currently tests the app logic around it.
7. **No privacy policy / terms of service** — the app handles merchant financial data (cost/sell prices) and photographed bills. RLS is a technical control; a privacy policy is a legal/trust one, and neither this PRD nor any prior version mentions it. Needed before any real pilot store's data is collected, not after.
8. **Map/tile provider was never actually decided** — Section 11's discovery screen references the PostGIS query but not what renders the map itself. **Now resolved: Mapbox free tier**, confirmed above.

---

## MVP-1 Definition of Done (handoff-ready checklist)

- [ ] Schema confirmed: `is_branded`, `lead_time_days`, `safety_stock_units` exist or migrated in
- [ ] Margin query run against real transaction data, output hand-verified on 5 rows, logged with date
- [ ] Reorder-point view run against real data, hand-verified on 5 SKUs, zero calculation mismatch, logged with date
- [ ] 20-30 real bills collected across print/handwritten/language variants; ground truth hand-transcribed
- [ ] `eval-cascade.ts` run on the above set; `results.csv` produced with real (non-null) cost and accuracy figures
- [ ] Manual correction UI built for invoice review (Section 19)
- [ ] Dead-stock/aging rule defined with a specific, justified threshold (not an arbitrary round number)
- [ ] Basic telemetry wired for the three product metrics in Section 17
- [ ] Real interview evidence (Open Question #1) logged with count/dates/notes, or explicitly retired as a claim if it can't be substantiated

## AI Evaluation Scorecard

| Metric | Dataset | Measurement method | Threshold status | Launch gate? |
|---|---|---|---|---|
| Field-level extraction accuracy | 20-30 hand-verified real bills | Exact-match comparison, `eval-cascade.ts` | [Proposed — requires validation after first run] | Yes |
| Line-item extraction accuracy | Same | Count comparison (missed/hallucinated) | [Proposed] | Yes |
| Tier 2 escalation rate | Same | `tier_used` field in cascade output | Informational, not gating | No |
| Cost per invoice | Same | Token estimate × real Gemini pricing (pulled live) | [Proposed, pending real pricing check] | Informational |
| End-to-end latency | Same | `tier1_latency_ms` + `tier2_latency_ms` | [Proposed] | No, unless found unacceptable in practice |
| Human correction rate | Pilot usage, post-launch | % of scans requiring manual edit | [Proposed] | Yes — high rate blocks claiming the feature "works" |

## 30-Day Execution Plan

Only items with a clear evidence-based next step are included — no invented external dependencies or dates:

**Week 1:** Confirm schema columns (Section 29 #2). Run margin query and reorder view against whatever real data currently exists, even if limited. Log real interview data or retire the claim.

**Week 2:** Begin collecting the 20-30 bill ground-truth set from real pilot stores. Build the manual-correction UI for invoice review.

**Week 3:** Run `eval-cascade.ts` against the completed ground-truth set. Log results. Select real thresholds based on the actual error distribution observed, not before.

**Week 4:** Wire basic telemetry (Section 17). Write the launch-gate decision memo: does the evidence collected support calling MVP-1 "done," per the checklist above, or does the timeline extend to close remaining gaps?

**Explicitly not scheduled:** pilot store count and external timelines depend on real-world recruitment of retailers, which has no evidence-based estimate yet — this is an [Open Question], not a date to guess at.