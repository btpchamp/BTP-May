## What You'll Learn Today

By the end of this session, you will be able to:
- Configure multi-level navigation (List → Object → Sub-Object Page)
- Set up Value Helps (`@Common.ValueList`) for dropdown fields
- Apply criticality-based color coding to any field
- Use `@UI.DataPoint` for KPIs, progress indicators, and ratings
- Understand `@UI.Chart` annotations for visual data
- Choose the right table type (Responsive, Grid, Analytical)
- Build a complete 3-level Fiori app for Purchase Order management

---

## Day 22 Recap — Quick Fire (09:00 - 09:15)

1. `@UI.SelectionFields` drives which UI component? → _____
2. `@UI.LineItem` drives which UI component? → _____
3. `@UI.HeaderInfo.Title` sets what on the Object Page? → _____
4. To show child entities as a table, the Facet target is? → _____
5. `@UI.FieldGroup#Name` organizes fields into what? → _____
6. Criticality value `1` shows which color? → _____
7. `@UI.DataPoint` values appear where? → _____
8. The file for annotations in a CAP project is typically? → _____

<details>
<summary>Answers</summary>

1. The **filter bar** (which fields can be filtered)
2. The **table columns** (List Report)
3. The **main title** of the record (e.g., "Laptop Pro")
4. `'navigationProperty/@UI.LineItem'` (e.g., `'items/@UI.LineItem'`)
5. A **form section** on the Object Page
6. **Red** (1=Negative/Error, 2=Orange/Warning, 3=Green/Success)
7. In the Object Page **header** (as KPI boxes, referenced by HeaderFacets)
8. `srv/annotations.cds` or `app/<appname>/annotations.cds`

</details>

---

## Session 1: Multi-Page Navigation & Drill-Down (09:15 - 10:30)

### The Standard Navigation Pattern

Most business apps have this drill-down flow:

```
Level 1                Level 2                  Level 3
LIST REPORT    ───►    OBJECT PAGE      ───►    SUB-OBJECT PAGE
(All Orders)           (One Order)              (One Order Item)

┌────────────┐         ┌────────────────┐       ┌────────────────┐
│ SO-001  ▸  │──click─►│ Order: SO-001  │       │ Item: Laptop   │
│ SO-002     │         │ § General Info │       │ Qty: 5         │
│ SO-003     │         │ § Items:       │       │ Price: $999    │
│            │         │  Laptop  ▸ ────│─click─►│ § Details      │
│            │         │  Mouse   ▸     │       │ § Delivery     │
└────────────┘         └────────────────┘       └────────────────┘
```

**Why 3 levels?**
- Level 1: Browse and find the right order
- Level 2: See order details + summary of items
- Level 3: See full details of ONE specific item (delivery tracking, specifications, etc.)

---

### How Navigation Works in Fiori Elements

Navigation is AUTOMATIC when you have:
1. A List Report → clicking a row navigates to the Object Page
2. A Composition/Association → clicking a child row navigates to the Sub-Object Page

**You DON'T write navigation code!** Fiori Elements handles it based on:
- Your service entity relationships (Associations/Compositions)
- Your `manifest.json` routing configuration
- Your annotations (which child entities are shown as tables)

---

### Configuring 3-Level Navigation in manifest.json

```json
{
  "sap.ui5": {
    "routing": {
      "routes": [
        {
          "pattern": ":?query:",
          "name": "OrdersList",
          "target": "OrdersList"
        },
        {
          "pattern": "Orders({key}):?query:",
          "name": "OrdersObjectPage",
          "target": "OrdersObjectPage"
        },
        {
          "pattern": "Orders({key})/items({key2}):?query:",
          "name": "OrderItemsObjectPage",
          "target": "OrderItemsObjectPage"
        }
      ],
      "targets": {
        "OrdersList": {
          "type": "Component",
          "id": "OrdersList",
          "name": "sap.fe.templates.ListReport",
          "options": {
            "settings": { "entitySet": "Orders" }
          }
        },
        "OrdersObjectPage": {
          "type": "Component",
          "id": "OrdersObjectPage",
          "name": "sap.fe.templates.ObjectPage",
          "options": {
            "settings": {
              "entitySet": "Orders",
              "navigation": {
                "items": {
                  "detail": { "route": "OrderItemsObjectPage" }
                }
              }
            }
          }
        },
        "OrderItemsObjectPage": {
          "type": "Component",
          "id": "OrderItemsObjectPage",
          "name": "sap.fe.templates.ObjectPage",
          "options": {
            "settings": { "entitySet": "OrderItems" }
          }
        }
      }
    }
  }
}
```

