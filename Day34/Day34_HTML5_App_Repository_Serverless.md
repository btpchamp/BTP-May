# Day 34: HTML5 Application Repository & Serverless Fiori

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 33 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Why Serverless Fiori? The HTML5 App Repository | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Managed Approuter & Destination Service | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Create Destinations & Configure App | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: HTML5 Deployment Configuration (ui5.yaml & manifest.json) | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Deploy Fiori App to HTML5 Repository | 60 min |
| 16:00 - 16:45 | Session 6: Verify & Test Serverless Fiori App | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain why serverless Fiori apps save cost and improve performance
- Describe the SAP BTP HTML5 Application Repository and its role
- Differentiate Managed Application Router from Standalone Approuter
- Configure destinations in BTP Cockpit to connect UI to backend
- Set up HTML5 app deployment configuration (ui5.yaml)
- Deploy your Fiori app to the HTML5 Application Repository
- Verify the deployed app in the BTP Cockpit's HTML5 Applications list

---

## Day 33 Recap — Quick Fire (09:00 - 09:15)

1. Users access the deployed app via the _____ URL? → _____
2. After deployment, roles must be assigned in _____? → _____
3. A user who can login but gets 403 needs _____? → _____
4. `cf scale -i 3` does what? → _____
5. `cf set-env` requires _____ afterward? → _____
6. `[RTR/0]` in logs represents _____? → _____
7. Role templates appear in BTP Cockpit after _____? → _____
8. `cf undeploy --delete-services` does _____? → _____

<details>
<summary>Answers</summary>

1. **Approuter** URL (single entry point with authentication)
2. **BTP Cockpit** → Security → Role Collections
3. A **Role Collection** assigned to their user
4. Runs **3 instances** of the app (horizontal scaling)
5. `cf restart` (app must restart to read new env vars)
6. **HTTP router** — incoming request/response with status code
7. **Successful deployment** (XSUAA service instance creation)
8. **Removes everything** — apps, services, bindings (complete cleanup)

</details>

---

## Session 1: Why Serverless Fiori? The HTML5 App Repository (09:15 - 10:30)

### The Problem with Standalone Approuter

In Day 31, we learned about the standalone approuter. It works great, but has a drawback:

```
STANDALONE APPROUTER (what we've been using):
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ┌──────────────────┐     Running 24/7!                     │
│  │  APPROUTER        │     ├── Uses 256MB RAM constantly    │
│  │  (Node.js app)    │     ├── Costs money even when nobody  │
│  │                   │     │   is using the app              │
│  │  Serves UI files  │     ├── Must be updated/maintained    │
│  │  Handles auth     │     └── Takes quota from your space   │
│  │  Routes requests  │                                       │
│  └──────────────────┘                                        │
│                                                              │
│  For a simple Fiori app that serves static files...          │
│  this is OVERKILL! 😤                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**The question:** Do we really need a constantly running Node.js app just to serve some HTML, JS, and CSS files?

**The answer:** NO! That's why the **Managed Application Router** and **HTML5 App Repository** exist.

---

### The Serverless Solution

```
MANAGED APPROUTER + HTML5 REPOSITORY (serverless):
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ┌──────────────────────┐    NOT your app!                  │
│  │  MANAGED APPROUTER    │    ├── Provided by SAP (shared)  │
│  │  (SAP-managed)        │    ├── You DON'T deploy it       │
│  │                       │    ├── You DON'T pay for it      │
│  │  Handles auth         │    ├── Scales automatically      │
│  │  Routes requests      │    └── Zero maintenance!         │
│  └──────────┬────────────┘                                   │
│             │                                                │
│             │ serves static files from:                      │
│             ▼                                                │
│  ┌──────────────────────┐                                    │
│  │  HTML5 APP REPOSITORY │    Just file storage!            │
│  │  (SAP-managed)        │    ├── Upload your HTML/JS/CSS   │
│  │                       │    ├── No running server needed   │
│  │  Stores: index.html   │    ├── Pay only for storage      │
│  │         Component.js  │    │   (pennies!)                │
│  │         manifest.json │    └── CDN-like performance      │
│  └──────────────────────┘                                    │
│                                                              │
│  Your Fiori UI is "serverless" — no running instance!       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### Cost & Performance Comparison

