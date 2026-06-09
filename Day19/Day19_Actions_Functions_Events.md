# Day 19: Actions, Functions & Events

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 18 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Actions — Beyond CRUD | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Functions & The Difference from Actions | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Create Actions & Functions | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Events — Asynchronous Communication | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Complete Business Workflow | 60 min |
| 16:00 - 16:45 | Session 6: Testing Actions, Functions & Events | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

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

### Unbound vs Bound — Side by Side

```
UNBOUND ACTION:
POST /analytics/GenerateReport          ← No entity, no key
{ "reportType": "Sales", ... }             Just calling the service

BOUND ACTION:
POST /sales/SalesOrders(uuid)/SalesService.confirm   ← Specific order
{ }                                                      Calling ON that order
```

| Feature | Unbound | Bound |
|---------|---------|-------|
| **Belongs to** | Service | Entity |
| **Targets** | Nothing specific | One specific record |
| **URL** | `/service/ActionName` | `/service/Entity(key)/Service.Action` |
| **CDS location** | Directly in `service { }` | Inside entity's `actions { }` |
| **Handler registration** | `this.on('ActionName', handler)` | `this.on('actionName', 'Entity', handler)` |
| **`req.params`** | Empty (no key) | Contains the entity key |
| **Example** | GenerateReport, HealthCheck | Approve, Cancel, Ship |

---

### Action Parameters — Input & Output Types

#### Simple Parameters (Flat)

```cds
action doSomething(
  name    : String(100);
  amount  : Decimal(10,2);
  active  : Boolean
) returns {
  success : Boolean;
  message : String(200);
};
```

#### Array Parameter (Multiple Items)

```cds
action bulkApprove(
  orderIds : array of UUID    // Accept multiple IDs
) returns {
  approvedCount : Integer;
  failedCount   : Integer;
  message       : String(200);
};
```

Handler:
```javascript
this.on('bulkApprove', async (req) => {
  const { orderIds } = req.data;
  let approved = 0, failed = 0;

  for (const id of orderIds) {
    const order = await SELECT.one.from(SalesOrders).where({ ID: id });
    if (order && order.status === 'New') {
      await UPDATE(SalesOrders).set({ status: 'Confirmed' }).where({ ID: id });
      approved++;
    } else {
      failed++;
    }
  }

  return {
    approvedCount: approved,
    failedCount: failed,
    message: `Processed ${orderIds.length} orders: ${approved} approved, ${failed} failed`
  };
});
```

#### Returning an Entity Type

```cds
action getTopProducts(
  limit : Integer
) returns array of Products;   // Returns full Product entities!
```

Handler:
```javascript
this.on('getTopProducts', async (req) => {
  const { limit } = req.data;
  const products = await SELECT.from('Products')
    .orderBy('rating desc')
    .limit(limit || 5);
  return products;
});
```

---

### Quick Check: Actions

**Which is an unbound action?**
- A) `POST /sales/Orders(123)/SalesService.approve`
- B) `POST /analytics/GenerateReport`
- C) `GET /catalog/Books`
- D) `PATCH /admin/Books(123)`

<details>
<summary>Answer</summary>

**B** — `GenerateReport` is called directly on the service (no entity key in URL). That's an unbound action.
- A is a bound action (targets a specific order)
- C is a READ operation (not an action)
- D is an UPDATE operation (not an action)

</details>

---

## Session 2: Functions & The Difference from Actions (10:45 - 12:00)

### What is a Function?

A **Function** is like an action but with one critical difference: **it's read-only**. It never changes data.

```
Actions   → Can change data (create, update, delete records)  → POST
Functions → ONLY read data (no side effects, no changes)      → GET
```

**Real-world analogy:**

```
Action   = Pressing a button (something HAPPENS — state changes)
           Like: "Submit Order" button → order status changes

Function = Asking a question (you get info, nothing changes)  
           Like: "What's my order status?" → you see the status, nothing changes
```

---

### When to Use Actions vs Functions

| Use Action When... | Use Function When... |
|-------------------|---------------------|
| You CREATE something | You just READ/calculate data |
| You UPDATE something | You return computed results |
| You DELETE something | You aggregate/summarize data |
| Something in the DB changes | Nothing in the DB changes |
| There are side effects (email, log) | It's purely informational |
| Operation is NOT idempotent | Operation IS idempotent |

**Idempotent** = calling it 10 times gives the same result with no side effects.
- `GetOrderTotal()` → always returns the same total (Function)
- `ApproveOrder()` → first call changes status, second call fails (Action)

---

### Defining Functions in CDS

```cds
service AnalyticsService @(path: '/analytics') {

  // Unbound function — service-level, read-only
  function GetOrderStatistics(
    year  : Integer;
    month : Integer
  ) returns {
    totalOrders     : Integer;
    totalRevenue    : Decimal(15,2);
    averageOrderValue : Decimal(10,2);
    topProduct      : String(100);
  };

  // Another unbound function — no parameters
  function GetSystemInfo() returns {
    serverTime : DateTime;
    dbType     : String(20);
    entityCount : Integer;
  };

  @readonly entity ProductCatalog as projection on db.ProductCatalog;
}
```

---

