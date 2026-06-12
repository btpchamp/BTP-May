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
