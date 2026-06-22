# Day 29: Authentication & Authorization in SAP BTP

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 28 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Authentication vs Authorization — The Fundamentals | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: XSUAA, JWT Tokens & xs-security.json | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Create xs-security.json & Define Roles | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: @requires and @restrict in CAP | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Secure Services & Test with Mock Users | 60 min |
| 16:00 - 16:45 | Session 6: End-to-End Security Testing | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Clearly explain the difference between Authentication and Authorization
- Understand how Identity Providers (IdP), SAML, and OAuth2 work together
- Describe XSUAA and its role in SAP BTP security
- Read and understand JWT token structure
- Create an `xs-security.json` file with scopes, roles, and role collections
- Use `@requires` to protect entire services
- Use `@restrict` for fine-grained, field-level access control
- Test security with mock users locally

---

## Day 28 Recap — Quick Fire (09:00 - 09:15)

1. Database Explorer is a _____ tool for browsing HANA? → _____
2. HANA SQL uses `_____` instead of LIMIT for top N? → _____
3. CDS `String(100)` becomes what HANA type? → _____
4. The `DUMMY` table is used for _____? → _____
5. UUID in HANA is stored as _____? → _____
6. If CSV loading fails, first check _____, _____, and _____? → _____
7. `M_TABLES` is a _____ view showing _____? → _____
8. `cds build --production` generates _____ files? → _____

<details>
<summary>Answers</summary>

1. **Web-based** tool
2. `TOP N` (e.g., `SELECT TOP 5`)
3. `NVARCHAR(100)` (Unicode)
4. Calculations without a real table (`SELECT 1+1 FROM DUMMY`)
5. `NVARCHAR(36)` — just a 36-character string
6. **File naming**, **semicolons** (separator), and **column headers**
7. **System monitoring** view showing table **metadata** (row counts, memory)
8. `.hdbtable` and `.hdbview` (HANA design-time artifacts)

</details>

---

## Session 1: Authentication vs Authorization — The Fundamentals (09:15 - 10:30)

### The Building Analogy

Imagine you're entering a corporate office building:

```
┌─────────────────────────────────────────────────────────────┐
│                    OFFICE BUILDING                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  STEP 1: AUTHENTICATION (at the front door)                 │
│  ┌───────────────────────────────────────────────┐          │
│  │  Security Guard: "WHO are you?"               │          │
│  │  You: Show your ID badge                      │          │
│  │  Guard: "OK, you are John Smith. Come in."    │          │
│  │                                               │          │
│  │  = Proving your IDENTITY                      │          │
│  └───────────────────────────────────────────────┘          │
│                                                             │
│  STEP 2: AUTHORIZATION (at each room/floor)                 │
│  ┌───────────────────────────────────────────────┐          │
│  │  Floor 3 (Finance): "John, you're in Sales.   │          │
│  │                      You can't enter Finance." │          │
│  │  Server Room: "Only IT Admin can enter here."  │          │
│  │  Your Office: "Yes, this is YOUR desk."        │          │
│  │                                               │          │
│  │  = Checking WHAT you're ALLOWED to do          │          │
│  └───────────────────────────────────────────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Authentication vs Authorization — Clear Definitions

| Concept | Question It Answers | Example |
|---------|-------------------|---------|
| **Authentication** | "WHO are you?" | Login with email + password |
| **Authorization** | "WHAT can you do?" | Admin can delete, Viewer can only read |

```
Authentication = Verifying IDENTITY (are you who you claim to be?)
Authorization  = Granting PERMISSIONS (what are you allowed to do?)

Think of it this way:
  Authentication = Your PASSPORT (proves who you are)
  Authorization  = Your VISA (determines what you can do in the country)
```

**Key insight:** Authentication ALWAYS comes first. You can't authorize someone whose identity you haven't verified!

---

### Real-World Example: SAP Application

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  User: sarah@company.com                                    │
│                                                             │
│  AUTHENTICATION:                                            │
│    ✅ "Sarah logged in with correct password"               │
│    → Identity confirmed. She IS Sarah.                      │
│                                                             │
│  AUTHORIZATION:                                             │
│    Role: "PurchasingManager"                                │
│    ✅ Can view all Purchase Orders                          │
│    ✅ Can approve Purchase Orders                           │
│    ❌ Cannot delete Purchase Orders (only Admin can)        │
│    ❌ Cannot access HR module (not in her roles)            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### How It Works in SAP BTP

```
User (Browser)
     │
     │ 1. User clicks "Login"
     ▼
┌──────────────┐
│   Identity   │  2. User enters email + password
│   Provider   │     (SAP IAS, Azure AD, Google, etc.)
│   (IdP)      │  3. IdP verifies identity ✅
└──────┬───────┘
       │
       │ 4. IdP sends back a TOKEN (proof of identity)
       ▼
