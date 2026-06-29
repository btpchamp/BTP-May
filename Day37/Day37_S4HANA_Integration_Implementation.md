# Day 37: SAP S/4HANA Integration — Part 2 (Implementation)

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 36 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Implementing Remote Service Consumption | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Mashup Services — Combining Local & Remote Data | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Build the Sales Order Integration | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Error Handling & Caching Strategies | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Error Handling & Complete Flow | 60 min |
| 16:00 - 16:45 | Session 6: Architecture Quiz & Integration Patterns | 45 min |
| 16:45 - 17:00 | Assessment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Implement a complete remote service call from your CAP application
- Write CAP external service handlers that delegate queries to S/4HANA
- Build mashup services that combine local entities with remote data
- Handle errors gracefully when external systems are unavailable
- Apply caching strategies to reduce calls to remote systems
- Design integration patterns for real-world business scenarios
- Test the end-to-end flow from your Fiori UI through CAP to S/4HANA

---

## Day 36 Recap — Quick Fire (09:00 - 09:15)

1. Side-by-side extension means building on _____ and connecting via _____? → _____
2. The SAP Business Accelerator Hub URL is _____? → _____
3. An EDMX file contains _____? → _____
4. `cds import *.edmx` generates _____? → _____
5. `cds.connect.to('ServiceName')` does _____? → _____
6. In development, external service connects to _____? → _____
7. In production, external service connects via _____? → _____
8. Never _____ the generated CDS files from import? → _____

<details>
<summary>Answers</summary>

1. Building on **BTP** and connecting via **APIs**
2. **https://api.sap.com**
3. **OData API specification** (entity types, properties, keys, navigation)
4. A **.cds** file representing the external API + updates **package.json**
5. **Connects to the external service** (resolves URL from config)
6. The **sandbox** (sandbox.api.sap.com) or mock data
7. A **BTP Destination** (name → URL + credentials)
8. Never **edit** the generated files (re-import if API changes)

</details>

---

## Session 1: Implementing Remote Service Consumption (09:15 - 10:30)

### Recap: The Integration Architecture

```
YOUR CAP APPLICATION                         S/4HANA (via Sandbox)
┌─────────────────────────────┐             ┌──────────────────────┐
│                             │             │                      │
│  srv/sales-integration.cds  │             │  Sales Order API     │
│  srv/sales-integration.js   │             │  Business Partner API│
│                             │             │                      │
│  "Give me Sales Order 1000" │──HTTP/OData─►│  Returns data        │
│                             │◄────────────│                      │
│  "Show it in MY Fiori app"  │             │                      │
│                             │             │                      │
└─────────────────────────────┘             └──────────────────────┘
```

---

### The 4 Patterns for Consuming External Services

```
┌─────────────────────────────────────────────────────────────┐
│          EXTERNAL SERVICE CONSUMPTION PATTERNS                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1️⃣  PASS-THROUGH (Proxy)                                   │
│     Your service simply forwards requests to S/4HANA        │
│     No local data, no transformation                        │
│     Use: "Just expose S/4HANA data in my app"              │
│                                                             │
│  2️⃣  MASHUP (Combine)                                       │
│     Your service combines local data WITH remote data       │
│     PO (local) + BusinessPartner details (remote)           │
│     Use: "Enrich MY data with S/4HANA master data"         │
│                                                             │
│  3️⃣  REPLICATE (Cache)                                      │
│     Copy remote data into your local DB periodically        │
│     Fast reads (no network call), but data may be stale    │
│     Use: "I need this data very frequently"                │
│                                                             │
│  4️⃣  EVENT-DRIVEN (React)                                   │
│     Listen for events from S/4HANA                         │
│     When "PO Created" event fires → do something          │
│     Use: "React to changes in S/4HANA in real-time"        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Today we focus on **Pattern 1 (Pass-Through)** and **Pattern 2 (Mashup)**.

---

### Pattern 1: Pass-Through (Complete Implementation)

**Scenario:** Expose S/4HANA Sales Orders through your own service so your Fiori app can display them.

**Step 1: Import the API (already done in Day 36)**

```
srv/external/
├── API_SALES_ORDER_SRV.edmx
└── API_SALES_ORDER_SRV.cds    (generated)
```

**Step 2: Define your service with the remote entity**

```cds
// srv/sales-integration.cds
using { API_SALES_ORDER_SRV as S4Sales } from './external/API_SALES_ORDER_SRV';

