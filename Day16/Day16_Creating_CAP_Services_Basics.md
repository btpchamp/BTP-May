# Day 16: Creating CAP Services — Basics

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Week 3 Kickoff & Quick Recap | 15 min |
| 09:15 - 10:30 | Session 1: What is a Service in CAP? | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Service Definition Syntax & Projections | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Create CatalogService for Library | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: OData V4 Basics & $metadata | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: CAP vs Plain Express.js — A Comparison | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Test Services with Browser & REST Client | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain what a Service is in CAP and why it matters
- Write service definitions in CDS using proper syntax
- Expose database entities through services using `as projection on`
- Understand how service URLs are formed automatically
- Read and interpret the `$metadata` document
- Explain the basics of OData V4 protocol
- Compare CAP services with plain Express.js endpoints
- Create and test multiple services for different use cases

---

## Week 3 Kickoff & Quick Recap (09:00 - 09:15)

### Where Are We in the Journey?

```
Week 1 (Days 1-5):   Foundations    → SAP BTP, HTML, JavaScript, Node.js
Week 2 (Days 6-10):  Web Dev       → Express.js, REST APIs, NPM
Week 3 (Days 11-15): Database      → CAP intro, CDS modeling, views, data ✅
Week 3 (Days 16-20): Services      → 👈 YOU ARE HERE!
```

### What We've Built So Far

```
┌─────────────────────────────────────────────┐
│                YOUR CAP APP                  │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─────────────────────────────────┐        │
│  │   db/schema.cds                 │        │
│  │   ✅ Entities (tables)          │        │
│  │   ✅ Associations/Compositions  │        │
│  │   ✅ Views & Projections        │        │
│  │   ✅ CSV sample data            │        │
│  └─────────────────────────────────┘        │
│                                             │
│  ┌─────────────────────────────────┐        │
│  │   srv/???                       │  ← 🆕 │
│  │   How do we expose this data?   │  TODAY │
│  │   How do users access it?       │        │
│  └─────────────────────────────────┘        │
│                                             │
└─────────────────────────────────────────────┘
```

### Quick Fire: Days 11-15 Essentials

1. CDS entity becomes a _____ in the database? → _____
2. `cuid` gives you `key ID : _____`? → _____
3. Association means entities have _____ lifecycles? → _____
4. CSV separator in CAP? → _____
5. `cds watch` does what? → _____

<details>
<summary>Answers</summary>

1. Table
2. UUID
3. Independent
4. Semicolons (`;`)
5. Starts server with auto-reload on file changes

</details>

---

## Session 1: What is a Service in CAP? (09:15 - 10:30)

### The Restaurant Analogy

Imagine your database is a **kitchen** full of ingredients and recipes:

```
┌──────────────────────────────────────────────────────────────┐
│                         RESTAURANT                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  🍳 KITCHEN (Database)         📋 MENU (Service)             │
│  ┌──────────────────┐         ┌──────────────────┐          │
│  │ All ingredients  │         │ What customers   │          │
│  │ All recipes      │   ──►   │ can order        │          │
│  │ Secret sauces    │         │ (not everything!)│          │
│  │ Prep work        │         │                  │          │
│  └──────────────────┘         └──────────────────┘          │
│                                        │                     │
│                                        ▼                     │
│                               🧑 CUSTOMER (Client/UI)        │
│                               ┌──────────────────┐          │
│                               │ Orders from menu │          │
│                               │ Doesn't see      │          │
│                               │ kitchen chaos    │          │
│                               └──────────────────┘          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Key insight:** Customers don't walk into the kitchen and grab whatever they want. They order from the **menu** — which shows only what's available and in what format.

Similarly in CAP:
- **Database** (db/) = Kitchen — contains ALL your data and internal structures
- **Service** (srv/) = Menu — controls WHAT data is exposed and HOW
- **Client** (app/) = Customer — consumes data through the service

---

### Why Do We Need Services?

Without a service layer, everyone can access everything directly. That's dangerous!

```
❌ WITHOUT SERVICE (direct database access):
┌──────────┐     ┌──────────┐
│  Client  │ ──► │ Database │   Anyone can read/write anything!
└──────────┘     └──────────┘   No control, no security, no structure.

✅ WITH SERVICE (controlled access):
┌──────────┐     ┌──────────┐     ┌──────────┐
│  Client  │ ──► │ Service  │ ──► │ Database │
└──────────┘     └──────────┘     └──────────┘
                  Controls:
                  • WHAT you can see
                  • WHAT you can do (read/write/delete)
                  • HOW data is shaped
                  • WHO can access it
```

### What Does a Service Do?

| Responsibility | Example |
|---------------|---------|
| **Exposes data** | Makes Products available as an API endpoint |
| **Controls access** | Read-only for catalog, full CRUD for admin |
| **Shapes data** | Hides internal fields (createdBy, modifiedAt) |
| **Groups entities** | Sales-related endpoints in one service |
| **Defines the API contract** | Clients know exactly what's available |

---

### Services in the MVC Pattern

Remember MVC from Day 11?

```
┌──────────────────────────────────────────────────────┐
│                  CAP Application                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌────────────┐   ┌────────────┐   ┌────────────┐   │
│  │   Model    │   │ Controller │   │    View    │   │
│  │  (db/)     │   │   (srv/)   │   │   (app/)   │   │
│  │            │   │            │   │            │   │
│  │ schema.cds │◄──│ service.cds│──►│ Fiori UI   │   │
│  │ Entities   │   │ Endpoints  │   │ REST Client│   │
│  │ Tables     │   │ Logic      │   │ Browser    │   │
│  └────────────┘   └────────────┘   └────────────┘   │
│                                                      │
│   "What data       "What's exposed   "How users      │
│    exists"          & how"            see it"         │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Today's focus is the middle box — the Service/Controller layer.**

---

### Your First Service — It's Just One File!

Here's the simplest possible service:

```cds
// File: srv/cat-service.cds

using { my.bookshop as db } from '../db/schema';

service CatalogService {
  entity Books as projection on db.Books;
}
```

That's it! **3 lines of code** and you have:
- A fully functional REST API
- OData V4 support
- CRUD operations (Create, Read, Update, Delete)
- Query capabilities ($filter, $orderby, $top, $skip, $expand)
- A metadata document describing the API
- Auto-generated URL endpoints

Let that sink in. In Express.js, this would take 50-100 lines of code!

---

### What Happens When `cds watch` Reads This?

```
You write:                    CAP generates:
─────────────────────────     ─────────────────────────────────
service CatalogService {      → URL: /catalog
  entity Books ...            → Endpoint: /catalog/Books
}                             → Operations: GET, POST, PUT, DELETE
                              → Queries: $filter, $select, $expand...
                              → Metadata: /catalog/$metadata
```

