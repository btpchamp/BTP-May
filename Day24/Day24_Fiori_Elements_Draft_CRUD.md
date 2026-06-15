# Day 24: Fiori Elements — Draft & CRUD Operations

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 23 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: What is Draft Handling? Why Do We Need It? | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Enabling Draft in CAP & The Draft Lifecycle | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Enable Draft for PO Management | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Actions, Side Effects & Dynamic Expressions | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Complete CRUD & Actions in the UI | 60 min |
| 16:00 - 16:45 | Session 6: End-to-End PO Creation Flow | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain what Draft handling is and why it's essential for enterprise apps
- Enable draft on any CAP service entity with a single keyword
- Understand the draft lifecycle (New → Edit → Activate/Discard)
- Perform Create, Edit, and Delete through the Fiori Elements UI
- Configure Side Effects that refresh data when fields change
- Add action buttons with confirmation dialogs
- Use dynamic expressions to show/hide fields and buttons based on state
- Build a complete end-to-end Purchase Order creation experience

---

## Day 23 Recap — Quick Fire (09:00 - 09:15)

1. To configure 3-level drill-down, you define _____ in manifest.json? → _____
2. `@Common.ValueList` creates what in the UI? → _____
3. `@Common.Text: supplier.supplierName` shows _____ instead of _____? → _____
4. Criticality value `3` renders which color? → _____
5. `@UI.DataPoint` with `Visualization: #Rating` shows what? → _____
6. `CollectionFacet` does what? → _____
7. `@UI.DataFieldForAction` placed in `@UI.Identification` shows what? → _____
8. Grid Table is best for what scenario? → _____

<details>
<summary>Answers</summary>

1. **Routes** with patterns like `Entity({key})/childNav({key2})`
2. A **dropdown or dialog** to pick values from a related entity
3. The **supplier name** instead of the **UUID**
4. **Green** (Positive/Success)
5. **Star rating** display (⭐⭐⭐⭐☆)
6. **Groups multiple sections** into one parent section with sub-tabs
7. **Action buttons** in the Object Page header toolbar
8. **Desktop** with **many columns** (20+) and large datasets

</details>

---

## Session 1: What is Draft Handling? Why Do We Need It? (09:15 - 10:30)

### The Problem: Losing Work

Imagine you're filling out a Purchase Order form with 20 fields. After entering 15 fields...

```
WITHOUT DRAFT:
┌─────────────────────────────────────────────────┐
│  Create Purchase Order                          │
│                                                 │
│  PO Number: [PO-001       ]  ✅                 │
│  Supplier:  [TechCorp     ]  ✅                 │
│  Date:      [2026-06-15   ]  ✅                 │
│  Priority:  [High         ]  ✅                 │
│  Item 1:    [Laptop x5    ]  ✅                 │
│  Item 2:    [Mouse x10    ]  ✅                 │
│  ...15 fields filled...                         │
│                                                 │
│  💥 BROWSER CRASHES / INTERNET DROPS / PHONE    │
│     RINGS AND YOU NAVIGATE AWAY                 │
│                                                 │
│  ⚠️ ALL DATA IS LOST! Start from scratch! 😤    │
└─────────────────────────────────────────────────┘
```

```
WITH DRAFT:
┌─────────────────────────────────────────────────┐
│  Create Purchase Order                          │
│                                                 │
│  PO Number: [PO-001       ]  💾 Auto-saved!    │
│  Supplier:  [TechCorp     ]  💾 Auto-saved!    │
│  Date:      [2026-06-15   ]  💾 Auto-saved!    │
│  ...                                            │
│                                                 │
│  💥 BROWSER CRASHES                             │
│                                                 │
│  ✅ Come back later → Your work is still there! │
│  ✅ Continue exactly where you left off          │
│  ✅ Nothing is lost!                             │
└─────────────────────────────────────────────────┘
```

---

### What is Draft Handling?

**Draft** = A temporary, auto-saved version of your work that hasn't been officially submitted yet.

Think of it like writing an email:
- **Without draft:** Type the entire email → if browser crashes → email is gone
- **With draft:** Gmail auto-saves every few seconds → crash → open Gmail → draft is there!

**In CAP/Fiori:**
- Draft = temporary save to the database (in a special draft table)
- The real table is only updated when the user clicks "Save" (Activate)
- Until then, other users can't see your in-progress work

---

### Draft vs Non-Draft: Visual Comparison

