# Day 17: OData Query Operations — Mastering Data Queries

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 16 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: OData Query Options Overview & $filter Deep Dive | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: $select, $orderby, $top, $skip, $count | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Practice Filtering, Sorting & Pagination | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: $expand — Navigating Relationships | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: $search, Combining Queries & Real-World Patterns | 60 min |
| 16:00 - 16:30 | Session 6: Hands-on — Build Complex Queries & Document APIs | 30 min |
| 16:30 - 17:00 | Assessment & Business Scenario Quiz | 30 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Use ALL OData query options fluently ($filter, $select, $expand, $orderby, $top, $skip, $count, $search)
- Build complex filter expressions with comparison, logical, and string operators
- Navigate deep relationships using nested $expand
- Implement pagination for large datasets
- Combine multiple query options for real business scenarios
- Write OData queries as confidently as SQL queries

---

## Day 16 Recap — Quick Fire (09:00 - 09:15)

1. A CAP service controls _____ to your data? → _____
2. `as projection on` connects _____ entity to _____ entity? → _____
3. `@readonly` blocks which HTTP methods? → _____
4. `CatalogService` generates which URL path? → _____
5. Where do you find the API schema? → _____
6. `$expand=author` does what? → _____
7. CAP auto-generates _____, _____, _____, and _____ operations? → _____
8. Compared to Express.js, a CAP service needs how much code? → _____

<details>
<summary>Answers</summary>

1. **Access** — services are the controlled access layer
2. **Service** entity to **database** entity
3. POST, PUT, PATCH, DELETE (only GET allowed)
4. `/catalog` (removes "Service" suffix, lowercases)
5. `/$metadata` endpoint (e.g., `/catalog/$metadata`)
6. Includes related entity data inline (like a SQL JOIN)
7. **Create, Read, Update, Delete** (CRUD)
8. ~90% less code (20 lines vs 90+ lines)

</details>

---

## Session 1: OData Query Options Overview & $filter Deep Dive (09:15 - 10:30)

### The Google Search Analogy

Think of OData query options like Google search features:

```
Google:                              OData:
─────────────────────────────        ─────────────────────────────
"cat videos"                    →    $search=cat videos
site:youtube.com                →    (specific service endpoint)
filetype:mp4                    →    $filter=format eq 'mp4'
sort by date                    →    $orderby=date desc
first 10 results                →    $top=10
page 2                          →    $skip=10
"About 1,240,000 results"       →    $count=true
```

OData gives you the SAME power over your data API that Google gives you over web search!

---

### The Complete OData Query Options Map

```
┌──────────────────────────────────────────────────────────────────┐
│              OData Query Options — Your Toolkit                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WHAT to return:                                                 │
│    $select    → Which columns/fields to include                  │
│    $expand    → Which related entities to include                │
│                                                                  │
│  WHICH records:                                                  │
│    $filter    → Conditions to match (WHERE clause)               │
│    $search    → Free-text search across fields                   │
│                                                                  │
│  HOW to arrange:                                                 │
│    $orderby   → Sort order (ascending/descending)                │
│                                                                  │
│  HOW MANY:                                                       │
│    $top       → Maximum number to return                         │
│    $skip      → Number to skip (for pagination)                  │
│    $count     → Include total count in response                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### URL Structure Reminder

All query options go AFTER the `?` in the URL, separated by `&`:

```
GET /catalog/Books ? $filter=price lt 20 & $orderby=title & $top=10
                   ↑                     ↑               ↑
                 start                 AND             AND
              of query              another          another
              options               option           option
```

**Important:** The `$` prefix is required for ALL query options. Without it, it's not recognized!

---

### $filter — The Most Powerful Query Option

`$filter` is like the WHERE clause in SQL. It decides WHICH records come back.

```
SQL:     SELECT * FROM Books WHERE price < 20
OData:   GET /catalog/Books?$filter=price lt 20
```

---

### $filter: Comparison Operators

These compare a field to a value:

| Operator | Meaning | SQL Equivalent | Example |
|----------|---------|----------------|---------|
| `eq` | Equals | `=` | `$filter=genre eq 'Fantasy'` |
| `ne` | Not equals | `!=` or `<>` | `$filter=status ne 'Cancelled'` |
| `gt` | Greater than | `>` | `$filter=price gt 10` |
| `lt` | Less than | `<` | `$filter=stock lt 20` |
| `ge` | Greater than or equal | `>=` | `$filter=rating ge 4.5` |
| `le` | Less than or equal | `<=` | `$filter=price le 15.99` |

**Memory trick:** 
- **eq** = **eq**uals
- **ne** = **n**ot **e**qual
- **gt** = **g**reater **t**han
- **lt** = **l**ess **t**han
- **ge** = **g**reater-or-**e**qual
- **le** = **l**ess-or-**e**qual

---

### Examples with Different Data Types

```
// String comparison (MUST use single quotes!)
$filter=genre eq 'Fantasy'
$filter=status ne 'Cancelled'
$filter=country eq 'India'

// Number comparison (no quotes needed)
$filter=price gt 10
$filter=stock lt 50
$filter=rating ge 4.0

// Boolean comparison
$filter=isActive eq true
$filter=isAvailable eq false

// Date comparison (use YYYY-MM-DD format)
$filter=publishDate gt 2020-01-01
$filter=orderDate le 2026-06-01

// UUID comparison
$filter=ID eq 3f4a2b1c-d5e6-7890-abcd-ef1234567890

// Null check
$filter=email eq null
$filter=phone ne null
```

**Common mistake:** Forgetting quotes around strings!
```
❌ $filter=genre eq Fantasy        → ERROR!
✅ $filter=genre eq 'Fantasy'      → Works!
```

---

### $filter: Logical Operators

Combine multiple conditions:

| Operator | Meaning | SQL Equivalent | Example |
|----------|---------|----------------|---------|
| `and` | Both must be true | `AND` | `price gt 10 and price lt 20` |
| `or` | Either must be true | `OR` | `genre eq 'Fantasy' or genre eq 'Mystery'` |
| `not` | Reverse the condition | `NOT` | `not contains(title,'Test')` |

---

### Combining Conditions — Examples

```
// Price between 10 and 20 (range query)
$filter=price ge 10 and price le 20

// Fantasy OR Mystery genre
$filter=genre eq 'Fantasy' or genre eq 'Mystery'

// Active products with low stock
$filter=isActive eq true and stock lt 10

// Orders this year that are not cancelled
$filter=orderDate ge 2026-01-01 and status ne 'Cancelled'

// High-rated books under $15
$filter=rating ge 4.5 and price lt 15
```

---

### Grouping with Parentheses

When mixing `and` and `or`, use parentheses to be clear:

```
// WRONG — ambiguous! What does this mean?
$filter=genre eq 'Fantasy' or genre eq 'Mystery' and price lt 15

// Does it mean:
//   (Fantasy) OR (Mystery AND price<15) ?
//   (Fantasy OR Mystery) AND (price<15) ?

