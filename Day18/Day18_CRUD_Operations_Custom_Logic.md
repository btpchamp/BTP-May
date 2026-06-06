# Day 18: CRUD Operations & Custom Logic

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 17 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: CRUD Operations — The Complete Picture | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Custom Handlers — before, on, after | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Perform CRUD with REST Client | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: The `req` Object & Input Validation | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Error Handling & Custom Error Messages | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Implement Validations for EPM | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Perform all CRUD operations (Create, Read, Update, Delete) via HTTP requests
- Understand the difference between PUT and PATCH
- Write custom event handlers in JavaScript (before, on, after)
- Use the `req` object to access request data
- Implement input validation that rejects bad data with clear error messages
- Handle errors gracefully with proper HTTP status codes
- Add business logic that runs automatically on CRUD operations

---

## Day 17 Recap — Quick Fire (09:00 - 09:15)

1. `$filter=price gt 10` is equivalent to SQL _____ ? → _____
2. What separator goes between options inside `$expand()`? → _____
3. `$top=10&$skip=20` returns which records? → _____
4. `$count=true` counts records before or after `$top`? → _____
5. `contains(tolower(title),'harry')` does what? → _____
6. String values in `$filter` need what kind of quotes? → _____
7. To get only 2 fields in the response, use what option? → _____
8. `$orderby=price desc,title` sorts by _____ first, then _____?

<details>
<summary>Answers</summary>

1. `WHERE price > 10`
2. Semicolons (`;`) — NOT ampersands!
3. Records 21-30 (skip first 20, take next 10)
4. **Before** — counts all matching records regardless of pagination
5. Case-insensitive search for "harry" in the title
6. Single quotes (`'value'`)
7. `$select=field1,field2`
8. Price descending first, then title ascending for ties

</details>

---

## Session 1: CRUD Operations — The Complete Picture (09:15 - 10:30)

### What is CRUD?

CRUD is the four basic operations you can do with ANY data:

```
┌─────────────────────────────────────────────────────────────┐
│                     C.R.U.D.                                 │
├──────────┬──────────┬──────────────┬───────────────────────┤
│ CREATE   │ READ     │ UPDATE       │ DELETE                 │
│          │          │              │                        │
│ Add new  │ Get      │ Change       │ Remove                 │
│ records  │ existing │ existing     │ existing               │
│          │ records  │ records      │ records                │
│          │          │              │                        │
│ POST     │ GET      │ PUT / PATCH  │ DELETE                 │
│          │          │              │                        │
│ INSERT   │ SELECT   │ UPDATE       │ DELETE                 │
│ INTO     │ FROM     │ SET          │ FROM                   │
│ (SQL)    │ (SQL)    │ (SQL)        │ (SQL)                  │
└──────────┴──────────┴──────────────┴───────────────────────┘
```

**Real-world analogy — Your Contacts App:**
- **C**reate → Add a new contact
- **R**ead → Look up a contact or browse the list
- **U**pdate → Change someone's phone number
- **D**elete → Remove a contact you no longer need

---

### HTTP Methods Map to CRUD

| CRUD Operation | HTTP Method | URL Pattern | Body? |
|---------------|-------------|-------------|-------|
| **C**reate | `POST` | `/entity` | YES — the new data |
| **R**ead (all) | `GET` | `/entity` | No |
| **R**ead (one) | `GET` | `/entity(key)` | No |
| **U**pdate (full) | `PUT` | `/entity(key)` | YES — all fields |
| **U**pdate (partial) | `PATCH` | `/entity(key)` | YES — only changed fields |
| **D**elete | `DELETE` | `/entity(key)` | No |

---

### CREATE — Adding New Records (POST)

**How it works:**

```
POST /admin/Books
Content-Type: application/json

{
  "title": "The Great Gatsby",
  "genre": "Fiction",
  "price": 10.99,
  "stock": 50,
  "author_ID": "a1000001-0000-0000-0000-000000000003"
}
```

**What happens behind the scenes:**

```
1. Client sends POST with JSON body
2. CAP receives the request
3. CAP validates data types (String length, Decimal precision, etc.)
4. CAP generates UUID for "ID" field (because of cuid aspect)
5. CAP fills createdAt, createdBy (because of managed aspect)
6. CAP inserts the record into the database
7. CAP returns the created record with status 201 (Created)
```

**Response (201 Created):**
```json
{
  "ID": "auto-generated-uuid-here",
  "title": "The Great Gatsby",
  "genre": "Fiction",
  "price": 10.99,
  "stock": 50,
  "author_ID": "a1000001-0000-0000-0000-000000000003",
  "createdAt": "2026-06-02T09:30:00Z",
  "createdBy": "anonymous",
  "modifiedAt": "2026-06-02T09:30:00Z",
  "modifiedBy": "anonymous"
}
```

**Notice:** `ID`, `createdAt`, `createdBy`, `modifiedAt`, `modifiedBy` are filled automatically!

---

### CREATE with Associations — Deep Insert

With Composition, you can create parent AND children in ONE request:

```
POST /sales/Orders
Content-Type: application/json

{
  "orderNumber": "SO-100",
  "customer_ID": "c1000001-...",
  "orderDate": "2026-06-02",
  "status": "New",
  "items": [
    {
      "product_ID": "p1000001-...",
      "quantity": 2,
      "unitPrice": 499.99
    },
    {
      "product_ID": "p1000002-...",
      "quantity": 1,
      "unitPrice": 29.99
    }
  ]
}
```

**Result:** Creates 1 Order + 2 OrderItems — all in a single request! This is called a **Deep Insert** and only works with **Composition** (not Association).

---

### READ — Getting Data (GET)

You already know this from Day 17! Here's the complete picture:

```
// Read ALL records
GET /catalog/Books

// Read ONE record by key
GET /catalog/Books(b1000001-0000-0000-0000-000000000001)

// Read with query options
GET /catalog/Books?$filter=genre eq 'Fantasy'&$expand=author

// Read nested/related data directly
GET /catalog/Books(b1000001-...)/author
// Returns just the author of this specific book!

// Count records
GET /catalog/Books/$count
```

**Deep Read** — accessing nested associations via URL path:
```
// Get the author of a specific book (navigate the association)
GET /catalog/Books(book-uuid)/author

// Get all items of a specific order
GET /sales/Orders(order-uuid)/items

// Get the product of a specific order item
GET /sales/Orders(order-uuid)/items(item-uuid)/product
```

This is called **navigation** — you follow the relationship path in the URL.

---

### UPDATE — Changing Existing Records

There are TWO ways to update: **PUT** (replace everything) and **PATCH** (change only some fields).

---

#### PUT — Full Replacement

PUT replaces the ENTIRE record with what you send. Fields you don't include become null!

```
PUT /admin/Books(b1000001-0000-0000-0000-000000000001)
Content-Type: application/json

{
  "title": "Harry Potter and the Philosopher's Stone",
  "genre": "Fantasy",
  "price": 14.99,
  "stock": 200,
  "rating": 4.9,
  "publishDate": "1997-06-26",
  "isbn": "9780747532699",
  "author_ID": "a1000001-0000-0000-0000-000000000001"
}
```

**PUT danger:**
```
// If you PUT with only 2 fields:
PUT /admin/Books(id)
{ "title": "New Title", "price": 9.99 }

// ALL other fields become null!
// genre → null, stock → null, rating → null, isbn → null
// 😱 You just lost data!
```

---

#### PATCH — Partial Update (Recommended!)

PATCH only changes the fields you send. Everything else stays the same.

```
PATCH /admin/Books(b1000001-0000-0000-0000-000000000001)
Content-Type: application/json

{
  "price": 14.99,
  "stock": 200
}
```

**Only `price` and `stock` change.** Title, genre, rating, author — all stay untouched!

---

#### PUT vs PATCH — Visual Comparison

```
Original record:
{ "title": "Harry Potter", "price": 12.99, "stock": 150, "genre": "Fantasy" }

───────────────────────────────────────────────────────────────────

PUT { "price": 14.99 }
Result: { "title": null, "price": 14.99, "stock": null, "genre": null }
                 ↑ GONE!                        ↑ GONE!       ↑ GONE!

PATCH { "price": 14.99 }
Result: { "title": "Harry Potter", "price": 14.99, "stock": 150, "genre": "Fantasy" }
                  ↑ untouched!                         ↑ safe!      ↑ safe!
```

**Rule of thumb:** Always use **PATCH** for updates unless you intentionally want to replace everything.

---

#### Update Response

Both PUT and PATCH return the FULL updated record:

