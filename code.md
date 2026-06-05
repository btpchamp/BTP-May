### $top and $skip — Pagination

**The problem:** You have 10,000 products. You can't send ALL of them to the mobile app at once!

**Solution:** Send them in pages.

```
$top = How many records to return (page size)
$skip = How many records to skip (which page)
```



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
