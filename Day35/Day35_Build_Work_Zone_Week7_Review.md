# Day 35: SAP Build Work Zone & Week 7 Review

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 34 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: What is SAP Build Work Zone? | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Subscribing & Configuring the Launchpad | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Setup Work Zone & Add Your App | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 15:00 | Session 4: Weekly Quiz — Days 31-35 (30 Questions) | 75 min |
| 15:00 - 15:15 | Break | 15 min |
| 15:15 - 17:00 | Session 5 & 6: Mini Project — Full Deployment Pipeline | 105 min |

---

## What You'll Achieve Today

By the end of this review day, you will:
- Understand what SAP Build Work Zone is and its role as the Fiori Launchpad
- Subscribe to Build Work Zone Standard Edition in your BTP account
- Configure a content provider to discover your deployed apps
- Add your PO Management app as a tile on the Launchpad
- Assign launchpad roles so users can access it
- Complete a 30-question quiz covering the entire deployment week
- Execute the full pipeline: Build → Deploy → Configure → Access from Launchpad

---

## Day 34 Recap — Quick Fire (09:00 - 09:15)

1. HTML5 App Repository stores _____? → _____
2. Managed Approuter is provided by _____? → _____
3. Main cost saving of serverless Fiori: _____? → _____
4. A Destination defines _____? → _____
5. `HTML5.ForwardAuthToken: true` does _____? → _____
6. `cf html5-push` does _____? → _____
7. `service: html5-apps-repo-rt` in a route means _____? → _____
8. Moving from standalone to managed saves about _____ MB? → _____

<details>
<summary>Answers</summary>

1. **Static web files** (HTML, JS, CSS) — no running server needed
2. **SAP** — shared infrastructure, you don't deploy or pay for it
3. **No running approuter app** = 0 MB of your memory quota used for UI
4. A **named connection** from UI to backend (URL + authentication method)
5. Passes the **user's JWT token** to the backend so it knows WHO is calling
6. **Quick UI-only deployment** directly to HTML5 repo without full MTA rebuild
7. **Serve from HTML5 App Repository** (static file routes)
8. **256 MB** (approuter app is eliminated)

</details>

---

## Session 1: What is SAP Build Work Zone? (09:15 - 10:30)

### The Final Piece: Where Users Find Your App

You've built, deployed, and secured your app. But how do users FIND it? They need a **home screen** — a place where all their business apps are organized into one experience.

```
WITHOUT Work Zone:
  "Hey, what's the URL for that PO app?"
  "Hmm, I bookmarked it somewhere..."
  "Let me search my emails for the link..."
  → Users can't find apps. Each app has a different URL. Chaos! 😤

WITH Work Zone (Fiori Launchpad):
  User opens ONE URL → sees ALL their apps organized as tiles:
  ┌─────────────────────────────────────────────────────────┐
  │  My Home                                    👤 Sarah    │
  │                                                         │
  │  ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐          │
  │  │ 📋    │  │ 📝    │  │ 📊    │  │ 🛒    │          │
  │  │Manage │  │Create │  │Reports│  │Product│          │
  │  │  POs  │  │ Order │  │       │  │Catalog│          │
  │  │  7    │  │       │  │       │  │       │          │
  │  └───────┘  └───────┘  └───────┘  └───────┘          │
  │                                                         │
  │  One place. Role-based. Clean. Professional! ✅         │
  └─────────────────────────────────────────────────────────┘
```

---

### What is SAP Build Work Zone?

**SAP Build Work Zone** (Standard Edition) is SAP's service that provides the **Fiori Launchpad** experience on SAP BTP. It's the "home page" for all your Fiori applications.

