# Sprint 1 - Foundation & CSV Import

**Sprint Duration:** 2 weeks (10 working days)
**Sprint Goal:** Establish technical foundation and ship CSV import capability
**Status:** Planning

---

## Sprint Goal

Build the core technical foundation for Thrift_Consignment and deliver the first piece of value: CSV sales import with column mapping wizard. By end of sprint, a store owner can upload a CSV export from their POS, map columns, and have sales data imported into the system ready for payout calculations.

**Success Criteria:**
- .NET solution scaffolded with EF Core + PostgreSQL
- Multi-tenant infrastructure working (subdomain routing, tenant isolation)
- CSV importer can parse Square, Shopify, and Lightspeed export formats
- Mapping wizard allows custom column configuration
- Sales data stored in database with proper tenant isolation
- All code deployed to Azure dev environment

---

## Sprint Capacity

**Available Days:** 10 working days
**Capacity:** ~60-80 hours (solo founder, complete build approach)
**Time Allocation:**
- 40% - Core infrastructure (solution, database, multi-tenancy)
- 35% - CSV importer + mapping wizard
- 15% - Azure deployment setup
- 10% - Testing, documentation, overhead

**Commitments/Time Off:** None

---

## Sprint Backlog

### High Priority (Must Complete)

| Story | Description | Estimate | Status | Notes |
|-------|-------------|----------|--------|-------|
| **US-001** | Set up .NET 8 solution structure | M | 📋 Todo | Backend API + EF Core |
| **US-002** | Configure Azure PostgreSQL dev database | S | 📋 Todo | Bicep or Azure CLI |
| **US-003** | Implement multi-tenant base entities | M | 📋 Todo | TenantEntity, GlobalEntity |
| **US-004** | Build tenant middleware (subdomain routing) | M | 📋 Todo | Parse subdomain, set context |
| **US-005** | Create EF Core global query filters | M | 📋 Todo | Auto-filter by TenantId |
| **US-006** | Design database schema (tenants, consignors, sales) | L | 📋 Todo | Migrations for core tables |
| **US-007** | Build CSV upload endpoint + Azure Blob storage | M | 📋 Todo | POST /api/csv/upload |
| **US-008** | Implement CSV parser with CsvHelper | M | 📋 Todo | Parse headers, sample rows |
| **US-009** | Build mapping wizard API | L | 📋 Todo | Column mapping, validation |
| **US-010** | Create sales import service | M | 📋 Todo | Bulk insert with validation |
| **US-011** | Build basic frontend (upload UI + mapping wizard) | L | 📋 Todo | Next.js minimal UI |
| **US-012** | Deploy to Azure App Service (dev) | M | 📋 Todo | CI/CD pipeline |

### Medium Priority (Should Complete)

| Story | Description | Estimate | Status | Notes |
|-------|-------------|----------|--------|-------|
| **US-013** | Add unit tests for tenant isolation | M | 📋 Todo | Prevent cross-tenant leaks |
| **US-014** | Create POS-specific CSV templates | S | 📋 Todo | Square, Shopify, Lightspeed |
| **US-015** | Add duplicate transaction detection | S | 📋 Todo | Check transaction_id exists |
| **US-016** | Build reconciliation report (basic) | S | 📋 Todo | Total sales, count by consignor |

### Low Priority (Nice to Have)

| Story | Description | Estimate | Status | Notes |
|-------|-------------|----------|--------|-------|
| **US-017** | Add CSV import history view | S | 📋 Todo | Show past imports |
| **US-018** | Error handling + retry logic | S | 📋 Todo | Graceful failures |
| **US-019** | API documentation (Swagger) | XS | 📋 Todo | Auto-generated docs |

**Story Status Legend:**
- 📋 Todo
- 🏗️ In Progress
- 👀 In Review
- ✅ Done
- ❌ Blocked

**Estimation:**
- XS: 1-2 hours
- S: 3-6 hours
- M: 1-2 days
- L: 2-3 days
- XL: 3-5 days

---

## Detailed User Stories

### US-001: Set up .NET 8 Solution Structure

**As a** developer
**I want** a well-structured .NET solution
**So that** I can build the backend API with best practices

**Acceptance Criteria:**
- [ ] .NET 8 solution created with ASP.NET Core Web API project
- [ ] Project structure follows clean architecture (Controllers, Services, Models, Data)
- [ ] Npgsql.EntityFrameworkCore.PostgreSQL package installed
- [ ] Application settings configured (connection strings, CORS, logging)
- [ ] Solution builds without errors

