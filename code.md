### $orderby — Sort Your Results

Like ORDER BY in SQL — arrange results in a specific order.

**Syntax:**
```
$orderby=field              (ascending — default)
$orderby=field asc          (ascending — explicit)
$orderby=field desc         (descending — highest first)
$orderby=field1,field2      (sort by field1, then field2 for ties)
```

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
