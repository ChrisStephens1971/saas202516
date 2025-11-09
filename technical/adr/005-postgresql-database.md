# ADR-005: PostgreSQL for Primary Database

**Date:** 2025-11-09
**Status:** Accepted
**Deciders:** Solo Founder
**Technical Story:** Foundation phase - database selection

---

## Context

Thrift_Consignment requires a relational database for:

1. **Financial ledger:** Double-entry accounting, immutable audit trail
2. **Consignor data:** Personal info, Stripe account IDs, payout schedules
3. **Sales transactions:** Item sales, returns, adjustments
4. **Rule configurations:** JSON-based rule engine settings (per-tenant)
5. **Multi-tenancy:** Row-level security with tenant isolation
6. **ACID compliance:** Critical for money-handling operations
7. **Complex queries:** Reconciliation reports, payout calculations, analytics
8. **Azure deployment:** Must work with Azure Database services

**Key Requirements:**
- **Decimal precision:** No floating-point errors with money
- **Transactions:** ACID guarantees for payout processing
- **JSON support:** Store flexible rule engine configurations
- **Full-text search:** Consignor/item lookup
- **Multi-tenant friendly:** Row-level security, efficient tenant filtering
- **Azure-native:** Managed service with backups, scaling
- **Cost-effective:** Solo founder budget (~$50-150/month for dev/staging)

**Constraints:**
- .NET 8 + Entity Framework Core (already decided)
- Azure infrastructure (already committed)
- Complete build approach (need production-ready from day 1)

---

## Decision

We will use **PostgreSQL 16 via Azure Database for PostgreSQL Flexible Server**.

**Deployment:**
- **Dev/Staging:** Burstable B2s (2 vCPU, 4 GB RAM) - ~$50/month
- **Production:** General Purpose D2s_v3 (2 vCPU, 8 GB RAM) - ~$150/month
- **High Availability:** Enabled in production (zone-redundant)
- **Backups:** 35-day retention, geo-redundant
- **Encryption:** TLS 1.3 in transit, Azure Storage encryption at rest

**Connection:**
- **ORM:** Entity Framework Core with Npgsql provider
- **Migrations:** EF Core migrations for schema versioning
- **Connection pooling:** Npgsql built-in pooling (max 100 connections)

---

## Consequences

### Positive Consequences

- **Cost-effective:** Significantly cheaper than SQL Server on Azure
  - PostgreSQL Flexible B2s: ~$50/month
  - Azure SQL Database S2: ~$150/month
  - **Savings:** ~$1,200/year in dev/staging alone
