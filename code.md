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
