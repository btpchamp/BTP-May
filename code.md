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