```
┌─────────────────────────────────────────────────────────────┐
│         SAP BUILD WORK ZONE — Standard Edition               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  WHAT IT IS:                                                │
│    The Fiori Launchpad for BTP                              │
│    A central place where users launch their business apps   │
│                                                             │
│  WHAT IT DOES:                                              │
│    • Shows apps as TILES (role-based)                       │
│    • Provides navigation between apps                       │
│    • Handles authentication (single sign-on)                │
│    • Integrates with HTML5 App Repository                   │
│    • Supports personalization (arrange tiles, themes)       │
│    • Shows notifications                                    │
│                                                             │
│  EDITIONS:                                                  │
│    Standard — basic launchpad (what we use) ← FREE in trial │
│    Advanced — workspaces, workflows, content (paid)         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### How Work Zone Connects Everything

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  User opens: https://your-launchpad.launchpad.cfapps.us10...│
│                         │                                   │
│                         ▼                                   │
│  ┌─────────────────────────────────────────┐                │
│  │        SAP BUILD WORK ZONE              │                │
│  │        (Fiori Launchpad)                │                │
│  │                                         │                │
│  │  "What apps does this user have?"       │                │
│  │    → Checks role collections            │                │
│  │    → Shows appropriate tiles            │                │
│  │                                         │                │
│  │  User clicks "Manage POs" tile:         │                │
│  │    → Loads app from HTML5 Repository    │                │
│  │    → Routes API calls via destinations  │                │
│  │    → Backend returns data               │                │
│  └─────────────────────────────────────────┘                │
│                                                             │
│  Everything connects:                                       │
│  Work Zone ←→ HTML5 Repo ←→ Destinations ←→ CAP Backend   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Work Zone vs HTML5 Repository vs Managed Approuter

| Component | Role |
|-----------|------|
| **HTML5 App Repository** | STORES your Fiori app files (like a file server) |
| **Managed Approuter** | SERVES the files + routes API calls (invisible to users) |
| **SAP Build Work Zone** | ORGANIZES apps into a launchpad + tile navigation |

Think of it like a library:
- **HTML5 Repo** = The shelves (stores the books/apps)
- **Managed Approuter** = The librarian (fetches the right book when asked)
- **Work Zone** = The catalog/homepage (tells you what books are available)

---

### Key Concepts in Work Zone

| Concept | Meaning |
|---------|---------|
| **Site** | A launchpad instance (the home page) |
| **Content Provider** | Where Work Zone discovers apps (HTML5 repo, S/4HANA, etc.) |
| **Content Explorer** | Browse available apps from content providers |
| **Group** | Organizational category for tiles (e.g., "Purchasing", "Reports") |
| **Tile** | A clickable card that launches an app |
| **Role** | Determines which tiles a user sees |
| **Intent** | The navigation target (`SemanticObject-Action`, e.g., `PurchaseOrder-manage`) |

---

### The Site Manager

The **Site Manager** is the admin tool where you configure your launchpad:

```
Site Manager:
├── Content Manager
│   ├── Content Explorer    ← Discover apps from providers
│   ├── My Content          ← Apps you've added to your site
│   └── Roles              ← Which users see which apps
├── Provider Manager
│   └── Content Providers  ← Where to find apps (HTML5 repo)
├── Site Directory
│   └── Your Sites         ← Create/manage launchpad sites
└── Settings
    └── Theming            ← Visual customization
