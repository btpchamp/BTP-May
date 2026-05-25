# Day 6: Group Assignment — Answer Key (For Trainer Only)

---

## Chamber 1: The Variable Vault — ANSWERS

### Puzzle 1A: Find the Bugs (6 bugs)

```javascript
// Bug 1: Cannot reassign a const
const joiningYear = 2026;
// joiningYear = 2025;  ← ERROR! Fix: change const to let
let joiningYear = 2026;  // ✅ Fixed

// Bug 2: Variable names cannot start with a number
// let 1stProject = "BTP Migration";  ← ERROR!
let firstProject = "BTP Migration";  // ✅ Fixed

// Bug 3: let is block-scoped — can't access outside the block
if (true) {
  let secretCode = "ALPHA";
}
// console.log(secretCode);  ← ERROR! secretCode doesn't exist here
// Fix: declare secretCode outside the block
let secretCode;
if (true) {
  secretCode = "ALPHA";
}
console.log(secretCode);  // ✅ Fixed

// Bug 4: Not technically a bug, but should use template literals
let badge = `Employee: ${employeeName}, ID: ${employeeID}, Dept: ${department}`;  // ✅ Fixed

// Bug 5: typeof null returns "object", not "null"
let salary = null;
// if (typeof salary === "null")  ← WRONG! typeof null === "object"
if (salary === null) {  // ✅ Fixed — direct comparison
  console.log("Salary not set");
}

// Bug 6: Method name is wrong — it's toUpperCase(), not uppercase()
// console.log(department.uppercase());  ← ERROR!
console.log(department.toUpperCase());  // ✅ Fixed: "CLOUD ENGINEERING"
```

### Puzzle 1B: Variable Detective — ANSWERS

```javascript
let a = 5;
let b = a;      // b gets a COPY of a's value (5)
a = 10;         // changing a doesn't affect b
console.log("Line 1:", a);     // 10 (a was reassigned)
console.log("Line 2:", b);     // 5  (b still holds the original copy)

const obj = { x: 1 };
obj.x = 99;    // const prevents REASSIGNMENT, not mutation of properties!
console.log("Line 3:", obj.x); // 99 (object properties CAN be changed with const)

let c;          // declared but no value assigned
console.log("Line 4:", c);     // undefined

var d = 1;
var d = 2;      // var allows redeclaration (dangerous!)
var d = 3;
console.log("Line 5:", d);     // 3 (last assignment wins)
```

**Secret code fragment: Types are number, number, number, undefined, number → N, N, N, U, N**

---

## Chamber 2: The Type Labyrinth — ANSWERS

### Puzzle 2A: Type Prediction Challenge

