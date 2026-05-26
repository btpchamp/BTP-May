### Practical Examples

#### Example 1: API Fetching (Simulated)

```javascript
const fetchUserData = async (userId) => {
  try {
    // Simulate API call:
    const user = await new Promise(resolve => 
      setTimeout(() => resolve({ id: userId, name: "Priya", role: "Developer" }), 500)
    );
    
    const orders = await new Promise(resolve => 
      setTimeout(() => resolve([
        { id: "PO-001", amount: 5000 },
        { id: "PO-002", amount: 12000 }
      ]), 700)
    );
    
    return {
      ...user,
      orders,
      totalOrders: orders.length,
      totalAmount: orders.reduce((sum, o) => sum + o.amount, 0)
    };
    
  } catch (error) {
    console.log("Failed to load user data:", error.message);
    return null;
  }
};

// Usage:
const main = async () => {
  const userData = await fetchUserData("USER-001");
  console.log("User Data:", JSON.stringify(userData, null, 2));
};
main();
```

#### Example 2: Sequential Processing with Loop

```javascript
const processAllOrders = async (orderIds) => {
  const results = [];
  
  for (const id of orderIds) {
    try {
      console.log(`Processing ${id}...`);
      const result = await processOrder(id);
      results.push({ id, status: "success", data: result });
    } catch (error) {
      results.push({ id, status: "failed", error: error.message });
    }
  }
  
  return results;
};

// Process one by one (sequential):
processAllOrders(["PO-001", "PO-002", "PO-003"])
  .then(results => console.log("All done:", results));
```

---

### Async Patterns Cheat Sheet

| Pattern | When to Use | Syntax |
|---------|-------------|--------|
| Sequential | Step B needs Step A's result | `const a = await A(); const b = await B(a);` |
| Parallel | Independent operations | `const [a, b] = await Promise.all([A(), B()]);` |
| Error handling | Might fail | `try { await op(); } catch(e) { handle(e); }` |
| Loop (sequential) | Process items one by one | `for (const item of items) { await process(item); }` |
| Loop (parallel) | Process all items at once | `await Promise.all(items.map(i => process(i)));` |

---
