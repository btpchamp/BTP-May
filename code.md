## Assignment: Complete EPM Data Model

### Due: Start of Day 15

Build the complete Enterprise Procurement Model (EPM) from scratch, demonstrating ALL concepts from Days 12-14.

### Requirements

Create a new CAP project `epm-complete` with namespace `com.epm` containing:

### Part A: Entities (4 marks)

Create these entities with proper types, aspects, and associations:

| Entity | Key Fields | Must Include |
|--------|-----------|-------------|
| **Suppliers** | cuid | name, contact, email, phone, city, country, isActive |
| **Categories** | cuid | name, description, parentCategory (self-reference) |
| **Products** | cuid, managed | name, description, price, currency, stock, minStock, rating, supplier (assoc), category (assoc) |
| **Customers** | cuid, managed | name, email, phone, city, country, creditLimit |
| **SalesOrders** | cuid, managed | orderNumber, customer (assoc), orderDate, amounts, currency, status |
| **SalesOrderItems** | cuid | order (assoc to parent), product (assoc), quantity, unitPrice, netAmount |
| **PurchaseOrders** | cuid, managed | poNumber, supplier (assoc), orderDate, amounts, currency, status |
| **PurchaseOrderItems** | cuid | order (assoc to parent), product (assoc), quantity, unitPrice |

**Relationship rules:**
- SalesOrders → SalesOrderItems = **Composition**
- PurchaseOrders → PurchaseOrderItems = **Composition**
- All other entity connections = **Association**
- Use `Currency` and `Country` from `@sap/cds/common`

---

### Part B: Views & Projections (2 marks)

Create at least 3 views:

1. **ProductCatalog** — Flat view with product name, price, supplier name, category name, stock status (calculated)
2. **OrderReport** — Flat view with order number, customer name, total amount, order date, status
3. **LowStockAlert** — Products where stock ≤ minStock with supplier contact details

Create at least 2 different services:
1. **SalesService** — Customer-facing (maybe hide internal pricing)
2. **AdminService** — Full access to everything
3. **ReportService** — Read-only views

---

### Part C: CSV Data (2 marks)

Provide complete CSV data:
- At least 4 Suppliers
- At least 5 Categories
- At least 8 Products (each with supplier and category)
- At least 4 Customers
- At least 4 Sales Orders with items (test deep insert for more)
- Currency and Country data
- All FK references must be valid!

---

### Part D: Deploy & Verify (2 marks)

1. `cds deploy --to sqlite:db/epm.db` runs without errors
2. `sqlite3` commands show all tables with correct schema
3. Data counts match CSV rows
4. `cds watch` starts without errors
5. Provide a `test.http` file with at least 10 test queries including:
   - `$expand` queries
   - `$filter` and `$orderby` queries
   - Reports with calculated fields
   - One deep insert POST request