// CORRECT — clear with parentheses:
$filter=(genre eq 'Fantasy' or genre eq 'Mystery') and price lt 15
// This means: genre is Fantasy or Mystery, AND price must be under 15
```

**Rule:** Always use parentheses when mixing `and` and `or`!

---

### $filter: String Functions

OData provides special functions for text searching:

| Function | What It Does | Example |
|----------|-------------|---------|
| `contains(field,'text')` | Field contains text anywhere | `contains(title,'Harry')` |
| `startswith(field,'text')` | Field starts with text | `startswith(name,'J')` |
| `endswith(field,'text')` | Field ends with text | `endswith(email,'.com')` |
| `tolower(field)` | Lowercase the field for comparison | `tolower(name) eq 'orwell'` |
| `toupper(field)` | Uppercase the field for comparison | `toupper(code) eq 'USD'` |
| `length(field)` | Get string length | `length(name) gt 10` |
| `trim(field)` | Remove leading/trailing spaces | `trim(name) eq 'Alice'` |
| `concat(a,b)` | Join two strings | `contains(concat(firstName,lastName),'John')` |
| `substring(field,start,length)` | Extract part of string | `substring(isbn,0,3) eq '978'` |
| `indexof(field,'text')` | Position of text (-1 if not found) | `indexof(email,'@') gt 0` |

---

### String Function Examples

```
// Books with "Potter" anywhere in the title
$filter=contains(title,'Potter')

// Authors whose name starts with "J"
$filter=startswith(name,'J')

// Emails ending with ".com"
$filter=endswith(email,'.com')

// Case-insensitive search (convert to lowercase first)
$filter=contains(tolower(title),'harry')

// Names longer than 20 characters
$filter=length(name) gt 20

// ISBN starting with 978 (standard book prefix)
$filter=startswith(isbn,'978')
```

---

### $filter: Arithmetic Operators

You can do math in filters:

| Operator | Meaning | Example |
|----------|---------|---------|
| `add` | Addition | `price add 5 gt 20` |
| `sub` | Subtraction | `stock sub minStock lt 0` |
| `mul` | Multiplication | `price mul quantity gt 100` |
| `div` | Division | `total div quantity le 50` |
| `mod` | Modulo (remainder) | `ID mod 2 eq 0` |

```
// Products where stock is below minimum (need reorder)
$filter=stock sub minStock lt 0

// Orders where total amount exceeds 1000
$filter=quantity mul unitPrice gt 1000

// Products with more than 20% discount
$filter=price sub discountedPrice div price gt 0.2
```

---

### $filter: Date & Time Functions

| Function | What It Does | Example |
|----------|-------------|---------|
| `year(field)` | Extract year | `year(orderDate) eq 2026` |
| `month(field)` | Extract month (1-12) | `month(hireDate) eq 6` |
| `day(field)` | Extract day (1-31) | `day(birthDate) eq 15` |
| `hour(field)` | Extract hour (0-23) | `hour(createdAt) ge 9` |
| `now()` | Current date/time | `orderDate lt now()` |

```
// Orders placed in 2026
$filter=year(orderDate) eq 2026

// Employees hired in June
$filter=month(hireDate) eq 6

// Books published after 2000
$filter=year(publishDate) gt 2000

// Orders from this month
$filter=year(orderDate) eq 2026 and month(orderDate) eq 6
```

---

### $filter Cheat Sheet — Visual Summary

```
┌─────────────────────────────────────────────────────────────┐
│                    $filter CHEAT SHEET                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  COMPARISON:  eq  ne  gt  lt  ge  le                        │
│               =   !=  >   <   >=  <=                        │
│                                                             │
│  LOGICAL:     and    or    not    (parentheses)             │
│               &&     ||    !      for grouping              │
│                                                             │
│  STRINGS:     contains()   startswith()   endswith()        │
│               tolower()    toupper()      length()          │
│               concat()     substring()    indexof()         │
│                                                             │
│  MATH:        add    sub    mul    div    mod               │
│               +      -      ×      ÷      %                │
│                                                             │
│  DATES:       year()   month()   day()   hour()   now()    │
│                                                             │
│  SPECIAL:     null (for null checks)                        │
│               true / false (for booleans)                   │
│                                                             │
│  QUOTES:      Strings: 'single quotes'                     │
│               Numbers: no quotes                            │
│               Booleans: no quotes                           │
│               Dates: no quotes (YYYY-MM-DD)                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Practice: Write the $filter

For each business requirement, write the `$filter` expression:

1. "Show me all products that cost more than $50" → _____
2. "Find customers from India" → _____
3. "Get all active employees" → _____
4. "Show orders placed after March 2026" → _____
5. "Find books with 'Lord' in the title" → _____
6. "Products with stock between 10 and 100" → _____
7. "Authors from UK or USA" → _____
8. "Orders that are NOT delivered and cost over $500" → _____

<details>
<summary>Answers</summary>

1. `$filter=price gt 50`
2. `$filter=country eq 'India'`
3. `$filter=isActive eq true`
4. `$filter=orderDate gt 2026-03-31`
5. `$filter=contains(title,'Lord')`
6. `$filter=stock ge 10 and stock le 100`
7. `$filter=country eq 'United Kingdom' or country eq 'United States'`
8. `$filter=status ne 'Delivered' and grossAmount gt 500`

</details>

---

## Session 2: $select, $orderby, $top, $skip, $count (10:45 - 12:00)

### $select — Choose Which Fields to Return

**Problem:** Your entity has 15 fields but you only need 3. Why download all 15?

```
WITHOUT $select:
GET /catalog/Books

Response (ALL fields):
{
  "ID": "...",
  "title": "Harry Potter...",
  "genre": "Fantasy",
  "price": 12.99,
  "stock": 150,
  "rating": 4.8,
  "publishDate": "1997-06-26",
  "isbn": "9780747532699",
  "createdAt": "...",
  "createdBy": "...",
  "modifiedAt": "...",
  "modifiedBy": "..."    ← 12 fields! Most not needed!
}
```

```
WITH $select:
GET /catalog/Books?$select=title,price,rating

Response (ONLY what you asked for):
{
  "title": "Harry Potter...",
  "price": 12.99,
  "rating": 4.8             ← Just 3 fields! Clean and fast!
}
```

---

### $select Syntax

```
$select=field1,field2,field3
```

**Rules:**
- Separate field names with commas (no spaces after comma)
- Field names are case-sensitive (must match exactly)
- You can include association fields (they show as IDs unless you also $expand)

**Examples:**
```
// Only title and price
GET /catalog/Books?$select=title,price

// Only name and email of customers
GET /sales/Customers?$select=customerName,email

// ID is usually included automatically (but you can be explicit)
GET /catalog/Books?$select=ID,title,price
```

---

### $select with Associations

When you `$select` an association field without `$expand`, you get just the foreign key:

```
// Without $expand — just the ID reference
GET /catalog/Books?$select=title,author

Response:
{
  "title": "Harry Potter...",
  "author_ID": "a1000001-..."    ← Just the FK
}
```

```
// With $expand — get the full related object
GET /catalog/Books?$select=title&$expand=author($select=name)

Response:
{
  "title": "Harry Potter...",
  "author": {
    "name": "J.K. Rowling"      ← Full details!
  }
}
```

---

### Why Use $select?

| Benefit | Explanation |
|---------|-------------|
| **Faster responses** | Less data to transfer over network |
| **Less bandwidth** | Mobile apps need less data |
| **Cleaner code** | Frontend gets exactly what it needs |
| **Better performance** | Database reads fewer columns |
| **Security** | Ensures sensitive fields aren't accidentally exposed |

**Rule of thumb:** In production apps, ALWAYS use `$select` — never fetch all fields when you only need a few.

---

### $orderby — Sort Your Results

Like ORDER BY in SQL — arrange results in a specific order.

**Syntax:**
```
$orderby=field              (ascending — default)
$orderby=field asc          (ascending — explicit)
$orderby=field desc         (descending — highest first)
$orderby=field1,field2      (sort by field1, then field2 for ties)
```

---

### $orderby Examples

