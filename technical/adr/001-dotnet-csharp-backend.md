# ADR-001: .NET/C# Backend with ASP.NET Core

**Date:** 2025-11-09
**Status:** Accepted
**Deciders:** Solo Founder
**Technical Story:** Foundation phase - backend technology selection

---

## Context

Thrift_Consignment is a financial ledger system that handles consignor payouts, complex split calculations, aging markdowns, returns, and audit trails. The backend must:

1. **Financial accuracy:** No floating-point errors in money calculations
2. **Complex business logic:** Rule engine for splits, fees, markdowns (per-store customization)
3. **Data integrity:** Immutable audit trail, double-entry accounting
4. **Payment integration:** Stripe Connect for automated payouts
5. **Multi-tenancy:** Per-location data isolation
6. **Azure deployment:** Cloud-native, scalable
7. **Tax compliance:** W-9 collection, 1099-K reporting support

Key constraints:
- Solo founder (need strong typing to catch errors)
- Complete build approach (cannot compromise on financial accuracy)
- Azure infrastructure (already committed)
- No prior production fintech experience (need guardrails)

---

## Decision

We will use **.NET 8 (LTS) with ASP.NET Core** as the backend technology stack.

**Core Stack:**
- Runtime: .NET 8 (LTS, supported until November 2026)
- Framework: ASP.NET Core 8.0 (Web API)
- ORM: Entity Framework Core 8.0
- Testing: xUnit, FluentAssertions, Moq
- Logging: Serilog with Application Insights
- Deployment: Azure App Service (Linux containers)

**Key Libraries:**
- Stripe.NET (official SDK)
- CsvHelper (CSV processing)
- QuestPDF (statement generation)
- FluentValidation (input validation)
- MediatR (CQRS pattern for complex workflows)

---

## Consequences

### Positive Consequences

- **Decimal precision:** Built-in `decimal` type prevents floating-point money errors (critical for financial systems)
- **Strong typing:** Compile-time error detection reduces runtime financial bugs
- **EF Core:** Excellent ORM for audit trails, ledgers, multi-tenancy patterns, change tracking
- **Azure-native:** First-class App Service support, seamless Application Insights integration
- **Stripe.NET:** Official SDK with strong typing, async/await, webhook signature verification
- **Mature ecosystem:** Extensive libraries for PDF generation, CSV processing, tax calculations
- **Performance:** Faster than Node.js/Python for CPU-intensive rule engine calculations
- **Maintainability:** Solo founder benefits from compiler catching mistakes
- **Industry standard:** Financial systems (banking, payroll, accounting) predominantly use .NET

### Negative Consequences

- **Slower initial development:** More ceremony than Node.js/Python (requires more boilerplate)
- **Steeper learning curve:** If founder has no C# experience (mitigated by strong typing reducing errors)
- **Larger deployment footprint:** Heavier container images than Node.js (mitigated by Azure App Service optimization)

### Neutral Consequences

- **Windows development preference:** Best IDE is Visual Studio (Windows) or VS Code with C# DevKit (cross-platform)
- **Cross-platform capable:** Runs on Linux containers (our Azure deployment strategy)

---

## Alternatives Considered

### Alternative 1: Node.js/TypeScript

**Description:** Express or Fastify backend with TypeScript, Prisma ORM, Decimal.js for money

**Pros:**
- Faster initial development (less boilerplate)
- Large npm ecosystem
- Great for real-time webhooks (async I/O)
- Excellent Stripe SDK (Stripe Node.js)

**Cons:**
- **No native decimal type:** Must use libraries (Decimal.js, dinero.js) - easy to forget and introduce float bugs
- **Weaker type safety:** TypeScript is compile-time only; runtime errors more common
- **ORM limitations:** Prisma good but less mature than EF Core for complex ledger/audit patterns
- **Financial risk:** Node.js less common in fintech due to precision concerns

