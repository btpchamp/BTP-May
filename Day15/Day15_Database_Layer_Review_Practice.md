# Day 15: Database Layer Review & Practice Day

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 14 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: CDS Modeling — Complete Recap | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Best Practices & Anti-Patterns | 75 min |
| 12:00 - 13:00 | Session 3: Peer Review Exercise — Review EPM Models | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 15:00 | Session 4: Fix Issues & Extend the Model | 75 min |
| 15:00 - 15:15 | Break | 15 min |
| 15:15 - 16:15 | Session 5: Weekly Quiz — Days 11-15 (30 Questions) | 60 min |
| 16:15 - 17:00 | Session 6: Mini Project — HR Management Data Model | 45 min |

---

## What You'll Achieve Today

By the end of this review day, you will:
- Confidently recall ALL CDS modeling concepts from Days 11-14
- Apply best practices and avoid common mistakes in data modeling
- Review someone else's code and give constructive feedback
- Fix real issues found during peer review
- Extend an existing model with new entities and relationships
- Design a complete HR Management data model from scratch

---

## Day 14 Recap — Quick Fire (09:00 - 09:15)

1. A CDS View creates a new table in the database? (True/False) → _____
2. What keyword creates a read-only perspective on data? → _____
3. `excluding { field1, field2 }` is used in what type of definition? → _____
4. CSV file name must match this pattern: → _____
5. CSV separator in CAP is what character? → _____
6. What command deploys data to SQLite? → _____
7. `case when ... then ... end` in CDS creates what? → _____
8. For association fields in CSV, you use the _____ column name?

<details>
<summary>Answers</summary>

1. **False** — Views don't create new tables, they provide a different perspective on existing data
2. `select from` (or `as select from`) — used to define views
3. `as projection on` (excluding hides specific fields from a projection)
4. `namespace-EntityName.csv` (e.g., `my.bookshop-Books.csv`)
5. Semicolons (`;`) — NOT commas!
6. `cds deploy --to sqlite:db/data.db`
7. A **calculated field** (conditional/computed value)
8. **Foreign key** column name (e.g., `author_ID` not `author`)

</details>

---

## Session 1: CDS Modeling — Complete Recap (09:15 - 10:30)

### The Big Picture: What We've Learned

Over Days 11-14, we built up a complete understanding of the database layer in CAP:

```
Day 11: CAP Framework Intro
   └─► What is CAP? MVC architecture, project structure

Day 12: CDS Basics
   └─► Entities, types, enums, namespaces, basic fields

Day 13: Relationships
   └─► Associations, compositions, aspects, @sap/cds/common

Day 14: Views & Data
   └─► Views, projections, calculated fields, CSV loading, cds deploy
```

Today we tie it ALL together.

---

### Concept Map: How Everything Connects

```
┌─────────────────────────────────────────────────────────────┐
│                    CDS SCHEMA (db/schema.cds)                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  namespace ──► Scope for all entity names                   │
│       │                                                     │
│       ▼                                                     │
│  ┌─────────┐    Association    ┌─────────┐                  │
│  │ Entity A │ ─────────────── │ Entity B │                  │
│  │ (cuid,   │    (relates)    │ (cuid,   │                  │
│  │  managed)│                 │  managed)│                  │
│  └────┬─────┘                 └──────────┘                  │
│       │                                                     │
│       │ Composition                                         │
│       ▼                                                     │
│  ┌──────────┐                                               │
│  │ Entity C  │ ◄── Child (owned by A)                       │
│  │ (items)   │                                              │
│  └──────────┘                                               │
│       │                                                     │
│       ▼                                                     │
│  ┌──────────────┐     ┌─────────────────┐                   │
│  │  View/Select │     │  CSV Data Files │                   │
│  │  (read-only) │     │  (db/data/*.csv)│                   │
│  └──────────────┘     └─────────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Quick Reference: CDS Building Blocks

| Building Block | Purpose | Example |
|----------------|---------|---------|
| `entity` | Define a database table | `entity Products { ... }` |
| `type` | Custom reusable type | `type Amount : Decimal(12,2);` |
| `enum` | Restrict values | `type Status : String enum { New; Done; }` |
| `Association` | Link entities (reference) | `author : Association to Authors;` |
| `Composition` | Own child entities | `items : Composition of many Items on ...` |
| `aspect` | Reusable field group | `aspect addressed { street; city; }` |
| `cuid` | Auto UUID key | `entity X : cuid { ... }` |
| `managed` | Auto timestamps + user | `entity X : managed { ... }` |
| `view` | Read-only data shape | `entity V as select from X { ... }` |
| `projection` | API-level data shape | `entity P as projection on X { ... }` |
| `namespace` | Group entities | `namespace com.training;` |

---

### The Three Relationship Rules

| Question | Answer | Use |
|----------|--------|-----|
| Can child exist without parent? | YES → **Association** | Products ↔ Categories |
| Can child exist without parent? | NO → **Composition** | Order ↔ Order Items |
| Do you need deep insert/delete? | YES → **Composition** | Parent manages children |

**Memory trick:** 
- Association = "I know where you live" (just a reference)
- Composition = "You live in my house" (I own you)

---

### CDS Types — Complete Cheat Sheet

| CDS Type | SQL Equivalent | Use For |
|----------|---------------|---------|
| `String(n)` | VARCHAR(n) | Names, descriptions, codes |
| `Integer` | INTEGER | Counts, quantities, whole numbers |
| `Integer64` | BIGINT | Large numbers |
| `Decimal(p,s)` | DECIMAL(p,s) | Money, precise numbers |
| `Double` | DOUBLE | Scientific, approximate numbers |
| `Boolean` | BOOLEAN | Flags, toggles |
| `Date` | DATE | Calendar dates |
| `Time` | TIME | Time of day |
| `DateTime` | TIMESTAMP | Date + time combined |
| `Timestamp` | TIMESTAMP | High-precision timestamp |
| `UUID` | NVARCHAR(36) | Unique identifiers |
| `LargeString` | NCLOB | Long text, articles |
| `LargeBinary` | BLOB | Files, images |

---

### Recap Exercise: Fill the Gaps

Without looking at notes, complete this entity definition:

```cds
// Fill in the blanks:
namespace com.training;

using { _____, _____, _____ } from '@sap/cds/common';
                 // (hint: 3 things we commonly import)

entity Employees : _____, _____ {
                   // (hint: 2 aspects for UUID key + timestamps)
  employeeName  : ______(100);
  email         : ______(255);
  salary        : _______(10,2);
  isActive      : _______ default true;
  hireDate      : _______;
  department    : _____________ to Departments;
                  // (hint: relationship type)
}

entity Departments : _____, _____ {
  deptName      : String(100);
  employees     : Association to _____ Employees on employees.department = $self;
                  // (hint: one-to-many keyword)
}
```

<details>
<summary>Complete Answer</summary>

```cds
namespace com.training;

using { cuid, managed, Currency } from '@sap/cds/common';

entity Employees : cuid, managed {
  employeeName  : String(100);
  email         : String(255);
  salary        : Decimal(10,2);
  isActive      : Boolean default true;
  hireDate      : Date;
  department    : Association to Departments;
}

