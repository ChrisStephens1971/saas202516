# ADR-003: Multi-Tenancy with Row-Level Security

**Date:** 2025-11-09
**Status:** Accepted
**Deciders:** Solo Founder
**Technical Story:** Foundation phase - multi-tenant architecture

---

## Context

Thrift_Consignment is a multi-tenant SaaS where each consignment store ("tenant") has:
- Isolated data (consignors, inventory, sales, payouts, rules)
- Separate subdomain (e.g., `bloomconsignment.thrift-consignment.com`)
- Independent pricing ($129/mo per location)
- Distinct configuration (split rules, payout schedules, branding)

**Requirements:**
1. **Data isolation:** Store A cannot see Store B's consignors or sales
2. **Performance:** Query efficiency at scale (1000+ stores)
3. **Security:** Prevent accidental data leakage across tenants
4. **Scalability:** Support unlimited tenants without infrastructure changes
5. **Cost efficiency:** Single database for all tenants (not database-per-tenant)
6. **Compliance:** Audit trail showing no cross-tenant access
7. **Development speed:** Easy to enforce tenant isolation in code

**Constraints:**
- Azure Database for PostgreSQL (already decided)
- Entity Framework Core (already decided)
- Solo founder (need guardrails to prevent mistakes)

---

## Decision

We will implement **Row-Level Security (RLS) using Entity Framework Core Global Query Filters** with subdomain-based tenant identification.

**Architecture:**
1. **Tenant identification:** Subdomain parsed from HTTP request (e.g., `bloomconsignment.thrift-consignment.com` → `bloomconsignment`)
2. **Tenant context:** Middleware sets `ITenantContext.TenantId` for the request
3. **Database schema:** All tenant-scoped tables include `TenantId` column (GUID)
4. **Global query filters:** EF Core automatically injects `WHERE TenantId = @currentTenant` on all queries
5. **Fail-safe:** Queries without tenant context throw exception (prevent accidental data leaks)

**Multi-Tenancy Model:** **Shared Database, Shared Schema** with row-level filtering

---

## Consequences

### Positive Consequences

- **Cost-efficient:** Single database for all tenants (vs database-per-tenant)
- **Easy onboarding:** New store = new row in `Tenants` table (instant)
- **Automatic isolation:** EF Core enforces tenant filtering on all queries
- **Developer safety:** Impossible to accidentally query cross-tenant data
- **Performance:** PostgreSQL indexes on `TenantId` keep queries fast
- **Simple backup/restore:** Single database to manage
- **Azure-friendly:** Works with Azure Database for PostgreSQL Flexible Server
- **Tenant-scoped migrations:** Can enable/disable features per tenant with feature flags

### Negative Consequences

- **Noisy neighbor risk:** One store's heavy queries can impact others
  - **Mitigation:** Azure Query Performance Insights, connection pooling limits per tenant
- **Compliance complexity:** GDPR data deletion requires scrubbing rows (not dropping database)
  - **Mitigation:** Soft deletes with `WHERE IsDeleted = false` filters
- **Scale limits:** Single database eventually hits limits (~10,000 tenants)
  - **Mitigation:** Shard by tenant ranges if needed (future problem)
- **Accidental leaks possible:** If developer forgets to set tenant context
  - **Mitigation:** Fail-safe throws exception if `TenantId` is null

### Neutral Consequences

- **Cannot use tenant-specific database features:** E.g., per-tenant collations, extensions
- **Global schema changes:** Migrations apply to all tenants simultaneously

---

## Alternatives Considered

### Alternative 1: Database-Per-Tenant

**Description:** Each store gets a separate PostgreSQL database

**Pros:**
- **Perfect isolation:** No risk of cross-tenant data leakage
- **Custom schemas:** Each tenant can have different schema versions
- **Performance isolation:** Noisy neighbors don't affect others
- **Easy GDPR compliance:** Delete database to remove all tenant data

