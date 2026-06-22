# Day 27: SAP HANA Cloud — Setup & Basics

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 26 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: What is SAP HANA? Column Store Explained | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: SAP HANA Cloud & HDI Containers | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Create HANA Cloud Instance in BTP Trial | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: CAP with HANA — Configuration & `cds add hana` | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Deploy to HANA & Explore with DB Explorer | 60 min |
| 16:00 - 16:45 | Session 6: HANA Cloud Architecture Quiz & Review | 45 min |
| 16:45 - 17:00 | Assessment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain what SAP HANA is and why it's revolutionary (column store vs row store)
- Describe SAP HANA Cloud features and how it differs from on-premise HANA
- Set up a HANA Cloud instance in your BTP Trial account
- Understand HDI (HANA Deployment Infrastructure) and its purpose
- Run `cds add hana` to prepare your CAP project for HANA deployment
- Deploy your CAP data model to HANA Cloud
- Connect to HANA using the Database Explorer and inspect tables

---

## Day 26 Recap — Quick Fire (09:00 - 09:15)

1. The 3 areas in Git are _____, _____, _____? → _____
2. `git add .` moves files from _____ to _____? → _____
3. A branch allows you to _____? → _____
4. `git push` sends code to _____? → _____
5. What does `.gitignore` do? → _____
6. A Pull Request is for _____? → _____
7. When two people edit the same line → _____? → _____
8. Fix for "push rejected: fetch first" → _____?

<details>
<summary>Answers</summary>

1. Working Directory, Staging Area, Repository
2. Working Directory → Staging Area
3. Work in isolation without affecting the main branch
4. The remote repository (GitHub)
5. Specifies files Git should NOT track (node_modules, .env, *.db)
6. Code review before merging a branch into main
7. Merge conflict (resolve manually)
8. `git pull --rebase origin main` then `git push`

</details>

---

## Session 1: What is SAP HANA? Column Store Explained (09:15 - 10:30)

### What is SAP HANA?

**SAP HANA** = **H**igh-performance **AN**alytic **A**ppliance

It's SAP's in-memory database that stores data in RAM instead of on disk. This makes it incredibly fast — like the difference between searching a book on your desk vs searching a book in a warehouse.

```
Traditional Database (Oracle, SQL Server):
┌──────────────────────────────────────────┐
│  Data stored on HARD DISK (slow)         │
│  Read from disk → Process → Return       │
│  Like finding a book in a warehouse      │
│  Speed: Seconds to minutes for complex   │
│         queries on large datasets        │
└──────────────────────────────────────────┘

SAP HANA:
┌──────────────────────────────────────────┐
│  Data stored in RAM/MEMORY (blazing fast!)│
│  Already in memory → Process → Return    │
│  Like finding a book on your desk        │
│  Speed: Milliseconds for the SAME query! │
└──────────────────────────────────────────┘
```

---

### Why is HANA So Fast? — The Column Store Secret

Traditional databases store data **row by row** (like reading a book line by line). HANA stores data **column by column**. This matters HUGELY for analytics.

---

### Row Store vs Column Store — The Phone Book Analogy

Imagine a phone book with 1 million contacts:

```
ROW STORE (Traditional):
┌────────────────────────────────────────────────────┐
│ Row 1: | John  | Smith  | +1-555-0101 | New York  │
│ Row 2: | Sarah | Jones  | +1-555-0102 | Chicago   │
│ Row 3: | Mike  | Chen   | +1-555-0103 | New York  │
│ Row 4: | Lisa  | Park   | +1-555-0104 | Boston    │
│ ...                                                │
│ Row 1M: | Alex | Kumar | +1-555-9999 | Seattle    │
└────────────────────────────────────────────────────┘

Query: "How many people live in New York?"
→ Must scan ALL 1 million rows, reading ALL columns
→ Read: 1M × 4 columns = 4M data points read
→ SLOW! (Most data read was irrelevant — we only needed 'city')
```

