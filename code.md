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
