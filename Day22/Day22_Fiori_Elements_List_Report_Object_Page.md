# Day 22: Fiori Elements — List Report & Object Page

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 21 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: List Report Floorplan — Deep Dive | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Object Page Floorplan — Deep Dive | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Generate a Fiori Elements App | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: UI Annotations — The Complete Syntax | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Annotate Your EPM Model | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Object Page with Sections & Fields | 45 min |
| 16:45 - 17:00 | Assessment & Annotation Quiz | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain the complete anatomy of a List Report page
- Explain the complete anatomy of an Object Page
- Use SAP Fiori Tools to generate a Fiori Elements application
- Write `@UI.SelectionFields` annotations for filter bars
- Write `@UI.LineItem` annotations for table columns
- Write `@UI.HeaderInfo` for the Object Page header
- Write `@UI.FieldGroup` to organize fields into sections
- Configure `@UI.Facets` to structure the Object Page layout
- See your annotations rendered as a live, working Fiori app

---

## Day 21 Recap — Quick Fire (09:00 - 09:15)

1. Name the 5 SAP Fiori design principles → _____
2. Fiori Elements generates UI from _____ ? → _____
3. Freestyle UI5 requires writing _____ and _____ ? → _____
4. Which floorplan shows a filterable table of records? → _____
5. Which floorplan shows full details of ONE record? → _____
6. `$metadata` describes what to Fiori Elements? → _____
7. Annotations go in which file? → _____
8. SAP's recommendation: use _____ first, then Freestyle only when needed? → _____

<details>
<summary>Answers</summary>

1. **R**ole-based, **A**daptive, **C**oherent, **S**imple, **D**elightful
2. **Annotations** (and $metadata)
3. **XML views** and **JavaScript controllers**
4. **List Report**
5. **Object Page**
6. The **API schema** — entity types, fields, relationships, actions
7. `srv/annotations.cds` (or a CDS file in srv/ or app/)
8. **Fiori Elements**

</details>

---

## Session 1: List Report Floorplan — Deep Dive (09:15 - 10:30)

### Anatomy of a List Report

Every List Report has these components:

