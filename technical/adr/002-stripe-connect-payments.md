# ADR-002: Stripe Connect for Consignor Payouts

**Date:** 2025-11-09
**Status:** Accepted
**Deciders:** Solo Founder
**Technical Story:** V1 Wedge Product - automated payout system

---

## Context

Thrift_Consignment's core value proposition is automating consignor payouts. Stores currently pay consignors via manual checks, cash, or ACH transfers, requiring hours of reconciliation work and frequent errors.

**Requirements:**
1. **Automated payouts:** ACH transfers to consignor bank accounts
2. **Tax compliance:** W-9 collection, 1099-K reporting for $600+ annual payments
3. **Platform fees:** Deduct $0.25 per payout + processor fees
4. **Multi-recipient:** Each consignor has their own payout destination
5. **Payout scheduling:** Weekly, bi-weekly, monthly, or threshold-based
6. **Dispute handling:** Ability to reverse/adjust payouts
7. **Security:** PCI compliance, no storing of bank account numbers
8. **US-only (V1):** Domestic ACH transfers initially

**Constraints:**
- Solo founder (limited compliance expertise)
- Complete build (need production-ready from day 1)
- $129/mo pricing (cannot afford expensive processor fees)
- Sticky product (payments create lock-in)

---

## Decision

We will use **Stripe Connect (Express Accounts)** for all consignor payouts.

**Implementation:**
- **Platform account:** Thrift_Consignment has a single Stripe account
- **Connected accounts:** Each consignor onboards via Stripe Express (simplified onboarding)
- **Payout flow:** Platform transfers funds to consignor's connected account via Stripe Connect transfers
- **Tax compliance:** Stripe collects W-9 during onboarding, issues 1099-K automatically
- **Platform fees:** Deducted via `application_fee_amount` parameter

**Stripe Connect Model:** **Express Accounts**
- Consignors onboard in ~2 minutes (SSN, DOB, bank account)
- Stripe handles compliance, fraud prevention, tax reporting
- Platform controls payout timing and amounts
- Consignors can optionally access Stripe Express dashboard

---

## Consequences

### Positive Consequences

- **Compliance handled:** Stripe automatically collects W-9, issues 1099-K, handles Know Your Customer (KYC)
- **Fraud protection:** Stripe vets consignors, prevents fraudulent bank accounts
- **Minimal code:** Stripe.NET SDK handles complex payout orchestration
- **PCI compliance:** No sensitive bank data touches our servers
- **Tax reporting:** 1099-K automatically sent to consignors and IRS ($600+ threshold)
- **Dispute resolution:** Can reverse transfers if needed (within 90 days)
- **Excellent documentation:** Stripe has best-in-class developer experience
- **Fast onboarding:** Consignors onboard in minutes (not days like ACH/Dwolla)
- **Mobile-friendly:** Stripe Express onboarding works on phones
- **Platform fees:** Built-in `application_fee` parameter for $0.25/payout
- **Payout speed:** 1-2 business days to consignor bank account

### Negative Consequences

- **Cost:** 0.25% + $0.25 per payout (for Express accounts)
  - Example: $500 payout costs $1.50 in Stripe fees
  - Platform charges consignor $0.25, absorbs $1.25
  - **Mitigation:** Pass through processor fees to store owner or consignor
- **US-only:** International expansion requires Stripe expansion into those countries
- **Vendor lock-in:** Difficult to switch payment processors later
  - **Mitigation:** Abstract behind `IPayoutService` interface
- **Consignor onboarding friction:** Requires SSN, bank account (some may resist)
  - **Mitigation:** Explain tax compliance, offer check fallback for holdouts

### Neutral Consequences

- **Stripe brand:** Consignors see Stripe branding during onboarding (not white-labeled)
- **Payout delays:** Not instant (1-2 business days); stores used to checks anyway
- **Account reserves:** Stripe may hold funds for new platforms (common for fintech)

---

## Alternatives Considered

### Alternative 1: Dwolla

**Description:** ACH payment API, popular for fintech startups, lower fees than Stripe