**Key parts:**
- **Route pattern** `"Orders({key})/items({key2})"` → defines the URL path for level 3
- **Navigation config** in the parent target → tells Fiori which child entity navigates where
- **Each level** gets its own target with `sap.fe.templates.ObjectPage`

---

### Making Child Rows Clickable (Navigation from Table)

For the items table on the Order's Object Page to be clickable (navigating to the item detail), you need:

1. The `items` entity exposed in the service
2. A route defined for it in manifest.json
3. The `navigation` property in the parent's target settings

The items table on the Order page automatically shows a "▸" chevron on each row, indicating it's navigable.

---

### Annotations for Each Level

```cds
// ─── LEVEL 1: Orders List Report ────────────────
annotate OrderService.Orders with @UI: {
  SelectionFields: [ orderNumber, status, customer_ID, orderDate ],
  LineItem: [
    { Value: orderNumber, Label: 'Order' },
    { Value: customer.customerName, Label: 'Customer' },
    { Value: orderDate, Label: 'Date' },
    { Value: grossAmount, Label: 'Amount' },
    { Value: status, Label: 'Status', Criticality: statusCriticality }
  ]
};

// ─── LEVEL 2: Order Object Page ─────────────────
annotate OrderService.Orders with @UI: {
  HeaderInfo: {
    TypeName: 'Sales Order',
    TypeNamePlural: 'Sales Orders',
    Title: { Value: orderNumber },
    Description: { Value: customer.customerName }
  },
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#OrderInfo', Label: 'Order Details' },
    { $Type: 'UI.ReferenceFacet', Target: 'items/@UI.LineItem', Label: 'Line Items' }
  ],
  FieldGroup#OrderInfo: {
    Data: [
      { Value: orderDate },
      { Value: status },
      { Value: netAmount },
      { Value: taxAmount },
      { Value: grossAmount }
    ]
  }
};

// ─── LEVEL 3: Order Item Sub-Object Page ────────
annotate OrderService.OrderItems with @UI: {
  // This LineItem is used in the ORDER's object page (parent table)
  LineItem: [
    { Value: product.productName, Label: 'Product' },
    { Value: quantity, Label: 'Qty' },
    { Value: unitPrice, Label: 'Unit Price' },
    { Value: netAmount, Label: 'Total' }
  ],
  // This HeaderInfo is used when the item has its OWN detail page
  HeaderInfo: {
    TypeName: 'Order Item',
    TypeNamePlural: 'Order Items',
    Title: { Value: product.productName },
    Description: { Value: order.orderNumber }
  },
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#ItemDetails', Label: 'Item Details' }
  ],
  FieldGroup#ItemDetails: {
    Data: [
      { Value: product.productName, Label: 'Product' },
      { Value: quantity, Label: 'Quantity' },
      { Value: unitPrice, Label: 'Unit Price' },
      { Value: netAmount, Label: 'Line Total' },
      { Value: currency_code, Label: 'Currency' }
    ]
  }
};
```

---

### Navigation Breadcrumb

When the user navigates deeper, Fiori Elements shows a breadcrumb trail:

```
Products > Laptop Pro > Review #3
   ↑          ↑            ↑
 Level 1   Level 2     Level 3 (current)
```

Users can click any level to go back. This is automatic — no coding needed!

---

## Session 2: Value Helps & Dropdown Configurations (10:45 - 12:00)

### What is a Value Help?

When users fill in a form field that references another entity (like selecting a Category or Supplier), they need a way to pick from valid values. A **Value Help** provides this:

```
Without Value Help:                 With Value Help:
┌─────────────────────┐            ┌─────────────────────┐
│ Category: [________]│            │ Category: [▼ Select ]│ ← Click
│ (type the UUID?? 🤯)│            │                     │
└─────────────────────┘            │  ┌───────────────┐  │
                                   │  │ Electronics   │  │ ← Pick from list!
                                   │  │ Accessories   │  │
                                   │  │ Software      │  │
                                   │  │ Furniture     │  │
                                   │  └───────────────┘  │
                                   └─────────────────────┘
```

---

### How Value Help Works

1. User clicks the field (or the dropdown icon)
2. Fiori Elements reads the `@Common.ValueList` annotation
3. It calls the specified entity's OData endpoint
4. It shows the values in a dropdown or dialog
5. User picks a value → the ID is saved to the field

---

### `@Common.ValueList` — The Annotation

