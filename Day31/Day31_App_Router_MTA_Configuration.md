# Day 31: App Router & MTA Configuration

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 30 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: What is the Application Router & Why We Need It | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: xs-app.json — Routes & Destinations | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Add Approuter to Your CAP Project | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: MTA — Multi-Target Application Architecture | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: mta.yaml Deep Dive — Modules, Resources, Parameters | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Generate & Customize mta.yaml | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain what the Application Router (approuter) is and why it's essential
- Configure `xs-app.json` to route requests between UI and backend
- Understand routes, destinations, and authentication flow through the approuter
- Explain the MTA (Multi-Target Application) concept and why it exists
- Read and understand every section of an `mta.yaml` file
- Generate `mta.yaml` using `cds add mta` and customize it
- Identify module types (nodejs, html5, deployer) and resource types (xsuaa, hdi-container)

---

## Day 30 Recap — Quick Fire (09:00 - 09:15)

1. Row-level security ensures users only see _____? → _____
2. `where: 'createdBy = $user'` does what? → _____
3. `$user.attr.Department` accesses what? → _____
4. A common pitfall: trusting _____ instead of `req.user.id`? → _____
5. Separation of duties means _____? → _____
6. New entities without @restrict are _____? → _____
7. 401 = _____, 403 = _____? → _____
8. The 6 security layers are? → _____

<details>
<summary>Answers</summary>

1. Their **own records** (filtered by identity/attributes)
2. Filters so users **only see records they created**
3. A **user attribute** (like department name) from xs-security.json
4. Trusting **client-sent data** (`req.data.createdBy`) instead of server-validated token
5. A user **cannot approve their own** PO/request (different person must review)
6. **Open to all authenticated users** for all CRUD operations (security hole!)
7. 401 = **Not authenticated**; 403 = **Authenticated but not authorized**
8. Authentication → Service-level → Entity-level → Row-level → Instance-based → Field-level

</details>

---

## Session 1: What is the Application Router & Why We Need It (09:15 - 10:30)

### The Problem: Multiple Apps, One Entry Point

In production, your CAP application has multiple components:
- A backend service (Node.js running OData)
- A Fiori UI app (static HTML/JS files)
- An authentication service (XSUAA)
- A database (HANA Cloud)

**How does the user's browser know where to send each request?**

```
❌ WITHOUT APPROUTER:
Browser must know every service URL:
  "API is at https://service-abc123.cfapps.us10.hana.ondemand.com"
  "UI is at https://ui-def456.cfapps.us10.hana.ondemand.com"
  "Auth is at https://auth-ghi789.authentication.us10.hana.ondemand.com"

User would need to manage tokens manually... 😱
Cross-origin (CORS) issues everywhere...
No single login experience...
```

```
✅ WITH APPROUTER:
Browser talks to ONE URL:
  "Everything goes to https://my-app.cfapps.us10.hana.ondemand.com"

Approuter figures out where to send each request:
  /catalog/Products → forward to backend service
  /index.html      → serve from UI module
  /login           → redirect to XSUAA

Single entry point. Single login. No CORS. Clean! 🎉
```

---

### What is the Application Router?

The **Application Router** (approuter) is a small Node.js application that acts as a **reverse proxy** and **authentication gateway** for your entire application.

```
┌─────────────────────────────────────────────────────────────┐
│                APPLICATION ROUTER                             │
│               (The Front Door)                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Browser → https://my-app.cfapps.us10.hanacloud.com         │
│               │                                             │
│               ▼                                             │
│  ┌─────────────────────────────────────────────────┐        │
│  │          APPROUTER                               │        │
│  │                                                  │        │
│  │  1. Handles AUTHENTICATION (login/logout)        │        │
│  │  2. Manages user SESSION (cookies/tokens)        │        │
│  │  3. ROUTES requests to the right destination     │        │
│  │  4. Injects AUTH TOKEN into backend requests     │        │
│  │  5. Serves STATIC UI files                       │        │
│  │                                                  │        │
│  │     /api/* ──────► Backend (CAP service)         │        │
│  │     /app/* ──────► Static UI files               │        │
│  │     /login ──────► XSUAA (authentication)        │        │
│  │                                                  │        │
│  └─────────────────────────────────────────────────┘        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Why Do We Need the Approuter?

| Without Approuter | With Approuter |
|-------------------|---------------|
| Each component has a separate URL | Single URL for everything |
| CORS errors between UI and backend | No CORS (same origin) |
| User must login to each component | Single Sign-On (login once) |
| Token management is manual | Approuter handles tokens automatically |
| UI can't reach backend securely | Approuter forwards with auth token |
| Hard to deploy consistently | Deploy as one unit (MTA) |

---

### The Approuter's 5 Responsibilities

```
1️⃣  AUTHENTICATION GATEWAY
    User hits the app → not logged in? → redirect to XSUAA login page
    After login → store session cookie → user is authenticated

