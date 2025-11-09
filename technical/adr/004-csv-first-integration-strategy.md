# ADR-004: CSV-First POS Integration Strategy

**Date:** 2025-11-09
**Status:** Accepted
**Deciders:** Solo Founder
**Technical Story:** V1 Wedge Product - sales data ingestion

---

## Context

Thrift_Consignment needs sales data from stores' POS systems to calculate consignor payouts. The consignment software market is fragmented with 100+ POS systems, including:

**Major Players:**
- Square for Retail
- Shopify POS
- Lightspeed Retail
- Clover
- Vend (now Lightspeed)
- Legacy systems (ConsignPro, Liberty/Resaleworld, Ricochet, SimpleConsign)

**Integration Requirements:**
1. **Sales data:** Transaction date, item ID, sale amount, consignor ID, store fees
2. **Real-time preferred:** Enable instant consignor notifications
3. **Reliability:** No missed transactions (money on the line)
4. **Pilot timeline:** Need to onboard 3 pilots within 8 weeks
5. **Solo founder:** Limited time to build integrations
6. **Migration support:** Must import historical data (90 days) for validation

**Constraints:**
- POS webhooks require API approval (weeks/months)
- Each POS has different data formats
- Stores may use custom/legacy POS (no API)
- Complete build approach (must work reliably before launch)

---

## Decision

We will implement a **CSV-First Integration Strategy**:

1. **V1 (Weeks 1-8): CSV Import Only**
   - Build Excel template + mapping wizard
   - Support Square, Shopify, Lightspeed export formats
   - Manual upload via web interface
   - Daily or weekly batch processing

2. **V2 (Months 3-5): Add Webhooks (Incremental)**
   - Square for Retail real-time webhooks
   - Shopify POS real-time webhooks
   - Lightspeed real-time webhooks
   - Automatic fallback to CSV if webhooks fail

3. **V3 (Months 6+): Advanced Integrations**
   - Direct API polling (for POS without webhooks)
   - Zapier/Make.com integrations (no-code option)
   - Email-to-CSV parsing (forward export emails)

**Primary Flow (V1):** Store exports sales from POS → uploads CSV → system processes payouts

**Future Flow (V2+):** POS sends webhook → system processes in real-time → falls back to CSV if webhook fails

---

## Consequences

### Positive Consequences

- **Fast time-to-market:** CSV importer built in 1-2 weeks (vs months for webhooks)
- **Universal compatibility:** Works with ANY POS system (even legacy/custom)
- **Pilot-ready:** Can onboard 3 pilots immediately without waiting for API approvals
- **Historical import:** CSV enables 90-day lookback for validation
- **No vendor dependency:** Not blocked by POS API access or rate limits
- **Debugging:** Stores can visually inspect CSV before upload (catch errors early)
- **Fallback mechanism:** If webhooks fail in V2, CSV always works
- **Compliance:** CSV provides audit trail (can re-import to verify calculations)

### Negative Consequences

- **Manual work:** Store owners must export and upload (not automated)
  - **Mitigation:** V2 adds webhooks; V1 targets stores willing to do manual process
- **Not real-time:** Payout calculations delayed until CSV upload
  - **Mitigation:** Most stores pay consignors weekly/monthly anyway (not daily)
- **Data quality:** Typos, missing columns, formatting issues
  - **Mitigation:** Validation wizard catches errors before import
- **Training required:** Store owners need to learn export process
  - **Mitigation:** Video tutorials + screenshots for each POS system

### Neutral Consequences

- **Batch processing mindset:** System designed for periodic imports (not streaming)
- **Storage overhead:** CSV files stored in Azure Blob Storage

---

## Alternatives Considered

### Alternative 1: Webhooks-First (Build Top 3 POS Integrations Before Launch)

**Description:** Build Square, Shopify, Lightspeed webhooks in V1; require stores to use supported POS

**Pros:**
- **Real-time:** Instant consignor notifications when item sells
- **Automated:** Zero manual work for store owners
- **Modern:** Perceived as "cutting-edge" SaaS

