# Day 1: Introduction to Cloud Computing & SAP Ecosystem

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:30 | Welcome, Introductions & Course Overview | 30 min |
| 09:30 - 10:30 | Session 1: Cloud Computing Fundamentals | 60 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 11:45 | Session 2: Cloud Deployment Models & Providers | 60 min |
| 11:45 - 12:45 | Session 3: On-Premise vs Cloud & Enterprise Benefits | 60 min |
| 12:45 - 13:30 | Lunch Break | 45 min |
| 13:30 - 14:30 | Session 4: SAP Ecosystem & Product Landscape | 60 min |
| 14:30 - 15:30 | Session 5: SAP BTP Deep-Dive | 60 min |
| 15:30 - 15:45 | Break | 15 min |
| 15:45 - 16:45 | Session 6: Hands-on Exploration & Activities | 60 min |
| 16:45 - 17:00 | Assessment & Wrap-up | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain what Cloud Computing is and how it works
- Differentiate between IaaS, PaaS, and SaaS with real examples
- Compare Public, Private, and Hybrid cloud deployment models
- Identify the major cloud providers and their strengths
- List the benefits and challenges of Cloud adoption for enterprises
- Describe the complete SAP product ecosystem and how products connect
- Explain SAP BTP's role in SAP's intelligent enterprise strategy
- Navigate SAP Discovery Center and SAP BTP documentation

---

## Welcome & Course Overview (09:00 - 09:30)

### About This Training Program

- **Duration:** 45 days, 8 hours/day
- **Goal:** Transform you from zero coding experience to building full-stack SAP BTP applications
- **End Result:** You will build and deploy a complete enterprise application on SAP BTP

### What You Will Build During This Course

```
Week 1-2: Foundations (Cloud + BTP + JavaScript)
Week 3-4: Backend Development (CAP Framework + Database + Services)
Week 5:   Frontend (SAP Fiori UI)
Week 6:   HANA Cloud + Security
Week 7:   Cloud Deployment
Week 8-9: Integration + AI (Joule) + Capstone Project
```

### Daily Format

| Block | Duration | Focus |
|-------|----------|-------|
| Theory Sessions | ~3 hours | Concepts, demonstrations, diagrams |
| Hands-on Labs | ~3.5 hours | Guided exercises, practice |
| Assessment | ~0.5 hours | MCQ quiz to verify understanding |
| Buffer/Discussions | ~1 hour | Q&A, group activities, breaks |

### Icebreaker Activity (10 minutes)

