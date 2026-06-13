# Day 23: Fiori Elements — Advanced Annotations & Navigation

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 22 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Multi-Page Navigation & Drill-Down | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Value Helps & Dropdown Configurations | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Navigation + Value Helps | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Criticality, DataPoints & Chart Annotations | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Table Types & Advanced Facets | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Build PO Management Fiori App | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

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

## Session 3: Hands-on — Navigation + Value Helps (12:00 - 13:00)

### Exercise 1: Setup 3-Level Navigation (20 minutes)

Modify your EPM Order app to support drill-down from Order → Order Item detail.

**Step 1:** Ensure your service exposes both Orders and OrderItems:
```cds
service SalesService {
  entity Orders as projection on db.SalesOrders;
  entity OrderItems as projection on db.SalesOrderItems;
  entity Products as projection on db.Products;
  entity Customers as projection on db.Customers;
}
```

**Step 2:** Add the third route in `manifest.json`:
```json
{
  "pattern": "Orders({key})/items({key2}):?query:",
  "name": "ItemDetail",
  "target": "ItemDetail"
}
```

And add the target:
```json
"ItemDetail": {
  "type": "Component",
  "id": "ItemDetail",
  "name": "sap.fe.templates.ObjectPage",
  "options": { "settings": { "entitySet": "OrderItems" } }
}
```

**Step 3:** Add navigation config in the Orders target:
```json
"OrdersObjectPage": {
  "options": {
    "settings": {
      "entitySet": "Orders",
      "navigation": {
        "items": { "detail": { "route": "ItemDetail" } }
      }
    }
  }
}
```

**Step 4:** Add annotations for OrderItems (both LineItem for parent table AND HeaderInfo for its own page):
```cds
annotate SalesService.OrderItems with @UI: {
  LineItem: [
    { Value: product.productName, Label: 'Product' },
    { Value: quantity, Label: 'Qty' },
    { Value: unitPrice, Label: 'Price' },
    { Value: netAmount, Label: 'Total' }
  ],
  HeaderInfo: {
    TypeName: 'Order Item',
    TypeNamePlural: 'Order Items',
    Title: { Value: product.productName },
    Description: { Value: quantity }
  },
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#ItemInfo', Label: 'Details' }
  ],
  FieldGroup#ItemInfo: {
    Data: [
      { Value: product.productName },
      { Value: quantity },
      { Value: unitPrice },
      { Value: netAmount },
      { Value: currency_code }
    ]
  }
};
```

**Test:** Click an order → see items table → click an item row → see item detail page!

---

### Exercise 2: Add Value Helps (20 minutes)

Add Value Helps for the `customer` and `product` fields:

```cds
// Customer value help on Orders
annotate SalesService.Orders with {
  customer @(
    Common: {
      Text: customer.customerName,
      TextArrangement: #TextOnly,
      ValueList: {
        CollectionPath: 'Customers',
        Parameters: [
          { $Type: 'Common.ValueListParameterInOut', LocalDataProperty: customer_ID, ValueListProperty: 'ID' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'customerName' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'email' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'city' }
        ]
      }
    }
  );
};

// Product value help on OrderItems
annotate SalesService.OrderItems with {
  product @(
    Common: {
      Text: product.productName,
      TextArrangement: #TextOnly,
      ValueList: {
        CollectionPath: 'Products',
        Parameters: [
          { $Type: 'Common.ValueListParameterInOut', LocalDataProperty: product_ID, ValueListProperty: 'ID' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'productName' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'price' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'stock' }
        ]
      }
    }
  );
};
```

**Test:** Click Create on an Order → the Customer field should show a value help dialog with names, emails, and cities!

---

### Exercise 3: Show Names Instead of UUIDs (20 minutes)

Make sure ALL association fields show names everywhere:

```cds
annotate SalesService.Orders with {
  customer @Common.Text: customer.customerName  @Common.TextArrangement: #TextOnly;
};

annotate SalesService.OrderItems with {
  product @Common.Text: product.productName  @Common.TextArrangement: #TextOnly;
};

annotate CatalogService.Products with {
  category @Common.Text: category.categoryName  @Common.TextArrangement: #TextOnly;
  supplier @Common.Text: supplier.supplierName  @Common.TextArrangement: #TextOnly;
};
```

