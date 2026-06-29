# Day 36: SAP S/4HANA Integration — Part 1 (API Discovery)

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Week 8 Kickoff & Motivation | 15 min |
| 09:15 - 10:30 | Session 1: Side-by-Side Extensions & Integration Concepts | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: SAP Business Accelerator Hub & API Specifications | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Explore the API Hub & Download Specs | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Consuming External Services in CAP | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Import API & Configure Destination | 60 min |
| 16:00 - 16:45 | Session 6: Remote Service Definition & Generated Files | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain the Side-by-Side extension model and why it's the modern approach
- Navigate the SAP Business Accelerator Hub to discover APIs
- Understand API specifications (OData metadata, OpenAPI/REST)
- Download and import an external service definition into a CAP project
- Understand the generated `.csn` and `.edmx` files
- Configure a destination to connect your CAP app to S/4HANA (or sandbox)
- Define a remote service in CAP that consumes an external API

---

## Week 8 Kickoff & Motivation (09:00 - 09:15)

### Where We Are in the Journey

```
Weeks 1-2:  Foundations       → HTML, JS, Node.js, Express
Weeks 3-4:  CAP Core         → CDS, Services, Handlers, Actions
Week 5:     UI Layer         → Fiori Elements, Annotations, Draft
Weeks 6-7:  DevOps & Deploy  → Git, HANA, Auth, MTA, Work Zone
Week 8:     Integration      → 👈 YOU ARE HERE!
```

### Why Integration Matters

Until now, your app lives in isolation — it manages its OWN data. But in the real enterprise world:

```
YOUR CAP APP (standalone):
┌──────────────────────┐
│  Purchase Orders     │
│  Products            │
│  Suppliers           │
│  (all YOUR data)     │
└──────────────────────┘

REAL ENTERPRISE (integrated):
┌──────────────────────┐     ┌──────────────────────┐
│  YOUR CAP APP        │────►│  SAP S/4HANA         │
│  Purchase Orders     │     │  Business Partners   │
│                      │◄────│  Materials           │
│  "Who is Supplier X?"│     │  Finance Postings    │
│  → Ask S/4HANA!      │     │  (master data)       │
└──────────────────────┘     └──────────────────────┘
```

**Your app doesn't need to store everything.** It can ask S/4HANA: "Give me the business partner details for supplier #1000."

---

## Session 1: Side-by-Side Extensions & Integration Concepts (09:15 - 10:30)

### Two Ways to Extend SAP

```
┌─────────────────────────────────────────────────────────────┐
│          HOW TO EXTEND SAP SYSTEMS                            │
├────────────────────────────┬────────────────────────────────┤
│                            │                                │
│   IN-APP EXTENSION         │   SIDE-BY-SIDE EXTENSION       │
│   (Key User / Classic)     │   (Developer / Modern)         │
│                            │                                │
│   • Changes INSIDE S/4HANA │   • Separate app on BTP       │
│   • Custom fields, logic   │   • Connects via APIs         │
│   • Risk: upgrade issues   │   • No S/4HANA modification   │
│   • Tightly coupled        │   • Loosely coupled           │
│   • Hard to test           │   • Easy to test & deploy     │
│                            │                                │
│   Example:                 │   Example:                     │
│   Add a field to the       │   Build a PO approval app     │
│   standard PO screen       │   that READS data from        │
│   in S/4HANA               │   S/4HANA via API             │
│                            │                                │
└────────────────────────────┴────────────────────────────────┘
```

**CAP = Side-by-Side extension tool.** You build on BTP and CONNECT to S/4HANA via APIs.

---

### Why Side-by-Side is the Future

