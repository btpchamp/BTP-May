### Exercise 2: Implement Callback Patterns (15 minutes)

```javascript
// EXERCISE: Build a data processing pipeline using callbacks

// Given this data:
const orders = [
  { id: "ORD-001", customer: "Acme Corp", amount: 15000, status: "pending" },
  { id: "ORD-002", customer: "Beta Inc", amount: 8000, status: "approved" },
  { id: "ORD-003", customer: "Gamma Ltd", amount: 52000, status: "pending" },
  { id: "ORD-004", customer: "Delta Co", amount: 3000, status: "rejected" },
  { id: "ORD-005", customer: "Epsilon SA", amount: 95000, status: "pending" }
];

// TASK 1: Create a function 'processOrders' that takes:
// - an array of orders
// - a filter callback (which orders to include)
// - a transform callback (how to change each order)
// - a summary callback (what to do with the final result)

const processOrders = (orderList, filterFn, transformFn, summaryFn) => {
  // Your code: filter → transform → summary
};

// TASK 2: Use processOrders to:
// a) Get total amount of all pending orders over ₹10000
// b) Get a list of approved customer names in uppercase
// c) Get count of orders by status

// Example call for (a):
processOrders(
  orders,
  order => order.status === "pending" && order.amount > 10000,
  order => order.amount,
  amounts => console.log(`Total pending (>₹10k): ₹${amounts.reduce((s,a) => s+a, 0)}`)
);
```

### Exercise 3: Closure Challenge (10 minutes)

```javascript
// EXERCISE: Build these closure-based utilities

// 1. createIdGenerator — generates sequential IDs with a prefix
// Usage:
//   const poIdGen = createIdGenerator("PO");
//   poIdGen() → "PO-001"
//   poIdGen() → "PO-002"
//   poIdGen() → "PO-003"

const createIdGenerator = (prefix) => {
  // Your code here
};

// 2. createLogger — logs messages with a component name and timestamp
// Usage:
//   const dbLogger = createLogger("Database");
//   dbLogger("Connected");     → "[Database] 10:30:45 - Connected"
//   dbLogger("Query executed"); → "[Database] 10:30:46 - Query executed"

const createLogger = (component) => {
  // Your code here
};

// 3. createCache — simple key-value cache with hit tracking
// Usage:
//   const cache = createCache();
//   cache.set("user1", {name: "Priya"});
//   cache.get("user1"); → {name: "Priya"}
//   cache.get("user2"); → undefined
//   cache.stats(); → {size: 1, hits: 1, misses: 1}

const createCache = () => {
  // Your code here
};
```
