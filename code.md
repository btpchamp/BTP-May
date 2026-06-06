### Basic Handler Structure

```javascript
// srv/cat-service.js

module.exports = function () {

  // BEFORE handler — runs before the operation
  this.before('CREATE', 'Books', (req) => {
    // Your validation logic here
  });

  // AFTER handler — runs after the operation
  this.after('READ', 'Books', (results, req) => {
    // Your transformation logic here
  });

  // ON handler — replaces the default operation
  this.on('READ', 'Books', (req) => {
    // Your custom implementation here
  });

};
```
