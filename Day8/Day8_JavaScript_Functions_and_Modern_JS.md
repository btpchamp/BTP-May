# Day 8: JavaScript Functions & Modern JS (ES6+)

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 7 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Functions — Declaration, Expression & Arrow Functions | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Callbacks, Higher-Order Functions & Closures | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Function Conversions & Callback Patterns | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 15:00 | Session 4: Classes, Prototypes & Modules (ES6+) | 75 min |
| 15:00 - 15:15 | Break | 15 min |
| 15:15 - 16:00 | Session 5: Error Handling & Hands-on — Mini Library with Classes | 45 min |
| 16:00 - 16:30 | Session 6: Live Coding Quiz (5 Problems in 30 min) | 30 min |
| 16:30 - 17:00 | Assessment & Wrap-up | 30 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Write functions using declarations, expressions, and arrow syntax
- Use default and rest parameters effectively
- Understand and use callbacks and higher-order functions
- Explain closures and the scope chain
- Create classes with constructors, methods, and inheritance
- Use ES6 modules (import/export)
- Handle errors gracefully with try/catch/finally
- Build a mini library system using classes

---

## Day 7 Recap — Quick Fire (09:00 - 09:15)

1. What does `.map()` do? → _____
2. What does `.filter()` do? → _____
3. What does `.reduce()` do? → _____
4. What is destructuring? → _____
5. What's the difference between spread and rest? → _____
6. `JSON.parse()` converts _____ → _____
7. `JSON.stringify()` converts _____ → _____
8. `for...of` iterates over _____, `for...in` iterates over _____

<details>
<summary>Answers</summary>

1. Creates a NEW array by transforming each element
2. Creates a NEW array with only elements that pass a test
3. Reduces an array to a single value (sum, max, etc.)
4. Extracting values from objects/arrays into variables in one line
5. Spread EXPANDS (outward), Rest COLLECTS (inward)
6. JSON string → JavaScript object
7. JavaScript object → JSON string
8. `for...of` = array/string VALUES; `for...in` = object KEYS

</details>

---

## Session 1: Functions — Declaration, Expression & Arrow (09:15 - 10:30)

### What is a Function?

A **function** is a reusable block of code that performs a specific task. Think of it as a **machine**:

```
         INPUT(S)          FUNCTION           OUTPUT
            ↓                 ↓                 ↓
         [5, 3]    →    [ ADD MACHINE ]    →    8
         ["Hi"]    →    [ SHOUT MACHINE ] →   "HI!"
         [order]   →    [ APPROVE MACHINE] →  "Approved"
```

**Why functions?**
- ❌ Without functions: Copy-paste same code 50 times
- ✅ With functions: Write once, call 50 times

---

### 3 Ways to Create Functions

#### 1. Function Declaration (The Classic)

```javascript
function greet(name) {
  return `Hello, ${name}! Welcome to SAP CAP training.`;
}

console.log(greet("Priya"));   // "Hello, Priya! Welcome to SAP CAP training."
console.log(greet("Rahul"));   // "Hello, Rahul! Welcome to SAP CAP training."
```

**Characteristics:**
- Starts with the `function` keyword
- Has a name (can be called anywhere in the file — hoisted!)
- Most readable for beginners

---

#### 2. Function Expression (Store in a Variable)

```javascript
const greet = function(name) {
  return `Hello, ${name}!`;
};

console.log(greet("Arun"));   // "Hello, Arun!"
```

**Characteristics:**
- Stored in a variable (treated like any value)
- NOT hoisted (must be defined before use)
- Can be anonymous (no name after `function`)

---

#### 3. Arrow Function (Modern & Concise) ⭐

```javascript
const greet = (name) => {
  return `Hello, ${name}!`;
};

// Even shorter for one-line returns:
const greet = (name) => `Hello, ${name}!`;

console.log(greet("Sneha"));  // "Hello, Sneha!"
```

**Characteristics:**
- Shortest syntax
- Implicit return for single expressions (no `{}` or `return` needed!)
- Most commonly used in modern JavaScript
- Used heavily in CAP development

---

### Arrow Function Shorthand Rules

```javascript
// Full arrow function:
const add = (a, b) => {
  return a + b;
};

// Shorthand: If body is a single expression, remove {} and return:
const add = (a, b) => a + b;

// Shorthand: If only ONE parameter, remove parentheses:
const double = n => n * 2;

// Shorthand: If NO parameters, use empty parentheses:
const sayHello = () => "Hello!";

// Returning an object? Wrap in parentheses:
const makeUser = (name, age) => ({ name, age });
```

---

### Comparison Table

| Feature | Declaration | Expression | Arrow |
|---------|------------|-----------|-------|
| Syntax | `function name() {}` | `const name = function() {}` | `const name = () => {}` |
| Hoisted? | ✅ Yes | ❌ No | ❌ No |
| `this` binding | Own `this` | Own `this` | Inherits parent `this` |
| Best for | Named, standalone functions | Callbacks, variable storage | Short callbacks, modern code |
| Use in CAP? | Sometimes | Rarely | Most of the time! ⭐ |

---

### Parameters — Default Values

```javascript
// Without default: Missing argument → undefined
function greet(name) {
  return `Hello, ${name}!`;
}
console.log(greet());  // "Hello, undefined!" 😬

// With default value:
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}
console.log(greet());         // "Hello, Guest!"
console.log(greet("Priya"));  // "Hello, Priya!"
```

```javascript
// SAP context: Default status for new orders
const createOrder = (vendor, amount, status = "Draft", currency = "INR") => {
  return { vendor, amount, status, currency, createdAt: new Date().toISOString() };
};

console.log(createOrder("TechCorp", 50000));
// { vendor: "TechCorp", amount: 50000, status: "Draft", currency: "INR", createdAt: "..." }

console.log(createOrder("GlobalInc", 80000, "Submitted", "USD"));
// { vendor: "GlobalInc", amount: 80000, status: "Submitted", currency: "USD", createdAt: "..." }
```

