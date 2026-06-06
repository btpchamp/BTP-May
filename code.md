Let's build a real-world validation for a Book entity:

```javascript
// srv/admin-service.js
const cds = require('@sap/cds');

//module.exports = function () {. Try to use function
module.exports = class AdminService extends cds.ApplicationService { init(){
  // ══════════════════════════════════════════
  // VALIDATION: Before Creating a Book
  // ══════════════════════════════════════════

    this.before('CREATE', 'Books', async (req) => {
        const { title } = req.data;
        console.log('Creating a book with title:', req.data);
        console.log('Lenth of my Title', req.data.title.length);
        const data = await cds.run(SELECT.from('Books').where({ ID: req.data.ID }));
        console.log('Data from DB', data);
        console.log('Length of record', data.length);
        console.log('DB Array to JSON', data[0]);
        console.log('Price', data[0].price);
        // if (!title) {
        // req.error(400, 'This book is not allowed to be created.');
        // }
    });



    const { title, price, stock, isbn, genre, author_ID } = req.data;

    // --- Required field checks ---
    if (!title || title.trim().length === 0) {
      req.error(400, 'Title is required and cannot be empty', 'title');
    }

    if (price === undefined || price === null) {
      req.error(400, 'Price is required', 'price');
    }

    // --- Value range checks ---
    if (price !== undefined && price <= 0) {
      req.error(400, 'Price must be greater than zero', 'price');
    }

    if (price !== undefined && price > 9999.99) {
      req.error(400, 'Price cannot exceed 9999.99', 'price');
    }

    if (stock !== undefined && stock < 0) {
      req.error(400, 'Stock cannot be negative', 'stock');
    }

    // --- Format checks ---
    if (isbn && isbn.length !== 13) {
      req.error(400, 'ISBN must be exactly 13 characters', 'isbn');
    }

    // --- Business rule: Valid genres ---
    const validGenres = ['Fantasy', 'Fiction', 'Mystery', 'Thriller',
                         'Science Fiction', 'Romance', 'Non-Fiction',
                         'Biography', 'Dystopian', 'Literary Fiction'];
    if (genre && !validGenres.includes(genre)) {
      req.error(400, `Invalid genre. Must be one of: ${validGenres.join(', ')}`, 'genre');
    }

    // --- Referential check: Does the author exist? ---
    if (author_ID) {
      const author = await SELECT.one.from('my.bookshop.Authors').where({ ID: author_ID });
      if (!author) {
        req.error(400, 'Author not found. Please provide a valid author_ID', 'author_ID');
      }
    }
  });

  // ══════════════════════════════════════════
  // VALIDATION: Before Updating a Book
  // ══════════════════════════════════════════
  this.before('UPDATE', 'Books', (req) => {
    const { price, stock, isbn } = req.data;

    if (price !== undefined && price <= 0) {
      req.error(400, 'Price must be greater than zero', 'price');
    }

    if (stock !== undefined && stock < 0) {
      req.error(400, 'Stock cannot be negative', 'stock');
    }

    if (isbn !== undefined && isbn.length !== 13) {
      req.error(400, 'ISBN must be exactly 13 characters', 'isbn');
    }
  });

};
```