┌──────────────┐
│   XSUAA      │  5. XSUAA reads the token
│   (BTP)      │  6. Looks up user's ROLES
│              │  7. Creates a JWT token with roles included
└──────┬───────┘
       │
       │ 8. JWT token sent to the CAP application
       ▼
┌──────────────┐
│   CAP App    │  9. CAP checks: Does this JWT have the right role?
│   (Your app) │     @requires('Manager') → Is user a Manager?
│              │     YES → Allow access ✅
│              │     NO  → Return 403 Forbidden ❌
└──────────────┘
```

---

### Identity Providers (IdP) — Who Verifies Identity?

An **Identity Provider** is the service that handles the actual login. It's where usernames and passwords are stored.

| IdP | Used By |
|-----|---------|
| **SAP Identity Authentication Service (IAS)** | SAP BTP (default) |
| **Azure Active Directory** | Microsoft/corporate environments |
| **Google Identity** | Google Workspace users |
| **Okta** | Enterprise SSO |
| **Custom LDAP** | On-premise corporate directories |

**You don't build login pages!** The IdP handles it. Your CAP app just checks the token.

---

### Protocols: SAML vs OAuth2

| Protocol | Purpose | Analogy |
|----------|---------|---------|
| **SAML** | Enterprise SSO (login once, access many apps) | A master key for all doors |
| **OAuth2** | API authorization (grant limited access to resources) | A valet parking ticket (limited access) |

**In BTP:**
- **SAML** is used between the IdP and XSUAA (for initial login)
- **OAuth2** is used between XSUAA and your CAP app (for API access tokens)

You rarely deal with these protocols directly — XSUAA handles everything!

---

## Session 2: XSUAA, JWT Tokens & xs-security.json (10:45 - 12:00)

### What is XSUAA?

**XSUAA** = **Extended Services for User Account and Authentication**

It's the security service on SAP BTP that:
- Manages OAuth2 tokens
- Stores scope/role definitions
- Issues JWT tokens to authenticated users
- Validates tokens on every request

```
┌─────────────────────────────────────────────────────────────┐
│                    XSUAA — Security Broker                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  INPUT:  "User sarah@company.com logged in via IdP"         │
│                                                             │
│  XSUAA does:                                                │
│    1. Confirms the IdP authentication ✅                    │
│    2. Looks up Sarah's assigned ROLES                       │
│    3. Generates a JWT token containing:                     │
│       - User identity (sarah@company.com)                   │
│       - Assigned scopes (PO.Approve, PO.Read, PO.Create)   │
│       - Expiration time (e.g., 12 hours)                    │
│    4. Signs the token (tamper-proof!)                        │
│                                                             │
│  OUTPUT: Signed JWT token → sent to CAP application         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### JWT Tokens — Your Digital ID Card

**JWT** = **JSON Web Token** (pronounced "jot")

A JWT is a digitally signed string that contains user info and permissions. It looks like this:

```
eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJzYXJhaEBjb21wYW55LmNvbSIsInNjb3BlIjpbIlBPLkFwcHJvdmUiLCJQTy5SZWFkIl0sImV4cCI6MTcxNzU5OTk5OX0.signature_here
```

This ugly string has 3 parts (separated by dots):

```
HEADER.PAYLOAD.SIGNATURE

Part 1 — HEADER (how it's signed):
{
  "alg": "RS256",           ← Algorithm used to sign
  "typ": "JWT"              ← It's a JWT
}

Part 2 — PAYLOAD (the actual data!):
{
  "sub": "sarah@company.com",       ← WHO (subject/user)
  "scope": ["PO.Approve", "PO.Read", "PO.Create"],  ← WHAT they can do
  "exp": 1717599999,                ← WHEN it expires
  "client_id": "po-management",     ← WHICH app
  "zid": "tenant-id-here"           ← WHICH tenant
}

Part 3 — SIGNATURE (proof it wasn't tampered with):
  HMAC(header + payload, secret_key)
  → If anyone modifies the payload, the signature becomes invalid!
```

---

### JWT Flow — How It Works Per Request

```
Every single API request includes the JWT:

GET /purchasing/PurchaseOrders
Authorization: Bearer eyJhbGciOi....(full JWT token)

CAP receives the request:
  1. Reads the "Authorization: Bearer <token>" header
  2. Validates the signature (is it real? not tampered?)
  3. Checks expiration (has it expired?)
  4. Reads the scopes: ["PO.Approve", "PO.Read"]
  5. Checks: Does this endpoint require any scope?
     → @requires('PO.Read') → User has PO.Read? → ✅ ALLOW
     → @requires('Admin') → User doesn't have Admin? → ❌ 403 FORBIDDEN
```

---

### The Security Configuration File: `xs-security.json`

This file defines YOUR application's security model: what scopes exist, what roles exist, and how they map together.

