# Day 7: JavaScript Basics — Part 2

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 6 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Control Flow — if/else, switch, ternary | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Loops — for, while, do-while, for...of, for...in | 75 min |
| 12:00 - 13:00 | Session 3: Arrays — Creation, Methods & Manipulation | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 15:00 | Session 4: Objects, Destructuring, Spread & Rest | 75 min |
| 15:00 - 15:15 | Break | 15 min |
| 15:15 - 16:00 | Session 5: JSON — Parse, Stringify & Real-World Usage | 45 min |
| 16:00 - 16:45 | Session 6: Hands-on — Grade Calculator & Exercises | 45 min |
| 16:45 - 17:00 | Assessment & Wrap-up | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Write conditional logic using if/else, switch, and ternary operators
- Create loops to repeat actions (for, while, for...of)
- Create and manipulate arrays using powerful built-in methods
- Create objects, access properties, and use destructuring
- Use spread (`...`) and rest (`...`) operators
- Parse and create JSON data (the format APIs use everywhere)
- Build a complete student grade calculator program

---

## Day 6 Recap — Quick Fire (09:00 - 09:15)

Answer in 10 seconds each:

1. Name the 3 ways to declare a variable → _____, _____, _____
2. Which one should you NEVER use? → _____
3. What does `typeof null` return? → _____
4. What does `"5" + 3` equal? → _____
5. Which equality operator should you ALWAYS use? → _____
6. Name 3 string methods → _____, _____, _____
7. What are backtick strings called? → _____
8. Name the 6 falsy values → _____, _____, _____, _____, _____, _____

<details>
<summary>Answers</summary>

1. `var`, `let`, `const`
2. `var` (function-scoped, leaks out of blocks)
3. `"object"` (famous JavaScript bug!)
4. `"53"` (string concatenation, not addition)
5. `===` (strict equality — no type coercion)
6. `.toUpperCase()`, `.toLowerCase()`, `.trim()`, `.split()`, `.includes()`, `.slice()` (any 3)
7. Template literals
8. `false`, `0`, `""`, `null`, `undefined`, `NaN`

</details>

---

## Session 1: Control Flow — if/else, switch, ternary (09:15 - 10:30)

### Why Control Flow?

Without control flow, code runs top to bottom, every line, every time. That's boring and useless!

**Control flow** lets your program make **decisions** — like a GPS: "If the road is blocked, take the alternate route."

```
Without control flow:         With control flow:
Line 1 ✓                     Line 1 ✓
Line 2 ✓                     IF (condition) → Line 2A ✓
Line 3 ✓                     ELSE → Line 2B (skipped)
Line 4 ✓                     Line 3 ✓
(Always same path)            (Different paths based on data!)
```

---

### if / else — The Basic Decision

```javascript
let temperature = 35;

if (temperature > 30) {
  console.log("It's hot! Stay hydrated 🥵");
} else {
  console.log("Nice weather! Enjoy outside 😊");
}
// Output: "It's hot! Stay hydrated 🥵"
```

**Structure:**
```javascript
if (condition) {
  // runs if condition is TRUE
} else {
  // runs if condition is FALSE
}
```

---

### if / else if / else — Multiple Conditions

```javascript
let score = 75;

if (score >= 90) {
  console.log("Grade: A — Excellent!");
} else if (score >= 80) {
  console.log("Grade: B — Very Good!");
} else if (score >= 70) {
  console.log("Grade: C — Good");
} else if (score >= 60) {
  console.log("Grade: D — Pass");
} else {
  console.log("Grade: F — Fail");
}
// Output: "Grade: C — Good"
```

#### SAP Context: Order Approval Logic

```javascript
let orderAmount = 75000;
let role = "Manager";
let isUrgent = false;

if (orderAmount > 100000) {
  console.log("Requires VP Approval");
} else if (orderAmount > 50000 && role !== "Manager") {
  console.log("Requires Manager Approval");
} else if (isUrgent) {
  console.log("Auto-approved (urgent, under limit)");
} else {
  console.log("Auto-approved");
}
// Output: "Auto-approved"
```

---

### Nested if/else

```javascript
let age = 20;
let hasID = true;

if (age >= 18) {
  if (hasID) {
    console.log("Entry allowed ✅");
  } else {
    console.log("Bring your ID next time ❌");
  }
} else {
  console.log("Too young — entry denied ❌");
}
```

**Tip:** Deeply nested if/else is hard to read. Prefer `else if` chains or early returns.

---

### switch — Clean Multiple Choice

When you have many exact-value comparisons, `switch` is cleaner than long if/else chains:

```javascript
let day = "Wednesday";

switch (day) {
  case "Monday":
    console.log("Start of the work week 😩");
    break;
  case "Tuesday":
  case "Wednesday":
  case "Thursday":
    console.log("Midweek grind 💪");
    break;
  case "Friday":
    console.log("TGIF! Almost weekend 🎉");
    break;
  case "Saturday":
  case "Sunday":
    console.log("Weekend! Relax 🏖️");
    break;
  default:
    console.log("Invalid day");
}
// Output: "Midweek grind 💪"
```

**Important:** Always include `break`! Without it, execution "falls through" to the next case.

#### SAP Context: Order Status Handler

```javascript
let status = "APPROVED";

switch (status) {
  case "DRAFT":
    console.log("Order is in draft — editable");
    break;
  case "SUBMITTED":
    console.log("Order submitted — awaiting approval");
    break;
  case "APPROVED":
    console.log("Order approved — ready for processing");
    break;
  case "REJECTED":
    console.log("Order rejected — review comments");
    break;
  case "COMPLETED":
    console.log("Order completed — no further action");
    break;
  default:
    console.log(`Unknown status: ${status}`);
}
```

