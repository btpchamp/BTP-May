# Day 13: CDS Data Modeling — Part 2 (Associations & Compositions)

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 12 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Associations — Connecting Entities | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Compositions — Parent-Child Ownership | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Add Relationships to Library Model | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Aspects & @sap/cds/common | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Build EPM Data Model | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Complete EPM & Practice | 45 min |
| 16:45 - 17:00 | Assessment & Wrap-up | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Connect entities using managed and unmanaged associations
- Model one-to-one, one-to-many, and many-to-many relationships
- Use compositions for parent-child ownership (deep insert/delete)
- Clearly explain when to use Association vs. Composition
- Apply built-in aspects (`cuid`, `managed`, `temporal`) to avoid repetitive code
- Use `@sap/cds/common` for standard Currencies, Countries, and Languages
- Build a complete enterprise data model with relationships

---

## Day 12 Recap — Quick Fire (09:00 - 09:15)

1. CDS entity corresponds to a _____ in the database?
2. Which type should you use for money? → _____
3. `key ID : UUID` gives you what benefit? → _____
4. `namespace my.project;` does what? → _____
5. `type Amount : Decimal(12,2);` is called a _____ type?
6. What does an enum restrict? → _____
7. CSV files in CAP use which separator? → _____
8. `cds deploy --to sqlite:db/data.db` does what? → _____

<details>
<summary>Answers</summary>

1. A database table
2. `Decimal(p,s)` — never Double!
3. Auto-generated globally unique identifier
4. Prefixes all entities with `my.project.` to avoid naming conflicts
5. Reusable type
6. Restricts a field to only predefined values
7. Semicolons (`;`)
8. Compiles CDS to SQL, creates SQLite file, creates tables, loads CSV data

</details>

---

## Session 1: Associations — Connecting Entities (09:15 - 10:30)

### The Problem: Disconnected Entities

Yesterday we built a Library model with these entities:

```
Books    Authors    Members    Borrowings    Genres
```

But they had NO connection between them! The `Borrowings` entity stored `memberId` and `bookId` as plain UUIDs — just text fields with no relationship enforced.

```cds
// Yesterday's approach (Day 12) — NO relationship!
entity Borrowings {
  key ID     : UUID;
  memberId   : UUID;   // Just a text field — could store ANYTHING!
  bookId     : UUID;   // No link to Books entity
}
```

**Problems with this approach:**
- No navigation: Can't go from a Borrowing directly to the Book details
- No validation: Could store a fake UUID that doesn't match any book
- No $expand: Can't ask "give me borrowings WITH their book details"
- No cascading: Deleting a book doesn't affect borrowings

**Solution: Associations!**

---

### What is an Association?

An **Association** is a relationship between two entities. Think of it like drawing an arrow from one entity to another.

```
┌──────────┐         ┌──────────┐
│  Books   │◄────────│Borrowings│
└──────────┘  "book" └──────────┘
                         │
                         │ "member"
                         ▼
                    ┌──────────┐
                    │ Members  │
                    └──────────┘
```

In CDS:

```cds
entity Borrowings {
  key ID   : UUID;
  book     : Association to Books;      // Points to ONE book
  member   : Association to Members;    // Points to ONE member
}
```

**That's it!** Now `Borrowings` is connected to `Books` and `Members`.

---

### Managed Associations (The Easy Way)

**Managed associations** are the most common type. CAP handles everything for you — foreign keys, joins, navigation.

```cds
entity Books {
  key ID    : UUID;
  title     : String(200);
  author    : Association to Authors;   // This book has ONE author
}

entity Authors {
  key ID    : UUID;
  name      : String(100);
}
```

**What CAP does behind the scenes:**

```sql
-- CAP automatically creates a foreign key column:
CREATE TABLE Books (
  ID NVARCHAR(36) PRIMARY KEY,
  title NVARCHAR(200),
  author_ID NVARCHAR(36)    -- ← CAP added this automatically!
);
```

You write `author : Association to Authors;` and CAP:
1. Creates a column `author_ID` in the Books table
2. Links it to the `Authors` table's primary key
3. Enables `$expand` in OData (get author details with the book)
4. Enables navigation in URLs (`/Books(id)/author`)

---

### How to READ Associated Data

Once you have associations, OData `$expand` becomes available:

```http
// Get all books WITH their author details:
GET /library/Books?$expand=author

// Response:
{
  "value": [
    {
      "ID": "...",
      "title": "Clean Code",
      "author": {                    // ← Expanded! Full author details
        "ID": "...",
        "name": "Robert C. Martin"
      }
    }
  ]
}
```

Without `$expand`, you only get the foreign key:

```http
GET /library/Books

// Response:
{
  "value": [
    {
      "ID": "...",
      "title": "Clean Code",
      "author_ID": "a1a1a1a1-..."   // ← Just the ID, no details
    }
  ]
}
```

---

### One-to-Many: "A book has MANY reviews"

When one entity has multiple related records, use `Association to many`:

```cds
entity Books {
  key ID    : UUID;
  title     : String(200);
  reviews   : Association to many Reviews on reviews.book = $self;
}

entity Reviews {
  key ID    : UUID;
  book      : Association to Books;    // Each review belongs to ONE book
  rating    : Integer;
  comment   : String(500);
}
```

**Think about it:**
- One Book → has many Reviews (one-to-many)
- One Review → belongs to one Book (many-to-one)

The `on reviews.book = $self` part means: "find all Reviews where the review's `book` field points back to THIS book."

---

### Relationship Types — Visual Guide

```
┌─────────────────────────────────────────────────────────┐
│          ONE-TO-ONE                                      │
│                                                         │
│  Employee ──────── ParkingSpot                          │
│  "one employee has exactly one parking spot"            │
│                                                         │
│  entity Employee {                                      │
│    parking : Association to ParkingSpot;                │
│  }                                                      │
├─────────────────────────────────────────────────────────┤
│          ONE-TO-MANY                                    │
│                                                         │
│  Author ──┬────── Book1                                 │
│           ├────── Book2                                 │
│           └────── Book3                                 │
│  "one author can write many books"                      │
│                                                         │
│  entity Authors {                                       │
│    books : Association to many Books on books.author = $self; │
│  }                                                      │
│  entity Books {                                         │
│    author : Association to Authors;                     │
│  }                                                      │
├─────────────────────────────────────────────────────────┤
│          MANY-TO-MANY                                   │
│                                                         │
│  Book1 ──┬── Genre1                                     │
│          └── Genre2                                     │
│  Book2 ──┬── Genre1                                     │
│          └── Genre3                                     │
│  "a book can have many genres, a genre has many books"  │
│                                                         │
│  → Needs a LINK TABLE (junction entity)                 │
├─────────────────────────────────────────────────────────┘
```

---

### Many-to-Many Relationships

A Book can belong to MULTIPLE Genres, and a Genre can have MULTIPLE Books. This requires a **link table** (also called junction table):

```cds
entity Books {
  key ID    : UUID;
  title     : String(200);
  genres    : Association to many BookGenres on genres.book = $self;
}

entity Genres {
  key code  : String(20);
  name      : String(50);
  books     : Association to many BookGenres on books.genre = $self;
}

// Link table — connects Books and Genres
entity BookGenres {
  key book  : Association to Books;
  key genre : Association to Genres;
}
```

**Database result:**

```
Books table:         BookGenres table:       Genres table:
─────────────        ─────────────────       ────────────
ID | title           book_ID | genre_code    code | name
1  | Clean Code      1       | PROG          PROG | Programming
2  | Sapiens         1       | SELF          SELF | Self-Help
                     2       | HIST          HIST | History
                     2       | SCIF          SCIF | Science Fiction
```

---

### Unmanaged Associations (The Manual Way)

Sometimes you need full control over the foreign key or the join condition. Use **unmanaged associations** for this:

```cds
entity Orders {
  key ID           : UUID;
  customerCode     : String(10);    // FK field you define yourself
  customer         : Association to Customers 
                     on customer.code = customerCode;  // YOU specify the join
}

entity Customers {
  key ID   : UUID;
  code     : String(10);   // Not the primary key!
  name     : String(100);
}
```

