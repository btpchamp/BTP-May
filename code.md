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