```
COLUMN STORE (HANA):
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ First Names  │ │ Last Names   │ │ Phones       │ │ Cities       │
│──────────────│ │──────────────│ │──────────────│ │──────────────│
│ John         │ │ Smith        │ │ +1-555-0101  │ │ New York     │
│ Sarah        │ │ Jones        │ │ +1-555-0102  │ │ Chicago      │
│ Mike         │ │ Chen         │ │ +1-555-0103  │ │ New York     │
│ Lisa         │ │ Park         │ │ +1-555-0104  │ │ Boston       │
│ ...          │ │ ...          │ │ ...          │ │ ...          │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘

Query: "How many people live in New York?"
→ Only scan the CITIES column (ignore first names, last names, phones!)
→ Read: 1M × 1 column = 1M data points read
→ FAST! (75% less data to scan!)
```

---

### Column Store Benefits — Visual Summary

```
┌─────────────────────────────────────────────────────────────┐
│         WHY COLUMN STORE IS PERFECT FOR ANALYTICS            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. FEWER READS — Only read the columns you query           │
│     "Sum of all prices" → reads ONLY price column           │
│     (ignores name, description, supplier, 15 other cols)    │
│                                                             │
│  2. BETTER COMPRESSION — Same values compress well          │
│     City column: [NY, NY, NY, Chicago, NY, NY] → very      │
│     compressible! (lots of repeated values in one column)   │
│                                                             │
│  3. PARALLEL PROCESSING — Each column scanned independently │
│     CPU cores work on different columns simultaneously      │
│                                                             │
│  4. AGGREGATION SPEED — SUM, AVG, COUNT are column ops     │
│     "Average salary" → scan just the salary column → FAST!  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### When to Use Row Store vs Column Store

| Feature | Row Store | Column Store |
|---------|-----------|--------------|
| **Best for** | Single record lookup (by ID) | Analytics on large datasets |
| **Example** | "Get order SO-001" | "Total revenue by region last year" |
| **INSERT speed** | Fast (append one row) | Slightly slower (update many columns) |
| **SELECT all columns** | Fast (one row = one read) | Slower (must combine columns) |
| **SELECT few columns** | Slow (reads ALL columns anyway) | Very fast (only needed columns) |
| **Aggregation (SUM, AVG)** | Slow | Extremely fast |
| **Compression** | Lower | Much higher (similar values together) |

**HANA supports BOTH!** By default, tables are column store (best for enterprise reporting). You can explicitly create row store tables when needed.

---

### SAP HANA — Key Features

| Feature | Description |
|---------|-------------|
| **In-Memory** | All data in RAM — no disk reads for queries |
| **Column Store** | Data stored by column — analytics are instant |
| **Multi-Model** | Relational + Graph + Spatial + JSON + Text search |
| **Real-Time Analytics** | No separate data warehouse needed |
| **Advanced Processing** | Predictive, ML, text analysis built in |
| **ACID Compliant** | Full transaction support (same as traditional DB) |
| **Compression** | Data compressed 5-10x in memory |

---

### How HANA Fits in Your CAP Application

```
DEVELOPMENT (Local):
┌─────────────┐     ┌──────────┐
│  CAP App    │────►│  SQLite  │   Simple, local, no setup
│  cds watch  │     │  (file)  │   Good for development!
└─────────────┘     └──────────┘

