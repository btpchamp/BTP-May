## What You'll Learn Today

By the end of this session, you will be able to:
- Define and implement unbound actions (service-level operations)
- Define and implement bound actions (entity-specific operations)
- Create functions for read-only data retrieval
- Clearly explain the difference between Actions and Functions
- Define input and output parameters for actions/functions
- Understand Events in CAP and how to emit/subscribe to them
- Build real-world business operations (Approve, Reject, Submit)
- Test custom actions and functions via REST Client

---

## Day 18 Recap — Quick Fire (09:00 - 09:15)

1. PUT replaces the _____ record, PATCH updates only _____ fields? → _____
2. Custom handler files must be named the same as _____? → _____
3. `before` handlers are used for _____? → _____
4. `after` handlers receive which parameters? → _____
5. `req.error()` collects errors; `req.reject()` does what? → _____
6. `req.data` in a CREATE handler contains _____? → _____
7. What makes a handler `async`? → _____
8. Deep Insert only works with _____, not Association? → _____

<details>
<summary>Answers</summary>

1. **Entire** record / only **sent** fields
2. The service **CDS file** (e.g., `cat-service.cds` → `cat-service.js`)
3. **Validating input** and rejecting bad data before the database operation
4. `(results, req)` — data first, then request object
5. **Immediately stops** processing (no more handlers run)
6. The **request body** (the JSON payload the client sent)
7. When you need `await` for database lookups: `async (req) => { ... }`
8. **Composition** (parent owns children)

</details>

---

## Session 1: Actions — Beyond CRUD (09:15 - 10:30)

### Why Do We Need Actions?

CRUD gives you Create, Read, Update, Delete. But real business applications have operations that don't fit neatly into CRUD:

```
┌─────────────────────────────────────────────────────────────┐
│  Operations that AREN'T simple CRUD:                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  • Approve a purchase order                                 │
│  • Submit a leave request                                   │
│  • Cancel an order (changes status + sends email + refund)  │
│  • Generate a report (PDF/Excel)                            │
│  • Calculate shipping costs                                 │
│  • Send a reminder notification                             │
│  • Bulk update 100 products at once                         │
│  • Transfer stock between warehouses                        │
│                                                             │
│  These are BUSINESS OPERATIONS, not simple field updates!   │
└─────────────────────────────────────────────────────────────┘
```

**Question:** "Can't I just PATCH the order status to 'Approved'?"

**Answer:** You COULD, but approving an order involves MORE than just changing a status field:
1. Check if the user has permission to approve
2. Verify the order meets approval criteria
3. Update the status
4. Record WHO approved and WHEN
5. Send notification to the requester
6. Trigger the next workflow step (e.g., start shipping)

That's a **business action**, not a simple field update!

---

### What is an Action?

An **Action** is a custom operation that you define in your service. It:
- Has a name (like a function name)
- Can accept input parameters
- Can return output data
- Contains custom logic (implemented in JavaScript)
- Is called via POST request

**Real-world analogy:** 

```
CRUD = The basic controls on a washing machine (on/off, add water, drain)
Actions = The PROGRAMS: "Quick Wash", "Heavy Duty", "Delicate Cycle"

A "Quick Wash" program doesn't just turn the machine on — it:
  - Sets temperature to 30°C
  - Sets spin speed to 800rpm  
  - Adds detergent at the right time
  - Runs for 30 minutes
  - Alerts you when done

That's a business action — a coordinated set of steps with a clear purpose.
```

---

### Two Types of Actions

| Type | Meaning | Example | URL Pattern |
|------|---------|---------|-------------|
| **Unbound Action** | Service-level (not tied to one entity) | GenerateReport, SendBulkEmail | `POST /service/ActionName` |
| **Bound Action** | Entity-level (operates on a specific record) | Approve(this order), Cancel(this PO) | `POST /service/Entity(id)/ActionName` |

---

### Unbound Actions — Service-Level Operations

An unbound action belongs to the SERVICE, not to any specific entity.

**When to use:**
- The operation doesn't target a specific record
- It works across multiple entities
- It's a general utility function

**Examples:**
- `GenerateReport` — creates a report from multiple data sources
- `SendBulkNotifications` — sends emails to many users
- `SyncWithExternalSystem` — pulls data from an external API
- `RunMonthlyCleanup` — deletes old records across tables

---

### Defining an Unbound Action in CDS