### Bound Functions (Entity-Level, Read-Only)

```cds
service SalesService @(path: '/sales') {

  entity SalesOrders as projection on db.SalesOrders
    actions {
      // Bound actions (can modify)
      action confirm() returns { status: String; message: String; };
      action cancel(reason: String) returns { status: String; };

      // Bound function (read-only, just calculates)
      function getTotal() returns { netAmount: Decimal; tax: Decimal; gross: Decimal; };
      function getItemCount() returns Integer;
    };

  entity Products as projection on db.Products
    actions {
      function checkAvailability(quantity: Integer) returns {
        available : Boolean;
        currentStock : Integer;
        message : String;
      };
    };
}
```

---

### Implementing Function Handlers

```javascript
// srv/analytics-service.js
const cds = require('@sap/cds');

module.exports = function () {

  // ═══════════════════════════════════════════════
  //  FUNCTION: GetOrderStatistics
  // ═══════════════════════════════════════════════
  this.on('GetOrderStatistics', async (req) => {
    const { year, month } = req.data;

    // Build date range for the requested month
    const startDate = `${year}-${String(month).padStart(2, '0')}-01`;
    const endMonth = month === 12 ? 1 : month + 1;
    const endYear = month === 12 ? year + 1 : year;
    const endDate = `${endYear}-${String(endMonth).padStart(2, '0')}-01`;

    // Query orders in that period
    const orders = await SELECT.from('com.epm.SalesOrders')
      .where('orderDate >=', startDate)
      .and('orderDate <', endDate);

    if (orders.length === 0) {
      return {
        totalOrders: 0,
        totalRevenue: 0,
        averageOrderValue: 0,
        topProduct: 'N/A'
      };
    }

    // Calculate statistics
    const totalRevenue = orders.reduce((sum, o) => sum + (o.grossAmount || 0), 0);
    const avgValue = totalRevenue / orders.length;

    return {
      totalOrders: orders.length,
      totalRevenue: +totalRevenue.toFixed(2),
      averageOrderValue: +avgValue.toFixed(2),
      topProduct: 'Laptop Pro' // In real app: query most-ordered product
    };
  });

  // ═══════════════════════════════════════════════
  //  FUNCTION: GetSystemInfo
  // ═══════════════════════════════════════════════
  this.on('GetSystemInfo', async (req) => {
    const products = await SELECT.from('com.epm.Products');

    return {
      serverTime: new Date().toISOString(),
      dbType: 'SQLite (Development)',
      entityCount: products.length
    };
  });

};
```

---

### Calling Functions (REST Client)

**Key difference from actions:** Functions use **GET** requests (not POST)!

```http
### Call unbound function: GetOrderStatistics
GET http://localhost:4004/analytics/GetOrderStatistics(year=2026,month=5)

### Call unbound function: GetSystemInfo (no parameters)
GET http://localhost:4004/analytics/GetSystemInfo()

### Call bound function: getTotal on a specific order
GET http://localhost:4004/sales/SalesOrders(order-uuid)/SalesService.getTotal()

### Call bound function: checkAvailability on a specific product
GET http://localhost:4004/sales/Products(product-uuid)/SalesService.checkAvailability(quantity=50)
```

**Notice the URL differences:**

```
FUNCTION (GET):  GET  /service/FunctionName(param=value)
ACTION (POST):   POST /service/ActionName   with JSON body

BOUND FUNCTION (GET):  GET  /service/Entity(key)/Service.funcName()
BOUND ACTION (POST):   POST /service/Entity(key)/Service.actionName  with body
```

---

### Functions: Passing Parameters

Parameters are passed in the URL (like query params), not in the request body:

```
// Single parameter
GET /analytics/GetOrderStatistics(year=2026,month=5)

// Multiple parameters
GET /sales/Products(uuid)/SalesService.checkAvailability(quantity=50)

// No parameters (empty parentheses required!)
GET /analytics/GetSystemInfo()
```

---

### Implementing Bound Functions

```javascript
// srv/sales-service.js (add to existing file)

// Bound function: getTotal on SalesOrders
this.on('getTotal', 'SalesOrders', async (req) => {
  const orderId = req.params[0]?.ID || req.params[0];

  const items = await SELECT.from('com.epm.SalesOrderItems')
    .where({ order_ID: orderId });

  const netAmount = items.reduce((sum, item) =>
    sum + (item.quantity * item.unitPrice), 0);
  const tax = netAmount * 0.18;
  const gross = netAmount + tax;

  return {
    netAmount: +netAmount.toFixed(2),
    tax: +tax.toFixed(2),
    gross: +gross.toFixed(2)
  };
});

// Bound function: getItemCount on SalesOrders
this.on('getItemCount', 'SalesOrders', async (req) => {
  const orderId = req.params[0]?.ID || req.params[0];

  const items = await SELECT.from('com.epm.SalesOrderItems')
    .where({ order_ID: orderId });

  return items.length;
});

// Bound function: checkAvailability on Products
this.on('checkAvailability', 'Products', async (req) => {
  const productId = req.params[0]?.ID || req.params[0];
  const { quantity } = req.data;

  const product = await SELECT.one.from('com.epm.Products')
    .where({ ID: productId });

  if (!product) req.reject(404, 'Product not found');

  const available = product.stock >= quantity;

  return {
    available: available,
    currentStock: product.stock,
    message: available
      ? `Yes! ${product.stock} units available (you need ${quantity})`
      : `Sorry, only ${product.stock} units in stock (you need ${quantity})`
  };
});
```