**When to use unmanaged:**
- Joining on a non-primary-key field
- Complex join conditions (multiple fields)
- Legacy database integration

**Rule of thumb:** Use **managed** associations 95% of the time. Only use unmanaged when you have a specific reason.

---

### Managed vs. Unmanaged — Side by Side

| Feature | Managed | Unmanaged |
|---------|---------|-----------|
| **Syntax** | `field : Association to Entity;` | `field : Association to Entity on ...;` |
| **Foreign key** | Auto-created (`field_ID`) | You define it yourself |
| **Join condition** | Auto-generated | You write it explicitly |
| **Ease of use** | Simple (recommended) | More code needed |
| **Use case** | 95% of cases | Non-PK joins, complex conditions |
| **Example** | `author : Association to Authors;` | `author : Association to Authors on author.code = authorCode;` |

---

### Practice: Mental Model Check

Look at these real-world relationships. Which type is each?

| Relationship | Type | Why |
|-------------|------|-----|
| Student → College | Many-to-One | Many students attend one college |
| Order → OrderItems | One-to-Many | One order has many items |
| Employee → PAN Card | One-to-One | Each employee has exactly one PAN |
| Student → Courses | Many-to-Many | Students take many courses, courses have many students |
| Book → Author | Many-to-One | Many books by one author |
| Author → Books | One-to-Many | One author writes many books |

---

## Session 2: Compositions — Parent-Child Ownership (10:45 - 12:00)

### The Problem Associations Don't Solve

Consider an **Order** and its **OrderItems**:

```
Order #1001
├── Item 1: Laptop x 1    ← These items BELONG to Order #1001
├── Item 2: Mouse x 2     ← They have NO meaning without the order
└── Item 3: Keyboard x 1  ← Delete the order → items must go too!
```

If we use Association:

```cds
// With Association — items are INDEPENDENT:
entity Orders {
  key ID   : UUID;
  items    : Association to many OrderItems on items.order = $self;
}

entity OrderItems {
  key ID   : UUID;
  order    : Association to Orders;
  product  : String(100);
  quantity : Integer;
}
```