Introduce yourself to the group:
1. Your name
2. Your educational background
3. One app/website you use daily (we'll connect this to cloud concepts later!)
4. What excites you about learning SAP development?

---

## Session 1: Cloud Computing Fundamentals (09:30 - 10:30)

### What is Cloud Computing?

#### The Simple Definition

**Cloud Computing** = Delivering computing services (servers, storage, databases, networking, software, analytics, AI) over the **internet** ("the cloud"), offering faster innovation, flexible resources, and pay-as-you-go pricing.

#### Think of it this way...

Imagine you want to watch a movie:
- **Old way (On-Premise):** You buy a DVD player, buy the DVD, store it at home, maintain the player yourself.
- **New way (Cloud):** You open Netflix, pick a movie, and stream it. No equipment to buy or maintain!

**Cloud Computing** works the same way for businesses. Instead of buying and maintaining their own servers (big computers), companies **rent** computing power over the internet.

#### The NIST Definition (Industry Standard)

The National Institute of Standards and Technology (NIST) defines cloud computing with 5 essential characteristics:

| Characteristic | What It Means | Example |
|---|---|---|
| **On-demand self-service** | Get resources whenever you need without calling anyone | Spin up a new server at 2 AM from your laptop |
| **Broad network access** | Available over the internet from any device | Access your app from phone, laptop, tablet |
| **Resource pooling** | Provider shares resources among multiple customers | 1000 companies sharing the same data center |
| **Rapid elasticity** | Scale up or down instantly | Handle 10x traffic during a sale event |
| **Measured service** | Pay only for what you use | Like an electricity bill — use more, pay more |

---

### The 3 Types of Cloud Services (Service Models)

Think of it like ordering food:

| Service Type | What It Means | Real-Life Example | Food Analogy |
|---|---|---|---|
| **IaaS** (Infrastructure as a Service) | You rent the basic hardware (servers, storage, network) | AWS EC2, Azure VMs, Google Compute Engine | Renting a kitchen - you cook everything yourself |
| **PaaS** (Platform as a Service) | You get a ready platform to build apps on | SAP BTP, Google App Engine, Heroku | Meal kit delivery - ingredients are prepped, you just assemble |
| **SaaS** (Software as a Service) | You use ready-made software over the internet | Gmail, SAP S/4HANA Cloud, Zoom, Slack | Restaurant - food is made and served to you |

### The Responsibility Model — Who Manages What?

```
                    On-Premise    IaaS         PaaS         SaaS
                    ----------    ----         ----         ----
Applications      | You      |  | You    |  | You    |  | Provider |
Data              | You      |  | You    |  | You    |  | Provider |
Runtime           | You      |  | You    |  | Provider|  | Provider |
Middleware        | You      |  | You    |  | Provider|  | Provider |
Operating System  | You      |  | You    |  | Provider|  | Provider |
Virtualization    | You      |  | Provider|  | Provider|  | Provider |
Servers           | You      |  | Provider|  | Provider|  | Provider |
Storage           | You      |  | Provider|  | Provider|  | Provider |
Networking        | You      |  | Provider|  | Provider|  | Provider |

Legend:  You = Customer manages    Provider = Cloud provider manages
```

**Key Insight:** As you move from IaaS → PaaS → SaaS, you manage LESS, and the cloud provider manages MORE.

### Where Does SAP CAP Fit?

SAP BTP is a **PaaS** — it gives you:
- Runtime environment (Node.js, Java)
- Database (HANA Cloud)
- Pre-built services (authentication, integration)
- Deployment tools

You focus on: **Writing business logic and building your application**

---

### Real-World Examples of Each Service Model

#### IaaS Examples in Practice
| Company | Use Case | Why IaaS? |
|---------|----------|-----------|
| A startup | Renting virtual servers on AWS | Need full control over OS and software stack |
| A bank | Running custom security software on Azure VMs | Regulatory requirement to control every layer |
| A gaming company | Powerful GPU servers for game rendering | Need raw compute power |

#### PaaS Examples in Practice
| Company | Use Case | Why PaaS? |
|---------|----------|-----------|
| An enterprise | Building SAP BTP apps (this is what we'll do!) | Focus on business logic, not infrastructure |
| A web agency | Deploying Node.js apps on Heroku | Don't want to manage servers |
| A data team | Running analytics on Google BigQuery | Just upload data and query |

#### SaaS Examples in Practice
| Company | Use Case | Why SaaS? |
|---------|----------|-----------|
| Any company | Email via Gmail/Outlook 365 | No one wants to run their own email server |
| An HR department | Managing employees in SAP SuccessFactors | Ready-to-use HR features |
| A sales team | Tracking deals in Salesforce | CRM out-of-the-box |

---

### Quick Check - Can you classify these?

Try to identify if each is IaaS, PaaS, or SaaS:
1. Google Docs - ?
2. Amazon Web Services (renting a virtual server) - ?
3. Heroku (deploying your app without worrying about servers) - ?
4. Dropbox - ?
5. SAP BTP - ?
6. Microsoft Azure Virtual Machines - ?
7. Salesforce CRM - ?
8. AWS Lambda (running code without managing servers) - ?

<details>
<summary>Click to see answers</summary>

1. Google Docs - **SaaS** (ready-to-use software)
2. AWS EC2 - **IaaS** (raw infrastructure)
3. Heroku - **PaaS** (platform to build on)
4. Dropbox - **SaaS** (ready-to-use storage service)
5. SAP BTP - **PaaS** (platform to build applications)
6. Azure VMs - **IaaS** (you manage the virtual machine)
7. Salesforce CRM - **SaaS** (ready-to-use CRM)
8. AWS Lambda - **PaaS** (serverless — you just write code)

</details>

---

### How Cloud Computing Actually Works (Technical Overview)

```
                         THE CLOUD
    +----------------------------------------------------+
    |                                                    |
    |   +----------+    +----------+    +----------+    |
    |   | Server 1 |    | Server 2 |    | Server 3 |    |
    |   +----------+    +----------+    +----------+    |
    |        |               |               |          |
    |   +--------------------------------------------+  |
    |   |         VIRTUALIZATION LAYER               |  |
    |   | (Splits physical servers into virtual ones) |  |
    |   +--------------------------------------------+  |
    |        |               |               |          |
    |   +--------+  +--------+  +--------+  +--------+ |
    |   | VM-A   |  | VM-B   |  | VM-C   |  | VM-D   | |
    |   |Customer|  |Customer|  |Customer|  |Customer| |
    |   |   1    |  |   2    |  |   1    |  |   3    | |
    |   +--------+  +--------+  +--------+  +--------+ |
    |                                                    |
    +----------------------------------------------------+
                         |
                    (Internet)
                         |
         +------+   +------+   +------+
         |User A|   |User B|   |User C|
         +------+   +------+   +------+
```

**Key Concepts:**
- **Virtualization:** One physical server is split into multiple virtual machines (VMs)
- **Multi-tenancy:** Multiple customers share the same physical hardware (but isolated from each other)
- **API-driven:** Everything is controlled through software, not by physically touching servers

---

## Session 2: Cloud Deployment Models & Providers (10:45 - 11:45)

### Cloud Deployment Models

Not all clouds are the same! There are 4 deployment models:

| Model | What It Is | Who Uses It | Example |
|-------|-----------|-------------|---------|
| **Public Cloud** | Shared infrastructure, open to everyone | Startups, SMBs, most companies | AWS, Azure, GCP, SAP BTP |
| **Private Cloud** | Dedicated infrastructure for one organization | Banks, government, healthcare | Company's own OpenStack setup |
| **Hybrid Cloud** | Mix of public and private cloud | Large enterprises | Sensitive data on-prem + apps in public cloud |
| **Community Cloud** | Shared among organizations with common needs | Healthcare consortiums, government agencies | Government community cloud (GovCloud) |

### Public Cloud — Deep Dive

```
Public Cloud:
+--------------------------------------------------+
|  Cloud Provider's Data Center                    |
|                                                  |
|  +--------+ +--------+ +--------+ +--------+    |
|  |Company | |Company | |Company | |Company |    |
|  |   A    | |   B    | |   C    | |   D    |    |
|  +--------+ +--------+ +--------+ +--------+    |
|                                                  |
|  All sharing the same physical infrastructure    |
|  but logically isolated from each other          |
+--------------------------------------------------+
```

**Pros:** Low cost, no maintenance, instant scaling, global reach
**Cons:** Less control, shared resources, data sovereignty concerns

### Private Cloud — Deep Dive

```
Private Cloud:
+--------------------------------------------------+
|  Company's Own Data Center (or dedicated hosted) |
|                                                  |
|  +--------+ +--------+ +--------+ +--------+    |
|  | Dept A | | Dept B | | Dept C | | Dept D |    |
|  | (HR)   | |(Finance)| |(Sales) | | (IT)   |    |
|  +--------+ +--------+ +--------+ +--------+    |
|                                                  |
|  All resources dedicated to ONE organization     |
+--------------------------------------------------+
```

**Pros:** Full control, highest security, compliance-friendly
**Cons:** Expensive, requires IT staff, slower to scale

### Hybrid Cloud — Deep Dive

```
Hybrid Cloud:
+-------------------+          +-------------------+
|  Private Cloud    |          |  Public Cloud     |
|  (Sensitive Data) |  <--->   |  (Apps & Scale)   |
|                   |  Secure  |                   |
|  - Customer PII   |  Link    |  - Web app        |
|  - Financial data  |          |  - Analytics      |
|  - Compliance      |          |  - AI/ML          |
+-------------------+          +-------------------+
```

**Why Hybrid?** Many companies can't put ALL data in public cloud (regulations), but still want cloud benefits for their applications.

**Real Example:** A hospital keeps patient records in private cloud (HIPAA compliance) but uses public cloud for their appointment booking app.

---

### Major Cloud Providers — The Big Three

| Provider | Market Share | Strengths | SAP Relationship |
|----------|-------------|-----------|------------------|
| **AWS** (Amazon) | ~32% | Widest service catalog, most mature | SAP BTP runs on AWS |
| **Microsoft Azure** | ~23% | Enterprise integration, Microsoft ecosystem | SAP BTP runs on Azure |
| **Google Cloud (GCP)** | ~11% | Data/AI/ML, Kubernetes | SAP BTP runs on GCP |

#### Other Notable Providers
- **Alibaba Cloud** — Dominant in China/Asia
- **IBM Cloud** — Enterprise/mainframe workloads
- **Oracle Cloud** — Database-heavy workloads
- **SAP** — SAP BTP (runs ON top of AWS/Azure/GCP)

### SAP's Multi-Cloud Strategy

SAP BTP is unique — it runs on **multiple** cloud providers:

```
+--------------------------------------------+
|              SAP BTP                        |
|   (One platform, multiple cloud backends)  |
+--------------------------------------------+
        |            |            |
   +---------+  +---------+  +---------+
   |   AWS   |  |  Azure  |  |   GCP   |
   +---------+  +---------+  +---------+
```

**Why does this matter?**
- Companies choose the region/provider closest to their users
- Avoid vendor lock-in
- Meet data residency requirements (data must stay in certain countries)

---

### Cloud Computing Market Size

> The global cloud computing market is valued at over **$600 billion** (2024) and growing at **20%+ per year**.

This means: **Learning cloud skills = job security!**

### Discussion Activity (10 minutes)

In pairs, discuss:
1. What cloud services do you use in daily life? (Think about: email, streaming, file storage, social media)
2. Are these IaaS, PaaS, or SaaS?
3. Do you think they use Public, Private, or Hybrid cloud?

---

## Session 3: On-Premise vs Cloud & Enterprise Benefits (11:45 - 12:45)

### What is On-Premise?

"On-Premise" means a company buys its own servers, installs them in their office (or data center), and manages everything themselves.

### The Traditional IT Setup (Before Cloud)

```
Company's Office/Data Center:
+--------------------------------------------------+
|                                                  |
|  +------+  +------+  +------+  +------+         |
|  |Server|  |Server|  |Server|  |Server|         |
|  |  1   |  |  2   |  |  3   |  |  4   |         |
|  +------+  +------+  +------+  +------+         |
|                                                  |
|  +------------------------------------------+   |
|  | Networking Equipment (routers, switches)  |   |
|  +------------------------------------------+   |
|                                                  |
|  +------------------------------------------+   |
|  | Cooling Systems (AC units)                |   |
|  +------------------------------------------+   |
|                                                  |
|  +------------------------------------------+   |
|  | UPS & Backup Power                        |   |
|  +------------------------------------------+   |
|                                                  |
|  IT Team: 5-10 people to manage all this         |
+--------------------------------------------------+
```

---

### Comprehensive Comparison: On-Premise vs Cloud

| Factor | On-Premise | Cloud |
|---|---|---|
| **Upfront Cost** | ₹50L - ₹5Cr+ (servers, networking, cooling) | ₹0 (pay monthly) |
| **Monthly Cost** | Electricity + maintenance + salaries | Usage-based subscription |
| **Setup Time** | 3-6 months (procurement, installation, configuration) | Minutes to hours |
| **Maintenance** | You handle everything (hardware, updates, patches, security) | Cloud provider handles it |
| **Scaling Up** | Buy more hardware, wait weeks for delivery | Click a button, ready in minutes |
| **Scaling Down** | Hardware sits idle (wasted money) | Stop paying for what you don't use |
| **Access** | Usually only from office network (or VPN) | From anywhere with internet |
| **Security** | 100% your responsibility | Shared responsibility with provider |
| **Disaster Recovery** | Build your own backup site (expensive!) | Built-in, multiple copies across regions |
| **Updates** | Manual — schedule downtime, test, deploy | Automatic or one-click |
| **Hardware Refresh** | Every 3-5 years (expensive cycle) | Provider handles it transparently |
| **Compliance** | Full control (good for strict regulations) | Provider has certifications, but less control |
| **Team Required** | Large IT team needed | Smaller team, focus on business logic |

---

### Total Cost of Ownership (TCO) — A Real Example

**Scenario:** A company needs to run a web application for 100 concurrent users.

#### On-Premise Cost (Year 1)

| Item | Cost (₹) |
|------|-----------|
| 2 Physical Servers | 8,00,000 |
| Networking equipment | 2,00,000 |
| Operating System licenses | 1,50,000 |
| Server room setup (cooling, power) | 3,00,000 |
| IT personnel (1 admin, partial) | 6,00,000/year |
| Electricity | 1,20,000/year |
| **Total Year 1** | **~21,70,000** |

#### Cloud Cost (Year 1)

| Item | Cost (₹) |
|------|-----------|
| 2 Virtual servers (monthly) | 12,000/month = 1,44,000/year |
| Managed database | 8,000/month = 96,000/year |
| Backup & monitoring | 3,000/month = 36,000/year |
| **Total Year 1** | **~2,76,000** |

**Savings in Year 1: ~₹19 Lakhs!**

---

### Benefits of Cloud for Enterprises

#### 1. Cost Optimization
- **No capital expenditure (CapEx)** — only operational expenditure (OpEx)
- Pay only for what you consume
- No over-provisioning "just in case"

#### 2. Speed & Agility
- Deploy new applications in minutes, not months
- Experiment with new ideas quickly (fail fast, learn fast)
- Faster time-to-market for new products

#### 3. Global Scale
- Deploy in 25+ regions worldwide
- Serve customers close to where they are (low latency)
- Expand to new markets instantly

#### 4. Innovation Access
- AI/ML services ready to use (no PhD required!)
- IoT platforms, blockchain, analytics — all available as services
- Always the latest technology without manual upgrades

#### 5. Reliability & Availability
- Cloud providers guarantee 99.95%+ uptime (SLA)
- Automatic failover if hardware fails
- Data replicated across multiple locations

#### 6. Security Investment
- AWS/Azure/GCP spend **billions** per year on security
- 2000+ security certifications
- 24/7 security monitoring by thousands of experts
- Physical security (biometrics, guards, cameras)

#### 7. Sustainability
- Shared resources = less waste
- Cloud data centers are more energy-efficient
- Major providers committed to 100% renewable energy

---

### When Cloud is NOT the Best Choice

Cloud isn't always the answer! Consider on-premise when:

| Scenario | Why On-Premise Might Be Better |
|----------|-------------------------------|
| Ultra-low latency needed | Data center right next to the factory floor |
| Strict data sovereignty | Government mandates data cannot leave the country |
| Legacy systems that can't be migrated | Old mainframe applications too complex to move |
| Predictable, constant workload | If usage never changes, owning hardware may be cheaper long-term |
| No internet connectivity | Remote locations like oil rigs, ships |

---

### Real-World Case Studies

#### Case Study 1: Netflix
- **Before:** Owned data centers, DVD mailing business
- **After:** 100% on AWS cloud
- **Result:** Serves 250M+ subscribers across 190 countries, scales during peak hours automatically
- **Key Insight:** During a popular show launch, Netflix scales to handle 10x normal traffic, then scales back down

#### Case Study 2: Airbnb
- **Before:** Started on 3 physical servers
- **After:** Moved entirely to AWS
- **Result:** Can handle millions of listings and bookings globally
- **Key Insight:** Pay-per-use model perfect for seasonal travel business (more usage in summer)

#### Case Study 3: An Indian Bank (HDFC-like scenario)
- **Approach:** Hybrid cloud
- **Private:** Core banking data (customer accounts, transactions) — regulatory requirement
- **Public Cloud:** Mobile app, analytics, AI-based fraud detection
- **Key Insight:** Hybrid lets them innovate while staying compliant

---

### Group Activity: Cloud Migration Decision (15 minutes)

**Scenario:** You are consulting for these 3 companies. Recommend: Public Cloud, Private Cloud, Hybrid Cloud, or Stay On-Premise.

| Company | Situation | Your Recommendation? |
|---------|-----------|---------------------|
| **TechStartup Inc.** | 10 employees, building a SaaS product, zero IT team, rapid growth expected | ? |
| **National Defense Agency** | Top-secret data, maximum security needed, cost is not a concern | ? |
| **MegaRetail Corp.** | 500 stores, customer data + loyalty program, seasonal spikes during festivals | ? |

<details>
<summary>Click to see suggested answers</summary>

1. **TechStartup Inc.** → **Public Cloud** (no CapEx, scale with growth, no IT team needed)
2. **National Defense Agency** → **Private Cloud** or On-Premise (security and sovereignty requirements)
3. **MegaRetail Corp.** → **Hybrid Cloud** (customer data in private, loyalty app in public for scalability during festivals)

</details>

---

## Session 4: SAP Ecosystem & Product Landscape (13:30 - 14:30)

### What is SAP?

**SAP** (Systems, Applications, and Products in Data Processing) is a German software company founded in 1972. It is the **world's largest enterprise software company**.

- Headquarters: Walldorf, Germany
- Employees: 100,000+
- Customers: 400,000+ in 180+ countries
- Revenue: €30+ billion/year

### Why SAP Matters

- **77% of the world's transaction revenue** touches an SAP system
- Used by 99 of the top 100 companies in the world
- Runs critical business processes: finance, supply chain, HR, procurement, manufacturing

---

### SAP's History — The Journey to Cloud

```
Timeline:
1972 ──── SAP R/1 (Mainframe ERP)
  |
1979 ──── SAP R/2 (Improved mainframe)
  |
1992 ──── SAP R/3 (Client-Server architecture) ← Revolution!
  |
2004 ──── SAP ECC (ERP Central Component)
  |
2010 ──── SAP HANA database launched (in-memory computing)
  |
2015 ──── SAP S/4HANA (Next-gen ERP on HANA)
  |
2018 ──── SAP Cloud Platform (now SAP BTP)
  |
2021 ──── SAP BTP (Business Technology Platform) ← Where we are!
  |
2024 ──── SAP Joule (AI Copilot) integrated across products
```

---

### What is ERP?

**ERP (Enterprise Resource Planning)** = Software that manages ALL core business processes in ONE system.

```
                    +------------------+
                    |                  |
                    |   ERP System     |
                    |   (SAP S/4HANA)  |
                    |                  |
                    +------------------+
                   /   |    |    |    \
                  /    |    |    |     \
         +------+ +----+ +-----+ +------+ +-------+
         |Finance| | HR | |Sales| |Supply| |Manufac|
         |      | |    | |     | |Chain | |turing |
         +------+ +----+ +-----+ +------+ +-------+
              |       |      |       |         |
         Invoices  Payroll  Orders  Purchase  Production
         Budgets  Leave    CRM     Inventory  Planning
         Tax      Hiring   Pricing Warehouse  Quality
```

**Without ERP:** Each department has its own system → data silos, manual reconciliation, errors
**With ERP:** Single source of truth → real-time visibility, automation, consistency

---

### SAP's Product Portfolio (The Big Picture)

#### Core ERP

| Product | What It Does | Simple Analogy |
|---|---|---|
| **SAP S/4HANA** | Core business operations (finance, supply chain, manufacturing, procurement) | The backbone/spine of a company |
| **SAP S/4HANA Cloud** | Same as above, but fully in the cloud | Backbone-as-a-service |

#### Line of Business Solutions

| Product | Department | What It Does |
|---|---|---|
| **SAP SuccessFactors** | HR | Hiring, payroll, performance reviews, learning |
| **SAP Ariba** | Procurement | Finding suppliers, purchase orders, invoices |
| **SAP Concur** | Finance | Employee expense management and travel |
| **SAP Customer Experience (CX)** | Sales/Marketing | CRM, e-commerce, marketing campaigns |
| **SAP Fieldglass** | HR | Managing external workforce (contractors) |
| **SAP Integrated Business Planning** | Supply Chain | Demand forecasting, supply planning |

#### Technology & Platform

| Product | What It Does |
|---|---|
| **SAP BTP** | Platform to build, integrate, extend applications |
| **SAP HANA Cloud** | In-memory database (super-fast data processing) |
| **SAP Integration Suite** | Connect SAP and non-SAP systems |
| **SAP Analytics Cloud** | Business intelligence and reporting |
| **SAP AI Core** | AI/ML model training and serving |
| **SAP Build** | Low-code/no-code application development |
| **SAP Joule** | AI copilot across SAP products |

---

### The "Intelligent Enterprise" Framework

SAP's vision is the **Intelligent Enterprise** — where all systems are connected and intelligent:

```
+------------------------------------------------------------------+
|                     INTELLIGENT ENTERPRISE                        |
|                                                                  |
|  +------------------------------------------------------------+  |
|  |                  SAP BTP (Foundation)                       |  |
|  |  Database | Analytics | AI | Integration | Development     |  |
|  +------------------------------------------------------------+  |
|           |              |              |                         |
|  +----------------+ +----------------+ +------------------+      |
|  | Intelligent    | | Intelligent    | | Intelligent      |      |
|  | Suite          | | Spend Mgmt     | | People Mgmt      |      |
|  | (S/4HANA +    | | (Ariba +       | | (SuccessFactors  |      |
|  |  CX)          | |  Concur +      | |  + Fieldglass)   |      |
|  |               | |  Fieldglass)   | |                  |      |
|  +----------------+ +----------------+ +------------------+      |
|                                                                  |
|  Connected by: SAP Integration Suite + SAP BTP                   |
+------------------------------------------------------------------+
```

---

### How SAP Products Work Together — A Story

**Scenario:** A customer orders a laptop from an e-commerce store running on SAP:

```
Step 1: Customer places order
         → SAP Customer Experience (CX) captures the order

Step 2: Order is created in the ERP
         → SAP S/4HANA creates a Sales Order

Step 3: Check inventory
         → SAP S/4HANA checks warehouse stock

Step 4: If not in stock, create purchase order
         → SAP Ariba sends request to supplier

Step 5: Supplier ships to warehouse
         → SAP S/4HANA tracks incoming delivery

Step 6: Ship to customer
         → SAP S/4HANA creates outbound delivery

Step 7: Create invoice
         → SAP S/4HANA generates customer invoice

Step 8: Customer pays
         → SAP S/4HANA records payment in Finance

Throughout: SAP Analytics Cloud shows real-time dashboards
            SAP BTP hosts custom extensions (loyalty program, etc.)
```

---

### SAP S/4HANA — The Heart of SAP

**S/4HANA** stands for:
- **S** = Simple (simplified data model)
- **4** = for
- **HANA** = High-performance Analytics Appliance (the database)

#### Key Modules in S/4HANA

| Module Code | Full Name | What It Does |
|---|---|---|
| **FI** | Financial Accounting | General ledger, accounts payable/receivable |
| **CO** | Controlling | Cost centers, profit analysis |
| **MM** | Materials Management | Procurement, inventory |
| **SD** | Sales & Distribution | Sales orders, delivery, billing |
| **PP** | Production Planning | Manufacturing, BOM, routing |
| **PM** | Plant Maintenance | Equipment maintenance, work orders |
| **QM** | Quality Management | Quality inspections, certificates |
| **HCM** | Human Capital Management | Payroll, time management |

---

### Discussion Activity (10 minutes)

Think about a company you know (a store, a bank, a hospital). What SAP modules would they need?

**Example:** A hospital
- FI/CO: Managing finances and budgets
- MM: Procuring medicines and equipment
- PM: Maintaining medical equipment
- HCM: Managing doctors, nurses, staff
- Would NOT need: SD (they don't "sell" products) or PP (they don't manufacture)

**Your Turn:** Pick a company and list which SAP modules they'd use!

---

## Session 5: SAP BTP Deep-Dive (14:30 - 15:30)

### SAP Business Technology Platform (BTP)

**SAP BTP** is SAP's cloud platform that allows companies to:
- **Build** custom applications (this is what we'll do with CAP!)
- **Integrate** different systems together
- **Extend** existing SAP products with new features
- **Analyze** data for better decisions
- **Plan** with AI-powered insights

### Why does SAP BTP exist?

**Problem:** SAP S/4HANA is a standard product. Every company has unique needs. But you shouldn't modify the standard product (makes upgrades impossible).

**Solution:** SAP BTP! Build extensions that connect to S/4HANA WITHOUT modifying the core.

```
                     WRONG WAY:
    +----------------------------------+
    |  SAP S/4HANA                     |
    |  + Custom Code mixed in          |  ← Upgrade nightmare!
    |  + Modifications everywhere      |
    +----------------------------------+

                     RIGHT WAY (BTP):
    +----------------------------------+
    |  SAP S/4HANA (clean, standard)   |  ← Easy to upgrade!
    +----------------------------------+
              |  APIs  |
    +----------------------------------+
    |  SAP BTP                         |
    |  + Custom Extensions             |  ← Our playground!
    |  + Integrations                  |
    |  + Custom Apps                   |
    +----------------------------------+
```

---

### SAP BTP Service Categories

| Service Category | What It Offers | Example Use Case |
|---|---|---|
| **Database & Data Management** | SAP HANA Cloud (super-fast database) | Store and process millions of transactions |
| **Analytics** | SAP Analytics Cloud | Create dashboards to visualize sales data |
| **Integration** | SAP Integration Suite | Connect SAP with non-SAP systems (like Salesforce) |
| **AI & Machine Learning** | SAP AI Core, SAP Joule | Automate invoice processing using AI |
| **Application Development** | SAP Build, CAP Framework | Create custom business apps |
| **Automation** | SAP Build Process Automation | Automate approval workflows |
| **Security** | Identity Authentication, XSUAA | Manage user access and authentication |

---

### SAP BTP Architecture

```
+---------------------------------------------------------------+
|                        SAP BTP                                 |
|                                                               |
|  +-------------------+  +-------------------+                 |
|  | RUNTIME           |  | SERVICES          |                 |
|  | ENVIRONMENTS      |  |                   |                 |
|  |                   |  | • HANA Cloud      |                 |
|  | • Cloud Foundry   |  | • XSUAA           |                 |
|  | • Kyma (K8s)     |  | • Destination     |                 |
|  | • ABAP Environment|  | • Connectivity    |                 |
|  |                   |  | • HTML5 Repo      |                 |
|  +-------------------+  +-------------------+                 |
|                                                               |
|  +-------------------+  +-------------------+                 |
|  | DEVELOPMENT TOOLS |  | INTEGRATION       |                 |
|  |                   |  |                   |                 |
|  | • BAS (IDE)       |  | • Integration     |                 |
|  | • SAP Build       |  |   Suite           |                 |
|  | • Joule Studio    |  | • API Management  |                 |
|  | • CAP Framework   |  | • Event Mesh      |                 |
|  +-------------------+  +-------------------+                 |
|                                                               |
+---------------------------------------------------------------+
        |                    |                    |
   +---------+          +---------+         +---------+
   |   AWS   |          |  Azure  |         |   GCP   |
   +---------+          +---------+         +---------+
```

---

### SAP BTP Account Model

Understanding how SAP BTP is organized:

```
+------------------------------------------+
| GLOBAL ACCOUNT                           |
| (Your company's main BTP account)        |
|                                          |
|  +-----------------+ +-----------------+ |
|  | SUBACCOUNT 1    | | SUBACCOUNT 2    | |
|  | (Development)   | | (Production)    | |
|  |                 | |                 | |
|  | Region: EU10    | | Region: US10    | |
|  |                 | |                 | |
|  | +-------------+ | | +-------------+ | |
|  | | CF Org      | | | | CF Org      | | |
|  | | +--------+  | | | | +--------+  | | |
|  | | | Space: |  | | | | | Space: |  | | |
|  | | | dev    |  | | | | | prod   |  | | |
|  | | +--------+  | | | | +--------+  | | |
|  | +-------------+ | | +-------------+ | |
|  +-----------------+ +-----------------+ |
+------------------------------------------+
```

| Level | Purpose | Example |
|-------|---------|---------|
| **Global Account** | Top-level container, manages entitlements and billing | "Acme Corp BTP Account" |
| **Subaccount** | Isolated environment, specific region | "Acme-Dev (EU10)", "Acme-Prod (US10)" |
| **Organization** | Cloud Foundry concept, groups spaces | Usually one per subaccount |
| **Space** | Where apps actually run | "development", "testing", "production" |

---

### Runtime Environments in BTP

| Runtime | What It Is | Best For | What We'll Use |
|---------|-----------|----------|----------------|
| **Cloud Foundry** | PaaS runtime for deploying apps | CAP Node.js/Java apps | ✅ This one! |
| **Kyma** | Kubernetes-based runtime | Containerized microservices | Not in this course |
| **ABAP Environment** | Cloud ABAP development | ABAP developers extending S/4HANA | Not in this course |

---

### SAP BTP Commercial Models

| Model | How It Works | Best For |
|-------|-------------|----------|
| **CPEA** (Cloud Platform Enterprise Agreement) | Pre-purchase credits, use across services | Large enterprises with diverse needs |
| **Pay-As-You-Go** | Pay for what you use monthly | Exploration, small projects |
| **Free Tier** | Always-free limited services | Learning and prototyping |
| **Trial** | 90-day free account with limited resources | Students and evaluation |

**For this course:** We'll use the **Trial** account (free for 90 days!)

---

### What is SAP CAP? (Preview for the course)

**CAP** = Cloud Application Programming Model

It's SAP's recommended framework for building business applications on BTP:

```
What CAP Gives You:
+--------------------------------------------------+
|  CDS (Core Data Services)                        |
|  → Define your data model declaratively          |
+--------------------------------------------------+
|  Service Framework                               |
|  → Expose data as OData/REST APIs automatically  |
+--------------------------------------------------+
|  Built-in Features                               |
|  → Authentication, authorization, draft,         |
|    localization, multitenancy... all free!        |
+--------------------------------------------------+
|  Node.js or Java Runtime                         |
|  → We'll use Node.js                            |
+--------------------------------------------------+
```

**Why CAP?**
- Write **10x less code** compared to building from scratch
- Focus on **business logic**, not boilerplate
- Automatically get OData APIs, database handling, security
- SAP's recommended approach for ALL new development on BTP

---

### SAP BTP in SAP's Strategy

SAP BTP is the **foundation** of SAP's clean core strategy:

| Concept | Meaning |
|---------|---------|
| **Clean Core** | Keep S/4HANA standard, no custom modifications |
| **Side-by-Side Extension** | Build custom apps on BTP that connect to S/4HANA via APIs |
| **Keep the Core Clean** | All customizations live on BTP, not inside S/4HANA |
| **Easy Upgrades** | Standard S/4HANA can be upgraded without breaking customizations |

This is **exactly** what we're training you to do: Build side-by-side extensions on SAP BTP using CAP!

---

## Session 6: Hands-on Exploration & Activities (15:45 - 16:45)

### Activity 1: Explore SAP Discovery Center (15 minutes)

1. Go to: [SAP Discovery Center](https://discovery-center.cloud.sap/)
2. Browse through the available **missions** (guided learning paths)
3. Find and explore:
   - One mission related to **SAP BTP**
   - One mission related to **SAP CAP** or **Cloud Application Programming**
4. For each mission, note down:
   - Mission name
   - What it teaches
   - Estimated time to complete
   - Which BTP services it uses

---

### Activity 2: Browse SAP BTP Documentation (15 minutes)

1. Visit: [SAP BTP Documentation](https://help.sap.com/docs/btp)
2. Explore the **"Get Started"** section
3. Find and read about:
   - SAP BTP Cockpit overview
   - Cloud Foundry Environment documentation
   - SAP CAP documentation at [https://cap.cloud.sap/](https://cap.cloud.sap/)
4. Write down:
   - 3 services that interest you and why
   - 1 thing that surprised you about BTP

---

### Activity 3: Explore SAP Community (10 minutes)

1. Visit: [SAP Community](https://community.sap.com/)
2. Browse:
   - Blog posts about BTP
   - Questions about CAP development
   - Upcoming events/webinars
3. Note down one interesting blog post or discussion you found

---

### Activity 4: Cloud Provider Exploration (10 minutes)

Visit ONE of these and browse their service catalog:
- [AWS Services](https://aws.amazon.com/products/)
- [Azure Services](https://azure.microsoft.com/en-in/products/)
- [Google Cloud Products](https://cloud.google.com/products)

Try to find:
1. A database service (compare with SAP HANA Cloud)
2. An AI/ML service (compare with SAP AI Core)
3. A Platform-as-a-Service offering (compare with SAP BTP)

---

### Activity 5: Map the SAP Landscape (Group Activity - 10 minutes)

In groups of 3-4, create a diagram (on paper or whiteboard) showing:
1. SAP S/4HANA at the center
2. All the SAP products we discussed today around it
3. SAP BTP connecting them all
4. Mark which are Cloud vs On-Premise vs Both

Share with the class and explain your diagram.

---

## Assessment: MCQ (15 Questions)

Test your understanding! Choose the correct answer for each question.

### Cloud Computing Fundamentals

**Q1.** What is Cloud Computing?
- a) A type of weather forecasting technology
- b) Delivering computing services over the internet
- c) A new programming language
- d) A type of hardware

<details><summary>Answer</summary>b) Delivering computing services over the internet</details>

---

**Q2.** Which cloud service model provides ready-to-use software over the internet?
- a) IaaS
- b) PaaS
- c) SaaS
- d) DaaS

<details><summary>Answer</summary>c) SaaS - Software as a Service</details>