```
┌────────────────────────────────────────────────────────────────┐
│                   NON-DRAFT (Direct Save)                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  User fills form → clicks Save → IMMEDIATELY saved to DB      │
│                                                                │
│  Problems:                                                     │
│  • No auto-save (lose work on crash)                          │
│  • Can't pause and come back later                            │
│  • Incomplete records may be saved to DB                      │
│  • Other users see half-finished data immediately             │
│  • No "undo" after save                                       │
│                                                                │
├────────────────────────────────────────────────────────────────┤
│                   WITH DRAFT (Two-Step Save)                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  User fills form → auto-saved to DRAFT table → user clicks    │
│  "Save" → moved to REAL table (activated)                     │
│                                                                │
│  Benefits:                                                     │
│  • Auto-save every few seconds (never lose work)              │
│  • Can pause, close browser, come back tomorrow               │
│  • Incomplete data only in draft (not polluting real table)   │
│  • Other users don't see drafts                               │
│  • Easy to discard and start over                             │
│  • Exclusive editing (lock prevents two users editing same)   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

### Real-World Examples of Draft

| Application | Draft Use Case |
|-------------|---------------|
| Gmail | Email drafts (auto-saved, not sent yet) |
| Google Docs | Every keystroke is a "draft" (auto-saved) |
| Purchase Orders | Fill in complex PO over multiple sessions |
| Leave Requests | Start a request, come back to finish later |
| Blog posts | Write over several days before publishing |
| SAP Sales Orders | Complex orders with many items, multiple reviewers |

---

### How Draft Works Technically

```
┌──────────────┐         ┌──────────────────┐         ┌──────────────┐
│  User fills  │ ──────► │  DRAFT TABLE     │ ──────► │  REAL TABLE  │
│  form fields │  Auto   │  (temporary)     │  User   │  (permanent) │
│              │  Save   │                  │  clicks │              │
│              │  (every │  Only the user   │  SAVE   │  Everyone    │
│              │  change)│  can see this    │ (Activate)│ can see     │
└──────────────┘         └──────────────────┘         └──────────────┘
                              │
                              │ User clicks DISCARD
                              ▼
                         🗑️ Draft deleted
                         (nothing saved to real table)
```

**Under the hood, CAP creates:**
- A shadow `_drafts` table for each draft-enabled entity
- Records move between draft table ↔ real table based on user actions

---

### The Draft Lifecycle

```
                    ┌───────────────────────────────────────┐
                    │         DRAFT LIFECYCLE                │
                    └───────────────────────────────────────┘

  ┌─────────┐         ┌──────────┐         ┌──────────────┐
  │  START  │ ──NEW──►│  DRAFT   │──SAVE───►│   ACTIVE     │
  │         │         │(editing) │(activate)│  (persisted) │
  └─────────┘         └────┬─────┘         └──────┬───────┘
                           │                       │
                           │ DISCARD               │ EDIT
                           ▼                       ▼
                      🗑️ Deleted              ┌──────────┐
                      (back to start)         │  DRAFT   │
                                              │(editing) │──SAVE──► back to ACTIVE
                                              └────┬─────┘
                                                   │ DISCARD
                                                   ▼
                                              Reverts to last saved ACTIVE state
```

**Four key operations:**
1. **New (Create Draft)** → Start filling in a new record
2. **Edit (Create Draft from Active)** → Lock an existing record for editing
3. **Activate (Save)** → Move draft to the real table (commit)
4. **Discard** → Throw away the draft (changes lost)

---

### Draft Locking — One Editor at a Time

When User A starts editing an order, it's LOCKED:

```
User A clicks "Edit" on Order SO-001:
  → Draft created
  → Order SO-001 is now LOCKED by User A

User B opens the same Order SO-001:
  → Sees: "This order is being edited by User A"
  → Cannot edit (read-only mode)
  → Can choose: "Take Over Draft" (steals the lock)

User A clicks "Save" or "Discard":
  → Lock is released
  → Order is available for others to edit
```

This prevents the classic problem of two people editing the same record and overwriting each other's changes.

---

## Session 2: Enabling Draft in CAP & The Draft Lifecycle (10:45 - 12:00)

### Enabling Draft — It's ONE WORD!

In your CDS service definition, add `with draft` to your entity:

```cds
// srv/purchasing-service.cds

service PurchasingService @(path: '/purchasing') {
  
  entity PurchaseOrders as projection on db.PurchaseOrders
    with draft;     // ← THIS IS ALL YOU NEED! 🎉

  entity PurchaseOrderItems as projection on db.PurchaseOrderItems;
}
```

**That's it!** One keyword. CAP now:
- Creates a `PURCHASEORDERS_DRAFTS` table automatically
- Adds draft-related OData actions (EditAction, ActivateAction, etc.)
- Handles auto-save on every field change
- Manages locking/unlocking
- Handles discard cleanup

---

### What Happens When You Add `with draft`

```
BEFORE (without draft):
┌─────────────────────────────────────┐
│  Database Tables:                   │
│  • PurchaseOrders (real data only)  │
└─────────────────────────────────────┘

