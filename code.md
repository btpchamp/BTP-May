### Adding Criticality (Status Colors)

Criticality makes values appear with color coding:

```cds
// In your entity, add a criticality field:
entity Products : cuid, managed {
  productName : String(100);
  stock       : Integer;
  // Virtual/computed field for criticality
}

// In annotations:
annotate CatalogService.Products with {
  stock @UI.Criticality: stockCriticality;
};
```

Or inline in LineItem:
```cds
LineItem: [
  { Value: stock, Criticality: stockCriticality }
]
```

**Criticality values (Integer):**
| Value | Color | Meaning |
|-------|-------|---------|
| 0 | Grey | Neutral |
| 1 | Red | Negative / Error |
| 2 | Orange/Yellow | Critical / Warning |
| 3 | Green | Positive / Success |

You can compute this in an `after READ` handler:
```javascript
this.after('READ', 'Products', (results) => {
  for (const p of results) {
    if (p.stock === 0) p.stockCriticality = 1;        // Red
    else if (p.stock < 10) p.stockCriticality = 2;    // Orange
    else p.stockCriticality = 3;                       // Green
  }
});
```