---

**Q3.** Gmail, Zoom, and Microsoft 365 are examples of:
- a) IaaS
- b) PaaS
- c) SaaS
- d) On-Premise software

<details><summary>Answer</summary>c) SaaS</details>

---

**Q4.** Which is NOT a benefit of cloud computing?
- a) Pay-as-you-go pricing
- b) Need to buy physical servers
- c) Easy scaling
- d) Access from anywhere

<details><summary>Answer</summary>b) Need to buy physical servers - this is actually a characteristic of On-Premise, not Cloud</details>

---

**Q5.** In IaaS, who manages the operating system?
- a) The cloud provider only
- b) The customer
- c) No one
- d) The internet service provider

<details><summary>Answer</summary>b) The customer - In IaaS, you rent the hardware but manage the OS and software yourself</details>

---

### On-Premise vs Cloud

**Q6.** A company needs to launch an app within 2 days. Which deployment is better?
- a) On-Premise
- b) Cloud
- c) Both are equally fast
- d) Neither can do it

<details><summary>Answer</summary>b) Cloud - it can be set up in minutes/hours vs weeks for on-premise</details>

---

**Q7.** Which pricing model does Cloud typically follow?
- a) One-time purchase
- b) Pay-as-you-go (subscription)
- c) Free forever
- d) Annual hardware purchase