**Cons:**
- **Time-to-market:** 6-12 weeks to build 3 integrations (vs 1-2 weeks CSV)
- **API approval delays:** Square/Shopify/Lightspeed review apps (2-6 weeks each)
- **Limited addressable market:** Only stores on supported POS can use us
- **Pilot risk:** If pilots use different POS, can't onboard them
- **No historical import:** Webhooks only capture new sales (can't validate with 90-day lookback)
- **Brittle:** Webhook failures break entire system (no fallback)

**Why rejected:** Delays pilots by 2-3 months; limits initial market; increases risk.

---

### Alternative 2: Direct API Polling (No Webhooks)

**Description:** Poll POS API every 5-15 minutes to fetch new sales

**Pros:**
- **Automated:** No manual uploads
- **Simpler than webhooks:** No need to handle webhook verification, retries
- **Works with POS that don't support webhooks**

**Cons:**
- **Rate limits:** POS APIs restrict polling frequency (e.g., 100 calls/hour)
- **Inefficient:** 99% of polls return "no new data"
- **Delayed:** 5-15 minute lag (not real-time)
- **API approval still needed:** Same approval delays as webhooks
- **Cost:** Azure Functions running every 5 minutes = $50-100/month

**Why rejected:** Doesn't solve time-to-market problem; rate limits are problematic; not much better than CSV.

---

### Alternative 3: Zapier/Make.com No-Code Integrations

**Description:** Use no-code platforms to connect POS → our API

**Pros:**
- **Fast setup:** Pre-built POS integrations
- **Store owner controlled:** They set up their own Zaps
- **No API approval needed:** Zapier already has POS access

**Cons:**
- **Extra cost:** Stores pay $20-50/month for Zapier Premium (on top of our $129/mo)
- **Complexity:** Store owners must learn Zapier (not trivial)
- **Reliability:** Zapier outages break payouts
- **Data mapping:** Still need to parse Zapier's output format
- **Limited control:** Can't debug Zapier issues ourselves

**Why rejected:** Too much complexity for small store owners; unreliable; adds cost.

---

### Alternative 4: Email-to-CSV Parsing

**Description:** Stores forward POS export emails to us; we extract CSV attachment and process

**Pros:**
- **Very simple for stores:** Just forward an email
- **Works with any POS:** If POS emails reports, we can parse them

**Cons:**
- **Unreliable:** Stores forget to forward, emails go to spam
- **Inconsistent formats:** Every POS's email format is different
- **No validation:** Can't show preview before import
- **Hard to debug:** "Did the email send? Did we receive it?"

**Why rejected:** Too fragile; poor user experience; hard to troubleshoot.

---

## References

- [Square Export Sales Documentation](https://squareup.com/help/us/en/article/5643-export-sales-and-items)
- [Shopify POS Export Guide](https://help.shopify.com/en/manual/orders/export-import-orders)
- [Lightspeed Retail Exports](https://retail-support.lightspeedhq.com/hc/en-us/articles/115004134668)
- [CSV Best Practices for Data Import](https://en.wikipedia.org/wiki/Comma-separated_values)
- [Azure Blob Storage Pricing](https://azure.microsoft.com/en-us/pricing/details/storage/blobs/)

---

## Notes

**Implementation Plan:**

### Phase 1: CSV Template Design (Week 1)

**Standard CSV Format:**
```csv
transaction_date,transaction_id,item_id,consignor_id,item_description,sale_price,store_fee,consignor_split,payment_method
2025-11-09,TXN-001,ITEM-12345,CSG-789,Vintage Blazer,45.00,9.00,36.00,credit_card
2025-11-09,TXN-002,ITEM-67890,CSG-456,Designer Handbag,120.00,36.00,84.00,cash
```

**Required Columns:**
- `transaction_date` (YYYY-MM-DD)
- `transaction_id` (unique per sale)
- `consignor_id` (matches consignor in system)
- `sale_price` (decimal)

**Optional Columns:**
- `item_id`, `item_description`, `store_fee`, `consignor_split`, `payment_method`

**POS-Specific Templates:**
- `template-square.xlsx` (matches Square export format)
- `template-shopify.xlsx` (matches Shopify export format)
- `template-lightspeed.xlsx` (matches Lightspeed export format)
- `template-generic.xlsx` (for custom POS)

### Phase 2: Mapping Wizard (Week 2)

**Frontend Flow:**
1. Store uploads CSV file
2. System parses headers and first 5 rows (preview)
3. Store maps columns to required fields:
   - "Which column is the sale date?" → Dropdown: `Date`, `Transaction Date`, `Sold On`
   - "Which column is the consignor ID?" → Dropdown: `Consignor`, `Vendor ID`, `Seller`
4. System validates mappings:
   - Date format recognized? (Try common formats: MM/DD/YYYY, YYYY-MM-DD)
   - Consignor IDs match existing records?
   - Sale prices are valid decimals?
5. Store saves mapping as template for future uploads
6. System shows summary: "Found 150 sales for 35 consignors totaling $12,450"
7. Store confirms import

**Backend (C# + CsvHelper):**
```csharp
public class CsvImportService
{
    public async Task<ImportPreview> PreviewCsv(Stream csvStream, Guid tenantId)
    {
        using var reader = new StreamReader(csvStream);
        using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

        var records = csv.GetRecords<dynamic>().Take(5).ToList();
        var headers = csv.HeaderRecord;

        return new ImportPreview
        {
            Headers = headers,
            SampleRows = records,
            DetectedFormat = DetectPosFormat(headers) // "Square", "Shopify", "Generic"
        };
    }

    public async Task<ImportResult> ImportCsv(Stream csvStream, ColumnMapping mapping, Guid tenantId)
    {
        using var reader = new StreamReader(csvStream);
        using var csv = new CsvReader(reader, CultureInfo.InvariantCulture);

        var sales = new List<Sale>();
        var errors = new List<string>();

        await foreach (var record in csv.GetRecordsAsync<dynamic>())
        {
            try
            {
                var sale = new Sale
                {
                    TenantId = tenantId,
                    TransactionDate = ParseDate(record[mapping.DateColumn]),
                    TransactionId = record[mapping.TransactionIdColumn],
                    ConsignorId = await LookupConsignorId(record[mapping.ConsignorIdColumn], tenantId),
                    SalePrice = decimal.Parse(record[mapping.SalePriceColumn]),
                    CreatedAt = DateTime.UtcNow
                };

                sales.Add(sale);
            }
            catch (Exception ex)
            {
                errors.Add($"Row {csv.Context.Row}: {ex.Message}");
            }
        }

        // Save to database
        if (errors.Count == 0)
        {
            await _db.Sales.AddRangeAsync(sales);
            await _db.SaveChangesAsync();
        }

        return new ImportResult
        {
            SuccessCount = sales.Count,
            ErrorCount = errors.Count,
            Errors = errors,
            TotalAmount = sales.Sum(s => s.SalePrice)
        };
    }
}
```

### Phase 3: Validation Rules (Week 2)

**Pre-Import Validation:**
1. **Duplicate detection:** Check if `transaction_id` already exists (prevent double imports)
2. **Date validation:** Reject dates in the future or >1 year old
3. **Consignor matching:** Warn if consignor IDs don't match (offer to create new consignors)
4. **Price validation:** Reject negative prices or prices >$100,000
5. **Required fields:** Reject rows missing date, transaction ID, or consignor ID

**Post-Import Reconciliation:**
```csharp
public class ReconciliationService
{
    public async Task<ReconciliationReport> Reconcile(Guid tenantId, DateTime startDate, DateTime endDate)
    {
        var sales = await _db.Sales
            .Where(s => s.TransactionDate >= startDate && s.TransactionDate <= endDate)
            .ToListAsync();

        var totalSales = sales.Sum(s => s.SalePrice);
        var consignorPayouts = await CalculatePayouts(sales);
        var storeFees = totalSales - consignorPayouts;

        return new ReconciliationReport
        {
            TotalSales = totalSales,
            ConsignorPayouts = consignorPayouts,
            StoreFees = storeFees,
            TransactionCount = sales.Count,
            ConsignorCount = sales.Select(s => s.ConsignorId).Distinct().Count()
        };
    }
}
```

### Phase 4: File Storage (Week 2)

**Azure Blob Storage:**
- Container: `csv-imports`
- Path: `{tenantId}/{year}/{month}/{filename}`
- Retention: 7 years (tax compliance)
- Access: Tenant-scoped SAS tokens

```csharp
public async Task<string> UploadCsv(Stream csvStream, string filename, Guid tenantId)
{
    var blobClient = _blobServiceClient.GetBlobContainerClient("csv-imports");
    var blobPath = $"{tenantId}/{DateTime.UtcNow:yyyy/MM}/{filename}";

    await blobClient.GetBlobClient(blobPath).UploadAsync(csvStream);

    return blobPath; // Store in database for audit trail
}
```

### Phase 5: User Documentation (Week 2)

**For each POS, create:**
1. **Video tutorial:** How to export sales (2-3 minutes)
2. **Screenshot guide:** Step-by-step export instructions
3. **Example CSV:** Sample file showing correct format
4. **Troubleshooting:** Common errors and fixes

**Example: Square Export Guide**
```markdown
# How to Export Sales from Square

1. Log into Square Dashboard (squareup.com/dashboard)
2. Go to **Reports** → **Sales Summary**
3. Select date range (e.g., last 7 days)
4. Click **Export** → **CSV**
5. Download file (e.g., `sales-2025-11-09.csv`)
6. Go to Thrift_Consignment → **Import Sales**
7. Upload CSV and review preview
8. Confirm import

**Troubleshooting:**
- "Consignor not found" → Check consignor ID format (Square uses customer ID)
- "Invalid date" → Ensure dates are MM/DD/YYYY format
```

### Future: Webhook Migration Path (V2)

**When adding webhooks:**
1. Detect if store's POS supports webhooks
2. Show prompt: "Want to enable automatic imports? Connect your Square account"
3. OAuth flow to authorize webhook access
4. Set up webhook endpoint: `POST /api/webhooks/square`
5. **Keep CSV as fallback:** If webhook fails 3 times, email store owner to upload CSV

**Gradual Migration:**
- Pilot stores use CSV (V1)
- New stores after Month 3 offered webhooks
- By Month 6, 80%+ on webhooks, 20% still using CSV (legacy POS)

---

**Cost Analysis:**

**CSV Storage (Azure Blob):**
- 100 stores × 4 CSV uploads/month × 1 MB = 400 MB/month
- Azure Blob Hot Tier: $0.018/GB = $0.007/month
- 7-year retention: 400 MB × 84 months = 33.6 GB = $0.60/month
- **Negligible cost**

**Processing Time:**
- Parse 1000-row CSV: ~2 seconds
- Validation + mapping: ~5 seconds
- Database insert: ~3 seconds
- **Total: ~10 seconds per import**

**Alternative (Webhook) Cost:**
- Azure Functions (consumption): ~$0.20/million executions
- 100 stores × 50 sales/day × 30 days = 150k webhook calls/month
- Cost: ~$0.03/month
- **Also negligible, but requires months to build**

**Decision:** CSV is fast, cheap, universally compatible. Perfect for V1.

---

**Next Steps:**
- Design CSV templates for Square, Shopify, Lightspeed
- Build mapping wizard UI (React)
- Implement CsvHelper parser and validation
- Set up Azure Blob Storage container
- Create video tutorials for top 3 POS systems
- Test with pilot stores' actual exports

---

**Documented by:** Claude Code
**Project Phase:** Planning - ADR 4 of 5
