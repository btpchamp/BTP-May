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
