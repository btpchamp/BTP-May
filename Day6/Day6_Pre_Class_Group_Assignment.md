# Day 6: Pre-Class Group Assignment

## "JavaScript Escape Room" — Solve Puzzles to Unlock the Code

### Duration: 1 Hour (Before Class Starts)
### Team Size: 4-5 members per team

---

## The Story

Your team is a group of **junior developers** who just joined "CodeVault Technologies." On your first day, the senior developer left a challenge: she locked the company's main codebase behind **5 locked chambers**. Each chamber has a puzzle that can only be solved using JavaScript fundamentals.

**Solve all 5 chambers to prove you're ready for the team!**

The twist: Each chamber gives you a **secret code fragment**. Combine all 5 to unlock the final vault!

---

## Rules

- You can use VS Code and Node.js to test your answers
- Every team member must contribute (different chambers assigned to different members)
- You earn points for speed AND correctness
- Partial solutions get partial points
- **Time limit: 50 minutes for chambers + 10 minutes for final vault**

---

## Chamber 1: The Variable Vault (10 minutes)

**Assigned to: Member 1**

### Puzzle 1A: Find the Bugs

This code has **6 bugs**. Find and fix ALL of them. Write the corrected version.

```javascript
// This code should print a formatted employee badge
// But it has 6 bugs — find and fix them!

const employeeName = "Arun Krishnan";
let employeeID = "EMP-2026-042";
var department = "Cloud Engineering";
const joiningYear = 2026;

// Bug 1: Something wrong with this reassignment
employeeID = "EMP-2026-043";
joiningYear = 2025;

// Bug 2: Something wrong with this variable name
let 1stProject = "BTP Migration";

// Bug 3: Something wrong with the scope
if (true) {
  let secretCode = "ALPHA";
}
console.log(secretCode);

// Bug 4: Something wrong with this string
let badge = 'Employee: ' + employeeName + ', ID: ' + employeeID + ', Dept: ' + department;
console.log(badge);
// (This works but is ugly — rewrite using template literals)

// Bug 5: What's wrong with typeof here?
let salary = null;
if (typeof salary === "null") {
  console.log("Salary not set");
}

// Bug 6: This should print "CLOUD ENGINEERING" but doesn't
console.log(department.uppercase());
```

### Puzzle 1B: Variable Detective

Without running the code, predict the output of EACH line:

```javascript
let a = 5;
let b = a;
a = 10;
console.log("Line 1:", a);     // ???
console.log("Line 2:", b);     // ???

const obj = { x: 1 };
obj.x = 99;
console.log("Line 3:", obj.x); // ???

let c;
console.log("Line 4:", c);     // ???

var d = 1;
var d = 2;
var d = 3;
console.log("Line 5:", d);     // ???
```

**Your secret code fragment from Chamber 1: The first letter of each answer type (e.g., "number" = N, "undefined" = U)**

---

## Chamber 2: The Type Labyrinth (10 minutes)

**Assigned to: Member 2**

### Puzzle 2A: Type Prediction Challenge

For each expression, write the **typeof** result AND the **value**:

| # | Expression | Value = ? | typeof = ? |
|---|-----------|-----------|-----------|
| 1 | `"5" + 3` | | |
| 2 | `"5" - 3` | | |
| 3 | `5 + null` | | |
| 4 | `"5" * "2"` | | |
| 5 | `true + true` | | |
| 6 | `"hello" - 1` | | |
| 7 | `false + "1"` | | |
| 8 | `undefined + 1` | | |
| 9 | `"" + 0 + 1` | | |
| 10 | `1 + 2 + "3"` | | |

### Puzzle 2B: The Coercion Trap

This function is supposed to add two numbers, but customers keep reporting wrong results. Explain WHY each test fails and fix the code:

```javascript
function addNumbers(a, b) {
  return a + b;
}

// Test cases — some give wrong answers!
console.log(addNumbers(5, 3));        // Expected: 8   Got: 8   ✅
console.log(addNumbers("5", 3));      // Expected: 8   Got: ???  ❌
console.log(addNumbers("10", "20"));  // Expected: 30  Got: ???  ❌
console.log(addNumbers(true, 1));     // Expected: ???  Got: ???  🤔
console.log(addNumbers(null, 5));     // Expected: 5   Got: ???  🤔
```

Write a FIXED version of `addNumbers` that always returns a proper number result.

**Your secret code fragment from Chamber 2: The sum of all numeric answers from 2A (mod 26 = letter of alphabet)**

---

## Chamber 3: The Operator Arena (10 minutes)

**Assigned to: Member 3**

### Puzzle 3A: Operator Output Challenge

Predict the output of each — write TRUE or FALSE:

```javascript
console.log(1 === 1);            // ???
console.log(1 === "1");          // ???
console.log(0 == false);         // ???
console.log(0 === false);        // ???
console.log("" == false);        // ???
console.log("" === false);       // ???
console.log(null == undefined);  // ???
console.log(null === undefined); // ???
console.log(NaN === NaN);        // ???  (TRICKY!)
console.log(2 > 1 > 0);         // ???  (VERY TRICKY!)
console.log(2 > 1 > 1);         // ???  (MIND-BENDING!)
```

### Puzzle 3B: Build the Condition

For each business scenario, write a SINGLE boolean expression using operators:

```javascript
// Given these variables:
let age = 25;
let role = "Manager";
let orderAmount = 75000;
let isUrgent = true;
let stockLevel = 3;
let customerRating = 4.5;

// Write the condition for each scenario:

// 1. Can approve order? (must be Manager AND amount < 100000)
let canApprove = ???;

// 2. Needs VP approval? (amount >= 100000 OR isUrgent is true)
let needsVP = ???;

// 3. Eligible for discount? (rating > 4.0 AND order > 50000 AND NOT urgent)
let getDiscount = ???;

// 4. Low stock alert? (stock < 5 AND stock > 0)
let lowStock = ???;

// 5. Can vote AND can drink? (age >= 18 AND age >= 21)
// Simplify this to the minimum expression!
let canVoteAndDrink = ???;

// 6. Free shipping? (amount > 1000 OR customerRating === 5 OR isUrgent)
let freeShipping = ???;
```

### Puzzle 3C: The Mysterious Expression

What does this evaluate to? Solve step by step (show your work!):

```javascript
let result = (5 > 3) && (2 + 2 === 4) || !(10 % 2 === 0) && (true || false);
```

**Your secret code fragment from Chamber 3: Count the number of TRUE answers in 3A**

---

## Chamber 4: The String Cipher (10 minutes)

**Assigned to: Member 4**

### Puzzle 4A: Decode the Message

A secret message is hidden in this data. Use string methods to extract it:

```javascript
let data = [
  "xxHxx",
  "Hello World Every Living Organism",
  "  PYTHON java JAVASCRIPT ruby  ",
  "error-404-not-found",
  "aaa@bbb.ccc"
];

// STEP 1: From data[0], extract the character at index 2
let char1 = ???;  // Should be: "H"

// STEP 2: From data[1], get the first letter of each word and join them
// Hint: split by space, map first letters, join
let char2 = ???;  // Should be: "HWELO"

// STEP 3: From data[2], trim it, split by space, find the longest word
let char3 = ???;  // Should be: "JAVASCRIPT"

// STEP 4: From data[3], replace all "-" with " ", then capitalize each word
let char4 = ???;  // Should be: "Error 404 Not Found"

// STEP 5: From data[4], extract just the domain name (between @ and .)
let char5 = ???;  // Should be: "bbb"
```

### Puzzle 4B: String Transformer Challenge

Write code for each transformation (one-liners preferred!):