```

---

## Session 2: Subscribing & Configuring the Launchpad (10:45 - 12:00)

### Step 1: Subscribe to SAP Build Work Zone Standard

1. **BTP Cockpit** → Your Subaccount
2. Go to **"Service Marketplace"** (or "Instances and Subscriptions")
3. Search for: **"SAP Build Work Zone, standard edition"**
4. Click **"Create"** (or "Subscribe")
5. Plan: **Standard** (free in trial)
6. Click **"Create"**
7. Wait 1-2 minutes → Status: "Subscribed" ✅

---

### Step 2: Assign the Work Zone Admin Role

After subscribing, you need the admin role to manage the launchpad:

1. Go to **Security → Role Collections**
2. Find: **"Launchpad_Admin"** (created by Work Zone subscription)
3. Click on it → **Users** tab → **Assign User**
4. Add YOUR email
5. Also find and assign: **"Launchpad_External_User"** (to access the launchpad as end user)

**Available Work Zone roles:**

| Role Collection | Purpose |
|----------------|---------|
| `Launchpad_Admin` | Full admin — configure sites, content, providers |
| `Launchpad_External_User` | End user access to the launchpad site |

---

### Step 3: Open the Site Manager

1. BTP Cockpit → Instances and Subscriptions
2. Find "SAP Build Work Zone, standard edition"
3. Click **"Go to Application"** (opens Site Manager)

Or construct the URL:
```
https://<subaccount-subdomain>.launchpad.cfapps.<region>.hana.ondemand.com
```

---

### Step 4: Create a Content Provider

A Content Provider tells Work Zone WHERE to find your apps (in the HTML5 App Repository):

1. In Site Manager → **Provider Manager** (or Channel Manager)
2. Click **"+ New"** (or refresh the default HTML5 Apps content provider)
3. The default "HTML5 Apps" provider should already exist
4. Click the **refresh/fetch** icon → it discovers apps from your HTML5 repository

```
Provider Manager:
┌──────────────────────────────────────────────────────┐
│  Content Providers                                    │
├──────────────────────────────────────────────────────┤
│  Name              │ Status    │ Last Updated         │
│  HTML5 Apps        │ ✅ Active │ June 5, 2026 10:30  │
│                    │           │                      │
│  → Contains your deployed Fiori apps!                │
└──────────────────────────────────────────────────────┘
```

After refresh, your apps from the HTML5 repository appear as available content.

---

### Step 5: Add Apps to Your Launchpad

1. In Site Manager → **Content Manager** → **Content Explorer**
2. Click on "HTML5 Apps" provider
3. You should see your deployed apps (e.g., "Manage Products", "PO Management")
4. Select your app → click **"Add to My Content"**

```
Content Explorer:
┌──────────────────────────────────────────────────────┐
│  Available Apps from HTML5 Apps provider              │
├──────────────────────────────────────────────────────┤
│  □ po-management-products    │ v1.0.0 │ [+ Add]     │
│  □ po-management-orders      │ v1.0.0 │ [+ Add]     │
│                                                      │
│  Select apps → click "Add to My Content"             │
└──────────────────────────────────────────────────────┘
```

---

### Step 6: Create a Group and Assign Tiles

1. Go to **Content Manager** → **My Content**
2. Click **"+ New"** → **"Group"**
3. Name it: "Purchase Management"
4. In the Group editor → **Assign Items** → add your apps
5. Save

```
Group: Purchase Management
├── Tile: Manage Products
├── Tile: Manage Purchase Orders
└── Tile: PO Analytics
```

---

### Step 7: Assign the App to a Role

1. In **Content Manager** → **My Content** → click on "Everyone" role (or create a new role)
2. Click **Edit** → **Assign Items**
3. Select your apps + groups
4. Save

```
Role: Everyone
├── Group: Purchase Management
│   ├── App: Manage Products
│   └── App: Manage POs
└── Group: Analytics
    └── App: Dashboard