```json
{
  "xsappname": "po-management",
  "tenant-mode": "dedicated",
  "scopes": [
    { "name": "$XSAPPNAME.Read", "description": "Read access to PO data" },
    { "name": "$XSAPPNAME.Create", "description": "Create Purchase Orders" },
    { "name": "$XSAPPNAME.Approve", "description": "Approve Purchase Orders" },
    { "name": "$XSAPPNAME.Admin", "description": "Full admin access" }
  ],
  "role-templates": [
    {
      "name": "Viewer",
      "description": "Read-only access",
      "scope-references": ["$XSAPPNAME.Read"]
    },
    {
      "name": "Manager",
      "description": "Can create and approve POs",
      "scope-references": ["$XSAPPNAME.Read", "$XSAPPNAME.Create", "$XSAPPNAME.Approve"]
    },
    {
      "name": "Administrator",
      "description": "Full access to everything",
      "scope-references": ["$XSAPPNAME.Read", "$XSAPPNAME.Create", "$XSAPPNAME.Approve", "$XSAPPNAME.Admin"]
    }
  ],
  "role-collections": [
    {
      "name": "PO_Viewer",
      "description": "PO Viewers",
      "role-template-references": ["$XSAPPNAME.Viewer"]
    },
    {
      "name": "PO_Manager",
      "description": "PO Managers",
      "role-template-references": ["$XSAPPNAME.Manager"]
    },
    {
      "name": "PO_Admin",
      "description": "PO Administrators",
      "role-template-references": ["$XSAPPNAME.Administrator"]
    }
  ]
}
```

---

### Breaking Down the Security Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│              SECURITY HIERARCHY                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SCOPES (the atomic permissions)                            │
│  ├── Read      → Can view data                             │
│  ├── Create    → Can create records                        │
│  ├── Approve   → Can approve POs                           │
│  └── Admin     → Can delete and configure                  │
│         │                                                   │
│         ▼                                                   │
│  ROLE TEMPLATES (bundles of scopes)                         │
│  ├── Viewer        = [Read]                                │
│  ├── Manager       = [Read, Create, Approve]               │
│  └── Administrator = [Read, Create, Approve, Admin]        │
│         │                                                   │
│         ▼                                                   │
│  ROLE COLLECTIONS (assigned to actual users)                │
│  ├── PO_Viewer  → assigned to: viewer1@company.com         │
│  ├── PO_Manager → assigned to: sarah@company.com           │
│  └── PO_Admin   → assigned to: admin@company.com           │
│                                                             │
│  Hierarchy: Scopes → Role Templates → Role Collections     │
│             (what)    (bundles)        (who gets them)      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**In simple terms:**
- **Scope** = a single permission (like a key to one door)
- **Role Template** = a keyring with multiple keys (bundle of scopes)
- **Role Collection** = assigning a keyring to a specific person

---

### `$XSAPPNAME` — The Prefix

In `xs-security.json`, `$XSAPPNAME` is a variable that gets replaced with your app name at deployment:

```
$XSAPPNAME.Read  →  po-management.Read  (at runtime)
$XSAPPNAME.Admin →  po-management.Admin (at runtime)
```

This ensures scopes are unique across multiple apps in the same BTP space.

---

## Session 3: Hands-on — Create xs-security.json & Define Roles (12:00 - 13:00)

### Step 1: Generate the Security Configuration

```bash
cd ~/projects/po-management

# CAP can generate a starter xs-security.json:
cds add xsuaa
```

This creates `xs-security.json` in your project root with a basic template.

---

### Step 2: Customize xs-security.json

Edit the generated file to define YOUR roles:

**File: `xs-security.json`**

```json
{
  "xsappname": "po-management",
  "tenant-mode": "dedicated",
  "scopes": [
    {
      "name": "$XSAPPNAME.Read",
      "description": "Read PO data"
    },
    {
      "name": "$XSAPPNAME.Create",
      "description": "Create and edit Purchase Orders"
    },
    {
      "name": "$XSAPPNAME.Approve",
      "description": "Approve or reject Purchase Orders"
    },
    {
      "name": "$XSAPPNAME.Delete",
      "description": "Delete Purchase Orders"
    },
    {
      "name": "$XSAPPNAME.Admin",
      "description": "Full administrative access"
    }
  ],
  "attributes": [],
  "role-templates": [
    {
      "name": "Viewer",
      "description": "Read-only access to PO data",
      "scope-references": [
        "$XSAPPNAME.Read"
      ]
    },
    {
      "name": "PurchaseManager",
      "description": "Create and approve Purchase Orders",
      "scope-references": [
        "$XSAPPNAME.Read",
        "$XSAPPNAME.Create",
        "$XSAPPNAME.Approve"
      ]
    },
    {
      "name": "Administrator",
      "description": "Full access including delete and admin",
      "scope-references": [
        "$XSAPPNAME.Read",
        "$XSAPPNAME.Create",
        "$XSAPPNAME.Approve",
        "$XSAPPNAME.Delete",
        "$XSAPPNAME.Admin"
      ]
    }
  ],
  "role-collections": [
    {
      "name": "PO_ViewerRC",
      "description": "Role collection for PO viewers",
      "role-template-references": [
        "$XSAPPNAME.Viewer"
      ]
    },
    {
      "name": "PO_ManagerRC",
      "description": "Role collection for PO managers",
      "role-template-references": [
        "$XSAPPNAME.PurchaseManager"
      ]
    },
    {
      "name": "PO_AdminRC",
      "description": "Role collection for PO administrators",
      "role-template-references": [
        "$XSAPPNAME.Administrator"
      ]
    }
  ]
}
```