service SalesIntegrationService @(path: '/sales-s4') {

  // Pass-through: expose S/4 Sales Orders
  @readonly entity SalesOrders as projection on S4Sales.A_SalesOrder {
    key SalesOrder,
    SalesOrderType,
    SalesOrganization,
    SoldToParty,
    TotalNetAmount,
    TransactionCurrency,
    SalesOrderDate,
    CreationDate
  };

  // Pass-through: Sales Order Items
  @readonly entity SalesOrderItems as projection on S4Sales.A_SalesOrderItem {
    key SalesOrder,
    key SalesOrderItem,
    Material,
    SalesOrderItemText,
    RequestedQuantity,
    NetAmount,
    TransactionCurrency
  };
}
```

**Step 3: Implement the handler (delegate to remote)**

```javascript
// srv/sales-integration.js
const cds = require('@sap/cds');

module.exports = async function () {

  // Connect to the external S/4HANA service
  const S4Sales = await cds.connect.to('API_SALES_ORDER_SRV');

  // Delegate ALL reads to S/4HANA
  this.on('READ', 'SalesOrders', (req) => {
    return S4Sales.run(req.query);
  });

  this.on('READ', 'SalesOrderItems', (req) => {
    return S4Sales.run(req.query);
  });
};
```

**That's it!** When a user opens `/sales-s4/SalesOrders`, CAP:
1. Receives the OData request
2. Translates it to an S/4HANA OData call
3. Forwards to the sandbox (dev) or real system (prod)
4. Returns the response to the user

---

### How Query Delegation Works

When user calls:
```
GET /sales-s4/SalesOrders?$filter=SoldToParty eq '1000001'&$top=5&$select=SalesOrder,TotalNetAmount
```

CAP translates to:
```
GET https://sandbox.api.sap.com/.../A_SalesOrder?$filter=SoldToParty eq '1000001'&$top=5&$select=SalesOrder,TotalNetAmount
```

**All OData query options pass through:** `$filter`, `$select`, `$orderby`, `$top`, `$skip`, `$expand` — they all work!

---

### Handling $expand Across Services

If you want to expand Sales Order Items when reading Sales Orders:

```javascript
this.on('READ', 'SalesOrders', async (req) => {
  // Check if client requested $expand=to_Item
  const expandItems = req.query.SELECT.columns?.some(
    c => c.ref?.[0] === 'to_Item' || c.expand
  );

  // Forward the query (including expand) to S/4HANA
  return S4Sales.run(req.query);
});
```

Or define the navigation in CDS:
```cds
entity SalesOrders as projection on S4Sales.A_SalesOrder {
  key SalesOrder,
  TotalNetAmount,
  SoldToParty,
  to_Item: redirected to SalesOrderItems  // Enable $expand
};
```

---

### package.json Configuration (Complete)

```json
{
  "cds": {
    "requires": {
      "API_SALES_ORDER_SRV": {
        "kind": "odata-v2",
        "model": "srv/external/API_SALES_ORDER_SRV",
        "[development]": {
          "credentials": {
            "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_SALES_ORDER_SRV",
            "headers": {
              "APIKey": "YOUR-API-KEY"
            }
          }
        },
        "[production]": {
          "credentials": {
            "destination": "S4HANA_SalesOrder",
            "path": "/sap/opu/odata/sap/API_SALES_ORDER_SRV"
          }
        }
      }
    }
  }
}
```

---

## Session 2: Mashup Services — Combining Local & Remote Data (10:45 - 12:00)

### What is a Mashup?

A **mashup** combines data from YOUR local database with data from an EXTERNAL system into a single unified service.

```
MASHUP = Local Data + Remote Data = Unified Experience

Example:
┌─────────────────────┐      ┌─────────────────────┐
│  YOUR LOCAL DB       │      │  S/4HANA (Remote)   │
│  ─────────────────   │      │  ─────────────────  │
│  PurchaseOrders      │      │  BusinessPartners   │
│  ├── poNumber        │      │  ├── BP ID          │
│  ├── supplierBP: "1000"     │  ├── Full Name      │
│  ├── totalAmount     │      │  ├── City           │
│  └── status          │      │  └── Industry       │
└─────────────────────┘      └─────────────────────┘
         │                              │
         └────────── MASHUP ────────────┘
                      │
                      ▼
┌─────────────────────────────────────────┐
│  WHAT THE USER SEES:                     │
│  ┌────────────────────────────────────┐ │
│  │ PO-001 │ TechCorp Int'l │ $5,000 │ │ ← Supplier name from S/4!
│  │ PO-002 │ Office Supplies│ $1,200 │ │
│  └────────────────────────────────────┘ │
│                                         │
│  Local: PO number, amount, status       │
│  Remote: Supplier name, city, industry  │
│  Combined: Rich, complete view!         │
└─────────────────────────────────────────┘
```

---

### Implementing a Mashup — Step by Step

**The data model (local):**
```cds
// db/schema.cds
entity PurchaseOrders : cuid, managed {
  poNumber    : String(20);
  supplierBP  : String(10);     // S/4HANA Business Partner ID!
  totalAmount : Decimal(12,2);
  status      : String(20);
}
```

**The mashup service:**
```cds
// srv/po-mashup-service.cds
using { com.epm as db } from '../db/schema';
using { API_BUSINESS_PARTNER as BPApi } from './external/API_BUSINESS_PARTNER';

