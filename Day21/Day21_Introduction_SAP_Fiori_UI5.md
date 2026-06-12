# Day 21: Introduction to SAP Fiori & UI5

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Week 5 Kickoff & Motivation | 15 min |
| 09:15 - 10:30 | Session 1: What is SAP Fiori? Design Principles | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: SAPUI5 Framework & Fiori Elements vs Freestyle | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Explore Fiori Design Guidelines & Sample Apps | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Fiori Elements Floorplans — The Complete Guide | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Annotations for UI & Preview in CAP | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Preview Application & Identify Floorplans | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

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

### Fiori Elements — The "Smart" Approach

**Fiori Elements** generates a complete UI automatically from your service metadata and annotations. You don't write HTML/XML views — you describe what data to show, and it builds the app for you.

**How it works:**

```
1. You define your CDS model + service         (db/schema.cds + srv/service.cds)
2. You add UI annotations                     (srv/annotations.cds)
3. CAP reads $metadata + annotations
4. Fiori Elements runtime generates the UI     (automatic!)
5. User sees a complete, working application   (tables, forms, navigation!)
```

**What you write (annotations):**
```cds
annotate Products with @UI: {
  HeaderInfo: { Title: { Value: productName } },
  LineItem: [
    { Value: productName },
    { Value: price },
    { Value: stock }
  ]
};
```

**What you get (auto-generated app):**
```
┌─────────────────────────────────────────────┐
│  Products (3)                    [+ Create] │
├─────────────────────────────────────────────┤
│  Product Name      │ Price    │ Stock       │
├────────────────────┼──────────┼─────────────┤
│  Laptop Pro        │ $999.00  │ 45          │
│  Wireless Mouse    │ $29.99   │ 120         │
│  USB-C Hub         │ $49.99   │ 80          │
└─────────────────────────────────────────────┘
```

You wrote 10 lines of annotations → got a complete working app with:
- Sortable/filterable table
- Create/Edit/Delete buttons
- Detail page on click
- Responsive layout
- Search bar
- Pagination

---

### Freestyle UI5 — The "Custom" Approach

**Freestyle** means you code every part of the UI yourself using XML views and JavaScript:

**What you write:**
```xml
<!-- view/ProductList.view.xml -->
<mvc:View xmlns="sap.m" xmlns:mvc="sap.ui.core.mvc" controllerName="app.controller.ProductList">
  <Page title="Products">
    <Table items="{/Products}">
      <columns>
        <Column><Text text="Name"/></Column>
        <Column><Text text="Price"/></Column>
        <Column><Text text="Stock"/></Column>
      </columns>
      <items>
        <ColumnListItem>
          <cells>
            <Text text="{productName}"/>
            <ObjectNumber number="{price}" unit="USD"/>
            <Text text="{stock}"/>
          </cells>
        </ColumnListItem>
      </items>
    </Table>
  </Page>
</mvc:View>
```

```javascript
// controller/ProductList.controller.js
sap.ui.define(["sap/ui/core/mvc/Controller"], function(Controller) {
  return Controller.extend("app.controller.ProductList", {
    onInit: function() {
      // initialization logic
    },
    onItemPress: function(oEvent) {
      // navigation to detail page
    }
  });
});
```

**You get more control, but you write 10x more code!**

---

### Fiori Elements vs Freestyle — Detailed Comparison

| Criteria | Fiori Elements | Freestyle UI5 |
|----------|---------------|---------------|
| **Code to write** | Minimal (just annotations) | A lot (XML + JS + CSS) |
| **Development speed** | Very fast | Slower |
| **Customization** | Limited to what annotations allow | Unlimited |
| **UI consistency** | Guaranteed (same patterns always) | Developer's responsibility |
| **Maintenance** | Auto-upgrades with SAP updates | You maintain everything |
| **Learning curve** | Learn annotations | Learn UI5 framework |
| **Use case** | Standard CRUD apps, admin panels | Custom UIs, dashboards, games |
| **Looks like** | Standard Fiori (professional) | Whatever you design |
| **SAP recommendation** | "Use this first!" | "Only when Elements can't do it" |
| **Floorplans** | Pre-defined (List Report, Object Page, etc.) | You design your own |

