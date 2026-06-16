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