service POMashupService @(path: '/po-mashup') {

  // Local entity with a "virtual" supplier info
  entity PurchaseOrders as projection on db.PurchaseOrders;

  // Remote entity exposed for value help / lookup
  @readonly entity Suppliers as projection on BPApi.A_BusinessPartner {
    key BusinessPartner as ID,
    BusinessPartnerFullName as name,
    Industry as industry
  };
}
```

**The mashup handler:**
```javascript
// srv/po-mashup-service.js
const cds = require('@sap/cds');

module.exports = async function () {

  const BPService = await cds.connect.to('API_BUSINESS_PARTNER');
  const { PurchaseOrders } = cds.entities;

  // ═══ READ Suppliers → delegate to S/4HANA ═══
  this.on('READ', 'Suppliers', (req) => {
    return BPService.run(req.query);
  });

  // ═══ AFTER READ POs → enrich with supplier names from S/4 ═══
  this.after('READ', 'PurchaseOrders', async (results) => {
    const pos = Array.isArray(results) ? results : [results];

    // Collect unique supplier BP IDs
    const bpIds = [...new Set(pos.map(po => po.supplierBP).filter(Boolean))];

    if (bpIds.length === 0) return;

    // Fetch ALL needed suppliers from S/4HANA in ONE call (batch!)
    const suppliers = await BPService.run(
      SELECT.from('A_BusinessPartner')
        .columns('BusinessPartner', 'BusinessPartnerFullName', 'Industry')
        .where({ BusinessPartner: { in: bpIds } })
    );

    // Create a lookup map for fast access
    const supplierMap = {};
    for (const s of suppliers) {
      supplierMap[s.BusinessPartner] = s;
    }

    // Enrich each PO with supplier info
    for (const po of pos) {
      const supplier = supplierMap[po.supplierBP];
      if (supplier) {
        po.supplierName = supplier.BusinessPartnerFullName;
        po.supplierIndustry = supplier.Industry;
      }
    }
  });

  // ═══ BEFORE CREATE → validate supplier exists in S/4 ═══
  this.before('CREATE', 'PurchaseOrders', async (req) => {
    if (req.data.supplierBP) {
      const bp = await BPService.run(
        SELECT.one.from('A_BusinessPartner')
          .where({ BusinessPartner: req.data.supplierBP })
      );
      if (!bp) {
        req.reject(400, `Supplier ${req.data.supplierBP} not found in S/4HANA`);
      }
    }
  });
};
```

---

### The Key Mashup Techniques

#### Technique 1: Batch Lookup (Efficient!)

```javascript
// ❌ BAD: One API call per record (N+1 problem!)
for (const po of pos) {
  const supplier = await BPService.run(
    SELECT.one.from('A_BusinessPartner').where({ BusinessPartner: po.supplierBP })
  );
  po.supplierName = supplier?.BusinessPartnerFullName;
}
// If 100 POs → 100 separate API calls! Slow! 🐌

