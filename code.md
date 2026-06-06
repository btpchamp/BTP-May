### Async Handlers — Database Lookups in Validation

Sometimes validation requires checking the database:

```javascript
this.before('CREATE', 'Orders', async (req) => {
  //                              ↑ Note: async!
  const { customer_ID, items } = req.data;

  // Check if customer exists in the database
  const customer = await SELECT.one
    .from('com.epm.Customers')
    .where({ ID: customer_ID });

  if (!customer) {
    req.reject(404, `Customer ${customer_ID} not found`);
  }

  // Check if customer has sufficient credit
  if (customer.creditLimit < req.data.grossAmount) {
    req.reject(400, `Order amount exceeds customer credit limit of ${customer.creditLimit}`);
  }

  // Check stock for each item
  if (items) {
    for (const item of items) {
      const product = await SELECT.one
        .from('com.epm.Products')
        .where({ ID: item.product_ID });

      if (!product) {
        req.error(400, `Product ${item.product_ID} not found`);
      } else if (product.stock < item.quantity) {
        req.error(400, `Insufficient stock for ${product.productName}. Available: ${product.stock}`);
      }
    }
  }
});
```

---

### AFTER Handlers — Practical Examples

#### Example 1: Add Computed Fields to Response

```javascript
this.after('READ', 'Books', (results) => {
  const books = Array.isArray(results) ? results : [results];

  for (const book of books) {
    // Compute tax (18% GST)
    if (book.price) {
      book.priceWithTax = +(book.price * 1.18).toFixed(2);
      book.taxAmount = +(book.price * 0.18).toFixed(2);
    }

    // Compute availability status
    if (book.stock !== undefined) {
      if (book.stock === 0) book.availability = 'Out of Stock';
      else if (book.stock < 10) book.availability = 'Low Stock';
      else book.availability = 'In Stock';
    }
  }
});
```

---

#### Example 2: Auto-Calculate Order Total

```javascript
this.before('CREATE', 'Orders', (req) => {
  const { items } = req.data;

  if (items && items.length > 0) {
    // Calculate totals from items
    let netAmount = 0;
    for (const item of items) {
      item.netAmount = item.quantity * item.unitPrice;
      netAmount += item.netAmount;
    }

    req.data.netAmount = netAmount;
    req.data.taxAmount = +(netAmount * 0.18).toFixed(2);
    req.data.grossAmount = +(netAmount + req.data.taxAmount).toFixed(2);
  }
});
```

---

#### Example 3: Log All Delete Operations

```javascript
this.after('DELETE', 'Books', (_, req) => {
  const bookId = req.params[0]?.ID || req.params[0];
  console.log(`[AUDIT] Book ${bookId} deleted by ${req.user.id} at ${new Date().toISOString()}`);
});
```

---

#### Example 4: Prevent Deletion Under Certain Conditions

```javascript
this.before('DELETE', 'Books', async (req) => {
  const bookId = req.params[0]?.ID || req.params[0];

  // Check if book has any active orders
  const activeOrders = await SELECT.from('SalesOrderItems')
    .where({ product_ID: bookId });

  if (activeOrders.length > 0) {
    req.reject(409, `Cannot delete book. It has ${activeOrders.length} order references.`);
  }
});
```
