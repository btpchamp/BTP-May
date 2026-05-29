# Day 12: CDS Data Modeling — Part 1 (Entities & Types)

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 11 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: CDS Language Syntax & Defining Entities | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Data Types, Keys & Namespaces | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Build Library Management Data Model | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Reusable Types, Enums & Best Practices | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Deploy to SQLite & Verify Tables | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Extend the Model & Practice | 45 min |
| 16:45 - 17:00 | Assessment & Wrap-up | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Write CDS entity definitions with correct syntax
- Choose appropriate CDS data types for different fields
- Define primary keys (including auto-generated UUIDs)
- Organize models using namespaces
- Create reusable types with the `type` keyword
- Use enums for fixed sets of values
- Deploy your model to SQLite and verify the tables were created
- Design a complete data model for a real business scenario

---

## Day 11 Recap — Quick Fire (09:00 - 09:15)

1. CAP stands for? → _____
2. Which command creates a new CAP project? → _____
3. Which command starts the local dev server? → _____
4. Data models go in which folder? → _____
5. Services go in which folder? → _____
6. What database does CAP use locally? → _____
7. `@sap/cds-dk` is installed where (global/local)? → _____
8. What does `as projection on` mean in a service? → _____

<details>
<summary>Answers</summary>

1. Cloud Application Programming Model
2. `cds init`
3. `cds watch`
4. `db/` folder
5. `srv/` folder
6. SQLite (in-memory, zero config)
7. Globally (`npm i -g @sap/cds-dk`)
8. Expose an entity through the service (create API endpoints for it)

</details>

---

## Session 1: CDS Language Syntax & Defining Entities (09:15 - 10:30)

### What is CDS?

**CDS** (Core Data Services) is CAP's own language for defining data models and services. It's NOT JavaScript — it's a separate, simpler, declarative language.