**Pros:**
- **Lower fees:** $0.50 flat fee per ACH transfer (vs Stripe's 0.25% + $0.25)
- **White-label:** More branding control
- **Direct ACH:** No intermediary accounts (direct bank-to-bank)

**Cons:**
- **Complex onboarding:** Requires manual compliance work (W-9 collection, 1099-K generation)
  - Must build custom tax reporting system
  - Must handle KYC verification ourselves
- **Slower onboarding:** 2-5 days for micro-deposit verification (vs Stripe's instant)
- **Less documentation:** Smaller ecosystem, fewer code examples
- **More compliance risk:** Solo founder responsible for tax/fraud compliance
- **No 1099-K automation:** Must integrate with tax software separately

**Why rejected:** Compliance burden too high for solo founder; time-to-market delay; savings (~$1/payout) not worth the risk.

---

### Alternative 2: PayPal Payouts API

**Description:** PayPal's mass payout API for sending money to email/PayPal accounts

**Pros:**
- **Ubiquitous:** Most consignors already have PayPal accounts
- **Instant payouts:** Funds available immediately (if paid to PayPal balance)
- **No onboarding:** Just need consignor's email address
- **International:** Works globally

**Cons:**
- **High fees:** 2% per payout (vs Stripe's 0.25%)
  - Example: $500 payout costs $10 in PayPal fees (vs $1.50 Stripe)
- **No W-9/1099-K:** Must handle tax compliance separately
- **PayPal required:** Consignors must create PayPal accounts (friction)
- **Disputes:** PayPal favors consumers; hard to reverse fraudulent payouts
- **Poor developer experience:** API less modern than Stripe

**Why rejected:** Fees 8x higher than Stripe; no tax compliance features; PayPal account requirement a barrier.

---

### Alternative 3: Direct ACH via Plaid + Modern Treasury

**Description:** Use Plaid for bank verification, Modern Treasury for ACH transfers

**Pros:**
- **Full control:** Own the entire payment flow
- **Lower fees:** ~$0.50 per ACH transfer
- **White-label:** Complete branding control
- **Best-in-class bank linking:** Plaid instant verification

**Cons:**
- **Extreme complexity:** Two vendors (Plaid + Modern Treasury), complex integration
- **Compliance burden:** Must build W-9 collection, 1099-K generation, KYC verification
- **Fraud risk:** Responsible for detecting fraudulent bank accounts
- **Time-to-market:** 4-6 weeks of integration work (vs 1 week for Stripe)
- **Cost:** Plaid ($0.60/verification) + Modern Treasury ($0.50/transfer) = $1.10 total
- **Maintenance:** Two vendor relationships, two SDKs, two support channels

**Why rejected:** Massive over-engineering for V1; compliance risk unacceptable; time-to-market delay kills pilots.

---

### Alternative 4: Manual ACH via Bank Portal

**Description:** Store owners manually upload ACH files to their bank

**Pros:**
- **No fees:** Bank charges minimal ACH fees (or free)
- **Full control:** Store's existing bank relationship

**Cons:**
- **Not automated:** Still manual work (defeats our value prop)
- **Error-prone:** Typos in account/routing numbers
- **No tax compliance:** Still requires manual W-9/1099-K
- **Not sticky:** Easy to switch away from us (we don't control payments)

**Why rejected:** Doesn't solve the core problem; not a SaaS solution.

---

## References

- [Stripe Connect Documentation](https://stripe.com/docs/connect)
- [Stripe Connect Pricing](https://stripe.com/connect/pricing)
- [Stripe Express Accounts Overview](https://stripe.com/docs/connect/express-accounts)
- [Stripe 1099-K Reporting](https://stripe.com/docs/connect/taxes)
- [Dwolla Pricing](https://www.dwolla.com/pricing/)
- [Comparison: Stripe vs Dwolla for Marketplaces](https://www.moderntreasury.com/journal/stripe-connect-vs-dwolla)

---

## Notes

**Implementation Plan:**

### Phase 1: Platform Setup (Week 6)
1. Create Stripe platform account
2. Enable Stripe Connect in dashboard
3. Configure Express account settings
4. Set up webhook endpoints (for `account.updated`, `payout.paid`, etc.)
5. Install Stripe.NET SDK in backend

### Phase 2: Consignor Onboarding Flow (Week 6-7)
```csharp
// Backend API: Generate Stripe Connect onboarding link
[HttpPost("api/consignors/{id}/stripe-onboarding")]
public async Task<ActionResult<string>> CreateOnboardingLink(Guid id)
{
    var consignor = await _db.Consignors.FindAsync(id);

    var account = await _stripe.Accounts.CreateAsync(new AccountCreateOptions
    {
        Type = "express",
        Country = "US",
        Email = consignor.Email,
        Capabilities = new AccountCapabilitiesOptions
        {
            CardPayments = new AccountCapabilitiesCardPaymentsOptions { Requested = false },
            Transfers = new AccountCapabilitiesTransfersOptions { Requested = true }
        }
    });

    var accountLink = await _stripe.AccountLinks.CreateAsync(new AccountLinkCreateOptions
    {
        Account = account.Id,
        RefreshUrl = "https://portal.thrift-consignment.com/onboarding/refresh",
        ReturnUrl = "https://portal.thrift-consignment.com/dashboard",
        Type = "account_onboarding"
    });

    consignor.StripeConnectedAccountId = account.Id;
    await _db.SaveChangesAsync();

    return Ok(accountLink.Url); // Frontend redirects consignor here
}
```

### Phase 3: Payout Processing (Week 7)
```csharp
// Payout service
public async Task<Payout> ProcessPayout(Guid consignorId)
{
    var consignor = await _db.Consignors
        .Include(c => c.Ledger)
        .FirstAsync(c => c.Id == consignorId);

    var balance = consignor.Ledger.CalculateBalance();
    if (balance < 10) // Minimum payout threshold
        throw new InvalidOperationException("Balance too low");

    // Create Stripe transfer
    var transfer = await _stripe.Transfers.CreateAsync(new TransferCreateOptions
    {
        Amount = (long)(balance * 100), // Convert to cents
        Currency = "usd",
        Destination = consignor.StripeConnectedAccountId,
        TransferGroup = $"payout-{Guid.NewGuid()}",
        Metadata = new Dictionary<string, string>
        {
            { "consignor_id", consignorId.ToString() },
            { "store_id", consignor.StoreId.ToString() }
        }
    });

    // Deduct platform fee
    await _stripe.ApplicationFees.CreateAsync(new ApplicationFeeCreateOptions
    {
        Amount = 25, // $0.25 in cents
        Currency = "usd",
        ChargeId = transfer.Id
    });

    // Record in ledger
    consignor.Ledger.AddEntry(new LedgerEntry
    {
        TransactionType = "Payout",
        CreditAmount = balance, // Debit consignor's balance
        StripeTransferId = transfer.Id,
        CreatedAt = DateTime.UtcNow
    });

    await _db.SaveChangesAsync();

    return new Payout { Amount = balance, TransferId = transfer.Id };
}
```

### Phase 4: Webhook Handling (Week 7)
```csharp
// Handle Stripe webhooks
[HttpPost("api/webhooks/stripe")]
public async Task<IActionResult> HandleWebhook()
{
    var json = await new StreamReader(Request.Body).ReadToEndAsync();
    var sig = Request.Headers["Stripe-Signature"];

    var stripeEvent = EventUtility.ConstructEvent(json, sig, _webhookSecret);

    switch (stripeEvent.Type)
    {
        case "account.updated":
            // Consignor completed onboarding
            var account = (Account)stripeEvent.Data.Object;
            await MarkConsignorOnboardingComplete(account.Id);
            break;

        case "payout.paid":
            // Payout reached consignor's bank
            var payout = (Payout)stripeEvent.Data.Object;
            await MarkPayoutCompleted(payout.Id);
            break;

        case "payout.failed":
            // Payout failed (bad bank account, etc.)
            var failedPayout = (Payout)stripeEvent.Data.Object;
            await HandlePayoutFailure(failedPayout.Id);
            break;
    }

    return Ok();
}
```

### Tax Compliance (Automatic)
- Stripe collects W-9 during Express onboarding
- Stripe issues 1099-K to consignors who receive $600+ in a calendar year
- 1099-K sent to IRS automatically (January 31st deadline)
- No action required from our platform

### Payout Scheduling
```csharp
// Background job (Hangfire/Quartz.NET)
public async Task ProcessScheduledPayouts()
{
    var dueConsignors = await _db.Consignors
        .Where(c => c.NextPayoutDate <= DateTime.UtcNow)
        .Where(c => c.Ledger.Balance >= 10) // Min threshold
        .ToListAsync();

    foreach (var consignor in dueConsignors)
    {
        await ProcessPayout(consignor.Id);
        consignor.NextPayoutDate = CalculateNextPayoutDate(consignor.PayoutFrequency);
    }

    await _db.SaveChangesAsync();
}
```

**Cost Analysis:**

Example store: 50 consignors, $25,000/month in consignor payouts
- Average payout: $500/consignor (monthly)
- Stripe fee per payout: 0.25% + $0.25 = $1.25 + $0.25 = $1.50
- Total monthly Stripe fees: 50 × $1.50 = $75
- Platform revenue from payouts: 50 × $0.25 = $12.50
- Net cost to platform: $75 - $12.50 = $62.50/month

**Mitigation:** Add payout fee to pricing (e.g., "$129/mo + processor fees" or "$0.50/payout to cover processing")

**Next Steps:**
- Apply for Stripe Connect access (1-2 business days)
- Set up Stripe test mode accounts
- Build onboarding flow in consignor portal
- Test payouts in sandbox
- Document onboarding process for store owners

---

**Documented by:** Claude Code
**Project Phase:** Planning - ADR 2 of 5