entity Departments : cuid, managed {
  deptName      : String(100);
  employees     : Association to many Employees on employees.department = $self;
}
```

</details>

---

## Session 2: Best Practices & Anti-Patterns (10:45 - 12:00)

### Best Practices for CDS Data Modeling

#### Practice 1: Always Use `cuid` and `managed`

```cds
// BAD — manual key and no audit fields
entity Products {
  key ID        : Integer;     // Manual counter? Who manages this?
  name          : String(100);
}

// GOOD — auto UUID key + automatic audit trail
entity Products : cuid, managed {
  name          : String(100);
}
```

**Why?**
- `cuid` gives you a globally unique key (no conflicts, no sequence management)
- `managed` tracks who created/changed what and when (free audit trail!)

---

#### Practice 2: Use Meaningful Field Names

```cds
// BAD — cryptic field names
entity Products : cuid {
  n    : String(100);    // What is 'n'?
  p    : Decimal(10,2);  // What is 'p'?
  s    : Integer;        // What is 's'?
  cat  : String(50);     // category? catalog? catastrophe?
}

// GOOD — clear, descriptive names
entity Products : cuid {
  productName   : String(100);
  price         : Decimal(10,2);
  stock         : Integer;
  category      : Association to Categories;
}
```

**Rule of thumb:** If a new team member can't understand the field without asking, rename it.

---

#### Practice 3: Use Associations, Not Manual Foreign Keys

```cds
// BAD — manual foreign key (you lose CDS magic)
entity Orders : cuid {
  customerID   : UUID;           // Just a raw UUID
  productID    : UUID;           // No relationship info
}

// GOOD — proper associations (CDS manages the FK for you)
entity Orders : cuid {
  customer     : Association to Customers;  // Creates customer_ID automatically
  product      : Association to Products;   // Creates product_ID automatically
}
```

**Why?** 
- Associations give you `$expand` in OData for free
- CDS creates the FK column (e.g., `customer_ID`) automatically
- Referential integrity is maintained
- Code completion in editors works

---

#### Practice 4: Use `@sap/cds/common` for Standard Fields

```cds
// BAD — reinventing the wheel
entity Products : cuid {
  currencyCode : String(3);      // No validation, no standard
  countryCode  : String(2);      // No code list, no description
}

// GOOD — use the built-in types
using { Currency, Country } from '@sap/cds/common';

entity Products : cuid {
  currency     : Currency;       // Links to sap.common.Currencies (with name, symbol)
  country      : Country;        // Links to sap.common.Countries (with name, code)
}
```

**Why?** 
- Standard code lists with descriptions already provided
- Works with SAP Fiori value helps out of the box
- Consistent across all SAP applications

---

#### Practice 5: Choose the Right String Length

```cds
// BAD — everything is String(255)
entity Products : cuid {
  name         : String(255);    // Product names are rarely > 100 chars
  status       : String(255);    // Status is like 10 chars max!
  description  : String(255);    // What if description needs 1000 chars?
}

// GOOD — intentional sizing
entity Products : cuid {
  name         : String(100);    // Reasonable max for product names
  status       : String(20);     // 'New', 'Shipped', 'Cancelled'
  description  : String(1000);   // Long text needs more space
  notes        : LargeString;    // Unlimited text? Use LargeString
}
```

**Guidelines:**
| Field Type | Suggested Length |
|-----------|----------------|
| Names | 100-200 |
| Short codes/status | 10-20 |
| Email | 255 |
| Descriptions | 500-1000 |
| URLs | 500 |
| Phone numbers | 20 |
| Unlimited text | LargeString |

---

#### Practice 6: Use Enums for Fixed Values

```cds
// BAD — any string value allowed (typo-prone!)
entity Orders : cuid {
  status : String(20);  // "new"? "New"? "NEW"? "Nwe"?
}

// GOOD — restricted to valid values only
type OrderStatus : String(20) enum {
  New       = 'New';
  Confirmed = 'Confirmed';
  Shipped   = 'Shipped';
  Delivered = 'Delivered';
  Cancelled = 'Cancelled';
}

entity Orders : cuid {
  status : OrderStatus default 'New';
}
```

---

#### Practice 7: Use Defaults Where Sensible

```cds
// GOOD — logical defaults reduce required input
entity Products : cuid, managed {
  productName : String(100);
  price       : Decimal(10,2);
  stock       : Integer default 0;        // New product starts with 0 stock
  rating      : Decimal(2,1) default 0.0; // No rating yet
  isActive    : Boolean default true;     // Active by default
  status      : String(20) default 'Draft'; // Starts as draft
}
```

---

#### Practice 8: Group Related Entities Using Namespaces

```cds
// BAD — flat list of 50 entities, all jumbled together
entity Employee { }
entity Product { }
entity Invoice { }
entity Skill { }
entity Department { }

// GOOD — logical grouping with namespace
namespace com.training.hr;
// Contains: Employee, Department, Skill, Assignment

// In another file:
namespace com.training.sales;
// Contains: Product, Order, Invoice, Customer
```

---

### Common Anti-Patterns (What NOT To Do)

#### Anti-Pattern 1: The God Entity

```cds
// TERRIBLE — one entity with everything!
entity BusinessData : cuid {
  // Employee stuff
  employeeName  : String(100);
  employeeEmail : String(255);
  department    : String(100);
  salary        : Decimal(10,2);
  // Product stuff
  productName   : String(100);
  productPrice  : Decimal(10,2);
  // Order stuff
  orderNumber   : String(20);
  orderDate     : Date;
  // ... 50 more fields
}
```

**Problem:** 
- Impossible to query efficiently
- Every row has tons of null values
- Can't model relationships
- Changes affect everything

**Fix:** Split into focused entities with proper relationships.

---

#### Anti-Pattern 2: Redundant Data (Denormalization Without Reason)

```cds
// BAD — storing the same data in multiple places
entity Orders : cuid {
  customer      : Association to Customers;
  customerName  : String(100);   // DUPLICATE! Already in Customers table
  customerEmail : String(255);   // DUPLICATE! Already in Customers table
  customerPhone : String(20);    // DUPLICATE! What if customer updates phone?
}
```

**Problem:**
- When customer updates their name, Orders still has the old name
- Data inconsistency nightmare
- Wastes storage

**Fix:** Use the association and `$expand` to get customer data:
```cds
entity Orders : cuid {
  customer : Association to Customers;  // Just the reference
}
// Access via: GET /Orders?$expand=customer → gives you name, email, phone
```

**Exception:** Sometimes denormalization is OK for performance (e.g., in read-only reporting views). But in transactional entities, avoid it.

---

#### Anti-Pattern 3: Missing Relationships

```cds
// BAD — no relationships, just raw IDs
entity OrderItems : cuid {
  orderID    : UUID;      // Which order? No way to navigate!
  productID  : UUID;      // Which product? Can't $expand!
  quantity   : Integer;
}

