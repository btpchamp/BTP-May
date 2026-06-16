## What You'll Achieve Today

By the end of this review day, you will:
- Debug and fix common annotation errors confidently
- Apply best practices for annotation organization and naming
- Add custom sections and fragments to extend Fiori Elements
- Understand the Flexible Programming Model for advanced customization
- Complete a fully working "Manage Products" Fiori app from scratch
- Pass a 30-question quiz covering the entire UI layer (Days 21-25)

---

## Week 5 Wrap-up & Quick Recap (09:00 - 09:15)

### The UI Layer Journey

```
Day 21: Introduction to Fiori & UI5
   └─► 5 Fiori principles, Fiori Elements vs Freestyle, Floorplans

Day 22: List Report & Object Page
   └─► @UI.LineItem, SelectionFields, HeaderInfo, FieldGroup, Facets

Day 23: Advanced Annotations & Navigation
   └─► ValueList, Criticality, DataPoints, multi-level drill-down, actions

Day 24: Draft & CRUD Operations
   └─► with draft, auto-save, FieldControl, SideEffects, dynamic expressions

Day 25: Review & Practice  ← TODAY
   └─► Debugging, best practices, mini project
```

### Quick Fire: Can You Name These?

1. The 4 main Fiori Elements floorplans → _____
2. Annotation for table columns → _____
3. Annotation for filter bar fields → _____
4. Keyword to enable draft → _____
5. Criticality colors: 1=_____, 2=_____, 3=_____
6. Show name instead of UUID → _____
7. Show/hide actions conditionally → _____
8. Auto-refresh fields when others change → _____

<details>
<summary>Answers</summary>

1. List Report, Object Page, Worklist, Overview Page
2. `@UI.LineItem`
3. `@UI.SelectionFields`
4. `with draft`
5. 1=Red, 2=Orange, 3=Green
6. `@Common.Text` + `@Common.TextArrangement: #TextOnly`
7. `@Core.OperationAvailable`
8. `@Common.SideEffects`

</details>

---

## Session 1: Common Annotation Issues & Debugging (09:15 - 10:30)

### The Top 10 Annotation Mistakes (and How to Fix Them)

---

#### Mistake 1: Table Shows All Fields (No Columns Configured)

**Symptom:** List Report shows every single field in the entity (20+ columns, ugly layout).

**Cause:** Missing `@UI.LineItem` annotation.

**Fix:**
```cds
// Add this to your annotations file:
annotate MyService.Products with @UI.LineItem: [
  { Value: productName, Label: 'Product' },
  { Value: price, Label: 'Price' },
  { Value: stock, Label: 'Stock' }
];
```

**Rule:** If you don't specify `@UI.LineItem`, Fiori Elements shows ALL fields. Always define it.

---

#### Mistake 2: Object Page is Completely Blank

**Symptom:** Click a row in the list → Object Page opens but shows nothing (empty page).

**Cause:** Missing `@UI.Facets` annotation (no sections defined).

**Fix:**
```cds
annotate MyService.Products with @UI: {
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Main', Label: 'Details' }
  ],
  FieldGroup#Main: {
    Data: [
      { Value: productName },
      { Value: price },
      { Value: stock }
    ]
  }
};
```

**Rule:** Object Page renders NOTHING without `@UI.Facets`. It needs to know what sections to show.

---

#### Mistake 3: "Unknown Entity" or "Cannot find entity" Error

**Symptom:** App fails to load with metadata errors.

**Cause:** The `annotate` statement references a wrong entity name.

```cds
// WRONG — entity name doesn't match the SERVICE entity name:
annotate db.Products with @UI.LineItem: [...];

// CORRECT — must use the service-level entity name:
annotate CatalogService.Products with @UI.LineItem: [...];
```

**Rule:** Always annotate the SERVICE entity (e.g., `CatalogService.Products`), not the DB entity (e.g., `com.epm.Products`).

---

#### Mistake 4: Value Help Shows UUIDs Instead of Names

**Symptom:** Dropdown shows `a1b2c3d4-e5f6-...` instead of "Electronics".

**Cause:** Missing `@Common.Text` annotation.

**Fix:**
```cds
annotate CatalogService.Products with {
  category @Common.Text: category.categoryName  @Common.TextArrangement: #TextOnly;
};
```

---

#### Mistake 5: Filter Bar is Empty (No Filters)

**Symptom:** No filter fields above the table.

**Cause:** Missing `@UI.SelectionFields`.

**Fix:**
```cds
annotate CatalogService.Products with @UI.SelectionFields: [
  productName, price, category_ID, supplier_ID
];
```

---

#### Mistake 6: Child Table Doesn't Appear on Object Page

**Symptom:** Items/Reviews section is missing from the Object Page.

**Cause:** Wrong navigation path in Facets, or missing LineItem on the child entity.

**Checklist:**
```cds
// 1. Child entity MUST have @UI.LineItem:
annotate CatalogService.OrderItems with @UI.LineItem: [
  { Value: product.productName },
  { Value: quantity },
  { Value: unitPrice }
];

// 2. Parent's Facets must use CORRECT navigation property name:
annotate CatalogService.Orders with @UI.Facets: [
  { $Type: 'UI.ReferenceFacet', Target: 'items/@UI.LineItem', Label: 'Items' }
  //                                     ↑ Must match the association name in CDS!
];
```

**Common mistake:** Using `'orderItems/@UI.LineItem'` when the association is named `'items'`.

---

#### Mistake 7: Action Button Doesn't Appear

**Symptom:** You defined an action but no button shows on the Object Page.

**Cause:** Missing `@UI.Identification` with `DataFieldForAction`.