2️⃣  SESSION MANAGEMENT
    Maintains user sessions (cookies)
    Refreshes expired tokens automatically
    Handles logout

3️⃣  REQUEST ROUTING
    /api/catalog/Products → forward to CAP backend
    /app/products/index.html → serve static file
    Based on rules in xs-app.json

4️⃣  TOKEN INJECTION
    Before forwarding to backend, adds:
    Authorization: Bearer <JWT-token>
    Backend receives the token → knows who the user is

5️⃣  STATIC FILE SERVING
    Serves HTML, JS, CSS files for Fiori UI
    (Or proxies to HTML5 repository)
```

---

### The Request Flow — Step by Step

```
Step 1: User opens https://my-app.cfapps.us10.hana.ondemand.com
           │
           ▼
Step 2: Approuter checks: "Is user logged in?"
        ├── NO → Redirect to XSUAA login page
        │         User enters credentials
        │         XSUAA validates → issues token → redirects back
        │
        └── YES → Continue to Step 3

Step 3: Approuter receives the request path:
        GET /catalog/Products
           │
           ▼
Step 4: Approuter checks xs-app.json routes:
        Route: "/catalog" → destination: "srv-api"
           │
           ▼
Step 5: Approuter forwards request to backend:
        GET https://backend-service-url/catalog/Products
        Authorization: Bearer eyJhbGciOi... (JWT injected!)
           │
           ▼
Step 6: CAP backend receives request + validates JWT
        Returns data to approuter → approuter returns to browser
```

---

### Where Does the Approuter Live?

```
my-cap-project/
├── db/                    ← Database layer
├── srv/                   ← Service layer
├── app/                   ← UI layer (Fiori apps)
│   ├── products/          ← Product management UI
│   └── orders/            ← Order management UI
├── approuter/             ← 🆕 APPLICATION ROUTER
│   ├── package.json       ← Approuter dependencies
│   └── xs-app.json        ← Routing rules
├── mta.yaml               ← 🆕 Deployment descriptor
├── xs-security.json       ← Security configuration
└── package.json           ← Root project
```

---

## Session 2: xs-app.json — Routes & Destinations (10:45 - 12:00)

### What is xs-app.json?

`xs-app.json` is the configuration file for the approuter. It defines:
- Which URL patterns go where (routes)
- What authentication is needed for each route
- Which destinations exist (backend services)

---

### Basic xs-app.json Structure

```json
{
  "welcomeFile": "/app/products/webapp/index.html",
  "authenticationMethod": "route",
  "routes": [
    {
      "source": "^/catalog/(.*)$",
      "target": "/catalog/$1",
      "destination": "srv-api",
      "authenticationType": "xsuaa"
    },
    {
      "source": "^/purchasing/(.*)$",
      "target": "/purchasing/$1",
      "destination": "srv-api",
      "authenticationType": "xsuaa"
    },
    {
      "source": "^/app/(.*)$",
      "target": "$1",
      "localDir": "resources",
      "authenticationType": "xsuaa"
    }
  ]
}
```

---

### Breaking Down Each Part

#### `welcomeFile`
```json
"welcomeFile": "/app/products/webapp/index.html"
```
The first page shown when a user opens the app without a specific path.

---

#### `authenticationMethod`
```json
"authenticationMethod": "route"
```
- `"route"` → Each route defines its own auth requirement (most common)
- `"none"` → No authentication for any route (for public apps)

---

#### Routes — The Routing Rules

Each route is a rule: "If the URL matches THIS pattern → send it THERE."

```json
{
  "source": "^/catalog/(.*)$",
  "target": "/catalog/$1",
  "destination": "srv-api",
  "authenticationType": "xsuaa"
}
```

| Property | Meaning | Example |
|----------|---------|---------|
| `source` | URL pattern to match (regex) | `^/catalog/(.*)$` matches `/catalog/Products` |
| `target` | Where to forward (rewrite path) | `/catalog/$1` → `/catalog/Products` |
| `destination` | Which backend to send it to | `srv-api` (defined in mta.yaml) |
| `authenticationType` | Auth requirement | `xsuaa` (login required) or `none` |
| `localDir` | Serve from local folder (static files) | `resources` folder |

---

### Understanding Route Patterns (Regex)

```
"source": "^/catalog/(.*)$"

