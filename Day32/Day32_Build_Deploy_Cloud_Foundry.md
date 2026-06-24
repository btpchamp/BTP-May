# Day 32: Build & Deploy to Cloud Foundry

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 31 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: MTA Build Tool & Building the Archive | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Cloud Foundry CLI & Deployment Process | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Build MTAR & Deploy | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Monitoring, Logs & Service Binding | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Troubleshooting Deployment Failures | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Test Deployed Application | 45 min |
| 16:45 - 17:00 | Assessment & Troubleshooting Quiz | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Use the MTA Build Tool (`mbt`) to create a deployable archive
- Authenticate to Cloud Foundry using the CF CLI
- Deploy your application using `cf deploy`
- Monitor deployment progress and check app status
- View and interpret application logs using `cf logs`
- Understand how services get bound to applications during deployment
- Troubleshoot common deployment failures by reading error messages
- Test your deployed application end-to-end in the cloud

---

## Day 31 Recap — Quick Fire (09:00 - 09:15)

1. The approuter's main job is _____? → _____
2. `xs-app.json` defines _____? → _____
3. `forwardAuthToken: true` ensures _____? → _____
4. MTA stands for _____? → _____
5. In mta.yaml, modules are _____, resources are _____? → _____
6. `requires` in a module means _____? → _____
7. `provides` in a module means _____? → _____
8. The command to generate mta.yaml is _____? → _____

<details>
<summary>Answers</summary>

1. **Single entry point** — handles auth, routing, and token injection
2. **Routing rules** — which URL patterns go to which destinations
3. The **JWT token is passed** from approuter to the backend
4. **Multi-Target Application**
5. Modules = **your deployable code**; Resources = **BTP services** (HANA, XSUAA)
6. "I **depend on** this service — bind it to me"
7. "I **offer** something (like my URL) for others to consume"
8. `cds add mta`

</details>

---

## Session 1: MTA Build Tool & Building the Archive (09:15 - 10:30)

### The Deployment Pipeline

```
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│  SOURCE  │      │  BUILD   │      │  ARCHIVE │      │  DEPLOY  │
│  CODE    │─────►│  (mbt)   │─────►│  (.mtar) │─────►│  (cf)    │
│          │      │          │      │          │      │          │
│ mta.yaml │      │ npm ci   │      │ Single   │      │ Creates  │
│ db/      │      │ cds build│      │ ZIP file │      │ services │
│ srv/     │      │ package  │      │ with     │      │ Deploys  │
│ app/     │      │          │      │ everything│     │ apps     │
└──────────┘      └──────────┘      └──────────┘      └──────────┘

  You write code → mbt packages it → cf deploys it to the cloud
```

---

### What is the MTA Build Tool (mbt)?

**mbt** = **MTA Build Tool** — a command-line tool that reads your `mta.yaml` and creates a deployable archive (`.mtar` file).

**What it does:**
1. Reads `mta.yaml` to understand your project structure
2. Runs build commands for each module (npm install, cds build)
3. Packages everything into a single `.mtar` file (like a ZIP)
4. The `.mtar` file contains everything needed for deployment

---

### Installing mbt

```bash
# Install MTA Build Tool globally
npm install -g mbt

# Verify installation
mbt --version
# Output: v1.2.x (or similar)
```

**Alternative:** In SAP Business Application Studio, `mbt` is pre-installed.

---

### The Build Command

```bash
# From your project root (where mta.yaml lives):
mbt build
```

**What happens during build:**

```
$ mbt build

[2026-06-05 09:30:00] INFO  generating the "Makefile.mta"...
[2026-06-05 09:30:00] INFO  executing the "make -f Makefile.mta p=cf" command...
[2026-06-05 09:30:01] INFO  executing the "before-all" build...
[2026-06-05 09:30:01] INFO    executing: npm ci
[2026-06-05 09:30:15] INFO    executing: npx cds build --production
[2026-06-05 09:30:20] INFO  building module "po-management-db-deployer"...
[2026-06-05 09:30:22] INFO  building module "po-management-srv"...
[2026-06-05 09:30:25] INFO  building module "po-management-approuter"...
[2026-06-05 09:30:28] INFO  generating the MTA archive...

[2026-06-05 09:30:30] INFO  the MTA archive generated at:
                             mta_archives/po-management_1.0.0.mtar
```

