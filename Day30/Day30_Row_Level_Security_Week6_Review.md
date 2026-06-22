# Day 30: Row-Level Security & Week 6 Review

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 29 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Row-Level Security & Instance-Based Authorization | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Attribute-Based Access Control & Security Pitfalls | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Implement & Test Row-Level Security | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 15:00 | Session 4: Weekly Quiz — Days 26-30 (30 Questions) | 75 min |
| 15:00 - 15:15 | Break | 15 min |
| 15:15 - 17:00 | Session 5 & 6: Mini Project — Complete EPM Security Model | 105 min |

---

## What You'll Achieve Today

By the end of this review day, you will:
- Implement row-level security so users only see THEIR OWN data
- Use `$user` and `$user.attr` variables in restriction `where` clauses
- Understand instance-based vs role-based authorization
- Identify and avoid common security pitfalls
- Test data isolation between different users
- Debug authorization issues when access is unexpectedly denied
- Build a complete security model combining roles + row-level + attributes

---

## Day 29 Recap — Quick Fire (09:00 - 09:15)

1. Authentication proves _____, Authorization determines _____? → _____
2. XSUAA stands for _____? → _____
3. JWT contains: user identity + _____ + _____? → _____
4. `@requires: 'authenticated-user'` means _____? → _____
5. `@restrict` with `grant: 'READ', to: 'Viewer'` means _____? → _____
6. `where: 'createdBy = $user'` does what? → _____
7. 401 means _____, 403 means _____? → _____
8. In development, CAP uses _____ authentication? → _____

<details>
<summary>Answers</summary>

1. Authentication = **WHO** you are; Authorization = **WHAT** you can do
2. **Extended Services for User Account and Authentication**
3. **Scopes/roles** + **expiration time** (+ signature)
4. Any **logged-in user** can access (no specific role required)
5. Users with Viewer role can **only READ** (not create/update/delete)
6. **Row-level filter** — users only see records they created
7. 401 = **Not authenticated** (no login); 403 = **Not authorized** (wrong role)
8. **Mocked** authentication (fake users in package.json)

</details>

---

## Session 1: Row-Level Security & Instance-Based Authorization (09:15 - 10:30)

### What is Row-Level Security?

Yesterday we learned role-based access: "Viewers can READ, Managers can WRITE." But what if you need:

> "Managers can only see THEIR OWN department's Purchase Orders — not other departments' POs."

That's **row-level security** — filtering WHICH records a user can see based on their identity or attributes.

```
┌─────────────────────────────────────────────────────────────┐
│        ROLE-BASED vs ROW-LEVEL SECURITY                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ROLE-BASED (Day 29):                                       │
│    "Managers can READ Purchase Orders"                      │
│    → ALL managers see ALL Purchase Orders                   │
│    → Everyone with the role sees the same data              │
│                                                             │
│  ROW-LEVEL (Today):                                         │
│    "Managers can READ their OWN Purchase Orders"            │
│    → Manager A sees only POs they created                   │
│    → Manager B sees only POs they created                   │
│    → Same role, DIFFERENT data!                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### The Apartment Analogy

```
ROLE-BASED = Floor access
  "All residents (role) can access the building"
  "Only maintenance (role) can access the roof"

ROW-LEVEL = Individual apartment access
  "Resident A can only enter Apartment 301 (their own!)"
  "Resident B can only enter Apartment 405 (their own!)"
  "Both are 'residents' but each sees different data (apartment)"
```

---

### How Row-Level Security Works in CAP

Use the `where` clause in `@restrict`:

```cds
entity PurchaseOrders @(restrict: [
  // Viewers see only approved POs
  { grant: 'READ', to: 'Viewer', where: 'status = ''Approved''' },

  // Managers see only POs THEY created
  { grant: '*', to: 'PurchaseManager', where: 'createdBy = $user' },

  // Admins see everything (no where clause = no filter)
  { grant: '*', to: 'Administrator' }
]) as projection on db.PurchaseOrders;
```

**What happens at runtime:**

```
Manager "sarah@company.com" calls: GET /purchasing/PurchaseOrders

CAP automatically adds a filter:
  SELECT * FROM PurchaseOrders WHERE createdBy = 'sarah@company.com'

Sarah only sees HER OWN POs! Other managers' POs are invisible to her.
```

---

### The `$user` Variable

`$user` is automatically replaced with the current authenticated user's ID:

```cds
// User can only see records they created
where: 'createdBy = $user'

// At runtime, if user is "john@company.com":
// Becomes: WHERE createdBy = 'john@company.com'
```

**Common patterns using `$user`:**

```cds
// Pattern 1: Users see only their own records
{ grant: 'READ', to: 'Employee', where: 'createdBy = $user' }

