# Day 14: CDS Data Modeling — Part 3 (Advanced & CSV Data)

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 13 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Views in CDS — Shaping Your Data | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Projections & Calculated Fields | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Build Views & Projections | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: CSV Data Loading — The Complete Guide | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: cds deploy, SQLite Inspection & Build Process | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Complete EPM with Full Sample Data | 45 min |
| 16:45 - 17:00 | Assessment & Wrap-up | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Create CDS views to reshape and filter data from existing entities
- Build projections that expose only specific fields to the API
- Write calculated fields using CDS expressions
- Load initial/sample data into your application using CSV files
- Follow correct CSV naming conventions so CAP auto-discovers your data
- Use `cds deploy` to persist data into SQLite
- Inspect your SQLite database to verify tables and data
- Understand the CDS build process from source to deployment

---

## Day 13 Recap — Quick Fire (09:00 - 09:15)

1. What keyword connects two entities in CDS? → _____
2. `author : Association to Authors;` creates what column in the database? → _____
3. What OData query option fetches related entity details inline? → _____
4. Composition means the parent _____ the children?
5. Deep Insert allows creating _____ and _____ in one request?
6. The `cuid` aspect provides which field automatically? → _____
7. The `managed` aspect auto-fills which 4 fields? → _____
8. `Currency` from `@sap/cds/common` is an association to what entity? → _____

<details>
<summary>Answers</summary>