| Aspect | Standalone Approuter | Managed + HTML5 Repo |
|--------|---------------------|---------------------|
| **Running cost** | 256MB RAM × 24/7 = ~$30-50/month | Almost nothing (storage only) |
| **Your maintenance** | Update Node.js, patch approuter | Zero — SAP manages everything |
| **Scaling** | You manage (`cf scale`) | Automatic (SAP handles) |
| **Cold start** | Always warm (running) | Slight delay on first access |
| **Memory quota used** | 256MB+ of YOUR quota | 0 MB of your quota! |
| **Best for** | Complex routing, custom middleware | Standard Fiori apps (90% of cases) |

**Bottom line:** For standard Fiori Elements apps, the managed approach is ALWAYS better.

---

### How Serverless Fiori Works

```
User opens app URL:
     │
     ▼
┌────────────────────┐
│  Managed Approuter │  (SAP provides this — shared across all apps)
│  (Launchpad/FLP)   │
└────────┬───────────┘
         │
         ├── UI files? → Read from HTML5 App Repository
         │                (index.html, Component.js, etc.)
         │
         └── API calls? → Route to backend via DESTINATION
                          (https://my-cap-srv.cfapps.../catalog/Products)
```

**You deploy:**
1. Backend (CAP service) → as a CF app (same as before)
2. UI files → to HTML5 App Repository (NOT as a CF app!)
3. A destination → connecting UI to backend

**You DON'T deploy:**
- No approuter app
- No UI hosting app
- Nothing running for the frontend!

---

### What is the HTML5 Application Repository?

The **HTML5 Application Repository** is a BTP service that stores and serves static web files (HTML, JavaScript, CSS, images).

Think of it as:
- **Google Drive for web apps** — you upload files, it serves them
- **CDN** — files are served from SAP's infrastructure, close to users
- **Serverless hosting** — no server management, just files