| Aspect | In-App Extension | Side-by-Side (CAP on BTP) |
|--------|-----------------|--------------------------|
| **Upgrade safety** | May break during S/4HANA updates | Independent — unaffected by S/4 upgrades |
| **Deployment** | Tied to S/4HANA release cycle | Deploy anytime (your own schedule) |
| **Technology** | ABAP only | Any language (Node.js, Java, etc.) |
| **Testing** | Test in S/4HANA system | Test independently (mock APIs) |
| **Team** | Need S/4HANA access | Only need BTP access + API specs |
| **Innovation** | Limited to S/4HANA features | Use any BTP service (AI, ML, events) |

---

### The Integration Pattern: Your App ↔ S/4HANA

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  SAP BTP                              SAP S/4HANA           │
│                                       (On-Premise or Cloud) │
│  ┌──────────────┐                     ┌──────────────────┐ │
│  │  YOUR CAP    │     Destination     │                  │ │
│  │  Application │────────────────────►│  OData APIs      │ │
│  │              │     Service         │  /sap/opu/odata/ │ │
│  │  "Get me     │◄────────────────────│                  │ │
│  │   Business   │     JSON response   │  • Business      │ │
│  │   Partner    │                     │    Partners      │ │
│  │   #1000"     │                     │  • Materials     │ │
│  └──────────────┘                     │  • Sales Orders  │ │
│                                       │  • Finance       │ │
│                                       └──────────────────┘ │
│                                                             │
│  Connection: via BTP Destination Service                    │
│  (handles auth, URL mapping, Cloud Connector if on-premise) │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### What Can You Do with S/4HANA APIs?

| Use Case | API Call | Your App Does |
|----------|---------|--------------|
| Look up a supplier | GET Business Partner | Show supplier name in your PO form |
| Check material stock | GET Material Stock | Verify stock before ordering |
| Create a purchase order | POST Purchase Order | Send a PO to S/4HANA for processing |
| Get price conditions | GET Pricing | Show current prices from S/4HANA |
| Fetch cost centers | GET Cost Center | Dropdown in your approval form |
| Post a finance document | POST Journal Entry | Record financial impact |

---

### The Technology Stack for Integration

```
YOUR CAP APP                              S/4HANA
┌──────────────────┐                     ┌──────────────┐
│ srv/              │                     │ OData API    │
│ ├── my-service.cds│                    │              │
│ ├── my-service.js │                    │ $metadata    │
│ └── external/     │  ← API specs       │ (describes   │
│     └── BP.edmx   │    imported here   │  the API)    │
│                   │                     └──────────────┘
│ package.json:     │
│   requires:       │
│     S4_BP:        │──── BTP Destination ───►  S/4HANA URL
│       kind: odata │     (configured in       + credentials
│       credentials:│      BTP Cockpit)
│         destination│
└──────────────────┘
```

---

## Session 2: SAP Business Accelerator Hub & API Specifications (10:45 - 12:00)

### What is the SAP Business Accelerator Hub?

The **SAP Business Accelerator Hub** (formerly SAP API Business Hub) is a website where SAP publishes ALL its APIs. Think of it as an "App Store for APIs."

**URL:** https://api.sap.com

```
┌─────────────────────────────────────────────────────────────┐
│           SAP BUSINESS ACCELERATOR HUB                        │
│           https://api.sap.com                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  WHAT'S HERE:                                               │
│  • 2000+ APIs from SAP products                             │
│  • API documentation (what each endpoint does)              │
│  • Try-it-out (sandbox environment to test APIs)            │
│  • Download specifications (EDMX, OpenAPI JSON)             │
│  • Code snippets (how to call the API)                      │
│  • Authentication details                                   │
│                                                             │
│  PRODUCTS COVERED:                                          │
│  • SAP S/4HANA Cloud & On-Premise                           │
│  • SAP SuccessFactors (HR)                                  │
│  • SAP Ariba (Procurement)                                  │
│  • SAP Concur (Travel & Expense)                            │
│  • SAP BTP services                                         │
│  • And many more...                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Navigating the API Hub

```
api.sap.com
├── Discover
│   ├── APIs              ← Browse all available APIs
│   ├── Events            ← Event triggers
│   └── Integrations      ← Pre-built integration content
├── Search: "Business Partner"
│   └── Results:
│       ├── Business Partner (A2X) — S/4HANA Cloud
│       ├── Business Partner — S/4HANA On-Premise
│       └── Business Partner Read — SuccessFactors
└── API Details Page:
    ├── Overview (what it does)
    ├── API Reference (endpoints, parameters)
    ├── Try Out (sandbox testing)
    └── Download (EDMX/JSON spec files)