**Before:** Table shows `a1b2c3d4-e5f6-7890-abcd-ef1234567890`
**After:** Table shows `Acme Corp` or `Electronics`

---

## Session 4: Criticality, DataPoints & Chart Annotations (13:45 - 14:45)

### Criticality — Color Coding Everything

Criticality adds semantic colors to values. It tells the user: "Is this good, bad, or neutral?"

```
┌────────────────────────────────────────────────┐
│ Value │ Color   │ Meaning        │ Icon        │
├───────┼─────────┼────────────────┼─────────────┤
│   0   │ Grey    │ Neutral        │ (none)      │
│   1   │ Red     │ Negative/Error │ ✖ or ⚠     │
│   2   │ Orange  │ Critical/Warn  │ ⚠           │
│   3   │ Green   │ Positive/OK    │ ✔           │
│   5   │ Blue    │ Information    │ ℹ           │
└────────────────────────────────────────────────┘
```

---

### Applying Criticality — Three Approaches

#### Approach 1: Static Field Reference

If you have a field in your entity that stores the criticality value:

```cds
entity Products : cuid {
  stock            : Integer;
  stockCriticality : Integer; // 1, 2, or 3
}

annotate CatalogService.Products with @UI.LineItem: [
  { Value: stock, Criticality: stockCriticality }
];
```

You compute `stockCriticality` in an `after READ` handler:
```javascript
this.after('READ', 'Products', (results) => {
  for (const p of results) {
    if (p.stock === 0) p.stockCriticality = 1;       // Red
    else if (p.stock < p.minStock) p.stockCriticality = 2; // Orange
    else p.stockCriticality = 3;                     // Green
  }
});
```

---

#### Approach 2: Virtual Element (Cleaner)

Add a virtual element (not persisted) for criticality:

```cds
// In your service entity projection or schema:
entity Products : cuid {
  stock    : Integer;
  minStock : Integer;
  virtual stockCriticality : Integer;  // Not in DB, computed at runtime
}
```

---

#### Approach 3: Criticality on Status Fields

Perfect for order status coloring:

```cds
annotate OrderService.Orders with @UI.LineItem: [
  { Value: status, Criticality: statusCriticality }
];
```

Handler:
```javascript
this.after('READ', 'Orders', (results) => {
  for (const o of results) {
    switch (o.status) {
      case 'New':       o.statusCriticality = 0; break; // Grey
      case 'Confirmed': o.statusCriticality = 3; break; // Green
      case 'Shipped':   o.statusCriticality = 2; break; // Orange (in transit)
      case 'Delivered': o.statusCriticality = 3; break; // Green
      case 'Cancelled': o.statusCriticality = 1; break; // Red
    }
  }
});
```

---

### Criticality on the Entire Row

You can color the entire row indicator (the left strip):

```cds
annotate OrderService.Orders with @UI.LineItem: [
  { Value: orderNumber, Label: 'Order #' },
  { Value: status, Label: 'Status', Criticality: statusCriticality },
  { Value: grossAmount, Label: 'Amount' }
] {
  // Row-level criticality:
  $Type: 'UI.DataFieldDefault';
  Criticality: statusCriticality;
};
```

---

### `@UI.DataPoint` — Rich KPI Display

DataPoints show values in enhanced formats: numbers with units, progress bars, ratings, trends.