**Cons:**
- **Expensive:** 100 stores = 100 databases = 100× Azure Database for PostgreSQL cost (~$50k/month)
- **Operational nightmare:** Migrations must run 100× (sequential = slow)
- **Backup complexity:** 100 databases to backup/restore
- **Connection overhead:** Need connection pooling per database
- **Slow onboarding:** Provisioning new database takes minutes

**Why rejected:** Cost prohibitive; operational complexity unmanageable for solo founder; overkill for our scale.

---

### Alternative 2: Schema-Per-Tenant

**Description:** Shared database, each tenant gets a separate PostgreSQL schema (namespace)

**Pros:**
- **Good isolation:** Tenants in separate schemas (e.g., `bloom.consignors`, `vintage.consignors`)
- **Per-tenant backups:** Can backup individual schemas
- **Moderate cost:** Single database, but isolated tables

**Cons:**
- **Complex routing:** Must switch schema context per request (`SET search_path = 'bloom'`)
- **EF Core limitations:** Poor support for dynamic schema switching
- **Migration complexity:** Must create tables in every schema
- **Query performance:** Cross-schema queries expensive
- **Schema limit:** PostgreSQL has practical limit of ~1000 schemas

**Why rejected:** EF Core doesn't handle this well; migrations become complex; not worth the added isolation.

---

### Alternative 3: Discriminator Column (Without Global Filters)

**Description:** Add `TenantId` column but rely on developers to filter manually

**Pros:**
- **Simple schema:** Just add `TenantId` to tables
- **Full control:** Developers decide when to filter by tenant

**Cons:**
- **Extremely dangerous:** Easy to forget `WHERE TenantId = @id` and leak data
- **No compile-time safety:** Bug won't be caught until production
- **Audit nightmare:** Hard to prove all queries are tenant-scoped

**Why rejected:** Unacceptable security risk; one missed filter exposes all data.

---

### Alternative 4: Multi-Database Clusters by Tenant Tier

**Description:** Free/trial tenants share a database, paid tenants get dedicated databases

**Pros:**
- **Cost-effective:** Most tenants on shared DB, premium tenants isolated
- **Performance SLA:** Paid tenants guaranteed resources

**Cons:**
- **Extreme complexity:** Two codebases (shared vs dedicated routing)
- **Migration hell:** Migrating tenant from shared → dedicated is painful
- **Over-engineering:** We're targeting $2-5M ARR (100-200 tenants), not 100k tenants

**Why rejected:** Premature optimization; adds complexity without near-term benefit.

---

## References