---

### Parameters — Rest Parameters

Collect an unknown number of arguments into an array:

```javascript
const sum = (...numbers) => {
  return numbers.reduce((total, n) => total + n, 0);
};

console.log(sum(1, 2, 3));           // 6
console.log(sum(10, 20, 30, 40));    // 100
console.log(sum(5));                 // 5
```

```javascript
// First param separate, rest collected:
const logAction = (user, ...actions) => {
  console.log(`User: ${user}`);
  console.log(`Actions: ${actions.join(", ")}`);
};

logAction("Priya", "login", "view orders", "approve PO", "logout");
// User: Priya
// Actions: login, view orders, approve PO, logout
```

---

### Functions Returning Functions

Functions can return other functions!

```javascript
const createMultiplier = (factor) => {
  return (number) => number * factor;
};

const double = createMultiplier(2);
const triple = createMultiplier(3);
const gstAdder = createMultiplier(1.18);

console.log(double(5));       // 10
console.log(triple(5));       // 15
console.log(gstAdder(1000));  // 1180
```

---

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

<details>
<summary>Answers</summary>

```javascript
const square = n => n * n;
const isEven = n => n % 2 === 0;
const fullName = (first, last) => `${first} ${last}`;
const findMax = arr => Math.max(...arr);
const createPO = (vendor, amount) => ({
  id: `PO-${Date.now()}`,
  vendor,
  amount,
  status: "Draft"
});
```

</details>

---

## Session 2: Callbacks, Higher-Order Functions & Closures (10:45 - 12:00)

### What is a Callback?

A **callback** is a function passed as an argument to another function, to be called later.

**Analogy:** You order food at a restaurant. You give the waiter your order (callback). The kitchen prepares it. When ready, the waiter calls you back with the food.

```javascript
// greet is a CALLBACK — it's passed to processUser and called inside it
const greet = (name) => console.log(`Hello, ${name}!`);

const processUser = (userName, callback) => {
  console.log("Processing user...");
  callback(userName);  // Call the function that was passed in
};

processUser("Priya", greet);
// "Processing user..."
// "Hello, Priya!"
```

---

### Why Callbacks?

Callbacks let you customize behavior without changing the function:

```javascript
const processOrder = (order, onSuccess, onFailure) => {
  if (order.amount > 0 && order.vendor) {
    onSuccess(order);
  } else {
    onFailure("Invalid order: missing vendor or amount");
  }
};

// Different behaviors for different situations:
processOrder(
  { vendor: "TechCorp", amount: 5000 },
  (order) => console.log(`✅ Order approved: ${order.vendor} - ₹${order.amount}`),
  (error) => console.log(`❌ Error: ${error}`)
);
// ✅ Order approved: TechCorp - ₹5000

processOrder(
  { vendor: "", amount: 0 },
  (order) => console.log(`✅ Order approved`),
  (error) => console.log(`❌ Error: ${error}`)
);
// ❌ Error: Invalid order: missing vendor or amount
```

---

### Higher-Order Functions

A **higher-order function** is a function that:
- Takes a function as an argument (like `map`, `filter`, `reduce`), OR
- Returns a function

**You've already used them!** `map`, `filter`, and `reduce` are higher-order functions:

```javascript
let numbers = [1, 2, 3, 4, 5];

// .map() takes a FUNCTION as argument → higher-order function!
let doubled = numbers.map(n => n * 2);

// .filter() takes a FUNCTION as argument → higher-order function!
let evens = numbers.filter(n => n % 2 === 0);

// .sort() takes a FUNCTION as argument → higher-order function!
let sorted = numbers.sort((a, b) => a - b);
```

#### Building Your Own Higher-Order Function

```javascript
// A function that takes a function and applies it twice:
const applyTwice = (fn, value) => fn(fn(value));

const addThree = n => n + 3;
const double = n => n * 2;

console.log(applyTwice(addThree, 5));  // addThree(addThree(5)) = addThree(8) = 11
console.log(applyTwice(double, 3));    // double(double(3)) = double(6) = 12
```

```javascript
// A function that creates validators:
const createValidator = (minAmount, maxAmount) => {
  return (order) => order.amount >= minAmount && order.amount <= maxAmount;
};

const isSmallOrder = createValidator(0, 10000);
const isMediumOrder = createValidator(10001, 50000);
const isLargeOrder = createValidator(50001, Infinity);

let order = { vendor: "ABC", amount: 25000 };
console.log(isSmallOrder(order));   // false
console.log(isMediumOrder(order));  // true
console.log(isLargeOrder(order));   // false
```

---

### Closures — Functions Remember Their Birthplace

A **closure** is when a function "remembers" variables from the scope where it was created, even after that scope is gone.

```javascript
const createCounter = () => {
  let count = 0;  // This variable lives in the closure!
  
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  };
};

const counter = createCounter();
console.log(counter.increment());  // 1
console.log(counter.increment());  // 2
console.log(counter.increment());  // 3
console.log(counter.decrement());  // 2
console.log(counter.getCount());   // 2

// 'count' is private! Cannot be accessed directly:
// console.log(counter.count);  // undefined — it's hidden inside the closure!
```

**Why this matters:** `count` is completely private. No code outside can modify it directly. This is how you create private state in JavaScript!

---

#### Closure Analogy: A Backpack

When a function is created, it packs a "backpack" with all the variables from its surrounding scope. It carries this backpack wherever it goes:

```javascript
const makeGreeter = (greeting) => {
  // 'greeting' is packed in the closure backpack
  return (name) => `${greeting}, ${name}!`;
};

const sayHello = makeGreeter("Hello");    // backpack contains: greeting = "Hello"
const sayNamaste = makeGreeter("Namaste"); // backpack contains: greeting = "Namaste"

console.log(sayHello("Priya"));     // "Hello, Priya!"
console.log(sayNamaste("Rahul"));   // "Namaste, Rahul!"
// Each function remembers its own greeting from when it was created!
```