^        → Start of string
/catalog → Literal match
/(.*)    → Capture everything after /catalog/
$        → End of string

Examples that MATCH:
  /catalog/Products         → $1 = "Products"
  /catalog/Products(uuid)   → $1 = "Products(uuid)"
  /catalog/$metadata        → $1 = "$metadata"

"target": "/catalog/$1"
  → Forwards: /catalog/Products → /catalog/Products (same path to backend)
```

---

### Common Route Configurations

```json
{
  "routes": [
    // Route 1: Backend API (authenticated)
    {
      "source": "^/api/(.*)$",
      "target": "$1",
      "destination": "srv-api",
      "authenticationType": "xsuaa",
      "csrfProtection": true
    },

    // Route 2: Static UI files (authenticated)
    {
      "source": "^/app/(.*)$",
      "target": "$1",
      "localDir": "resources",
      "authenticationType": "xsuaa"
    },

    // Route 3: Public health check (no auth)
    {
      "source": "^/health$",
      "target": "/health",
      "destination": "srv-api",
      "authenticationType": "none"
    },

    // Route 4: Catch-all (serve index.html for SPA routing)
    {
      "source": "^(.*)$",
      "target": "$1",
      "localDir": "resources",
      "authenticationType": "xsuaa"
    }
  ]
}
```

**Route order matters!** Routes are checked top-to-bottom. First match wins.

---

### Destinations — Where Backends Live

A **destination** is a named reference to a backend service URL. Defined in `mta.yaml` (not in xs-app.json):

```
xs-app.json says:  "destination": "srv-api"
mta.yaml defines:  srv-api → https://my-service.cfapps.us10.hana.ondemand.com
```

This way, xs-app.json doesn't hardcode URLs — they're resolved at deployment time.

---

### Authentication Flow Through the Approuter

```
Browser                    Approuter                   XSUAA                    CAP Backend
  │                           │                         │                          │
  │  GET /catalog/Products    │                         │                          │
  │──────────────────────────►│                         │                          │
  │                           │  (No session cookie)    │                          │
  │  ◄─── 302 Redirect ──────│                         │                          │
  │                           │                         │                          │
  │  GET /oauth/authorize     │                         │                          │
  │───────────────────────────────────────────────────►│                          │
  │                           │                         │  (Login page shown)      │
  │  POST /oauth/token (credentials)                   │                          │
  │───────────────────────────────────────────────────►│                          │
  │                           │                         │  ✅ Validated!           │
  │  ◄──── JWT Token + Redirect ────────────────────── │                          │
  │                           │                         │                          │
  │  GET /catalog/Products    │                         │                          │
  │  (with session cookie)    │                         │                          │
  │──────────────────────────►│                         │                          │
  │                           │  Forward + JWT token    │                          │
  │                           │─────────────────────────────────────────────────►  │
  │                           │                         │                          │
  │                           │  ◄──────── Data ────────────────────────────────── │
  │  ◄──────── Data ─────────│                         │                          │
  │                           │                         │                          │
```

---

## Session 3: Hands-on — Add Approuter to Your CAP Project (12:00 - 13:00)

### Step 1: Add Approuter Configuration

```bash
cd ~/projects/po-management

# CAP generates the approuter setup:
cds add approuter
```

This creates:
```
app/
└── router/               ← OR approuter/ (depending on version)
    ├── package.json      ← Dependencies for @sap/approuter
    └── xs-app.json       ← Routing configuration