1. `Association` (or `Composition`)
2. `author_ID` — a foreign key column
3. `$expand` (e.g., `?$expand=author`)
4. OWNS (children can't exist without parent)
5. Parent and children (e.g., Order and OrderItems)
6. `key ID : UUID`
7. `createdAt`, `createdBy`, `modifiedAt`, `modifiedBy`
8. `sap.common.Currencies` (a code-list entity with code, name, symbol)

</details>

---

## Session 1: Views in CDS — Shaping Your Data (09:15 - 10:30)

### Why Do We Need Views?

Imagine you have a `Products` entity with 15 fields. But your mobile app only needs 4 fields (name, price, image, rating). Sending all 15 fields wastes bandwidth and exposes internal data.

**Without views:**
```
Database has 15 fields → API sends all 15 fields → Mobile app only uses 4
                                                    (11 fields wasted!)
```

**With views:**
```
Database has 15 fields → View selects 4 fields → API sends only 4 → Mobile app happy!
```

Views let you:
- **Select** only the fields you need (hide internal fields)
- **Rename** fields for better API names
- **Filter** to show only relevant records
- **Calculate** new fields from existing ones
- **Combine** data from multiple entities into one view

---

### What is a CDS View?

A **CDS View** is like a "window" into your data. It doesn't create a new table — it just provides a different way to look at existing data.

**Real-world analogy:** Think of an entity as a full bookshelf with all books. A view is like a curated display shelf that shows only bestsellers — the books aren't duplicated, just presented differently.

```cds
// The FULL entity (all fields, stored in database):
entity Products {
  key ID      : UUID;
  name        : String(100);
  description : String(500);
  price       : Decimal(10,2);
  costPrice   : Decimal(10,2);   // Internal! Don't expose!
  margin      : Decimal(5,2);    // Internal!
  stock       : Integer;
  minStock    : Integer;         // Internal!
  supplier_ID : UUID;            // Internal!
  rating      : Decimal(2,1);
  image       : String(500);
  isActive    : Boolean;
}

// A VIEW — shows only what the mobile app needs:
entity ProductCatalog as select from Products {
  ID,
  name,
  price,
  rating,
  image
} where isActive = true;
```

**What happened:**
- No new table created in the database
- `ProductCatalog` reads from `Products` but only returns 5 fields
- The `where` clause filters out inactive products automatically
- Internal fields like `costPrice`, `margin`, `supplier_ID` are HIDDEN

---

### CDS View Syntax — Step by Step

The basic view syntax is:

```cds
entity <ViewName> as select from <SourceEntity> {
  field1,
  field2,
  field3
};
```

Let's break it down piece by piece:

#### Part 1: `entity <ViewName>`
This is the name of your view. It becomes an API endpoint when exposed in a service.

#### Part 2: `as select from <SourceEntity>`
This tells CDS which entity (table) to read data from.

#### Part 3: `{ field1, field2, ... }`
This lists which columns you want. If you skip this part and write no field list, ALL fields are included.

#### Part 4: `where <condition>` (optional)
Filters which rows are returned.

---

### View Examples — From Simple to Complex

#### Example 1: Select Specific Fields

```cds
// Only show ID, title, and price from Books:
entity BookList as select from Books {
  ID,
  title,
  price
};
```

**Use case:** A simple book listing page that doesn't need all 12 fields.

---

#### Example 2: Select with WHERE Filter

```cds
// Only books that are currently available:
entity AvailableBooks as select from Books {
  ID,
  title,
  author,
  price,
  availableCopies
} where availableCopies > 0;
```

**Use case:** Library kiosk that should ONLY show books members can actually borrow.

---

#### Example 3: Rename Fields with `as`

```cds
// Rename fields for a cleaner API:
entity BookSummary as select from Books {
  ID,
  title        as bookTitle,
  price        as sellingPrice,
  author.name  as authorName,     // Flatten: get author's name directly
  genre.name   as genreName       // Flatten: get genre's name directly
};
```

**Use case:** Frontend team wants simpler field names without navigating associations.

**API Response:**
```json
{
  "bookTitle": "Clean Code",
  "sellingPrice": 450.00,
  "authorName": "Robert C. Martin",
  "genreName": "Programming"
}
```

Instead of:
```json
{
  "title": "Clean Code",
  "price": 450.00,
  "author": { "name": "Robert C. Martin" },
  "genre": { "name": "Programming" }
}
```

---

#### Example 4: Views with Associations (Flattening)

One of the most powerful features — you can "flatten" nested data:

```cds
entity BorrowingDetails as select from Borrowings {
  ID,
  borrowDate,
  dueDate,
  status,
  book.title       as bookTitle,      // Reach INTO the book entity
  book.isbn        as bookIsbn,
  member.firstName as memberFirst,    // Reach INTO the member entity
  member.lastName  as memberLast,
  member.email     as memberEmail
};
```

**What this does:** Instead of needing `$expand=book,member`, this view gives you a flat table with all the important info from three entities combined.

**Result — One flat row:**
```json
{
  "borrowDate": "2026-05-10",
  "dueDate": "2026-05-24",
  "status": "Borrowed",
  "bookTitle": "Clean Code",
  "bookIsbn": "9780132350884",
  "memberFirst": "Priya",
  "memberLast": "Sharma",
  "memberEmail": "priya@email.com"
}
```

**Without the view**, you'd need: `GET /Borrowings?$expand=book($select=title,isbn),member($select=firstName,lastName,email)` — much more complex!

---

#### Example 5: Views with Aggregation

```cds
// Count books per genre:
entity BooksPerGenre as select from Books {
  genre.name as genreName,
  count(ID)  as bookCount : Integer
} group by genre.name;
```

**Result:**
```json
[
  { "genreName": "Programming", "bookCount": 3 },
  { "genreName": "History", "bookCount": 1 },
  { "genreName": "Self-Help", "bookCount": 1 }
]
```

---

### When to Use Views vs. Direct Entities

| Scenario | Use What | Why |
|----------|----------|-----|
| Full CRUD operations (Create, Read, Update, Delete) | Direct Entity | Views are usually read-only |
| Read-only reports/dashboards | View | Filter and shape data |
| Hide internal fields from external users | View | Security: don't expose costPrice, margin |
| Flatten nested associations for simple display | View | No need for complex $expand |
| Different APIs for different consumers (mobile vs web) | Multiple Views | Each view shows what that consumer needs |
| Aggregate data (counts, sums, averages) | View | Group by and count |

---

### Views Are Read-Only (Usually)

Important: By default, CDS views are **read-only**. You can SELECT from them, but you cannot INSERT/UPDATE/DELETE through them.

```cds
// This view is READ-ONLY:
entity ProductCatalog as select from Products {
  ID, name, price
};

// You CAN: GET /catalog/ProductCatalog
// You CANNOT: POST /catalog/ProductCatalog ← Error!
```

If you need write access, use a **projection** instead (covered next!).

---

## Session 2: Projections & Calculated Fields (10:45 - 12:00)

### What is a Projection?

A **projection** is like a view but with an important difference — it maintains a write-back connection to the original entity. This means you CAN create, update, and delete through a projection.

**View** = "Show me this data" (read-only window)
**Projection** = "Give me this shape for full CRUD" (read + write)

```cds
// VIEW — read-only:
entity BookList as select from Books { ID, title, price };

// PROJECTION — full CRUD:
entity BookCatalog as projection on Books { ID, title, price, stock };
```

---

### Projection Syntax

```cds
entity <Name> as projection on <SourceEntity> {
  field1,
  field2,
  field3
}
```

Or exclude specific fields instead of listing what to include:

```cds
// Include everything EXCEPT these fields:
entity PublicBooks as projection on Books excluding {
  costPrice,
  margin,
  supplier
};
```

---

### Projections in Services — The Most Common Use

You've already been using projections! Every time you wrote this in a service, that's a projection:

```cds
service LibraryService {
  entity Books as projection on management.Books;
}
```

This is the most common pattern in CAP:
1. Define your full entity in `db/schema.cds` (all fields, all relationships)
2. Expose a projection in `srv/service.cds` (maybe excluding internal fields)

---

### Projection Use Cases

#### Use Case 1: Hide Fields from the API

```cds
// In db/schema.cds — ALL fields:
entity Products : cuid, managed {
  name        : String(100);
  price       : Decimal(10,2);
  costPrice   : Decimal(10,2);    // INTERNAL — don't expose!
  margin      : Decimal(5,2);     // INTERNAL — don't expose!
  stock       : Integer;
  supplier    : Association to Suppliers;
}

// In srv/service.cds — expose only what's safe:
service CatalogService {
  // Public API — hides costPrice, margin:
  entity Products as projection on db.Products excluding {
    costPrice,
    margin
  };
}
```

Now the API user will NEVER see `costPrice` or `margin`. Those fields only exist in the database for internal use.

---

#### Use Case 2: Different Services, Different Views

The same entity can be exposed differently in different services:

```cds
// Customer-facing service — limited fields:
service ShopService {
  @readonly
  entity Products as projection on db.Products {
    ID,
    name,
    price,
    rating,
    image
  };
}

// Admin service — full access:
service AdminService {
  entity Products as projection on db.Products;  // ALL fields
}
```

**Result:**
- `/shop/Products` → returns only 5 fields, read-only
- `/admin/Products` → returns ALL fields, full CRUD

Same database table, two completely different APIs!

---

#### Use Case 3: Read-Only Entities

Use `@readonly` to make an entity read-only in a service (even if it's a projection):

```cds
service ReportService {
  @readonly entity Books as projection on management.Books;
  @readonly entity Authors as projection on management.Authors;
}
```

Now you can GET data but not POST/PUT/DELETE. Useful for reporting APIs.

---

### Calculated Fields — Computing Values On the Fly

**Calculated fields** are fields whose value is computed from other fields. They don't exist in the database — they're generated when data is read.

#### Simple Calculated Fields in CDS Views

```cds
entity BookPricing as select from Books {
  ID,
  title,
  price,
  // Calculate discounted price (20% off):
  (price * 0.8) as discountedPrice : Decimal(10,2),
  // Calculate tax (18% GST):
  (price * 0.18) as taxAmount : Decimal(10,2),
  // Calculate total with tax:
  (price * 1.18) as priceWithTax : Decimal(10,2)
};
```

**Result:**
```json
{
  "title": "Clean Code",
  "price": 450.00,
  "discountedPrice": 360.00,
  "taxAmount": 81.00,
  "priceWithTax": 531.00
}
```

The database only stores `price` (450). The other three values are calculated every time you read the data!

---

#### String Concatenation

```cds
entity AuthorList as select from Authors {
  ID,
  // Combine first + last name:
  (firstName || ' ' || lastName) as fullName : String(101)
};
```

**Result:**
```json
{ "fullName": "Robert Martin" }
```

The `||` operator joins strings together (concatenation).

---

#### CASE Expressions (Conditional Logic)

```cds
entity BookStatus as select from Books {
  ID,
  title,
  availableCopies,
  // Calculate status based on available copies:
  case
    when availableCopies = 0 then 'Out of Stock'
    when availableCopies <= 2 then 'Low Stock'
    else 'Available'
  end as availability : String(20)
};
```

**Result:**
```json
[
  { "title": "Clean Code", "availableCopies": 3, "availability": "Available" },
  { "title": "Design Patterns", "availableCopies": 1, "availability": "Low Stock" },
  { "title": "Algorithms", "availableCopies": 0, "availability": "Out of Stock" }
]
```

---

#### Date/Time Calculations

```cds
entity OverdueBorrowings as select from Borrowings {
  ID,
  book.title as bookTitle,
  member.firstName as memberName,
  borrowDate,
  dueDate,
  status
} where status = 'Overdue';
```

---

### Calculated Fields in Handlers (JavaScript)

For complex calculations that CDS can't express, use JavaScript handlers:

In `db/schema.cds`:
```cds
entity Products : cuid, managed {
  name         : String(100);
  price        : Decimal(10,2);
  costPrice    : Decimal(10,2);
  stock        : Integer;
  minStock     : Integer;
  // Virtual = NOT stored in DB, calculated at runtime:
  virtual profitMargin   : Decimal(5,2);
  virtual stockStatus    : String(20);
  virtual totalValue     : Decimal(12,2);
}
```

In `srv/service.js`:
```javascript
module.exports = (srv) => {

  srv.after('READ', 'Products', (results) => {
    for (const product of results) {
      // Calculate profit margin percentage:
      if (product.price && product.costPrice) {
        product.profitMargin = ((product.price - product.costPrice) / product.price * 100).toFixed(2);
      }
      
      // Calculate stock status:
      if (product.stock === 0) {
        product.stockStatus = 'Out of Stock';
      } else if (product.stock <= product.minStock) {
        product.stockStatus = 'Reorder Needed';
      } else {
        product.stockStatus = 'Healthy';
      }
      
      // Calculate total inventory value:
      product.totalValue = product.price * product.stock;
    }
  });

};
```

**When to use `virtual` fields vs. CDS expressions:**

| Feature | CDS Expressions | Virtual Fields (JS) |
|---------|----------------|---------------------|
| Simple math (+, -, *, /) | Use CDS | Overkill |
| String concatenation | Use CDS | Overkill |
| CASE/WHEN logic | Use CDS | Either works |
| Complex business logic | Can't do | Use JS |
| External API calls | Can't do | Use JS |
| Conditional logic with multiple entities | Limited | Use JS |

---

### Combining Views, Projections, and Calculated Fields

Here's a complete real-world example putting it all together:

```cds
namespace com.bookshop;
using { cuid, managed } from '@sap/cds/common';

// ─── BASE ENTITIES (in db/schema.cds) ────────────
entity Books : cuid, managed {
  title     : String(200);
  price     : Decimal(8,2);
  stock     : Integer;
  author    : Association to Authors;
  genre     : Association to Genres;
  reviews   : Composition of many Reviews on reviews.book = $self;
}

entity Authors : cuid, managed {
  firstName : String(50);
  lastName  : String(50);
  books     : Association to many Books on books.author = $self;
}

entity Genres {
  key code  : String(20);
  name      : String(50);
}

entity Reviews : cuid {
  book      : Association to Books;
  rating    : Integer;
  comment   : String(500);
}

// ─── VIEWS (in db/schema.cds or srv/) ────────────

// View 1: Book catalog for customers (read-only, flat)
entity BookCatalogView as select from Books {
  ID,
  title,
  price,
  stock,
  (price * 0.9) as salePrice : Decimal(8,2),
  author.firstName || ' ' || author.lastName as authorName : String(101),
  genre.name as genreName,
  case
    when stock > 5 then 'In Stock'
    when stock > 0 then 'Limited'
    else 'Sold Out'
  end as availability : String(20)
} where stock > 0;

// View 2: Author statistics
entity AuthorStats as select from Authors {
  ID,
  firstName || ' ' || lastName as fullName : String(101),
  count(books.ID) as totalBooks : Integer
} group by ID, firstName, lastName;
```

---

## Session 3: Hands-on — Build Views & Projections (12:00 - 13:00)

### Exercise: Add Views to Your Library Project

Open your Library Management project from Day 13.

#### Step 1: Create a New Views File

Create `db/views.cds`:

```cds
using lib.management from './schema';

// ─── VIEW 1: Available Books (for library kiosk) ───
entity AvailableBooks as select from management.Books {
  ID,
  title,
  isbn,
  price,
  availableCopies,
  author.firstName || ' ' || author.lastName as authorName : String(101),
  genre.name as genreName
} where availableCopies > 0;

// ─── VIEW 2: Overdue Borrowings (for staff) ────────
entity OverdueBorrowings as select from management.Borrowings {
  ID,
  borrowDate,
  dueDate,
  fineAmount,
  book.title       as bookTitle,
  member.firstName || ' ' || member.lastName as memberName : String(101),
  member.email     as memberEmail,
  member.phone     as memberPhone
} where status = 'Overdue';

// ─── VIEW 3: Book Summary with pricing ─────────────
entity BookPricing as select from management.Books {
  ID,
  title,
  price,
  (price * 0.9)  as memberPrice  : Decimal(8,2),
  (price * 0.18) as gstAmount    : Decimal(8,2),
  (price * 1.18) as priceWithGST : Decimal(8,2),
  case
    when price > 500 then 'Premium'
    when price > 300 then 'Standard'
    else 'Budget'
  end as priceCategory : String(20)
};

// ─── VIEW 4: Member Activity ───────────────────────
entity MemberActivity as select from management.Members {
  ID,
  memberNumber,
  firstName || ' ' || lastName as fullName : String(101),
  email,
  memberType,
  maxBooks,
  isActive
};
```

---

#### Step 2: Expose Views in a Separate Reporting Service

Create `srv/reporting-service.cds`:

```cds
using from '../db/views';

service ReportingService @(path: '/reports') {
  @readonly entity AvailableBooks   as projection on AvailableBooks;
  @readonly entity OverdueBorrowings as projection on OverdueBorrowings;
  @readonly entity BookPricing      as projection on BookPricing;
  @readonly entity MemberActivity   as projection on MemberActivity;
}
```

#### Step 3: Update Main Service with Projections

Update `srv/library-service.cds` to use exclusions:

```cds
using lib.management from '../db/schema';

service LibraryService @(path: '/library') {
  // Full CRUD:
  entity Books      as projection on management.Books;
  entity Authors    as projection on management.Authors;
  entity Genres     as projection on management.Genres;
  entity Members    as projection on management.Members;
  entity Borrowings as projection on management.Borrowings;
  entity Reviews    as projection on management.Reviews;
  entity PurchaseOrders as projection on management.PurchaseOrders;
}
```

#### Step 4: Test

```bash
cds watch
```

Test the reporting service:

```http
// Available books with calculated fields:
GET http://localhost:4004/reports/AvailableBooks

// Overdue borrowings with member contact info:
GET http://localhost:4004/reports/OverdueBorrowings

// Book pricing with GST calculations:
GET http://localhost:4004/reports/BookPricing

// Filter premium books only:
GET http://localhost:4004/reports/BookPricing?$filter=priceCategory eq 'Premium'

// Member activity:
GET http://localhost:4004/reports/MemberActivity?$filter=isActive eq true
```

---

## Session 4: CSV Data Loading — The Complete Guide (13:45 - 14:45)

### Why CSV Data?

When you build an application, you often need **sample data** for:
- Testing during development ("does my filter work?")
- Demos to stakeholders ("look, the app has real-looking data!")
- Master data that should always exist (countries, currencies, statuses)
- Initial configuration (default settings, roles, categories)

CAP has a built-in mechanism to load CSV files into your database automatically. No SQL scripts needed!

---

### How CSV Loading Works in CAP

```
┌──────────────────┐     ┌──────────────┐     ┌──────────────┐
│  Your CSV files  │ ──► │  cds deploy  │ ──► │  Database    │
│  in db/data/     │     │  (or cds     │     │  (SQLite/    │
│                  │     │   watch)     │     │   HANA)      │
└──────────────────┘     └──────────────┘     └──────────────┘
```

1. You create CSV files in the `db/data/` folder
2. When `cds watch` starts (or `cds deploy` runs), CAP:
   - Reads all CSV files from `db/data/`
   - Matches them to entities by file name
   - Inserts the data into the corresponding database tables
3. Data is available immediately via API!

---

### CSV File Naming Convention — CRITICAL!

The file name MUST match this exact pattern:

```
<namespace>-<EntityName>.csv
```

Replace dots (`.`) in the namespace with dashes (`-`) for safety, OR keep dots — both work:

| Namespace | Entity | File Name |
|-----------|--------|-----------|
| `my.bookshop` | `Books` | `my.bookshop-Books.csv` |
| `lib.management` | `Authors` | `lib.management-Authors.csv` |
| `com.epm` | `Products` | `com.epm-Products.csv` |
| `sap.common` | `Currencies` | `sap.common-Currencies.csv` |

**If the name doesn't match exactly, CAP won't load the data!** This is the #1 mistake beginners make.

---

### Finding the Correct File Name

Not sure what to name your CSV file? Use this command:

```bash
cds compile db/schema.cds --to sql
```

This shows you the full table names. Use that name (replacing `_` with `-` and adding `.csv`).

Or simply remember the rule:
```
File name = namespace-EntityName.csv
           (exactly as written in your CDS, case-sensitive!)
```

---

### CSV File Format Rules

CAP CSV files follow specific rules that differ from standard Excel CSVs:

| Rule | Detail | Example |
|------|--------|---------|
| **Separator** | Semicolons (`;`) NOT commas | `ID;name;price` |
| **First row** | Column headers (must match CDS field names) | `ID;title;price` |
| **Encoding** | UTF-8 | Save as UTF-8 in your editor |
| **Empty values** | Just leave blank between separators | `id1;;500` (name is empty) |
| **Boolean** | `true` or `false` (lowercase) | `isActive` → `true` |
| **Dates** | ISO format: `YYYY-MM-DD` | `2026-06-01` |
| **DateTime** | ISO format: `YYYY-MM-DDTHH:MM:SSZ` | `2026-06-01T09:30:00Z` |
| **Decimal** | Use dot (.) as decimal separator | `450.99` |
| **UUIDs** | Full 36-character format | `a1b2c3d4-e5f6-...` |
| **Association fields** | Use the FK column name (`_ID` suffix) | `author_ID` (not `author`) |
| **Null** | Just empty | `;;` means null |
| **Strings with semicolons** | Wrap in double quotes | `"Hello; World"` |
| **Strings with quotes** | Escape with double quotes | `"He said ""hi"""` |

---

### CSV Data for Associations — How Foreign Keys Work

This is where beginners get confused. When you have associations, the CSV must use the **foreign key column name**, not the association field name.

```cds
entity Books : cuid {
  title  : String(200);
  author : Association to Authors;  // CDS field name: "author"
  genre  : Association to Genres;   // CDS field name: "genre"
}
```

In the database, CAP creates:
- `author_ID` column (UUID foreign key)
- `genre_code` column (if Genres has `key code : String`)

So your CSV header must use:
```csv
ID;title;author_ID;genre_code
```

NOT:
```csv
ID;title;author;genre     ← WRONG! These are association names, not column names
```

---

### How to Determine FK Column Names

The foreign key column name follows this pattern:

| Association Definition | Key Field of Target | FK Column Name |
|----------------------|-------------------|----------------|
| `author : Association to Authors` | `key ID : UUID` | `author_ID` |
| `genre : Association to Genres` | `key code : String` | `genre_code` |
| `country : Country` | `key code : String(2)` | `country_code` |
| `currency : Currency` | `key code : String(3)` | `currency_code` |
| `category : Association to Categories` | `key ID : UUID` | `category_ID` |

**Pattern:** `<fieldName>_<keyFieldName>`

---

### Complete CSV Data Example Set

Let's build a complete set of CSVs for a bookshop:

#### `db/data/my.bookshop-Authors.csv`

```csv
ID;firstName;lastName;nationality;birthDate;email;isActive
a1a1a1a1-1111-1111-1111-111111111111;Robert;Martin;American;1952-12-05;rmartin@email.com;true
a2a2a2a2-2222-2222-2222-222222222222;Martin;Fowler;British;1963-12-18;mfowler@email.com;true
a3a3a3a3-3333-3333-3333-333333333333;Yuval Noah;Harari;Israeli;1976-02-24;yharari@email.com;true
a4a4a4a4-4444-4444-4444-444444444444;James;Clear;American;1986-01-22;jclear@email.com;true
a5a5a5a5-5555-5555-5555-555555555555;Joshua;Bloch;American;1961-08-28;jbloch@email.com;true
```

**Tips for this CSV:**
- UUIDs are made up but consistent (easy to reference in other CSVs)
- Dates are ISO format (YYYY-MM-DD)
- Boolean values are lowercase (`true`/`false`)

---

#### `db/data/my.bookshop-Genres.csv`

```csv
code;name;description;isActive
PROG;Programming;Books about software development;true
ARCH;Architecture;Software design and architecture;true
HIST;History;Books about historical events;true
SELF;Self-Help;Personal development books;true
SCIF;Science Fiction;Futuristic fiction;true
BUSI;Business;Management and entrepreneurship;true
```

**Notice:** `Genres` has `key code : String(20)`, so the key column is `code`, not `ID`.

---

#### `db/data/my.bookshop-Books.csv`

```csv
ID;title;isbn;pages;price;publishedDate;language;edition;stock;author_ID;genre_code
b1b1b1b1-1111-1111-1111-111111111111;Clean Code;9780132350884;464;450.00;2008-08-01;English;1;5;a1a1a1a1-1111-1111-1111-111111111111;PROG
b2b2b2b2-2222-2222-2222-222222222222;Refactoring;9780201485677;431;550.00;1999-07-08;English;2;3;a2a2a2a2-2222-2222-2222-222222222222;ARCH
b3b3b3b3-3333-3333-3333-333333333333;Sapiens;9780062316097;443;400.00;2015-02-10;English;1;8;a3a3a3a3-3333-3333-3333-333333333333;HIST
b4b4b4b4-4444-4444-4444-444444444444;Atomic Habits;9780735211292;320;350.00;2018-10-16;English;1;12;a4a4a4a4-4444-4444-4444-444444444444;SELF
b5b5b5b5-5555-5555-5555-555555555555;Effective Java;9780134685991;416;600.00;2018-01-06;English;3;4;a5a5a5a5-5555-5555-5555-555555555555;PROG
```

**Key points:**
- `author_ID` references the UUID from Authors.csv
- `genre_code` references the code from Genres.csv
- These MUST match values that exist in the referenced CSVs!

---

#### `db/data/sap.common-Currencies.csv`

```csv
code;name;descr;symbol
INR;Indian Rupee;Currency of India;₹
USD;US Dollar;Currency of United States;$
EUR;Euro;Currency of European Union;€
GBP;British Pound;Currency of United Kingdom;£
JPY;Japanese Yen;Currency of Japan;¥
```

**This file provides data for `@sap/cds/common`'s built-in Currencies entity.**

---

#### `db/data/sap.common-Countries.csv`

```csv
code;name;descr
IN;India;Republic of India
US;United States;United States of America
DE;Germany;Federal Republic of Germany
GB;United Kingdom;United Kingdom of Great Britain
JP;Japan;Japan
FR;France;French Republic
```

---

### Common CSV Mistakes and How to Fix Them

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Wrong file name | Data not loaded, no error! | Check `namespace-Entity.csv` exactly |
| Using commas instead of semicolons | Parse errors, data in wrong columns | Replace `,` with `;` |
| Wrong FK column name (`author` instead of `author_ID`) | Column not found error | Use `<field>_<key>` pattern |
| UUID mismatch (referencing non-existent ID) | FK violation or orphan records | Ensure referenced IDs exist in target CSV |
| Date format wrong (`01-06-2026`) | Invalid date | Use `2026-06-01` (ISO) |
| Extra spaces in headers | Column not matched | Trim spaces: `ID;name` not `ID; name` |
| BOM character (saving from Windows) | First column not matched | Save as UTF-8 without BOM |
| File in wrong folder | Not discovered by CAP | Must be in `db/data/` (or `db/csv/`) |

---

### Where to Place CSV Files

CAP looks for CSV files in these locations (in order of priority):

```
project/
├── db/
│   ├── data/                ← PRIMARY location (recommended)
│   │   ├── my.bookshop-Books.csv
│   │   ├── my.bookshop-Authors.csv
│   │   └── sap.common-Currencies.csv
│   ├── csv/                 ← ALTERNATIVE location (also works)
│   │   └── ...
│   └── schema.cds
└── ...
```

**Recommendation:** Always use `db/data/` — it's the standard convention.

---

### Test Data vs. Production Data

You can have separate data for test and production:

```
project/
├── db/
│   └── data/              ← Loaded in ALL environments
│       └── sap.common-Currencies.csv   (always needed)
├── test/
│   └── data/              ← Loaded ONLY during testing
│       └── my.bookshop-Books.csv       (sample test data)
```

Configure in `package.json`:
```json
{
  "cds": {
    "requires": {
      "db": {
        "[development]": {
          "kind": "sqlite",
          "credentials": { "database": ":memory:" }
        }
      }
    }
  }
}
```

---

## Session 5: cds deploy, SQLite Inspection & Build Process (15:00 - 16:00)

### Understanding `cds deploy`

The `cds deploy` command takes your CDS model and CSV data, and creates a real database from them.

```bash
cds deploy --to sqlite:db/my-data.db
```

**What happens step by step:**

```
Step 1: CDS Compilation
─────────────────────
Your .cds files → Compiled to SQL DDL (CREATE TABLE statements)

Step 2: Database Creation
─────────────────────────
SQL DDL → Creates SQLite file → Creates all tables

Step 3: Data Loading
────────────────────
CSV files → Read and parsed → INSERT INTO each table

Step 4: Done!
─────────────
db/my-data.db is ready with schema + data
```

---

### `cds deploy` Options

| Command | What It Does |
|---------|-------------|
| `cds deploy --to sqlite` | Deploy to in-memory SQLite (lost when command ends) |
| `cds deploy --to sqlite:db/data.db` | Deploy to a persistent SQLite file |
| `cds deploy --to hana` | Deploy to SAP HANA (production) |
| `cds deploy --with-mocks` | Add mock data for missing associations |
| `cds deploy --dry` | Show what SQL would be generated, don't execute |

---

### `cds deploy` vs `cds watch`

| Feature | `cds watch` | `cds deploy` |
|---------|------------|--------------|
| Creates database? | Yes (in-memory by default) | Yes (to file or HANA) |
| Loads CSV data? | Yes | Yes |
| Starts a server? | Yes (with API endpoints) | No (just deploys) |
| Data persists? | No (lost on restart, unless configured) | Yes (saved to file) |
| Use case | Development & testing | Persistence & production |
| Hot reload? | Yes (watches file changes) | No (run once) |

**Most of the time during development, `cds watch` is all you need.** It creates an in-memory database, loads CSVs, and starts the server — all in one command.

Use `cds deploy` when you want to:
- Create a persistent SQLite file you can inspect with SQLite tools
- Deploy to SAP HANA for production
- Verify that your model compiles and data loads without errors

---

### Inspecting SQLite Database

After deploying to a SQLite file, you can inspect it directly:

#### Step 1: Deploy to a file

```bash
cds deploy --to sqlite:db/library.db
```

#### Step 2: Open with SQLite CLI

```bash
sqlite3 db/library.db
```

#### Step 3: Useful SQLite Commands

```sql
-- List all tables:
.tables

-- Show table structure (columns, types):
.schema lib_management_Books

-- Count rows:
SELECT COUNT(*) FROM lib_management_Books;

-- View all data:
SELECT * FROM lib_management_Books;

-- View specific columns:
SELECT title, price, author_ID FROM lib_management_Books;

-- Check foreign key data:
SELECT b.title, a.firstName || ' ' || a.lastName as author
FROM lib_management_Books b
JOIN lib_management_Authors a ON b.author_ID = a.ID;

-- Pretty print mode:
.mode column
.headers on
SELECT title, price FROM lib_management_Books;

-- Exit:
.exit
```

---

### Understanding Namespace → Table Name Mapping

When CAP creates database tables, it converts your namespace:

| CDS Definition | Database Table Name |
|---------------|-------------------|
| `namespace my.bookshop; entity Books` | `my_bookshop_Books` |
| `namespace lib.management; entity Authors` | `lib_management_Authors` |
| `namespace com.epm; entity Products` | `com_epm_Products` |
| `sap.common.Currencies` | `sap_common_Currencies` |

**Pattern:** Dots (`.`) become underscores (`_`) in table names.

---

### The CDS Build Process — From Source to Running App

Understanding the full build process helps you debug issues:

```
┌─────────────────────────────────────────────────────────┐
│  1. CDS SOURCE FILES                                     │
│  ─────────────────                                       │
│  db/schema.cds + srv/service.cds                        │
│  (Your human-readable CDS definitions)                  │
└────────────────────────┬────────────────────────────────┘
                         │ cds compile
                         ▼
┌─────────────────────────────────────────────────────────┐
│  2. CSN (CDS Schema Notation) — JSON format             │
│  ─────────────────────────────────────────              │
│  Internal representation of your model                  │
│  (machine-readable, used by the runtime)                │
└────────────────────────┬────────────────────────────────┘
                         │ cds compile --to sql / --to hana
                         ▼
┌─────────────────────────────────────────────────────────┐
│  3. SQL DDL (CREATE TABLE statements)                   │
│  ─────────────────────────────────────                  │
│  Native database language                               │
│  (SQLite SQL or HANA SQL)                              │
└────────────────────────┬────────────────────────────────┘
                         │ cds deploy
                         ▼
┌─────────────────────────────────────────────────────────┐
│  4. DATABASE (Tables + Data)                            │
│  ───────────────────────────                            │
│  Tables created, CSV data loaded                        │
│  (SQLite file or HANA database)                        │
└────────────────────────┬────────────────────────────────┘
                         │ cds serve / cds watch
                         ▼
┌─────────────────────────────────────────────────────────┐
│  5. RUNNING APPLICATION                                 │
│  ──────────────────────                                 │
│  OData/REST API endpoints                              │
│  (Accessible at http://localhost:4004)                  │
└─────────────────────────────────────────────────────────┘
```

---

### `cds compile` — Seeing What CAP Generates

Use `cds compile` to see intermediate outputs at each stage:

```bash
# See the compiled JSON model (CSN):
cds compile db/schema.cds --to json

# See the SQL that would be generated:
cds compile db/schema.cds --to sql

# See HANA-specific artifacts:
cds compile db/schema.cds --to hana

# See the OData metadata (service definition):
cds compile srv/service.cds --to edmx

# See everything compiled together:
cds compile srv/ --to json
```

#### Example: CDS → SQL output

```bash
cds compile db/schema.cds --to sql
```

Output:
```sql
CREATE TABLE lib_management_Books (
  ID NVARCHAR(36) NOT NULL,
  title NVARCHAR(200),
  isbn NVARCHAR(13),
  pages INTEGER,
  price DECIMAL(8,2),
  publishedDate DATE_TEXT,
  language NVARCHAR(30),
  edition INTEGER,
  totalCopies INTEGER,
  availableCopies INTEGER,
  summary NVARCHAR(2000),
  author_ID NVARCHAR(36),
  genre_code NVARCHAR(20),
  createdAt TIMESTAMP_TEXT,
  createdBy NVARCHAR(255),
  modifiedAt TIMESTAMP_TEXT,
  modifiedBy NVARCHAR(255),
  PRIMARY KEY(ID)
);
```

**This is incredibly useful for debugging!** If your CSV isn't loading, check the compiled SQL to see the exact column names.

---

### `cds build` — Production Build

For production deployment, use `cds build`:

```bash
cds build
```

This creates a `gen/` folder with everything needed for deployment:

```
gen/
├── db/                    ← Database artifacts
│   ├── src/              ← Compiled SQL
│   └── csv/             ← CSV data files
└── srv/                  ← Service artifacts
    ├── srv/             ← Compiled service
    └── node_modules/    ← Dependencies
```

You don't need `cds build` for local development — `cds watch` handles everything. Use it when preparing for production deployment to BTP.

---

### Verifying Your Deployment — Checklist

After `cds deploy --to sqlite:db/data.db`, verify everything:

```bash
# 1. Check tables were created:
sqlite3 db/data.db ".tables"

# 2. Check a specific table's schema:
sqlite3 db/data.db ".schema lib_management_Books"

# 3. Verify data was loaded:
sqlite3 db/data.db "SELECT COUNT(*) as total FROM lib_management_Books;"

# 4. Spot-check actual data:
sqlite3 db/data.db "SELECT title, price FROM lib_management_Books LIMIT 3;"

# 5. Verify foreign keys populated:
sqlite3 db/data.db "SELECT title, author_ID FROM lib_management_Books;"

# 6. Join check (verify relationships):
sqlite3 db/data.db "
  SELECT b.title, a.firstName || ' ' || a.lastName as author
  FROM lib_management_Books b
  JOIN lib_management_Authors a ON b.author_ID = a.ID;
"
```

---

## Session 6: Hands-on — Complete EPM with Full Sample Data (16:00 - 16:45)

### Goal: Build a Complete EPM Model with Views, Projections, and CSV Data

Let's bring everything from Days 12-14 together in one complete project.

#### Step 1: Open (or Create) the EPM Project

```bash
cd ~/cap-training/epm-model
# or: cds init epm-model && cd epm-model && npm install
```

#### Step 2: Complete Data Model with Views

Create/update `db/schema.cds`:

```cds
using { cuid, managed, Currency, Country } from '@sap/cds/common';

namespace com.epm;

// ─── SUPPLIERS ───────────────────────────────────
entity Suppliers : cuid, managed {
  supplierName  : String(100);
  contactPerson : String(100);
  email         : String(255);
  phone         : String(20);
  city          : String(100);
  country       : Country;
  isActive      : Boolean default true;
  products      : Association to many Products on products.supplier = $self;
}

// ─── CATEGORIES ──────────────────────────────────
entity Categories : cuid, managed {
  categoryName  : String(100);
  description   : String(500);
  products      : Association to many Products on products.category = $self;
}

// ─── PRODUCTS ────────────────────────────────────
entity Products : cuid, managed {
  productName   : String(100);
  description   : String(500);
  price         : Decimal(10,2);
  currency      : Currency;
  stock         : Integer default 0;
  minStock      : Integer default 10;
  rating        : Decimal(2,1);
  supplier      : Association to Suppliers;
  category      : Association to Categories;
  isAvailable   : Boolean default true;
}

// ─── CUSTOMERS ───────────────────────────────────
entity Customers : cuid, managed {
  customerName  : String(100);
  email         : String(255);
  phone         : String(20);
  city          : String(100);
  country       : Country;
  creditLimit   : Decimal(12,2) default 100000;
  orders        : Association to many SalesOrders on orders.customer = $self;
}

// ─── SALES ORDERS ────────────────────────────────
entity SalesOrders : cuid, managed {
  orderNumber    : String(20);
  customer       : Association to Customers;
  orderDate      : Date;
  grossAmount    : Decimal(12,2);
  netAmount      : Decimal(12,2);
  taxAmount      : Decimal(10,2);
  currency       : Currency;
  status         : String(20) default 'New';
  items          : Composition of many SalesOrderItems on items.order = $self;
}

// ─── SALES ORDER ITEMS ───────────────────────────
entity SalesOrderItems : cuid {
  order          : Association to SalesOrders;
  product        : Association to Products;
  quantity       : Integer;
  unitPrice      : Decimal(10,2);
  netAmount      : Decimal(12,2);
  currency       : Currency;
}

// ═══════════════════════════════════════════════════
// VIEWS — Read-only analytical perspectives
// ═══════════════════════════════════════════════════

// View: Product Catalog (customer-facing, flat)
entity ProductCatalog as select from Products {
  ID,
  productName,
  description,
  price,
  currency,
  rating,
  stock,
  (price * 0.82) as priceExTax : Decimal(10,2),
  (price * 0.18) as taxAmount  : Decimal(10,2),
  supplier.supplierName as supplierName,
  category.categoryName as categoryName,
  case
    when stock > 20 then 'In Stock'
    when stock > 0  then 'Low Stock'
    else 'Out of Stock'
  end as availability : String(20)
} where isAvailable = true;

// View: Order Summary (flattened for reports)
entity OrderSummary as select from SalesOrders {
  ID,
  orderNumber,
  orderDate,
  grossAmount,
  netAmount,
  taxAmount,
  status,
  customer.customerName as customerName,
  customer.email        as customerEmail,
  customer.city         as customerCity
};

// View: Low Stock Alert
entity LowStockProducts as select from Products {
  ID,
  productName,
  stock,
  minStock,
  supplier.supplierName as supplierName,
  supplier.email        as supplierEmail,
  supplier.phone        as supplierPhone
} where stock <= minStock and isAvailable = true;
```

---

#### Step 3: Create Services

Create `srv/sales-service.cds`:

```cds
using com.epm from '../db/schema';

service SalesService @(path: '/sales') {
  entity Products     as projection on epm.Products;
  entity Categories   as projection on epm.Categories;
  entity Customers    as projection on epm.Customers;
  entity SalesOrders  as projection on epm.SalesOrders;
}

service ReportsService @(path: '/reports') {
  @readonly entity ProductCatalog    as projection on epm.ProductCatalog;
  @readonly entity OrderSummary      as projection on epm.OrderSummary;
  @readonly entity LowStockProducts  as projection on epm.LowStockProducts;
}
```

---

#### Step 4: Create Complete CSV Data Set

Create `db/data/sap.common-Currencies.csv`:
```csv
code;name;descr;symbol
INR;Indian Rupee;Currency of India;₹
USD;US Dollar;Currency of United States;$
EUR;Euro;Currency of European Union;€
GBP;British Pound;Currency of United Kingdom;£
```

Create `db/data/sap.common-Countries.csv`:
```csv
code;name;descr
IN;India;Republic of India
US;United States;United States of America
DE;Germany;Federal Republic of Germany
JP;Japan;Japan
SG;Singapore;Republic of Singapore
```

Create `db/data/com.epm-Suppliers.csv`:
```csv
ID;supplierName;contactPerson;email;phone;city;country_code;isActive
s001s001-0001-0001-0001-000000000001;TechParts India;Amit Shah;amit@techparts.in;+91-9876543210;Bangalore;IN;true
s002s002-0002-0002-0002-000000000002;Global Electronics;Sarah Johnson;sarah@globalelec.com;+1-555-0123;San Jose;US;true
s003s003-0003-0003-0003-000000000003;Tokyo Components;Yuki Tanaka;yuki@tokyocomp.jp;+81-3-1234-5678;Tokyo;JP;true
s004s004-0004-0004-0004-000000000004;Berlin Hardware;Klaus Schmidt;klaus@berlinhw.de;+49-30-12345;Berlin;DE;true
```

Create `db/data/com.epm-Categories.csv`:
```csv
ID;categoryName;description
c001c001-0001-0001-0001-000000000001;Electronics;Electronic devices and accessories
c002c002-0002-0002-0002-000000000002;Office Supplies;Paper, pens, and office equipment
c003c003-0003-0003-0003-000000000003;Furniture;Desks, chairs, and storage
c004c004-0004-0004-0004-000000000004;Software;Licenses and subscriptions
c005c005-0005-0005-0005-000000000005;Networking;Routers, switches, and cables
```

Create `db/data/com.epm-Products.csv`:
```csv
ID;productName;description;price;currency_code;stock;minStock;rating;supplier_ID;category_ID;isAvailable
p001p001-0001-0001-0001-000000000001;Laptop Pro 15;15-inch high performance laptop;85000.00;INR;45;10;4.5;s001s001-0001-0001-0001-000000000001;c001c001-0001-0001-0001-000000000001;true
p002p002-0002-0002-0002-000000000002;Wireless Mouse;Ergonomic bluetooth mouse;1500.00;INR;200;25;4.2;s002s002-0002-0002-0002-000000000002;c001c001-0001-0001-0001-000000000001;true
p003p003-0003-0003-0003-000000000003;Standing Desk;Height adjustable desk 150cm;28000.00;INR;8;5;4.7;s004s004-0004-0004-0004-000000000004;c003c003-0003-0003-0003-000000000003;true
p004p004-0004-0004-0004-000000000004;A4 Paper Ream;500 sheets white 80gsm;280.00;INR;450;100;3.8;s001s001-0001-0001-0001-000000000001;c002c002-0002-0002-0002-000000000002;true
p005p005-0005-0005-0005-000000000005;27 inch Monitor;4K IPS Display 144Hz;35000.00;INR;22;8;4.6;s003s003-0003-0003-0003-000000000003;c001c001-0001-0001-0001-000000000001;true
p006p006-0006-0006-0006-000000000006;Mechanical Keyboard;Cherry MX Blue switches;6500.00;INR;60;15;4.4;s003s003-0003-0003-0003-000000000003;c001c001-0001-0001-0001-000000000001;true
p007p007-0007-0007-0007-000000000007;Office Chair Pro;Ergonomic mesh chair;18000.00;INR;12;5;4.3;s004s004-0004-0004-0004-000000000004;c003c003-0003-0003-0003-000000000003;true
p008p008-0008-0008-0008-000000000008;Network Switch 24-port;Managed gigabit switch;15000.00;INR;5;10;4.1;s002s002-0002-0002-0002-000000000002;c005c005-0005-0005-0005-000000000005;true
```

Create `db/data/com.epm-Customers.csv`:
```csv
ID;customerName;email;phone;city;country_code;creditLimit
cu01cu01-0001-0001-0001-000000000001;Infosys Technologies;procurement@infosys.com;+91-80-12345678;Bangalore;IN;5000000.00
cu02cu02-0002-0002-0002-000000000002;TCS Digital;orders@tcs.com;+91-22-87654321;Mumbai;IN;3000000.00
cu03cu03-0003-0003-0003-000000000003;Wipro Ltd;supplies@wipro.com;+91-80-11223344;Bangalore;IN;4000000.00
cu04cu04-0004-0004-0004-000000000004;HCL Technologies;purchase@hcl.com;+91-120-9988776;Noida;IN;2500000.00
```

Create `db/data/com.epm-SalesOrders.csv`:
```csv
ID;orderNumber;customer_ID;orderDate;grossAmount;netAmount;taxAmount;currency_code;status
so01so01-0001-0001-0001-000000000001;SO-2026-001;cu01cu01-0001-0001-0001-000000000001;2026-05-15;100300.00;85000.00;15300.00;INR;Delivered
so02so02-0002-0002-0002-000000000002;SO-2026-002;cu02cu02-0002-0002-0002-000000000002;2026-05-20;38350.00;32500.00;5850.00;INR;Shipped
so03so03-0003-0003-0003-000000000003;SO-2026-003;cu03cu03-0003-0003-0003-000000000003;2026-05-28;62360.00;52848.00;9512.00;INR;New
so04so04-0004-0004-0004-000000000004;SO-2026-004;cu01cu01-0001-0001-0001-000000000001;2026-06-01;21240.00;18000.00;3240.00;INR;New
```

Create `db/data/com.epm-SalesOrderItems.csv`:
```csv
ID;order_ID;product_ID;quantity;unitPrice;netAmount;currency_code
si01si01-0001-0001-0001-000000000001;so01so01-0001-0001-0001-000000000001;p001p001-0001-0001-0001-000000000001;1;85000.00;85000.00;INR
si02si02-0002-0002-0002-000000000002;so01so01-0001-0001-0001-000000000001;p002p002-0002-0002-0002-000000000002;10;1500.00;15000.00;INR
si03si03-0003-0003-0003-000000000003;so02so02-0002-0002-0002-000000000002;p005p005-0005-0005-0005-000000000005;1;32000.00;32000.00;INR
si04si04-0004-0004-0004-000000000004;so02so02-0002-0002-0002-000000000002;p006p006-0006-0006-0006-000000000006;1;6500.00;6500.00;INR
si05si05-0005-0005-0005-000000000005;so03so03-0003-0003-0003-000000000003;p003p003-0003-0003-0003-000000000003;1;28000.00;28000.00;INR
si06si06-0006-0006-0006-000000000006;so03so03-0003-0003-0003-000000000003;p007p007-0007-0007-0007-000000000007;1;18000.00;18000.00;INR
si07si07-0007-0007-0007-000000000007;so04so04-0004-0004-0004-000000000004;p007p007-0007-0007-0007-000000000007;1;18000.00;18000.00;INR
```

---

#### Step 5: Deploy and Verify

```bash
# Deploy to persistent SQLite:
cds deploy --to sqlite:db/epm.db

# Verify tables:
sqlite3 db/epm.db ".tables"

# Expected output:
# com_epm_Categories       com_epm_Products
# com_epm_Customers        com_epm_SalesOrderItems
# com_epm_SalesOrders      com_epm_Suppliers
# sap_common_Countries     sap_common_Currencies

# Verify data counts:
sqlite3 db/epm.db "SELECT 'Suppliers:' , COUNT(*) FROM com_epm_Suppliers UNION ALL SELECT 'Products:', COUNT(*) FROM com_epm_Products UNION ALL SELECT 'Customers:', COUNT(*) FROM com_epm_Customers UNION ALL SELECT 'Orders:', COUNT(*) FROM com_epm_SalesOrders;"

# Check product → supplier join:
sqlite3 db/epm.db "
  .mode column
  .headers on
  SELECT p.productName, p.price, s.supplierName
  FROM com_epm_Products p
  JOIN com_epm_Suppliers s ON p.supplier_ID = s.ID
  LIMIT 5;
"

# Check order → customer join:
sqlite3 db/epm.db "
  .mode column
  .headers on
  SELECT o.orderNumber, o.grossAmount, c.customerName
  FROM com_epm_SalesOrders o
  JOIN com_epm_Customers c ON o.customer_ID = c.ID;
"
```

---

#### Step 6: Run and Test All Services

```bash
cds watch
```

**Test Sales Service:**
```http
GET http://localhost:4004/sales/Products
GET http://localhost:4004/sales/Products?$expand=supplier,category
GET http://localhost:4004/sales/Products?$filter=price gt 10000&$orderby=price desc
GET http://localhost:4004/sales/Customers?$expand=orders
GET http://localhost:4004/sales/SalesOrders?$expand=items($expand=product),customer
```

**Test Reports Service (views with calculated fields):**
```http
GET http://localhost:4004/reports/ProductCatalog
GET http://localhost:4004/reports/ProductCatalog?$filter=availability eq 'Low Stock'
GET http://localhost:4004/reports/OrderSummary
GET http://localhost:4004/reports/OrderSummary?$filter=status eq 'New'
GET http://localhost:4004/reports/LowStockProducts
```

**Test Deep Insert:**
```http
POST http://localhost:4004/sales/SalesOrders
Content-Type: application/json

{
  "orderNumber": "SO-2026-005",
  "customer_ID": "cu04cu04-0004-0004-0004-000000000004",
  "orderDate": "2026-06-02",
  "grossAmount": 7670,
  "netAmount": 6500,
  "taxAmount": 1170,
  "currency_code": "INR",
  "status": "New",
  "items": [
    {
      "product_ID": "p006p006-0006-0006-0006-000000000006",
      "quantity": 1,
      "unitPrice": 6500,
      "netAmount": 6500,
      "currency_code": "INR"
    }
  ]
}
```

---

## Assessment: MCQ (15 Questions)

**Q1.** A CDS View is best described as:
- a) A new database table with its own data
- b) A read-only "window" into existing data that reshapes or filters it
- c) A JavaScript function
- d) A backup copy of a table