---

### When to Use Which?

```
Start with this question:
"Is my app a standard data management app?"

YES → Fiori Elements (90% of enterprise apps!)
  ├── List of records → List Report
  ├── Detail view → Object Page
  ├── Create/Edit → Object Page (edit mode)
  ├── Wizard → Guided process
  └── Overview → Overview Page

NO → Freestyle UI5
  ├── Custom dashboard with charts
  ├── Map-based application
  ├── Multi-step complex wizard
  ├── Game or creative UI
  └── Something SAP patterns don't support
```

**Rule of thumb:** If you can describe your app as "show a list of [things], click to see details, edit/create/delete" → USE FIORI ELEMENTS. This covers 90% of business apps!

---

### The Power of Annotations — One Definition, Many UIs

With Fiori Elements, the SAME annotations generate different UIs depending on the device:

```
Your annotations define:
  "Show productName, price, stock in a list"
  "On detail page, show description and supplier info"

DESKTOP gets:                    PHONE gets:
┌──────────────────────────┐    ┌───────────────┐
│ List         │ Detail    │    │ Product Name  │
│ ─────────── │ ───────── │    │ Price         │
│ Laptop Pro  │ Laptop Pro│    │ Stock         │
│ Mouse       │ $999.00   │    │               │
│ Hub         │ 45 units  │    │ → tap for     │
│             │ Desc...   │    │   detail      │
└──────────────────────────┘    └───────────────┘
Split-view (master-detail)       Simple list → detail
```

You write annotations ONCE — the framework handles responsiveness!

---

## Session 3: Hands-on — Explore Fiori Design Guidelines & Sample Apps (12:00 - 13:00)

### Exercise 1: Explore Fiori Design Guidelines (20 minutes)

Open the official SAP Fiori Design Guidelines:
**https://experience.sap.com/fiori-design-web/**

Navigate through and answer these questions:

1. Find the "Design Principles" section. List all 5 principles.
2. Go to "Floorplans" → Find "List Report". Read the "When to Use" section.
3. Go to "Controls" → Find "Table". How many table types exist?
4. Look at the color palette. What does the color "Negative" (red) represent?
5. Find the "Object Page" floorplan. What are its main sections?

---

### Exercise 2: Explore Live Fiori Apps Demo (20 minutes)

Open SAP's Fiori apps library:
**https://fioriappslibrary.hana.ondemand.com/**

Browse through and find:

1. Search for "Purchase Order" — how many apps exist for this area?
2. Click on any app → note its **Floorplan type** (listed in the details)
3. Find an app that uses "List Report" floorplan 
4. Find an app that uses "Object Page" floorplan
5. Note the common UI patterns across different apps (consistent design!)

---

### Exercise 3: See Your CAP App's Data in the Browser (20 minutes)

Make sure your CAP project is running:
```bash
cds watch
```

Open `http://localhost:4004` and explore:

1. Click on any service endpoint (e.g., `/catalog/Products`)
2. Notice: this is RAW JSON data — no UI yet!
3. Try: `/catalog/Products?$top=5&$select=productName,price`
4. Try: `/catalog/$metadata` — this is what Fiori Elements reads to build the UI

**Key insight:** The data is ready. The API is ready. All we need now is the UI layer (annotations + Fiori Elements) to turn this JSON into a beautiful application.

---

## Session 4: Fiori Elements Floorplans — The Complete Guide (13:45 - 14:45)

### What is a Floorplan?

A **floorplan** is a pre-defined page layout pattern. Think of it as a "template" for a type of page.