```
┌─────────────────────────────────────────────────────────────┐
│           HTML5 APPLICATION REPOSITORY                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  App: "po-management-ui"                                    │
│  ├── webapp/                                                │
│  │   ├── index.html                                         │
│  │   ├── Component.js                                       │
│  │   ├── manifest.json                                      │
│  │   └── i18n/                                              │
│  │       └── i18n.properties                                │
│  │                                                          │
│  App: "product-catalog-ui"                                  │
│  ├── webapp/                                                │
│  │   ├── index.html                                         │
│  │   ├── Component.js                                       │
│  │   └── manifest.json                                      │
│  │                                                          │
│  (Multiple apps stored in one repository)                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Where to See HTML5 Apps in BTP Cockpit

```
BTP Cockpit → Subaccount → HTML5 Applications
┌─────────────────────────────────────────────────────────────┐
│  HTML5 Applications                                          │
├─────────────────────────────────────────────────────────────┤
│  Application Name          │ Version │ Updated      │ URL   │
│  po-management-ui          │ 1.0.0   │ 2026-06-05  │ 🔗    │
│  product-catalog-ui        │ 1.0.0   │ 2026-06-03  │ 🔗    │
│  employee-portal           │ 2.1.0   │ 2026-06-01  │ 🔗    │
└─────────────────────────────────────────────────────────────┘
```

Each app gets a URL that serves it through the Managed Approuter.

---

## Session 2: Managed Approuter & Destination Service (10:45 - 12:00)

### Managed vs Standalone Approuter

```
┌─────────────────────────────────────────────────────────────┐
│        STANDALONE                    MANAGED                 │
├──────────────────────────────┬──────────────────────────────┤
│                              │                              │
│  YOU deploy an approuter     │  SAP provides the approuter  │
│  app in your space           │  (shared, centralized)       │
│                              │                              │
│  ┌──────────────┐            │  ┌──────────────┐           │
│  │  YOUR app    │            │  │  SAP's app   │           │
│  │  (Node.js)   │            │  │  (managed)   │           │
│  │  256MB RAM   │            │  │  FREE for you│           │
│  │  YOUR cost   │            │  │  SAP's cost  │           │
│  └──────────────┘            │  └──────────────┘           │
│                              │                              │
│  You write xs-app.json       │  You configure DESTINATIONS  │
│  You maintain it             │  SAP maintains the router    │
│                              │                              │
│  USE WHEN:                   │  USE WHEN:                   │
│  • Custom routing logic      │  • Standard Fiori apps       │
│  • Custom middleware         │  • Cost optimization         │
│  • WebSocket support         │  • Zero maintenance          │
│  • Complex multi-app setup   │  • 90% of projects!          │
│                              │                              │
└──────────────────────────────┴──────────────────────────────┘
```

---

### The Destination Service

With the managed approuter, you don't have `xs-app.json` routes. Instead, you configure **Destinations** — named connections from your UI to your backend.

**Destination** = "Where is my backend? How do I authenticate to it?"

```
┌───────────────────────────────────────────────────────────┐
│  DESTINATION: "po-management-api"                          │
├───────────────────────────────────────────────────────────┤
│  Name:            po-management-api                        │
│  Type:            HTTP                                     │
│  URL:             https://po-management-srv.cfapps.us10... │
│  Authentication:  OAuth2JWTBearer                          │
│  Token Service:   https://auth.us10.hana.ondemand.com     │
│  Client ID:       sb-po-management!t12345                 │
│  Client Secret:   (auto from XSUAA binding)               │
│                                                           │
│  This tells the managed approuter:                         │
│  "When the UI calls /api/* → forward to THIS URL          │
│   with THIS authentication method"                        │
└───────────────────────────────────────────────────────────┘
```

---

### Creating Destinations — Where?

**Method 1: BTP Cockpit (manual)**
```
BTP Cockpit → Subaccount → Connectivity → Destinations → [New Destination]
```

**Method 2: mta.yaml (automatic at deploy time)**
```yaml
resources:
  - name: po-management-destination
    type: org.cloudfoundry.managed-service
    parameters:
      service: destination
      service-plan: lite
      config:
        init_data:
          instance:
            destinations:
              - Name: po-management-api
                URL: ~{srv-api/srv-url}
                Authentication: NoAuthentication
                Type: HTTP
                ProxyType: Internet
                HTML5.ForwardAuthToken: true
                HTML5.DynamicDestination: true
```

---

### Destination Configuration Fields

| Field | Meaning | Example Value |
|-------|---------|---------------|
| **Name** | Unique identifier for this destination | `po-management-api` |
| **Type** | Protocol type | `HTTP` |
| **URL** | Backend service URL | `https://po-srv.cfapps.us10...` |
| **ProxyType** | `Internet` (cloud) or `OnPremise` (via Cloud Connector) | `Internet` |
| **Authentication** | How to authenticate to the backend | `NoAuthentication` / `OAuth2JWTBearer` |
| **HTML5.ForwardAuthToken** | Pass user's JWT to the backend | `true` |
| **HTML5.DynamicDestination** | Allow runtime destination resolution | `true` |

---

### How the UI Finds the Backend (Managed Mode)

In your Fiori app's `manifest.json`, instead of hardcoding the backend URL, you reference a destination:

```json
{
  "sap.app": {
    "dataSources": {
      "mainService": {
        "uri": "/catalog/",
        "type": "OData",
        "settings": {
          "odataVersion": "4.0"
        }
      }
    }
  }
}
```

The managed approuter sees `/catalog/` → looks up the destination → forwards to the backend URL. Clean!

---