**Problem:** Items exist independently! You can:
- Delete an Order... but its items remain as orphans
- Create an item without an order (makes no sense!)
- Items have their own API endpoint (don't want that!)

---

### Composition = Ownership

**Composition** means "this entity OWNS those child entities." The children cannot exist without the parent.

```cds
// With Composition — items are OWNED by the order:
entity Orders {
  key ID     : UUID;
  orderNo    : String(20);
  total      : Decimal(12,2);
  items      : Composition of many OrderItems on items.order = $self;
}

entity OrderItems {
  key ID       : UUID;
  order        : Association to Orders;
  product      : String(100);
  quantity     : Integer;
  unitPrice    : Decimal(10,2);
}
```

**What Composition gives you:**

| Feature | With Association | With Composition |
|---------|-----------------|------------------|
| Deep Insert (create parent + children in one request) | No | Yes |
| Deep Delete (delete parent → children auto-deleted) | No | Yes |
| Children accessible independently? | Yes (own endpoint) | No (only through parent) |
| Children meaningful without parent? | Yes | No |
| ETags/drafts propagate to children? | No | Yes |

---

### Deep Insert Example

With Composition, you can create an Order with all its items in **one single POST request**:

```http
POST /orders/Orders
Content-Type: application/json

{
  "orderNo": "ORD-1001",
  "total": 75000,
  "items": [
    { "product": "Laptop", "quantity": 1, "unitPrice": 65000 },
    { "product": "Mouse", "quantity": 2, "unitPrice": 500 },
    { "product": "Keyboard", "quantity": 1, "unitPrice": 2000 }
  ]
}
```

**CAP creates the Order AND all 3 items in one transaction!** If any item fails, everything rolls back.

---

### Deep Delete Example

```http
DELETE /orders/Orders('order-uuid-here')
```

**CAP automatically deletes:**
- The Order
- ALL its OrderItems

No orphan records! The data stays consistent.

---

### Association vs. Composition — The Decision Guide

Ask yourself: **"Can the child exist without the parent?"**

| Question | If YES → Association | If NO → Composition |
|----------|---------------------|---------------------|
| Can child exist alone? | Book exists without a shelf | OrderItem can't exist without an Order |
| Delete parent → child? | Don't delete the book if shelf removed | Delete items when order is deleted |
| Own API endpoint? | Yes, access books directly | No, items only via orders |
| Example | Book → Author | Order → OrderItems |

---

### Real-World Examples

| Relationship | Type | Reasoning |
|-------------|------|-----------|
| Order → OrderItems | **Composition** | Items belong to the order, meaningless alone |
| Invoice → LineItems | **Composition** | Line items are part of the invoice |
| Blog → Comments | **Composition** | Comments belong to the blog post |
| Travel Request → Expenses | **Composition** | Expenses are part of the request |
| Book → Author | **Association** | Author exists independently |
| Employee → Department | **Association** | Department exists without any specific employee |
| Student → University | **Association** | University doesn't cease to exist if student leaves |
| Product → Category | **Association** | Category exists independently of any product |

---

### Composition Syntax Variations

```cds
// 1. Composition of MANY (most common):
entity Orders {
  key ID   : UUID;
  items    : Composition of many OrderItems on items.order = $self;
}

// 2. Composition of ONE (rare but valid):
entity Employees {
  key ID      : UUID;
  name        : String(100);
  address     : Composition of one Address;  // Exactly one address, owned by employee
}

entity Address {
  key ID      : UUID;
  street      : String(200);
  city        : String(100);
  pincode     : String(10);
}
```

---

### Nested Compositions (Multi-Level)

Compositions can be nested! Order → Items → SubItems:

```cds
entity Orders {
  key ID   : UUID;
  items    : Composition of many OrderItems on items.order = $self;
}

entity OrderItems {
  key ID       : UUID;
  order        : Association to Orders;
  product      : String(100);
  quantity     : Integer;
  schedules    : Composition of many DeliverySchedules on schedules.item = $self;
}

entity DeliverySchedules {
  key ID        : UUID;
  item          : Association to OrderItems;
  deliveryDate  : Date;
  quantity      : Integer;
}
```

Deleting an Order → deletes its Items → deletes their DeliverySchedules. All in one transaction!

---

### Key Differences Summary

```
┌─────────────────────────────────────────────────────────────┐
│                    ASSOCIATION                                │
│                                                             │
│  ┌──────┐         ┌──────┐                                  │
│  │ Book │─ ─ ─ ─ ─│Author│   "Book REFERS to Author"       │
│  └──────┘         └──────┘                                  │
│                                                             │
│  • Author exists on its own                                 │
│  • Delete Book → Author stays                               │
│  • Both have their own API endpoints                        │
│  • Think: "knows about" / "points to"                       │
├─────────────────────────────────────────────────────────────┤
│                    COMPOSITION                               │
│                                                             │
│  ┌──────┐         ┌──────────┐                              │
│  │Order │●━━━━━━━━│OrderItems│  "Order OWNS OrderItems"     │
│  └──────┘         └──────────┘                              │
│                                                             │
│  • Items can't exist without the Order                      │
│  • Delete Order → Items are deleted too                     │
│  • Items accessed ONLY through the Order                    │
│  • Think: "contains" / "is made of"                         │
└─────────────────────────────────────────────────────────────┘
```

---

## Session 3: Hands-on — Add Relationships to Library Model (12:00 - 13:00)

### Let's Upgrade Our Library Model!

We'll take yesterday's Library Management entities and add proper relationships.

#### Step 1: Open Your Library Project

```bash
cd ~/cap-training/library-management
code .
```

#### Step 2: Update `db/schema.cds` with Associations

Replace the entire content with:

```cds
namespace lib.management;

// ─── REUSABLE TYPES ─────────────────────────────
type Name    : String(100);
type Email   : String(255);
type Phone   : String(20);
type Amount  : Decimal(8,2);

type BorrowStatus : String(20) enum {
  Borrowed;
  Returned;
  Overdue;
}

// ─── AUTHORS ─────────────────────────────────────
entity Authors {
  key ID        : UUID;
  firstName     : String(50);
  lastName      : String(50);
  nationality   : String(50);
  birthDate     : Date;
  biography     : String(1000);
  email         : Email;
  isActive      : Boolean;
  // One author can write MANY books:
  books         : Association to many Books on books.author = $self;
}

// ─── GENRES ──────────────────────────────────────
entity Genres {
  key code      : String(20);
  name          : String(50);
  description   : String(200);
  isActive      : Boolean;
}

// ─── BOOKS ───────────────────────────────────────
entity Books {
  key ID          : UUID;
  title           : String(200);
  isbn            : String(13);
  pages           : Integer;
  price           : Amount;
  publishedDate   : Date;
  language        : String(30);
  edition         : Integer;
  totalCopies     : Integer;
  availableCopies : Integer;
  summary         : String(2000);
  // Managed associations:
  author          : Association to Authors;       // Many books → one author
  genre           : Association to Genres;        // Many books → one genre
  // One book can have MANY reviews:
  reviews         : Association to many Reviews on reviews.book = $self;
}

// ─── MEMBERS ─────────────────────────────────────
entity Members {
  key ID          : UUID;
  memberNumber    : String(10);
  firstName       : String(50);
  lastName        : String(50);
  email           : Email;
  phone           : Phone;
  address         : String(200);
  joinDate        : Date;
  memberType      : String(20);
  maxBooks        : Integer;
  isActive        : Boolean;
  // One member can have MANY borrowings:
  borrowings      : Association to many Borrowings on borrowings.member = $self;
}

// ─── REVIEWS ─────────────────────────────────────
entity Reviews {
  key ID        : UUID;
  book          : Association to Books;       // This review is for WHICH book
  member        : Association to Members;     // WHO wrote this review
  rating        : Integer;
  comment       : String(500);
  reviewDate    : Date;
}

// ─── BORROWINGS ──────────────────────────────────
entity Borrowings {
  key ID          : UUID;
  member          : Association to Members;   // WHO borrowed
  book            : Association to Books;     // WHAT was borrowed
  borrowDate      : Date;
  dueDate         : Date;
  returnDate      : Date;
  status          : BorrowStatus;
  fineAmount      : Amount;
}
```

---

#### Step 3: Update the Service

Update `srv/library-service.cds`:

```cds
using lib.management from '../db/schema';

service CatalogService {
  entity Books      as projection on management.Books;
  entity Authors    as projection on management.Authors;
  entity Genres     as projection on management.Genres;
  entity Members    as projection on management.Members;
  entity Borrowings as projection on management.Borrowings;
  entity Reviews    as projection on management.Reviews;
}
```

#### Step 4: Update CSV Data

Update `db/data/lib.management-Books.csv` to include author and genre references:

```csv
ID,title,isbn,pages,price,publishedDate,language,edition,totalCopies,availableCopies,summary,author_ID,genre_code
1a2b3c4d-1111-1111-1111-111111111111,Clean Code,9780132350884,464,450.00,2008-08-01,English,1,5,3,A handbook of agile software craftsmanship,a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa,PROG
5ed1c590-e7bc-4dcb-a1f3-220892265591,CAP BTP Bookk,9780132350883,500,700,2008-08-05,English,1,5,3,CAP BTP Programming Knowldge,a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa,SCI
2a2b3c4d-2222-2222-2222-222222222222,The Pragmatic Programmer,9780135957059,352,550.00,2019-09-13,English,2,3,2,Journey to mastery,b2b2b2b2-bbbb-bbbb-bbbb-bbbbbbbbbbbb,PROG
3a2b3c4d-3333-3333-3333-333333333333,Sapiens,9780062316097,443,400.00,2015-02-10,English,1,4,4,A brief history of humankind,c3c3c3c3-cccc-cccc-cccc-cccccccccccc,HIST
4a2b3c4d-4444-4444-4444-444444444444,Atomic Habits,9780735211292,320,350.00,2018-10-16,English,1,6,5,Tiny changes remarkable results,d4d4d4d4-dddd-dddd-dddd-dddddddddddd,SELF
5a2b3c4d-5555-5555-5555-555555555555,Foundation,9780553293357,255,300.00,1951-06-01,English,1,5,4,Classic science fiction novel,e5e5e5e5-eeee-eeee-eeee-eeeeeeeeeeee,SCI
```

Update `db/data/lib.management-Authors.csv`:

```csv
ID,firstName,lastName,nationality,birthDate,biography,email,isActive
a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa,Robert,Martin,USA,1952-12-05,Clean Code author and software consultant,unclebob@example.com,true
b2b2b2b2-bbbb-bbbb-bbbb-bbbbbbbbbbbb,David,Thomas,USA,1956-01-01,Co-author of The Pragmatic Programmer,david.thomas@example.com,true
c3c3c3c3-cccc-cccc-cccc-cccccccccccc,Yuval,Harari,Israel,1976-02-24,Historian and philosopher,yuval@example.com,true
d4d4d4d4-dddd-dddd-dddd-dddddddddddd,James,Clear,USA,1986-01-22,Author of Atomic Habits,james.clear@example.com,true
e5e5e5e5-eeee-eeee-eeee-eeeeeeeeeeee,Isaac,Asimov,Russia,1920-01-02,Famous science fiction author,asimov@example.com,true
```

Update `db/data/`:

```csv
ID,firstName,lastName,nationality,birthDate,biography,email,isActive
a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa,Robert,Martin,USA,1952-12-05,Clean Code author and software consultant,unclebob@example.com,true
b2b2b2b2-bbbb-bbbb-bbbb-bbbbbbbbbbbb,David,Thomas,USA,1956-01-01,Co-author of The Pragmatic Programmer,david.thomas@example.com,true
c3c3c3c3-cccc-cccc-cccc-cccccccccccc,Yuval,Harari,Israel,1976-02-24,Historian and philosopher,yuval@example.com,true
d4d4d4d4-dddd-dddd-dddd-dddddddddddd,James,Clear,USA,1986-01-22,Author of Atomic Habits,james.clear@example.com,true
e5e5e5e5-eeee-eeee-eeee-eeeeeeeeeeee,Isaac,Asimov,Russia,1920-01-02,Famous science fiction author,asimov@example.com,true
```

#### Step 5: Run and Test with $expand

```bash
cds watch
```

Now try these powerful queries:

```http
// Get books WITH author details:
GET http://localhost:4004/Catalog/Books?$expand=author

// Get books WITH genre details:
GET http://localhost:4004/Catalog/Books?$expand=genre

// Get books with BOTH author and genre:
GET http://localhost:4004/Catalog/Books?$expand=author,genre

// Get an author WITH all their books:
GET http://localhost:4004/Catalog/Authors?$expand=books

// Get borrowings with book and member details:
GET http://localhost:4004/Catalog/Borrowings?$expand=book,member

// Get members with their borrowings AND the borrowed book:
GET http://localhost:4004/Catalog/Members?$expand=borrowings($expand=book)

// Filter books by a specific author:
GET http://localhost:4004/Catalog/Books?$filter=author_ID eq a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa
```

---

### Adding a Composition Example: Orders

Let's add an Orders feature to our library (for book purchases):

Add to `db/schema.cds`:

```cds
// ─── PURCHASE ORDERS (Composition example) ───────
entity PurchaseOrders {
  key ID        : UUID;
  orderNumber   : String(20);
  orderDate     : Date;
  totalAmount   : Amount;
  status        : String(20) default 'Draft';
  // Composition — items BELONG to this order:
  items         : Composition of many PurchaseOrderItems on items.order = $self;
}

entity PurchaseOrderItems {
  key ID        : UUID;
  order         : Association to PurchaseOrders;  // Parent reference
  book          : Association to Books;           // Which book to purchase
  quantity      : Integer;
  unitPrice     : Amount;
  totalPrice    : Amount;
}
```

Add to service:

```cds
entity PurchaseOrders as projection on management.PurchaseOrders;
```

Now test deep insert:

```http
POST http://localhost:4004/library/PurchaseOrders
Content-Type: application/json

{
  "orderNumber": "PO-001",
  "orderDate": "2026-06-01",
  "totalAmount": 1400,
  "status": "Draft",
  "items": [
    {
      "book_ID": "1a2b3c4d-1111-1111-1111-111111111111",
      "quantity": 2,
      "unitPrice": 450,
      "totalPrice": 900
    },
    {
      "book_ID": "4a2b3c4d-4444-4444-4444-444444444444",
      "quantity": 1,
      "unitPrice": 350,
      "totalPrice": 350
    }
  ]
}
```

One request creates the order AND both items!

---

## Session 4: Aspects — Reusable Model Fragments (13:45 - 14:45)

### The Problem: Repetitive Fields

Look at almost ANY business entity — they all have these same fields:

```cds
entity Books {
  key ID    : UUID;           // ← Every entity has this!
  ...
  createdAt : DateTime;       // ← Every entity has this!
  createdBy : String(255);    // ← Every entity has this!
  modifiedAt: DateTime;       // ← Every entity has this!
  modifiedBy: String(255);    // ← Every entity has this!
}

entity Authors {
  key ID    : UUID;           // ← Same thing again!
  ...
  createdAt : DateTime;       // ← Same thing again!
  createdBy : String(255);    // ← Same thing again!
  modifiedAt: DateTime;       // ← Same thing again!
  modifiedBy: String(255);    // ← Same thing again!
}
```

That's **5 repeated fields** in EVERY entity! Imagine 20 entities — that's 100 lines of repeated code.

---

### What is an Aspect?

An **Aspect** is a reusable fragment of model definition that you can "mix into" any entity. Think of it like a "plugin" for your entity.

```cds
// Define an aspect:
aspect Trackable {
  createdAt  : DateTime;
  createdBy  : String(255);
  modifiedAt : DateTime;
  modifiedBy : String(255);
}

// Use it with ":" (includes/extends):
entity Books : Trackable {
  key ID   : UUID;
  title    : String(200);
  price    : Decimal(8,2);
}
```

The `:` means "include all fields from Trackable into Books."

**Result:** Books now has: ID, title, price, createdAt, createdBy, modifiedAt, modifiedBy — 7 fields total!

---

### Built-in Aspects from CAP

CAP provides three powerful built-in aspects. You don't need to define them — just import and use!

```cds
using { cuid, managed, temporal } from '@sap/cds/common';
```

---

#### 1. `cuid` — Auto-Generated UUID Key

```cds
// What cuid gives you:
aspect cuid {
  key ID : UUID;
}

// Usage:
entity Books : cuid {
  title : String(200);    // ID field is added automatically!
  price : Decimal(8,2);
}
```

**Before and After:**

```cds
// ❌ Without cuid — repetitive:
entity Books {
  key ID : UUID;
  title  : String(200);
}
entity Authors {
  key ID : UUID;
  name   : String(100);
}

// ✅ With cuid — cleaner:
using { cuid } from '@sap/cds/common';

entity Books : cuid {
  title : String(200);
}
entity Authors : cuid {
  name : String(100);
}
```

---

#### 2. `managed` — Auto-Tracked Creation & Modification

```cds
// What managed gives you:
aspect managed {
  createdAt  : Timestamp  @cds.on.insert: $now;
  createdBy  : String(255) @cds.on.insert: $user;
  modifiedAt : Timestamp  @cds.on.insert: $now  @cds.on.update: $now;
  modifiedBy : String(255) @cds.on.insert: $user @cds.on.update: $user;
}
```

**The magic:** These fields are **auto-filled by CAP!**
- `createdAt` → automatically set to current time when record is created
- `createdBy` → automatically set to the logged-in user
- `modifiedAt` → automatically updated whenever the record changes
- `modifiedBy` → automatically set to whoever made the change

```cds
// Usage:
using { cuid, managed } from '@sap/cds/common';

entity Books : cuid, managed {
  title : String(200);
  price : Decimal(8,2);
}
```

**Result:** Books now has ID, title, price, createdAt, createdBy, modifiedAt, modifiedBy — and the tracking fields are automatic!

---

#### 3. `temporal` — Valid-From/Valid-To Time Slices

```cds
// What temporal gives you:
aspect temporal {
  validFrom : Timestamp  @cds.valid.from;
  validTo   : Timestamp  @cds.valid.to;
}
```

Used for data that changes over time (e.g., price history, exchange rates):

```cds
using { temporal } from '@sap/cds/common';

entity PriceHistory : temporal {
  key product : Association to Products;
  price       : Decimal(10,2);
}
```

This means: "This price was valid FROM this date TO that date."

---

#### Combining Multiple Aspects

You can use multiple aspects together — separate with commas:

```cds
using { cuid, managed } from '@sap/cds/common';

// Combines BOTH cuid and managed:
entity Books : cuid, managed {
  title    : String(200);
  price    : Decimal(8,2);
}
```

**Books now has ALL these fields automatically:**
- `key ID : UUID` (from cuid)
- `title` (your field)
- `price` (your field)
- `createdAt` (from managed — auto-filled)
- `createdBy` (from managed — auto-filled)
- `modifiedAt` (from managed — auto-filled)
- `modifiedBy` (from managed — auto-filled)

**7 functional fields, but you only wrote 2 lines!**

---

### Creating Your Own Aspects

Define aspects for patterns you repeat across entities:

```cds
// Aspect for entities that can be activated/deactivated:
aspect Activatable {
  isActive    : Boolean default true;
  deactivatedAt : DateTime;
  deactivatedBy : String(255);
}

// Aspect for entities with descriptions:
aspect Described {
  name        : String(100);
  description : String(1000);
}

// Aspect for entities with addresses:
aspect Addressed {
  street   : String(200);
  city     : String(100);
  state    : String(50);
  pincode  : String(10);
  country  : String(50);
}

// Use them:
entity Stores : cuid, managed, Addressed, Activatable {
  storeName : String(100);
  phone     : String(20);
}

entity Warehouses : cuid, managed, Addressed {
  capacity : Integer;
}
```

---

### The `@sap/cds/common` Library

This is SAP's built-in library of reusable types and entities. It provides standardized definitions for common business concepts.

```cds
using { 
  cuid, 
  managed, 
  temporal,
  Currency,
  Country,
  Language,
  sap.common.CodeList 
} from '@sap/cds/common';
```

---

### Code Lists — Standardized Master Data

**Code Lists** are pre-built entity patterns for things like Countries, Currencies, and Languages:

```cds
using { Currency, Country, Language } from '@sap/cds/common';

entity Products : cuid, managed {
  name     : String(100);
  price    : Decimal(10,2);
  currency : Currency;    // Auto-linked to sap.common.Currencies!
  origin   : Country;     // Auto-linked to sap.common.Countries!
}
```

**What `Currency` type gives you:**

```cds
// Behind the scenes, Currency is defined as:
type Currency : Association to sap.common.Currencies;

// And sap.common.Currencies is:
entity Currencies : CodeList {
  key code : String(3);   // ISO code: INR, USD, EUR
}

// CodeList provides:
aspect CodeList {
  name  : localized String(255);
  descr : localized String(1000);
}
```

This means when you use `currency : Currency;`:
- You get a `currency_code` column (String(3) — like "INR")
- You can `$expand=currency` to get the full name ("Indian Rupee")
- The currency names are **localized** (different languages!)

---

### Using Currencies in Practice

```cds
using { cuid, managed, Currency } from '@sap/cds/common';

entity Products : cuid, managed {
  name     : String(100);
  price    : Decimal(10,2);
  currency : Currency;        // Just this one line!
}
```

Add sample currency data in `db/data/sap.common-Currencies.csv`:

```csv
code;name;descr;symbol
INR;Indian Rupee;Currency of India;₹
USD;US Dollar;Currency of United States;$
EUR;Euro;Currency of European Union;€
GBP;British Pound;Currency of United Kingdom;£
```

Now your product data can reference currencies:

`db/data/my.project-Products.csv`:
```csv
ID;name;price;currency_code
...;Laptop;65000;INR
...;Mouse;25;USD
```

And query with expand:
```http
GET /catalog/Products?$expand=currency
// Returns: { "name": "Laptop", "price": 65000, "currency": { "code": "INR", "name": "Indian Rupee", "symbol": "₹" } }
```

---

### Countries and Languages — Same Pattern

```cds
using { Country, Language } from '@sap/cds/common';

entity Customers : cuid, managed {
  name     : String(100);
  country  : Country;     // Links to sap.common.Countries
  language : Language;    // Links to sap.common.Languages
}
```

Provide data in:
- `db/data/sap.common-Countries.csv`
- `db/data/sap.common-Languages.csv`

```csv
// sap.common-Countries.csv
code;name;descr
IN;India;Republic of India
US;United States;United States of America
DE;Germany;Federal Republic of Germany
GB;United Kingdom;United Kingdom of Great Britain
```

```csv
// sap.common-Languages.csv
code;name;descr
en;English;English Language
hi;Hindi;Hindi Language
de;German;German Language
```

---

### Building Your Own Code Lists

You can create your own code-list entities following the same pattern:

```cds
using { sap.common.CodeList } from '@sap/cds/common';

// Your own code list — follows SAP standard pattern:
entity BookCategories : CodeList {
  key code : String(10);    // "FICTION", "TECH", etc.
}

entity OrderStatuses : CodeList {
  key code : String(20);    // "DRAFT", "SUBMITTED", etc.
  criticality : Integer;    // For UI colors (1=red, 2=yellow, 3=green)
}

// Use them like Currency/Country:
entity Books : cuid, managed {
  title    : String(200);
  category : Association to BookCategories;
}
```

---

### Complete Example: Library with Aspects & Common

Here's our Library model rewritten with aspects and `@sap/cds/common`:

```cds
using { cuid, managed, Currency, Country } from '@sap/cds/common';

namespace lib.management;

// ─── AUTHORS ─────────────────────────────────────
entity Authors : cuid, managed {
  firstName     : String(50);
  lastName      : String(50);
  nationality   : Country;
  birthDate     : Date;
  biography     : String(1000);
  email         : String(255);
  isActive      : Boolean default true;
  books         : Association to many Books on books.author = $self;
}

// ─── BOOKS ───────────────────────────────────────
entity Books : cuid, managed {
  title           : String(200);
  isbn            : String(13);
  pages           : Integer;
  price           : Decimal(8,2);
  currency        : Currency;
  publishedDate   : Date;
  language        : String(30);
  edition         : Integer;
  totalCopies     : Integer;
  availableCopies : Integer;
  summary         : String(2000);
  author          : Association to Authors;
  genre           : Association to Genres;
  reviews         : Association to many Reviews on reviews.book = $self;
}

// ─── GENRES ──────────────────────────────────────
entity Genres : managed {
  key code      : String(20);
  name          : String(50);
  description   : String(200);
  isActive      : Boolean default true;
}

// ─── MEMBERS ─────────────────────────────────────
entity Members : cuid, managed {
  memberNumber    : String(10);
  firstName       : String(50);
  lastName        : String(50);
  email           : String(255);
  phone           : String(20);
  address         : String(200);
  country         : Country;
  joinDate        : Date;
  memberType      : String(20);
  maxBooks        : Integer default 3;
  isActive        : Boolean default true;
  borrowings      : Association to many Borrowings on borrowings.member = $self;
}

// ─── REVIEWS ─────────────────────────────────────
entity Reviews : cuid, managed {
  book          : Association to Books;
  member        : Association to Members;
  rating        : Integer;
  comment       : String(500);
  reviewDate    : Date;
}

// ─── BORROWINGS ──────────────────────────────────
entity Borrowings : cuid, managed {
  member        : Association to Members;
  book          : Association to Books;
  borrowDate    : Date;
  dueDate       : Date;
  returnDate    : Date;
  status        : String(20) default 'Borrowed';
  fineAmount    : Decimal(8,2) default 0;
}

// ─── PURCHASE ORDERS (with Composition) ──────────
entity PurchaseOrders : cuid, managed {
  orderNumber   : String(20);
  orderDate     : Date;
  supplier      : String(100);
  totalAmount   : Decimal(10,2);
  currency      : Currency;
  status        : String(20) default 'Draft';
  items         : Composition of many PurchaseOrderItems on items.order = $self;
}

entity PurchaseOrderItems : cuid, managed {
  order         : Association to PurchaseOrders;
  book          : Association to Books;
  quantity      : Integer;
  unitPrice     : Decimal(8,2);
  totalPrice    : Decimal(10,2);
}
```

**Notice how much cleaner this is!** No more repeating `key ID : UUID` and `createdAt/modifiedAt` everywhere.

---

## Session 5: Hands-on — Build EPM Data Model (15:00 - 16:00)

### Project: Enterprise Procurement Model (EPM)

Let's build a real-world procurement system from scratch using everything we learned today.

#### Step 1: Create a New CAP Project

```bash
cd ~/cap-training
cds init epm-model
cd epm-model
npm install
code .
```

#### Step 2: Create the Data Model

Create `db/schema.cds`:

```cds
using { cuid, managed, Currency, Country } from '@sap/cds/common';

namespace com.epm;

// ─── SUPPLIERS ───────────────────────────────────
entity Suppliers : cuid, managed {
  supplierName  : String(100);
  contactPerson : String(100);
  email         : String(255);
  phone         : String(20);
  street        : String(200);
  city          : String(100);
  state         : String(50);
  pincode       : String(10);
  country       : Country;
  isActive      : Boolean default true;
  products      : Association to many Products on products.supplier = $self;
}

// ─── PRODUCT CATEGORIES ──────────────────────────
entity Categories : cuid, managed {
  categoryName  : String(100);
  description   : String(500);
  parentCategory: Association to Categories;  // Self-reference for sub-categories!
  products      : Association to many Products on products.category = $self;
}

// ─── PRODUCTS ────────────────────────────────────
entity Products : cuid, managed {
  productName   : String(100);
  description   : String(500);
  price         : Decimal(10,2);
  currency      : Currency;
  weight        : Decimal(8,3);
  weightUnit    : String(5) default 'KG';
  stock         : Integer default 0;
  minStock      : Integer default 10;
  supplier      : Association to Suppliers;
  category      : Association to Categories;
  isAvailable   : Boolean default true;
}

// ─── CUSTOMERS ───────────────────────────────────
entity Customers : cuid, managed {
  customerName  : String(100);
  email         : String(255);
  phone         : String(20);
  street        : String(200);
  city          : String(100);
  state         : String(50);
  pincode       : String(10);
  country       : Country;
  creditLimit   : Decimal(12,2) default 100000;
  orders        : Association to many SalesOrders on orders.customer = $self;
}

// ─── SALES ORDERS ────────────────────────────────
entity SalesOrders : cuid, managed {
  orderNumber    : String(20);
  customer       : Association to Customers;
  orderDate      : Date;
  deliveryDate   : Date;
  grossAmount    : Decimal(12,2);
  netAmount      : Decimal(12,2);
  taxAmount      : Decimal(10,2);
  currency       : Currency;
  status         : String(20) default 'New';
  priority       : String(10) default 'Medium';
  // Composition — order items BELONG to this order:
  items          : Composition of many SalesOrderItems on items.order = $self;
}

// ─── SALES ORDER ITEMS ───────────────────────────
entity SalesOrderItems : cuid {
  order          : Association to SalesOrders;
  product        : Association to Products;
  quantity       : Integer;
  unitPrice      : Decimal(10,2);
  grossAmount    : Decimal(12,2);
  netAmount      : Decimal(12,2);
  taxAmount      : Decimal(10,2);
  currency       : Currency;
  deliveryDate   : Date;
}

// ─── PURCHASE ORDERS (from Suppliers) ────────────
entity PurchaseOrders : cuid, managed {
  poNumber       : String(20);
  supplier       : Association to Suppliers;
  orderDate      : Date;
  grossAmount    : Decimal(12,2);
  netAmount      : Decimal(12,2);
  currency       : Currency;
  status         : String(20) default 'Draft';
  items          : Composition of many PurchaseOrderItems on items.order = $self;
}

// ─── PURCHASE ORDER ITEMS ────────────────────────
entity PurchaseOrderItems : cuid {
  order          : Association to PurchaseOrders;
  product        : Association to Products;
  quantity       : Integer;
  unitPrice      : Decimal(10,2);
  grossAmount    : Decimal(12,2);
  currency       : Currency;
  deliveryDate   : Date;
}

// ─── STOCK (Inventory tracking) ──────────────────
entity Stock : managed {
  key product    : Association to Products;
  key warehouse  : String(20);
  quantity       : Integer;
  minQuantity    : Integer default 10;
  lastUpdated    : DateTime;
}
```

---

#### Step 3: Create the Service

Create `srv/epm-service.cds`:

```cds
using com.epm from '../db/schema';

// ─── Sales Service (Customer-facing) ─────────────
service SalesService @(path: '/sales') {
  entity Products     as projection on epm.Products;
  entity Customers    as projection on epm.Customers;
  entity SalesOrders  as projection on epm.SalesOrders;
  entity Categories   as projection on epm.Categories;
}

// ─── Procurement Service (Internal) ──────────────
service ProcurementService @(path: '/procurement') {
  entity Suppliers       as projection on epm.Suppliers;
  entity PurchaseOrders  as projection on epm.PurchaseOrders;
  entity Products        as projection on epm.Products;
  entity Stock           as projection on epm.Stock;
}
```

---

#### Step 4: Add Sample Data

Create `db/data/sap.common-Currencies.csv`:

```csv
code;name;descr;symbol
INR;Indian Rupee;Currency of India;₹
USD;US Dollar;Currency of United States;$
EUR;Euro;Currency of European Union;€
```

Create `db/data/sap.common-Countries.csv`:

```csv
code;name;descr
IN;India;Republic of India
US;United States;United States of America
DE;Germany;Federal Republic of Germany
CN;China;People's Republic of China
JP;Japan;Japan
```

Create `db/data/com.epm-Suppliers.csv`:

```csv
ID;supplierName;contactPerson;email;phone;street;city;state;pincode;country_code;isActive
s1s1s1s1-1111-1111-1111-111111111111;TechParts India;Amit Shah;amit@techparts.in;9876543210;100 MG Road;Bangalore;Karnataka;560001;IN;true
s2s2s2s2-2222-2222-2222-222222222222;Global Electronics;Sarah Johnson;sarah@globalelec.com;+1-555-0101;200 Tech Blvd;San Jose;California;95110;US;true
s3s3s3s3-3333-3333-3333-333333333333;Berlin Components;Klaus Mueller;klaus@berlincomp.de;+49-30-1234;50 Technikstr;Berlin;Berlin;10115;DE;true
```

Create `db/data/com.epm-Categories.csv`:

```csv
ID;categoryName;description
c1c1c1c1-1111-1111-1111-111111111111;Electronics;Electronic devices and components
c2c2c2c2-2222-2222-2222-222222222222;Office Supplies;Stationery and office equipment
c3c3c3c3-3333-3333-3333-333333333333;Furniture;Office and home furniture
c4c4c4c4-4444-4444-4444-444444444444;Software;Software licenses and subscriptions
```

Create `db/data/com.epm-Products.csv`:

```csv
ID;productName;description;price;currency_code;weight;weightUnit;stock;minStock;supplier_ID;category_ID;isAvailable
p1p1p1p1-1111-1111-1111-111111111111;Laptop Pro 15;High performance laptop;85000.00;INR;2.100;KG;50;10;s1s1s1s1-1111-1111-1111-111111111111;c1c1c1c1-1111-1111-1111-111111111111;true
p2p2p2p2-2222-2222-2222-222222222222;Wireless Mouse;Ergonomic wireless mouse;1500.00;INR;0.150;KG;200;20;s2s2s2s2-2222-2222-2222-222222222222;c1c1c1c1-1111-1111-1111-111111111111;true
p3p3p3p3-3333-3333-3333-333333333333;Standing Desk;Height-adjustable desk;25000.00;INR;35.000;KG;15;5;s3s3s3s3-3333-3333-3333-333333333333;c3c3c3c3-3333-3333-3333-333333333333;true
p4p4p4p4-4444-4444-4444-444444444444;A4 Paper Ream;500 sheets white paper;250.00;INR;2.500;KG;500;50;s1s1s1s1-1111-1111-1111-111111111111;c2c2c2c2-2222-2222-2222-222222222222;true
p5p5p5p5-5555-5555-5555-555555555555;Monitor 27 inch;4K IPS Display;32000.00;INR;5.500;KG;30;8;s2s2s2s2-2222-2222-2222-222222222222;c1c1c1c1-1111-1111-1111-111111111111;true
```

Create `db/data/com.epm-Customers.csv`:

```csv
ID;customerName;email;phone;street;city;state;pincode;country_code;creditLimit
cu1cu1cu-1111-1111-1111-111111111111;Infosys Ltd;procurement@infosys.com;080-12345678;Electronics City;Bangalore;Karnataka;560100;IN;5000000.00
cu2cu2cu-2222-2222-2222-222222222222;TCS Mumbai;orders@tcs.com;022-98765432;Hiranandani Gardens;Mumbai;Maharashtra;400076;IN;3000000.00
cu3cu3cu-3333-3333-3333-333333333333;Wipro Technologies;supplies@wipro.com;080-87654321;Sarjapur Road;Bangalore;Karnataka;560035;IN;4000000.00
```

---

#### Step 5: Run and Test

```bash
cds watch
```

Test queries:

```http
// Sales Service:
GET http://localhost:4004/sales/Products?$expand=supplier,category
GET http://localhost:4004/sales/Customers?$expand=orders
GET http://localhost:4004/sales/Categories?$expand=products

// Procurement Service:
GET http://localhost:4004/procurement/Suppliers?$expand=products
GET http://localhost:4004/procurement/Products?$filter=price gt 10000&$expand=supplier

// Deep Insert — Create a Sales Order with items:
POST http://localhost:4004/sales/SalesOrders
Content-Type: application/json

{
  "orderNumber": "SO-001",
  "customer_ID": "cu1cu1cu-1111-1111-1111-111111111111",
  "orderDate": "2026-06-01",
  "grossAmount": 87500,
  "netAmount": 85000,
  "taxAmount": 2500,
  "currency_code": "INR",
  "status": "New",
  "priority": "High",
  "items": [
    {
      "product_ID": "p1p1p1p1-1111-1111-1111-111111111111",
      "quantity": 1,
      "unitPrice": 85000,
      "grossAmount": 85000,
      "netAmount": 83000,
      "taxAmount": 2000,
      "currency_code": "INR"
    },
    {
      "product_ID": "p2p2p2p2-2222-2222-2222-222222222222",
      "quantity": 2,
      "unitPrice": 1500,
      "grossAmount": 3000,
      "netAmount": 2800,
      "taxAmount": 200,
      "currency_code": "INR"
    }
  ]
}
```

---

## Session 6: Hands-on — Complete EPM & Practice (16:00 - 16:45)

### Challenge 1: Add Reviews to Products (5 minutes)

Add a `ProductReviews` entity with composition to Products:

```cds
entity ProductReviews : cuid, managed {
  product   : Association to Products;
  rating    : Integer;      // 1-5
  reviewer  : String(100);
  comment   : String(500);
  helpful   : Integer default 0;
}
```

Then add in `Products`:
```cds
reviews : Composition of many ProductReviews on reviews.product = $self;
```

Test: Create a product review via deep insert through a Product.

---

### Challenge 2: Add a self-referencing Category hierarchy (5 minutes)

Our Categories already have `parentCategory`. Now test:

```http
GET http://localhost:4004/sales/Categories?$expand=parentCategory
```

Add sub-categories in your CSV:
```csv
c5c5c5c5-5555-5555-5555-555555555555;Laptops;Laptop computers;c1c1c1c1-1111-1111-1111-111111111111
c6c6c6c6-6666-6666-6666-666666666666;Desks;Office desks;c3c3c3c3-3333-3333-3333-333333333333
```

---

### Challenge 3: Add Address as Composition (10 minutes)

Create an `Addresses` entity that is a Composition of Customers:

```cds
entity Addresses : cuid {
  customer    : Association to Customers;
  addressType : String(20);   // "Billing", "Shipping"
  street      : String(200);
  city        : String(100);
  state       : String(50);
  pincode     : String(10);
  country     : Country;
  isDefault   : Boolean default false;
}
```

Add to Customers:
```cds
addresses : Composition of many Addresses on addresses.customer = $self;
```

Test deep insert: Create a customer with billing and shipping addresses in ONE request.

---

### Challenge 4: Verify Associations in SQLite (5 minutes)

```bash
cds deploy --to sqlite:db/epm.db
sqlite3 db/epm.db ".schema com_epm_Products"
```

Notice the auto-generated foreign key columns: `supplier_ID`, `category_ID`.

```bash
sqlite3 db/epm.db "SELECT productName, supplier_ID, category_ID FROM com_epm_Products;"
```

---

## Assessment: MCQ (15 Questions)

**Q1.** An Association in CDS is used to:
- a) Define a new data type
- b) Create a relationship between two entities
- c) Add a primary key
- d) Create a namespace

