# Project Discovery - Thrift_Consignment

**Date:** 2025-11-09
**Project:** saas202516
**Trade Name:** Thrift_Consignment

---

## Project Overview

**Thesis:** Medium-viable vertical SaaS targeting thrift and consignment stores with automated consignor payouts and real-time consignor portal that integrates with existing POS systems.

**Pain Point:** Complex split calculations, aging markdowns, returns, and payout reconciliation consuming hours of manual work and causing errors.

**Value Proposition:** Replace hours of manual payout work, reduce errors, provide consignors real-time visibility, sticky through payments integration.

**Target Market:** Thrift and consignment stores (fragmented, small merchants)

**Business Model:**
- $129/mo per location (up to 10k items)
- $0.25 per payout + processor fees
- Add-ons: $29/mo white-label portal, $99 onboarding

**Target Economics:**
- CAC < $300 (FB groups, r/ThriftStore, NARTS, POS marketplaces)
- LTV $3.1k–$4.6k (24–36 months, ≤3% monthly churn)
- Payback < 3 months

**Realistic Target:** $2–5M ARR with disciplined execution

---

## Team Composition

**Structure:** Solo founder
**Phase:** Planning → Complete build
**Technical Approach:** Full-stack development with emphasis on financial accuracy and compliance

**Required Expertise:**
- Backend development (financial ledger systems)
- Payment processing (Stripe Connect)
- POS integrations (webhooks, APIs)
- Multi-tenant SaaS architecture
- Tax compliance (W-9, 1099-K)

---

## Build Approach

**Strategy:** Complete Build

**Rationale:**
- Financial systems require comprehensive testing before production
- Compliance requirements (tax, audit trail) must be complete
- Integration complexity (POS systems) needs thorough validation
- Payments/money handling cannot be MVP'd partially

**Build Order (5 phases):**
1. CSV importer, rule engine, ledger, PDF statements
2. Stripe Connect payouts + onboarding
3. Consignor portal
4. POS webhooks + label printing
5. Cross-listing + analytics

**Pilot Strategy:**
- Pre-sell 3 pilots during build
- Target: -70% payout prep time, -50% inquiry volume
- 90-day historical data import for validation

---

## Tech Stack

### Backend: .NET/C# with ASP.NET Core

**Rationale:**
- **Financial accuracy:** Built-in `decimal` type (no floating-point money errors)
- **Type safety:** Compile-time error catching for complex calculations
- **Ledger systems:** Entity Framework Core excellent for audit trails, immutable records
- **Stripe integration:** Official Stripe.NET SDK
- **Azure native:** First-class App Service, Functions, SQL support
- **Rule engine:** LINQ, pattern matching, expression trees for complex business logic
- **Industry standard:** Financial systems prefer .NET for money handling
- **CSV processing:** CsvHelper library
- **PDF generation:** QuestPDF

**Framework:** ASP.NET Core 8.0 (LTS)
**ORM:** Entity Framework Core 8.0
**Testing:** xUnit, FluentAssertions

### Frontend: Next.js/React with TypeScript

**Rationale:**
- **Consignor portal:** Real-time inventory, statements, payout tracking
- **Type safety:** TypeScript reduces runtime errors
- **SEO:** Next.js SSR for landing pages
- **Developer experience:** Fast iteration, component reuse

**Framework:** Next.js 14 (App Router)
**UI:** Tailwind CSS + shadcn/ui
**State:** React Query for API calls
**Forms:** React Hook Form + Zod validation

### Database: PostgreSQL 16

**Rationale:**
- **ACID compliance:** Critical for financial transactions
- **JSON support:** Flexible rule configuration storage
- **Row-level security:** Multi-tenant isolation
- **Full-text search:** Consignor/item lookup
- **Audit trail:** Immutable ledger entries

**Hosting:** Azure Database for PostgreSQL (Flexible Server)
**Migration:** Entity Framework Core Migrations