```

---

### Step 8: Create and Access the Site

1. Go to **Site Directory**
2. Click **"+ Create Site"**
3. Name: "PO Management Portal"
4. Click **Create**
5. Click the **Open** icon (🔗) → Your launchpad opens!

```
┌─────────────────────────────────────────────────────────┐
│  PO Management Portal                      👤 You       │
│                                                         │
│  Purchase Management                                    │
│  ┌───────────┐  ┌───────────────┐  ┌───────────────┐  │
│  │  📦       │  │  📋           │  │  📊           │  │
│  │  Manage   │  │  Purchase     │  │  Analytics    │  │
│  │  Products │  │  Orders       │  │  Dashboard    │  │
│  │           │  │    7          │  │               │  │
│  └───────────┘  └───────────────┘  └───────────────┘  │
│                                                         │
│  Click any tile to open the app! 🎉                     │
└─────────────────────────────────────────────────────────┘
```

---

## Session 3: Hands-on — Setup Work Zone & Add Your App (12:00 - 13:00)

### Complete Exercise: From Zero to Launchpad

Follow these steps in order:

---

**Step 1: Subscribe (5 minutes)**
```
BTP Cockpit → Service Marketplace → "SAP Build Work Zone, standard edition" → Subscribe
```
Wait for: "Subscribed" status.

---

**Step 2: Assign Admin Role (5 minutes)**
```
Security → Role Collections → "Launchpad_Admin" → Assign your user
Also: "Launchpad_External_User" → Assign your user
```

---

**Step 3: Open Site Manager (2 minutes)**
```
Instances and Subscriptions → Work Zone → "Go to Application"
```

---

**Step 4: Refresh Content Provider (5 minutes)**
```
Site Manager → Provider Manager (or Channel Manager)
→ Click refresh on "HTML5 Apps" provider
→ Wait for apps to be discovered
```

---

**Step 5: Add Apps to Content (5 minutes)**
```
Content Manager → Content Explorer → HTML5 Apps
→ Select your apps → "Add to My Content"
```

---

**Step 6: Create a Group (5 minutes)**
```
Content Manager → My Content → + New → Group
→ Name: "PO Management"
→ Assign Items: add your apps
→ Save
```

---

**Step 7: Assign to Role (5 minutes)**
```
Content Manager → My Content → "Everyone" role
→ Edit → Assign Items → add your apps and group
→ Save
```

---

**Step 8: Create Site (3 minutes)**
```
Site Directory → + Create Site
→ Name: "My Business Portal"
→ Create → Open (🔗)
```

---

**Step 9: Test! (10 minutes)**
```
1. Launchpad opens → you should see your tiles
2. Click a tile → Fiori app opens!
3. Data loads from backend (via destination) ✅
4. Authentication works (JWT from launchpad) ✅
5. Navigation back to launchpad works (shell bar) ✅
```

---

### Troubleshooting Work Zone Setup

| Problem | Fix |
|---------|-----|
| "Access Denied" on Site Manager | Assign `Launchpad_Admin` role to yourself |
| No apps in Content Explorer | Refresh the content provider; check HTML5 apps are deployed |
| Tile shows but app won't open | Check destination configuration + HTML5.ForwardAuthToken |
| "App could not be opened" | Verify xs-app.json routes in your HTML5 app |
| Empty launchpad (no tiles) | Assign apps to a role AND assign role to your user |
| App opens but no data | Backend not running or destination URL is wrong |

---

## Session 4: Weekly Quiz — Days 31-35 (13:45 - 15:00)

### Instructions
- 30 questions covering Days 31-35 (MTA, Deployment, HTML5, Work Zone)
- Time: 75 minutes
- Passing score: 21/30 (70%)

---

**Q1.** The Application Router's primary role is:
- A) Storing database tables
- B) Single entry point — handles auth, routing, and token injection
- C) Building the application
- D) Generating CDS files

**Answer: B**

---

**Q2.** `xs-app.json` defines:
- A) Database schema
- B) Routing rules — URL patterns → destinations
- C) UI annotations
- D) Service definitions

**Answer: B**

---

**Q3.** MTA stands for:
- A) Multi-Tenant Architecture
- B) Multi-Target Application
- C) Managed Technology Application
- D) Module Transfer Archive

**Answer: B**

---

**Q4.** In mta.yaml, "modules" represent:
- A) BTP services
- B) Your deployable code (backend, approuter, DB deployer)
- C) User roles
- D) Network settings

**Answer: B**

---

**Q5.** In mta.yaml, "resources" represent:
- A) Your source code files
- B) BTP platform services your app depends on (HANA, XSUAA)
- C) HTML pages
- D) Git branches

**Answer: B**

---

**Q6.** `mbt build` creates:
- A) A Docker container
- B) An .mtar archive ready for deployment to Cloud Foundry
- C) A GitHub repository
- D) An HTML page

**Answer: B**

---

**Q7.** `cf deploy *.mtar` does what?
- A) Only uploads files
- B) Creates services, deploys modules, binds everything — full deployment
- C) Only creates databases
- D) Only starts the UI

**Answer: B**

---

**Q8.** The deployment order in Cloud Foundry is:
- A) UI → Backend → Database
- B) Resources (services) → DB Deployer → Backend → Approuter/UI
- C) Backend → UI → Database
- D) All simultaneously

**Answer: B**

---

**Q9.** `cf logs <app> --recent` shows:
- A) Build output
- B) Recent application logs (stdout, errors, HTTP requests)
- C) Source code
- D) Database queries

**Answer: B**

---

**Q10.** `VCAP_SERVICES` contains:
- A) User passwords
- B) Connection credentials for bound services (HANA URL, XSUAA secrets)
- C) Application source code
- D) HTML templates

**Answer: B**

---

**Q11.** After deployment, Role Collections must be:
- A) Created in source code
- B) Created in BTP Cockpit and assigned to users manually
- C) Automatically assigned to everyone
- D) Configured in Git

**Answer: B**

---

**Q12.** A user logs in but gets 403. The issue is:
- A) Wrong password
- B) Authenticated but NO Role Collections assigned (no authorization)
- C) Server is down
- D) Browser incompatible

**Answer: B**

---

**Q13.** `cf scale -i 3 -m 512M` does:
- A) Deletes 3 apps
- B) Runs 3 instances with 512MB RAM each (horizontal + vertical scaling)
- C) Creates 3 databases
- D) Sets timeout to 3 minutes

**Answer: B**

---

**Q14.** The HTML5 Application Repository stores:
- A) Running Node.js applications
- B) Static web files (HTML, JS, CSS) — served without a running app
- C) Database tables
- D) Docker images

**Answer: B**

---

**Q15.** Managed Approuter vs Standalone: the main advantage is:
- A) Faster performance
- B) SAP manages it — zero memory cost, zero maintenance for you
- C) Better security
- D) More features

**Answer: B**

---

**Q16.** A BTP Destination connects:
- A) Two databases
- B) UI to backend — maps a name to a URL + authentication method
- C) Two users
- D) Git to BTP

**Answer: B**

---

**Q17.** `HTML5.ForwardAuthToken: true` is critical because:
- A) It encrypts the connection
- B) Without it, the backend doesn't receive the user's JWT (returns 401)
- C) It speeds up requests
- D) It enables caching

**Answer: B**

---

**Q18.** `cf html5-push` is used to:
- A) Build the application
- B) Update only UI files in HTML5 repo (without full MTA redeployment)
- C) Push to GitHub
- D) Create a HANA table

**Answer: B**

---

**Q19.** SAP Build Work Zone (Standard Edition) provides:
- A) A database service
- B) The Fiori Launchpad — a home page with tiles where users find all their apps
- C) A code editor
- D) Version control

**Answer: B**

---

**Q20.** In Work Zone, a "Content Provider" is:
- A) A user who creates content
- B) A source where Work Zone discovers available apps (like HTML5 repository)
- C) A payment gateway
- D) A data feed

**Answer: B**

---

**Q21.** To see your app as a tile on the launchpad, you must:
- A) Just deploy the app (automatic)
- B) Add it from Content Explorer → create a group → assign to a role → create a site
- C) Write HTML for the tile
- D) Email SAP support

**Answer: B**

---

**Q22.** The `crossNavigation.inbounds` in manifest.json enables:
- A) Cross-browser support
- B) Fiori Launchpad discovery (defines semantic object + action for the tile)
- C) Cross-origin requests
- D) Multi-language support

**Answer: B**

---

**Q23.** "Launchpad_Admin" role is needed to:
- A) Access any app
- B) Manage the Work Zone site (configure content, providers, roles)
- C) Deploy applications
- D) Create databases

**Answer: B**

---

**Q24.** If tiles show on launchpad but clicking one gives "App could not be opened":
- A) Reinstall the browser
- B) Check destination config, xs-app.json routes, and HTML5 app deployment
- C) Reboot the computer
- D) Wait 24 hours

**Answer: B**

---

**Q25.** The correct end-to-end flow for a user is:
- A) Login → BTP Cockpit → find app URL → copy paste
- B) Open Launchpad URL → Login → See tiles → Click tile → App loads from HTML5 repo → API via destination → Data from HANA
- C) SSH into server → run app
- D) Download app → install locally

**Answer: B**

---

**Q26.** `cf undeploy <mta-id> --delete-services` removes:
- A) Only the UI
- B) EVERYTHING: all apps, services, and bindings for that MTA
- C) Only the database
- D) Only logs

**Answer: B**

---

**Q27.** In the serverless model, how many CF apps run for the UI layer?
- A) 3 (approuter + UI + deployer)
- B) 0 — UI files are in HTML5 repo, no running app needed
- C) 1 (a static file server)
- D) 2 (approuter + UI)

**Answer: B**

---

**Q28.** Work Zone Site Directory is where you:
- A) Configure destinations
- B) Create and manage launchpad sites (the actual portal users access)
- C) Write code
- D) Deploy applications

**Answer: B**

---

**Q29.** For a tile to show for a specific user, you need:
- A) Nothing — all tiles show for everyone
- B) The app assigned to a role, and that role assigned to the user
- C) A special license
- D) Admin approval each time

**Answer: B**

---

**Q30.** The complete deployment architecture (in order) is:
- A) Git → BTP → Done
- B) CDS → cds build → mbt build (.mtar) → cf deploy → Create destinations → Configure Work Zone → User accesses launchpad
- C) Write HTML → Upload → Done
- D) Install SAP → Configure → Deploy

**Answer: B**

---

### Quiz Scoring

| Range | Grade | Feedback |
|-------|-------|----------|
| 27-30 | Excellent | Full-stack deployment mastery! |
| 21-26 | Good | Review specific missed areas |
| 15-20 | Fair | Re-study Days 31-34 hands-on |
| Below 15 | Needs Work | Go through the deployment pipeline step by step |

---

## Session 5 & 6: Mini Project — Full Deployment Pipeline (15:15 - 17:00)

### Project Brief

Execute the COMPLETE end-to-end deployment pipeline from source code to user-accessible launchpad tile.

---

### The Complete Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│              FULL DEPLOYMENT PIPELINE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. SOURCE CODE                                             │
│     └── db/ + srv/ + app/ + mta.yaml + xs-security.json    │
│                         │                                   │
│  2. BUILD               ▼                                   │
│     └── mbt build → po-management_1.0.0.mtar               │
│                         │                                   │
│  3. DEPLOY              ▼                                   │
│     └── cf deploy → services + apps running on CF           │
│                         │                                   │
│  4. SECURITY            ▼                                   │
│     └── BTP Cockpit → Role Collections → assign users       │
│                         │                                   │
│  5. DESTINATIONS        ▼                                   │
│     └── BTP Cockpit → Create/verify destination to backend  │
│                         │                                   │
│  6. LAUNCHPAD           ▼                                   │
│     └── Work Zone → Add app → Group → Role → Site → Open!  │
│                         │                                   │
│  7. USER ACCESS         ▼                                   │
│     └── User opens launchpad URL → sees tiles → uses app!  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Step-by-Step Execution

#### Phase 1: Verify Source Code (10 minutes)

```bash
cd ~/projects/po-management