// Pattern 2: Users can only edit their own records
{ grant: 'UPDATE', to: 'Employee', where: 'createdBy = $user' }

// Pattern 3: Managers see their department's records
{ grant: 'READ', to: 'Manager', where: 'department = $user.department' }

// Pattern 4: Combined — read own OR read all if admin
@(restrict: [
  { grant: 'READ', to: 'Employee', where: 'createdBy = $user' },
  { grant: 'READ', to: 'Administrator' }  // no where = see all
])
```

---

### Instance-Based Authorization — Explained

**Instance-based** means: "Can this user access THIS specific record?"

It's not about roles alone — it's about the relationship between the USER and the DATA INSTANCE.

```
┌─────────────────────────────────────────────────────────────┐
│  INSTANCE-BASED SCENARIOS                                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  "Can Sarah approve THIS purchase order?"                   │
│    → Check: Is Sarah the designated approver? (not just     │
│      any manager — specifically assigned to THIS PO)        │
│                                                             │
│  "Can John edit THIS document?"                             │
│    → Check: Is John the owner (createdBy)? Or was John      │
│      explicitly granted edit permission on THIS doc?         │
│                                                             │
│  "Can Alex view THIS patient record?"                       │
│    → Check: Is Alex the assigned doctor for THIS patient?   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Implementing Instance-Based Access in CAP

```cds
// Only the assigned approver can approve a specific PO
entity PurchaseOrders @(restrict: [
  { grant: 'READ', to: 'authenticated-user' },
  { grant: 'UPDATE', to: 'PurchaseManager', where: 'createdBy = $user' },
  { grant: '*', to: 'Administrator' }
]) as projection on db.PurchaseOrders
  actions {
    // Only if user is the designated approver for THIS PO
    action approve(comment: String) returns { status: String; };
  };
```

**Handler adds the instance check:**
```javascript
this.on('approve', 'PurchaseOrders', async (req) => {
  const { ID } = req.params[0];
  const po = await SELECT.one.from(PurchaseOrders).where({ ID });

  // Instance-based check: Is this user the designated approver?
  if (po.assignedApprover && po.assignedApprover !== req.user.id) {
    req.reject(403, `You are not the assigned approver for this PO. Assigned to: ${po.assignedApprover}`);
  }

  // ... proceed with approval
});
```

---

### Multiple `where` Conditions

You can use OR logic by having multiple restrict entries for the same role:

```cds
entity Documents @(restrict: [
  // Users can see docs they CREATED
  { grant: 'READ', to: 'Employee', where: 'createdBy = $user' },
  // Users can ALSO see docs SHARED with them
  { grant: 'READ', to: 'Employee', where: 'sharedWith = $user' },
  // Admins see everything
  { grant: '*', to: 'Administrator' }
]) as projection on db.Documents;
```

**Result:** An employee sees documents they created OR documents shared with them (union of both conditions).

---

### Real-World Row-Level Patterns

| Scenario | Where Clause | Meaning |
|----------|-------------|---------|
| My own records | `createdBy = $user` | See only what you created |
| My department | `department = $user.department` | See your team's data |
| Active/approved only | `status = 'Active'` | Hide drafts from non-owners |
| Assigned to me | `assignedTo = $user` | Task-based access |
| My region | `region = $user.region` | Geographic data isolation |
| Combination | `createdBy = $user or assignedTo = $user` | Multiple access paths |

---

## Session 2: Attribute-Based Access Control & Security Pitfalls (10:45 - 12:00)

### Attribute-Based Access Control (ABAC)

Beyond simple roles, you can use **user attributes** for finer control. Attributes are properties assigned to users (department, country, cost center, etc.).

---

### Configuring User Attributes

**Step 1: Define attributes in xs-security.json:**

```json
{
  "xsappname": "po-management",
  "scopes": [...],
  "attributes": [
    { "name": "Department", "description": "User's department", "valueType": "string" },
    { "name": "CostCenter", "description": "User's cost center", "valueType": "string" },
    { "name": "Region", "description": "User's region", "valueType": "string" }
  ],
  "role-templates": [
    {
      "name": "RegionalManager",
      "description": "Manager for a specific region",
      "scope-references": ["$XSAPPNAME.Read", "$XSAPPNAME.Approve"],
      "attribute-references": ["Department", "Region"]
    }
  ]
}
```

**Step 2: Use attributes in CAP restrictions:**

```cds
entity PurchaseOrders @(restrict: [
  // Regional managers see only their region's POs
  { grant: 'READ', to: 'RegionalManager', where: 'region = $user.Region' },

  // Department heads see only their department's POs
  { grant: 'READ', to: 'DepartmentHead', where: 'department = $user.Department' },

  // Global admins see everything
  { grant: '*', to: 'Administrator' }
]) as projection on db.PurchaseOrders;
```

