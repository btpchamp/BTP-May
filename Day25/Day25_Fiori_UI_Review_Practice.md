# Day 25: Fiori UI Review & Practice Day

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Week 5 Wrap-up & Quick Recap | 15 min |
| 09:15 - 10:30 | Session 1: Common Annotation Issues & Debugging | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Best Practices, Extensibility & Custom Sections | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Fix Issues & Polish PO App | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 15:00 | Session 4: Weekly Quiz — Days 21-25 (30 Questions) | 75 min |
| 15:00 - 15:15 | Break | 15 min |
| 15:15 - 17:00 | Session 5 & 6: Mini Project — Manage Products Fiori App | 105 min |

---

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

### Fiori Elements Extensibility — Custom Sections

Sometimes annotations aren't enough. You need custom UI that annotations can't express. Fiori Elements supports **extensions** for this:

#### Adding a Custom Section (XML Fragment)

You can inject a custom XML fragment into the Object Page:

**Step 1: Create a fragment file**

`app/products/webapp/ext/CustomSection.fragment.xml`:
```xml
<core:FragmentDefinition xmlns:core="sap.ui.core" xmlns="sap.m">
  <VBox class="sapUiSmallMargin">
    <Title text="Stock Visualization" level="H3"/>
    <ProgressIndicator
      percentValue="{= ${stock} / ${maxStock} * 100}"
      displayValue="{= ${stock} + ' / ' + ${maxStock}}"
      state="{= ${stock} > 20 ? 'Success' : ${stock} > 0 ? 'Warning' : 'Error'}"
    />
    <Text text="Reorder needed: {= ${stock} &lt; ${minStock} ? 'Yes' : 'No'}" class="sapUiTinyMarginTop"/>
  </VBox>
</core:FragmentDefinition>
```

**Step 2: Register it in manifest.json**

```json
"ProductsObjectPage": {
  "options": {
    "settings": {
      "entitySet": "Products",
      "content": {
        "body": {
          "sections": {
            "customStockSection": {
              "template": "products.ext.CustomSection",
              "title": "Stock Visualization",
              "position": { "placement": "After", "anchor": "GeneralInfoSection" }
            }
          }
        }
      }
    }
  }
}
```

**Result:** A custom section appears on the Object Page with a progress bar showing stock level — something annotations alone can't do!

---

### The Flexible Programming Model

SAP introduced the **Flexible Programming Model** for Fiori Elements V4. It allows mixing annotations with custom code:

```
┌─────────────────────────────────────────────────────────────┐
│  FLEXIBILITY SPECTRUM                                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Pure Fiori        Fiori Elements          Freestyle        │
│  Elements          + Extensions            UI5              │
│  (annotations      (custom sections,       (100% manual    │
│   only)            custom columns,          XML + JS)       │
│                    custom actions)                           │
│                                                             │
│  ◄──────────────────────────────────────────────────────►  │
│  Less Control                             More Control      │
│  Less Code                                More Code         │
│  More Consistency                         Less Consistency  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**What you can customize with extensions:**
- Custom columns in tables
- Custom sections on Object Page
- Custom header content
- Custom filter fields
- Custom actions with complex dialogs
- Controller extensions (JavaScript hooks)

**When to use extensions:**
- When annotations can't express what you need
- When you need a chart, custom visualization, or complex interaction
- When the business requires something non-standard

---

### UI Annotation Summary Card

```
┌─────────────────────────────────────────────────────────────┐
│             ANNOTATION CHEAT SHEET                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  LIST REPORT:                                               │
│    @UI.SelectionFields     → Filter bar fields              │
│    @UI.LineItem            → Table columns                  │
│    @UI.LineItem + DataFieldForAction → Inline buttons       │
│                                                             │
│  OBJECT PAGE:                                               │
│    @UI.HeaderInfo          → Title + subtitle               │
│    @UI.HeaderFacets        → KPI boxes in header            │
│    @UI.DataPoint           → KPI values                     │
│    @UI.Facets              → Page sections                  │
│    @UI.FieldGroup          → Form fields in a section       │
│    @UI.Identification      → Action buttons in toolbar      │
│    nav/@UI.LineItem        → Child entity table section     │
│                                                             │
│  FIELD-LEVEL:                                               │
│    @title                  → Default label everywhere       │
│    @Common.Text            → Show name instead of ID        │
│    @Common.TextArrangement → #TextOnly / #TextFirst         │
│    @Common.ValueList       → Dropdown/dialog picker         │
│    @Common.FieldControl    → ReadOnly/Optional/Mandatory    │
│    @UI.Hidden              → Don't show this field          │
│    @UI.MultiLineText       → Textarea for long text         │
│    @Measures.ISOCurrency   → Show currency with price       │
│    Criticality             → Color coding (1/2/3)           │
│                                                             │
│  ACTIONS & BEHAVIOR:                                        │
│    @Core.OperationAvailable → Show/hide action buttons      │
│    @Common.IsActionCritical → Confirmation dialog           │
│    @Common.SideEffects      → Auto-refresh on field change  │
│    with draft               → Enable draft editing          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Session 3: Hands-on — Fix Issues & Polish PO App (12:00 - 13:00)

