# Product Roadmap - Thrift_Consignment

**Period:** 2025 (9-12 month horizon)
**Owner:** Solo Founder
**Last Updated:** 2025-11-09
**Status:** Draft

---

## Vision & Strategy

### Product Vision

Thrift_Consignment becomes the payments-driven backbone of consignment stores, automating the complex, error-prone work of consignor payouts while providing real-time transparency through a modern consignor portal. We win by integrating with existing POS systems rather than replacing them, making us sticky through money flow and consignor relationships.

Within 18 months, we aim to power payouts for 100-200 consignment stores, processing $10-20M in annual consignor payments while achieving $2-5M ARR through disciplined execution.

### Strategic Themes

1. **Payments-Driven Stickiness:** Control money flow to create lock-in
2. **Integration-First:** Work with existing POS (don't compete)
3. **Consignor Trust:** Real-time transparency builds merchant value
4. **Lean Acquisition:** Product-led growth through trials and word-of-mouth

---

## Roadmap Overview

### NOW - V1 Wedge Product (Months 1-2)

**Focus:** Core payout automation + consignor portal

**Strategic Goal:** Pre-sell 3 pilots, validate -70% time savings and -50% inquiry reduction

| Feature | Status | Target | Priority |
|---------|--------|--------|----------|
| CSV Sales Importer | Not Started | Week 2 | P0 |
| Rule Engine (splits, fees, markdowns, returns) | Not Started | Week 3-4 | P0 |
| Per-Consignor Ledger + Audit Trail | Not Started | Week 4-5 | P0 |
| PDF Statement Generation | Not Started | Week 5 | P0 |
| Stripe Connect Payout Integration | Not Started | Week 6-7 | P0 |
| W-9 Collection + 1099-K Setup | Not Started | Week 7 | P0 |
| Consignor Portal (auth, dashboard, statements) | Not Started | Week 8 | P0 |
| Multi-Tenant Store Setup | Not Started | Week 2 | P0 |

**V1 Success Criteria:**
- Import 90 days of sales for 1 pilot store
- Recompute payouts with <1% variance vs manual
- Generate statements identical to store's current format
- Process test payout via Stripe Connect
- 3 pilots signed at $99/mo + $0.25/payout

---

### NEXT - V2 Integration Expansion (Months 3-5)

**Focus:** POS webhooks, operational features, retention improvements

**Strategic Goal:** Reduce manual CSV uploads, expand to 10-20 paying customers

| Feature | Description | Strategic Theme | Target | Priority |
|---------|-------------|-----------------|--------|----------|
| Square for Retail Webhooks | Real-time sales sync (order.created) | Integration-First | Month 3 | P0 |
| Shopify POS Webhooks | Real-time sales sync (orders/create) | Integration-First | Month 4 | P1 |
| Lightspeed Webhooks | Real-time sales sync (Sale.creating) | Integration-First | Month 5 | P1 |
| Label Printing System | QR codes + barcodes for consignor tags | Operational Efficiency | Month 4 | P1 |
| Enhanced Portal Features | Email notifications, payout preferences | Consignor Trust | Month 3 | P1 |
| Multi-Store Dashboard | Single owner managing multiple locations | Scalability | Month 5 | P2 |
| Inventory Analytics | Best sellers, turnover rates, markdown analysis | Merchant Value | Month 5 | P2 |
| Webhook Retry Logic | Exponential backoff, deduplication with Redis | Reliability | Month 3 | P0 |

**V2 Success Criteria:**
- 80%+ of active stores using webhooks (not CSV)
- 60%+ consignor portal login rate within 30 days
- 95%+ payout accuracy (zero critical errors)
- <3% monthly churn

---

### LATER - V3 Cross-Platform Expansion (Months 6-9)

**Focus:** Cross-listing, mobile apps, advanced analytics

**Strategic Goal:** Expand value beyond payouts, reduce churn, enable upsells

| Feature | Description | Strategic Theme | Estimated | Priority |
|---------|-------------|-----------------|-----------|----------|
| Cross-Listing Engine | Auto-list unsold items to eBay, Poshmark, Mercari | Merchant Value | Month 6-7 | P1 |
| Bi-Directional Inventory Sync | Update inventory when items sell externally | Operational Efficiency | Month 7 | P1 |
| Mobile App (Consignor) | iOS/Android app for consignors to track inventory | Consignor Trust | Month 8-9 | P2 |
| Advanced Reporting | Custom reports, exportable data, trend analysis | Merchant Value | Month 6 | P2 |
| Automated Relisting | Re-list items after markdown periods | Merchant Value | Month 7 | P2 |
| White-Label Portal | Branded portal for $29/mo add-on | Revenue Expansion | Month 8 | P1 |
| Marketplace for Consignors | Connect stores with potential consignors | Network Effects | Month 9 | P3 |

**V3 Success Criteria:**
- 30%+ of stores using cross-listing (upsell to $228/mo)
- 40%+ consignors using mobile app
- NRR >100% (expansion revenue > churn)

---

### FUTURE/BACKLOG (9-12+ Months)

Ideas and initiatives being considered but not yet scheduled:

- **AI-Powered Pricing:** ML recommendations for optimal consignor splits and markdown schedules
- **Inventory Photography Automation:** Auto-crop, background removal, cross-listing optimization
- **Consignor Acquisition Tools:** Landing pages, digital contracts, onboarding flows for stores to recruit consignors
- **Payment Method Expansion:** ACH direct, PayPal, Venmo for consignor payouts
- **API Platform:** Public API for 3rd-party integrations and custom workflows
- **Franchise Mode:** Multi-location chains with centralized reporting
- **International Expansion:** Multi-currency, VAT handling, international payouts

---

## Detailed Feature Breakdown

### CSV Sales Importer

**Problem:** Stores use diverse POS systems; direct integrations take time
**Solution:** Excel template + mapping wizard to import sales from any POS export
**Impact:** Enables immediate onboarding while webhooks are being built
**Effort:** Small (1 week)
**Dependencies:** Multi-tenant database schema, file upload storage (Azure Blob)
**Status:** Not Started
**PRD:** `product/prds/csv-importer-prd.md` (to be created)

---

### Rule Engine

**Problem:** Every store has unique split percentages, fees, markdown schedules, and return policies
**Solution:** Flexible rule engine with templates + custom overrides
**Impact:** Core differentiation; handles store heterogeneity without custom code
**Effort:** Large (2-3 weeks)
**Dependencies:** Ledger database schema, decimal precision calculations
**Status:** Not Started
**PRD:** `product/prds/rule-engine-prd.md` (to be created)

**Rule Types:**
- **Split rules:** Consignor gets X%, store gets Y% (can vary by item category, age, price tier)
- **Fee rules:** Listing fees, processing fees, restocking fees
- **Markdown rules:** Time-based price reductions (e.g., -20% after 30 days, -40% after 60 days)
- **Return rules:** Full refund, partial refund, no refund, restock fee

---

### Per-Consignor Ledger

**Problem:** Manual tracking of hundreds of consignors' balances is error-prone and time-consuming
**Solution:** Double-entry accounting ledger with immutable audit trail
**Impact:** Eliminates payout errors, provides dispute resolution capability
**Effort:** Medium (1.5 weeks)
**Dependencies:** PostgreSQL database, EF Core migrations
**Status:** Not Started
**PRD:** `product/prds/ledger-prd.md` (to be created)

**Transaction Types:**
- **Credit:** Item sold (consignor owed money)
- **Debit:** Payout sent, fees charged, item returned
- **Adjustment:** Manual corrections (admin only, logged)

---

### Stripe Connect Integration

**Problem:** Manual payouts via check/cash are slow and create reconciliation overhead
**Solution:** Automated ACH/debit payouts via Stripe Connect
**Impact:** Core value prop; replaces hours of manual work
**Effort:** Medium (2 weeks including W-9/1099-K setup)
**Dependencies:** Stripe account, legal entity verification, tax compliance research
**Status:** Not Started
**PRD:** `product/prds/stripe-connect-prd.md` (to be created)

**Onboarding Flow:**
1. Store owner connects Stripe account (platform-level)
2. Consignors complete Stripe Express onboarding (W-9, bank details)
3. Platform automatically sends payouts on schedule
4. Stripe handles 1099-K reporting for $600+ annual

**Payout Triggers:**
- **Scheduled:** Weekly, bi-weekly, monthly
- **Threshold:** When balance reaches $X
- **Manual:** Store owner initiates on-demand

---

### Consignor Portal

**Problem:** Consignors constantly call/email stores asking "Did my item sell?" "When do I get paid?"
**Solution:** Self-service portal with real-time inventory and payout visibility
**Impact:** Reduces inquiry volume by 50%, improves consignor satisfaction
**Effort:** Medium (1.5 weeks)
**Dependencies:** Frontend framework (Next.js), authentication, ledger API
**Status:** Not Started
**PRD:** `product/prds/consignor-portal-prd.md` (to be created)

**Portal Features:**
- Dashboard: Current balance, next payout date, sold/unsold counts
- Inventory: All items, status (available, sold, returned), pricing, markdown schedule
- Sales History: Transaction list, filtering by date/item/status
- Statements: Download PDF for any date range
- Notifications: Email/SMS when item sells, payout processes

---

### POS Webhooks

**Problem:** CSV imports require manual work; not real-time
**Solution:** Direct webhook integrations with major POS systems
**Impact:** Eliminates manual uploads, enables real-time consignor notifications
**Effort:** Medium per POS (1-2 weeks each)
**Dependencies:** POS API access, sandbox accounts, deduplication logic
**Status:** Not Started
**PRD:** `product/prds/pos-webhooks-prd.md` (to be created)

**Integration Priority:**
1. **Square for Retail** (largest market share for indie/boutique)
2. **Shopify POS** (growing consignment adoption)
3. **Lightspeed Retail** (traditional retail POS)

**Webhook Handling:**
- Idempotency keys prevent duplicate processing
- Redis deduplication cache (24-hour TTL)
- Exponential backoff retry (3 attempts)
- Fallback to CSV if webhooks fail repeatedly

---

### Cross-Listing Engine

**Problem:** Unsold inventory sits idle; stores want to maximize exposure
**Solution:** Auto-list items to eBay, Poshmark, Mercari after X days
**Impact:** Upsell opportunity ($99/mo add-on), increases consignor satisfaction
**Effort:** Large (3-4 weeks)
**Dependencies:** Marketplace APIs, image hosting, bi-directional sync logic
**Status:** Not Started (V3)
**PRD:** To be created

**Workflow:**
1. Item unsold for X days (configurable)
2. System cross-lists to selected marketplaces
3. When item sells externally, sync back to ledger
4. Remove from all other listings
5. Process consignor payout minus cross-listing fee

---

## Success Metrics

### V1 Pilot KPIs (First 3 Pilots)

| Metric | Baseline | Target | Status |
|--------|----------|--------|--------|
| Payout Prep Time | 4 hours/cycle | 1.2 hours (-70%) | 🔵 Not Started |
| Consignor Inquiries | 40/week | 20/week (-50%) | 🔵 Not Started |
| Payout Errors | 5-10/cycle | <1/cycle (-90%) | 🔵 Not Started |
| Portal Adoption | 0% | 60% within 30 days | 🔵 Not Started |
| Time to First Payout | N/A | <7 days from signup | 🔵 Not Started |

### V2 Growth KPIs (10-20 Customers)

| Metric | Current | Target (Month 5) | Status |
|--------|---------|------------------|--------|
| Active Stores | 0 | 15 | 🔵 Not Started |
| MRR | $0 | $1,935 (15 stores × $129) | 🔵 Not Started |
| Monthly Churn | N/A | <3% | 🔵 Not Started |
| NPS Score | N/A | 60+ | 🔵 Not Started |
| Webhook Adoption | N/A | 80%+ (vs CSV) | 🔵 Not Started |

### V3 Expansion KPIs (50+ Customers)

| Metric | Current | Target (Month 9) | Status |
|--------|---------|------------------|--------|
| Active Stores | 0 | 50 | 🔵 Not Started |
| MRR | $0 | $6,450 | 🔵 Not Started |
| ARR | $0 | $77,400 | 🔵 Not Started |
| Upsell Penetration | N/A | 30% (cross-listing) | 🔵 Not Started |
| NRR | N/A | >100% | 🔵 Not Started |

---

## Resource Allocation

### Team Capacity
- **Engineering:** Solo founder (full-stack)
- **Design:** Tailwind + shadcn/ui components (minimal custom design)
- **Product:** Solo founder

### Effort Distribution (Complete Build Approach)

**V1 (Months 1-2): 100% New Features**
- Focus: Ship core product
- No distractions until pilots are live

**V2 (Months 3-5): 70% Features / 20% Refinement / 10% Support**
- New features: Webhooks, integrations
- Refinement: Based on pilot feedback
- Support: Onboarding + bug fixes

**V3 (Months 6-9): 50% Features / 30% Scaling / 20% Support**
- Features: Cross-listing, mobile
- Scaling: Performance, multi-region
- Support: Growing customer base

---

## Risks and Dependencies

| Risk/Dependency | Impact | Mitigation | Owner |
|-----------------|--------|------------|-------|
| Stripe Connect compliance delays | High | Start account verification early; consult fintech attorney | Solo Founder |
| POS API access denied | Medium | Lead with CSV; build relationships with POS resellers | Solo Founder |
| Store heterogeneity exceeds rule engine flexibility | High | Template 5-10 common scenarios; iterate based on pilot feedback | Solo Founder |
| Low consignor tech adoption (portal usage) | Medium | Mobile-first design; SMS notifications; in-store QR codes | Solo Founder |
| Incumbent lock-in (hard to switch) | High | Build data importers; offer 30-day parallel run; white-glove onboarding | Solo Founder |
| Tax/compliance missteps | High | Research W-9/1099-K requirements; use Stripe's built-in compliance | Solo Founder |

---

## What We're NOT Doing

It's important to be explicit about what we're deprioritizing:

- ❌ **Full POS Replacement** - We integrate, not compete. Stores already have POS; we're payments layer only.
- ❌ **Inventory Management** - Not building intake, tagging, floor management. POS handles this.
- ❌ **Point-of-Sale Hardware** - No barcode scanners, receipt printers, cash drawers. Existing POS covers this.
- ❌ **B2C Marketplace** - Not building a Poshmark competitor. We enable cross-listing, not host our own.
- ❌ **Consignor Acquisition Marketing** - Stores recruit their own consignors. We provide tools, not leads.
- ❌ **Multi-Currency (V1-V3)** - US-only initially. International expansion is Future/Backlog.
- ❌ **Custom Development** - No per-store custom features. Templated rules only.

---

## Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2025-11-09 | .NET/C# backend over Node.js | Financial accuracy (decimal precision), strong typing, EF Core for ledger, Azure-native |
| 2025-11-09 | Stripe Connect over Dwolla | Easier onboarding, better docs, built-in 1099-K compliance, faster time-to-market |
| 2025-11-09 | Complete Build over MVP | Financial systems require full compliance and accuracy before production; can't MVP taxes/audit trails |
| 2025-11-09 | CSV-first, webhooks later | Enables immediate pilots while integrations are built; works with any POS |
| 2025-11-09 | PostgreSQL over SQL Server | Open-source, lower Azure costs, JSON support for rule configs, row-level security for multi-tenancy |

---

## Next Steps (From This Roadmap)

1. ✅ **Roadmap complete** (this document)
2. ⏳ **Create ADRs** for key architectural decisions (payments, multi-tenancy, POS integrations)
3. ⏳ **Define OKRs** for first quarter (pilot success criteria)
4. ⏳ **Create Sprint 1 plan** (CSV importer → ledger → Stripe Connect)
5. ⏳ **Transition to Foundation phase** (once planning complete)

---

**Documented by:** Claude Code
**Date:** 2025-11-09
**Project Phase:** Planning (Task 2 of 5 complete)