**Answer: b)** — A view doesn't create a new table. It provides a different way to look at existing data — selecting specific fields, filtering rows, or computing new values on the fly.

---

**Q2.** The correct syntax for a CDS view that selects specific fields is:
- a) `entity MyView = select from Books { title, price };`
- b) `entity MyView as select from Books { title, price };`
- c) `view MyView as select Books.title, Books.price;`
- d) `create view MyView as Books(title, price);`

**Answer: b)** — CDS views use `entity <Name> as select from <Source> { fields }` syntax.

---

**Q3.** What does `as` do in `title as bookTitle` inside a view?
- a) Creates a new column in the database
- b) Renames the field in the view's output (aliasing)
- c) Creates an association
- d) Sets a default value

**Answer: b)** — `as` gives the field a new name (alias) in the view output. The database column name stays the same — only the API name changes.

---

**Q4.** What is the difference between a View and a Projection?
- a) They are exactly the same thing
- b) A view is read-only; a projection allows full CRUD (create, update, delete)
- c) A view is for the database; a projection is for JavaScript
- d) A projection is always faster

**Answer: b)** — Views are read-only windows into data. Projections maintain a write-back connection to the source entity, allowing full CRUD operations through them.

---

**Q5.** `entity PublicBooks as projection on Books excluding { costPrice, margin };` means:
- a) Only show costPrice and margin
- b) Show ALL fields EXCEPT costPrice and margin
- c) Delete costPrice and margin from the database
- d) Make costPrice and margin read-only