```

---

### Understanding API Specifications

When you find an API, you need its **specification** — a machine-readable file describing all endpoints, fields, and types.

**Two main formats:**

| Format | File Extension | Used For | Example |
|--------|--------------|----------|---------|
| **OData EDMX** | `.edmx` or `.xml` | SAP OData APIs | S/4HANA Business Partner |
| **OpenAPI/Swagger** | `.json` or `.yaml` | REST APIs | Third-party services |

**OData EDMX example (simplified):**
```xml
<edmx:Edmx Version="4.0">
  <edmx:DataServices>
    <Schema Namespace="API_BUSINESS_PARTNER">
      <EntityType Name="A_BusinessPartner">
        <Key>
          <PropertyRef Name="BusinessPartner"/>
        </Key>
        <Property Name="BusinessPartner" Type="Edm.String" MaxLength="10"/>
        <Property Name="BusinessPartnerFullName" Type="Edm.String" MaxLength="81"/>
        <Property Name="BusinessPartnerCategory" Type="Edm.String" MaxLength="1"/>
        <Property Name="Industry" Type="Edm.String" MaxLength="10"/>
        <NavigationProperty Name="to_BusinessPartnerAddress" Type="Collection(...)"/>
      </EntityType>
    </Schema>
  </edmx:DataServices>
</edmx:Edmx>
```

**This tells CAP:** "The Business Partner API has an entity `A_BusinessPartner` with fields `BusinessPartner` (String 10), `BusinessPartnerFullName` (String 81), etc."

---

### Key APIs for Procurement (Your EPM App)

| API Name | What It Provides | Use In Your App |
|----------|-----------------|-----------------|
| **Business Partner** | Customer/Supplier master data | Look up supplier details |
| **Purchase Order** | PO header + items | Create/read POs in S/4HANA |
| **Product** | Material master data | Get product info, descriptions |
| **Purchase Requisition** | PR details | Convert PR to PO |
| **Supplier Invoice** | Invoice documents | Match invoices to POs |
| **Cost Center** | Financial org units | Assign costs in POs |

---

### The Sandbox Environment

The API Hub provides a **sandbox** — a fake S/4HANA system with sample data that you can call without needing a real S/4HANA system:

```
SANDBOX:
  URL: https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER
  API Key: (get from your api.sap.com account)
  Contains: Sample business partners, materials, orders (fake data!)
  Purpose: Development and testing without a real system