**Technical Notes:**
```bash
dotnet new webapi -n ThriftConsignment.Api
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Serilog.AspNetCore
dotnet add package Swashbuckle.AspNetCore
```

**Folder Structure:**
```
ThriftConsignment.Api/
├── Controllers/
├── Services/
│   ├── ICsvImportService.cs
│   └── CsvImportService.cs
├── Models/
│   ├── Entities/
│   │   ├── Tenant.cs
│   │   ├── Consignor.cs
│   │   └── Sale.cs
│   └── DTOs/
├── Data/
│   ├── ApplicationDbContext.cs
│   └── Migrations/
├── Middleware/
│   └── TenantMiddleware.cs
└── Program.cs
```

---

### US-002: Configure Azure PostgreSQL Dev Database

**As a** developer
**I want** a PostgreSQL database in Azure
**So that** I can develop and test with a production-like environment

**Acceptance Criteria:**
- [ ] Azure PostgreSQL Flexible Server provisioned (B2s tier)
- [ ] Firewall rules configured (allow dev machine IP + Azure services)
- [ ] Connection string stored in Azure Key Vault
- [ ] Database connection tested from local dev machine
- [ ] SSL/TLS enforced

**Azure CLI Commands:**
```bash
# Create resource group
az group create \
  --name rg-vrd-202516-dev-eus2-app \
  --location eastus2

# Create PostgreSQL server
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
  --high-availability Disabled

# Create database
az postgres flexible-server db create \
  --resource-group rg-vrd-202516-dev-eus2-app \
  --server-name pg-vrd-202516-dev-eus2 \
  --database-name thriftconsignment

# Add firewall rule for dev machine
az postgres flexible-server firewall-rule create \
  --resource-group rg-vrd-202516-dev-eus2-app \
  --name pg-vrd-202516-dev-eus2 \
  --rule-name allow-dev-machine \
  --start-ip-address <your-ip>
```

---

### US-003: Implement Multi-Tenant Base Entities

**As a** developer
**I want** base entity classes with tenant isolation built-in
**So that** all tenant-scoped data is automatically filtered

**Acceptance Criteria:**
- [ ] `TenantEntity` abstract class created (Id, TenantId, timestamps)
- [ ] `GlobalEntity` abstract class created (for non-tenant data)
- [ ] All tenant-scoped entities inherit from `TenantEntity`
- [ ] Unit tests verify TenantId is required

**Code:**
```csharp
public abstract class TenantEntity
{
    public Guid Id { get; set; }
    public Guid TenantId { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
}

public abstract class GlobalEntity
{
    public Guid Id { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class Tenant : GlobalEntity
{
    public string Subdomain { get; set; } = null!;
    public string StoreName { get; set; } = null!;
    public string OwnerEmail { get; set; } = null!;
    public SubscriptionStatus Status { get; set; }
}

public class Consignor : TenantEntity
{
    public string FirstName { get; set; } = null!;
    public string LastName { get; set; } = null!;
    public string Email { get; set; } = null!;
    public string? Phone { get; set; }
    public string? StripeConnectedAccountId { get; set; }
    public PayoutFrequency PayoutFrequency { get; set; }
    public DateTime? NextPayoutDate { get; set; }
}

public class Sale : TenantEntity
{
    public Guid ConsignorId { get; set; }
    public Consignor Consignor { get; set; } = null!;
    public string TransactionId { get; set; } = null!;
    public DateTime TransactionDate { get; set; }
    public decimal SalePrice { get; set; }
    public string? ItemDescription { get; set; }
}
```

---

### US-004: Build Tenant Middleware (Subdomain Routing)

**As a** system
**I want** to automatically identify the tenant from the subdomain
**So that** all requests are scoped to the correct store

**Acceptance Criteria:**
- [ ] `ITenantContext` interface created
- [ ] `TenantContext` service implemented (scoped lifetime)
- [ ] `TenantMiddleware` parses subdomain and looks up tenant
- [ ] 404 returned if subdomain not found
- [ ] Tenant context available in controllers/services
- [ ] Unit tests for subdomain parsing