---

### Ternary Operator — One-Line if/else

For simple conditions, use the ternary operator (`? :`):

```javascript
// Syntax: condition ? valueIfTrue : valueIfFalse

let age = 20;
let canVote = age >= 18 ? "Yes" : "No";
console.log(`Can vote: ${canVote}`);  // "Can vote: Yes"

// Same as:
// if (age >= 18) { canVote = "Yes"; } else { canVote = "No"; }
```

```javascript
// More examples:
let temp = 35;
let weather = temp > 30 ? "Hot" : "Cool";

let stock = 0;
let availability = stock > 0 ? "In Stock" : "Out of Stock";

let score = 85;
let result = score >= 60 ? "Pass" : "Fail";

// Nested ternary (use sparingly — hard to read!):
let grade = score >= 90 ? "A" : score >= 80 ? "B" : score >= 70 ? "C" : "F";
```

**Rule:** Use ternary for simple one-value assignments. Use if/else for complex logic with multiple statements.

---

### Practice: Control Flow (5 minutes)

```javascript
// Try this: What gets printed?
let x = 15;

if (x > 20) {
  console.log("A");
} else if (x > 10) {
  console.log("B");
} else if (x > 5) {
  console.log("C");
} else {
  console.log("D");
}
// Answer: "B" (first matching condition wins, rest are skipped)
```

---

## Session 2: Loops (10:45 - 12:00)

### Why Loops?

Loops repeat a block of code multiple times. Without loops:

```javascript
// Print numbers 1 to 5 without a loop:
console.log(1);
console.log(2);
console.log(3);
console.log(4);
console.log(5);
// What if you need 1 to 1000? 😱
```

With a loop:
```javascript
for (let i = 1; i <= 5; i++) {
  console.log(i);
}
// Works for 5, 50, or 5 million!
```

---

### 1. for Loop — The Classic

```javascript
// Syntax: for (initialize; condition; update)
for (let i = 1; i <= 5; i++) {
  console.log(`Iteration ${i}`);
}
// Output: Iteration 1, Iteration 2, ... Iteration 5
```

**How it works:**

```
Step 1: let i = 1        (initialize — runs ONCE at start)
Step 2: is i <= 5? YES   (condition check)
Step 3: run the code     (loop body)
Step 4: i++              (update — i becomes 2)
Step 5: is i <= 5? YES   (condition check again)
... repeats until condition is FALSE
```

#### Practical Examples

```javascript
// Sum numbers 1 to 100:
let sum = 0;
for (let i = 1; i <= 100; i++) {
  sum += i;
}
console.log(`Sum of 1-100 = ${sum}`);  // 5050

// Print multiplication table:
let num = 7;
for (let i = 1; i <= 10; i++) {
  console.log(`${num} × ${i} = ${num * i}`);
}

// Countdown:
for (let i = 10; i >= 1; i--) {
  console.log(i);
}
console.log("🚀 Launch!");
```

---

### 2. while Loop — Condition First

Runs while a condition is true. Use when you don't know how many iterations you need:

```javascript
let count = 1;
while (count <= 5) {
  console.log(`Count: ${count}`);
  count++;
}
```

```javascript
// Real use case: Keep doubling until you exceed 1000
let value = 1;
let doublings = 0;
while (value < 1000) {
  value *= 2;
  doublings++;
}
console.log(`Took ${doublings} doublings to exceed 1000. Final value: ${value}`);
// Took 10 doublings. Final value: 1024
```

---

### 3. do...while Loop — Run At Least Once

```javascript
let input = "";
do {
  input = "some value"; // In real app, this would be user input
  console.log(`Processing: ${input}`);
} while (input === "");
// Body runs at LEAST ONCE, then checks condition
```

**Difference from while:** `do...while` always runs the body at least once, even if the condition is false from the start.

---

### 4. for...of — Loop Through Values (Arrays, Strings)

```javascript
let fruits = ["Apple", "Banana", "Mango", "Orange"];

for (let fruit of fruits) {
  console.log(`I like ${fruit}`);
}
// I like Apple
// I like Banana
// I like Mango
// I like Orange
```

```javascript
// Loop through characters of a string:
let word = "SAP";
for (let char of word) {
  console.log(char);
}
// S, A, P
```

**Use `for...of` when:** You want the VALUES and don't need the index.

---

### 5. for...in — Loop Through Object Keys

```javascript
let employee = {
  name: "Priya",
  age: 28,
  department: "Engineering",
  city: "Bangalore"
};

for (let key in employee) {
  console.log(`${key}: ${employee[key]}`);
}
// name: Priya
// age: 28
// department: Engineering
// city: Bangalore
```

**Use `for...in` when:** You want to loop through an object's property names (keys).

---

### Loop Control: break & continue

```javascript
// break — EXIT the loop immediately
for (let i = 1; i <= 10; i++) {
  if (i === 5) break;
  console.log(i);
}
// Output: 1, 2, 3, 4 (stops at 5, doesn't print it)

// continue — SKIP this iteration, go to next
for (let i = 1; i <= 10; i++) {
  if (i % 2 === 0) continue;  // Skip even numbers
  console.log(i);
}
// Output: 1, 3, 5, 7, 9 (only odd numbers)
```

---

### Loop Comparison Table

| Loop | Best For | Example Use Case |
|------|----------|-----------------|
| `for` | Known number of iterations | "Do this 10 times" |
| `while` | Unknown iterations, condition-based | "Keep trying until success" |
| `do...while` | Must run at least once | "Ask for input, validate, repeat" |
| `for...of` | Iterating array/string values | "Process each item in a list" |
| `for...in` | Iterating object keys | "List all properties of an object" |

---