PRODUCTION (Cloud):
┌─────────────┐     ┌──────────────────┐
│  CAP App    │────►│  SAP HANA Cloud  │   Fast, scalable, enterprise
│  (deployed) │     │  (in-memory)     │   Real production database!
└─────────────┘     └──────────────────┘
```

**Your CDS code doesn't change!** Same `schema.cds` works with SQLite (dev) AND HANA (production). CAP handles the translation.

---

## Session 2: SAP HANA Cloud & HDI Containers (10:45 - 12:00)

### SAP HANA Cloud vs HANA On-Premise

| Aspect | HANA On-Premise | HANA Cloud |
|--------|-----------------|------------|
| **Where** | Your own servers/data center | SAP manages in the cloud |
| **Setup** | Weeks (hardware + install) | Minutes (click and go!) |
| **Maintenance** | Your team patches/updates | SAP handles automatically |
| **Scaling** | Buy more hardware | Slider to add more RAM/CPU |
| **Cost** | Large upfront investment | Pay-per-use (monthly) |
| **Availability** | You manage uptime | SAP guarantees 99.9% SLA |
| **Access** | Only from your network | From anywhere (internet) |
| **Trial** | Not available | Free trial on BTP! |

**Bottom line:** HANA Cloud is the same powerful engine, managed by SAP in the cloud so you focus on building apps.

---

### SAP HANA Cloud — Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    SAP BTP                                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          SAP HANA Cloud Instance                      │  │
│  │                                                       │  │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐        │  │
│  │  │ HDI       │  │ HDI       │  │ HDI       │        │  │
│  │  │ Container │  │ Container │  │ Container │        │  │
│  │  │ (App 1)   │  │ (App 2)   │  │ (App 3)   │        │  │
│  │  │           │  │           │  │           │        │  │
│  │  │ Tables    │  │ Tables    │  │ Tables    │        │  │
│  │  │ Views     │  │ Views     │  │ Views     │        │  │
│  │  │ Procedures│  │ Procedures│  │ Procedures│        │  │
│  │  └───────────┘  └───────────┘  └───────────┘        │  │
│  │                                                       │  │
│  │  One HANA instance → multiple isolated containers     │  │
│  │  Each app gets its own container (isolated!)          │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### What is HDI? (HANA Deployment Infrastructure)

**HDI** is the deployment mechanism for HANA Cloud. Think of it as "npm for databases."

```
Traditional Database Deployment:
  Write SQL scripts → Run them manually → Hope nothing breaks
  "CREATE TABLE Products (...)"
  "ALTER TABLE Products ADD COLUMN rating ..."
  → Manual, error-prone, no undo!

HDI Deployment:
  Write design-time files → Deploy → HDI handles everything!
  → Automatic table creation
  → Automatic migration (add/remove columns safely)
  → Automatic rollback on errors
  → Version controlled!
```

---

### HDI Containers — Isolation for Each App

An **HDI Container** is like an apartment in a building:
- Each app gets its own container (apartment)
- Containers are isolated — one app can't see another's data
- Each has its own tables, views, procedures
- Managed by HDI (the building manager)

```
HANA Cloud Instance = The Building
├── HDI Container "po-management"  = Apartment 1
│   ├── Table: PurchaseOrders
│   ├── Table: PurchaseOrderItems
│   └── Table: Products
├── HDI Container "hr-app"         = Apartment 2
│   ├── Table: Employees
│   ├── Table: Departments
│   └── Table: LeaveRequests
└── HDI Container "sales-app"      = Apartment 3
    ├── Table: SalesOrders
    ├── Table: Customers
    └── Table: Products (separate copy!)
```

**Why HDI Containers?**
- **Security:** App A can't access App B's data
- **Deployment:** Each app deploys independently (no conflicts)
- **Cleanup:** Delete a container = delete all its objects cleanly
- **Multiple environments:** Dev container, Test container, Prod container — all isolated

---

### HDI Design-Time vs Run-Time

```
DESIGN TIME (what you write):                RUN TIME (what HANA creates):
┌────────────────────────────┐              ┌────────────────────────────┐
│ src/                       │              │                            │
│ ├── Products.hdbtable      │   Deploy →   │ Table: PRODUCTS            │
│ ├── Orders.hdbtable        │              │ Table: ORDERS              │
│ ├── ProductView.hdbview    │              │ View: PRODUCT_VIEW         │
│ └── CalcLogic.hdbprocedure │              │ Procedure: CALC_LOGIC      │
│                            │              │                            │
│ (source code — you write)  │              │ (live objects — HANA runs) │
└────────────────────────────┘              └────────────────────────────┘
```

**In CAP:** You NEVER write `.hdbtable` or `.hdbview` files manually! CAP generates them from your CDS files.

```
YOUR CDS:                    CAP GENERATES:              HANA CREATES:
schema.cds     ──build──►   .hdbtable files  ──deploy──► Live tables
(entity Products)            (generated)                  (in memory!)
```

---

### The CAP → HANA Pipeline

```
Step 1: You write CDS            schema.cds
                                    │