---

### Step 3: Configure CAP to Use XSUAA

Update `package.json`:

```json
{
  "cds": {
    "requires": {
      "auth": {
        "kind": "xsuaa",
        "[development]": {
          "kind": "mocked",
          "users": {
            "viewer@test.com": {
              "password": "pass",
              "roles": ["Viewer"]
            },
            "manager@test.com": {
              "password": "pass",
              "roles": ["PurchaseManager"]
            },
            "admin@test.com": {
              "password": "pass",
              "roles": ["Administrator"]
            }
          }
        }
      }
    }
  }
}
```

**Key parts:**
- `[development]` → uses mocked auth (no real XSUAA needed locally!)
- `users` → test users with passwords and roles for local testing
- `[production]` inherits `kind: "xsuaa"` → real XSUAA on BTP

---

### Alternative: Mock Users via `.cdsrc.json`

Create `.cdsrc.json` in the project root:

```json
{
  "requires": {
    "auth": {
      "kind": "mocked",
      "users": {
        "viewer": { "roles": ["Viewer"] },
        "manager": { "roles": ["PurchaseManager"] },
        "admin": { "roles": ["Administrator", "PurchaseManager", "Viewer"] }
      }
    }
  }
}
```

---

## Session 4: @requires and @restrict in CAP (13:45 - 14:45)

### `@requires` — Protect an Entire Service or Entity

The simplest way to add security: require a role to access anything.

```cds
// Only authenticated users can access this service
service PurchasingService @(requires: 'authenticated-user') {
  entity PurchaseOrders as projection on db.PurchaseOrders;
}

// Only users with 'Admin' role can access this
service AdminService @(requires: 'Admin') {
  entity Products as projection on db.Products;
}

// Multiple roles allowed (OR logic — any of these roles)
service ManagerService @(requires: ['PurchaseManager', 'Administrator']) {
  entity PurchaseOrders as projection on db.PurchaseOrders;
}
```

**What happens:**
- User WITHOUT the required role → `403 Forbidden`
- User WITH the role → request proceeds normally
- No auth at all → `401 Unauthorized`

---

### `@requires` on Individual Entities

```cds
service PurchasingService @(requires: 'authenticated-user') {
  // Everyone can read (authenticated)
  @readonly entity Products as projection on db.Products;

  // Only managers and admins can access POs
  @(requires: ['PurchaseManager', 'Administrator'])
  entity PurchaseOrders as projection on db.PurchaseOrders;

  // Only admins can manage suppliers
  @(requires: 'Administrator')
  entity Suppliers as projection on db.Suppliers;
}
```

---

### `@restrict` — Fine-Grained Access Control

`@restrict` gives you MUCH more control: you can specify which OPERATIONS each role can perform.

```cds
service PurchasingService @(requires: 'authenticated-user') {

  entity PurchaseOrders @(restrict: [
    { grant: 'READ', to: 'Viewer' },
    { grant: ['READ', 'CREATE', 'UPDATE'], to: 'PurchaseManager' },
    { grant: '*', to: 'Administrator' }
  ]) as projection on db.PurchaseOrders;

}
```

**What this means:**
| User Role | Can READ? | Can CREATE? | Can UPDATE? | Can DELETE? |
|-----------|-----------|-------------|-------------|------------|
| Viewer | ✅ | ❌ | ❌ | ❌ |
| PurchaseManager | ✅ | ✅ | ✅ | ❌ |
| Administrator | ✅ | ✅ | ✅ | ✅ |

---

### `@restrict` with `where` — Row-Level Security

The `where` clause limits WHICH records a user can see/modify:

```cds
entity PurchaseOrders @(restrict: [
  // Viewers can only see approved POs
  { grant: 'READ', to: 'Viewer', where: 'status = ''Approved''' },

  // Managers can see and edit only their own POs
  { grant: ['READ', 'UPDATE'], to: 'PurchaseManager', where: 'createdBy = $user' },

  // Admins can do everything on all POs
  { grant: '*', to: 'Administrator' }
]) as projection on db.PurchaseOrders;
```