**Middleware Code:**
```csharp
public class TenantMiddleware
{
    private readonly RequestDelegate _next;

    public async Task InvokeAsync(
        HttpContext context,
        ITenantContext tenantContext,
        ApplicationDbContext db)
    {
        var host = context.Request.Host.Value;
        var subdomain = ExtractSubdomain(host);

        if (string.IsNullOrEmpty(subdomain))
        {
            context.Response.StatusCode = 400;
            await context.Response.WriteAsync("Invalid subdomain");
            return;
        }

        var tenant = await db.Tenants
            .Where(t => t.Subdomain == subdomain)
            .FirstOrDefaultAsync();

        if (tenant == null)
        {
            context.Response.StatusCode = 404;
            await context.Response.WriteAsync("Store not found");
            return;
        }

        ((TenantContext)tenantContext).SetTenant(tenant.Id, subdomain);
        await _next(context);
    }

    private string? ExtractSubdomain(string host)
    {
        // bloomconsignment.thrift-consignment.com -> bloomconsignment
        // localhost:5000 -> null (handle in dev config)
        var parts = host.Split('.');
        return parts.Length >= 3 ? parts[0] : null;
    }
}
```

---

### US-005: Create EF Core Global Query Filters

**As a** developer
**I want** EF Core to automatically filter queries by TenantId
**So that** I cannot accidentally leak data across tenants

**Acceptance Criteria:**
- [ ] Global query filter applied to all `TenantEntity` subclasses
- [ ] Queries without tenant context throw exception (fail-safe)
- [ ] `IgnoreQueryFilters()` documented for admin queries
- [ ] Unit tests verify cross-tenant queries blocked

**DbContext Configuration:**
```csharp
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
    modelBuilder.Entity<Sale>().HasIndex(s => s.TenantId);
    modelBuilder.Entity<Sale>().HasIndex(s => new { s.TenantId, s.TransactionId }).IsUnique();
}
```

---

### US-006: Design Database Schema (Core Tables)

**As a** developer
**I want** a well-designed database schema
**So that** I can store sales, consignors, and ledger data efficiently

**Acceptance Criteria:**
- [ ] EF Core migration created for core tables (Tenants, Consignors, Sales)
- [ ] Indexes added (tenant_id, transaction_id, etc.)
- [ ] Foreign keys configured with proper cascade rules
- [ ] Migration applied to dev database
- [ ] Seed data added (1 test tenant)

**Migration Command:**
```bash
dotnet ef migrations add InitialSchema
dotnet ef database update
```

**Seed Data:**
```csharp
// In OnModelCreating or separate seeder
modelBuilder.Entity<Tenant>().HasData(new Tenant
{
    Id = Guid.Parse("11111111-1111-1111-1111-111111111111"),
    Subdomain = "demo",
    StoreName = "Demo Consignment Store",
    OwnerEmail = "demo@thrift-consignment.com",
    Status = SubscriptionStatus.Active,
    CreatedAt = DateTime.UtcNow
});
```

---

### US-007: Build CSV Upload Endpoint + Azure Blob Storage

**As a** store owner
**I want** to upload a CSV file via the web interface
**So that** I can import sales data

**Acceptance Criteria:**
- [ ] Azure Blob Storage container created (`csv-imports`)
- [ ] POST `/api/csv/upload` endpoint accepts multipart/form-data
- [ ] CSV file validated (max 10MB, .csv extension)
- [ ] File uploaded to Azure Blob with tenant-scoped path
- [ ] File metadata stored in database (UploadHistory table)
- [ ] API returns upload ID and blob URL

**API Endpoint:**
```csharp
[HttpPost("upload")]
public async Task<ActionResult<CsvUploadResponse>> UploadCsv(IFormFile file)
{
    if (file == null || file.Length == 0)
        return BadRequest("No file uploaded");

    if (!file.FileName.EndsWith(".csv"))
        return BadRequest("File must be CSV format");

    if (file.Length > 10 * 1024 * 1024) // 10MB
        return BadRequest("File too large (max 10MB)");

    var blobPath = $"{_tenantContext.TenantId}/{DateTime.UtcNow:yyyy/MM}/{file.FileName}";
    var blobClient = _blobContainerClient.GetBlobClient(blobPath);

    using var stream = file.OpenReadStream();
    await blobClient.UploadAsync(stream);

    var uploadHistory = new CsvUploadHistory
    {
        TenantId = _tenantContext.TenantId,
        FileName = file.FileName,
        BlobPath = blobPath,
        FileSizeBytes = file.Length,
        UploadedAt = DateTime.UtcNow
    };

    _db.CsvUploadHistory.Add(uploadHistory);
    await _db.SaveChangesAsync();

    return Ok(new CsvUploadResponse
    {
        UploadId = uploadHistory.Id,
        BlobUrl = blobClient.Uri.ToString()
    });
}
```

---

### US-008: Implement CSV Parser with CsvHelper

**As a** system
**I want** to parse CSV files and preview headers + sample rows
**So that** the store owner can map columns