**Analogy:** When you visit any e-commerce website:
- The "search results" page always looks similar (list with filters on the side)
- The "product detail" page always has an image, title, price, description
- The "checkout" page always has steps (address → payment → confirm)

These are **patterns/floorplans** — and Fiori Elements provides similar patterns for business apps.

---

### The 4 Main Fiori Elements Floorplans

```
┌─────────────────────────────────────────────────────────────┐
│              FIORI ELEMENTS FLOORPLANS                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. LIST REPORT           "Show me a list of things"        │
│     ┌─────────────────────────────────────────┐             │
│     │ Filter Bar: [Status ▼] [Date Range]     │             │
│     │ ┌─────────────────────────────────────┐ │             │
│     │ │ Name      │ Status │ Amount │ Date  │ │             │
│     │ │ Order 1   │ New    │ $500   │ Jun 1 │ │             │
│     │ │ Order 2   │ Done   │ $300   │ Jun 2 │ │             │
│     │ └─────────────────────────────────────┘ │             │
│     └─────────────────────────────────────────┘             │
│                                                             │
│  2. OBJECT PAGE           "Show me details of ONE thing"    │
│     ┌─────────────────────────────────────────┐             │
│     │ ◀ Back    Order: SO-001    [Edit] [Del] │             │
│     │ ┌───────────────────────────────────┐   │             │
│     │ │ Status: ✅ Confirmed              │   │             │
│     │ │ Customer: Acme Corp               │   │             │
│     │ │ Amount: $1,500.00                 │   │             │
│     │ ├───────────────────────────────────┤   │             │
│     │ │ Tab: Items │ Tab: History          │   │             │
│     │ │ ─────────────────────────────     │   │             │
│     │ │ Product A  │ Qty: 5  │ $500      │   │             │
│     │ │ Product B  │ Qty: 2  │ $1000     │   │             │
│     │ └───────────────────────────────────┘   │             │
│     └─────────────────────────────────────────┘             │
│                                                             │
│  3. WORKLIST              "Things I need to act on"         │
│     ┌─────────────────────────────────────────┐             │
│     │ My Tasks (7)                            │             │
│     │ ┌─────────────────────────────────────┐ │             │
│     │ │ ⏳ Approve PO-001    │ Due: Today  │ │             │
│     │ │ ⏳ Review Invoice    │ Due: Jun 5  │ │             │
│     │ │ ⏳ Confirm Delivery  │ Due: Jun 3  │ │             │
│     │ └─────────────────────────────────────┘ │             │
│     └─────────────────────────────────────────┘             │
│                                                             │
│  4. OVERVIEW PAGE         "Dashboard / KPIs at a glance"    │
│     ┌─────────────────────────────────────────┐             │
│     │ ┌────────┐ ┌────────┐ ┌────────────┐   │             │
│     │ │Revenue │ │Orders  │ │ Top Products│   │             │
│     │ │$45,000 │ │  127   │ │ ┌─────────┐│   │             │
│     │ │ 📈+12% │ │ 📈+5%  │ │ │ ██████  ││   │             │
│     │ └────────┘ └────────┘ │ │ ████    ││   │             │
│     │                       │ │ ██      ││   │             │
│     │                       │ └─────────┘│   │             │
│     │                       └────────────┘   │             │
│     └─────────────────────────────────────────┘             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Floorplan 1: List Report — The Most Common

**What it is:** A filterable, sortable table showing a list of business objects.

**When to use:** Any time you need to show a collection of records that users browse, filter, and select.

**Features (auto-generated!):**
- Smart Filter Bar (filter by any field)
- Table with configurable columns
- Sorting, grouping, search
- Row selection → navigates to Object Page
- Export to Excel
- Create button
- Variant management (save filter combinations)

**Real-world examples:**
- List of Purchase Orders
- List of Products
- List of Employees
- List of Sales Orders

---

### Floorplan 2: Object Page — The Detail View

**What it is:** A detailed view of ONE specific business object.

**When to use:** When a user clicks a row in the List Report to see full details.

**Structure:**

```
┌──────────────────────────────────────────┐
│  HEADER (always visible when scrolling)  │
│  ┌────────────────────────────────────┐  │
│  │ Title: Order SO-001               │  │
│  │ Subtitle: Customer: Acme Corp     │  │
│  │ Status: ✅ Confirmed              │  │
│  │ KPI: Total $1,500 | Items: 5      │  │
│  └────────────────────────────────────┘  │
│                                          │
│  SECTIONS (scrollable content)           │
│  ┌────────────────────────────────────┐  │
│  │ § General Information              │  │
│  │   Order Date: June 1, 2026        │  │
│  │   Payment: Credit Card            │  │
│  │   Delivery: Express               │  │
│  ├────────────────────────────────────┤  │
│  │ § Order Items                      │  │
│  │   Laptop Pro    │ 2 │ $999 │$1998 │  │
│  │   Mouse         │ 5 │ $30  │ $150 │  │
│  ├────────────────────────────────────┤  │
│  │ § History / Notes                  │  │
│  │   Jun 1: Created by John          │  │
│  │   Jun 2: Confirmed by Manager     │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