```cds
annotate CatalogService.Products with @UI: {

  // Simple numeric KPI
  DataPoint#Price: {
    Value: price,
    Title: 'Price',
    Description: 'Selling price in USD'
  },

  // With criticality coloring
  DataPoint#Stock: {
    Value: stock,
    Title: 'Stock Level',
    Criticality: stockCriticality,
    CriticalityRepresentation: #WithoutIcon   // or #WithIcon
  },

  // As a star rating (1-5)
  DataPoint#Rating: {
    Value: rating,
    Title: 'Customer Rating',
    TargetValue: 5,                          // Max stars
    Visualization: #Rating                   // Shows ⭐⭐⭐⭐☆
  },

  // As a progress bar
  DataPoint#Completion: {
    Value: completionPercent,
    Title: 'Completion',
    TargetValue: 100,
    Visualization: #Progress,                // Shows progress bar
    Criticality: completionCriticality
  },

  // With trend arrow (up/down/stable)
  DataPoint#Revenue: {
    Value: monthlyRevenue,
    Title: 'Revenue',
    Trend: revenueTrend                      // 1=Up, 2=Down, 3=Stable (shows ↑ ↓ →)
  }
};
```

**DataPoint Visualizations:**

| Visualization | What It Shows | Use For |
|---------------|---------------|---------|
| (default) | Plain number | Prices, amounts, counts |
| `#Rating` | Star display ⭐⭐⭐⭐☆ | Ratings out of 5 |
| `#Progress` | Progress bar ████░░░░ 60% | Completion percentage |
| (with Trend) | Number with arrow ↑↓→ | KPIs with period comparison |

---

### `@UI.Chart` — Micro Charts

You can add small charts (micro charts) to the Object Page header or table cells:

```cds
annotate CatalogService.Products with @UI: {
  // Bullet micro chart in header
  Chart#StockChart: {
    ChartType: #Bullet,
    Measures: [stock],
    MeasureAttributes: [{
      Measure: stock,
      Role: #Axis1,
      DataPoint: '@UI.DataPoint#Stock'
    }]
  }
};
```

**Available micro chart types:**
- `#Bullet` — bar against a target
- `#Radial` — circular progress
- `#Area` — small area/line chart
- `#Column` — mini bar chart
- `#Line` — trend line
- `#Harvey` — pie-like segment

**Note:** Full chart annotations are complex. For simple KPI display, `DataPoint` with `#Rating` or `#Progress` is usually sufficient.

---

### Putting DataPoints in the Header

Remember: DataPoints must be referenced by `@UI.HeaderFacets` to appear:

```cds
annotate OrderService.Orders with @UI: {
  HeaderFacets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Amount' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Status' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Items' }
  ],
  DataPoint#Amount: {
    Value: grossAmount,
    Title: 'Total Amount',
    Criticality: amountCriticality
  },
  DataPoint#Status: {
    Value: status,
    Title: 'Order Status',
    Criticality: statusCriticality
  },
  DataPoint#Items: {
    Value: itemCount,
    Title: 'Line Items'
  }
};
```

**Result:**
```
┌───────────────┐ ┌──────────────┐ ┌──────────────┐
│ Total Amount  │ │ Order Status │ │ Line Items   │
│ 🟢 $4,500.00 │ │ 🟢 Confirmed │ │ 5            │
└───────────────┘ └──────────────┘ └──────────────┘
```

---

## Session 5: Table Types & Advanced Facets (15:00 - 16:00)

### Table Types in Fiori Elements

Fiori Elements supports different table types for different use cases:

```
┌─────────────────────────────────────────────────────────────┐
│  TABLE TYPES                                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. RESPONSIVE TABLE (Default)                              │
│     Best for: Lists with < 20 columns                       │
│     Features: Adapts to screen size, hides columns on       │
│               mobile, good for most use cases               │
│     ┌─────────────────────────────────────────┐             │
│     │ Name       │ Price  │ Stock │ Status    │             │
│     │ Laptop     │ $999   │ 45    │ ✅ Active │             │
│     └─────────────────────────────────────────┘             │
│                                                             │
│  2. GRID TABLE                                              │
│     Best for: Large datasets with many columns (20+)        │
│     Features: Fixed header, virtual scrolling,              │
│               resizable columns, Excel-like feel            │
│     ┌─────────────────────────────────────────────────────┐ │
│     │ A    │ B    │ C    │ D    │ E    │ F    │ G   │ ▸  │ │
│     │──────┼──────┼──────┼──────┼──────┼──────┼─────┼────│ │
│     │ val  │ val  │ val  │ val  │ val  │ val  │ val │    │ │
│     │ val  │ val  │ val  │ val  │ val  │ val  │ val │    │ │
│     └─────────────────────────────────────────────────────┘ │
│                                                             │
│  3. ANALYTICAL TABLE                                        │
│     Best for: Aggregated data with totals/subtotals         │
│     Features: Group by, sum, average, hierarchies           │
│     ┌─────────────────────────────────────────────────┐     │
│     │ ▼ Region    │ Revenue  │ Orders │ Avg Value    │     │
│     │   ► North   │ $45,000  │ 120    │ $375         │     │
│     │   ► South   │ $38,000  │ 95     │ $400         │     │
│     │   TOTAL     │ $83,000  │ 215    │ $386         │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Configuring Table Type in manifest.json

```json
"ProductsList": {
  "type": "Component",
  "name": "sap.fe.templates.ListReport",
  "options": {
    "settings": {
      "entitySet": "Products",
      "controlConfiguration": {
        "@com.sap.vocabularies.UI.v1.LineItem": {
          "tableSettings": {
            "type": "ResponsiveTable"
          }
        }
      }
    }
  }
}
```

**Table type values:**
- `"ResponsiveTable"` — default, mobile-friendly
- `"GridTable"` — for wide datasets, desktop-focused
- `"AnalyticalTable"` — for aggregation/grouping

---

### When to Use Which Table

| Table Type | Records | Columns | Device | Use Case |
|-----------|---------|---------|--------|----------|
| Responsive | < 200 visible | < 10 | All devices | Standard lists, mobile-first |
| Grid | 200+ | 10-50 | Desktop only | Financial data, admin panels |
| Analytical | Any | Any | Desktop | Reports with totals, grouping |

---

### Advanced Facets — Collection Facets (Grouping Sections)

A `CollectionFacet` groups multiple `ReferenceFacets` under one section with tabs or sub-sections:

```cds
annotate OrderService.Orders with @UI: {
  Facets: [
    // Section 1: Simple reference
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#General', Label: 'General' },

    // Section 2: Collection of sub-sections (tabs within a section!)
    {
      $Type: 'UI.CollectionFacet',
      Label: 'Financial Details',
      ID: 'FinancialSection',
      Facets: [
        { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Pricing', Label: 'Pricing' },
        { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Tax', Label: 'Tax Info' },
        { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Payment', Label: 'Payment' }
      ]
    },

    // Section 3: Items table
    { $Type: 'UI.ReferenceFacet', Target: 'items/@UI.LineItem', Label: 'Order Items' }
  ]
};
```

**Result:** The "Financial Details" section appears as ONE section with 3 sub-tabs: Pricing | Tax Info | Payment.

---

### Adding Actions to the Object Page

You can show bound action buttons on the Object Page header:

```cds
annotate OrderService.Orders with @UI.Identification: [
  { $Type: 'UI.DataFieldForAction', Action: 'OrderService.confirm', Label: 'Confirm Order' },
  { $Type: 'UI.DataFieldForAction', Action: 'OrderService.cancel', Label: 'Cancel Order' },
  { $Type: 'UI.DataFieldForAction', Action: 'OrderService.ship', Label: 'Ship Order' }
];
```

**Result:** Buttons appear in the Object Page header toolbar:
```
◀ Back    Order: SO-001    [Confirm Order] [Ship Order] [Cancel Order] [Edit] [Delete]
```

You can also add actions to the table toolbar (List Report):

```cds
annotate OrderService.Orders with @UI.LineItem: [
  { Value: orderNumber },
  { Value: status },
  { $Type: 'UI.DataFieldForAction', Action: 'OrderService.confirm', Label: 'Confirm', Inline: true }
];
```

`Inline: true` shows the button in each row. Without it, the button appears in the table toolbar.

---

### Conditional Actions (Show/Hide Based on Status)

You can control when actions are available using `@Core.OperationAvailable`:

```cds
annotate OrderService.confirm with @Core.OperationAvailable: {
  $edmJson: { $If: [{ $Eq: [{ $Path: 'status' }, 'New'] }, true, false] }
};
```

Simpler approach — use a computed Boolean field:
```cds
annotate OrderService.Orders with {
  confirmEnabled @Core.Computed;  // Computed in handler
};

annotate OrderService.confirm with @Core.OperationAvailable: confirmEnabled;
```

Handler:
```javascript
this.after('READ', 'Orders', (results) => {
  for (const o of results) {
    o.confirmEnabled = o.status === 'New';
    o.shipEnabled = o.status === 'Confirmed';
    o.cancelEnabled = !['Delivered', 'Cancelled'].includes(o.status);
  }
});
```

---

## Session 6: Hands-on — Build PO Management Fiori App (16:00 - 16:45)

### Exercise: Complete PO Management App

Build a 3-level Fiori Elements app for Purchase Orders:

```
PO List (List Report) → PO Detail (Object Page) → PO Item (Sub-Object Page)
```

---

### Complete Annotations for PO App

**File: `app/po-management/annotations.cds`**

```cds
using { PurchasingService } from '../../srv/purchasing-service';

// ═══════════════════════════════════════════════════
// PURCHASE ORDERS — List Report + Object Page
// ═══════════════════════════════════════════════════

annotate PurchasingService.PurchaseOrders with @UI: {

  // ─── FILTER BAR ────────────────────────────────
  SelectionFields: [ poNumber, status, priority, supplier_ID, orderDate ],

  // ─── TABLE COLUMNS ─────────────────────────────
  LineItem: [
    { Value: poNumber, Label: 'PO Number' },
    { Value: supplier.supplierName, Label: 'Supplier' },
    { Value: orderDate, Label: 'Order Date' },
    { Value: totalAmount, Label: 'Amount' },
    { Value: priority, Label: 'Priority', Criticality: priorityCriticality },
    { Value: status, Label: 'Status', Criticality: statusCriticality }
  ],

  // ─── OBJECT PAGE HEADER ────────────────────────
  HeaderInfo: {
    TypeName: 'Purchase Order',
    TypeNamePlural: 'Purchase Orders',
    Title: { Value: poNumber },
    Description: { Value: supplier.supplierName }
  },

  // ─── HEADER KPIs ──────────────────────────────
  HeaderFacets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Amount' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Status' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Priority' }
  ],
  DataPoint#Amount: { Value: totalAmount, Title: 'Total Amount' },
  DataPoint#Status: { Value: status, Title: 'Status', Criticality: statusCriticality },
  DataPoint#Priority: { Value: priority, Title: 'Priority', Criticality: priorityCriticality },

  // ─── PAGE SECTIONS ─────────────────────────────
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#General', Label: 'General Information' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Dates', Label: 'Dates & Delivery' },
    { $Type: 'UI.ReferenceFacet', Target: 'items/@UI.LineItem', Label: 'Line Items' }
  ],

  FieldGroup#General: {
    Data: [
      { Value: poNumber, Label: 'PO Number' },
      { Value: supplier_ID, Label: 'Supplier' },
      { Value: status, Label: 'Status' },
      { Value: priority, Label: 'Priority' },
      { Value: totalAmount, Label: 'Total Amount' },
      { Value: notes, Label: 'Notes' }
    ]
  },
  FieldGroup#Dates: {
    Data: [
      { Value: orderDate, Label: 'Order Date' },
      { Value: expectedDate, Label: 'Expected Delivery' },
      { Value: createdAt, Label: 'Created On' },
      { Value: createdBy, Label: 'Created By' }
    ]
  },

  // ─── ACTIONS (Buttons on Object Page) ──────────
  Identification: [
    { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.submit', Label: 'Submit for Approval' },
    { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.approve', Label: 'Approve' },
    { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.reject', Label: 'Reject' }
  ]
};

// ─── VALUE HELPS ─────────────────────────────────
annotate PurchasingService.PurchaseOrders with {
  supplier @(
    Common: {
      Text: supplier.supplierName,
      TextArrangement: #TextOnly,
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
  status @Common.ValueListWithFixedValues;
  priority @Common.ValueListWithFixedValues;
};

// ═══════════════════════════════════════════════════
// PO ITEMS — Table in PO + Sub-Object Page
// ═══════════════════════════════════════════════════

annotate PurchasingService.PurchaseOrderItems with @UI: {

  // Shown as table in PO Object Page
  LineItem: [
    { Value: product.productName, Label: 'Product' },
    { Value: quantity, Label: 'Quantity' },
    { Value: unitPrice, Label: 'Unit Price' },
    { Value: totalPrice, Label: 'Total' },
    { Value: receivedQty, Label: 'Received' }
  ],

  // When item has its own detail page
  HeaderInfo: {
    TypeName: 'PO Item',
    TypeNamePlural: 'PO Items',
    Title: { Value: product.productName },
    Description: { Value: quantity }
  },

  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#ItemDetail', Label: 'Item Details' }
  ],
  FieldGroup#ItemDetail: {
    Data: [
      { Value: product_ID, Label: 'Product' },
      { Value: quantity, Label: 'Ordered Qty' },
      { Value: unitPrice, Label: 'Unit Price' },
      { Value: totalPrice, Label: 'Line Total' },
      { Value: receivedQty, Label: 'Received Qty' },
      { Value: notes, Label: 'Notes' }
    ]
  }
};

// Product value help for items
annotate PurchasingService.PurchaseOrderItems with {
  product @(
    Common: {
      Text: product.productName,
      TextArrangement: #TextOnly,
      ValueList: {
        CollectionPath: 'Products',
        Parameters: [
          { $Type: 'Common.ValueListParameterInOut', LocalDataProperty: product_ID, ValueListProperty: 'ID' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'productName' },
          { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'unitPrice' }
        ]
      }
    }
  );
};
```

---

### Handler for Criticality Values

```javascript
// srv/purchasing-service.js (add to existing handlers)

this.after('READ', 'PurchaseOrders', (results) => {
  const pos = Array.isArray(results) ? results : [results];
  for (const po of pos) {
    // Status criticality
    switch (po.status) {
      case 'Draft':     po.statusCriticality = 0; break;
      case 'Pending':   po.statusCriticality = 2; break;
      case 'Approved':  po.statusCriticality = 3; break;
      case 'Rejected':  po.statusCriticality = 1; break;
      case 'Ordered':   po.statusCriticality = 3; break;
      case 'Received':  po.statusCriticality = 3; break;
      default:          po.statusCriticality = 0;
    }
    // Priority criticality
    switch (po.priority) {
      case 'Low':      po.priorityCriticality = 0; break;
      case 'Medium':   po.priorityCriticality = 0; break;
      case 'High':     po.priorityCriticality = 2; break;
      case 'Critical': po.priorityCriticality = 1; break;
      default:         po.priorityCriticality = 0;
    }
  }
});
```

---

## Assessment: MCQ — 15 Questions on Advanced Annotations

**Q1.** `@Common.ValueList` is used for:

- A) Sorting table columns
- B) Providing a dropdown/dialog to pick values from a related entity
- C) Validating input values
- D) Creating charts

**Answer: B** — ValueList creates a value help (dropdown or dialog) allowing users to select from valid options.

---

**Q2.** `@Common.Text: supplier.supplierName` does what?

- A) Creates a text search field
- B) Shows the supplier NAME instead of the raw UUID in display mode
- C) Adds a tooltip
- D) Makes the field required

**Answer: B** — `@Common.Text` tells Fiori to display the human-readable text (name) instead of the technical key (UUID).

---

**Q3.** `@Common.TextArrangement: #TextOnly` means:

- A) Show only the text, hide the ID completely
- B) Show only the ID, hide the text
- C) Show text and ID together
- D) Show text in uppercase

