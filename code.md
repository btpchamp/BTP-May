## Session 3: Hands-on — Create CatalogService for Library (14:00 - 15:00)

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