// GOOD — proper associations
entity OrderItems : cuid {
  order      : Association to Orders;     // Can navigate to order
  product    : Association to Products;   // Can $expand product details
  quantity   : Integer;
}
```

---

#### Anti-Pattern 4: Overusing Composition

```cds
// BAD — Composition where Association should be used
entity Orders : cuid {
  customer : Composition of Customers;  // WRONG! Deleting order deletes customer??
  product  : Composition of Products;   // WRONG! Deleting order deletes product??
}

// GOOD — only use Composition for true parent-child ownership
entity Orders : cuid {
  customer : Association to Customers;   // Customer exists independently
  product  : Association to Products;    // Product exists independently
  items    : Composition of many OrderItems on items.order = $self;
             // Order items ARE owned by this order → Composition correct!
}
```

**Remember:** Composition = "If I die, you die." Only use when the child genuinely can't exist without the parent.

---

#### Anti-Pattern 5: No Namespace

```cds
// BAD — pollutes the global scope
entity Products { }   // Collision risk with SAP standard entities!
entity Orders { }     // Another common name collision

// GOOD — scoped safely
namespace com.mycompany.sales;
entity Products { }   // Full name: com.mycompany.sales.Products (unique!)
entity Orders { }     // Full name: com.mycompany.sales.Orders (unique!)
```

---

### Performance Considerations in Data Modeling

#### Tip 1: Index Your Search Fields

If you frequently search/filter by a field, it helps to know which fields will need indexes:

```cds
entity Products : cuid, managed {
  productName : String(100);   // Users search by name → might need index
  price       : Decimal(10,2); // Users filter by price range → might need index
  status      : String(20);    // Users filter by status → might need index
  internalRef : String(50);    // Only used internally, rarely queried
}
```

In CAP, you can add annotations for database indexes:
```cds
@assert.unique: { name: [productName] }
entity Products : cuid, managed { ... }
```

---

#### Tip 2: Keep Entities Focused (Single Responsibility)

```
Fewer fields per entity = Faster queries

entity with 5 fields   → Database reads 5 columns  → FAST
entity with 50 fields  → Database reads 50 columns → SLOW (for simple queries)
```

Split large entities:
```cds
// Core product data (queried often)
entity Products : cuid, managed {
  productName : String(100);
  price       : Decimal(10,2);
  stock       : Integer;
  category    : Association to Categories;
}

// Extended product details (queried rarely, only on detail page)
entity ProductDetails : cuid {
  product       : Association to Products;
  longDescription : LargeString;
  specifications  : LargeString;
  warranty       : String(500);
  manufacturer   : String(200);
}
```

---

#### Tip 3: Use Views for Complex Reads

Instead of complex OData queries with multiple `$expand` and `$select`, create a view:

```cds
// Without view: Client must build complex query
// GET /Orders?$expand=customer,items($expand=product)&$select=orderNumber,...
// That's a lot of work for every client!

// With view: One simple call
entity OrderDashboard as select from Orders {
  orderNumber,
  orderDate,
  grossAmount,
  status,
  customer.customerName,
  customer.email
};
// GET /OrderDashboard → Clean, flat, fast!
```

---

#### Tip 4: Don't Over-Normalize

While avoiding redundancy is good, TOO many small entities create JOIN overhead:

```cds
// OVER-NORMALIZED (too many tiny entities)
entity Products : cuid {
  name     : Association to ProductNames;     // Really?
  price    : Association to ProductPrices;    // Come on!
  status   : Association to ProductStatuses;  // Too much!
}

// RIGHT BALANCE
entity Products : cuid {
  productName : String(100);           // Simple field is fine
  price       : Decimal(10,2);         // Simple field is fine
  status      : String(20);           // Or use enum type
  category    : Association to Categories;  // THIS deserves its own entity
  supplier    : Association to Suppliers;   // THIS deserves its own entity
}
```

**Rule:** Create a separate entity only when:
- It has its own lifecycle (can be created/updated independently)
- It has multiple fields of its own
- It's referenced by multiple entities
- It needs its own list/detail page in the UI

---

### Best Practices Summary Card

| # | DO | DON'T |
|---|-----|-------|
| 1 | Use `cuid, managed` on every entity | Define manual Integer keys |
| 2 | Use Associations for relationships | Store raw foreign key UUIDs |
| 3 | Use meaningful field names | Use abbreviations (n, p, desc) |
| 4 | Use enums for fixed value lists | Leave status fields as free String |
| 5 | Set sensible defaults | Force users to fill every field |
| 6 | Use `@sap/cds/common` types | Create custom Currency/Country types |
| 7 | Keep entities focused (5-15 fields) | Create God entities (50+ fields) |
| 8 | Use namespace | Pollute global scope |
| 9 | Use Composition only for owned children | Use Composition for all relationships |
| 10 | Create views for complex reads | Make clients build complex queries |

---

## Session 3: Peer Review Exercise (12:00 - 13:00)

### What is a Peer Review?

A peer review is when another developer looks at your code to find issues you might have missed. It's one of the most effective ways to improve code quality.

**In professional SAP development:**
- Every change goes through a code review before merging
- Reviewers check for correctness, naming, relationships, and best practices
- Fresh eyes catch things the author missed

---

### Peer Review Instructions

**Setup:** Pair up with another participant. Exchange your EPM data model files (from Day 13-14 work).

**Your job as a reviewer:** Look at their `schema.cds` and check the following:

---

### Review Checklist

Use this checklist to review your partner's EPM data model:

#### Structure & Basics
- [ ] Is a `namespace` defined?
- [ ] Are `cuid` and `managed` used on all entities?
- [ ] Are proper imports from `@sap/cds/common` present?
- [ ] Is the file well-organized (entities grouped logically)?

#### Naming
- [ ] Are entity names in PascalCase? (e.g., `SalesOrders`)
- [ ] Are field names in camelCase? (e.g., `productName`)
- [ ] Are names meaningful and not abbreviated?
- [ ] Is naming consistent across entities? (e.g., not `name` in one and `empName` in another)

#### Data Types
- [ ] Are String lengths appropriate (not all 255)?
- [ ] Is `Decimal(p,s)` used for money (not Integer or Double)?
- [ ] Are Boolean fields used for true/false flags?
- [ ] Are Date/DateTime types used correctly?

#### Relationships
- [ ] Are Associations used correctly (independent entities)?
- [ ] Are Compositions used only for parent-child ownership?
- [ ] Is the `on` condition correct for back-links?
- [ ] Can you navigate from parent to children AND children to parent?

#### Best Practices
- [ ] Are defaults provided where sensible?
- [ ] Are enums used for fixed value fields (status, type)?
- [ ] No God entities (too many unrelated fields)?
- [ ] No redundant data stored in multiple places?

#### Views (if present)
- [ ] Do views serve a clear purpose?
- [ ] Are calculated fields correct?
- [ ] Are `where` filters appropriate?

#### CSV Data (if present)
- [ ] Are file names correct (namespace-EntityName.csv)?
- [ ] Do column headers match CDS field names exactly?
- [ ] Are semicolons used as separators?
- [ ] Are foreign keys referenced correctly (e.g., `supplier_ID`)?

---

### How to Give Feedback

Use this format for each issue found:

```
[SEVERITY] Location: Description
  Suggestion: How to fix it