```json
// Response (200 OK):
{
  "ID": "b1000001-...",
  "title": "Harry Potter and the Philosopher's Stone",
  "price": 14.99,
  "stock": 200,
  "modifiedAt": "2026-06-02T10:15:00Z",
  "modifiedBy": "anonymous"
}
```

**Notice:** `modifiedAt` and `modifiedBy` auto-update (thanks to `managed` aspect)!

---

### DELETE — Removing Records

```
DELETE /admin/Books(b1000001-0000-0000-0000-000000000001)
```

**Response:** `204 No Content` (success, nothing to return)

**If record doesn't exist:** `404 Not Found`

---

#### DELETE with Composition — Cascade Delete

When you delete a parent with Composition, children are automatically deleted:

```
DELETE /sales/Orders(order-uuid)

// This ALSO deletes:
// - All OrderItems that belong to this order (cascade!)
// Because items : Composition of many OrderItems...
```

**With Association — NO cascade:**
```
DELETE /admin/Authors(author-uuid)

// This does NOT delete the author's books!
// Books have their own lifecycle (independent via Association)
// But... it might fail if books still reference this author (referential integrity)
```

---

### CRUD Status Codes Summary

| Operation | Success Code | Meaning |
|-----------|-------------|---------|
| POST (Create) | `201 Created` | Record successfully created |
| GET (Read) | `200 OK` | Data returned successfully |
| PUT/PATCH (Update) | `200 OK` | Record updated, returns new state |
| DELETE | `204 No Content` | Record deleted, nothing to return |

| Error Code | Meaning | When |
|-----------|---------|------|
| `400 Bad Request` | Invalid data sent | Wrong types, missing required fields |
| `404 Not Found` | Record doesn't exist | Wrong ID in URL |
| `405 Method Not Allowed` | Operation blocked | `@readonly` entity, POST attempt |
| `409 Conflict` | Data conflict | Duplicate unique values |
| `500 Server Error` | Something broke | Bug in custom handler |

---

### CRUD in CAP — What You Get for Free

Here's the amazing thing: **ALL of this works WITHOUT writing any JavaScript!**

Just by defining your service in CDS:
```cds
service AdminService {
  entity Books as projection on db.Books;
}
```

You automatically get:
- POST /admin/Books (create)
- GET /admin/Books (read all)
- GET /admin/Books(id) (read one)
- PUT /admin/Books(id) (full update)
- PATCH /admin/Books(id) (partial update)
- DELETE /admin/Books(id) (delete)
- All OData query options
- UUID generation
- Timestamp auto-fill
- Type validation
- Deep insert/delete for compositions

**But what if you need CUSTOM logic?** That's where handlers come in...

---

## Session 2: Custom Handlers — before, on, after (10:45 - 12:00)

### Why Do We Need Custom Handlers?

CAP gives you standard CRUD for free. But real applications need more:

| Need | Example | Standard CRUD? |
|------|---------|----------------|
| Validate input | "Price must be > 0" | ❌ Need custom logic |
| Calculate fields | "Total = quantity × price" | ❌ Need custom logic |
| Check permissions | "Only managers can delete" | ❌ Need custom logic |
| Call external APIs | "Verify address via Google Maps" | ❌ Need custom logic |
| Send notifications | "Email customer on order confirm" | ❌ Need custom logic |
| Transform output | "Add computed fullName field" | ❌ Need custom logic |
| Business rules | "Can't order if stock is 0" | ❌ Need custom logic |

**Custom handlers** let you inject YOUR logic into the CRUD lifecycle.

---

### The Request Lifecycle — before, on, after

When a request comes in, it flows through THREE phases:

```
Client Request
      │
      ▼
┌─────────────────┐
│   BEFORE        │  ← Validate, reject, modify input
│   handlers      │     Runs BEFORE the operation
│                 │     Can REJECT the request (throw error)
│                 │     Can MODIFY the incoming data
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   ON            │  ← The actual operation
│   handler       │     DEFAULT: CAP's built-in CRUD
│                 │     You can REPLACE it entirely
│                 │     Or skip it for custom implementations
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   AFTER         │  ← Transform output, side effects
│   handlers      │     Runs AFTER the operation succeeded
│                 │     Can MODIFY the response
│                 │     Can trigger side effects (email, log)
└────────┬────────┘
         │
         ▼
   Response to Client
```

---

### The Pizza Order Analogy

Think of it like ordering a pizza:

```
BEFORE (Validation):
  "Is this a valid order?"
  - Do you have a delivery address? → if not, REJECT
  - Is the pizza size valid (S/M/L)? → if not, REJECT
  - Modify: add default crust type if not specified

ON (Execution):
  "Actually make the pizza"
  - The kitchen prepares your order
  - This is the REAL work (database insert)

AFTER (Post-processing):
  "What happens after the pizza is ready?"
  - Calculate delivery time estimate
  - Send SMS confirmation to customer
  - Add "loyalty points earned" to response
```

---

### Where Do Custom Handlers Live?

Handlers are JavaScript files in the `srv/` folder that MATCH the name of your service CDS file:

```
srv/
├── cat-service.cds        ← Service definition (CDS)
├── cat-service.js         ← Custom handlers (JavaScript) ← SAME NAME!
├── admin-service.cds
└── admin-service.js       ← Handlers for AdminService
```

**Naming rule:** The `.js` file MUST have the same name as the `.cds` file (minus extension).

- `srv/cat-service.cds` → `srv/cat-service.js`
- `srv/sales-service.cds` → `srv/sales-service.js`
- `srv/admin-service.cds` → `srv/admin-service.js`

CAP automatically discovers and loads the matching `.js` file.

---

### Basic Handler Structure

```javascript
// srv/cat-service.js

module.exports = function () {

  // BEFORE handler — runs before the operation
  this.before('CREATE', 'Books', (req) => {
    // Your validation logic here
  });

  // AFTER handler — runs after the operation
  this.after('READ', 'Books', (results, req) => {
    // Your transformation logic here
  });

  // ON handler — replaces the default operation
  this.on('READ', 'Books', (req) => {
    // Your custom implementation here
  });

};
```

**Key pattern:**
```javascript
this.<phase>('<EVENT>', '<Entity>', handlerFunction)
```

Where:
- `<phase>` = `before`, `on`, or `after`
- `<EVENT>` = `CREATE`, `READ`, `UPDATE`, `DELETE` (or `NEW`, `EDIT`, etc.)
- `<Entity>` = The entity name from your service (e.g., 'Books', 'Orders')

---

### BEFORE Handlers — Validate & Modify Input

`before` handlers run BEFORE the database operation. Use them to:
- ✅ Validate input data
- ✅ Reject bad requests
- ✅ Add/modify fields before saving
- ✅ Check business rules

```javascript
// srv/admin-service.js
module.exports = function () {

  // Validate before creating a book
  this.before('CREATE', 'Books', (req) => {
    const { title, price, stock } = req.data;

    // Rule 1: Title is required
    if (!title) {
      req.error(400, 'Title is required!');
    }

    // Rule 2: Price must be positive
    if (price <= 0) {
      req.error(400, 'Price must be greater than zero');
    }

    // Rule 3: Stock can't be negative
    if (stock < 0) {
      req.error(400, 'Stock cannot be negative');
    }
  });

  // Validate before updating
  this.before('UPDATE', 'Books', (req) => {
    if (req.data.price !== undefined && req.data.price <= 0) {
      req.error(400, 'Price must be greater than zero');
    }
  });

};
```

**What happens when `req.error()` is called:**
- The request is REJECTED
- No database operation occurs
- Client receives an error response with your message
- Status code 400 (or whatever you specify)

---

### Modifying Data in BEFORE Handler

You can change the incoming data before it reaches the database:

```javascript
this.before('CREATE', 'Books', (req) => {
  // Auto-capitalize the title
  if (req.data.title) {
    req.data.title = req.data.title.trim();
  }

  // Set default genre if not provided
  if (!req.data.genre) {
    req.data.genre = 'Uncategorized';
  }

  // Set creation timestamp (alternative to managed aspect)
  req.data.processedAt = new Date().toISOString();
});
```

---

### AFTER Handlers — Transform Output & Side Effects

`after` handlers run AFTER the database operation succeeds. Use them to:
- ✅ Add computed fields to the response
- ✅ Transform data format
- ✅ Trigger side effects (logging, notifications)
- ✅ Enrich response with additional info

```javascript
module.exports = function () {

  // Add a computed field after reading books
  this.after('READ', 'Books', (results, req) => {
    // 'results' can be an array (READ all) or a single object (READ one)
    const books = Array.isArray(results) ? results : [results];

    for (const book of books) {
      // Add discount price (20% off)
      if (book.price) {
        book.discountPrice = (book.price * 0.8).toFixed(2);
      }

      // Add availability status
      if (book.stock > 20) {
        book.availability = 'In Stock';
      } else if (book.stock > 0) {
        book.availability = 'Low Stock';
      } else {
        book.availability = 'Out of Stock';
      }
    }
  });

  // Log after delete (side effect)
  this.after('DELETE', 'Books', (_, req) => {
    console.log(`Book deleted by ${req.user.id} at ${new Date().toISOString()}`);
  });

};
```