**Naming convention:** `CatalogService` → URL path becomes `/catalog`
- Removes the word "Service" from the end
- Converts to lowercase
- That becomes the URL path!

| Service Name | URL Path |
|-------------|----------|
| `CatalogService` | `/catalog` |
| `AdminService` | `/admin` |
| `SalesService` | `/sales` |
| `HRService` | `/hr` |
| `MyCustomService` | `/my-custom` |

---

### The Service Index Page

When you run `cds watch` and open `http://localhost:4004`, you see something like:

```
Welcome to cds.services

Service Endpoints:
  /catalog
    - Books
    - Authors

Web Applications:
  (none)
```

This is the **service index** — it lists all services and their entities. Think of it as the restaurant's menu board by the door.

---

### Key Concept: Service = API Boundary

A service creates a **boundary** between your internal data model and the outside world:

```
                    ┌─── API BOUNDARY ───┐
                    │                    │
  Internal          │    Service         │    External
  (db/)             │    (srv/)          │    (Clients)
                    │                    │
  15 fields    ──►  │  Expose 8 fields   │  ──►  Clients see 8
  Secret data  ──►  │  Hide secrets      │  ──►  Never exposed
  Internal IDs ──►  │  Show public IDs   │  ──►  Clean API
  Raw tables   ──►  │  Renamed/reshaped  │  ──►  User-friendly
                    │                    │
                    └────────────────────┘
```

---

### Multiple Services for Different Audiences

A real application often has multiple services for different users:

```cds
// Public catalog — anyone can browse (read-only)
service CatalogService {
  @readonly entity Products as projection on db.Products;
  @readonly entity Categories as projection on db.Categories;
}

// Admin service — only admins can manage everything
service AdminService {
  entity Products as projection on db.Products;
  entity Categories as projection on db.Categories;
  entity Suppliers as projection on db.Suppliers;
}

// Reporting service — read-only aggregate views
service ReportingService {
  @readonly entity SalesSummary as projection on db.SalesSummary;
  @readonly entity TopProducts as projection on db.TopProducts;
}
```

**Why separate services?**
- Different security levels (public vs admin)
- Different data shapes (catalog shows less, admin shows everything)
- Different operations (catalog = read, admin = read+write)
- Cleaner API design (each service has a clear purpose)

---

### Quick Check: Understanding Services

**Question:** Which statement is TRUE about CAP services?

- A) A service creates new database tables
- B) A service is the only way clients can access data
- C) You can only have one service per application
- D) Services and database entities must have the same name

<details>
<summary>Answer</summary>

**B** — A service is the controlled access point. Clients access data THROUGH services, not directly from the database. Services don't create tables (entities do), you can have multiple services, and you can rename entities in services.

</details>

---

## Session 2: Service Definition Syntax & Projections (10:45 - 12:00)

### The `service` Keyword

Every service starts with the `service` keyword:

```cds
service <ServiceName> {
  // entities exposed here
}
```

**Rules for service names:**
- PascalCase (e.g., `CatalogService`, `AdminService`)
- Usually ends with "Service" (convention, not required)
- The name determines the URL path

---

### Importing Your Data Model

Before you can expose entities, you need to reference your schema:

```cds
// Method 1: Import specific namespace
using { my.bookshop } from '../db/schema';

service CatalogService {
  entity Books as projection on my.bookshop.Books;
}
```

```cds
// Method 2: Import with alias (cleaner!)
using { my.bookshop as db } from '../db/schema';

service CatalogService {
  entity Books as projection on db.Books;
}
```

```cds
// Method 3: Import specific entities
using { my.bookshop.Books, my.bookshop.Authors } from '../db/schema';

service CatalogService {
  entity Books as projection on Books;
  entity Authors as projection on Authors;
}
```

**Recommended:** Method 2 (alias) — short, clean, and clear.

---

### The Magic Words: `as projection on`

This is the key phrase that connects your service entity to your database entity:

```cds
entity <ServiceEntityName> as projection on <DatabaseEntity>;
```

What does `as projection on` mean?

```
Database Entity (db.Products):        Service Entity (Products):
┌──────────────────────────────┐      ┌──────────────────────────┐
│ ID            : UUID         │      │ ID            : UUID     │
│ productName   : String(100)  │ ──►  │ productName   : String   │
│ description   : String(500)  │      │ description   : String   │
│ price         : Decimal      │      │ price         : Decimal  │
│ stock         : Integer      │      │ stock         : Integer  │
│ internalCode  : String(20)   │      │ (exposed to API!)        │
│ costPrice     : Decimal      │      └──────────────────────────┘
│ createdAt     : Timestamp    │
│ createdBy     : String       │       Without exclusions,
│ modifiedAt    : Timestamp    │       ALL fields are exposed.
│ modifiedBy    : String       │
└──────────────────────────────┘
```

**Simple rule:** `as projection on` = "expose this entity through the service"

---

### Controlling What Gets Exposed

#### Option 1: Expose Everything (Default)

```cds
service CatalogService {
  entity Products as projection on db.Products;
  // ALL fields from db.Products are visible in the API
}
```

#### Option 2: Expose Only Specific Fields

```cds
service CatalogService {
  entity Products as projection on db.Products {
    ID,
    productName,
    description,
    price,
    stock
    // internalCode, costPrice, timestamps are HIDDEN
  };
}
```

#### Option 3: Exclude Specific Fields

```cds
service CatalogService {
  entity Products as projection on db.Products
    excluding { internalCode, costPrice };
  // Everything EXCEPT internalCode and costPrice
}
```

#### Option 4: Rename Fields in the API

```cds
service CatalogService {
  entity Products as projection on db.Products {
    ID,
    productName as name,        // Renamed: productName → name in API
    description as desc,        // Renamed in API only
    price,
    stock as quantity            // stock → quantity in API
  };
}
```

---

### The Difference: Full Exposure vs Controlled Exposure

```cds
// ─── DATABASE LAYER ─────────────────────────────
// db/schema.cds
namespace com.bookshop;

entity Books : cuid, managed {
  title       : String(200);
  author      : Association to Authors;
  price       : Decimal(10,2);
  stock       : Integer;
  isbn        : String(13);
  costPrice   : Decimal(10,2);   // Internal: what we paid
  internalRef : String(50);      // Internal reference code
}

entity Authors : cuid, managed {
  name        : String(100);
  email       : String(255);     // Private contact info
  books       : Association to many Books on books.author = $self;
}
```