### Exercise 1: Debug These Annotation Problems (20 minutes)

Below are broken annotations. Find and fix each issue:

**Problem A:**
```cds
// The table shows all 15 fields instead of just 4
annotate PurchasingService.PurchaseOrders with @UI: {
  LineItems: [
    { Value: poNumber },
    { Value: supplier.supplierName },
    { Value: totalAmount },
    { Value: status }
  ]
};
```

<details>
<summary>Fix</summary>

**Issue:** `LineItems` should be `LineItem` (no plural 's')!

```cds
LineItem: [  // ← Correct: "LineItem" not "LineItems"
  { Value: poNumber },
  ...
]
```
</details>

---

**Problem B:**
```cds
// Object Page shows "Unknown" for supplier field
annotate PurchasingService.PurchaseOrders with {
  supplier @Common.Text: supplierName;
};
```

<details>
<summary>Fix</summary>

**Issue:** Must use navigation path `supplier.supplierName` (not just `supplierName`).

```cds
supplier @Common.Text: supplier.supplierName;  // ← Need the full path!
```
</details>

---

**Problem C:**
```cds
// Items table doesn't appear on the Object Page
annotate PurchasingService.PurchaseOrders with @UI.Facets: [
  { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#General', Label: 'General' },
  { $Type: 'UI.ReferenceFacet', Target: 'purchaseOrderItems/@UI.LineItem', Label: 'Items' }
];
```

<details>
<summary>Fix</summary>

**Issue:** The navigation property name is probably `items` (as defined in the CDS model), not `purchaseOrderItems`.

```cds
{ $Type: 'UI.ReferenceFacet', Target: 'items/@UI.LineItem', Label: 'Items' }
// ↑ Must match the exact association/composition name in your CDS entity!
```

Check your schema: if it says `items : Composition of many PurchaseOrderItems`, then use `'items/@UI.LineItem'`.
</details>

---

**Problem D:**
```cds
// Action button "Submit" never appears
annotate PurchasingService.PurchaseOrders with @UI.Identification: [
  { $Type: 'UI.DataFieldForAction', Action: 'submit', Label: 'Submit' }
];
```

<details>
<summary>Fix</summary>

**Issue:** Action must include the full service name: `PurchasingService.submit`.

```cds
{ $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.submit', Label: 'Submit' }
// ↑ Must be: ServiceName.actionName
```
</details>

---

### Exercise 2: Polish Your PO App (40 minutes)

Apply these improvements to your Purchase Order Management app:

1. **Add `@title` to all fields** (centralized labels):
```cds
annotate PurchasingService.PurchaseOrders with {
  poNumber     @title: 'PO Number';
  status       @title: 'Status';
  priority     @title: 'Priority';
  totalAmount  @title: 'Total Amount';
  orderDate    @title: 'Order Date';
  expectedDate @title: 'Expected Delivery';
  notes        @title: 'Notes'  @UI.MultiLineText;
};
```