// ✅ GOOD: One batch call for ALL suppliers
const bpIds = pos.map(po => po.supplierBP).filter(Boolean);
const suppliers = await BPService.run(
  SELECT.from('A_BusinessPartner').where({ BusinessPartner: { in: bpIds } })
);
// If 100 POs with 20 unique suppliers → 1 API call! Fast! 🚀
```

---

#### Technique 2: Virtual Fields for Enriched Data

Add virtual fields to your entity (not persisted in DB, filled at runtime):

```cds
// In your entity or service projection:
entity PurchaseOrders : cuid, managed {
  poNumber       : String(20);
  supplierBP     : String(10);
  totalAmount    : Decimal(12,2);
  // Virtual (computed, not in DB):
  virtual supplierName     : String(81);
  virtual supplierIndustry : String(10);
}
```

These fields exist in the API response but NOT in the database. Your `after READ` handler fills them.

---

#### Technique 3: Validation Against Remote System

```javascript
this.before('CREATE', 'PurchaseOrders', async (req) => {
  const { supplierBP } = req.data;

  if (supplierBP) {
    try {
      const bp = await BPService.run(
        SELECT.one.from('A_BusinessPartner')
          .where({ BusinessPartner: supplierBP })
      );

      if (!bp) {
        req.reject(400, `Supplier BP "${supplierBP}" not found in S/4HANA. Please enter a valid Business Partner ID.`);
      }

      // Auto-fill name from S/4HANA
      req.data.supplierName = bp.BusinessPartnerFullName;

    } catch (error) {
      // S/4HANA might be down — handle gracefully
      console.error('S/4HANA unavailable:', error.message);
      req.warn(200, 'Could not verify supplier in S/4HANA. Please verify manually.');
    }
  }
});
```

---

## Session 3: Hands-on — Build the Sales Order Integration (12:00 - 13:00)

### Exercise: Complete Sales Order Integration

**Goal:** Build a service that reads Sales Orders from S/4HANA's sandbox and displays them in your app.

---

**Step 1: Import the Sales Order API (if not done)**

```bash
# Download API_SALES_ORDER_SRV.edmx from api.sap.com
# OR create a simplified mock:
mkdir -p srv/external
```

If you can't download the EDMX, create a simplified external CDS file:

```cds
// srv/external/API_SALES_ORDER_SRV.cds
// (Simplified mock of S/4HANA Sales Order API)
service API_SALES_ORDER_SRV {
  entity A_SalesOrder {
    key SalesOrder           : String(10);
    SalesOrderType           : String(4);
    SoldToParty              : String(10);
    SalesOrganization        : String(4);
    TotalNetAmount           : Decimal(16,3);
    TransactionCurrency      : String(5);
    SalesOrderDate           : Date;
    OverallSDProcessStatus   : String(1);
  }

  entity A_SalesOrderItem {
    key SalesOrder           : String(10);
    key SalesOrderItem       : String(6);
    Material                 : String(40);
    SalesOrderItemText       : String(40);
    RequestedQuantity        : Decimal(15,3);
    NetAmount                : Decimal(16,3);
    TransactionCurrency      : String(5);
  }
}
```

---

**Step 2: Create the service**

```cds
// srv/s4-sales-service.cds
using { API_SALES_ORDER_SRV as S4 } from './external/API_SALES_ORDER_SRV';

service S4SalesService @(path: '/s4-sales') {

  @readonly entity SalesOrders as projection on S4.A_SalesOrder {
    key SalesOrder,
    SalesOrderType,
    SoldToParty,
    TotalNetAmount,
    TransactionCurrency,
    SalesOrderDate,
    OverallSDProcessStatus
  };

  @readonly entity SalesOrderItems as projection on S4.A_SalesOrderItem {
    key SalesOrder,
    key SalesOrderItem,
    Material,
    SalesOrderItemText,
    RequestedQuantity,
    NetAmount
  };
}
```

---

**Step 3: Create the handler**

```javascript
// srv/s4-sales-service.js
const cds = require('@sap/cds');

module.exports = async function () {

  const S4Sales = await cds.connect.to('API_SALES_ORDER_SRV');

  this.on('READ', 'SalesOrders', async (req) => {
    try {
      return await S4Sales.run(req.query);
    } catch (error) {
      console.error('Failed to read from S/4HANA:', error.message);
      req.reject(502, 'S/4HANA Sales Order service is currently unavailable');
    }
  });

  this.on('READ', 'SalesOrderItems', async (req) => {
    try {
      return await S4Sales.run(req.query);
    } catch (error) {
      req.reject(502, 'S/4HANA service unavailable');
    }
  });
};
```

---

**Step 4: Configure in package.json**

```json
{
  "cds": {
    "requires": {
      "API_SALES_ORDER_SRV": {
        "kind": "odata-v2",
        "model": "srv/external/API_SALES_ORDER_SRV",
        "[development]": {
          "kind": "mocked",
          "credentials": {
            "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_SALES_ORDER_SRV"
          }
        }
      }
    }
  }
}
```

For mock mode (no API key), use `"kind": "mocked"` and provide CSV data:

```bash
mkdir -p srv/external/data
```

```csv
# srv/external/data/API_SALES_ORDER_SRV-A_SalesOrder.csv
SalesOrder;SalesOrderType;SoldToParty;TotalNetAmount;TransactionCurrency;SalesOrderDate;OverallSDProcessStatus
1000001;OR;10100001;15000.000;USD;2026-05-15;C
1000002;OR;10100002;8500.000;EUR;2026-05-20;B
1000003;OR;10100001;22000.000;USD;2026-06-01;A
```

---

**Step 5: Test**

```bash
cds watch

# Open:
# http://localhost:4004/s4-sales/SalesOrders
# http://localhost:4004/s4-sales/SalesOrders?$top=3&$select=SalesOrder,TotalNetAmount
```

---

## Session 4: Error Handling & Caching Strategies (13:45 - 14:45)

### Why Error Handling Matters for Remote Calls

Local database calls almost never fail. Remote API calls fail ALL THE TIME:

```
THINGS THAT CAN GO WRONG:
├── S/4HANA system is down (maintenance window)
├── Network timeout (slow response)
├── Authentication expired (token invalid)
├── Rate limited (too many calls per second)
├── API endpoint changed (version update)
├── Data not found (invalid Business Partner ID)
└── Malformed response (unexpected format)