### The Complete Serverless Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SAP BTP                                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  User → Managed Approuter (SAP FLP / launchpad)             │
│              │                                              │
│              ├── /app-name/* → HTML5 App Repository          │
│              │                  (serves your static files)   │
│              │                                              │
│              └── /api/* → DESTINATION → CAP Backend          │
│                              │            (your OData service)│
│                              │                              │
│                              └── Configured in BTP Cockpit   │
│                                  or mta.yaml                 │
│                                                             │
│  WHAT YOU DEPLOY:                    WHAT SAP PROVIDES:      │
│  ├── CAP Backend (cf app)            ├── Managed Approuter   │
│  ├── UI files (HTML5 repo)           ├── HTML5 Repo service  │
│  └── Destination config              └── Destination service │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Session 3: Hands-on — Create Destinations & Configure App (12:00 - 13:00)

### Exercise 1: Create a Destination in BTP Cockpit (20 minutes)

1. **Navigate:** BTP Cockpit → Subaccount → Connectivity → Destinations
2. Click **"New Destination"**
3. Fill in:

```
Name:               po-management-api
Type:               HTTP
URL:                https://po-management-srv.cfapps.us10.hana.ondemand.com
Proxy Type:         Internet
Authentication:     NoAuthentication

Additional Properties (click "New Property"):
  HTML5.ForwardAuthToken = true
  HTML5.DynamicDestination = true
  WebIDEUsage = odata_gen
  WebIDEEnabled = true
```

4. Click **"Save"**
5. Click **"Check Connection"** → should show `200 OK`

---

### Exercise 2: Verify Destination Works (10 minutes)

After creating the destination:

1. Click **"Check Connection"** button
2. Expected result: `Connection to "po-management-api" established. Response: 200 OK`
3. If it fails:
   - Check URL is correct (copy from `cf app po-management-srv | grep routes`)
   - Ensure backend app is running (`cf apps`)
   - Verify no typos in destination name

---

### Exercise 3: Update Fiori App for Managed Approuter (30 minutes)

Modify your Fiori app to work with the managed approuter:

**File: `app/products/webapp/manifest.json`** (update dataSources):

```json
{
  "sap.app": {
    "id": "po.management.products",
    "type": "application",
    "title": "Manage Products",
    "dataSources": {
      "mainService": {
        "uri": "/catalog/",
        "type": "OData",
        "settings": {
          "odataVersion": "4.0"
        }
      }
    },
    "crossNavigation": {
      "inbounds": {
        "intent1": {
          "signature": {
            "parameters": {},
            "additionalParameters": "allowed"
          },
          "semanticObject": "Products",
          "action": "manage",
          "title": "Manage Products"
        }
      }
    }
  }
}
```

**Key changes for managed approuter:**
- `uri` uses a relative path (`/catalog/`)
- The managed approuter resolves this via the destination
- `crossNavigation` enables Fiori Launchpad integration

---

## Session 4: HTML5 Deployment Configuration (13:45 - 14:45)

### Understanding ui5.yaml

The `ui5.yaml` file configures how your UI5/Fiori app is built and served. It's used by the UI5 tooling.

**File: `app/products/ui5.yaml`**

```yaml
specVersion: "3.0"
metadata:
  name: po-management-products
type: application
framework:
  name: SAPUI5
  version: "1.120.0"
  libraries:
    - name: sap.m
    - name: sap.f
    - name: sap.fe.templates
builder:
  customTasks:
    - name: ui5-tooling-transpile-task
      afterTask: replaceVersion
```

---

### Deployment Configuration — xs-app.json for HTML5 Repo

Even with the managed approuter, each HTML5 app needs a small `xs-app.json` that defines its routes:

**File: `app/products/xs-app.json`**

```json
{
  "welcomeFile": "/index.html",
  "authenticationMethod": "route",
  "routes": [
    {
      "source": "^/catalog/(.*)$",
      "target": "/catalog/$1",
      "destination": "po-management-api",
      "authenticationType": "xsuaa",
      "csrfProtection": true
    },
    {
      "source": "^/purchasing/(.*)$",
      "target": "/purchasing/$1",
      "destination": "po-management-api",
      "authenticationType": "xsuaa",
      "csrfProtection": true
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

**This xs-app.json travels WITH the HTML5 app** into the repository. The managed approuter reads it to know how to route requests for THIS specific app.

---

### The mta.yaml Changes for HTML5 Repository Deployment

Replace the standalone approuter module with HTML5 repo deployment:

```yaml
modules:
  # ─── Backend (same as before) ─────────────
  - name: po-management-srv
    type: nodejs
    path: gen/srv
    requires:
      - name: po-management-db
      - name: po-management-auth
    provides:
      - name: srv-api
        properties:
          srv-url: ${default-url}

  # ─── DB Deployer (same as before) ─────────
  - name: po-management-db-deployer
    type: hdb
    path: gen/db
    requires:
      - name: po-management-db

  # ─── 🆕 HTML5 App Deployer (replaces standalone approuter!) ────
  - name: po-management-ui-deployer
    type: com.sap.application.content
    path: .
    requires:
      - name: po-management-html5-repo-host
        parameters:
          content-target: true
    build-parameters:
      build-result: resources
      requires:
        - name: po-management-products-app
          artifacts:
            - "./*"
          target-path: resources/products

  # ─── 🆕 Fiori App Build (generates the deployable UI) ─────
  - name: po-management-products-app
    type: html5
    path: app/products
    build-parameters:
      build-result: dist
      builder: custom
      commands:
        - npm install
        - npm run build
      supported-platforms: []

resources:
  # ─── HANA (same) ─────────
  - name: po-management-db
    type: com.sap.xs.hdi-container

  # ─── XSUAA (same) ─────────
  - name: po-management-auth
    type: org.cloudfoundry.managed-service
    parameters:
      service: xsuaa
      service-plan: application
      path: ./xs-security.json

  # ─── 🆕 HTML5 App Repository Host ─────────
  - name: po-management-html5-repo-host
    type: org.cloudfoundry.managed-service
    parameters:
      service: html5-apps-repo
      service-plan: app-host

  # ─── 🆕 HTML5 App Repository Runtime ─────────
  - name: po-management-html5-repo-rt
    type: org.cloudfoundry.managed-service
    parameters:
      service: html5-apps-repo
      service-plan: app-runtime

  # ─── 🆕 Destination Service ─────────
  - name: po-management-destination
    type: org.cloudfoundry.managed-service
    parameters:
      service: destination
      service-plan: lite
      config:
        HTML5Runtime_enabled: true
        init_data:
          instance:
            existing_destinations_policy: update
            destinations:
              - Name: po-management-api
                URL: ~{srv-api/srv-url}
                Authentication: NoAuthentication
                Type: HTTP
                ProxyType: Internet
                HTML5.ForwardAuthToken: true
                HTML5.DynamicDestination: true
    requires:
      - name: srv-api
```

---

### What Changed From Standalone to Managed?

| Removed | Added |
|---------|-------|
| `po-management-approuter` module | `po-management-ui-deployer` module |
| Standalone approuter (256MB running) | HTML5 repo host + runtime (no running app!) |
| xs-app.json in approuter folder | xs-app.json inside each UI app |
| — | Destination service + destination config |

**Net result:** One fewer running application! Lower cost! Same functionality!

---

### Build Script for the Fiori App

Add a build script to your Fiori app's `package.json`:

**File: `app/products/package.json`**

```json
{
  "name": "po-management-products",
  "version": "1.0.0",
  "scripts": {
    "build": "ui5 build --clean-dest --include-task=generateManifestBundle",
    "start": "ui5 serve",
    "deploy": "fiori deploy --config ui5-deploy.yaml"
  },
  "devDependencies": {
    "@ui5/cli": "^3",
    "@sap/ux-ui5-tooling": "^1"
  }
}
```

---

## Session 5: Hands-on — Deploy Fiori App to HTML5 Repository (15:00 - 16:00)

### Exercise 1: Prepare the UI App for Deployment (15 minutes)

Make sure your app folder has all required files:

```
app/products/
├── webapp/
│   ├── index.html
│   ├── Component.js
│   ├── manifest.json      ← Updated with dataSources
│   └── i18n/
│       └── i18n.properties
├── xs-app.json            ← Routes for managed approuter
├── package.json           ← With build script
└── ui5.yaml               ← UI5 tooling config
```

---

### Exercise 2: Build and Deploy via MTA (25 minutes)

```bash
# 1. Make sure mta.yaml has HTML5 repo config (from Session 4)
cat mta.yaml | grep "html5-apps-repo"

# 2. Build the MTA archive
mbt build

# 3. Deploy
cf deploy mta_archives/po-management_1.0.0.mtar

# 4. Verify HTML5 app is deployed
cf html5-list -di po-management-destination -u

# Or check in BTP Cockpit:
# Subaccount → HTML5 Applications
```

---

### Exercise 3: Alternative — Deploy UI Only (20 minutes)

If you just want to update the UI without redeploying the entire MTA:

```bash
# Build only the UI app
cd app/products
npm install
npm run build

# Deploy to HTML5 repo directly
cf html5-push -p dist/

# Verify
cf html5-list
```

**This is much faster than full MTA deployment when you're only changing the UI!**

---

## Session 6: Verify & Test Serverless Fiori App (16:00 - 16:45)

### Verifying Deployment

**Method 1: BTP Cockpit**
1. Navigate: Subaccount → HTML5 Applications
2. You should see your app listed with its name and version
3. Click the URL → app opens (through managed approuter)

**Method 2: CF CLI**
```bash
# List HTML5 apps
cf html5-list

# Output:
# Getting list of HTML5 applications...
# name                        version   app-host-id
# po-management-products      1.0.0     abc123-def456

# Get the app URL
cf html5-info -n po-management-products
```

**Method 3: Launchpad (if configured)**

If your BTP subaccount has SAP Launchpad Service:
1. Open the Launchpad site
2. Your app should appear as a tile
3. Click to launch — it loads from HTML5 repo!

---

### Testing the Complete Flow

```
1. Open app URL (from HTML5 Applications in Cockpit)
   → Managed approuter handles authentication
   → Redirects to login if needed

2. After login → Fiori app loads from HTML5 repo
   → index.html, Component.js served from repository
   → No approuter app running!

3. App calls OData API (e.g., GET /catalog/Products)
   → Managed approuter reads xs-app.json
   → Matches route: /catalog/* → destination "po-management-api"
   → Forwards to backend with JWT token
   → Data returned to UI

4. Full app is working! ✅
   → No standalone approuter consuming memory
   → Only backend is running as a CF app
   → UI is "serverless"!
```

---

### Comparison: Before and After

```
BEFORE (Standalone):                    AFTER (Managed + HTML5 Repo):
┌────────────────────────┐             ┌────────────────────────┐
│  Running apps: 3        │             │  Running apps: 1        │
│  ├── approuter (256MB) │             │  └── backend-srv (256MB)│
│  ├── backend   (256MB) │             │                         │
│  └── db-deployer (done)│             │  Services:              │
│                         │             │  ├── HANA (existing)    │
│  Total memory: 512MB    │             │  ├── XSUAA (existing)   │
│                         │             │  ├── HTML5 Repo (new)   │
│                         │             │  └── Destination (new)  │
│                         │             │                         │
│                         │             │  Total memory: 256MB    │
│                         │             │  SAVED: 256MB! 💰       │
└────────────────────────┘             └────────────────────────┘
```

---

### Troubleshooting HTML5 App Deployment

| Issue | Cause | Fix |
|-------|-------|-----|
| App not in HTML5 list | Deployment failed silently | Check `cf logs <deployer> --recent` |
| "App not found" when opening URL | Wrong app ID in manifest.json | Verify `sap.app.id` matches |
| API calls return 404 | Destination not configured or wrong URL | Check destination in BTP Cockpit |
| API calls return 401 | `HTML5.ForwardAuthToken` not set | Add to destination properties |
| Blank page loads | Build failed (no dist/ created) | Run `npm run build` locally first |
| Old version showing | Browser cache | Hard refresh (Ctrl+Shift+R) |

---

## Assessment: MCQ — 15 Questions on HTML5 Repository

**Q1.** The HTML5 Application Repository stores:

- A) Backend Node.js applications
- B) Static web files (HTML, JS, CSS) for Fiori apps — no running server needed
- C) Database tables
- D) Docker containers

**Answer: B** — It's a storage service for static files. No server runs for your UI — it's serverless!

---

**Q2.** The Managed Application Router differs from standalone because:

- A) It's faster
- B) SAP provides and manages it (you don't deploy or pay for an approuter app)
- C) It doesn't support authentication
- D) It only works with SAP GUI

**Answer: B** — Managed = SAP-provided, shared infrastructure. You save memory, cost, and maintenance.

---

**Q3.** The main cost benefit of serverless Fiori is:

- A) Cheaper HANA database
- B) No memory consumption for the UI layer (no running approuter app in your space)
- C) Free backend service
- D) No authentication needed

**Answer: B** — Eliminating the standalone approuter (256MB 24/7) saves significant memory quota and cost.

---

**Q4.** A Destination in BTP defines:

- A) Where users login
- B) A named connection from the UI to a backend service (URL + auth method)
- C) The database location
- D) The Git repository URL

**Answer: B** — Destinations map a name to a backend URL + authentication config. The managed approuter uses them for routing.

---

**Q5.** `HTML5.ForwardAuthToken: true` on a destination ensures:

- A) The backend URL is encrypted
- B) The user's JWT token is passed to the backend (so it knows WHO is calling)
- C) A new token is created for each request
- D) Authentication is disabled

**Answer: B** — Without this, the backend gets requests without user identity and returns 401.

---

**Q6.** In managed approuter mode, routing rules (`xs-app.json`) are placed:

- A) In a standalone approuter folder
- B) Inside each HTML5 app (travels with the app into the repository)
- C) In BTP Cockpit only
- D) In package.json

**Answer: B** — Each HTML5 app carries its own xs-app.json. The managed approuter reads it from the repository.

---

**Q7.** The HTML5 App Repository service plans include:

- A) `app-host` (for uploading/storing apps) and `app-runtime` (for serving apps)
- B) `lite` and `premium`
- C) `free` and `paid`
- D) `small` and `large`

**Answer: A** — `app-host` = upload/store files. `app-runtime` = serve files to users via managed approuter.

---

**Q8.** To update ONLY the UI (without redeploying backend):

- A) Must redeploy entire MTA
- B) `cf html5-push -p dist/` pushes updated UI files directly to the repository
- C) Delete and recreate the HANA database
- D) Run `cf restart`

**Answer: B** — `cf html5-push` updates UI files independently — much faster than full MTA deployment!

---

**Q9.** `cf html5-list` shows:

- A) All CF applications
- B) All HTML5 apps stored in the repository (name, version, host ID)
- C) All destinations
- D) All service instances

**Answer: B** — Lists HTML5 applications in the repository with their versions.

---

**Q10.** For standard Fiori Elements apps, the recommended approach is:

- A) Always use standalone approuter
- B) Managed approuter + HTML5 Repository (serverless, cheaper, zero maintenance)
- C) Deploy UI on a separate server
- D) Embed UI code in the backend

**Answer: B** — SAP recommends managed + HTML5 repo for standard Fiori apps (90% of cases).

---

**Q11.** The `type: com.sap.application.content` module in mta.yaml is:

- A) A running Node.js application
- B) A content deployer that uploads files to HTML5 repository (runs once, then stops)
- C) A database deployer
- D) An authentication service

**Answer: B** — It's a one-time deployer: pushes UI files to HTML5 repo, then exits. No persistent running app.

---

**Q12.** `crossNavigation.inbounds` in manifest.json enables:

- A) Cross-origin requests
- B) Integration with SAP Fiori Launchpad (app appears as a tile with defined intent)
- C) Multi-browser support
- D) API versioning

**Answer: B** — Inbounds define how the Fiori Launchpad discovers and launches your app (semantic object + action).

---

**Q13.** When the managed approuter receives a request for `/catalog/Products`:

- A) It serves a static HTML file
- B) It matches the route in xs-app.json → looks up the destination → forwards to the backend with auth token
- C) It queries the database directly
- D) It returns an error

**Answer: B** — Route matching → destination resolution → forward with JWT. Same behavior as standalone, just managed by SAP.

---

**Q14.** The `"service": "html5-apps-repo-rt"` in a route means:

- A) Route to the backend API
- B) Serve the file from HTML5 App Repository runtime (static files)
- C) Restart the application
- D) Route to HANA

**Answer: B** — `html5-apps-repo-rt` = serve from the HTML5 repository (for static file routes like index.html, *.js, *.css).

---

**Q15.** After moving from standalone to managed approuter, your backend (CAP service):

- A) No longer needs XSUAA
- B) Stays EXACTLY the same — no changes needed to the CAP application
- C) Must be rewritten
- D) Loses OData capability

**Answer: B** — The backend is unchanged. Only the UI deployment and routing infrastructure change.

---

## Assignment: Deploy Fiori App as Serverless HTML5 Application

### Task

Convert your PO Management Fiori app from standalone approuter to serverless (managed approuter + HTML5 repository).

### Deliverables

1. **Destination created** in BTP Cockpit pointing to your CAP backend
2. **xs-app.json** inside your Fiori app with proper routes
3. **Updated mta.yaml** with HTML5 repo modules (no standalone approuter)
4. **Successful deployment** — app visible in HTML5 Applications list
5. **Working test** — open app through HTML5 Application URL, verify data loads

### Verification Checklist

- [ ] No standalone approuter running (`cf apps` doesn't show approuter)
- [ ] Backend service is running
- [ ] Destination exists in BTP Cockpit and "Check Connection" returns 200
- [ ] HTML5 app appears in: Subaccount → HTML5 Applications
- [ ] Clicking the app URL opens the Fiori app
- [ ] Authentication works (redirects to login)
- [ ] API calls work (Products/Orders load from backend)
- [ ] Memory usage is lower (only backend app consuming quota)

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Serverless Fiori | UI files stored in a repository, served without running an app |
| HTML5 App Repository | BTP service for storing static web files (HTML, JS, CSS) |
| Managed Approuter | SAP-provided, shared router — you don't deploy or manage it |
| Standalone vs Managed | Managed saves 256MB+ memory, zero maintenance, same functionality |
| Destinations | Named connections (URL + auth) from UI to backend |
| `HTML5.ForwardAuthToken` | Critical: passes JWT to backend so it knows the user |
| xs-app.json (in app) | Travels with the HTML5 app into the repository |
| `cf html5-push` | Quick UI-only deployment without full MTA build |
| Cost savings | Eliminate running approuter = save ~$30-50/month |

### The Architecture Decision

```
"Do I need custom approuter logic (middleware, WebSocket, complex routing)?"

YES → Standalone Approuter (full control)
NO  → Managed Approuter + HTML5 Repo (cheaper, simpler, recommended!)
```

### The One-Liner

> **HTML5 App Repository = "Put your Fiori files here, SAP serves them for you." No server. No cost. No maintenance. Just upload and go.**

---

### Looking Ahead: Day 35

Tomorrow: **Week 7 Review & Final Integration**
- Complete review of Weeks 6-7
- Weekly quiz (30 questions: Git, HANA, Auth, Deploy, HTML5)
- Final mini project: Complete application deployment pipeline

---

*End of Day 34 — Your Fiori app is now serverless and cost-optimized! 🚀*