**Answer: b)** — Associations connect entities, enabling navigation and $expand.

---

**Q2.** `author : Association to Authors;` creates which column in the database?
- a) `author` (stores the full author object)
- b) `author_ID` (stores the UUID of the author)
- c) `Authors_ID` (stores a reference)
- d) No column is created

**Answer: b)** — Managed associations auto-create a foreign key column named `<field>_ID`.

---

**Q3.** What does `$expand=author` do in an OData query?
- a) Expands the table size
- b) Returns the full author details inline with the book data
- c) Creates a new author
- d) Deletes the author reference

**Answer: b)** — $expand follows the association and includes the related entity's data in the response.

---

**Q4.** The difference between `Association to many` and `Composition of many` is:
- a) No difference — they are the same
- b) Composition creates more columns
- c) Association is faster than Composition
- d) Composition means the child entity is OWNED by the parent (deep insert/delete)   

**Answer: b)** — Composition = ownership. Children can't exist without parent. Deep insert/delete works.

---

**Q5.** When should you use Composition?
- a) When both entities are independent (like Book → Author)
- b) When the child entity has no meaning without the parent (like Order → OrderItems)
- c) Always — it's better than Association
- d) Never — use Association instead

**Answer: b)** — Use Composition when children BELONG to the parent and are meaningless alone.

