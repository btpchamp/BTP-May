### $filter: Date & Time Functions

| Function | What It Does | Example |
|----------|-------------|---------|
| `year(field)` | Extract year | `year(orderDate) eq 2026` |
| `month(field)` | Extract month (1-12) | `month(hireDate) eq 6` |
| `day(field)` | Extract day (1-31) | `day(birthDate) eq 15` |
| `hour(field)` | Extract hour (0-23) | `hour(createdAt) ge 9` |
| `now()` | Current date/time | `orderDate lt now()` |


```
// Orders placed in 2026
$filter=year(orderDate) eq 2026

// Employees hired in June
$filter=month(hireDate) eq 6

// Books published after 2000
$filter=year(publishDate) gt 2000

// Orders from this month
$filter=year(orderDate) eq 2026 and month(orderDate) eq 6
```
