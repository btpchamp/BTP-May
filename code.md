## Session 5: Hands-on — Complete Business Workflow (15:00 - 16:00)

### Exercise: Purchase Order Approval Workflow

Build a complete purchase order workflow with actions, functions, and events:

```
PO Lifecycle:
  [Draft] ──submit──► [Submitted] ──approve──► [Approved] ──receive──► [Received]
     │                     │
     │                     └──reject──► [Rejected]
     │
     └──(can edit freely while in Draft)
```

---

### Step 1: Update CDS Service Definition

**File: `srv/purchasing-service.cds`**

```cds
using { com.epm as db } from '../db/schema';

service PurchasingService @(path: '/purchasing') {

  entity PurchaseOrders as projection on db.PurchaseOrders
    actions {
      // Workflow actions
      action submit() returns { status: String; message: String; };
      action approve(comment: String(500)) returns { status: String; message: String; approvedAt: DateTime; };
      action reject(reason: String(500)) returns { status: String; message: String; };
      action receive(receivedQty: Integer; notes: String(500)) returns { status: String; message: String; };

      // Read-only functions
      function getSummary() returns {
        poNumber: String;
        supplier: String;
        itemCount: Integer;
        totalAmount: Decimal;
        status: String;
        daysOpen: Integer;
      };
    };

  entity PurchaseOrderItems as projection on db.PurchaseOrderItems;
  @readonly entity Suppliers as projection on db.Suppliers;
  @readonly entity Products as projection on db.Products;

  // Unbound function: Dashboard stats
  function getPurchasingDashboard() returns {
    totalPOs: Integer;
    draftCount: Integer;
    pendingApproval: Integer;
    approvedCount: Integer;
    totalSpend: Decimal;
  };

  // Events
  event POSubmitted {
    poId: UUID;
    poNumber: String;
    supplierName: String;
    totalAmount: Decimal;
    submittedBy: String;
  }

  event POApproved {
    poId: UUID;
    poNumber: String;
    approvedBy: String;
    comment: String;
  }

  event POrejected {
    poId: UUID;
    poNumber: String;
    rejectedBy: String;
    reason: String;
  }
}
```

---

### Step 2: Implement the Handlers

**File: `srv/purchasing-service.js`**

