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
