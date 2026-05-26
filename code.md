### Promise Methods Summary

| Method | What It Does | Resolves When | Rejects When |
|--------|-------------|--------------|-------------|
| `Promise.all([p1, p2, p3])` | Run all in parallel | ALL succeed | ANY one fails |
| `Promise.race([p1, p2, p3])` | Race — first result wins | First one settles | First one rejects |
| `Promise.allSettled([...])` | Wait for all (success or failure) | All settle | Never rejects |
| `Promise.any([p1, p2, p3])` | First SUCCESS wins | First one resolves | ALL fail |


## Session 3: Hands-on — Promises Practice (12:00 - 13:00)

### Exercise 1: Create Your Own Promises (15 minutes)

Create `promises.js`:

```javascript
// EXERCISE 1: Simulate async operations with Promises

// 1. Create a function 'delay(ms)' that returns a Promise
// that resolves after 'ms' milliseconds
const delay = (ms) => {
  // Your code here
};

// Test:
// delay(1000).then(() => console.log("1 second passed!"));


// 2. Create 'fetchProduct(id)' that simulates fetching a product
// - If id starts with "PRD", resolve with product object after 500ms
// - If id doesn't start with "PRD", reject with "Invalid product ID"
const fetchProduct = (id) => {
  // Your code here
};

// Test:
// fetchProduct("PRD-001").then(p => console.log(p)).catch(e => console.log(e));
// fetchProduct("INVALID").then(p => console.log(p)).catch(e => console.log(e));


// 3. Create a 'retryFetch(url, maxRetries)' function that:
// - Simulates a fetch that fails 70% of the time
// - Retries up to maxRetries times
// - Resolves with data if any attempt succeeds
// - Rejects if ALL attempts fail
const retryFetch = (url, maxRetries = 3) => {
  // Your code here
};
