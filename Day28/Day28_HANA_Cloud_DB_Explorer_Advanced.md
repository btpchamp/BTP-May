# Day 28: SAP HANA Cloud — Database Explorer & Advanced Features

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 27 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Database Explorer — Your HANA Command Center | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: SQL Console — Querying HANA Cloud | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Explore Tables & Run SQL | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: HANA-Specific Features & Native Artifacts | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Deployment Logs & Troubleshooting | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Deploy EPM Model to HANA Cloud | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Navigate the SAP HANA Database Explorer confidently
- Run SQL queries (SELECT, INSERT, UPDATE, DELETE) on HANA Cloud
- Understand HANA-specific data types and how CDS maps to them
- Know what native HANA artifacts (Calculation Views) are and when they're used
- Read deployment logs and identify common errors
- Troubleshoot failed HANA deployments
- Deploy your complete EPM model to HANA Cloud and verify the data

---

## Day 27 Recap — Quick Fire (09:00 - 09:15)

1. HANA stores data in _____ instead of on disk? → _____
2. Column store is better for _____ queries (SUM, AVG)? → _____
3. HDI stands for _____? → _____
4. An HDI Container provides _____ between applications? → _____
5. `cds add hana` adds what to your project? → _____
6. `cds build --production` generates what files? → _____
7. In BTP Trial, HANA instances _____ after inactivity? → _____
8. Same CDS code works with both _____ and _____? → _____

<details>
<summary>Answers</summary>