**Special variables in `where`:**
- `$user` → current user's ID (e.g., `sarah@company.com`)
- `$user.attr.department` → user attribute (if configured)

**Example: Users can only see their own orders:**
```cds
entity MyOrders @(restrict: [
  { grant: '*', where: 'createdBy = $user' }
]) as projection on db.SalesOrders;
```

---

### `@restrict` — Complete Syntax

```cds
@(restrict: [
  {
    grant: 'READ' | 'CREATE' | 'UPDATE' | 'DELETE' | '*' | ['READ','CREATE'],
    to: 'RoleName' | ['Role1', 'Role2'],
    where: 'optional filter expression'
  }
])
```

**Examples:**

```cds
// Only managers can approve (call the action)
entity PurchaseOrders as projection on db.PurchaseOrders
  actions {
    @(requires: 'PurchaseManager')
    action approve(comment: String);

    @(requires: 'PurchaseManager')
    action reject(reason: String);

    @(requires: 'Administrator')
    action delete();
  };
```

---

### Combining @requires and @restrict

```cds
// Service level: must be authenticated
service PurchasingService @(requires: 'authenticated-user') {

  // Entity level: fine-grained per role
  entity PurchaseOrders @(restrict: [
    { grant: 'READ', to: 'Viewer' },
    { grant: ['READ', 'CREATE', 'UPDATE'], to: 'PurchaseManager' },
    { grant: '*', to: 'Administrator' }
  ]) as projection on db.PurchaseOrders with draft
    actions {
      @(requires: 'PurchaseManager')
      action submit();

      @(requires: 'PurchaseManager')
      action approve(comment: String);

      @(requires: 'Administrator')
      action forceDelete();
    };

  // Read-only for everyone (no restrict needed)
  @readonly entity Products as projection on db.Products;

  // Only admins manage suppliers
  @(requires: 'Administrator')
  entity Suppliers as projection on db.Suppliers;
}
```

---

### How CAP Processes Security

```
Request arrives: POST /purchasing/PurchaseOrders
Token: { user: "sarah@company.com", roles: ["PurchaseManager"] }

Step 1: Check @requires on SERVICE
  → Service requires 'authenticated-user' → Sarah is authenticated ✅

Step 2: Check @restrict on ENTITY
  → PurchaseOrders restrict: PurchaseManager can CREATE ✅

Step 3: Check @requires on ACTION (if calling an action)
  → action approve requires 'PurchaseManager' → Sarah has it ✅

Step 4: Check WHERE clause (if present)
  → where 'createdBy = $user' → filter applied to query

RESULT: Request allowed! ✅
```

---

### Quick Reference: Security Annotations

| Annotation | Level | What It Does |
|-----------|-------|-------------|
| `@requires: 'Role'` | Service or Entity | Block ALL access unless user has role |
| `@requires: ['R1','R2']` | Service or Entity | Allow if user has ANY listed role |
| `@requires: 'authenticated-user'` | Service | Any logged-in user can access |
| `@restrict: [{...}]` | Entity | Fine-grained: control per operation per role |
| `grant: 'READ'` | In @restrict | Allow only reading |
| `grant: '*'` | In @restrict | Allow everything |
| `where: 'field = $user'` | In @restrict | Row-level filtering |
| `@requires` on action | Action | Protect specific action |

---

## Session 5: Hands-on — Secure Services & Test with Mock Users (15:00 - 16:00)

### Exercise 1: Add Security Annotations (20 minutes)

Update your purchasing service:

```cds
// srv/purchasing-service.cds
using { com.epm as db } from '../db/schema';

service PurchasingService @(path: '/purchasing', requires: 'authenticated-user') {

  entity PurchaseOrders @(restrict: [
    { grant: 'READ', to: 'Viewer' },
    { grant: ['READ', 'CREATE', 'UPDATE'], to: 'PurchaseManager' },
    { grant: '*', to: 'Administrator' }
  ]) as projection on db.PurchaseOrders with draft
    actions {
      @(requires: 'PurchaseManager')
      action submit() returns { status: String; message: String; };

      @(requires: 'PurchaseManager')
      action approve(comment: String) returns { status: String; message: String; };

      @(requires: 'PurchaseManager')
      action reject(reason: String) returns { status: String; message: String; };
    };

  entity PurchaseOrderItems @(restrict: [
    { grant: 'READ', to: 'Viewer' },
    { grant: '*', to: ['PurchaseManager', 'Administrator'] }
  ]) as projection on db.PurchaseOrderItems;

  @readonly entity Products as projection on db.Products;

  @(requires: 'Administrator')
  entity Suppliers as projection on db.Suppliers;
}
```

---

### Exercise 2: Configure Mock Users (10 minutes)

Update `package.json`:

```json
{
  "cds": {
    "requires": {
      "auth": {
        "kind": "xsuaa",
        "[development]": {
          "kind": "mocked",
          "users": {
            "viewer": {
              "password": "viewer",
              "roles": ["Viewer"]
            },
            "manager": {
              "password": "manager",
              "roles": ["PurchaseManager", "Viewer"]
            },
            "admin": {
              "password": "admin",
              "roles": ["Administrator", "PurchaseManager", "Viewer"]
            },
            "anonymous": {
              "password": "anon",
              "roles": []
            }
          }
        }
      }
    }
  }
}
```

---

### Exercise 3: Test Security (30 minutes)

Restart `cds watch` and test with different users:

```http
### ═══════════════════════════════════════
### TEST AS VIEWER (read-only access)
### ═══════════════════════════════════════

### Should SUCCEED (Viewer can READ)
GET http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic dmlld2VyOnZpZXdlcg==

### Should FAIL 403 (Viewer can't CREATE)
POST http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic dmlld2VyOnZpZXdlcg==
Content-Type: application/json

{ "poNumber": "PO-TEST", "status": "Draft" }

### Should FAIL 403 (Viewer can't access Suppliers)
GET http://localhost:4004/purchasing/Suppliers
Authorization: Basic dmlld2VyOnZpZXdlcg==


### ═══════════════════════════════════════
### TEST AS MANAGER (create + approve)
### ═══════════════════════════════════════

### Should SUCCEED (Manager can READ)
GET http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic bWFuYWdlcjptYW5hZ2Vy

### Should SUCCEED (Manager can CREATE)
POST http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic bWFuYWdlcjptYW5hZ2Vy
Content-Type: application/json

{ "poNumber": "PO-MGR-001", "status": "Draft" }

### Should FAIL 403 (Manager can't access Suppliers)
GET http://localhost:4004/purchasing/Suppliers
Authorization: Basic bWFuYWdlcjptYW5hZ2Vy

### Should FAIL 403 (Manager can't DELETE)
DELETE http://localhost:4004/purchasing/PurchaseOrders(some-uuid)
Authorization: Basic bWFuYWdlcjptYW5hZ2Vy


### ═══════════════════════════════════════
### TEST AS ADMIN (full access)
### ═══════════════════════════════════════

### Should SUCCEED (Admin can do everything)
GET http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic YWRtaW46YWRtaW4=

### Should SUCCEED (Admin can access Suppliers)
GET http://localhost:4004/purchasing/Suppliers
Authorization: Basic YWRtaW46YWRtaW4=

### Should SUCCEED (Admin can DELETE)
DELETE http://localhost:4004/purchasing/PurchaseOrders(some-uuid)
Authorization: Basic YWRtaW46YWRtaW4=


### ═══════════════════════════════════════
### TEST WITHOUT AUTH (should fail)
### ═══════════════════════════════════════

### Should FAIL 401 (No authentication)
GET http://localhost:4004/purchasing/PurchaseOrders
```

**How to create Base64 auth headers:**
```
viewer:viewer  → Base64 → dmlld2VyOnZpZXdlcg==
manager:manager → Base64 → bWFuYWdlcjptYW5hZ2Vy
admin:admin    → Base64 → YWRtaW46YWRtaW4=
```

Or simply use the username:password in a REST client that supports Basic Auth.

---

## Session 6: End-to-End Security Testing (16:00 - 16:45)

### Complete Security Test Matrix

Fill in with ✅ (allowed) or ❌ (blocked):

| Operation | No Auth | Viewer | Manager | Admin |
|-----------|---------|--------|---------|-------|
| GET /purchasing/PurchaseOrders | ? | ? | ? | ? |
| POST /purchasing/PurchaseOrders | ? | ? | ? | ? |
| PATCH /purchasing/PurchaseOrders(id) | ? | ? | ? | ? |
| DELETE /purchasing/PurchaseOrders(id) | ? | ? | ? | ? |
| POST .../PurchaseOrders(id)/submit | ? | ? | ? | ? |
| POST .../PurchaseOrders(id)/approve | ? | ? | ? | ? |
| GET /purchasing/Suppliers | ? | ? | ? | ? |
| POST /purchasing/Suppliers | ? | ? | ? | ? |
| GET /purchasing/Products | ? | ? | ? | ? |

<details>
<summary>Expected Results</summary>