# Verify all files exist:
ls mta.yaml                    # Deployment descriptor
ls xs-security.json            # Security config
ls db/schema.cds               # Data model
ls srv/purchasing-service.cds  # Service definition
ls srv/purchasing-service.js   # Handlers
ls app/products/               # Fiori app
```

---

#### Phase 2: Build (10 minutes)

```bash
# Build the MTA archive
mbt build

# Verify:
ls -lh mta_archives/*.mtar
# Should be 5-20 MB
```

---

#### Phase 3: Deploy (15 minutes)

```bash
# Login to CF
cf login -a https://api.cf.us10-001.hana.ondemand.com

# Verify HANA is running (BTP Cockpit)

# Deploy!
cf deploy mta_archives/po-management_1.0.0.mtar

# Verify:
cf apps      # All started?
cf services  # All created?
```

---

#### Phase 4: Security Setup (15 minutes)

```
BTP Cockpit → Security → Role Collections:
1. Find/Create "PO_AdminRC" → add Administrator role → assign your user
2. Find/Create "PO_ManagerRC" → add PurchaseManager role → assign your user
3. Find/Create "PO_ViewerRC" → add Viewer role
```

---

#### Phase 5: Destination (10 minutes)

```
BTP Cockpit → Connectivity → Destinations:
1. Create destination "po-management-api"
2. URL: (from cf app po-management-srv → routes)
3. Authentication: NoAuthentication
4. Properties: HTML5.ForwardAuthToken = true
5. Check Connection → 200 OK ✅
```

---

#### Phase 6: Work Zone Configuration (20 minutes)

```
1. Subscribe to SAP Build Work Zone (if not done)
2. Assign Launchpad_Admin + Launchpad_External_User roles
3. Open Site Manager
4. Refresh content provider
5. Content Explorer → add your app(s)
6. Create Group "PO Management"
7. Assign apps to "Everyone" role
8. Create Site → Open
```

---

#### Phase 7: End-to-End Test (25 minutes)

```
1. Open the launchpad URL
2. Login (redirects to SAP IAS / IdP)
3. See tiles on the homepage ✅
4. Click "Manage Products" tile → app opens ✅
5. Data loads from HANA via CAP backend ✅
6. Create a new product → Save ✅
7. Navigate back to launchpad (shell bar) ✅
8. Click "Purchase Orders" tile → app opens ✅
9. Create a PO → Submit → Verify workflow ✅
10. Celebrate! 🎉
```

---

### Final Verification Checklist

```
┌─────────────────────────────────────────────────────────────┐
│         COMPLETE DEPLOYMENT VERIFICATION                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  INFRASTRUCTURE:                                            │
│  □ HANA Cloud instance running                              │
│  □ All CF apps in "started" state (cf apps)                 │
│  □ All services created (cf services)                       │
│  □ No errors in logs (cf logs <app> --recent)               │
│                                                             │
│  SECURITY:                                                  │
│  □ XSUAA service created with correct roles                 │
│  □ Role Collections exist in BTP Cockpit                    │
│  □ Roles assigned to test user(s)                           │
│  □ Login works (no 401)                                     │
│  □ Authorization works per role (403 when expected)          │
│                                                             │
│  CONNECTIVITY:                                              │
│  □ Destination created and connection check passes          │
│  □ HTML5.ForwardAuthToken = true set                        │
│  □ App loads data from backend (not empty screens)          │
│                                                             │
│  LAUNCHPAD:                                                 │
│  □ Work Zone subscribed and accessible                      │
│  □ Content provider refreshed — apps discovered             │
│  □ Apps added to content, assigned to group and role         │
│  □ Site created and accessible                              │
│  □ Tiles visible for the assigned user                      │
│  □ Clicking tile opens the app successfully                 │
│  □ App is fully functional (CRUD, actions work)             │
│                                                             │
│  COMPLETE! Your application is PRODUCTION-READY! 🚀        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## End of Day Summary — Week 7 Complete!

### What We Covered This Week (Days 31-35)

| Day | Topic | Key Takeaway |
|-----|-------|-------------|
| 31 | App Router & MTA | Approuter is the front door; mta.yaml is the deployment blueprint |
| 32 | Build & Deploy to CF | `mbt build` packages → `cf deploy` ships to cloud |
| 33 | Testing & Role Assignment | Create Role Collections in BTP Cockpit → assign to users |
| 34 | HTML5 Repo & Serverless | UI stored in HTML5 repo → served without running app = savings! |
| 35 | Work Zone & Launchpad | Fiori Launchpad — users find all apps as tiles in one place |

---

### The Complete Architecture (What You've Built)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  USER → Launchpad (Work Zone)                               │
│              │                                              │
│              ├── Tiles (role-based)                         │
│              │                                              │
│              ▼ click tile                                    │
│         Managed Approuter                                   │
│              │                                              │
│              ├── Static files ← HTML5 App Repository        │
│              │                                              │
│              └── API calls → Destination → CAP Backend      │
│                                              │              │
│                                              ├── Handlers   │
│                                              ├── XSUAA      │
│                                              └── HANA Cloud │
│                                                             │
│  LAYERS: Launchpad → UI → Routing → Backend → Database     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### The Week 7 One-Liner

> **From `mbt build` to a tile on the Fiori Launchpad — your full-stack CAP application is now deployed, secured, serverless, and accessible to real business users.**

---

### Looking Ahead: Days 36-45

The final stretch covers:
- Advanced CAP topics (remote services, messaging)
- Performance optimization
- CI/CD pipelines
- Real-world project patterns
- Capstone project: Build a complete enterprise application end-to-end!

---

*End of Day 35 — Week 7 Complete! You've mastered the entire deployment lifecycle! 🎉🚀*