---

**Q6.** Deep Insert means:
- a) Inserting data deep into the database
- b) Creating a parent record AND its child records in one single POST request
- c) Inserting very long text
- d) Inserting with maximum permissions

**Answer: b)** — Deep Insert creates the entire parent-child tree in one request, one transaction.

---

**Q7.** The `cuid` aspect from `@sap/cds/common` provides:
- a) Currency support
- b) `key ID : UUID` automatically
- c) Created/modified timestamps
- d) Localized text support

**Answer: b)** — `cuid` adds `key ID : UUID` so you don't have to write it in every entity.

---

**Q8.** The `managed` aspect automatically fills:
- a) Name and description
- b) createdAt, createdBy, modifiedAt, modifiedBy
- c) Price and currency
- d) Key and UUID

**Answer: b)** — `managed` auto-tracks who created/modified the record and when.

---

**Q9.** `entity Books : cuid, managed { ... }` means:
- a) Books IS a cuid and IS a managed entity
- b) Books INCLUDES fields from both cuid (UUID key) and managed (tracking fields)
- c) Books cannot have its own fields
- d) Only cuid fields are included

**Answer: b)** — The `:` syntax includes/extends — Books gets all fields from both aspects PLUS its own.

---

**Q10.** `Currency` from `@sap/cds/common` gives you:
- a) A Decimal field for amount
- b) Automatic currency conversion
- c) An association to a standardized Currencies code list (with code, name, symbol)   
- d) A String(3) field only

