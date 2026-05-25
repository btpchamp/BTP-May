### Practice: Convert to Arrow Functions (5 minutes)

```javascript
// Convert these to arrow functions:

// 1.
function square(n) {
  return n * n;
}

// 2.
function isEven(n) {
  return n % 2 === 0;
}

// 3.
function fullName(first, last) {
  return `${first} ${last}`;
}

// 4.
function findMax(arr) {
  return Math.max(...arr);
}

// 5.
function createPO(vendor, amount) {
  return {
    id: `PO-${Date.now()}`,
    vendor: vendor,
    amount: amount,
    status: "Draft"
  };
}
```