**Answer: A** — `#TextOnly` displays only the readable text. Other options: `#TextFirst` (text + ID), `#TextLast` (ID + text).

---

**Q4.** Criticality value `2` renders as which color?

- A) Red
- B) Orange/Yellow (Warning)
- C) Green
- D) Grey

**Answer: B** — 0=Grey, 1=Red, 2=Orange(Warning), 3=Green.

---

**Q5.** To show a star rating (⭐⭐⭐⭐☆) in the Object Page header:

- A) Use `@UI.LineItem` with type 'Rating'
- B) Use `@UI.DataPoint` with `Visualization: #Rating` and `TargetValue: 5`
- C) Use `@UI.Chart` of type Rating
- D) Use a custom HTML fragment

**Answer: B** — `DataPoint` with `#Rating` visualization renders as stars. `TargetValue` is the maximum (5 stars).

---

**Q6.** To make items in a table row navigable to a sub-object page, you need:

- A) A `@UI.Navigation` annotation
- B) A route in `manifest.json` and `navigation` config in the parent target settings
- C) A separate Freestyle page
- D) A `@UI.Link` annotation on the field

**Answer: B** — Navigation between pages is configured in `manifest.json` routing. Fiori Elements auto-shows the chevron (▸) when a route exists for the child.

---

**Q7.** `@UI.DataFieldForAction` is used for:

- A) Creating form fields
- B) Adding action buttons (like "Approve", "Reject") to the Object Page or table
- C) Defining filter fields
- D) Adding computed columns

**Answer: B** — `DataFieldForAction` adds bound action buttons to the UI. Place in `@UI.Identification` for Object Page toolbar, or in `@UI.LineItem` for table toolbar/inline buttons.

---

**Q8.** A `CollectionFacet` differs from a `ReferenceFacet` because it:

- A) Groups multiple sub-sections/tabs into one parent section
- B) References an external URL
- C) Collects data from multiple services
- D) Only works with charts

**Answer: A** — CollectionFacet is a container that groups multiple ReferenceFacets into one section (rendered as sub-tabs).

---

**Q9.** For a field with few fixed values (like an enum with 4 options), the best value help approach is:

- A) Full ValueList with dialog
- B) `@Common.ValueListWithFixedValues` (renders as simple dropdown)
- C) A custom freestyle component
- D) No annotation needed — enums auto-generate dropdowns

**Answer: B** — `@Common.ValueListWithFixedValues` renders a simple dropdown without a search dialog. Perfect for enums.

---

**Q10.** The `Grid Table` type is best used when:

- A) The app runs on mobile phones
- B) You have many columns (20+) and large datasets, primarily for desktop
- C) You need animated transitions
- D) You have less than 10 records