<details><summary>Answer</summary>b) Pay-as-you-go (subscription)</details>

---

**Q8.** A hospital needs to keep patient data in their own data center but wants to use cloud for their mobile app. Which model should they use?
- a) Public Cloud
- b) Private Cloud
- c) Hybrid Cloud
- d) Community Cloud

<details><summary>Answer</summary>c) Hybrid Cloud - combining private (for sensitive data) with public (for the app)</details>

---

### SAP Ecosystem

**Q9.** What does SAP stand for?
- a) Software Application Programming
- b) Systems, Applications, and Products
- c) Secure Access Platform
- d) Service and Performance

<details><summary>Answer</summary>b) Systems, Applications, and Products</details>

---

**Q10.** Which SAP product is considered the core ERP system?
- a) SAP Ariba
- b) SAP SuccessFactors
- c) SAP S/4HANA
- d) SAP Concur

<details><summary>Answer</summary>c) SAP S/4HANA</details>

---

**Q11.** SAP SuccessFactors is used for:
- a) Financial accounting
- b) Human Resource Management
- c) Supply chain
- d) Customer sales

<details><summary>Answer</summary>b) Human Resource Management</details>

---

### SAP BTP

**Q12.** What is SAP BTP?
- a) A programming language by SAP
- b) SAP's cloud platform for building, integrating, and extending applications
- c) A type of database
- d) SAP's email service