---

### What's Inside the .mtar Archive?

```
po-management_1.0.0.mtar (ZIP archive):
├── META-INF/
│   ├── MANIFEST.MF          ← Module metadata
│   └── mtad.yaml            ← Deployment descriptor
├── po-management-db-deployer/
│   ├── package.json
│   ├── src/
│   │   ├── Products.hdbtable
│   │   ├── Orders.hdbtable
│   │   └── ... (all HANA artifacts)
│   └── data/
│       └── *.csv             ← Initial data
├── po-management-srv/
│   ├── package.json
│   ├── node_modules/         ← Dependencies
│   ├── srv/
│   │   ├── catalog-service.js
│   │   └── purchasing-service.js
│   └── ... (compiled service code)
└── po-management-approuter/
    ├── package.json
    ├── node_modules/
    └── xs-app.json           ← Routing config
```

**The .mtar is a self-contained deployment package.** Everything needed to run the app is inside it.

---

### Build Options

```bash
# Standard build
mbt build

# Build with verbose output (for debugging)
mbt build --mtar my-archive.mtar

# Build only specific modules
mbt build --modules po-management-srv

# Build with a specific platform target
mbt build --platform cf        # Cloud Foundry (default)
```

---

### Pre-Build Checklist

Before running `mbt build`, verify:

```bash
# 1. mta.yaml exists in project root
ls mta.yaml

# 2. All modules have correct paths
# Check that gen/db, gen/srv exist (or will be created by cds build)

# 3. package.json has correct dependencies
cat package.json | grep -A3 "dependencies"

# 4. xs-security.json exists (for XSUAA resource)
ls xs-security.json

# 5. No build errors in CDS
cds build --production

# 6. Everything committed (good practice)
git status
```

---

## Session 2: Cloud Foundry CLI & Deployment Process (10:45 - 12:00)

### What is Cloud Foundry?

**Cloud Foundry (CF)** is the runtime platform on SAP BTP where your application actually runs. Think of it as the "server room in the cloud."

```
SAP BTP
└── Cloud Foundry Environment
    └── Organization (your company)
        └── Space (e.g., "dev", "test", "prod")
            └── Your apps run here!
```

---

### Installing Cloud Foundry CLI

```bash
# Check if CF CLI is installed
cf --version
# Output: cf version 8.x.x

# If not installed:
# macOS:
brew install cloudfoundry/tap/cf-cli@8

# Windows: Download from https://github.com/cloudfoundry/cli/releases

# Linux:
wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
sudo apt-get install cf8-cli
```

---

### CF CLI — Essential Commands

| Command | What It Does |
|---------|-------------|
| `cf login` | Authenticate to Cloud Foundry |
| `cf target` | Show/set current org and space |
| `cf apps` | List all deployed applications |
| `cf services` | List all service instances |
| `cf deploy` | Deploy an MTAR archive |
| `cf logs <app>` | View application logs |
| `cf env <app>` | View app environment variables |
| `cf stop <app>` | Stop an application |
| `cf start <app>` | Start an application |
| `cf restart <app>` | Restart an application |
| `cf delete <app>` | Delete an application |
| `cf undeploy <mta-id>` | Remove entire MTA deployment |

---

### Step 1: Login to Cloud Foundry

```bash
# Login (interactive — prompts for email/password)
cf login

# Or specify the API endpoint directly:
cf login -a https://api.cf.us10-001.hana.ondemand.com

# You'll be prompted:
# Email: your-email@company.com
# Password: ********
# Select org: trial_org
# Select space: dev
```

**Finding your CF API endpoint:**
1. BTP Cockpit → Your Subaccount → Overview
2. Look for "Cloud Foundry Environment" → "API Endpoint"

```
Common BTP Trial endpoints:
  US: https://api.cf.us10-001.hana.ondemand.com
  EU: https://api.cf.eu10-004.hana.ondemand.com
  AP: https://api.cf.ap21.hana.ondemand.com
```

---

### Step 2: Verify Your Target

```bash
cf target

# Output:
# API endpoint:   https://api.cf.us10-001.hana.ondemand.com
# User:           your-email@company.com
# Org:            your-org-trial
# Space:          dev
```

