## Session 3: Hands-on — Create xs-security.json & Define Roles (12:00 - 13:00)

### Step 1: Generate the Security Configuration

```bash
cd ~/projects/po-management

# CAP can generate a starter xs-security.json:
cds add xsuaa
```

This creates `xs-security.json` in your project root with a basic template.

---
## Very Importnt (service.cds )

service CatalogService @(requires: 'authenticated-user')

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