2. **Add FieldControl** (read-only after submission):
```cds
annotate PurchasingService.PurchaseOrders with {
  poNumber @Common.FieldControl: poNumberEditable;
  supplier @Common.FieldControl: supplierEditable;
};
```

3. **Add semantic object status** for colored status badges:
```cds
annotate PurchasingService.PurchaseOrders with @UI.LineItem: [
  { Value: status, Criticality: statusCriticality }
];
```

4. **Ensure Value Help works** for supplier and products

5. **Test the complete workflow:** Create → Save → Submit → Approve

---

## Session 4: Weekly Quiz — Days 21-25 (13:45 - 15:00)

### Instructions
- 30 questions covering the UI layer (Days 21-25)
- Mix of multiple choice, true/false, and short answer
- Time: 75 minutes
- Passing score: 70% (21/30)

---

**Q1.** The 5 SAP Fiori design principles are (in order):
- A) Responsive, Accessible, Consistent, Simple, Delightful
- B) Role-based, Adaptive, Coherent, Simple, Delightful
- C) Reliable, Available, Configurable, Scalable, Durable
- D) Robust, Agile, Clean, Secure, Dynamic

**Answer: B**

---

**Q2.** SAPUI5 is:
- A) A database
- B) A JavaScript UI framework for building Fiori applications
- C) A design tool
- D) A testing framework

**Answer: B**

---

**Q3.** Fiori Elements generates UI automatically from:
- A) HTML templates
- B) OData $metadata + CDS annotations
- C) CSS files
- D) JavaScript modules

**Answer: B**

---

**Q4.** Which floorplan shows a filterable table of records?
- A) Object Page
- B) Overview Page
- C) List Report
- D) Worklist

**Answer: C**

---

**Q5.** `@UI.LineItem` defines:
- A) Filter bar fields
- B) Table columns in the List Report
- C) Object Page sections
- D) Navigation links

**Answer: B**

---

**Q6.** `@UI.SelectionFields` defines:
- A) Which fields appear in the filter bar
- B) Which fields are required
- C) Table columns
- D) Object Page layout

**Answer: A**

---

**Q7.** `@UI.HeaderInfo.Title` appears where?
- A) Browser tab
- B) List Report title
- C) Object Page header — main title of the record
- D) Filter bar

**Answer: C**

---

**Q8.** To show related OrderItems as a table on the Order's Object Page, the Facet Target is:
- A) `'@UI.LineItem#Items'`
- B) `'items/@UI.LineItem'`
- C) `'OrderItems/@UI.Table'`
- D) `'@UI.FieldGroup#Items'`

**Answer: B**

---

**Q9.** `@UI.FieldGroup#Pricing` is used for:
- A) Price calculations
- B) Grouping form fields into a labeled section on the Object Page
- C) Filter configuration
- D) Table sorting

**Answer: B**

---

**Q10.** Criticality value meanings: 0=_____, 1=_____, 2=_____, 3=_____
- A) 0=Hidden, 1=Error, 2=Warning, 3=Info
- B) 0=Grey/Neutral, 1=Red/Negative, 2=Orange/Warning, 3=Green/Positive
- C) 0=Green, 1=Yellow, 2=Red, 3=Blue
- D) 0=None, 1=Low, 2=Medium, 3=High

**Answer: B**

---

**Q11.** `@Common.ValueList` provides:
- A) Form validation
- B) A dropdown or dialog for selecting values from a related entity
- C) Default field values
- D) Sort options

**Answer: B**

---

**Q12.** `@Common.Text: supplier.supplierName` with `TextArrangement: #TextOnly` shows:
- A) The UUID of the supplier
- B) Only the supplier's name (hiding the UUID)
- C) Both UUID and name
- D) An error

**Answer: B**

---

**Q13.** `@UI.DataPoint` with `Visualization: #Rating` shows:
- A) A number
- B) A progress bar
- C) Star rating (⭐⭐⭐⭐☆)
- D) A chart

**Answer: C**

---

**Q14.** Multi-level navigation (List → Object → Sub-Object) is configured in:
- A) annotations.cds only
- B) manifest.json routes + targets
- C) package.json
- D) index.html

