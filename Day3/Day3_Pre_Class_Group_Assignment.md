# Day 3: Pre-Class Group Assignment

## "BTP Account Architect" — Design Challenge

### Duration: 1 Hour (Before Class Starts)
### Team Size: 4-5 members per team

---

## The Game: You Are the SAP BTP Admin!

Your team has been promoted to **SAP BTP Administrators** at a fictional company. The CEO is demanding that multiple departments get access to SAP BTP — and they ALL want it TODAY. Your job is to design the perfect account structure, assign the right services, and keep everyone happy (and secure!).

But there's a twist: **each department has conflicting requirements**, and you have a **limited budget** to work with!

---

## The Company: "SparkWheels Motors"

SparkWheels is an electric vehicle manufacturer with:
- 🏭 **Manufacturing** — 2 factories (Pune, Chennai)
- 🛒 **Sales** — Online store + 25 showrooms across India
- 🔧 **After-Sales Service** — Mobile app for customers to book service appointments
- 💰 **Finance** — Handles billing, vendor payments, and compliance reporting
- 👥 **HR** — 3,000 employees, hiring 500 more this year
- 🧪 **R&D** — Building a self-driving AI prototype (top secret!)

**Current IT Setup:**
- Running SAP S/4HANA on-premise in Pune
- No cloud presence yet
- 5-person IT team (overworked!)

---

## Your Mission

### Part A: The Budget Game (20 minutes)

SparkWheels has a budget of **100 BTP Credits** per month. Each service/feature costs credits. Your team must decide how to spend them wisely!

#### Price List

| Item | Monthly Cost (Credits) | What It Gives You |
|------|----------------------|-------------------|
| Subaccount | 5 per subaccount | An isolated environment in a region |
| Cloud Foundry Space | 2 per space | A workspace for apps |
| CF Runtime (per 1 GB) | 8 per GB | Memory to run applications |
| SAP HANA Cloud (shared) | 15 | 30 GB database |
| SAP HANA Cloud (dedicated) | 30 | 256 GB database, better performance |
| XSUAA (authentication) | 3 | Security for apps |
| SAP Build Work Zone | 10 | Launchpad for all apps |
| SAP Integration Suite | 12 | Connect to S/4HANA |
| SAP Business Application Studio | 5 | Cloud IDE for developers |
| Destination Service | 2 | Connect to external systems |
| SAP Analytics Cloud | 10 | Dashboards and reporting |

#### Your Budget Sheet

Fill in what you'd buy:

| Item | Quantity | Cost | Justification |
|------|----------|------|---------------|
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
| **TOTAL** | | **/100** | |

**Rules:**
- You CANNOT exceed 100 credits
- Every department must have SOMETHING (no one gets left out!)
- The R&D team's work must be isolated from everyone else (security!)
- Manufacturing needs real-time data from S/4HANA

---

### Part B: Account Structure Design (20 minutes)

Design the complete SAP BTP account hierarchy for SparkWheels.

#### Template to Fill

```
🏢 Global Account: SparkWheels-BTP
│
├── 📁 Subaccount 1: _____________ (Region: _______)
│   └── 🏗️ Org: _____________
│       ├── 💻 Space: _____________ [Purpose: _____________]
│       └── 💻 Space: _____________ [Purpose: _____________]
│
├── 📁 Subaccount 2: _____________ (Region: _______)
│   └── 🏗️ Org: _____________
│       ├── 💻 Space: _____________ [Purpose: _____________]
│       └── 💻 Space: _____________ [Purpose: _____________]
│
├── 📁 Subaccount 3: _____________ (Region: _______)
│   └── 🏗️ Org: _____________
│       └── 💻 Space: _____________ [Purpose: _____________]
│
└── (Add more if needed within budget!)
```

#### Decision Questions

Answer these for your design:

| # | Question | Your Answer |
|---|----------|-------------|
| 1 | Why did you choose those specific regions? | |
| 2 | Why did you separate the R&D subaccount? | |
| 3 | How does your design handle the Sales team's seasonal traffic spikes (Diwali, New Year)? | |
| 4 | Which department shares a subaccount and why? | |
| 5 | If the budget was cut to 70 credits, what would you remove first? | |

---

### Part C: The Conflict Resolution Round (15 minutes)

Here's where it gets interesting! Each department has sent you an angry email. Your team must resolve the conflicts.

#### 📧 Email 1: From Manufacturing Head

> "We need DEDICATED HANA Cloud (30 credits) because our production data is critical. If the database is shared and slows down, the assembly line stops. That costs us ₹50 lakhs per HOUR!"

#### 📧 Email 2: From R&D Director

> "Our self-driving AI project is CONFIDENTIAL. We need our own subaccount in a region where NO OTHER department can access our data. Also, we need 3 GB of CF Runtime for our ML models (24 credits for runtime alone!)."

