# Day 5: Pre-Class Group Assignment

## "Cloud Startup Pitch" — Build a SaaS Product on Paper

### Duration: 1 Hour (Before Class Starts)
### Team Size: 4-5 members per team

---

## The Challenge

Your team is a **startup company** that just received ₹50 Lakhs in funding. Your mission: design a **SaaS product** that solves a real business problem — and pitch it to investors (the class)!

But here's the catch: you must use **software engineering best practices** and explain HOW you'd build it using cloud technology.

---

## The Rules

1. Your product must be a **cloud-based SaaS application**
2. It must solve a real problem for businesses in India
3. You must explain the **technical architecture** (not just the business idea)
4. You must demonstrate understanding of **software engineering fundamentals**

---

## Your Mission (60 minutes)

### Part A: The Product (15 minutes)

#### Choose ONE Problem to Solve

Pick a problem from the list below (or invent your own):

| # | Problem | Target Customer |
|---|---------|----------------|
| 1 | Small shops can't track inventory properly (use notebooks/Excel) | Kirana stores, small retailers |
| 2 | Colleges struggle to manage student attendance and grades | Schools and colleges |
| 3 | Restaurants waste food because they can't predict daily demand | Restaurants, cloud kitchens |
| 4 | Small manufacturers can't track raw material and production | SME manufacturers |
| 5 | Freelancers struggle to send professional invoices and track payments | Freelancers, small agencies |
| 6 | Housing societies can't manage maintenance payments and complaints | Apartment complexes |

#### Define Your Product

Fill in:

| Question | Your Answer |
|----------|-------------|
| **Product Name** | (Choose something catchy!) |
| **One-line description** | (What it does in 1 sentence) |
| **Target users** | (Who uses it?) |
| **Top 3 features** | (What can users do?) |
| **How users access it** | (Web app? Mobile? Both?) |
| **Pricing model** | (Free tier + paid? Monthly subscription?) |

---

### Part B: The Technical Architecture (20 minutes)

Now design HOW you'd build it. You must answer these questions:

#### 1. Cloud Deployment

| Decision | Your Choice | Why? |
|----------|-------------|------|
| IaaS, PaaS, or SaaS platform? | | |
| Which cloud provider? (AWS/Azure/GCP/SAP BTP) | | |
| Which region to host in? | | |
| Public, Private, or Hybrid cloud? | | |

#### 2. Application Architecture

Draw a diagram showing:

```
Template:
+--------------------------------------------------+
|  USERS (who accesses your product?)              |
|  - Type 1: _____________                        |
|  - Type 2: _____________                        |
+--------------------------------------------------+
              |
              | (How do they access? Browser/Mobile/Both?)
              ↓
+--------------------------------------------------+
|  FRONTEND (User Interface)                       |
|  Technology: _____________                       |
+--------------------------------------------------+
              |
              | (API calls)
              ↓
+--------------------------------------------------+
|  BACKEND (Business Logic)                        |
|  Technology: _____________                       |
|  Key Logic: _____________                        |
+--------------------------------------------------+
              |
              | (Read/Write data)
              ↓
+--------------------------------------------------+
|  DATABASE                                        |
|  Technology: _____________                       |
|  Key Data Stored: _____________                  |
+--------------------------------------------------+
              |
              | (External connections?)
              ↓
+--------------------------------------------------+
|  EXTERNAL SERVICES                               |
|  - Payment gateway: _____________                |
|  - Email/SMS: _____________                      |
|  - Any other: _____________                      |
+--------------------------------------------------+
```

#### 3. Software Engineering Practices

For each practice, explain how YOUR product would use it:

| Practice | How Your Product Uses It |
|----------|------------------------|
| **Version Control (Git)** | How would your dev team manage code? Branching strategy? |
| **Environment Separation** | How would you separate dev/test/production? |
| **Authentication** | How do users log in? (Email+password? Google login? OTP?) |
| **Authorization** | What roles exist? (Admin, User, Viewer?) Who can do what? |
| **Database Design** | What are your main data tables/entities? (Name at least 4) |
| **API Design** | What are 3 key API endpoints? (e.g., GET /orders, POST /invoices) |
| **Error Handling** | What happens when something goes wrong? (user sees what?) |
| **Scalability** | What if you get 100x users tomorrow? How does your design handle it? |

---

### Part C: The "Day in the Life" of Your Code (15 minutes)

Describe what happens technically when a user performs ONE key action in your app.

**Example:** If your product is an invoice app, describe what happens when a user clicks "Create Invoice":

```
Write out the flow like this:

User Action: [What the user clicks/does]
    ↓
Step 1: Frontend _______________
    ↓
Step 2: API call _______________
    ↓
Step 3: Backend validates _______________
    ↓
Step 4: Database _______________
    ↓
Step 5: Response _______________
    ↓
Step 6: User sees _______________
```

Also answer:
- What if the user enters invalid data? What happens?
- What if the database is down? What does the user see?
- What if 1000 users do this at the same time? Does it break?

---

### Part D: The Development Plan (10 minutes)

#### Your Team Roles

Assign roles to each team member as if you were a real startup:

| Role | Team Member | Responsibility |
|------|-------------|---------------|
| **Product Owner** | | Decides features, talks to customers |
| **Frontend Developer** | | Builds the user interface |
| **Backend Developer** | | Builds the business logic and APIs |
| **DevOps / Cloud Engineer** | | Manages deployment, monitoring, scaling |
| **QA / Tester** | | Tests everything, finds bugs |

#### Development Timeline

Plan a 4-week sprint to build v1.0:

| Week | What Gets Built | Who Leads |
|------|----------------|-----------|
| Week 1 | | |
| Week 2 | | |
| Week 3 | | |
| Week 4 | | |

#### Git Workflow

How would your team use Git?

```
Choose and draw your branching strategy:

Option A: Simple (good for small teams)
  main ─────────────────────────────►
    └── feature-1 ──── merge back ──┘
    └── feature-2 ──── merge back ──┘

Option B: GitFlow (good for releases)
  main ──────────────────────────────►
    └── develop ─────────────────────►
         └── feature-login ── merge ─┘
         └── feature-payment ─ merge ┘

Which do you choose and why?
```

---

## The Pitch (Presentation)

Prepare a **4-minute pitch** to the class structured as:

| Section | Time | Content |
|---------|------|---------|
| **The Problem** | 30 sec | What problem exists? Who suffers? |
| **The Solution** | 30 sec | Your product name, what it does, demo the UI (on paper!) |
| **Technical Architecture** | 90 sec | Show your architecture diagram, explain stack choices |
| **Engineering Practices** | 60 sec | Git workflow, environments, security, scalability |
| **Why It Will Succeed** | 30 sec | What makes your solution better? |

---

## Judging Criteria

| Criteria | Max Points | What Judges Look For |
|----------|-----------|---------------------|
| **Problem Clarity** | 3 | Is the problem real and well-defined? |
| **Product Design** | 3 | Does the solution make sense? Would people use it? |
| **Architecture Diagram** | 4 | Correct layers? Logical flow? Technology choices justified? |
| **SE Practices** | 4 | Git workflow, environments, auth, error handling explained? |
| **Scalability Thinking** | 3 | Can it handle growth? Cloud benefits used? |
| **Presentation** | 3 | Clear, organized, all members spoke? |
| **Total** | **20** | |

### Bonus Points (+2 each)

| Bonus | Description |
|-------|-------------|
| 🏆 **Best Pitch** | Most convincing presentation (would you invest?) |
| 🏆 **Most Technical** | Deepest technical detail and correctness |
| 🏆 **Most Creative** | Most innovative problem/solution combination |

---

## Concept Reference Sheet

Use these concepts in your design (prove you learned them this week!):

### Cloud Concepts (Day 1)
- [ ] Identified correct service model (IaaS/PaaS/SaaS) for your product
- [ ] Explained why cloud (not on-premise) is right for your product
- [ ] Mentioned scaling benefits

### BTP Architecture (Day 2)
- [ ] Mentioned runtime environment (where code runs)
- [ ] Mentioned separation of frontend/backend
- [ ] Thought about regions (data residency)

### Account & Access (Day 3)
- [ ] Defined user roles (who can do what)
- [ ] Separated environments (dev/test/prod)
- [ ] Thought about entitlements/quotas (limits)

### Development Tools (Day 4)
- [ ] Mentioned version control (Git) workflow
- [ ] Identified tech stack (frontend + backend + DB)
- [ ] Mentioned package management (npm/dependencies)

### Services (Day 5)
- [ ] Identified what services the app needs (database, auth, integration)
- [ ] Explained how services connect (binding/credentials)
- [ ] Thought about external integrations (payments, SMS, etc.)

---

## Sample: What a Good Answer Looks Like

**Product:** "QuickBill" — Invoicing for Indian Freelancers

**Architecture:**
```
Users: Freelancers (create invoices), Clients (view/pay invoices)
         |
         | Browser + Mobile (PWA)
         ↓
Frontend: React.js (responsive web app)
         |
         | REST API (HTTPS)
         ↓
Backend: Node.js (CAP framework on SAP BTP)
  - Invoice creation logic
  - GST calculation
  - Payment status tracking
  - PDF generation
         |
         | SQL queries
         ↓
Database: SAP HANA Cloud
  - Tables: Users, Invoices, LineItems, Payments, Clients
         |
         | External APIs
         ↓
External Services:
  - Razorpay (payments)
  - SendGrid (email invoices)
  - AWS S3 (store PDF files)
```

**Git Workflow:** GitFlow — main (production), develop (integration), feature branches per developer. PRs required for merge.

**Environments:** 3 subaccounts — Dev (ap21), Staging (ap21), Production (ap21). Only DevOps can deploy to production.

**Scalability:** Cloud Foundry auto-scales. During tax season (March), scale to 5 instances. Rest of year, 2 instances.

---

## Why This Assignment?

| Skill Practiced | How |
|----------------|-----|
| **Systems thinking** | Designing an architecture end-to-end |
| **Cloud decisions** | Choosing cloud model, provider, region |
| **Software engineering** | Git workflow, environments, roles, APIs |
| **Security mindset** | Authentication, authorization, data protection |
| **Collaboration** | Team roles, parallel work, coordination |
| **Communication** | Pitching technical ideas clearly |
| **Connecting Week 1** | Using ALL concepts from Days 1-5 in one project |

---

*Time to build something amazing! Good luck, founders!* 🚀