**Answer: B**

---

**Q15.** `@UI.DataFieldForAction` placed in `@UI.Identification` adds:
- A) Filter fields
- B) Action buttons in the Object Page header toolbar
- C) New columns in the table
- D) Navigation links

**Answer: B**

---

**Q16.** `with draft` in a service entity enables:
- A) Read-only mode
- B) Auto-save, exclusive locking, and the full draft editing lifecycle
- C) Faster queries
- D) Export to Excel

**Answer: B**

---

**Q17.** Draft data is stored in:
- A) Browser localStorage
- B) A separate `_drafts` table in the database
- C) A cookie
- D) Server memory

**Answer: B**

---

**Q18.** "Activate" in draft means:
- A) Start editing
- B) Move draft to the real table (save/commit)
- C) Lock the record
- D) Delete the draft

**Answer: B**

---

**Q19.** Bound actions (approve, reject) can only be called on:
- A) Draft records
- B) Active (saved) records
- C) Deleted records
- D) Any record regardless of state

**Answer: B**

---

**Q20.** `@Common.IsActionCritical` does what?
- A) Marks the action as requiring admin role
- B) Shows a confirmation dialog before executing
- C) Colors the button red
- D) Logs the action

**Answer: B**

---

**Q21.** `@Common.SideEffects` with `SourceProperties: ['quantity']` and `TargetProperties: ['totalPrice']` means:
- A) quantity depends on totalPrice
- B) When quantity changes, Fiori re-reads totalPrice from the server
- C) totalPrice is always equal to quantity
- D) Both fields become read-only

**Answer: B**

---

**Q22.** `@Common.FieldControl` value `1` means:
- A) Hidden
- B) Read-Only
- C) Editable (Optional)
- D) Mandatory

**Answer: B**

---

**Q23.** `@Core.OperationAvailable: submitEnabled` does what?
- A) Defines if the service is online
- B) Shows/hides the Submit action button based on the submitEnabled field value
- C) Makes the entity available for queries
- D) Enables the submit endpoint

**Answer: B**

---

**Q24.** When does `before('SAVE')` run in draft-enabled entities?
- A) On every auto-save (every field change)
- B) When the user clicks the final "Save" button (activation)
- C) When the record is first created
- D) When the record is deleted

**Answer: B**

---

**Q25.** TRUE or FALSE: Fiori Elements requires you to write HTML and CSS for table layout.
- A) TRUE
- B) FALSE

**Answer: B (FALSE)** — Everything is generated from annotations. No HTML/CSS needed.

---

**Q26.** Annotations should be placed on:
- A) Database entities (e.g., `annotate db.Products`)
- B) Service entities (e.g., `annotate CatalogService.Products`)
- C) Either works identically
- D) package.json

**Answer: B** — Always annotate the service-level entity. The same DB entity may appear differently in different services.

---

**Q27.** A `CollectionFacet` is used to:
- A) Collect data from multiple services
- B) Group multiple sections under one parent tab/section
- C) Filter collections
- D) Paginate large lists

**Answer: B**

---

**Q28.** `@Common.ValueListWithFixedValues` on an enum field renders as:
- A) A text input
- B) A simple dropdown (no search dialog)
- C) A multi-select
- D) A radio button group

**Answer: B**

---

**Q29.** The `@UI.Hidden` annotation:
- A) Encrypts the field
- B) Removes the field from the UI but it still exists in the data/API
- C) Deletes the field from the entity
- D) Makes the field password-masked

**Answer: B**

---

**Q30.** If a Fiori Elements app shows no data in the table, the FIRST thing to check is:
- A) The annotations file
- B) Whether the service actually has data (test via REST: `GET /service/Entity`)
- C) The CSS styles
- D) The HTML template

**Answer: B** — Always verify the data layer works first. If the OData service returns data, the problem is annotations. If not, fix the service/data first.

---

### Quiz Scoring

