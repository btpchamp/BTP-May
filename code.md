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