**Answer: b)** — The `excluding` keyword removes specific fields from the projection. All other fields pass through.

---

**Q6.** CSV files in CAP must use which separator?
- a) Commas (`,`)
- b) Tabs
- c) Semicolons (`;`)
- d) Pipes (`|`)

**Answer: c)** — CAP CSV files use semicolons (`;`) as separators. This is different from standard CSV (comma-separated) and catches many beginners off guard.

---

**Q7.** The correct file name for a CSV providing data for entity `Products` in namespace `com.epm` is:
- a) `Products.csv`
- b) `com.epm.Products.csv`
- c) `com.epm-Products.csv`
- d) `com_epm_Products.csv`

**Answer: c)** — Pattern is `<namespace>-<EntityName>.csv`. The namespace keeps its dots, and a dash separates it from the entity name.

---

**Q8.** If `entity Books` has `author : Association to Authors` (where Authors has `key ID : UUID`), what should the CSV column header be?
- a) `author`
- b) `author_ID`
- c) `Authors_ID`
- d) `authorId`

**Answer: b)** — For managed associations, the FK column is `<fieldName>_<targetKeyName>`. Since the field is `author` and the target key is `ID`, the column is `author_ID`.

---

**Q9.** Where should CSV data files be placed?
- a) In the project root folder
- b) In `srv/data/`
- c) In `db/data/`
- d) In `node_modules/`

