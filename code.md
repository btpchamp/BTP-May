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
