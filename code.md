
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