---

### The Complete Comparison: Action vs Function vs CRUD

```
┌───────────────────────────────────────────────────────────────┐
│                CRUD vs Actions vs Functions                     │
├────────────────┬────────────────┬────────────────┬────────────┤
│   Feature      │     CRUD       │    Action      │  Function  │
├────────────────┼────────────────┼────────────────┼────────────┤
│ HTTP Method    │ GET/POST/      │ POST           │ GET        │
│                │ PUT/PATCH/     │                │            │
│                │ DELETE         │                │            │
├────────────────┼────────────────┼────────────────┼────────────┤
│ Changes Data?  │ Yes (except    │ Yes (can       │ No (read-  │
│                │ GET)           │ modify DB)     │ only!)     │
├────────────────┼────────────────┼────────────────┼────────────┤
│ Defined In     │ Automatic from │ CDS: action    │ CDS:       │
│                │ entity         │ keyword        │ function   │
│                │ projection     │                │ keyword    │
├────────────────┼────────────────┼────────────────┼────────────┤
│ Parameters     │ URL/body/query │ JSON body      │ URL params │
│ Passed Via     │ options        │ (POST)         │ (GET)      │
├────────────────┼────────────────┼────────────────┼────────────┤
│ Example        │ Create Book,   │ Approve Order, │ Get Stats, │
│                │ Read Books,    │ Cancel Order,  │ Check Stock│
│                │ Update price   │ Send Email     │ Get Total  │
├────────────────┼────────────────┼────────────────┼────────────┤
│ Auto-generated?│ Yes! (free)    │ No (you write) │ No (you    │
│                │                │                │ write)     │
└────────────────┴────────────────┴────────────────┴────────────┘
```

---

### Quick Decision Tree

```
"I need to..."

├── Just read/query existing data → Use CRUD (GET with $filter, $expand)
│
├── Create/Update/Delete a single record → Use CRUD (POST/PATCH/DELETE)
│
├── Perform a complex operation that changes state?
│   ├── Targets a specific record? → BOUND ACTION
│   └── Service-level (no specific record)? → UNBOUND ACTION
│
└── Get calculated/aggregated data without changing anything?
    ├── Targets a specific record? → BOUND FUNCTION
    └── Service-level? → UNBOUND FUNCTION
```

---

## Session 3: Hands-on — Create Actions & Functions (12:00 - 13:00)

### Exercise: Build a Complete Order Management Service

We'll build a service with CRUD + Actions + Functions to manage the full order lifecycle:

```
Order Lifecycle:
  [New] ──confirm──► [Confirmed] ──ship──► [Shipped] ──deliver──► [Delivered]
    │                      │
    └────cancel────────────┘──────────────► [Cancelled]
```

---

### Step 1: Define the Service (CDS)

**File: `srv/order-service.cds`**

```cds
using { com.epm as db } from '../db/schema';

service OrderService @(path: '/orders') {

  // Standard CRUD entities
  entity Orders as projection on db.SalesOrders
    actions {
      // Bound actions (entity-specific, modify data)
      action confirm() returns { status: String; message: String; };
      action cancel(reason: String(500)) returns { status: String; message: String; refund: Decimal; };
      action ship(trackingNumber: String(50); carrier: String(50)) returns { status: String; message: String; };
      action deliver() returns { status: String; message: String; };

      // Bound functions (entity-specific, read-only)
      function getTotal() returns { net: Decimal; tax: Decimal; gross: Decimal; };
      function getTimeline() returns array of {
        event: String;
        timestamp: DateTime;
        description: String;
      };
    };

  entity OrderItems as projection on db.SalesOrderItems;
  entity Customers as projection on db.Customers;
  @readonly entity Products as projection on db.Products;

  // Unbound actions (service-level)
  action bulkConfirm(orderIds: array of UUID) returns {
    confirmed: Integer;
    failed: Integer;
    message: String;
  };

  // Unbound functions (service-level, read-only)
  function getOrderStats(year: Integer; month: Integer) returns {
    totalOrders: Integer;
    newOrders: Integer;
    confirmedOrders: Integer;
    shippedOrders: Integer;
    deliveredOrders: Integer;
    cancelledOrders: Integer;
    totalRevenue: Decimal;
  };

  function getTopCustomers(limit: Integer) returns array of {
    customerName: String;
    orderCount: Integer;
    totalSpent: Decimal;
  };
}
```

---

### Step 2: Implement the Handlers

**File: `srv/order-service.js`**