| Operation | No Auth | Viewer | Manager | Admin |
|-----------|---------|--------|---------|-------|
| GET /PurchaseOrders | ❌ 401 | ✅ | ✅ | ✅ |
| POST /PurchaseOrders | ❌ 401 | ❌ 403 | ✅ | ✅ |
| PATCH /PurchaseOrders(id) | ❌ 401 | ❌ 403 | ✅ | ✅ |
| DELETE /PurchaseOrders(id) | ❌ 401 | ❌ 403 | ❌ 403 | ✅ |
| POST .../submit | ❌ 401 | ❌ 403 | ✅ | ✅ |
| POST .../approve | ❌ 401 | ❌ 403 | ✅ | ✅ |
| GET /Suppliers | ❌ 401 | ❌ 403 | ❌ 403 | ✅ |
| POST /Suppliers | ❌ 401 | ❌ 403 | ❌ 403 | ✅ |
| GET /Products | ❌ 401 | ✅ | ✅ | ✅ |

</details>

---

### Checking the User in Handlers

You can access the current user info in your handlers:

```javascript
this.before('CREATE', 'PurchaseOrders', (req) => {
  // Who is making this request?
  console.log('User:', req.user.id);           // "sarah@company.com"
  console.log('Roles:', req.user.roles);       // ["PurchaseManager"]
  console.log('Is admin?', req.user.is('Administrator'));  // false

  // Set the creator
  req.data.createdBy = req.user.id;
});

this.on('approve', 'PurchaseOrders', async (req) => {
  // Record WHO approved
  const approver = req.user.id;
  await UPDATE(PurchaseOrders).set({
    status: 'Approved',
    approvedBy: approver,
    approvedAt: new Date().toISOString()
  }).where({ ID: req.params[0].ID });

  return { status: 'Approved', message: `Approved by ${approver}` };
});
```

---

## Assessment: MCQ — 15 Questions on Authentication & Authorization

**Q1.** Authentication answers the question:

- A) "What can you do?"
- B) "WHO are you?" (proving identity)
- C) "Where are you from?"
- D) "How much data can you read?"

**Answer: B** — Authentication = verify identity. Authorization = determine permissions.

---

**Q2.** Authorization answers the question:

- A) "Who are you?"
- B) "What browser are you using?"
- C) "WHAT are you ALLOWED to do?" (permissions)
- D) "When did you log in?"

**Answer: C** — Authorization checks permissions AFTER identity is confirmed.

---

**Q3.** XSUAA stands for:

- A) Extended Secure User Access Authentication
- B) Extended Services for User Account and Authentication
- C) External Security User Admin Application
- D) XML Service for User Authorization

**Answer: B** — Extended Services for User Account and Authentication — BTP's security service.

---

**Q4.** A JWT token contains:

- A) Only the user's password
- B) User identity, assigned scopes/roles, expiration time, and a tamper-proof signature
- C) The entire database
- D) Only the username

**Answer: B** — JWT carries identity + permissions + expiry, signed to prevent tampering.

---

**Q5.** In `xs-security.json`, the security hierarchy from smallest to largest is:

- A) Role Collection → Role Template → Scope
- B) Scope → Role Template → Role Collection
- C) Role Template → Scope → Role Collection
- D) Scope → Role Collection → Role Template

**Answer: B** — Scopes (atomic) → Role Templates (bundles of scopes) → Role Collections (assigned to users).

---

**Q6.** `@requires: 'authenticated-user'` means:

- A) Only admins can access
- B) Any logged-in user can access (no specific role needed)
- C) Anonymous access allowed
- D) Only users from the same organization

**Answer: B** — `authenticated-user` is a special value meaning "any user who is logged in."

---

**Q7.** `@restrict: [{ grant: 'READ', to: 'Viewer' }]` means:

- A) Everyone can read
- B) Users with 'Viewer' role can ONLY read (not create/update/delete)
- C) Viewers are blocked from reading
- D) The entity is read-only for everyone

**Answer: B** — Users with Viewer role are granted READ permission only. All other operations are denied.

---

**Q8.** `grant: '*'` in @restrict means:

- A) Grant to all users
- B) Grant ALL operations (read, create, update, delete) to the specified role
- C) Grant wildcard search
- D) Deny all access

**Answer: B** — `'*'` = all CRUD operations. The role gets full access to that entity.

---

**Q9.** `where: 'createdBy = $user'` in @restrict does what?

- A) Filters the table to show only records created by the current user
- B) Sets the createdBy field to the current user
- C) Validates that createdBy is not empty
- D) Joins with the users table

**Answer: A** — Row-level security! Users only see/modify records they created. `$user` = current user's ID.

---

**Q10.** In development mode, CAP uses _____ for authentication:

- A) Real XSUAA service
- B) Mocked authentication with test users defined in package.json
- C) No authentication (always open)
- D) Windows Active Directory

**Answer: B** — `[development]: { kind: "mocked", users: {...} }` provides fake users for testing locally.

---

**Q11.** If a user without any role tries to access a service with `@requires: 'authenticated-user'`:

- A) Gets 200 OK (it just requires being logged in, any role)
- B) Gets 403 Forbidden (has no roles)
- C) Gets 200 OK (authenticated-user doesn't need specific roles)
- D) Gets 500 Server Error