## Session 3: Arrays — Creation, Methods & Manipulation (12:00 - 13:00)

### What is an Array?

An **Array** is an ordered list of values. Think of it as a **numbered shelf**:

```
Array: ["Apple", "Banana", "Cherry", "Date"]
Index:     0         1         2        3

Shelf analogy:
+--------+--------+--------+--------+
| Slot 0 | Slot 1 | Slot 2 | Slot 3 |
| Apple  | Banana | Cherry | Date   |
+--------+--------+--------+--------+
```

---

### Creating Arrays

```javascript
// Method 1: Array literal (most common)
let fruits = ["Apple", "Banana", "Mango"];
let numbers = [10, 20, 30, 40, 50];
let mixed = ["Hello", 42, true, null];  // Can mix types (but don't!)

// Method 2: Empty array, add later
let orders = [];
orders.push("ORD-001");
orders.push("ORD-002");

// Method 3: Array.from (create from other things)
let letters = Array.from("HELLO");  // ["H", "E", "L", "L", "O"]
```

---

### Accessing & Modifying Arrays

```javascript
let colors = ["Red", "Green", "Blue", "Yellow"];

// Access by index (0-based):
console.log(colors[0]);   // "Red" (first)
console.log(colors[2]);   // "Blue" (third)
console.log(colors[colors.length - 1]);  // "Yellow" (last)

// Modify by index:
colors[1] = "Purple";     // Replace "Green" with "Purple"
console.log(colors);      // ["Red", "Purple", "Blue", "Yellow"]

// Array length:
console.log(colors.length);  // 4
```

---

### Essential Array Methods

#### Adding & Removing

| Method | What It Does | Example | Returns |
|--------|-------------|---------|---------|
| `.push(item)` | Add to END | `arr.push("new")` | New length |
| `.pop()` | Remove from END | `arr.pop()` | Removed item |
| `.unshift(item)` | Add to START | `arr.unshift("first")` | New length |
| `.shift()` | Remove from START | `arr.shift()` | Removed item |
| `.splice(i, n)` | Remove n items at index i | `arr.splice(1, 2)` | Removed items |

```javascript
let stack = ["A", "B", "C"];

stack.push("D");       // ["A", "B", "C", "D"]
stack.pop();           // ["A", "B", "C"] → returns "D"
stack.unshift("Z");    // ["Z", "A", "B", "C"]
stack.shift();         // ["A", "B", "C"] → returns "Z"
stack.splice(1, 1);    // ["A", "C"] → removes "B" at index 1
```

---

#### Searching

| Method | What It Does | Example | Returns |
|--------|-------------|---------|---------|
| `.indexOf(item)` | Find index of item | `arr.indexOf("B")` | Index or -1 |
| `.includes(item)` | Check if item exists | `arr.includes("B")` | true/false |
| `.find(fn)` | Find first matching item | `arr.find(x => x > 5)` | Item or undefined |
| `.findIndex(fn)` | Find index of first match | `arr.findIndex(x => x > 5)` | Index or -1 |

```javascript
let nums = [10, 25, 30, 45, 50];

console.log(nums.indexOf(30));          // 2
console.log(nums.includes(99));         // false
console.log(nums.find(n => n > 20));    // 25 (first match)
console.log(nums.findIndex(n => n > 40)); // 3 (index of 45)
```

---

#### The Big Three: map, filter, reduce

These are the MOST IMPORTANT array methods. You'll use them daily!

##### `.map()` — Transform Every Item

Creates a NEW array by applying a function to each item:

```javascript
let prices = [100, 200, 300, 400];

// Add 18% GST to each price:
let withGST = prices.map(price => price * 1.18);
console.log(withGST);  // [118, 236, 354, 472]

// Get just the names from objects:
let users = [
  { name: "Arun", age: 25 },
  { name: "Priya", age: 30 },
  { name: "Rahul", age: 22 }
];
let names = users.map(user => user.name);
console.log(names);  // ["Arun", "Priya", "Rahul"]
```

##### `.filter()` — Keep Only Matching Items

Creates a NEW array with only items that pass a test:

```javascript
let numbers = [5, 12, 8, 130, 44, 3, 99];

// Keep only numbers greater than 10:
let big = numbers.filter(n => n > 10);
console.log(big);  // [12, 130, 44, 99]

// Filter orders by status:
let orders = [
  { id: "PO-001", status: "approved", amount: 5000 },
  { id: "PO-002", status: "pending", amount: 15000 },
  { id: "PO-003", status: "approved", amount: 8000 },
  { id: "PO-004", status: "rejected", amount: 3000 }
];

let approved = orders.filter(o => o.status === "approved");
console.log(approved);
// [{ id: "PO-001", ... }, { id: "PO-003", ... }]
```

##### `.reduce()` — Combine All Items Into One Value

Reduces an entire array down to a single value:

```javascript
let amounts = [100, 200, 300, 400, 500];

// Sum all amounts:
let total = amounts.reduce((accumulator, current) => accumulator + current, 0);
console.log(total);  // 1500

// How it works step by step:
// Start: accumulator = 0
// Step 1: 0 + 100 = 100
// Step 2: 100 + 200 = 300
// Step 3: 300 + 300 = 600
// Step 4: 600 + 400 = 1000
// Step 5: 1000 + 500 = 1500 ← final result!
```

```javascript
// Find the maximum value:
let scores = [72, 88, 95, 60, 81];
let highest = scores.reduce((max, score) => score > max ? score : max, 0);
console.log(highest);  // 95
```

---

#### Other Useful Methods