**Answer: b)** — It's an association to `sap.common.Currencies`, a code-list entity with name, description, symbol.

---

**Q11.** For a Many-to-Many relationship (like Books ↔ Genres), you need:
- a) Two Composition fields
- b) A link/junction entity with associations to both sides
- c) A special CDS keyword `many-to-many`
- d) Duplicate fields in both entities

**Answer: b)** — Create a junction entity (like `BookGenres`) with key associations to both `Books` and `Genres`.

---

**Q12.** A managed association differs from unmanaged in that:
- a) Managed auto-creates the FK column; unmanaged requires you to define it and the join
- b) Managed is slower
- c) Unmanaged is the recommended approach
- d) There is no difference

**Answer: a)** — Managed: `field : Association to X;` (FK auto-created). Unmanaged: you define the FK and write `on` condition.

---

**Q13.** `on reviews.book = $self` in `Association to many Reviews on reviews.book = $self` means:
- a) Reviews must have a book
- b) Find all Reviews where the review's `book` field points back to THIS entity instance
- c) Create a self-reference
- d) The review is about self-help books

**Answer: b)** — `$self` refers to the current entity instance. It defines how to navigate from parent to children.

---

**Q14.** What does the `temporal` aspect provide?
- a) Time zone support
- b) `validFrom` and `validTo` fields for time-based data versioning
- c) Automatic deletion after time expires
- d) DateTime fields for all columns