```
┌─────────────────────────────────────────────────────────────────┐
│  ① SHELL BAR (Fiori Launchpad header — app name, user, etc.)   │
├─────────────────────────────────────────────────────────────────┤
│  ② PAGE HEADER                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Products (47)                               [+ Create]   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ③ FILTER BAR (Smart Filter / Selection Fields)                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Product Name: [________]  Status: [▼ All]  Price: [_____] │  │
│  │                                   [Go]  [Adapt Filters]   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ④ TABLE TOOLBAR                                                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Products (47)  │ [Create] [Delete] │ ⚙ Settings │ ⬇ Export│  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ⑤ TABLE CONTENT (LineItem annotations)                         │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ □ │ Product Name     │ Price    │ Stock │ Status    │ ▸   │  │
│  ├───┼──────────────────┼──────────┼───────┼───────────┼─────┤  │
│  │ □ │ Laptop Pro       │ $999.00  │ 45    │ ✅ Active │ ▸   │  │
│  │ □ │ Wireless Mouse   │ $29.99   │ 120   │ ✅ Active │ ▸   │  │
│  │ □ │ USB-C Hub        │ $49.99   │ 80    │ ⚠️ Low   │ ▸   │  │
│  │ □ │ Old Keyboard     │ $19.99   │ 0     │ ❌ Out   │ ▸   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ⑥ FOOTER / PAGINATION                                          │
│  │ Showing 1-20 of 47 │  [< Previous] [Next >]               │  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### How Each Part Maps to Annotations

| # | Component | Driven By | Annotation |
|---|-----------|-----------|------------|
| ① | Shell Bar | Fiori Launchpad config | (automatic) |
| ② | Page Header | Entity name | `@UI.HeaderInfo.TypeNamePlural` |
| ③ | Filter Bar | Selection fields | `@UI.SelectionFields` |
| ④ | Table Toolbar | Built-in | (automatic: Create, Delete, Settings) |
| ⑤ | Table Columns | Line items | `@UI.LineItem` |
| ⑥ | Pagination | OData $top/$skip | (automatic) |

**You control ②, ③, and ⑤ through annotations. The rest is automatic!**

---

### The Filter Bar — `@UI.SelectionFields`

The filter bar lets users narrow down the table results. You control which fields appear as filter options:

```cds
annotate CatalogService.Products with @UI: {
  SelectionFields: [
    productName,
    price,
    stock,
    category_ID,
    supplier_ID
  ]
};
```

**Result:**
```
┌─────────────────────────────────────────────────────────────┐
│ Product Name: [________] Price: [___] Stock: [___]          │
│ Category: [▼ Select]     Supplier: [▼ Select]    [Go]       │
└─────────────────────────────────────────────────────────────┘
```

**Rules:**
- List fields users will commonly search/filter by
- Association fields (like `category_ID`) become dropdown selects
- String fields become text inputs
- Number fields become range inputs
- Date fields become date pickers
- Keep it to 3-7 fields (too many overwhelms users)

---

### The Table — `@UI.LineItem`

`@UI.LineItem` defines which columns appear in the table and in what order:

```cds
annotate CatalogService.Products with @UI: {
  LineItem: [
    { Value: productName, Label: 'Product' },
    { Value: price, Label: 'Price' },
    { Value: stock, Label: 'In Stock' },
    { Value: rating, Label: 'Rating' },
    { Value: supplier.supplierName, Label: 'Supplier' },
    { Value: category.categoryName, Label: 'Category' }
  ]
};
```

**Result:**
```
│ Product         │ Price    │ In Stock │ Rating │ Supplier   │ Category    │
├─────────────────┼──────────┼──────────┼────────┼────────────┼─────────────┤
│ Laptop Pro      │ $999.00  │ 45       │ ⭐ 4.8 │ TechCorp   │ Electronics │
│ Wireless Mouse  │ $29.99   │ 120      │ ⭐ 4.5 │ TechCorp   │ Accessories │
```

---

### LineItem — Advanced Column Options

```cds
LineItem: [
  // Basic column
  { Value: productName },

  // With custom label
  { Value: productName, Label: 'Product Name' },

  // With importance (controls visibility on small screens)
  { Value: description, Importance: #Low },

  // Navigation path (access related entity's field)
  { Value: supplier.supplierName, Label: 'Supplier' },

  // With criticality (colored status indicators)
  { Value: stock, Criticality: stockCriticality },

  // As a rating indicator
  { Value: rating, ![@UI.Importance]: #High }
]
```

**Column Importance:**
- `#High` → Always shown (even on phone)
- `#Medium` → Shown on tablet and desktop (default)
- `#Low` → Only shown on desktop (hidden on smaller screens)

---

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

---

### Quick Exercise: Design a List Report

For an `Employees` entity, write the SelectionFields and LineItem annotations:

**Entity:**
```cds
entity Employees : cuid, managed {
  firstName  : String(50);
  lastName   : String(50);
  email      : String(255);
  department : Association to Departments;
  hireDate   : Date;
  salary     : Decimal(10,2);
  isActive   : Boolean;
}
```

**Your task:** Users should be able to filter by department and active status. The table should show full name, email, department, hire date, and status.

<details>
<summary>Answer</summary>

```cds
annotate HRService.Employees with @UI: {
  SelectionFields: [ department_ID, isActive ],
  LineItem: [
    { Value: firstName, Label: 'First Name' },
    { Value: lastName, Label: 'Last Name' },
    { Value: email },
    { Value: department.deptName, Label: 'Department' },
    { Value: hireDate, Label: 'Hired' },
    { Value: isActive, Label: 'Active' }
  ]
};
```

</details>

---

## Session 2: Object Page Floorplan — Deep Dive (10:45 - 12:00)

### Anatomy of an Object Page

When a user clicks a row in the List Report, they land on the Object Page:

```
┌─────────────────────────────────────────────────────────────────┐
│  ◀ Back                                        [Edit] [Delete]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ① HEADER — Always Visible (Sticky)                             │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                           │  │
│  │  🖼️  Laptop Pro                                           │  │
│  │      Electronics | TechCorp Supplies                      │  │
│  │                                                           │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐               │  │
│  │  │ Price    │  │ Stock    │  │ Rating   │               │  │
│  │  │ $999.00  │  │ 45 units │  │ ⭐ 4.8   │               │  │
│  │  └──────────┘  └──────────┘  └──────────┘               │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ② ANCHOR BAR (Section Navigation)                              │
│  │ General Info │ Pricing │ Supplier │ Reviews │                │
│                                                                 │
│  ③ SECTIONS (Scrollable Content)                                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ § General Information                                     │  │
│  │ ┌─────────────────────────────────────────────────────┐   │  │
│  │ │ Product Name:  Laptop Pro                           │   │  │
│  │ │ Description:   High-performance business laptop     │   │  │
│  │ │ Category:      Electronics                          │   │  │
│  │ │ SKU:           PRD-00001                            │   │  │
│  │ │ Created:       June 1, 2026                         │   │  │
│  │ └─────────────────────────────────────────────────────┘   │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │ § Pricing & Stock                                         │  │
│  │ ┌─────────────────────────────────────────────────────┐   │  │
│  │ │ Price:         $999.00                              │   │  │
│  │ │ Currency:      USD                                  │   │  │
│  │ │ Stock:         45                                   │   │  │
│  │ │ Min Stock:     10                                   │   │  │
│  │ │ Rating:        4.8 / 5.0                            │   │  │
│  │ └─────────────────────────────────────────────────────┘   │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │ § Order Items (Table)                                     │  │
│  │ ┌─────────────────────────────────────────────────────┐   │  │
│  │ │ Order#    │ Customer  │ Qty │ Date       │ Amount  │   │  │
│  │ │ SO-001    │ Acme Corp │ 5   │ Jun 1      │ $4,995  │   │  │
│  │ │ SO-005    │ Beta Ltd  │ 2   │ Jun 3      │ $1,998  │   │  │
│  │ └─────────────────────────────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### How the Object Page Maps to Annotations

| # | Component | Annotation |
|---|-----------|------------|
| ① | Header title + subtitle | `@UI.HeaderInfo` |
| ① | Header KPI values | `@UI.DataPoint` |
| ① | Header facets (the KPI boxes) | `@UI.HeaderFacets` |
| ② | Anchor bar (section tabs) | Auto from `@UI.Facets` |
| ③ | Each section | `@UI.Facets` → references `@UI.FieldGroup` or `@UI.LineItem` |

---

### `@UI.HeaderInfo` — The Identity of the Record

```cds
annotate CatalogService.Products with @UI: {
  HeaderInfo: {
    TypeName: 'Product',              // Singular name
    TypeNamePlural: 'Products',       // Plural name (used in List Report title)
    Title: {
      Value: productName              // Main title on Object Page
    },
    Description: {
      Value: category.categoryName    // Subtitle below the title
    },
    ImageUrl: imageUrl                // Optional: show an image/avatar
  }
};
```

**What this renders:**
```
┌──────────────────────────────────────┐
│  Laptop Pro                          │  ← Title (productName)
│  Electronics                         │  ← Description (category name)
└──────────────────────────────────────┘
```

---

### `@UI.HeaderFacets` — KPI Boxes in the Header

```cds
annotate CatalogService.Products with @UI: {
  HeaderFacets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Price' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Stock' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Rating' }
  ],
  DataPoint#Price: {
    Value: price,
    Title: 'Price'
  },
  DataPoint#Stock: {
    Value: stock,
    Title: 'Stock',
    Criticality: stockCriticality
  },
  DataPoint#Rating: {
    Value: rating,
    Title: 'Rating',
    TargetValue: 5,
    Visualization: #Rating
  }
};
```

**What this renders in the header:**
```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Price    │  │ Stock    │  │ Rating   │
│ $999.00  │  │ 🟢 45   │  │ ⭐⭐⭐⭐⭐ │
└──────────┘  └──────────┘  └──────────┘
```

---

### `@UI.Facets` — The Page Structure (Sections)

Facets define WHAT sections appear on the Object Page and in what order:

```cds
annotate CatalogService.Products with @UI: {
  Facets: [
    // Section 1: General Information (form fields)
    {
      $Type: 'UI.ReferenceFacet',
      Target: '@UI.FieldGroup#General',
      Label: 'General Information'
    },
    // Section 2: Pricing & Stock (form fields)
    {
      $Type: 'UI.ReferenceFacet',
      Target: '@UI.FieldGroup#Pricing',
      Label: 'Pricing & Stock'
    },
    // Section 3: Reviews (table of related items)
    {
      $Type: 'UI.ReferenceFacet',
      Target: 'reviews/@UI.LineItem',
      Label: 'Customer Reviews'
    }
  ]
};
```

**Types of facets:**
- `UI.ReferenceFacet` → Points to a FieldGroup (form) or LineItem (table)
- `UI.CollectionFacet` → Groups multiple ReferenceFacets into one section with tabs

---

### `@UI.FieldGroup` — Form Fields in a Section

```cds
annotate CatalogService.Products with @UI: {
  FieldGroup#General: {
    Label: 'General Information',
    Data: [
      { Value: productName, Label: 'Product Name' },
      { Value: description, Label: 'Description' },
      { Value: category.categoryName, Label: 'Category' },
      { Value: sku, Label: 'SKU' },
      { Value: createdAt, Label: 'Created On' },
      { Value: modifiedAt, Label: 'Last Modified' }
    ]
  },
  FieldGroup#Pricing: {
    Label: 'Pricing & Stock',
    Data: [
      { Value: price, Label: 'Price' },
      { Value: stock, Label: 'Current Stock' },
      { Value: minStock, Label: 'Minimum Stock' },
      { Value: rating, Label: 'Rating' }
    ]
  }
};
```

**Result — Two form sections:**
```
§ General Information
┌─────────────────────────────────────┐
│ Product Name:  Laptop Pro           │
│ Description:   High-performance...  │
│ Category:      Electronics          │
│ SKU:           PRD-00001            │
│ Created On:    June 1, 2026         │
│ Last Modified: June 4, 2026         │
└─────────────────────────────────────┘