```
// Cheapest books first (ascending is default)
GET /catalog/Books?$orderby=price

// Most expensive books first
GET /catalog/Books?$orderby=price desc

// Highest rated books first
GET /catalog/Books?$orderby=rating desc

// Alphabetical by title
GET /catalog/Books?$orderby=title

// Sort by genre, then by title within each genre
GET /catalog/Books?$orderby=genre,title

// Newest orders first, then by amount descending
GET /sales/Orders?$orderby=orderDate desc,grossAmount desc

// Sort by multiple criteria with mixed directions
GET /catalog/Books?$orderby=genre asc,price desc
// First groups by genre A-Z, then within each genre: most expensive first
```

---

### Visualizing Multi-Column Sort

```
$orderby=genre,price desc

Result:
┌──────────────┬────────────────────────────────────┬────────┐
│ Genre        │ Title                              │ Price  │
├──────────────┼────────────────────────────────────┼────────┤
│ Dystopian    │ 1984                               │  9.99  │ ← Genre: D
│ Fantasy      │ Harry Potter...                    │ 12.99  │ ← Genre: F (higher price first)
│ Fantasy      │ The Hobbit                         │  8.99  │ ← Genre: F (lower price)
│ Fiction      │ The Alchemist                      │ 11.50  │ ← Genre: F(iction)
│ Mystery      │ Murder on the Orient Express       │ 10.99  │ ← Genre: M
└──────────────┴────────────────────────────────────┴────────┘
  ↑ Sorted alphabetically A→Z                        ↑ Within genre: desc
```

---

### $top and $skip — Pagination

**The problem:** You have 10,000 products. You can't send ALL of them to the mobile app at once!

**Solution:** Send them in pages.

```
$top = How many records to return (page size)
$skip = How many records to skip (which page)
```

---

### Pagination — The Book Analogy

```
Your data = A book with 100 pages
$top = 10  → Show 10 items per page
$skip      → Which page to turn to

Page 1: $top=10&$skip=0    → Items 1-10    (skip nothing)
Page 2: $top=10&$skip=10   → Items 11-20   (skip first 10)
Page 3: $top=10&$skip=20   → Items 21-30   (skip first 20)
Page 4: $top=10&$skip=30   → Items 31-40   (skip first 30)
...
Page N: $top=10&$skip=(N-1)*10
```

---

### Pagination Examples

```
// First 5 books
GET /catalog/Books?$top=5

// First 5 books, sorted by price
GET /catalog/Books?$orderby=price&$top=5

// Page 1: First 10 items
GET /catalog/Books?$top=10&$skip=0

// Page 2: Next 10 items
GET /catalog/Books?$top=10&$skip=10

// Page 3: Next 10 items
GET /catalog/Books?$top=10&$skip=20

// Top 3 most expensive books
GET /catalog/Books?$orderby=price desc&$top=3

// Second page of results, 20 per page, sorted by title
GET /catalog/Books?$orderby=title&$top=20&$skip=20
```

---

### $top Without $skip — "Give Me the First N"

Sometimes you just want the top records:

```
// Top 5 highest-rated books
GET /catalog/Books?$orderby=rating desc&$top=5

// 3 cheapest products
GET /sales/Products?$orderby=price&$top=3

// Latest 10 orders
GET /sales/Orders?$orderby=orderDate desc&$top=10

// Single newest order
GET /sales/Orders?$orderby=orderDate desc&$top=1
```

---

### Pagination Formula

For building pagination in your frontend app:

```
Page Size = 10 (how many items per page)
Page Number = user clicks "Page 3"

URL = /catalog/Books?$top={PageSize}&$skip={(PageNumber - 1) × PageSize}

Examples:
  Page 1 → $top=10&$skip=0     (0 × 10 = 0)
  Page 2 → $top=10&$skip=10    (1 × 10 = 10)
  Page 3 → $top=10&$skip=20    (2 × 10 = 20)
  Page 4 → $top=10&$skip=30    (3 × 10 = 30)
```

---

### $count — How Many Records Exist?

Two ways to use count:

#### Way 1: Just the count (a number)

```
GET /catalog/Books/$count

Response: 5
```

Returns ONLY the number — not the actual records. Useful for showing "Total: 5 books" in the UI.

---

#### Way 2: Include count WITH results

```
GET /catalog/Books?$count=true

Response:
{
  "@odata.count": 5,        ← Total matching records
  "value": [
    { "title": "Harry Potter...", ... },
    { "title": "1984", ... },
    ...
  ]
}
```

---

#### Why $count Matters for Pagination

When you paginate, you need to know the TOTAL to show "Page 3 of 12":

```
GET /catalog/Books?$count=true&$top=10&$skip=20

Response:
{
  "@odata.count": 120,     ← Total: 120 books
  "value": [...]           ← This page: 10 books (items 21-30)
}

UI shows: "Showing 21-30 of 120 results | Page 3 of 12"
```

---

#### $count with $filter

Count respects your filters — it counts MATCHING records, not all records:

```
// How many Fantasy books exist?
GET /catalog/Books/$count?$filter=genre eq 'Fantasy'
Response: 3

// How many active products with low stock?
GET /sales/Products/$count?$filter=isActive eq true and stock lt 10
Response: 7
```

---

### Quick Exercise: $select, $orderby, $top, $skip, $count

Build the URL for each requirement:

1. Get only the title and price of all books → _____
2. Sort products by stock (lowest first) → _____
3. Get the 3 newest orders → _____
4. Get page 4 with 5 items per page → _____
5. How many products cost more than $100? → _____
6. Get first 10 books sorted by rating desc, include total count → _____

<details>
<summary>Answers</summary>

1. `GET /catalog/Books?$select=title,price`
2. `GET /sales/Products?$orderby=stock`
3. `GET /sales/Orders?$orderby=orderDate desc&$top=3`
4. `GET /catalog/Books?$top=5&$skip=15`
5. `GET /sales/Products/$count?$filter=price gt 100`
6. `GET /catalog/Books?$orderby=rating desc&$top=10&$count=true`

</details>

---

## Session 3: Hands-on — Practice Filtering, Sorting & Pagination (12:00 - 13:00)

### Setup: The Data We're Working With

Make sure your bookshop project is running with `cds watch`. You should have these services:
- `/catalog` — CatalogService (Books, Authors)
- `/admin` — AdminService (full access)

If you also have the EPM model:
- `/master-data` — MasterDataService (Products, Suppliers, Categories)
- `/sales` — SalesService (Orders, Customers)

---

### Exercise 1: Basic Filtering (10 minutes)

Open your browser or REST client. Test each query and verify the results:

```http
### 1.1 Get all Fantasy books
GET http://localhost:4004/catalog/Books?$filter=genre eq 'Fantasy'

### 1.2 Get books cheaper than $12
GET http://localhost:4004/catalog/Books?$filter=price lt 12

### 1.3 Get books with rating 4.5 or higher
GET http://localhost:4004/catalog/Books?$filter=rating ge 4.5

### 1.4 Get books NOT in Mystery genre
GET http://localhost:4004/catalog/Books?$filter=genre ne 'Mystery'

### 1.5 Get books published after 1990
GET http://localhost:4004/catalog/Books?$filter=publishDate gt 1990-01-01
```

**Record your observations:**
- How many Fantasy books? _____
- Cheapest book under $12? _____
- Highest-rated book? _____

---

### Exercise 2: String Functions (10 minutes)