| Method | What It Does | Example |
|--------|-------------|---------|
| `.sort()` | Sort array | `[3,1,2].sort()` → `[1,2,3]` |
| `.reverse()` | Reverse array | `[1,2,3].reverse()` → `[3,2,1]` |
| `.join(sep)` | Join into string | `["a","b","c"].join("-")` → `"a-b-c"` |
| `.slice(start, end)` | Extract portion | `[1,2,3,4,5].slice(1,4)` → `[2,3,4]` |
| `.concat(arr2)` | Merge arrays | `[1,2].concat([3,4])` → `[1,2,3,4]` |
| `.flat()` | Flatten nested arrays | `[[1,2],[3,4]].flat()` → `[1,2,3,4]` |
| `.every(fn)` | ALL items pass test? | `[2,4,6].every(n => n%2===0)` → `true` |
| `.some(fn)` | ANY item passes test? | `[1,2,3].some(n => n>2)` → `true` |

```javascript
// Chaining methods together:
let products = [
  { name: "Laptop", price: 60000, inStock: true },
  { name: "Mouse", price: 500, inStock: false },
  { name: "Keyboard", price: 2000, inStock: true },
  { name: "Monitor", price: 25000, inStock: true }
];

// Get names of in-stock products over ₹1000, sorted:
let result = products
  .filter(p => p.inStock && p.price > 1000)
  .map(p => p.name)
  .sort();

console.log(result);  // ["Keyboard", "Laptop", "Monitor"]
```

---

## Session 4: Objects, Destructuring, Spread & Rest (13:45 - 15:00)

### Objects — Deep Dive

Objects store data as **key-value pairs**. They represent real-world entities:

```javascript
let purchaseOrder = {
  poNumber: "PO-2026-001",
  vendor: "TechSupplies Ltd",
  amount: 75000,
  currency: "INR",
  items: 5,
  isUrgent: true,
  status: "Submitted",
  createdDate: "2026-05-23"
};
```

---

### Accessing Object Properties

```javascript
// Dot notation (preferred):
console.log(purchaseOrder.vendor);      // "TechSupplies Ltd"
console.log(purchaseOrder.amount);      // 75000

// Bracket notation (for dynamic keys or special characters):
console.log(purchaseOrder["status"]);   // "Submitted"

let key = "currency";
console.log(purchaseOrder[key]);        // "INR"
```

---

### Modifying Objects

```javascript
// Add new property:
purchaseOrder.approver = "Priya Sharma";

// Change existing property:
purchaseOrder.status = "Approved";

// Delete a property:
delete purchaseOrder.isUrgent;

// Check if property exists:
console.log("vendor" in purchaseOrder);       // true
console.log("discount" in purchaseOrder);     // false
```

---

### Object Methods

```javascript
let employee = {
  firstName: "Arun",
  lastName: "Kumar",
  salary: 50000,
  
  // Method — a function inside an object:
  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  
  getAnnualSalary() {
    return this.salary * 12;
  }
};

console.log(employee.getFullName());       // "Arun Kumar"
console.log(employee.getAnnualSalary());   // 600000
```

---

### Object Utility Methods

```javascript
let car = { brand: "Tata", model: "Nexon", year: 2024, fuel: "Electric" };

// Get all keys:
console.log(Object.keys(car));     // ["brand", "model", "year", "fuel"]

// Get all values:
console.log(Object.values(car));   // ["Tata", "Nexon", 2024, "Electric"]

// Get key-value pairs:
console.log(Object.entries(car));
// [["brand","Tata"], ["model","Nexon"], ["year",2024], ["fuel","Electric"]]

// Loop through an object:
for (let [key, value] of Object.entries(car)) {
  console.log(`${key}: ${value}`);
}
```

---

### Destructuring — Unpack Values Easily

#### Object Destructuring

Extract properties into variables in one line:

```javascript
let user = {
  name: "Rahul",
  age: 25,
  city: "Mumbai",
  role: "Developer"
};

// OLD way (tedious):
let name = user.name;
let age = user.age;
let city = user.city;

// NEW way (destructuring):
let { name, age, city, role } = user;
console.log(name);  // "Rahul"
console.log(city);  // "Mumbai"

// Rename while destructuring:
let { name: fullName, role: jobRole } = user;
console.log(fullName);   // "Rahul"
console.log(jobRole);    // "Developer"

// Default values:
let { name, country = "India" } = user;
console.log(country);  // "India" (default, since user.country is undefined)
```

#### Array Destructuring

```javascript
let scores = [95, 88, 72, 60];

// Extract first two:
let [first, second] = scores;
console.log(first);   // 95
console.log(second);  // 88

// Skip items with commas:
let [, , third] = scores;
console.log(third);   // 72

// Swap two variables:
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b);  // 2, 1 — Swapped!
```

---

### Spread Operator (`...`) — Expand

The spread operator **expands** an array or object into individual elements:

#### Spread with Arrays

```javascript
// Copy an array:
let original = [1, 2, 3];
let copy = [...original];
console.log(copy);  // [1, 2, 3] (independent copy!)

// Merge arrays:
let arr1 = [1, 2, 3];
let arr2 = [4, 5, 6];
let merged = [...arr1, ...arr2];
console.log(merged);  // [1, 2, 3, 4, 5, 6]

// Add items while spreading:
let withExtras = [0, ...arr1, 4, 5];
console.log(withExtras);  // [0, 1, 2, 3, 4, 5]
```

#### Spread with Objects

```javascript
// Copy an object:
let person = { name: "Arun", age: 25 };
let personCopy = { ...person };

// Merge objects:
let defaults = { color: "blue", size: "medium", country: "India" };
let userPrefs = { color: "red", size: "large" };
let settings = { ...defaults, ...userPrefs };
console.log(settings);
// { color: "red", size: "large", country: "India" }
// userPrefs overrides defaults!

// Add properties while spreading:
let employee = { ...person, department: "Engineering", salary: 50000 };
console.log(employee);
// { name: "Arun", age: 25, department: "Engineering", salary: 50000 }
```

