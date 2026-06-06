### ON Handlers — Replace the Default Behavior

`on` handlers REPLACE the default CAP behavior entirely. Use with caution!

```javascript
module.exports = function () {

  // Custom READ that adds extra logic
  this.on('READ', 'Books', async (req, next) => {
    // Call the default handler first
    const results = await next();

    // Then modify the results
    if (results) {
      const books = Array.isArray(results) ? results : [results];
      books.forEach(book => {
        book.readAt = new Date().toISOString();
      });
    }

    return results;
  });

};
```

**The `next()` function:**
- Calls the DEFAULT handler (CAP's built-in CRUD)
- Returns the results
- You can modify and return them
- If you DON'T call `next()`, the default behavior is completely skipped!


### ON Handler — Completely Custom Implementation

```javascript
// Custom action: Calculate order total
this.on('calcTotal', 'Orders', async (req) => {
  const { ID } = req.params[0];

  // Custom query
  const order = await SELECT.one.from('SalesOrders')
    .where({ ID })
    .columns('ID', 'grossAmount');

  const items = await SELECT.from('SalesOrderItems')
    .where({ order_ID: ID });

  // Custom calculation
  const total = items.reduce((sum, item) => {
    return sum + (item.quantity * item.unitPrice);
  }, 0);

  return { orderID: ID, calculatedTotal: total };
});
```