```

**This is perfect for learning!** You can test API calls without S/4HANA.

---

### Getting Your API Key

1. Go to https://api.sap.com
2. Login (create account if needed)
3. Click your profile icon → "Settings" → "Show API Key"
4. Copy your API key — you'll use it in requests:

```
Header: APIKey: <your-api-key>
```

---

## Session 3: Hands-on — Explore the API Hub & Download Specs (12:00 - 13:00)

### Exercise 1: Explore the Business Partner API (20 minutes)

1. **Open:** https://api.sap.com
2. **Search:** "Business Partner"
3. **Select:** "Business Partner (A2X)" under SAP S/4HANA Cloud
4. **Browse the API Reference:**
   - How many entity sets are there? _____
   - What is the key field for `A_BusinessPartner`? _____
   - What navigation properties does it have? _____
   - What is the URL pattern for reading one partner? _____

5. **Try it out (Sandbox):**
   - Click "Try Out"
   - Select `GET /A_BusinessPartner`
   - Set `$top=5`
   - Click "Execute"
   - Observe the response — real-looking business partner data!

---

### Exercise 2: Download the API Specification (10 minutes)

1. On the Business Partner API page
2. Click **"API Specification"** (or Download icon)
3. Download the **EDMX** file (`.edmx`)
4. Also download the **JSON** file (OpenAPI format) if available
5. Save both to your Downloads folder

**The EDMX file is what CAP uses to understand the external API.**

---

### Exercise 3: Explore the Purchase Order API (15 minutes)

1. **Search:** "Purchase Order" on api.sap.com
2. **Select:** "Purchase Order" under SAP S/4HANA Cloud
3. **Browse:**
   - Find the entity: `A_PurchaseOrder`
   - What fields does it have? (PurchaseOrder, Supplier, PurchaseOrderDate, etc.)
   - Find the items: `A_PurchaseOrderItem`
   - How are they related? (Navigation property)
4. **Try out:**
   - GET /A_PurchaseOrder?$top=3&$expand=to_PurchaseOrderItem
   - Observe the nested structure (order with items!)

---

### Exercise 4: Test with REST Client (15 minutes)

```http
### Get Business Partners from Sandbox
GET https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER/A_BusinessPartner?$top=5&$select=BusinessPartner,BusinessPartnerFullName,Industry
APIKey: YOUR-API-KEY-HERE

### Get a specific Business Partner
GET https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER/A_BusinessPartner('1000001')
APIKey: YOUR-API-KEY-HERE

### Get Purchase Orders
GET https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_PURCHASEORDER_PROCESS_SRV/A_PurchaseOrder?$top=3
APIKey: YOUR-API-KEY-HERE
```

---

## Session 4: Consuming External Services in CAP (13:45 - 14:45)

### The 3-Step Process

```
Step 1: IMPORT the API specification into your CAP project
Step 2: DEFINE a remote service in CDS that references it
Step 3: CONNECT via destination and USE it in your handlers
```

---

### Step 1: Import the API Specification

```bash
cd ~/projects/po-management

# Create a folder for external service definitions
mkdir -p srv/external

# Copy your downloaded EDMX file here:
cp ~/Downloads/API_BUSINESS_PARTNER.edmx srv/external/

# OR use CAP's import command (preferred!):
cds import srv/external/API_BUSINESS_PARTNER.edmx --as cds
```

**What `cds import` does:**
- Reads the EDMX file
- Generates a CDS file (`.cds`) representing the external entities
- Generates a `.csn` file (compiled CDS in JSON format)
- Adds configuration to `package.json`

---

### What Gets Generated

After import:
```
srv/external/
├── API_BUSINESS_PARTNER.edmx     ← Original spec (keep for reference)
├── API_BUSINESS_PARTNER.cds      ← 🆕 CDS representation of the API
└── API_BUSINESS_PARTNER.csn      ← 🆕 Compiled service definition
```

**Generated `.cds` file (simplified):**
```cds
// srv/external/API_BUSINESS_PARTNER.cds (auto-generated — don't edit!)

service API_BUSINESS_PARTNER {

  entity A_BusinessPartner {
    key BusinessPartner         : String(10);
    BusinessPartnerFullName     : String(81);
    BusinessPartnerCategory     : String(1);
    FirstName                   : String(40);
    LastName                    : String(40);
    Industry                    : String(10);
    IsFemale                    : Boolean;
    IsMale                      : Boolean;
    to_BusinessPartnerAddress   : Association to many A_BusinessPartnerAddress;
  }

  entity A_BusinessPartnerAddress {
    key BusinessPartner         : String(10);
    key AddressID               : String(10);
    CityName                    : String(40);
    Country                     : String(3);
    StreetName                  : String(60);
    PostalCode                  : String(10);
  }
}
```

**This is how CAP "understands" the external API.** It knows the entity names, fields, types, and relationships.

---

### Step 2: Define the Remote Service in package.json

After import, `package.json` gets updated:

```json
{
  "cds": {
    "requires": {
      "API_BUSINESS_PARTNER": {
        "kind": "odata-v2",
        "model": "srv/external/API_BUSINESS_PARTNER",
        "[production]": {
          "credentials": {
            "destination": "S4HANA_BP",
            "path": "/sap/opu/odata/sap/API_BUSINESS_PARTNER"
          }
        },
        "[development]": {
          "credentials": {
            "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER",
            "headers": {
              "APIKey": "<your-api-key>"
            }
          }
        }
      }
    }
  }
}
```

**Profiles:**
- `[development]` → Uses the sandbox directly (for local testing)
- `[production]` → Uses a BTP Destination named "S4HANA_BP"

---

### Step 3: Use the External Service in Your Code

**In your service CDS (expose a "mash-up"):**

```cds
// srv/purchasing-service.cds
using { API_BUSINESS_PARTNER as bp } from './external/API_BUSINESS_PARTNER';

