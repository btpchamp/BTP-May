
## What You'll Achieve Today

By the end of this review day, you will:
- Apply service design best practices and avoid common pitfalls
- Design multi-service architectures with clear boundaries
- Implement service-to-service communication within one project
- Use annotations like `@readonly`, `@insertonly`, `@requires` effectively
- Integrate ALL layers: database + service + custom handlers + actions/functions
- Perform end-to-end testing covering the complete data flow
- Build a production-quality Purchase Order management backend from scratch

---

## Week 4 Kickoff & Quick Recap (09:00 - 09:15)

### The Service Layer Journey (Days 16-19)

```
Day 16: CAP Services Basics
   └─► What is a service? CDS definitions, projections, URLs

Day 17: OData Query Operations
   └─► $filter, $select, $expand, $orderby, $top/$skip, $count

Day 18: CRUD & Custom Logic
   └─► POST/GET/PUT/PATCH/DELETE, before/on/after handlers, validation

Day 19: Actions, Functions & Events
   └─► Bound/unbound actions, functions, events, business workflows

Day 20: Review & Integration  ← YOU ARE HERE
   └─► Best practices, multi-service, integration, mini project
```

### Quick Fire: Core Concepts

1. Services expose data through what keyword? → _____
2. `@readonly` blocks which HTTP methods? → _____
3. Before handlers are used for? → _____
4. Actions use which HTTP method? → _____
5. Functions use which HTTP method? → _____
6. `req.error()` vs `req.reject()` — which stops immediately? → _____
7. Events are synchronous or asynchronous? → _____
8. Bound actions target a specific _____ ? → _____

<details>
<summary>Answers</summary>

1. `as projection on` (exposes entities through the service)
2. POST, PUT, PATCH, DELETE (only GET works)
3. **Validation** — rejecting bad input before DB operation
4. **POST** (actions can modify data)
5. **GET** (functions are read-only)
6. `req.reject()` stops immediately; `req.error()` collects multiple errors
7. **Asynchronous** — fire and forget
8. A specific **entity instance** (record)

</details>

---

## Session 1: Service Best Practices & Patterns (09:15 - 10:30)

### The Architecture We've Built

```
┌──────────────────────────────────────────────────────────────┐
│                    CAP Application Stack                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐    ┌───────────────┐    ┌──────────────┐  │
│  │  app/ (UI)   │───►│  srv/ (API)   │───►│  db/ (Data)  │  │
│  │              │    │               │    │              │  │
│  │  Fiori       │    │  Services     │    │  schema.cds  │  │
│  │  REST Client │    │  Handlers     │    │  CSV data    │  │
│  │  Mobile App  │    │  Actions      │    │  SQLite/HANA │  │
│  └──────────────┘    └───────────────┘    └──────────────┘  │
│                                                              │
│         VIEW              CONTROLLER           MODEL         │
│    (presentation)      (business logic)     (data layer)     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### Best Practice 1: One Service Per Business Domain

Don't create one giant service with everything. Split by business domain:

```cds
// ❌ BAD: One service for everything
service EverythingService {
  entity Products as projection on db.Products;
  entity Orders as projection on db.Orders;
  entity Employees as projection on db.Employees;
  entity Reports as projection on db.Reports;
  entity Invoices as projection on db.Invoices;
  entity Customers as projection on db.Customers;
  entity Suppliers as projection on db.Suppliers;
  // ... 20 more entities
}
```

```cds
// ✅ GOOD: Split by business domain
service CatalogService @(path:'/catalog') {
  @readonly entity Products as projection on db.Products;
  @readonly entity Categories as projection on db.Categories;
}

service SalesService @(path:'/sales') {
  entity Orders as projection on db.SalesOrders;
  entity Customers as projection on db.Customers;
}

service PurchasingService @(path:'/purchasing') {
  entity PurchaseOrders as projection on db.PurchaseOrders;
  entity Suppliers as projection on db.Suppliers;
}

service AdminService @(path:'/admin') {
  entity Products as projection on db.Products;
  entity Categories as projection on db.Categories;
  entity Suppliers as projection on db.Suppliers;
}
```

**Why?**
- Each service has a clear purpose and audience
- Security is easier (admin service needs auth, catalog doesn't)
- API is self-documenting (URL tells you the domain)
- Services can evolve independently

---

### Best Practice 2: Control Access with Annotations

```cds
service CatalogService {
  // Public browsing — no modifications
  @readonly entity Products as projection on db.Products;
  @readonly entity Categories as projection on db.Categories;

  // Anyone can write a review but can't update/delete
  @insertonly entity Reviews as projection on db.Reviews;
}
```

| Annotation | Allows | Blocks |
|-----------|--------|--------|
| `@readonly` | GET | POST, PUT, PATCH, DELETE |
| `@insertonly` | POST | PUT, PATCH, DELETE (and sometimes GET on single) |
| (none) | Everything | Nothing (full CRUD) |

**When to use:**
- `@readonly` → Public catalogs, reports, reference data
- `@insertonly` → Event logs, reviews, audit trails (write once, never modify)
- No annotation → Admin interfaces, full CRUD operations

---

### Best Practice 3: Hide Internal Fields with Projections

Don't expose everything from your database model:

```cds
// Database model (internal, complete):
entity Products : cuid, managed {
  productName   : String(100);
  price         : Decimal(10,2);
  costPrice     : Decimal(10,2);   // INTERNAL: what we pay
  marginPercent : Decimal(5,2);    // INTERNAL: profit margin
  stock         : Integer;
  internalNotes : String(500);     // INTERNAL: not for customers
  supplier      : Association to Suppliers;
}
```

```cds
// Public service — hide sensitive fields:
service CatalogService {
  @readonly entity Products as projection on db.Products {
    ID, productName, price, stock, supplier
    // costPrice, marginPercent, internalNotes are HIDDEN
  };
}

