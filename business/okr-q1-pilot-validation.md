# OKRs - Q1: Pilot Validation & V1 Build

**Period:** Initial Quarter (Planning → Pilot Launch)
**Owner:** Solo Founder
**Status:** Planning
**Last Updated:** 2025-11-09

---

## Company Vision & Strategy

**Vision:** Become the payments-driven backbone of consignment stores, automating error-prone payout work while providing real-time consignor transparency. Win through POS integration (not replacement) and achieve $2-5M ARR with disciplined execution.

**This Quarter's Focus:** Validate product-market fit with 3 paying pilots and build production-ready V1 wedge product (CSV import → automated payouts → consignor portal).

---

## Objective 1: Validate Product-Market Fit with Paying Pilots

**Why this matters:** We need to prove stores will pay $129/month and that our solution actually saves 70% payout prep time. Three successful pilots de-risk the entire business model and provide case studies for growth.

### Key Result 1.1: Pre-sell 3 pilot customers at $99/mo + $0.25/payout

- **Target:** 3 paying pilots (signed LOIs or contracts)
- **Current:** 0
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** Medium

**Success Criteria:**
- All 3 pilots have different POS systems (validate CSV approach works universally)
- Average store processes $15k-30k/month in consignor payouts
- At least 2 pilots willing to provide testimonials/references

**Progress Updates:**
- [Date]:

---

### Key Result 1.2: Achieve -70% payout prep time reduction (4 hours → 1.2 hours)

- **Target:** Pilot stores report ≥70% time savings on payout preparation
- **Current:** Baseline 4 hours/payout cycle (manual process)
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** High

**Measurement Method:**
- Week 1: Time pilot stores' current manual process (baseline)
- Week 4: Time same process using Thrift_Consignment (test)
- Compare: (Baseline - Test) / Baseline × 100%

**Success Criteria:**
- All 3 pilots achieve ≥60% time savings (target 70%)
- Time savings documented with before/after screenshots
- Store owners verbally confirm "this saves me hours"

**Progress Updates:**
- [Date]:

---

### Key Result 1.3: Reduce consignor inquiry volume by 50%

- **Target:** Pilot stores report 50% reduction in "when do I get paid?" inquiries
- **Current:** Baseline ~40 inquiries/week (manual process)
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** Medium

**Measurement Method:**
- Baseline: Ask pilots to estimate weekly consignor inquiries (pre-launch)
- Test: Track inquiries for 4 weeks post-portal launch
- Metric: (Baseline - Test) / Baseline × 100%

**Success Criteria:**
- At least 60% of consignors log into portal within 30 days
- Inquiry reduction validated by pilot owner testimonials
- Portal usage tracked via analytics (login frequency, pages viewed)

**Progress Updates:**
- [Date]:

---

## Objective 2: Ship Production-Ready V1 Wedge Product

**Why this matters:** Complete build approach requires production-quality code from day 1. Financial systems cannot be "MVP'd" - tax compliance, audit trails, and money handling must be correct. This objective ensures we ship a reliable, compliant product.

### Key Result 2.1: Deploy V1 feature set to production (8-week build)

- **Target:** All V1 features deployed and tested
- **Current:** 0% (planning phase)
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** High

**V1 Feature Checklist:**
- [ ] CSV importer with mapping wizard (Square, Shopify, Lightspeed formats)
- [ ] Rule engine (splits, fees, markdowns, returns)
- [ ] Per-consignor ledger with double-entry accounting
- [ ] PDF statement generation
- [ ] Stripe Connect integration (onboarding + payouts)
- [ ] W-9 collection + 1099-K setup
- [ ] Consignor portal (auth, dashboard, statements, payout history)
- [ ] Multi-tenant subdomain routing

**Definition of Done:**
- All features pass UAT with pilot stores
- No P0/P1 bugs in production
- End-to-end test: Import CSV → calculate payouts → send via Stripe → consignor views in portal

**Progress Updates:**
- [Date]:

---

### Key Result 2.2: Achieve 100% payout calculation accuracy (< 1% variance vs manual)

- **Target:** Recompute 90 days of historical payouts with <1% total variance
- **Current:** 0 historical imports completed
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** High

**Measurement Method:**
- Import 90 days of sales data from each pilot store
- Recompute payouts using rule engine
- Compare against pilot's manual calculations
- Document discrepancies and root causes