```

---

### Step 2: Review Generated xs-app.json

```bash
cat app/router/xs-app.json
```

Typical generated content:
```json
{
  "welcomeFile": "/index.html",
  "authenticationMethod": "route",
  "routes": [
    {
      "source": "^/purchasing/(.*)$",
      "target": "/purchasing/$1",
      "destination": "po-management-srv",
      "authenticationType": "xsuaa",
      "csrfProtection": true
    },
    {
      "source": "^/catalog/(.*)$",
      "target": "/catalog/$1",
      "destination": "po-management-srv",
      "authenticationType": "xsuaa"
    },
    {
      "source": "^(.*)$",
      "target": "$1",
      "service": "html5-apps-repo-rt",
      "authenticationType": "xsuaa"
    }
  ]
}
```

---

### Step 3: Review Approuter package.json

```bash
cat app/router/package.json
```

```json
{
  "name": "po-management-approuter",
  "dependencies": {
    "@sap/approuter": "^14"
  },
  "scripts": {
    "start": "node node_modules/@sap/approuter/approuter.js"
  }
}
```

The approuter is just the `@sap/approuter` npm package with your `xs-app.json` config!

---

### Step 4: Test Locally (Simplified)

For local development, `cds watch` handles routing for you automatically. The approuter is mainly needed in production (Cloud Foundry deployment).

To test the approuter locally:
```bash
# Install approuter dependencies
cd app/router
npm install
cd ../..

# Run everything together
cds watch
```

CAP's development server mimics the approuter behavior locally — you don't need to run the actual approuter for development.

---

## Session 4: MTA — Multi-Target Application Architecture (13:45 - 14:45)

### The Deployment Problem

Your CAP app has MULTIPLE components that need to be deployed together:

```
┌─────────────────────────────────────────────────────────────┐
│  YOUR APPLICATION = 5+ COMPONENTS                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ① Approuter (Node.js app — routes & auth)                  │
│  ② Backend Service (Node.js app — CAP + OData)              │
│  ③ Database Deployer (deploys schema to HANA)               │
│  ④ UI Content (static HTML/JS/CSS files)                    │
│  ⑤ XSUAA Service Instance (security)                        │
│  ⑥ HANA HDI Container (database)                            │
│  ⑦ Destination Service (URL registry)                       │
│                                                             │
│  ALL of these must be deployed TOGETHER and CONNECTED!       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Without MTA:** Deploy each piece manually, configure connections manually, pray nothing breaks.

**With MTA:** One file (`mta.yaml`) describes everything. One command (`cf deploy`) deploys ALL of it.

---

### What is MTA?

**MTA** = **Multi-Target Application**

It's a packaging and deployment format that:
- Describes ALL components of your app in ONE file (`mta.yaml`)
- Defines how components depend on each other
- Handles service creation (HANA, XSUAA) automatically
- Deploys everything in the right order
- Binds services to apps automatically

```
┌─────────────────────────────────────────────────────────────┐
│                MTA = Your Deployment Blueprint                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  mta.yaml says:                                             │
│    "Create a HANA database container"                       │
│    "Deploy the schema to it"                                │
│    "Create an XSUAA service instance"                       │
│    "Deploy the backend, connect it to HANA and XSUAA"       │
│    "Deploy the approuter, connect it to XSUAA and backend"  │
│    "Deploy the UI files"                                    │
│                                                             │
│  One command: cf deploy → EVERYTHING is set up!             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### MTA Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Cloud Foundry                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  MODULES (deployed apps):            RESOURCES (services):  │
│                                                             │
│  ┌──────────────┐                   ┌──────────────┐       │
│  │  Approuter   │───────────────────│   XSUAA      │       │
│  │  (router)    │                   │   Service    │       │
│  └──────┬───────┘                   └──────────────┘       │
│         │ routes to                                         │
│  ┌──────▼───────┐                   ┌──────────────┐       │
│  │  CAP Backend │───────────────────│   HDI        │       │
│  │  (srv)       │                   │   Container  │       │
│  └──────────────┘                   └──────────────┘       │
│                                                             │
│  ┌──────────────┐                   ┌──────────────┐       │
│  │  DB Deployer │───────────────────│   HDI        │       │
│  │  (db)        │                   │   Container  │       │
│  └──────────────┘                   └──────────────┘       │
│                                                             │
│  ┌──────────────┐                   ┌──────────────┐       │
│  │  UI Content  │───────────────────│   HTML5 Repo │       │
│  │  (app)       │                   │   Service    │       │
│  └──────────────┘                   └──────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Modules vs Resources

| Concept | What It Is | Examples |
|---------|-----------|---------|
| **Modules** | YOUR code that gets deployed (runs) | Backend app, approuter, DB deployer, UI |
| **Resources** | SERVICES your modules depend on (provided by BTP) | HANA database, XSUAA, Destination service |

**Analogy:**
- **Modules** = Rooms in your house (you build them)
- **Resources** = Utilities (electricity, water, internet — provided by the city)
- **mta.yaml** = The architectural blueprint showing how rooms connect to utilities

---

## Session 5: mta.yaml Deep Dive — Modules, Resources, Parameters (15:00 - 16:00)

### Generating mta.yaml

```bash
cds add mta
```

This creates `mta.yaml` in your project root.

---

### Complete mta.yaml — Annotated Line by Line

```yaml
# ═══════════════════════════════════════════════════════
# MTA Deployment Descriptor for PO Management App
# ═══════════════════════════════════════════════════════