---

### Step 3: Deploy the MTAR Archive

```bash
# Deploy the built archive
cf deploy mta_archives/po-management_1.0.0.mtar

# With retry on failures:
cf deploy mta_archives/po-management_1.0.0.mtar --retries 2
```

---

### What Happens During `cf deploy`

```
$ cf deploy mta_archives/po-management_1.0.0.mtar

Deploying multi-target app archive mta_archives/po-management_1.0.0.mtar...

  ① Uploading archive to Cloud Foundry...
     Uploading 12.5 MB... done

  ② Processing resources...
     Creating service "po-management-db" (hdi-shared)...        ✅
     Creating service "po-management-auth" (xsuaa/application)... ✅

  ③ Deploying module "po-management-db-deployer"...
     Staging... done
     Starting (deploying to HANA)... done
     Database schema deployed successfully!
     Stopping deployer (one-time task)... done                  ✅

  ④ Deploying module "po-management-srv"...
     Staging... done
     Starting application... 
     Binding "po-management-db"... done
     Binding "po-management-auth"... done
     Application started successfully!                          ✅
     URL: https://po-management-srv.cfapps.us10.hana.ondemand.com

  ⑤ Deploying module "po-management-approuter"...
     Staging... done
     Starting application...
     Binding "po-management-auth"... done
     Configuring destination "srv-api"... done
     Application started successfully!                          ✅
     URL: https://po-management.cfapps.us10.hana.ondemand.com

Process finished successfully.
```

---

### Deployment Order

Cloud Foundry deploys in dependency order:

```
1. RESOURCES first (create services):
   ├── HANA HDI Container (database)
   └── XSUAA Instance (authentication)

2. DB DEPLOYER second (needs HANA):
   └── Creates tables, loads CSV data

3. BACKEND third (needs HANA + XSUAA):
   └── Starts the CAP Node.js server

4. APPROUTER last (needs backend URL + XSUAA):
   └── Routes requests to backend
```

**Why this order?** Each component depends on the previous one. You can't start the backend without a database!

---

### Step 4: Verify Deployment

```bash
# List all apps
cf apps

# Output:
# name                          state     instances   memory   disk   urls
# po-management-srv             started   1/1         256M     1G     po-management-srv.cfapps.us10...
# po-management-approuter       started   1/1         256M     512M   po-management.cfapps.us10...

# List all services
cf services

# Output:
# name                    service    plan          bound apps
# po-management-db        hana       hdi-shared    po-management-srv, po-management-db-deployer
# po-management-auth      xsuaa      application   po-management-srv, po-management-approuter
```

---

### Step 5: Access Your Deployed App

```bash
# Get the approuter URL
cf app po-management-approuter | grep routes

# Output:
# routes: po-management.cfapps.us10.hana.ondemand.com
```

Open in browser: `https://po-management.cfapps.us10.hana.ondemand.com`

You'll be redirected to the login page → enter credentials → see your Fiori app! 🎉

---

## Session 3: Hands-on — Build MTAR & Deploy (12:00 - 13:00)

### Exercise: Complete Build and Deployment

```bash
# ═══ STEP 1: Verify prerequisites ═══
cd ~/projects/po-management

# Check mta.yaml exists
cat mta.yaml | head -5

# Check xs-security.json exists
cat xs-security.json | head -3

# ═══ STEP 2: Build the archive ═══
mbt build

# Verify the archive was created:
ls mta_archives/
# Expected: po-management_1.0.0.mtar

# Check file size (should be 5-20 MB)
ls -lh mta_archives/*.mtar

# ═══ STEP 3: Login to Cloud Foundry ═══
cf login -a https://api.cf.us10-001.hana.ondemand.com
# Enter email and password
# Select org and space

# Verify:
cf target

# ═══ STEP 4: Deploy ═══
cf deploy mta_archives/po-management_1.0.0.mtar

# Wait... (3-10 minutes for first deployment)

# ═══ STEP 5: Verify ═══
cf apps
cf services

# ═══ STEP 6: Get the URL ═══
cf app po-management-approuter | grep routes

# ═══ STEP 7: Test in browser ═══
# Open the URL → login → verify your app works!
```