```http
### 2.1 Books with "the" in the title (case-sensitive!)
GET http://localhost:4004/catalog/Books?$filter=contains(title,'the')

### 2.2 Books with "The" in the title (capital T)
GET http://localhost:4004/catalog/Books?$filter=contains(title,'The')

### 2.3 Case-insensitive search for "the"
GET http://localhost:4004/catalog/Books?$filter=contains(tolower(title),'the')

### 2.4 Authors whose name starts with "J"
GET http://localhost:4004/catalog/Authors?$filter=startswith(name,'J')

### 2.5 Books with ISBN starting with "978"
GET http://localhost:4004/catalog/Books?$filter=startswith(isbn,'978')
```

**Notice:** `contains(title,'the')` and `contains(title,'The')` return different results! OData string functions are case-sensitive by default. Use `tolower()` for case-insensitive searches.

---

### Exercise 3: Combined Filters (10 minutes)

```http
### 3.1 Fantasy books under $15
GET http://localhost:4004/catalog/Books?$filter=genre eq 'Fantasy' and price lt 15

### 3.2 Books that are Fantasy OR Mystery
GET http://localhost:4004/catalog/Books?$filter=genre eq 'Fantasy' or genre eq 'Mystery'

### 3.3 High-rated (4.5+) AND affordable (under $13)
GET http://localhost:4004/catalog/Books?$filter=rating ge 4.5 and price lt 13

### 3.4 Books with "Harry" in title OR rating above 4.7
GET http://localhost:4004/catalog/Books?$filter=contains(title,'Harry') or rating gt 4.7

### 3.5 Books in stock (stock > 0) that are in Fantasy or Fiction genre
GET http://localhost:4004/catalog/Books?$filter=stock gt 0 and (genre eq 'Fantasy' or genre eq 'Fiction')
```

---

### Exercise 4: Sorting & Pagination (10 minutes)

```http
### 4.1 All books sorted by title A-Z
GET http://localhost:4004/catalog/Books?$orderby=title

### 4.2 Most expensive books first
GET http://localhost:4004/catalog/Books?$orderby=price desc

### 4.3 Top 3 highest rated books
GET http://localhost:4004/catalog/Books?$orderby=rating desc&$top=3

### 4.4 Cheapest 2 books
GET http://localhost:4004/catalog/Books?$orderby=price&$top=2

### 4.5 Page 1 (2 items per page)
GET http://localhost:4004/catalog/Books?$orderby=title&$top=2&$skip=0

### 4.6 Page 2 (2 items per page)
GET http://localhost:4004/catalog/Books?$orderby=title&$top=2&$skip=2

### 4.7 Page 3 (2 items per page)
GET http://localhost:4004/catalog/Books?$orderby=title&$top=2&$skip=4

### 4.8 Get total count with first page
GET http://localhost:4004/catalog/Books?$orderby=title&$top=2&$count=true
```

---

### Exercise 5: $select — Lean Responses (10 minutes)

```http
### 5.1 Only titles
GET http://localhost:4004/catalog/Books?$select=title

### 5.2 Title and price only
GET http://localhost:4004/catalog/Books?$select=title,price

### 5.3 Title, price, and rating — sorted by price
GET http://localhost:4004/catalog/Books?$select=title,price,rating&$orderby=price

### 5.4 Author names only
GET http://localhost:4004/catalog/Authors?$select=name

### 5.5 Combine $select with $filter
GET http://localhost:4004/catalog/Books?$select=title,price&$filter=price lt 12
```

Compare the response SIZE between 5.1 and a request without `$select`. Notice how much smaller it is!

---

### Exercise 6: Challenge Combinations (10 minutes)

Build these complex queries (combine everything learned):

```http
### 6.1 Top 3 cheapest Fantasy books — show only title and price
GET http://localhost:4004/catalog/Books?$filter=genre eq 'Fantasy'&$orderby=price&$top=3&$select=title,price

### 6.2 All books sorted by genre then price desc, page 1 of 2 items, with count
GET http://localhost:4004/catalog/Books?$orderby=genre,price desc&$top=2&$skip=0&$count=true

### 6.3 Books with "the" in title, rated 4.0+, sorted by rating desc, only title and rating
GET http://localhost:4004/catalog/Books?$filter=contains(tolower(title),'the') and rating ge 4.0&$orderby=rating desc&$select=title,rating
```

---

## Session 4: $expand — Navigating Relationships (13:45 - 14:45)

### Why $expand Exists

Your entities are related — Books have Authors, Orders have Items. Without `$expand`, you only see the foreign key ID:

```
WITHOUT $expand:
GET /catalog/Books

{
  "title": "Harry Potter...",
  "author_ID": "a1000001-0000-0000-0000-000000000001"   ← Just an ID!
  // Who is this author? You have to make ANOTHER request to find out!
}
```

```
WITH $expand:
GET /catalog/Books?$expand=author

{
  "title": "Harry Potter...",
  "author": {                                            ← Full details!
    "ID": "a1000001-0000-0000-0000-000000000001",
    "name": "J.K. Rowling",
    "country": "United Kingdom"
  }
}
```

**Without $expand:** 2 requests needed (one for book, one for author)
**With $expand:** 1 request gets everything (like a SQL JOIN)

---

### $expand — The Family Tree Analogy

Think of `$expand` as opening folders in a file explorer:

```
📁 Book: "Harry Potter"
   ├── title: "Harry Potter..."
   ├── price: 12.99
   └── 📁 author          ← CLOSED (just shows author_ID)
       
After $expand=author:

📁 Book: "Harry Potter"
   ├── title: "Harry Potter..."
   ├── price: 12.99
   └── 📂 author          ← OPENED! (shows all author fields)
       ├── name: "J.K. Rowling"
       └── country: "United Kingdom"
```

---

### Basic $expand Syntax

```
// Expand a single association
$expand=author

// Expand multiple associations (comma-separated)
$expand=author,category

// Expand with nested options (limit what comes back)
$expand=author($select=name)
```

---

### $expand with One-to-One (Association)

A Book has ONE Author:

```
GET /catalog/Books?$expand=author

Response:
{
  "value": [
    {
      "ID": "b1000001-...",
      "title": "Harry Potter and the Philosopher's Stone",
      "price": 12.99,
      "author": {                    ← ONE object (not an array)
        "ID": "a1000001-...",
        "name": "J.K. Rowling",
        "country": "United Kingdom"
      }
    },
    {
      "ID": "b1000002-...",
      "title": "1984",
      "price": 9.99,
      "author": {
        "ID": "a1000002-...",
        "name": "George Orwell",
        "country": "United Kingdom"
      }
    }
  ]
}
```

---

### $expand with One-to-Many (Back Navigation)

An Author has MANY Books:

```
GET /catalog/Authors?$expand=books

Response:
{
  "value": [
    {
      "ID": "a1000001-...",
      "name": "J.K. Rowling",
      "country": "United Kingdom",
      "books": [                     ← ARRAY of books (one-to-many!)
        {
          "ID": "b1000001-...",
          "title": "Harry Potter and the Philosopher's Stone",
          "price": 12.99
        },
        {
          "ID": "b1000007-...",
          "title": "Harry Potter and the Chamber of Secrets",
          "price": 11.99
        }
      ]
    },
    {
      "ID": "a1000002-...",
      "name": "George Orwell",
      "books": [
        {
          "ID": "b1000002-...",
          "title": "1984",
          "price": 9.99
        }
      ]
    }
  ]
}
```

**Key difference:**
- Many-to-one (Book → Author): `"author": { ... }` — single object
- One-to-many (Author → Books): `"books": [ {...}, {...} ]` — array

---

### Nested $expand Options — Control What Gets Expanded

You can add query options INSIDE `$expand` using parentheses:

```
$expand=association($select=..., $filter=..., $orderby=..., $top=...)
```

---