```javascript
const cds = require('@sap/cds');

module.exports = function () {

  // ═══════════════════════════════════════════════
  //  SUBMIT Purchase Order
  // ═══════════════════════════════════════════════
  this.on('submit', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const { PurchaseOrders, PurchaseOrderItems, Suppliers } = cds.entities;

    // Fetch the PO
    const po = await SELECT.one.from(PurchaseOrders).where({ ID });
    if (!po) req.reject(404, 'Purchase Order not found');

    // Rule: Only Draft POs can be submitted
    if (po.status !== 'Draft') {
      req.reject(400,
        `Cannot submit: PO is in "${po.status}" status. Only Draft POs can be submitted.`
      );
    }

    // Rule: PO must have at least one item
    const items = await SELECT.from(PurchaseOrderItems).where({ purchaseOrder_ID: ID });
    if (items.length === 0) {
      req.reject(400, 'Cannot submit: PO has no items. Add at least one item first.');
    }

    // Rule: Total amount must be calculated
    const total = items.reduce((sum, item) => sum + (item.quantity * item.unitPrice), 0);

    // Update status
    await UPDATE(PurchaseOrders).set({
      status: 'Submitted',
      totalAmount: +total.toFixed(2)
    }).where({ ID });

    // Get supplier name for event
    const supplier = await SELECT.one.from(Suppliers).where({ ID: po.supplier_ID });

    // Emit event
    await this.emit('POSubmitted', {
      poId: ID,
      poNumber: po.poNumber,
      supplierName: supplier?.supplierName || 'Unknown',
      totalAmount: +total.toFixed(2),
      submittedBy: req.user.id
    });

    return {
      status: 'Submitted',
      message: `PO ${po.poNumber} submitted for approval. Total: $${total.toFixed(2)} (${items.length} items)`
    };
  });

  // ═══════════════════════════════════════════════
  //  APPROVE Purchase Order
  // ═══════════════════════════════════════════════
  this.on('approve', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const { comment } = req.data;
    const { PurchaseOrders } = cds.entities;

    const po = await SELECT.one.from(PurchaseOrders).where({ ID });
    if (!po) req.reject(404, 'Purchase Order not found');

    if (po.status !== 'Submitted') {
      req.reject(400,
        `Cannot approve: PO is in "${po.status}" status. Only Submitted POs can be approved.`
      );
    }

    await UPDATE(PurchaseOrders).set({
      status: 'Approved'
    }).where({ ID });

    // Emit event
    await this.emit('POApproved', {
      poId: ID,
      poNumber: po.poNumber,
      approvedBy: req.user.id,
      comment: comment || ''
    });

    return {
      status: 'Approved',
      message: `PO ${po.poNumber} has been approved.${comment ? ' Comment: ' + comment : ''}`,
      approvedAt: new Date().toISOString()
    };
  });

  // ═══════════════════════════════════════════════
  //  REJECT Purchase Order
  // ═══════════════════════════════════════════════
  this.on('reject', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const { reason } = req.data;
    const { PurchaseOrders } = cds.entities;

    const po = await SELECT.one.from(PurchaseOrders).where({ ID });
    if (!po) req.reject(404, 'Purchase Order not found');

    if (po.status !== 'Submitted') {
      req.reject(400, `Cannot reject: PO is in "${po.status}" status. Only Submitted POs can be rejected.`);
    }

    if (!reason || reason.trim() === '') {
      req.reject(400, 'Rejection reason is required. Please explain why this PO is being rejected.');
    }

    await UPDATE(PurchaseOrders).set({
      status: 'Rejected'
    }).where({ ID });

    // Emit event
    await this.emit('POrejected', {
      poId: ID,
      poNumber: po.poNumber,
      rejectedBy: req.user.id,
      reason: reason
    });

    return {
      status: 'Rejected',
      message: `PO ${po.poNumber} rejected. Reason: ${reason}`
    };
  });

  // ═══════════════════════════════════════════════
  //  RECEIVE Purchase Order (goods arrived)
  // ═══════════════════════════════════════════════
  this.on('receive', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const { notes } = req.data;
    const { PurchaseOrders, PurchaseOrderItems, Products } = cds.entities;

    const po = await SELECT.one.from(PurchaseOrders).where({ ID });
    if (!po) req.reject(404, 'Purchase Order not found');

    if (po.status !== 'Approved') {
      req.reject(400, `Cannot receive: PO must be "Approved". Current status: "${po.status}"`);
    }

    // Update PO status
    await UPDATE(PurchaseOrders).set({
      status: 'Received'
    }).where({ ID });

    // Increase stock for each item
    const items = await SELECT.from(PurchaseOrderItems).where({ purchaseOrder_ID: ID });

    for (const item of items) {
      const product = await SELECT.one.from(Products).where({ ID: item.product_ID });
      if (product) {
        await UPDATE(Products).set({
          stock: product.stock + item.quantity
        }).where({ ID: item.product_ID });
      }
    }

    return {
      status: 'Received',
      message: `PO ${po.poNumber} received. Stock updated for ${items.length} products.${notes ? ' Notes: ' + notes : ''}`
    };
  });

  // ═══════════════════════════════════════════════
  //  BOUND FUNCTION: getSummary
  // ═══════════════════════════════════════════════
  this.on('getSummary', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const { PurchaseOrders, PurchaseOrderItems, Suppliers } = cds.entities;

    const po = await SELECT.one.from(PurchaseOrders).where({ ID });
    if (!po) req.reject(404, 'Purchase Order not found');

    const items = await SELECT.from(PurchaseOrderItems).where({ purchaseOrder_ID: ID });
    const supplier = await SELECT.one.from(Suppliers).where({ ID: po.supplier_ID });

    // Calculate days open
    const createdDate = new Date(po.createdAt || po.orderDate);
    const today = new Date();
    const daysOpen = Math.floor((today - createdDate) / (1000 * 60 * 60 * 24));

    const totalAmount = items.reduce((sum, item) => sum + (item.quantity * item.unitPrice), 0);

    return {
      poNumber: po.poNumber,
      supplier: supplier?.supplierName || 'Unknown',
      itemCount: items.length,
      totalAmount: +totalAmount.toFixed(2),
      status: po.status,
      daysOpen: daysOpen
    };
  });

  // ═══════════════════════════════════════════════
  //  UNBOUND FUNCTION: getPurchasingDashboard
  // ═══════════════════════════════════════════════
  this.on('getPurchasingDashboard', async (req) => {
    const { PurchaseOrders } = cds.entities;

    const allPOs = await SELECT.from(PurchaseOrders);

    return {
      totalPOs: allPOs.length,
      draftCount: allPOs.filter(p => p.status === 'Draft').length,
      pendingApproval: allPOs.filter(p => p.status === 'Submitted').length,
      approvedCount: allPOs.filter(p => p.status === 'Approved').length,
      totalSpend: +allPOs
        .filter(p => ['Approved', 'Received'].includes(p.status))
        .reduce((sum, p) => sum + (p.totalAmount || 0), 0)
        .toFixed(2)
    };
  });

  // ═══════════════════════════════════════════════
  //  EVENT LISTENERS
  // ═══════════════════════════════════════════════

  this.on('POSubmitted', (msg) => {
    const { poNumber, supplierName, totalAmount, submittedBy } = msg.data;
    console.log(`\n📋 [PO SUBMITTED] ${poNumber}`);
    console.log(`   Supplier: ${supplierName}`);
    console.log(`   Amount: $${totalAmount}`);
    console.log(`   By: ${submittedBy}`);
    console.log(`   → Waiting for approval...\n`);
  });

  this.on('POApproved', (msg) => {
    const { poNumber, approvedBy, comment } = msg.data;
    console.log(`\n✅ [PO APPROVED] ${poNumber}`);
    console.log(`   Approved by: ${approvedBy}`);
    if (comment) console.log(`   Comment: ${comment}`);
    console.log(`   → Ready for goods receipt\n`);
  });

  this.on('POrejected', (msg) => {
    const { poNumber, rejectedBy, reason } = msg.data;
    console.log(`\n❌ [PO REJECTED] ${poNumber}`);
    console.log(`   Rejected by: ${rejectedBy}`);
    console.log(`   Reason: ${reason}`);
    console.log(`   → Returned to requester\n`);
  });

};
```