```javascript
const cds = require('@sap/cds');

module.exports = function () {

  // ═══════════════════════════════════════════════
  //  BOUND ACTION: confirm
  // ═══════════════════════════════════════════════
  this.on('confirm', 'Orders', async (req) => {
    const { ID } = req.params[0];
    const { SalesOrders } = cds.entities;

    const order = await SELECT.one.from(SalesOrders).where({ ID });
    if (!order) req.reject(404, 'Order not found');

    if (order.status !== 'New') {
      req.reject(400, `Cannot confirm: Order is "${order.status}". Only "New" orders can be confirmed.`);
    }

    await UPDATE(SalesOrders)
      .set({ status: 'Confirmed' })
      .where({ ID });

    return {
      status: 'Confirmed',
      message: `Order ${order.orderNumber} has been confirmed and is ready for processing.`
    };
  });

  // ═══════════════════════════════════════════════
  //  BOUND ACTION: cancel
  // ═══════════════════════════════════════════════
  this.on('cancel', 'Orders', async (req) => {
    const { ID } = req.params[0];
    const { reason } = req.data;
    const { SalesOrders } = cds.entities;

    const order = await SELECT.one.from(SalesOrders).where({ ID });
    if (!order) req.reject(404, 'Order not found');

    if (order.status === 'Delivered') {
      req.reject(400, 'Cannot cancel a delivered order. Please initiate a return.');
    }
    if (order.status === 'Cancelled') {
      req.reject(400, 'Order is already cancelled.');
    }
    if (!reason) {
      req.reject(400, 'Please provide a reason for cancellation.');
    }

    await UPDATE(SalesOrders)
      .set({ status: 'Cancelled' })
      .where({ ID });

    const refund = ['Confirmed', 'Shipped'].includes(order.status)
      ? order.grossAmount : 0;

    return {
      status: 'Cancelled',
      message: `Order ${order.orderNumber} cancelled. Reason: ${reason}`,
      refund: refund
    };
  });

  // ═══════════════════════════════════════════════
  //  BOUND ACTION: ship
  // ═══════════════════════════════════════════════
  this.on('ship', 'Orders', async (req) => {
    const { ID } = req.params[0];
    const { trackingNumber, carrier } = req.data;
    const { SalesOrders } = cds.entities;

    const order = await SELECT.one.from(SalesOrders).where({ ID });
    if (!order) req.reject(404, 'Order not found');

    if (order.status !== 'Confirmed') {
      req.reject(400, `Cannot ship: Order must be "Confirmed". Current status: "${order.status}"`);
    }
    if (!trackingNumber) req.reject(400, 'Tracking number is required');
    if (!carrier) req.reject(400, 'Carrier name is required');

    await UPDATE(SalesOrders)
      .set({ status: 'Shipped' })
      .where({ ID });

    return {
      status: 'Shipped',
      message: `Order ${order.orderNumber} shipped via ${carrier}. Tracking: ${trackingNumber}`
    };
  });

  // ═══════════════════════════════════════════════
  //  BOUND ACTION: deliver
  // ═══════════════════════════════════════════════
  this.on('deliver', 'Orders', async (req) => {
    const { ID } = req.params[0];
    const { SalesOrders } = cds.entities;

    const order = await SELECT.one.from(SalesOrders).where({ ID });
    if (!order) req.reject(404, 'Order not found');

    if (order.status !== 'Shipped') {
      req.reject(400, `Cannot mark as delivered: Order must be "Shipped". Current: "${order.status}"`);
    }

    await UPDATE(SalesOrders)
      .set({ status: 'Delivered' })
      .where({ ID });

    return {
      status: 'Delivered',
      message: `Order ${order.orderNumber} has been delivered successfully!`
    };
  });

  // ═══════════════════════════════════════════════
  //  BOUND FUNCTION: getTotal
  // ═══════════════════════════════════════════════
  this.on('getTotal', 'Orders', async (req) => {
    const { ID } = req.params[0];

    const items = await SELECT.from('com.epm.SalesOrderItems')
      .where({ order_ID: ID });

    const net = items.reduce((sum, item) =>
      sum + (item.quantity * item.unitPrice), 0);
    const tax = net * 0.18;

    return {
      net: +net.toFixed(2),
      tax: +tax.toFixed(2),
      gross: +(net + tax).toFixed(2)
    };
  });

  // ═══════════════════════════════════════════════
  //  UNBOUND ACTION: bulkConfirm
  // ═══════════════════════════════════════════════
  this.on('bulkConfirm', async (req) => {
    const { orderIds } = req.data;
    const { SalesOrders } = cds.entities;

    if (!orderIds || orderIds.length === 0) {
      req.reject(400, 'Please provide at least one order ID');
    }

    let confirmed = 0, failed = 0;

    for (const id of orderIds) {
      const order = await SELECT.one.from(SalesOrders).where({ ID: id });
      if (order && order.status === 'New') {
        await UPDATE(SalesOrders).set({ status: 'Confirmed' }).where({ ID: id });
        confirmed++;
      } else {
        failed++;
      }
    }

    return {
      confirmed,
      failed,
      message: `Bulk operation complete: ${confirmed} confirmed, ${failed} skipped/failed`
    };
  });

  // ═══════════════════════════════════════════════
  //  UNBOUND FUNCTION: getOrderStats
  // ═══════════════════════════════════════════════
  this.on('getOrderStats', async (req) => {
    const { year, month } = req.data;

    const startDate = `${year}-${String(month).padStart(2, '0')}-01`;
    const nextMonth = month === 12 ? 1 : month + 1;
    const nextYear = month === 12 ? year + 1 : year;
    const endDate = `${nextYear}-${String(nextMonth).padStart(2, '0')}-01`;

    const orders = await SELECT.from('com.epm.SalesOrders')
      .where('orderDate >=', startDate)
      .and('orderDate <', endDate);

    const stats = {
      totalOrders: orders.length,
      newOrders: orders.filter(o => o.status === 'New').length,
      confirmedOrders: orders.filter(o => o.status === 'Confirmed').length,
      shippedOrders: orders.filter(o => o.status === 'Shipped').length,
      deliveredOrders: orders.filter(o => o.status === 'Delivered').length,
      cancelledOrders: orders.filter(o => o.status === 'Cancelled').length,
      totalRevenue: +orders.reduce((sum, o) => sum + (o.grossAmount || 0), 0).toFixed(2)
    };

    return stats;
  });

  // ═══════════════════════════════════════════════
  //  UNBOUND FUNCTION: getTopCustomers
  // ═══════════════════════════════════════════════
  this.on('getTopCustomers', async (req) => {
    const limit = req.data.limit || 5;

    const orders = await SELECT.from('com.epm.SalesOrders')
      .columns('customer_ID', 'grossAmount');

    // Group by customer
    const customerMap = {};
    for (const order of orders) {
      if (!customerMap[order.customer_ID]) {
        customerMap[order.customer_ID] = { count: 0, total: 0 };
      }
      customerMap[order.customer_ID].count++;
      customerMap[order.customer_ID].total += order.grossAmount || 0;
    }

    // Get customer names
    const results = [];
    for (const [id, data] of Object.entries(customerMap)) {
      const customer = await SELECT.one.from('com.epm.Customers')
        .where({ ID: id })
        .columns('customerName');
      if (customer) {
        results.push({
          customerName: customer.customerName,
          orderCount: data.count,
          totalSpent: +data.total.toFixed(2)
        });
      }
    }

    // Sort by total spent and return top N
    return results
      .sort((a, b) => b.totalSpent - a.totalSpent)
      .slice(0, limit);
  });

};
```