_schema-version: "3.1"            # MTA specification version
ID: po-management                  # Unique ID for your app
version: 1.0.0                     # Your app version
description: "Purchase Order Management CAP Application"

parameters:
  enable-parallel-deployments: true   # Deploy modules in parallel (faster)

# ═══════════════════════════════════════════════════════
# BUILD PARAMETERS — How to build the project
# ═══════════════════════════════════════════════════════
build-parameters:
  before-all:
    - builder: custom
      commands:
        - npm ci                  # Install root dependencies
        - npx cds build --production  # Build CDS to HANA artifacts

# ═══════════════════════════════════════════════════════
# MODULES — Your deployable applications
# ═══════════════════════════════════════════════════════
modules:

  # ─── MODULE 1: Database Deployer ─────────────────────
  # Deploys schema + data to HANA HDI Container
  - name: po-management-db-deployer
    type: hdb                         # Type: HANA database deployer
    path: gen/db                      # Path to generated DB artifacts
    parameters:
      buildpack: nodejs_buildpack     # Uses Node.js to run deployer
    requires:
      - name: po-management-db        # Needs the HDI container (resource)

  # ─── MODULE 2: Backend Service (CAP) ─────────────────
  # Your Node.js CAP application (OData services)
  - name: po-management-srv
    type: nodejs                      # Type: Node.js application
    path: gen/srv                     # Path to generated service code
    parameters:
      buildpack: nodejs_buildpack
      memory: 256M                    # RAM allocation
      disk-quota: 1024M
    requires:
      - name: po-management-db        # Needs HANA database
      - name: po-management-auth      # Needs XSUAA (authentication)
    provides:
      - name: srv-api                 # Provides an API endpoint
        properties:
          srv-url: ${default-url}     # "My URL is available as 'srv-api'"

  # ─── MODULE 3: Application Router ───────────────────
  # Entry point for users (handles auth + routing)
  - name: po-management-approuter
    type: approuter.nodejs            # Type: Application Router
    path: app/router                  # Path to approuter config
    parameters:
      memory: 256M
      disk-quota: 512M
    requires:
      - name: po-management-auth      # Needs XSUAA
      - name: srv-api                 # Needs to know backend URL
        group: destinations
        properties:
          name: srv-api               # Destination name (used in xs-app.json)
          url: ~{srv-url}             # URL from backend module
          forwardAuthToken: true      # Pass JWT to backend!

  # ─── MODULE 4: UI Deployer (optional) ───────────────
  # Deploys Fiori UI files to HTML5 repository
  - name: po-management-ui-deployer
    type: com.sap.application.content
    path: app/
    requires:
      - name: po-management-html5-repo-host
    build-parameters:
      build-result: resources/
      requires:
        - name: po-management-products
          artifacts:
            - "./*"
          target-path: resources/products/