Your app must handle ALL of these gracefully!
```

---

### Error Handling Pattern: try-catch-fallback

```javascript
this.on('READ', 'SalesOrders', async (req) => {
  const S4Sales = await cds.connect.to('API_SALES_ORDER_SRV');

  try {
    // Try to get data from S/4HANA
    const result = await S4Sales.run(req.query);
    return result;

  } catch (error) {

    // Categorize the error
    if (error.status === 401 || error.status === 403) {
      // Authentication issue
      console.error('Auth failed for S/4HANA:', error.message);
      req.reject(502, 'Unable to authenticate with S/4HANA. Please contact admin.');

    } else if (error.code === 'ECONNREFUSED' || error.code === 'ETIMEDOUT') {
      // System unavailable
      console.error('S/4HANA unreachable:', error.message);
      req.reject(503, 'S/4HANA system is currently unavailable. Please try later.');

    } else if (error.status === 404) {
      // Data not found
      return [];  // Return empty result (not an error for the user)

    } else {
      // Unknown error
      console.error('Unexpected S/4HANA error:', error);
      req.reject(500, 'An unexpected error occurred while fetching data.');
    }
  }
});
```

---

### Error Handling for Mashup (Enrichment)

For mashups, external service failures should NOT break your local data:

```javascript
this.after('READ', 'PurchaseOrders', async (results) => {
  const pos = Array.isArray(results) ? results : [results];
  const bpIds = [...new Set(pos.map(po => po.supplierBP).filter(Boolean))];

  if (bpIds.length === 0) return;

  try {
    // Try to enrich with S/4HANA data
    const suppliers = await BPService.run(
      SELECT.from('A_BusinessPartner')
        .where({ BusinessPartner: { in: bpIds } })
    );

    const map = Object.fromEntries(
      suppliers.map(s => [s.BusinessPartner, s])
    );

    for (const po of pos) {
      const s = map[po.supplierBP];
      po.supplierName = s?.BusinessPartnerFullName || '(Name unavailable)';
    }

  } catch (error) {
    // S/4HANA is down — DON'T fail the whole request!
    // Just leave supplier names empty
    console.warn('Could not enrich with S/4HANA data:', error.message);
    for (const po of pos) {
      po.supplierName = '(S/4HANA unavailable)';
    }
  }
});
```

**Key principle:** If enrichment fails, show local data with a placeholder. NEVER crash the entire request because a secondary system is down!

---

### Caching Strategies

Calling S/4HANA for every request is expensive (slow + rate limits). Caching helps:

#### Strategy 1: In-Memory Cache (Simple)

```javascript
// Simple cache with TTL (Time To Live)
const cache = new Map();
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

async function getSupplier(bpService, bpId) {
  const cacheKey = `BP_${bpId}`;
  const cached = cache.get(cacheKey);

  if (cached && Date.now() - cached.timestamp < CACHE_TTL) {
    return cached.data; // Return from cache!
  }

  // Cache miss — fetch from S/4HANA
  const data = await bpService.run(
    SELECT.one.from('A_BusinessPartner').where({ BusinessPartner: bpId })
  );

  // Store in cache
  cache.set(cacheKey, { data, timestamp: Date.now() });
  return data;
}
```

---

#### Strategy 2: Batch + Cache for Lists

```javascript
const supplierCache = new Map();
const CACHE_DURATION = 10 * 60 * 1000; // 10 minutes