service PurchasingService {
  // Your own entities
  entity PurchaseOrders as projection on db.PurchaseOrders;

  // Expose external entity (pass-through or projection)
  @readonly entity BusinessPartners as projection on bp.A_BusinessPartner {
    key BusinessPartner as ID,
    BusinessPartnerFullName as name,
    Industry as industry
  };
}
```

**In your handler (call the API):**

```javascript
// srv/purchasing-service.js
const cds = require('@sap/cds');

module.exports = async function () {

  // Connect to the external service
  const BPService = await cds.connect.to('API_BUSINESS_PARTNER');

  // Delegate reads to the external API
  this.on('READ', 'BusinessPartners', async (req) => {
    return BPService.run(req.query);
  });

  // Use in another handler (look up supplier)
  this.before('CREATE', 'PurchaseOrders', async (req) => {
    if (req.data.supplierBP) {
      // Call S/4HANA to verify the business partner exists
      const bp = await BPService.run(
        SELECT.one.from('A_BusinessPartner')
          .where({ BusinessPartner: req.data.supplierBP })
      );

      if (!bp) {
        req.reject(400, `Business Partner ${req.data.supplierBP} not found in S/4HANA`);
      }

      // Auto-fill supplier name from S/4HANA
      req.data.supplierName = bp.BusinessPartnerFullName;
    }
  });
};
```

---

### How CAP Calls the External API

When your code runs `BPService.run(SELECT.from('A_BusinessPartner')...)`:

```
1. CAP translates the CDS query into an OData request
2. Reads the destination configuration (URL + auth)
3. Makes an HTTP call to S/4HANA:
   GET https://s4hana-system/sap/opu/odata/sap/API_BUSINESS_PARTNER/A_BusinessPartner('1000')
4. Receives JSON response
5. Maps it back to CDS entity format
6. Returns to your handler
```

**You write CDS queries. CAP handles the HTTP/OData translation!**

---

## Session 5: Hands-on — Import API & Configure Destination (15:00 - 16:00)

### Exercise 1: Import Business Partner API (20 minutes)

```bash
cd ~/projects/po-management

# Create the external services folder
mkdir -p srv/external

# Option A: If you downloaded the EDMX from API Hub:
cp ~/Downloads/API_BUSINESS_PARTNER.edmx srv/external/

# Import it into CAP:
cds import srv/external/API_BUSINESS_PARTNER.edmx --as cds

# Verify generated files:
ls srv/external/
# Should show: .edmx, .cds, possibly .csn

