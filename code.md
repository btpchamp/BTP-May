### $filter: String Functions

OData provides special functions for text searching:

| Function | What It Does | Example |
|----------|-------------|---------|
| `contains(field,'text')` | Field contains text anywhere | `contains(title,'Harry')` |
| `startswith(field,'text')` | Field starts with text | `startswith(name,'J')` |
| `endswith(field,'text')` | Field ends with text | `endswith(email,'.com')` |
| `tolower(field)` | Lowercase the field for comparison | `tolower(name) eq 'orwell'` |
| `toupper(field)` | Uppercase the field for comparison | `toupper(code) eq 'USD'` |
| `length(field)` | Get string length | `length(name) gt 10` |
| `trim(field)` | Remove leading/trailing spaces | `trim(name) eq 'Alice'` |
| `concat(a,b)` | Join two strings | `contains(concat(firstName,lastName),'John')` |
| `substring(field,start,length)` | Extract part of string | `substring(isbn,0,3) eq '978'` |
| `indexof(field,'text')` | Position of text (-1 if not found) | `indexof(email,'@') gt 0` |



```javascript

  console.log("Hello")