**Features:**
- Sticky header with key info
- Multiple sections (collapsible)
- Tabs for sub-collections (items, attachments, history)
- Edit mode toggle
- Action buttons (Approve, Reject, Delete)

---

### Floorplan 3: Worklist — Task-Oriented

**What it is:** A focused list for users who need to process items one by one.

**Difference from List Report:**
- List Report = "Browse and explore data"
- Worklist = "Here are things you need to DO right now"

**When to use:** Approvals, task queues, inbox-style apps.

---

### Floorplan 4: Overview Page — Dashboard

**What it is:** A card-based dashboard showing KPIs and summaries from multiple data sources.

**When to use:** Executive dashboards, home pages, performance monitoring.

**Features:**
- Multiple cards (each with its own data source)
- KPI values with trend arrows
- Mini charts (bar, donut, line)
- Quick links to detailed apps

---

### Floorplan Selection Guide

```
"What does the user need to do?"

├── Browse/search a list of records     → LIST REPORT
├── View full details of one record     → OBJECT PAGE
├── Process a queue of tasks            → WORKLIST
├── See KPIs and summaries at a glance  → OVERVIEW PAGE
└── Step-by-step data entry             → WIZARD (less common)
```

---

### The Typical App Flow

Most business apps follow this pattern:

```
List Report ──click row──► Object Page ──click Edit──► Object Page (Edit Mode)
                                       ──click Action──► Status Change
```

**Example: Managing Products:**
1. User opens "Manage Products" → sees **List Report** with all products
2. User clicks "Laptop Pro" → sees **Object Page** with full details
3. User clicks "Edit" → same Object Page switches to edit mode
4. User saves → back to display mode

---

## Session 5: Annotations for UI & Preview in CAP (15:00 - 16:00)

### How Fiori Elements Gets Its Instructions

Fiori Elements needs to know: "Which fields to show? In what order? What's the title?"

These instructions come from **annotations** — metadata you add to your CDS entities.

```
CDS Entity (data structure)  +  Annotations (UI hints)  =  Complete Fiori App
```

---

### Where Do Annotations Go?

You write annotations in a separate file (keeps your schema clean):

```
Project structure:
├── db/
│   └── schema.cds            ← Data model (entities, types)
├── srv/
│   ├── catalog-service.cds   ← Service definition
│   └── annotations.cds       ← 🆕 UI Annotations go HERE
└── app/
    └── (Fiori app config)
```

---

### Annotation Basics — Syntax