```cds
// ─── SERVICE LAYER ──────────────────────────────
// srv/catalog-service.cds
using { com.bookshop as db } from '../db/schema';

// PUBLIC service — for customers browsing books
service CatalogService {
  @readonly entity Books as projection on db.Books {
    ID, title, price, stock, isbn,
    author    // Include the association (for $expand)
  };
  // Hides: costPrice, internalRef, createdAt, createdBy, etc.

  @readonly entity Authors as projection on db.Authors {
    ID, name
  };
  // Hides: email (private!), createdAt, createdBy, etc.
}

// ADMIN service — for bookshop staff
service AdminService {
  entity Books as projection on db.Books;    // ALL fields visible
  entity Authors as projection on db.Authors; // ALL fields visible
}
```

**Result:**
- `/catalog/Books` → shows title, price, stock, isbn (safe for public)
- `/admin/Books` → shows everything including costPrice (staff only)

---

### `@readonly` Annotation — Read-Only Entities

Adding `@readonly` before an entity means: **NO create, update, or delete — only GET requests.**

```cds
service CatalogService {
  @readonly entity Products as projection on db.Products;
  // ✅ GET /catalog/Products          → works
  // ✅ GET /catalog/Products/123      → works
  // ❌ POST /catalog/Products         → 405 Method Not Allowed
  // ❌ PUT /catalog/Products/123      → 405 Method Not Allowed
  // ❌ DELETE /catalog/Products/123   → 405 Method Not Allowed
}
```