### $expand with $select — Choose Fields from Related Entity

```
// Only get author's name (not all author fields)
GET /catalog/Books?$expand=author($select=name)

Response:
{
  "title": "Harry Potter...",
  "author": {
    "name": "J.K. Rowling"         ← Only name! Clean and lean.
  }
}
```

```
// Get author's name and country
GET /catalog/Books?$expand=author($select=name,country)
```

---

### $expand with $filter — Filter the Related Records

```
// Get authors, but only expand their Fantasy books
GET /catalog/Authors?$expand=books($filter=genre eq 'Fantasy')

Response:
{
  "name": "J.K. Rowling",
  "books": [
    { "title": "Harry Potter...", "genre": "Fantasy" }
    // Only Fantasy books shown! Others are filtered out.
  ]
}
```

---

### $expand with $orderby and $top — Sort and Limit Related Records

```
// Get authors with their top 3 books by rating
GET /catalog/Authors?$expand=books($orderby=rating desc;$top=3)

// Get customers with their 5 most recent orders
GET /sales/Customers?$expand=orders($orderby=orderDate desc;$top=5)
```

**Note:** Inside `$expand()`, use `;` (semicolons) to separate nested options, NOT `&`:
```
✅ $expand=books($orderby=rating desc;$top=3;$select=title,rating)
❌ $expand=books($orderby=rating desc&$top=3&$select=title,rating)
```

---

### Multiple $expand — Expand Several Associations

```
// Expand both author AND category
GET /sales/Products?$expand=supplier,category

Response:
{
  "productName": "Laptop Pro",
  "supplier": {
    "supplierName": "Tech Corp",
    "email": "sales@techcorp.com"
  },
  "category": {
    "categoryName": "Electronics"
  }
}
```

---

### Deep (Nested) $expand — Multiple Levels

You can expand within an expansion:

```
// Orders → Items → Product details
GET /sales/Orders?$expand=items($expand=product)

Response:
{
  "orderNumber": "SO-001",
  "orderDate": "2026-05-15",
  "items": [                               ← Level 1: Order Items
    {
      "quantity": 2,
      "unitPrice": 499.99,
      "product": {                         ← Level 2: Product details!
        "productName": "Laptop Pro",
        "price": 499.99
      }
    },
    {
      "quantity": 1,
      "unitPrice": 29.99,
      "product": {
        "productName": "USB Cable",
        "price": 29.99
      }
    }
  ]
}
```

**Visualization:**
```
Order "SO-001"
├── orderNumber: "SO-001"
├── orderDate: "2026-05-15"
└── 📂 items (expanded — level 1)
    ├── Item 1
    │   ├── quantity: 2
    │   ├── unitPrice: 499.99
    │   └── 📂 product (expanded — level 2!)
    │       ├── productName: "Laptop Pro"
    │       └── price: 499.99
    └── Item 2
        ├── quantity: 1
        ├── unitPrice: 29.99
        └── 📂 product (expanded — level 2!)
            ├── productName: "USB Cable"
            └── price: 29.99
```

---

### Deep $expand with Filtering at Each Level

```
// Orders with only "Shipped" status,
// expand items, but only items with quantity > 1,
// expand the product showing only name and price
GET /sales/Orders?$filter=status eq 'Shipped'
                 &$expand=items(
                    $filter=quantity gt 1;
                    $expand=product($select=productName,price)
                  )
```

---

### $expand with Compositions vs Associations

Remember from Day 13:
- **Association** = independent entities (Book ↔ Author)
- **Composition** = parent owns children (Order ↔ OrderItems)

Both work with `$expand`, but compositions have extra behavior:

```
// With Composition: Deep operations work!
// Creating an order with items in ONE request:
POST /sales/Orders
{
  "orderNumber": "SO-999",
  "customer_ID": "c123...",
  "items": [                    ← Can include children directly!
    { "product_ID": "p1...", "quantity": 3 },
    { "product_ID": "p2...", "quantity": 1 }
  ]
}

// Deleting an order also deletes its items (cascade!)
DELETE /sales/Orders(orderId)   ← Items auto-deleted too!
```

This "deep" behavior only works with **Composition**, not Association.

---

### $expand Performance Tip

```
⚠️  Be careful with deep $expand on large datasets!

// This could return THOUSANDS of records:
GET /sales/Customers?$expand=orders($expand=items($expand=product))

// Better: combine $expand with $top and $select
GET /sales/Customers?$top=10
    &$expand=orders(
      $top=5;
      $orderby=orderDate desc;
      $select=orderNumber,orderDate,grossAmount;
      $expand=items($top=3;$select=quantity,unitPrice)
    )
```

**Rules for $expand in production:**
1. Always `$select` within your expansion — don't fetch all fields
2. Use `$top` on one-to-many expansions — limit how many children load
3. Avoid more than 3 levels deep — it gets slow
4. Use views (from Day 14) instead of deep $expand for reporting

---

### $expand Practice

Build the URL for each requirement:

1. Get all books with their author name → _____
2. Get all authors with their list of books (title only) → _____
3. Get orders with customer details and order items → _____
4. Get products with supplier name and category name → _____
5. Get the 5 most recent orders with items and product names → _____

<details>
<summary>Answers</summary>

1. `GET /catalog/Books?$expand=author($select=name)`
2. `GET /catalog/Authors?$expand=books($select=title)`
3. `GET /sales/Orders?$expand=customer,items`
4. `GET /sales/Products?$expand=supplier($select=supplierName),category($select=categoryName)`
5. `GET /sales/Orders?$orderby=orderDate desc&$top=5&$expand=items($expand=product($select=productName))`

</details>

---

## Session 5: $search, Combining Queries & Real-World Patterns (15:00 - 16:00)

### $search — Free-Text Search

`$search` is like a Google search for your data. It searches across MULTIPLE text fields at once:

```
// Search for "Harry" across all searchable text fields
GET /catalog/Books?$search=Harry

// This might match:
// - title: "Harry Potter..."
// - (any other text field that matches "Harry")
```

**How $search differs from $filter:**

| Feature | $filter | $search |
|---------|---------|---------|
| Target | ONE specific field | ALL text fields |
| Precision | Exact conditions | Fuzzy/free-text |
| Use case | "Price above 10" | "Find anything related to 'Harry'" |
| Operators | `eq`, `gt`, `contains()`, etc. | Just a search term |

---

### $search Syntax

```
// Single word
$search=Harry

// Multiple words (space = AND by default in CAP)
$search=Harry Potter

// Phrase (exact match)
$search="Harry Potter"
```

---

### $search Limitations in CAP

**Important for freshers:** `$search` support in CAP depends on the database:
- **SQLite (development):** Limited support — searches across annotated fields
- **SAP HANA (production):** Full fuzzy search support

For development, `$filter` with `contains()` is more reliable:
```
// More reliable than $search in SQLite:
$filter=contains(title,'Harry') or contains(genre,'Fantasy')
```

---

### Combining All Query Options — The Master Pattern

You can combine ANY query options together. Here's the order:

```
GET /service/Entity
  ? $filter = <conditions>          ← WHICH records
  & $search = <text>                ← Free-text filter
  & $select = <fields>             ← WHAT fields to return
  & $expand = <associations>       ← Related data to include
  & $orderby = <sort>             ← HOW to sort
  & $top = <number>               ← HOW MANY to return
  & $skip = <number>              ← WHERE to start
  & $count = true                 ← Include total count
```

**Order in the URL doesn't matter** — you can put them in any sequence!

---

### Real-World Query Patterns

#### Pattern 1: List Page with Search and Pagination