**Success Criteria:**
- Absolute variance < 1% of total payout amount
- All variances explainable (e.g., store made calculation error, not our bug)
- Zero instances of "we paid the wrong consignor"

**Example:** Store paid out $50,000 manually; our system calculates $49,800. Variance = $200 / $50,000 = 0.4% ✅

**Progress Updates:**
- [Date]:

---

### Key Result 2.3: Zero critical security/compliance failures

- **Target:** Pass security audit with no P0 issues
- **Current:** Not audited
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** High

**Audit Checklist:**
- [ ] PCI compliance: No card data stored (Stripe tokenization)
- [ ] Data encryption: TLS 1.3 in transit, AES-256 at rest
- [ ] Multi-tenant isolation: No cross-tenant data leakage (unit tested)
- [ ] Audit trail: All money movement logged with timestamps + user IDs
- [ ] GDPR compliance: Data export/deletion endpoints
- [ ] W-9/1099-K: Stripe handles tax reporting (validated in docs)
- [ ] SQL injection: Parameterized queries (EF Core enforces)
- [ ] CSRF/XSS protection: ASP.NET Core built-in protections enabled

**Testing:**
- Run OWASP ZAP against staging environment
- Attempt cross-tenant data access via API fuzzing
- Validate Stripe webhook signature verification
- Test data deletion flow (GDPR right to be forgotten)

**Progress Updates:**
- [Date]:

---

## Objective 3: Establish Foundation for Scalable Growth

**Why this matters:** Pilots validate demand, but we need infrastructure and processes to scale to 50-100 stores in Year 1. This objective ensures we're not hacking together pilots, but building a real SaaS business.

### Key Result 3.1: Maintain 99.9% uptime in production (after pilot launch)

- **Target:** <43 minutes downtime/month
- **Current:** Not in production
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** High

**Monitoring:**
- Azure Application Insights uptime monitoring
- Alerting: Slack/email on 5xx errors, database connection failures
- SLA: Documented in customer agreements

**Availability Calculation:**
- Total minutes in month: 43,200 (30 days)
- Allowed downtime: 43 minutes
- Uptime required: 99.9%

**Incidents Excluded from SLA:**
- Planned maintenance (announced 7 days in advance)
- Customer-caused outages (bad CSV format crashing import)
- Third-party failures (Stripe API down, Azure outage)

**Progress Updates:**
- [Date]:

---

### Key Result 3.2: Document all critical processes (onboarding, support, deployment)

- **Target:** 3 runbooks + 2 SOPs completed
- **Current:** 0 runbooks
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** Medium

**Required Documentation:**
1. **Customer Onboarding Runbook**
   - Store signup → subdomain setup → CSV mapping → first payout (checklist format)
   - Video: How to export sales from Square/Shopify/Lightspeed (3 videos)
   - Troubleshooting: Common CSV errors and fixes

2. **Support Runbook**
   - FAQ: Top 20 questions (with answers + screenshots)
   - Escalation: When to issue refunds, how to reverse payouts
   - Debugging: How to investigate "wrong payout amount" claims

3. **Deployment Runbook**
   - CI/CD: GitHub Actions workflow (staging → production)
   - Database migrations: How to run EF Core migrations safely
   - Rollback: How to revert a bad deployment

**SOPs:**
1. **Weekly Health Check:** GitHub Actions status, database performance, error rate dashboard
2. **Monthly Financial Close:** MRR calculation, churn analysis, pilot feedback synthesis

**Progress Updates:**
- [Date]:

---

### Key Result 3.3: Achieve <3% monthly churn (after 3-month pilot stabilization)

- **Target:** ≤3% MRR churn per month
- **Current:** 0 customers (not applicable yet)
- **Progress:** 0% 🔵
- **Owner:** Solo Founder
- **Confidence:** Medium

**Churn Calculation:**
- Churned MRR / Beginning-of-Month MRR × 100%
- Example: 3 pilots @ $99/mo = $297 MRR. 1 churns = $99 / $297 = 33.3% churn (FAIL)
- Target: After 10 customers, churn ≤ $129 / $1,290 = 10% (acceptable for early stage)

**Churn Prevention:**
- Weekly check-ins with pilots (first 30 days)
- Proactive support (monitor for payout errors, CSV import failures)
- Success metrics: Share time savings data with pilots monthly
- Iterate on feedback: Fix top 3 pain points each month

**Acceptable Churn Reasons:**
- Store goes out of business
- Store switches to full POS replacement (rare, but happens)