§ Pricing & Stock
┌─────────────────────────────────────┐
│ Price:         $999.00              │
│ Current Stock: 45                   │
│ Minimum Stock: 10                   │
│ Rating:        4.8                  │
└─────────────────────────────────────┘
```

---

### Showing Related Entities (Tables in Object Page)

To show a list of related records (e.g., Order Items, Reviews) as a table within the Object Page:

```cds
// Annotate the CHILD entity's LineItem:
annotate CatalogService.Reviews with @UI: {
  LineItem: [
    { Value: reviewer, Label: 'Reviewer' },
    { Value: rating, Label: 'Rating' },
    { Value: comment, Label: 'Comment' },
    { Value: reviewDate, Label: 'Date' }
  ]
};

// Then reference it in the parent's Facets:
annotate CatalogService.Products with @UI: {
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#General', Label: 'General' },
    { $Type: 'UI.ReferenceFacet', Target: 'reviews/@UI.LineItem', Label: 'Reviews' }
    //                                     ↑ navigation property path!
  ]
};
```

**The key:** `Target: 'reviews/@UI.LineItem'` means "show the LineItem annotation of the `reviews` navigation property (association) as a table in this section."

---

### The Complete Object Page Pattern

Here's the full annotation for a typical Object Page:

```cds
annotate CatalogService.Products with @UI: {
  // ─── HEADER ───────────────────────
  HeaderInfo: {
    TypeName: 'Product',
    TypeNamePlural: 'Products',
    Title: { Value: productName },
    Description: { Value: description }
  },

  // ─── HEADER KPIs ──────────────────
  HeaderFacets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Price' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Stock' }
  ],
  DataPoint#Price: { Value: price, Title: 'Price' },
  DataPoint#Stock: { Value: stock, Title: 'Stock', Criticality: stockCriticality },

  // ─── PAGE SECTIONS ────────────────
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#General', Label: 'General' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Details', Label: 'Details' },
    { $Type: 'UI.ReferenceFacet', Target: 'reviews/@UI.LineItem', Label: 'Reviews' }
  ],

  // ─── SECTION CONTENT ──────────────
  FieldGroup#General: {
    Data: [
      { Value: productName },
      { Value: description },
      { Value: category.categoryName, Label: 'Category' },
      { Value: supplier.supplierName, Label: 'Supplier' }
    ]
  },
  FieldGroup#Details: {
    Data: [
      { Value: price },
      { Value: stock },
      { Value: minStock },
      { Value: rating },
      { Value: sku },
      { Value: createdAt }
    ]
  }
};
```

---

## Session 3: Hands-on — Generate a Fiori Elements App (12:00 - 13:00)

### Step 1: Ensure Your CAP Project is Ready

Your project should have:
```
my-cap-project/
├── db/
│   └── schema.cds        (with Products, Suppliers, etc.)
├── srv/
│   ├── catalog-service.cds
│   └── catalog-service.js
├── app/                   ← We'll add stuff here!
└── package.json
```

Run `cds watch` to verify everything is working:
```bash
cds watch
```

---

### Step 2: Generate the Fiori Elements App

**Option A: Using SAP Fiori Tools Generator (in BAS/VS Code)**

1. Open the Command Palette: `Ctrl+Shift+P` (or `Cmd+Shift+P` on Mac)
2. Search for: "Fiori: Open Application Generator"
3. Select Template: **List Report Page**
4. Data Source: **Use a Local CAP Project**
5. Choose your CAP project folder
6. Select OData Service: **CatalogService**
7. Main Entity: **Products**
8. Navigation Entity: (leave blank or select Reviews if available)
9. Module Name: `products` (this creates `app/products/`)
10. Application Title: "Manage Products"
11. Add FLP config: Yes

**Option B: Manual Setup (if generator not available)**

Create these files manually:

**File: `app/products/webapp/manifest.json`**
```json
{
  "_version": "1.49.0",
  "sap.app": {
    "id": "products",
    "type": "application",
    "title": "Manage Products",
    "dataSources": {
      "mainService": {
        "uri": "/catalog/",
        "type": "OData",
        "settings": { "odataVersion": "4.0" }
      }
    }
  },
  "sap.ui5": {
    "models": {
      "": {
        "dataSource": "mainService",
        "settings": {
          "synchronizationMode": "None",
          "operationMode": "Server",
          "autoExpandSelect": true
        }
      }
    },
    "routing": {
      "routes": [
        {
          "pattern": ":?query:",
          "name": "ProductsList",
          "target": "ProductsList"
        },
        {
          "pattern": "Products({key}):?query:",
          "name": "ProductsObjectPage",
          "target": "ProductsObjectPage"
        }
      ],
      "targets": {
        "ProductsList": {
          "type": "Component",
          "id": "ProductsList",
          "name": "sap.fe.templates.ListReport",
          "options": {
            "settings": {
              "entitySet": "Products"
            }
          }
        },
        "ProductsObjectPage": {
          "type": "Component",
          "id": "ProductsObjectPage",
          "name": "sap.fe.templates.ObjectPage",
          "options": {
            "settings": {
              "entitySet": "Products"
            }
          }
        }
      }
    }
  }
}
```

**File: `app/products/webapp/Component.js`**
```javascript
sap.ui.define(["sap/fe/core/AppComponent"], function(AppComponent) {
  return AppComponent.extend("products.Component", {
    metadata: { manifest: "json" }
  });
});
```

**File: `app/products/webapp/index.html`**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Manage Products</title>
  <script id="sap-ui-bootstrap"
    src="https://ui5.sap.com/resources/sap-ui-core.js"
    data-sap-ui-theme="sap_horizon"
    data-sap-ui-resourceroots='{ "products": "./" }'
    data-sap-ui-compatVersion="edge"
    data-sap-ui-async="true"
    data-sap-ui-frameOptions="allow">
  </script>
  <script>
    sap.ui.require(["sap/fe/core/ComponentContainer"], function(ComponentContainer) {
      new ComponentContainer({
        name: "products",
        settings: { id: "products" },
        async: true
      }).placeAt("content");
    });
  </script>
</head>
<body class="sapUiBody" id="content"></body>
</html>
```