**Answer: B** — Grid Tables are for desktop-heavy scenarios with lots of columns. They support virtual scrolling and resizable columns.

---

**Q11.** `Target: 'items/@UI.LineItem'` in a Facet means:

- A) Filter items by the LineItem value
- B) Show the `items` navigation property as a table using its LineItem annotation
- C) Link to an external items page
- D) Show a chart of items

**Answer: B** — The path `'items/@UI.LineItem'` means: follow the `items` navigation property (association), use its `@UI.LineItem` annotation to render a table.

---

**Q12.** `ValueListParameterIn` in a ValueList does what?

- A) Pre-filters the value help list based on another field's current value
- B) Maps the selected value back to the form field
- C) Marks the field as required
- D) Enables inline editing

**Answer: A** — `ParameterIn` filters the value list. Example: Only show products for the currently selected supplier.

---

**Q13.** To show a progress bar (████░░ 60%) in the header:

- A) `@UI.DataPoint` with `Visualization: #Progress` and `TargetValue: 100`
- B) `@UI.Chart` with ChartType `#Bar`
- C) `@UI.LineItem` with type Progress
- D) Custom CSS styling

**Answer: A** — DataPoint with `#Progress` visualization shows a progress bar. TargetValue defines 100%.

---

**Q14.** `@UI.Identification` annotations appear where?

- A) In the filter bar
- B) As action buttons in the Object Page header toolbar
- C) In the table footer
- D) In the browser title

**Answer: B** — `@UI.Identification` with `DataFieldForAction` entries renders as buttons in the Object Page header toolbar.

---

**Q15.** The `Trend` property in a DataPoint (values: 1=Up, 2=Down, 3=Stable) shows:

- A) A color change
- B) An arrow icon (↑ ↓ →) next to the KPI value indicating direction of change
- C) A sparkline chart
- D) A percentage change