### Caching: Redis

**Use cases:**
- Session storage
- Rate limiting (API, payouts)
- POS webhook deduplication
- Calculation result caching

**Hosting:** Azure Cache for Redis

### Payment Processing: Stripe Connect

**Why Stripe Connect:**
- **Automated payouts:** Direct to consignor bank accounts
- **Compliance:** Stripe handles 1099-K reporting
- **Multi-recipient:** Express or Standard accounts per consignor
- **Platform fees:** Automatically deduct per-payout fees
- **Documentation:** Excellent developer experience

**Alternative considered:** Dwolla (more complex onboarding)

### Infrastructure: Azure

**Services:**
- **App Service:** ASP.NET Core backend (Standard S1)
- **Static Web Apps:** Next.js frontend (or App Service)
- **Database for PostgreSQL:** Flexible Server (Burstable B2s)
- **Cache for Redis:** Basic C0
- **Key Vault:** Secrets, connection strings, Stripe keys
- **Blob Storage:** CSV uploads, PDF statements
- **Application Insights:** Monitoring, logging
- **Front Door:** (Future) Multi-region, CDN

**Estimated Cost:**
- Dev/Test: ~$150/month
- Production: ~$400/month (single region)

**Naming Convention:** Verdaio Azure Standard v1.2
- Example: `app-vrd-202516-prd-eus2-01`

---

## Core Features (V1 Wedge Product)

### 1. Sales Ingestion
- CSV upload from POS exports
- Mapping wizard for column assignment
- Validation rules (required fields, data types)
- Historical import (90 days for pilot validation)
- **Future:** Direct webhooks (Square, Shopify, Lightspeed)

### 2. Rule Engine
- **Split calculations:** Configurable consignor/store percentages
- **Fees:** Processing fees, listing fees, per-item fees
- **Aging markdowns:** Time-based price reductions
- **Returns:** Reversible ledger entries, restock fees
- **Store-specific templates:** Pre-configured rule sets

### 3. Per-Consignor Ledger
- **Immutable audit trail:** All transactions logged
- **Double-entry accounting:** Debits/credits for accuracy
- **Transaction types:** Sales, returns, fees, markdowns, payouts
- **Balance tracking:** Real-time owed amount per consignor
- **Historical statements:** Downloadable PDF by date range

### 4. Automated Payouts
- **Stripe Connect onboarding:** Custom or Express accounts
- **W-9 collection:** Integrated into onboarding flow
- **Payout scheduling:** Weekly, bi-weekly, monthly, threshold-based
- **Batch processing:** Efficient multi-consignor payouts
- **1099-K handling:** Stripe automatic reporting
- **Platform fees:** $0.25 per payout + Stripe fees

### 5. Consignor Portal
- **Authentication:** Magic link or password-based
- **Dashboard:** Current balance, next payout date, sold/unsold items
- **Inventory view:** All items, status, pricing, markdown schedule
- **Sales history:** Transaction list, filtering, search
- **Statements:** Download PDF for any date range
- **Notifications:** Item sold, payout processed, markdown applied

---

## Integrations (Phased)

### Phase 1: CSV Import (V1)
- Square for Retail export format
- Shopify POS export format
- Lightspeed export format
- Generic template with mapping wizard

### Phase 2: Webhooks (V2)
- Square webhooks (order.created)
- Shopify webhooks (orders/create)
- Lightspeed webhooks (Sale.creating)
- Deduplication with Redis
- Retry logic with exponential backoff

### Phase 3: Label Printing (V3)
- Generate consignor tags (QR codes, barcodes)
- Print directly from portal
- Batch printing for new inventory

### Phase 4: Cross-Listing (V4)
- List unsold items on eBay, Poshmark, Mercari
- Sync inventory status bidirectionally
- Automated relisting after markdown periods

---

## Compliance & Operations