**Think of CDS as:**
- A blueprint language (like an architect's drawing)
- You describe WHAT you want (entities, fields, types)
- CAP takes that blueprint and BUILDS everything (tables, APIs, validations)

---

### Your First CDS File — The Basic Syntax

```cds
// This is a comment in CDS (same as JavaScript)

/* 
   Multi-line comments
   also work like this
*/

// Define an entity (= a database table):
entity Products {
  key ID   : Integer;       // Primary key
  name     : String(100);   // Column: VARCHAR(100)
  price    : Decimal(10,2); // Column: DECIMAL(10,2)
  stock    : Integer;       // Column: INTEGER
}
```

**That's it!** From this CDS definition, CAP creates:
- A database table called `Products`
- Columns: ID, name, price, stock
- Primary key on ID
- A full CRUD API when exposed in a service

---

### CDS Syntax Rules

| Rule | Example | Notes |
|------|---------|-------|
| Entities start with `entity` keyword | `entity Books { }` | Like `CREATE TABLE` in SQL |
| Fields are `name : Type;` | `title : String(100);` | Colon separates name from type |
| Primary key uses `key` keyword | `key ID : Integer;` | At least one key field required |
| Statements end with semicolons `;` | `name : String;` | Required! |
| Curly braces `{ }` wrap fields | `entity X { ... }` | Same as JavaScript objects |
| Case-sensitive | `Books` ≠ `books` | Convention: PascalCase for entities |

---

### Defining Entities — From Simple to Complex

#### Level 1: Basic Entity

```cds
entity Employees {
  key ID        : Integer;
  firstName     : String(50);
  lastName      : String(50);
  salary        : Decimal(10,2);
  isActive      : Boolean;
}
```

#### Level 2: More Data Types

```cds
entity Orders {
  key ID        : UUID;
  orderNumber   : String(20);
  customerName  : String(100);
  totalAmount   : Decimal(12,2);
  itemCount     : Integer;
  orderDate     : Date;
  deliveryDate  : Date;
  createdAt     : DateTime;
  notes         : String(500);
  isUrgent      : Boolean;
  status        : String(20);
}
```

#### Level 3: Multiple Entities (Related)

```cds
entity Departments {
  key ID   : Integer;
  name     : String(100);
  location : String(100);
}

entity Employees {
  key ID         : Integer;
  firstName      : String(50);
  lastName       : String(50);
  email          : String(100);
  departmentId   : Integer;   // FK to Departments (we'll do proper associations tomorrow!)
  hireDate       : Date;
  salary         : Decimal(10,2);
}
```

---

### Naming Conventions in CDS

| What | Convention | Example |
|------|-----------|---------|
| **Entities** | PascalCase, plural | `Products`, `PurchaseOrders`, `OrderItems` |
| **Fields** | camelCase | `firstName`, `orderDate`, `totalAmount` |
| **Keys** | Usually `ID` | `key ID : UUID;` |
| **Namespaces** | lowercase.dot.separated | `my.company.bookshop` |
| **Custom types** | PascalCase | `type Currency : String(3);` |

---

### Entity = Table — The Mental Model

```
CDS Entity:                        Database Table:
─────────────                      ──────────────
entity Products {                  CREATE TABLE Products (
  key ID   : Integer;                ID INTEGER PRIMARY KEY,
  name     : String(100);            name NVARCHAR(100),
  price    : Decimal(10,2);          price DECIMAL(10,2),
  stock    : Integer;                stock INTEGER
}                                  );
```

You write CDS (simple) → CAP generates SQL (complex) → Database creates the table.

---

### Practice: Write Your First Entity (5 minutes)

In your CAP project from Day 11, replace `db/schema.cds` with:

```cds
namespace my.bookshop;

entity Books {
  key ID     : Integer;
  title      : String(200);
  author     : String(100);
  genre      : String(50);
  price      : Decimal(8,2);
  stock      : Integer;
  pages      : Integer;
  isbn       : String(13);
  published  : Date;
  rating     : Decimal(2,1);
  inPrint    : Boolean;
}
```

Save the file. If `cds watch` is running, it auto-restarts!

---

## Session 2: Data Types, Keys & Namespaces (10:45 - 12:00)

### CDS Data Types — Complete Reference

#### Primitive Types

| CDS Type | SQL Equivalent | Description | Example Value |
|----------|---------------|-------------|--------------|
| `String(n)` | NVARCHAR(n) | Text up to n characters | `"Hello World"` |
| `String` | NVARCHAR(5000) | Text (default max 5000) | `"Long text..."` |
| `LargeString` | NCLOB | Unlimited text | Articles, descriptions |
| `Integer` | INTEGER | Whole numbers | `42`, `-5`, `0` |
| `Integer64` | BIGINT | Large whole numbers | `9007199254740991` |
| `Decimal(p,s)` | DECIMAL(p,s) | Exact decimal (p=total digits, s=after dot) | `99999.99` |
| `Double` | DOUBLE | Floating point (approximate) | `3.14159` |
| `Boolean` | BOOLEAN | True or false | `true`, `false` |
| `Date` | DATE | Date only (no time) | `"2026-05-29"` |
| `Time` | TIME | Time only (no date) | `"14:30:00"` |
| `DateTime` | TIMESTAMP | Date + time | `"2026-05-29T14:30:00Z"` |
| `Timestamp` | TIMESTAMP | Same as DateTime (alias) | `"2026-05-29T14:30:00Z"` |
| `UUID` | NVARCHAR(36) | Universally unique ID | `"a1b2c3d4-e5f6-..."` |
| `Binary(n)` | VARBINARY(n) | Binary data | File content |
| `LargeBinary` | BLOB | Large binary (images, PDFs) | Uploaded files |

---

#### Choosing the Right Type — Decision Guide

| Data You're Storing | Recommended Type | Why |
|--------------------|-----------------|-----|
| Person's name | `String(100)` | Names rarely exceed 100 chars |
| Email address | `String(255)` | RFC allows up to 254 chars |
| Product description | `String(1000)` or `LargeString` | Descriptions can be long |
| Price / Amount | `Decimal(12,2)` | Exact — no floating point errors! |
| Quantity / Count | `Integer` | Whole numbers only |
| True/False flag | `Boolean` | Only two values |
| Birth date | `Date` | No time component needed |
| Created timestamp | `DateTime` | Need both date and time |
| Primary key | `UUID` or `Integer` | UUID is globally unique (preferred) |
| Phone number | `String(20)` | Store as text (leading zeros, +, -) |
| Zip code | `String(10)` | Some codes have letters (UK, Canada) |
| Rating (1-5) | `Integer` or `Decimal(2,1)` | Integer if whole, Decimal for 4.5 |

---

#### Why Use `Decimal` Instead of `Double` for Money?

```javascript
// JavaScript floating-point problem:
console.log(0.1 + 0.2);  // 0.30000000000000004 ← WRONG!

// This is why prices MUST use Decimal (exact arithmetic):
// Decimal(10,2) stores EXACTLY ₹99.99 — no floating-point errors!
```

**Rule:** ALWAYS use `Decimal` for money, prices, amounts. NEVER use `Double` for financial data.

---

### Keys — Identifying Records Uniquely

Every entity MUST have at least one key field — it uniquely identifies each record.

#### Option 1: Integer Keys (Simple)

```cds
entity Products {
  key ID : Integer;    // You assign: 1, 2, 3, 4...
  name   : String(100);
}
```

**Pros:** Simple, readable, short URLs (`/Products(42)`)
**Cons:** You manage uniqueness, not globally unique, conflicts in distributed systems

#### Option 2: UUID Keys (Recommended for CAP!)

```cds
entity Products {
  key ID : UUID;       // Auto-generated: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
  name   : String(100);
}
```

**Pros:** Globally unique (no conflicts ever), auto-generated by CAP, works in distributed/multi-tenant systems
**Cons:** Long URLs, not human-readable

#### Option 3: Composite Keys (Multiple Fields)

```cds
entity OrderItems {
  key orderID : UUID;      // Part 1 of key
  key itemNumber : Integer; // Part 2 of key
  product : String(100);
  quantity : Integer;
}
```

**When to use:** When no single field is unique, but the combination is (e.g., order items).

#### UUID Auto-Generation

When you use `UUID` as key and POST a new entity WITHOUT providing an ID, CAP generates one automatically:

```http
POST /catalog/Products
Content-Type: application/json

{
  "name": "Laptop",
  "price": 65000
}

Response:
{
  "ID": "7c8f2a31-b5e9-4d12-8a3c-f1e2d3c4b5a6",   ← Auto-generated!
  "name": "Laptop",
  "price": 65000
}
```

---

### Namespaces — Organizing Your Models

**Namespaces** prevent naming conflicts when projects grow large or combine with others.

```cds
// Without namespace — risky if you have multiple modules:
entity Books { ... }       // Which "Books"? Bookshop? Library? Store?

// With namespace — clear and organized:
namespace my.company.bookshop;
entity Books { ... }       // Full name: my.company.bookshop.Books
```

#### Namespace Rules

```cds
namespace my.company.project;

// All entities BELOW this line belong to the namespace
entity Products { ... }    // Full: my.company.project.Products
entity Orders { ... }      // Full: my.company.project.Orders
```

#### Referencing Entities from Another Namespace

```cds
// In srv/service.cds:
using my.company.project from '../db/schema';

service CatalogService {
  entity Products as projection on project.Products;
}
```

#### Common Namespace Patterns

| Project Type | Namespace Example |
|-------------|-------------------|
| Training project | `my.bookshop` |
| Company project | `com.acme.procurement` |
| SAP standard | `sap.common` |
| Partner extension | `partner.xyz.extension` |

---

### Multiple Entities in One File

You can define as many entities as you want in a single `.cds` file:

```cds
namespace my.bookshop;

entity Books {
  key ID     : UUID;
  title      : String(200);
  price      : Decimal(8,2);
  stock      : Integer;
}

entity Authors {
  key ID     : UUID;
  name       : String(100);
  country    : String(50);
  birthYear  : Integer;
}

entity Genres {
  key code   : String(20);
  name       : String(50);
  description: String(200);
}

entity Publishers {
  key ID     : UUID;
  name       : String(100);
  city       : String(50);
  founded    : Integer;
}
```

Or split across multiple files in the same folder — CAP merges them automatically!

```
db/
├── books.cds         ← entity Books, entity Reviews
├── authors.cds       ← entity Authors
└── common.cds        ← shared types, enums
```

---

## Session 3: Hands-on — Build Library Management Data Model (12:00 - 13:00)

### Project: Library Management System

Let's build a real data model for a library. Create a new project or use your existing one.

#### Step 1: Create/Reset the Project

```bash
cd ~/cap-training
cds init library-management
cd library-management
npm install
code .
```

#### Step 2: Define the Data Model

Create `db/schema.cds`:

```cds
namespace lib.management;

// ─── BOOKS ───────────────────────────────────────
entity Books {
  key ID          : UUID;
  title           : String(200);
  isbn            : String(13);
  pages           : Integer;
  price           : Decimal(8,2);
  publishedDate   : Date;
  language        : String(30);
  edition         : Integer;
  totalCopies     : Integer;
  availableCopies : Integer;
  summary         : String(2000);
  coverImage      : String(500);  // URL to cover image
}

// ─── AUTHORS ─────────────────────────────────────
entity Authors {
  key ID        : UUID;
  firstName     : String(50);
  lastName      : String(50);
  nationality   : String(50);
  birthDate     : Date;
  biography     : String(1000);
  email         : String(100);
  isActive      : Boolean;
}

// ─── GENRES ──────────────────────────────────────
entity Genres {
  key code      : String(20);  // "FICTION", "SCIFI", "HISTORY"
  name          : String(50);
  description   : String(200);
  isActive      : Boolean;
}

// ─── MEMBERS ─────────────────────────────────────
entity Members {
  key ID          : UUID;
  memberNumber    : String(10);   // LIB-001, LIB-002
  firstName       : String(50);
  lastName        : String(50);
  email           : String(100);
  phone           : String(15);
  address         : String(200);
  joinDate        : Date;
  memberType      : String(20);   // "Student", "Faculty", "General"
  maxBooks        : Integer;       // Max books they can borrow
  isActive        : Boolean;
}

// ─── BORROWINGS (Transactions) ───────────────────
entity Borrowings {
  key ID          : UUID;
  memberId        : UUID;
  bookId          : UUID;
  borrowDate      : Date;
  dueDate         : Date;
  returnDate      : Date;         // null if not yet returned
  status          : String(20);   // "Borrowed", "Returned", "Overdue"
  fineAmount      : Decimal(8,2);
}
```

#### Step 3: Create the Service

Create `srv/library-service.cds`:

```cds
using lib.management from '../db/schema';

service LibraryService {
  entity Books      as projection on management.Books;
  entity Authors    as projection on management.Authors;
  entity Genres     as projection on management.Genres;
  entity Members    as projection on management.Members;
  entity Borrowings as projection on management.Borrowings;
}
```

#### Step 4: Add Sample Data

Create `db/data/lib.management-Books.csv`:

```csv
ID;title;isbn;pages;price;publishedDate;language;edition;totalCopies;availableCopies;summary;coverImage
1a2b3c4d-1111-1111-1111-111111111111;Clean Code;9780132350884;464;450.00;2008-08-01;English;1;5;3;A handbook of agile software craftsmanship;
2a2b3c4d-2222-2222-2222-222222222222;The Pragmatic Programmer;9780135957059;352;550.00;2019-09-13;English;2;3;2;Journey to mastery in software development;
3a2b3c4d-3333-3333-3333-333333333333;Sapiens;9780062316097;443;400.00;2015-02-10;English;1;4;4;A brief history of humankind;
4a2b3c4d-4444-4444-4444-444444444444;Atomic Habits;9780735211292;320;350.00;2018-10-16;English;1;6;5;Tiny changes remarkable results;
5a2b3c4d-5555-5555-5555-555555555555;Design Patterns;9780201633610;395;700.00;1994-10-31;English;1;2;1;Elements of reusable object-oriented software;
```

Create `db/data/lib.management-Authors.csv`:

```csv
ID;firstName;lastName;nationality;birthDate;biography;email;isActive
a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa;Robert;Martin;American;1952-12-05;Software engineer and author known for clean code principles;rmartin@email.com;true
b2b2b2b2-bbbb-bbbb-bbbb-bbbbbbbbbbbb;David;Thomas;British;1956-01-01;Co-author of The Pragmatic Programmer;dthomas@email.com;true
c3c3c3c3-cccc-cccc-cccc-cccccccccccc;Yuval Noah;Harari;Israeli;1976-02-24;Historian and professor at Hebrew University;yharari@email.com;true
d4d4d4d4-dddd-dddd-dddd-dddddddddddd;James;Clear;American;1986-01-22;Author and speaker focused on habits and decision making;jclear@email.com;true
```

Create `db/data/lib.management-Genres.csv`:

```csv
code;name;description;isActive
PROG;Programming;Books about software development and coding;true
HIST;History;Books about historical events and civilizations;true
SELF;Self-Help;Books about personal development and productivity;true
FICT;Fiction;Novels and fictional stories;true
SCIF;Science Fiction;Futuristic and speculative fiction;true
BUSI;Business;Books about management and entrepreneurship;true
```

Create `db/data/lib.management-Members.csv`:

```csv
ID;memberNumber;firstName;lastName;email;phone;address;joinDate;memberType;maxBooks;isActive
m1m1m1m1-1111-1111-1111-111111111111;LIB-001;Priya;Sharma;priya@email.com;9876543210;Bangalore, India;2024-01-15;Faculty;5;true
m2m2m2m2-2222-2222-2222-222222222222;LIB-002;Rahul;Kumar;rahul@email.com;9876543211;Mumbai, India;2024-03-20;Student;3;true
m3m3m3m3-3333-3333-3333-333333333333;LIB-003;Sneha;Patel;sneha@email.com;9876543212;Delhi, India;2024-06-10;General;4;true
```

#### Step 5: Run and Test

```bash
cds watch
```

Open browser:
- `http://localhost:4004/library/Books`
- `http://localhost:4004/library/Authors`
- `http://localhost:4004/library/Genres`
- `http://localhost:4004/library/Members`

Try OData queries:
- `http://localhost:4004/library/Books?$filter=price gt 400`
- `http://localhost:4004/library/Books?$select=title,price&$orderby=price desc`
- `http://localhost:4004/library/Members?$filter=memberType eq 'Student'`
- `http://localhost:4004/library/Genres?$filter=isActive eq true`

---

## Session 4: Reusable Types, Enums & Best Practices (13:45 - 14:45)

### Reusable Types with `type` Keyword

When multiple entities use the same field pattern, create a reusable type:

```cds
// ❌ WITHOUT reusable types (repetitive):
entity Products {
  key ID    : UUID;
  currency  : String(3);   // INR, USD, EUR
  status    : String(20);  // Draft, Active, Closed
}

entity Orders {
  key ID    : UUID;
  currency  : String(3);   // Same pattern repeated!
  status    : String(20);  // Same pattern repeated!
}
```

```cds
// ✅ WITH reusable types (DRY — Don't Repeat Yourself):
type Currency : String(3);     // Reusable!
type Status   : String(20);    // Reusable!
type Amount   : Decimal(12,2); // Reusable!
type Email    : String(255);   // Reusable!
type Phone    : String(20);    // Reusable!

entity Products {
  key ID    : UUID;
  price     : Amount;     // Uses the type
  currency  : Currency;   // Uses the type
  status    : Status;     // Uses the type
}

entity Orders {
  key ID     : UUID;
  total      : Amount;    // Same type, consistent!
  currency   : Currency;  // Same type, consistent!
  status     : Status;    // Same type, consistent!
}
```

**Benefits of reusable types:**
- Consistency: Same field always has same length/precision
- Single change: Update the type → updates everywhere
- Readability: `Amount` is clearer than `Decimal(12,2)`
- Documentation: Type name tells you what the field is for

---

### Common Reusable Types You Should Define

```cds
// Put these in a common file: db/common.cds

namespace my.types;

// ─── String Types ────────────────
type Name        : String(100);
type Description : String(1000);
type Email       : String(255);
type Phone       : String(20);
type URL         : String(500);
type Code        : String(20);

// ─── Numeric Types ───────────────
type Amount      : Decimal(12,2);
type Quantity    : Integer;
type Percentage  : Decimal(5,2);
type Rating      : Decimal(2,1);

// ─── Business Types ──────────────
type Currency    : String(3);     // ISO currency: INR, USD, EUR
type Country     : String(2);     // ISO country: IN, US, DE
type Language    : String(2);     // ISO language: en, de, hi
type Status      : String(20);
```

Then use them:

```cds
using my.types from './common';

entity Products {
  key ID      : UUID;
  name        : types.Name;
  description : types.Description;
  price       : types.Amount;
  currency    : types.Currency;
  rating      : types.Rating;
}
```

---

### Enums — Fixed Set of Values

Enums define a field that can ONLY have specific predefined values:

```cds
// Define enum type:
type OrderStatus : String enum {
  Draft;
  Submitted;
  Approved;
  Rejected;
  Completed;
  Cancelled;
}

type Priority : String enum {
  Low;
  Medium;
  High;
  Critical;
}

type MemberType : String enum {
  Student;
  Faculty;
  General;
  VIP;
}

// Use in an entity:
entity Orders {
  key ID     : UUID;
  status     : OrderStatus;     // Can only be one of the defined values
  priority   : Priority;
}

entity Members {
  key ID       : UUID;
  name         : String(100);
  memberType   : MemberType;    // Only: Student, Faculty, General, VIP
}
```

**What enums give you:**
- Validation: CAP rejects values not in the enum
- Documentation: API metadata shows allowed values
- Type safety: IDE auto-completes enum values

---

### Enums with Values

You can assign specific values (useful for codes):

```cds
type Currency : String(3) enum {
  INR = 'INR';
  USD = 'USD';
  EUR = 'EUR';
  GBP = 'GBP';
}

type BookStatus : Integer enum {
  Available  = 1;
  Borrowed   = 2;
  Reserved   = 3;
  Lost       = 4;
  Damaged    = 5;
}
```

---

### Default Values

Set default values for fields that are usually the same:

```cds
entity Orders {
  key ID      : UUID;
  status      : String(20) default 'Draft';
  priority    : String(10) default 'Medium';
  currency    : String(3)  default 'INR';
  quantity    : Integer    default 1;
  isActive    : Boolean    default true;
  createdAt   : DateTime   default $now;    // Current timestamp!
}
```

When a user creates a new order without specifying `status`, it automatically becomes `'Draft'`.

---

### Virtual Fields (Calculated, Not Stored in DB)

```cds
entity Products {
  key ID       : UUID;
  name         : String(100);
  price        : Decimal(8,2);
  stock        : Integer;
  virtual availability : String(20);  // Calculated at runtime, NOT in database
}
```

Virtual fields are populated in your JavaScript handler:

```javascript
srv.after('READ', 'Products', (products) => {
  for (const p of products) {
    p.availability = p.stock > 0 ? 'In Stock' : 'Out of Stock';
  }
});
```

---

### Data Modeling Best Practices

| Practice | Good ✅ | Bad ❌ | Why |
|----------|---------|--------|-----|
| Use UUID for keys | `key ID : UUID` | `key ID : Integer` | Globally unique, no conflicts |
| Use Decimal for money | `price : Decimal(10,2)` | `price : Double` | Exact arithmetic, no rounding errors |
| Use String for codes | `phone : String(20)` | `phone : Integer` | Phones have +, leading zeros |
| PascalCase entities | `PurchaseOrders` | `purchase_orders` | CDS convention |
| camelCase fields | `firstName` | `first_name` | CDS convention |
| Define reusable types | `type Amount : Decimal(12,2)` | Repeat `Decimal(12,2)` everywhere | DRY principle |
| Add namespaces | `namespace my.project` | No namespace | Prevents naming conflicts |
| Be specific with lengths | `String(100)` | `String` (defaults to 5000!) | Save storage, add validation |

---

## Session 5: Hands-on — Deploy to SQLite & Verify Tables (15:00 - 16:00)

### Deploying the Model to a Persistent SQLite Database

By default, `cds watch` uses an **in-memory** SQLite database (data disappears on restart). Let's deploy to a **persistent** file-based SQLite database.

#### Step 1: Deploy to SQLite File

```bash
cds deploy --to sqlite:db/library.db
```

**What this does:**
1. Compiles your CDS model to SQL
2. Creates a SQLite database file: `db/library.db`
3. Creates all tables
4. Loads all CSV data into tables

**Expected output:**
```
> filling lib.management.Books from db/data/lib.management-Books.csv
> filling lib.management.Authors from db/data/lib.management-Authors.csv
> filling lib.management.Genres from db/data/lib.management-Genres.csv
> filling lib.management.Members from db/data/lib.management-Members.csv
/> successfully deployed to db/library.db
```

---

#### Step 2: Verify Tables with SQLite

```bash
# See what's in the database:
sqlite3 db/library.db ".tables"
```

**Output:**
```
lib_management_Authors    lib_management_Genres
lib_management_Books      lib_management_Members
lib_management_Borrowings
```

```bash
# See the schema of a table:
sqlite3 db/library.db ".schema lib_management_Books"
```

**Output:**
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
  coverImage NVARCHAR(500),
  PRIMARY KEY(ID)
);
```

**See! CDS → SQL happened automatically. Your CDS types became SQL types!**

---

#### Step 3: Query the Data

```bash
# See all books:
sqlite3 db/library.db "SELECT title, price FROM lib_management_Books;"