**Answer: c)** — CAP automatically discovers CSV files in the `db/data/` folder (or `db/csv/` as an alternative).

---

**Q10.** `cds deploy --to sqlite:db/data.db` does what?
- a) Starts the development server
- b) Compiles CDS to SQL, creates a SQLite database file, creates tables, and loads CSV data
- c) Deploys to SAP BTP cloud
- d) Only creates tables, no data loading

**Answer: b)** — It's a four-step process: compile CDS → generate SQL → create database file with tables → load CSV data into tables.

---

**Q11.** What is the main difference between `cds watch` and `cds deploy`?
- a) `cds watch` also starts a server; `cds deploy` only creates the database
- b) `cds deploy` is faster
- c) `cds watch` doesn't load CSV data
- d) They are identical

**Answer: a)** — `cds watch` creates an in-memory database, loads data, AND starts the server with hot reload. `cds deploy` only creates/populates the database (no server).

---

**Q12.** A calculated field `(price * 0.18) as gst : Decimal(10,2)` in a view:
- a) Stores the GST value in the database
- b) Computes the value each time the view is queried (not stored)
- c) Requires a JavaScript handler
- d) Only works with HANA database

**Answer: b)** — Calculated fields in views are computed on-the-fly when data is read. No extra database column is created — it's pure computation at query time.