Severity Levels:
  🔴 CRITICAL  — Will cause errors or data loss
  🟡 IMPORTANT — Should fix for quality/maintenance
  🟢 NICE TO HAVE — Minor improvement suggestion
```

**Example feedback:**

```
🔴 CRITICAL — entity Orders, line 45:
  `customer : Composition of Customers;`
  Suggestion: Change to Association. Deleting an order should NOT delete the customer!

🟡 IMPORTANT — entity Products, field naming:
  Using `desc` instead of `description`
  Suggestion: Use full word `description` for clarity

🟢 NICE TO HAVE — entity Suppliers:
  No default for `isActive` field
  Suggestion: Add `default true` since most new suppliers are active
```

---

### Sample Model for Review Practice

If you don't have a partner's model, review this intentionally flawed model and find all the issues:

```cds
// This model has 10+ issues. Find them all!

entity products {
  key id        : Integer;
  n             : String(255);
  desc          : String(255);
  p             : Double;
  currCode      : String(3);
  qty           : Integer;
  catName       : String(100);
  supplierName  : String(200);
  supplierEmail : String(255);
  supplierPhone : String(20);
  stat          : String(255);
  items         : Composition of many orderitems on items.prod = $self;
}

entity orderitems {
  key id    : Integer;
  prod      : Association to products;
  orderId   : UUID;
  qty       : Integer;
  price     : Integer;
}
```

<details>
<summary>Issues Found (Try to find them yourself first!)</summary>

| # | Severity | Issue | Fix |
|---|----------|-------|-----|
| 1 | 🔴 | No namespace | Add `namespace com.training.sales;` |
| 2 | 🔴 | No `cuid` or `managed` aspects | Add `: cuid, managed` to both entities |
| 3 | 🔴 | Manual Integer keys | Remove and use `cuid` aspect |
| 4 | 🔴 | `Double` for price | Use `Decimal(10,2)` for money |
| 5 | 🔴 | `Integer` for price in items | Use `Decimal(10,2)` |
| 6 | 🔴 | `items : Composition` on Products | OrderItems belong to Orders, not Products! |
| 7 | 🔴 | Missing `Order` entity | Items reference `orderId` but entity doesn't exist |
| 8 | 🟡 | Entity names not PascalCase | `products` → `Products`, `orderitems` → `OrderItems` |
| 9 | 🟡 | Cryptic field names | `n` → `productName`, `p` → `price`, `qty` → `quantity` |
| 10 | 🟡 | Redundant supplier data | `supplierName/Email/Phone` should be an Association to Suppliers entity |
| 11 | 🟡 | Raw `currCode` String | Use `Currency` from `@sap/cds/common` |
| 12 | 🟡 | `catName` stored directly | Should be Association to Categories entity |
| 13 | 🟡 | `stat` not an enum | Create `ProductStatus` enum type |
| 14 | 🟢 | All Strings are 255 | Use appropriate lengths (100, 20, etc.) |
| 15 | 🟢 | No defaults | Add `default 0` for stock, `default true` for status |
| 16 | 🟢 | No `@sap/cds/common` imports | Import Currency, Country, etc. |

**Corrected version:**

```cds
namespace com.training.sales;

using { cuid, managed, Currency } from '@sap/cds/common';

type ProductStatus : String(20) enum {
  Draft; Active; Discontinued;
}

entity Products : cuid, managed {
  productName   : String(100);
  description   : String(500);
  price         : Decimal(10,2);
  currency      : Currency;
  stock         : Integer default 0;
  status        : ProductStatus default 'Draft';
  category      : Association to Categories;
  supplier      : Association to Suppliers;
}

entity Categories : cuid, managed {
  categoryName  : String(100);
  products      : Association to many Products on products.category = $self;
}

entity Suppliers : cuid, managed {
  supplierName  : String(200);
  email         : String(255);
  phone         : String(20);
  products      : Association to many Products on products.supplier = $self;
}

entity Orders : cuid, managed {
  orderNumber   : String(20);
  orderDate     : Date;
  customer      : Association to Customers;
  items         : Composition of many OrderItems on items.order = $self;
}

entity OrderItems : cuid {
  order         : Association to Orders;
  product       : Association to Products;
  quantity      : Integer;
  unitPrice     : Decimal(10,2);
  currency      : Currency;
}
```

</details>

---

## Session 4: Fix Issues & Extend the Model (13:45 - 15:00)

### Part A: Fix Issues From Peer Review (30 minutes)

Based on the feedback you received from your partner:

1. Open your `db/schema.cds` in BAS
2. Address each issue marked 🔴 CRITICAL first
3. Then fix 🟡 IMPORTANT issues
4. Run `cds watch` to verify no compilation errors
5. Test with REST client to ensure everything still works

---

### Part B: Extend the EPM Model (45 minutes)

Now extend your EPM data model with these additional entities:

#### Task: Add Purchase Order Support

Your EPM model has Sales Orders (selling to customers). Now add **Purchase Orders** (buying from suppliers):

```
Business scenario:
- We buy products FROM suppliers (Purchase Orders)
- We sell products TO customers (Sales Orders)
- When stock is low, we create a Purchase Order to restock
```

**Requirements:**

1. Create a `PurchaseOrders` entity with:
   - Order number, order date
   - Reference to supplier (who we're buying from)
   - Total amount, currency
   - Status (enum: Draft, Submitted, Approved, Received, Cancelled)
   - Link to items (composition)

2. Create a `PurchaseOrderItems` entity with:
   - Reference to the purchase order (parent)
   - Reference to the product (what we're buying)
   - Quantity, unit price
   - Expected delivery date

3. Create a view `LowStockProducts` that shows:
   - Products where `stock < minStock`
   - Include supplier name and contact info
   - Include how many units need to be ordered (`minStock - stock`)

---

#### Expected Solution

Try it yourself first, then check:

<details>
<summary>Solution: Purchase Orders Extension</summary>

```cds
// Add to your existing schema.cds:

// ─── PURCHASE ORDER STATUS ──────────────────────
type PurchaseOrderStatus : String(20) enum {
  Draft     = 'Draft';
  Submitted = 'Submitted';
  Approved  = 'Approved';
  Received  = 'Received';
  Cancelled = 'Cancelled';
}

// ─── PURCHASE ORDERS ────────────────────────────
entity PurchaseOrders : cuid, managed {
  poNumber       : String(20);
  supplier       : Association to Suppliers;
  orderDate      : Date;
  expectedDate   : Date;
  totalAmount    : Decimal(12,2);
  currency       : Currency;
  status         : PurchaseOrderStatus default 'Draft';
  notes          : String(500);
  items          : Composition of many PurchaseOrderItems on items.purchaseOrder = $self;
}

// ─── PURCHASE ORDER ITEMS ───────────────────────
entity PurchaseOrderItems : cuid {
  purchaseOrder   : Association to PurchaseOrders;
  product         : Association to Products;
  quantity        : Integer;
  unitPrice       : Decimal(10,2);
  currency        : Currency;
  expectedDate    : Date;
  receivedQty     : Integer default 0;
}