# Count members:
sqlite3 db/library.db "SELECT COUNT(*) FROM lib_management_Members;"

# Find expensive books:
sqlite3 db/library.db "SELECT title, price FROM lib_management_Books WHERE price > 400;"
```

---

#### Step 4: Run with Persistent Database

Update `package.json` to use the file-based database:

```json
{
  "cds": {
    "requires": {
      "db": {
        "kind": "sqlite",
        "credentials": {
          "database": "db/library.db"
        }
      }
    }
  }
}
```

Now `cds watch` will use the persistent `library.db` file (data survives restarts!).

---

#### Step 5: CDS Compile — See Generated Output

```bash
# See the compiled SQL:
cds compile db/schema.cds --to sql

# See the compiled JSON (CSN - CDS Schema Notation):
cds compile db/schema.cds --to json

# See as HANA artifacts (what would be deployed to HANA):
cds compile db/schema.cds --to hana
```

**This shows you exactly what CAP generates from your CDS!**

---

## Session 6: Hands-on — Extend the Model & Practice (16:00 - 16:45)

### Challenge 1: Add a `Reviews` Entity (5 minutes)

Add to your `db/schema.cds`:

```cds
entity Reviews {
  key ID        : UUID;
  bookId        : UUID;
  memberId      : UUID;
  rating        : Integer;       // 1-5
  comment       : String(500);
  reviewDate    : Date;
}
```

Expose it in the service and test with `cds watch`.

---

### Challenge 2: Add a `Reservations` Entity (5 minutes)

```cds
entity Reservations {
  key ID           : UUID;
  bookId           : UUID;
  memberId         : UUID;
  reservationDate  : Date;
  expiryDate       : Date;
  status           : String(20);  // "Active", "Fulfilled", "Expired", "Cancelled"
}
```

---

### Challenge 3: Create Custom Types (5 minutes)

Add to the top of `db/schema.cds`:

```cds
// Reusable types for the library:
type Name        : String(100);
type Email       : String(255);
type Phone       : String(20);
type Amount      : Decimal(8,2);
type MemberCode  : String(10);