---

### If Build Fails: Quick Fixes

| Error Message | Fix |
|--------------|-----|
| `mta.yaml not found` | Are you in the project root? `ls mta.yaml` |
| `npm ci failed` | Delete `node_modules` and `package-lock.json`, run `npm install` |
| `cds build failed` | Run `cds build --production` separately to see CDS errors |
| `Module path not found` | Check `path:` in mta.yaml points to correct folders |

---

### If Deploy Fails: Quick Fixes

| Error Message | Fix |
|--------------|-----|
| `Not authorized` | Run `cf login` again (token may have expired) |
| `Service creation failed` | Check BTP entitlements (do you have HANA/XSUAA?) |
| `Out of memory` | Increase `memory` in mta.yaml or free space with `cf delete` |
| `App crashed` | Check logs: `cf logs po-management-srv --recent` |
| `HANA connection failed` | Is your HANA Cloud instance running? |

---

## Session 4: Monitoring, Logs & Service Binding (13:45 - 14:45)

### Viewing Logs

```bash
# View recent logs (last ~100 lines)
cf logs po-management-srv --recent

# Stream logs in real-time (live tail)
cf logs po-management-srv
# Press Ctrl+C to stop

# View logs for the approuter
cf logs po-management-approuter --recent
```

---

### Reading Log Output

```
2026-06-05T09:30:00.000+00:00 [APP/PROC/WEB/0] OUT [cds] - serving PurchasingService { path: '/purchasing' }
2026-06-05T09:30:00.100+00:00 [APP/PROC/WEB/0] OUT [cds] - serving CatalogService { path: '/catalog' }
2026-06-05T09:30:00.200+00:00 [APP/PROC/WEB/0] OUT [cds] - connect to hana { credentials: { ... } }
2026-06-05T09:30:01.000+00:00 [APP/PROC/WEB/0] OUT [cds] - server listening on { url: 'http://0.0.0.0:8080' }
```

**Log format:**
```
TIMESTAMP [SOURCE] LEVEL MESSAGE

SOURCE types:
  APP/PROC/WEB/0  → Your application output (console.log)
  API             → Cloud Foundry API events
  RTR             → Router (HTTP requests)
  CELL            → Container events (start/stop/crash)
  STG             → Staging (build) events
```

---

### Useful Log Patterns to Watch For

```bash
# Filter for errors only
cf logs po-management-srv --recent | grep -i "error"

# Filter for HTTP requests
cf logs po-management-srv --recent | grep "RTR"

# See startup messages
cf logs po-management-srv --recent | grep "cds"
```

**Common patterns in logs:**

| Log Pattern | Meaning |
|-------------|---------|
| `[cds] - server listening on` | App started successfully ✅ |
| `[cds] - connect to hana` | HANA connection established ✅ |
| `Error: ECONNREFUSED` | Can't connect to HANA (is it running?) |
| `401 Unauthorized` | Authentication token missing/invalid |
| `403 Forbidden` | User doesn't have the right role |
| `CRASHED` | App failed to start (check logs for why) |
| `OUT OF MEMORY` | App needs more RAM (increase in mta.yaml) |

---

### Service Binding — How Apps Connect to Services

When `cf deploy` runs, it **binds** services to apps. This provides connection details as environment variables:

```bash
# See all environment variables (including service bindings)
cf env po-management-srv
```

Output (simplified):
```json
{
  "VCAP_SERVICES": {
    "hana": [{
      "name": "po-management-db",
      "credentials": {
        "host": "abc123.hana.trial-us10.hanacloud.ondemand.com",
        "port": "443",
        "user": "HDI_CONTAINER_USER",
        "password": "auto-generated-password",
        "schema": "PO_MANAGEMENT_DB_HDI_1234"
      }
    }],
    "xsuaa": [{
      "name": "po-management-auth",
      "credentials": {
        "clientid": "sb-po-management!t12345",
        "clientsecret": "auto-generated-secret",
        "url": "https://auth.us10.hana.ondemand.com"
      }
    }]
  }
}
```

**CAP reads these automatically!** You never hardcode database URLs or passwords. Cloud Foundry provides them via `VCAP_SERVICES`.

---

### Checking App Health