A typical product list page needs: filter + sort + paginate + count.

**Business requirement:** "Show me active products, sorted by name, 20 per page, with total count for pagination."

```
GET /sales/Products
  ?$filter=isActive eq true
  &$select=ID,productName,price,stock
  &$orderby=productName
  &$top=20
  &$skip=0
  &$count=true
```

**Response:**
```json
{
  "@odata.count": 87,
  "value": [
    { "ID": "...", "productName": "Apple iPhone", "price": 999.99, "stock": 45 },
    { "ID": "...", "productName": "Dell Monitor", "price": 349.99, "stock": 23 },
    // ... 18 more items
  ]
}
```

---

#### Pattern 2: Detail Page with Related Data

When a user clicks on a specific record, fetch all details with related entities.

**Business requirement:** "Show order details with customer info and line items (including product names)."

```
GET /sales/Orders(order-uuid-here)
  ?$select=orderNumber,orderDate,grossAmount,netAmount,status
  &$expand=
    customer($select=customerName,email,phone),
    items(
      $select=quantity,unitPrice,netAmount;
      $expand=product($select=productName,price)
    )
```

---

#### Pattern 3: Dashboard / Analytics Query

A dashboard shows summarized data — top items, counts, filtered views.

**Business requirement:** "Show top 5 best-selling products this year."

```
GET /analytics/ProductCatalog
  ?$orderby=rating desc
  &$top=5
  &$select=productName,price,rating,supplierName
```

---

#### Pattern 4: Search with Autocomplete

For a search box that suggests results as you type:

**Business requirement:** "As user types 'har', show matching books (max 5 suggestions)."

```
GET /catalog/Books
  ?$filter=contains(tolower(title),'har')
  &$select=ID,title
  &$top=5
  &$orderby=title
```

---

#### Pattern 5: Filtered Report with Export

**Business requirement:** "Get all orders from January 2026 that are delivered, include customer name."

```
GET /sales/Orders
  ?$filter=orderDate ge 2026-01-01 and orderDate le 2026-01-31 and status eq 'Delivered'
  &$select=orderNumber,orderDate,grossAmount,status
  &$expand=customer($select=customerName,city)
  &$orderby=orderDate desc
```

---

#### Pattern 6: Low Stock Alert

**Business requirement:** "Find products where stock is below minimum, show supplier contact info."

```
GET /sales/Products
  ?$filter=stock lt minStock and isActive eq true
  &$select=productName,stock,minStock
  &$expand=supplier($select=supplierName,email,phone)
  &$orderby=stock
```

---

### Combining Query Options — Order of Execution

When you combine multiple options, OData processes them in this order:

```
Step 1: $filter & $search    → Select matching records
Step 2: $orderby             → Sort the matching records
Step 3: $skip                → Skip the first N
Step 4: $top                 → Take only N from what remains
Step 5: $select              → Return only chosen fields
Step 6: $expand              → Include related entities
Step 7: $count               → Count matching (before $skip/$top!)

⚠️ Important: $count counts records AFTER $filter but BEFORE $top/$skip!
```

**This means:**
```
GET /catalog/Books?$filter=genre eq 'Fantasy'&$top=3&$count=true

$count = total Fantasy books (say 8)
$top = 3 books returned in this response
Response: { "@odata.count": 8, "value": [3 books] }
```

The count shows 8 (total matching), even though only 3 are returned. This is how pagination UIs know the total!

---

### Common Mistakes and How to Fix Them

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Missing quotes on strings | `$filter=genre eq Fantasy` | `$filter=genre eq 'Fantasy'` |
| Using `=` instead of `eq` | `$filter=price=10` | `$filter=price eq 10` |
| Space in `$select` | `$select=title, price` | `$select=title,price` |
| Wrong separator in $expand | `$expand=a($top=3&$select=x)` | `$expand=a($top=3;$select=x)` |
| Missing `$` prefix | `filter=price gt 10` | `$filter=price gt 10` |
| Using `>` instead of `gt` | `$filter=price > 10` | `$filter=price gt 10` |
| Quotes on numbers | `$filter=price eq '10'` | `$filter=price eq 10` |
| Quotes on booleans | `$filter=isActive eq 'true'` | `$filter=isActive eq true` |

---

### Query Options Quick Reference Card

```
┌────────────────────────────────────────────────────────────────┐
│              ODATA QUERY OPTIONS — QUICK REFERENCE               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  $select=field1,field2              Choose fields              │
│  $filter=field op value             Filter records             │
│  $orderby=field [asc|desc]          Sort results               │
│  $top=N                             Limit to N records         │
│  $skip=N                            Skip first N              │
│  $count=true                        Include total count        │
│  $expand=assoc                      Include related data       │
│  $search=text                       Free-text search           │
│                                                                │
│  FILTER OPERATORS:                                             │
│  eq ne gt lt ge le | and or not | contains startswith endswith │
│                                                                │
│  EXPAND NESTING (use semicolons!):                             │
│  $expand=assoc($select=x;$filter=y;$orderby=z;$top=n)         │
│                                                                │
│  PAGINATION FORMULA:                                           │
│  Page N: $top=pageSize & $skip=(N-1)*pageSize & $count=true    │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Session 6: Hands-on — Build Complex Queries & Document APIs (16:00 - 16:30)

### Exercise: Build a REST Client Test File

Create a comprehensive test file that documents all API endpoints with examples.

**File: `tests/odata-queries.http`** (VS Code REST Client format)

```http
### ═══════════════════════════════════════════════════════
### ODATA QUERY OPERATIONS — COMPLETE TEST SUITE
### ═══════════════════════════════════════════════════════

@base = http://localhost:4004


### ═══════ $filter EXAMPLES ═══════

### Filter: Exact string match
GET {{base}}/catalog/Books?$filter=genre eq 'Fantasy'

### Filter: Number comparison
GET {{base}}/catalog/Books?$filter=price gt 10

### Filter: Boolean
GET {{base}}/catalog/Books?$filter=stock gt 0

### Filter: Date comparison
GET {{base}}/catalog/Books?$filter=publishDate gt 1990-01-01

### Filter: String contains (case-sensitive)
GET {{base}}/catalog/Books?$filter=contains(title,'Potter')

### Filter: String contains (case-insensitive)
GET {{base}}/catalog/Books?$filter=contains(tolower(title),'potter')

### Filter: Starts with
GET {{base}}/catalog/Authors?$filter=startswith(name,'J')

### Filter: Combined with AND
GET {{base}}/catalog/Books?$filter=price gt 10 and rating ge 4.5

### Filter: Combined with OR
GET {{base}}/catalog/Books?$filter=genre eq 'Fantasy' or genre eq 'Mystery'

### Filter: Complex with parentheses
GET {{base}}/catalog/Books?$filter=(genre eq 'Fantasy' or genre eq 'Fiction') and price lt 15

### Filter: Null check
GET {{base}}/catalog/Books?$filter=isbn ne null


### ═══════ $select EXAMPLES ═══════

### Select: Two fields only
GET {{base}}/catalog/Books?$select=title,price

### Select: Multiple fields
GET {{base}}/catalog/Books?$select=ID,title,genre,price,rating

### Select: Combined with filter
GET {{base}}/catalog/Books?$select=title,price&$filter=genre eq 'Fantasy'


### ═══════ $orderby EXAMPLES ═══════

### Sort: Ascending (default)
GET {{base}}/catalog/Books?$orderby=price

### Sort: Descending
GET {{base}}/catalog/Books?$orderby=price desc

### Sort: Multiple columns
GET {{base}}/catalog/Books?$orderby=genre,price desc