```cds
annotate CatalogService.Products with {
  category @(
    Common: {
      Text: category.categoryName,          // Show name, not ID
      TextArrangement: #TextOnly,           // Only show text (hide ID)
      ValueList: {
        Label: 'Categories',
        CollectionPath: 'Categories',        // Which entity to fetch values from
        Parameters: [
          {
            $Type: 'Common.ValueListParameterInOut',
            LocalDataProperty: category_ID,  // Field in Products
            ValueListProperty: 'ID'          // Field in Categories
          },
          {
            $Type: 'Common.ValueListParameterDisplayOnly',
            ValueListProperty: 'categoryName' // Show this column in the dialog
          },
          {
            $Type: 'Common.ValueListParameterDisplayOnly',
            ValueListProperty: 'description'  // Also show description
          }
        ]
      }
    }
  );
};
```

---

### Breaking Down the ValueList Annotation

```
@Common.ValueList: {
  Label: 'Categories',              ← Title of the value help dialog
  CollectionPath: 'Categories',     ← Entity to query for values
  Parameters: [
    InOut:   Maps selected value to the form field (ID ↔ category_ID)
    Display: Additional columns shown in the picker (name, description)
  ]
}
```

**Parameter Types:**

| Type | Purpose | Example |
|------|---------|---------|
| `ValueListParameterInOut` | Maps the selected ID to the local field | `category_ID ↔ ID` |
| `ValueListParameterOut` | Sets a local field from the selection (one-way) | Set `categoryName` from selected |
| `ValueListParameterDisplayOnly` | Extra column shown in picker (not mapped) | Show description |
| `ValueListParameterIn` | Pre-filter the list based on another field | Filter by status |

---

### Simple Value Help — Dropdown Style

For fields with few options (< 20 values), use a simple dropdown:

```cds
annotate CatalogService.Products with {
  supplier @(
    Common: {
      Text: supplier.supplierName,
      TextArrangement: #TextOnly,
      ValueListWithFixedValues: false,    // false = dialog, true = dropdown
      ValueList: {
        CollectionPath: 'Suppliers',
        Parameters: [
          { $Type: 'Common.ValueListParameterInOut', LocalDataProperty: supplier_ID, ValueListProperty: 'ID' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'supplierName' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'city' }
        ]
      }
    }
  );
};
```

**For truly fixed lists (enums), use `ValueListWithFixedValues: true`:**

```cds
annotate CatalogService.Products with {
  status @Common.ValueListWithFixedValues;
};
```

This makes enum fields render as a simple dropdown without a search dialog.

---

### `@Common.Text` — Show Name Instead of ID

By default, association fields show the raw UUID. Fix this with `@Common.Text`:

```cds
annotate CatalogService.Products with {
  // Instead of showing "a1b2c3-d4e5..." show "Electronics"
  category @Common.Text: category.categoryName;

  // Instead of showing "s1t2u3-v4w5..." show "TechCorp Supplies"
  supplier @Common.Text: supplier.supplierName;
};

// Control how text and ID appear together:
annotate CatalogService.Products with {
  category @Common.TextArrangement: #TextOnly;       // Show only "Electronics"
  // Other options:
  // #TextFirst   → "Electronics (a1b2c3...)"
  // #TextLast    → "(a1b2c3...) Electronics"
  // #TextOnly    → "Electronics"
};
```

---

### Value Help with Dependent Filtering

Sometimes you want the value help to filter based on another field. Example: Show only products from the selected supplier:

```cds
annotate OrderService.OrderItems with {
  product @(
    Common.ValueList: {
      CollectionPath: 'Products',
      Parameters: [
        { $Type: 'Common.ValueListParameterInOut', LocalDataProperty: product_ID, ValueListProperty: 'ID' },
        { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'productName' },
        { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'price' },
        // Filter products by the order's supplier:
        { $Type: 'Common.ValueListParameterIn', LocalDataProperty: 'order/supplier_ID', ValueListProperty: 'supplier_ID' }
      ]
    }
  );
};
```

---

### Quick Reference: Value Help Patterns

| Scenario | Approach |
|----------|----------|
| Enum fields (status, type) | `@Common.ValueListWithFixedValues` |
| Small lookup table (< 20 items) | ValueList with dropdown appearance |
| Large lookup table (> 20 items) | ValueList with dialog (default) |
| Show name instead of UUID | `@Common.Text: nav.fieldName` |
| Dependent filtering | `ValueListParameterIn` |
| Hide UUID everywhere | `@Common.TextArrangement: #TextOnly` |

---