---

### Step 3: Test Everything

**File: `tests/actions-functions.http`**

```http
@base = http://localhost:4004/orders

### ═══════════════════════════════════════
### SETUP: Create test data
### ═══════════════════════════════════════

### Create a customer
POST {{base}}/Customers
Content-Type: application/json

{
  "customerName": "Test Customer",
  "email": "test@example.com",
  "city": "Mumbai",
  "creditLimit": 500000
}

### Create an order (use customer ID from above)
POST {{base}}/Orders
Content-Type: application/json

{
  "orderNumber": "SO-TEST-001",
  "customer_ID": "PASTE-CUSTOMER-ID",
  "orderDate": "2026-06-02",
  "grossAmount": 1500.00,
  "netAmount": 1271.19,
  "taxAmount": 228.81,
  "status": "New"
}


### ═══════════════════════════════════════
### BOUND ACTIONS (on specific orders)
### ═══════════════════════════════════════

### Confirm the order
POST {{base}}/Orders(PASTE-ORDER-ID)/OrderService.confirm
Content-Type: application/json

{}

### Try to confirm again (should fail — already confirmed)
POST {{base}}/Orders(PASTE-ORDER-ID)/OrderService.confirm
Content-Type: application/json

{}

### Ship the order
POST {{base}}/Orders(PASTE-ORDER-ID)/OrderService.ship
Content-Type: application/json

{
  "trackingNumber": "TRK-2026-001",
  "carrier": "BlueDart"
}

### Deliver the order
POST {{base}}/Orders(PASTE-ORDER-ID)/OrderService.deliver
Content-Type: application/json

{}

### Try to cancel a delivered order (should fail)
POST {{base}}/Orders(PASTE-ORDER-ID)/OrderService.cancel
Content-Type: application/json

{
  "reason": "Changed my mind"
}


### ═══════════════════════════════════════
### BOUND FUNCTIONS (read-only, on specific orders)
### ═══════════════════════════════════════

### Get order total (calculated)
GET {{base}}/Orders(PASTE-ORDER-ID)/OrderService.getTotal()


### ═══════════════════════════════════════
### UNBOUND ACTIONS (service-level)
### ═══════════════════════════════════════

### Bulk confirm multiple orders
POST {{base}}/bulkConfirm
Content-Type: application/json

{
  "orderIds": ["order-uuid-1", "order-uuid-2", "order-uuid-3"]
}


### ═══════════════════════════════════════
### UNBOUND FUNCTIONS (service-level, GET request!)
### ═══════════════════════════════════════

### Get order statistics for June 2026
GET {{base}}/getOrderStats(year=2026,month=6)

### Get top 3 customers by spending
GET {{base}}/getTopCustomers(limit=3)
```

---

## Session 4: Events — Asynchronous Communication (13:45 - 14:45)

### What are Events in CAP?

Events are like **announcements** — when something happens, you broadcast a message, and anyone who cares can listen and react.