```cds
// srv/annotations.cds
using { CatalogService } from './catalog-service';

// Annotate an entity in the service:
annotate CatalogService.Products with @UI: {

  // What to show in the table header
  HeaderInfo: {
    TypeName: 'Product',
    TypeNamePlural: 'Products',
    Title: { Value: productName },
    Description: { Value: description }
  },

  // Which columns to show in the list (table)
  LineItem: [
    { Value: productName, Label: 'Product Name' },
    { Value: price, Label: 'Price' },
    { Value: stock, Label: 'Stock' },
    { Value: rating, Label: 'Rating' }
  ]
};
```

**What this produces:**

A List Report with:
- Page title: "Products"
- Table columns: Product Name, Price, Stock, Rating
- Each row shows those 4 fields from your data

---

### Common Annotation Types — Quick Reference

| Annotation | Purpose | Used In |
|------------|---------|---------|
| `@UI.HeaderInfo` | Title and subtitle of the page | Object Page header |
| `@UI.LineItem` | Columns in the table | List Report |
| `@UI.FieldGroup` | Group of fields on the detail page | Object Page sections |
| `@UI.SelectionFields` | Fields available in the filter bar | List Report filter |
| `@UI.Facets` | Layout sections on the detail page | Object Page structure |
| `@UI.DataPoint` | KPI values shown in the header | Object Page header |
| `@UI.Chart` | Chart configuration | Overview Page |
| `@UI.Identification` | Actions available on the entity | Object Page actions |

---

### Preview in CAP — Seeing Your Data

Even without a full Fiori app setup, CAP provides a basic preview:

```bash
cds watch
```

Open `http://localhost:4004` → You see:
- Service index with all endpoints
- Click any entity → see raw JSON data
- Click `$metadata` → see the OData schema

**For a Fiori-style preview**, you'll add a Fiori Elements app in the `app/` folder (we'll do this in Day 22). For now, understand that:

1. Your service is already Fiori-compatible (it speaks OData V4)
2. Adding annotations is all that's needed for Fiori Elements to render a UI
3. The `$metadata` endpoint already describes your entire API to Fiori

---

### How `$metadata` Drives the UI

When a Fiori Elements app loads, it:

```
1. Reads $metadata → "What entities exist? What fields? What types?"
2. Reads annotations → "Which fields go in the table? What's the title?"
3. Generates the UI → Creates table, form, filters based on instructions
4. Fetches data → Calls the OData service for actual records
5. Renders → Shows the complete app to the user
```

Everything is driven by metadata. Change your annotations → UI updates. No redeployment needed!

---

### Quick Annotation Example: Products

```cds
// srv/annotations.cds
using { CatalogService } from './catalog-service';

annotate CatalogService.Products with @UI: {
  HeaderInfo: {
    TypeName: 'Product',
    TypeNamePlural: 'Products',
    Title: { Value: productName },
    Description: { Value: category }
  },
  SelectionFields: [ productName, price, stock ],
  LineItem: [
    { Value: productName },
    { Value: price },
    { Value: stock },
    { Value: rating },
    { Value: supplier.supplierName, Label: 'Supplier' }
  ],
  Facets: [
    {
      $Type: 'UI.ReferenceFacet',
      Target: '@UI.FieldGroup#General',
      Label: 'General Information'
    }
  ],
  FieldGroup#General: {
    Data: [
      { Value: productName },
      { Value: description },
      { Value: price },
      { Value: stock },
      { Value: rating }
    ]
  }
};
```

**Result:**
- **List Report:** Table with productName, price, stock, rating, supplier columns
- **Filter Bar:** Can filter by productName, price, stock
- **Object Page:** Header shows product name + category. Body has "General Information" section with all key fields.

We'll implement this in detail on Day 22!

---

## Session 6: Hands-on — Preview Application & Identify Floorplans (16:00 - 16:45)

### Exercise 1: Preview Your CAP Service Data (15 minutes)

Run `cds watch` and explore these URLs in the browser:

| URL | What You See |
|-----|-------------|
| `http://localhost:4004` | Service index — all available services and entities |
| `http://localhost:4004/catalog/Products` | JSON array of all products |
| `http://localhost:4004/catalog/Products?$top=3&$select=productName,price` | Minimal data |
| `http://localhost:4004/catalog/Products?$expand=supplier` | Nested supplier data |
| `http://localhost:4004/catalog/$metadata` | XML schema — what Fiori reads |

**Questions to answer:**
1. How many entities does your CatalogService expose? _____
2. What is the OData type of the `price` field in $metadata? _____
3. Can you find NavigationProperties in $metadata? What do they represent? _____

---

### Exercise 2: Identify Floorplan Types (15 minutes)

For each screenshot description below, identify the Fiori Elements floorplan:

**Scenario A:**
> A page showing a table with 500 employees. At the top there's a filter bar with fields for Department, Status, and Hire Date. Each row shows Name, Email, Department, and Status. Clicking a row takes you to the employee's full profile.

**Floorplan:** _____________

---

**Scenario B:**
> A page showing the full details of Purchase Order PO-001. At the top: PO number, supplier name, status badge, and total amount. Below: sections for "General Info", "Line Items" (table), and "Approval History".

**Floorplan:** _____________

---

**Scenario C:**
> A dashboard page with 4 cards: "Revenue This Month" (big number with arrow up), "Pending Approvals" (count: 12), "Top 5 Products" (bar chart), and "Recent Orders" (mini table with last 5 orders).

**Floorplan:** _____________

---

**Scenario D:**
> A focused list showing 7 items titled "My Pending Approvals". Each row has a document title, requestor name, amount, and due date. The user processes items one by one, clicking Approve or Reject on each.

**Floorplan:** _____________

<details>
<summary>Answers</summary>

- **A:** List Report (filterable table of records)
- **B:** Object Page (detailed view of one record with sections)
- **C:** Overview Page (dashboard with KPI cards)
- **D:** Worklist (task queue for processing)

</details>

---

### Exercise 3: Map Your EPM Model to Floorplans (15 minutes)

Think about your EPM data model (Products, Orders, Suppliers, Customers). For each app below, decide which floorplan you would use:

| App | Primary User | Floorplan Choice | Why? |
|-----|-------------|-----------------|------|
| Manage Products | Admin | _____ | _____ |
| View Product Detail | Admin | _____ | _____ |
| Approve Purchase Orders | Manager | _____ | _____ |
| Sales Dashboard | Executive | _____ | _____ |
| Browse Product Catalog | Customer | _____ | _____ |

<details>
<summary>Answers</summary>

| App | Floorplan | Why |
|-----|-----------|-----|
| Manage Products | **List Report** | Browse, filter, sort all products |
| View Product Detail | **Object Page** | See all details of one product |
| Approve Purchase Orders | **Worklist** | Process pending approvals one by one |
| Sales Dashboard | **Overview Page** | KPIs and charts at a glance |
| Browse Product Catalog | **List Report** | Filter and search products |

</details>

---

## Assessment: MCQ — 15 Questions on Fiori Fundamentals (16:45 - 17:00)

**Q1.** What are the 5 SAP Fiori design principles?

- A) Fast, Secure, Scalable, Testable, Maintainable
- B) Role-based, Adaptive, Coherent, Simple, Delightful
- C) Modular, Responsive, Accessible, Performant, Localized
- D) Model, View, Controller, Service, Database

**Answer: B** — R-A-C-S-D: Role-based, Adaptive, Coherent, Simple, Delightful.

---

**Q2.** "Adaptive" in Fiori design means:

- A) The app adapts to the user's skill level
- B) The app works on desktop, tablet, and phone — same app, different layouts
- C) The app adapts to different databases
- D) The app changes color based on theme

**Answer: B** — Adaptive = responsive across all device sizes. One app, auto-adjusting layout.

---

**Q3.** What is SAPUI5?