<details><summary>Answer</summary>b) SAP's cloud platform for building, integrating, and extending applications</details>

---

**Q13.** Which SAP BTP service would you use to connect SAP with a non-SAP system?
- a) SAP HANA Cloud
- b) SAP Analytics Cloud
- c) SAP Integration Suite
- d) SAP AI Core

<details><summary>Answer</summary>c) SAP Integration Suite</details>

---

**Q14.** What is SAP's "Clean Core" strategy?
- a) Deleting all old SAP products
- b) Keeping S/4HANA standard and building extensions on BTP
- c) Cleaning the server hardware regularly
- d) Using only free software

<details><summary>Answer</summary>b) Keeping S/4HANA standard and building extensions on BTP - customizations should live on BTP, not inside S/4HANA</details>

---

**Q15.** What runtime environment will we use in this course for deploying CAP applications?
- a) Kyma
- b) ABAP Environment
- c) Cloud Foundry
- d) Docker

<details><summary>Answer</summary>c) Cloud Foundry</details>

---

## Bonus MCQs (5 Additional Questions)

**Q16.** Which NIST characteristic of cloud computing means you can get resources without calling anyone?
- a) Broad network access
- b) On-demand self-service
- c) Resource pooling
- d) Measured service

<details><summary>Answer</summary>b) On-demand self-service</details>