AFTER (with draft):
┌─────────────────────────────────────┐
│  Database Tables:                   │
│  • PurchaseOrders (real data)       │
│  • PurchaseOrders_drafts (temp!)    │
│                                     │
│  OData Actions (auto-added):        │
│  • draftPrepare                     │
│  • draftActivate (= Save)          │
│  • draftEdit (= Enter edit mode)   │
│                                     │
│  Fiori Elements knows how to:       │
│  • Show Create/Edit buttons         │
│  • Auto-save on field change        │
│  • Show "Unsaved changes" indicator │
│  • Handle Save/Discard flow         │
└─────────────────────────────────────┘
```

---

### Draft for Composition Children

If your parent has `with draft`, the children are AUTOMATICALLY included in the draft:

```cds
service PurchasingService {
  entity PurchaseOrders as projection on db.PurchaseOrders
    with draft;    // Parent has draft

  // Children don't need "with draft" — they're included automatically!
  entity PurchaseOrderItems as projection on db.PurchaseOrderItems;
}
```

When a user creates a PO and adds items, BOTH the PO and its items are stored in the draft table until "Save" is clicked.

---

### The User Experience with Draft

Here's what the user sees step by step:

#### Creating a New Record:

```
1. User clicks [+ Create] on List Report
   → New Object Page opens in EDIT mode
   → A draft record is created immediately (empty)

2. User types "PO-001" in the PO Number field
   → 💾 Auto-saved to draft table (every field change!)
   → User sees "Draft" indicator

3. User fills more fields, adds items
   → Each change auto-saves to draft
   → User can close browser safely!

4. User opens the app again later
   → Sees their draft in the list (with "Draft" badge)
   → Clicks it → continues editing!

5. User clicks [Save]
   → Draft is "activated" → moved to real table
   → Other users can now see the PO
   → Draft badge disappears

   OR

5. User clicks [Discard Draft]
   → Draft is deleted
   → Nothing saved to real table
   → Clean slate
```

---

#### Editing an Existing Record:

```
1. User clicks [Edit] on an existing PO (Object Page)
   → A draft is created from the current data
   → Object Page switches to edit mode
   → Record is LOCKED (others see "Locked by User A")

2. User changes the priority field
   → 💾 Change saved to draft (original unchanged!)

3. User clicks [Save]
   → Draft changes applied to the real record
   → Lock released

   OR

3. User clicks [Cancel]
   → Draft discarded
   → Original record unchanged
   → Lock released
```

---

### Draft Indicators in the UI

```
List Report with drafts:
┌─────────────────────────────────────────────────────────────┐
│ PO Number   │ Supplier     │ Amount  │ Status    │ State    │
├─────────────┼──────────────┼─────────┼───────────┼──────────┤
│ PO-001      │ TechCorp     │ $4,500  │ Approved  │          │ ← Active (saved)
│ PO-002      │ OfficeMax    │ $1,200  │ Draft     │ 📝 Draft │ ← Your draft!
│ PO-003      │ CloudInc     │ $8,900  │ Submitted │ 🔒 Locked│ ← Being edited by someone
└─────────────────────────────────────────────────────────────┘
```

**Visual indicators:**
- No badge = Active (saved, normal record)
- 📝 "Draft" badge = Your unsaved draft
- 🔒 "Locked" indicator = Someone else is editing
- ⚠️ "Unsaved Changes" banner = On the Object Page when in edit mode

---

### Important Draft Rules

| Rule | Detail |
|------|--------|
| Auto-save | Every field change triggers a save to the draft table |
| Exclusive lock | Only one user can edit a record at a time |
| Draft timeout | Drafts expire after a configurable period (default: none in dev) |
| Validation | Full validation runs on "Save" (Activate), not on every keystroke |
| Cancel = Revert | Cancelling an edit reverts to the last activated state |
| Draft list | Drafts appear in the List Report with a badge |
| No partial data in production | Real table only gets complete, validated data |

---

### `@odata.draft.enabled` — The Annotation Behind the Scenes

When you write `with draft`, CAP adds this annotation to `$metadata`:

```xml
<Annotation Term="Common.DraftRoot">
  <Record>
    <PropertyValue Property="ActivationAction" String="...draftActivate"/>
    <PropertyValue Property="EditAction" String="...draftEdit"/>
    <PropertyValue Property="PreparationAction" String="...draftPrepare"/>
  </Record>
</Annotation>
```

Fiori Elements reads this and knows: "This entity supports draft! I should show Edit/Save/Discard buttons."

---

## Session 3: Hands-on — Enable Draft for PO Management (12:00 - 13:00)

### Step 1: Add `with draft` to Your Service

Update your purchasing service:

```cds
// srv/purchasing-service.cds
using { com.epm as db } from '../db/schema';