// Admin service — show everything:
service AdminService {
  entity Products as projection on db.Products;
  // All fields visible including costPrice, marginPercent
}
```

---

### Best Practice 4: Use `@(path)` for Clean URLs

```cds
// Without @path — URL derived from service name:
service MasterDataManagementService { }   // → /master-data-management (ugly!)

// With @path — explicit clean URL:
service MasterDataManagementService @(path: '/master-data') { }  // → /master-data (clean!)
```

**URL conventions:**
- Use lowercase with dashes: `/purchase-orders`, `/master-data`
- Keep it short: `/sales` not `/sales-order-management-service`
- Be RESTful: noun-based (`/orders`) not verb-based (`/get-orders`)

---

### Best Practice 5: Separate Read and Write Services

For complex domains, consider separate read (query) and write (command) services:

```cds
// Write service — full CRUD + actions
service OrderManagementService @(path: '/order-mgmt') {
  entity Orders as projection on db.SalesOrders
    actions {
      action confirm();
      action cancel(reason: String);
      action ship(tracking: String);
    };
  entity OrderItems as projection on db.SalesOrderItems;
}

// Read service — optimized views for UIs and reports
service OrderQueryService @(path: '/order-query') {
  @readonly entity OrderList as projection on db.OrderListView;
  @readonly entity OrderDetail as projection on db.OrderDetailView;

  function getStats(year: Integer) returns { ... };
  function getTopProducts(limit: Integer) returns array of { ... };
}
```

**Why separate?**
- Write service has handlers, validations, complex logic
- Read service is simple, fast, optimized for display
- Can scale independently in production
- Clearer API documentation

---

### Best Practice 6: Consistent Error Handling

Create a pattern for error responses across all handlers:

```javascript
// srv/utils/errors.js (shared utility)
module.exports = {
  notFound: (entity, id) => `${entity} with ID ${id} not found`,
  invalidStatus: (current, action) =>
    `Cannot ${action}: Current status is "${current}"`,
  required: (field) => `${field} is required`,
  invalid: (field, reason) => `Invalid ${field}: ${reason}`,
  conflict: (reason) => reason
};
```

Usage:
```javascript
const err = require('./utils/errors');

this.before('CREATE', 'Orders', (req) => {
  if (!req.data.customer_ID) req.error(400, err.required('Customer'));
  if (req.data.amount <= 0)  req.error(400, err.invalid('amount', 'must be positive'));
});

this.on('confirm', 'Orders', async (req) => {
  const order = await SELECT.one.from(Orders).where({ ID });
  if (!order) req.reject(404, err.notFound('Order', ID));
  if (order.status !== 'New') req.reject(400, err.invalidStatus(order.status, 'confirm'));
});
```

---

### Best Practice 7: Organize Handler Files

For large services, split handlers into logical groups:

```
srv/
├── order-service.cds              ← Service definition
├── order-service.js               ← Main handler file (registers all)
├── handlers/
│   ├── orders-validation.js       ← Before handlers (validation)
│   ├── orders-actions.js          ← Action implementations
│   ├── orders-functions.js        ← Function implementations
│   └── orders-events.js           ← Event handlers
```

**Main handler file imports sub-handlers:**
```javascript
// srv/order-service.js
const cds = require('@sap/cds');

module.exports = function () {
  // Register validation handlers
  require('./handlers/orders-validation').call(this);
  // Register action handlers
  require('./handlers/orders-actions').call(this);
  // Register function handlers
  require('./handlers/orders-functions').call(this);
  // Register event listeners
  require('./handlers/orders-events').call(this);
};
```

```javascript
// srv/handlers/orders-validation.js
module.exports = function () {
  this.before('CREATE', 'Orders', (req) => {
    // validation logic...
  });
  this.before('UPDATE', 'Orders', (req) => {
    // validation logic...
  });
};
```

---

### Best Practice 8: Use `cds.entities` for Entity References

```javascript
// ❌ BAD: Hardcoded entity strings (fragile, typo-prone)
const order = await SELECT.one.from('com.epm.SalesOrders').where({ ID });
await UPDATE('com.epm.Products').set({ stock: 0 }).where({ ID: productId });

