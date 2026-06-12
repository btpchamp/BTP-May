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