#### 📧 Email 3: From Sales VP

> "Diwali is coming in 5 months. Last year our website crashed with 10x traffic. We need enough CF Runtime to handle peak load — at least 2 GB (16 credits). And we need SAP Analytics Cloud (10 credits) to track real-time sales!"

#### 📧 Email 4: From Finance Controller

> "Compliance audit is next month. We MUST have SAP Analytics Cloud for the audit report (10 credits). Also, we need Integration Suite (12 credits) to pull data from our on-premise S/4HANA for reporting."

#### 📧 Email 5: From HR Manager

> "We're hiring 500 people! We need SAP Build Work Zone (10 credits) for the employee self-service portal. Urgently!"

---

#### The Math Problem

Add up ALL their demands:
- Manufacturing: 30 (dedicated HANA)
- R&D: 5 (subaccount) + 24 (3 GB runtime) + 3 (XSUAA) = 32
- Sales: 16 (2 GB runtime) + 10 (Analytics) = 26
- Finance: 10 (Analytics) + 12 (Integration) = 22
- HR: 10 (Work Zone)
- **Total demanded: 120 credits** 😱

**BUT YOUR BUDGET IS ONLY 100!**

#### Your Resolution

For each conflict, write your decision and how you'd explain it to the department:

| Department | What They Want | What They Get | How You Justify It |
|-----------|---------------|---------------|-------------------|
| Manufacturing | Dedicated HANA (30) | ? | ? |
| R&D | Own subaccount + 3 GB (32) | ? | ? |
| Sales | 2 GB + Analytics (26) | ? | ? |
| Finance | Analytics + Integration (22) | ? | ? |
| HR | Work Zone (10) | ? | ? |
| **Total** | **120** | **≤ 100** | |

**Tips for resolving:**
- Can any departments SHARE a service? (Analytics Cloud can serve multiple teams!)
- Can you reduce some allocations? (Does R&D really need 3 GB?)
- Are there cheaper alternatives? (Shared HANA vs Dedicated?)
- Can anything be phased? (HR portal in Month 2, not Month 1?)

---

### Part D: Present Your Case (5 minutes prep)

Prepare a **3-minute presentation** covering:

1. **Your account structure** — draw it on whiteboard
2. **Budget breakdown** — how you spent 100 credits
3. **The conflict** — how you resolved the impossible math
4. **Your boldest decision** — which department did you push back on, and why?

---

## Judging Criteria

Teams present and are scored by the trainer + peers:

| Criteria | Max Points | What Judges Look For |
|----------|-----------|---------------------|
| **Budget Accuracy** | 5 | Adds up correctly? Within 100 credits? |
| **Architecture Design** | 5 | Logical structure? Good region choices? |
| **Conflict Resolution** | 5 | Creative solutions? Fair to all departments? |
| **Security** | 3 | R&D isolated? Proper separation? |
| **Presentation** | 2 | Clear explanation? Team participated? |
| **Total** | **20** | |

### Bonus Points (+3 each)

- 🏆 **Best Negotiator:** Team with the most creative conflict resolution
- 🏆 **Most Realistic:** Team whose design most closely matches real-world practice
- 🏆 **Under Budget:** Team that solves everything with the FEWEST credits

---

## Fun Twist: "What If?" Cards

After presentations, the trainer draws one "What If?" card for each team. You have 60 seconds to respond!

| Card | Scenario |
|------|----------|
| 🃏 Card 1 | "The CEO just acquired a company in Germany. You need a new subaccount in EU10. Adjust your budget!" |
| 🃏 Card 2 | "A data breach happened! The R&D AI data leaked because someone had access they shouldn't. What went wrong in your design?" |
| 🃏 Card 3 | "Diwali sale broke ALL records — you need 5 GB of CF runtime RIGHT NOW. Where do you steal credits from?" |
| 🃏 Card 4 | "The Finance team's compliance audit found that production data and test data were in the same subaccount. Is this a problem? How would you fix it?" |
| 🃏 Card 5 | "SAP just announced a 50% discount on Integration Suite. How does this change your budget allocation?" |

---

## What You'll Learn From This Exercise

| Concept | How This Assignment Teaches It |
|---------|-------------------------------|
| Account Hierarchy | You design Global Account → Subaccount → Org → Space |
| Entitlements & Quotas | Budget = entitlements. Credits = quotas. You assign them! |
| Service Plans | You choose between shared vs dedicated HANA |
| Isolation & Security | R&D needs separate subaccount for confidentiality |
| Regions | You pick regions based on factory/user locations |
| Real-world tradeoffs | Budget forces you to make hard choices (just like real BTP admins!) |

---

*Good luck, BTP Admins! May the best architecture win!* 🏆