**When to use `@readonly`:**
- Public catalogs (browse products, don't modify)
- Reporting views (read data, don't change it)
- Reference data (countries, currencies — users shouldn't modify)

There's also `@insertonly` (can create but not update/delete) — less common but useful for event logs.

---

### Service URL Structure — How URLs Are Formed

CAP automatically generates URLs following this pattern:

```
http://localhost:4004/<service-path>/<entity-name>
```

Let's trace how it works:

```cds
service CatalogService {           // → /catalog
  entity Books as projection ...;  // → /catalog/Books
  entity Authors as projection ..; // → /catalog/Authors
}

service AdminService {             // → /admin
  entity Products as projection ..; // → /admin/Products
  entity Orders as projection ...;  // → /admin/Orders
}
```

**Full URL examples:**

| Action | URL | HTTP Method |
|--------|-----|-------------|
| List all books | `GET /catalog/Books` | GET |
| Get one book | `GET /catalog/Books(ID)` | GET |
| Create a book | `POST /admin/Books` | POST |
| Update a book | `PUT /admin/Books(ID)` | PUT / PATCH |
| Delete a book | `DELETE /admin/Books(ID)` | DELETE |
| Get metadata | `GET /catalog/$metadata` | GET |

---

### Accessing a Single Entity by Key

To get ONE specific record, you put the key in parentheses:

```
GET /catalog/Books(3f4a2b1c-...)

For UUID keys: /catalog/Books(3f4a2b1c-d5e6-7890-abcd-ef1234567890)
For String keys: /catalog/Books('ISBN-123')
For Integer keys: /catalog/Books(42)
```

---

### Multiple Entities in One Service

A service typically groups related entities together:

```cds
using { com.bookshop as db } from '../db/schema';

service CatalogService {
  // All book-browsing related entities
  entity Books as projection on db.Books;
  entity Authors as projection on db.Authors;
  entity Genres as projection on db.Genres;
  entity Reviews as projection on db.Reviews;
}
```

This creates:
```
/catalog/Books
/catalog/Authors
/catalog/Genres
/catalog/Reviews
```

All under the same service URL prefix (`/catalog`).

---

### Practice: Predict the URLs

Given this service definition, write the URL for each operation:

```cds
service SalesService {
  entity Orders as projection on db.SalesOrders;
  entity Customers as projection on db.Customers;
  @readonly entity Products as projection on db.Products;
}
```

1. List all orders: → _____
2. Get customer with ID `abc-123`: → _____
3. Create a new order: → _____
4. Can you delete a product? → _____
5. Service metadata: → _____

<details>
<summary>Answers</summary>

1. `GET /sales/Orders`
2. `GET /sales/Customers(abc-123)`
3. `POST /sales/Orders` (with body)
4. **NO!** Products is `@readonly` — DELETE returns 405 Method Not Allowed
5. `GET /sales/$metadata`

</details>

---

## Session 3: Hands-on — Create CatalogService for Library (12:00 - 13:00)

### Exercise Overview

We'll build a complete service for the Library bookshop system. You'll create the service file, run it, and test every endpoint.

---

### Step 1: Verify Your Database Model

First, make sure you have a working data model. If you don't have one from previous days, create this:

**File: `db/schema.cds`**
```cds
namespace my.bookshop;

using { cuid, managed } from '@sap/cds/common';

entity Books : cuid, managed {
  title       : String(200);
  author      : Association to Authors;
  genre       : String(50);
  price       : Decimal(10,2);
  stock       : Integer default 0;
  rating      : Decimal(2,1);
  publishDate : Date;
  isbn        : String(13);
}

entity Authors : cuid, managed {
  name        : String(100);
  country     : String(50);
  email       : String(255);
  books       : Association to many Books on books.author = $self;
}

entity Reviews : cuid, managed {
  book        : Association to Books;
  reviewer    : String(100);
  rating      : Integer;
  comment     : String(500);
  reviewDate  : Date;
}
```

---

### Step 2: Create Sample Data

**File: `db/data/my.bookshop-Authors.csv`**
```csv
ID;name;country;email
a1000001-0000-0000-0000-000000000001;J.K. Rowling;United Kingdom;jk@example.com
a1000001-0000-0000-0000-000000000002;George Orwell;United Kingdom;george@example.com
a1000001-0000-0000-0000-000000000003;Paulo Coelho;Brazil;paulo@example.com
a1000001-0000-0000-0000-000000000004;Agatha Christie;United Kingdom;agatha@example.com
a1000001-0000-0000-0000-000000000005;Haruki Murakami;Japan;haruki@example.com
```

**File: `db/data/my.bookshop-Books.csv`**
```csv
ID;title;author_ID;genre;price;stock;rating;publishDate;isbn
b1000001-0000-0000-0000-000000000001;Harry Potter and the Philosopher's Stone;a1000001-0000-0000-0000-000000000001;Fantasy;12.99;150;4.8;1997-06-26;9780747532699
b1000001-0000-0000-0000-000000000002;1984;a1000001-0000-0000-0000-000000000002;Dystopian;9.99;80;4.7;1949-06-08;9780451524935
b1000001-0000-0000-0000-000000000003;The Alchemist;a1000001-0000-0000-0000-000000000003;Fiction;11.50;200;4.5;1988-01-01;9780062315007
b1000001-0000-0000-0000-000000000004;Murder on the Orient Express;a1000001-0000-0000-0000-000000000004;Mystery;10.99;60;4.6;1934-01-01;9780062693662
b1000001-0000-0000-0000-000000000005;Norwegian Wood;a1000001-0000-0000-0000-000000000005;Literary Fiction;13.99;45;4.3;1987-09-04;9780375704024
```

---

### Step 3: Create the Service File

**Create this file: `srv/cat-service.cds`**

```cds
using { my.bookshop as db } from '../db/schema';

/**
 * Public Catalog Service — for browsing books
 */
service CatalogService @(path: '/catalog') {

  // Books: public can browse but not modify
  @readonly entity Books as projection on db.Books {
    ID,
    title,
    genre,
    price,
    stock,
    rating,
    publishDate,
    isbn,
    author     // Include association for $expand
  };

  // Authors: show name and country only (hide email)
  @readonly entity Authors as projection on db.Authors {
    ID,
    name,
    country,
    books      // Include back-navigation
  };

  // Reviews: everyone can read, anyone can create
  @insertonly entity Reviews as projection on db.Reviews;
}
```

---

### Step 4: Run and Test

```bash
cds watch
```

You should see output like:
```
[cds] - serving CatalogService { path: '/catalog' }

[cds] - server listening on { url: 'http://localhost:4004' }
[cds] - launched at 6/2/2026, 9:30:00 AM, version: 8.x.x, in: 1.2s
```

---

### Step 5: Test Each Endpoint

Open your browser or REST client and test:

| # | Test | URL | Expected Result |
|---|------|-----|-----------------|
| 1 | Service index | `http://localhost:4004` | Shows CatalogService with Books, Authors, Reviews |
| 2 | All books | `http://localhost:4004/catalog/Books` | JSON array with 5 books |
| 3 | Single book | `http://localhost:4004/catalog/Books(b1000001-0000-0000-0000-000000000001)` | Harry Potter details |
| 4 | All authors | `http://localhost:4004/catalog/Authors` | JSON array with 5 authors |
| 5 | Books with author | `http://localhost:4004/catalog/Books?$expand=author` | Books with author details inline |
| 6 | Author with books | `http://localhost:4004/catalog/Authors?$expand=books` | Authors with their books |
| 7 | Filter by genre | `http://localhost:4004/catalog/Books?$filter=genre eq 'Fantasy'` | Only Harry Potter |
| 8 | Sort by price | `http://localhost:4004/catalog/Books?$orderby=price desc` | Most expensive first |
| 9 | Top 3 books | `http://localhost:4004/catalog/Books?$top=3` | First 3 books |
| 10 | Metadata | `http://localhost:4004/catalog/$metadata` | XML schema document |

---

### Step 6: Verify What's Hidden

Try accessing the email field that we excluded from Authors:

```
GET /catalog/Authors?$select=name,email
```

**Expected:** Error or email not returned — because we only exposed `ID`, `name`, `country`, and `books` in our projection.

Try creating a book (should fail because of `@readonly`):

```http
POST http://localhost:4004/catalog/Books
Content-Type: application/json

{
  "title": "New Book",
  "price": 15.99
}
```

**Expected:** `405 Method Not Allowed` — the catalog is read-only!

---

### Step 7: Add an Admin Service

Now add a second service for admin operations. Create or update the file:

**File: `srv/admin-service.cds`**

```cds
using { my.bookshop as db } from '../db/schema';

service AdminService @(path: '/admin') {
  entity Books as projection on db.Books;
  entity Authors as projection on db.Authors;
  entity Reviews as projection on db.Reviews;
}
```

Now test admin operations:

```http
### Create a new author (should work in admin!)
POST http://localhost:4004/admin/Authors
Content-Type: application/json

{
  "name": "Dan Brown",
  "country": "United States",
  "email": "dan@example.com"
}
```

```http
### Create a book for the new author
POST http://localhost:4004/admin/Books
Content-Type: application/json

{
  "title": "The Da Vinci Code",
  "genre": "Thriller",
  "price": 14.99,
  "stock": 100,
  "rating": 4.2,
  "isbn": "9780307474278",
  "author_ID": "<paste-the-new-author-ID-here>"
}
```

**Verification:**
- `GET /admin/Authors` → shows email (all fields visible!)
- `GET /catalog/Authors` → hides email (only ID, name, country!)
- Same data, different views through different services!

---

## Session 4: OData V4 Basics & $metadata (13:45 - 14:45)

### What is OData?

**OData** (Open Data Protocol) is a standard protocol for building REST APIs. Think of it as **"SQL for the web"** — it lets you query data using URL parameters.

```
Traditional REST:    You get whatever the server decided to send
OData:               You tell the server exactly what you want!
```

**Real-world analogy:** 
- Traditional REST = You order "a meal" and get whatever the chef sends
- OData = You say "I want pasta, no onions, extra cheese, sorted by spiciness"

---

### OData Version: V4

CAP uses **OData V4** — the latest standard. Key features:
- JSON responses by default (V2 used XML)
- Better query capabilities
- Standardized URL patterns
- Better support for actions and functions

---

### OData Query Options — Your Querying Superpowers

These are URL parameters that start with `$`:

| Query Option | What It Does | SQL Equivalent |
|-------------|--------------|----------------|
| `$select` | Choose which fields to return | `SELECT field1, field2` |
| `$filter` | Filter records by condition | `WHERE` |
| `$orderby` | Sort results | `ORDER BY` |
| `$top` | Limit number of results | `LIMIT` |
| `$skip` | Skip first N results | `OFFSET` |
| `$expand` | Include related entities | `JOIN` |
| `$count` | Include total count in response | `COUNT(*)` |
| `$search` | Full-text search | `LIKE '%term%'` |

---

### $select — Choose Your Fields

Only return the fields you need (saves bandwidth!):

```
GET /catalog/Books?$select=title,price

Response:
{
  "value": [
    { "title": "Harry Potter...", "price": 12.99 },
    { "title": "1984", "price": 9.99 },
    ...
  ]
}
```

Without `$select`, you get ALL fields. With it, you get only what you asked for.

---

### $filter — Filter Your Data

Like a WHERE clause in SQL:

```
// Books cheaper than 12
GET /catalog/Books?$filter=price lt 12

// Books in the Fantasy genre
GET /catalog/Books?$filter=genre eq 'Fantasy'

// Books with stock greater than 100
GET /catalog/Books?$filter=stock gt 100

// Combine conditions with 'and' / 'or'
GET /catalog/Books?$filter=price lt 15 and stock gt 50

// String contains
GET /catalog/Books?$filter=contains(title,'Harry')

// Starts with
GET /catalog/Books?$filter=startswith(title,'The')
```

**Filter operators:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `eq` | Equals | `genre eq 'Fantasy'` |
| `ne` | Not equals | `status ne 'Cancelled'` |
| `gt` | Greater than | `price gt 10` |
| `lt` | Less than | `stock lt 20` |
| `ge` | Greater than or equal | `rating ge 4.0` |
| `le` | Less than or equal | `price le 15.99` |
| `and` | Both conditions | `price gt 5 and price lt 20` |
| `or` | Either condition | `genre eq 'Fantasy' or genre eq 'Mystery'` |
| `not` | Negate | `not contains(title,'Test')` |

**String functions:**

| Function | Example |
|----------|---------|
| `contains(field,'text')` | `contains(title,'Potter')` |
| `startswith(field,'text')` | `startswith(name,'J')` |
| `endswith(field,'text')` | `endswith(email,'.com')` |
| `tolower(field)` | `tolower(name) eq 'orwell'` |

---

### $orderby — Sort Results

```
// Sort by price (cheapest first)
GET /catalog/Books?$orderby=price

// Sort by price (most expensive first)
GET /catalog/Books?$orderby=price desc

// Sort by genre, then by title within each genre
GET /catalog/Books?$orderby=genre,title

// Sort by rating descending, then price ascending
GET /catalog/Books?$orderby=rating desc,price asc
```

---

### $top and $skip — Pagination

```
// First 3 books
GET /catalog/Books?$top=3

// Skip first 3, get next 3 (page 2)
GET /catalog/Books?$top=3&$skip=3

// Page 3
GET /catalog/Books?$top=3&$skip=6
```

**Pagination pattern:**
```
Page 1: $top=10&$skip=0   (records 1-10)
Page 2: $top=10&$skip=10  (records 11-20)
Page 3: $top=10&$skip=20  (records 21-30)
Formula: $skip = (pageNumber - 1) × pageSize
```

---

### $expand — Include Related Data

Remember associations? `$expand` follows them and includes the related data inline:

```
// Books with their author details
GET /catalog/Books?$expand=author

Response:
{
  "value": [
    {
      "title": "Harry Potter...",
      "price": 12.99,
      "author": {                    ← Expanded!
        "name": "J.K. Rowling",
        "country": "United Kingdom"
      }
    },
    ...
  ]
}
```

Without `$expand`:
```json
{
  "title": "Harry Potter...",
  "author_ID": "a1000001-..."        ← Just the ID (no details)
}
```

With `$expand=author`:
```json
{
  "title": "Harry Potter...",
  "author": {                         ← Full author object!
    "ID": "a1000001-...",
    "name": "J.K. Rowling",
    "country": "United Kingdom"
  }
}
```

---

### Combining Query Options

You can combine ALL of these in one request:

```
GET /catalog/Books?$select=title,price,rating
                  &$filter=price lt 15 and rating ge 4.5
                  &$orderby=rating desc
                  &$top=5
                  &$expand=author($select=name)
```

This says: "Give me the title, price, and rating of books under $15 with rating 4.5+, sorted by rating, top 5, with author name."

**In SQL, that would be:**
```sql
SELECT b.title, b.price, b.rating, a.name
FROM Books b
JOIN Authors a ON b.author_ID = a.ID
WHERE b.price < 15 AND b.rating >= 4.5
ORDER BY b.rating DESC
LIMIT 5;
```

OData gives you this power in the URL!

---

### The $metadata Document — Your API Blueprint

Every OData service has a metadata endpoint:

```
GET /catalog/$metadata
```

This returns an XML document describing:
- Every entity in the service
- Every field, its type, and constraints
- Relationships between entities
- Which operations are allowed

**Example response (simplified):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<edmx:Edmx Version="4.0">
  <edmx:DataServices>
    <Schema Namespace="CatalogService">
      
      <EntityType Name="Books">
        <Key>
          <PropertyRef Name="ID"/>
        </Key>
        <Property Name="ID" Type="Edm.Guid" Nullable="false"/>
        <Property Name="title" Type="Edm.String" MaxLength="200"/>
        <Property Name="genre" Type="Edm.String" MaxLength="50"/>
        <Property Name="price" Type="Edm.Decimal" Precision="10" Scale="2"/>
        <Property Name="stock" Type="Edm.Int32"/>
        <Property Name="rating" Type="Edm.Decimal" Precision="2" Scale="1"/>
        <NavigationProperty Name="author" Type="CatalogService.Authors"/>
      </EntityType>
      
      <EntityType Name="Authors">
        <Key>
          <PropertyRef Name="ID"/>
        </Key>
        <Property Name="ID" Type="Edm.Guid" Nullable="false"/>
        <Property Name="name" Type="Edm.String" MaxLength="100"/>
        <Property Name="country" Type="Edm.String" MaxLength="50"/>
        <NavigationProperty Name="books" Type="Collection(CatalogService.Books)"/>
      </EntityType>
      
    </Schema>
  </edmx:DataServices>
</edmx:Edmx>
```

**Why does $metadata matter?**
- Tools like SAP Fiori use it to auto-generate UIs
- Postman/REST clients use it to understand your API
- It's the contract: "This is what my API provides"
- If you change your service definition, $metadata updates automatically

---

### Reading $metadata: Key Things to Spot

| XML Element | What It Tells You |
|-------------|-------------------|
| `<EntityType Name="Books">` | An entity called Books exists |
| `<Property Name="title" Type="Edm.String">` | Books has a string field "title" |
| `<NavigationProperty>` | Relationship to another entity |
| `Type="Collection(...)"` | One-to-many (returns array) |
| `Nullable="false"` | This field is required |
| `MaxLength="200"` | String can be up to 200 chars |

---

### OData Type Mapping

How CDS types appear in OData $metadata:

| CDS Type | OData Type (Edm) |
|----------|-----------------|
| `String(n)` | `Edm.String` (MaxLength=n) |
| `Integer` | `Edm.Int32` |
| `Integer64` | `Edm.Int64` |
| `Decimal(p,s)` | `Edm.Decimal` (Precision=p, Scale=s) |
| `Boolean` | `Edm.Boolean` |
| `Date` | `Edm.Date` |
| `DateTime` | `Edm.DateTimeOffset` |
| `UUID` | `Edm.Guid` |

---

## Session 5: CAP vs Plain Express.js — A Comparison (15:00 - 16:00)

### Why This Comparison Matters

You've already built REST APIs with Express.js (Days 7-9). Now see how CAP does the same thing with 90% less code. This comparison will make you appreciate what CAP gives you for free.

---

### Scenario: Build a Books API

**Requirements:**
- List all books (with filtering, sorting, pagination)
- Get a single book by ID
- Create a new book
- Update a book
- Delete a book
- Include author details via join

---

### Express.js Approach (What You Did Before)

```javascript
// server.js — Express.js (approximately 80-100 lines)
const express = require('express');
const sqlite3 = require('better-sqlite3');
const app = express();
app.use(express.json());

const db = sqlite3('books.db');

// GET /books — List with filtering and pagination
app.get('/books', (req, res) => {
  try {
    let sql = 'SELECT * FROM books';
    const conditions = [];
    const params = [];
    
    // Manual filtering
    if (req.query.genre) {
      conditions.push('genre = ?');
      params.push(req.query.genre);
    }
    if (req.query.minPrice) {
      conditions.push('price >= ?');
      params.push(req.query.minPrice);
    }
    
    if (conditions.length > 0) {
      sql += ' WHERE ' + conditions.join(' AND ');
    }
    
    // Manual sorting
    if (req.query.sort) {
      sql += ` ORDER BY ${req.query.sort}`;
      if (req.query.order === 'desc') sql += ' DESC';
    }
    
    // Manual pagination
    const limit = parseInt(req.query.limit) || 10;
    const offset = parseInt(req.query.offset) || 0;
    sql += ` LIMIT ${limit} OFFSET ${offset}`;
    
    const books = db.prepare(sql).all(...params);
    res.json(books);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// GET /books/:id — Single book with author
app.get('/books/:id', (req, res) => {
  try {
    const book = db.prepare(`
      SELECT b.*, a.name as authorName, a.country as authorCountry
      FROM books b
      LEFT JOIN authors a ON b.author_id = a.id
      WHERE b.id = ?
    `).get(req.params.id);
    
    if (!book) return res.status(404).json({ error: 'Not found' });
    res.json(book);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// POST /books — Create
app.post('/books', (req, res) => {
  try {
    const { title, genre, price, stock, author_id } = req.body;
    
    // Manual validation
    if (!title) return res.status(400).json({ error: 'Title required' });
    if (!price) return res.status(400).json({ error: 'Price required' });
    
    const id = crypto.randomUUID();
    db.prepare(`
      INSERT INTO books (id, title, genre, price, stock, author_id)
      VALUES (?, ?, ?, ?, ?, ?)
    `).run(id, title, genre, price, stock || 0, author_id);
    
    const created = db.prepare('SELECT * FROM books WHERE id = ?').get(id);
    res.status(201).json(created);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// PUT /books/:id — Update
app.put('/books/:id', (req, res) => {
  try {
    const existing = db.prepare('SELECT * FROM books WHERE id = ?').get(req.params.id);
    if (!existing) return res.status(404).json({ error: 'Not found' });
    
    const { title, genre, price, stock } = req.body;
    db.prepare(`
      UPDATE books SET title=?, genre=?, price=?, stock=? WHERE id=?
    `).run(title, genre, price, stock, req.params.id);
    
    const updated = db.prepare('SELECT * FROM books WHERE id = ?').get(req.params.id);
    res.json(updated);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// DELETE /books/:id — Delete
app.delete('/books/:id', (req, res) => {
  try {
    const result = db.prepare('DELETE FROM books WHERE id = ?').run(req.params.id);
    if (result.changes === 0) return res.status(404).json({ error: 'Not found' });
    res.status(204).send();
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Plus: you'd need to create the DB schema manually!
// Plus: no auto-documentation ($metadata)!
// Plus: no standardized query language!

app.listen(3000);
```

**Lines of code: ~90+**

---

### CAP Approach (What You Do Now)

```cds
// db/schema.cds (data model)
namespace my.bookshop;
using { cuid, managed } from '@sap/cds/common';

entity Books : cuid, managed {
  title  : String(200);
  author : Association to Authors;
  genre  : String(50);
  price  : Decimal(10,2);
  stock  : Integer default 0;
}

entity Authors : cuid, managed {
  name   : String(100);
  country: String(50);
  books  : Association to many Books on books.author = $self;
}
```

```cds
// srv/cat-service.cds (service)
using { my.bookshop as db } from '../db/schema';

service CatalogService {
  entity Books as projection on db.Books;
  entity Authors as projection on db.Authors;
}
```

**Lines of CDS: ~20**

**And you get MORE features:**
- All CRUD operations (GET, POST, PUT, PATCH, DELETE)
- $filter with ANY condition (not just genre and price!)
- $orderby on ANY field
- $top, $skip (pagination)
- $expand (joins)
- $select (field selection)
- $metadata (auto-documentation)
- Automatic UUID generation
- Automatic timestamps (createdAt, modifiedAt)
- Input validation based on types
- Proper error responses

---

### Side-by-Side Comparison

| Feature | Express.js | CAP |
|---------|-----------|-----|
| **List all items** | Write route + SQL | FREE (auto-generated) |
| **Get by ID** | Write route + SQL | FREE |
| **Create** | Write route + validation + SQL | FREE |
| **Update** | Write route + SQL | FREE |
| **Delete** | Write route + SQL | FREE |
| **Filtering** | Manual per field | $filter (ANY field, ANY condition) |
| **Sorting** | Manual implementation | $orderby (automatic) |
| **Pagination** | Manual limit/offset | $top/$skip (automatic) |
| **Joins/Relations** | Manual SQL JOINs | $expand (automatic) |
| **Field selection** | Manual (if at all) | $select (automatic) |
| **Documentation** | Write Swagger/OpenAPI manually | $metadata (auto-generated) |
| **Error handling** | Write try-catch everywhere | Built-in |
| **Input validation** | Write checks manually | Based on CDS types |
| **UUID generation** | `crypto.randomUUID()` | Automatic with `cuid` |
| **Audit fields** | Manual timestamp code | Automatic with `managed` |
| **Database schema** | Write CREATE TABLE SQL | Auto from CDS entities |
| **Code to maintain** | ~90+ lines per entity | ~5 lines per entity |

---

### The Power of Convention Over Configuration

CAP follows the principle of **"convention over configuration"**:

```
Express.js thought: "I'll let you configure everything manually"
CAP thought:        "I'll give you the most common thing by default.
                     You only write code when you need something different."
```

**CAP's conventions:**
- Entity → Database table (automatic)
- Service entity → REST endpoint (automatic)
- `cuid` → UUID primary key (automatic)
- `managed` → Timestamp fields (automatic)
- Associations → JOIN capability via $expand (automatic)
- CDS types → Input validation (automatic)

---

### When Do You Still Write JavaScript in CAP?

CAP handles the standard CRUD for free. You write custom JavaScript (in `srv/*.js`) only when you need:

- Custom validation logic ("price must be > 0 AND < 10000")
- Calculations ("grossAmount = quantity × unitPrice")
- External API calls ("check credit score before creating order")
- Side effects ("send email when order is created")
- Complex authorization ("user can only see their own orders")

We'll cover custom handlers in upcoming days!

---

### Express.js Isn't Dead — Here's When to Use Each

| Use Express.js When... | Use CAP When... |
|----------------------|----------------|
| Building a non-data-centric API | Building enterprise data applications |
| Need WebSocket/real-time | Need standard CRUD with OData |
| Building a custom auth server | Building SAP-integrated apps |
| Very simple 1-2 endpoint API | Multi-entity, relationship-heavy apps |
| Learning Node.js fundamentals | Building production SAP BTP apps |

**Bottom line:** You learned Express.js so you understand what CAP does for you under the hood. Now you know WHY CAP is so powerful.

---

## Session 6: Hands-on — Test Services with Browser & REST Client (16:00 - 16:45)

### Exercise: Complete Testing Workbook

Use either your browser or a REST client (VS Code REST Client extension or Postman) to complete these tasks.

---

### Part A: Basic Operations (10 minutes)

Test these queries against your CatalogService:

| # | Task | URL to Use | Expected |
|---|------|-----------|----------|
| 1 | Get all books | `/catalog/Books` | Array of books |
| 2 | Get only titles and prices | `/catalog/Books?$select=title,price` | Only 2 fields |
| 3 | Count total books | `/catalog/Books/$count` | A number |
| 4 | Get books with inline count | `/catalog/Books?$count=true` | Array + `@odata.count` |
| 5 | Get service metadata | `/catalog/$metadata` | XML document |

---

### Part B: Filtering (10 minutes)

| # | Task | Build the URL |
|---|------|--------------|
| 1 | Books with price less than 12 | `?$filter=_____` |
| 2 | Books in the "Fantasy" genre | `?$filter=_____` |
| 3 | Books with stock more than 100 | `?$filter=_____` |
| 4 | Books with rating 4.5 or higher | `?$filter=_____` |
| 5 | Books with "the" in the title | `?$filter=_____` |
| 6 | Fantasy OR Mystery books under $12 | `?$filter=_____` |

<details>
<summary>Answers</summary>

1. `$filter=price lt 12`
2. `$filter=genre eq 'Fantasy'`
3. `$filter=stock gt 100`
4. `$filter=rating ge 4.5`
5. `$filter=contains(title,'the')`
6. `$filter=(genre eq 'Fantasy' or genre eq 'Mystery') and price lt 12`

</details>

---

### Part C: Sorting & Pagination (5 minutes)

| # | Task | Build the URL |
|---|------|--------------|
| 1 | Books sorted by price (cheapest first) | `?$orderby=_____` |
| 2 | Books sorted by rating (highest first) | `?$orderby=_____` |
| 3 | Top 2 most expensive books | `?$orderby=_____&$top=_____` |
| 4 | Page 2 with 2 items per page | `?$top=_____&$skip=_____` |

<details>
<summary>Answers</summary>

1. `$orderby=price`
2. `$orderby=rating desc`
3. `$orderby=price desc&$top=2`
4. `$top=2&$skip=2`

</details>

---

### Part D: Expand (Relationships) (5 minutes)

| # | Task | Build the URL |
|---|------|--------------|
| 1 | Books with full author details | `?$expand=_____` |
| 2 | Authors with their list of books | `?$expand=_____` |
| 3 | Books with only author name | `?$expand=_____` |

<details>
<summary>Answers</summary>

1. `$expand=author`
2. `/catalog/Authors?$expand=books`
3. `$expand=author($select=name)`

</details>

---

### Part E: CRUD via REST Client (15 minutes)

Create a file `test.http` in your project root (for VS Code REST Client extension):

```http
### Variables
@base = http://localhost:4004

### 1. GET all books from catalog (read-only)
GET {{base}}/catalog/Books

### 2. GET books with filtering and expansion
GET {{base}}/catalog/Books?$filter=price lt 13&$expand=author&$orderby=title

### 3. TRY to create a book in CATALOG (should FAIL — readonly!)
POST {{base}}/catalog/Books
Content-Type: application/json

{
  "title": "Test Book",
  "price": 9.99
}

### 4. Create an author in ADMIN (should SUCCEED)
POST {{base}}/admin/Authors
Content-Type: application/json

{
  "name": "Isaac Asimov",
  "country": "United States",
  "email": "isaac@example.com"
}

### 5. Create a book in ADMIN (use the author ID from step 4)
POST {{base}}/admin/Books
Content-Type: application/json

{
  "title": "Foundation",
  "genre": "Science Fiction",
  "price": 11.99,
  "stock": 75,
  "rating": 4.6,
  "isbn": "9780553293357",
  "author_ID": "<paste-author-ID-here>"
}

### 6. Update the book's stock
PATCH {{base}}/admin/Books(<paste-book-ID-here>)
Content-Type: application/json

{
  "stock": 120
}

### 7. Delete the book
DELETE {{base}}/admin/Books(<paste-book-ID-here>)

### 8. Verify it's gone
GET {{base}}/catalog/Books?$filter=title eq 'Foundation'
```

---

### What You Should Observe

| Observation | Why |
|-------------|-----|
| POST to `/catalog/` fails (405) | `@readonly` blocks writes |
| POST to `/admin/` succeeds (201) | No restriction on admin |
| `createdAt`/`createdBy` auto-filled | `managed` aspect works! |
| ID auto-generated | `cuid` aspect works! |
| Same data visible in both services | Both project on same DB entities |
| `/catalog/Authors` hides email | Projection only exposes selected fields |
| `/admin/Authors` shows email | Full projection shows everything |

---

## Assessment: MCQ — 15 Questions on CAP Services (16:45 - 17:00)

### Instructions
- 15 questions, all on today's topic
- Choose the best answer
- Time: 10 minutes

---

**Q1.** What is a Service in CAP?

- A) A database table
- B) A controlled API layer that exposes data to clients
- C) A UI component
- D) A npm package

---

**Q2.** What keyword defines a service in CDS?

- A) `define service`
- B) `create service`
- C) `service`
- D) `api`

---

**Q3.** Given `service CatalogService { }`, what URL path is generated?

- A) `/CatalogService`
- B) `/catalogservice`
- C) `/catalog`
- D) `/catalog-service`

---

**Q4.** What does `as projection on` do in a service?

- A) Creates a copy of the database table
- B) Exposes a database entity through the service
- C) Deletes the original entity
- D) Renames the database table

---

**Q5.** What does `@readonly` do on a service entity?

- A) Hides the entity from the API
- B) Allows only GET requests (blocks POST, PUT, DELETE)
- C) Makes the entity invisible in $metadata
- D) Encrypts the data

---

**Q6.** Which OData query option fetches related entities inline?

- A) `$join`
- B) `$include`
- C) `$expand`
- D) `$relate`

---

**Q7.** To get books with price greater than 10, the correct URL is:

- A) `/Books?price>10`
- B) `/Books?$filter=price gt 10`
- C) `/Books?$where=price > 10`
- D) `/Books?filter=price greater 10`

---

**Q8.** What does `$top=5&$skip=10` return?

- A) The first 5 records
- B) Records 6-10
- C) Records 11-15
- D) The last 5 records

---

**Q9.** What does `/catalog/$metadata` return?

- A) JSON data
- B) An XML document describing the API schema
- C) The service source code
- D) Authentication credentials

---

**Q10.** You want to expose Products but hide the costPrice field. Which is correct?

- A) `entity Products as projection on db.Products excluding { costPrice };`
- B) `entity Products as projection on db.Products where costPrice = null;`
- C) `entity Products as projection on db.Products hide costPrice;`
- D) `entity Products as projection on db.Products - costPrice;`

---

**Q11.** TRUE or FALSE: A single entity can be exposed by multiple services.

---

**Q12.** What is the main advantage of CAP services over plain Express.js?

- A) CAP is faster at runtime
- B) CAP auto-generates CRUD, filtering, sorting, pagination, and documentation
- C) Express.js cannot connect to databases
- D) CAP doesn't require Node.js

---

**Q13.** To sort books by price descending, the OData query is:

- A) `$sort=price DESC`
- B) `$orderby=price desc`
- C) `$order=price descending`
- D) `$sort=-price`

---

**Q14.** Which file extension is used for CAP service definitions?

- A) `.js`
- B) `.json`
- C) `.cds`
- D) `.srv`

---

**Q15.** What is the `$count` query option used for?

- A) Counts characters in a field
- B) Returns the total number of matching records
- C) Limits results to a count
- D) Creates a new counter field

---

### Answer Key

<details>
<summary>Click to reveal answers</summary>

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | A service is the API layer controlling data access |
| 2 | **C** | The `service` keyword defines services in CDS |
| 3 | **C** | Removes "Service" suffix and lowercases → `/catalog` |
| 4 | **B** | `as projection on` connects service entity to DB entity |
| 5 | **B** | `@readonly` blocks all write operations (POST/PUT/DELETE) |
| 6 | **C** | `$expand` fetches related data inline |
| 7 | **B** | OData uses `$filter` with operators like `gt`, `lt`, `eq` |
| 8 | **C** | Skip 10, then take 5 = records 11-15 |
| 9 | **B** | `$metadata` returns the XML schema describing the API |
| 10 | **A** | `excluding { field }` hides specific fields |
| 11 | **TRUE** | Same DB entity can appear in CatalogService AND AdminService |
| 12 | **B** | CAP auto-generates all standard operations and query capabilities |
| 13 | **B** | `$orderby=field desc` is the OData V4 syntax |
| 14 | **C** | `.cds` files for all CDS definitions (model and service) |
| 15 | **B** | `$count=true` includes total count in response |

</details>

---

## Assignment: Create Services for EPM Model

### Task

Create three separate services for your EPM data model from Days 12-14. Each service serves a different business function:

---

### Service 1: MasterDataService

**Purpose:** Manage reference/master data (for admin users)

**Expose these entities (full CRUD):**
- Suppliers
- Categories
- Products

**URL path:** `/master-data`

---

### Service 2: SalesService

**Purpose:** Handle sales operations (for sales team)

**Expose:**
- Customers (full CRUD)
- SalesOrders (full CRUD)
- SalesOrderItems (full CRUD)
- Products (`@readonly` — sales team can see but not modify products)

**URL path:** `/sales`

---

### Service 3: AnalyticsService

**Purpose:** Read-only reporting and analytics

**Expose (all `@readonly`):**
- ProductCatalog (view)
- OrderSummary (view)
- LowStockProducts (view — if you created this in Day 15)

**URL path:** `/analytics`

---

### Expected File Structure

```
srv/
├── master-data-service.cds    ← Service 1
├── sales-service.cds          ← Service 2
└── analytics-service.cds      ← Service 3
```

---

### Template to Get Started

```cds
// srv/master-data-service.cds
using { com.epm as db } from '../db/schema';

service MasterDataService @(path: '/master-data') {
  // Your entities here...
}
```

---

### Solution

<details>
<summary>Complete Solution (try yourself first!)</summary>

**srv/master-data-service.cds:**
```cds
using { com.epm as db } from '../db/schema';

service MasterDataService @(path: '/master-data') {
  entity Suppliers  as projection on db.Suppliers;
  entity Categories as projection on db.Categories;
  entity Products   as projection on db.Products;
}
```

**srv/sales-service.cds:**
```cds
using { com.epm as db } from '../db/schema';

service SalesService @(path: '/sales') {
  entity Customers      as projection on db.Customers;
  entity SalesOrders    as projection on db.SalesOrders;
  entity SalesOrderItems as projection on db.SalesOrderItems;
  @readonly entity Products as projection on db.Products;
}
```

**srv/analytics-service.cds:**
```cds
using { com.epm as db } from '../db/schema';

service AnalyticsService @(path: '/analytics') {
  @readonly entity ProductCatalog  as projection on db.ProductCatalog;
  @readonly entity OrderSummary    as projection on db.OrderSummary;
}
```

</details>

---

### Verification Checklist

After creating your services:

- [ ] `cds watch` starts without errors
- [ ] Service index at `http://localhost:4004` shows all 3 services
- [ ] `GET /master-data/Products` returns product list
- [ ] `POST /master-data/Products` can create a new product
- [ ] `GET /sales/Products` returns products (read-only)
- [ ] `POST /sales/Products` returns 405 (blocked by @readonly)
- [ ] `GET /sales/SalesOrders?$expand=items` shows orders with items
- [ ] `GET /analytics/ProductCatalog` returns the view data
- [ ] `GET /master-data/$metadata` shows the XML schema
- [ ] `GET /sales/Customers?$filter=city eq 'New York'` filters correctly

---

## End of Day Summary

### What We Learned

| Concept | Key Takeaway |
|---------|-------------|
| Service | API boundary that controls what data is exposed and how |
| `as projection on` | Connects service entity to database entity |
| `@readonly` | Blocks write operations (POST, PUT, DELETE) |
| URL structure | `ServiceName` → `/service-path/EntityName` |
| `$metadata` | Auto-generated XML describing the entire API |
| OData V4 | Standard protocol with $filter, $orderby, $expand, etc. |
| CAP vs Express | CAP gives you CRUD + queries + docs for free |

### The One-Liner

> **A CAP service is a menu — it decides what the customer can see and order from the kitchen.**

---

### Looking Ahead: Day 17

Tomorrow we dive deeper into services:
- Custom service handlers (JavaScript logic)
- Before/After hooks
- Input validation
- Calculated fields at the service level
- Error handling in services

---

*End of Day 16 — You now know how to expose your data to the world!*