```bash
# App overview (memory, instances, status)
cf app po-management-srv

# Output:
# name:              po-management-srv
# requested state:   started
# routes:            po-management-srv.cfapps.us10.hana.ondemand.com
# last uploaded:     Thu 05 Jun 09:30:00 UTC 2026
# stack:             cflinuxfs4
# instances:         1/1
#      state     since                  cpu    memory        disk
# #0   running   2026-06-05T09:30:05Z   0.3%   125M of 256M  200M of 1G
```

**Key things to check:**
- `state: running` → app is healthy
- `instances: 1/1` → requested instance is running
- `memory: 125M of 256M` → not running out of RAM
- `disk: 200M of 1G` → not running out of disk

---

### Managing Your Deployed App

```bash
# Stop the app (saves resources when not testing)
cf stop po-management-srv
cf stop po-management-approuter

# Start it again
cf start po-management-srv
cf start po-management-approuter

# Restart (pick up env changes)
cf restart po-management-srv

# Scale up (more instances for load)
cf scale po-management-srv -i 2    # Run 2 instances

# Scale memory
cf scale po-management-srv -m 512M  # Increase to 512MB RAM

# Delete everything (clean up trial)
cf undeploy po-management --delete-services --delete-service-keys
```

---

## Session 5: Troubleshooting Deployment Failures (15:00 - 16:00)

### The Troubleshooting Mindset

```
Deployment failed? → Don't panic! Follow this process:

1. READ the error message (it usually tells you exactly what's wrong)
2. CHECK the logs (cf logs <app> --recent)
3. VERIFY prerequisites (HANA running? Correct login? Entitlements?)
4. SEARCH for the error (SAP Community, Stack Overflow)
5. FIX and RETRY (cf deploy again)
```

---

### Top 10 Deployment Errors & Solutions

#### Error 1: "Insufficient resources / Quota exceeded"

```
Error: You have exceeded your organization's memory limit
```

**Cause:** BTP Trial has limited memory (usually 2-4 GB total).

**Fix:**
```bash
# Check current usage
cf apps   # See memory per app

# Stop apps you're not using
cf stop old-test-app

# Delete old deployments
cf undeploy old-project --delete-services

# Reduce memory in mta.yaml
parameters:
  memory: 128M   # Reduce from 256M
```

---

#### Error 2: "Service instance creation failed"

```
Error: Could not create service instance "po-management-db"
```

**Causes:**
- HANA Cloud instance not running
- Service plan not available in your space
- Already at service limit

**Fix:**
```bash
# Check available service plans
cf marketplace -e hana

# Check if HANA is running (BTP Cockpit → HANA Cloud → Status)

# Check existing services (maybe already exists from a failed deploy)
cf services
cf delete-service po-management-db   # Delete old one, retry
```

---

#### Error 3: "App crashed during startup"

```
state: crashed
ERR Error: Cannot find module '@sap/cds'
```

**Cause:** Missing dependencies (npm install didn't run properly during build).

**Fix:**
```bash
# Clean and rebuild
rm -rf node_modules package-lock.json
npm install
mbt build
cf deploy mta_archives/*.mtar
```

---

#### Error 4: "HANA deployment failed"

```
Error: deploy to container failed: table COM_EPM_PRODUCTS already exists
```

**Fix:**
```bash
# Redeploy with auto-undeploy option
cf deploy mta_archives/*.mtar --retries 1

# Or delete and recreate the HDI container
cf delete-service po-management-db -f
cf deploy mta_archives/*.mtar
```

---

#### Error 5: "Authentication error / Invalid redirect URI"

```
Error: Invalid redirect_uri
```

**Cause:** The XSUAA service has wrong OAuth callback URL.

**Fix:** Check `xs-security.json`:
```json
{
  "oauth2-configuration": {
    "redirect-uris": [
      "https://*.cfapps.us10.hana.ondemand.com/**"
    ]
  }
}
```

---

#### Error 6: "502 Bad Gateway" when accessing the app

**Cause:** Backend app isn't running or approuter can't reach it.

**Fix:**
```bash
# Check if backend is running
cf app po-management-srv

# If crashed, check logs
cf logs po-management-srv --recent

# Restart
cf restart po-management-srv
```

---

#### Error 7: "Buildpack compilation failed"

```
Error: Staging failed: buildpack compilation step failed
```

**Cause:** Wrong Node.js version or incompatible dependencies.

**Fix:** Add engine specification to `package.json`:
```json
{
  "engines": {
    "node": "^18 || ^20"
  }
}
```

---

#### Error 8: "mbt build fails with module not found"

```
Error: could not find module "gen/db"
```

**Cause:** `cds build --production` hasn't been run (gen/ folder doesn't exist).