---

### Rest Operator (`...`) — Collect

The rest operator **collects** remaining items into an array or object:

#### Rest with Arrays

```javascript
let numbers = [1, 2, 3, 4, 5, 6, 7];

let [first, second, ...rest] = numbers;
console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5, 6, 7] — "the rest"
```

#### Rest with Objects

```javascript
let product = {
  id: "PROD-001",
  name: "Laptop",
  price: 60000,
  weight: 1.5,
  color: "Silver"
};

let { id, name, ...details } = product;
console.log(id);       // "PROD-001"
console.log(name);     // "Laptop"
console.log(details);  // { price: 60000, weight: 1.5, color: "Silver" }
```

#### Rest in Function Parameters

```javascript
// Accept any number of arguments:
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3));       // 6
console.log(sum(10, 20, 30, 40)); // 100
```

---

### Spread vs Rest — How to Tell Them Apart

```
SPREAD (...) = EXPANDS values OUT (used when CALLING/CREATING)
REST (...)   = COLLECTS values IN (used when RECEIVING/DESTRUCTURING)

Examples:
  let merged = [...arr1, ...arr2];     // SPREAD — expanding arrays
  let [first, ...rest] = arr;          // REST — collecting remaining

  let newObj = { ...oldObj, key: val }; // SPREAD — expanding object
  let { id, ...others } = obj;         // REST — collecting remaining
```

---

## Session 5: JSON — Parse, Stringify & Real-World Usage (15:15 - 16:00)

### What is JSON?

**JSON** = JavaScript Object Notation

It's a text format for storing and exchanging data. Used EVERYWHERE:
- APIs send/receive data as JSON
- Configuration files (package.json!)
- Databases store data as JSON
- Communication between frontend and backend

```json
{
  "name": "Priya Sharma",
  "age": 28,
  "isActive": true,
  "department": "Engineering",
  "skills": ["JavaScript", "Node.js", "SAP CAP"],
  "address": {
    "city": "Bangalore",
    "state": "Karnataka"
  }
}
```

---

### JSON Rules (Different from JavaScript Objects!)

| Rule | JSON | JavaScript Object |
|------|------|-------------------|
| Keys | MUST be in double quotes `"key"` | Can be without quotes |
| Strings | MUST use double quotes `"value"` | Can use single or backtick |
| Trailing comma | ❌ NOT allowed | ✅ Allowed |
| Comments | ❌ NOT allowed | ✅ Allowed |
| Functions | ❌ NOT allowed | ✅ Allowed |
| undefined | ❌ NOT allowed | ✅ Allowed |

```javascript
// Valid JSON:
'{"name": "Arun", "age": 25}'

// INVALID JSON:
'{name: "Arun", age: 25}'            // Keys not in quotes
"{'name': 'Arun'}"                   // Single quotes
'{"name": "Arun", "age": 25,}'      // Trailing comma
```

---

### JSON.stringify() — Object → JSON String

Converts a JavaScript object into a JSON string:

```javascript
let employee = {
  name: "Rahul",
  age: 25,
  skills: ["JS", "Node.js"],
  isActive: true
};

let jsonString = JSON.stringify(employee);
console.log(jsonString);
// '{"name":"Rahul","age":25,"skills":["JS","Node.js"],"isActive":true}'
console.log(typeof jsonString);  // "string"

// Pretty-printed (indented):
let prettyJSON = JSON.stringify(employee, null, 2);
console.log(prettyJSON);
/*
{
  "name": "Rahul",
  "age": 25,
  "skills": [
    "JS",
    "Node.js"
  ],
  "isActive": true
}
*/
```

**When to use:** Sending data to an API, saving to a file, storing in localStorage.

---

### JSON.parse() — JSON String → Object

Converts a JSON string back into a JavaScript object:

```javascript
let jsonData = '{"name": "Priya", "age": 28, "city": "Mumbai"}';

let person = JSON.parse(jsonData);
console.log(person.name);   // "Priya"
console.log(person.age);    // 28
console.log(typeof person); // "object"
```

**When to use:** Receiving data from an API, reading a config file.

---

### Real-World: API Response Handling

```javascript
// Simulating an API response (this is what you'd get from a server):
let apiResponse = `{
  "status": "success",
  "data": {
    "orders": [
      {"id": "PO-001", "amount": 15000, "status": "approved"},
      {"id": "PO-002", "amount": 8000, "status": "pending"},
      {"id": "PO-003", "amount": 52000, "status": "approved"}
    ],
    "totalCount": 3
  }
}`;

// Parse the response:
let response = JSON.parse(apiResponse);

// Access the data:
console.log(response.status);                    // "success"
console.log(response.data.totalCount);           // 3
console.log(response.data.orders[0].id);         // "PO-001"

// Process the orders:
let approvedOrders = response.data.orders.filter(o => o.status === "approved");
let totalApproved = approvedOrders.reduce((sum, o) => sum + o.amount, 0);
console.log(`Approved orders total: ₹${totalApproved}`);  // ₹67000
```

---

### Deep Copy with JSON (Quick Trick)

```javascript
let original = { name: "Arun", scores: [80, 90, 95] };

// Shallow copy problem:
let shallowCopy = { ...original };
shallowCopy.scores.push(100);
console.log(original.scores);  // [80, 90, 95, 100] — ORIGINAL was modified! 😱

// Deep copy with JSON:
let deepCopy = JSON.parse(JSON.stringify(original));
deepCopy.scores.push(100);
console.log(original.scores);  // [80, 90, 95] — Original is SAFE! ✅
```

---