**Fix:**
```cds
annotate CatalogService.Orders with @UI.Identification: [
  { $Type: 'UI.DataFieldForAction', Action: 'CatalogService.confirm', Label: 'Confirm' }
];
```

**Also check:** Is the action defined in your CDS service? Is the entity name correct in the Action path?

---

#### Mistake 8: Criticality Colors Don't Show

**Symptom:** Fields show plain text without colored indicators.

**Cause:** The criticality field is null/undefined (handler not computing it).

**Checklist:**
1. `Criticality: fieldName` references an actual field
2. The handler computes the value (1, 2, or 3)
3. The field is included in the query (use `$select` carefully)

```javascript
// Make sure the after READ handler runs:
this.after('READ', 'Products', (results) => {
  const items = Array.isArray(results) ? results : [results];
  for (const item of items) {
    // MUST set an integer value!
    item.stockCriticality = item.stock > 10 ? 3 : item.stock > 0 ? 2 : 1;
  }
});
```

---

#### Mistake 9: Draft Not Working (No Edit Button)

**Symptom:** Object Page shows data but no "Edit" button. Can't modify records.

**Cause:** `with draft` not added to the service entity.

**Fix:**
```cds
entity Products as projection on db.Products with draft;
```

**Also check:** Is the manifest.json configured for ObjectPage (not just ListReport)?

---

#### Mistake 10: Navigation from List to Object Page Not Working

**Symptom:** Clicking a table row does nothing (no navigation).

**Cause:** Missing route in `manifest.json` for the Object Page.

**Fix:** Ensure you have BOTH routes:
```json
"routes": [
  { "pattern": ":?query:", "name": "List", "target": "List" },
  { "pattern": "Products({key}):?query:", "name": "Detail", "target": "Detail" }
]
```

AND both targets:
```json
"targets": {
  "List": { "name": "sap.fe.templates.ListReport", ... },
  "Detail": { "name": "sap.fe.templates.ObjectPage", ... }
}
```

---

### Debugging Annotations: Quick Steps

```
Problem: Something isn't rendering correctly

Step 1: Check the browser console (F12 → Console tab)
  → Look for OData or metadata errors

Step 2: Check $metadata
  → Open /service/$metadata in browser
  → Search for your entity and annotation terms
  → If annotations aren't there → CDS file not loaded

Step 3: Check CDS compilation
  → Run: cds compile srv/ --to edmx
  → Look for your annotations in the output

Step 4: Check the annotation file imports
  → Does it have: using { ServiceName } from './service-file';
  → Is it referenced/discovered by CAP?

Step 5: Compare with a working example
  → Use SAP's sample Fiori Elements projects as reference
```

---

## Session 2: Best Practices, Extensibility & Custom Sections (10:45 - 12:00)

### Best Practices for UI Annotations

#### Practice 1: Organize Annotations in Separate Files

```
project/
├── srv/
│   ├── catalog-service.cds       ← Service definition only
│   └── catalog-service.js        ← Handlers only
├── app/
│   ├── products/
│   │   ├── webapp/               ← Fiori app files
│   │   ├── annotations.cds      ← ✅ UI annotations for Products app
│   │   └── fiori-service.cds    ← Service-specific overrides
│   └── orders/
│       ├── webapp/
│       └── annotations.cds      ← ✅ UI annotations for Orders app
```

**Why?** Each Fiori app can have different annotations for the same service entity. A "Browse Products" app shows fewer columns than a "Manage Products" admin app.

---

#### Practice 2: Use Consistent Label Patterns

```cds
// ❌ BAD — inconsistent naming
LineItem: [
  { Value: productName, Label: 'product' },        // lowercase
  { Value: price, Label: 'PRICE ($)' },            // all caps with symbol
  { Value: stock, Label: 'Qty in Warehouse' },     // verbose
  { Value: rating }                                 // no label at all
]

// ✅ GOOD — consistent, clean
LineItem: [
  { Value: productName, Label: 'Product Name' },   // Title Case
  { Value: price, Label: 'Price' },                // Short, clear
  { Value: stock, Label: 'Stock' },                // One word when possible
  { Value: rating, Label: 'Rating' }               // Always provide a label
]
```

**Or better — use `@title` on fields so labels are centralized:**
```cds
annotate CatalogService.Products with {
  productName @title: 'Product Name';
  price       @title: 'Price';
  stock       @title: 'Stock';
  rating      @title: 'Rating';
};
// Now LineItem doesn't need individual labels — it inherits from @title!
```

---

#### Practice 3: Keep SelectionFields Minimal (3-7 Fields)

```cds
// ❌ BAD — too many filters (overwhelming)
SelectionFields: [ name, price, stock, rating, sku, description,
                   category_ID, supplier_ID, isActive, createdAt,
                   modifiedAt, weight, color, size ]

// ✅ GOOD — focused on what users actually search by
SelectionFields: [ name, category_ID, supplier_ID, isActive ]
```

---

#### Practice 4: Use FieldGroups with Clear Names

```cds
// ❌ BAD — generic names
FieldGroup#Group1: { ... }
FieldGroup#Group2: { ... }

// ✅ GOOD — descriptive names
FieldGroup#GeneralInfo: { ... }
FieldGroup#PricingStock: { ... }
FieldGroup#SupplierDetails: { ... }
FieldGroup#AdminData: { ... }
```

---

#### Practice 5: Order Facets by User Priority

Put the most important information first:

```cds
Facets: [
  // Most important — users see this first
  { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#General', Label: 'General' },
  // Second most important
  { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Pricing', Label: 'Pricing' },
  // Related data (table)
  { $Type: 'UI.ReferenceFacet', Target: 'items/@UI.LineItem', Label: 'Items' },
  // Least important — admin data last
  { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Admin', Label: 'Administration' }
]
```

---