---

### Step 3: Add the App to `package.json`

Add this to your project root `package.json` (the `sapux` section tells CAP about your Fiori app):

```json
{
  "sapux": ["app/products"]
}
```

Or add an `xs-app.json` in the app folder or let CAP auto-serve static files.

---

### Step 4: Restart and Test

```bash
cds watch
```

Open the browser:
- `http://localhost:4004` → Service index should now show a link to your Fiori app
- `http://localhost:4004/products/webapp/index.html` → Your Fiori Elements app!

**What you should see:** A basic List Report with all Product fields (unsorted, no filter bar yet). It works, but it's not pretty. We need annotations to make it beautiful!

---

## Session 4: UI Annotations — The Complete Syntax (13:45 - 14:45)

### Create the Annotations File

**File: `srv/annotations.cds`** (or `app/products/annotations.cds`)

```cds
using { CatalogService } from './catalog-service';

// ═══════════════════════════════════════════════════
// PRODUCTS — Full Annotation Set
// ═══════════════════════════════════════════════════

annotate CatalogService.Products with @UI: {

  // ─── LIST REPORT: Filter Bar ───────────────────
  SelectionFields: [
    productName,
    price,
    stock,
    category_ID,
    supplier_ID,
    isAvailable
  ],

  // ─── LIST REPORT: Table Columns ────────────────
  LineItem: [
    { Value: productName, Label: 'Product', ![@UI.Importance]: #High },
    { Value: price, Label: 'Price' },
    { Value: stock, Label: 'Stock', Criticality: stockCriticality },
    { Value: rating, Label: 'Rating' },
    { Value: supplier.supplierName, Label: 'Supplier' },
    { Value: category.categoryName, Label: 'Category', ![@UI.Importance]: #Low },
    { Value: isAvailable, Label: 'Available' }
  ],

  // ─── OBJECT PAGE: Header ───────────────────────
  HeaderInfo: {
    TypeName: 'Product',
    TypeNamePlural: 'Products',
    Title: { Value: productName },
    Description: { Value: description }
  },

  // ─── OBJECT PAGE: Header KPIs ─────────────────
  HeaderFacets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Price' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Stock' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Rating' }
  ],
  DataPoint#Price: {
    Value: price,
    Title: 'Price'
  },
  DataPoint#Stock: {
    Value: stock,
    Title: 'In Stock',
    Criticality: stockCriticality
  },
  DataPoint#Rating: {
    Value: rating,
    Title: 'Rating',
    TargetValue: 5,
    Visualization: #Rating
  },

  // ─── OBJECT PAGE: Sections ────────────────────
  Facets: [
    {
      $Type: 'UI.ReferenceFacet',
      Target: '@UI.FieldGroup#GeneralInfo',
      Label: 'General Information'
    },
    {
      $Type: 'UI.ReferenceFacet',
      Target: '@UI.FieldGroup#StockPricing',
      Label: 'Stock & Pricing'
    },
    {
      $Type: 'UI.ReferenceFacet',
      Target: '@UI.FieldGroup#Admin',
      Label: 'Administration'
    }
  ],

  // ─── FIELD GROUPS (Section Content) ────────────
  FieldGroup#GeneralInfo: {
    Data: [
      { Value: productName, Label: 'Product Name' },
      { Value: description, Label: 'Description' },
      { Value: category.categoryName, Label: 'Category' },
      { Value: supplier.supplierName, Label: 'Supplier' }
    ]
  },
  FieldGroup#StockPricing: {
    Data: [
      { Value: price, Label: 'Price' },
      { Value: stock, Label: 'Current Stock' },
      { Value: minStock, Label: 'Minimum Stock Level' },
      { Value: rating, Label: 'Customer Rating' },
      { Value: isAvailable, Label: 'Available for Sale' }
    ]
  },
  FieldGroup#Admin: {
    Data: [
      { Value: createdAt, Label: 'Created On' },
      { Value: createdBy, Label: 'Created By' },
      { Value: modifiedAt, Label: 'Last Modified' },
      { Value: modifiedBy, Label: 'Modified By' }
    ]
  }
};
```