**Answer: b)** — `temporal` adds `validFrom/validTo` for tracking when data is valid (e.g., price changes over time).

---

**Q15.** If you delete a `SalesOrder` that has `items : Composition of many SalesOrderItems`, what happens?
- a) Only the order is deleted, items remain
- b) The order AND all its items are deleted automatically
- c) An error occurs — you must delete items first
- d) Items become null

**Answer: b)** — Composition means ownership. Deleting the parent cascades to all children automatically.

---

## Quiz: Data Modeling Challenge

### Scenario: Design a "Food Delivery App" Database Schema

You are building the backend for a food delivery app called "QuickBite." Design the CDS data model.

### Requirements

Design entities for:

1. **Restaurants** — Name, address, rating, cuisine type, is open, delivery radius
2. **MenuItems** — Name, description, price, category (Starter/Main/Dessert/Drink), is vegetarian, is available, spice level
3. **Customers** — Name, email, phone, default address
4. **Orders** — Order number, date, status, total, delivery address, payment method, delivery fee, estimated time
5. **OrderItems** — Which menu item, quantity, price, special instructions

### Questions to Answer

**Q1.** What is the relationship between Restaurant and MenuItems?
- Think: Can a menu item exist without a restaurant?
- Should it be Association or Composition? Why?

<details>
<summary>Answer</summary>

**Composition** — Menu items BELONG to the restaurant. If the restaurant is removed, its menu items are meaningless. You'd also want deep insert (create restaurant + menu in one go).

```cds
entity Restaurants : cuid, managed {
  name     : String(100);
  menu     : Composition of many MenuItems on menu.restaurant = $self;
}
```
</details>

---

**Q2.** What is the relationship between Order and OrderItems?
- Think: Can an order item exist without an order?

<details>
<summary>Answer</summary>

**Composition** — Order items are owned by the order. Delete order → delete its items. Deep insert to create order + items together.

```cds
entity Orders : cuid, managed {
  items : Composition of many OrderItems on items.order = $self;
}
```
</details>

---

**Q3.** What is the relationship between Order and Customer?
- Think: Does the customer exist independently of any order?

<details>
<summary>Answer</summary>

**Association** — Customer exists independently. Many orders belong to one customer. Deleting an order doesn't delete the customer!

```cds
entity Orders : cuid, managed {
  customer : Association to Customers;
}
```
</details>