// ✅ GOOD: Use cds.entities (type-safe, refactor-friendly)
const { SalesOrders, Products } = cds.entities;
const order = await SELECT.one.from(SalesOrders).where({ ID });
await UPDATE(Products).set({ stock: 0 }).where({ ID: productId });
```

---

### Anti-Patterns to Avoid

| # | Anti-Pattern | Problem | Fix |
|---|-------------|---------|-----|
| 1 | One giant service with 30+ entities | Hard to secure, maintain, document | Split by business domain |
| 2 | Exposing all DB fields in public API | Security risk, leaks internal data | Use projections to hide fields |
| 3 | No `@readonly` on catalog services | Users can accidentally modify data | Always annotate read-only entities |
| 4 | Business logic in `on` handlers without `next()` | Loses default CRUD functionality | Use `before`/`after` when possible |
| 5 | No validation in handlers | Bad data enters the database | Always validate in `before` handlers |
| 6 | Catching errors silently | Bugs are hidden, hard to debug | Log errors, return proper messages |
| 7 | Synchronous operations for notifications | Slows down the main request | Use events for side effects |
| 8 | Status changes via PATCH without validation | Invalid state transitions possible | Use bound actions for workflow |

---

### Service Design Summary Card

```
┌─────────────────────────────────────────────────────────────┐
│              SERVICE DESIGN CHECKLIST                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  □ One service per business domain                          │
│  □ @readonly for public/read-only entities                  │
│  □ Projections hide internal fields                         │
│  □ Clean @path annotations                                  │
│  □ before handlers for ALL validations                      │
│  □ after handlers for computed fields                       │
│  □ Bound actions for workflow operations                    │
│  □ Functions for read-only calculations                     │
│  □ Events for async side effects                            │
│  □ Consistent error messages                                │
│  □ req.error() for multiple validations                     │
│  □ req.reject() for critical failures                       │
│  □ async handlers for DB lookups                            │
│  □ Test every endpoint with REST client                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Session 2: Multiple Services & Service-to-Service Communication (10:45 - 12:00)

### Multi-Service Architecture

A typical enterprise CAP application has several services:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ┌───────────────┐   ┌───────────────┐  ┌──────────────┐  │
│  │ CatalogService│   │ SalesService  │  │ AdminService │  │
│  │ /catalog      │   │ /sales        │  │ /admin       │  │
│  │               │   │               │  │              │  │
│  │ @readonly     │   │ Orders        │  │ All entities │  │
│  │ Products      │   │ Customers     │  │ Full CRUD    │  │
│  │ Categories    │   │ OrderItems    │  │              │  │
│  └───────┬───────┘   └───────┬───────┘  └──────┬───────┘  │
│          │                    │                  │          │
│          └────────────────────┼──────────────────┘          │
│                               │                             │
│                    ┌──────────▼──────────┐                  │
│                    │    db/schema.cds     │                  │
│                    │   (Shared Models)    │                  │
│                    └─────────────────────┘                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

All services share the same database model but expose different views of the data.

---

### Multiple Services in Practice

**File structure:**
```
srv/
├── catalog-service.cds      → /catalog (public, read-only)
├── catalog-service.js
├── sales-service.cds        → /sales (sales team, orders)
├── sales-service.js
├── purchasing-service.cds   → /purchasing (procurement team)
├── purchasing-service.js
├── admin-service.cds        → /admin (administrators only)
└── admin-service.js
```

**Each service has its own:**
- CDS definition (what to expose)
- JavaScript handler (custom logic)
- URL path (how to access it)
- Access rules (who can use it)

---

### Service-to-Service Communication

Sometimes one service needs to call another service's logic. In CAP, you use `cds.connect.to()`:

```javascript
// srv/purchasing-service.js
const cds = require('@sap/cds');

module.exports = function () {

  this.on('receive', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];

    // ... receive PO logic ...

    // Need to update product stock → that's managed by CatalogService
    // Connect to the other service:
    const catalog = await cds.connect.to('CatalogService');

    // Call an action on the other service
    await catalog.emit('StockUpdated', {
      productId: item.product_ID,
      addedQuantity: item.quantity
    });
  });

};
```

---

### Direct Database Access Across Services

In practice, within the same project, services often access the database directly rather than going through another service:

```javascript
// srv/sales-service.js
const cds = require('@sap/cds');

module.exports = function () {

  this.on('confirm', 'Orders', async (req) => {
    const { ID } = req.params[0];
    const { SalesOrders, Products, SalesOrderItems } = cds.entities;

    // Directly access Products (even though it's "owned" by CatalogService)
    // This is OK within the same project!
    const items = await SELECT.from(SalesOrderItems).where({ order_ID: ID });

    for (const item of items) {
      const product = await SELECT.one.from(Products).where({ ID: item.product_ID });

      if (product.stock < item.quantity) {
        req.reject(400, `Insufficient stock for ${product.productName}`);
      }

      // Reduce stock
      await UPDATE(Products)
        .set({ stock: product.stock - item.quantity })
        .where({ ID: item.product_ID });
    }

    await UPDATE(SalesOrders).set({ status: 'Confirmed' }).where({ ID });
    return { status: 'Confirmed', message: 'Order confirmed, stock reserved' };
  });

};
```