---

#### Closure — SAP Practical Example

```javascript
// Create a service that tracks API calls with rate limiting:
const createAPIRateLimiter = (maxCallsPerMinute) => {
  let callCount = 0;
  let resetTime = Date.now() + 60000;
  
  return (apiName) => {
    if (Date.now() > resetTime) {
      callCount = 0;
      resetTime = Date.now() + 60000;
    }
    
    if (callCount >= maxCallsPerMinute) {
      return `❌ Rate limit exceeded for ${apiName}. Try after 1 minute.`;
    }
    
    callCount++;
    return `✅ API call #${callCount} to ${apiName} — allowed`;
  };
};

const s4hanaLimiter = createAPIRateLimiter(5);

console.log(s4hanaLimiter("BusinessPartner"));  // ✅ call #1
console.log(s4hanaLimiter("SalesOrder"));       // ✅ call #2
console.log(s4hanaLimiter("PurchaseOrder"));    // ✅ call #3
console.log(s4hanaLimiter("Product"));          // ✅ call #4
console.log(s4hanaLimiter("Invoice"));          // ✅ call #5
console.log(s4hanaLimiter("Delivery"));         // ❌ Rate limit exceeded
```

---

### Scope Chain — How JavaScript Finds Variables

When you use a variable, JavaScript looks for it in this order:

```
1. Local scope (inside current function)
      ↓ not found?
2. Outer function scope (enclosing function)
      ↓ not found?
3. Outer outer scope...
      ↓ not found?
4. Global scope
      ↓ not found?
5. ReferenceError! ❌
```

```javascript
const globalVar = "I'm global";

const outer = () => {
  const outerVar = "I'm in outer";
  
  const inner = () => {
    const innerVar = "I'm in inner";
    
    // inner can access ALL levels:
    console.log(innerVar);   // ✅ Own scope
    console.log(outerVar);   // ✅ Parent scope
    console.log(globalVar);  // ✅ Global scope
  };
  
  inner();
  // console.log(innerVar);  // ❌ Cannot access child scope!
};

outer();
```

---

## Session 3: Hands-on — Function Conversions & Callbacks (12:00 - 13:00)

### Exercise 1: Convert to Arrow Functions (10 minutes)

```javascript
// Convert ALL these to the shortest possible arrow function syntax:

// 1.
function add(a, b) {
  return a + b;
}

// 2.
function isAdult(age) {
  return age >= 18;
}

// 3.
function getFirstChar(str) {
  return str.charAt(0);
}

// 4.
function multiply(a, b) {
  return a * b;
}

// 5.
function formatCurrency(amount) {
  return `₹${amount.toLocaleString()}`;
}

// 6.
function getOrderStatus(order) {
  if (order.amount > 50000) {
    return "Requires approval";
  } else {
    return "Auto-approved";
  }
}

// 7.
function createEmployee(name, dept, salary) {
  return {
    name: name,
    dept: dept,
    salary: salary,
    id: `EMP-${Date.now()}`
  };
}

// 8.
function sumArray(numbers) {
  return numbers.reduce(function(total, num) {
    return total + num;
  }, 0);
}
```

<details>
<summary>Solutions</summary>

```javascript
const add = (a, b) => a + b;
const isAdult = age => age >= 18;
const getFirstChar = str => str.charAt(0);
const multiply = (a, b) => a * b;
const formatCurrency = amount => `₹${amount.toLocaleString()}`;
const getOrderStatus = order => order.amount > 50000 ? "Requires approval" : "Auto-approved";
const createEmployee = (name, dept, salary) => ({ name, dept, salary, id: `EMP-${Date.now()}` });
const sumArray = numbers => numbers.reduce((total, num) => total + num, 0);
```

</details>

---

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

<details>
<summary>Solution</summary>

```javascript
const processOrders = (orderList, filterFn, transformFn, summaryFn) => {
  const filtered = orderList.filter(filterFn);
  const transformed = filtered.map(transformFn);
  summaryFn(transformed);
};

// a) Total pending over ₹10000:
processOrders(
  orders,
  o => o.status === "pending" && o.amount > 10000,
  o => o.amount,
  amounts => console.log(`Total: ₹${amounts.reduce((s,a) => s+a, 0)}`)
);
// Total: ₹162000

// b) Approved customer names in uppercase:
processOrders(
  orders,
  o => o.status === "approved",
  o => o.customer.toUpperCase(),
  names => console.log("Approved:", names)
);
// Approved: ["BETA INC"]

// c) Count by status:
const countByStatus = orders.reduce((counts, o) => {
  counts[o.status] = (counts[o.status] || 0) + 1;
  return counts;
}, {});
console.log(countByStatus);
// { pending: 3, approved: 1, rejected: 1 }
```

</details>

---

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

<details>
<summary>Solutions</summary>

```javascript
// 1. ID Generator
const createIdGenerator = (prefix) => {
  let count = 0;
  return () => {
    count++;
    return `${prefix}-${String(count).padStart(3, "0")}`;
  };
};

const poIdGen = createIdGenerator("PO");
console.log(poIdGen());  // "PO-001"
console.log(poIdGen());  // "PO-002"
console.log(poIdGen());  // "PO-003"

// 2. Logger
const createLogger = (component) => {
  return (message) => {
    const time = new Date().toLocaleTimeString();
    console.log(`[${component}] ${time} - ${message}`);
  };
};

const dbLogger = createLogger("Database");
dbLogger("Connected");
dbLogger("Query executed");