```javascript
// 1. Convert "hello-world-program" to "helloWorldProgram" (camelCase!)
let input1 = "hello-world-program";
// Your code (Hint: split by "-", capitalize first letter of each word except first, join)


// 2. Convert "camelCaseText" to "camel-case-text" (kebab-case!)
let input2 = "camelCaseText";
// Your code (Hint: replace uppercase letters with "-" + lowercase)


// 3. Create an acronym from "SAP Business Technology Platform"
let input3 = "SAP Business Technology Platform";
// Expected: "SBTP"


// 4. Censor bad words: replace "damn" and "hell" with "****"
let input4 = "What the hell, this damn code won't work!";
// Expected: "What the ****, this **** code won't work!"


// 5. Count vowels in "JavaScript is awesome"
let input5 = "JavaScript is awesome";
// Expected: 8
```

### Puzzle 4C: The Password Validator

Build a password strength checker. Given a password string, check ALL these rules and report which pass/fail:

```javascript
let password = "MyS@p2026!";

// Rules:
// 1. At least 8 characters long
// 2. Contains at least one uppercase letter (A-Z)
// 3. Contains at least one lowercase letter (a-z)
// 4. Contains at least one number (0-9)
// 5. Contains at least one special character (!@#$%^&*)
// 6. Does NOT contain spaces
// 7. Does NOT start with a number

// Print result like:
// Password: "MyS@p2026!"
// Length >= 8:     ✅ PASS
// Has uppercase:   ✅ PASS
// Has lowercase:   ✅ PASS
// Has number:      ✅ PASS
// Has special:     ✅ PASS
// No spaces:       ✅ PASS
// No leading num:  ✅ PASS
// Strength: STRONG (7/7)
```

**Your secret code fragment from Chamber 4: The length of the decoded message from 4A step 2**

---

## Chamber 5: The Logic Engine (10 minutes)

**Assigned to: Member 5 (or shared)**

### Puzzle 5A: Truth Table Challenge

Complete the truth table:

| A | B | `A && B` | `A \|\| B` | `!A` | `A && !B` | `!A \|\| B` |
|---|---|---------|-----------|------|----------|------------|
| true | true | ? | ? | ? | ? | ? |
| true | false | ? | ? | ? | ? | ? |
| false | true | ? | ? | ? | ? | ? |
| false | false | ? | ? | ? | ? | ? |

### Puzzle 5B: Real-World Logic

A movie ticket booking system has these rules:

```
PRICING RULES:
- Base price: ₹200
- Weekend (Sat/Sun): +₹100
- 3D movie: +₹150
- IMAX: +₹200
- Children (age < 12): 50% off the TOTAL
- Senior citizens (age >= 60): 30% off the TOTAL
- Combo: If both 3D AND IMAX, extra ₹50 off

Given:
- day = "Saturday"
- is3D = true
- isIMAX = false
- age = 8
```

Calculate the final ticket price. Show EVERY step with the variable values.

```javascript
let basePrice = 200;
let day = "Saturday";
let is3D = true;
let isIMAX = false;
let age = 8;

// YOUR CALCULATION:
// Step 1: Base price = ???
// Step 2: Weekend surcharge? = ???
// Step 3: 3D surcharge? = ???
// Step 4: IMAX surcharge? = ???
// Step 5: Combo discount? = ???
// Step 6: Subtotal = ???
// Step 7: Age discount = ???
// Step 8: FINAL PRICE = ???
```

### Puzzle 5C: The Debugging Challenge

This code calculates an employee's net salary but gives WRONG answers. Find the **4 bugs** and fix them:

```javascript
let basicSalary = "50000";  // Bug might be here...
let hra = 40;               // 40% of basic
let da = 20;                // 20% of basic
let taxRate = 30;           // 30% tax

// Calculate components
let hraAmount = basicSalary * hra / 100;
let daAmount = basicSalary * da / 100;
let grossSalary = basicSalary + hraAmount + daAmount;

// Calculate tax and net
let taxAmount = grossSalary * taxRate / 100;
let netSalary = grossSalary - taxAmount;

console.log(`Basic: ₹${basicSalary}`);
console.log(`HRA (${hra}%): ₹${hraAmount}`);
console.log(`DA (${da}%): ₹${daAmount}`);
console.log(`Gross: ₹${grossSalary}`);
console.log(`Tax (${taxRate}%): ₹${taxAmount}`);
console.log(`Net Salary: ₹${netSalary}`);

// Expected output:
// Basic: ₹50000
// HRA (40%): ₹20000
// DA (20%): ₹10000
// Gross: ₹80000
// Tax (30%): ₹24000
// Net Salary: ₹56000
```