| Range | Grade | Feedback |
|-------|-------|----------|
| 27-30 | Excellent | You've mastered the Fiori UI layer! |
| 21-26 | Good | Review the specific topics you missed |
| 15-20 | Fair | Re-read Days 22-24, focus on annotation syntax |
| Below 15 | Needs Work | Go through each day again with hands-on practice |

---

## Session 5 & 6: Mini Project — Manage Products Fiori App (15:15 - 17:00)

### Project Brief

Build a complete **"Manage Products"** Fiori Elements application from scratch that includes:
- List Report with filters, sortable columns, and criticality
- Object Page with header KPIs, multiple sections, and child table
- Draft support for Create/Edit
- Action buttons for business operations
- Value Helps for all association fields
- Navigation from Product → Reviews (sub-object)

---

### Step 1: Data Model (Use Existing or Create)

```cds
namespace com.shop;
using { cuid, managed, Currency } from '@sap/cds/common';

type ProductStatus : String(20) enum {
  Draft; Active; Discontinued;
}

entity Categories : cuid, managed {
  categoryName : String(100);
  description  : String(300);
  products     : Association to many Products on products.category = $self;
}

entity Suppliers : cuid, managed {
  supplierName : String(150);
  email        : String(255);
  city         : String(100);
  rating       : Decimal(2,1);
  products     : Association to many Products on products.supplier = $self;
}

entity Products : cuid, managed {
  productName  : String(100);
  description  : String(500);
  price        : Decimal(10,2);
  currency     : Currency;
  stock        : Integer default 0;
  minStock     : Integer default 10;
  rating       : Decimal(2,1) default 0.0;
  status       : ProductStatus default 'Draft';
  imageUrl     : String(500);
  category     : Association to Categories;
  supplier     : Association to Suppliers;
  reviews      : Composition of many Reviews on reviews.product = $self;
}

entity Reviews : cuid, managed {
  product   : Association to Products;
  reviewer  : String(100);
  rating    : Integer;
  comment   : String(500);
  helpful   : Integer default 0;
}
```

---

### Step 2: Service Definition

```cds
using { com.shop as db } from '../db/schema';

service ProductService @(path: '/products') {
  entity Products as projection on db.Products
    with draft
    actions {
      action activate() returns { status: String; message: String; };
      action discontinue(reason: String(500)) returns { status: String; message: String; };
    };
  entity Reviews as projection on db.Reviews;
  @readonly entity Categories as projection on db.Categories;
  @readonly entity Suppliers as projection on db.Suppliers;
}
```

---

### Step 3: Complete Annotations

Create `app/manage-products/annotations.cds`:

```cds
using { ProductService } from '../../srv/product-service';

// ═══════════════════════════════════════════════
// FIELD LABELS (centralized)
// ═══════════════════════════════════════════════
annotate ProductService.Products with {
  productName  @title: 'Product Name';
  description  @title: 'Description'  @UI.MultiLineText;
  price        @title: 'Price'  @Measures.ISOCurrency: currency_code;
  stock        @title: 'Stock';
  minStock     @title: 'Min. Stock';
  rating       @title: 'Rating';
  status       @title: 'Status';
  category     @title: 'Category'
               @Common: { Text: category.categoryName, TextArrangement: #TextOnly,
                 ValueList: { CollectionPath: 'Categories', Parameters: [
                   { $Type: 'Common.ValueListParameterInOut', LocalDataProperty: category_ID, ValueListProperty: 'ID' },
                   { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'categoryName' }
                 ]}};
  supplier     @title: 'Supplier'
               @Common: { Text: supplier.supplierName, TextArrangement: #TextOnly,
                 ValueList: { CollectionPath: 'Suppliers', Parameters: [
                   { $Type: 'Common.ValueListParameterInOut', LocalDataProperty: supplier_ID, ValueListProperty: 'ID' },
                   { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'supplierName' },
                   { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'city' }
                 ]}};
  status       @Common.ValueListWithFixedValues;
};

// ═══════════════════════════════════════════════
// LIST REPORT
// ═══════════════════════════════════════════════
annotate ProductService.Products with @UI: {
  SelectionFields: [ productName, category_ID, supplier_ID, status, price ],

  LineItem: [
    { Value: productName, ![@UI.Importance]: #High },
    { Value: price },
    { Value: stock, Criticality: stockCriticality },
    { Value: rating },
    { Value: category.categoryName, Label: 'Category' },
    { Value: supplier.supplierName, Label: 'Supplier', ![@UI.Importance]: #Low },
    { Value: status, Criticality: statusCriticality }
  ],

  // ═══════════════════════════════════════════════
  // OBJECT PAGE
  // ═══════════════════════════════════════════════
  HeaderInfo: {
    TypeName: 'Product',
    TypeNamePlural: 'Products',
    Title: { Value: productName },
    Description: { Value: description }
  },

  HeaderFacets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Price' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Stock' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Rating' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Status' }
  ],
  DataPoint#Price:  { Value: price, Title: 'Price' },
  DataPoint#Stock:  { Value: stock, Title: 'Stock', Criticality: stockCriticality },
  DataPoint#Rating: { Value: rating, Title: 'Rating', TargetValue: 5, Visualization: #Rating },
  DataPoint#Status: { Value: status, Title: 'Status', Criticality: statusCriticality },

  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#General', Label: 'General Information' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#StockPricing', Label: 'Stock & Pricing' },
    { $Type: 'UI.ReferenceFacet', Target: 'reviews/@UI.LineItem', Label: 'Customer Reviews' }
  ],

  FieldGroup#General: {
    Data: [
      { Value: productName },
      { Value: description },
      { Value: category_ID },
      { Value: supplier_ID },
      { Value: status }
    ]
  },
  FieldGroup#StockPricing: {
    Data: [
      { Value: price },
      { Value: currency_code, Label: 'Currency' },
      { Value: stock },
      { Value: minStock },
      { Value: rating }
    ]
  },

  Identification: [
    { $Type: 'UI.DataFieldForAction', Action: 'ProductService.activate', Label: 'Activate Product' },
    { $Type: 'UI.DataFieldForAction', Action: 'ProductService.discontinue', Label: 'Discontinue' }
  ]
};

// ═══════════════════════════════════════════════
// REVIEWS (Child Table + Sub-Object)
// ═══════════════════════════════════════════════
annotate ProductService.Reviews with @UI: {
  LineItem: [
    { Value: reviewer, Label: 'Reviewer' },
    { Value: rating, Label: 'Rating' },
    { Value: comment, Label: 'Comment' },
    { Value: helpful, Label: 'Helpful Votes' },
    { Value: createdAt, Label: 'Date' }
  ],
  HeaderInfo: {
    TypeName: 'Review',
    TypeNamePlural: 'Reviews',
    Title: { Value: reviewer },
    Description: { Value: comment }
  },
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#ReviewDetail', Label: 'Review Details' }
  ],
  FieldGroup#ReviewDetail: {
    Data: [
      { Value: reviewer },
      { Value: rating },
      { Value: comment },
      { Value: helpful },
      { Value: createdAt }
    ]
  }
};
```

---

### Step 4: Handler for Criticality & Actions