**Acceptance Criteria:**
- [ ] CsvHelper package installed
- [ ] GET `/api/csv/{uploadId}/preview` returns headers and first 5 rows
- [ ] Auto-detect POS format (Square, Shopify, Lightspeed) based on headers
- [ ] Handle common encoding issues (UTF-8, UTF-16, Windows-1252)
- [ ] Return sample data as JSON for frontend

**Service Implementation:**
```csharp
public async Task<CsvPreview> PreviewCsv(Guid uploadId)
{
    var upload = await _db.CsvUploadHistory.FindAsync(uploadId);
    if (upload == null)
        throw new NotFoundException("CSV upload not found");

    var blobClient = _blobContainerClient.GetBlobClient(upload.BlobPath);
    var stream = await blobClient.OpenReadAsync();

    using var reader = new StreamReader(stream);
    using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

    var records = csv.GetRecords<dynamic>().Take(5).ToList();
    var headers = csv.HeaderRecord;

    return new CsvPreview
    {
        UploadId = uploadId,
        Headers = headers.ToList(),
        SampleRows = records,
        DetectedFormat = DetectPosFormat(headers)
    };
}

private string DetectPosFormat(string[] headers)
{
    if (headers.Contains("Transaction ID") && headers.Contains("Net Sales"))
        return "Square";
    if (headers.Contains("Name") && headers.Contains("Lineitem price"))
        return "Shopify";
    if (headers.Contains("Sale ID") && headers.Contains("Customer"))
        return "Lightspeed";
    return "Generic";
}
```

---

### US-009: Build Mapping Wizard API

**As a** store owner
**I want** to map my CSV columns to the system's expected fields
**So that** sales data is imported correctly

**Acceptance Criteria:**
- [ ] POST `/api/csv/{uploadId}/map` saves column mappings
- [ ] Mapping stored in `CsvColumnMapping` table (per tenant)
- [ ] Validation: Required fields (date, consignor_id, sale_price) must be mapped
- [ ] Optional fields (item_description, transaction_id) can be skipped
- [ ] Saved mappings reusable for future imports

**Mapping Model:**
```csharp
public class CsvColumnMapping
{
    public Guid Id { get; set; }
    public Guid TenantId { get; set; }
    public string MappingName { get; set; } = null!; // "Square Default", "Custom"
    public string DateColumn { get; set; } = null!;
    public string ConsignorIdColumn { get; set; } = null!;
    public string SalePriceColumn { get; set; } = null!;
    public string? TransactionIdColumn { get; set; }
    public string? ItemDescriptionColumn { get; set; }
    public DateTime CreatedAt { get; set; }
}

[HttpPost("{uploadId}/map")]
public async Task<ActionResult<CsvColumnMapping>> SaveMapping(
    Guid uploadId,
    [FromBody] SaveMappingRequest request)
{
    // Validate required fields mapped
    if (string.IsNullOrEmpty(request.DateColumn) ||
        string.IsNullOrEmpty(request.ConsignorIdColumn) ||
        string.IsNullOrEmpty(request.SalePriceColumn))
    {
        return BadRequest("Required fields must be mapped: date, consignor_id, sale_price");
    }

    var mapping = new CsvColumnMapping
    {
        TenantId = _tenantContext.TenantId,
        MappingName = request.MappingName,
        DateColumn = request.DateColumn,
        ConsignorIdColumn = request.ConsignorIdColumn,
        SalePriceColumn = request.SalePriceColumn,
        TransactionIdColumn = request.TransactionIdColumn,
        ItemDescriptionColumn = request.ItemDescriptionColumn,
        CreatedAt = DateTime.UtcNow
    };

    _db.CsvColumnMappings.Add(mapping);
    await _db.SaveChangesAsync();

    return Ok(mapping);
}
```

---

### US-010: Create Sales Import Service

**As a** system
**I want** to import mapped CSV data into the Sales table
**So that** sales are ready for payout calculations

**Acceptance Criteria:**
- [ ] POST `/api/csv/{uploadId}/import` triggers import with mappingId
- [ ] Parse all rows using saved mapping
- [ ] Validate data (dates, decimals, consignor IDs exist)
- [ ] Detect duplicates (skip if transaction_id already exists)
- [ ] Bulk insert sales (batch of 1000 rows at a time)
- [ ] Return summary (total rows, imported, skipped, errors)

