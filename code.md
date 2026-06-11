## What You'll Learn Today

By the end of this session, you will be able to:
- Explain what SAP Fiori is and its 5 core design principles
- Describe the role of SAP Fiori Launchpad
- Understand the SAPUI5 framework and its relationship with Fiori
- Clearly differentiate Fiori Elements from Freestyle UI5 development
- Choose the right approach (Fiori Elements vs Freestyle) for a given scenario
- Identify the 4 main Fiori Elements floorplan types
- Understand how CDS annotations drive the UI automatically
- Preview your CAP application's data using `cds watch`

---

## Week 5 Kickoff & Motivation (09:00 - 09:15)

### Where We Are in the Journey

```
Week 1 (Days 1-5):   Foundations    → HTML, CSS, JavaScript, Node.js
Week 2 (Days 6-10):  Web Dev       → Express.js, REST APIs
Week 3 (Days 11-15): Database      → CAP intro, CDS modeling, views
Week 4 (Days 16-20): Services      → OData, CRUD, handlers, actions
Week 5 (Days 21-25): UI Layer      → 👈 YOU ARE HERE!
```

### The Big Picture: You've Built the Backend — Now Let's Add a Face!

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR CAP APPLICATION                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────┐                                    │
│  │   app/ (UI Layer)   │  ← 🆕 THIS WEEK!                  │
│  │   SAP Fiori         │     How users SEE and INTERACT     │
│  │   UI5 / Elements    │     with your data                 │
│  └──────────┬──────────┘                                    │
│             │ uses                                           │
│  ┌──────────▼──────────┐                                    │
│  │   srv/ (Service)    │  ✅ Done! (Days 16-20)             │
│  │   OData APIs        │     CRUD + Actions + Functions     │
│  └──────────┬──────────┘                                    │
│             │ reads/writes                                   │
│  ┌──────────▼──────────┐                                    │
│  │   db/ (Database)    │  ✅ Done! (Days 11-15)             │
│  │   CDS Entities      │     Schema + Data + Views          │
│  └─────────────────────┘                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Today's mindset shift:** Until now you've been building for machines (APIs return JSON). Starting today, you're building for HUMANS (UIs show forms, tables, buttons).

---

## Session 1: What is SAP Fiori? Design Principles (09:15 - 10:30)

### What is SAP Fiori?

**SAP Fiori** is SAP's design language and set of UI guidelines for building enterprise applications. Think of it as SAP's answer to the question: "How should business apps look and behave?"

**Simple definition:** Fiori = Design rules + UI technology + Pre-built app patterns for SAP systems.

---

### Before Fiori vs After Fiori

```
BEFORE FIORI (SAP GUI — the old world):
┌────────────────────────────────────────────────┐
│ ███████████████ MENU BAR █████████████████████ │
│ ┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐┌─┐│
│ └─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘└─┘│
│ ├── Too many buttons                          │
│ ├── Tiny text, no whitespace                  │
│ ├── Looks like a 1995 Windows app             │
│ ├── Needs training to use                     │
│ └── Works only on desktop                     │
└────────────────────────────────────────────────┘

AFTER FIORI (modern world):
┌────────────────────────────────────────────────┐
│                                                │
│    📋 My Purchase Orders                       │
│                                                │
│    ┌──────────────────────────────────────┐    │
│    │ PO-001  │ Tech Parts │ $4,500 │ ✅  │    │
│    │ PO-002  │ Office Co  │ $1,200 │ ⏳  │    │
│    │ PO-003  │ Cloud Inc  │ $8,900 │ ❌  │    │
│    └──────────────────────────────────────┘    │
│                                                │
│    Clean, simple, works on phone & laptop      │
└────────────────────────────────────────────────┘
```

---

### The 5 SAP Fiori Design Principles

Every Fiori app MUST follow these 5 principles. Memorize them — they're asked in interviews!