**Fix:** mta.yaml should include `cds build` in before-all:
```yaml
build-parameters:
  before-all:
    - builder: custom
      commands:
        - npm ci
        - npx cds build --production
```

---

#### Error 9: "Timeout during staging"

**Cause:** Build takes too long (large node_modules, slow network).

**Fix:**
```bash
# Increase staging timeout
cf config --staging-timeout 30   # 30 minutes

# Or ensure package-lock.json is committed (faster npm ci)
```

---

#### Error 10: "App started but shows 'Cannot GET /'"

**Cause:** Approuter can't find the welcome file.

**Fix:** Check `xs-app.json`:
```json
{
  "welcomeFile": "/index.html"
}
```
Ensure the file exists in the approuter's `resources/` directory (or HTML5 repo).

---

### Quick Troubleshooting Reference

```
┌─────────────────────────────────────────────────────────────┐
│  DEPLOYMENT TROUBLESHOOTING FLOWCHART                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Build failed?                                              │
│  ├── Check: mta.yaml exists? paths correct?                │
│  ├── Run: cds build --production (separately)              │
│  └── Run: npm ci (in project root)                         │
│                                                             │
│  Deploy failed?                                             │
│  ├── "Not authorized" → cf login again                     │
│  ├── "Quota exceeded" → delete old apps/services           │
│  ├── "Service failed" → check HANA running + marketplace   │
│  └── "App crashed" → cf logs <app> --recent                │
│                                                             │
│  App running but errors?                                    │
│  ├── 401/403 → check XSUAA binding + roles                │
│  ├── 502 → backend not reachable (restart it)             │
│  ├── Empty data → HANA deployed but no CSV loaded          │
│  └── "Cannot GET /" → xs-app.json welcomeFile wrong        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Session 6: Hands-on — Test Deployed Application (16:00 - 16:45)

### Exercise: End-to-End Testing in the Cloud

After successful deployment, test your application:

---

**Test 1: Access the Application**
```bash
# Get the approuter URL
cf app po-management-approuter | grep routes
# Open in browser: https://po-management.cfapps.us10.hana.ondemand.com
```

Expected: Redirected to login → enter credentials → see Fiori Launchpad or app

---

**Test 2: Check Backend Directly**
```bash
# Get backend URL
cf app po-management-srv | grep routes
# Try: https://po-management-srv.cfapps.us10.hana.ondemand.com/catalog/Products
```

Expected: Should require authentication (401 if accessed directly without token)

---

**Test 3: Verify Data Exists**

If your CSV data was included in the deployment:
1. Open the app via approuter (authenticated)
2. Navigate to Products → should see data from CSVs
3. Navigate to Purchase Orders → may be empty (create one!)

---

**Test 4: Test CRUD Operations**
1. Create a new Purchase Order through the Fiori UI
2. Add items
3. Save (draft → activate)
4. Submit for approval
5. Verify status changed

---

**Test 5: Check Logs During Usage**
```bash
# In a terminal, stream logs while using the app:
cf logs po-management-srv

# In another browser tab, use the application
# Watch the terminal — you'll see each request logged!
```

---

**Test 6: Verify Security**

Test with different role collections assigned in BTP Cockpit:
1. Assign "PO_Viewer" role collection to a user → can only read
2. Assign "PO_Manager" role collection → can create and approve
3. Remove all roles → gets 403 Forbidden

---

### Cleanup (When Done Testing)

```bash
# Stop apps to save trial quota
cf stop po-management-srv
cf stop po-management-approuter

# Or remove everything completely
cf undeploy po-management --delete-services --delete-service-keys