---

### Annotation for Individual Fields (Labels & Visibility)

You can also annotate fields directly:

```cds
annotate CatalogService.Products with {
  productName @title: 'Product Name';
  price       @title: 'Unit Price' @Measures.ISOCurrency: currency_code;
  stock       @title: 'Stock Quantity';
  rating      @title: 'Avg. Rating';
  description @UI.MultiLineText: true;       // Renders as textarea
  isAvailable @title: 'Available';
};
```

**`@title`** sets the default label everywhere (table headers, form labels, filter labels).

**`@Measures.ISOCurrency`** tells Fiori to show the currency symbol next to the price.

**`@UI.MultiLineText`** renders the field as a multiline text area instead of a single-line input.

---

### Value Help — Dropdown for Associations

When a field is an association (like `category` or `supplier`), Fiori Elements automatically provides a Value Help dialog. But you can enhance it:

```cds
annotate CatalogService.Products with {
  category @(
    Common.ValueList: {
      Label: 'Categories',
      CollectionPath: 'Categories',
      Parameters: [
        { $Type: 'Common.ValueListParameterInOut', LocalDataProperty: category_ID, ValueListProperty: 'ID' },
        { $Type: 'Common.ValueListParameterDisplayOnly', ValueListProperty: 'categoryName' }
      ]
    }
  );
};
```

This creates a dropdown/dialog showing category names when editing the category field.

---

### Making Fields Read-Only or Hidden

```cds
annotate CatalogService.Products with {
  // Read-only fields (can't edit)
  createdAt  @UI.ReadOnly;
  createdBy  @UI.ReadOnly;
  ID         @UI.Hidden;          // Don't show UUID to users

  // Required fields (shows * in forms)
  productName @Common.FieldControl: #Mandatory;
  price       @Common.FieldControl: #Mandatory;
};
```