type BookStatus : String(20) enum {
  Available;
  Borrowed;
  Reserved;
  Lost;
  Maintenance;
}

type BorrowStatus : String(20) enum {
  Active;
  Returned;
  Overdue;
}
```

Now refactor your entities to use these types!

---

### Challenge 4: Add CSV Data for Borrowings (10 minutes)

Create `db/data/lib.management-Borrowings.csv`:

```csv
ID;memberId;bookId;borrowDate;dueDate;returnDate;status;fineAmount
b1b1b1b1-1111-1111-1111-111111111111;m1m1m1m1-1111-1111-1111-111111111111;1a2b3c4d-1111-1111-1111-111111111111;2026-05-01;2026-05-15;2026-05-12;Returned;0.00
b2b2b2b2-2222-2222-2222-222222222222;m2m2m2m2-2222-2222-2222-222222222222;2a2b3c4d-2222-2222-2222-222222222222;2026-05-10;2026-05-24;;Borrowed;0.00
b3b3b3b3-3333-3333-3333-333333333333;m3m3m3m3-3333-3333-3333-333333333333;5a2b3c4d-5555-5555-5555-555555555555;2026-05-05;2026-05-19;;Overdue;25.00
```

Verify data loaded: `http://localhost:4004/library/Borrowings`

---