Step 2: cds build                   ▼
        compiles CDS to          gen/db/src/
        HANA artifacts           ├── Products.hdbtable
                                 ├── Orders.hdbtable
                                 └── ...
                                    │
Step 3: Deploy (cf deploy)          ▼
        pushes to HDI container  SAP HANA Cloud
                                 ├── TABLE: Products
                                 ├── TABLE: Orders
                                 └── ...
```

**Key insight:** Your CDS files are the "single source of truth." You never write SQL or HANA-specific files. CAP generates everything!

---

## Session 3: Hands-on — Create HANA Cloud Instance in BTP Trial (12:00 - 13:00)

### Prerequisites

- SAP BTP Trial account (https://account.hanatrial.ondemand.com)
- Access to SAP BTP Cockpit

---

### Step 1: Access BTP Cockpit

1. Log in to https://cockpit.hanatrial.ondemand.com
2. Click on your **Subaccount** (e.g., "trial")
3. You should see the subaccount overview

---

### Step 2: Create HANA Cloud Instance

1. In the left menu: **SAP HANA Cloud** → Click **"Create Instance"**
   (Or: go to **Service Marketplace** → search "SAP HANA Cloud" → Create)

2. Fill in the form:
   - **Instance Name:** `hana-trial` (or any name)
   - **Administrator Password:** Choose a strong password (save it!)
   - **Memory:** 30 GB (trial default)
   - **Allowed Connections:** Select "Allow all IP addresses" for trial

3. Click **"Create Instance"**

4. Wait 5-10 minutes for provisioning (grab a coffee ☕)

5. Status should change to: **"Running"** ✅

---

### Step 3: Verify HANA Cloud is Running

1. In BTP Cockpit → SAP HANA Cloud → Your instance
2. Status: **Running** (green indicator)
3. Note the **SQL Endpoint** — you'll need this later:
   ```
   abc123.hana.trial-us10.hanacloud.ondemand.com:443
   ```

---

### Step 4: Open Database Explorer

1. Click **"Open in SAP HANA Database Explorer"** (from instance actions)
2. Or go to: SAP HANA Database Explorer (from BTP tools)
3. Add your database connection:
   - Host: (your SQL endpoint)
   - Port: 443
   - User: DBADMIN
   - Password: (the one you set)
4. You should see the database catalog (empty for now — no tables yet!)

---

### Important Notes for Trial

| Note | Details |
|------|---------|
| **Auto-stop** | Trial HANA stops after inactivity. Restart from BTP Cockpit. |
| **Memory limit** | 30 GB in trial (plenty for learning!) |
| **Cost** | Free in trial. In production: pay-per-use based on memory. |
| **Region** | Choose a region close to you for lower latency |
| **Allowed IPs** | In trial, allow all. In production, restrict! |

---

## Session 4: CAP with HANA — Configuration & `cds add hana` (13:45 - 14:45)

### Switching from SQLite to HANA

During development, you used SQLite:
```bash
cds watch    # Uses SQLite automatically (in-memory or file)
```

For production, switch to HANA Cloud. CAP makes this EASY:

```bash
cds add hana    # ← This single command configures everything!
```

---

### What `cds add hana` Does

Running `cds add hana` in your project root:

```bash
cd my-cap-project
cds add hana
```

**Changes it makes:**

1. **Adds to `package.json`:**
```json
{
  "cds": {
    "requires": {
      "db": {
        "kind": "hana-cloud",
        "[development]": { "kind": "sql" },
        "[production]": { "kind": "hana-cloud" }
      }
    }
  },
  "dependencies": {
    "@sap/hdi-deploy": "^4",
    "hdb": "^0.19"
  }
}
```

2. **Creates/updates `.cdsrc.json` or `package.json` cds section:**
   - Development mode uses SQLite (local)
   - Production mode uses HANA Cloud

3. **Adds the `@sap/hdi-deploy` dependency** (for deploying to HDI containers)

---

### Understanding the Configuration

```json
"cds": {
  "requires": {
    "db": {
      "kind": "hana-cloud",                    // Default: HANA
      "[development]": { "kind": "sql" },      // Override for dev: SQLite
      "[production]": { "kind": "hana-cloud" } // Explicit for prod: HANA
    }
  }
}
```

**Profile-based configuration:**
- When you run `cds watch` → uses `[development]` profile → SQLite
- When deployed to BTP → uses `[production]` profile → HANA Cloud

**You don't change your CDS code!** Same schema, same services — only the database engine changes.

---

### Building for HANA

```bash
# Build generates HANA artifacts
cds build --production
```

This creates:
```
gen/
├── db/
│   ├── package.json         (HDI deployer config)
│   ├── src/
│   │   ├── .hdinamespace    (namespace config)
│   │   ├── Products.hdbtable
│   │   ├── Orders.hdbtable
│   │   ├── OrderItems.hdbtable
│   │   ├── ProductView.hdbview
│   │   └── ...
│   └── undeploy.json
└── srv/
    └── ... (service artifacts)