---

## Session 5: Hands-on — Annotate Your EPM Model (15:00 - 16:00)

### Exercise: Annotate Products for List Report

Create or update `srv/annotations.cds` with annotations for your EPM Products entity.

**Requirements:**

1. **Filter Bar** should have: productName, category, supplier, price range, isAvailable
2. **Table** should show: Product Name, Price (with currency), Stock (with color), Rating, Supplier, Category
3. **Header** should show: Product name as title, description as subtitle
4. **Header KPIs**: Price, Stock (with criticality colors), Rating (as stars)
5. **Sections**: "General Info" (name, description, category, supplier), "Stock & Pricing" (price, stock, minStock, rating)

**Step-by-step:**

1. Create the file `srv/annotations.cds`
2. Import your service: `using { CatalogService } from './catalog-service';`
3. Add `SelectionFields`
4. Add `LineItem`
5. Add `HeaderInfo`
6. Add `HeaderFacets` with `DataPoints`
7. Add `Facets` pointing to `FieldGroups`
8. Define each `FieldGroup`

After writing annotations, restart `cds watch` and refresh your Fiori app!

---

### Expected Result After Annotations

**List Report should look like:**
```
Products (47)
┌──────────────────────────────────────────────────────────────┐
│ Product Name: [______] Category: [▼] Supplier: [▼]  [Go]    │
├──────────────────────────────────────────────────────────────┤
│ Product       │ Price   │ Stock │ Rating │ Supplier │ Cat.   │
│ Laptop Pro    │ $999.00 │ 🟢 45 │ ⭐4.8  │ TechCorp │ Elec. │
│ Mouse         │ $29.99  │ 🟢120 │ ⭐4.5  │ TechCorp │ Acc.  │
│ Old Keyboard  │ $19.99  │ 🔴 0  │ ⭐3.2  │ PartsCo  │ Acc.  │
└──────────────────────────────────────────────────────────────┘
```

**Object Page (click "Laptop Pro") should look like:**
```
◀ Back    Laptop Pro                              [Edit] [Delete]
          High-performance business laptop

┌────────┐ ┌────────┐ ┌────────┐
│ $999   │ │ 🟢 45  │ │ ⭐⭐⭐⭐⭐│
│ Price  │ │ Stock  │ │ Rating │
└────────┘ └────────┘ └────────┘

§ General Information
  Product Name:  Laptop Pro
  Description:   High-performance business laptop
  Category:      Electronics
  Supplier:      TechCorp Supplies

§ Stock & Pricing
  Price:         $999.00
  Current Stock: 45
  Min Stock:     10
  Rating:        4.8
```

---

## Session 6: Hands-on — Object Page with Sections & Fields (16:00 - 16:45)

### Exercise: Add a Related Table (Reviews/Order Items)

If your Products entity has a `reviews` Composition, add it as a section on the Object Page:

**Step 1: Annotate the child entity's LineItem:**
```cds
annotate CatalogService.Reviews with @UI: {
  LineItem: [
    { Value: reviewer, Label: 'Reviewer' },
    { Value: rating, Label: 'Rating' },
    { Value: comment, Label: 'Comment' },
    { Value: reviewDate, Label: 'Date' }
  ]
};
```

**Step 2: Add a Facet in the parent pointing to the child:**
```cds
annotate CatalogService.Products with @UI: {
  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#General', Label: 'General' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Pricing', Label: 'Pricing' },
    { $Type: 'UI.ReferenceFacet', Target: 'reviews/@UI.LineItem', Label: 'Reviews' }
    //                                     ↑ navigation path to child entity
  ]
};
```

**Step 3: Refresh browser — the Object Page now has a Reviews table section!**

---

### Exercise: Try Creating and Editing

With your Fiori Elements app running:

1. Click **[+ Create]** on the List Report → an empty Object Page (edit mode) opens
2. Fill in fields → click **Save** → record is created!
3. Click a row → see the Object Page in display mode
4. Click **Edit** → fields become editable
5. Change a value → click **Save**
6. Click **Delete** → confirmation dialog → record removed

All of this works WITHOUT writing any JavaScript! Fiori Elements + your service annotations handle everything.

---

### Troubleshooting Common Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| App shows empty table | No data in database | Add CSV data or create records via REST |
| All columns show (no filtering) | Missing `@UI.LineItem` | Add LineItem annotation |
| No filter bar | Missing `@UI.SelectionFields` | Add SelectionFields annotation |
| Object Page is blank | Missing `@UI.Facets` | Add Facets + FieldGroups |
| "Cannot read property" error | Wrong annotation path | Check entity name, field names match CDS exactly |
| App doesn't load | Wrong `manifest.json` config | Verify entitySet name matches service entity |
| Navigation doesn't work | Missing route in manifest | Check routing config in manifest.json |

---

## Assessment: MCQ — 15 Questions on Fiori Elements Annotations

**Q1.** `@UI.SelectionFields` controls which part of the List Report?

- A) The table columns
- B) The filter bar (which fields appear as filters)
- C) The page title
- D) The navigation

**Answer: B** — SelectionFields defines which fields appear in the filter bar at the top of the List Report.

---