---

**Q17.** Which cloud deployment model is shared among organizations with common requirements?
- a) Public Cloud
- b) Private Cloud
- c) Hybrid Cloud
- d) Community Cloud

<details><summary>Answer</summary>d) Community Cloud</details>

---

**Q18.** SAP BTP runs on which cloud providers?
- a) Only AWS
- b) Only Azure
- c) AWS, Azure, and Google Cloud (multi-cloud)
- d) SAP's own data centers only

<details><summary>Answer</summary>c) AWS, Azure, and Google Cloud (multi-cloud)</details>

---

**Q19.** What does CAP stand for in SAP development?
- a) Cloud Access Protocol
- b) Cloud Application Programming
- c) Core Application Platform
- d) Custom API Programming

<details><summary>Answer</summary>b) Cloud Application Programming (Model)</details>

---

**Q20.** In SAP BTP's account hierarchy, which level is where applications actually run?
- a) Global Account
- b) Subaccount
- c) Organization
- d) Space

<details><summary>Answer</summary>d) Space - this is where apps are deployed and run in Cloud Foundry</details>

---

## Assignment

### Write a 1-Page Comparison: On-Premise vs Cloud for a Retail Company

**Scenario:**
"FreshMart" is a retail company with 50 stores across India. They currently run all their systems on-premise (their own servers in their head office). They are considering moving to the cloud.