**Rule of thumb:**
- Same project, same database → Direct DB access is fine
- Different projects or microservices → Use service-to-service calls
- Side effects that shouldn't block → Use events

---

### The @requires Annotation (Authorization Preview)

You can mark services or entities as requiring specific roles:

```cds
// Only authenticated users can access sales
service SalesService @(requires: 'authenticated-user') {
  entity Orders as projection on db.Orders;
}

// Only admins can access admin service
service AdminService @(requires: 'admin') {
  entity Products as projection on db.Products;
  entity Users as projection on db.Users;
}

// Public — no auth required
service CatalogService {
  @readonly entity Products as projection on db.Products;
}
```

**Note:** In development (`cds watch`), auth is disabled by default. We'll cover full authentication in later days. For now, know that `@requires` exists and what it does.

---

### Service Consumption Patterns

How different clients consume your services:

| Consumer | Protocol | Tools |
|----------|----------|-------|
| SAP Fiori UI | OData V4 | Auto-generated from $metadata |
| REST Client / Postman | HTTP/JSON | Manual requests |
| Mobile App | OData or REST | SDK or fetch API |
| Another CAP Service | CDS/JS | `cds.connect.to()` |
| External System | REST/OData | Standard HTTP |
| SAP Build Apps | OData | Service integration |

All of them access the SAME service endpoints — CAP handles everything through OData!

---

### Understanding the Full Request Flow

When a client calls your API, here's what happens internally:

```
Client: POST /sales/Orders  { ... body ... }
         │
         ▼
┌────────────────────────┐
│ 1. CAP Framework       │  Route to SalesService, entity: Orders
│    (routing + parsing) │  Parse JSON body, validate types
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐
│ 2. BEFORE handlers     │  Validate: required fields? correct ranges?
│    (validation)        │  Modify: trim strings, set defaults
│                        │  REJECT if invalid → return 400 error
└──────────┬─────────────┘
           │ (only if all validations pass)
           ▼
┌────────────────────────┐
│ 3. ON handler          │  DEFAULT: Insert into database
│    (execution)         │  CUSTOM: Your replacement logic
│                        │  Generates UUID, fills timestamps
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐
│ 4. AFTER handlers      │  Add computed fields to response
│    (post-processing)   │  Emit events (async notifications)
│                        │  Log the operation
└──────────┬─────────────┘
           │
           ▼
Client: 201 Created  { ... response with all fields ... }
```

---

## Session 3: Hands-on — Peer Review & Code Walkthrough (12:00 - 13:00)

### Peer Review: Service Layer Code

Exchange your service code from Days 16-19 with a partner. Review using this checklist:

---

### Service Layer Review Checklist

#### CDS Definitions
- [ ] Services are split by business domain (not one mega-service)
- [ ] `@(path: '/...')` is defined for clean URLs
- [ ] `@readonly` is used for public/browsing entities
- [ ] Projections hide internal/sensitive fields
- [ ] Bound actions are inside `actions { }` block
- [ ] Functions are marked as `function` (not `action`)
- [ ] Events have meaningful field definitions

#### Handler Code (JavaScript)
- [ ] Handler file name matches CDS file name
- [ ] `before` handlers validate ALL required fields
- [ ] `before` handlers check value ranges
- [ ] `async` is used when DB lookups are needed
- [ ] `req.error()` used for multiple validations
- [ ] `req.reject()` used for critical/blocking failures
- [ ] Error messages are clear and actionable
- [ ] `after` handlers add computed fields correctly
- [ ] Action handlers check status transitions
- [ ] No business logic leaked into CRUD (should be in actions)

#### Testing
- [ ] `.http` test file exists with all scenarios
- [ ] Happy path tested for all CRUD operations
- [ ] Error cases tested (validation failures)
- [ ] Action workflow tested end-to-end
- [ ] Functions tested with various parameters
- [ ] `@readonly` enforcement verified

---

### Common Issues Found in Reviews

| Issue | Severity | Fix |
|-------|----------|-----|
| No validation for required fields | 🔴 Critical | Add `before` CREATE handler |
| Status changed via PATCH (not action) | 🔴 Critical | Use bound actions for workflow |
| All fields exposed in public service | 🟡 Important | Add projection with specific fields |
| Missing `async` on DB-access handler | 🟡 Important | Add `async` and `await` |
| Generic error messages ("Error occurred") | 🟡 Important | Be specific about what failed |
| No `@readonly` on catalog entities | 🟡 Important | Add annotation |
| Functions using POST instead of GET | 🟡 Important | Functions must be read-only (GET) |
| Hardcoded entity strings | 🟢 Nice-to-have | Use `cds.entities` |

