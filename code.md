### AFTER Handlers — Transform Output & Side Effects

`after` handlers run AFTER the database operation succeeds. Use them to:
- ✅ Add computed fields to the response
- ✅ Transform data format
- ✅ Trigger side effects (logging, notifications)
- ✅ Enrich response with additional info

```javascript
module.exports = function () {

  // Add a computed field after reading books
  this.after('READ', 'Books', (results, req) => {
    // 'results' can be an array (READ all) or a single object (READ one)
    const books = Array.isArray(results) ? results : [results];

    for (const book of books) {
      // Add discount price (20% off)
      if (book.price) {
        book.discountPrice = (book.price * 0.8).toFixed(2);
      }

      // Add availability status
      if (book.stock > 20) {
        book.availability = 'In Stock';
      } else if (book.stock > 0) {
        book.availability = 'Low Stock';
      } else {
        book.availability = 'Out of Stock';
      }
    }
  });

  // Log after delete (side effect)
  this.after('DELETE', 'Books', (_, req) => {
    console.log(`Book deleted by ${req.user.id} at ${new Date().toISOString()}`);
  });
};
```
**Important:** In `after` handlers, the first parameter is `results` (the data), the second is `req`.