## Assessment: MCQ (15 Questions)

**Q1.** In CDS, an entity corresponds to:
- a) A JavaScript function
- b) A database table
- c) An API endpoint only
- d) A CSS stylesheet

**Answer: b)** — An entity in CDS generates a database table. When exposed in a service, it also creates API endpoints.

---

**Q2.** The correct CDS syntax for defining a string field is:
- a) `name = String(100);`
- b) `name : String(100);`
- c) `name String(100);`
- d) `String name(100);`

**Answer: b)** — CDS uses colon `:` to separate field name from type, and ends with semicolon `;`.

---

**Q3.** For storing money/prices, which CDS type should you use?
- a) `Double`
- b) `Integer`
- c) `Decimal(10,2)`
- d) `String(20)`

**Answer: c)** — Decimal provides exact arithmetic. Double has floating-point errors (0.1 + 0.2 ≠ 0.3).

---

**Q4.** `key ID : UUID;` means:
- a) A regular field that stores text
- b) A primary key that auto-generates a globally unique identifier
- c) A foreign key reference
- d) A computed field

**Answer: b)** — UUID keys are auto-generated by CAP when not provided, ensuring global uniqueness.

---

**Q5.** The `namespace` keyword in CDS:
- a) Creates a new folder
- b) Organizes entities under a prefix to avoid naming conflicts
- c) Is required for every file
- d) Defines a JavaScript scope