---

**Q4.** What is the relationship between OrderItem and MenuItem?
- Think: Does the menu item exist on its own?

<details>
<summary>Answer</summary>

**Association** — The menu item is a reference (product catalog). It exists regardless of any order. Many order items can reference the same menu item.

```cds
entity OrderItems : cuid {
  order    : Association to Orders;
  menuItem : Association to MenuItems;
}
```
</details>

---

**Q5.** Write the complete CDS model for QuickBite using:
- `cuid` and `managed` aspects
- `Currency` from `@sap/cds/common`
- Proper Associations and Compositions
- At least one enum type

<details>
<summary>Complete Answer</summary>

```cds
using { cuid, managed, Currency } from '@sap/cds/common';

namespace com.quickbite;

type SpiceLevel : String(10) enum {
  Mild;
  Medium;
  Hot;
  ExtraHot;
}

type OrderStatus : String(20) enum {
  Placed;
  Confirmed;
  Preparing;
  OutForDelivery;
  Delivered;
  Cancelled;
}

type MenuCategory : String(20) enum {
  Starter;
  Main;
  Dessert;
  Drink;
  Snack;
}

// ─── RESTAURANTS ─────────────────────────────────
entity Restaurants : cuid, managed {
  name            : String(100);
  street          : String(200);
  city            : String(100);
  pincode         : String(10);
  rating          : Decimal(2,1);
  cuisineType     : String(50);
  isOpen          : Boolean default true;
  deliveryRadius  : Integer;     // in km
  menu            : Composition of many MenuItems on menu.restaurant = $self;
}

// ─── MENU ITEMS (owned by Restaurant) ────────────
entity MenuItems : cuid {
  restaurant      : Association to Restaurants;
  itemName        : String(100);
  description     : String(500);
  price           : Decimal(8,2);
  currency        : Currency;
  category        : MenuCategory;
  isVegetarian    : Boolean default false;
  isAvailable     : Boolean default true;
  spiceLevel      : SpiceLevel;
  preparationTime : Integer;     // in minutes
}

// ─── CUSTOMERS ───────────────────────────────────
entity Customers : cuid, managed {
  name            : String(100);
  email           : String(255);
  phone           : String(20);
  defaultAddress  : String(300);
  orders          : Association to many Orders on orders.customer = $self;
}

// ─── ORDERS (with Composition to OrderItems) ─────
entity Orders : cuid, managed {
  orderNumber     : String(20);
  customer        : Association to Customers;
  restaurant      : Association to Restaurants;
  orderDate       : DateTime;
  status          : OrderStatus;
  totalAmount     : Decimal(10,2);
  deliveryFee     : Decimal(6,2) default 30.00;
  currency        : Currency;
  deliveryAddress : String(300);
  paymentMethod   : String(20);
  estimatedTime   : Integer;    // in minutes
  items           : Composition of many OrderItems on items.order = $self;
}

// ─── ORDER ITEMS (owned by Order) ────────────────
entity OrderItems : cuid {
  order           : Association to Orders;
  menuItem        : Association to MenuItems;
  quantity        : Integer default 1;
  unitPrice       : Decimal(8,2);
  totalPrice      : Decimal(8,2);
  specialInstructions : String(200);
}
```
</details>

---

**Q6.** Draw the relationship diagram showing Association (→) and Composition (●→):

<details>
<summary>Answer</summary>

```
┌────────────┐          ┌────────────┐
│ Restaurants│●━━━━━━━━━│ MenuItems  │  (Composition)
└────────────┘          └────────────┘
      ▲                       ▲
      │ Association           │ Association
      │                       │
┌─────┴──────┐          ┌────┴───────┐
│   Orders   │●━━━━━━━━━│ OrderItems │  (Composition)
└────────────┘          └────────────┘
      │
      │ Association
      ▼
┌────────────┐
│ Customers  │
└────────────┘
```

- Restaurant ●→ MenuItems = Composition (restaurant OWNS its menu)
- Order ●→ OrderItems = Composition (order OWNS its items)
- Order → Customer = Association (customer exists independently)
- Order → Restaurant = Association (restaurant exists independently)
- OrderItem → MenuItem = Association (menu item is just a reference)
</details>

---

## Key Takeaways

| # | Topic | One-Line Summary |
|---|---|---|
| 1 | Association | Connects two entities — like a "points to" relationship |
| 2 | Managed Association | `field : Association to X;` — CAP auto-creates FK column (`field_ID`) |
| 3 | Unmanaged Association | You define the FK and write the `on` condition manually |
| 4 | One-to-Many | Use `Association to many X on X.field = $self` on the "one" side |
| 5 | Many-to-Many | Needs a junction/link table entity with key associations to both sides |
| 6 | Composition | `Composition of many X` — parent OWNS children (deep insert/delete) |
| 7 | Association vs Composition | Can child exist alone? Yes → Association. No → Composition |
| 8 | Deep Insert | Composition allows creating parent + children in ONE request |
| 9 | $expand | OData feature to fetch related data inline: `?$expand=author` |
| 10 | Aspect | Reusable model fragment included with `:` syntax |
| 11 | cuid | Adds `key ID : UUID` automatically |
| 12 | managed | Auto-fills createdAt/By, modifiedAt/By |
| 13 | temporal | Adds validFrom/validTo for time-sliced data |
| 14 | @sap/cds/common | SAP's library: cuid, managed, Currency, Country, Language, CodeList |
| 15 | Currency/Country/Language | Pre-built code-list associations with localized names |

---

## Assignment: Due Start of Day 14

### Task: Complete the EPM Model + Add a "Hotel Booking" Model

**Part A: Enhance EPM (5 marks)**

1. Add `ProductReviews` entity as a Composition of Products
2. Add `Addresses` as Composition of Customers (billing + shipping)
3. Add a `DeliverySchedules` entity as Composition of SalesOrderItems
4. Use `Currency` from `@sap/cds/common` in all amount fields
5. Test deep insert: Create a Sales Order with 3 items in one request

**Part B: Build a Hotel Booking Model (5 marks)**

Create a new project `hotel-booking` with namespace `com.hotelbooking` and these entities:

| Entity | Type | Key Relationship |
|--------|------|-----------------|
| Hotels | Parent | - |
| Rooms | Child of Hotel | Composition: hotel owns its rooms |
| Guests | Independent | - |
| Bookings | Independent | Association to Guest and Room |
| BookingExtras | Child of Booking | Composition: extras belong to booking |

Requirements:
- Use `cuid`, `managed` aspects
- Use `Currency` and `Country` from `@sap/cds/common`
- Hotels should have: name, address, city, country, stars (1-5), total rooms
- Rooms should have: room number, type (Single/Double/Suite), price per night, floor, is available
- Guests should have: name, email, phone, country, loyalty tier
- Bookings should have: check-in, check-out, total price, status, payment method
- BookingExtras: extra name (breakfast, spa, parking), price, quantity

Provide:
- Complete `db/schema.cds`
- Service exposing all entities
- CSV data for at least: 2 Hotels, 5 Rooms, 3 Guests
- Test deep insert: Create a booking with 2 extras

### Grading

| Criteria | Marks |
|----------|-------|
| EPM enhancements correct (Part A) | 5 |
| Hotel model with correct Association/Composition | 3 |
| Uses aspects and @sap/cds/common | 1 |
| Deep insert works | 1 |
| **Total** | **10** |

---

## Preparation for Day 14

Tomorrow: **CDS Services & OData — Exposing Data as APIs**

You'll learn:
- Service definitions in detail
- Projections, excluding/renaming fields
- Read-only vs. read-write entities in services
- Actions and Functions in CDS
- OData query options deep-dive
- Custom logic with JavaScript handlers

**To prepare:**
- Make sure both your Library and EPM projects run with `cds watch`
- Try all `$expand` queries to understand navigation
- Think about: What if you want to HIDE some fields from the API? (e.g., internal cost price)
- Think about: What if you need a custom operation like "approve order" that isn't just CRUD?

---

*End of Day 13*