**Answer: B** — Trend adds directional arrows next to the value: ↑ (improving), ↓ (declining), → (stable).

---

## Assignment: Create Complete Fiori App for Employee Management

### Task

Build a fully annotated 2-level Fiori Elements app for **Employee Management** with all advanced features learned today.

---

### Requirements

**Data Model (use existing or create):**
```cds
entity Employees : cuid, managed {
  firstName  : String(50);
  lastName   : String(50);
  email      : String(255);
  phone      : String(20);
  hireDate   : Date;
  salary     : Decimal(10,2);
  jobTitle   : String(100);
  isActive   : Boolean default true;
  department : Association to Departments;
  skills     : Association to many EmployeeSkills on skills.employee = $self;
}
entity Departments : cuid, managed {
  deptName : String(100);
  location : String(100);
}
entity EmployeeSkills : cuid {
  employee    : Association to Employees;
  skillName   : String(50);
  proficiency : String(20);
}
```

---

### Annotation Requirements

| Feature | Details |
|---------|---------|
| **Filter Bar** | firstName, lastName, department, jobTitle, isActive |
| **Table Columns** | Full name (first+last), email, department name, job title, hire date, active status (with criticality) |
| **Object Page Header** | Title = firstName + lastName, Description = jobTitle |
| **Header KPIs** | Salary (with currency), Hire Date, Active Status (green/red) |
| **Section 1** | "Personal Info": firstName, lastName, email, phone |
| **Section 2** | "Employment": jobTitle, department, hireDate, salary, isActive |
| **Section 3** | "Skills" table (from EmployeeSkills: skillName, proficiency) |
| **Value Helps** | Department dropdown with name + location |
| **Text Annotations** | Show department NAME instead of UUID |
| **Actions** | "Deactivate" and "Promote" buttons on Object Page |
| **Criticality** | isActive: true=Green, false=Red |

---

### Deliverables

1. `srv/annotations.cds` with all annotations
2. Handler code for criticality computation
3. Screenshots or description of the working app (List Report + Object Page)

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Multi-level navigation | manifest.json routes + child entity annotations |
| Value Helps | `@Common.ValueList` for dropdowns/dialogs |
| `@Common.Text` | Show names instead of UUIDs |
| `TextArrangement` | `#TextOnly`, `#TextFirst`, `#TextLast` |
| Criticality | 0=Grey, 1=Red, 2=Orange, 3=Green |
| DataPoint visualizations | `#Rating` (stars), `#Progress` (bar), Trend (arrows) |
| Chart annotations | Micro charts for header/inline display |
| Table types | Responsive (mobile), Grid (desktop, many columns), Analytical (aggregation) |
| CollectionFacet | Groups multiple sections under one tab |
| `@UI.Identification` | Action buttons on Object Page |
| `DataFieldForAction` | Button in table or Object Page toolbar |

### The Annotation Relationship Map

```
@UI.SelectionFields    → Filter bar fields
@UI.LineItem           → Table columns (+ inline actions)
@UI.HeaderInfo         → Object Page title/subtitle
@UI.HeaderFacets       → KPI boxes → reference DataPoints
@UI.DataPoint          → KPI values (number, rating, progress, trend)
@UI.Facets             → Page sections → reference FieldGroups or child LineItems
@UI.FieldGroup         → Form fields within a section
@UI.Identification     → Action buttons on Object Page
@UI.Chart              → Micro charts
@Common.ValueList      → Dropdown/dialog for picking related values
@Common.Text           → Display text instead of ID
Criticality            → Color coding (0-3)
```

### The One-Liner

> **Advanced annotations turn a basic data form into a rich, colorful, interactive business application — all without writing a single line of JavaScript UI code.**

---

### Looking Ahead: Day 24

Tomorrow: **Fiori Elements — Draft Handling & Create/Edit Flows**
- Draft-enabled editing (save progress without committing)
- Create and Edit workflows
- Side effects (field changes trigger other updates)
- Validation messages in the UI
- Custom columns and sections via extensions

---

*End of Day 23 — Your Fiori apps now have navigation, colors, dropdowns, and action buttons!*