```
┌─────────────────────────────────────────────────────────────┐
│              THE 5 FIORI DESIGN PRINCIPLES                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1️⃣  ROLE-BASED                                             │
│      Show only what THIS user needs for THEIR job.          │
│      A manager sees approvals. An employee sees requests.   │
│                                                             │
│  2️⃣  ADAPTIVE                                               │
│      Works on desktop, tablet, AND phone.                   │
│      Same app, adjusts layout automatically.                │
│                                                             │
│  3️⃣  COHERENT                                               │
│      All Fiori apps look and feel the same.                 │
│      Learn one → use any. Consistent patterns everywhere.   │
│                                                             │
│  4️⃣  SIMPLE                                                 │
│      Remove complexity. Show less, mean more.               │
│      1-1-3 rule: 1 user, 1 use case, 3 screens max.        │
│                                                             │
│  5️⃣  DELIGHTFUL                                             │
│      Beautiful, fast, enjoyable to use.                     │
│      Users should WANT to use it, not dread it.             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Memory Trick: R-A-C-S-D

**R**ole-based → **A**daptive → **C**oherent → **S**imple → **D**elightful

Think: "**R**eal **A**pps **C**an't **S**ucceed without **D**esign"

---

### Principle 1: Role-Based — Show Only What Matters

```
WRONG: One app for everyone with 50 features
┌────────────────────────────────────────┐
│ Employee sees: HR stuff + Finance +    │
│ Purchasing + Admin + Reports + ...     │
│ (overwhelmed, confused, can't find    │
│  their leave balance!)                 │
└────────────────────────────────────────┘

RIGHT: Targeted apps per role
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ My Leave     │ │ Approve      │ │ Leave        │
│ Requests     │ │ Requests     │ │ Reports      │
│ (Employee)   │ │ (Manager)    │ │ (HR Admin)   │
└──────────────┘ └──────────────┘ └──────────────┘
Each person sees exactly what they need — nothing more.
```

---

### Principle 2: Adaptive — One App, Every Screen

```
Desktop (1920px):
┌────────────────────────────────────────────────────────┐
│ [List]                    │      [Detail Panel]        │
│ Order 1                   │  Order 1: Full details     │
│ Order 2  ← selected      │  Items, amounts, status    │
│ Order 3                   │  Actions: Approve, Reject  │
└────────────────────────────────────────────────────────┘

Tablet (768px):
┌──────────────────────────────────┐
│ [List — full width]              │
│ Order 1                          │
│ Order 2                          │ → tap → full detail page
│ Order 3                          │
└──────────────────────────────────┘

Phone (375px):
┌──────────────────┐
│ Order 1          │
│ Order 2          │ → tap → detail
│ Order 3          │
└──────────────────┘
```

Same data, same app — layout adapts to screen size. No separate mobile app needed!

---

### Principle 3: Coherent — Learn One, Know All

Every Fiori app uses the same:
- Header layout
- Navigation patterns (back button, breadcrumbs)
- Color meanings (green = positive, red = negative, orange = warning)
- Action button placement (top-right)
- Table behaviors (sort, filter, group)
- Form layouts

**Result:** A user who knows "My Leave Requests" can immediately use "My Purchase Orders" without training.

---

### Principle 4: Simple — The 1-1-3 Rule

```
1 User     → designed for ONE specific role
1 Use Case → solves ONE specific task
3 Screens  → completed in maximum 3 screens (list → detail → edit)
```

**Examples:**
| App | User | Task | Screens |
|-----|------|------|---------|
| Approve Leave | Manager | Review and approve/reject | List → Detail + Action |
| Create PO | Buyer | Submit a purchase order | Form → Review → Confirm |
| Track Delivery | Customer | See where my order is | Status page |

---

### Principle 5: Delightful — Not Just Functional

- Smooth animations (not jarring page jumps)
- Instant feedback (button changes color on click)
- Friendly messages ("Your order is confirmed!" not "Record 4823 updated successfully")
- Fast loading (skeleton screens, not blank waits)
- Celebrate success (checkmark animation on completion)

---

### What is SAP Fiori Launchpad (FLP)?

The **Fiori Launchpad** is like the "home screen" of your phone — but for SAP apps.

```
┌─────────────────────────────────────────────────────────────┐
│  SAP Fiori Launchpad                           👤 John Doe  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐      │
│  │  📋     │  │  📝     │  │  📊     │  │  ✅     │      │
│  │ My POs  │  │ Create  │  │ Reports │  │ Approve │      │
│  │         │  │ Order   │  │         │  │ Orders  │      │
│  │   3     │  │         │  │         │  │   7     │      │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘      │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                    │
│  │  🏢     │  │  👥     │  │  💰     │                    │
│  │ Manage  │  │ Team    │  │ Budget  │                    │
│  │ Products│  │ Members │  │ Monitor │                    │
│  └─────────┘  └─────────┘  └─────────┘                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Key features of FLP:**
- Each tile = one Fiori app
- Tiles show live counts/KPIs (badge "7" means 7 pending approvals)
- Personalized per user role (manager sees approval tiles, employee sees request tiles)
- Single sign-on (login once, access all apps)
- Runs in a browser — no installation needed
- Works on BTP, S/4HANA, or any SAP system