service PurchasingService @(path: '/purchasing') {

  entity PurchaseOrders as projection on db.PurchaseOrders
    with draft
    actions {
      action submit() returns { status: String; message: String; };
      action approve(comment: String) returns { status: String; message: String; };
      action reject(reason: String) returns { status: String; message: String; };
    };

  entity PurchaseOrderItems as projection on db.PurchaseOrderItems;
  @readonly entity Suppliers as projection on db.Suppliers;
  @readonly entity Products as projection on db.Products;
}
```

---

### Step 2: Restart and Observe

```bash
cds watch
```

Check the console output:
```
[cds] - serving PurchasingService { path: '/purchasing', impl: 'srv/purchasing-service.js' }
[cds] - drafted entity: PurchasingService.PurchaseOrders
```

CAP confirms: `PurchaseOrders` is now draft-enabled!

---

### Step 3: Test the Draft Flow in the Browser

Open your Fiori Elements app and try:

**Test 1: Create with Draft**
1. Click **[+ Create]** → Empty form opens (edit mode)
2. Fill in PO Number: "PO-TEST-001"
3. Notice: No "Save" needed yet! Data is auto-saving to draft
4. Close the browser tab (simulate crash)
5. Re-open the app → You should see "PO-TEST-001" with a Draft badge!
6. Click it → Continue editing
7. Fill remaining fields → Click **[Save]** → Draft becomes active

**Test 2: Edit with Draft**
1. Click on an existing PO in the list
2. Click **[Edit]** → Object Page switches to edit mode
3. Change the priority to "High"
4. Notice the "Unsaved Changes" indicator
5. Click **[Save]** → Change committed
6. OR Click **[Cancel]** → Change reverted to original

**Test 3: Discard Draft**
1. Click **[+ Create]** → Start a new PO
2. Fill in some fields
3. Click **[Discard Draft]** or navigate away and confirm discard
4. The draft is deleted — nothing saved

---

### Step 4: Check the Database

While a draft exists, run:
```bash
sqlite3 db/data.db "SELECT * FROM PurchasingService_PurchaseOrders_drafts;"
```

You'll see your in-progress data in the draft table!

After saving (activating):
```bash
sqlite3 db/data.db "SELECT * FROM com_epm_PurchaseOrders WHERE poNumber = 'PO-TEST-001';"
```

Now it's in the real table.

---

### Draft Handler Adjustments

When you enable draft, your `before` handlers run at **activation time** (when user clicks Save), NOT on every auto-save. This is important!

```javascript
// This runs when the user clicks SAVE (not on every keystroke!)
this.before('CREATE', 'PurchaseOrders', (req) => {
  // Validation runs here — at activation time
  if (!req.data.supplier_ID) req.error(400, 'Supplier is required');
});
```

If you want validation during draft editing (before activation), use:
```javascript
// Runs during draft preparation (before save attempt)
this.before('SAVE', 'PurchaseOrders', (req) => {
  // This runs when user clicks Save but BEFORE activation
  if (!req.data.poNumber) req.error(400, 'PO Number is required');
});
```

---

## Session 4: Actions, Side Effects & Dynamic Expressions (13:45 - 14:45)

### Actions in Draft-Enabled Entities

When your entity has `with draft`, bound actions work differently:

```cds
entity PurchaseOrders as projection on db.PurchaseOrders
  with draft
  actions {
    action submit() returns { status: String; message: String; };
    action approve(comment: String) returns { status: String; message: String; };
    action reject(reason: String) returns { status: String; message: String; };
  };
```

**Important:** Bound actions can only be called on **active (saved) records** — not on drafts!

```
User creates PO → fills fields → clicks SAVE → PO is now ACTIVE
                                                    │
                                                    ▼
                                             NOW actions work!
                                             [Submit] [Approve] [Reject]
```

Why? Because actions change business state — you shouldn't approve something that hasn't been saved yet.

---

### Action Buttons with Confirmation Dialogs

You can make actions show a confirmation dialog before executing:

**In annotations:**
```cds
annotate PurchasingService.reject with @(
  Common.IsActionCritical: true   // Shows "Are you sure?" dialog
);
```

For actions that need user input (like reject reason):
```cds
// The parameter automatically becomes a dialog field!
action reject(reason: String(500)) returns { status: String; };
```

When the user clicks "Reject", Fiori shows:
```
┌──────────────────────────────────┐
│  Reject Purchase Order           │
│                                  │
│  Reason: [____________________ ] │
│                                  │
│        [Cancel]  [Reject]        │
└──────────────────────────────────┘
```

The `reason` parameter becomes an input field in the dialog automatically!

---

### Displaying Actions Conditionally

You only want to show "Approve" when the PO is in "Pending" status. Use `@Core.OperationAvailable`:

```cds
// Define virtual (computed) fields for action availability:
annotate PurchasingService.PurchaseOrders with {
  // These need to be virtual or computed fields
  submitEnabled  : Boolean @Core.Computed @UI.Hidden;
  approveEnabled : Boolean @Core.Computed @UI.Hidden;
  rejectEnabled  : Boolean @Core.Computed @UI.Hidden;
};