// 3. Cache
const createCache = () => {
  const store = {};
  let hits = 0;
  let misses = 0;
  
  return {
    set: (key, value) => { store[key] = value; },
    get: (key) => {
      if (key in store) { hits++; return store[key]; }
      misses++;
      return undefined;
    },
    stats: () => ({ size: Object.keys(store).length, hits, misses })
  };
};
```

</details>

---

## Session 4: Classes, Prototypes & Modules (13:45 - 15:00)

### What is a Class?

A **class** is a blueprint for creating objects. Think of it as a **cookie cutter** — one shape, many cookies:

```
Class (blueprint):          Objects (instances):
+------------------+        +------------------+
| PurchaseOrder    |  ───►  | PO-001: TechCorp |
| - vendor         |  ───►  | PO-002: GlobalInc|
| - amount         |  ───►  | PO-003: LocalLtd |
| - approve()      |
| - reject()       |
+------------------+
```

---

### Creating a Class

```javascript
class PurchaseOrder {
  // Constructor — runs when you create a new instance
  constructor(vendor, amount, items) {
    this.id = `PO-${Date.now()}`;
    this.vendor = vendor;
    this.amount = amount;
    this.items = items;
    this.status = "Draft";
    this.createdAt = new Date().toISOString();
  }
  
  // Methods — actions the object can perform
  submit() {
    if (this.status === "Draft") {
      this.status = "Submitted";
      return `✅ PO submitted to ${this.vendor}`;
    }
    return `❌ Cannot submit — current status: ${this.status}`;
  }
  
  approve(approver) {
    if (this.status === "Submitted") {
      this.status = "Approved";
      this.approvedBy = approver;
      this.approvedAt = new Date().toISOString();
      return `✅ PO approved by ${approver}`;
    }
    return `❌ Cannot approve — current status: ${this.status}`;
  }
  
  reject(reason) {
    if (this.status === "Submitted") {
      this.status = "Rejected";
      this.rejectionReason = reason;
      return `❌ PO rejected: ${reason}`;
    }
    return `❌ Cannot reject — current status: ${this.status}`;
  }
  
  getSummary() {
    return `[${this.id}] ${this.vendor} | ₹${this.amount.toLocaleString()} | Status: ${this.status}`;
  }
}

// Create instances (objects) from the class:
const po1 = new PurchaseOrder("TechCorp", 75000, 5);
const po2 = new PurchaseOrder("OfficeSupplies", 12000, 20);

console.log(po1.getSummary());
console.log(po1.submit());
console.log(po1.approve("Priya Sharma"));
console.log(po1.getSummary());

console.log(po2.submit());
console.log(po2.reject("Budget exceeded"));
```

---

### Class Anatomy

```javascript
class ClassName {
  // 1. CONSTRUCTOR — initializes the object
  constructor(param1, param2) {
    this.prop1 = param1;    // 'this' = the current object being created
    this.prop2 = param2;
  }
  
  // 2. METHODS — functions that belong to the object
  methodName() {
    return this.prop1;
  }
  
  // 3. GETTERS — computed properties (accessed like properties, not called like functions)
  get fullInfo() {
    return `${this.prop1} - ${this.prop2}`;
  }
  
  // 4. SETTERS — validate before setting a value
  set price(value) {
    if (value < 0) throw new Error("Price cannot be negative");
    this._price = value;
  }
  
  // 5. STATIC METHODS — belong to the class itself, not instances
  static createDefault() {
    return new ClassName("default", "default");
  }
}
```

---

### Inheritance — extends & super

One class can inherit from another (child gets parent's features + adds its own):

```javascript
// Parent class:
class Employee {
  constructor(name, department, salary) {
    this.name = name;
    this.department = department;
    this.salary = salary;
    this.id = `EMP-${Math.floor(Math.random() * 10000)}`;
  }
  
  getDetails() {
    return `${this.name} | ${this.department} | ₹${this.salary.toLocaleString()}`;
  }
  
  getAnnualSalary() {
    return this.salary * 12;
  }
}

// Child class (extends Employee):
class Manager extends Employee {
  constructor(name, department, salary, teamSize) {
    super(name, department, salary);  // Call parent constructor
    this.teamSize = teamSize;
    this.role = "Manager";
  }
  
  // New method (only managers have this):
  approveLeave(employeeName) {
    return `${this.name} approved leave for ${employeeName}`;
  }
  
  // Override parent method:
  getDetails() {
    return `${super.getDetails()} | Team: ${this.teamSize} people | Role: ${this.role}`;
  }
}

// Child class (Developer):
class Developer extends Employee {
  constructor(name, department, salary, skills) {
    super(name, department, salary);
    this.skills = skills;
    this.role = "Developer";
  }
  
  code(feature) {
    return `${this.name} is coding: ${feature}`;
  }
  
  getDetails() {
    return `${super.getDetails()} | Skills: ${this.skills.join(", ")}`;
  }
}

// Usage:
const manager = new Manager("Priya", "Engineering", 120000, 8);
const dev = new Developer("Rahul", "Engineering", 80000, ["JavaScript", "Node.js", "CAP"]);

console.log(manager.getDetails());
// "Priya | Engineering | ₹1,20,000 | Team: 8 people | Role: Manager"

console.log(dev.getDetails());
// "Rahul | Engineering | ₹80,000 | Skills: JavaScript, Node.js, CAP"

console.log(manager.approveLeave("Rahul"));
// "Priya approved leave for Rahul"

console.log(dev.code("Purchase Order validation"));
// "Rahul is coding: Purchase Order validation"

// Both have parent's method:
console.log(manager.getAnnualSalary());  // 1440000
console.log(dev.getAnnualSalary());      // 960000
```

---

### Static Methods

Static methods belong to the CLASS, not to instances:

```javascript
class MathUtils {
  static add(a, b) { return a + b; }
  static multiply(a, b) { return a * b; }
  static percentage(value, total) { return ((value / total) * 100).toFixed(2) + "%"; }
  static gst(amount, rate = 18) { return amount * (rate / 100); }
}