```
┌─────────────────────────────────────────────────────────────┐
│              EVENTS = ANNOUNCEMENTS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📢 "ORDER CONFIRMED!"                                      │
│      │                                                      │
│      ├──► 📧 Email Service: "Send confirmation email"       │
│      ├──► 📦 Warehouse: "Start picking items"               │
│      ├──► 📊 Analytics: "Update daily stats"                │
│      └──► 📱 Mobile App: "Show push notification"           │
│                                                             │
│  The order service doesn't know or care who's listening.    │
│  It just announces — listeners react on their own.          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Events vs Actions — Key Difference

| Feature | Action | Event |
|---------|--------|-------|
| **Direction** | Request → Response (synchronous) | Fire and forget (asynchronous) |
| **Caller expects** | A response back | Nothing back |
| **Timing** | Waits for result | Doesn't wait |
| **Coupling** | Caller knows who handles it | Emitter doesn't know listeners |
| **Analogy** | Phone call (you wait for answer) | Radio broadcast (anyone can listen) |

---

### Real-World Analogy: WhatsApp Group vs Private Message

```
ACTION = Private Message:
  You → "Hey Rahul, please approve my PO"
  Rahul → "Done! Approved."
  (Direct, synchronous, one-to-one)

EVENT = WhatsApp Group Announcement:
  You → [Group: Team Updates] "PO-001 has been submitted!"
  (Anyone in the group can see it and react however they want)
  - Rahul sees it and starts reviewing
  - Finance sees it and notes the budget impact
  - Your manager sees it and says "nice"
  - You don't wait for any of them
```

---

### Defining Events in CDS

```cds
service OrderService @(path: '/orders') {

  // ... entities and actions ...

  // Event definitions
  event OrderConfirmed {
    orderId     : UUID;
    orderNumber : String(20);
    customer    : String(100);
    amount      : Decimal(12,2);
    confirmedBy : String(100);
    confirmedAt : DateTime;
  }

  event OrderShipped {
    orderId        : UUID;
    orderNumber    : String(20);
    trackingNumber : String(50);
    carrier        : String(50);
    estimatedDelivery : Date;
  }

  event OrderCancelled {
    orderId     : UUID;
    orderNumber : String(20);
    reason      : String(500);
    cancelledBy : String(100);
    refundAmount : Decimal(12,2);
  }

  event LowStockAlert {
    productId   : UUID;
    productName : String(100);
    currentStock : Integer;
    minStock    : Integer;
  }
}
```

---

### Emitting Events (Broadcasting)

You emit events from within your action handlers — after something important happens:

```javascript
// srv/order-service.js

this.on('confirm', 'Orders', async (req) => {
  const { ID } = req.params[0];
  const { SalesOrders, Customers } = cds.entities;

  const order = await SELECT.one.from(SalesOrders).where({ ID });
  if (!order) req.reject(404, 'Order not found');
  if (order.status !== 'New') req.reject(400, 'Only New orders can be confirmed');

  // Update the order
  await UPDATE(SalesOrders).set({ status: 'Confirmed' }).where({ ID });

  // Get customer name for the event
  const customer = await SELECT.one.from(Customers).where({ ID: order.customer_ID });

  // ★ EMIT THE EVENT ★
  await this.emit('OrderConfirmed', {
    orderId: ID,
    orderNumber: order.orderNumber,
    customer: customer?.customerName || 'Unknown',
    amount: order.grossAmount,
    confirmedBy: req.user.id,
    confirmedAt: new Date().toISOString()
  });

  return {
    status: 'Confirmed',
    message: `Order ${order.orderNumber} confirmed`
  };
});
```

---

### Subscribing to Events (Listening)

Other services (or the same service) can listen for events:

```javascript
// srv/notification-service.js
const cds = require('@sap/cds');

module.exports = function () {

  // Listen for OrderConfirmed events
  const OrderService = cds.connect.to('OrderService');

  this.on('OrderConfirmed', async (msg) => {
    const { orderNumber, customer, amount, confirmedAt } = msg.data;

    console.log(`📧 [NOTIFICATION] Order ${orderNumber} confirmed!`);
    console.log(`   Customer: ${customer}`);
    console.log(`   Amount: $${amount}`);
    console.log(`   Time: ${confirmedAt}`);

    // In real app: send email, push notification, etc.
    // await sendEmail(customer.email, `Your order ${orderNumber} is confirmed!`);
  });

  // Listen for LowStockAlert events
  this.on('LowStockAlert', async (msg) => {
    const { productName, currentStock, minStock } = msg.data;

    console.log(`⚠️ [STOCK ALERT] ${productName}: ${currentStock} units left (min: ${minStock})`);

    // In real app: notify purchasing team
    // await sendSlackAlert(`Low stock: ${productName}`);
  });

};
```

---

### Emitting Events in Action Handlers — Complete Flow

Here's how the full flow works when an order is confirmed:

```
1. Client calls:  POST /orders/Orders(uuid)/OrderService.confirm

2. Handler runs:
   ├── Validates order status
   ├── Updates order in DB
   ├── Emits 'OrderConfirmed' event ←──── BROADCAST!
   └── Returns success response to client

3. Event listeners react (asynchronously):
   ├── NotificationService → sends confirmation email
   ├── AnalyticsService → updates dashboard
   └── WarehouseService → starts picking process