---

**Q13.** `@readonly` annotation on a service entity means:
- a) The entity cannot be read
- b) The entity supports GET (read) but NOT POST/PUT/DELETE (write)
- c) The entity is hidden from the API
- d) Only admins can access it

**Answer: b)** — `@readonly` allows reading data but blocks any creation, update, or deletion operations through that service endpoint.

---

**Q14.** In the CDS build process, what is CSN?
- a) A CSS framework
- b) CDS Schema Notation — the JSON representation of compiled CDS models
- c) A type of database
- d) A deployment tool

**Answer: b)** — CSN (CDS Schema Notation) is the internal JSON format that CAP uses to represent your compiled CDS model. It's the intermediate step between human-readable CDS and database-specific SQL.

---

**Q15.** When a CSV value needs to contain a semicolon, you should:
- a) Use a different separator for the whole file
- b) Wrap the value in double quotes: `"Hello; World"`
- c) Escape it with a backslash: `Hello\; World`
- d) It's not possible

**Answer: b)** — Wrap the value in double quotes. This is the standard CSV escaping mechanism. Example: `ID;"Text with; semicolon";price`

---

## Assignment: Complete EPM Data Model

### Due: Start of Day 15

Build the complete Enterprise Procurement Model (EPM) from scratch, demonstrating ALL concepts from Days 12-14.