**Why rejected:** Financial accuracy is non-negotiable. Native decimal support and compile-time guarantees outweigh development speed.

---

### Alternative 2: Python/FastAPI

**Description:** FastAPI backend with SQLAlchemy ORM, pandas for CSV processing

**Pros:**
- Excellent for data processing (pandas, numpy)
- FastAPI modern and fast (async)
- Great for rule engines (Python's expressiveness)
- Good CSV handling

**Cons:**
- **Slower performance:** GIL limits concurrency for rule engine calculations
- **Type safety optional:** Type hints not enforced at runtime
- **Decimal precision:** Have to remember to use `decimal.Decimal` (not default)
- **Smaller fintech ecosystem:** Less common for payment processing apps
- **Azure integration:** Less mature than .NET (App Service prefers .NET/Node)

**Why rejected:** Performance concerns for rule engine, type safety gaps, Azure fit

---

### Alternative 3: Go

**Description:** Go backend with GORM or sqlx, excellent for high-performance APIs

**Pros:**
- Very fast (compiled, concurrent)
- Great for webhooks (goroutines)
- Strong typing
- Single binary deployment

**Cons:**
- **Smaller ecosystem:** Fewer financial/ledger libraries
- **No decimal type:** Must use third-party libraries (shopspring/decimal)
- **Stripe SDK:** Community-maintained, not official
- **Less fintech adoption:** Newer language for this domain
- **Solo founder learning curve:** If no Go experience

**Why rejected:** Ecosystem and Stripe SDK concerns; performance advantage not needed at our scale

---

## References

- [.NET Decimal Type Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types#the-decimal-type)
- [Entity Framework Core Multi-Tenancy](https://learn.microsoft.com/en-us/ef/core/miscellaneous/multitenancy)
- [Stripe.NET SDK](https://github.com/stripe/stripe-dotnet)
- [Azure App Service .NET Support](https://learn.microsoft.com/en-us/azure/app-service/overview)
- [Financial Application Best Practices (Stack Overflow)](https://stackoverflow.com/questions/3730019/why-not-use-double-or-float-to-represent-currency)

---

## Notes

**Implementation Considerations:**

1. **Money Representation:**
   ```csharp
   public class Money
   {
       public decimal Amount { get; set; }
       public string Currency { get; set; } = "USD";

       // Always use decimal, never double/float
   }
   ```

2. **Rule Engine Pattern:**
   ```csharp
   public interface IPayoutRule
   {
       decimal Apply(decimal saleAmount, RuleContext context);
   }

   // Use strategy pattern for different split/fee/markdown rules
   ```

3. **Ledger Entry Pattern:**
   ```csharp
   public class LedgerEntry
   {
       public Guid Id { get; set; }
       public Guid ConsignorId { get; set; }
       public decimal DebitAmount { get; set; }  // Money owed TO consignor
       public decimal CreditAmount { get; set; } // Money FROM consignor (fees, returns)
       public DateTime CreatedAt { get; set; }
       public string TransactionType { get; set; } // Sale, Payout, Fee, Return
       public bool IsReversible { get; set; } = false; // For returns
   }
   ```

4. **Audit Trail:**
   - Use EF Core's `ChangeTracker` to log all database modifications
   - Store `CreatedBy`, `CreatedAt`, `ModifiedBy`, `ModifiedAt` on all entities
   - Never delete; use soft deletes (IsDeleted flag)

5. **Multi-Tenancy:**
   - Use EF Core Global Query Filters for tenant isolation
   - Every entity has `TenantId` (store's unique ID)
   - Middleware sets tenant context from subdomain/JWT

**Next Steps:**
- Set up .NET 8 solution structure
- Configure EF Core with PostgreSQL provider
- Implement base entity classes (AuditableEntity, TenantEntity)
- Set up Stripe.NET SDK and webhook handling

---

**Documented by:** Claude Code
**Project Phase:** Planning - ADR 1 of 5