# ═══════════════════════════════════════════════════════
# RESOURCES — BTP services your app depends on
# ═══════════════════════════════════════════════════════
resources:

  # ─── RESOURCE 1: HANA HDI Container ─────────────────
  # The database container for your tables
  - name: po-management-db
    type: com.sap.xs.hdi-container    # Type: HDI Container
    parameters:
      service: hana                   # BTP service name
      service-plan: hdi-shared        # Shared plan (trial-compatible)

  # ─── RESOURCE 2: XSUAA Service ──────────────────────
  # Authentication and authorization
  - name: po-management-auth
    type: org.cloudfoundry.managed-service
    parameters:
      service: xsuaa                  # BTP service name
      service-plan: application       # Application plan
      path: ./xs-security.json        # Your security config file!
      config:
        xsappname: po-management-${org}-${space}  # Unique name
        tenant-mode: dedicated

  # ─── RESOURCE 3: HTML5 Repository ───────────────────
  # Stores Fiori UI files
  - name: po-management-html5-repo-host
    type: org.cloudfoundry.managed-service
    parameters:
      service: html5-apps-repo
      service-plan: app-host

  # ─── RESOURCE 4: Destination Service (optional) ─────
  - name: po-management-destination
    type: org.cloudfoundry.managed-service
    parameters:
      service: destination
      service-plan: lite
```

---

### Understanding the Key Concepts

#### `requires` — "I need this to work"

```yaml
# Backend REQUIRES database and auth:
- name: po-management-srv
  requires:
    - name: po-management-db      # "I need the database"
    - name: po-management-auth    # "I need authentication"
```

When deployed, Cloud Foundry automatically BINDS these services to the module (provides connection details via environment variables).

---

#### `provides` — "I offer this to others"

```yaml
# Backend PROVIDES an API URL:
- name: po-management-srv
  provides:
    - name: srv-api
      properties:
        srv-url: ${default-url}   # "Here's my URL for others to use"
```

---

#### `requires` + `provides` = Wiring

```yaml
# Approuter REQUIRES the backend's URL:
- name: po-management-approuter
  requires:
    - name: srv-api               # "Give me the backend URL"
      group: destinations
      properties:
        name: srv-api
        url: ~{srv-url}           # ~{} = read from provider
        forwardAuthToken: true
```

**Flow:** Backend provides `srv-url` → Approuter consumes it as a destination → xs-app.json references it by name.

---

### Module Types Explained

| Type | What It Is | Example |
|------|-----------|---------|
| `nodejs` | A Node.js application | CAP backend server |
| `approuter.nodejs` | The Application Router | Request routing + auth |
| `hdb` | HANA Database deployer | Deploys tables/views to HDI |
| `com.sap.application.content` | Static content deployer | UI files to HTML5 repo |
| `html5` | HTML5 application | Fiori app (alternative to above) |

---

### Resource Types Explained

| Type | Service | What It Provides |
|------|---------|-----------------|
| `com.sap.xs.hdi-container` | hana | An isolated database container |
| `org.cloudfoundry.managed-service` (xsuaa) | xsuaa | Authentication/authorization |
| `org.cloudfoundry.managed-service` (html5-apps-repo) | html5-apps-repo | Static file hosting |
| `org.cloudfoundry.managed-service` (destination) | destination | URL registry service |

---

### The Build → Deploy Flow

```
1. You run: mbt build
   → Reads mta.yaml
   → Builds all modules (npm install, cds build)
   → Creates: mta_archives/po-management_1.0.0.mtar (archive file)

2. You run: cf deploy mta_archives/po-management_1.0.0.mtar
   → Uploads the archive to Cloud Foundry
   → Creates resources (HANA container, XSUAA instance)
   → Deploys modules (backend, approuter, DB deployer)
   → Binds resources to modules
   → Starts applications
   → DONE! Your app is live! 🎉
```

---

## Session 6: Hands-on — Generate & Customize mta.yaml (16:00 - 16:45)

### Exercise 1: Generate mta.yaml (10 minutes)

```bash
cd ~/projects/po-management

# Generate mta.yaml
cds add mta