// Link actions to their availability field:
annotate PurchasingService.submit with @Core.OperationAvailable: submitEnabled;
annotate PurchasingService.approve with @Core.OperationAvailable: approveEnabled;
annotate PurchasingService.reject with @Core.OperationAvailable: rejectEnabled;
```

**Handler computes the values:**
```javascript
this.after('READ', 'PurchaseOrders', (results) => {
  const pos = Array.isArray(results) ? results : [results];
  for (const po of pos) {
    po.submitEnabled = po.status === 'Draft';
    po.approveEnabled = po.status === 'Pending';
    po.rejectEnabled = po.status === 'Pending';
  }
});
```

**Result:**
- When status = "Draft" → only "Submit" button is visible
- When status = "Pending" → "Approve" and "Reject" are visible
- When status = "Approved" → none of the workflow buttons shown

---

### Side Effects — Auto-Refresh When Fields Change

**Problem:** User changes the quantity of an order item. The line total and order total should recalculate automatically without reloading the page.

**Solution:** Side Effects tell Fiori: "When THIS field changes, refresh THOSE fields."

```cds
annotate PurchasingService.PurchaseOrderItems with @(
  Common.SideEffects: {
    SourceProperties: ['quantity', 'unitPrice'],    // When these change...
    TargetProperties: ['totalPrice']                // ...refresh this
  }
);

annotate PurchasingService.PurchaseOrders with @(
  Common.SideEffects#ItemsChanged: {
    SourceEntities: ['items'],                      // When items change...
    TargetProperties: ['totalAmount']               // ...refresh the total
  }
);
```

**How it works:**
1. User changes `quantity` from 5 to 10
2. Fiori sees the Side Effect: "quantity changed → refresh totalPrice"
3. Fiori sends a PATCH to save the draft (with new quantity)
4. Then re-reads the entity to get the updated `totalPrice`
5. UI refreshes with new calculated value

**Your handler must calculate it:**
```javascript
this.before('SAVE', 'PurchaseOrders', async (req) => {
  const { items } = req.data;
  if (items) {
    let total = 0;
    for (const item of items) {
      item.totalPrice = item.quantity * item.unitPrice;
      total += item.totalPrice;
    }
    req.data.totalAmount = total;
  }
});
```

---

### Dynamic Field Control — Show/Hide/ReadOnly Based on State

You can make fields editable or read-only based on the record's state:

```cds
annotate PurchasingService.PurchaseOrders with {
  // PO Number: editable only in Draft, read-only after submission
  poNumber @Common.FieldControl: poNumberControl;

  // Status: always read-only (system-managed)
  status @Common.FieldControl: #ReadOnly;

  // Supplier: editable only in Draft
  supplier @Common.FieldControl: supplierControl;
};
```

**Handler computes the control values:**
```javascript
this.after('READ', 'PurchaseOrders', (results) => {
  const pos = Array.isArray(results) ? results : [results];
  for (const po of pos) {
    // #ReadOnly = 1, #Optional = 3, #Mandatory = 7
    if (po.status === 'Draft') {
      po.poNumberControl = 3;    // Editable (Optional)
      po.supplierControl = 7;    // Editable + Required (Mandatory)
    } else {
      po.poNumberControl = 1;    // Read-only (after submission)
      po.supplierControl = 1;    // Read-only
    }
  }
});
```

**Field Control Values:**

| Value | Constant | Meaning |
|-------|----------|---------|
| 0 | `#Hidden` | Field is completely hidden |
| 1 | `#ReadOnly` | Field is visible but not editable |
| 3 | `#Optional` | Field is editable (optional) |
| 7 | `#Mandatory` | Field is editable and required (shows *) |

---

### Dynamic Expressions — Annotation-Level Logic

Instead of computing in handlers, you can use expressions directly in annotations:

```cds
// Hide "Rejection Note" field unless status is Rejected
annotate PurchasingService.PurchaseOrders with {
  rejectionNote @UI.Hidden: {
    $edmJson: { $Ne: [{ $Path: 'status' }, 'Rejected'] }
  };
};
```

This says: "Hide the rejectionNote field when status is NOT 'Rejected'." (Show it only when rejected.)

**Common expression patterns:**

```cds
// Show field only when another field has a specific value
field @UI.Hidden: { $edmJson: { $Ne: [{ $Path: 'status' }, 'Rejected'] } };

// Equivalent: Show only when status = 'Rejected'
// (Hidden when status != 'Rejected')
```

---

## Session 5: Hands-on — Complete CRUD & Actions in the UI (15:00 - 16:00)

### Exercise 1: Test All CRUD Operations (20 minutes)

With draft enabled and your app running, test each operation:

**CREATE:**
1. List Report → Click [+ Create]
2. Fill in: PO Number, Supplier (use Value Help!), Priority, Expected Date
3. Add 2-3 items (Product + Quantity + Price)
4. Click [Save] → Verify it appears in the list

**READ:**
1. Click any row → Object Page shows full details
2. Verify all sections and fields display correctly
3. Check that the items table shows correctly

**UPDATE:**
1. Click [Edit] on an existing PO
2. Change the priority
3. Add another item
4. Click [Save] → Verify changes persisted

