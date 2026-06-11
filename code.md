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


https://fioriappslibrary.hana.ondemand.com/sap/fix/externalViewer/#