```javascript
// srv/product-service.js
const cds = require('@sap/cds');

module.exports = function () {
  const { Products } = cds.entities;

  // Criticality computation
  this.after('READ', 'Products', (results) => {
    const items = Array.isArray(results) ? results : [results];
    for (const p of items) {
      // Stock criticality
      if (p.stock === 0) p.stockCriticality = 1;
      else if (p.stock < p.minStock) p.stockCriticality = 2;
      else p.stockCriticality = 3;

      // Status criticality
      switch(p.status) {
        case 'Draft': p.statusCriticality = 0; break;
        case 'Active': p.statusCriticality = 3; break;
        case 'Discontinued': p.statusCriticality = 1; break;
        default: p.statusCriticality = 0;
      }

      // Action availability
      p.activateEnabled = p.status === 'Draft';
      p.discontinueEnabled = p.status === 'Active';
    }
  });

  // Activate action
  this.on('activate', 'Products', async (req) => {
    const { ID } = req.params[0];
    const product = await SELECT.one.from(Products).where({ ID });
    if (!product) req.reject(404, 'Product not found');
    if (product.status !== 'Draft') req.reject(400, 'Only Draft products can be activated');
    if (!product.price || product.price <= 0) req.reject(400, 'Price must be set before activation');

    await UPDATE(Products).set({ status: 'Active' }).where({ ID });
    return { status: 'Active', message: `"${product.productName}" is now active and visible to customers!` };
  });

  // Discontinue action
  this.on('discontinue', 'Products', async (req) => {
    const { ID } = req.params[0];
    const { reason } = req.data;
    const product = await SELECT.one.from(Products).where({ ID });
    if (!product) req.reject(404, 'Product not found');
    if (product.status !== 'Active') req.reject(400, 'Only Active products can be discontinued');
    if (!reason) req.reject(400, 'Please provide a reason for discontinuation');

    await UPDATE(Products).set({ status: 'Discontinued' }).where({ ID });
    return { status: 'Discontinued', message: `"${product.productName}" discontinued. Reason: ${reason}` };
  });

  // Validation on save
  this.before('SAVE', 'Products', (req) => {
    const { productName, price } = req.data;
    if (!productName || productName.trim() === '') req.error(400, 'Product name is required', 'productName');
    if (price !== undefined && price < 0) req.error(400, 'Price cannot be negative', 'price');
  });
};
```

---

### Step 5: Verification Checklist

Run `cds watch` and test everything:

- [ ] List Report shows products with colored status and stock
- [ ] Filter bar has: product name, category, supplier, status, price
- [ ] Category and Supplier columns show NAMES (not UUIDs)
- [ ] Click a row → Object Page opens with header info
- [ ] Header shows: Price, Stock (colored), Rating (stars), Status (colored)
- [ ] General section shows product details
- [ ] Stock & Pricing section shows numeric fields
- [ ] Reviews section shows as a table
- [ ] Click [+ Create] → Draft form opens
- [ ] Category/Supplier have Value Help dropdowns
- [ ] Save validates required fields
- [ ] After save: "Activate" button appears (for Draft products)
- [ ] After activate: "Discontinue" button appears (for Active products)
- [ ] Click Discontinue → asks for reason → status changes
- [ ] Close browser mid-edit → reopen → draft still there

---

## End of Day Summary

### What We Covered Today

| Session | Topic | Key Takeaway |
|---------|-------|--------------|
| 1 | Debugging | Top 10 mistakes with fixes + debugging steps |
| 2 | Best Practices | Organization, labels, extensibility, custom sections |
| 3 | Hands-on | Fix real issues, polish the PO app |
| 4 | Weekly Quiz | 30 questions testing Days 21-25 knowledge |
| 5-6 | Mini Project | Complete "Manage Products" app from scratch |

---

### The Complete UI Layer Stack (Week 5 Summary)

```
┌─────────────────────────────────────────────────────────────┐
│           WHAT YOU CAN NOW BUILD                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ List Report with filters, columns, actions, colors      │
│  ✅ Object Page with header KPIs, sections, child tables    │
│  ✅ 3-level navigation (List → Detail → Sub-Detail)        │
│  ✅ Value Helps (dropdowns with search)                     │
│  ✅ Draft editing (auto-save, resume later)                 │
│  ✅ CRUD via UI (Create, Edit, Delete through forms)        │
│  ✅ Action buttons with confirmation dialogs                │
│  ✅ Conditional buttons (show/hide based on state)          │
│  ✅ Side Effects (auto-recalculate totals)                  │
│  ✅ Dynamic field control (read-only when submitted)        │
│  ✅ Criticality colors (red/orange/green indicators)        │
│  ✅ Custom extensions when annotations aren't enough        │
│                                                             │
│  All of this WITHOUT writing HTML, CSS, or UI JavaScript!   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Looking Ahead: Day 26

Starting next week — **Authentication, Authorization & Deployment**:
- Adding security to your application
- User authentication on BTP
- Role-based authorization
- Deploying to SAP BTP Cloud Foundry
- Connecting to SAP HANA Cloud

---

*End of Day 25 — The UI layer is complete! You can now build full-stack SAP applications!*