---

## Session 6: Testing Actions, Functions & Events (16:00 - 16:45)

### Complete Test File

**File: `tests/purchasing-workflow.http`**

```http
@base = http://localhost:4004/purchasing

### ═══════════════════════════════════════════════
### STEP 1: View Dashboard (unbound function)
### ═══════════════════════════════════════════════
GET {{base}}/getPurchasingDashboard()


### ═══════════════════════════════════════════════
### STEP 2: Create a Purchase Order (standard CRUD)
### ═══════════════════════════════════════════════
POST {{base}}/PurchaseOrders
Content-Type: application/json

{
  "poNumber": "PO-2026-001",
  "supplier_ID": "PASTE-SUPPLIER-ID",
  "orderDate": "2026-06-02",
  "expectedDate": "2026-06-15",
  "status": "Draft",
  "currency_code": "USD",
  "notes": "Monthly restock order"
}


### ═══════════════════════════════════════════════
### STEP 3: Add items to PO (standard CRUD)
### ═══════════════════════════════════════════════
POST {{base}}/PurchaseOrderItems
Content-Type: application/json

{
  "purchaseOrder_ID": "PASTE-PO-ID",
  "product_ID": "PASTE-PRODUCT-ID",
  "quantity": 100,
  "unitPrice": 45.00,
  "currency_code": "USD"
}


### ═══════════════════════════════════════════════
### STEP 4: Get PO Summary (bound function)
### ═══════════════════════════════════════════════
GET {{base}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.getSummary()


### ═══════════════════════════════════════════════
### STEP 5: Try to submit without items (should fail if no items)
### ═══════════════════════════════════════════════
POST {{base}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.submit
Content-Type: application/json

{}


### ═══════════════════════════════════════════════
### STEP 6: Submit PO for approval (bound action)
### ═══════════════════════════════════════════════
POST {{base}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.submit
Content-Type: application/json

{}


### ═══════════════════════════════════════════════
### STEP 7: Try to submit again (should fail — not Draft)
### ═══════════════════════════════════════════════
POST {{base}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.submit
Content-Type: application/json

{}


### ═══════════════════════════════════════════════
### STEP 8: Approve the PO (bound action)
### ═══════════════════════════════════════════════
POST {{base}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.approve
Content-Type: application/json

{
  "comment": "Approved. Budget available for Q2 restock."
}


### ═══════════════════════════════════════════════
### STEP 9: Receive goods (bound action — also updates stock!)
### ═══════════════════════════════════════════════
POST {{base}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.receive
Content-Type: application/json

{
  "notes": "All items received in good condition"
}


### ═══════════════════════════════════════════════
### STEP 10: Verify stock was updated
### ═══════════════════════════════════════════════
GET {{base}}/Products(PASTE-PRODUCT-ID)?$select=productName,stock


### ═══════════════════════════════════════════════
### BONUS: Test rejection workflow
### ═══════════════════════════════════════════════

### Create another PO and submit it
POST {{base}}/PurchaseOrders
Content-Type: application/json

{
  "poNumber": "PO-2026-002",
  "supplier_ID": "PASTE-SUPPLIER-ID",
  "orderDate": "2026-06-02",
  "status": "Draft",
  "currency_code": "USD"
}

### (add items, then submit...)

### Reject without reason (should fail)
POST {{base}}/PurchaseOrders(PO-2-ID)/PurchasingService.reject
Content-Type: application/json

{}

### Reject with reason
POST {{base}}/PurchaseOrders(PO-2-ID)/PurchasingService.reject
Content-Type: application/json

{
  "reason": "Budget exceeded for this quarter. Please reduce quantities and resubmit."
}


### ═══════════════════════════════════════════════
### Check dashboard after all operations
### ═══════════════════════════════════════════════
GET {{base}}/getPurchasingDashboard()
```

---

### Expected Console Output (Watch the Terminal!)

When you run through the workflow, your terminal (where `cds watch` is running) should show:

```
📋 [PO SUBMITTED] PO-2026-001
   Supplier: Tech Supplies Inc.
   Amount: $4500.00
   By: anonymous
   → Waiting for approval...

✅ [PO APPROVED] PO-2026-001
   Approved by: anonymous
   Comment: Approved. Budget available for Q2 restock.
   → Ready for goods receipt

❌ [PO REJECTED] PO-2026-002
   Rejected by: anonymous
   Reason: Budget exceeded for this quarter.
   → Returned to requester
```

These messages come from the event listeners — they fire automatically when events are emitted!