```

**Generated files explained:**

| File Extension | What It Is |
|---------------|------------|
| `.hdbtable` | Table definition (from your CDS entity) |
| `.hdbview` | View definition (from CDS views) |
| `.hdbconstraint` | Foreign key constraints (from associations) |
| `.hdbtabledata` | Initial data (from CSV files) |
| `.hdbprocedure` | Stored procedure (if you define one) |
| `.hdinamespace` | Namespace configuration |

---

### Example: What Gets Generated

**Your CDS:**
```cds
namespace com.epm;
entity Products : cuid, managed {
  productName : String(100);
  price       : Decimal(10,2);
  stock       : Integer default 0;
  supplier    : Association to Suppliers;
}
```

**Generated `Products.hdbtable` (simplified):**
```json
{
  "format_version": 1,
  "table_type": "COLUMN",
  "schema_name": "",
  "table_name": "COM_EPM_PRODUCTS",
  "columns": [
    { "name": "ID", "sql_type": "NVARCHAR", "length": 36, "nullable": false },
    { "name": "PRODUCTNAME", "sql_type": "NVARCHAR", "length": 100 },
    { "name": "PRICE", "sql_type": "DECIMAL", "precision": 10, "scale": 2 },
    { "name": "STOCK", "sql_type": "INTEGER", "default_value": "0" },
    { "name": "SUPPLIER_ID", "sql_type": "NVARCHAR", "length": 36 },
    { "name": "CREATEDAT", "sql_type": "TIMESTAMP" },
    { "name": "CREATEDBY", "sql_type": "NVARCHAR", "length": 255 },
    { "name": "MODIFIEDAT", "sql_type": "TIMESTAMP" },
    { "name": "MODIFIEDBY", "sql_type": "NVARCHAR", "length": 255 }
  ],
  "primary_key": { "columns": ["ID"] }
}
```

**Notice:** `table_type: "COLUMN"` → HANA creates a column-store table by default!

---

### Hybrid Testing — Use HANA Locally

You can connect your local `cds watch` to HANA Cloud for testing:

```bash
# Bind to your HANA Cloud service
cds bind --to hana:hana-trial

# Now cds watch uses HANA Cloud instead of SQLite!
cds watch --profile production
```

This is useful for testing HANA-specific features before full deployment.

---

## Session 5: Hands-on — Deploy to HANA & Explore with DB Explorer (15:00 - 16:00)

### Exercise 1: Prepare Your Project (15 minutes)

```bash
# 1. Navigate to your project
cd ~/projects/po-management

# 2. Add HANA configuration
cds add hana

# 3. Install new dependencies
npm install