**Import Service:**
```csharp
public async Task<ImportResult> ImportCsv(Guid uploadId, Guid mappingId)
{
    var upload = await _db.CsvUploadHistory.FindAsync(uploadId);
    var mapping = await _db.CsvColumnMappings.FindAsync(mappingId);

    var blobClient = _blobContainerClient.GetBlobClient(upload.BlobPath);
    var stream = await blobClient.OpenReadAsync();

    using var reader = new StreamReader(stream);
    using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

    var sales = new List<Sale>();
    var errors = new List<string>();
    var skipped = 0;

    await foreach (var record in csv.GetRecordsAsync<dynamic>())
    {
        try
        {
            var transactionId = mapping.TransactionIdColumn != null
                ? record[mapping.TransactionIdColumn]
                : $"CSV-{uploadId}-{csv.Context.Row}";

            // Check for duplicate
            if (await _db.Sales.AnyAsync(s => s.TransactionId == transactionId))
            {
                skipped++;
                continue;
            }

            var consignorId = record[mapping.ConsignorIdColumn];
            var consignor = await _db.Consignors
                .Where(c => c.Email == consignorId || c.Id.ToString() == consignorId)
                .FirstOrDefaultAsync();

            if (consignor == null)
            {
                errors.Add($"Row {csv.Context.Row}: Consignor '{consignorId}' not found");
                continue;
            }

            var sale = new Sale
            {
                TenantId = _tenantContext.TenantId,
                ConsignorId = consignor.Id,
                TransactionId = transactionId,
                TransactionDate = ParseDate(record[mapping.DateColumn]),
                SalePrice = decimal.Parse(record[mapping.SalePriceColumn]),
                ItemDescription = mapping.ItemDescriptionColumn != null
                    ? record[mapping.ItemDescriptionColumn]
                    : null,
                CreatedAt = DateTime.UtcNow
            };

            sales.Add(sale);

            // Bulk insert every 1000 rows
            if (sales.Count >= 1000)
            {
                await _db.Sales.AddRangeAsync(sales);
                await _db.SaveChangesAsync();
                sales.Clear();
            }
        }
        catch (Exception ex)
        {
            errors.Add($"Row {csv.Context.Row}: {ex.Message}");
        }
    }

    // Insert remaining
    if (sales.Any())
    {
        await _db.Sales.AddRangeAsync(sales);
        await _db.SaveChangesAsync();
    }

    return new ImportResult
    {
        TotalRows = csv.Context.Row - 1,
        ImportedCount = sales.Count,
        SkippedCount = skipped,
        ErrorCount = errors.Count,
        Errors = errors
    };
}
```

---

## Technical Debt / Maintenance

Items to address if time permits:

- [ ] Set up automated database backups (Azure Backup configured)
- [ ] Add Application Insights for logging/monitoring
- [ ] Configure CORS for frontend (localhost:3016 for dev)
- [ ] Add health check endpoint (`/health`)
- [ ] Set up GitHub Actions CI (build + test on PR)

---

## Definition of Done

Sprint considered complete when:

1. **All High Priority stories are ✅ Done**
2. **Code deployed to Azure dev environment and tested**
3. **End-to-end test passes:** Upload CSV → preview → map → import → verify sales in database
4. **No P0 bugs** (blockers) in dev environment
5. **Code reviewed** (self-review with checklist)
6. **Basic documentation** written (README with setup instructions)

---

## Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Azure PostgreSQL setup delays | Medium | High | Provision database first (Day 1) to unblock development |
| CSV parsing edge cases | High | Medium | Test with real POS exports from Square/Shopify early |
| EF Core global filters not working | Low | Critical | Write unit tests first (TDD approach) |
| Multi-tenant middleware bugs | Medium | Critical | Extensive testing with multiple subdomains |
| Blob storage upload failures | Low | Medium | Add retry logic with exponential backoff |

---

## Success Metrics

At end of Sprint 1, we should have:

1. **Backend deployed:** Azure App Service running .NET API
2. **Database provisioned:** PostgreSQL with core schema
3. **CSV import working:** Able to import 1000+ row CSV in <30 seconds
4. **Multi-tenancy proven:** Can switch between 2+ test tenants via subdomain
5. **Zero data leakage:** Unit tests confirm cross-tenant queries blocked

**Demo-Ready:** Can show a store owner: "Upload your Square export, map the columns, and your sales are imported."

---

## Daily Progress

(To be filled during sprint execution)

### Day 1 - [Date]
**What I worked on:**
-

**Blockers:**
-

**Plan for tomorrow:**
-

---

### Day 2 - [Date]
**What I worked on:**
-

**Blockers:**
-

**Plan for tomorrow:**
-

---

(Continue for 10 days...)

---

**Documented by:** Claude Code
**Project Phase:** Planning - Task 5 of 5 COMPLETE