**Step 3: Configure mock users with attributes (for testing):**

```json
{
  "cds": {
    "requires": {
      "auth": {
        "[development]": {
          "kind": "mocked",
          "users": {
            "sarah": {
              "roles": ["RegionalManager"],
              "attr": { "Department": "Purchasing", "Region": "APAC" }
            },
            "john": {
              "roles": ["RegionalManager"],
              "attr": { "Department": "Engineering", "Region": "EMEA" }
            },
            "admin": {
              "roles": ["Administrator"],
              "attr": { "Department": "IT", "Region": "Global" }
            }
          }
        }
      }
    }
  }
}
```

**Result:**
- Sarah sees only POs from APAC region
- John sees only POs from EMEA region
- Admin sees ALL POs (no where clause)

---

### Accessing Attributes in Handlers

```javascript
this.before('CREATE', 'PurchaseOrders', (req) => {
  // Auto-set region from user's attribute
  req.data.region = req.user.attr.Region;
  req.data.department = req.user.attr.Department;
});

this.on('READ', 'PurchaseOrders', async (req, next) => {
  // Log who's accessing what
  console.log(`User ${req.user.id} (${req.user.attr.Region}) reading POs`);
  return next();
});
```

---

### Common Security Pitfalls (What NOT to Do)

---

#### Pitfall 1: Forgetting to Secure New Entities

```cds
// ❌ DANGER: New entity added without any restriction!
service MyService @(requires: 'authenticated-user') {
  entity PurchaseOrders @(restrict: [...]) as projection on ...;  // ✅ Secured
  entity Suppliers as projection on db.Suppliers;                  // ❌ NO restriction!
  // Any authenticated user can CREATE/DELETE suppliers!
}
```

**Fix:** ALWAYS add restrictions to every entity. At minimum, use `@readonly` for reference data.

---

#### Pitfall 2: Trusting Client-Sent Data for Authorization

```javascript
// ❌ NEVER do this — client can lie!
this.before('CREATE', 'PurchaseOrders', (req) => {
  // Don't trust req.data.createdBy — the CLIENT sent it!
  // A malicious user could set createdBy to "admin@company.com"
});

// ✅ ALWAYS use req.user (from validated JWT — can't be faked!)
this.before('CREATE', 'PurchaseOrders', (req) => {
  req.data.createdBy = req.user.id;  // Server sets it from the token
});
```

---

#### Pitfall 3: Only Securing READ but Not WRITE

```cds
// ❌ Viewers can't READ other people's data — but CAN UPDATE it?!
@(restrict: [
  { grant: 'READ', to: 'Employee', where: 'createdBy = $user' }
  // Missing: UPDATE and DELETE restrictions!
])
```

**Fix:** Restrict ALL relevant operations:
```cds
@(restrict: [
  { grant: 'READ', to: 'Employee', where: 'createdBy = $user' },
  { grant: 'UPDATE', to: 'Employee', where: 'createdBy = $user' },
  { grant: 'DELETE', to: 'Employee', where: 'createdBy = $user' },
  { grant: '*', to: 'Administrator' }
])
```

---

#### Pitfall 4: Exposing Sensitive Data in @readonly Entities

```cds
// ❌ Everyone can see ALL customer data including credit limits!
@readonly entity Customers as projection on db.Customers;

// ✅ Use projection to hide sensitive fields
@readonly entity Customers as projection on db.Customers {
  ID, customerName, city, country  // Hide: creditLimit, email, phone
};
```

---

#### Pitfall 5: Not Testing with Different User Profiles

```
"It works for me" (developer testing with admin role)
≠
"It works for all users"

ALWAYS test with:
  • No auth (should get 401)
  • Viewer role (read-only, limited data)
  • Manager role (their own data only)
  • Admin role (everything)
  • Wrong role (should get 403)
```

---

#### Pitfall 6: WHERE Clause Bypass via $expand

```cds
// User can only see their own orders
entity Orders @(restrict: [
  { grant: 'READ', where: 'createdBy = $user' }
]) as projection on db.Orders;

// But what about the items?
entity OrderItems as projection on db.OrderItems;  // ❌ No restriction!
// User could call: GET /OrderItems?$filter=order_ID eq 'someone-elses-order-id'
// and see items from other people's orders!
```

**Fix:** Restrict child entities too:
```cds
entity OrderItems @(restrict: [
  { grant: 'READ', where: 'order.createdBy = $user' },
  { grant: '*', to: 'Administrator' }
]) as projection on db.OrderItems;
```

---

### Security Checklist