---

### Walkthrough Exercise: Trace a Request

As a group, trace what happens for this request through the entire stack:

```http
POST http://localhost:4004/sales/Orders
Content-Type: application/json

{
  "orderNumber": "SO-500",
  "customer_ID": "c1000001-...",
  "orderDate": "2026-06-02",
  "items": [
    { "product_ID": "p1...", "quantity": 3, "unitPrice": 99.99 },
    { "product_ID": "p2...", "quantity": 1, "unitPrice": 499.99 }
  ]
}
```

**Trace the path:**

1. **Routing:** Which service handles this? → _____
2. **Entity:** Which entity is being created? → _____
3. **Before handler:** What validations should run? → _____
4. **Data modification:** What should be auto-calculated? → _____
5. **Database:** What tables get written to? → _____
6. **After handler:** What computed fields should be added? → _____
7. **Response:** What does the client receive? → _____

<details>
<summary>Answers</summary>

1. `SalesService` (path: `/sales`)
2. `SalesOrders` (with Composition → also creates `SalesOrderItems`)
3. Required check: customer_ID, orderDate. Check customer exists. Check products exist. Check stock available. Validate quantities > 0.
4. For each item: `netAmount = quantity × unitPrice`. For order: `netAmount = sum of items`, `taxAmount = netAmount × 0.18`, `grossAmount = net + tax`
5. `SalesOrders` table (1 row) + `SalesOrderItems` table (2 rows) — Deep Insert via Composition
6. Possibly: `statusDisplay`, `priority`, `estimatedDelivery`
7. `201 Created` with the full order including auto-generated `ID`, `createdAt`, `createdBy`, calculated amounts, and both items

</details>

---

## Session 4: End-to-End Integration & Testing (13:45 - 15:00)

### The Integration Challenge

So far we've tested things in isolation. Now let's test the FULL flow end-to-end:

```
Create Master Data → Create Orders → Process Workflow → Check Reports
       │                    │                │                │
       ▼                    ▼                ▼                ▼
   Suppliers           Orders with       Confirm →        Dashboard
   Products            Deep Insert       Ship →           Statistics
   Categories          Items             Deliver          Reports
   Customers           Validations       Events fire
```

---

### End-to-End Test Scenario

**Scenario:** Complete purchase and sales cycle

```
1. Admin creates a supplier
2. Admin creates products (linked to supplier)
3. Customer places an order (creates sales order with items)
4. System validates stock availability
5. Manager confirms the order (stock is reserved)
6. Warehouse ships the order (tracking info added)
7. Customer receives the order (status: Delivered)
8. Stock gets low → Purchase Order is created
9. PO is submitted → approved → received → stock increases
10. Reports show the full activity
```

---

### Complete Integration Test File

**File: `tests/integration-test.http`**