**Q2.** `@UI.LineItem` controls which part of the List Report?

- A) The filter bar
- B) The table columns (which fields appear as columns and their order)
- C) The Object Page header
- D) The page footer

**Answer: B** — LineItem defines the table columns: which fields to show, their labels, and their order.

---

**Q3.** `@UI.HeaderInfo.Title` is used for:

- A) The List Report page title
- B) The main title shown on the Object Page header (the record's "name")
- C) The browser tab title
- D) The filter bar title

**Answer: B** — HeaderInfo.Title appears as the main heading on the Object Page (e.g., "Laptop Pro").

---

**Q4.** To show related OrderItems as a table on the Order's Object Page, you use:

- A) `@UI.LineItem` on the Order entity
- B) `Target: 'items/@UI.LineItem'` in the Order's Facets
- C) `@UI.Table` on the OrderItems entity
- D) `@UI.ChildTable` pointing to items

**Answer: B** — Reference the child's LineItem via the navigation path: `'items/@UI.LineItem'` (where `items` is the Composition/Association name).

---

**Q5.** `@UI.FieldGroup#General` is used to:

- A) Filter records by general category
- B) Group related fields into a section on the Object Page
- C) Create a database index
- D) Define a new entity

**Answer: B** — FieldGroups organize fields into labeled sections (forms) on the Object Page.

---

**Q6.** What does `Criticality: stockCriticality` do in a LineItem?

- A) Sorts by stock level
- B) Shows a colored status indicator (red/orange/green) based on the criticality value
- C) Makes the column mandatory
- D) Hides the column on mobile

**Answer: B** — Criticality adds color coding: 1=red (negative), 2=orange (warning), 3=green (positive).

---

**Q7.** Where do annotations typically go in a CAP project?

- A) `db/schema.cds`
- B) `package.json`
- C) A separate CDS file like `srv/annotations.cds` or `app/annotations.cds`
- D) Inside `manifest.json`

**Answer: C** — Annotations go in a separate CDS file to keep the data model clean. Common locations: `srv/annotations.cds` or `app/<appname>/annotations.cds`.

---

**Q8.** `@UI.DataPoint#Stock: { Value: stock, Title: 'Stock' }` appears where?

- A) In the table as a column
- B) In the Object Page header as a KPI box (referenced by HeaderFacets)
- C) In the filter bar
- D) In the page footer

**Answer: B** — DataPoints are KPI values shown in the Object Page header. They must be referenced by `@UI.HeaderFacets` to appear.

---

**Q9.** What does `![@UI.Importance]: #High` do on a LineItem column?

- A) Sorts that column first
- B) Ensures the column is always visible even on small screens (phone)
- C) Makes the field required
- D) Highlights the column with bold text

**Answer: B** — Importance controls responsive behavior. `#High` = always shown. `#Low` = hidden on small screens.

---

**Q10.** To make a field render as a multiline text area in edit mode:

- A) `@UI.MultiLineText: true`
- B) `@UI.TextArea: true`
- C) Use `LargeString` type (automatic)
- D) Both A and C achieve the same result

**Answer: D** — `@UI.MultiLineText: true` explicitly makes a field render as textarea. Fields typed as `LargeString` may also get this treatment automatically.

---

**Q11.** `@Common.FieldControl: #Mandatory` does what?

- A) Deletes the field if empty
- B) Shows a red asterisk (*) next to the field and requires input before saving
- C) Hides the field
- D) Makes the field read-only

**Answer: B** — `#Mandatory` marks the field as required in the UI (shows * indicator). Form won't submit without it.

---

**Q12.** `@UI.Hidden` on a field means:

- A) The field is encrypted
- B) The field is not shown anywhere in the UI (but still exists in the data)
- C) The field is shown but greyed out
- D) The field is deleted from the entity

**Answer: B** — Hidden fields exist in the data model and API but are not rendered in the UI. Useful for IDs, internal flags.

---

**Q13.** The correct way to show currency with price is:

- A) `@UI.Currency: 'USD'`
- B) `@Measures.ISOCurrency: currency_code`
- C) `{ Value: price, Currency: 'USD' }` in LineItem
- D) Just show the currency field as a separate column

**Answer: B** — `@Measures.ISOCurrency: currency_code` tells Fiori to display the associated currency symbol alongside the price value.

---

**Q14.** What is the minimum annotation needed to get a working (but basic) List Report?

- A) Just `@UI.LineItem` — the table columns
- B) All annotations: HeaderInfo, Facets, FieldGroups, LineItem, SelectionFields
- C) No annotations needed — Fiori Elements shows all fields by default
- D) Only `@UI.SelectionFields`

**Answer: C** — Fiori Elements can render a basic UI with NO annotations (shows all fields). But it's ugly. Annotations make it polished and usable.

---

**Q15.** `HeaderInfo.TypeNamePlural: 'Products'` is displayed where?

- A) Only on the Object Page
- B) As the table title/page title on the List Report (e.g., "Products (47)")
- C) In the browser URL
- D) In the $metadata document only

**Answer: B** — TypeNamePlural becomes the page/table title on the List Report: "Products (47)" showing the entity name and count.

---

## Annotation Quiz: Write It Yourself

### Scenario