### Requirements

Create a new CAP project `epm-complete` with namespace `com.epm` containing:

### Part A: Entities (4 marks)

Create these entities with proper types, aspects, and associations:

| Entity | Key Fields | Must Include |
|--------|-----------|-------------|
| **Suppliers** | cuid | name, contact, email, phone, city, country, isActive |
| **Categories** | cuid | name, description, parentCategory (self-reference) |
| **Products** | cuid, managed | name, description, price, currency, stock, minStock, rating, supplier (assoc), category (assoc) |
| **Customers** | cuid, managed | name, email, phone, city, country, creditLimit |
| **SalesOrders** | cuid, managed | orderNumber, customer (assoc), orderDate, amounts, currency, status |
| **SalesOrderItems** | cuid | order (assoc to parent), product (assoc), quantity, unitPrice, netAmount |
| **PurchaseOrders** | cuid, managed | poNumber, supplier (assoc), orderDate, amounts, currency, status |
| **PurchaseOrderItems** | cuid | order (assoc to parent), product (assoc), quantity, unitPrice |

**Relationship rules:**
- SalesOrders → SalesOrderItems = **Composition**
- PurchaseOrders → PurchaseOrderItems = **Composition**
- All other entity connections = **Association**
- Use `Currency` and `Country` from `@sap/cds/common`