4. Client already has their response (doesn't wait for listeners)
```

---

### Emitting Events After Stock Changes

```javascript
// In your BEFORE handler for order creation:
this.after('CREATE', 'OrderItems', async (data, req) => {
  const { product_ID, quantity } = data;
  const { Products } = cds.entities;

  // Reduce stock
  const product = await SELECT.one.from(Products).where({ ID: product_ID });
  const newStock = product.stock - quantity;

  await UPDATE(Products).set({ stock: newStock }).where({ ID: product_ID });

  // Check if stock is low → emit alert
  if (newStock < product.minStock) {
    await this.emit('LowStockAlert', {
      productId: product_ID,
      productName: product.productName,
      currentStock: newStock,
      minStock: product.minStock
    });
  }
});
```

---

### Events in the Same Service (Self-Listening)

You can emit and listen within the same service:

```javascript
module.exports = function () {

  // Emit when order is cancelled
  this.on('cancel', 'Orders', async (req) => {
    // ... cancel logic ...

    await this.emit('OrderCancelled', {
      orderId: ID,
      orderNumber: order.orderNumber,
      reason: reason,
      cancelledBy: req.user.id,
      refundAmount: order.grossAmount
    });

    return { status: 'Cancelled', message: '...' };
  });

  // Listen in the SAME service — handle refund
  this.on('OrderCancelled', async (msg) => {
    const { orderId, refundAmount, orderNumber } = msg.data;

    if (refundAmount > 0) {
      console.log(`💰 [REFUND] Processing refund of $${refundAmount} for order ${orderNumber}`);
      // In real app: call payment gateway to process refund
    }
  });

};
```

---

### Event Patterns Summary

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Notification** | Tell someone something happened | "Order confirmed → send email" |
| **Integration** | Trigger action in another system | "Order shipped → update ERP" |
| **Audit** | Log business events | "Order cancelled → write to audit log" |
| **Workflow** | Trigger next step | "PO submitted → notify approver" |
| **Alert** | Warn about conditions | "Stock low → alert purchasing team" |

---

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

---

## Assessment: MCQ — 15 Questions (16:45 - 17:00)

**Q1.** What HTTP method is used to call an Action?
- A) GET
- B) POST
- C) PUT
- D) PATCH

---

**Q2.** What HTTP method is used to call a Function?
- A) GET
- B) POST
- C) PUT
- D) DELETE

---

**Q3.** The key difference between Actions and Functions is:
- A) Actions are faster
- B) Actions can modify data; Functions are read-only
- C) Functions can modify data; Actions are read-only
- D) There is no difference

---

**Q4.** Where are bound actions defined in CDS?
- A) Directly in the `service { }` block
- B) In a separate `.js` file only
- C) Inside the entity's `actions { }` block
- D) In `db/schema.cds`

---

**Q5.** The URL for calling a bound action `confirm` on an Order with ID `abc-123` is:
- A) `POST /orders/confirm?id=abc-123`
- B) `POST /orders/Orders(abc-123)/OrderService.confirm`
- C) `GET /orders/Orders(abc-123)/confirm`
- D) `POST /orders/Orders/abc-123/confirm`

---

**Q6.** An unbound action URL looks like:
- A) `POST /service/Entity(key)/action`
- B) `POST /service/ActionName`
- C) `GET /service/ActionName`
- D) `PUT /service/ActionName`

---

**Q7.** Function parameters are passed via:
- A) JSON request body
- B) URL parameters (in parentheses)
- C) HTTP headers
- D) Query string after `?`

---

**Q8.** `this.on('approve', 'PurchaseOrders', handler)` registers a handler for:
- A) An unbound action
- B) A bound action on PurchaseOrders
- C) A CRUD READ on PurchaseOrders
- D) An event listener

---

**Q9.** Events in CAP are:
- A) Synchronous — caller waits for all listeners
- B) Asynchronous — fire and forget, listeners react independently
- C) Only for error handling
- D) Only available in production (HANA)

---

**Q10.** To emit an event from a handler, you use:
- A) `req.emit('EventName', data)`
- B) `this.emit('EventName', data)`
- C) `cds.emit('EventName', data)`
- D) `event.fire('EventName', data)`

---

**Q11.** Which is a valid use case for a Function (not an Action)?
- A) Approve a purchase order
- B) Calculate order statistics (read-only)
- C) Send an email notification
- D) Delete expired records

---

**Q12.** TRUE or FALSE: A bound action can access the entity key via `req.params[0]`.
- A) TRUE
- B) FALSE

---

**Q13.** What happens if a Function tries to UPDATE the database?
- A) It works fine
- B) It throws an error at runtime — functions must be read-only
- C) It works but logs a warning
- D) It creates a new record instead

---

**Q14.** In the CDS definition `action submit() returns { status: String; message: String; }`, what does `returns` define?
- A) The input parameters
- B) The structure of the response data
- C) Error codes
- D) The entity it belongs to

---

**Q15.** Event listeners in CAP are registered using:
- A) `this.on('EventName', handler)`
- B) `this.before('EventName', handler)`
- C) `this.after('EventName', handler)`
- D) `this.listen('EventName', handler)`

---

### Answer Key

<details>
<summary>Click to reveal answers</summary>

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | Actions use POST (they can change state) |
| 2 | **A** | Functions use GET (they're read-only) |
| 3 | **B** | Actions modify data, Functions only read |
| 4 | **C** | Bound actions go inside `entity X ... actions { }` |
| 5 | **B** | `POST /service/Entity(key)/ServiceName.actionName` |
| 6 | **B** | Unbound: `POST /service/ActionName` (no entity key) |
| 7 | **B** | Function params in URL: `FuncName(param=value)` |
| 8 | **B** | Second param `'PurchaseOrders'` makes it bound to that entity |
| 9 | **B** | Events are asynchronous — emitter doesn't wait for listeners |
| 10 | **B** | `this.emit('EventName', data)` from within a handler |
| 11 | **B** | Stats calculation is read-only → perfect for a function |
| 12 | **TRUE** | `req.params[0]` contains the entity key for bound actions |
| 13 | **B** | Functions are declared read-only; CAP enforces this |
| 14 | **B** | `returns` defines the shape of the response |
| 15 | **A** | `this.on('EventName', handler)` — same syntax as action handlers |

</details>

---

## Assignment: Add Business Actions to EPM Model

### Task

Add a complete set of business actions to your EPM Purchase Orders:

---

### Requirements

Add these operations to your `PurchasingService` or `SalesService`:

| # | Type | Name | Description |
|---|------|------|-------------|
| 1 | Bound Action | `submit` | Submit a Draft PO for approval |
| 2 | Bound Action | `approve` | Approve a submitted PO (with optional comment) |
| 3 | Bound Action | `reject` | Reject a submitted PO (reason required!) |
| 4 | Bound Action | `reopen` | Move a Rejected PO back to Draft for editing |
| 5 | Bound Function | `getSummary` | Get PO summary (items, total, days open) |
| 6 | Unbound Function | `getDashboard` | Get counts by status + total spend |
| 7 | Event | `POSubmitted` | Emitted when a PO is submitted |
| 8 | Event | `POApproved` | Emitted when a PO is approved |
| 9 | Event | `PORejected` | Emitted when a PO is rejected |

---

### Business Rules to Implement

| Action | Rules |
|--------|-------|
| `submit` | Only "Draft" POs can be submitted |
| | PO must have at least 1 item |
| | Total amount is auto-calculated from items |
| `approve` | Only "Submitted" POs can be approved |
| | Optional comment parameter |
| `reject` | Only "Submitted" POs can be rejected |
| | Reason is REQUIRED (400 if missing) |
| `reopen` | Only "Rejected" POs can be reopened |
| | Status goes back to "Draft" |

---

### Status Flow Diagram

```
       submit         approve         receive
[Draft] ────► [Submitted] ────► [Approved] ────► [Received]
   ▲                │
   │    reopen      │ reject
   └────────────────└────────► [Rejected]
```

---

### Deliverables

1. **CDS file** with action/function/event definitions
2. **JS handler file** with all implementations
3. **HTTP test file** with test cases covering:
   - Happy path (Draft → Submit → Approve → Receive)
   - Rejection path (Draft → Submit → Reject → Reopen → Submit again)
   - Error cases (wrong status transitions, missing parameters)
   - Function calls (getSummary, getDashboard)
4. **Console output** showing events firing correctly

---

### Verification Checklist

- [ ] `cds watch` starts without errors
- [ ] Submit action works on Draft POs only
- [ ] Submit fails if PO has no items
- [ ] Approve works on Submitted POs only
- [ ] Reject requires a reason
- [ ] Reopen works on Rejected POs only
- [ ] getSummary returns correct item count and total
- [ ] getDashboard returns correct counts per status
- [ ] Console shows event messages on submit/approve/reject
- [ ] Stock increases when PO is received

---

## End of Day Summary

### What We Learned

| Concept | Key Takeaway |
|---------|-------------|
| **Actions** | Custom operations that CAN modify data (POST) |
| **Functions** | Custom operations that are READ-ONLY (GET) |
| **Unbound** | Belongs to the service (no specific record) |
| **Bound** | Belongs to an entity instance (targets one record) |
| **Events** | Asynchronous announcements — fire and forget |
| **`this.emit()`** | Broadcast an event from a handler |
| **`this.on()`** | Listen for events OR implement actions/functions |
| **Parameters** | Actions: JSON body / Functions: URL params |

### The Decision Tree

```
Do you need to modify data?
├── YES → ACTION (POST)
│   ├── On a specific record? → Bound Action
│   └── Service-level? → Unbound Action
│
└── NO (read-only) → FUNCTION (GET)
    ├── On a specific record? → Bound Function
    └── Service-level? → Unbound Function

After something happens, need to notify others?
└── EMIT an EVENT (asynchronous, fire-and-forget)
```

### The One-Liner

> **Actions DO things, Functions ASK things, Events ANNOUNCE things.**

---

### Looking Ahead: Day 20

Tomorrow: **Service Layer Review & Practice Day**
- Recap of Days 16-19 (Services, OData, CRUD, Handlers, Actions)
- Weekly quiz covering the service layer
- Mini project: Complete e-commerce backend with full workflow

---

*End of Day 19 — You now have the tools to build real business workflows in CAP!*