# 4. Verify package.json has hana config
cat package.json | grep -A5 "requires"
```

**Expected output in package.json:**
```json
"cds": {
  "requires": {
    "db": {
      "kind": "hana-cloud"
    }
  }
}
```

---

### Exercise 2: Build HANA Artifacts (15 minutes)

```bash
# Build for production (generates HANA files)
cds build --production

# Check what was generated
ls gen/db/src/

# You should see .hdbtable, .hdbview files
# Each entity in your CDS becomes a .hdbtable file
```

**Explore a generated file:**
```bash
cat gen/db/src/com.epm-PurchaseOrders.hdbtable
```

---

### Exercise 3: Deploy to HANA Cloud (15 minutes)

**Option A: Using `cf` CLI (Cloud Foundry)**

```bash
# Login to Cloud Foundry
cf login -a https://api.cf.us10-001.hana.ondemand.com

# Deploy (creates HDI container + deploys tables)
cf deploy
```

**Option B: Using `cds deploy` with bound service**

```bash
# If you have the HANA service bound:
cds deploy --to hana
```

**Option C: Manual HDI deployment (from BAS)**

In SAP Business Application Studio:
1. Right-click `gen/db` folder
2. Select "Deploy to HANA Database"
3. Choose your HANA instance
4. Wait for deployment...
5. Success! Tables created.

---

### Exercise 4: Explore with Database Explorer (15 minutes)

1. Open **SAP HANA Database Explorer** from BTP Cockpit
2. Connect to your HDI container (it's created automatically)
3. Navigate: **Catalog → Tables**
4. You should see your tables:
   ```
   COM_EPM_PURCHASEORDERS
   COM_EPM_PURCHASEORDERITEMS
   COM_EPM_PRODUCTS
   COM_EPM_SUPPLIERS
   COM_EPM_CATEGORIES
   ```
5. Right-click a table → **"Open Data"** → See the data!
6. Try running a SQL query:
   ```sql
   SELECT * FROM COM_EPM_PRODUCTS WHERE STOCK < 10;
   ```

---

### What You Should See in Database Explorer

```
Database Explorer:
├── Catalog
│   ├── Tables
│   │   ├── COM_EPM_PURCHASEORDERS
│   │   ├── COM_EPM_PURCHASEORDERITEMS
│   │   ├── COM_EPM_PRODUCTS
│   │   ├── COM_EPM_SUPPLIERS
│   │   └── COM_EPM_CATEGORIES
│   ├── Views
│   │   ├── COM_EPM_PRODUCTCATALOG    (from your CDS view)
│   │   └── COM_EPM_ORDERSUMMARY
│   └── Column Views
│       └── (system-generated)
```

**Try these queries:**
```sql
-- Count all products
SELECT COUNT(*) FROM COM_EPM_PRODUCTS;

-- Products with low stock
SELECT PRODUCTNAME, STOCK, MINSTOCK
FROM COM_EPM_PRODUCTS
WHERE STOCK < MINSTOCK;