- [EF Core Global Query Filters](https://learn.microsoft.com/en-us/ef/core/querying/filters)
- [Multi-Tenancy in EF Core](https://learn.microsoft.com/en-us/ef/core/miscellaneous/multitenancy)
- [Azure Database for PostgreSQL Multi-Tenancy](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-multi-tenant-apps)
- [PostgreSQL Row-Level Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [SaaS Tenant Isolation Strategies](https://learn.microsoft.com/en-us/azure/architecture/guide/multitenant/considerations/tenancy-models)

---

## Notes

**Implementation Plan:**

### 1. Base Entity Classes

```csharp
// All tenant-scoped entities inherit from this
public abstract class TenantEntity
{
    public Guid Id { get; set; }
    public Guid TenantId { get; set; } // Every row has this
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
}

// Global entities (not tenant-scoped)
public abstract class GlobalEntity
{
    public Guid Id { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Example tenant-scoped entity
public class Consignor : TenantEntity
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public string StripeConnectedAccountId { get; set; }
    // ... no need to manually filter by TenantId, EF Core does it!
}

// Example global entity (tenant metadata)
public class Tenant : GlobalEntity
{
    public string Subdomain { get; set; } // "bloomconsignment"
    public string StoreName { get; set; }
    public string OwnerEmail { get; set; }
    public SubscriptionStatus Status { get; set; }
}
```

### 2. Tenant Context Service

```csharp
public interface ITenantContext
{
    Guid TenantId { get; }
    string Subdomain { get; }
}

public class TenantContext : ITenantContext
{
    private Guid? _tenantId;
    private string _subdomain;

    public Guid TenantId => _tenantId
        ?? throw new InvalidOperationException("TenantId not set. Ensure TenantMiddleware ran.");

    public string Subdomain => _subdomain
        ?? throw new InvalidOperationException("Subdomain not set.");

    public void SetTenant(Guid tenantId, string subdomain)
    {
        _tenantId = tenantId;
        _subdomain = subdomain;
    }
}
```

### 3. Tenant Middleware (Subdomain Parsing)

```csharp
public class TenantMiddleware
{
    private readonly RequestDelegate _next;

    public TenantMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context, ITenantContext tenantContext, ApplicationDbContext db)
    {
        var host = context.Request.Host.Value; // "bloomconsignment.thrift-consignment.com"
        var subdomain = ExtractSubdomain(host); // "bloomconsignment"

        if (string.IsNullOrEmpty(subdomain))
        {
            context.Response.StatusCode = 400;
            await context.Response.WriteAsync("Invalid subdomain");
            return;
        }

        // Look up tenant
        var tenant = await db.Tenants
            .Where(t => t.Subdomain == subdomain)
            .FirstOrDefaultAsync();

        if (tenant == null)
        {
            context.Response.StatusCode = 404;
            await context.Response.WriteAsync("Store not found");
            return;
        }

        // Set tenant context for this request
        ((TenantContext)tenantContext).SetTenant(tenant.Id, subdomain);

        await _next(context);
    }

    private string ExtractSubdomain(string host)
    {
        // "bloomconsignment.thrift-consignment.com" -> "bloomconsignment"
        // "localhost:5000" -> null (dev environment, handle separately)
        var parts = host.Split('.');
        return parts.Length >= 3 ? parts[0] : null;
    }
}

// Program.cs
app.UseMiddleware<TenantMiddleware>(); // Before MVC routes
```

### 4. EF Core Global Query Filters

```csharp
public class ApplicationDbContext : DbContext
{
    private readonly ITenantContext _tenantContext;

    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options, ITenantContext tenantContext)
        : base(options)
    {
        _tenantContext = tenantContext;
    }

    public DbSet<Tenant> Tenants { get; set; } // Global, no filter
    public DbSet<Consignor> Consignors { get; set; } // Tenant-scoped
    public DbSet<LedgerEntry> LedgerEntries { get; set; } // Tenant-scoped
    public DbSet<Sale> Sales { get; set; } // Tenant-scoped

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Apply global filter to all TenantEntity subclasses
        foreach (var entityType in modelBuilder.Model.GetEntityTypes())
        {
            if (typeof(TenantEntity).IsAssignableFrom(entityType.ClrType))
            {
                var parameter = Expression.Parameter(entityType.ClrType, "e");
                var property = Expression.Property(parameter, nameof(TenantEntity.TenantId));
                var tenantId = Expression.Constant(_tenantContext.TenantId);
                var equals = Expression.Equal(property, tenantId);
                var lambda = Expression.Lambda(equals, parameter);

                modelBuilder.Entity(entityType.ClrType).HasQueryFilter(lambda);
            }
        }

        // Indexes for performance
        modelBuilder.Entity<Consignor>().HasIndex(c => c.TenantId);
        modelBuilder.Entity<LedgerEntry>().HasIndex(l => l.TenantId);
        modelBuilder.Entity<Sale>().HasIndex(s => s.TenantId);
    }
}
```

### 5. Usage Example (Automatic Filtering)

```csharp
[HttpGet("api/consignors")]
public async Task<ActionResult<List<Consignor>>> GetConsignors()
{
    // NO need to filter by TenantId! EF Core does it automatically.
    // This query becomes: SELECT * FROM consignors WHERE tenant_id = @currentTenantId
    var consignors = await _db.Consignors.ToListAsync();

    return Ok(consignors);
}

[HttpPost("api/consignors")]
public async Task<ActionResult<Consignor>> CreateConsignor(CreateConsignorRequest request)
{
    var consignor = new Consignor
    {
        TenantId = _tenantContext.TenantId, // Set from context
        FirstName = request.FirstName,
        LastName = request.LastName,
        Email = request.Email
    };

    _db.Consignors.Add(consignor);
    await _db.SaveChangesAsync();

    return CreatedAtAction(nameof(GetConsignor), new { id = consignor.Id }, consignor);
}
```

### 6. Admin/Support Queries (Bypass Filter)

```csharp
// For support team to view all tenants' data (e.g., debugging)
public async Task<List<Consignor>> GetAllConsignorsAllTenants()
{
    return await _db.Consignors
        .IgnoreQueryFilters() // Bypass tenant filter
        .ToListAsync();
}

// IMPORTANT: Only expose this via admin-only API with strict auth!
```

### 7. Database Schema

```sql
-- Tenants table (global, no tenant_id column)
CREATE TABLE tenants (
    id UUID PRIMARY KEY,
    subdomain VARCHAR(100) UNIQUE NOT NULL,
    store_name VARCHAR(255) NOT NULL,
    owner_email VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL, -- Active, Suspended, Churned
    created_at TIMESTAMP NOT NULL
);

-- Consignors table (tenant-scoped)
CREATE TABLE consignors (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    stripe_connected_account_id VARCHAR(255),
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP
);

-- Index for tenant-scoped queries (CRITICAL for performance)
CREATE INDEX idx_consignors_tenant_id ON consignors(tenant_id);

-- Composite index for common queries
CREATE INDEX idx_consignors_tenant_email ON consignors(tenant_id, email);
```

### 8. Tenant Onboarding Flow

```csharp
[HttpPost("api/admin/tenants")]
public async Task<ActionResult<Tenant>> CreateTenant(CreateTenantRequest request)
{
    // Validate subdomain is available
    var exists = await _db.Tenants.AnyAsync(t => t.Subdomain == request.Subdomain);
    if (exists)
        return BadRequest("Subdomain already taken");

    var tenant = new Tenant
    {
        Id = Guid.NewGuid(),
        Subdomain = request.Subdomain,
        StoreName = request.StoreName,
        OwnerEmail = request.OwnerEmail,
        Status = SubscriptionStatus.Trial,
        CreatedAt = DateTime.UtcNow
    };

    _db.Tenants.Add(tenant);
    await _db.SaveChangesAsync();

    // Send welcome email with portal link
    await _emailService.SendWelcomeEmail(tenant);

    return Ok(tenant);
}
```

**Performance Considerations:**
- PostgreSQL will use `tenant_id` index efficiently
- Queries like `SELECT * FROM consignors WHERE tenant_id = 'abc-123'` are fast
- For 1000 tenants with 100 consignors each = 100k rows, indexed queries < 10ms
- Connection pooling: Set max connections = 100 (default Azure PostgreSQL)

**GDPR Compliance:**
```csharp
// Delete all tenant data
public async Task DeleteTenantData(Guid tenantId)
{
    using var transaction = await _db.Database.BeginTransactionAsync();

    // Soft delete approach (recommended)
    var consignors = await _db.Consignors.IgnoreQueryFilters()
        .Where(c => c.TenantId == tenantId)
        .ToListAsync();

    foreach (var consignor in consignors)
    {
        consignor.IsDeleted = true;
        consignor.Email = "[REDACTED]"; // Anonymize PII
        consignor.StripeConnectedAccountId = null;
    }

    await _db.SaveChangesAsync();
    await transaction.CommitAsync();
}
```

**Next Steps:**
- Implement base entity classes
- Create tenant context service and middleware
- Configure EF Core global query filters
- Test tenant isolation (unit tests to verify filtering)
- Add tenant ID to all API responses (for debugging)
- Set up monitoring for cross-tenant query attempts

---

**Documented by:** Claude Code
**Project Phase:** Planning - ADR 3 of 5