// Called on the CLASS directly (no 'new' needed):
console.log(MathUtils.add(5, 3));            // 8
console.log(MathUtils.gst(10000));           // 1800
console.log(MathUtils.percentage(75, 100));  // "75.00%"
```

---

### Modules — import/export (ES6)

In real projects, code is split across multiple files. Modules let files share code:

#### Exporting (file: `utils.js`)

```javascript
// Named exports:
export const formatCurrency = (amount) => `₹${amount.toLocaleString()}`;
export const calculateGST = (amount, rate = 18) => amount * (rate / 100);

// Default export (one per file):
export default class OrderService {
  constructor() {
    this.orders = [];
  }
  
  addOrder(order) {
    this.orders.push(order);
  }
  
  getAll() {
    return this.orders;
  }
}
```

#### Importing (file: `app.js`)

```javascript
// Import default export:
import OrderService from './utils.js';

// Import named exports:
import { formatCurrency, calculateGST } from './utils.js';

// Import everything:
import * as Utils from './utils.js';

// Usage:
const service = new OrderService();
console.log(formatCurrency(50000));     // "₹50,000"
console.log(calculateGST(10000));       // 1800
```

#### CommonJS (Node.js style — what CAP uses)

```javascript
// Exporting (utils.js):
const formatCurrency = (amount) => `₹${amount.toLocaleString()}`;
const calculateGST = (amount) => amount * 0.18;
module.exports = { formatCurrency, calculateGST };

// Importing (app.js):
const { formatCurrency, calculateGST } = require('./utils');
console.log(formatCurrency(50000));
```

**For CAP development:** We primarily use `module.exports` / `require` (CommonJS), since Node.js in CAP uses this format.

---

## Session 5: Error Handling & Mini Library (15:15 - 16:00)

### Why Error Handling?

Without error handling, one error CRASHES your entire application:

```javascript
// This crashes everything:
const data = JSON.parse("invalid json");  // 💥 SyntaxError!
console.log("This never runs...");        // Never reached
```

With error handling, you gracefully manage failures:

```javascript
try {
  const data = JSON.parse("invalid json");
} catch (error) {
  console.log("Oops! Invalid data format.");
  console.log("Error:", error.message);
}
console.log("App continues running! ✅");  // Still runs!
```

---

### try / catch / finally

```javascript
try {
  // Code that MIGHT fail:
  let result = riskyOperation();
  console.log("Success:", result);
  
} catch (error) {
  // Runs ONLY if try-block throws an error:
  console.log("Error caught:", error.message);
  
} finally {
  // Runs ALWAYS (success or failure):
  console.log("Cleanup complete");
}
```

| Block | When It Runs | Use For |
|-------|-------------|---------|
| `try` | Always (first) | Code that might fail |
| `catch` | Only if try fails | Handle the error gracefully |
| `finally` | Always (last) | Cleanup (close files, connections) |

---

### Throwing Custom Errors

You can throw your own errors for validation:

```javascript
const createOrder = (vendor, amount) => {
  // Validation — throw errors for invalid input:
  if (!vendor || vendor.trim() === "") {
    throw new Error("Vendor name is required");
  }
  if (typeof amount !== "number" || amount <= 0) {
    throw new Error("Amount must be a positive number");
  }
  if (amount > 1000000) {
    throw new Error("Amount exceeds maximum limit of ₹10,00,000");
  }
  
  return {
    id: `PO-${Date.now()}`,
    vendor,
    amount,
    status: "Created"
  };
};

// Usage with error handling:
try {
  const order1 = createOrder("TechCorp", 50000);
  console.log("✅ Order created:", order1);
  
  const order2 = createOrder("", 5000);  // This will throw!
  console.log("This won't run");
  
} catch (error) {
  console.log(`❌ Failed: ${error.message}`);
}
// ❌ Failed: Vendor name is required
```

---

### Custom Error Classes

```javascript
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

class NotFoundError extends Error {
  constructor(entity, id) {
    super(`${entity} with ID '${id}' not found`);
    this.name = "NotFoundError";
    this.entity = entity;
    this.id = id;
  }
}

// Usage:
const findOrder = (orders, id) => {
  const order = orders.find(o => o.id === id);
  if (!order) {
    throw new NotFoundError("Order", id);
  }
  return order;
};

try {
  const order = findOrder([], "PO-999");
} catch (error) {
  if (error instanceof NotFoundError) {
    console.log(`🔍 ${error.message}`);  // "Order with ID 'PO-999' not found"
  } else if (error instanceof ValidationError) {
    console.log(`⚠️ Validation: ${error.field} - ${error.message}`);
  } else {
    console.log(`💥 Unexpected: ${error.message}`);
  }
}
```

---

### Hands-on: Build a Mini Library System (20 minutes)

Create `library.js`:

```javascript
// ============================================
//  MINI LIBRARY MANAGEMENT SYSTEM
//  Using Classes, Error Handling, and Closures
// ============================================

class Book {
  constructor(title, author, isbn, copies = 1) {
    if (!title || !author || !isbn) {
      throw new Error("Title, author, and ISBN are required");
    }
    this.title = title;
    this.author = author;
    this.isbn = isbn;
    this.totalCopies = copies;
    this.availableCopies = copies;
    this.borrowHistory = [];
  }
  
  getInfo() {
    return `"${this.title}" by ${this.author} [${this.isbn}] — ${this.availableCopies}/${this.totalCopies} available`;
  }
}

class Member {
  constructor(name, memberId) {
    this.name = name;
    this.memberId = memberId;
    this.borrowedBooks = [];
    this.borrowLimit = 3;
  }
  
  canBorrow() {
    return this.borrowedBooks.length < this.borrowLimit;
  }
}