// ─── VIEW: LOW STOCK PRODUCTS ───────────────────
entity LowStockProducts as select from Products {
  ID,
  productName,
  stock,
  minStock,
  (minStock - stock) as reorderQuantity : Integer,
  supplier.supplierName as supplierName,
  supplier.email        as supplierEmail,
  supplier.phone        as supplierPhone,
  category.categoryName as categoryName
} where stock < minStock and isAvailable = true;
```

</details>

---

#### Bonus Challenge: Add CSV Data for Purchase Orders

Create sample CSV files for your new entities. Remember the naming convention!

<details>
<summary>Hint: CSV file names</summary>

```
db/data/com.epm-PurchaseOrders.csv
db/data/com.epm-PurchaseOrderItems.csv
```

(Replace `com.epm` with YOUR actual namespace)

</details>

<details>
<summary>Solution: Sample CSV Data</summary>

**com.epm-PurchaseOrders.csv:**
```csv
ID;poNumber;supplier_ID;orderDate;expectedDate;totalAmount;currency_code;status;notes
f1a2b3c4-d5e6-7890-abcd-ef1234567890;PO-001;(your-supplier-id-1);2026-01-15;2026-01-30;15000.00;USD;Received;Regular monthly restock
f2b3c4d5-e6f7-8901-bcde-f12345678901;PO-002;(your-supplier-id-2);2026-02-01;2026-02-15;8500.00;EUR;Approved;Urgent restock for low items
f3c4d5e6-f7a8-9012-cdef-123456789012;PO-003;(your-supplier-id-1);2026-02-20;2026-03-05;22000.00;USD;Draft;Quarterly bulk order
```

**com.epm-PurchaseOrderItems.csv:**
```csv
ID;purchaseOrder_ID;product_ID;quantity;unitPrice;currency_code;expectedDate;receivedQty
a1b2c3d4-e5f6-7890-1234-567890abcdef;f1a2b3c4-d5e6-7890-abcd-ef1234567890;(product-id-1);100;50.00;USD;2026-01-30;100
a2b3c4d5-f6a7-8901-2345-67890abcdef1;f1a2b3c4-d5e6-7890-abcd-ef1234567890;(product-id-2);50;120.00;USD;2026-01-30;50
a3c4d5e6-a7b8-9012-3456-7890abcdef12;f2b3c4d5-e6f7-8901-bcde-f12345678901;(product-id-3);200;42.50;EUR;2026-02-15;0
```

</details>

---

## Session 5: Weekly Quiz — Days 11-15 (15:15 - 16:15)

### Instructions
- 30 questions covering everything from Days 11 to 15
- Mix of multiple choice, true/false, and fill-in-the-blank
- Time: 60 minutes
- Passing score: 70% (21/30)

---

### Quiz Questions

**Day 11 — CAP Framework Basics**

**Q1.** What does CAP stand for?
- A) Cloud Access Protocol
- B) Cloud Application Programming Model
- C) Core Application Platform
- D) Cloud API Provider

---

**Q2.** In the CAP MVC architecture, which folder contains the data model definitions?
- A) `srv/`
- B) `app/`
- C) `db/`
- D) `src/`

---

**Q3.** What command creates a new CAP project?
- A) `npm init cap`
- B) `cds init`
- C) `cap create`
- D) `cds new`

---

**Q4.** The command `cds watch` does what? (Select the BEST answer)
- A) Only compiles CDS files
- B) Starts the server with auto-restart on file changes
- C) Deploys to production
- D) Runs unit tests

---

**Q5.** TRUE or FALSE: In CAP, the service layer (srv/) directly accesses the database without going through the model layer.

---

**Day 12 — CDS Basics**

**Q6.** Which CDS type should you use for monetary amounts?
- A) `Integer`
- B) `Double`
- C) `Decimal(precision, scale)`
- D) `String`

---

**Q7.** What does `namespace com.training.epm;` do?

Fill in: It __________ all entities defined in the file to avoid __________.

---

**Q8.** Which of the following is a VALID CDS entity definition?

- A) `entity Products { key ID : UUID; name : String; }`
- B) `table Products { key ID : UUID; name : String; }`
- C) `create entity Products { key ID : UUID; name : String; }`
- D) `define Products { key ID : UUID; name : String; }`

---

**Q9.** What does this code define?
```cds
type Rating : Integer enum { Poor = 1; Average = 2; Good = 3; Excellent = 4; }
```
- A) A table with 4 rows
- B) A custom type restricted to specific integer values
- C) A view with 4 columns
- D) A function that returns 4 values

---

**Q10.** TRUE or FALSE: A CDS `type` creates its own table in the database.

---

**Day 13 — Associations & Compositions**

**Q11.** What is the difference between Association and Composition?
- A) Association is for one-to-one, Composition is for one-to-many
- B) Association means independent lifecycle, Composition means parent owns child
- C) Association is faster, Composition is safer
- D) There is no difference; they are interchangeable

---

**Q12.** Given: `author : Association to Authors;`

What column does CDS create in the database?
- A) `author`
- B) `author_ID`
- C) `authorId`
- D) `authors_id`

---

**Q13.** Fill in the blank to create a one-to-many back-link:
```cds
entity Authors : cuid {
  name  : String(100);
  books : Association to _____ Books on books._____ = $self;
}
```

---

**Q14.** When should you use Composition? (Select ALL that apply)
- A) Order → OrderItems (items belong to the order)
- B) Order → Customer (customer exists independently)
- C) Invoice → InvoiceLines (lines are part of the invoice)
- D) Product → Category (category exists independently)

---

**Q15.** What does the `managed` aspect provide?
- A) `key ID : UUID`
- B) `createdAt, createdBy, modifiedAt, modifiedBy`
- C) `name, description`
- D) `currency, country, language`

---

**Q16.** What does `using { cuid, managed } from '@sap/cds/common';` import?
- A) Database drivers
- B) Reusable aspects for common patterns
- C) Authentication modules
- D) UI components

---

**Q17.** TRUE or FALSE: With Composition, deleting the parent automatically deletes all children.

---

**Day 14 — Views, Projections, CSV Data**

**Q18.** What is a CDS view?
- A) A new database table with copied data
- B) A read-only perspective on existing data (no new table)
- C) A UI component for displaying data
- D) A cached query result

---

**Q19.** What is the correct syntax to create a view?
- A) `create view ProductList as select from Products { ... }`
- B) `entity ProductList as select from Products { ... }`
- C) `view ProductList from Products { ... }`
- D) `define view ProductList as Products { ... }`

---

**Q20.** What does `excluding` do in a projection?
```cds
entity PublicProducts as projection on Products excluding { internalNotes, costPrice };
```
- A) Deletes the excluded fields from the database
- B) Hides the specified fields from the API (they remain in DB)
- C) Marks fields as deprecated
- D) Adds validation to prevent those fields from being used

---

**Q21.** What separator do CAP CSV files use?
- A) Comma (`,`)
- B) Tab
- C) Semicolon (`;`)
- D) Pipe (`|`)

---

**Q22.** Given namespace `com.training` and entity `Products`, what must the CSV file be named?
- A) `products.csv`
- B) `com.training.Products.csv`
- C) `com.training-Products.csv`
- D) `Products-com.training.csv`

---

**Q23.** In a CSV file, how do you reference an association field?
```cds
entity Books : cuid { author : Association to Authors; }
```
- A) Column header: `author`
- B) Column header: `author_ID`
- C) Column header: `authorId`
- D) Column header: `author.ID`

---

**Q24.** What command deploys your schema and data to a SQLite database?
- A) `cds build --to sqlite`
- B) `cds deploy --to sqlite:db/data.db`
- C) `npm run deploy:sqlite`
- D) `sqlite3 deploy schema.cds`

---

**Day 15 — Best Practices & Patterns**

**Q25.** Which of the following is an anti-pattern? (Select ALL)
- A) Using `cuid, managed` on all entities
- B) Creating a single entity with 50+ fields (God Entity)
- C) Storing customer name directly in Orders AND in Customers (redundancy)
- D) Using `Association` for independent entities

---

**Q26.** When should you create a separate entity instead of adding fields to an existing one?
- A) When the data has its own lifecycle and is referenced by multiple entities
- B) Always — every field should be its own entity
- C) Never — fewer entities means better performance
- D) Only when you have more than 100 records

---

**Q27.** Fill in: The `cuid` aspect provides `key ID : _____` automatically.

---

**Q28.** What's wrong with this code?
```cds
entity Orders : cuid {
  customer : Composition of Customers;
}
```
- A) Nothing, it's correct
- B) Should be Association, not Composition (customer exists independently)
- C) Missing the `on` condition
- D) `cuid` can't be used with Composition

---

**Q29.** Which is the BEST practice for status fields?
- A) `status : String(255);`
- B) `status : Integer;` (1=New, 2=Done, etc.)
- C) `status : String(20) enum { New; InProgress; Done; };`
- D) `status : Boolean;` (true=done, false=not done)

---

**Q30.** TRUE or FALSE: Views in CDS create new database tables that store a copy of the data.

---

### Answer Key

<details>
<summary>Click to reveal answers (try the quiz first!)</summary>

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | Cloud Application Programming Model |
| 2 | **C** | `db/` folder contains schema.cds and data |
| 3 | **B** | `cds init` creates a new CAP project |
| 4 | **B** | Starts server with live-reload on changes |
| 5 | **FALSE** | Service layer uses the model layer; CDS runtime handles DB access |
| 6 | **C** | `Decimal(precision, scale)` for exact money representation |
| 7 | scopes/prefixes; name collisions | Namespace groups entities under a prefix |
| 8 | **A** | Standard CDS entity syntax |
| 9 | **B** | An enum creates a restricted custom type |
| 10 | **FALSE** | Types are reusable type definitions, not tables |
| 11 | **B** | Association = independent, Composition = parent owns child |
| 12 | **B** | CDS creates `author_ID` as the foreign key column |
| 13 | `many`, `author` | `Association to many Books on books.author = $self` |
| 14 | **A, C** | Only OrderItems and InvoiceLines are truly owned by parent |
| 15 | **B** | `createdAt`, `createdBy`, `modifiedAt`, `modifiedBy` |
| 16 | **B** | Aspects are reusable field patterns |
| 17 | **TRUE** | Composition cascades delete from parent to children |
| 18 | **B** | Views don't create tables; they reshape existing data |
| 19 | **B** | `entity ViewName as select from ...` |
| 20 | **B** | Hides fields from the API but they remain in the database |
| 21 | **C** | Semicolons (`;`) as separators |
| 22 | **C** | `namespace-EntityName.csv` pattern |
| 23 | **B** | Use FK column name `author_ID` |
| 24 | **B** | `cds deploy --to sqlite:db/data.db` |
| 25 | **B, C** | God Entity and redundant data are anti-patterns |
| 26 | **A** | Separate entity when it has own lifecycle and is shared |
| 27 | **UUID** | `cuid` provides `key ID : UUID` |
| 28 | **B** | Customer exists independently; should be Association |
| 29 | **C** | Enum restricts values and is self-documenting |
| 30 | **FALSE** | Views are read-only perspectives, NOT new tables |

**Scoring:**
- 27-30: Excellent! You've mastered the database layer!
- 21-26: Good! Review the topics where you made mistakes.
- 15-20: Fair. Re-read the relevant day's material and redo the exercises.
- Below 15: Go back to Day 11 and work through the content again with hands-on practice.

</details>

---

## Session 6: Mini Project — HR Management Data Model (16:15 - 17:00)

### Project Brief

**Scenario:** Your company needs an HR Management system. Design and implement a complete data model from scratch.

**Business Requirements:**
- Track employees with their personal details and employment info
- Organize employees into departments
- Manage projects and assign employees to projects
- Track employee skills and proficiency levels
- A department has one head (who is also an employee)
- Employees can work on multiple projects simultaneously
- Each project assignment has start/end dates and an allocation percentage
- Skills have proficiency levels (Beginner, Intermediate, Advanced, Expert)

---

### Entity Relationship Diagram

```
┌──────────────┐         ┌──────────────────┐
│ Departments  │◄────────│   Employees      │
│              │  belongs │                  │
│ deptName     │  to     │ firstName        │
│ description  │         │ lastName         │
│ budget       │         │ email            │
│ head ────────│─────────│ hireDate         │
│              │         │ salary           │
└──────────────┘         │ jobTitle         │
                         │ isActive         │
                         └───────┬──────────┘
                                 │
                    ┌────────────┼────────────┐
                    │            │            │
                    ▼            ▼            │
         ┌──────────────┐  ┌──────────┐     │
         │ Assignments  │  │  Skills  │     │
         │              │  │          │     │
         │ role         │  │ skillName│     │
         │ allocation   │  │ category │     │
         │ startDate    │  └──────────┘     │
         │ endDate      │       │           │
         └──────┬───────┘       ▼           │
                │         ┌─────────────┐   │
                ▼         │EmployeeSkills│   │
         ┌──────────┐    │             │◄──┘
         │ Projects  │    │ proficiency │
         │           │    │ yearsExp    │
         │ name      │    └─────────────┘
         │ startDate │
         │ endDate   │
         │ budget    │
         │ status    │
         └──────────┘