**Current State:**
- 5 physical servers in Mumbai office (purchased 4 years ago)
- 3-person IT team managing infrastructure
- Customer data, inventory, and billing systems all on-premise
- They want to expand to 200 stores in the next 2 years
- Festive season (Oct-Dec) sees 3x normal traffic
- Currently no disaster recovery plan

**Your Task:**
Write a 1-page document covering:

1. **Current Challenges** (at least 3 points on problems with on-premise for FreshMart)
2. **How Cloud Solves These** (match each challenge with a cloud benefit)
3. **Recommended Cloud Deployment Model** (Public/Private/Hybrid — and justify why)
4. **Which SAP Products Would Help?** (pick at least 2 SAP products suitable for FreshMart)
5. **One Risk** of moving to cloud that FreshMart should consider
6. **Cost Comparison** (rough estimate — use the numbers from Session 3 as a guide)

**Format:** Use bullet points and short sentences. Keep it to 1 page.

**Submission:** Save as a Word/PDF document named: `YourName_Day1_Assignment`

---

## Key Takeaways

| # | Topic | One-Line Summary |
|---|---|---|
| 1 | Cloud Computing | Renting computing resources over the internet instead of owning them |
| 2 | IaaS vs PaaS vs SaaS | Raw infrastructure vs Platform vs Ready software (less control = less responsibility) |
| 3 | Deployment Models | Public (shared), Private (dedicated), Hybrid (best of both) |
| 4 | On-Premise vs Cloud | Own everything vs Rent everything — cloud wins for most modern use cases |
| 5 | SAP | World's largest enterprise software company, runs 77% of global business transactions |
| 6 | SAP Ecosystem | S/4HANA (core) + SuccessFactors (HR) + Ariba (procurement) + more |
| 7 | SAP BTP | The platform that connects, extends, and innovates across SAP products |
| 8 | Clean Core | Keep S/4HANA standard, build extensions on BTP |
| 9 | CAP | SAP's recommended framework for building apps on BTP (what this course teaches!) |
| 10 | Cloud Foundry | The runtime environment where our CAP apps will be deployed |