---

### Fiori Launchpad Architecture

```
User opens browser → Fiori Launchpad → Clicks a Tile → Fiori App loads
                                                              │
                                                              ▼
                                                    ┌──────────────────┐
                                                    │  OData Service   │
                                                    │  (YOUR CAP app!) │
                                                    └──────────────────┘
```

Your CAP service provides the DATA. Fiori provides the UI. They connect via OData.

---

## Session 2: SAPUI5 Framework & Fiori Elements vs Freestyle (10:45 - 12:00)

### What is SAPUI5?

**SAPUI5** is the JavaScript UI framework that Fiori apps are built with. Think of it as SAP's version of React or Angular.

```
Fiori = Design rules (HOW it should look)
UI5   = Technology (HOW you build it)

Analogy:
  Fiori = Architecture blueprint (what the house should look like)
  UI5   = Bricks and tools (what you build it with)
```

**Key facts about SAPUI5:**
- JavaScript-based framework (runs in browser)
- MVC architecture (Model-View-Controller)
- Rich set of UI controls (tables, forms, charts, buttons, dialogs)
- Built-in responsive design
- OData integration out of the box
- Open-source version available: **OpenUI5**

---

### UI5 Controls — What You Get

SAPUI5 provides 500+ pre-built UI controls:

| Category | Controls |
|----------|----------|
| **Display** | Text, Label, Image, Icon, Avatar |
| **Input** | Input Field, DatePicker, Select, MultiInput, Switch |
| **Container** | Page, Panel, Dialog, Popover, VBox, HBox |
| **List** | Table, List, Tree, Timeline |
| **Chart** | Bar, Line, Pie, Donut (via VizFrame) |
| **Navigation** | Shell, NavContainer, Breadcrumbs, Tabs |
| **Action** | Button, MenuButton, SegmentedButton, Link |
| **Feedback** | MessageBox, MessageStrip, BusyIndicator, Rating |

You don't build these from scratch — you use them like Lego blocks!

---

### Two Ways to Build Fiori Apps

Here's the most important concept of today:

```
┌─────────────────────────────────────────────────────────────┐
│         TWO APPROACHES TO BUILD FIORI UIS                    │
├──────────────────────────────┬──────────────────────────────┤
│                              │                              │
│     FIORI ELEMENTS           │      FREESTYLE UI5           │
│     (Annotation-driven)      │      (Code-driven)           │
│                              │                              │
│  "Tell me WHAT to show"      │  "Tell me HOW to show it"   │
│                              │                              │
│  You write: annotations      │  You write: XML views +     │
│  CAP generates: the UI       │  JavaScript controllers     │
│                              │                              │
│  Like ordering from a menu   │  Like cooking from scratch  │
│                              │                              │
│  ┌──────────────────────┐    │  ┌──────────────────────┐   │
│  │  @UI.LineItem: [...]  │    │  │  <Table items="..."   │   │
│  │  @UI.HeaderInfo: {..} │    │  │    <Column>           │   │
│  │  @UI.FieldGroup: ...  │    │  │      <Text text=".."/>│   │
│  │                       │    │  │    </Column>          │   │
│  │  → CAP auto-generates │    │  │  </Table>             │   │
│  │    a complete UI!     │    │  │  + controller.js      │   │
│  └──────────────────────┘    │  └──────────────────────┘   │
│                              │                              │
└──────────────────────────────┴──────────────────────────────┘
```

---


https://openui5.org/documentation.html

https://ui5.sap.com/#/