```http
### ═══════════════════════════════════════════════════════
###  END-TO-END INTEGRATION TEST
###  Tests: Master Data → Sales → Purchasing → Reports
### ═══════════════════════════════════════════════════════

### ═══════ PHASE 1: MASTER DATA SETUP ═══════

@admin = http://localhost:4004/admin
@sales = http://localhost:4004/sales
@purchasing = http://localhost:4004/purchasing
@catalog = http://localhost:4004/catalog

### 1.1 Create a Supplier
POST {{admin}}/Suppliers
Content-Type: application/json

{
  "supplierName": "TechParts Inc.",
  "email": "orders@techparts.com",
  "phone": "+1-555-0100",
  "city": "San Jose",
  "isActive": true
}

### 1.2 Create a Category
POST {{admin}}/Categories
Content-Type: application/json

{
  "categoryName": "Electronics",
  "description": "Electronic components and devices"
}

### 1.3 Create Product A (use supplier & category IDs from above)
POST {{admin}}/Products
Content-Type: application/json

{
  "productName": "Wireless Mouse",
  "description": "Ergonomic wireless mouse with USB receiver",
  "price": 29.99,
  "stock": 50,
  "minStock": 20,
  "rating": 4.5,
  "isAvailable": true,
  "supplier_ID": "PASTE-SUPPLIER-ID",
  "category_ID": "PASTE-CATEGORY-ID"
}

### 1.4 Create Product B
POST {{admin}}/Products
Content-Type: application/json

{
  "productName": "Mechanical Keyboard",
  "description": "Cherry MX Blue switches, RGB backlit",
  "price": 89.99,
  "stock": 30,
  "minStock": 10,
  "rating": 4.7,
  "isAvailable": true,
  "supplier_ID": "PASTE-SUPPLIER-ID",
  "category_ID": "PASTE-CATEGORY-ID"
}

### 1.5 Create a Customer
POST {{sales}}/Customers
Content-Type: application/json

{
  "customerName": "Acme Corp",
  "email": "purchasing@acme.com",
  "phone": "+1-555-0200",
  "city": "New York",
  "creditLimit": 100000
}


### ═══════ PHASE 2: VERIFY CATALOG (Public Read) ═══════

### 2.1 Browse products in public catalog
GET {{catalog}}/Products?$select=productName,price,stock&$orderby=productName

### 2.2 Check product with supplier details
GET {{catalog}}/Products?$expand=supplier($select=supplierName),category($select=categoryName)


### ═══════ PHASE 3: SALES ORDER CREATION ═══════

### 3.1 Create a Sales Order with items (Deep Insert)
POST {{sales}}/Orders
Content-Type: application/json

{
  "orderNumber": "SO-INT-001",
  "customer_ID": "PASTE-CUSTOMER-ID",
  "orderDate": "2026-06-02",
  "status": "New",
  "currency_code": "USD",
  "items": [
    {
      "product_ID": "PASTE-PRODUCT-A-ID",
      "quantity": 35,
      "unitPrice": 29.99,
      "currency_code": "USD"
    },
    {
      "product_ID": "PASTE-PRODUCT-B-ID",
      "quantity": 25,
      "unitPrice": 89.99,
      "currency_code": "USD"
    }
  ]
}

### 3.2 Verify order was created with all details
GET {{sales}}/Orders?$filter=orderNumber eq 'SO-INT-001'&$expand=items,customer


### ═══════ PHASE 4: ORDER WORKFLOW ═══════

### 4.1 Confirm the order (bound action)
POST {{sales}}/Orders(PASTE-ORDER-ID)/SalesService.confirm
Content-Type: application/json

{}

### 4.2 Verify status changed
GET {{sales}}/Orders(PASTE-ORDER-ID)?$select=orderNumber,status

### 4.3 Ship the order (bound action)
POST {{sales}}/Orders(PASTE-ORDER-ID)/SalesService.ship
Content-Type: application/json

{
  "trackingNumber": "TRK-FDX-2026-001",
  "carrier": "FedEx"
}

### 4.4 Mark as delivered (bound action)
POST {{sales}}/Orders(PASTE-ORDER-ID)/SalesService.deliver
Content-Type: application/json

{}

### 4.5 Final verification — order is delivered
GET {{sales}}/Orders(PASTE-ORDER-ID)?$select=orderNumber,status


### ═══════ PHASE 5: CHECK STOCK (should be reduced) ═══════

### 5.1 Check product stock (should be reduced after order confirm)
GET {{admin}}/Products(PASTE-PRODUCT-A-ID)?$select=productName,stock,minStock
# Stock was 50, ordered 35 → should be 15 (below minStock of 20!)

### 5.2 Check if low stock alert is needed
GET {{admin}}/Products?$filter=stock lt minStock&$select=productName,stock,minStock


### ═══════ PHASE 6: PURCHASE ORDER (Restock) ═══════

### 6.1 Create PO to restock
POST {{purchasing}}/PurchaseOrders
Content-Type: application/json

{
  "poNumber": "PO-INT-001",
  "supplier_ID": "PASTE-SUPPLIER-ID",
  "orderDate": "2026-06-02",
  "expectedDate": "2026-06-10",
  "status": "Draft",
  "currency_code": "USD"
}

### 6.2 Add items to PO
POST {{purchasing}}/PurchaseOrderItems
Content-Type: application/json

{
  "purchaseOrder_ID": "PASTE-PO-ID",
  "product_ID": "PASTE-PRODUCT-A-ID",
  "quantity": 100,
  "unitPrice": 15.00,
  "currency_code": "USD"
}

### 6.3 Submit PO (bound action)
POST {{purchasing}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.submit
Content-Type: application/json

{}

### 6.4 Approve PO (bound action)
POST {{purchasing}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.approve
Content-Type: application/json

{
  "comment": "Approved - budget available"
}

### 6.5 Receive goods (bound action — updates stock!)
POST {{purchasing}}/PurchaseOrders(PASTE-PO-ID)/PurchasingService.receive
Content-Type: application/json

{
  "notes": "All items received, quality OK"
}


### ═══════ PHASE 7: VERIFY FINAL STATE ═══════

### 7.1 Check stock after PO received (should be higher!)
GET {{admin}}/Products(PASTE-PRODUCT-A-ID)?$select=productName,stock,minStock
# Was 15, received 100 → should be 115 now!

### 7.2 Check order statistics (unbound function)
GET {{sales}}/getOrderStats(year=2026,month=6)

### 7.3 Dashboard
GET {{purchasing}}/getPurchasingDashboard()
```

---

### Performance Testing Basics

When your application is working correctly, consider basic performance:

#### Tip 1: Use `$select` — Don't Fetch Unnecessary Data

```
// SLOW: Fetches ALL fields for ALL products
GET /catalog/Products

// FAST: Only fetches what's needed
GET /catalog/Products?$select=productName,price,stock
```

#### Tip 2: Limit `$expand` Depth