# Check package.json was updated:
cat package.json | grep -A10 "API_BUSINESS_PARTNER"
```

---

### Exercise 2: Configure for Sandbox (Development) (15 minutes)

Update `package.json` for local development with the sandbox:

```json
{
  "cds": {
    "requires": {
      "API_BUSINESS_PARTNER": {
        "kind": "odata-v2",
        "model": "srv/external/API_BUSINESS_PARTNER",
        "[development]": {
          "credentials": {
            "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER",
            "headers": {
              "APIKey": "YOUR-API-KEY-FROM-API-HUB"
            }
          }
        },
        "[production]": {
          "credentials": {
            "destination": "S4HANA_BusinessPartner",
            "path": "/sap/opu/odata/sap/API_BUSINESS_PARTNER"
          }
        }
      }
    }
  }
}
```

---

### Exercise 3: Create a Simple Pass-Through Service (25 minutes)

**Add to your service CDS:**
```cds
// srv/bp-service.cds
using { API_BUSINESS_PARTNER as external } from './external/API_BUSINESS_PARTNER';

service BPService @(path: '/bp') {
  @readonly entity BusinessPartners as projection on external.A_BusinessPartner {
    key BusinessPartner,
    BusinessPartnerFullName,
    BusinessPartnerCategory,
    Industry
  };
}
```

**Add the handler:**
```javascript
// srv/bp-service.js
const cds = require('@sap/cds');

module.exports = async function () {
  const BPApi = await cds.connect.to('API_BUSINESS_PARTNER');

  this.on('READ', 'BusinessPartners', async (req) => {
    return BPApi.run(req.query);
  });
};
```

**Test:**
```bash
cds watch

# Then open:
# http://localhost:4004/bp/BusinessPartners?$top=5
# Should return data from the SAP Sandbox!
```

---

## Session 6: Remote Service Definition & Generated Files (16:00 - 16:45)

### Understanding the Generated Files

| File | Purpose | Edit? |
|------|---------|-------|
| `.edmx` | Original API specification (OData metadata) | NO (reference only) |
| `.cds` | CDS representation of the external service | NO (auto-generated) |
| `.csn` | Compiled CDS (JSON format, used internally by CAP) | NO |

**Rule:** NEVER edit the generated files! If the API changes, re-import the new EDMX.

---

### How CDS Import Maps OData to CDS

| OData Concept | CDS Equivalent |
|--------------|----------------|
| EntityType | entity |
| Property | field (with type) |
| Key | `key` keyword |
| NavigationProperty | Association |
| EntitySet | (exposed via service) |
| Edm.String(10) | String(10) |
| Edm.Decimal | Decimal |
| Edm.DateTime | DateTime |

---

### Production Destination Configuration

For production (BTP), create a destination in the Cockpit:

```
BTP Cockpit → Connectivity → Destinations → New Destination:

Name:            S4HANA_BusinessPartner
Type:            HTTP
URL:             https://my-s4hana.com/sap/opu/odata/sap/API_BUSINESS_PARTNER
Proxy Type:      Internet (or OnPremise with Cloud Connector)
Authentication:  OAuth2SAMLBearerAssertion (or BasicAuthentication)
User:            API_USER
Password:        ***
```

**For S/4HANA On-Premise:** You'd also need the **SAP Cloud Connector** to tunnel from BTP into your corporate network.

---

### The Complete Integration Picture

```
LOCAL DEVELOPMENT:
  Your CAP app → reads package.json [development] profile
               → calls sandbox.api.sap.com directly
               → gets sample data (for testing)

PRODUCTION (BTP):
  Your CAP app → reads package.json [production] profile
               → resolves destination "S4HANA_BusinessPartner"
               → BTP Destination Service provides URL + credentials
               → calls real S/4HANA system
               → gets production data (real business partners!)
```

---

### Mocking the External Service (for Offline Development)

If you don't have an API key or internet:

```bash
# Create mock data for the external service:
mkdir -p srv/external/data