```cds
// srv/analytics-service.cds
using { com.epm as db } from '../db/schema';

service AnalyticsService @(path: '/analytics') {

  // Unbound action — belongs to the service, not an entity
  action GenerateReport(
    reportType : String(20);     // Input parameter
    startDate  : Date;           // Input parameter
    endDate    : Date            // Input parameter
  ) returns {                    // Output
    reportId   : UUID;
    status     : String(20);
    message    : String(200);
  };

  // Another unbound action — no parameters
  action PingHealth() returns {
    status    : String(10);
    timestamp : DateTime;
    version   : String(20);
  };

  @readonly entity ProductCatalog as projection on db.ProductCatalog;
}
```

**Syntax breakdown:**

```cds
action <ActionName>(
  <param1> : <Type>;       // Input parameters (optional)
  <param2> : <Type>
) returns {                // Return type (optional)
  <field1> : <Type>;
  <field2> : <Type>;
};
```

---

### Implementing Unbound Action Handlers

```javascript
// srv/analytics-service.js
const cds = require('@sap/cds');

module.exports = function () {

  // Handler for the GenerateReport action
  this.on('GenerateReport', async (req) => {
    const { reportType, startDate, endDate } = req.data;

    // Validate inputs
    if (!reportType) {
      req.reject(400, 'Report type is required');
    }

    const validTypes = ['Sales', 'Inventory', 'Customers'];
    if (!validTypes.includes(reportType)) {
      req.reject(400, `Invalid report type. Must be: ${validTypes.join(', ')}`);
    }

    if (startDate > endDate) {
      req.reject(400, 'Start date must be before end date');
    }

    // Simulate report generation
    const reportId = cds.utils.uuid();

    console.log(`Generating ${reportType} report from ${startDate} to ${endDate}...`);

    // In real app: query data, generate PDF, store somewhere
    return {
      reportId: reportId,
      status: 'Generated',
      message: `${reportType} report generated successfully for ${startDate} to ${endDate}`
    };
  });

  // Handler for PingHealth
  this.on('PingHealth', (req) => {
    return {
      status: 'healthy',
      timestamp: new Date().toISOString(),
      version: '1.0.0'
    };
  });

};
```

---

### Calling an Unbound Action (REST Client)

```http
### Call unbound action: GenerateReport
POST http://localhost:4004/analytics/GenerateReport
Content-Type: application/json

{
  "reportType": "Sales",
  "startDate": "2026-01-01",
  "endDate": "2026-05-31"
}
```

**Response (200 OK):**
```json
{
  "reportId": "a1b2c3d4-e5f6-7890-...",
  "status": "Generated",
  "message": "Sales report generated successfully for 2026-01-01 to 2026-05-31"
}
```

---

### Bound Actions — Entity-Level Operations

A bound action is TIED to a specific entity instance. It says: "Do THIS to THIS particular record."

**When to use:**
- The operation targets ONE specific record
- You need the record's current data to perform the action
- Examples: approve THIS order, cancel THIS booking, ship THIS package

---

### Defining a Bound Action in CDS

```cds
// srv/sales-service.cds
using { com.epm as db } from '../db/schema';

service SalesService @(path: '/sales') {

  entity SalesOrders as projection on db.SalesOrders;
  entity Customers as projection on db.Customers;

  // Bound action — tied to SalesOrders entity
  // "You can call this action ON a specific order"
  action confirmOrder() returns {
    status  : String(20);
    message : String(200);
  };

  action cancelOrder(
    reason : String(500)       // Why are you cancelling?
  ) returns {
    status  : String(20);
    message : String(200);
    refundAmount : Decimal(12,2);
  };

  action shipOrder(
    trackingNumber : String(50);
    carrier        : String(50)
  ) returns {
    status       : String(20);
    message      : String(200);
    estimatedDelivery : Date;
  };

}
```

**Wait — where's the "bound" part?** 

In CDS, you declare bound actions INSIDE the entity definition using `actions` block:

```cds
service SalesService @(path: '/sales') {

  entity SalesOrders as projection on db.SalesOrders
    actions {
      action confirm() returns { status: String; message: String; };
      action cancel(reason: String(500)) returns { status: String; message: String; };
      action ship(trackingNumber: String(50); carrier: String(50)) returns { status: String; };
    };

  entity Customers as projection on db.Customers;
}
```

**This is the correct syntax for bound actions** — they go inside the entity's `actions { }` block.

---

### Implementing Bound Action Handlers