class Library {
  constructor(name) {
    this.name = name;
    this.books = [];
    this.members = [];
    this.transactions = [];
  }
  
  // Add a book to the library
  addBook(title, author, isbn, copies) {
    const existing = this.books.find(b => b.isbn === isbn);
    if (existing) {
      existing.totalCopies += copies;
      existing.availableCopies += copies;
      return `Updated: "${title}" now has ${existing.totalCopies} copies`;
    }
    const book = new Book(title, author, isbn, copies);
    this.books.push(book);
    return `Added: "${title}" (${copies} copies)`;
  }
  
  // Register a new member
  addMember(name) {
    const id = `MEM-${String(this.members.length + 1).padStart(3, "0")}`;
    const member = new Member(name, id);
    this.members.push(member);
    return `Welcome, ${name}! Your ID: ${id}`;
  }
  
  // Borrow a book
  borrowBook(memberId, isbn) {
    const member = this.members.find(m => m.memberId === memberId);
    if (!member) throw new Error(`Member ${memberId} not found`);
    
    const book = this.books.find(b => b.isbn === isbn);
    if (!book) throw new Error(`Book with ISBN ${isbn} not found`);
    
    if (!member.canBorrow()) {
      throw new Error(`${member.name} has reached the borrow limit (${member.borrowLimit} books)`);
    }
    
    if (book.availableCopies <= 0) {
      throw new Error(`"${book.title}" is not available. All copies are borrowed.`);
    }
    
    // Process borrow:
    book.availableCopies--;
    member.borrowedBooks.push(isbn);
    book.borrowHistory.push({ memberId, date: new Date().toISOString(), action: "borrow" });
    
    this.transactions.push({
      type: "BORROW",
      memberId,
      isbn,
      date: new Date().toISOString()
    });
    
    return `✅ ${member.name} borrowed "${book.title}" (${book.availableCopies} copies remaining)`;
  }
  
  // Return a book
  returnBook(memberId, isbn) {
    const member = this.members.find(m => m.memberId === memberId);
    if (!member) throw new Error(`Member ${memberId} not found`);
    
    const book = this.books.find(b => b.isbn === isbn);
    if (!book) throw new Error(`Book with ISBN ${isbn} not found`);
    
    const borrowIndex = member.borrowedBooks.indexOf(isbn);
    if (borrowIndex === -1) {
      throw new Error(`${member.name} hasn't borrowed "${book.title}"`);
    }
    
    // Process return:
    book.availableCopies++;
    member.borrowedBooks.splice(borrowIndex, 1);
    book.borrowHistory.push({ memberId, date: new Date().toISOString(), action: "return" });
    
    this.transactions.push({
      type: "RETURN",
      memberId,
      isbn,
      date: new Date().toISOString()
    });
    
    return `✅ ${member.name} returned "${book.title}" (${book.availableCopies} copies available)`;
  }
  
  // Search books
  searchBooks(query) {
    const q = query.toLowerCase();
    return this.books.filter(b => 
      b.title.toLowerCase().includes(q) || 
      b.author.toLowerCase().includes(q)
    );
  }
  
  // Get library stats
  getStats() {
    return {
      totalBooks: this.books.length,
      totalCopies: this.books.reduce((sum, b) => sum + b.totalCopies, 0),
      availableCopies: this.books.reduce((sum, b) => sum + b.availableCopies, 0),
      totalMembers: this.members.length,
      totalTransactions: this.transactions.length
    };
  }
  
  // Display all books
  displayBooks() {
    console.log(`\n📚 ${this.name} — Book Catalog:`);
    console.log("─".repeat(70));
    this.books.forEach(book => console.log(`  ${book.getInfo()}`));
    console.log("─".repeat(70));
  }
}

// ========== TEST THE LIBRARY ==========

const lib = new Library("SAP Academy Library");

// Add books:
console.log(lib.addBook("Clean Code", "Robert Martin", "978-0132350884", 3));
console.log(lib.addBook("JavaScript: The Good Parts", "Douglas Crockford", "978-0596517748", 2));
console.log(lib.addBook("Design Patterns", "Gang of Four", "978-0201633610", 1));
console.log(lib.addBook("Node.js in Action", "Mike Cantelon", "978-1617290572", 4));

// Add members:
console.log(lib.addMember("Priya Sharma"));
console.log(lib.addMember("Rahul Kumar"));

// Display catalog:
lib.displayBooks();

// Borrow books:
try {
  console.log(lib.borrowBook("MEM-001", "978-0132350884"));
  console.log(lib.borrowBook("MEM-001", "978-0596517748"));
  console.log(lib.borrowBook("MEM-002", "978-0132350884"));
  
  // Try borrowing unavailable book:
  // console.log(lib.borrowBook("MEM-002", "978-0201633610"));
  // console.log(lib.borrowBook("MEM-002", "978-0201633610"));  // Would fail!
} catch (error) {
  console.log(`❌ Error: ${error.message}`);
}

// Return a book:
console.log(lib.returnBook("MEM-001", "978-0132350884"));

// Search:
console.log("\n🔍 Search results for 'java':");
lib.searchBooks("java").forEach(b => console.log(`  ${b.getInfo()}`));

// Stats:
console.log("\n📊 Library Stats:", lib.getStats());
```

---

## Session 6: Live Coding Quiz (16:00 - 16:30)

### 5 Problems — 30 Minutes — Code It Live!

Open VS Code. Create `quiz.js`. Solve each problem. Run with `node quiz.js`.

---

**Problem 1: Function Factory (5 min)**

Create a function `createTaxCalculator(rate)` that returns a new function. The returned function takes an amount and returns the tax.

```javascript
// Expected behavior:
const gstCalc = createTaxCalculator(18);
const vatCalc = createTaxCalculator(5);