## Session 6: Hands-on — Grade Calculator & Exercises (16:00 - 16:45)

### Build: Student Grade Calculator

Create `gradeCalculator.js`:

```javascript
// ============================================
//  STUDENT GRADE CALCULATOR
// ============================================

// Student data:
let students = [
  { name: "Arun Kumar", scores: { math: 85, science: 92, english: 78, hindi: 88, computer: 95 } },
  { name: "Priya Sharma", scores: { math: 92, science: 88, english: 95, hindi: 90, computer: 97 } },
  { name: "Rahul Verma", scores: { math: 65, science: 70, english: 55, hindi: 72, computer: 80 } },
  { name: "Sneha Patel", scores: { math: 45, science: 52, english: 48, hindi: 60, computer: 55 } },
  { name: "Vikram Singh", scores: { math: 78, science: 82, english: 74, hindi: 85, computer: 90 } }
];

// TASK 1: Calculate average for each student
function calculateAverage(scores) {
  let values = Object.values(scores);
  let total = values.reduce((sum, score) => sum + score, 0);
  return total / values.length;
}

// TASK 2: Determine grade based on average
function getGrade(average) {
  if (average >= 90) return "A+";
  if (average >= 80) return "A";
  if (average >= 70) return "B";
  if (average >= 60) return "C";
  if (average >= 50) return "D";
  return "F";
}

// TASK 3: Determine pass/fail (must score >= 40 in EVERY subject)
function isPassed(scores) {
  return Object.values(scores).every(score => score >= 40);
}

// TASK 4: Process all students
console.log("═".repeat(70));
console.log("                    STUDENT REPORT CARD");
console.log("═".repeat(70));
console.log(`${"Name".padEnd(18)} | ${"Avg".padStart(5)} | Grade | Status | Highest Subject`);
console.log("─".repeat(70));

let results = students.map(student => {
  let avg = calculateAverage(student.scores);
  let grade = getGrade(avg);
  let passed = isPassed(student.scores);
  
  // Find highest scoring subject:
  let subjects = Object.entries(student.scores);
  let [highestSubject] = subjects.reduce((max, current) => 
    current[1] > max[1] ? current : max
  );
  
  return { ...student, average: avg, grade, passed, highestSubject };
});

// Display results:
results.forEach(r => {
  let status = r.passed ? "PASS ✅" : "FAIL ❌";
  console.log(
    `${r.name.padEnd(18)} | ${r.average.toFixed(1).padStart(5)} | ${r.grade.padEnd(5)} | ${status} | ${r.highestSubject}`
  );
});

console.log("─".repeat(70));

// TASK 5: Class statistics
let averages = results.map(r => r.average);
let classAverage = averages.reduce((sum, a) => sum + a, 0) / averages.length;
let topStudent = results.reduce((top, r) => r.average > top.average ? r : top);
let passCount = results.filter(r => r.passed).length;
let failCount = results.length - passCount;

console.log(`\nClass Average: ${classAverage.toFixed(1)}`);
console.log(`Top Student: ${topStudent.name} (${topStudent.average.toFixed(1)})`);
console.log(`Passed: ${passCount} | Failed: ${failCount}`);
console.log(`Pass Rate: ${((passCount / results.length) * 100).toFixed(0)}%`);
console.log("═".repeat(70));
```

---

### Array Exercises (15 Problems — Quick Reference)

Create `arrayExercises.js`:

```javascript
// ===== ARRAY EXERCISES (15 Problems) =====
let numbers = [23, 45, 12, 67, 34, 89, 5, 43, 78, 56];

// 1. Find the sum of all numbers

// 2. Find the largest number

// 3. Find the smallest number

// 4. Filter only even numbers

// 5. Double every number (multiply by 2)

// 6. Sort in ascending order

// 7. Sort in descending order

// 8. Find the average

// 9. Check if ANY number is greater than 100

// 10. Check if ALL numbers are positive

// 11. Remove duplicates from [1,2,2,3,3,3,4,5,5]

// 12. Flatten [[1,2],[3,4],[5,6]] into [1,2,3,4,5,6]

// 13. Count how many numbers are > 50

// 14. Find the second largest number

// 15. Reverse the array without modifying the original
```

<details>
<summary>Solutions</summary>

```javascript
let numbers = [23, 45, 12, 67, 34, 89, 5, 43, 78, 56];

// 1. Sum
let sum = numbers.reduce((a, b) => a + b, 0); // 452

// 2. Largest
let max = Math.max(...numbers); // 89

// 3. Smallest
let min = Math.min(...numbers); // 5

// 4. Even numbers
let evens = numbers.filter(n => n % 2 === 0); // [12, 34, 78, 56]

// 5. Double
let doubled = numbers.map(n => n * 2); // [46, 90, 24, ...]

// 6. Sort ascending
let asc = [...numbers].sort((a, b) => a - b); // [5, 12, 23, ...]

// 7. Sort descending
let desc = [...numbers].sort((a, b) => b - a); // [89, 78, 67, ...]

// 8. Average
let avg = sum / numbers.length; // 45.2

// 9. Any > 100?
let anyOver100 = numbers.some(n => n > 100); // false

// 10. All positive?
let allPositive = numbers.every(n => n > 0); // true

// 11. Remove duplicates
let dupes = [1,2,2,3,3,3,4,5,5];
let unique = [...new Set(dupes)]; // [1, 2, 3, 4, 5]

// 12. Flatten
let nested = [[1,2],[3,4],[5,6]];
let flat = nested.flat(); // [1, 2, 3, 4, 5, 6]

// 13. Count > 50
let over50 = numbers.filter(n => n > 50).length; // 4

// 14. Second largest
let sorted = [...numbers].sort((a, b) => b - a);
let secondLargest = sorted[1]; // 78

// 15. Reverse without modifying original
let reversed = [...numbers].reverse();
```