- A) A database management tool
- B) A JavaScript UI framework for building Fiori applications
- C) A deployment platform
- D) An API testing tool

**Answer: B** — UI5 is the JavaScript framework (like React/Angular) that provides the controls and patterns for Fiori apps.

---

**Q4.** The main difference between Fiori Elements and Freestyle UI5 is:

- A) Fiori Elements is faster at runtime; Freestyle is slower
- B) Fiori Elements generates UI from annotations; Freestyle requires manual XML/JS coding
- C) Fiori Elements works only on desktop; Freestyle works on all devices
- D) Fiori Elements is free; Freestyle requires a license

**Answer: B** — Fiori Elements = annotation-driven (automatic). Freestyle = code-driven (manual). Both are responsive, both use UI5.

---

**Q5.** When should you choose Fiori Elements over Freestyle?

- A) Only for simple apps with less than 10 fields
- B) For standard data management apps (list, detail, create, edit, delete)
- C) Only for mobile apps
- D) Only when the app has no custom logic

**Answer: B** — Fiori Elements is ideal for standard CRUD/data management apps — which covers ~90% of enterprise applications.

---

**Q6.** What is the SAP Fiori Launchpad?

- A) A code editor for building Fiori apps
- B) A home screen with tiles, providing single entry point to all Fiori apps for a user
- C) A database management interface
- D) A testing framework

**Answer: B** — FLP is the "home screen" where users see role-based tiles that launch Fiori apps. One login, all apps accessible.

---

**Q7.** Which floorplan shows a filterable table of business objects?

- A) Object Page
- B) Overview Page
- C) List Report
- D) Worklist

**Answer: C** — List Report = table with smart filter bar, sorting, search, and navigation to detail page.

---

**Q8.** Which floorplan shows the full details of ONE record with sections?

- A) List Report
- B) Object Page
- C) Worklist
- D) Overview Page

**Answer: B** — Object Page = detailed view of a single entity instance with header, sections, tabs.

---

**Q9.** Which floorplan is best for a KPI dashboard with charts and cards?

- A) List Report
- B) Object Page
- C) Worklist
- D) Overview Page

**Answer: D** — Overview Page = card-based dashboard showing KPIs, mini charts, and summaries.

---

**Q10.** Where do you write UI annotations in a CAP project?

- A) `db/schema.cds`
- B) `srv/annotations.cds` (or similar separate CDS file in srv/)
- C) `package.json`
- D) `app/index.html`

**Answer: B** — Annotations go in a CDS file (commonly `srv/annotations.cds` or `app/annotations.cds`) — separate from the data model.

---

**Q11.** `@UI.LineItem` annotation is used for:

- A) Defining which fields appear as table columns in a List Report
- B) Creating a new database table
- C) Defining the detail page layout
- D) Setting up authentication

**Answer: A** — `@UI.LineItem` defines the columns shown in the table (List Report).

---

**Q12.** What does Fiori Elements read to auto-generate the UI?

- A) The HTML files in the app folder
- B) The `$metadata` endpoint + CDS annotations
- C) The CSS stylesheets
- D) The JavaScript controller files

**Answer: B** — Fiori Elements reads OData $metadata (entity structure) + annotations (UI configuration) to generate the complete UI.

---

**Q13.** TRUE or FALSE: With Fiori Elements, you need to write HTML and CSS for the table layout.

- A) TRUE
- B) FALSE

**Answer: B (FALSE)** — Fiori Elements generates all UI code automatically. You only write annotations. No HTML, CSS, or JavaScript needed.

---

**Q14.** The "1-1-3 rule" in Fiori design means:

- A) 1 database, 1 service, 3 entities
- B) 1 user role, 1 use case, maximum 3 screens to complete the task
- C) 1 page, 1 table, 3 buttons
- D) 1 app, 1 floorplan, 3 annotations

**Answer: B** — 1 user, 1 use case, 3 screens max. Keep it focused and simple.