console.log(gstCalc(10000));  // 1800
console.log(vatCalc(10000));  // 500
console.log(gstCalc(5000));   // 900
```

---

**Problem 2: Array Transformer (5 min)**

Given an array of student objects, use ONLY `map`, `filter`, and `reduce` (no loops!) to:
1. Filter students with grade >= 80
2. Add a "status" field: "Distinction" if >= 90, else "First Class"
3. Calculate total marks of filtered students

```javascript
const students = [
  { name: "Arun", grade: 92 },
  { name: "Priya", grade: 85 },
  { name: "Rahul", grade: 67 },
  { name: "Sneha", grade: 95 },
  { name: "Vikram", grade: 78 }
];

// Your code → Expected output:
// Filtered: [{name: "Arun", grade: 92, status: "Distinction"}, ...]
// Total marks of top students: 272
```

---

**Problem 3: Error-Safe Parser (5 min)**

Write a function `safeJsonParse(jsonString)` that:
- Returns the parsed object if valid JSON
- Returns `{ error: true, message: "..." }` if invalid
- Never crashes!

```javascript
console.log(safeJsonParse('{"name": "Priya"}'));  // { name: "Priya" }
console.log(safeJsonParse('invalid!!!'));          // { error: true, message: "..." }
console.log(safeJsonParse(''));                    // { error: true, message: "..." }
```

---

**Problem 4: Class Inheritance (8 min)**

Create a `Shape` base class and two child classes `Circle` and `Rectangle`:
- `Shape` has: `color`, `getColor()` method
- `Circle` has: `radius`, `getArea()` → π × r²
- `Rectangle` has: `width`, `height`, `getArea()` → w × h
- Both have `describe()` → "A [color] [shape] with area [area]"

```javascript
const c = new Circle("red", 5);
console.log(c.describe());  // "A red circle with area 78.54"

const r = new Rectangle("blue", 4, 6);
console.log(r.describe());  // "A blue rectangle with area 24.00"
```

---

**Problem 5: Mini Event System (7 min)**

Build a simple event emitter using closures:

```javascript
const createEventEmitter = () => {
  // Your code: return { on, emit, off }
};

const emitter = createEventEmitter();

emitter.on("orderCreated", (data) => console.log(`New order: ${data.id}`));
emitter.on("orderCreated", (data) => console.log(`Amount: ₹${data.amount}`));
emitter.on("error", (msg) => console.log(`ERROR: ${msg}`));

emitter.emit("orderCreated", { id: "PO-001", amount: 50000 });
// "New order: PO-001"
// "Amount: ₹50000"

emitter.emit("error", "Something went wrong");
// "ERROR: Something went wrong"
```

---

<details>
<summary>Quiz Solutions</summary>

```javascript
// Problem 1:
const createTaxCalculator = (rate) => (amount) => amount * (rate / 100);

// Problem 2:
const topStudents = students
  .filter(s => s.grade >= 80)
  .map(s => ({ ...s, status: s.grade >= 90 ? "Distinction" : "First Class" }));
const totalMarks = topStudents.reduce((sum, s) => sum + s.grade, 0);

// Problem 3:
const safeJsonParse = (jsonString) => {
  try {
    if (!jsonString) throw new Error("Empty input");
    return JSON.parse(jsonString);
  } catch (error) {
    return { error: true, message: error.message };
  }
};

// Problem 4:
class Shape {
  constructor(color) { this.color = color; }
  getColor() { return this.color; }
}
class Circle extends Shape {
  constructor(color, radius) { super(color); this.radius = radius; }
  getArea() { return Math.PI * this.radius ** 2; }
  describe() { return `A ${this.color} circle with area ${this.getArea().toFixed(2)}`; }
}
class Rectangle extends Shape {
  constructor(color, width, height) { super(color); this.width = width; this.height = height; }
  getArea() { return this.width * this.height; }
  describe() { return `A ${this.color} rectangle with area ${this.getArea().toFixed(2)}`; }
}