```

---

### Step-by-Step Guide

#### Step 1: Create the Project

```bash
cds init hr-management
cd hr-management
npm install
```

#### Step 2: Design Your Schema

Create `db/schema.cds`. Think about:
- What namespace will you use? (suggestion: `com.hr`)
- What aspects do you need? (`cuid`, `managed`)
- What enums will you need? (status, proficiency level)

#### Step 3: Implement the Entities

Build these entities in order (from least dependent to most):

1. **Departments** — standalone (no dependencies)
2. **Employees** — depends on Departments
3. **Projects** — standalone
4. **Skills** — standalone
5. **Assignments** — connects Employees and Projects (many-to-many bridge)
6. **EmployeeSkills** — connects Employees and Skills (many-to-many bridge)

---

### Requirements Checklist

| Entity | Required Fields | Relationships |
|--------|----------------|---------------|
| **Departments** | deptName, description, budget, location | head → Employee |
| **Employees** | firstName, lastName, email, phone, hireDate, salary, jobTitle, isActive | department → Department |
| **Projects** | projectName, description, startDate, endDate, budget, status | manager → Employee |
| **Skills** | skillName, category, description | — |
| **Assignments** | role, allocation (%), startDate, endDate | employee → Employee, project → Project |
| **EmployeeSkills** | proficiency, yearsOfExperience, certifiedDate | employee → Employee, skill → Skill |

---

### Helpful Enums to Define

```cds
type ProjectStatus : String(20) enum {
  Planning   = 'Planning';
  Active     = 'Active';
  OnHold     = 'On Hold';
  Completed  = 'Completed';
  Cancelled  = 'Cancelled';
}