**Answer: b)** — `namespace my.bookshop;` makes `Books` become `my.bookshop.Books` internally, preventing conflicts.

---

**Q6.** A reusable type `type Amount : Decimal(12,2);` is useful because:
- a) It makes the code run faster
- b) It ensures consistency and you only change it in one place
- c) It's required by CAP
- d) It creates a new table

**Answer: b)** — DRY principle: define once, use everywhere. Change the type definition → all fields using it update.

---

**Q7.** CDS enum `type Status : String enum { Draft; Active; Closed; }` means:
- a) The field can contain any string
- b) The field can ONLY contain "Draft", "Active", or "Closed"
- c) Three entities are created
- d) The field is optional

**Answer: b)** — Enums restrict values to predefined options. Invalid values are rejected.

---

**Q8.** CSV data files in CAP must use which separator?
- a) Comma `,`
- b) Semicolon `;`
- c) Tab
- d) Pipe `|`

**Answer: b)** — CAP CSV files use semicolons (`;`) as separators, NOT commas.

---

**Q9.** The command `cds deploy --to sqlite:db/data.db` does:
- a) Deploys to SAP BTP
- b) Compiles CDS to SQL, creates SQLite file, creates tables, and loads CSV data
- c) Starts the development server
- d) Installs dependencies

**Answer: b)** — It creates a persistent SQLite database from your CDS model and populates it with CSV data.