---

### Part B: Views & Projections (2 marks)

Create at least 3 views:

1. **ProductCatalog** — Flat view with product name, price, supplier name, category name, stock status (calculated)
2. **OrderReport** — Flat view with order number, customer name, total amount, order date, status
3. **LowStockAlert** — Products where stock ≤ minStock with supplier contact details

Create at least 2 different services:
1. **SalesService** — Customer-facing (maybe hide internal pricing)
2. **AdminService** — Full access to everything
3. **ReportService** — Read-only views

---

### Part C: CSV Data (2 marks)

Provide complete CSV data:
- At least 4 Suppliers
- At least 5 Categories
- At least 8 Products (each with supplier and category)
- At least 4 Customers
- At least 4 Sales Orders with items (test deep insert for more)
- Currency and Country data
- All FK references must be valid!

---

### Part D: Deploy & Verify (2 marks)

1. `cds deploy --to sqlite:db/epm.db` runs without errors
2. `sqlite3` commands show all tables with correct schema
3. Data counts match CSV rows
4. `cds watch` starts without errors
5. Provide a `test.http` file with at least 10 test queries including:
   - `$expand` queries
   - `$filter` and `$orderby` queries
   - Reports with calculated fields
   - One deep insert POST request

---

### Grading

| Criteria | Marks |
|----------|-------|
| All entities with correct types, aspects, associations, compositions | 4 |
| Views with calculations + service projections | 2 |
| Complete CSV data with valid FK references | 2 |
| Deploy + verify + cds watch works + test queries | 2 |
| **Total** | **10** |

---

### Submission Checklist

- [ ] Project runs with `cds watch` (no errors)
- [ ] All 8+ entities defined in `db/schema.cds`
- [ ] Uses `cuid`, `managed`, `Currency`, `Country`
- [ ] At least 2 Compositions (Order→Items)
- [ ] At least 3 views with calculated fields
- [ ] All CSV files in `db/data/` with correct naming
- [ ] `cds deploy --to sqlite:db/epm.db` succeeds
- [ ] `test.http` with 10+ working queries
- [ ] Screenshot of `sqlite3 db/epm.db ".tables"` output

---

## Key Takeaways

| # | Topic | One-Line Summary |
|---|---|---|
| 1 | CDS View | `entity X as select from Y { fields }` — read-only window into data |
| 2 | Projection | `entity X as projection on Y` — full CRUD, can exclude fields |
| 3 | `excluding` | Remove specific fields: `projection on X excluding { secret }` |
| 4 | Renaming | `fieldName as newName` — alias fields in views |
| 5 | Calculated fields | `(price * 1.18) as total : Decimal` — computed on read, not stored |
| 6 | CASE expressions | `case when ... then ... else ... end as status` — conditional values |
| 7 | `@readonly` | Makes a service entity read-only (GET only, no POST/PUT/DELETE) |
| 8 | CSV location | Place files in `db/data/` folder |
| 9 | CSV naming | `<namespace>-<EntityName>.csv` — must match exactly! |
| 10 | CSV separator | Semicolons (`;`) — NOT commas |
| 11 | FK in CSV | Use `<field>_<key>` pattern (e.g., `author_ID`, `currency_code`) |
| 12 | `cds deploy` | Compiles CDS → SQL → creates DB → loads CSV data |
| 13 | `cds watch` | All-in-one: deploy + serve + hot reload (dev mode) |
| 14 | `cds compile` | Shows intermediate output: `--to sql`, `--to json`, `--to edmx` |
| 15 | SQLite inspect | `sqlite3 db/file.db ".tables"` / `.schema` / `SELECT` queries |

---

## Preparation for Day 15

Tomorrow: **CDS Services — Defining APIs, Actions & Functions**

You'll learn:
- How services transform entities into API endpoints
- Custom actions (`POST /Orders/approve`)
- Custom functions (`GET /Products/topSellers()`)
- Input validation annotations
- Service-level events and handlers
- Multiple services from one data model

**To prepare:**
- Ensure your EPM project runs perfectly with `cds watch`
- Test all your views and verify calculated fields work
- Think about: What operations can't be done with simple CRUD? (e.g., "approve an order", "restock a product") — these will become Actions tomorrow!
- Think about: How would you validate that `stock` can never go below 0?

---

*End of Day 14*
