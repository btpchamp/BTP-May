# Day 2: Pre-Class Group Assignment

## "Build Your Own Cloud Company" — Group Simulation Exercise

### Duration: 1 Hour (Before Class Starts)
### Team Size: 4-5 members per team

---

## The Scenario

Your team has just been hired as **Cloud Architects** at a fictional consulting company. A client — **"QuickMed India"** — has approached you with a problem.

### About QuickMed India

QuickMed is a healthcare startup with:
- 30 clinics across 5 Indian cities (Mumbai, Delhi, Bangalore, Chennai, Hyderabad)
- 200 doctors, 500 nurses, 1000+ support staff
- Currently running everything on 2 physical servers in their Mumbai office
- Their patient appointment app crashes every Monday morning (peak booking time)
- They want to expand to 100 clinics in the next year
- They handle sensitive patient medical records (must comply with Indian data protection laws)
- Budget: Limited — they're a startup

---

## Your Mission (60 minutes)

### Part A: Cloud Architecture Design (25 minutes)

Design a cloud setup for QuickMed. Your team must decide and document:

#### 1. Cloud Strategy Decisions

| Decision | Your Answer | Why? |
|----------|-------------|------|
| Deployment Model (Public/Private/Hybrid)? | | |
| Primary Cloud Provider (AWS/Azure/GCP)? | | |
| Region for primary data center? | | |
| Need for multiple regions? Which ones? | | |
| What data goes where? (patient records vs app) | | |

#### 2. Draw the Architecture

On paper/whiteboard, draw QuickMed's cloud architecture showing:

```
Draw something like this (but specific to QuickMed):

+--------------------------------------------------+
|  [Your chosen platform]                          |
|                                                  |
|  +----------+  +----------+  +----------+       |
|  | Service 1|  | Service 2|  | Service 3|       |
|  | (what?)  |  | (what?)  |  | (what?)  |       |
|  +----------+  +----------+  +----------+       |
|                                                  |
|  +------------------------------------------+   |
|  | Database Layer (what type? where?)        |   |
|  +------------------------------------------+   |
|                                                  |
+--------------------------------------------------+
        |               |               |
   [City 1]        [City 2]        [City 3]
   Users            Users           Users
```

Include:
- Where the appointment booking app runs
- Where patient data is stored
- How doctors in different cities access the system
- What happens during Monday morning peak load
- How you handle the app crashing problem

#### 3. The Monday Morning Problem

QuickMed's app crashes every Monday at 9 AM when 500+ patients try to book appointments simultaneously.

Explain (in bullet points):
- **Why** does this happen with their current 2-server setup?
- **How** would your cloud design solve this?
- **What specific cloud feature** would prevent future crashes?

---

### Part B: SAP BTP Mapping (20 minutes)

Now imagine QuickMed decides to use **SAP BTP** as their platform. Map their needs to SAP BTP:

#### 1. Account Structure

Design QuickMed's SAP BTP account hierarchy:

| Level | Name | Purpose |
|-------|------|---------|
| Global Account | QuickMed-Global | ? |
| Subaccount 1 | ? | ? |
| Subaccount 2 | ? | ? |
| Subaccount 3 (optional) | ? | ? |
| Space 1 | ? | ? |
| Space 2 | ? | ? |

#### 2. Service Selection

Pick SAP BTP services QuickMed would need:

| QuickMed Need | SAP BTP Service | Why This Service? |
|---------------|----------------|-------------------|
| Database for patient records | ? | ? |
| User login for doctors/staff | ? | ? |
| Connect clinic devices to cloud | ? | ? |
| Build the appointment app | ? | ? |
| Dashboard for clinic managers | ? | ? |

*Hint: Think about HANA Cloud, XSUAA, Integration Suite, CAP, Analytics Cloud*

#### 3. Runtime Choice

Which SAP BTP runtime would you recommend for QuickMed and why?
- Cloud Foundry?
- Kyma?
- ABAP Environment?

Justify your choice in 2-3 sentences.

---

### Part C: Presentation Prep (15 minutes)

Prepare a **3-minute presentation** for the class covering:

1. **Your architecture diagram** — walk through it
2. **The Monday Morning fix** — how cloud solves the crash
3. **SAP BTP mapping** — your account structure and service choices
4. **One risk** — what could go wrong with your design?

#### Presentation Tips:
- Every team member must speak (divide sections)
- Use the whiteboard/paper diagram
- Be ready for questions from other teams!

---

## Judging Criteria (Peer-Scored)

Each team scores the other teams (1-5 per category):

| Criteria | Max Points | What Judges Look For |
|----------|-----------|---------------------|
| **Architecture Completeness** | 5 | All components addressed? Makes sense? |
| **Monday Morning Solution** | 5 | Clear explanation? Would it actually work? |
| **SAP BTP Mapping** | 5 | Correct services chosen? Good justification? |
| **Creativity** | 5 | Any unique/smart ideas? |
| **Presentation Clarity** | 5 | Easy to follow? Good diagram? |
| **Total** | 25 | |

---

## Resources You Can Use

- Day 1 training material (Cloud Computing & SAP Ecosystem)
- Google/internet for researching cloud services
- SAP Discovery Center: https://discovery-center.cloud.sap/
- SAP BTP Services: https://help.sap.com/docs/btp

---

## Deliverables (Submit Before Class)

Each team submits:
1. **Architecture diagram** (photo of whiteboard/paper drawing)
2. **Decision table** (filled in - Part A, Question 1)
3. **SAP BTP mapping table** (filled in - Part B)
4. **Team presentation** (3 minutes during class)

---

## Bonus Challenge (Optional - Extra Points)

If your team finishes early, tackle this:

> QuickMed's competitor "HealthFirst" just launched an AI-powered symptom checker chatbot. QuickMed's CEO wants the same thing within 2 months.
>
> - Which SAP BTP service(s) would help?
> - Where in your architecture would this fit?
> - Add it to your diagram!

---

*Good luck, Cloud Architects!*