```
┌─────────────────────────────────────────────────────────────┐
│             SECURITY REVIEW CHECKLIST                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  □ Every service has @requires (at minimum 'authenticated') │
│  □ Every entity has @restrict or @readonly                  │
│  □ Child entities are also secured (not just parent)        │
│  □ Sensitive fields hidden via projections                  │
│  □ Actions have @requires for the right role                │
│  □ createdBy set from req.user.id (not client data)        │
│  □ Row-level WHERE applied for data isolation               │
│  □ Tested with: no auth, viewer, manager, admin             │
│  □ $expand can't bypass row-level restrictions              │
│  □ No hardcoded passwords or secrets in code                │
│  □ .env and credentials in .gitignore                       │
│  □ Error messages don't expose internal details             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Session 3: Hands-on — Implement & Test Row-Level Security (12:00 - 13:00)

### Exercise 1: Implement Row-Level Security (20 minutes)

Update your service with row-level restrictions:

```cds
// srv/purchasing-service.cds
service PurchasingService @(path: '/purchasing', requires: 'authenticated-user') {

  entity PurchaseOrders @(restrict: [
    // Viewers see only Approved and Received POs
    { grant: 'READ', to: 'Viewer', where: 'status = ''Approved'' or status = ''Received''' },

    // Managers see and manage only their OWN POs
    { grant: ['READ', 'CREATE', 'UPDATE'], to: 'PurchaseManager', where: 'createdBy = $user' },

    // Managers can also READ all submitted POs (for approval workflow)
    { grant: 'READ', to: 'PurchaseManager', where: 'status = ''Submitted''' },

    // Admins can do everything on all POs
    { grant: '*', to: 'Administrator' }
  ]) as projection on db.PurchaseOrders with draft
    actions {
      @(requires: 'PurchaseManager')
      action submit() returns { status: String; message: String; };
      @(requires: ['PurchaseManager', 'Administrator'])
      action approve(comment: String) returns { status: String; message: String; };
      @(requires: ['PurchaseManager', 'Administrator'])
      action reject(reason: String) returns { status: String; message: String; };
    };

  entity PurchaseOrderItems @(restrict: [
    { grant: 'READ', to: 'Viewer' },
    { grant: '*', to: ['PurchaseManager', 'Administrator'] }
  ]) as projection on db.PurchaseOrderItems;

  @readonly entity Products as projection on db.Products;
  @(requires: 'Administrator') entity Suppliers as projection on db.Suppliers;
}
```

---

### Exercise 2: Configure Test Users with Different Data (15 minutes)

```json
{
  "cds": {
    "requires": {
      "auth": {
        "[development]": {
          "kind": "mocked",
          "users": {
            "alice": {
              "password": "alice",
              "roles": ["PurchaseManager"],
              "attr": { "Department": "Engineering" }
            },
            "bob": {
              "password": "bob",
              "roles": ["PurchaseManager"],
              "attr": { "Department": "Marketing" }
            },
            "carol": {
              "password": "carol",
              "roles": ["Viewer"]
            },
            "dave": {
              "password": "dave",
              "roles": ["Administrator"]
            }
          }
        }
      }
    }
  }
}
```

---

### Exercise 3: Test Data Isolation (25 minutes)

**Step 1:** Login as Alice and create a PO:
```http
POST http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic YWxpY2U6YWxpY2U=
Content-Type: application/json

{ "poNumber": "PO-ALICE-001", "status": "Draft" }
```

**Step 2:** Login as Bob and create a PO:
```http
POST http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic Ym9iOmJvYg==
Content-Type: application/json

{ "poNumber": "PO-BOB-001", "status": "Draft" }
```

**Step 3:** Alice reads POs — should ONLY see her own:
```http
GET http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic YWxpY2U6YWxpY2U=

# Expected: Only "PO-ALICE-001" (NOT PO-BOB-001!)
```

**Step 4:** Bob reads POs — should only see his own:
```http
GET http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic Ym9iOmJvYg==

# Expected: Only "PO-BOB-001" (NOT PO-ALICE-001!)
```

**Step 5:** Admin reads POs — sees EVERYTHING:
```http
GET http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic ZGF2ZTpkYXZl

# Expected: Both PO-ALICE-001 AND PO-BOB-001
```

**Step 6:** Viewer reads POs — only Approved ones:
```http
GET http://localhost:4004/purchasing/PurchaseOrders
Authorization: Basic Y2Fyb2w6Y2Fyb2w=

# Expected: Only POs with status 'Approved' (empty if none approved)
```

**Step 7:** Alice tries to edit Bob's PO — should FAIL:
```http
PATCH http://localhost:4004/purchasing/PurchaseOrders(bob-po-uuid)
Authorization: Basic YWxpY2U6YWxpY2U=
Content-Type: application/json

{ "priority": "High" }