**DELETE:**
1. Select a PO in the list (checkbox)
2. Click [Delete] → Confirm in dialog
3. Verify it's gone from the list

---

### Exercise 2: Add Action Buttons (20 minutes)

Make sure your annotations include actions:

```cds
annotate PurchasingService.PurchaseOrders with @UI.Identification: [
  { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.submit', Label: 'Submit for Approval' },
  { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.approve', Label: 'Approve' },
  { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.reject', Label: 'Reject' }
];
```

**Test the flow:**
1. Create a new PO → Save it (status = "Draft")
2. Click [Submit for Approval] → status changes to "Pending"
3. Click [Approve] → status changes to "Approved"

Test rejection:
1. Create another PO → Save → Submit
2. Click [Reject] → Dialog appears asking for reason
3. Enter reason → Click Reject → Status = "Rejected"

---

### Exercise 3: Configure Side Effects (20 minutes)

Add side effects so totals recalculate:

```cds
// When item quantity or price changes → recalculate line total
annotate PurchasingService.PurchaseOrderItems with @(
  Common.SideEffects: {
    SourceProperties: ['quantity', 'unitPrice'],
    TargetProperties: ['totalPrice']
  }
);

// When items change → recalculate PO total
annotate PurchasingService.PurchaseOrders with @(
  Common.SideEffects#TotalRefresh: {
    SourceEntities: ['items'],
    TargetProperties: ['totalAmount']
  }
);
```

**Test:**
1. Create a new PO → go to edit mode
2. Add an item: Product = Laptop, Quantity = 3, Unit Price = 999
3. After typing quantity → `totalPrice` should auto-update to 2997
4. The order's `totalAmount` should also refresh

---

## Session 6: End-to-End PO Creation Flow (16:00 - 16:45)

### The Complete User Journey

Here's the full flow a user experiences when creating a PO in your app:

```
Step 1: Open the app
┌─────────────────────────────────────────────┐
│ Purchase Orders (12)            [+ Create]  │
│ ┌─────────────────────────────────────────┐ │
│ │ PO-001 │ TechCorp │ $4,500  │ Approved │ │
│ │ PO-002 │ OfficeMax│ $1,200  │ Pending  │ │
│ └─────────────────────────────────────────┘ │
│                                             │
│ User clicks [+ Create]                      │
└─────────────────────────────────────────────┘
            │
            ▼
Step 2: Fill in the form (Object Page in edit mode)
┌─────────────────────────────────────────────┐
│ ◀ Back    New Purchase Order         [Save] │
│                                    [Discard]│
│ § General Information                       │
│ ┌─────────────────────────────────────────┐ │
│ │ PO Number:  [PO-2026-013    ]           │ │
│ │ Supplier:   [▼ TechCorp Supplies ]←VH!  │ │
│ │ Priority:   [▼ High           ]         │ │
│ │ Expected:   [📅 2026-06-20    ]         │ │
│ │ Notes:      [Urgent restock   ]         │ │
│ └─────────────────────────────────────────┘ │
│                                             │
│ § Line Items                     [+ Add]    │
│ ┌─────────────────────────────────────────┐ │
│ │ Product      │ Qty │ Price  │ Total     │ │
│ │ Laptop Pro   │ 5   │ $800   │ $4,000   │ │
│ │ Mouse        │ 20  │ $25    │ $500     │ │
│ └─────────────────────────────────────────┘ │
│ TOTAL: $4,500                               │
│                                             │
│ User clicks [Save]                          │
└─────────────────────────────────────────────┘
            │
            ▼
Step 3: PO is saved (Active state)
┌─────────────────────────────────────────────┐
│ ◀ Back    PO-2026-013     [Submit] [Edit]   │
│           TechCorp Supplies                 │
│                                             │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐    │
│ │ $4,500   │ │ Draft    │ │ High     │    │
│ │ Amount   │ │ Status   │ │ Priority │    │
│ └──────────┘ └──────────┘ └──────────┘    │
│                                             │
│ § Details (read-only now)                   │
│ § Line Items (table, read-only)             │
│                                             │
│ User clicks [Submit for Approval]           │
└─────────────────────────────────────────────┘
            │
            ▼
Step 4: Status changes to "Pending"
┌─────────────────────────────────────────────┐
│  ✅ "Purchase Order submitted successfully" │
│                                             │
│  Status: ⏳ Pending Approval                │
│  [Submit] button disappears                 │
│  [Approve] [Reject] appear (for managers)   │
└─────────────────────────────────────────────┘
```

---

### Complete Service + Annotations for the PO App

Here's the full working configuration:

**`srv/purchasing-service.cds`:**
```cds
using { com.epm as db } from '../db/schema';

service PurchasingService @(path: '/purchasing') {

  entity PurchaseOrders as projection on db.PurchaseOrders
    with draft
    actions {
      action submit() returns { status: String; message: String; };
      action approve(comment: String(500)) returns { status: String; message: String; };
      @Common.IsActionCritical
      action reject(reason: String(500)) returns { status: String; message: String; };
    };

  entity PurchaseOrderItems as projection on db.PurchaseOrderItems;
  @readonly entity Suppliers as projection on db.Suppliers;
  @readonly entity Products as projection on db.Products;
}
```

**Key annotations summary (from Days 22-24):**
```cds
// All annotations in one consolidated view:
annotate PurchasingService.PurchaseOrders with @UI: {
  SelectionFields: [ poNumber, status, priority, supplier_ID ],
  LineItem: [...],
  HeaderInfo: { Title: { Value: poNumber }, ... },
  HeaderFacets: [...],
  Facets: [...],
  Identification: [
    { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.submit', Label: 'Submit' },
    { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.approve', Label: 'Approve' },
    { $Type: 'UI.DataFieldForAction', Action: 'PurchasingService.reject', Label: 'Reject' }
  ]
};
```

---

### Validation Summary: When Does What Run?

| User Action | What Fires | Example |
|-------------|-----------|---------|
| Types in a field | Draft auto-save (PATCH to draft) | Saves partial data |
| Clicks Save | `before('SAVE')` + `before('CREATE')` or `before('UPDATE')` | Full validation |
| Clicks an Action | `this.on('actionName')` handler | Business logic |
| Clicks Delete | `before('DELETE')` handler | Dependency check |
| Clicks Cancel/Discard | Draft is deleted (no handler needed) | Cleanup automatic |

---

## Assessment: MCQ — 15 Questions on Draft & CRUD in Fiori Elements

**Q1.** What keyword enables draft handling on a CAP service entity?

- A) `@odata.draft.enabled`
- B) `with draft`
- C) `draft: true`
- D) `@draft`

**Answer: B** — `entity X as projection on db.X with draft;` — that's all you need in CDS.

---

**Q2.** What does draft handling primarily protect against?

- A) SQL injection attacks
- B) Data loss when users close the browser before saving
- C) Unauthorized access
- D) Database corruption

**Answer: B** — Draft auto-saves work-in-progress to a temporary table. Users can close the browser and resume later.

---

**Q3.** Where is draft data stored before the user clicks "Save"?

- A) In the browser's localStorage
- B) In a separate `_drafts` table in the database
- C) In memory on the server
- D) In a cookie

**Answer: B** — CAP creates a shadow `_drafts` table. Draft data lives in the database (persistent) but separate from the real table.

---

**Q4.** When user A starts editing a record, what happens to user B who opens the same record?

- A) Nothing — both can edit simultaneously
- B) User B sees the record as locked/read-only (exclusive editing)
- C) User B's changes automatically merge with User A's
- D) The server crashes

**Answer: B** — Draft uses exclusive locking. Only one user can edit at a time. Others see a "Locked by User A" indicator.

---

**Q5.** "Activate" in draft terminology means:

- A) Starting to edit a record
- B) Moving draft data to the real table (committing/saving)
- C) Locking the record
- D) Deleting the draft

**Answer: B** — Activate = Save. Draft data moves from the drafts table to the real production table.

---

**Q6.** "Discard" in draft terminology means:

- A) Saving the record
- B) Deleting the draft without saving (changes are lost)
- C) Locking the record
- D) Publishing the record

**Answer: B** — Discard deletes the draft. For new records: nothing is saved. For edits: reverts to the last saved state.

---

**Q7.** When you add `with draft` to a parent entity with Composition children, what happens to the children?

- A) Children also need `with draft` added individually
- B) Children are automatically included in the draft lifecycle
- C) Children don't support draft at all
- D) Children are saved to the real table immediately (no draft)

**Answer: B** — Composition children are automatically part of the parent's draft. Adding items to a draft PO stores them in the drafts table too.

---

**Q8.** `before('SAVE', 'Entity', handler)` in draft mode runs when?

- A) On every auto-save (every field change)
- B) When the user clicks the final "Save" button (activation)
- C) When the user opens the edit form
- D) When the user discards the draft

**Answer: B** — `SAVE` event fires at activation time (user clicks Save). Auto-saves during editing don't trigger this — they go directly to the draft table.

---

**Q9.** Bound actions (like approve, reject) can be called on:

- A) Only draft records (before saving)
- B) Only active/saved records (after saving)
- C) Both draft and active records
- D) Only deleted records

**Answer: B** — Bound actions work on active (saved) records only. You must Save first, then Submit/Approve/Reject.

---

**Q10.** `@Common.IsActionCritical` does what?

- A) Marks the action as dangerous (requires admin role)
- B) Shows a confirmation dialog ("Are you sure?") before executing the action
- C) Highlights the button in red
- D) Logs the action to an audit trail

**Answer: B** — Critical actions show a confirmation dialog. Useful for destructive operations like Delete or Reject.

---

**Q11.** Side Effects (`@Common.SideEffects`) are used for:

- A) Sending email notifications
- B) Telling Fiori to refresh certain fields when other fields change
- C) Triggering server-side validations
- D) Logging field changes

**Answer: B** — Side Effects tell the UI: "When field X changes, re-read field Y from the server" (e.g., recalculate totals).

---

**Q12.** `@Common.FieldControl: #ReadOnly` makes a field:

- A) Hidden
- B) Visible but not editable (greyed out in edit mode)
- C) Required
- D) Deleted

**Answer: B** — `#ReadOnly` (value 1) shows the field but prevents editing. Users can see it but not change it.

---

**Q13.** The Field Control values are:

- A) 0=Hidden, 1=ReadOnly, 3=Optional, 7=Mandatory
- B) 0=ReadOnly, 1=Optional, 2=Required, 3=Hidden
- C) 1=Visible, 2=Editable, 3=Required
- D) true=Editable, false=ReadOnly

**Answer: A** — 0=Hidden, 1=ReadOnly, 3=Optional(Editable), 7=Mandatory(Editable+Required).

---

**Q14.** Action parameters (like `reject(reason: String)`) appear in the UI as:

- A) Hidden fields
- B) A dialog with input fields that the user fills before confirming the action
- C) URL parameters
- D) Tooltip text

**Answer: B** — Action parameters become input fields in a dialog popup. User fills them → clicks the action button → handler receives the values.

---

**Q15.** `@Core.OperationAvailable: submitEnabled` does what?

- A) Makes the entity available for OData operations
- B) Shows/hides an action button based on the value of the `submitEnabled` field
- C) Enables the Submit functionality in the service
- D) Marks the operation as available in $metadata

**Answer: B** — When `submitEnabled` is `true`, the button is shown. When `false`, it's hidden. Computed in the handler based on record state.

---

## Assignment: Complete End-to-End Purchase Order Application

### Task

Build a fully functional Purchase Order application with draft, CRUD, actions, and side effects.

### Deliverables

1. **Service definition** with `with draft` and all actions
2. **Annotations file** with:
   - Complete List Report (filters, table, criticality)
   - Complete Object Page (header, KPIs, sections, items table)
   - Value Helps for supplier and product
   - Action buttons (Submit, Approve, Reject)
   - Side Effects for total recalculation
3. **Handler file** with:
   - Validation on Save (required fields, business rules)
   - Criticality computation
   - Action implementations (with status checks)
   - Total calculation logic
4. **Test it** by performing this complete flow:
   - Create a PO (with items) using the Fiori UI
   - Save it (activate from draft)
   - Submit for approval
   - Approve it
   - Verify status changes and button visibility

### Verification Checklist

- [ ] `cds watch` starts without errors
- [ ] App shows List Report with filter bar and table
- [ ] Create button opens edit form (draft mode)
- [ ] Supplier field shows Value Help dropdown
- [ ] Items can be added/removed in the items table
- [ ] Auto-save works (close browser, reopen → draft still there)
- [ ] Save validates required fields
- [ ] After Save: Submit button appears (status = Draft)
- [ ] After Submit: Approve/Reject buttons appear (status = Pending)
- [ ] Reject shows confirmation dialog with reason input
- [ ] After Approve: no more action buttons (status = Approved)
- [ ] Totals recalculate when item quantities change (Side Effects)
- [ ] Cannot edit PO fields after submission (FieldControl = ReadOnly)

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Draft | Auto-saves work-in-progress; prevents data loss |
| `with draft` | Single keyword enables the entire draft infrastructure |
| Draft lifecycle | New → Edit → Activate (Save) or Discard |
| Exclusive locking | Only one user can edit at a time |
| Validation timing | Full validation runs on Save (activation), not every keystroke |
| Actions + Draft | Actions only work on active (saved) records |
| `@Common.IsActionCritical` | Shows confirmation dialog before action executes |
| Action parameters | Become dialog input fields automatically |
| `@Core.OperationAvailable` | Shows/hides buttons based on computed state |
| Side Effects | Auto-refresh fields when other fields change |
| Field Control | Dynamic read-only/editable/hidden/mandatory based on state |
| Dynamic expressions | Annotation-level show/hide logic |

### The Draft Equation

```
with draft  +  Annotations  +  Handlers  =  Complete Enterprise Edit Experience
(auto-save)   (UI layout)    (business     (that rivals SAP S/4HANA apps!)
                              rules)
```

### The One-Liner

> **Draft handling means your users NEVER lose their work — and your database ONLY gets validated, complete data.**

---

### Looking Ahead: Day 25

Tomorrow: **UI Layer Review & Integration Day**
- Recap of Days 21-24 (Fiori, annotations, draft)
- Weekly quiz on the UI layer
- Mini project: Complete EPM Fiori application
- Connecting all three layers (DB + Service + UI)

---

*End of Day 24 — Your app now has auto-save, smart buttons, and a professional editing experience!*
