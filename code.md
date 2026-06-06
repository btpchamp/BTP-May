### BEFORE Handlers — Validate & Modify Input

`before` handlers run BEFORE the database operation. Use them to:
- ✅ Validate input data
- ✅ Reject bad requests
- ✅ Add/modify fields bef```javascript

```javascript
// srv/admin-service.js
module.exports = function () {

  // Validate before creating a book
  this.before('CREATE', 'Books', (req) => {
    const { title, price, stock } = req.data;

    // Rule 1: Title is required
    if (!title) {
      req.error(400, 'Title is required!');
    }

    // Rule 2: Price must be positive
    if (price <= 0) {
      req.error(400, 'Price must be greater than zero');
    }

    // Rule 3: Stock can't be negative
    if (stock < 0) {
      req.error(400, 'Stock cannot be negative');
    }
  });

  // Validate before updating
  this.before('UPDATE', 'Books', (req) => {
    if (req.data.price !== undefined && req.data.price <= 0) {
      req.error(400, 'Price must be greater than zero');
    }
  });

};
```


**What happens when `req.error()` is called:**
- The request is REJECTED
- No database operation occurs
- Client receives an error response with your message
- Status code 400 (or whatever you specify)


### Modifying Data in BEFORE Handler

You can change the incoming data before it reaches the database:

```javascript
this.before('CREATE', 'Books', (req) => {
  // Auto-capitalize the title
  if (req.data.title) {
    req.data.title = req.data.title.trim();
  }

  // Set default genre if not provided
  if (!req.data.genre) {
    req.data.genre = 'Uncategorized';
  }

  // Set creation timestamp (alternative to managed aspect)
  req.data.processedAt = new Date().toISOString();
});
```