**Answer: C** — `authenticated-user` only checks that the user IS logged in. No specific role is needed. Even a user with zero custom roles passes this check (as long as they're authenticated).

---

**Q12.** `@requires` on a bound action means:

- A) The entire entity is protected
- B) Only that specific action requires the role (other operations unaffected)
- C) The action can never be called
- D) The action doesn't need authentication

**Answer: B** — Putting `@requires` on an action protects only that action. Other CRUD operations follow their own restrictions.

---

**Q13.** The `$XSAPPNAME` prefix in xs-security.json ensures:

- A) The app runs faster
- B) Scopes are unique per application (prevents conflicts with other apps)
- C) The app is encrypted
- D) Users are unique

**Answer: B** — `$XSAPPNAME` becomes `po-management.Read` at runtime, preventing scope collisions between different apps.

---

**Q14.** To access the current user's ID in a handler:

- A) `req.headers.user`
- B) `req.user.id`
- C) `req.auth.username`
- D) `cds.user.current`

**Answer: B** — `req.user.id` gives the authenticated user's ID (email). `req.user.is('Role')` checks roles.

---

**Q15.** A user gets `401 Unauthorized`. This means:

- A) They have the wrong role
- B) They are NOT authenticated at all (no token/login)
- C) Their password expired
- D) The server is down

**Answer: B** — 401 = "I don't know who you are" (no/invalid token). 403 = "I know who you are but you're not allowed."

---

## Assignment: Secure PO Management Application

### Task

Implement complete security for your PO Management application with three roles.

### Role Definitions

| Role | Permissions |
|------|------------|
| **Viewer** | Read all POs and Products. Cannot create, edit, or delete. |
| **PurchaseManager** | Read all. Create and edit POs. Submit and approve POs. Cannot delete. Cannot manage suppliers. |
| **Administrator** | Full access to everything. Can delete POs. Can manage suppliers. |

### Deliverables

1. **`xs-security.json`** with:
   - 5 scopes: Read, Create, Approve, Delete, Admin
   - 3 role templates: Viewer, PurchaseManager, Administrator
   - 3 role collections: PO_Viewer, PO_Manager, PO_Admin

2. **Updated service CDS** with:
   - `@requires: 'authenticated-user'` on the service
   - `@restrict` on PurchaseOrders (per role permissions)
   - `@requires: 'Administrator'` on Suppliers entity
   - `@requires: 'PurchaseManager'` on approve/reject actions
   - `@readonly` Products accessible to all authenticated users

3. **Mock users in `package.json`** (for testing)

4. **Test results** — verify all cells in this matrix:

| Operation | Viewer | Manager | Admin |
|-----------|--------|---------|-------|
| Read POs | ✅ | ✅ | ✅ |
| Create PO | ❌ | ✅ | ✅ |
| Update PO | ❌ | ✅ | ✅ |
| Delete PO | ❌ | ❌ | ✅ |
| Submit PO | ❌ | ✅ | ✅ |
| Approve PO | ❌ | ✅ | ✅ |
| Read Suppliers | ❌ | ❌ | ✅ |
| Manage Suppliers | ❌ | ❌ | ✅ |
| Read Products | ✅ | ✅ | ✅ |

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Authentication | Proving WHO you are (login, password, token) |
| Authorization | Determining WHAT you can do (roles, permissions) |
| XSUAA | BTP's security service — manages tokens and roles |
| JWT Token | Signed JSON containing user identity + permissions |
| xs-security.json | Defines scopes, role templates, role collections |
| Scopes | Atomic permissions (Read, Create, Approve, etc.) |
| Role Templates | Bundles of scopes (Viewer = Read only) |
| Role Collections | Assigned to actual users in BTP Cockpit |
| `@requires` | Protect entire service/entity/action with a role |
| `@restrict` | Fine-grained: which operations each role can do |
| `where: $user` | Row-level security (see only your own records) |
| Mock users | Test locally without real XSUAA (`kind: "mocked"`) |
| 401 vs 403 | 401 = not logged in; 403 = logged in but not allowed |

### The Security Stack

```
User logs in (IdP) → XSUAA issues JWT → CAP validates JWT
                                              │
                                    Checks @requires (service level)
                                    Checks @restrict (entity level)
                                    Checks @requires (action level)
                                    Applies WHERE (row level)
                                              │
                                         ✅ or ❌
```

### The One-Liner

> **`@requires` says WHO can enter. `@restrict` says WHAT they can do inside. Together they make your app enterprise-secure.**

---

### Looking Ahead: Day 30

Tomorrow: **Deployment to SAP BTP Cloud Foundry**
- MTA (Multi-Target Application) architecture
- Building and deploying to BTP
- Binding services (HANA, XSUAA)
- Testing in the cloud

---

*End of Day 29 — Your application is now SECURED with proper authentication and authorization!*