### Sort: Combined with select
GET {{base}}/catalog/Books?$select=title,price,rating&$orderby=rating desc


### ═══════ $top & $skip (PAGINATION) ═══════

### Top: First 3 records
GET {{base}}/catalog/Books?$top=3

### Top 3 highest rated
GET {{base}}/catalog/Books?$orderby=rating desc&$top=3

### Pagination: Page 1 (3 per page)
GET {{base}}/catalog/Books?$orderby=title&$top=3&$skip=0

### Pagination: Page 2
GET {{base}}/catalog/Books?$orderby=title&$top=3&$skip=3

### Pagination: With count
GET {{base}}/catalog/Books?$orderby=title&$top=3&$skip=0&$count=true


### ═══════ $count EXAMPLES ═══════

### Count: Total books
GET {{base}}/catalog/Books/$count

### Count: With filter (how many Fantasy books?)
GET {{base}}/catalog/Books/$count?$filter=genre eq 'Fantasy'

### Count: Inline with results
GET {{base}}/catalog/Books?$count=true


### ═══════ $expand EXAMPLES ═══════

### Expand: Books with author
GET {{base}}/catalog/Books?$expand=author

### Expand: Authors with their books
GET {{base}}/catalog/Authors?$expand=books

### Expand: With $select inside
GET {{base}}/catalog/Books?$expand=author($select=name,country)

### Expand: Multiple associations
# (if your model has supplier and category)
# GET {{base}}/sales/Products?$expand=supplier,category

### Expand: Nested (deep)
# GET {{base}}/sales/Orders?$expand=items($expand=product)


### ═══════ COMBINED QUERIES (Real-World) ═══════

### Product List Page: filtered, sorted, paginated, with count
GET {{base}}/catalog/Books?$filter=rating ge 4.0&$select=title,price,rating,genre&$orderby=rating desc&$top=10&$skip=0&$count=true

### Detail View: single record with expanded relations
GET {{base}}/catalog/Books(b1000001-0000-0000-0000-000000000001)?$expand=author($select=name,country)

### Search-style: find by title keyword, return minimal data
GET {{base}}/catalog/Books?$filter=contains(tolower(title),'the')&$select=ID,title&$top=5&$orderby=title

### Report: All books by UK authors, sorted by rating
GET {{base}}/catalog/Authors?$filter=country eq 'United Kingdom'&$expand=books($orderby=rating desc;$select=title,rating)

### Dashboard: Highest rated book per result
GET {{base}}/catalog/Books?$orderby=rating desc&$top=1&$select=title,rating,genre&$expand=author($select=name)
```

---

### API Documentation Template

For each endpoint your application exposes, document it in this format:

```markdown
## GET /catalog/Books

**Description:** Retrieve a list of books from the catalog.

**Supported Query Options:**
- $filter: genre, price, rating, stock, publishDate, title (contains)
- $select: ID, title, genre, price, stock, rating, publishDate, isbn
- $expand: author
- $orderby: any field
- $top / $skip: supported

**Example Requests:**

| Use Case | URL |
|----------|-----|
| All books | `/catalog/Books` |
| Fantasy only | `/catalog/Books?$filter=genre eq 'Fantasy'` |
| Cheap books | `/catalog/Books?$filter=price lt 10` |
| With author | `/catalog/Books?$expand=author` |
| Paginated | `/catalog/Books?$top=10&$skip=0&$count=true` |

**Response Shape:**
```json
{
  "@odata.count": 5,
  "value": [
    {
      "ID": "uuid",
      "title": "string",
      "genre": "string",
      "price": 0.00,
      "stock": 0,
      "rating": 0.0,
      "publishDate": "YYYY-MM-DD",
      "isbn": "string",
      "author": { "name": "string", "country": "string" }
    }
  ]
}
```
```

---

## Assessment: MCQ — 15 Questions on OData Queries (16:30 - 16:45)

**Q1.** Which OData operator means "greater than or equal"?
- A) `gte`
- B) `>=`
- C) `ge`
- D) `gteq`

---

**Q2.** To find books with "magic" in the title, the correct filter is:
- A) `$filter=title like '%magic%'`
- B) `$filter=contains(title,'magic')`
- C) `$filter=title contains 'magic'`
- D) `$filter=title.includes('magic')`

---

**Q3.** What does `$top=10&$skip=30` return?
- A) Records 1-10
- B) Records 21-30
- C) Records 31-40
- D) Records 30-40

---

**Q4.** Which separator is used between options INSIDE `$expand()`?
- A) Comma (`,`)
- B) Ampersand (`&`)
- C) Semicolon (`;`)
- D) Pipe (`|`)

---

**Q5.** What does `$count=true` count — the filtered total or the $top-limited result?
- A) The number of records returned in this response
- B) The total matching records (before $top/$skip)
- C) All records in the table regardless of filters
- D) The number of fields in each record

---

**Q6.** String values in `$filter` must be wrapped in:
- A) Double quotes (`"value"`)
- B) Single quotes (`'value'`)
- C) Backticks (`` `value` ``)
- D) No quotes needed

---

**Q7.** To sort by price ascending then by name descending, you write:
- A) `$orderby=price ASC, name DESC`
- B) `$orderby=price asc,name desc`
- C) `$sort=price,name desc`
- D) `$orderby=price,-name`

---

**Q8.** `GET /catalog/Books?$expand=author($select=name)` does what?
- A) Filters books by author name
- B) Includes author data but only the name field
- C) Selects books that have an author named "name"
- D) Creates an author with the given name

---

**Q9.** Which returns only the count as a plain number?
- A) `GET /catalog/Books?$count=true`
- B) `GET /catalog/Books/$count`
- C) `GET /catalog/Books?count`
- D) `GET /catalog/Books/count()`

---

**Q10.** To do a case-insensitive search for "harry" in the title:
- A) `$filter=contains(title,'harry')`
- B) `$filter=containsIgnoreCase(title,'harry')`
- C) `$filter=contains(tolower(title),'harry')`
- D) `$filter=title ilike 'harry'`

---

**Q11.** TRUE or FALSE: `$select=title,price` returns all records but only those two fields.
- A) TRUE
- B) FALSE

---

**Q12.** What does `$filter=email ne null` return?
- A) Records where email is empty string
- B) Records where email field exists and is not null
- C) All records (null check doesn't work)
- D) Records where email equals the text "null"

---

**Q13.** Page 5 with 20 items per page requires:
- A) `$top=20&$skip=80`
- B) `$top=20&$skip=100`
- C) `$top=20&$skip=60`
- D) `$top=5&$skip=20`

---

**Q14.** Which is valid for expanding multiple associations?
- A) `$expand=author+category`
- B) `$expand=author,category`
- C) `$expand=[author,category]`
- D) `$expand=author&$expand=category`

---

**Q15.** What's the OData V4 way to filter for books published in 2020?
- A) `$filter=publishDate.year == 2020`
- B) `$filter=year(publishDate) eq 2020`
- C) `$filter=YEAR(publishDate) = 2020`
- D) `$filter=publishDate between '2020-01-01' and '2020-12-31'`

---

### Answer Key

<details>
<summary>Click to reveal answers</summary>

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **C** | `ge` = greater-or-equal (OData uses 2-letter abbreviations) |
| 2 | **B** | `contains(field,'text')` is the OData string function |
| 3 | **C** | Skip 30 + take 10 = records 31-40 |
| 4 | **C** | Inside `$expand()`, options are separated by semicolons |
| 5 | **B** | `$count` reflects the total BEFORE pagination ($top/$skip) |
| 6 | **B** | OData uses single quotes for string literals |
| 7 | **B** | `$orderby=field1 asc,field2 desc` (comma-separated, lowercase asc/desc) |
| 8 | **B** | Expands the author association but returns only the name field |
| 9 | **B** | `/$count` (as path segment) returns just the number |
| 10 | **C** | Use `tolower()` to convert before comparing |
| 11 | **A** | `$select` affects which fields are returned, not which records |
| 12 | **B** | `ne null` returns records where the field has a value |
| 13 | **A** | Page 5: skip = (5-1) × 20 = 80 |
| 14 | **B** | Comma-separated list of association names |
| 15 | **B** | `year()` function extracts year for comparison |

**Scoring:**
- 13-15: Excellent! You're an OData query master!
- 10-12: Good. Review string functions and $expand nesting.
- 7-9: Fair. Re-read Session 1 and 4, redo the hands-on exercises.
- Below 7: Go through each session slowly and test every example in the browser.

</details>

---

## Business Scenario Quiz — Write OData Queries (16:45 - 17:00)

### Instructions

For each business scenario, write the COMPLETE OData URL. Assume:
- Base URL: `http://localhost:4004`
- Services: `/catalog` (Books, Authors), `/sales` (Products, Orders, Customers)
- Time: 15 minutes for 10 scenarios