# View it
cat mta.yaml
```

**Verify it has:**
- At least 3 modules (db-deployer, srv, approuter or similar)
- At least 2 resources (hana, xsuaa)
- Correct paths (gen/db, gen/srv, app/)

---

### Exercise 2: Understand the Structure (15 minutes)

Open `mta.yaml` and answer these questions:

1. What is the ID of your application? → _____
2. How many modules are defined? → _____
3. What type is the backend module? → _____
4. What does the db-deployer module deploy to? → _____
5. Which resource provides authentication? → _____
6. What file does the XSUAA resource reference? → _____
7. How does the approuter know the backend's URL? → _____

<details>
<summary>Answers</summary>

1. The `ID` field at the top (e.g., `po-management`)
2. Typically 3-4 (db-deployer, srv, approuter, optional UI deployer)
3. `nodejs` (Node.js application)
4. The HDI container resource (HANA database)
5. The resource with `service: xsuaa`
6. `./xs-security.json` (path to your security config)
7. Backend `provides: srv-api` with URL → Approuter `requires: srv-api` → reads URL → uses as destination

</details>

---

### Exercise 3: Customize mta.yaml (20 minutes)

Make these modifications:

**1. Add memory limits:**
```yaml
- name: po-management-srv
  parameters:
    memory: 256M       # Add this
    disk-quota: 512M   # Add this
```

**2. Add a description:**
```yaml
description: "PO Management - CAP application with Fiori Elements UI"
```

**3. Verify xs-security.json path:**
```yaml
- name: po-management-auth
  parameters:
    path: ./xs-security.json  # Make sure this path is correct!
```

**4. Add environment variable for logging:**
```yaml
- name: po-management-srv
  properties:
    NODE_ENV: production
    LOG_LEVEL: info
```

---

## Assessment: MCQ — 15 Questions on Approuter & MTA

**Q1.** The Application Router's primary purpose is:

- A) Storing data in HANA
- B) Acting as a single entry point that handles authentication and routes requests
- C) Building the application
- D) Running database queries

**Answer: B** — Approuter = single entry point + authentication gateway + request router.

---

**Q2.** `xs-app.json` defines:

- A) Database schema
- B) Routing rules — which URL patterns go to which destinations
- C) UI layout
- D) Test cases

**Answer: B** — xs-app.json tells the approuter HOW to route each incoming request.

---

**Q3.** In a route, `"destination": "srv-api"` means:

- A) The route creates an API
- B) Forward matching requests to the backend named "srv-api" (defined in mta.yaml)
- C) Serve static files
- D) Delete the request

**Answer: B** — Destinations are named references to backend services. The actual URL is resolved from mta.yaml bindings.

---

**Q4.** `"authenticationType": "xsuaa"` on a route means:

- A) No authentication needed
- B) The user must be authenticated via XSUAA before accessing this route
- C) Basic authentication with username/password
- D) API key required

**Answer: B** — XSUAA authentication required. Unauthenticated users are redirected to the login page.

---

**Q5.** `"forwardAuthToken": true` in the approuter destination config:

- A) Removes the auth token
- B) Passes the user's JWT token to the backend service in the Authorization header
- C) Creates a new token
- D) Logs the token

**Answer: B** — This is crucial! Without it, the backend doesn't know WHO the user is. The JWT must be forwarded.

---

**Q6.** MTA stands for:

- A) Multi-Tenant Architecture
- B) Multi-Target Application
- C) Managed Technology Application
- D) Module Transfer Archive

**Answer: B** — Multi-Target Application — one descriptor for multiple deployable components.

---

**Q7.** In mta.yaml, "modules" are:

- A) BTP services (HANA, XSUAA)
- B) YOUR deployable code/apps (backend, approuter, DB deployer)
- C) Configuration files
- D) Test environments

**Answer: B** — Modules = your code that gets deployed and runs. Resources = BTP services they consume.

---

**Q8.** In mta.yaml, "resources" are:

- A) Your application code
- B) BTP platform services your app depends on (HANA, XSUAA, Destination)
- C) Static HTML files
- D) Build tools

**Answer: B** — Resources are services created/managed by BTP that your modules bind to.

---

**Q9.** A module with `type: hdb` is:

- A) A web server
- B) A HANA Database deployer (deploys schema + data to HDI container)
- C) An HTML page
- D) An authentication service

**Answer: B** — `hdb` type modules deploy database artifacts (tables, views, data) to an HDI container.

---

**Q10.** `requires` in a module means:

- A) "I provide this service to others"
- B) "I depend on this resource/service — bind it to me at deployment"
- C) "This module is required to exist"
- D) "This is a required field"

**Answer: B** — `requires` declares dependencies. Cloud Foundry binds the listed services to the module.

---

**Q11.** `provides` in a module means:

- A) "I need this to run"
- B) "I make this available for other modules to consume (like my URL)"
- C) "I provide user authentication"
- D) "I provide data storage"

**Answer: B** — `provides` exports properties (like URLs) that other modules can reference via their `requires`.

---

**Q12.** The command to generate mta.yaml in a CAP project:

- A) `npm init mta`
- B) `cds add mta`
- C) `mta create`
- D) `cf create-mta`

**Answer: B** — `cds add mta` generates the mta.yaml based on your project structure.

---

**Q13.** The command to build the MTAR archive:

- A) `npm build`
- B) `cds compile` 
- C) `cds build`
- D) `cf build`

**Answer: B** — `mbt build` (MTA Build Tool) reads mta.yaml, builds all modules, creates `.mtar` archive.

---

**Q14.** `cf deploy *.mtar` does what?

- A) Downloads the app from cloud
- B) Uploads the archive to Cloud Foundry, creates resources, deploys modules, binds everything
- C) Only creates the database
- D) Only starts the approuter

**Answer: B** — One command deploys the entire application: creates services, deploys code, binds together.

---

**Q15.** Route order in xs-app.json matters because:

- A) Later routes override earlier ones
- B) Routes are checked top-to-bottom; FIRST match wins
- C) Routes are sorted alphabetically
- D) Order doesn't matter at all

**Answer: B** — First matching route handles the request. Put specific routes before catch-all routes!

---

## Assignment: Document mta.yaml Structure

### Task

Create a documentation file that explains your project's `mta.yaml` with detailed annotations for each section.

### Deliverable

Create a file `docs/mta-explained.md` with:

1. **Overview diagram** showing modules and resources as boxes with arrows showing dependencies
2. **For each module:** explain its purpose, type, path, what it requires, what it provides
3. **For each resource:** explain what BTP service it uses, what plan, and why it's needed
4. **The deployment order:** which components deploy first and why
5. **Connection map:** how the approuter connects to backend, how backend connects to HANA

### Template

```markdown
# MTA Architecture — PO Management Application