type Proficiency : String(20) enum {
  Beginner     = 'Beginner';
  Intermediate = 'Intermediate';
  Advanced     = 'Advanced';
  Expert       = 'Expert';
}

type SkillCategory : String(30) enum {
  Technical    = 'Technical';
  Soft         = 'Soft';
  Management   = 'Management';
  Domain       = 'Domain';
}
```

---

### Try It Yourself First!

Spend 20-25 minutes implementing the model. Then check against the reference solution below.

<details>
<summary>Reference Solution: Complete HR Management Data Model</summary>

```cds
namespace com.hr;

using { cuid, managed, Currency, Country } from '@sap/cds/common';

// ═══════════════════════════════════════════════════
// TYPE DEFINITIONS
// ═══════════════════════════════════════════════════

type ProjectStatus : String(20) enum {
  Planning   = 'Planning';
  Active     = 'Active';
  OnHold     = 'On Hold';
  Completed  = 'Completed';
  Cancelled  = 'Cancelled';
}

type Proficiency : String(20) enum {
  Beginner     = 'Beginner';
  Intermediate = 'Intermediate';
  Advanced     = 'Advanced';
  Expert       = 'Expert';
}

type SkillCategory : String(30) enum {
  Technical    = 'Technical';
  Soft         = 'Soft';
  Management   = 'Management';
  Domain       = 'Domain';
}

// ═══════════════════════════════════════════════════
// CORE ENTITIES
// ═══════════════════════════════════════════════════

// ─── DEPARTMENTS ─────────────────────────────────
entity Departments : cuid, managed {
  deptName      : String(100);
  description   : String(500);
  budget        : Decimal(12,2);
  currency      : Currency;
  location      : String(100);
  head          : Association to Employees;
  employees     : Association to many Employees on employees.department = $self;
}

// ─── EMPLOYEES ───────────────────────────────────
entity Employees : cuid, managed {
  firstName     : String(50);
  lastName      : String(50);
  email         : String(255);
  phone         : String(20);
  hireDate      : Date;
  salary        : Decimal(10,2);
  currency      : Currency;
  jobTitle      : String(100);
  isActive      : Boolean default true;
  country       : Country;
  department    : Association to Employees;
  assignments   : Association to many Assignments on assignments.employee = $self;
  skills        : Association to many EmployeeSkills on skills.employee = $self;
}

// ─── PROJECTS ────────────────────────────────────
entity Projects : cuid, managed {
  projectName   : String(150);
  description   : String(1000);
  startDate     : Date;
  endDate       : Date;
  budget        : Decimal(12,2);
  currency      : Currency;
  status        : ProjectStatus default 'Planning';
  manager       : Association to Employees;
  assignments   : Association to many Assignments on assignments.project = $self;
}

// ─── SKILLS ──────────────────────────────────────
entity Skills : cuid, managed {
  skillName     : String(100);
  category      : SkillCategory;
  description   : String(500);
  employeeSkills : Association to many EmployeeSkills on employeeSkills.skill = $self;
}

// ═══════════════════════════════════════════════════
// BRIDGE / JUNCTION ENTITIES (Many-to-Many)
// ═══════════════════════════════════════════════════

// ─── ASSIGNMENTS (Employee ↔ Project) ────────────
entity Assignments : cuid, managed {
  employee      : Association to Employees;
  project       : Association to Projects;
  role          : String(100);
  allocation    : Integer;   // percentage: 25, 50, 75, 100
  startDate     : Date;
  endDate       : Date;
}

// ─── EMPLOYEE SKILLS (Employee ↔ Skill) ──────────
entity EmployeeSkills : cuid, managed {
  employee        : Association to Employees;
  skill           : Association to Skills;
  proficiency     : Proficiency default 'Beginner';
  yearsOfExperience : Decimal(3,1);
  certifiedDate   : Date;
}

// ═══════════════════════════════════════════════════
// VIEWS — Useful Perspectives
// ═══════════════════════════════════════════════════

// View: Team Overview (flat view for managers)
entity TeamOverview as select from Employees {
  ID,
  firstName,
  lastName,
  email,
  jobTitle,
  salary,
  hireDate,
  isActive,
  department.deptName as departmentName
} where isActive = true;

// View: Project Dashboard
entity ProjectDashboard as select from Projects {
  ID,
  projectName,
  startDate,
  endDate,
  budget,
  status,
  manager.firstName as managerFirstName,
  manager.lastName  as managerLastName
} where status != 'Cancelled';