---

**Q15.** A typical Fiori app navigation flow is:

- A) Object Page → List Report → Overview Page
- B) List Report → Object Page → Edit Mode (or Action)
- C) Dashboard → Worklist → Settings
- D) Login → Registration → Profile

**Answer: B** — The standard flow: List Report (browse) → Object Page (details) → Edit or perform Action.

---

## Assignment: Identify and Document 5 Fiori Apps

### Task

Visit the SAP Fiori Apps Library (https://fioriappslibrary.hana.ondemand.com/) and document 5 different Fiori apps.

### What to Document

For each app, record:

| Field | Your Answer |
|-------|-------------|
| App Name | |
| App ID (if shown) | |
| Business Area | (e.g., Finance, Procurement, HR) |
| Floorplan Type | (List Report, Object Page, Worklist, Overview Page) |
| Primary User Role | (e.g., Buyer, Manager, Accountant) |
| Key Features | (3-5 bullet points of what the app does) |
| SAP Fiori Principle Demonstrated | (Which of the 5 principles is most visible?) |

### Requirements

- Choose apps from at LEAST 3 different business areas
- Include at LEAST 2 different floorplan types
- Write 2-3 sentences explaining WHY that floorplan was chosen for that app

### Example Entry

| Field | Example |
|-------|---------|
| App Name | Manage Purchase Orders |
| App ID | F0842 |
| Business Area | Procurement |
| Floorplan Type | List Report + Object Page |
| Primary User Role | Buyer / Purchaser |
| Key Features | - Browse all POs with filters (date, status, supplier) |
| | - See PO details with line items |
| | - Create new POs |
| | - Change status (approve, cancel) |
| SAP Fiori Principle | Role-based (only buyers see this app) + Simple (clear list→detail flow) |
| Why this floorplan? | List Report is ideal because buyers need to browse/filter many POs. Object Page shows the full PO with items — a natural list→detail pattern. |

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| SAP Fiori | A design language + UI approach for SAP apps |
| 5 Principles | Role-based, Adaptive, Coherent, Simple, Delightful |
| Fiori Launchpad | The home screen — tiles for each app |
| SAPUI5 | JavaScript framework (the technology behind Fiori) |
| Fiori Elements | Annotation-driven — UI generated from metadata |
| Freestyle UI5 | Code-driven — full manual control via XML + JS |
| List Report | Filterable table for browsing records |
| Object Page | Detailed view of one record with sections |
| Worklist | Task-focused list for processing items |
| Overview Page | KPI dashboard with cards |
| Annotations | CDS metadata that tells Fiori Elements what to show |

### The Decision Flowchart

```
Need a UI for your CAP app?
│
├── Standard CRUD app? (list/detail/edit)
│   └── YES → Fiori Elements + Annotations (fastest, recommended)
│
├── Custom dashboard/map/creative UI?
│   └── YES → Freestyle UI5 (more work, more control)
│
└── Just want to test your API?
    └── Use REST Client or cds watch preview
```

### The One-Liner

> **Fiori Elements = Tell CAP WHAT to show (annotations), not HOW to show it (code). The framework builds the UI for you.**

---

### Looking Ahead: Day 22

Tomorrow we'll get hands-on:
- Add a Fiori Elements app to your CAP project
- Write `@UI` annotations for your EPM model
- Generate a complete List Report + Object Page
- See your data in a real, working Fiori application!

---

### Homework

1. Visit https://experience.sap.com/fiori-design-web/ — read the "Principles" section completely
2. Complete the Assignment (document 5 Fiori apps with floorplan types)
3. Look at your EPM model entities and think: "What annotations would I need to show Products in a table?"
4. Bonus: Find the SAP Fiori Tools extension in VS Code / BAS — we'll use it tomorrow

---

*End of Day 21 — You now understand the UI layer philosophy. Tomorrow we BUILD it!*
