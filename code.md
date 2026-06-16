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