```javascript
// srv/sales-service.js
const cds = require('@sap/cds');

module.exports = function () {

  const { SalesOrders } = cds.entities;

  // ═══════════════════════════════════════════════
  //  BOUND ACTION: confirm (on SalesOrders)
  // ═══════════════════════════════════════════════
  this.on('confirm', 'SalesOrders', async (req) => {
    // Get the specific order this action is called on
    const orderId = req.params[0]?.ID || req.params[0];

    // Read the current order
    const order = await SELECT.one.from(SalesOrders).where({ ID: orderId });

    if (!order) {
      req.reject(404, 'Order not found');
    }

    // Business rule: Can only confirm "New" orders
    if (order.status !== 'New') {
      req.reject(400, `Cannot confirm order in "${order.status}" status. Only "New" orders can be confirmed.`);
    }

    // Update the order status
    await UPDATE(SalesOrders).set({
      status: 'Confirmed',
      modifiedAt: new Date().toISOString()
    }).where({ ID: orderId });

    // Return success response
    return {
      status: 'Confirmed',
      message: `Order ${order.orderNumber} confirmed successfully`
    };
  });

  // ═══════════════════════════════════════════════
  //  BOUND ACTION: cancel (on SalesOrders)
  // ═══════════════════════════════════════════════
  this.on('cancel', 'SalesOrders', async (req) => {
    const orderId = req.params[0]?.ID || req.params[0];
    const { reason } = req.data;

    const order = await SELECT.one.from(SalesOrders).where({ ID: orderId });

    if (!order) {
      req.reject(404, 'Order not found');
    }

    // Business rule: Cannot cancel delivered orders
    if (order.status === 'Delivered') {
      req.reject(400, 'Cannot cancel a delivered order. Please initiate a return instead.');
    }

    if (order.status === 'Cancelled') {
      req.reject(400, 'Order is already cancelled');
    }

    // Cancel reason is required
    if (!reason || reason.trim() === '') {
      req.reject(400, 'Cancellation reason is required');
    }

    // Update order
    await UPDATE(SalesOrders).set({
      status: 'Cancelled',
      modifiedAt: new Date().toISOString()
    }).where({ ID: orderId });

    // Calculate refund (if already paid)
    const refundAmount = (order.status === 'Confirmed' || order.status === 'Shipped')
      ? order.grossAmount
      : 0;

    return {
      status: 'Cancelled',
      message: `Order ${order.orderNumber} cancelled. Reason: ${reason}`,
      refundAmount: refundAmount
    };
  });

  // ═══════════════════════════════════════════════
  //  BOUND ACTION: ship (on SalesOrders)
  // ═══════════════════════════════════════════════
  this.on('ship', 'SalesOrders', async (req) => {
    const orderId = req.params[0]?.ID || req.params[0];
    const { trackingNumber, carrier } = req.data;

    const order = await SELECT.one.from(SalesOrders).where({ ID: orderId });

    if (!order) req.reject(404, 'Order not found');

    if (order.status !== 'Confirmed') {
      req.reject(400, `Cannot ship order in "${order.status}" status. Order must be "Confirmed" first.`);
    }

    if (!trackingNumber) req.reject(400, 'Tracking number is required');
    if (!carrier) req.reject(400, 'Carrier name is required');

    await UPDATE(SalesOrders).set({
      status: 'Shipped',
      modifiedAt: new Date().toISOString()
    }).where({ ID: orderId });

    // Estimate delivery: 5 business days from now
    const deliveryDate = new Date();
    deliveryDate.setDate(deliveryDate.getDate() + 5);

    return {
      status: 'Shipped',
      message: `Order ${order.orderNumber} shipped via ${carrier}. Tracking: ${trackingNumber}`,
      estimatedDelivery: deliveryDate.toISOString().split('T')[0]
    };
  });

};
```

---

### Calling Bound Actions (REST Client)

The URL pattern for bound actions is:

```
POST /service/Entity(key)/ServiceName.actionName
```

```http
### Confirm a specific order
POST http://localhost:4004/sales/SalesOrders(order-uuid-here)/SalesService.confirm
Content-Type: application/json

{}

### Cancel a specific order (with reason parameter)
POST http://localhost:4004/sales/SalesOrders(order-uuid-here)/SalesService.cancel
Content-Type: application/json

{
  "reason": "Customer changed their mind"
}

### Ship an order (with tracking info)
POST http://localhost:4004/sales/SalesOrders(order-uuid-here)/SalesService.ship
Content-Type: application/json

{
  "trackingNumber": "TRK-2026-ABC123",
  "carrier": "FedEx"
}
```

**URL breakdown:**
```
POST /sales/SalesOrders(uuid)/SalesService.confirm
      ↑     ↑              ↑   ↑              ↑
      │     │              │   │              └── Action name
      │     │              │   └── Service name (namespace)
      │     │              └── Specific record key
      │     └── Entity name
      └── Service path
```

---