**Hint:** Run the buggy code first and compare actual vs expected output. The primary bug is a type issue!

**Your secret code fragment from Chamber 5: The final ticket price from 5B divided by 100**

---

## The Final Vault (10 minutes)

### Combine Your Codes

Each chamber gave a secret code fragment. Combine them:

```
Chamber 1: [first letter of each typeof result]
Chamber 2: [sum mod 26 → alphabet letter]
Chamber 3: [count of TRUE answers]
Chamber 4: [length of step 2 result]
Chamber 5: [ticket price / 100]
```

### The Final Challenge

Using ALL the concepts from the 5 chambers, solve this:

```javascript
// The Final Vault: SAP Order Processing Logic
// 
// Given this order data:
let orders = [
  { id: "ORD-001", amount: "12500", status: "pending", customer: "Acme Corp", priority: true },
  { id: "ORD-002", amount: "8000", status: "approved", customer: "Beta Inc", priority: false },
  { id: "ORD-003", amount: "150000", status: "pending", customer: "Gamma Ltd", priority: true },
  { id: "ORD-004", amount: "3500", status: "rejected", customer: "Delta Co", priority: false },
  { id: "ORD-005", amount: "95000", status: "pending", customer: "Epsilon SA", priority: true }
];

// YOUR MISSION: For each order, determine and print:
// 1. Convert amount from string to number (watch out for the type!)
// 2. Category: "Small" (< 10000), "Medium" (10000-99999), "Large" (>= 100000)
// 3. Approval needed? (pending + amount > 50000 = needs VP approval)
// 4. Priority flag display: priority=true → "🔴 URGENT" or "🟢 Normal"
//
// Print formatted output:
// ORD-001 | Acme Corp     | ₹12,500  | Medium | Auto-approve    | 🔴 URGENT
// ORD-002 | Beta Inc      | ₹8,000   | Small  | Already approved | 🟢 Normal
// etc.
```

**This tests: type coercion (string→number), comparison operators, logical operators, template literals, string methods — EVERYTHING from today!**

---

## Scoring

| Chamber | Max Points |
|---------|-----------|
| Chamber 1: Variable Vault | 15 |
| Chamber 2: Type Labyrinth | 15 |
| Chamber 3: Operator Arena | 15 |
| Chamber 4: String Cipher | 20 |
| Chamber 5: Logic Engine | 15 |
| Final Vault | 20 |
| **Total** | **100** |

### Bonus Points

| Bonus | Points | For |
|-------|--------|-----|
| First team to complete all chambers | +10 | Speed |
| All members can explain ANY answer when asked | +10 | Team understanding |
| Found an edge case the puzzle designers missed | +5 | Critical thinking |

---

## Presentation (After solving)

Each team has **2 minutes** to:
1. Share their hardest puzzle and how they solved it
2. Show one JavaScript "gotcha" they discovered
3. Ask the class a tricky JS question from what they learned

---

## What You'll Learn

| Concept | Chamber That Tests It |
|---------|----------------------|
| var/let/const & scoping | Chamber 1 |
| typeof & data types | Chamber 2 |
| Type coercion | Chamber 2, 5 |
| Comparison operators | Chamber 3 |
| Logical operators (&&, \|\|, !) | Chamber 3, 5 |
| String methods | Chamber 4 |
| Template literals | Final Vault |
| Debugging type bugs | Chamber 5 |
| Real-world application | Final Vault |

---

*The codebase awaits. Prove yourselves, developers!* 🔐