### Tax Compliance
- **W-9 Collection:** Integrated into Stripe Connect onboarding
- **1099-K Reporting:** Automatic via Stripe for $600+ annual payouts
- **Record Retention:** 7-year storage of all transactions

### Data Security
- **Encryption:** At-rest (Azure), in-transit (TLS 1.3)
- **PCI Compliance:** Stripe tokenization (no card data stored)
- **Multi-tenant isolation:** Row-level security, tenant-scoped queries
- **Audit logging:** All money-movement events tracked

### Returns & Disputes
- **Reversible entries:** Credit sale, debit consignor balance
- **Restock fees:** Configurable per-store rules
- **Dispute tracking:** Notes, resolution history

### Data Import
- **Excel templates:** Standardized column headers
- **Mapping wizard:** GUI for custom POS formats
- **Validation:** Pre-import error checking
- **Dry-run mode:** Preview changes before commit

---

## Go-to-Market Strategy

### Target POS Integrations (Priority Order)
1. **Square for Retail:** Largest indie/boutique market share
2. **Shopify POS:** Growing consignment adoption
3. **Lightspeed:** Traditional retail POS

### Marketing Channels
- **POS reseller partnerships:** Co-marketing, referral fees
- **Payment processor channels:** Stripe Partner ecosystem
- **Community groups:** FB groups, r/ThriftStore, NARTS forums
- **YouTube demos:** "How to automate consignor payouts"
- **Content marketing:** SEO for "consignment store software"

### Land & Expand
- **Land:** Payouts + portal ($129/mo)
- **Expand:** Inventory analytics ($49/mo), cross-listing ($99/mo)

---

## Key Risks & Mitigations

### Risk: Incumbent Lock-In
**Mitigation:**
- Build importers for all major legacy systems (ConsignPro, Liberty, Ricochet)
- Offer 30-day parallel run (old + new systems simultaneously)
- Free data migration support ($99 onboarding fee covers this)

### Risk: Store Heterogeneity
**Mitigation:**
- Templated rule engine (pre-configured for common scenarios)
- Custom rule builder for edge cases
- Store-specific overrides where needed

### Risk: Low Tech Adoption
**Mitigation:**
- Mobile-first UI (owner & consignor portals)
- Done-for-you setup (white-glove onboarding)
- Video tutorials and live chat support

### Risk: POS Integration Complexity
**Mitigation:**
- Start with CSV (works everywhere)
- Add webhooks incrementally (1-2 POS per quarter)
- Extensive testing with sandbox accounts
- Fallback to CSV if webhooks fail

---

## Success Metrics (Pilot KPIs)

### Operational Efficiency
- **-70% payout prep time** (from 4 hours → 1.2 hours per cycle)
- **-50% consignor inquiry volume** (portal self-service)
- **-90% payout errors** (automated vs manual calculations)

### Engagement
- **60%+ consignor portal usage** within 30 days
- **80%+ consignors** opt into automated payouts (vs checks)

### Financial
- **3-month payback** on $129/mo + setup fee
- **95%+ gross margin** (SaaS standard)
- **Churn ≤3%/month** after 6-month stabilization

### Validation Plan (2 weeks)
- 10–15 owner interviews
- Click-through prototype demo
- Import 90 days sales for 1 store
- Recompute payouts vs manual
- Quantify errors/time saved
- Pre-sell 3 pilots at $99/mo + $0.25/payout

---

## Next Steps

1. ✅ **Discovery complete** (this document)
2. ⏳ **Create product roadmap** (Now/Next/Later)
3. ⏳ **Define architecture** (ADRs for payments, multi-tenancy, integrations)
4. ⏳ **Set OKRs** (Quarterly objectives, pilot targets)
5. ⏳ **Create Sprint 1 plan** (CSV importer, rule engine, ledger)

---

**Documented by:** Claude Code
**Date:** 2025-11-09
**Project Phase:** Planning (Task 1 of 5 complete)
