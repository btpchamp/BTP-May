### Practice: Create `datatypes.js` (5 minutes)

```javascript
// Explore all data types:
let myString = "SAP CAP Developer";
let myNumber = 45;
let myDecimal = 99.99;
let myBoolean = true;
let myNull = null;
let myUndefined;
let myObject = { name: "CAP", version: 7 };

// Print each with its type:
console.log(myString, "→", typeof myString);
console.log(myNumber, "→", typeof myNumber);
console.log(myDecimal, "→", typeof myDecimal);
console.log(myBoolean, "→", typeof myBoolean);
console.log(myNull, "→", typeof myNull);         // Watch for the bug!
console.log(myUndefined, "→", typeof myUndefined);
console.log(myObject, "→", typeof myObject);

// Fun experiment:
console.log(typeof NaN);           // What do you expect?
console.log(typeof typeof 42);     // typeof always returns a string!
```