---

**Q10.** `entity Orders { key ID : UUID; status : String(20) default 'Draft'; }` — what does `default 'Draft'` do?
- a) Forces all orders to always be "Draft"
- b) Sets the initial value to "Draft" if not provided during creation
- c) Creates a validation rule
- d) Makes the field read-only

**Answer: b)** — Default values are applied when a record is created without specifying that field.

---

**Q11.** Which is the correct naming convention for CDS entities?
- a) `snake_case` — `purchase_orders`
- b) `PascalCase` — `PurchaseOrders`
- c) `camelCase` — `purchaseOrders`
- d) `UPPERCASE` — `PURCHASEORDERS`

**Answer: b)** — Entities use PascalCase (e.g., `PurchaseOrders`). Fields use camelCase (e.g., `orderDate`).

---

**Q12.** A `virtual` field in CDS:
- a) Is stored in the database like any other field
- b) Is NOT stored in the database — calculated at runtime in JavaScript handlers
- c) Creates a new table
- d) Is invisible to users

**Answer: b)** — Virtual fields exist only in the API response, calculated by your code. No database column is created.

---

**Q13.** Why store phone numbers as `String(20)` instead of `Integer`?
- a) Strings are faster
- b) Phone numbers can have leading zeros, +, -, spaces which Integer would lose
- c) CDS doesn't support Integer
- d) No reason — Integer works fine

**Answer: b)** — "+91 098765 43210" would lose the + and leading 0 as an Integer. Always use String for codes/identifiers.

---

**Q14.** To split your CDS model across multiple files, you:
- a) Can't — everything must be in one file
- b) Place multiple `.cds` files in the `db/` folder — CAP merges them automatically
- c) Must manually import each file
- d) Need a special configuration

**Answer: b)** — CAP auto-discovers and merges all `.cds` files in the `db/` folder. Split freely by domain!

---