-- Total order value
SELECT SUM(GROSSAMOUNT) as TOTAL_REVENUE
FROM COM_EPM_SALESORDERS
WHERE STATUS = 'Delivered';
```

---

## Session 6: HANA Cloud Architecture Quiz & Review (16:00 - 16:45)

### Architecture Quiz — Fill in the Blanks

**Q1:** SAP HANA stores data in _____ instead of on disk, which makes it extremely fast.

<details><summary>Answer</summary>RAM / Memory (in-memory database)</details>

---

**Q2:** Column store is better than row store for _____ queries (like SUM, AVG, COUNT).

<details><summary>Answer</summary>Analytical / Aggregation queries</details>

---

**Q3:** HDI stands for _____ _____ _____.

<details><summary>Answer</summary>HANA Deployment Infrastructure</details>

---

**Q4:** Each CAP application gets its own _____ _____ in HANA Cloud (isolation boundary).

<details><summary>Answer</summary>HDI Container</details>

---

**Q5:** The command _____ configures a CAP project for HANA Cloud deployment.

<details><summary>Answer</summary>`cds add hana`</details>

---

**Q6:** CAP uses _____ for development and _____ for production by default after `cds add hana`.

<details><summary>Answer</summary>SQLite for development, HANA Cloud for production</details>

---

**Q7:** The `cds build --production` command generates _____ files from your CDS entities.

<details><summary>Answer</summary>`.hdbtable` (and `.hdbview`) — HANA design-time artifacts</details>

---

**Q8:** In HANA Cloud, tables are created as _____ store by default (for analytics performance).

<details><summary>Answer</summary>Column store</details>

---

**Q9:** The advantage of HDI containers is that they provide _____ between applications.

<details><summary>Answer</summary>Isolation (each app's data is separate and secure)</details>

---

**Q10:** Your CDS code (schema.cds) works with both SQLite and HANA because CAP handles the _____.

<details><summary>Answer</summary>Translation / Adaptation (database-agnostic — same code, different engines)</details>

---

### Key Differences: Dev vs Production

```
┌────────────────────────────────────────────────────────────┐
│              DEVELOPMENT vs PRODUCTION                       │
├──────────────────────────┬─────────────────────────────────┤
│  LOCAL DEVELOPMENT       │  PRODUCTION (BTP)               │
├──────────────────────────┼─────────────────────────────────┤
│  cds watch               │  cf deploy                      │
│  SQLite (file/memory)    │  SAP HANA Cloud (in-memory)     │
│  db/data.db              │  HDI Container                  │
│  Instant (no setup)      │  Requires HANA instance         │
│  Single user             │  Multi-user (auth + roles)      │
│  No compression          │  5-10x compression              │
│  Good for testing        │  Good for real users            │
│  ~100 records            │  Millions of records            │
│  Free                    │  Pay-per-use                    │
└──────────────────────────┴─────────────────────────────────┘
```

---

## Assessment: MCQ — 15 Questions on HANA Cloud

**Q1.** What does SAP HANA stand for?

- A) Highly Available Network Architecture
- B) High-performance Analytic Appliance
- C) HANA Application Networking Application
- D) Hardware Accelerated New Architecture

**Answer: B** — High-performance ANalytic Appliance.

---

**Q2.** The key difference that makes HANA fast is:

- A) It uses a faster programming language
- B) It stores all data in RAM (in-memory) instead of on disk
- C) It has more hard drive space
- D) It doesn't support transactions

**Answer: B** — In-memory storage eliminates disk I/O, the biggest bottleneck in traditional databases.

---

**Q3.** Column store is better than row store for:

- A) Inserting single records quickly
- B) Analytical queries (SUM, AVG, COUNT) on large datasets
- C) Updating one specific row
- D) Storing images and videos

**Answer: B** — Column store reads only the needed columns, making aggregation queries dramatically faster.

---

**Q4.** SAP HANA Cloud differs from HANA on-premise because:

- A) HANA Cloud is slower
- B) HANA Cloud is managed by SAP in the cloud (no hardware/maintenance for you)
- C) HANA Cloud doesn't support SQL
- D) HANA Cloud is only for small databases

**Answer: B** — Cloud = SAP manages infrastructure, patching, backups. You just use it.

---

**Q5.** HDI (HANA Deployment Infrastructure) is:

- A) A programming language
- B) A deployment mechanism that manages database objects in containers
- C) A UI framework
- D) A backup tool

**Answer: B** — HDI manages the lifecycle of database objects (tables, views) in isolated containers.

---

**Q6.** An HDI Container provides:

- A) More memory for the application
- B) Isolation — each app has its own tables/views that other apps can't access
- C) Faster network speed
- D) A user interface

**Answer: B** — Containers isolate applications. App A can't see or affect App B's database objects.

---

**Q7.** `cds add hana` does what to your CAP project?

- A) Creates a HANA Cloud instance
- B) Adds HANA configuration to package.json and installs required dependencies
- C) Deploys the app to production
- D) Deletes the SQLite database

**Answer: B** — It configures the project for HANA (dependencies, profile settings) but doesn't deploy or create instances.

---

**Q8.** After `cds add hana`, `cds watch` in development mode uses:

- A) HANA Cloud
- B) SQLite (locally) — HANA is only used in production profile
- C) No database
- D) MongoDB

**Answer: B** — The `[development]` profile keeps using SQLite. Only `[production]` profile uses HANA.

---

**Q9.** `cds build --production` generates what files?

- A) HTML and CSS files
- B) `.hdbtable` and `.hdbview` files (HANA design-time artifacts)
- C) Docker containers
- D) Test reports

**Answer: B** — Build generates HANA-specific files from your CDS that HDI can deploy.

---

**Q10.** In HANA Cloud, the CDS entity name `com.epm.Products` becomes table name:

- A) `products`
- B) `COM_EPM_PRODUCTS`
- C) `com.epm.Products`
- D) `com-epm-Products`

**Answer: B** — HANA converts: dots → underscores, all UPPERCASE. `com.epm.Products` → `COM_EPM_PRODUCTS`.

---

**Q11.** TRUE or FALSE: You need to rewrite your CDS schema files when switching from SQLite to HANA.

- A) TRUE
- B) FALSE

**Answer: B (FALSE)** — Same CDS code works with both. CAP translates to the appropriate SQL dialect automatically.

---

**Q12.** The SAP HANA Database Explorer is used for:

- A) Writing CDS files
- B) Browsing tables, running SQL queries, and inspecting data in HANA
- C) Deploying applications
- D) Managing user authentication

**Answer: B** — Database Explorer is a web-based SQL tool for HANA (browse tables, run queries, view data).

---

**Q13.** In BTP Trial, HANA Cloud instances:

- A) Run 24/7 forever
- B) Auto-stop after inactivity (must restart manually)
- C) Are deleted every day
- D) Only work on weekends

**Answer: B** — Trial instances auto-stop to save resources. Restart from BTP Cockpit when needed.

---

**Q14.** HANA column store provides better compression because:

- A) It uses ZIP files
- B) Similar values in the same column compress well (many repeated values)
- C) It deletes old data automatically
- D) It stores less data

**Answer: B** — A "status" column with millions of rows might have only 5 unique values → extremely compressible!

---

**Q15.** The correct deployment flow for CAP to HANA is:

- A) Write CDS → Deploy directly to HANA
- B) Write CDS → `cds build` (generates artifacts) → Deploy artifacts to HDI container
- C) Write SQL → Upload to HANA
- D) Write CDS → Convert to JSON → Import

**Answer: B** — CDS → Build (`.hdbtable` files) → Deploy to HDI Container. Three clear steps.

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| SAP HANA | In-memory database — data in RAM = incredibly fast |
| Column Store | Stores data by column — analytics (SUM/AVG/COUNT) are instant |
| Row vs Column | Row = good for single lookups; Column = good for analytics |
| HANA Cloud | Same engine, managed by SAP in the cloud (no hardware hassle) |
| HDI | Deployment mechanism — manages DB objects in containers |
| HDI Containers | Isolation per app — security + independent deployment |
| `cds add hana` | Configures CAP project for HANA (adds deps + profiles) |
| `cds build` | Generates .hdbtable/.hdbview from your CDS |
| Dev vs Prod | SQLite for local dev, HANA for production (same CDS code!) |
| Database Explorer | Web tool to browse tables and run SQL on HANA |

### The Migration Path

```
Development:  schema.cds → cds watch → SQLite (instant, local)
                  │
                  │ Same CDS code!
                  │
Production:   schema.cds → cds build → .hdbtable → deploy → HANA Cloud
```

### The One-Liner

> **HANA Cloud = your CAP data model running in-memory at enterprise scale. Same CDS, same code — just faster and in the cloud.**

---

### Looking Ahead: Day 28

Tomorrow: **Authentication & Authorization in CAP**
- Securing your services with user login
- Role-based access control (@requires, @restrict)
- XSUAA service on BTP
- Testing with mock users locally

---

*End of Day 27 — You now understand where your data lives in production!*