```
// SLOW: Three levels of expansion
GET /sales/Orders?$expand=customer,items($expand=product($expand=supplier,category))

// BETTER: Flatten what you need
GET /sales/Orders?$expand=customer($select=customerName),items($select=quantity,unitPrice;$top=10)
```

#### Tip 3: Always Paginate

```
// DANGEROUS: Could return 10,000 records
GET /catalog/Products

// SAFE: Paginated response
GET /catalog/Products?$top=20&$skip=0&$count=true
```

#### Tip 4: Use Views for Complex Reads

Instead of multiple `$expand` calls, create a flattened view:

```cds
// Pre-joined, pre-filtered, fast!
entity OrderDashboard as select from SalesOrders {
  orderNumber,
  orderDate,
  status,
  grossAmount,
  customer.customerName as customer,
  customer.city as city
};
```

---

### Quick Performance Checklist

- [ ] All list pages use `$top` (never unbounded)
- [ ] Public APIs use `$select` (don't return extra fields)
- [ ] `$expand` goes max 2 levels deep
- [ ] Complex reporting uses views (not deep $expand)
- [ ] Database has appropriate data (CSV files for dev)
- [ ] Handler queries use `.where()` (never load ALL records to filter in JS)

---

## Session 5: Weekly Quiz — Days 16-20 (15:15 - 16:15)

### Instructions
- 30 questions covering Days 16 through 20
- Mix of multiple choice, true/false, and fill-in-the-blank
- Time: 60 minutes
- Passing score: 70% (21/30)

---

### Quiz Questions

**Day 16 — CAP Services Basics**

**Q1.** What does `as projection on` do in a service definition?
- A) Creates a copy of the database table
- B) Exposes a database entity through the service as an API endpoint
- C) Deletes the entity from the database
- D) Renames the entity

---

**Q2.** Given `service PurchaseService {}`, what URL path is generated?
- A) `/purchase-service`
- B) `/purchaseservice`
- C) `/purchase`
- D) `/PurchaseService`

---

**Q3.** What does `@readonly` do on a service entity?
- A) Makes the entity invisible
- B) Allows only GET requests, blocks POST/PUT/PATCH/DELETE
- C) Allows only POST requests
- D) Encrypts the data

---

**Q4.** Fill in: The `$metadata` endpoint returns an _____ document that describes the API schema.

---

**Q5.** How many services can one CAP project have?
- A) Exactly one
- B) Maximum three
- C) As many as needed
- D) One per entity

---

**Day 17 — OData Query Operations**

**Q6.** Which OData option filters records?
- A) `$search`
- B) `$filter`
- C) `$where`
- D) `$query`

---

**Q7.** To get books with price between 10 and 50, the filter is:
- A) `$filter=price between 10 and 50`
- B) `$filter=price ge 10 and price le 50`
- C) `$filter=price > 10 AND price < 50`
- D) `$filter=price in (10,50)`

---

**Q8.** `$top=10&$skip=20&$count=true` — what does `$count` return?
- A) 10 (the page size)
- B) 20 (the skip value)
- C) Total matching records BEFORE pagination
- D) Number of pages

---

**Q9.** Inside `$expand()`, options are separated by:
- A) Ampersand (`&`)
- B) Comma (`,`)
- C) Semicolon (`;`)
- D) Pipe (`|`)

---

**Q10.** To search case-insensitively for "phone" in product names:
- A) `$filter=contains(productName,'phone')`
- B) `$filter=contains(tolower(productName),'phone')`
- C) `$filter=productName like '%phone%'`
- D) `$search=phone`

---

**Day 18 — CRUD & Custom Logic**

**Q11.** What is the difference between PUT and PATCH?
- A) PUT is for create, PATCH is for update
- B) PUT replaces the entire record, PATCH updates only sent fields
- C) PUT is slower, PATCH is faster
- D) No difference

---

**Q12.** Custom handler files must be named:
- A) `handlers.js`
- B) Same as the CDS file with `.js` extension
- C) Any name you want
- D) `index.js`

---

**Q13.** In a `before('CREATE', 'Books', handler)`, `req.data` contains:
- A) The database record
- B) The incoming request body (JSON payload)
- C) The response data
- D) The error messages

---

**Q14.** `req.error(400, 'Price is required', 'price')` — the third parameter identifies:
- A) The HTTP method
- B) The target field that caused the error
- C) The error code
- D) The entity name

---

**Q15.** TRUE or FALSE: Deep Insert (creating parent + children in one POST) works only with Composition.
- A) TRUE
- B) FALSE

---

**Day 19 — Actions, Functions & Events**

**Q16.** Actions use which HTTP method?
- A) GET
- B) POST
- C) PUT
- D) DELETE

---

**Q17.** Functions use which HTTP method?
- A) GET
- B) POST
- C) PATCH
- D) OPTIONS

---

**Q18.** The key difference: Actions can _____ data; Functions are _____.
- A) read; write-only
- B) modify; read-only
- C) delete; create-only
- D) encrypt; decrypt