**Q15.** Composite keys like `key orderID : UUID; key itemNumber : Integer;` are used when:
- a) You want faster queries
- b) No single field uniquely identifies a record — but the combination does
- c) You need two primary keys
- d) The entity has too many fields

**Answer: b)** — Example: OrderItems need both orderID + itemNumber to identify a specific line item. Neither alone is unique.

---

## Assignment: Design an E-Commerce Database Schema

### Due: Start of Day 13

Create a complete CDS data model for an e-commerce platform called "ShopEasy." 

Create a new CAP project `shop-easy` with the following requirements:

### Entities Required

| Entity | Key Fields | Purpose |
|--------|-----------|---------|
| **Products** | UUID | Product catalog |
| **Categories** | String code | Product categories (Electronics, Fashion, etc.) |
| **Customers** | UUID | Customer profiles |
| **Orders** | UUID | Customer orders |
| **OrderItems** | Composite (orderId + itemNo) | Line items in each order |
| **Addresses** | UUID | Customer addresses (billing/shipping) |
| **Reviews** | UUID | Product reviews by customers |
| **Sellers** | UUID | Marketplace sellers |

### Field Requirements

**Products** must include: name, description, price, compareAtPrice (MRP), stock, weight, brand, SKU, image URL, isActive, createdAt

**Customers** must include: name, email, phone, dateOfBirth, tier (Bronze/Silver/Gold/Platinum), totalOrders, totalSpent, joinedAt, isActive

**Orders** must include: orderNumber (auto-incrementing pattern), status (enum: Placed/Confirmed/Shipped/Delivered/Cancelled/Returned), totalAmount, shippingAmount, discountAmount, paymentMethod, paymentStatus, orderedAt, deliveredAt

**OrderItems** must include: quantity, unitPrice, totalPrice, discount

### Requirements

1. Use appropriate data types for ALL fields (Decimal for money, UUID for keys, etc.)
2. Create at least 3 reusable types (e.g., Amount, Email, Phone)
3. Define at least 2 enums (OrderStatus, PaymentMethod, CustomerTier)
4. Use a namespace: `com.shopeasy`
5. Add default values where appropriate
6. Create CSV sample data for at least: 5 Products, 3 Customers, 3 Categories
7. Expose ALL entities in a service
8. Deploy to SQLite and verify tables
9. Test with `cds watch` and show $filter/$orderby queries working

### Submission

- Project folder with complete working code
- Run `cds watch` without errors
- Include at least 3 OData test queries in a `test.http` file
- Screenshot of `sqlite3` showing tables were created

### Grading

| Criteria | Marks |
|----------|-------|
| All 8 entities defined with correct types | 3 |
| Reusable types used (at least 3) | 2 |
| Enums defined and used | 1 |
| Namespace and naming conventions correct | 1 |
| CSV data loads correctly | 1 |
| Service exposes all entities | 1 |
| `cds watch` runs without errors | 1 |
| **Total** | **10** |

---

## Key Takeaways

| # | Topic | One-Line Summary |
|---|---|---|
| 1 | CDS | CAP's declarative language for defining data models and services |
| 2 | Entity | A CDS entity = a database table (fields + key) |
| 3 | Syntax | `entity Name { key ID : Type; field : Type; }` |
| 4 | UUID | Best primary key type — globally unique, auto-generated |
| 5 | Decimal vs Double | Always Decimal for money (exact). Never Double (approximate). |
| 6 | Namespace | Organizes entities: `namespace x.y.z;` → `x.y.z.EntityName` |
| 7 | Reusable types | `type Amount : Decimal(12,2);` → use everywhere for consistency |
| 8 | Enums | `type Status : String enum { A; B; C; }` → restrict allowed values |
| 9 | Default values | `field : String default 'value';` → auto-applied on create |
| 10 | Virtual fields | Not in DB — calculated at runtime via JavaScript |
| 11 | CSV loading | File naming: `<namespace>-<Entity>.csv` with `;` separator |
| 12 | cds deploy | Compiles CDS → SQL → creates SQLite DB with data |
| 13 | cds compile | Shows what CAP generates (SQL, JSON, HANA) from your CDS |
| 14 | Multiple files | Split .cds across files in db/ — CAP auto-merges them |
| 15 | Best practices | UUID keys, Decimal for money, String for codes, PascalCase entities |

---

## Preparation for Day 13

Tomorrow: **CDS Data Modeling Part 2 — Associations & Compositions**

You'll learn:
- Associations (managed & unmanaged) — how to connect entities
- One-to-one, One-to-many, Many-to-many relationships
- Compositions (parent-child ownership)
- Aspects (reusable model fragments: cuid, managed, temporal)
- `@sap/cds/common` library (built-in types for Currency, Country, Language)

**To prepare:**
- Make sure your Library Management project runs with `cds watch`
- Think about: How would you connect Books → Authors? (A book HAS an author)
- Think about: How is an Order → OrderItems relationship different from Book → Author?

---

*End of Day 12*