# Create mock CSV: srv/external/data/API_BUSINESS_PARTNER-A_BusinessPartner.csv
echo 'BusinessPartner;BusinessPartnerFullName;Industry
1000001;TechCorp International;TECH
1000002;Office Supplies Ltd;OFFC
1000003;Global Logistics Inc;TRAN' > srv/external/data/API_BUSINESS_PARTNER-A_BusinessPartner.csv
```

Update package.json to use mocked service locally:
```json
"[development]": {
  "kind": "odata-v2-mock"
}
```

CAP will serve mock data from the CSV file when running `cds watch`!

---

## Assessment: MCQ — 15 Questions on API Hub & External Services

**Q1.** "Side-by-Side extension" means:

- A) Modifying the S/4HANA source code directly
- B) Building a separate app on BTP that connects to S/4HANA via APIs
- C) Installing a plugin inside S/4HANA
- D) Copying the S/4HANA database

**Answer: B** — Side-by-side = separate app on BTP, communicating with S/4HANA through APIs. No S/4HANA modification.

---

**Q2.** The SAP Business Accelerator Hub (api.sap.com) provides:

- A) A code editor
- B) API documentation, specifications, sandbox testing, and downloadable specs for SAP products
- C) Cloud deployment services
- D) Database management

**Answer: B** — It's the central catalog for all SAP APIs with docs, try-it-out sandbox, and spec downloads.

---

**Q3.** An EDMX file contains:

- A) Source code
- B) OData API specification — entity types, properties, keys, and navigation properties
- C) CSS styles
- D) Test data

**Answer: B** — EDMX is the OData metadata format. CAP reads it to understand the external API's structure.

---

**Q4.** `cds import API_BUSINESS_PARTNER.edmx --as cds` does what?

- A) Deploys to S/4HANA
- B) Generates a CDS representation of the external API + updates package.json
- C) Creates a database table
- D) Builds the MTAR

**Answer: B** — Import converts the EDMX to CDS so your CAP project can understand and consume the external API.

---

**Q5.** After importing an external service, you should:

- A) Edit the generated .cds file to add your fields
- B) NEVER edit the generated files — they're auto-generated from the spec
- C) Delete the original .edmx file
- D) Move everything to db/ folder

**Answer: B** — Generated files are auto-created. If the API changes, re-import the new spec. Don't manually edit.

---

**Q6.** `cds.connect.to('API_BUSINESS_PARTNER')` in a handler:

- A) Creates a database connection
- B) Connects to the external service (resolves destination or sandbox URL)
- C) Creates a new CDS entity
- D) Sends an email

**Answer: B** — Connects to the configured external service. CAP resolves the URL from package.json (dev) or destination (prod).

---

**Q7.** In production, CAP resolves external service URLs via:

- A) Hardcoded URLs in source code
- B) BTP Destination Service (named destinations with URL + credentials)
- C) DNS lookup
- D) User input

**Answer: B** — Destinations in BTP Cockpit map a name to a URL + auth method. CAP reads them at runtime.

---

**Q8.** The sandbox on api.sap.com is:

- A) A real S/4HANA production system
- B) A test environment with sample data for trying APIs without a real system
- C) A deployment target
- D) A code editor

**Answer: B** — Sandbox = fake S/4HANA with sample data. Perfect for development and learning.

---

**Q9.** To call the sandbox API, you need:

- A) An S/4HANA license
- B) An API Key (from your api.sap.com profile) passed in the request header
- C) A VPN connection
- D) Admin approval for each request

**Answer: B** — The sandbox requires an API Key header: `APIKey: your-key-here`.

---

**Q10.** `kind: "odata-v2"` in package.json means:

- A) The service uses CDS queries
- B) The external API speaks OData V2 protocol (CAP translates accordingly)
- C) It's a local database
- D) It's a HANA connection

**Answer: B** — Many S/4HANA APIs use OData V2. CAP handles protocol translation automatically.

---

**Q11.** The pattern for exposing external data in your service is:

- A) Copy all data to your local database
- B) Create a projection on the external entity and delegate READ to the remote service
- C) Write raw HTTP calls with fetch()
- D) Export data to CSV

**Answer: B** — Project on external entity + delegate reads via `BPService.run(req.query)`. CAP handles the translation.

---

**Q12.** For S/4HANA On-Premise (behind a firewall), you additionally need:

- A) Nothing extra
- B) SAP Cloud Connector (to tunnel from BTP into the corporate network)
- C) A public URL for S/4HANA
- D) Moving S/4HANA to the cloud

**Answer: B** — Cloud Connector creates a secure tunnel from BTP to on-premise systems. Destination ProxyType = "OnPremise".

---

**Q13.** Mocking an external service in development means:

- A) Calling the real system
- B) Providing fake/sample data locally (CSV or mock service) so you can develop without connectivity
- C) Disabling the service
- D) Encrypting the connection

**Answer: B** — Mock = local fake data. Develop offline without needing the real API or API key.

---

**Q14.** The `[development]` and `[production]` sections in package.json exist because:

- A) They're for different programming languages
- B) Different environments need different connection settings (sandbox URL vs real destination)
- C) They define database schemas
- D) They're for testing only

**Answer: B** — Dev uses sandbox/mock; Prod uses real BTP destinations. Same code, different config per environment.

---

**Q15.** The main advantage of using CAP's `cds.connect.to()` over raw HTTP calls:

- A) It's faster
- B) CAP handles OData translation, error mapping, authentication, and query building — you write CDS, not HTTP
- C) It works offline
- D) It's free

**Answer: B** — You write `SELECT.from('A_BusinessPartner')` — CAP converts to `GET /A_BusinessPartner?...` automatically.

---

## Assignment: Research and Document 3 S/4HANA APIs

### Task

Go to https://api.sap.com and research 3 APIs relevant to a procurement/purchasing scenario.

### Deliverables

For each API, document:

| Field | API 1 | API 2 | API 3 |
|-------|-------|-------|-------|
| **API Name** | | | |
| **API Hub URL** | | | |
| **Main Entity** | | | |
| **Key Field(s)** | | | |
| **Useful Fields (list 5-8)** | | | |
| **Navigation Properties** | | | |
| **How You'd Use It in PO App** | | | |
| **OData Version** | V2 / V4 | | |
| **CRUD Supported** | Read-only / Full CRUD | | |

### Suggested APIs to Research

1. **Business Partner** (`API_BUSINESS_PARTNER`) — for supplier lookup
2. **Purchase Order** (`API_PURCHASEORDER_PROCESS_SRV`) — for PO integration
3. **Product** (`API_PRODUCT_SRV`) — for material/product master

### Bonus

- Download the EDMX file for one API
- Import it into your CAP project using `cds import`
- Verify the generated CDS file looks correct

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Side-by-Side Extension | Build on BTP, connect to S/4HANA via APIs (no S/4HANA modification) |
| SAP Business Accelerator Hub | api.sap.com — catalog of 2000+ APIs with docs and sandbox |
| API Specifications | EDMX files describe OData APIs (entities, fields, types, nav properties) |
| Sandbox | Test environment with sample data (API Key needed) |
| `cds import` | Converts EDMX to CDS so CAP understands the external API |
| Remote Service Config | package.json defines URL (dev=sandbox, prod=destination) |
| `cds.connect.to()` | Connects to external service at runtime |
| Delegating Queries | `BPService.run(req.query)` forwards CAP queries as OData calls |
| Destinations | BTP service that maps names to URLs + credentials |
| Mocking | CSV files provide fake data for offline development |

### The Integration Equation

```
EDMX (from API Hub) + cds import + package.json config + handler delegation
= Your CAP app talks to S/4HANA! 🔌
```

### The One-Liner

> **CAP turns complex S/4HANA integration into simple CDS queries — import the spec, connect the destination, and write `SELECT.from('A_BusinessPartner')` like it's a local entity.**

---

### Looking Ahead: Day 37

Tomorrow: **SAP S/4HANA Integration — Part 2 (Implementation)**
- Implement the Business Partner lookup in your PO app
- Mash-up: combine local data with remote data
- Handle errors from external services
- Caching strategies for remote data

---

*End of Day 36 — You can now discover, import, and connect to any SAP API!*