---

**Q19.** Bound actions are defined where in CDS?
- A) Directly inside `service { }`
- B) In a separate file
- C) Inside the entity's `actions { }` block
- D) In `db/schema.cds`

---

**Q20.** The URL for calling bound action `approve` on PurchaseOrder with ID `po-123`:
- A) `POST /purchasing/approve?id=po-123`
- B) `POST /purchasing/PurchaseOrders(po-123)/PurchasingService.approve`
- C) `GET /purchasing/PurchaseOrders(po-123)/approve`
- D) `POST /purchasing/PurchaseOrders/po-123/approve`

---

**Q21.** Events in CAP are:
- A) Synchronous — waits for all listeners
- B) Asynchronous — fire and forget
- C) Only for errors
- D) Only available in HANA

---

**Q22.** To emit an event from a handler:
- A) `req.emit('EventName', data)`
- B) `this.emit('EventName', data)`
- C) `cds.fire('EventName', data)`
- D) `event.send('EventName', data)`

---

**Day 20 — Best Practices & Integration**

**Q23.** How should you split services in a CAP project?
- A) One service per file
- B) One service per entity
- C) One service per business domain
- D) Only one service per project

---

**Q24.** Which annotation makes an entity write-once (create but no update/delete)?
- A) `@writeonce`
- B) `@insertonly`
- C) `@createonly`
- D) `@immutable`

---

**Q25.** For status changes in a workflow (New → Confirmed → Shipped), you should use:
- A) PATCH with status field (direct update)
- B) Bound actions (confirm, ship, deliver)
- C) DELETE and recreate with new status
- D) SQL UPDATE statement

---

**Q26.** TRUE or FALSE: Within the same CAP project, one service can directly access another service's database entities.
- A) TRUE
- B) FALSE

---

**Q27.** Which handler phase should you use for input validation?
- A) `on`
- B) `after`
- C) `before`
- D) `init`

---

**Q28.** Fill in: For pagination, the formula for $skip is: `$skip = (_____ - 1) × _____`

---

**Q29.** What happens when you call `req.reject(403, 'Forbidden')`?
- A) Logs a warning but continues
- B) Immediately stops processing and returns error to client
- C) Adds to error collection
- D) Retries the request

---

**Q30.** Which is the correct order of handler execution?
- A) after → on → before
- B) on → before → after
- C) before → on → after
- D) before → after → on

---

### Answer Key

<details>
<summary>Click to reveal answers</summary>

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | `as projection on` connects service entity to DB entity (exposes via API) |
| 2 | **C** | Removes "Service" suffix, lowercases → `/purchase` |
| 3 | **B** | `@readonly` blocks all write operations |
| 4 | **XML** | $metadata returns an XML document (OData EDMX format) |
| 5 | **C** | No limit — create as many services as your architecture needs |
| 6 | **B** | `$filter` applies conditions to select matching records |
| 7 | **B** | `ge` (>=) and `le` (<=) for inclusive range |
| 8 | **C** | $count reflects total matching records BEFORE $top/$skip |
| 9 | **C** | Semicolons separate nested options inside $expand parentheses |
| 10 | **B** | `tolower()` converts to lowercase for case-insensitive matching |
| 11 | **B** | PUT = full replace (missing fields → null), PATCH = partial |
| 12 | **B** | `srv/cat-service.cds` → `srv/cat-service.js` |
| 13 | **B** | `req.data` contains the JSON body the client sent |
| 14 | **B** | Third param is `target` — identifies which field has the error |
| 15 | **TRUE** | Deep Insert requires Composition (parent owns children) |
| 16 | **B** | Actions use POST (they can modify data) |
| 17 | **A** | Functions use GET (they're read-only) |
| 18 | **B** | Actions modify data; Functions are read-only |
| 19 | **C** | Bound actions go inside entity's `actions { }` block |
| 20 | **B** | `POST /service/Entity(key)/ServiceName.actionName` |
| 21 | **B** | Events are asynchronous — emitter doesn't wait |
| 22 | **B** | `this.emit('EventName', data)` from within a handler |
| 23 | **C** | One service per business domain (clear boundaries) |
| 24 | **B** | `@insertonly` allows create but blocks update/delete |
| 25 | **B** | Bound actions enforce business rules and status transitions |
| 26 | **TRUE** | Same project can share DB access via `cds.entities` |
| 27 | **C** | `before` handlers validate input before DB operation |
| 28 | **(pageNumber - 1) × pageSize** | e.g., Page 3, 10/page: (3-1)×10 = 20 |
| 29 | **B** | `req.reject()` immediately stops and returns error |
| 30 | **C** | before → on → after (validate → execute → post-process) |

**Scoring:**
- 27-30: Excellent! You've mastered the service layer!
- 21-26: Good! Review the topics where you missed.
- 15-20: Fair. Re-read Days 16-19 and redo hands-on exercises.
- Below 15: Go through each day systematically with practice.

</details>
