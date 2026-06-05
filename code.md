

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