this.after('READ', 'PurchaseOrders', async (results) => {
  const pos = Array.isArray(results) ? results : [results];
  const bpIds = [...new Set(pos.map(p => p.supplierBP).filter(Boolean))];

  // Separate cached vs uncached IDs
  const uncachedIds = [];
  const now = Date.now();

  for (const id of bpIds) {
    const cached = supplierCache.get(id);
    if (!cached || now - cached.timestamp > CACHE_DURATION) {
      uncachedIds.push(id);
    }
  }

  // Only fetch uncached ones from S/4HANA
  if (uncachedIds.length > 0) {
    try {
      const freshData = await BPService.run(
        SELECT.from('A_BusinessPartner').where({ BusinessPartner: { in: uncachedIds } })
      );
      for (const bp of freshData) {
        supplierCache.set(bp.BusinessPartner, { data: bp, timestamp: now });
      }
    } catch (e) {
      console.warn('Cache refresh failed:', e.message);
    }
  }

  // Enrich from cache
  for (const po of pos) {
    const cached = supplierCache.get(po.supplierBP);
    po.supplierName = cached?.data?.BusinessPartnerFullName || '(Unknown)';
  }
});
```

---

#### When to Use Which Caching Strategy

| Strategy | Best For | TTL | Complexity |
|----------|---------|-----|-----------|
| No cache | Rarely accessed data, always-fresh required | 0 | None |
| In-memory (Map) | Frequently accessed, small dataset | 5-15 min | Low |
| Local DB replica | Large master data, offline needed | Hours-Days | Medium |
| Event-driven sync | Real-time updates needed | Instant | High |

**For most mashup scenarios:** In-memory cache with 5-10 minute TTL is the sweet spot.

---

### Understanding the Connectivity Service

For production (S/4HANA On-Premise), the **Connectivity Service** + **Cloud Connector** provides the tunnel:

```
BTP (Cloud)                                    Corporate Network
┌──────────────────┐                          ┌──────────────────┐
│  Your CAP App    │                          │  S/4HANA         │
│                  │                          │  (On-Premise)    │
│  destination:    │─── Connectivity ───────► │                  │
│  ProxyType:      │    Service + Cloud       │  /sap/opu/odata/ │
│  OnPremise       │    Connector             │                  │
└──────────────────┘                          └──────────────────┘
```

**You don't need to change your code!** Just change the Destination in BTP Cockpit:
- `ProxyType: Internet` → S/4HANA Cloud (direct HTTPS)
- `ProxyType: OnPremise` → S/4HANA On-Prem (via Cloud Connector tunnel)

---

## Session 5: Hands-on — Error Handling & Complete Flow (15:00 - 16:00)

### Exercise 1: Add Error Handling (20 minutes)

Update your handler with robust error handling:

```javascript
// srv/s4-sales-service.js (updated with error handling)
const cds = require('@sap/cds');

module.exports = async function () {

  const S4Sales = await cds.connect.to('API_SALES_ORDER_SRV');

  this.on('READ', 'SalesOrders', async (req) => {
    try {
      const result = await S4Sales.run(req.query);

      // Add computed status text
      if (Array.isArray(result)) {
        result.forEach(order => {
          order.statusText = getStatusText(order.OverallSDProcessStatus);
        });
      }

      return result;

    } catch (error) {
      handleRemoteError(error, req, 'Sales Orders');
    }
  });

  this.on('READ', 'SalesOrderItems', async (req) => {
    try {
      return await S4Sales.run(req.query);
    } catch (error) {
      handleRemoteError(error, req, 'Sales Order Items');
    }
  });

  // Helper: Map status codes to readable text
  function getStatusText(code) {
    const map = { 'A': 'Open', 'B': 'In Process', 'C': 'Completed' };
    return map[code] || 'Unknown';
  }

  // Helper: Standard error handling for remote calls
  function handleRemoteError(error, req, entityName) {
    const status = error.status || error.statusCode;

    if (status === 401 || status === 403) {
      req.reject(502, `Authentication failed for S/4HANA ${entityName} API.`);
    } else if (error.code === 'ECONNREFUSED' || error.code === 'ETIMEDOUT') {
      req.reject(503, `S/4HANA is unreachable. Please try again later.`);
    } else if (status === 404) {
      return [];
    } else {
      console.error(`Error fetching ${entityName}:`, error.message);
      req.reject(500, `Failed to retrieve ${entityName} from S/4HANA.`);
    }
  }
};
```

---

### Exercise 2: Build the Mashup (20 minutes)

Combine your local POs with S/4HANA Business Partners:

```javascript
// srv/purchasing-service.js (add mashup logic)

const BPService = await cds.connect.to('API_BUSINESS_PARTNER');

// Enrich POs with supplier names after reading
this.after('READ', 'PurchaseOrders', async (results) => {
  const pos = Array.isArray(results) ? results : [results];
  const bpIds = [...new Set(pos.map(p => p.supplierBP).filter(Boolean))];

  if (bpIds.length === 0) return;

  try {
    const suppliers = await BPService.run(
      SELECT.from('A_BusinessPartner')
        .columns('BusinessPartner', 'BusinessPartnerFullName')
        .where({ BusinessPartner: { in: bpIds } })
    );

    const map = {};
    suppliers.forEach(s => { map[s.BusinessPartner] = s.BusinessPartnerFullName; });

    pos.forEach(po => {
      po.supplierName = map[po.supplierBP] || '(Not found)';
    });

  } catch (e) {
    console.warn('BP enrichment failed:', e.message);
    pos.forEach(po => { po.supplierName = '(Unavailable)'; });
  }
});
```

---

### Exercise 3: Test Complete Flow (20 minutes)

```bash
cds watch

# Test 1: Read Sales Orders
GET http://localhost:4004/s4-sales/SalesOrders

# Test 2: Read with filter
GET http://localhost:4004/s4-sales/SalesOrders?$filter=TotalNetAmount gt 10000

