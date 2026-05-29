### Hands-on Challenge: Extend Your CAP App (10 minutes)

Add a new entity `Reviews` to your bookshop:

1. Add to `db/schema.cds`:
```cds
entity Reviews {
  key ID    : Integer;
  bookId   : Integer;
  reviewer : String(100);
  rating   : Integer;
  comment  : String(500);
}
```

2. Expose in `srv/catalog-service.cds`:
```cds
service CatalogService {
  entity Books as projection on bookshop.Books;
  entity Authors as projection on bookshop.Authors;
  entity Reviews as projection on bookshop.Reviews;
}
```

3. Save both files — `cds watch` auto-restarts!

4. Test: `http://localhost:4004/catalog/Reviews` — new endpoint instantly available!

5. Create a review:
```http
POST http://localhost:4004/catalog/Reviews
Content-Type: application/json

{
  "ID": 1,
  "bookId": 1,
  "reviewer": "Priya",
  "rating": 5,
  "comment": "Excellent book on clean coding practices!"
}
```
