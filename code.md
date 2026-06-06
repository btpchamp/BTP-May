
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