**Important:** In `after` handlers, the first parameter is `results` (the data), the second is `req`.

---

### ON Handlers — Replace the Default Behavior

`on` handlers REPLACE the default CAP behavior entirely. Use with caution!

```javascript
module.exports = function () {

  // Custom READ that adds extra logic
  this.on('READ', 'Books', async (req, next) => {
    // Call the default handler first
    const results = await next();

    // Then modify the results
    if (results) {
      const books = Array.isArray(results) ? results : [results];
      books.forEach(book => {
        book.readAt = new Date().toISOString();
      });
    }

    return results;
  });

};
```

**The `next()` function:**
- Calls the DEFAULT handler (CAP's built-in CRUD)
- Returns the results
- You can modify and return them
- If you DON'T call `next()`, the default behavior is completely skipped!

---

### ON Handler — Completely Custom Implementation

```javascript
// Custom action: Calculate order total
this.on('calcTotal', 'Orders', async (req) => {
  const { ID } = req.params[0];

  // Custom query
  const order = await SELECT.one.from('SalesOrders')
    .where({ ID })
    .columns('ID', 'grossAmount');

  const items = await SELECT.from('SalesOrderItems')
    .where({ order_ID: ID });

  // Custom calculation
  const total = items.reduce((sum, item) => {
    return sum + (item.quantity * item.unitPrice);
  }, 0);

  return { orderID: ID, calculatedTotal: total };
});
```

---

### When to Use Which Handler

| Scenario | Use | Why |
|----------|-----|-----|
| Validate input before saving | `before` CREATE/UPDATE | Reject bad data early |
| Set default values | `before` CREATE | Add missing fields |
| Calculate fields before save | `before` CREATE/UPDATE | Compute before DB write |
| Add computed fields to response | `after` READ | Enrich output without storing |
| Send notifications | `after` CREATE/UPDATE | After success confirmed |
| Log operations | `after` any | Record what happened |
| Completely custom logic | `on` | Replace standard behavior |
| Check permissions | `before` any | Block unauthorized actions |

---

### Multiple Handlers for Same Event

You can register multiple handlers for the same event — they run in order:

```javascript
module.exports = function () {

  // First validation
  this.before('CREATE', 'Books', (req) => {
    if (!req.data.title) req.error(400, 'Title is required');
  });

  // Second validation (runs after first, if first didn't reject)
  this.before('CREATE', 'Books', (req) => {
    if (req.data.price < 0) req.error(400, 'Price must be positive');
  });

  // Third: set defaults (runs if both validations pass)
  this.before('CREATE', 'Books', (req) => {
    if (!req.data.stock) req.data.stock = 0;
  });

};
```

---

### Handler Events — Complete List

| Event | HTTP Method | When It Fires |
|-------|-------------|---------------|
| `CREATE` | POST | When creating a new record |
| `READ` | GET | When reading records |
| `UPDATE` | PUT / PATCH | When updating a record |
| `DELETE` | DELETE | When deleting a record |
| `NEW` | — | Before showing a "create" form (Fiori) |
| `EDIT` | — | Before showing an "edit" form (Fiori) |
| `SAVE` | — | When draft is saved (Fiori draft) |
| `CANCEL` | — | When edit is cancelled |

For today, focus on the big 4: `CREATE`, `READ`, `UPDATE`, `DELETE`.

---

### Wildcard Handlers — Handle ALL Entities

```javascript
// Runs before CREATE on ANY entity in this service
this.before('CREATE', '*', (req) => {
  console.log(`Creating a new ${req.target.name}`);
});

// Runs before ANY operation on Books
this.before('*', 'Books', (req) => {
  console.log(`Operation on Books: ${req.event}`);
});
```

---

## Session 3: Hands-on — Perform CRUD with REST Client (12:00 - 13:00)

### Setup

Make sure your project is running:
```bash
cds watch
```

Create a test file: `tests/crud-tests.http`

---

### Exercise 1: CREATE Operations (15 minutes)

```http
### ═══════════════════════════════════════════════
### CREATE OPERATIONS (POST)
### ═══════════════════════════════════════════════

@base = http://localhost:4004/admin

### 1.1 Create an Author
POST {{base}}/Authors
Content-Type: application/json

{
  "name": "Dan Brown",
  "country": "United States",
  "email": "dan.brown@example.com"
}

### 1.2 Create a Book (use the author ID from 1.1 response)
POST {{base}}/Books
Content-Type: application/json

{
  "title": "The Da Vinci Code",
  "genre": "Thriller",
  "price": 14.99,
  "stock": 85,
  "rating": 4.1,
  "isbn": "9780307474278",
  "author_ID": "PASTE-AUTHOR-ID-HERE"
}

### 1.3 Create another Book (same author)
POST {{base}}/Books
Content-Type: application/json

{
  "title": "Angels & Demons",
  "genre": "Thriller",
  "price": 12.99,
  "stock": 60,
  "rating": 4.0,
  "isbn": "9780743493468",
  "author_ID": "PASTE-AUTHOR-ID-HERE"
}

### 1.4 Create with missing required field (should it fail?)
POST {{base}}/Books
Content-Type: application/json

{
  "price": 9.99
}

### 1.5 Create with wrong data type (string for price)
POST {{base}}/Books
Content-Type: application/json

{
  "title": "Bad Data Book",
  "price": "not-a-number"
}
```

**Observe:**
- Response `201 Created` for successful creates
- The `ID` field is auto-generated
- `createdAt` and `createdBy` are auto-filled
- What happens with missing/wrong data?

---

### Exercise 2: READ Operations (10 minutes)

```http
### ═══════════════════════════════════════════════
### READ OPERATIONS (GET)
### ═══════════════════════════════════════════════

### 2.1 Read all books
GET {{base}}/Books

### 2.2 Read single book (paste a valid ID)
GET {{base}}/Books(PASTE-BOOK-ID-HERE)

### 2.3 Read a book that doesn't exist
GET {{base}}/Books(00000000-0000-0000-0000-000000000000)

### 2.4 Deep read: Get the author of a specific book
GET {{base}}/Books(PASTE-BOOK-ID-HERE)/author

### 2.5 Deep read: Get all books of an author
GET {{base}}/Authors(PASTE-AUTHOR-ID-HERE)/books

### 2.6 Read with expand
GET {{base}}/Books?$expand=author&$select=title,price
```

**Observe:**
- `200 OK` for successful reads
- `404 Not Found` for non-existent records
- Navigation paths work: `/Books(id)/author`

---

### Exercise 3: UPDATE Operations (15 minutes)

```http
### ═══════════════════════════════════════════════
### UPDATE OPERATIONS (PATCH and PUT)
### ═══════════════════════════════════════════════

### 3.1 PATCH: Update only the price (other fields unchanged)
PATCH {{base}}/Books(PASTE-BOOK-ID-HERE)
Content-Type: application/json

{
  "price": 16.99
}

### 3.2 PATCH: Update stock and rating
PATCH {{base}}/Books(PASTE-BOOK-ID-HERE)
Content-Type: application/json

{
  "stock": 120,
  "rating": 4.3
}

### 3.3 Verify the update (read the book back)
GET {{base}}/Books(PASTE-BOOK-ID-HERE)

### 3.4 PUT: Full replacement (careful! missing fields become null)
PUT {{base}}/Books(PASTE-BOOK-ID-HERE)
Content-Type: application/json

{
  "title": "The Da Vinci Code - Special Edition",
  "genre": "Thriller",
  "price": 19.99,
  "stock": 200,
  "rating": 4.5,
  "isbn": "9780307474278",
  "author_ID": "PASTE-AUTHOR-ID-HERE"
}

### 3.5 Verify after PUT (check all fields)
GET {{base}}/Books(PASTE-BOOK-ID-HERE)

### 3.6 Update a non-existent record
PATCH {{base}}/Books(00000000-0000-0000-0000-000000000000)
Content-Type: application/json

{
  "price": 99.99
}
```

**Observe:**
- After PATCH: only specified fields change, `modifiedAt` updates
- After PUT: ALL fields are what you sent
- Updating non-existent record returns 404

---

### Exercise 4: DELETE Operations (10 minutes)

```http
### ═══════════════════════════════════════════════
### DELETE OPERATIONS
### ═══════════════════════════════════════════════

### 4.1 Delete a book
DELETE {{base}}/Books(PASTE-BOOK-ID-HERE)

### 4.2 Verify it's gone
GET {{base}}/Books(PASTE-BOOK-ID-HERE)

### 4.3 Try to delete again (should return 404)
DELETE {{base}}/Books(PASTE-BOOK-ID-HERE)

### 4.4 Try to delete from read-only service
DELETE http://localhost:4004/catalog/Books(PASTE-BOOK-ID-HERE)
```

**Observe:**
- Successful delete: `204 No Content`
- Delete non-existent: `404 Not Found`
- Delete from @readonly: `405 Method Not Allowed`

---

### Exercise 5: CRUD on @readonly Service (10 minutes)

Verify that `@readonly` blocks write operations:

```http
### ═══════════════════════════════════════════════
### @readonly ENFORCEMENT
### ═══════════════════════════════════════════════

@catalog = http://localhost:4004/catalog

### 5.1 READ works on catalog (should succeed)
GET {{catalog}}/Books

### 5.2 CREATE blocked on catalog
POST {{catalog}}/Books
Content-Type: application/json

{
  "title": "Test",
  "price": 1.00
}

### 5.3 UPDATE blocked on catalog
PATCH {{catalog}}/Books(PASTE-ID)
Content-Type: application/json

{
  "price": 999.99
}

### 5.4 DELETE blocked on catalog
DELETE {{catalog}}/Books(PASTE-ID)
```

**All write operations should return `405 Method Not Allowed`!**

---

## Session 4: The `req` Object & Input Validation (13:45 - 14:45)

### What is the `req` Object?

Every handler receives a `req` (request) object that contains everything about the incoming request:

```javascript
this.before('CREATE', 'Books', (req) => {
  // req is your window into the request!
  console.log(req.data);       // The body/payload
  console.log(req.params);     // URL parameters (for READ/UPDATE/DELETE)
  console.log(req.query);      // Query options ($filter, $select, etc.)
  console.log(req.user);       // Who made the request
  console.log(req.event);      // 'CREATE', 'READ', 'UPDATE', 'DELETE'
  console.log(req.target);     // The entity being targeted
  console.log(req.headers);    // HTTP headers
});
```

---

### `req` Object — Complete Reference

```
┌───────────────────────────────────────────────────────────┐
│                    req Object                               │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  req.data                                                 │
│  ├── The request body (for CREATE/UPDATE)                 │
│  └── Contains the fields being created/updated            │
│                                                           │
│  req.params                                               │
│  ├── Array of key-value objects                           │
│  └── e.g., [{ ID: 'uuid-here' }] for /Books(uuid)       │
│                                                           │
│  req.query                                                │
│  ├── The parsed OData query                               │
│  └── Includes SELECT, WHERE, ORDER BY, LIMIT info        │
│                                                           │
│  req.user                                                 │
│  ├── .id — User ID (e.g., 'admin@company.com')           │
│  ├── .is('admin') — Check role                           │
│  └── .attr — User attributes                             │
│                                                           │
│  req.event                                                │
│  └── 'CREATE' | 'READ' | 'UPDATE' | 'DELETE'             │
│                                                           │
│  req.target                                               │
│  ├── .name — Entity name (e.g., 'Books')                 │
│  └── .elements — Field definitions                       │
│                                                           │
│  req.error(code, message)    — Add error to request       │
│  req.reject(code, message)   — Immediately reject         │
│  req.info(code, message)     — Add info message           │
│  req.warn(code, message)     — Add warning message        │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

---

### Accessing Request Data — Examples

```javascript
// CREATE: req.data contains the full body
this.before('CREATE', 'Books', (req) => {
  console.log(req.data);
  // { title: "New Book", price: 12.99, stock: 50, ... }

  console.log(req.data.title);  // "New Book"
  console.log(req.data.price);  // 12.99
});

// UPDATE: req.data contains only the changed fields
this.before('UPDATE', 'Books', (req) => {
  console.log(req.data);
  // PATCH: { price: 14.99 }  (only what client sent)
  // PUT: { title: "...", price: 14.99, ... }  (all fields)
});

// DELETE: req.data contains the key
this.before('DELETE', 'Books', (req) => {
  console.log(req.params);  // [{ ID: 'uuid-of-book' }]
});

// READ: req.query contains the OData query
this.before('READ', 'Books', (req) => {
  console.log(req.query);  // Parsed query info
});
```

---

### `req.error()` vs `req.reject()` — How to Report Errors

**`req.error(code, message)`** — Collects errors (can have multiple):
```javascript
this.before('CREATE', 'Books', (req) => {
  if (!req.data.title) req.error(400, 'Title is required');
  if (req.data.price <= 0) req.error(400, 'Price must be positive');
  if (req.data.stock < 0) req.error(400, 'Stock cannot be negative');
  // All errors are collected and returned together!
});
```

Response when multiple errors:
```json
{
  "error": {
    "code": "400",
    "message": "Title is required",
    "details": [
      { "code": "400", "message": "Title is required" },
      { "code": "400", "message": "Price must be positive" }
    ]
  }
}
```

**`req.reject(code, message)`** — Immediately stops execution:
```javascript
this.before('CREATE', 'Orders', (req) => {
  if (!req.data.customer_ID) {
    req.reject(400, 'Customer is required for orders');
    // Nothing after this line runs!
  }
  // This code only runs if customer_ID exists
  console.log('Proceeding with order...');
});
```

**When to use which:**

| Use `req.error()` | Use `req.reject()` |
|-------------------|---------------------|
| Multiple validations (show all errors) | Critical check (no point continuing) |
| Form validation (show all issues) | Permission check (unauthorized) |
| Data quality checks | Fatal condition |

---

### Building a Complete Validation Handler

Let's build a real-world validation for a Book entity:

```javascript
// srv/admin-service.js
const cds = require('@sap/cds');

//module.exports = function () {. Try to use function
module.exports = class AdminService extends cds.ApplicationService { init(){
  // ══════════════════════════════════════════
  // VALIDATION: Before Creating a Book
  // ══════════════════════════════════════════
  this.before('CREATE', 'Books', async (req) => {

const cds = require('@sap/cds');

module.exports = class AdminService extends cds.ApplicationService { init(){

  // ══════════════════════════════════════════
  // VALIDATION: Before Creating a Book
  // ══════════════════════════════════════════

    this.before('CREATE', 'Books', async (req) => {
        const { title } = req.data;
        console.log('Creating a book with title:', req.data);
        console.log('Lenth of my Title', req.data.title.length);
        const data = await cds.run(SELECT.from('Books').where({ ID: req.data.ID }));
        console.log('Data from DB', data);
        console.log('Length of record', data.length);
        console.log('DB Array to JSON', data[0]);
        console.log('Price', data[0].price);
        // if (!title) {
        // req.error(400, 'This book is not allowed to be created.');
        // }
    });



    const { title, price, stock, isbn, genre, author_ID } = req.data;

    // --- Required field checks ---
    if (!title || title.trim().length === 0) {
      req.error(400, 'Title is required and cannot be empty', 'title');
    }

    if (price === undefined || price === null) {
      req.error(400, 'Price is required', 'price');
    }

    // --- Value range checks ---
    if (price !== undefined && price <= 0) {
      req.error(400, 'Price must be greater than zero', 'price');
    }

    if (price !== undefined && price > 9999.99) {
      req.error(400, 'Price cannot exceed 9999.99', 'price');
    }

    if (stock !== undefined && stock < 0) {
      req.error(400, 'Stock cannot be negative', 'stock');
    }

    // --- Format checks ---
    if (isbn && isbn.length !== 13) {
      req.error(400, 'ISBN must be exactly 13 characters', 'isbn');
    }

    // --- Business rule: Valid genres ---
    const validGenres = ['Fantasy', 'Fiction', 'Mystery', 'Thriller',
                         'Science Fiction', 'Romance', 'Non-Fiction',
                         'Biography', 'Dystopian', 'Literary Fiction'];
    if (genre && !validGenres.includes(genre)) {
      req.error(400, `Invalid genre. Must be one of: ${validGenres.join(', ')}`, 'genre');
    }

    // --- Referential check: Does the author exist? ---
    if (author_ID) {
      const author = await SELECT.one.from('my.bookshop.Authors').where({ ID: author_ID });
      if (!author) {
        req.error(400, 'Author not found. Please provide a valid author_ID', 'author_ID');
      }
    }
  });

  // ══════════════════════════════════════════
  // VALIDATION: Before Updating a Book
  // ══════════════════════════════════════════
  this.before('UPDATE', 'Books', (req) => {
    const { price, stock, isbn } = req.data;

    if (price !== undefined && price <= 0) {
      req.error(400, 'Price must be greater than zero', 'price');
    }

    if (stock !== undefined && stock < 0) {
      req.error(400, 'Stock cannot be negative', 'stock');
    }

    if (isbn !== undefined && isbn.length !== 13) {
      req.error(400, 'ISBN must be exactly 13 characters', 'isbn');
    }
  });

};
```

---

### The Third Parameter in `req.error()` — Field Targeting

```javascript
req.error(statusCode, message, target);
//                              ↑ Which field caused the error
```

The `target` parameter tells the client WHICH field has the problem. This helps UIs highlight the right input field in red:

```javascript
req.error(400, 'Price must be positive', 'price');
// → UI can highlight the "price" input field
```

Response:
```json
{
  "error": {
    "code": "400",
    "message": "Price must be positive",
    "target": "price"
  }
}
```

---

### Async Handlers — Database Lookups in Validation

Sometimes validation requires checking the database:

```javascript
this.before('CREATE', 'Orders', async (req) => {
  //                              ↑ Note: async!
  const { customer_ID, items } = req.data;

  // Check if customer exists in the database
  const customer = await SELECT.one
    .from('com.epm.Customers')
    .where({ ID: customer_ID });

  if (!customer) {
    req.reject(404, `Customer ${customer_ID} not found`);
  }

  // Check if customer has sufficient credit
  if (customer.creditLimit < req.data.grossAmount) {
    req.reject(400, `Order amount exceeds customer credit limit of ${customer.creditLimit}`);
  }

  // Check stock for each item
  if (items) {
    for (const item of items) {
      const product = await SELECT.one
        .from('com.epm.Products')
        .where({ ID: item.product_ID });

      if (!product) {
        req.error(400, `Product ${item.product_ID} not found`);
      } else if (product.stock < item.quantity) {
        req.error(400, `Insufficient stock for ${product.productName}. Available: ${product.stock}`);
      }
    }
  }
});
```

---

### AFTER Handlers — Practical Examples

#### Example 1: Add Computed Fields to Response

```javascript
this.after('READ', 'Books', (results) => {
  const books = Array.isArray(results) ? results : [results];

  for (const book of books) {
    // Compute tax (18% GST)
    if (book.price) {
      book.priceWithTax = +(book.price * 1.18).toFixed(2);
      book.taxAmount = +(book.price * 0.18).toFixed(2);
    }

    // Compute availability status
    if (book.stock !== undefined) {
      if (book.stock === 0) book.availability = 'Out of Stock';
      else if (book.stock < 10) book.availability = 'Low Stock';
      else book.availability = 'In Stock';
    }
  }
});
```

---

#### Example 2: Auto-Calculate Order Total

```javascript
this.before('CREATE', 'Orders', (req) => {
  const { items } = req.data;

  if (items && items.length > 0) {
    // Calculate totals from items
    let netAmount = 0;
    for (const item of items) {
      item.netAmount = item.quantity * item.unitPrice;
      netAmount += item.netAmount;
    }

    req.data.netAmount = netAmount;
    req.data.taxAmount = +(netAmount * 0.18).toFixed(2);
    req.data.grossAmount = +(netAmount + req.data.taxAmount).toFixed(2);
  }
});
```

---

#### Example 3: Log All Delete Operations

```javascript
this.after('DELETE', 'Books', (_, req) => {
  const bookId = req.params[0]?.ID || req.params[0];
  console.log(`[AUDIT] Book ${bookId} deleted by ${req.user.id} at ${new Date().toISOString()}`);
});
```

---

#### Example 4: Prevent Deletion Under Certain Conditions

```javascript
this.before('DELETE', 'Books', async (req) => {
  const bookId = req.params[0]?.ID || req.params[0];

  // Check if book has any active orders
  const activeOrders = await SELECT.from('SalesOrderItems')
    .where({ product_ID: bookId });

  if (activeOrders.length > 0) {
    req.reject(409, `Cannot delete book. It has ${activeOrders.length} order references.`);
  }
});
```

---

## Session 5: Error Handling & Custom Error Messages (15:00 - 16:00)

### Error Handling Strategy

Good error handling means:
1. Clear messages that help the user fix the problem
2. Correct HTTP status codes
3. Validation catches errors BEFORE they hit the database
4. Unexpected errors are caught gracefully

---

### HTTP Status Codes for Errors

| Code | Name | When to Use |
|------|------|-------------|
| `400` | Bad Request | Invalid input data (validation failure) |
| `401` | Unauthorized | User not logged in |
| `403` | Forbidden | User logged in but lacks permission |
| `404` | Not Found | Entity/record doesn't exist |
| `409` | Conflict | Action conflicts with current state |
| `422` | Unprocessable | Syntactically valid but semantically wrong |
| `500` | Server Error | Unexpected bug (avoid this!) |

---

### Good vs Bad Error Messages

```javascript
// ❌ BAD — Cryptic, unhelpful messages
req.error(400, 'Error');
req.error(400, 'Invalid');
req.error(400, 'Validation failed');
req.error(500, 'Something went wrong');

// ✅ GOOD — Clear, actionable messages
req.error(400, 'Title is required. Please provide a book title.');
req.error(400, 'Price must be between 0.01 and 9999.99');
req.error(400, 'ISBN must be exactly 13 digits (e.g., 9780747532699)');
req.error(404, 'Author not found. Check the author_ID and try again.');
req.error(409, 'Cannot delete: This product has 3 active orders referencing it.');
```

**Rules for good error messages:**
1. Say WHAT went wrong
2. Say HOW to fix it
3. Include actual values when helpful
4. Use field-specific targeting

---

### Structured Error Response Format

CAP returns errors in OData-standard format:

```json
{
  "error": {
    "code": "400",
    "message": "Multiple validation errors occurred",
    "details": [
      {
        "code": "400",
        "message": "Title is required",
        "target": "title"
      },
      {
        "code": "400",
        "message": "Price must be greater than zero",
        "target": "price"
      },
      {
        "code": "400",
        "message": "Invalid genre 'Horror'. Valid options: Fantasy, Fiction, Mystery",
        "target": "genre"
      }
    ]
  }
}
```

This structure allows UI frameworks (like Fiori) to show errors next to the correct input field.

---

### try-catch for Unexpected Errors

When your handler does complex work (database lookups, calculations), wrap it in try-catch:

```javascript
this.before('CREATE', 'Orders', async (req) => {
  try {
    const customer = await SELECT.one.from('Customers')
      .where({ ID: req.data.customer_ID });

    if (!customer) {
      req.reject(404, 'Customer not found');
    }

    if (customer.creditLimit < req.data.grossAmount) {
      req.reject(400, `Insufficient credit. Limit: ${customer.creditLimit}, Order: ${req.data.grossAmount}`);
    }
  } catch (error) {
    // Log the actual error for debugging
    console.error('Order validation failed:', error);
    // Return a user-friendly message
    req.reject(500, 'Unable to validate order. Please try again later.');
  }
});
```

---

### Common Error Patterns

#### Pattern 1: Required Field Validation

```javascript
this.before('CREATE', 'Products', (req) => {
  const required = ['productName', 'price', 'supplier_ID', 'category_ID'];

  for (const field of required) {
    if (!req.data[field]) {
      req.error(400, `${field} is required`, field);
    }
  }
});
```

---

#### Pattern 2: Range Validation

```javascript
this.before(['CREATE', 'UPDATE'], 'Products', (req) => {
  const { price, stock, rating } = req.data;

  if (price !== undefined) {
    if (price < 0.01) req.error(400, 'Price must be at least 0.01', 'price');
    if (price > 99999.99) req.error(400, 'Price cannot exceed 99999.99', 'price');
  }

  if (stock !== undefined && stock < 0) {
    req.error(400, 'Stock cannot be negative', 'stock');
  }

  if (rating !== undefined) {
    if (rating < 0 || rating > 5) {
      req.error(400, 'Rating must be between 0 and 5', 'rating');
    }
  }
});
```

**Note:** `['CREATE', 'UPDATE']` — you can pass an array to handle multiple events!

---

#### Pattern 3: Format Validation

```javascript
this.before('CREATE', 'Customers', (req) => {
  const { email, phone } = req.data;

  // Email format check
  if (email && !email.includes('@')) {
    req.error(400, 'Please provide a valid email address', 'email');
  }

  // Phone format check (basic)
  if (phone && phone.length < 10) {
    req.error(400, 'Phone number must be at least 10 digits', 'phone');
  }
});
```

---

#### Pattern 4: Uniqueness Validation

```javascript
this.before('CREATE', 'Products', async (req) => {
  const { productName } = req.data;

  if (productName) {
    const existing = await SELECT.one.from('Products')
      .where({ productName });

    if (existing) {
      req.error(409, `Product "${productName}" already exists`, 'productName');
    }
  }
});
```

---

#### Pattern 5: State Transition Validation

```javascript
this.before('UPDATE', 'Orders', async (req) => {
  if (req.data.status) {
    const orderId = req.params[0]?.ID || req.params[0];
    const current = await SELECT.one.from('Orders').where({ ID: orderId });

    const validTransitions = {
      'New': ['Confirmed', 'Cancelled'],
      'Confirmed': ['Shipped', 'Cancelled'],
      'Shipped': ['Delivered'],
      'Delivered': [],
      'Cancelled': []
    };

    const allowed = validTransitions[current.status] || [];

    if (!allowed.includes(req.data.status)) {
      req.reject(400,
        `Cannot change status from "${current.status}" to "${req.data.status}". ` +
        `Allowed transitions: ${allowed.join(', ') || 'none (final state)'}`
      );
    }
  }
});
```

---

### Complete Handler File — Putting It All Together

Here's a complete, well-structured handler file:

```javascript
// srv/admin-service.js
const cds = require('@sap/cds');

module.exports = function () {

  // ═══════════════════════════════════════════════
  //  BOOKS — Validation & Logic
  // ═══════════════════════════════════════════════

  // --- Before CREATE: Validate input ---
  this.before('CREATE', 'Books', async (req) => {
    const { title, price, stock, isbn, author_ID } = req.data;

    // Required fields
    if (!title || title.trim() === '') {
      req.error(400, 'Title is required', 'title');
    }
    if (price === undefined || price === null) {
      req.error(400, 'Price is required', 'price');
    }

    // Range validation
    if (price !== undefined && price <= 0) {
      req.error(400, 'Price must be greater than zero', 'price');
    }
    if (stock !== undefined && stock < 0) {
      req.error(400, 'Stock cannot be negative', 'stock');
    }

    // Format validation
    if (isbn && !/^\d{13}$/.test(isbn)) {
      req.error(400, 'ISBN must be exactly 13 digits', 'isbn');
    }

    // Referential integrity
    if (author_ID) {
      const { Authors } = cds.entities;
      const author = await SELECT.one.from(Authors).where({ ID: author_ID });
      if (!author) {
        req.error(404, 'Author not found', 'author_ID');
      }
    }

    // Clean input
    if (title) req.data.title = title.trim();
  });

  // --- Before UPDATE: Same validations for changed fields ---
  this.before('UPDATE', 'Books', (req) => {
    const { price, stock, isbn } = req.data;

    if (price !== undefined && price <= 0) {
      req.error(400, 'Price must be greater than zero', 'price');
    }
    if (stock !== undefined && stock < 0) {
      req.error(400, 'Stock cannot be negative', 'stock');
    }
    if (isbn !== undefined && !/^\d{13}$/.test(isbn)) {
      req.error(400, 'ISBN must be exactly 13 digits', 'isbn');
    }
  });

  // --- After READ: Add computed fields ---
  this.after('READ', 'Books', (results) => {
    const books = Array.isArray(results) ? results : [results];

    for (const book of books) {
      if (book.price) {
        book.priceWithTax = +(book.price * 1.18).toFixed(2);
      }
      if (book.stock !== undefined) {
        book.availability = book.stock > 10 ? 'In Stock'
                          : book.stock > 0  ? 'Low Stock'
                          : 'Out of Stock';
      }
    }
  });

  // --- Before DELETE: Check dependencies ---
  this.before('DELETE', 'Books', async (req) => {
    const ID = req.params[0]?.ID || req.params[0];
    // In a real app, check if this book is referenced in any orders
    console.log(`[INFO] Deleting book ${ID}`);
  });

  // ═══════════════════════════════════════════════
  //  AUTHORS — Validation & Logic
  // ═══════════════════════════════════════════════

  this.before('CREATE', 'Authors', (req) => {
    const { name, email } = req.data;

    if (!name || name.trim() === '') {
      req.error(400, 'Author name is required', 'name');
    }

    if (email && !email.includes('@')) {
      req.error(400, 'Please provide a valid email address', 'email');
    }

    if (name) req.data.name = name.trim();
  });

  this.before('DELETE', 'Authors', async (req) => {
    const ID = req.params[0]?.ID || req.params[0];
    const { Books } = cds.entities;
    const books = await SELECT.from(Books).where({ author_ID: ID });

    if (books.length > 0) {
      req.reject(409,
        `Cannot delete author: ${books.length} book(s) reference this author. Delete or reassign books first.`
      );
    }
  });

};
```

---

## Session 6: Hands-on — Implement Validations for EPM (16:00 - 16:45)

### Task: Add Custom Handlers for EPM Purchase Orders

Create validation handlers for the EPM model's sales/purchasing services.

---

### Step 1: Create the Handler File

**File: `srv/sales-service.js`**

```javascript
const cds = require('@sap/cds');

module.exports = function () {

  // ═══════════════════════════════════════════════
  //  SALES ORDERS — Validations
  // ═══════════════════════════════════════════════

  this.before('CREATE', 'SalesOrders', async (req) => {
    const { customer_ID, orderDate, items } = req.data;

    // 1. Customer is required
    if (!customer_ID) {
      req.error(400, 'Customer is required for orders', 'customer_ID');
    }

    // 2. Order date cannot be in the past
    if (orderDate) {
      const today = new Date().toISOString().split('T')[0];
      if (orderDate < today) {
        req.error(400, 'Order date cannot be in the past', 'orderDate');
      }
    }

    // 3. Must have at least one item
    if (!items || items.length === 0) {
      req.error(400, 'Order must have at least one item');
    }

    // 4. Validate each item
    if (items) {
      for (let i = 0; i < items.length; i++) {
        const item = items[i];

        if (!item.product_ID) {
          req.error(400, `Item ${i + 1}: Product is required`);
        }
        if (!item.quantity || item.quantity <= 0) {
          req.error(400, `Item ${i + 1}: Quantity must be greater than zero`);
        }
        if (!item.unitPrice || item.unitPrice <= 0) {
          req.error(400, `Item ${i + 1}: Unit price must be greater than zero`);
        }
      }
    }

    // 5. Verify customer exists
    if (customer_ID) {
      const customer = await SELECT.one.from('com.epm.Customers')
        .where({ ID: customer_ID });
      if (!customer) {
        req.error(404, 'Customer not found', 'customer_ID');
      }
    }
  });

  // Auto-calculate totals before saving
  this.before('CREATE', 'SalesOrders', (req) => {
    const { items } = req.data;

    if (items && items.length > 0) {
      let netAmount = 0;

      for (const item of items) {
        item.netAmount = +(item.quantity * item.unitPrice).toFixed(2);
        netAmount += item.netAmount;
      }

      req.data.netAmount = +netAmount.toFixed(2);
      req.data.taxAmount = +(netAmount * 0.18).toFixed(2);
      req.data.grossAmount = +(netAmount * 1.18).toFixed(2);
    }

    // Set default status
    if (!req.data.status) {
      req.data.status = 'New';
    }
  });

  // Status transition validation
  this.before('UPDATE', 'SalesOrders', async (req) => {
    if (req.data.status) {
      const orderId = req.params[0]?.ID || req.params[0];
      const order = await SELECT.one.from('com.epm.SalesOrders')
        .where({ ID: orderId });

      if (!order) {
        req.reject(404, 'Order not found');
      }

      const transitions = {
        'New': ['Confirmed', 'Cancelled'],
        'Confirmed': ['Shipped', 'Cancelled'],
        'Shipped': ['Delivered'],
        'Delivered': [],
        'Cancelled': []
      };

      const allowed = transitions[order.status] || [];
      if (!allowed.includes(req.data.status)) {
        req.reject(400,
          `Cannot change status from "${order.status}" to "${req.data.status}". ` +
          `Allowed: ${allowed.join(', ') || 'none (final state)'}`
        );
      }
    }
  });

  // Prevent deleting delivered orders
  this.before('DELETE', 'SalesOrders', async (req) => {
    const orderId = req.params[0]?.ID || req.params[0];
    const order = await SELECT.one.from('com.epm.SalesOrders')
      .where({ ID: orderId });

    if (order && order.status === 'Delivered') {
      req.reject(409, 'Cannot delete a delivered order. Archive it instead.');
    }
  });

  // ═══════════════════════════════════════════════
  //  AFTER READ: Enrich order data
  // ═══════════════════════════════════════════════

  this.after('READ', 'SalesOrders', (results) => {
    const orders = Array.isArray(results) ? results : [results];

    for (const order of orders) {
      if (order.status) {
        const statusInfo = {
          'New': { priority: 'Normal', color: 'blue' },
          'Confirmed': { priority: 'Normal', color: 'green' },
          'Shipped': { priority: 'High', color: 'orange' },
          'Delivered': { priority: 'Low', color: 'grey' },
          'Cancelled': { priority: 'None', color: 'red' }
        };
        const info = statusInfo[order.status];
        if (info) {
          order.statusPriority = info.priority;
          order.statusColor = info.color;
        }
      }
    }
  });

};
```

---

### Step 2: Test Your Validations

```http
@sales = http://localhost:4004/sales

### Test 1: Create order without customer (should fail)
POST {{sales}}/SalesOrders
Content-Type: application/json

{
  "orderDate": "2026-06-15",
  "items": [
    { "product_ID": "p1000001-...", "quantity": 2, "unitPrice": 50.00 }
  ]
}

### Test 2: Create order with past date (should fail)
POST {{sales}}/SalesOrders
Content-Type: application/json

{
  "customer_ID": "c1000001-...",
  "orderDate": "2020-01-01",
  "items": [
    { "product_ID": "p1000001-...", "quantity": 2, "unitPrice": 50.00 }
  ]
}

### Test 3: Create order with invalid item (quantity 0)
POST {{sales}}/SalesOrders
Content-Type: application/json

{
  "customer_ID": "c1000001-...",
  "orderDate": "2026-06-15",
  "items": [
    { "product_ID": "p1000001-...", "quantity": 0, "unitPrice": 50.00 }
  ]
}

### Test 4: Create valid order (should succeed + auto-calculate totals!)
POST {{sales}}/SalesOrders
Content-Type: application/json

{
  "customer_ID": "c1000001-...",
  "orderNumber": "SO-201",
  "orderDate": "2026-06-15",
  "currency_code": "USD",
  "items": [
    { "product_ID": "p1000001-...", "quantity": 2, "unitPrice": 499.99 },
    { "product_ID": "p1000002-...", "quantity": 1, "unitPrice": 29.99 }
  ]
}

### Test 5: Try invalid status transition (New → Delivered should fail)
PATCH {{sales}}/SalesOrders(ORDER-ID-FROM-TEST-4)
Content-Type: application/json

{
  "status": "Delivered"
}

### Test 6: Valid status transition (New → Confirmed)
PATCH {{sales}}/SalesOrders(ORDER-ID-FROM-TEST-4)
Content-Type: application/json

{
  "status": "Confirmed"
}
```

---

### Expected Results

| Test | Expected |
|------|----------|
| 1 | 400 — "Customer is required" |
| 2 | 400 — "Order date cannot be in the past" |
| 3 | 400 — "Quantity must be greater than zero" |
| 4 | 201 — Order created with auto-calculated totals |
| 5 | 400 — "Cannot change status from New to Delivered" |
| 6 | 200 — Status updated to Confirmed |

For Test 4, check that `netAmount`, `taxAmount`, and `grossAmount` are calculated:
```
netAmount = (2 × 499.99) + (1 × 29.99) = 1029.97
taxAmount = 1029.97 × 0.18 = 185.39
grossAmount = 1029.97 + 185.39 = 1215.36
```

---

## Assessment: MCQ — 15 Questions on CRUD & Custom Handlers (16:45 - 17:00)

**Q1.** Which HTTP method is used for creating a new record?
- A) GET
- B) PUT
- C) POST
- D) PATCH

---

**Q2.** What is the difference between PUT and PATCH?
- A) PUT is faster, PATCH is slower
- B) PUT replaces the entire record, PATCH updates only sent fields
- C) PUT is for create, PATCH is for update
- D) There is no difference

---

**Q3.** What status code does a successful CREATE return?
- A) 200 OK
- B) 201 Created
- C) 204 No Content
- D) 202 Accepted

---

**Q4.** What status code does a successful DELETE return?
- A) 200 OK
- B) 201 Created
- C) 204 No Content
- D) 404 Not Found

---

**Q5.** Custom handler files must be named:
- A) Same as the service CDS file but with `.js` extension
- B) `handlers.js` always
- C) Any name you want
- D) Same as the entity name

---

**Q6.** A `before` handler is used for:
- A) Transforming the response data
- B) Validating input and rejecting bad data
- C) Replacing the default CRUD behavior
- D) Sending emails after success

---

**Q7.** An `after` handler receives which parameters?
- A) `(req)` only
- B) `(results, req)`
- C) `(error, req)`
- D) `(req, res, next)`

---

**Q8.** `req.error(400, 'Bad input', 'price')` — what does the third parameter do?
- A) Sets the error code
- B) Identifies which field caused the error
- C) Sets the HTTP method
- D) Names the entity

---

**Q9.** What is the difference between `req.error()` and `req.reject()`?
- A) `error` collects multiple errors; `reject` stops immediately
- B) `error` is for 400s; `reject` is for 500s
- C) They are identical
- D) `reject` is deprecated

---

**Q10.** In an `on` handler, what does `await next()` do?
- A) Goes to the next middleware
- B) Calls the default CAP handler (standard CRUD)
- C) Skips to the after handler
- D) Sends the response

---

**Q11.** TRUE or FALSE: Deep Insert (creating parent + children in one POST) works with Associations.
- A) TRUE
- B) FALSE

---

**Q12.** You can handle both CREATE and UPDATE in one handler by:
- A) `this.before('CREATE,UPDATE', 'Books', ...)`
- B) `this.before(['CREATE', 'UPDATE'], 'Books', ...)`
- C) `this.before('*', 'Books', ...)`
- D) Both B and C work

---

**Q13.** What happens if you PATCH with `{ "price": 14.99 }` on a book?
- A) Only price changes; all other fields stay the same
- B) All fields except price become null
- C) A new record is created
- D) The book is deleted and recreated

---

**Q14.** `req.data` in a DELETE handler contains:
- A) The full record being deleted
- B) Nothing (empty object)
- C) The key/ID of the record being deleted
- D) The deletion confirmation message

---

**Q15.** To check if an author exists before creating a book (database lookup in handler), you need:
- A) `this.before('CREATE', 'Books', (req) => { ... })` — synchronous
- B) `this.before('CREATE', 'Books', async (req) => { ... })` — async with await
- C) A separate cron job
- D) It's not possible in CAP

---

### Answer Key

<details>
<summary>Click to reveal answers</summary>

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **C** | POST creates new records |
| 2 | **B** | PUT = full replace (missing fields → null), PATCH = partial update |
| 3 | **B** | 201 Created is the standard response for successful POST |
| 4 | **C** | 204 No Content — deleted successfully, nothing to return |
| 5 | **A** | `srv/my-service.cds` → `srv/my-service.js` (same name) |
| 6 | **B** | `before` validates input and can reject requests |
| 7 | **B** | After handlers receive `(results, req)` — data first, then request |
| 8 | **B** | Third param is `target` — identifies the problematic field |
| 9 | **A** | `error()` collects multiple errors; `reject()` stops processing immediately |
| 10 | **B** | `next()` delegates to CAP's default CRUD handler |
| 11 | **FALSE** | Deep Insert only works with Composition, not Association |
| 12 | **D** | Both array syntax `['CREATE','UPDATE']` and wildcard `'*'` work |
| 13 | **A** | PATCH only changes specified fields; everything else is untouched |
| 14 | **C** | For DELETE, `req.params` contains the key (ID) |
| 15 | **B** | Database lookups require `async/await` pattern |

</details>

---

## Assignment: Implement Full CRUD with Validations for Product Catalog

### Task

Create a complete **Product Catalog** service with full CRUD operations and custom validation logic.

---

### Requirements

#### 1. Create the Data Model

**File: `db/product-catalog.cds`** (or add to your existing schema)

```cds
namespace com.catalog;
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
  minStock      : Integer default 10;
  rating        : Decimal(2,1) default 0.0;
  status        : ProductStatus default 'Draft';
  sku           : String(20);
  weight        : Decimal(6,2);
  category      : Association to Categories;
}

entity Categories : cuid, managed {
  categoryName  : String(100);
  description   : String(300);
  isActive      : Boolean default true;
  products      : Association to many Products on products.category = $self;
}
```

---

#### 2. Create the Service

**File: `srv/product-catalog-service.cds`**

```cds
using { com.catalog as db } from '../db/product-catalog';

service ProductCatalogService @(path: '/products') {
  entity Products as projection on db.Products;
  entity Categories as projection on db.Categories;
}
```

---

#### 3. Implement These Validations

**File: `srv/product-catalog-service.js`**

Implement the following rules:

| Entity | Event | Rule |
|--------|-------|------|
| Products | CREATE | `productName` is required |
| Products | CREATE | `price` is required and must be > 0 |
| Products | CREATE | `price` must not exceed 99999.99 |
| Products | CREATE | `sku` must be unique (no two products with same SKU) |
| Products | CREATE | `sku` format: 3 uppercase letters + dash + 5 digits (e.g., "PRD-00001") |
| Products | CREATE | `stock` cannot be negative |
| Products | CREATE | `category_ID` must reference an existing active category |
| Products | CREATE/UPDATE | Trim whitespace from `productName` |
| Products | UPDATE | Cannot update a "Discontinued" product (except to delete it) |
| Products | UPDATE | `price` if provided, must be > 0 |
| Products | DELETE | Cannot delete "Active" products with stock > 0 |
| Products | READ | Add computed field `needsReorder` (true if stock < minStock) |
| Products | READ | Add computed field `priceWithTax` (price × 1.18) |
| Categories | CREATE | `categoryName` is required |
| Categories | DELETE | Cannot delete if category has products |

---

#### 4. Test File

Create `tests/product-catalog.http` with test cases for:
- Creating a valid product ✅
- Creating without required fields ❌
- Creating with invalid SKU format ❌
- Creating with duplicate SKU ❌
- Creating with non-existent category ❌
- Updating price to negative ❌
- Updating a discontinued product ❌
- Deleting an active product with stock ❌
- Reading products (check computed fields) ✅
- Deleting a category that has products ❌

---

### Solution Template

<details>
<summary>Solution: product-catalog-service.js (try it yourself first!)</summary>

```javascript
const cds = require('@sap/cds');

module.exports = function () {

  // ═══════════════════════════════════════════════
  //  PRODUCTS — BEFORE CREATE
  // ═══════════════════════════════════════════════
  this.before('CREATE', 'Products', async (req) => {
    const { productName, price, stock, sku, category_ID } = req.data;

    // Required fields
    if (!productName || productName.trim() === '') {
      req.error(400, 'Product name is required', 'productName');
    }
    if (price === undefined || price === null) {
      req.error(400, 'Price is required', 'price');
    }

    // Price range
    if (price !== undefined && price <= 0) {
      req.error(400, 'Price must be greater than zero', 'price');
    }
    if (price !== undefined && price > 99999.99) {
      req.error(400, 'Price cannot exceed 99999.99', 'price');
    }

    // Stock validation
    if (stock !== undefined && stock < 0) {
      req.error(400, 'Stock cannot be negative', 'stock');
    }

    // SKU format: AAA-00000
    if (sku) {
      const skuPattern = /^[A-Z]{3}-\d{5}$/;
      if (!skuPattern.test(sku)) {
        req.error(400, 'SKU must match format: 3 uppercase letters + dash + 5 digits (e.g., PRD-00001)', 'sku');
      }

      // SKU uniqueness
      const existing = await SELECT.one.from('com.catalog.Products').where({ sku });
      if (existing) {
        req.error(409, `SKU "${sku}" already exists`, 'sku');
      }
    }

    // Category must exist and be active
    if (category_ID) {
      const category = await SELECT.one.from('com.catalog.Categories')
        .where({ ID: category_ID });
      if (!category) {
        req.error(404, 'Category not found', 'category_ID');
      } else if (!category.isActive) {
        req.error(400, 'Cannot assign product to an inactive category', 'category_ID');
      }
    }

    // Clean input
    if (productName) req.data.productName = productName.trim();
  });

  // ═══════════════════════════════════════════════
  //  PRODUCTS — BEFORE UPDATE
  // ═══════════════════════════════════════════════
  this.before('UPDATE', 'Products', async (req) => {
    const productId = req.params[0]?.ID || req.params[0];
    const current = await SELECT.one.from('com.catalog.Products').where({ ID: productId });

    if (!current) {
      req.reject(404, 'Product not found');
    }

    // Cannot update discontinued products
    if (current.status === 'Discontinued') {
      req.reject(400, 'Cannot update a discontinued product. Delete it or change its status first.');
    }

    // Price validation
    if (req.data.price !== undefined && req.data.price <= 0) {
      req.error(400, 'Price must be greater than zero', 'price');
    }

    // Stock validation
    if (req.data.stock !== undefined && req.data.stock < 0) {
      req.error(400, 'Stock cannot be negative', 'stock');
    }

    // Clean input
    if (req.data.productName) {
      req.data.productName = req.data.productName.trim();
    }
  });

  // ═══════════════════════════════════════════════
  //  PRODUCTS — BEFORE DELETE
  // ═══════════════════════════════════════════════
  this.before('DELETE', 'Products', async (req) => {
    const productId = req.params[0]?.ID || req.params[0];
    const product = await SELECT.one.from('com.catalog.Products').where({ ID: productId });

    if (product && product.status === 'Active' && product.stock > 0) {
      req.reject(409,
        `Cannot delete active product with ${product.stock} items in stock. ` +
        `Set status to "Discontinued" first or reduce stock to zero.`
      );
    }
  });

  // ═══════════════════════════════════════════════
  //  PRODUCTS — AFTER READ
  // ═══════════════════════════════════════════════
  this.after('READ', 'Products', (results) => {
    const products = Array.isArray(results) ? results : [results];

    for (const product of products) {
      // Needs reorder?
      if (product.stock !== undefined && product.minStock !== undefined) {
        product.needsReorder = product.stock < product.minStock;
      }

      // Price with tax
      if (product.price) {
        product.priceWithTax = +(product.price * 1.18).toFixed(2);
      }
    }
  });

  // ═══════════════════════════════════════════════
  //  CATEGORIES — VALIDATIONS
  // ═══════════════════════════════════════════════
  this.before('CREATE', 'Categories', (req) => {
    if (!req.data.categoryName || req.data.categoryName.trim() === '') {
      req.error(400, 'Category name is required', 'categoryName');
    }
    if (req.data.categoryName) {
      req.data.categoryName = req.data.categoryName.trim();
    }
  });

  this.before('DELETE', 'Categories', async (req) => {
    const catId = req.params[0]?.ID || req.params[0];
    const products = await SELECT.from('com.catalog.Products')
      .where({ category_ID: catId });

    if (products.length > 0) {
      req.reject(409,
        `Cannot delete category: ${products.length} product(s) belong to it. Reassign or delete them first.`
      );
    }
  });

};
```

</details>

---

### Verification Checklist

After implementing, verify:

- [ ] `cds watch` starts without errors
- [ ] Creating a product without name → 400 error
- [ ] Creating a product with negative price → 400 error
- [ ] Creating with invalid SKU → 400 error
- [ ] Creating with valid data → 201 success + auto-generated fields
- [ ] Reading products shows `needsReorder` and `priceWithTax`
- [ ] Updating a discontinued product → 400 error
- [ ] Deleting active product with stock → 409 error
- [ ] Deleting category with products → 409 error
- [ ] All error messages are clear and actionable

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| CRUD | POST=Create, GET=Read, PUT/PATCH=Update, DELETE=Delete |
| PUT vs PATCH | PUT replaces everything, PATCH updates only sent fields |
| `before` handlers | Validate input, reject bad data, modify data before save |
| `after` handlers | Add computed fields, trigger side effects, transform output |
| `on` handlers | Replace default CRUD with custom logic |
| `req.data` | The incoming request body |
| `req.error()` | Collect validation errors (can have multiple) |
| `req.reject()` | Immediately stop processing |
| `async` handlers | Required for database lookups in validation |
| Error messages | Should be clear, actionable, and field-specific |

### The Handler Decision Tree

```
Need to validate input before saving?         → BEFORE
Need to add computed fields to response?      → AFTER
Need to send notification on success?         → AFTER
Need to completely replace CRUD behavior?     → ON
Need to check database before allowing?       → BEFORE (async)
Need to prevent delete under conditions?      → BEFORE DELETE
Need to auto-calculate fields before insert?  → BEFORE CREATE
```

### The One-Liner

> **Custom handlers let you inject YOUR business rules into CAP's automatic CRUD — before the data is saved, or after it's read.**

---

### Looking Ahead: Day 19

Tomorrow we'll explore more advanced service features:
- Actions and Functions (custom operations beyond CRUD)
- Bound and unbound actions
- Service-level events
- Working with external services

---

### Homework

1. Complete the Product Catalog assignment above
2. Add these EXTRA validations to your handler:
   - Products cannot have rating > 5.0 or < 0
   - When status changes from "Draft" to "Active", price must be set
   - SKU is required for Active products (optional for Draft)
3. Create a `.http` test file that tests EVERY validation rule
4. Add an `after('READ', 'Categories')` handler that adds a `productCount` field showing how many products each category has

---

*End of Day 18 — You now control what happens when data is created, read, updated, or deleted!*