Given this entity:
```cds
entity Customers : cuid, managed {
  customerName : String(100);
  email        : String(255);
  phone        : String(20);
  city         : String(100);
  country      : String(50);
  creditLimit  : Decimal(12,2);
  totalOrders  : Integer default 0;
  isVIP        : Boolean default false;
  orders       : Association to many Orders on orders.customer = $self;
}
```

**Task:** Write complete annotations so that:

1. **List Report** filters by: customerName, city, country, isVIP
2. **Table** shows: customerName, email, city, country, creditLimit, totalOrders, isVIP
3. **Object Page** header: Title = customerName, Description = city + country
4. **Header KPIs**: creditLimit, totalOrders
5. **Sections**: "Contact Details" (email, phone, city, country), "Account Info" (creditLimit, totalOrders, isVIP)
6. **Orders section**: Show related orders as a table (orderNumber, orderDate, totalAmount, status)

Write your answer, then check below.

<details>
<summary>Complete Answer</summary>

```cds
annotate SalesService.Customers with @UI: {

  SelectionFields: [ customerName, city, country, isVIP ],

  LineItem: [
    { Value: customerName, Label: 'Customer' },
    { Value: email, Label: 'Email' },
    { Value: city, Label: 'City' },
    { Value: country, Label: 'Country' },
    { Value: creditLimit, Label: 'Credit Limit' },
    { Value: totalOrders, Label: 'Orders' },
    { Value: isVIP, Label: 'VIP' }
  ],

  HeaderInfo: {
    TypeName: 'Customer',
    TypeNamePlural: 'Customers',
    Title: { Value: customerName },
    Description: { Value: city }
  },

  HeaderFacets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Credit' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.DataPoint#Orders' }
  ],
  DataPoint#Credit: { Value: creditLimit, Title: 'Credit Limit' },
  DataPoint#Orders: { Value: totalOrders, Title: 'Total Orders' },

  Facets: [
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Contact', Label: 'Contact Details' },
    { $Type: 'UI.ReferenceFacet', Target: '@UI.FieldGroup#Account', Label: 'Account Info' },
    { $Type: 'UI.ReferenceFacet', Target: 'orders/@UI.LineItem', Label: 'Orders' }
  ],

  FieldGroup#Contact: {
    Data: [
      { Value: email, Label: 'Email' },
      { Value: phone, Label: 'Phone' },
      { Value: city, Label: 'City' },
      { Value: country, Label: 'Country' }
    ]
  },
  FieldGroup#Account: {
    Data: [
      { Value: creditLimit, Label: 'Credit Limit' },
      { Value: totalOrders, Label: 'Total Orders' },
      { Value: isVIP, Label: 'VIP Customer' }
    ]
  }
};

// Annotate the related Orders entity for the table section:
annotate SalesService.Orders with @UI: {
  LineItem: [
    { Value: orderNumber, Label: 'Order #' },
    { Value: orderDate, Label: 'Date' },
    { Value: totalAmount, Label: 'Amount' },
    { Value: status, Label: 'Status' }
  ]
};
```

</details>

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| List Report anatomy | Filter bar + table, driven by SelectionFields + LineItem |
| Object Page anatomy | Header + KPIs + sections, driven by HeaderInfo + Facets + FieldGroups |
| `@UI.SelectionFields` | Which fields appear in the filter bar |
| `@UI.LineItem` | Which columns appear in the table |
| `@UI.HeaderInfo` | Title, subtitle, type names for the Object Page |
| `@UI.HeaderFacets` + `DataPoint` | KPI boxes in the Object Page header |
| `@UI.Facets` | Sections on the Object Page (references FieldGroups or child LineItems) |
| `@UI.FieldGroup` | Form fields within a section |
| Child table | `Target: 'navProperty/@UI.LineItem'` in Facets |
| Criticality | Color-coded status (1=red, 2=orange, 3=green) |
| Fiori Tools | Generator creates the app skeleton (manifest.json, Component.js) |

### The Annotation Flow

```
@UI.SelectionFields  →  Filter Bar (List Report)
@UI.LineItem         →  Table Columns (List Report)
@UI.HeaderInfo       →  Page Title (Object Page)
@UI.HeaderFacets     →  KPI Boxes (Object Page header)
@UI.DataPoint        →  KPI Values (referenced by HeaderFacets)
@UI.Facets           →  Page Sections (Object Page body)
@UI.FieldGroup       →  Form Fields (inside a section)
child/@UI.LineItem   →  Related Table (inside a section)
```

### The One-Liner

> **Annotations = a recipe card. You list the ingredients (fields) and Fiori Elements cooks the entire meal (UI) for you.**

---

### Looking Ahead: Day 23

Tomorrow we'll add more advanced annotations:
- Value Helps for dropdowns
- Actions on the UI (approve/reject buttons)
- Navigation between entities
- Semantic coloring and status indicators
- Creating/editing records via the UI

---

### Homework

1. Add annotations for your EPM **Suppliers** entity (List Report + Object Page)
2. Add annotations for **SalesOrders** including an items table section
3. Verify all annotations by running `cds watch` and refreshing the Fiori app
4. Try creating a new Product through the Fiori UI — does validation still work?
5. Challenge: Add a `CollectionFacet` that groups two FieldGroups under one tab

---

*End of Day 22 — You can now build complete List Report + Object Page UIs with annotations!*
