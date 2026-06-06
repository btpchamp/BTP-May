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