---

## Glossary of Terms Learned Today

| Term | Definition |
|------|-----------|
| **Cloud Computing** | On-demand delivery of IT resources via the internet with pay-as-you-go pricing |
| **IaaS** | Infrastructure as a Service — rent raw computing resources |
| **PaaS** | Platform as a Service — rent a platform to build and deploy apps |
| **SaaS** | Software as a Service — use ready-made software via internet |
| **On-Premise** | IT infrastructure owned and operated by the company in their own facility |
| **Public Cloud** | Cloud infrastructure shared among multiple organizations |
| **Private Cloud** | Cloud infrastructure dedicated to a single organization |
| **Hybrid Cloud** | Combination of public and private cloud |
| **ERP** | Enterprise Resource Planning — software managing all core business processes |
| **SAP BTP** | SAP Business Technology Platform — SAP's cloud platform for building/extending apps |
| **Cloud Foundry** | An open-source PaaS runtime environment (used in SAP BTP) |
| **CAP** | Cloud Application Programming Model — SAP's framework for building business apps |
| **Clean Core** | Strategy of keeping S/4HANA modification-free by building extensions on BTP |
| **Multi-tenancy** | Multiple customers sharing the same infrastructure while remaining isolated |
| **Scalability** | Ability to increase or decrease resources based on demand |
| **SLA** | Service Level Agreement — provider's guarantee of uptime/performance |

---

## Preparation for Day 2

Tomorrow we will cover **SAP BTP Architecture & Cloud Foundry**. To prepare:
- Make sure you have a stable internet connection
- Create an SAP account at [SAP.com](https://www.sap.com/) if you don't have one
- Think about questions from today — we'll start with a 15-minute Q&A
- Optional reading: [What is Cloud Foundry?](https://www.cloudfoundry.org/what-is-cloud-foundry/)

---

## Additional Resources

| Resource | Link | Purpose |
|----------|------|---------|
| SAP Discovery Center | https://discovery-center.cloud.sap/ | Guided missions and learning |
| SAP BTP Documentation | https://help.sap.com/docs/btp | Official documentation |
| SAP Community | https://community.sap.com/ | Q&A, blogs, events |
| CAP Documentation | https://cap.cloud.sap/ | CAP framework docs (our main tool!) |
| SAP Learning | https://learning.sap.com/ | Free learning journeys |
| Cloud Computing Basics (AWS) | https://aws.amazon.com/what-is-cloud-computing/ | Alternative explanation of cloud concepts |

---

*End of Day 1*