// Problem 5:
const createEventEmitter = () => {
  const listeners = {};
  return {
    on: (event, callback) => {
      if (!listeners[event]) listeners[event] = [];
      listeners[event].push(callback);
    },
    emit: (event, data) => {
      if (listeners[event]) listeners[event].forEach(cb => cb(data));
    },
    off: (event, callback) => {
      if (listeners[event]) {
        listeners[event] = listeners[event].filter(cb => cb !== callback);
      }
    }
  };
};
```

</details>

---

## Assessment: MCQ (15 Questions)

**Q1.** What is the key difference between a function declaration and an arrow function?
- a) Arrow functions are slower
- b) Arrow functions don't have their own `this` binding
- c) Arrow functions can't return values
- d) Arrow functions can't take parameters

<details><summary>Answer</summary>b) Arrow functions inherit `this` from their parent scope instead of having their own. This is crucial when working with classes and callbacks.</details>

---

**Q2.** `const add = (a, b) => a + b;` — This arrow function works because:
- a) It has no parameters
- b) The single expression is automatically returned (implicit return)
- c) Arrow functions always return undefined
- d) It uses the `return` keyword internally

<details><summary>Answer</summary>b) When an arrow function body is a single expression (no curly braces), the result is automatically returned — no explicit `return` needed.</details>

---

**Q3.** What is a closure?
- a) A way to close a file
- b) A function that remembers variables from its creation scope, even after that scope is gone
- c) A method to terminate a program
- d) A type of loop

<details><summary>Answer</summary>b) A closure is a function bundled with its surrounding state — it "remembers" and can access variables from the scope where it was created.</details>

---

**Q4.** What is a callback function?
- a) A function that calls itself
- b) A function passed as an argument to another function, to be called later
- c) A function that returns another function
- d) A function that runs automatically

<details><summary>Answer</summary>b) A callback is a function passed to another function — the receiving function decides when to "call back" (execute) it.</details>

---

**Q5.** `const greet = (name = "Guest") => ...` — What does `= "Guest"` do?
- a) Creates a constant named Guest
- b) Provides a default value if no argument is passed
- c) Throws an error if name is Guest
- d) Assigns "Guest" to all parameters

<details><summary>Answer</summary>b) Default parameter — if `greet()` is called without an argument, `name` will be "Guest" instead of `undefined`.</details>

---

**Q6.** What does `...` mean in `function sum(...nums) { }`?
- a) Spread operator — expands the array
- b) Rest parameter — collects all arguments into an array called `nums`
- c) Syntax error
- d) Optional parameter

<details><summary>Answer</summary>b) Rest parameter — collects ALL passed arguments into a single array. `sum(1,2,3)` → nums = [1,2,3]</details>

---

**Q7.** In a class, the `constructor` method:
- a) Destroys the object
- b) Runs automatically when you create a new instance with `new`
- c) Is optional and never runs
- d) Can only be called manually

<details><summary>Answer</summary>b) The constructor is called automatically when you write `new ClassName(...)` — it initializes the new object's properties.</details>

---

**Q8.** `class Dog extends Animal` means:
- a) Dog replaces Animal
- b) Dog inherits all properties and methods from Animal
- c) Dog and Animal are the same class
- d) Animal inherits from Dog

<details><summary>Answer</summary>b) `extends` creates inheritance — Dog gets all of Animal's properties and methods, and can add its own or override them.</details>

---

**Q9.** What does `super()` do in a child class constructor?
- a) Calls the parent class's constructor
- b) Deletes the parent class
- c) Creates a new parent instance
- d) Skips the constructor

<details><summary>Answer</summary>a) `super()` calls the parent's constructor to initialize inherited properties. Must be called before using `this` in a child constructor.</details>

---

**Q10.** What does `try/catch` do?
- a) Tries to catch bugs during development
- b) Attempts code in `try` block; if it throws an error, handles it in `catch` block
- c) Makes code run faster
- d) Tests if a variable exists

<details><summary>Answer</summary>b) `try` runs code that might fail; if an error is thrown, execution jumps to `catch` where you handle it gracefully instead of crashing.</details>

---

**Q11.** What is a higher-order function?
- a) A function defined at the top of a file
- b) A function that takes another function as argument or returns a function
- c) A function with more than 5 parameters
- d) A function that runs before all others

<details><summary>Answer</summary>b) Higher-order functions operate on other functions — either by taking them as arguments (like `.map()`) or by returning them.</details>

---

**Q12.** `module.exports = { add, subtract }` does what?
- a) Imports functions from another file
- b) Exports/exposes `add` and `subtract` so other files can use them
- c) Deletes the functions
- d) Creates a new module

<details><summary>Answer</summary>b) `module.exports` makes functions/values available to other files that `require()` this module. It's how you share code between files in Node.js.</details>

---

**Q13.** A static method in a class:
- a) Cannot be changed
- b) Belongs to the class itself, not to instances
- c) Runs automatically
- d) Is always private

<details><summary>Answer</summary>b) Static methods are called on the class directly (`ClassName.method()`) — you don't need to create an instance with `new` first.</details>

---

**Q14.** What happens if you throw an error inside a `try` block but have no `catch`?
- a) Nothing — errors are always caught
- b) The error propagates up and crashes the program (unhandled)
- c) The `finally` block catches it
- d) JavaScript ignores thrown errors

<details><summary>Answer</summary>b) Without a `catch` block, the error is unhandled and will crash the program (or propagate to an outer try/catch if one exists). `finally` runs but doesn't catch errors.</details>

---

**Q15.** Which is the correct arrow function to return an object `{ x: 1, y: 2 }`?
- a) `() => { x: 1, y: 2 }`
- b) `() => ({ x: 1, y: 2 })`
- c) `() => return { x: 1, y: 2 }`
- d) `() => [x: 1, y: 2]`

<details><summary>Answer</summary>b) `() => ({ x: 1, y: 2 })` — You must wrap the object in parentheses, otherwise JS thinks `{}` is the function body, not an object literal.</details>

---

## Key Takeaways

| # | Topic | One-Line Summary |
|---|---|---|
| 1 | Function Declaration | `function name() {}` — hoisted, named, classic |
| 2 | Arrow Function | `() => {}` — concise, no own `this`, used everywhere in modern JS |
| 3 | Default Parameters | `(x = 10)` — fallback value if argument is missing |
| 4 | Rest Parameters | `(...args)` — collects all arguments into an array |
| 5 | Callbacks | Functions passed to other functions, called later |
| 6 | Higher-Order Functions | Functions that accept or return other functions (map, filter) |
| 7 | Closures | Functions remember variables from their creation scope |
| 8 | Classes | Blueprints for objects — constructor + methods |
| 9 | Inheritance | `extends` — child class gets parent's features |
| 10 | Static Methods | Belong to the class, not instances — `Class.method()` |
| 11 | Modules (CJS) | `module.exports` / `require()` — share code between files |
| 12 | Modules (ESM) | `export` / `import` — modern module syntax |
| 13 | try/catch | Gracefully handle errors without crashing |
| 14 | throw | Create and throw custom errors for validation |
| 15 | Scope Chain | JS finds variables: local → parent → grandparent → global |

---

## Preparation for Day 9

Tomorrow: **Asynchronous JavaScript & Node.js Introduction**

You'll learn:
- Synchronous vs Asynchronous programming
- The Event Loop
- Callbacks and "callback hell"
- Promises (create, chain, all, race)
- Async/Await syntax
- Introduction to Node.js architecture
- npm, package.json, and your first Node.js project

**To prepare:**
- Make sure you're comfortable with callbacks and closures (review today's exercises)
- Async programming is the HARDEST part of JavaScript for beginners — tomorrow requires focus!
- Optional: Read about "Why JavaScript is single-threaded"

---

*End of Day 8*