1. **RAM / Memory** (in-memory database)
2. **Analytical / Aggregation** queries
3. **HANA Deployment Infrastructure**
4. **Isolation** (each app's data is separate)
5. HANA configuration to `package.json` + dependencies (`hdb`, `@sap/hdi-deploy`)
6. `.hdbtable` and `.hdbview` files (HANA design-time artifacts)
7. **Auto-stop** (must restart from BTP Cockpit)
8. **SQLite** (dev) and **HANA Cloud** (production)

</details>

---

## Session 1: Database Explorer — Your HANA Command Center (09:15 - 10:30)

### What is the SAP HANA Database Explorer?

The **Database Explorer** is a web-based tool for working with your HANA Cloud database. Think of it as "your database's control panel" — you can browse tables, run queries, view data, check performance, and more.

```
┌─────────────────────────────────────────────────────────────┐
│  SAP HANA Database Explorer                                  │
│                                                             │
│  Like having a window directly into your database:          │
│  • See all tables and their structures                      │
│  • View the actual data stored                              │
│  • Run SQL queries (SELECT, INSERT, UPDATE, DELETE)          │
│  • Check indexes, views, and procedures                     │
│  • Monitor performance                                      │
│  • Export/import data                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### How to Open Database Explorer

**Method 1: From BTP Cockpit**
1. BTP Cockpit → SAP HANA Cloud → Your instance
2. Click "..." (actions) → "Open in SAP HANA Database Explorer"

**Method 2: From SAP Business Application Studio**
1. In BAS, left panel → SAP HANA icon
2. Click "Open Database Explorer"

**Method 3: Direct URL**
```
https://hana-cockpit.cfapps.{region}.hana.ondemand.com/sap/hana/cst/catalog/index.html
```

---

### Database Explorer Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  SAP HANA Database Explorer                                      │
├───────────────────┬─────────────────────────────────────────────┤
│                   │                                             │
│  ① DATABASES      │  ③ CONTENT AREA                            │
│  ┌─────────────┐  │  ┌─────────────────────────────────────┐   │
│  │ 📁 hdi-cont │  │  │  Table: COM_EPM_PRODUCTS             │   │
│  │   ├ Tables  │  │  │                                     │   │
│  │   ├ Views   │  │  │  Columns:                           │   │
│  │   ├ Indexes │  │  │  ID         | NVARCHAR(36) | PK     │   │
│  │   ├ Procs   │  │  │  PRODUCTNAME| NVARCHAR(100)         │   │
│  │   └ Triggers│  │  │  PRICE      | DECIMAL(10,2)         │   │
│  └─────────────┘  │  │  STOCK      | INTEGER               │   │
│                   │  │  ...                                 │   │
│  ② CATALOG NAV    │  └─────────────────────────────────────┘   │
│  ┌─────────────┐  │                                             │
│  │ Search: [__]│  │  ④ SQL CONSOLE (tab at top)                │
│  │ Filter by   │  │  ┌─────────────────────────────────────┐   │
│  │ schema      │  │  │ SELECT * FROM COM_EPM_PRODUCTS       │   │
│  └─────────────┘  │  │ WHERE STOCK < 10;                    │   │
│                   │  │                          [Run ▶]     │   │
│                   │  └─────────────────────────────────────┘   │
│                   │                                             │
└───────────────────┴─────────────────────────────────────────────┘
```

---

### Navigating the Catalog

When you open Database Explorer connected to your HDI container:

```
📁 Your HDI Container
├── 📋 Tables
│   ├── COM_EPM_PRODUCTS
│   ├── COM_EPM_PURCHASEORDERS
│   ├── COM_EPM_PURCHASEORDERITEMS
│   ├── COM_EPM_SUPPLIERS
│   ├── COM_EPM_CATEGORIES
│   ├── COM_EPM_SALESORDERS
│   ├── COM_EPM_SALESORDERITEMS
│   └── COM_EPM_CUSTOMERS
├── 👁️ Views
│   ├── COM_EPM_PRODUCTCATALOG
│   ├── COM_EPM_ORDERSUMMARY
│   └── COM_EPM_LOWSTOCKPRODUCTS
├── 📊 Column Views (system-generated)
├── 🔑 Indexes
├── ⚙️ Procedures
└── 📌 Sequences
```

---

### What You Can Do With Each Object

| Object Type | Right-Click Actions |
|-------------|-------------------|
| **Table** | Open Data, Open Definition, Generate SELECT, Generate INSERT, Export Data |
| **View** | Open Data, Open Definition, Check Dependencies |
| **Procedure** | Open Definition, Generate CALL statement |
| **Index** | View columns, check status |

---

### Viewing Table Structure

Right-click a table → "Open Definition":

```
Table: COM_EPM_PRODUCTS
┌──────────────┬──────────────────┬──────────┬─────────┬─────────┐
│ Column Name  │ Data Type        │ Nullable │ Default │ Key     │
├──────────────┼──────────────────┼──────────┼─────────┼─────────┤
│ ID           │ NVARCHAR(36)     │ No       │         │ PK ✓    │
│ PRODUCTNAME  │ NVARCHAR(100)    │ Yes      │         │         │
│ DESCRIPTION  │ NVARCHAR(500)    │ Yes      │         │         │
│ PRICE        │ DECIMAL(10,2)    │ Yes      │         │         │
│ STOCK        │ INTEGER          │ Yes      │ 0       │         │
│ MINSTOCK     │ INTEGER          │ Yes      │ 10      │         │
│ RATING       │ DECIMAL(2,1)     │ Yes      │         │         │
│ ISAVAILABLE  │ BOOLEAN          │ Yes      │ true    │         │
│ SUPPLIER_ID  │ NVARCHAR(36)     │ Yes      │         │ FK      │
│ CATEGORY_ID  │ NVARCHAR(36)     │ Yes      │         │ FK      │
│ CREATEDAT    │ TIMESTAMP        │ Yes      │         │         │
│ CREATEDBY    │ NVARCHAR(255)    │ Yes      │         │         │
│ MODIFIEDAT   │ TIMESTAMP        │ Yes      │         │         │
│ MODIFIEDBY   │ NVARCHAR(255)    │ Yes      │         │         │
└──────────────┴──────────────────┴──────────┴─────────┴─────────┘
```

---

### Viewing Data (Quick Look)

Right-click a table → "Open Data":

```
┌────────────────────────────────────────────────────────────────┐
│  COM_EPM_PRODUCTS (5 rows)                                      │
├──────────────────────┬─────────┬───────┬────────┬──────────────┤
│ PRODUCTNAME          │ PRICE   │ STOCK │ RATING │ SUPPLIER_ID  │
├──────────────────────┼─────────┼───────┼────────┼──────────────┤
│ Laptop Pro           │ 999.00  │ 45    │ 4.8    │ s1000001-... │
│ Wireless Mouse       │ 29.99   │ 120   │ 4.5    │ s1000001-... │
│ USB-C Hub            │ 49.99   │ 80    │ 4.2    │ s1000002-... │
│ Mechanical Keyboard  │ 89.99   │ 30    │ 4.7    │ s1000001-... │
│ 4K Monitor           │ 449.99  │ 15    │ 4.6    │ s1000002-... │
└──────────────────────┴─────────┴───────┴────────┴──────────────┘
```

---

## Session 2: SQL Console — Querying HANA Cloud (10:45 - 12:00)

### Opening the SQL Console

1. In Database Explorer, click the **"SQL Console"** icon (top toolbar)
2. Or: Right-click your database connection → "Open SQL Console"
3. A new tab opens with an editor where you type SQL

---

### HANA SQL vs Standard SQL

HANA SQL is mostly standard SQL with some extras. If you know basic SQL, you're 90% there!

```sql
-- These work exactly like any SQL database:
SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER
WHERE, AND, OR, NOT, IN, BETWEEN, LIKE
ORDER BY, GROUP BY, HAVING
JOIN (INNER, LEFT, RIGHT)
COUNT, SUM, AVG, MIN, MAX
```

**HANA-specific additions:**
```sql
-- String functions (HANA-specific names)
SELECT UPPER(productname) FROM com_epm_products;
SELECT LOWER(productname) FROM com_epm_products;
SELECT LENGTH(productname) FROM com_epm_products;
SELECT SUBSTR(productname, 1, 5) FROM com_epm_products;

-- Date functions
SELECT CURRENT_DATE FROM DUMMY;                 -- Today's date
SELECT CURRENT_TIMESTAMP FROM DUMMY;            -- Now with time
SELECT ADD_DAYS(CURRENT_DATE, 30) FROM DUMMY;   -- 30 days from now
SELECT DAYS_BETWEEN('2026-01-01', CURRENT_DATE) FROM DUMMY;  -- Days since Jan 1

-- The special DUMMY table (for calculations without a real table)
SELECT 100 * 0.18 AS tax FROM DUMMY;            -- Quick calculation
```

---

### SELECT Queries — Retrieve Data

```sql
-- All products
SELECT * FROM COM_EPM_PRODUCTS;

-- Specific columns
SELECT PRODUCTNAME, PRICE, STOCK FROM COM_EPM_PRODUCTS;

-- With filter
SELECT PRODUCTNAME, STOCK
FROM COM_EPM_PRODUCTS
WHERE STOCK < 20;

-- With sorting
SELECT PRODUCTNAME, PRICE
FROM COM_EPM_PRODUCTS
ORDER BY PRICE DESC;

-- Top N results
SELECT TOP 5 PRODUCTNAME, RATING
FROM COM_EPM_PRODUCTS
ORDER BY RATING DESC;

-- Count
SELECT COUNT(*) AS total_products FROM COM_EPM_PRODUCTS;

-- Aggregation
SELECT
  CATEGORY_ID,
  COUNT(*) AS product_count,
  AVG(PRICE) AS avg_price,
  SUM(STOCK) AS total_stock
FROM COM_EPM_PRODUCTS
GROUP BY CATEGORY_ID;
```

---

### JOIN Queries — Combine Tables

```sql
-- Products with supplier name
SELECT
  p.PRODUCTNAME,
  p.PRICE,
  p.STOCK,
  s.SUPPLIERNAME
FROM COM_EPM_PRODUCTS p
INNER JOIN COM_EPM_SUPPLIERS s ON p.SUPPLIER_ID = s.ID;

-- Orders with customer name and item count
SELECT
  o.ORDERNUMBER,
  o.ORDERDATE,
  o.GROSSAMOUNT,
  c.CUSTOMERNAME,
  COUNT(i.ID) AS item_count
FROM COM_EPM_SALESORDERS o
JOIN COM_EPM_CUSTOMERS c ON o.CUSTOMER_ID = c.ID
LEFT JOIN COM_EPM_SALESORDERITEMS i ON i.ORDER_ID = o.ID
GROUP BY o.ORDERNUMBER, o.ORDERDATE, o.GROSSAMOUNT, c.CUSTOMERNAME
ORDER BY o.ORDERDATE DESC;

-- Products with category and supplier (3-table join)
SELECT
  p.PRODUCTNAME,
  p.PRICE,
  cat.CATEGORYNAME,
  s.SUPPLIERNAME
FROM COM_EPM_PRODUCTS p
JOIN COM_EPM_CATEGORIES cat ON p.CATEGORY_ID = cat.ID
JOIN COM_EPM_SUPPLIERS s ON p.SUPPLIER_ID = s.ID
ORDER BY cat.CATEGORYNAME, p.PRODUCTNAME;
```

---

### INSERT — Add New Data

```sql
-- Insert a new supplier
INSERT INTO COM_EPM_SUPPLIERS (ID, SUPPLIERNAME, EMAIL, CITY, ISACTIVE, CREATEDAT, CREATEDBY)
VALUES (
  'sup-new-001-0000-000000000001',
  'New Supplier Co.',
  'contact@newsupplier.com',
  'Mumbai',
  TRUE,
  CURRENT_TIMESTAMP,
  'admin'
);

-- Insert a product
INSERT INTO COM_EPM_PRODUCTS (ID, PRODUCTNAME, PRICE, STOCK, MINSTOCK, CREATEDAT, CREATEDBY)
VALUES (
  'prd-new-001-0000-000000000001',
  'Bluetooth Speaker',
  79.99,
  50,
  15,
  CURRENT_TIMESTAMP,
  'admin'
);

-- Verify
SELECT * FROM COM_EPM_PRODUCTS WHERE PRODUCTNAME = 'Bluetooth Speaker';
```

---

### UPDATE — Modify Existing Data

```sql
-- Update product stock
UPDATE COM_EPM_PRODUCTS
SET STOCK = 200, MODIFIEDAT = CURRENT_TIMESTAMP, MODIFIEDBY = 'admin'
WHERE PRODUCTNAME = 'Wireless Mouse';

-- Increase all prices by 10%
UPDATE COM_EPM_PRODUCTS
SET PRICE = PRICE * 1.10, MODIFIEDAT = CURRENT_TIMESTAMP
WHERE PRICE < 100;

-- Change order status
UPDATE COM_EPM_SALESORDERS
SET STATUS = 'Confirmed', MODIFIEDAT = CURRENT_TIMESTAMP
WHERE ORDERNUMBER = 'SO-001';

-- Verify
SELECT PRODUCTNAME, PRICE FROM COM_EPM_PRODUCTS WHERE PRICE < 100;
```

---

### DELETE — Remove Data

```sql
-- Delete a specific product
DELETE FROM COM_EPM_PRODUCTS
WHERE ID = 'prd-new-001-0000-000000000001';

-- Delete all cancelled orders (careful!)
DELETE FROM COM_EPM_SALESORDERS
WHERE STATUS = 'Cancelled';

-- ALWAYS verify with SELECT first:
SELECT * FROM COM_EPM_SALESORDERS WHERE STATUS = 'Cancelled';
-- If that looks right, THEN delete.
```

**Safety tip:** Always run a `SELECT` with the same `WHERE` clause BEFORE running `DELETE` to see what will be removed!

---

### Useful HANA-Specific Queries

```sql
-- Check table row count (fast way)
SELECT TABLE_NAME, RECORD_COUNT
FROM M_TABLES
WHERE SCHEMA_NAME = CURRENT_SCHEMA
ORDER BY RECORD_COUNT DESC;

-- Check table size in memory
SELECT TABLE_NAME, MEMORY_SIZE_IN_TOTAL / 1024 / 1024 AS size_mb
FROM M_TABLES
WHERE SCHEMA_NAME = CURRENT_SCHEMA
ORDER BY MEMORY_SIZE_IN_TOTAL DESC;

-- Check which columns exist in a table
SELECT COLUMN_NAME, DATA_TYPE_NAME, LENGTH, IS_NULLABLE
FROM TABLE_COLUMNS
WHERE SCHEMA_NAME = CURRENT_SCHEMA
AND TABLE_NAME = 'COM_EPM_PRODUCTS'
ORDER BY POSITION;

-- Current date/time and calculations
SELECT
  CURRENT_DATE AS today,
  CURRENT_TIMESTAMP AS now,
  ADD_DAYS(CURRENT_DATE, 7) AS next_week,
  ADD_MONTHS(CURRENT_DATE, 1) AS next_month
FROM DUMMY;
```

---

## Session 3: Hands-on — Explore Tables & Run SQL (12:00 - 13:00)

### Exercise 1: Browse Your Deployed Tables (15 minutes)

After deploying to HANA, open Database Explorer and complete these tasks:

1. **Find all tables** in your HDI container. How many are there? _____
2. **Open the Products table definition.** What is the HANA data type for `price`? _____
3. **Open the Products data.** How many rows exist? _____
4. **Find the foreign keys.** What column links Products to Suppliers? _____
5. **Check the Views.** Are your CDS views (ProductCatalog, etc.) deployed? _____

---

### Exercise 2: Run Basic Queries (15 minutes)

Open the SQL Console and run:

```sql
-- Task 1: Count all products
SELECT COUNT(*) AS total FROM COM_EPM_PRODUCTS;

-- Task 2: Find the most expensive product
SELECT TOP 1 PRODUCTNAME, PRICE
FROM COM_EPM_PRODUCTS
ORDER BY PRICE DESC;

-- Task 3: Products below minimum stock
SELECT PRODUCTNAME, STOCK, MINSTOCK, (MINSTOCK - STOCK) AS deficit
FROM COM_EPM_PRODUCTS
WHERE STOCK < MINSTOCK;

-- Task 4: Total inventory value (price × stock)
SELECT SUM(PRICE * STOCK) AS total_inventory_value
FROM COM_EPM_PRODUCTS;

-- Task 5: Products per category (with names)
SELECT cat.CATEGORYNAME, COUNT(*) AS product_count, AVG(p.PRICE) AS avg_price
FROM COM_EPM_PRODUCTS p
JOIN COM_EPM_CATEGORIES cat ON p.CATEGORY_ID = cat.ID
GROUP BY cat.CATEGORYNAME;
```

---

### Exercise 3: Modify Data via SQL (15 minutes)

```sql
-- Task 1: Insert a new category
INSERT INTO COM_EPM_CATEGORIES (ID, CATEGORYNAME, DESCRIPTION, CREATEDAT, CREATEDBY)
VALUES ('cat-sql-001', 'Audio Equipment', 'Speakers and headphones', CURRENT_TIMESTAMP, 'admin');

-- Task 2: Update a product's stock (simulate receiving inventory)
UPDATE COM_EPM_PRODUCTS
SET STOCK = STOCK + 50, MODIFIEDAT = CURRENT_TIMESTAMP
WHERE PRODUCTNAME = 'Wireless Mouse';

-- Task 3: Verify your changes
SELECT PRODUCTNAME, STOCK FROM COM_EPM_PRODUCTS WHERE PRODUCTNAME = 'Wireless Mouse';
SELECT * FROM COM_EPM_CATEGORIES WHERE ID = 'cat-sql-001';

-- Task 4: Clean up (delete your test data)
DELETE FROM COM_EPM_CATEGORIES WHERE ID = 'cat-sql-001';
```

---

### Exercise 4: Write Analytical Queries (15 minutes)

```sql
-- Business Question 1: "What's our revenue by month?"
SELECT
  YEAR(ORDERDATE) AS year,
  MONTH(ORDERDATE) AS month,
  COUNT(*) AS order_count,
  SUM(GROSSAMOUNT) AS total_revenue
FROM COM_EPM_SALESORDERS
WHERE STATUS = 'Delivered'
GROUP BY YEAR(ORDERDATE), MONTH(ORDERDATE)
ORDER BY year, month;

-- Business Question 2: "Who are our top 5 customers by spending?"
SELECT TOP 5
  c.CUSTOMERNAME,
  COUNT(o.ID) AS total_orders,
  SUM(o.GROSSAMOUNT) AS total_spent
FROM COM_EPM_CUSTOMERS c
JOIN COM_EPM_SALESORDERS o ON o.CUSTOMER_ID = c.ID
GROUP BY c.CUSTOMERNAME
ORDER BY total_spent DESC;

-- Business Question 3: "Which suppliers have the most products?"
SELECT
  s.SUPPLIERNAME,
  s.CITY,
  COUNT(p.ID) AS product_count,
  SUM(p.STOCK) AS total_stock
FROM COM_EPM_SUPPLIERS s
LEFT JOIN COM_EPM_PRODUCTS p ON p.SUPPLIER_ID = s.ID
GROUP BY s.SUPPLIERNAME, s.CITY
ORDER BY product_count DESC;
```

---

## Session 4: HANA-Specific Features & Native Artifacts (13:45 - 14:45)

### CDS to HANA Type Mapping

When CAP builds for HANA, CDS types become HANA types:

| CDS Type | HANA Type | Notes |
|----------|-----------|-------|
| `String(n)` | `NVARCHAR(n)` | Unicode text, max 5000 |
| `LargeString` | `NCLOB` | Unlimited text |
| `Integer` | `INTEGER` | 32-bit integer |
| `Integer64` | `BIGINT` | 64-bit integer |
| `Decimal(p,s)` | `DECIMAL(p,s)` | Exact numeric |
| `Double` | `DOUBLE` | Floating point |
| `Boolean` | `BOOLEAN` | true/false |
| `Date` | `DATE` | Calendar date |
| `Time` | `TIME` | Time of day |
| `DateTime` | `TIMESTAMP` | Date + time |
| `Timestamp` | `TIMESTAMP` | Same as DateTime |
| `UUID` | `NVARCHAR(36)` | Stored as string |
| `LargeBinary` | `BLOB` | Binary data |

**Key insight:** `UUID` is NOT a special type in HANA — it's just a 36-character string (`NVARCHAR(36)`).

---

### HANA-Specific Features Available in CAP

| Feature | What It Is | How to Use in CAP |
|---------|------------|-------------------|
| **Fuzzy Search** | Approximate text matching | `@Search.defaultSearchElement` annotation |
| **Full-Text Index** | Fast text search on large fields | Automatic for `@Search` annotated fields |
| **Spatial** | Geographic data (points, areas) | `hana.ST_POINT` type |
| **Calculated Columns** | Computed at DB level (fast!) | CDS views or `@cds.persistence.calc` |
| **Partitioning** | Split large tables for performance | HANA native config |
| **Compression** | Automatic column compression | Built-in (no config needed) |

---

### Native HANA Artifacts — Calculation Views

**Calculation Views** are HANA's powerful analytical views — they combine, aggregate, and transform data visually. They're the "power tool" for complex reporting.

```
┌─────────────────────────────────────────────────────────────┐
│  CALCULATION VIEW — Visual Data Pipeline                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────┐    ┌──────────┐    ┌──────────────────┐    │
│   │ Products │    │ Orders   │    │ Final Result     │    │
│   │ (table)  │────│ (table)  │────│ (aggregated,     │    │
│   └──────────┘    └──────────┘    │  joined, filtered)│    │
│                                   └──────────────────┘    │
│                                                             │
│   Steps: Join → Filter → Aggregate → Project               │
│   Built visually in SAP HANA Modeling tools                 │
│                                                             │
│   When to use:                                              │
│   • Complex analytics that CDS views can't express          │
│   • Multi-table aggregations with custom logic              │
│   • Performance-critical reporting scenarios                │
│                                                             │
│   For most CAP apps: CDS views are sufficient!              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**For beginners:** You rarely need Calculation Views. CDS views handle most scenarios. Cal Views are for advanced analytics teams.

---

### Using Native HANA Features in CAP

If you need HANA-specific objects, place them in `db/src/`:

```
db/
├── schema.cds                    ← Your normal CDS model
├── data/                         ← CSV data
└── src/                          ← 🆕 Native HANA artifacts
    ├── MyCalcView.hdbcalculationview
    ├── MyProcedure.hdbprocedure
    └── MyIndex.hdbindex
```

CAP deploys these alongside the generated artifacts. They coexist peacefully.

**Example: Custom index for performance**

`db/src/product_name_index.hdbindex`:
```sql
INDEX product_name_idx ON COM_EPM_PRODUCTS (PRODUCTNAME)
```

---

### CAP Production Profile Configuration

Your `package.json` should have clear profile separation:

```json
{
  "cds": {
    "requires": {
      "db": {
        "kind": "sql",
        "[production]": {
          "kind": "hana-cloud",
          "impl": "@sap/cds/libx/_runtime/hana/Service.js"
        }
      },
      "auth": {
        "[production]": {
          "kind": "xsuaa"
        }
      }
    },
    "build": {
      "target": "gen",
      "tasks": [
        { "for": "hana", "src": "db", "options": { "model": ["db/", "srv/"] } },
        { "for": "nodejs", "src": "srv", "options": { "model": ["db/", "srv/"] } }
      ]
    }
  }
}
```

**Profiles explained:**
- No profile marker → default for all environments
- `[development]` → `cds watch` (local)
- `[production]` → deployed to BTP (Cloud Foundry)
- `[hybrid]` → local app connected to cloud services

---

## Session 5: Deployment Logs & Troubleshooting (15:00 - 16:00)

### Understanding Deployment Output

When you deploy (`cf deploy` or `cds deploy --to hana`), you'll see output like:

```
Deploying to HANA...
  Processing 12 artifacts...
  Creating table COM_EPM_PRODUCTS...          ✅
  Creating table COM_EPM_SUPPLIERS...         ✅
  Creating table COM_EPM_CATEGORIES...        ✅
  Creating table COM_EPM_PURCHASEORDERS...    ✅
  Creating table COM_EPM_PURCHASEORDERITEMS... ✅
  Creating view COM_EPM_PRODUCTCATALOG...     ✅
  Loading data from COM_EPM_PRODUCTS.csv...   ✅ (5 rows)
  Loading data from COM_EPM_SUPPLIERS.csv...  ✅ (3 rows)
  
  Deployment complete! 12/12 artifacts deployed.
```

---

### Common Deployment Errors & Fixes

#### Error 1: "Connection refused" or "Cannot connect to HANA"

```
Error: Could not connect to HANA Cloud instance
```

**Causes & Fixes:**
| Cause | Fix |
|-------|-----|
| HANA instance is stopped | Restart from BTP Cockpit |
| Wrong SQL endpoint | Check the endpoint URL in BTP |
| IP not allowed | Add your IP to "Allowed Connections" |
| Wrong credentials | Reset DBADMIN password |

---

#### Error 2: "Table already exists" or Schema Conflict

```
Error: Cannot create table COM_EPM_PRODUCTS: already exists
```

**Cause:** Deploying a second time without proper HDI handling.

**Fix:** HDI handles this automatically if you use `cds build` + `cf deploy`. If you manually created tables, drop them first:
```sql
DROP TABLE COM_EPM_PRODUCTS;
```

Or redeploy with `--auto-undeploy`:
```bash
cf deploy gen/db --auto-undeploy
```

---

#### Error 3: "Invalid column name" or Type Mismatch

```
Error: Column SUPPLIERNAME does not exist in COM_EPM_SUPPLIERS
```

**Cause:** Your CDS uses `supplierName` (camelCase) but HANA converts to UPPERCASE: `SUPPLIERNAME`. The issue is usually a typo or case mismatch in CSV headers.

**Fix:** Check your CSV column headers match CDS field names EXACTLY (case-sensitive before compilation).

---

#### Error 4: CSV Data Load Fails

```
Error: Could not load data for COM_EPM_PRODUCTS: wrong number of columns
```

**Causes:**
- CSV has wrong number of columns (missing or extra separator)
- CSV header doesn't match generated column names
- UUID format incorrect

**Fix checklist:**
1. Verify CSV uses semicolons (`;`) as separators
2. Column headers must match CDS field names (before HANA transforms them)
3. UUID must be full 36-character format
4. No trailing semicolons at end of lines
5. Dates in ISO format: `YYYY-MM-DD`

---

#### Error 5: "Insufficient privilege"

```
Error: Insufficient privilege: Not authorized
```

**Cause:** The HDI container technical user doesn't have permissions.

**Fix:** This usually means the service binding is wrong. Rebind:
```bash
cf unbind-service my-app my-hana-service
cf bind-service my-app my-hana-service
cf restage my-app
```

---

#### Error 6: "Foreign key constraint violated"

```
Error: Cannot insert: foreign key constraint violated for SUPPLIER_ID
```

**Cause:** Trying to load Products CSV before Suppliers CSV (the referenced supplier doesn't exist yet).

**Fix:** Check CSV data load order. Parent tables must be loaded first:
1. Categories (no dependencies)
2. Suppliers (no dependencies)
3. Products (depends on Categories + Suppliers)
4. Orders (depends on Customers)
5. OrderItems (depends on Orders + Products)

---

### Debugging Deployment — Step by Step

```
Deployment failed? Follow this checklist:

□ Step 1: Is HANA running?
  → BTP Cockpit → HANA Cloud → Status = "Running"?

□ Step 2: Can you connect?
  → Database Explorer → try to open your HDI container

□ Step 3: Did the build succeed?
  → Run: cds build --production
  → Check gen/db/src/ folder exists and has files

□ Step 4: Check the generated artifacts
  → Open gen/db/src/*.hdbtable files
  → Verify column names look correct

□ Step 5: Check CSV data
  → Are headers correct?
  → Are UUIDs in proper format?
  → Are separators semicolons?

□ Step 6: Check deployment logs
  → cf logs my-app --recent
  → Look for ERROR lines

□ Step 7: Try deploying without data first
  → Move CSV files temporarily, deploy, then add them back
```

---

### Monitoring in Production

```sql
-- Check when tables were last modified
SELECT TABLE_NAME, MODIFY_TIME
FROM M_TABLES
WHERE SCHEMA_NAME = CURRENT_SCHEMA
ORDER BY MODIFY_TIME DESC;

-- Check table sizes
SELECT TABLE_NAME, RECORD_COUNT, MEMORY_SIZE_IN_TOTAL / 1024 AS size_kb
FROM M_TABLES
WHERE SCHEMA_NAME = CURRENT_SCHEMA
AND RECORD_COUNT > 0
ORDER BY MEMORY_SIZE_IN_TOTAL DESC;

-- Active connections
SELECT * FROM M_CONNECTIONS WHERE CONNECTION_STATUS = 'RUNNING';
```

---

## Session 6: Hands-on — Deploy EPM Model to HANA Cloud (16:00 - 16:45)

### Exercise: Complete HANA Deployment

**Step 1: Prepare your project**
```bash
cd ~/projects/po-management

# Add HANA config (if not already done)
cds add hana
npm install
```

**Step 2: Build for production**
```bash
cds build --production

# Verify generated files
ls gen/db/src/
# Should see: .hdbtable files for each entity, .hdbview for views
```

**Step 3: Deploy**

Choose your method:

```bash
# Method A: Full MTA deployment
cds add mta        # Generate mta.yaml if not present
mbt build          # Build the deployable archive
cf deploy mta_archives/*.mtar

# Method B: Direct DB deployment (if service binding exists)
cds deploy --to hana

# Method C: From BAS
# Right-click gen/db → "Deploy to SAP HANA Database"
```

**Step 4: Verify in Database Explorer**

Open Database Explorer and run:
```sql
-- Check all tables exist
SELECT TABLE_NAME, RECORD_COUNT
FROM M_TABLES
WHERE SCHEMA_NAME = CURRENT_SCHEMA
AND TABLE_NAME LIKE 'COM_EPM%'
ORDER BY TABLE_NAME;

-- Check sample data loaded
SELECT COUNT(*) FROM COM_EPM_PRODUCTS;
SELECT COUNT(*) FROM COM_EPM_SUPPLIERS;
SELECT COUNT(*) FROM COM_EPM_PURCHASEORDERS;

-- Verify a join works (proves associations are correct)
SELECT p.PRODUCTNAME, s.SUPPLIERNAME
FROM COM_EPM_PRODUCTS p
JOIN COM_EPM_SUPPLIERS s ON p.SUPPLIER_ID = s.ID;
```

**Step 5: Test your CDS views**
```sql
-- ProductCatalog view (if you defined one in CDS)
SELECT * FROM COM_EPM_PRODUCTCATALOG;

-- Order summary view
SELECT * FROM COM_EPM_ORDERSUMMARY;
```

---

### Deployment Verification Checklist

- [ ] HANA Cloud instance is running (green status)
- [ ] `cds build --production` completes without errors
- [ ] `gen/db/src/` contains `.hdbtable` files for all entities
- [ ] Deployment succeeds (no error messages)
- [ ] All tables visible in Database Explorer
- [ ] CSV data loaded (row counts match expected)
- [ ] Views are accessible and return data
- [ ] JOINs work (associations translated to foreign keys correctly)
- [ ] Timestamps are in correct format
- [ ] Boolean fields stored correctly (TRUE/FALSE)

---

## Assessment: MCQ — 15 Questions on HANA DB Explorer

**Q1.** The SAP HANA Database Explorer is:

- A) A desktop application for Windows only
- B) A web-based tool for browsing tables, running SQL, and managing database objects
- C) A code editor for writing CDS
- D) A deployment tool

**Answer: B** — Database Explorer is browser-based. You browse tables, run SQL, view data, check performance.

---

**Q2.** To see the actual data in a table, you:

- A) Run `cds watch`
- B) Right-click the table → "Open Data" in Database Explorer
- C) Check the CSV files
- D) Open package.json

**Answer: B** — "Open Data" shows live data currently stored in the HANA table.

---

**Q3.** In HANA Cloud, the CDS type `String(100)` becomes:

- A) `VARCHAR(100)`
- B) `NVARCHAR(100)`
- C) `TEXT(100)`
- D) `CHAR(100)`

**Answer: B** — HANA uses `NVARCHAR` (Unicode) for CDS `String` types. The N = National (Unicode support).

---

**Q4.** To get the top 10 most expensive products in HANA SQL:

- A) `SELECT * FROM Products LIMIT 10 ORDER BY price DESC`
- B) `SELECT TOP 10 * FROM COM_EPM_PRODUCTS ORDER BY PRICE DESC`
- C) `SELECT * FROM COM_EPM_PRODUCTS WHERE ROWNUM <= 10`
- D) `SELECT FIRST 10 * FROM COM_EPM_PRODUCTS`

**Answer: B** — HANA uses `TOP N` syntax (not LIMIT). Column names are UPPERCASE.

---

**Q5.** The `DUMMY` table in HANA is used for:

- A) Testing insert operations
- B) Calculations without needing a real table (like `SELECT 1+1 FROM DUMMY`)
- C) Storing temporary data
- D) Debugging errors

**Answer: B** — `DUMMY` is a system table with one row. Use it for calculations: `SELECT CURRENT_DATE FROM DUMMY`.

---

**Q6.** If a HANA deployment fails with "foreign key constraint violated" on Products:

- A) Delete all products
- B) The referenced Suppliers must be loaded FIRST (parent before child)
- C) Change the data type
- D) Remove the association from CDS

**Answer: B** — Parent records must exist before children reference them. Load order: Suppliers → Products.

---

**Q7.** HANA stores UUID keys as:

- A) A special UUID type
- B) `NVARCHAR(36)` — just a 36-character string
- C) `INTEGER` — auto-incrementing
- D) `BINARY(16)`

**Answer: B** — There's no native UUID type in HANA. CDS UUID becomes `NVARCHAR(36)`.

---

**Q8.** `SELECT ADD_DAYS(CURRENT_DATE, 30) FROM DUMMY` returns:

- A) Today's date
- B) A date 30 days in the future
- C) An error
- D) The number 30

**Answer: B** — `ADD_DAYS` adds days to a date. With `CURRENT_DATE` + 30 = 30 days from today.

---

**Q9.** If your table has no data after deployment but CSVs exist:

- A) HANA doesn't support CSV loading
- B) Check CSV file naming (must match `namespace-Entity.csv`), separator (semicolons), and column headers
- C) You need to INSERT data manually
- D) Restart the HANA instance

**Answer: B** — CSV loading issues are almost always: wrong filename, wrong separator, or mismatched column headers.

---

**Q10.** Calculation Views in HANA are:

- A) Regular database tables
- B) Visual analytical models for complex reporting (join, aggregate, transform)
- C) CDS entity definitions
- D) JavaScript handlers

**Answer: B** — Calculation Views are HANA's visual modeling tool for complex analytics. Most CAP apps don't need them — CDS views suffice.

---

**Q11.** `M_TABLES` in HANA is a:

- A) Regular user table
- B) System monitoring view showing metadata about all tables (row count, memory, etc.)
- C) A backup table
- D) A configuration file

**Answer: B** — `M_` prefix = monitoring views. `M_TABLES` shows table statistics (record count, memory usage, etc.).

---

**Q12.** To safely delete records, the best practice is:

- A) Run DELETE immediately
- B) First run SELECT with the same WHERE clause to verify what will be deleted, then DELETE
- C) Drop and recreate the table
- D) Use TRUNCATE always

**Answer: B** — Always preview with SELECT first. Verify the rows are what you expect, then run DELETE.

---

**Q13.** "Auto-undeploy" during deployment means:

- A) Automatically deploy everything
- B) Remove database objects that no longer exist in your source files
- C) Undo the last deployment
- D) Delete all data

**Answer: B** — If you removed an entity from CDS, auto-undeploy drops the corresponding table from HANA.

---

**Q14.** To check if your HANA instance is running from the terminal:

- A) `cds watch --check-hana`
- B) Check BTP Cockpit → SAP HANA Cloud → instance status
- C) `ping hana-cloud`
- D) `cf services` shows bound HANA status

**Answer: B** — The BTP Cockpit is the authoritative source for HANA instance status (Running/Stopped).

---

**Q15.** CDS views (like `entity ProductCatalog as select from Products`) deploy to HANA as:

- A) New tables with copied data
- B) HANA views (`.hdbview` artifacts) — read-only, no data duplication
- C) Stored procedures
- D) Triggers

**Answer: B** — CDS views become HANA views. They don't store data — they read from underlying tables on the fly.

---

## Assignment: Deploy Complete PO Management to HANA Cloud

### Task

Deploy your complete PO Management CAP application to HANA Cloud and verify everything works.

### Steps

1. **Prepare the project:**
   ```bash
   cds add hana
   npm install
   ```

2. **Verify your CSV data:**
   - Check all CSV files have correct semicolons, UUIDs, and column names
   - Verify load order won't cause FK violations

3. **Build:**
   ```bash
   cds build --production
   ```

4. **Deploy to HANA Cloud**

5. **Verify in Database Explorer:**

   Run these queries and record the results:

   ```sql
   -- Query 1: Table inventory
   SELECT TABLE_NAME, RECORD_COUNT
   FROM M_TABLES WHERE SCHEMA_NAME = CURRENT_SCHEMA
   AND TABLE_NAME LIKE 'COM_%'
   ORDER BY TABLE_NAME;

   -- Query 2: Product details with supplier
   SELECT p.PRODUCTNAME, p.PRICE, p.STOCK, s.SUPPLIERNAME
   FROM COM_EPM_PRODUCTS p
   JOIN COM_EPM_SUPPLIERS s ON p.SUPPLIER_ID = s.ID;

   -- Query 3: Order summary
   SELECT STATUS, COUNT(*) AS order_count, SUM(GROSSAMOUNT) AS total
   FROM COM_EPM_PURCHASEORDERS
   GROUP BY STATUS;

   -- Query 4: Low stock products
   SELECT PRODUCTNAME, STOCK, MINSTOCK
   FROM COM_EPM_PRODUCTS
   WHERE STOCK < MINSTOCK;

   -- Query 5: Views work
   SELECT * FROM COM_EPM_PRODUCTCATALOG;
   ```

### Verification Checklist

- [ ] `cds build --production` succeeds
- [ ] All tables created in HANA
- [ ] CSV data loaded (correct row counts)
- [ ] JOINs return data (associations work)
- [ ] Views return data (CDS views compiled correctly)
- [ ] INSERT/UPDATE/DELETE work from SQL Console
- [ ] No orphaned records (FK integrity maintained)

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Database Explorer | Web-based tool to browse/query HANA (your DB control panel) |
| SQL Console | Run any SQL directly on HANA Cloud |
| HANA SQL | Standard SQL + TOP N, DUMMY table, ADD_DAYS, NVARCHAR |
| Type Mapping | String→NVARCHAR, Integer→INTEGER, UUID→NVARCHAR(36) |
| Calculation Views | HANA's visual analytics tool (advanced, rarely needed in CAP) |
| Native Artifacts | Place in `db/src/` for HANA-specific objects |
| Deployment Errors | Connection, FK violations, CSV format, case sensitivity |
| Troubleshooting | 7-step checklist: running? → connected? → built? → data correct? |
| Production Config | Profile-based: `[development]`=SQLite, `[production]`=HANA |

### Key SQL Patterns for HANA

```sql
-- Quick row count:      SELECT COUNT(*) FROM TABLE_NAME;
-- Top N:                SELECT TOP 5 * FROM TABLE ORDER BY field DESC;
-- Current time:         SELECT CURRENT_TIMESTAMP FROM DUMMY;
-- Date math:            SELECT ADD_DAYS(CURRENT_DATE, 30) FROM DUMMY;
-- Table metadata:       SELECT * FROM M_TABLES WHERE SCHEMA_NAME = CURRENT_SCHEMA;
-- Column info:          SELECT * FROM TABLE_COLUMNS WHERE TABLE_NAME = 'MY_TABLE';
```

### The One-Liner

> **Database Explorer is your window into HANA — browse, query, modify, and monitor your production data without leaving the browser.**

---

### Looking Ahead: Day 29

Tomorrow: **Authentication & Authorization in CAP**
- Securing your services with XSUAA
- Role-based access control
- `@requires` and `@restrict` annotations
- Testing with mock users locally

---

*End of Day 28 — You can now work directly with your HANA Cloud database like a pro!*