---

### Scenario 1: Product Search
> "A customer searches for 'laptop' in the product name. Show the first 5 matching products with their name and price, sorted by price (cheapest first)."

Your URL: _____________________________________________

---

### Scenario 2: Order History
> "Show me all orders placed by a specific customer (ID: c123-abc) in the last year (after June 1, 2025), sorted newest first."

Your URL: _____________________________________________

---

### Scenario 3: Inventory Alert
> "Find all products where stock is less than 10 AND the product is still active. Show product name, current stock, and supplier details."

Your URL: _____________________________________________

---

### Scenario 4: Author Page
> "Display author 'J.K. Rowling' details with all their books. For each book, only show the title, price, and rating. Sort books by rating (highest first)."

Your URL: _____________________________________________

---

### Scenario 5: Sales Report
> "Get the total number of orders that were delivered in January 2026."

Your URL: _____________________________________________

---

### Scenario 6: Product Catalog (Paginated)
> "Build a catalog page: Show page 3 of products (10 per page), sorted alphabetically by name. Include total count for pagination UI. Only return name, price, and rating fields."

Your URL: _____________________________________________

---

### Scenario 7: VIP Customers
> "Find customers whose credit limit is above 500,000. Show their name, email, and city. Sort by credit limit descending."

Your URL: _____________________________________________

---

### Scenario 8: Book Recommendation
> "Show the top 3 highest-rated books that cost less than $15, with their author name included."

Your URL: _____________________________________________

---

### Scenario 9: Order Detail
> "Get the details of a specific order (ID: ord-001-uuid). Include the customer name and all order items with their product names and quantities."

Your URL: _____________________________________________

---

### Scenario 10: Multi-condition Search
> "Find books that are either Fantasy or Science Fiction genre, published after 2000, with a rating of at least 4.0. Show title, genre, rating, and publish date. Sort by genre then by rating descending."

Your URL: _____________________________________________

---

### Answer Key

<details>
<summary>Click to reveal all answers</summary>

**Scenario 1: Product Search**
```
GET http://localhost:4004/sales/Products
  ?$filter=contains(tolower(productName),'laptop')
  &$select=productName,price
  &$orderby=price
  &$top=5
```

---

**Scenario 2: Order History**
```
GET http://localhost:4004/sales/Orders
  ?$filter=customer_ID eq c123-abc and orderDate gt 2025-06-01
  &$orderby=orderDate desc
```

---

**Scenario 3: Inventory Alert**
```
GET http://localhost:4004/sales/Products
  ?$filter=stock lt 10 and isActive eq true
  &$select=productName,stock
  &$expand=supplier($select=supplierName,email,phone)
```

---

**Scenario 4: Author Page**
```
GET http://localhost:4004/catalog/Authors
  ?$filter=name eq 'J.K. Rowling'
  &$expand=books($select=title,price,rating;$orderby=rating desc)
```

---

**Scenario 5: Sales Report**
```
GET http://localhost:4004/sales/Orders/$count
  ?$filter=status eq 'Delivered' and orderDate ge 2026-01-01 and orderDate le 2026-01-31
```

---

**Scenario 6: Product Catalog (Paginated)**
```
GET http://localhost:4004/sales/Products
  ?$select=productName,price,rating
  &$orderby=productName
  &$top=10
  &$skip=20
  &$count=true
```
(Page 3: skip = (3-1) × 10 = 20)

---

**Scenario 7: VIP Customers**
```
GET http://localhost:4004/sales/Customers
  ?$filter=creditLimit gt 500000
  &$select=customerName,email,city
  &$orderby=creditLimit desc
```

---

**Scenario 8: Book Recommendation**
```
GET http://localhost:4004/catalog/Books
  ?$filter=rating ge 4.0 and price lt 15
  &$orderby=rating desc
  &$top=3
  &$expand=author($select=name)
```

---

**Scenario 9: Order Detail**
```
GET http://localhost:4004/sales/Orders(ord-001-uuid)
  ?$expand=customer($select=customerName),items($expand=product($select=productName);$select=quantity,unitPrice)
```

---

**Scenario 10: Multi-condition Search**
```
GET http://localhost:4004/catalog/Books
  ?$filter=(genre eq 'Fantasy' or genre eq 'Science Fiction') and year(publishDate) gt 2000 and rating ge 4.0
  &$select=title,genre,rating,publishDate
  &$orderby=genre,rating desc
```

</details>

---

## End of Day Summary

### What You Mastered Today

| Query Option | Purpose | Key Syntax |
|-------------|---------|------------|
| `$filter` | Filter which records | `field op value` (eq, gt, lt, contains...) |
| `$select` | Choose fields to return | `field1,field2,field3` |
| `$orderby` | Sort results | `field [asc\|desc]` |
| `$top` | Limit results | Number of records |
| `$skip` | Skip records (pagination) | Records to skip |
| `$count` | Get total count | `true` or as path `/$count` |
| `$expand` | Include related entities | `assoc($select=...;$filter=...)` |
| `$search` | Free-text search | Search term |

### Key Takeaways

1. **$filter** is the most powerful — learn all operators (eq, ne, gt, lt, ge, le, and, or, contains, startswith)
2. **$expand** is the most complex — remember semicolons inside parentheses!
3. **Always paginate** in production — never send unlimited results
4. **Always $select** — never fetch more fields than you need
5. **$count counts BEFORE pagination** — that's how you calculate total pages

### The One-Liner

> **OData query options give you SQL power in a URL — so your frontend can ask for exactly what it needs, nothing more.**

---

### Looking Ahead: Day 18

Tomorrow we'll learn about **Custom Service Handlers** — adding JavaScript logic to your services:
- Before/After event hooks
- Custom validation
- Computed fields at runtime
- Error handling
- When standard CRUD isn't enough, you write custom handlers

---

### Homework

1. Write a `.http` file with at least 20 different queries covering all query options
2. For your EPM model, write OData queries for these scenarios:
   - "All suppliers from Germany with their products"
   - "Top 5 highest-value orders this month"
   - "Products in the Electronics category sorted by price"
   - "Page 2 of customers (15 per page) sorted by name"
   - "Orders with status 'New' and amount over $1000, with customer name"
3. Create a one-page "OData Query Cheat Sheet" that you'd pin to your desk

---

*End of Day 17 — You can now query any OData API like a pro!*