| # | Expression | Value | typeof |
|---|-----------|-------|--------|
| 1 | `"5" + 3` | `"53"` | `"string"` |
| 2 | `"5" - 3` | `2` | `"number"` |
| 3 | `5 + null` | `5` | `"number"` (null becomes 0) |
| 4 | `"5" * "2"` | `10` | `"number"` |
| 5 | `true + true` | `2` | `"number"` (true=1, so 1+1=2) |
| 6 | `"hello" - 1` | `NaN` | `"number"` ("hello" can't become a number) |
| 7 | `false + "1"` | `"false1"` | `"string"` (+ with string = concatenation) |
| 8 | `undefined + 1` | `NaN` | `"number"` (undefined in math = NaN) |
| 9 | `"" + 0 + 1` | `"01"` | `"string"` (""+0="0", "0"+1="01") |
| 10 | `1 + 2 + "3"` | `"33"` | `"string"` (1+2=3, then 3+"3"="33") |

### Puzzle 2B: The Coercion Trap — ANSWERS

```javascript
// Original results:
console.log(addNumbers(5, 3));        // 8  ✅ (both numbers)
console.log(addNumbers("5", 3));      // "53" ❌ (string + number = concatenation)
console.log(addNumbers("10", "20"));  // "1020" ❌ (string + string = concatenation)
console.log(addNumbers(true, 1));     // 2 🤔 (true becomes 1, so 1+1=2)
console.log(addNumbers(null, 5));     // 5 🤔 (null becomes 0, so 0+5=5)

// FIXED version:
function addNumbers(a, b) {
  return Number(a) + Number(b);
  // Alternative: return +a + +b;
  // Alternative: return parseFloat(a) + parseFloat(b);
}

// Now all work correctly:
// addNumbers("5", 3)       → 8
// addNumbers("10", "20")   → 30
// addNumbers(true, 1)      → 2
// addNumbers(null, 5)      → 5
```

**Secret code fragment: Numeric values from 2A: "53" isn't numeric, 2, 5, 10, 2, NaN(skip), "false1"(skip), NaN(skip), "01"(skip), "33"(skip) → Only pure numbers: 2+5+10+2 = 19. 19 mod 26 = 19 → Letter "S"**

---

## Chamber 3: The Operator Arena — ANSWERS

### Puzzle 3A: Operator Output Challenge

```javascript
console.log(1 === 1);            // true  (same value and type)
console.log(1 === "1");          // false (number ≠ string)
console.log(0 == false);         // true  (loose: false becomes 0)
console.log(0 === false);        // false (strict: number ≠ boolean)
console.log("" == false);        // true  (loose: both become 0)
console.log("" === false);       // false (strict: string ≠ boolean)
console.log(null == undefined);  // true  (special rule: null == undefined is true)
console.log(null === undefined); // false (strict: different types)
console.log(NaN === NaN);        // false (NaN is NEVER equal to anything, even itself!)
console.log(2 > 1 > 0);         // true  (2>1=true, true>0=1>0=true)
console.log(2 > 1 > 1);         // false (2>1=true, true>1=1>1=false)
```

**Answers: true, false, true, false, true, false, true, false, false, true, false**

### Puzzle 3B: Build the Condition — ANSWERS

```javascript
let age = 25;
let role = "Manager";
let orderAmount = 75000;
let isUrgent = true;
let stockLevel = 3;
let customerRating = 4.5;

// 1. Can approve order?
let canApprove = role === "Manager" && orderAmount < 100000;
// true (role is Manager AND 75000 < 100000)

// 2. Needs VP approval?
let needsVP = orderAmount >= 100000 || isUrgent === true;
// true (isUrgent is true, so the OR is satisfied)

// 3. Eligible for discount?
let getDiscount = customerRating > 4.0 && orderAmount > 50000 && !isUrgent;
// false (customerRating>4 ✅ AND orderAmount>50000 ✅ AND !isUrgent ❌ → false)

// 4. Low stock alert?
let lowStock = stockLevel < 5 && stockLevel > 0;
// true (3 < 5 AND 3 > 0)

// 5. Can vote AND can drink?
let canVoteAndDrink = age >= 21;
// true (if age >= 21, they automatically satisfy >= 18 too)
// Simplified! No need for: age >= 18 && age >= 21

// 6. Free shipping?
let freeShipping = orderAmount > 1000 || customerRating === 5 || isUrgent;
// true (orderAmount > 1000 is true, so entire OR is true)
```

### Puzzle 3C: The Mysterious Expression — ANSWER

```javascript
let result = (5 > 3) && (2 + 2 === 4) || !(10 % 2 === 0) && (true || false);

// Step by step:
// (5 > 3) = true
// (2 + 2 === 4) = (4 === 4) = true
// (10 % 2 === 0) = (0 === 0) = true
// !(true) = false
// (true || false) = true

// Now substitute:
// true && true || false && true

// Precedence: && before ||
// (true && true) || (false && true)
// true || false
// = true

// FINAL ANSWER: true
```

**Secret code fragment: Count of TRUE answers in 3A = true, false, true, false, true, false, true, false, false, true, false → 5 TRUEs**

---

## Chamber 4: The String Cipher — ANSWERS

### Puzzle 4A: Decode the Message

```javascript
let data = [
  "xxHxx",
  "Hello World Every Living Organism",
  "  PYTHON java JAVASCRIPT ruby  ",
  "error-404-not-found",
  "aaa@bbb.ccc"
];

// STEP 1: Character at index 2
let char1 = data[0].charAt(2);  // "H"
// or: data[0][2]

// STEP 2: First letter of each word
let char2 = data[1].split(" ").map(word => word[0]).join("");
// split → ["Hello", "World", "Every", "Living", "Organism"]
// map first letters → ["H", "W", "E", "L", "O"]
// join → "HWELO"

// STEP 3: Trim, split, find longest
let words = data[2].trim().split(" ").filter(w => w.length > 0);
let char3 = words.reduce((longest, current) => 
  current.length > longest.length ? current : longest, "");
// char3 = "JAVASCRIPT" (10 characters, longest word)

// STEP 4: Replace hyphens with spaces, capitalize each word
let char4 = data[3]
  .replace(/-/g, " ")
  .split(" ")
  .map(word => word.charAt(0).toUpperCase() + word.slice(1))
  .join(" ");
// char4 = "Error 404 Not Found"

// STEP 5: Extract between @ and .
let emailStr = data[4];
let atIndex = emailStr.indexOf("@");
let dotIndex = emailStr.indexOf(".", atIndex);
let char5 = emailStr.slice(atIndex + 1, dotIndex);
// char5 = "bbb"
```

### Puzzle 4B: String Transformer — ANSWERS

```javascript
// 1. "hello-world-program" → "helloWorldProgram" (camelCase)
let input1 = "hello-world-program";
let camel = input1.split("-").map((word, i) => 
  i === 0 ? word : word.charAt(0).toUpperCase() + word.slice(1)
).join("");
console.log(camel);  // "helloWorldProgram"

// 2. "camelCaseText" → "camel-case-text" (kebab-case)
let input2 = "camelCaseText";
let kebab = input2.replace(/([A-Z])/g, "-$1").toLowerCase();
console.log(kebab);  // "camel-case-text"

// 3. Acronym from "SAP Business Technology Platform"
let input3 = "SAP Business Technology Platform";
let acronym = input3.split(" ").map(word => word[0].toUpperCase()).join("");
console.log(acronym);  // "SBTP"

// 4. Censor bad words
let input4 = "What the hell, this damn code won't work!";
let censored = input4.replace(/hell/g, "****").replace(/damn/g, "****");
console.log(censored);  // "What the ****, this **** code won't work!"

// 5. Count vowels
let input5 = "JavaScript is awesome";
let vowelCount = input5.toLowerCase().split("").filter(c => "aeiou".includes(c)).length;
console.log(vowelCount);  // 8
// a, a, i = 3 in "JavaScript", i = 1 in "is", a, e,o,e = 4 in "awesome" → total 8
```

### Puzzle 4C: Password Validator — ANSWER

```javascript
let password = "MyS@p2026!";

let hasLength = password.length >= 8;
let hasUpper = password !== password.toLowerCase();
let hasLower = password !== password.toUpperCase();
let hasNumber = /\d/.test(password);  // or: password.split("").some(c => c >= "0" && c <= "9")
let hasSpecial = /[!@#$%^&*]/.test(password);
let noSpaces = !password.includes(" ");
let noLeadingNum = !(password[0] >= "0" && password[0] <= "9");

let score = [hasLength, hasUpper, hasLower, hasNumber, hasSpecial, noSpaces, noLeadingNum]
  .filter(Boolean).length;

let strength = score === 7 ? "STRONG" : score >= 5 ? "MEDIUM" : "WEAK";

console.log(`Password: "${password}"`);
console.log(`Length >= 8:     ${hasLength ? "✅ PASS" : "❌ FAIL"}`);
console.log(`Has uppercase:   ${hasUpper ? "✅ PASS" : "❌ FAIL"}`);
console.log(`Has lowercase:   ${hasLower ? "✅ PASS" : "❌ FAIL"}`);
console.log(`Has number:      ${hasNumber ? "✅ PASS" : "❌ FAIL"}`);
console.log(`Has special:     ${hasSpecial ? "✅ PASS" : "❌ FAIL"}`);
console.log(`No spaces:       ${noSpaces ? "✅ PASS" : "❌ FAIL"}`);
console.log(`No leading num:  ${noLeadingNum ? "✅ PASS" : "❌ FAIL"}`);
console.log(`Strength: ${strength} (${score}/7)`);

// Output:
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

**Secret code fragment from Chamber 4: Length of "HWELO" = 5**

---

## Chamber 5: The Logic Engine — ANSWERS

### Puzzle 5A: Truth Table — ANSWERS

| A | B | `A && B` | `A \|\| B` | `!A` | `A && !B` | `!A \|\| B` |
|---|---|---------|-----------|------|----------|------------|
| true | true | **true** | **true** | **false** | **false** | **true** |
| true | false | **false** | **true** | **false** | **true** | **false** |
| false | true | **false** | **true** | **true** | **false** | **true** |
| false | false | **false** | **false** | **true** | **false** | **true** |

### Puzzle 5B: Movie Ticket Pricing — ANSWER

```javascript
let basePrice = 200;
let day = "Saturday";
let is3D = true;
let isIMAX = false;
let age = 8;

// Step 1: Start with base price
let price = 200;

// Step 2: Weekend surcharge? Saturday = yes! +₹100
let isWeekend = (day === "Saturday" || day === "Sunday");  // true
price = price + 100;  // price = 300

// Step 3: 3D surcharge? Yes! +₹150
price = price + 150;  // price = 450

// Step 4: IMAX surcharge? No (isIMAX = false)
// price stays 450

// Step 5: Combo discount? (3D AND IMAX) — No (only 3D, not IMAX)
// price stays 450

// Step 6: Subtotal
let subtotal = 450;

// Step 7: Age discount — age 8 < 12 → child → 50% off
let discount = subtotal * 0.50;  // 225
price = subtotal - discount;     // price = 225

// Step 8: FINAL PRICE = ₹225
console.log(`Final ticket price: ₹${price}`);  // ₹225
```

### Puzzle 5C: Salary Bug Fix — ANSWERS

```javascript
// BUG 1: basicSalary is a STRING "50000", not a number!
// This causes concatenation instead of addition in grossSalary calculation
let basicSalary = 50000;  // ✅ Fixed: removed quotes

let hra = 40;
let da = 20;
let taxRate = 30;

// After fixing bug 1, all calculations work correctly:
let hraAmount = basicSalary * hra / 100;      // 50000 * 40 / 100 = 20000
let daAmount = basicSalary * da / 100;        // 50000 * 20 / 100 = 10000

// BUG 2 (caused by Bug 1): With string basicSalary:
// "50000" + 20000 + 10000 = "500002000010000" (concatenation!)
// Fix: basicSalary is now a number, so:
let grossSalary = basicSalary + hraAmount + daAmount;  // 50000 + 20000 + 10000 = 80000 ✅

let taxAmount = grossSalary * taxRate / 100;  // 80000 * 30 / 100 = 24000
let netSalary = grossSalary - taxAmount;      // 80000 - 24000 = 56000

// SUMMARY OF BUGS:
// Bug 1: basicSalary was a string "50000" instead of number 50000
// Bug 2: (consequence) grossSalary became string concatenation "500002000010000"
// Bug 3: (consequence) taxAmount calculated on the wrong grossSalary
// Bug 4: (consequence) netSalary was completely wrong
//
// ROOT CAUSE: All 4 bugs stem from Bug 1 — the string type of basicSalary!
// This is why type awareness is SO important in JavaScript!

// Correct output:
// Basic: ₹50000
// HRA (40%): ₹20000
// DA (20%): ₹10000
// Gross: ₹80000
// Tax (30%): ₹24000
// Net Salary: ₹56000
```

**Secret code fragment from Chamber 5: Ticket price 225 / 100 = 2.25 (round to 2)**

---

## The Final Vault — ANSWER

```javascript
let orders = [
  { id: "ORD-001", amount: "12500", status: "pending", customer: "Acme Corp", priority: true },
  { id: "ORD-002", amount: "8000", status: "approved", customer: "Beta Inc", priority: false },
  { id: "ORD-003", amount: "150000", status: "pending", customer: "Gamma Ltd", priority: true },
  { id: "ORD-004", amount: "3500", status: "rejected", customer: "Delta Co", priority: false },
  { id: "ORD-005", amount: "95000", status: "pending", customer: "Epsilon SA", priority: true }
];

// Process each order:
console.log("=".repeat(80));
console.log("ORDER PROCESSING REPORT");
console.log("=".repeat(80));

for (let order of orders) {
  // 1. Convert amount from string to number
  let amount = Number(order.amount);
  
  // 2. Determine category
  let category;
  if (amount < 10000) {
    category = "Small";
  } else if (amount < 100000) {
    category = "Medium";
  } else {
    category = "Large";
  }
  
  // 3. Approval logic
  let approval;
  if (order.status === "approved") {
    approval = "Already approved";
  } else if (order.status === "rejected") {
    approval = "Rejected";
  } else if (order.status === "pending" && amount > 50000) {
    approval = "Needs VP approval";
  } else {
    approval = "Auto-approve";
  }
  
  // 4. Priority display
  let priorityDisplay = order.priority ? "🔴 URGENT" : "🟢 Normal";
  
  // 5. Format and print
  let formattedAmount = `₹${amount.toLocaleString()}`;
  let paddedCustomer = order.customer.padEnd(12);
  let paddedCategory = category.padEnd(8);
  let paddedApproval = approval.padEnd(18);
  
  console.log(`${order.id} | ${paddedCustomer} | ${formattedAmount.padStart(10)} | ${paddedCategory} | ${paddedApproval} | ${priorityDisplay}`);
}

console.log("=".repeat(80));

// EXPECTED OUTPUT:
// ================================================================================
// ORDER PROCESSING REPORT
// ================================================================================
// ORD-001 | Acme Corp    |    ₹12,500 | Medium   | Auto-approve       | 🔴 URGENT
// ORD-002 | Beta Inc     |     ₹8,000 | Small    | Already approved   | 🟢 Normal
// ORD-003 | Gamma Ltd    |  ₹1,50,000 | Large    | Needs VP approval  | 🔴 URGENT
// ORD-004 | Delta Co     |     ₹3,500 | Small    | Rejected           | 🟢 Normal
// ORD-005 | Epsilon SA   |    ₹95,000 | Medium   | Needs VP approval  | 🔴 URGENT
// ================================================================================
```

---

## Secret Code Summary

| Chamber | Fragment | Value |
|---------|----------|-------|
| 1 | First letters of typeof results | N, N, N, U, N |
| 2 | Sum of numeric values mod 26 | 19 → "S" |
| 3 | Count of TRUE answers | 5 |
| 4 | Length of "HWELO" | 5 |
| 5 | Ticket price / 100 | 2.25 ≈ 2 |

**Combined code: "NNNU-N-S-5-5-2"** (or however you want to format it as a team activity)

---

*End of Answer Key*
