## What You'll Learn Today

By the end of this session, you will be able to:
- Explain WHY SAP CAP exists and what problems it solves
- Compare CAP with traditional Node.js/Express development
- Describe CAP architecture (CDS, Services, Runtime)
- Differentiate between CAP and RAP (ABAP-based approach)
- Create a CAP project from scratch using `cds init`
- Understand every file and folder in a CAP project
- Run a CAP app locally with `cds watch`
- Write your first CDS file

---

## Welcome to Week 3! (09:00 - 09:15)

### The Journey So Far

```
Week 1: SAP BTP Foundations (Cloud, Architecture, Tools)
Week 2: JavaScript & Node.js (Language + Express + REST APIs)
                    ↓
Week 3: NOW — SAP CAP Framework (The Main Event!) ⭐
```

### What Changes This Week

| Week 2 (What you did) | Week 3 (What CAP does for you) |
|----------------------|-------------------------------|
| Wrote 100+ lines for a REST API | CAP generates APIs from 5 lines of CDS |
| Manual routing with Express | Automatic OData endpoints |
| Manual database queries | Declarative data model → auto database |
| Manual input validation | Built-in validations |
| Manual authentication code | One annotation: `@requires: 'admin'` |
| Manual CRUD handlers | Generated automatically |

**Week 3 will feel like magic.** Everything you struggled with in Express.js — CAP does it in a few lines.

---

## Session 1: Why CAP? Problems It Solves (09:15 - 10:30)

### The Problem: Building Enterprise Apps Is HARD

Imagine you're building a Purchase Order app for a company. Here's what you need:

```
WITHOUT a framework (plain Node.js/Express):
─────────────────────────────────────────
✗ Design database schema (SQL)
✗ Write migration scripts
✗ Build REST/OData endpoints manually
✗ Implement CRUD for every entity
✗ Add input validation
✗ Add authentication (OAuth, JWT)
✗ Add authorization (roles, permissions)
✗ Handle draft/edit scenarios
✗ Add localization (multi-language)
✗ Add multi-tenancy (SaaS)
✗ Connect to SAP S/4HANA
✗ Deploy to Cloud Foundry
✗ Configure HANA database
✗ Build Fiori UI annotations

Effort: 3-6 months for a senior team
Lines of code: 10,000+
```

```
WITH SAP CAP:
─────────────
✓ Define data model in CDS (20 lines)
✓ Define service in CDS (10 lines)
✓ Add custom logic in JS (where needed)
✓ Everything else is AUTOMATIC!

Effort: 2-4 weeks
Lines of code: 500-1000
```

---

### What is SAP CAP?

**CAP** = **C**loud **A**pplication **P**rogramming Model

It's SAP's **opinionated framework** for building enterprise-grade cloud applications. Think of it as a "super-powered Express.js" designed specifically for business applications on SAP BTP.

```
CAP = CDS (define WHAT) + Runtime (does HOW) + Best Practices (knows WHY)
```

**Analogy:**
- Express.js = a blank canvas and paint (you decide everything)
- SAP CAP = a paint-by-numbers kit (structure is provided, you add creativity where needed)

---

### CAP's Core Philosophy

| Principle | What It Means | Benefit |
|-----------|-------------|---------|
| **Convention over Configuration** | Follow standard patterns, less setup | Less boilerplate, faster development |
| **Declarative over Imperative** | Describe WHAT you want, not HOW to do it | Shorter code, fewer bugs |
| **Focus on Domain** | Write business logic, not plumbing | Developer productivity |
| **Grow as You Go** | Start simple, add complexity only when needed | No over-engineering |
| **Open Standards** | Based on OData, SQL, REST, JSON | Interoperability |

---

### Problems CAP Solves

#### Problem 1: Repetitive CRUD Code

```javascript
// WITHOUT CAP — Express.js (you write ALL of this):
app.get('/products', async (req, res) => { /* query DB, format, return */ });
app.get('/products/:id', async (req, res) => { /* find one, 404 check, return */ });
app.post('/products', async (req, res) => { /* validate, insert, return 201 */ });
app.put('/products/:id', async (req, res) => { /* validate, update, return */ });
app.delete('/products/:id', async (req, res) => { /* find, delete, return */ });
// × 10 entities = 50 route handlers, 500+ lines of boilerplate!
```

```javascript
// WITH CAP — just define the service:
service CatalogService {
  entity Products as projection on db.Products;
}
// DONE! All CRUD endpoints generated automatically.
// GET, POST, PUT, PATCH, DELETE — all working!
```

#### Problem 2: Database Schema + Migration

```javascript
// WITHOUT CAP:
// - Write CREATE TABLE SQL
// - Write ALTER TABLE for changes
// - Track migrations
// - Handle SQLite (dev) vs HANA (prod) differences

// WITH CAP:
entity Products {
  key ID : UUID;
  name   : String(100);
  price  : Decimal(10,2);
  stock  : Integer;
}
// CAP generates SQL for SQLite (local) AND HANA (production) automatically!
```

#### Problem 3: Authentication & Authorization

```javascript
// WITHOUT CAP:
// - Configure XSUAA manually
// - Parse JWT tokens
// - Check roles in every route handler
// - Handle token refresh

// WITH CAP — one line:
service AdminService @(requires: 'admin') {
  entity Products as projection on db.Products;
}
// Done! Only users with 'admin' role can access this service.
```

#### Problem 4: Connecting to SAP Systems

```javascript
// WITHOUT CAP:
// - Download API spec from SAP API Hub
// - Write HTTP client code
// - Handle OAuth token flow
// - Map response data to your model
// - Handle errors, retries, timeouts

// WITH CAP:
// Just import the external service definition:
using { API_BUSINESS_PARTNER } from './external/API_BUSINESS_PARTNER';

service MyService {
  entity Suppliers as projection on API_BUSINESS_PARTNER.A_BusinessPartner;
}
// CAP handles authentication, connection, error handling!
```