- **Excellent JSON support:** `jsonb` type for rule engine configurations (queryable, indexed)
- **Full-text search:** Built-in `tsvector` for consignor/item search
- **Row-level security:** Native RLS for multi-tenancy (though we're using EF Core filters)
- **ACID compliance:** Rock-solid transactions for financial operations
- **Open-source:** No licensing costs, large community
- **EF Core support:** Excellent Npgsql provider, feature parity with SQL Server provider
- **Advanced data types:** Arrays, JSONB, range types, enums
- **Azure managed:** Automatic backups, patching, monitoring via Application Insights
- **Scalability:** Can scale up to 64 vCPUs, 672 GB RAM if needed
- **Geo-replication:** Read replicas for analytics (future)
- **Industry standard:** Used by Stripe, Instagram, Reddit (proven for fintech)

### Negative Consequences

- **Less .NET-native than SQL Server:** Microsoft's primary DB is SQL Server (but Npgsql is excellent)
- **Azure tooling:** SQL Server has tighter Azure integration (but difference is minimal)
- **Team familiarity:** If solo founder only knows SQL Server syntax
  - **Mitigation:** SQL is 95% identical; small learning curve
- **No T-SQL:** Use PL/pgSQL for stored procedures (if needed)
  - **Mitigation:** We're using EF Core, not stored procedures

### Neutral Consequences

- **Requires Npgsql package:** Extra NuGet package vs built-in SQL Server provider
- **Azure portal:** Slightly different UI vs Azure SQL Database (both managed well)

---

## Alternatives Considered

### Alternative 1: Azure SQL Database

**Description:** Microsoft's managed SQL Server in Azure

**Pros:**
- **Perfect .NET integration:** First-class EF Core support
- **Familiar:** T-SQL for developers with SQL Server background
- **Azure-native:** Tightest integration with Azure services
- **Advanced security:** Always Encrypted, Dynamic Data Masking, Transparent Data Encryption

**Cons:**
- **Expensive:** 3-4x cost of PostgreSQL for equivalent performance
  - Azure SQL S2 (50 DTUs): ~$150/month (equivalent to PostgreSQL B2s at $50/month)
  - DTU limits frustrating for bursty workloads
- **No JSON support (pre-2016):** Must use `nvarchar(max)` and parse JSON as strings
- **Less modern:** PostgreSQL innovates faster (JSONB, better indexing, extensions)
- **Licensing complexity:** DTU vs vCore models confusing
- **Limited full-text search:** Not as powerful as PostgreSQL's `tsvector`

**Why rejected:** Cost too high for solo founder; PostgreSQL has better JSON support for rule engine.

---

### Alternative 2: MySQL via Azure Database for MySQL

**Description:** Popular open-source relational database

**Pros:**
- **Cheap:** Similar cost to PostgreSQL (~$50/month)
- **Wide adoption:** Large community, many tutorials
- **Azure managed:** Flexible Server available
- **EF Core support:** Pomelo.EntityFrameworkCore.MySql provider

**Cons:**
- **Weaker JSON support:** JSON type exists but less performant than PostgreSQL's JSONB
- **Less advanced features:** No array types, range types, or advanced indexing
- **Full-text search:** Not as good as PostgreSQL
- **Transactions:** Less robust than PostgreSQL (historically)
- **EF Core provider:** Pomelo is community-maintained (vs Npgsql which is official-ish)

**Why rejected:** PostgreSQL superior for our use case (JSON, full-text search, financial data).

---

### Alternative 3: Azure Cosmos DB (NoSQL)

**Description:** Globally distributed NoSQL database (document model)

**Pros:**
- **Scalability:** Infinite horizontal scaling
- **Global distribution:** Multi-region writes
- **Low latency:** Single-digit millisecond reads/writes
- **Flexible schema:** JSON documents, no migrations needed

**Cons:**
- **Expensive:** RU-based pricing very costly for low-traffic apps
  - Minimum ~$25/month for 400 RU/s (barely usable)
  - Realistic usage ~$200-500/month for production
- **No ACID transactions across documents:** Only within single partition
  - **Deal-breaker for financial ledger** (need multi-row transactions)
- **Complex queries:** No SQL JOINs (must denormalize or multiple queries)
- **Learning curve:** Partition key design critical (easy to mess up)
- **EF Core support:** Experimental, not production-ready

**Why rejected:** No multi-document ACID transactions = unacceptable for financial system.

---

### Alternative 4: MongoDB (Self-Hosted or Atlas)

**Description:** Popular document database

**Pros:**
- **Flexible schema:** Easy to iterate on data model
- **JSON-native:** Documents stored as BSON
- **Good for rule engine:** Could store rules as documents

**Cons:**
- **No ACID guarantees (until v4):** Risky for financial data
- **No relational model:** Harder to model consignor → sales → payouts relationships
- **EF Core support:** MongoDB.EntityFrameworkCore exists but limited
- **Azure hosting:** Must self-host on VMs or use MongoDB Atlas (separate vendor)
- **Cost:** MongoDB Atlas not cheap; self-hosting adds operational burden

**Why rejected:** Relational model better for financial ledger; EF Core support weak.

---

## References

- [Azure Database for PostgreSQL Flexible Server](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/overview)
- [PostgreSQL 16 Documentation](https://www.postgresql.org/docs/16/index.html)
- [Npgsql - PostgreSQL EF Core Provider](https://www.npgsql.org/efcore/)
- [Azure PostgreSQL Pricing](https://azure.microsoft.com/en-us/pricing/details/postgresql/flexible-server/)
- [PostgreSQL vs SQL Server Comparison](https://www.timescale.com/blog/how-postgresql-compares-to-sql-server/)
- [PostgreSQL JSONB Performance](https://www.postgresql.org/docs/current/datatype-json.html)

---

## Notes

**Implementation Plan:**

### 1. Provision Azure PostgreSQL Flexible Server

```bash
# Azure CLI commands
az postgres flexible-server create \
  --resource-group rg-vrd-202516-dev-eus2-app \
  --name pg-vrd-202516-dev-eus2 \
  --location eastus2 \
  --admin-user pgadmin \
  --admin-password <secure-password> \
  --sku-name Standard_B2s \
  --tier Burstable \
  --storage-size 32 \
  --version 16 \
  --high-availability Disabled \
  --backup-retention 7 \
  --tags Org=vrd Project=202516 Environment=dev
```

**Production (when ready):**
```bash
az postgres flexible-server create \
  --resource-group rg-vrd-202516-prd-eus2-data \
  --name pg-vrd-202516-prd-eus2 \
  --location eastus2 \
  --admin-user pgadmin \
  --admin-password <secure-password> \
  --sku-name Standard_D2s_v3 \
  --tier GeneralPurpose \
  --storage-size 128 \
  --version 16 \
  --high-availability ZoneRedundant \
  --backup-retention 35 \
  --geo-redundant-backup Enabled \
  --tags Org=vrd Project=202516 Environment=prd
```

### 2. Connection String (Store in Azure Key Vault)

```csharp
// Development
"DefaultConnection": "Host=pg-vrd-202516-dev-eus2.postgres.database.azure.com;Database=thriftconsignment;Username=pgadmin;Password=<password>;SSL Mode=Require;Trust Server Certificate=true"

// Production (use managed identity)
"DefaultConnection": "Host=pg-vrd-202516-prd-eus2.postgres.database.azure.com;Database=thriftconsignment;Username=pgadmin;Password=<from-keyvault>;SSL Mode=Require;Trust Server Certificate=true"
```

### 3. EF Core Configuration

```csharp
// Program.cs
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseNpgsql(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        npgsqlOptions =>
        {
            npgsqlOptions.EnableRetryOnFailure(maxRetryCount: 3);
            npgsqlOptions.CommandTimeout(30);
        });

    if (builder.Environment.IsDevelopment())
    {
        options.EnableSensitiveDataLogging();
        options.EnableDetailedErrors();
    }
});
```

### 4. Initial Migration

```bash
# Install Npgsql EF Core provider
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL

# Create initial migration
dotnet ef migrations add InitialCreate

# Apply to database
dotnet ef database update
```

### 5. Database Schema Design

**Core Tables:**

```sql
-- Tenants (global)
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    subdomain VARCHAR(100) UNIQUE NOT NULL,
    store_name VARCHAR(255) NOT NULL,
    owner_email VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP
);

-- Consignors (tenant-scoped)
CREATE TABLE consignors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    stripe_connected_account_id VARCHAR(255),
    payout_frequency VARCHAR(20) NOT NULL DEFAULT 'monthly', -- weekly, biweekly, monthly, threshold
    next_payout_date TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP
);

CREATE INDEX idx_consignors_tenant_id ON consignors(tenant_id);
CREATE INDEX idx_consignors_stripe_id ON consignors(stripe_connected_account_id);

-- Sales (tenant-scoped)
CREATE TABLE sales (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    consignor_id UUID NOT NULL REFERENCES consignors(id),
    transaction_id VARCHAR(100) NOT NULL,
    transaction_date TIMESTAMP NOT NULL,
    sale_price DECIMAL(10,2) NOT NULL,
    item_description VARCHAR(500),
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_sales_tenant_id ON sales(tenant_id);
CREATE INDEX idx_sales_consignor_id ON sales(consignor_id);
CREATE INDEX idx_sales_transaction_id ON sales(tenant_id, transaction_id); -- Prevent duplicates
CREATE INDEX idx_sales_transaction_date ON sales(transaction_date);

-- Ledger entries (tenant-scoped)
CREATE TABLE ledger_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    consignor_id UUID NOT NULL REFERENCES consignors(id),
    transaction_type VARCHAR(50) NOT NULL, -- Sale, Payout, Fee, Return, Adjustment
    debit_amount DECIMAL(10,2) NOT NULL DEFAULT 0.00, -- Money owed TO consignor
    credit_amount DECIMAL(10,2) NOT NULL DEFAULT 0.00, -- Money FROM consignor (fees, returns)
    description TEXT,
    reference_id UUID, -- FK to sales.id, payouts.id, etc.
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    created_by VARCHAR(100) -- User ID for audit
);

CREATE INDEX idx_ledger_tenant_id ON ledger_entries(tenant_id);
CREATE INDEX idx_ledger_consignor_id ON ledger_entries(consignor_id);
CREATE INDEX idx_ledger_created_at ON ledger_entries(created_at);

-- Payouts (tenant-scoped)
CREATE TABLE payouts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    consignor_id UUID NOT NULL REFERENCES consignors(id),
    amount DECIMAL(10,2) NOT NULL,
    stripe_transfer_id VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL, -- Pending, Paid, Failed, Reversed
    payout_date TIMESTAMP NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_payouts_tenant_id ON payouts(tenant_id);
CREATE INDEX idx_payouts_consignor_id ON payouts(consignor_id);
CREATE INDEX idx_payouts_stripe_id ON payouts(stripe_transfer_id);

-- Rule configurations (tenant-scoped, uses JSONB)
CREATE TABLE rule_configurations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL, -- "Default Split Rule", "Markdown Schedule"
    rule_type VARCHAR(50) NOT NULL, -- split, fee, markdown, return
    configuration JSONB NOT NULL, -- Flexible JSON storage
    is_active BOOLEAN NOT NULL DEFAULT true,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP
);

CREATE INDEX idx_rules_tenant_id ON rule_configurations(tenant_id);
CREATE INDEX idx_rules_config_gin ON rule_configurations USING GIN (configuration); -- JSONB index
```

**Example JSONB Rule Configuration:**

```json
{
  "type": "split",
  "rules": [
    {
      "condition": "item_price >= 100",
      "consignor_percentage": 70,
      "store_percentage": 30
    },
    {
      "condition": "item_price < 100",
      "consignor_percentage": 60,
      "store_percentage": 40
    }
  ]
}
```

**Querying JSONB:**
```sql
-- Find all split rules with 70% consignor percentage
SELECT * FROM rule_configurations
WHERE rule_type = 'split'
  AND configuration @> '{"rules": [{"consignor_percentage": 70}]}'::jsonb;
```

### 6. Backup Strategy

**Automated Backups (Azure):**
- Point-in-time restore (35 days in production)
- Geo-redundant backups (production only)
- Automated daily snapshots

**Application-Level Exports:**
```bash
# Weekly CSV export of critical tables
pg_dump -h pg-vrd-202516-prd-eus2.postgres.database.azure.com \
  -U pgadmin \
  -d thriftconsignment \
  -t tenants -t consignors -t ledger_entries -t payouts \
  --format=custom \
  --file=backup-$(date +%Y%m%d).dump

# Upload to Azure Blob Storage (long-term retention)
az storage blob upload --account-name stvrd202516prdeus201 \
  --container-name backups \
  --file backup-$(date +%Y%m%d).dump
```

### 7. Performance Optimization

**Connection Pooling:**
```csharp
// appsettings.json
"ConnectionStrings": {
  "DefaultConnection": "Host=...;Pooling=true;Minimum Pool Size=5;Maximum Pool Size=100"
}
```

**Indexes (created above):**
- All `tenant_id` columns indexed (multi-tenant queries)
- Foreign keys indexed (JOIN performance)
- JSONB columns use GIN indexes (JSON query performance)

**Monitoring:**
- Enable Query Store in Azure portal
- Set up Application Insights for slow query alerts (>1 second)
- Monitor DTU/vCore usage (scale up if consistently >70%)

**Cost Estimate:**

**Development/Staging:**
- PostgreSQL Flexible B2s: $45/month
- 32 GB storage: $4/month
- **Total: ~$50/month**

**Production:**
- PostgreSQL Flexible D2s_v3: $145/month
- 128 GB storage: $16/month
- Zone-redundant HA: $145/month (doubles cost)
- Geo-redundant backups: $5/month
- **Total: ~$310/month**

**Optimization for solo founder:**
- Start with B2s in production (~$50/month)
- Upgrade to D2s_v3 + HA when revenue >$5k/month
- Add read replicas when analytics workload grows

---

**Next Steps:**
- Provision Azure PostgreSQL Flexible Server (dev environment)
- Store connection string in Azure Key Vault
- Install Npgsql EF Core provider
- Create initial EF Core models (Tenant, Consignor, Sale, LedgerEntry)
- Run first migration
- Test multi-tenant queries with Global Query Filters
- Set up Azure Monitor alerts for database metrics

---

**Documented by:** Claude Code
**Project Phase:** Planning - ADR 5 of 5
