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