# Test 3: Read POs (should show enriched supplier names)
GET http://localhost:4004/purchasing/PurchaseOrders

# Test 4: Simulate failure (change URL in package.json to invalid)
# → Should get 503 "unavailable" error, not a crash

# Test 5: Create PO with invalid BP (should get validation error)
POST http://localhost:4004/purchasing/PurchaseOrders
{ "supplierBP": "INVALID99", "poNumber": "PO-TEST" }
# → Expected: 400 "Supplier INVALID99 not found in S/4HANA"
```

---

## Session 6: Architecture Quiz & Integration Patterns (16:00 - 16:45)

### Architecture Quiz: Design the Integration

For each scenario below, choose the best integration pattern and explain why.

---

**Scenario 1:** Your PO app needs to show the supplier's full name and city on the PO detail page. The data lives in S/4HANA.

<details><summary>Best Pattern</summary>

**Mashup (Pattern 2)** — Enrich your local PO data with BP details from S/4HANA in an `after READ` handler. Use batch lookup for efficiency.

</details>

---

**Scenario 2:** Users need to browse all Sales Orders from S/4HANA within your app (no local copy needed).

<details><summary>Best Pattern</summary>

**Pass-Through (Pattern 1)** — Expose a projection on the external entity. Delegate all reads to S/4HANA. Simple proxy.

</details>

---

**Scenario 3:** Your app needs the complete list of 500 cost centers for dropdown fields. This list changes only once per quarter.

<details><summary>Best Pattern</summary>

**Replicate/Cache (Pattern 3)** — Import cost centers into a local table (or cache heavily). 500 records rarely changing = perfect for replication. Avoids slow API calls on every form load.

</details>

---

**Scenario 4:** When a new Sales Order is created in S/4HANA, your BTP app should automatically generate a delivery schedule.

<details><summary>Best Pattern</summary>

**Event-Driven (Pattern 4)** — Subscribe to the "Sales Order Created" event from S/4HANA. When fired, your app reacts and creates the delivery schedule. No polling needed.

</details>

---

**Scenario 5:** Before creating a PO, validate that the material (product) exists in S/4HANA's product master.

<details><summary>Best Pattern</summary>

**Mashup Validation** — In a `before CREATE` handler, call the Product API to verify existence. Reject if not found. Include error handling for when S/4HANA is unavailable.

</details>

---

## Assessment: MCQ — 15 Questions on S/4HANA Integration

**Q1.** The "pass-through" pattern means:

- A) Copying all S/4HANA data locally
- B) Forwarding client requests directly to S/4HANA (your service acts as a proxy)
- C) Ignoring S/4HANA completely
- D) Modifying S/4HANA data in transit

**Answer: B** — Pass-through = proxy. Your service delegates the query to S/4HANA and returns the result unchanged.

---

**Q2.** A "mashup" service:

- A) Only uses local data
- B) Combines local data with remote data into a unified response
- C) Deletes remote data
- D) Only uses remote data

**Answer: B** — Mashup = local + remote. Example: Your PO (local) + supplier name (from S/4HANA) in one response.

---

**Q3.** `BPService.run(req.query)` does what?

- A) Runs a local database query
- B) Translates the CDS query to an OData call and executes against the external service
- C) Creates a new CDS entity
- D) Validates the query syntax

**Answer: B** — CAP translates the query to OData format and calls the remote service (S/4HANA or sandbox).

---

**Q4.** To avoid the N+1 problem when enriching POs with supplier names:

- A) Make one API call per PO record
- B) Collect all unique supplier IDs, then make ONE batch call with `WHERE IN`
- C) Skip enrichment
- D) Call the API twice

**Answer: B** — Batch lookup: collect IDs → one call with `WHERE IN` → map results. One call instead of N calls.

---

**Q5.** If S/4HANA is down during a mashup enrichment, you should:

- A) Crash the entire request with a 500 error
- B) Return local data with placeholder text (e.g., "(Unavailable)") — graceful degradation
- C) Retry infinitely until it comes back
- D) Delete the local data

**Answer: B** — Graceful degradation: show what you have (local data) with a note that enrichment failed. Never crash.

---

**Q6.** `"kind": "odata-v2"` in external service config means:

- A) The service only supports 2 entities
- B) The remote API uses OData V2 protocol (common for S/4HANA)
- C) Version 2 of your local service
- D) Only 2 requests per second allowed

**Answer: B** — Many S/4HANA APIs use OData V2. CAP handles the V2 ↔ V4 protocol differences automatically.

---

**Q7.** In-memory caching with a 5-minute TTL is best for:

- A) Data that changes every second
- B) Frequently accessed reference data that changes rarely (names, cost centers)
- C) Real-time stock levels
- D) Large binary files

**Answer: B** — Master data (supplier names, cost centers) changes rarely. Cache reduces API calls significantly.

---

**Q8.** The Connectivity Service + Cloud Connector is needed when:

- A) Connecting to SAP HANA Cloud
- B) Connecting to S/4HANA On-Premise (behind a corporate firewall)
- C) Connecting to api.sap.com sandbox
- D) Connecting to GitHub

**Answer: B** — On-premise systems are behind firewalls. Cloud Connector creates a secure tunnel from BTP into the corporate network.

---

**Q9.** Virtual fields in CDS (`virtual supplierName : String`) are:

- A) Stored in the database
- B) Computed at runtime (filled in handlers, not persisted)
- C) Encrypted fields
- D) Auto-generated IDs

**Answer: B** — Virtual fields exist in the API response but NOT in the database. Handlers fill them from external sources.

---

**Q10.** If a `before CREATE` validation against S/4HANA fails with a network error, you should:

- A) Silently create the record anyway
- B) Warn the user but allow creation (with a note to verify later) OR reject based on business criticality
- C) Delete the database
- D) Always reject with 500

**Answer: B** — Business decision: for critical validations (money), reject. For nice-to-have enrichment, warn and allow.

---

**Q11.** `SELECT.from('A_BusinessPartner').where({ BusinessPartner: { in: ['1000','2000'] } })` generates:

- A) A local SQL query
- B) An OData call: `GET ...A_BusinessPartner?$filter=BusinessPartner eq '1000' or BusinessPartner eq '2000'`
- C) A DELETE operation
- D) A file read

**Answer: B** — CAP translates CDS queries to OData filter syntax when targeting external services.

---

**Q12.** The difference between `[development]` and `[production]` profiles for external services:

- A) Different CDS models
- B) Dev uses sandbox URL directly; Prod uses BTP Destination (name → URL resolved at runtime)
- C) Dev is faster
- D) Prod has no authentication

**Answer: B** — Dev = direct sandbox URL (no BTP needed). Prod = destination name (BTP resolves URL + handles auth).

---

**Q13.** When the external API response has different field names than your entity:

- A) It won't work
- B) Use projections with aliases: `BusinessPartnerFullName as name`
- C) Change the S/4HANA API
- D) Use a different database

**Answer: B** — Projections with aliases map external field names to your preferred names.

---

**Q14.** Rate limiting from an external API means:

- A) The API is free to use
- B) Too many requests in a short time get rejected (429 status) — implement caching/throttling
- C) The API is faster
- D) You need more memory

**Answer: B** — APIs often limit requests (e.g., 100/minute). Caching and batching help stay within limits.

---

**Q15.** The best integration pattern for "validate material exists in S/4HANA before creating a PO" is:

- A) Pass-through
- B) before('CREATE') handler that calls the external Product API with try-catch
- C) Event-driven
- D) Full data replication

**Answer: B** — Validation = synchronous check in a before handler. Call the API, reject if not found, handle errors gracefully.

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| 4 Integration Patterns | Pass-through, Mashup, Replicate, Event-driven |
| Pass-through | `BPService.run(req.query)` — delegate everything to S/4HANA |
| Mashup | `after READ` → batch-fetch from S/4HANA → enrich local data |
| Batch Lookup | Collect IDs → one `WHERE IN` call (not N+1!) |
| Virtual Fields | `virtual` in CDS = computed at runtime, not in DB |
| Error Handling | try-catch with graceful degradation (don't crash on remote failure) |
| Caching | In-memory Map with TTL for frequently accessed reference data |
| Connectivity | Cloud Connector for On-Premise systems behind firewalls |
| Validation | before CREATE → check remote → reject or warn |

### The Integration Decision Tree

```
"I need data from S/4HANA..."

├── "Just display it in my app"
│   └── PASS-THROUGH (proxy via delegation)
│
├── "Combine with MY data"
│   └── MASHUP (after READ enrichment + batch lookup)
│
├── "Use it very frequently / need offline"
│   └── REPLICATE (copy to local DB + periodic refresh)
│
└── "React when something changes in S/4HANA"
    └── EVENT-DRIVEN (subscribe to business events)
```

### The One-Liner

> **CAP makes S/4HANA integration feel like querying a local database — `SELECT.from('A_BusinessPartner')` handles all the HTTP, OData, and authentication complexity for you.**

---

### Looking Ahead: Day 38

Tomorrow: **Advanced CAP Topics — Messaging & Events**
- SAP Event Mesh
- Asynchronous messaging between services
- Event-driven architecture patterns
- Reacting to S/4HANA business events

---

*End of Day 37 — You can now build real integrations between your CAP app and S/4HANA! 🔌*