# Expected: 403 Forbidden (Alice can't edit Bob's data)
```

---

### Debugging Authorization Issues

When access is unexpectedly denied:

```javascript
// Add this temporary debug handler:
this.before('*', '*', (req) => {
  console.log('═══ AUTH DEBUG ═══');
  console.log('User:', req.user.id);
  console.log('Roles:', req.user.roles);
  console.log('Attr:', req.user.attr);
  console.log('Event:', req.event);
  console.log('Entity:', req.target?.name);
  console.log('═══════════════════');
});
```

**Common debug questions:**
1. Is the user authenticated? (Check for `req.user.id`)
2. Does the user have the expected role? (Check `req.user.roles`)
3. Does the WHERE clause match? (Is `createdBy` actually set on the record?)
4. Is the entity name in `@restrict` spelled correctly?

---

## Session 4: Weekly Quiz — Days 26-30 (13:45 - 15:00)

### Instructions
- 30 questions covering Days 26-30 (Git, HANA, Auth, Security)
- Time: 75 minutes
- Passing score: 21/30 (70%)

---

**Q1.** The 3 areas in Git are:
- A) Branch, Merge, Push
- B) Working Directory, Staging Area, Repository
- C) Local, Remote, Cloud
- D) Add, Commit, Deploy

**Answer: B**

---

**Q2.** `git add .` followed by `git commit -m "message"` does what?
- A) Pushes code to GitHub
- B) Stages all changes and saves a permanent snapshot with a description
- C) Creates a new branch
- D) Deletes uncommitted changes

**Answer: B**

---

**Q3.** A feature branch allows you to:
- A) Delete the main branch
- B) Work in isolation without affecting the main/stable code
- C) Speed up Git operations
- D) Skip code review

**Answer: B**

---

**Q4.** `.gitignore` should include all of these EXCEPT:
- A) `node_modules/`
- B) `.env`
- C) `db/schema.cds`
- D) `*.db`

**Answer: C** — `schema.cds` is your source code — it MUST be committed. The others are generated/secret/local.

---

**Q5.** A Pull Request (PR) is used for:
- A) Downloading code
- B) Requesting code review before merging a branch into main
- C) Pulling from multiple remotes
- D) Requesting git access

**Answer: B**

---

**Q6.** `git pull --rebase origin main` is used when:
- A) You want to delete remote changes
- B) Your push was rejected because remote has newer commits
- C) You want to create a new branch
- D) You want to revert to an older version

**Answer: B**

---

**Q7.** SAP HANA stores data in:
- A) Hard disk only
- B) RAM (in-memory) for speed, with persistence to disk
- C) The cloud only (no local storage)
- D) CSV files

**Answer: B**

---

**Q8.** Column store is better than row store for:
- A) Inserting one record at a time
- B) Aggregation queries (SUM, AVG, COUNT) on large datasets
- C) Looking up a single record by primary key
- D) Storing images

**Answer: B**

---

**Q9.** HDI Container provides:
- A) More CPU for your app
- B) Isolation between applications (each app has its own database objects)
- C) A container for Docker images
- D) User interface hosting

**Answer: B**

---

**Q10.** `cds add hana` configures your project to:
- A) Create a HANA Cloud instance
- B) Use HANA in production and SQLite in development (profile-based)
- C) Delete the SQLite database
- D) Generate HTML files

**Answer: B**

---

**Q11.** In HANA SQL, to get the top 5 most expensive products:
- A) `SELECT * FROM Products LIMIT 5 ORDER BY price DESC`
- B) `SELECT TOP 5 * FROM COM_EPM_PRODUCTS ORDER BY PRICE DESC`
- C) `SELECT * FROM Products WHERE ROWNUM <= 5`
- D) `SELECT FIRST 5 FROM Products`

**Answer: B**

---

**Q12.** CDS `String(100)` maps to HANA type:
- A) `VARCHAR(100)`
- B) `NVARCHAR(100)`
- C) `TEXT(100)`
- D) `STRING(100)`

**Answer: B** — HANA uses NVARCHAR (Unicode-capable).

---

**Q13.** If HANA deployment fails with "foreign key constraint violated":
- A) Delete all tables and retry
- B) Parent/reference tables must be loaded before child tables
- C) Change all associations to compositions
- D) Increase HANA memory

**Answer: B**

---

**Q14.** Authentication proves:
- A) What you can do
- B) WHO you are (identity verification)
- C) Where you are located
- D) How much data you can access

**Answer: B**

---

**Q15.** Authorization determines:
- A) Your password strength
- B) WHAT you are allowed to do (permissions)
- C) Your login time
- D) Your network speed

**Answer: B**

---

**Q16.** XSUAA issues what type of token?
- A) Session cookie
- B) JWT (JSON Web Token) containing identity and scopes
- C) API key
- D) SSL certificate

**Answer: B**

---

**Q17.** In `xs-security.json`, the hierarchy from smallest to largest is:
- A) Role Collection → Role Template → Scope
- B) Scope → Role Template → Role Collection
- C) Role Template → Role Collection → Scope
- D) Scope → Role Collection → Role Template

**Answer: B** — Scopes (atomic permissions) → Role Templates (bundles) → Role Collections (assigned to users).

---

**Q18.** `@requires: 'authenticated-user'` on a service means:
- A) Only admins can access
- B) ANY logged-in user can access (regardless of specific role)
- C) No authentication needed
- D) Only users from a specific company

**Answer: B**

---

**Q19.** `@restrict: [{ grant: 'READ', to: 'Viewer' }, { grant: '*', to: 'Admin' }]` means:
- A) Everyone can read and write
- B) Viewers can ONLY read; Admins can do everything
- C) Nobody can access
- D) Viewers and Admins have the same permissions

**Answer: B**

---

**Q20.** `where: 'createdBy = $user'` in @restrict:
- A) Sets createdBy to the current user
- B) Filters data so users only see records they created (row-level security)
- C) Validates that createdBy is not null
- D) Creates a new user

**Answer: B**

---

**Q21.** HTTP status code 401 means:
- A) Success
- B) Not Found
- C) Not Authenticated (no valid login/token)
- D) Server Error

**Answer: C**

---

**Q22.** HTTP status code 403 means:
- A) Success
- B) Not Found
- C) Authenticated but NOT Authorized (wrong role/permission)
- D) Server Error

**Answer: C**

---

**Q23.** To test security locally in CAP, you configure:
- A) Real XSUAA service
- B) Mocked users with roles in package.json (`kind: "mocked"`)
- C) A separate auth server
- D) Browser cookies

**Answer: B**

---

**Q24.** `req.user.id` in a handler returns:
- A) The user's password
- B) The authenticated user's identity (e.g., email address)
- C) The database user
- D) A random number

**Answer: B**

---

**Q25.** `$user.attr.Department` in a where clause accesses:
- A) A field in the entity
- B) A user attribute configured in xs-security.json (e.g., user's department)
- C) The department table
- D) An environment variable

**Answer: B**

---

**Q26.** Row-level security ensures:
- A) Users can't log in
- B) Users only see the specific RECORDS they're authorized to access
- C) Tables are encrypted
- D) Columns are hidden

**Answer: B**

---

**Q27.** If a new entity is added to a service WITHOUT @restrict:
- A) It's automatically secured
- B) ANY authenticated user can perform ALL operations on it (potential security hole!)
- C) It's invisible
- D) Only admins can access it

**Answer: B** — New entities without restrictions inherit only the service-level @requires (which allows all operations for anyone with that role).

---

**Q28.** To prevent a user from editing ANOTHER user's record:
- A) Use @readonly
- B) `{ grant: 'UPDATE', where: 'createdBy = $user' }` (only update own records)
- C) Remove the UPDATE button from the UI
- D) Don't expose the entity

**Answer: B** — Row-level WHERE clause on UPDATE ensures users can only modify their own data.

---

**Q29.** Which is a security pitfall?
- A) Using `req.user.id` to set createdBy
- B) Trusting client-sent `createdBy` field from the request body
- C) Adding @restrict to every entity
- D) Testing with multiple user profiles

**Answer: B** — Never trust client data for authorization decisions. Always use server-validated `req.user`.

---

**Q30.** The correct order of security checks in CAP is:
- A) Authorization → Authentication → Query
- B) Authentication → Service @requires → Entity @restrict → WHERE filter → Response
- C) Query → Authentication → Response
- D) Response → Authorization → Authentication

**Answer: B** — First verify identity, then check service access, then entity permissions, then row-level filtering.

---

### Quiz Scoring

| Range | Grade | Feedback |
|-------|-------|----------|
| 27-30 | Excellent | Production-security ready! |
| 21-26 | Good | Review row-level and attributes |
| 15-20 | Fair | Re-study Days 29-30 on auth |
| Below 15 | Needs Work | Go through the full week again |

---

## Session 5 & 6: Mini Project — Complete EPM Security Model (15:15 - 17:00)

### Project Brief

Implement a complete, production-quality security model for your EPM application that combines:
- Role-based access control (3 roles)
- Row-level security (users see only their data)
- Attribute-based filtering (department/region)
- Secured actions (workflow operations)
- Data isolation verification

---

### The Security Requirements

| Role | Can Access | Can Modify | Row Filter |
|------|-----------|-----------|------------|
| **Viewer** | Read Products, Read Approved POs | Nothing | Only Approved/Received POs |
| **PurchaseManager** | Read own POs, Read Products, Call actions | Create/Edit own POs | Only own POs (`createdBy = $user`) + Submitted POs (for review) |
| **Administrator** | Everything | Everything | No filter (sees all data) |

---

### Step 1: xs-security.json

```json
{
  "xsappname": "epm-management",
  "tenant-mode": "dedicated",
  "scopes": [
    { "name": "$XSAPPNAME.Read", "description": "Read access" },
    { "name": "$XSAPPNAME.Create", "description": "Create records" },
    { "name": "$XSAPPNAME.Approve", "description": "Approve POs" },
    { "name": "$XSAPPNAME.Delete", "description": "Delete records" },
    { "name": "$XSAPPNAME.Admin", "description": "Admin access" }
  ],
  "attributes": [
    { "name": "Department", "description": "User department", "valueType": "string" }
  ],
  "role-templates": [
    {
      "name": "Viewer",
      "scope-references": ["$XSAPPNAME.Read"]
    },
    {
      "name": "PurchaseManager",
      "scope-references": ["$XSAPPNAME.Read", "$XSAPPNAME.Create", "$XSAPPNAME.Approve"],
      "attribute-references": ["Department"]
    },
    {
      "name": "Administrator",
      "scope-references": ["$XSAPPNAME.Read", "$XSAPPNAME.Create", "$XSAPPNAME.Approve", "$XSAPPNAME.Delete", "$XSAPPNAME.Admin"]
    }
  ]
}
```

---

### Step 2: Secured Service Definition

```cds
service PurchasingService @(path: '/purchasing', requires: 'authenticated-user') {

  entity PurchaseOrders @(restrict: [
    { grant: 'READ', to: 'Viewer', where: 'status = ''Approved'' or status = ''Received''' },
    { grant: ['READ', 'CREATE', 'UPDATE'], to: 'PurchaseManager', where: 'createdBy = $user' },
    { grant: 'READ', to: 'PurchaseManager', where: 'status = ''Submitted''' },
    { grant: '*', to: 'Administrator' }
  ]) as projection on db.PurchaseOrders with draft
    actions {
      @(requires: 'PurchaseManager') action submit() returns { status: String; };
      @(requires: ['PurchaseManager', 'Administrator']) action approve(comment: String) returns { status: String; };
      @(requires: ['PurchaseManager', 'Administrator']) action reject(reason: String) returns { status: String; };
    };

  entity PurchaseOrderItems @(restrict: [
    { grant: 'READ', to: 'Viewer' },
    { grant: '*', to: ['PurchaseManager', 'Administrator'] }
  ]) as projection on db.PurchaseOrderItems;

  @readonly entity Products as projection on db.Products;
  @(requires: 'Administrator') entity Suppliers as projection on db.Suppliers;
  @(requires: 'Administrator') entity Categories as projection on db.Categories;
}