</details>

---

## Assessment: MCQ (15 Questions)

**Q1.** What does `if (x > 5 && x < 10)` mean?
- a) x is greater than 5 OR less than 10
- b) x is greater than 5 AND less than 10 (both must be true)
- c) x equals 5 and equals 10
- d) x is between 5 and 10 inclusive

<details><summary>Answer</summary>b) Both conditions must be true — x must be greater than 5 AND simultaneously less than 10</details>

---

**Q2.** What does `for...of` loop iterate over?
- a) Object keys
- b) Array values (or any iterable's values)
- c) Array indexes
- d) Object methods

<details><summary>Answer</summary>b) `for...of` gives you the VALUES of an iterable (array, string, etc.)</details>

---

**Q3.** `[1,2,3].map(x => x * 2)` returns:
- a) `6`
- b) `[1, 2, 3]`
- c) `[2, 4, 6]`
- d) `undefined`

<details><summary>Answer</summary>c) `[2, 4, 6]` — map creates a new array by transforming each element</details>

---

**Q4.** `[5, 12, 8, 30, 3].filter(n => n > 10)` returns:
- a) `[5, 8, 3]`
- b) `[12, 30]`
- c) `2`
- d) `true`

<details><summary>Answer</summary>b) `[12, 30]` — filter keeps only elements that pass the test (> 10)</details>

---

**Q5.** `[1, 2, 3, 4].reduce((sum, n) => sum + n, 0)` returns:
- a) `[1, 2, 3, 4]`
- b) `4`
- c) `10`
- d) `0`

<details><summary>Answer</summary>c) `10` — reduce accumulates: 0+1=1, 1+2=3, 3+3=6, 6+4=10</details>

---

**Q6.** What is destructuring?
- a) Deleting an object
- b) Extracting values from objects/arrays into individual variables in one line
- c) Converting JSON to a string
- d) Breaking code into functions

<details><summary>Answer</summary>b) Destructuring unpacks values: `let {name, age} = person;` extracts name and age into separate variables</details>

---

**Q7.** What does the spread operator (`...`) do with arrays?
- a) Deletes the array
- b) Expands an array into individual elements
- c) Sorts the array
- d) Filters the array

<details><summary>Answer</summary>b) Spread expands — `[...arr1, ...arr2]` merges both arrays by expanding their elements</details>

---

**Q8.** `JSON.parse('{"name":"Arun"}')` returns:
- a) A string
- b) A JavaScript object `{ name: "Arun" }`
- c) An error
- d) `undefined`

<details><summary>Answer</summary>b) JSON.parse converts a JSON string into a JavaScript object you can work with</details>

---

**Q9.** Which is NOT valid JSON?
- a) `{"name": "Arun"}`
- b) `{'name': 'Arun'}`
- c) `{"age": 25}`
- d) `{"items": [1, 2, 3]}`

<details><summary>Answer</summary>b) Single quotes are NOT valid in JSON — keys and strings must use double quotes</details>

---

**Q10.** `let [a, , b] = [1, 2, 3]` — what are a and b?
- a) a=1, b=2
- b) a=1, b=3
- c) a=2, b=3
- d) Error

<details><summary>Answer</summary>b) a=1, b=3 — the empty comma skips the second element (2)</details>

---

**Q11.** What does `break` do in a loop?
- a) Skips the current iteration
- b) Exits the loop immediately
- c) Restarts the loop
- d) Pauses the loop for 1 second

<details><summary>Answer</summary>b) `break` exits the entire loop immediately — no more iterations run</details>

---

**Q12.** `Object.keys({a: 1, b: 2, c: 3})` returns:
- a) `[1, 2, 3]`
- b) `["a", "b", "c"]`
- c) `{a: 1, b: 2, c: 3}`
- d) `3`

<details><summary>Answer</summary>b) `["a", "b", "c"]` — Object.keys returns an array of the object's property names</details>

---

**Q13.** The ternary operator `condition ? A : B` is equivalent to:
- a) `if (condition) { A } else { B }`
- b) `switch (condition) { case A: B }`
- c) `while (condition) { A; B; }`
- d) `for (condition) { A; B; }`

<details><summary>Answer</summary>a) It's a short-form if/else — returns A if condition is true, B if false</details>

---

**Q14.** `let {name, ...rest} = {name: "A", age: 25, city: "B"}` — what is `rest`?
- a) `"A"`
- b) `{age: 25, city: "B"}`
- c) `undefined`
- d) `["age", "city"]`

<details><summary>Answer</summary>b) `{age: 25, city: "B"}` — rest operator collects all remaining properties into a new object</details>

---

**Q15.** Which loop type should you use to iterate over an object's properties?
- a) `for...of`
- b) `for...in`
- c) `while`
- d) `do...while`

<details><summary>Answer</summary>b) `for...in` loops through object keys. (Remember: "in" = objects, "of" = arrays/iterables)</details>

---

## Assignment: Build an Inventory Management System

### Due: Start of Day 8

Create `inventory.js`:

```javascript
// ============================================
//  INVENTORY MANAGEMENT SYSTEM
// ============================================
// Build a simple inventory system using arrays and objects.
// Implement ALL the functions below.

// Starting inventory:
let inventory = [
  { id: "PRD-001", name: "Laptop", category: "Electronics", price: 65000, stock: 25, minStock: 10 },
  { id: "PRD-002", name: "Mouse", category: "Electronics", price: 500, stock: 150, minStock: 50 },
  { id: "PRD-003", name: "Desk Chair", category: "Furniture", price: 12000, stock: 8, minStock: 5 },
  { id: "PRD-004", name: "Notebook", category: "Stationery", price: 50, stock: 500, minStock: 100 },
  { id: "PRD-005", name: "Monitor", category: "Electronics", price: 25000, stock: 3, minStock: 10 },
  { id: "PRD-006", name: "Headphones", category: "Electronics", price: 3000, stock: 45, minStock: 20 },
  { id: "PRD-007", name: "Standing Desk", category: "Furniture", price: 35000, stock: 12, minStock: 5 },
  { id: "PRD-008", name: "Pen Pack", category: "Stationery", price: 120, stock: 30, minStock: 50 }
];

// ===== TASK 1: Display All Products =====
// Print all products in a formatted table
// Show: ID, Name, Category, Price (₹), Stock, Status
// Status: "Low Stock ⚠️" if stock < minStock, else "OK ✅"


// ===== TASK 2: Search by Name =====
// Write a function that takes a search term and returns matching products
// (case-insensitive, partial match)
// Example: searchByName("desk") → returns Desk Chair and Standing Desk


// ===== TASK 3: Filter by Category =====
// Write a function that returns all products in a given category
// Example: filterByCategory("Electronics") → returns 4 products


// ===== TASK 4: Get Low Stock Items =====
// Return all products where stock < minStock
// These need to be reordered!


// ===== TASK 5: Calculate Total Inventory Value =====
// Total value = sum of (price × stock) for all products
// Print the total value


// ===== TASK 6: Most Expensive Product =====
// Find and print the most expensive product


// ===== TASK 7: Add New Product =====
// Write a function to add a new product to inventory
// It should generate the next ID automatically (PRD-009, PRD-010, etc.)


// ===== TASK 8: Update Stock =====
// Write a function: updateStock(productId, quantityChange)
// Positive = stock received, Negative = stock sold
// Example: updateStock("PRD-001", -5) → reduces Laptop stock by 5
// Should NOT allow stock to go below 0


// ===== TASK 9: Generate Reorder Report (JSON) =====
// Create a JSON report of all low-stock items with:
// { reportDate, totalItems, items: [{id, name, currentStock, minStock, reorderQuantity}] }
// reorderQuantity = minStock * 2 - currentStock (order enough for double the minimum)
// Print the pretty JSON


// ===== TASK 10: Category Summary =====
// For each category, show:
// - Number of products
// - Total stock
// - Total value
// - Average price
// Format as a table


// Run all tasks:
console.log("\n===== TASK 1: All Products =====");
// Call your function

console.log("\n===== TASK 2: Search 'desk' =====");
// Call your function

console.log("\n===== TASK 3: Electronics =====");
// Call your function

console.log("\n===== TASK 4: Low Stock Alert =====");
// Call your function

console.log("\n===== TASK 5: Total Value =====");
// Call your function

console.log("\n===== TASK 6: Most Expensive =====");
// Call your function

console.log("\n===== TASK 7: Add Product =====");
// Call your function

console.log("\n===== TASK 8: Update Stock =====");
// Call your function

console.log("\n===== TASK 9: Reorder Report =====");
// Call your function

console.log("\n===== TASK 10: Category Summary =====");
// Call your function
```

### Grading Rubric

| Task | Points | What to Check |
|------|--------|---------------|
| Task 1: Display all | 1 | Formatted table, status shown correctly |
| Task 2: Search | 1 | Case-insensitive, partial match works |
| Task 3: Filter category | 1 | Correct filtering |
| Task 4: Low stock | 1 | Correctly identifies stock < minStock |
| Task 5: Total value | 1 | Math is correct (price × stock summed) |
| Task 6: Most expensive | 1 | Correctly uses reduce or sort |
| Task 7: Add product | 1 | ID auto-generated, product added |
| Task 8: Update stock | 1 | Handles positive/negative, no negative stock |
| Task 9: JSON report | 1 | Valid JSON, correct reorder quantities |
| Task 10: Category summary | 1 | Grouped correctly, all calculations right |
| **Total** | **10** | |

---

## Key Takeaways

| # | Topic | One-Line Summary |
|---|---|---|
| 1 | if/else | Make decisions — run different code based on conditions |
| 2 | switch | Clean way to handle multiple exact-value comparisons |
| 3 | Ternary | One-line if/else: `condition ? valueA : valueB` |
| 4 | for loop | Repeat code a known number of times |
| 5 | while loop | Repeat while a condition is true (unknown iterations) |
| 6 | for...of | Loop through array/string VALUES |
| 7 | for...in | Loop through object KEYS |
| 8 | Arrays | Ordered list — use map, filter, reduce for powerful transformations |
| 9 | Objects | Key-value pairs — represent real entities with properties |
| 10 | Destructuring | `let {a, b} = obj` — extract values in one line |
| 11 | Spread `...` | Expand arrays/objects into individual elements |
| 12 | Rest `...` | Collect remaining elements into an array/object |
| 13 | JSON.parse() | Convert JSON string → JavaScript object |
| 14 | JSON.stringify() | Convert JavaScript object → JSON string |
| 15 | Method chaining | `.filter().map().sort()` — combine operations elegantly |

---

## Preparation for Day 8

Tomorrow: **JavaScript Functions & Modern JS (ES6+)**

You'll learn:
- Function declarations, expressions, and arrow functions
- Default parameters, rest parameters
- Callbacks and higher-order functions
- Closures and scope chain
- ES6+ modules (import/export)
- Classes and prototypes
- Error handling (try/catch)

**To prepare:**
- Complete today's assignment (inventory system)
- Make sure you're comfortable with: arrays, objects, map, filter, reduce
- These concepts are FOUNDATIONAL — everything in CAP builds on them!

---

*End of Day 7*
