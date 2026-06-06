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