service CatalogService @(path: '/catalog') {
  @readonly entity Products as projection on db.Products {
    ID, productName, description, price, stock, rating, category
  };
  @readonly entity Categories as projection on db.Categories;
}
```

---

### Step 3: Mock Users for Testing

```json
{
  "cds": {
    "requires": {
      "auth": {
        "[development]": {
          "kind": "mocked",
          "users": {
            "alice": { "password": "pass", "roles": ["PurchaseManager"], "attr": { "Department": "Engineering" } },
            "bob": { "password": "pass", "roles": ["PurchaseManager"], "attr": { "Department": "Marketing" } },
            "carol": { "password": "pass", "roles": ["Viewer"] },
            "dave": { "password": "pass", "roles": ["Administrator"] },
            "eve": { "password": "pass", "roles": [] }
          }
        }
      }
    }
  }
}
```

---

### Step 4: Handler with Security Logic

```javascript
const cds = require('@sap/cds');

module.exports = function () {
  const { PurchaseOrders } = cds.entities;

  // Ensure createdBy comes from token, not client
  this.before('CREATE', 'PurchaseOrders', (req) => {
    req.data.createdBy = req.user.id;
  });

  // Only the owner OR admin can submit
  this.on('submit', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const po = await SELECT.one.from(PurchaseOrders).where({ ID });

    if (!po) req.reject(404, 'PO not found');

    // Instance-based: only owner can submit their own PO
    if (!req.user.is('Administrator') && po.createdBy !== req.user.id) {
      req.reject(403, 'You can only submit your own Purchase Orders');
    }

    if (po.status !== 'Draft') {
      req.reject(400, `Cannot submit: PO is "${po.status}"`);
    }

    await UPDATE(PurchaseOrders).set({ status: 'Submitted' }).where({ ID });
    return { status: 'Submitted' };
  });

  // Approver cannot approve their own PO (separation of duties)
  this.on('approve', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const po = await SELECT.one.from(PurchaseOrders).where({ ID });

    if (!po) req.reject(404, 'PO not found');
    if (po.status !== 'Submitted') req.reject(400, 'Only submitted POs can be approved');

    // Separation of duties: can't approve your own PO!
    if (po.createdBy === req.user.id && !req.user.is('Administrator')) {
      req.reject(403, 'You cannot approve your own Purchase Order (separation of duties)');
    }

    await UPDATE(PurchaseOrders).set({
      status: 'Approved',
      approvedBy: req.user.id,
      approvedAt: new Date().toISOString()
    }).where({ ID });

    return { status: 'Approved' };
  });
};
```

---

### Step 5: Complete Test Matrix

| Test | User | Expected |
|------|------|----------|
| Read POs (no login) | — | 401 Unauthorized |
| Read POs | eve (no roles) | 403 Forbidden (no matching grant) |
| Read POs | carol (Viewer) | Only Approved/Received POs |
| Read POs | alice (Manager) | Only Alice's POs + Submitted POs |
| Read POs | dave (Admin) | ALL POs |
| Create PO | carol (Viewer) | 403 Forbidden |
| Create PO | alice (Manager) | Success (createdBy = alice) |
| Read Alice's PO | bob (Manager) | NOT visible (row isolation!) |
| Edit Alice's PO | bob (Manager) | 403 Forbidden |
| Delete PO | alice (Manager) | 403 Forbidden |
| Delete PO | dave (Admin) | Success |
| Submit PO | alice (own PO) | Success |
| Submit PO | bob (Alice's PO) | 403 "only submit your own" |
| Approve PO | alice (own PO) | 403 "separation of duties" |
| Approve PO | bob (Alice's PO) | Success |
| Read Suppliers | carol/alice/bob | 403 (Admin only) |
| Read Suppliers | dave (Admin) | Success |
| Read Products (catalog) | anyone authed | Success (public readonly) |

---

### Verification Checklist

- [ ] Anonymous access returns 401 on all secured services
- [ ] User with no roles gets 403 on secured entities
- [ ] Viewer can only READ approved POs
- [ ] Manager A creates PO → Manager B CANNOT see it
- [ ] Manager A creates PO → Manager A CAN see it
- [ ] Admin sees ALL POs regardless of creator
- [ ] Manager cannot delete (only admin can)
- [ ] Manager cannot submit another manager's PO
- [ ] Manager cannot approve their OWN PO (separation of duties!)
- [ ] Supplier entity only accessible by Admin
- [ ] Catalog service (Products) is readable by everyone
- [ ] createdBy is ALWAYS set from server (req.user.id), not client

---

## End of Day Summary

### What We Covered Today

| Session | Topic | Key Takeaway |
|---------|-------|--------------|
| 1 | Row-Level Security | `where: 'createdBy = $user'` filters per user |
| 2 | Attributes & Pitfalls | `$user.attr.X` for department/region filtering |
| 3 | Hands-on | Tested data isolation between Alice, Bob, Carol, Dave |
| 4 | Weekly Quiz | 30 questions covering Days 26-30 |
| 5-6 | Mini Project | Complete security model with roles + rows + instances |

---

### Week 6 Complete: The Security Stack

```
┌─────────────────────────────────────────────────────────────┐
│          THE COMPLETE SECURITY PICTURE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Layer 1: AUTHENTICATION                                    │
│    → "Are you logged in?" (IdP + XSUAA + JWT)              │
│                                                             │
│  Layer 2: SERVICE-LEVEL AUTHORIZATION                       │
│    → @requires on service: "Can you enter this service?"    │
│                                                             │
│  Layer 3: ENTITY-LEVEL AUTHORIZATION                        │
│    → @restrict on entity: "What operations can you do?"     │
│                                                             │
│  Layer 4: ROW-LEVEL SECURITY                                │
│    → where clause: "Which records can you see?"             │
│                                                             │
│  Layer 5: INSTANCE-BASED CHECKS                             │
│    → Handler logic: "Can you do THIS to THIS record?"       │
│                                                             │
│  Layer 6: FIELD-LEVEL PROTECTION                            │
│    → Projections: "Which fields can you see?"               │
│                                                             │
│  ALL LAYERS TOGETHER = Enterprise-Grade Security!           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### The One-Liner

> **Row-level security ensures that even users with the SAME role see DIFFERENT data — your POs are invisible to me, and mine are invisible to you.**

---

### Looking Ahead: Day 31

Next up: **Deployment to SAP BTP Cloud Foundry**
- MTA (Multi-Target Application) architecture
- mta.yaml configuration
- Building and deploying the complete app
- Binding HANA + XSUAA + App Router
- Your app running live in the cloud!

---

*End of Day 30 — Week 6 Complete! Your application is now enterprise-secure with Git, HANA, and multi-layered authorization!*