## Architecture Diagram
(Draw the boxes and arrows)

## Modules

### 1. po-management-db-deployer
- **Purpose:** (what does it do?)
- **Type:** (hdb)
- **Path:** (where is the code?)
- **Requires:** (what services does it need?)
- **Deploy order:** (when does it run?)

### 2. po-management-srv
(repeat for each module)

## Resources

### 1. po-management-db
- **Purpose:** (what does it provide?)
- **BTP Service:** (hana / xsuaa / etc.)
- **Plan:** (hdi-shared / application / etc.)
- **Used by:** (which modules?)

## Deployment Flow
1. First: Resources are created (HANA, XSUAA)
2. Then: DB deployer runs (creates tables)
3. Then: Backend deploys (connects to HANA + XSUAA)
4. Finally: Approuter deploys (connects to backend + XSUAA)
```

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Approuter | Single entry point — handles auth, routing, token injection |
| xs-app.json | Routing rules: URL patterns → destinations |
| Destinations | Named references to backend URLs (resolved at deploy time) |
| `forwardAuthToken` | Passes JWT from browser session to backend |
| MTA | Multi-Target Application — one descriptor for all components |
| mta.yaml | Defines modules (your code) + resources (BTP services) |
| Modules | nodejs (backend), hdb (DB deployer), approuter.nodejs (router) |
| Resources | hdi-container (HANA), xsuaa (auth), html5-apps-repo (UI) |
| requires/provides | Dependency wiring between modules and resources |
| `mbt build` | Creates deployable .mtar archive |
| `cf deploy` | Deploys everything to Cloud Foundry in one command |

### The Deployment Equation

```
mta.yaml (blueprint) + mbt build (package) + cf deploy (ship) = App Live in Cloud! 🚀
```

### The One-Liner

> **The approuter is your app's front door (one URL, handles auth, routes requests), and mta.yaml is the blueprint that tells Cloud Foundry how to build and wire everything together.**

---

### Looking Ahead: Day 32

Tomorrow: **Deploying to SAP BTP Cloud Foundry**
- Building the MTAR archive
- `cf login` and deployment
- Verifying the deployed app
- Monitoring and logs in BTP
- End-to-end testing in the cloud

---

*End of Day 31 — You now understand the complete deployment architecture!*