**Unacceptable Churn Reasons:**
- Product doesn't work (bugs, calculation errors)
- Payouts take too long / too manual
- Portal confusing / consignors won't use it

**Progress Updates:**
- [Date]:

---

## Supporting Initiatives

Projects and work that support the OKRs but aren't KRs themselves:

- **Initiative 1: 10-15 Discovery Interviews**
  - Supports: Objective 1 (pilot validation)
  - Status: Not Started
  - Goal: Identify pilot candidates, understand pain points, refine pitch

- **Initiative 2: Azure Infrastructure Setup**
  - Supports: Objective 2 (production deployment)
  - Status: Not Started
  - Deliverables: PostgreSQL DB, App Service, Key Vault, Blob Storage, Application Insights

- **Initiative 3: Legal/Compliance Research**
  - Supports: Objective 2 (compliance)
  - Status: Not Started
  - Tasks: Fintech attorney consult (Stripe Connect setup), ToS/Privacy Policy, W-9/1099-K validation

- **Initiative 4: Competitor Analysis**
  - Supports: Objective 1 (positioning)
  - Status: Not Started
  - Competitors: ConsignPro, Liberty/Resaleworld, Ricochet, SimpleConsign
  - Deliverable: Feature comparison matrix, pricing analysis, migration strategy

- **Initiative 5: Build Marketing Site**
  - Supports: Objective 1 (pilot acquisition)
  - Status: Not Started
  - Pages: Home, Features, Pricing, POS Integrations, Contact
  - Goal: SEO for "consignment store software" "consignor payout automation"

---

## Weekly Check-ins

### Week of [Date]
**Overall Status:** 🟢 On Track | 🟡 At Risk | 🔴 Off Track

**Highlights:**
-

**Concerns:**
-

**Focus for next week:**
-

---

## End of Quarter Review

### Final Scores

| Objective | Score | Notes |
|-----------|-------|-------|
| Objective 1: Pilot Validation | 0.0 | [To be completed at end of quarter] |
| Objective 2: Ship V1 Product | 0.0 | [To be completed at end of quarter] |
| Objective 3: Growth Foundation | 0.0 | [To be completed at end of quarter] |

**Overall Score:** [Average score]

### Scoring Guide
- **1.0:** Achieved everything and more (e.g., 5 pilots instead of 3)
- **0.7-0.9:** Mostly achieved (e.g., 3 pilots, 65% time savings vs 70% target)
- **0.4-0.6:** Made progress but fell short (e.g., 2 pilots, 50% time savings)
- **0.0-0.3:** Little to no progress (e.g., 0-1 pilots)

**Target Score:** 0.7+ (ambitious but achievable goals)

---

## Retrospective

### What Went Well
- [To be filled at end of quarter]

### What Didn't Go Well
- [To be filled at end of quarter]

### Learnings
- [To be filled at end of quarter]

### Adjustments for Next Quarter
- [ ] [To be filled at end of quarter]

---

## Related Links

- Product Roadmap: `product/roadmap.md`
- Discovery Document: `product/discovery.md`
- ADRs: `technical/adr/` (001-005)
- Sprint 1 Plan: `sprints/sprint-01-plan.md` (to be created)

---

## Appendix: Pilot Success Criteria Checklist

**Pre-Launch (Discovery Phase):**
- [ ] 10-15 owner interviews completed
- [ ] 3 pilot customers pre-sold (signed LOIs)
- [ ] Pilots use different POS systems (Square, Shopify, Lightspeed or other)
- [ ] Baseline metrics captured (payout prep time, inquiry volume)

**During Pilot (Weeks 1-4):**
- [ ] Historical 90-day import completed (all 3 pilots)
- [ ] Payout accuracy validated (<1% variance)
- [ ] Time savings measured (≥70% reduction)
- [ ] Portal adoption tracked (≥60% consignor login rate)
- [ ] Weekly check-ins with pilots (feedback synthesis)

**Post-Pilot (Weeks 5-8):**
- [ ] Inquiry reduction validated (≥50%)
- [ ] Testimonials collected (2-3 quotes)
- [ ] Churn: 0 pilots churned (100% retention)
- [ ] Case studies written (before/after metrics)
- [ ] Referrals: At least 1 pilot provides 2+ warm intros

---

**Documented by:** Claude Code
**Project Phase:** Planning - Task 4 of 5