// View: Skills Matrix (who knows what)
entity SkillsMatrix as select from EmployeeSkills {
  employee.firstName as firstName,
  employee.lastName  as lastName,
  employee.jobTitle  as jobTitle,
  skill.skillName    as skillName,
  skill.category     as skillCategory,
  proficiency,
  yearsOfExperience
};
```

</details>

---

### Sample CSV Data for HR Model

<details>
<summary>CSV Files</summary>

**db/data/com.hr-Departments.csv:**
```csv
ID;deptName;description;budget;currency_code;location
d1a00001-0000-0000-0000-000000000001;Engineering;Software Development & IT;500000.00;USD;Building A - Floor 3
d1a00001-0000-0000-0000-000000000002;Human Resources;People Operations;200000.00;USD;Building A - Floor 1
d1a00001-0000-0000-0000-000000000003;Marketing;Brand & Growth;350000.00;USD;Building B - Floor 2
d1a00001-0000-0000-0000-000000000004;Finance;Accounting & Planning;250000.00;USD;Building A - Floor 2
```

**db/data/com.hr-Employees.csv:**
```csv
ID;firstName;lastName;email;phone;hireDate;salary;currency_code;jobTitle;isActive;department_ID
e1a00001-0000-0000-0000-000000000001;Sarah;Johnson;sarah.johnson@company.com;+1-555-0101;2020-03-15;95000.00;USD;Engineering Manager;true;d1a00001-0000-0000-0000-000000000001
e1a00001-0000-0000-0000-000000000002;Mike;Chen;mike.chen@company.com;+1-555-0102;2021-07-01;82000.00;USD;Senior Developer;true;d1a00001-0000-0000-0000-000000000001
e1a00001-0000-0000-0000-000000000003;Lisa;Park;lisa.park@company.com;+1-555-0103;2022-01-10;75000.00;USD;Developer;true;d1a00001-0000-0000-0000-000000000001
e1a00001-0000-0000-0000-000000000004;James;Wilson;james.wilson@company.com;+1-555-0104;2019-11-20;88000.00;USD;HR Director;true;d1a00001-0000-0000-0000-000000000002
e1a00001-0000-0000-0000-000000000005;Anna;Schmidt;anna.schmidt@company.com;+1-555-0105;2023-03-01;70000.00;USD;Marketing Specialist;true;d1a00001-0000-0000-0000-000000000003
```

**db/data/com.hr-Skills.csv:**
```csv
ID;skillName;category;description
s1a00001-0000-0000-0000-000000000001;JavaScript;Technical;Frontend and backend JavaScript development
s1a00001-0000-0000-0000-000000000002;SAP CAP;Technical;Cloud Application Programming Model
s1a00001-0000-0000-0000-000000000003;Project Management;Management;Planning and executing projects
s1a00001-0000-0000-0000-000000000004;Communication;Soft;Written and verbal communication
s1a00001-0000-0000-0000-000000000005;SAP HANA;Technical;In-memory database platform
```

**db/data/com.hr-Projects.csv:**
```csv
ID;projectName;description;startDate;endDate;budget;currency_code;status;manager_ID
p1a00001-0000-0000-0000-000000000001;ERP Migration;Migrate legacy ERP to SAP S/4HANA;2026-01-01;2026-12-31;200000.00;USD;Active;e1a00001-0000-0000-0000-000000000001
p1a00001-0000-0000-0000-000000000002;Mobile App v2;Redesign customer mobile application;2026-03-01;2026-08-31;150000.00;USD;Planning;e1a00001-0000-0000-0000-000000000002
p1a00001-0000-0000-0000-000000000003;Brand Refresh;Update company branding and website;2026-02-15;2026-06-30;80000.00;USD;Active;e1a00001-0000-0000-0000-000000000005
```

**db/data/com.hr-Assignments.csv:**
```csv
ID;employee_ID;project_ID;role;allocation;startDate;endDate
a1a00001-0000-0000-0000-000000000001;e1a00001-0000-0000-0000-000000000001;p1a00001-0000-0000-0000-000000000001;Technical Lead;50;2026-01-01;2026-12-31
a1a00001-0000-0000-0000-000000000002;e1a00001-0000-0000-0000-000000000002;p1a00001-0000-0000-0000-000000000001;Senior Developer;100;2026-01-01;2026-12-31
a1a00001-0000-0000-0000-000000000003;e1a00001-0000-0000-0000-000000000003;p1a00001-0000-0000-0000-000000000002;Developer;75;2026-03-01;2026-08-31
a1a00001-0000-0000-0000-000000000004;e1a00001-0000-0000-0000-000000000001;p1a00001-0000-0000-0000-000000000002;Project Manager;25;2026-03-01;2026-08-31
a1a00001-0000-0000-0000-000000000005;e1a00001-0000-0000-0000-000000000005;p1a00001-0000-0000-0000-000000000003;Lead Designer;100;2026-02-15;2026-06-30
```

**db/data/com.hr-EmployeeSkills.csv:**
```csv
ID;employee_ID;skill_ID;proficiency;yearsOfExperience;certifiedDate
es100001-0000-0000-0000-000000000001;e1a00001-0000-0000-0000-000000000001;s1a00001-0000-0000-0000-000000000001;Expert;8.0;2020-06-15
es100001-0000-0000-0000-000000000002;e1a00001-0000-0000-0000-000000000001;s1a00001-0000-0000-0000-000000000002;Advanced;3.0;2024-01-20
es100001-0000-0000-0000-000000000003;e1a00001-0000-0000-0000-000000000002;s1a00001-0000-0000-0000-000000000001;Advanced;5.0;2021-09-10
es100001-0000-0000-0000-000000000004;e1a00001-0000-0000-0000-000000000002;s1a00001-0000-0000-0000-000000000005;Intermediate;2.0;
es100001-0000-0000-0000-000000000005;e1a00001-0000-0000-0000-000000000003;s1a00001-0000-0000-0000-000000000001;Intermediate;2.5;
es100001-0000-0000-0000-000000000006;e1a00001-0000-0000-0000-000000000004;s1a00001-0000-0000-0000-000000000003;Expert;10.0;2019-05-01
es100001-0000-0000-0000-000000000007;e1a00001-0000-0000-0000-000000000004;s1a00001-0000-0000-0000-000000000004;Expert;12.0;
```

</details>

---

### Validation Checklist for Your Mini Project

Before calling it done, verify:

- [ ] `cds watch` starts without errors
- [ ] All entities appear in the service (check localhost:4004)
- [ ] You can create a Department via REST client
- [ ] You can create an Employee linked to a Department
- [ ] You can create a Project
- [ ] You can assign an Employee to a Project (Assignment)
- [ ] `$expand` works: `/Employees?$expand=department,assignments`
- [ ] CSV data loads correctly (check via GET requests)
- [ ] Views return the expected data

---

## End of Day Summary

### What We Covered Today

| Session | Topic | Key Takeaway |
|---------|-------|--------------|
| 1 | Complete Recap | All CDS concepts from Days 11-14 in one cheat sheet |
| 2 | Best Practices | 10 DOs and 5 anti-patterns to remember |
| 3 | Peer Review | How to review code and give constructive feedback |
| 4 | Fix & Extend | Applied review feedback and added Purchase Orders |
| 5 | Weekly Quiz | 30 questions testing full week's knowledge |
| 6 | Mini Project | Built complete HR model from scratch |

---

### Key Rules to Remember Forever

```
┌─────────────────────────────────────────────────────────┐
│              THE 5 GOLDEN RULES OF CDS                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Always use cuid + managed on entities               │
│  2. Association = reference, Composition = ownership    │
│  3. Use @sap/cds/common for Currency, Country, etc.     │
│  4. Views for read-only reshaping (no new tables)       │
│  5. CSV: semicolons, namespace-Entity.csv, FK columns   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### Looking Ahead: Day 16

Starting next week, we move from the **database layer** to the **service layer**:
- Defining CDS services to expose your entities as APIs
- Understanding how services control what data is accessible
- Writing service definitions with projections
- Testing your first OData API endpoints

You now have a solid foundation in data modeling — the service layer builds directly on top of everything you've learned this week!

---

### Homework / Self-Study

1. **Complete the HR Mini Project** if you didn't finish during class
2. **Add 2 more views** to your HR model:
   - `AvailableEmployees` — employees with total allocation < 100%
   - `ExpiringProjects` — projects ending within the next 30 days
3. **Create a short document** (5 bullet points) explaining Association vs Composition to a colleague who hasn't taken this course
4. **Challenge:** Can you add a `TimeSheets` entity to the HR model for tracking employee hours per project per week?

---

*End of Day 15 — Database Layer Complete!*