# Verify cleanup
cf apps       # Should show no apps
cf services   # Should show no services
```

---

## Assessment: MCQ — 15 Questions on Build & Deployment

**Q1.** `mbt build` creates:

- A) A Docker container
- B) An .mtar archive containing all modules ready for deployment
- C) A GitHub repository
- D) A HANA database

**Answer: B** — MBT packages all modules into a single `.mtar` archive that `cf deploy` can process.

---

**Q2.** The command to authenticate to Cloud Foundry:

- A) `cf authenticate`
- B) `cf login`
- C) `cf connect`
- D) `btp login`

**Answer: B** — `cf login` prompts for credentials and sets org/space target.

---

**Q3.** `cf deploy mta_archives/*.mtar` does what?

- A) Only uploads the file
- B) Creates services, deploys modules, binds everything — full deployment
- C) Only creates the database
- D) Only starts the approuter

**Answer: B** — One command handles the entire deployment: services + modules + bindings.

---

**Q4.** During deployment, which gets created FIRST?

- A) Approuter
- B) Backend service
- C) Resources (HANA, XSUAA) then DB deployer
- D) UI files

**Answer: C** — Resources first (services must exist), then DB deployer (tables needed), then backend, then approuter.

---

**Q5.** `cf logs po-management-srv --recent` shows:

- A) Build logs
- B) Recent application output/error logs
- C) mta.yaml content
- D) Memory usage graph

**Answer: B** — Shows the last ~100 lines of app logs (stdout/stderr, HTTP requests, errors).

---

**Q6.** `VCAP_SERVICES` environment variable contains:

- A) Your application source code
- B) Connection credentials for all bound services (HANA URL, XSUAA secrets)
- C) User login information
- D) The xs-app.json content

**Answer: B** — Cloud Foundry provides service credentials via VCAP_SERVICES. CAP reads them automatically.

---

**Q7.** If an app status shows "crashed", the first thing to do is:

- A) Delete and redeploy
- B) Check logs: `cf logs <app-name> --recent`
- C) Increase memory
- D) Contact SAP support

**Answer: B** — Always check logs first. The crash reason (missing module, connection error, etc.) is usually right there.

---

**Q8.** "Quota exceeded" during deployment means:

- A) Too many files
- B) Your BTP space doesn't have enough memory/services available
- C) Wrong password
- D) Git repository is full

**Answer: B** — Trial accounts have limits. Delete old apps/services or reduce memory allocations.

---

**Q9.** The DB deployer module (`type: hdb`) does what during deployment?

- A) Starts a web server
- B) Creates tables/views in HANA and loads CSV data, then stops
- C) Runs forever serving requests
- D) Creates the XSUAA configuration

**Answer: B** — It's a one-time task: deploy schema + data to HANA, then exit. It doesn't run continuously.

---

**Q10.** `cf undeploy po-management --delete-services` does what?

- A) Stops the app
- B) Removes ALL deployed modules AND their service instances (complete cleanup)
- C) Only removes the UI
- D) Backs up the data

**Answer: B** — Complete removal: stops apps, deletes apps, deletes bound services. Clean slate.

---

**Q11.** Service binding is the process of:

- A) Uploading code to the cloud
- B) Connecting an app to a service instance (providing credentials via env vars)
- C) Creating a Git branch
- D) Building the MTAR

**Answer: B** — Binding = Cloud Foundry gives your app the connection details for a service (HANA, XSUAA) as environment variables.

---

**Q12.** If the approuter shows "502 Bad Gateway":

- A) The login page is broken
- B) The backend app isn't running or reachable (check with `cf app <srv-name>`)
- C) The database is full
- D) The browser cache needs clearing

**Answer: B** — 502 means the approuter can't forward to the backend. Check if backend is running.

---

**Q13.** `cf apps` shows your app with `0/1` instances. This means:

- A) App is running perfectly
- B) App has crashed — 0 out of 1 requested instances are running
- C) App hasn't been deployed yet
- D) App is being scaled down

**Answer: B** — `0/1` means Cloud Foundry tried to run 1 instance but it crashed. Check logs.

---

**Q14.** Where is the deployed app's URL found?

- A) In mta.yaml only
- B) In `cf app <app-name>` output under "routes"
- C) In xs-security.json
- D) In package.json

**Answer: B** — `cf app <name>` shows the app's route (URL) along with status, memory, instances.

---

**Q15.** After updating code, to redeploy you must:

- A) Just `cf restart`
- B) `mbt build` (new archive) → `cf deploy` (deploy updated archive)
- C) Delete everything and start from scratch
- D) Only `cf push`

**Answer: B** — Code changes require rebuilding the archive (`mbt build`) and redeploying (`cf deploy`).

---

## Troubleshooting Quiz

### Identify the Problem from These Log Snippets

**Snippet 1:**
```
ERR Error: connect ECONNREFUSED 10.0.1.25:443
    at TCPConnectWrap.afterConnect
```

What's the issue? → _____

<details><summary>Answer</summary>HANA Cloud instance is not running (connection refused). Go to BTP Cockpit → start the HANA instance.</details>

---

**Snippet 2:**
```
ERR Error: Cannot find module '@sap/cds'
    Require stack: /home/vcap/app/srv/server.js
```

What's the issue? → _____

<details><summary>Answer</summary>Dependencies not installed properly. The `@sap/cds` package is missing. Fix: check package.json dependencies and rebuild with `mbt build`.</details>

---

**Snippet 3:**
```
OUT [cds] - connect to hana { database: 'PO_MGMT_DB' }
ERR Error: authentication failed [390]: invalid username or password
```

What's the issue? → _____

<details><summary>Answer</summary>Service binding credentials are wrong or stale. Fix: `cf unbind-service`, then `cf bind-service` again, then `cf restart`.</details>

---

**Snippet 4:**
```
CELL   stopping instance (reason: CRASHED)
CELL   destroying container
API    App instance exited with guid ... (reason: CRASHED, exit: 137)
```

What's the issue? → _____

<details><summary>Answer</summary>Exit code 137 = killed by OOM (Out Of Memory). The app needs more RAM. Increase `memory` in mta.yaml (e.g., from 256M to 512M).</details>

---

**Snippet 5:**
```
OUT [HPM] Error occurred while trying to proxy request /purchasing/PurchaseOrders
    ECONNRESET
```

What's the issue? → _____

<details><summary>Answer</summary>The approuter can't reach the backend service. Backend may have crashed or the destination URL is wrong. Check: `cf app po-management-srv` and verify the `srv-api` destination in the approuter binding.</details>

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| `mbt build` | Creates .mtar archive from mta.yaml (packages everything) |
| .mtar file | Self-contained deployment archive (all modules + dependencies) |
| `cf login` | Authenticate to Cloud Foundry |
| `cf deploy` | Deploys the .mtar — creates services, deploys apps, binds together |
| Deployment order | Resources → DB deployer → Backend → Approuter |
| `cf apps` | Check status of deployed applications |
| `cf logs` | View application logs (essential for debugging!) |
| VCAP_SERVICES | Environment variable with service credentials (auto-injected) |
| Service binding | Connecting apps to services (credentials in env vars) |
| Troubleshooting | Read errors → check logs → verify prerequisites → fix → retry |
| `cf undeploy` | Complete cleanup (remove everything) |

### The Deployment Cheat Sheet

```bash
# BUILD:
mbt build                                    # Create .mtar

# DEPLOY:
cf login -a <api-endpoint>                   # Login
cf deploy mta_archives/*.mtar                # Deploy everything

# MONITOR:
cf apps                                      # App status
cf services                                  # Service status
cf logs <app-name> --recent                  # Check logs
cf app <app-name>                            # Details + URL

# MANAGE:
cf restart <app-name>                        # Restart app
cf scale <app-name> -m 512M                  # More memory
cf stop <app-name>                           # Stop (save resources)

# CLEANUP:
cf undeploy <mta-id> --delete-services       # Remove everything
```

### The One-Liner

> **`mbt build` packages your app. `cf deploy` ships it to the cloud. `cf logs` tells you if it's happy. That's the entire deployment lifecycle.**

---

### Looking Ahead: Day 33

Tomorrow: **Testing & Monitoring in Production**
- Integration testing strategies
- End-to-end testing on deployed apps
- Monitoring and alerting
- Performance basics
- Week 7 review preparation

---

*End of Day 32 — Your application is now LIVE in the cloud! 🚀*
