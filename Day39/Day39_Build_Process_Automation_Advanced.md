# Day 39: SAP Build Process Automation — Advanced Concepts & CAP Integration

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 38 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: Advanced Workflow Patterns — Parallel & Multi-Level | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Decision Tables, Business Rules & Conditions | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Build Multi-Level Approval with Decisions | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Automations (RPA), Actions & API Integration | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Complete CAP-to-Workflow Integration | 60 min |
| 16:00 - 16:45 | Session 6: Real-World Patterns & Best Practices | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Design advanced workflow patterns: parallel approvals, multi-level chains, loops
- Build decision tables with complex rule logic (multiple conditions, priority handling)
- Understand Automations (RPA bots) and when to use them
- Configure Actions (API calls) within workflows for system integration
- Implement the full CAP-to-Workflow round-trip (trigger → approve → callback → status update)
- Apply real-world patterns for common enterprise scenarios
- Handle edge cases: timeouts, rejections, resubmissions, delegation

---

## Day 38 Recap — Quick Fire (09:00 - 09:15)

1. SBPA provides three capabilities: _____, _____, _____? → _____
2. A workflow "instance" is _____? → _____
3. "My Inbox" is where _____ see _____? → _____
4. To trigger a workflow from CAP, you call _____? → _____
5. A Decision Table automates _____? → _____
6. After the manager approves, your CAP app learns via _____? → _____
7. Workflow status "Erroneous" means _____? → _____
8. Escalation happens when _____? → _____

<details>
<summary>Answers</summary>

1. **Workflows**, **Decisions**, **Automations** (RPA)
2. A **running execution** of a workflow definition (a specific PO being approved)
3. **Approvers** see their **pending tasks** (and act: approve/reject)
4. The **Workflow REST API** (POST /v1/workflow-instances)
5. **Routing logic** without human intervention (if amount > X → Y approver)
6. **Callback** (workflow calls your CAP API) or **polling** (CAP checks workflow status)
7. An **error occurred** during execution (API failed, invalid data, etc.)
8. The task **isn't completed within a deadline** → reassigned to higher authority

</details>

---

## Session 1: Advanced Workflow Patterns (09:15 - 10:30)

### Beyond Simple Approval: Real-World Complexity

Simple workflows (Submit → Approve → Done) are rare in real companies. Real processes have:
- Multiple approval levels
- Parallel approvals (two people must approve)
- Conditional branches (different paths based on data)
- Loops (rejected → fix → resubmit → approve again)
- Escalations and delegations

---

### Pattern 1: Multi-Level (Sequential) Approval

When a PO needs approval from MULTIPLE people IN ORDER:

```
┌───────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌─────┐
│ Start │───►│ Manager  │───►│ Director │───►│  CFO     │───►│ End │
│       │    │ Approve  │    │ Approve  │    │ Approve  │    │     │
└───────┘    └────┬─────┘    └────┬─────┘    └────┬─────┘    └─────┘
                  │               │               │
                  │ Reject        │ Reject        │ Reject
                  ▼               ▼               ▼
             ┌─────────────────────────────────────────┐
             │  Notify Requester: "PO Rejected"         │
             │  Reason: {rejector's comment}            │
             └─────────────────────────────────────────┘
```

**When to use:** Large purchases ($50K+), sensitive operations, compliance requirements.

**Implementation:**
```
Process Builder:
  Start (API) → Approval Step 1 (Manager)
    ├── Approved → Approval Step 2 (Director)
    │                ├── Approved → Approval Step 3 (CFO)
    │                │                ├── Approved → End (Approved!)
    │                │                └── Rejected → Notify → End
    │                └── Rejected → Notify → End
    └── Rejected → Notify → End
```

**Key configuration:** Each approval step has:
- Different recipient (Manager → Director → CFO)
- Different deadline (3 days → 5 days → 7 days)
- Different escalation target

---

### Pattern 2: Parallel (Simultaneous) Approval

When MULTIPLE people must approve AT THE SAME TIME (all must agree):

```
                         ┌──────────────┐
                    ┌───►│ IT Manager   │───┐
                    │    │ Approve      │   │
┌───────┐    ┌─────┴─┐  └──────────────┘   │   ┌────────────┐    ┌─────┐
│ Start │───►│Parallel│                     ├──►│   All      │───►│ End │
│       │    │Gateway │                     │   │ Approved?  │    │     │
└───────┘    └─────┬──┘  ┌──────────────┐   │   └──────┬─────┘    └─────┘
                    │    │ Finance Mgr  │   │          │
                    └───►│ Approve      │───┘          │ Any rejected
                         └──────────────┘              ▼
                                                 ┌─────────┐
                                                 │Rejected │
                                                 └─────────┘
```

**When to use:** Cross-functional approvals (IT equipment needs IT approval AND budget approval).

**Configuration in Process Builder:**
1. Add a **Parallel Gateway** (split)
2. Add approval tasks on each branch
3. Add a **Parallel Gateway** (join) — waits for ALL branches
4. Join condition: "All must complete" (or "First to respond wins")

---

### Pattern 3: Conditional Routing (Different Paths)

Different approvers based on data:

```
                         ┌───── Category = "IT" ─────► IT Manager Approves
                         │
┌───────┐    ┌──────────┼───── Category = "Travel" ──► Travel Manager Approves
│ Start │───►│ Decision │
└───────┘    │ Gateway  ├───── Category = "Office" ──► Office Admin Approves
             │          │
             └──────────┼───── Amount > $10,000 ─────► Director (+ whoever above)
                         │
                         └───── Default ──────────────► General Manager
```

**When to use:** Different departments handle different categories, different amount thresholds route differently.

---

### Pattern 4: Rejection Loop (Resubmit After Fix)

When a PO is rejected, the requester can fix it and submit again WITHOUT starting a brand new workflow:

```
                                          ┌──────────────────────────────┐
                                          │                              │
┌───────┐    ┌──────────┐    ┌───────────┼────┐    ┌───────────────┐   │
│ Start │───►│ Manager  │───►│ Approved? │    │───►│ End (Approved)│   │
│       │    │ Reviews  │    │           │ NO │    └───────────────┘   │
└───────┘    └──────────┘    └───────────┼────┘                        │
                                         │                              │
                                         ▼                              │
                                   ┌───────────┐                        │
                                   │ Requester │   User fixes → Resubmit│
                                   │ Fixes PO  │────────────────────────┘
                                   │ (Task)    │
                                   └─────┬─────┘
                                         │ Cancel
                                         ▼
                                   ┌───────────┐
                                   │End (Cancel)│
                                   └───────────┘
```

**When to use:** Most approval scenarios! Don't force users to create a new PO just because of a typo.

**Implementation:**
- After rejection → create a "Revision Task" assigned to the requester
- Requester can modify context data (fix amount, change supplier)
- After fix → loop back to the approval step
- Option to cancel (give up)

---

### Pattern 5: Delegation

When the assigned approver is on vacation or unavailable:

```
Manager receives task → "I can't handle this" → Delegates to:
  ├── Specific person: "Send to Deputy Manager John"
  ├── Role-based: "Send to anyone in Finance Managers group"
  └── Automatic: (if no action in 2 days → auto-delegate to backup)
```

**Configuration:**
- In the Approval step settings → enable "Allow Delegation"
- Set auto-delegation rule: "After X days without action → forward to {backup person}"

---

### Comparing the Patterns

| Pattern | Steps | When to Use | Complexity |
|---------|-------|-------------|-----------|
| Single Approval | 1 | Small amounts, simple decisions | Low |
| Multi-Level | 2-4 | Large amounts, compliance needs | Medium |
| Parallel | 2+ simultaneous | Cross-functional (IT + Finance) | Medium |
| Conditional | Variable | Different categories/departments | Medium |
| Rejection Loop | Variable | Allow corrections without restart | High |
| Delegation | 1 + forward | Approver unavailability | Low (built-in) |

---

## Session 2: Decision Tables, Business Rules & Conditions (10:45 - 12:00)

### Decision Tables — Deep Dive

A Decision Table is like a spreadsheet of IF-THEN rules. It's the "brain" that routes your workflow.

---

### Anatomy of a Decision Table

```
┌─────────────────────────────────────────────────────────────────────────┐
│  DECISION TABLE: "Determine Approval Route"                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  INPUT COLUMNS (Conditions):        OUTPUT COLUMNS (Results):            │
│  ┌──────────┬──────────┬─────────┐  ┌───────────┬──────────┬────────┐  │
│  │  Amount  │ Category │Priority │  │  Approver │  Levels  │AutoAppr│  │
│  ├──────────┼──────────┼─────────┤  ├───────────┼──────────┼────────┤  │
│  │ <= 500   │   ANY    │  ANY    │  │ System    │    0     │  TRUE  │  │
│  │ 501-2000 │   ANY    │  ANY    │  │ Manager   │    1     │ FALSE  │  │
│  │ 2001-5000│   IT     │  ANY    │  │ IT Mgr    │    1     │ FALSE  │  │
│  │ 2001-5000│  Other   │  ANY    │  │ Manager   │    1     │ FALSE  │  │
│  │ 5001-50K │   ANY    │ Normal  │  │ Director  │    2     │ FALSE  │  │
│  │ 5001-50K │   ANY    │ Urgent  │  │ VP        │    2     │ FALSE  │  │
│  │  > 50000 │   ANY    │  ANY    │  │ CFO       │    3     │ FALSE  │  │
│  └──────────┴──────────┴─────────┘  └───────────┴──────────┴────────┘  │
│                                                                         │
│  Rules are evaluated TOP-TO-BOTTOM. First match wins!                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Decision Table Input Types

| Input Type | Operators Available | Example |
|-----------|-------------------|---------|
| **Number** | =, !=, <, >, <=, >=, between | `amount > 5000` |
| **String** | =, !=, contains, starts with | `category = 'IT'` |
| **Boolean** | true, false | `isUrgent = true` |
| **Date** | before, after, between | `requestDate after 2026-01-01` |
| **Enum/List** | in, not in | `status in ['New', 'Draft']` |

---

### Decision Table Output Types

| Output | Purpose | Example |
|--------|---------|---------|
| **Approver** | Who should approve | `"finance_managers_group"` |
| **Levels** | How many approval steps | `2` (Manager + Director) |
| **AutoApprove** | Skip human task? | `true` (for small amounts) |
| **Deadline** | How long to wait | `3` (days) |
| **Priority** | Urgency of the task | `"high"` |

---

### Building a Decision in the Process Builder

**Step 1:** Add a "Decision" step to your workflow

**Step 2:** Define Input Parameters (mapped from workflow context):
```
Input Variables:
  amount     : Number    (from context.amount)
  category   : String    (from context.category)
  priority   : String    (from context.priority)
  department : String    (from context.department)
```

**Step 3:** Define Output Parameters (used in subsequent steps):
```
Output Variables:
  approverGroup : String    (who should approve)
  approvalLevel : Number    (how many levels)
  autoApprove   : Boolean   (skip human approval?)
  deadline      : Number    (days until escalation)
```

**Step 4:** Build the Decision Table (fill in rules):
```
Rule 1: amount <= 500                    → approverGroup: "system", autoApprove: true
Rule 2: amount <= 5000 AND category = IT → approverGroup: "it_managers"
Rule 3: amount <= 5000                   → approverGroup: "department_managers"
Rule 4: amount <= 50000                  → approverGroup: "directors"
Rule 5: amount > 50000                   → approverGroup: "cfo_office"
```

**Step 5:** Use outputs in subsequent workflow steps:
- Approval Task recipients = `${decisionOutput.approverGroup}`
- Deadline = `${decisionOutput.deadline}` days

---

### Complex Decision: Multiple Outputs Based on Combined Conditions

```
┌───────────────────────────────────────────────────────────────────────────┐
│  Rule │ Amount    │ Category │ Priority │ → Approver      │ → Level │ → SLA │
├───────┼───────────┼──────────┼──────────┼─────────────────┼─────────┼───────┤
│  R1   │ ≤ 500    │ ANY      │ ANY      │ AUTO            │    0    │  0d   │
│  R2   │ ≤ 5000   │ IT       │ ANY      │ IT_Manager      │    1    │  3d   │
│  R3   │ ≤ 5000   │ Travel   │ ANY      │ Travel_Admin    │    1    │  2d   │
│  R4   │ ≤ 5000   │ Other    │ Normal   │ Line_Manager    │    1    │  3d   │
│  R5   │ ≤ 5000   │ Other    │ Urgent   │ Line_Manager    │    1    │  1d   │
│  R6   │ 5001-50K │ ANY      │ Normal   │ Director        │    2    │  5d   │
│  R7   │ 5001-50K │ ANY      │ Urgent   │ VP              │    2    │  2d   │
│  R8   │ > 50K    │ ANY      │ ANY      │ CFO + Director  │    3    │  7d   │
└───────┴───────────┴──────────┴──────────┴─────────────────┴─────────┴───────┘

Note: "SLA" = deadline for the approver to respond
```

---

### Using Decision Output in the Workflow

After the Decision step, use its output to control the flow:

```
Process Builder:

  Start → Decision Step → Condition Gateway:
                              │
                              ├── autoApprove = true → Skip to End (approved)
                              │
                              ├── approvalLevel = 1 → Single Approval Task
                              │                        (recipient: decisionOutput.approverGroup)
                              │
                              ├── approvalLevel = 2 → Sequential:
                              │                        Task 1 (Manager) → Task 2 (Director)
                              │
                              └── approvalLevel = 3 → Sequential:
                                                       Task 1 → Task 2 → Task 3 (CFO)
```

---

## Session 3: Hands-on — Build Multi-Level Approval with Decisions (12:00 - 13:00)

### Exercise: Complete PO Approval Process

Build a workflow that handles ALL amount tiers:

---

**Step 1: Create the Decision Table**

In your PO Approval project:
1. Add a **Decision** step after the Start event
2. Name it: "Determine Approval Route"
3. Define inputs: `amount` (Number), `category` (String), `priority` (String)
4. Define outputs: `approverGroup` (String), `autoApprove` (Boolean), `levels` (Number)
5. Add rules:

| # | If Amount | And Category | And Priority | Then Approver | Levels | Auto? |
|---|-----------|-------------|-------------|---------------|--------|-------|
| 1 | ≤ 500 | ANY | ANY | System | 0 | TRUE |
| 2 | 501 - 5000 | = "IT" | ANY | IT_Managers | 1 | FALSE |
| 3 | 501 - 5000 | ≠ "IT" | ANY | Department_Mgr | 1 | FALSE |
| 4 | 5001 - 50000 | ANY | ≠ "Urgent" | Director | 2 | FALSE |
| 5 | 5001 - 50000 | ANY | = "Urgent" | VP | 2 | FALSE |
| 6 | > 50000 | ANY | ANY | CFO | 3 | FALSE |

---

**Step 2: Add Condition Gateway**

After the Decision step:
- Branch 1: If `autoApprove = true` → End (Approved)
- Branch 2: If `levels = 1` → Single Approval Step → End
- Branch 3: If `levels >= 2` → Manager Approval → Director Approval → End

---

**Step 3: Configure Each Approval Step**

For the single-level approval (Branch 2):
- Subject: `[${priority}] Approve PO ${poNumber} — $${amount}`
- Recipients: Use `${approverGroup}` from decision output
- Deadline: 3 days
- Form: Display PO details + comment field

For the two-level approval (Branch 3):
- Step 1: Manager approval (same as above)
- Step 2 (only after Step 1 approves): Director approval
  - Subject includes: "Previously approved by Manager"
  - Show Manager's approval comment

---

**Step 4: Handle Rejection at Any Level**

Add a condition after EACH approval step:
- If Approved → continue to next step (or End)
- If Rejected → Notification to requester → End (Rejected)

---

**Step 5: Test with Different Amounts**

| Test | Amount | Expected Path |
|------|--------|---------------|
| Test 1 | $200 | Auto-approved (no human task) |
| Test 2 | $3,000 + Category "IT" | IT Manager approval (1 level) |
| Test 3 | $3,000 + Category "Office" | Department Manager (1 level) |
| Test 4 | $15,000 | Manager → Director (2 levels) |
| Test 5 | $75,000 | Manager → Director → CFO (3 levels) |

---

## Session 4: Automations (RPA), Actions & API Integration (13:45 - 14:45)

### What Are Automations?

**Automations** are software bots (RPA — Robotic Process Automation) that mimic human actions on a computer. They:
- Click buttons
- Fill forms
- Copy data between systems
- Download/upload files
- Read emails and extract information

```
┌─────────────────────────────────────────────────────────────┐
│         AUTOMATION EXAMPLE: Invoice Processing               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Bot does (automatically, no human!):                       │
│  1. Opens email inbox                                       │
│  2. Finds emails with PDF attachments                       │
│  3. Downloads the PDF invoice                               │
│  4. Reads the invoice (OCR — text extraction)               │
│  5. Extracts: vendor, amount, date, invoice number          │
│  6. Opens SAP system                                        │
│  7. Creates a purchase invoice with extracted data           │
│  8. Attaches the original PDF                               │
│  9. Sends confirmation email to the sender                  │
│                                                             │
│  TIME SAVED: 15 minutes per invoice × 50 invoices/day       │
│  = 12.5 hours saved daily! 🤖                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### When to Use Automations vs Workflows vs CAP Logic

| Need | Use | Why |
|------|-----|-----|
| Human must make a decision | **Workflow** | Requires judgment, context, approval |
| Rules can decide automatically | **Decision** | No human needed, deterministic logic |
| Repetitive screen-based work | **Automation (RPA)** | Bot mimics human clicks/typing |
| Data CRUD with business logic | **CAP Handler** | Code-based logic, APIs |
| Data transformation between systems | **Automation OR CAP** | Depends on complexity |
| Reading unstructured data (PDFs, emails) | **Automation** | OCR, text parsing capabilities |

---

### Actions — API Calls Within Workflows

An **Action** is a step in your workflow that calls an external API. This is how the workflow communicates back to your CAP app (or any other system).

```
┌─────────────────────────────────────────────────────────────┐
│  ACTION STEP IN WORKFLOW                                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Name: "Update PO Status in CAP"                            │
│                                                             │
│  Type: REST API Call                                        │
│  Method: POST                                               │
│  URL: https://po-mgmt-srv.cfapps.../purchasing/             │
│       PurchaseOrders(${context.poId})/PurchasingService.     │
│       updateFromWorkflow                                    │
│                                                             │
│  Headers:                                                   │
│    Content-Type: application/json                           │
│    Authorization: Bearer ${serviceToken}                    │
│                                                             │
│  Body:                                                      │
│    {                                                        │
│      "status": "${approvalResult}",                         │
│      "approvedBy": "${approverName}",                       │
│      "comment": "${approverComment}"                        │
│    }                                                        │
│                                                             │
│  On Success: Continue workflow                               │
│  On Error: Set error flag → notify admin                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Action Types Available

| Action Type | What It Does | Use Case |
|-------------|-------------|----------|
| **REST API** | Call any HTTP endpoint | Update CAP app, call external service |
| **OData** | Call an OData service | Read/write S/4HANA data |
| **SAP Ariba** | Pre-built Ariba connector | Create requisition in Ariba |
| **SAP SuccessFactors** | Pre-built SF connector | Update employee record |
| **Mail** | Send email notification | Inform stakeholders |
| **Custom** | Your own bot/script | Complex data transformation |

---

### Configuring the Callback Action to CAP

After approval completes, the workflow should update your PO:

**In the workflow:**
1. Add an **Action** step after the final approval
2. Configure it as a REST API call
3. URL: Your CAP backend endpoint
4. Body: Include approval result + approver info

**In your CAP app (receiver handler):**

```javascript
// srv/purchasing-service.js — handle callback from workflow
module.exports = async function () {

  // This action is called BY the workflow (not by a user!)
  this.on('updateFromWorkflow', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const { status, approvedBy, comment } = req.data;

    const { PurchaseOrders } = cds.entities;

    await UPDATE(PurchaseOrders).set({
      status: status === 'approved' ? 'Approved' : 'Rejected',
      approvedBy: approvedBy,
      approverComment: comment,
      approvedAt: new Date().toISOString()
    }).where({ ID });

    // Emit event for downstream processing
    if (status === 'approved') {
      await this.emit('POApproved', { poId: ID, approver: approvedBy });
    }

    return { success: true };
  });
};
```

**CDS definition:**
```cds
entity PurchaseOrders as projection on db.PurchaseOrders
  actions {
    action updateFromWorkflow(status: String; approvedBy: String; comment: String)
      returns { success: Boolean; };
  };
```

---

### The Complete Round-Trip: CAP → Workflow → CAP

```
┌─────────────────────────────────────────────────────────────┐
│              THE COMPLETE INTEGRATION FLOW                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. User in Fiori App clicks [Submit PO]                    │
│     └── CAP handler: update status → call Workflow API      │
│                                                             │
│  2. Workflow starts (instance created)                       │
│     └── Decision Table evaluates → determines approver      │
│                                                             │
│  3. Approval Task appears in Manager's Inbox                │
│     └── Manager reviews form → clicks [Approve]             │
│                                                             │
│  4. Workflow continues after approval                        │
│     └── Action step: calls CAP API (updateFromWorkflow)     │
│                                                             │
│  5. CAP handler receives callback                           │
│     └── Updates PO status to "Approved"                     │
│     └── Emits "POApproved" event                            │
│                                                             │
│  6. User refreshes Fiori App                                │
│     └── Sees PO status = "Approved" ✅                      │
│                                                             │
│  TOTAL TIME: Minutes (if manager is quick) to 3 days (SLA)  │
│  ZERO manual status updates! Everything automatic! 🎉       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Session 5: Hands-on — Complete CAP-to-Workflow Integration (15:00 - 16:00)

### Exercise: Full Round-Trip Implementation

---

**Part A: CAP Side — Trigger the Workflow (20 minutes)**

```javascript
// srv/purchasing-service.js

const cds = require('@sap/cds');

module.exports = async function () {
  const { PurchaseOrders, PurchaseOrderItems } = cds.entities;

  // ═══ SUBMIT ACTION: Triggers the workflow ═══
  this.on('submit', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];

    const po = await SELECT.one.from(PurchaseOrders).where({ ID });
    if (!po) req.reject(404, 'PO not found');
    if (po.status !== 'Draft') req.reject(400, 'Only Draft POs can be submitted');

    // Get items for context
    const items = await SELECT.from(PurchaseOrderItems).where({ po_ID: ID });

    // Update status locally
    await UPDATE(PurchaseOrders).set({ status: 'Pending' }).where({ ID });

    // Trigger the workflow
    try {
      const workflowResult = await startApprovalWorkflow({
        poId: po.ID,
        poNumber: po.poNumber,
        amount: po.totalAmount,
        supplierName: po.supplierName || 'Unknown',
        priority: po.priority || 'Medium',
        category: po.category || 'General',
        requesterName: req.user.id,
        requesterEmail: req.user.id,
        itemCount: items.length,
        items: JSON.stringify(items.map(i => ({
          product: i.productName,
          qty: i.quantity,
          price: i.unitPrice
        })))
      });

      console.log(`Workflow started: ${workflowResult.id}`);

      // Store workflow instance ID for tracking
      await UPDATE(PurchaseOrders).set({
        workflowInstanceId: workflowResult.id
      }).where({ ID });

    } catch (error) {
      console.error('Workflow trigger failed:', error.message);
      // Don't fail the submit — mark as pending for manual processing
      req.warn(200, 'PO submitted but automatic approval routing failed. Will be processed manually.');
    }

    return {
      status: 'Pending',
      message: `PO ${po.poNumber} submitted for approval ($${po.totalAmount})`
    };
  });

  // ═══ CALLBACK: Workflow calls this after approval decision ═══
  this.on('workflowCallback', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const { decision, approverName, approverComment, approvedAt } = req.data;

    const newStatus = decision === 'approve' ? 'Approved' : 'Rejected';

    await UPDATE(PurchaseOrders).set({
      status: newStatus,
      approvedBy: approverName,
      approverComment: approverComment || '',
      approvedAt: approvedAt || new Date().toISOString()
    }).where({ ID });

    // Emit event for further processing
    await this.emit(decision === 'approve' ? 'POApproved' : 'PORejected', {
      poId: ID,
      approver: approverName,
      comment: approverComment
    });

    return { status: newStatus, message: `PO ${newStatus.toLowerCase()} by ${approverName}` };
  });

  // ═══ CHECK STATUS: User can manually check workflow progress ═══
  this.on('checkApprovalStatus', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const po = await SELECT.one.from(PurchaseOrders).where({ ID });

    if (po.status !== 'Pending') {
      return { status: po.status, message: `PO is ${po.status}` };
    }

    if (!po.workflowInstanceId) {
      return { status: 'Pending', message: 'Awaiting manual processing' };
    }

    // Could poll workflow API here for detailed status
    return {
      status: 'Pending',
      workflowId: po.workflowInstanceId,
      message: 'Awaiting approval in workflow system'
    };
  });
};

// ═══ HELPER: Start the workflow via API ═══
async function startApprovalWorkflow(contextData) {
  // In production: use cds.connect.to('workflow')
  // For now: simulate the API call structure

  const workflowPayload = {
    definitionId: 'po_approval_workflow',
    context: contextData
  };

  // Production code:
  // const wfService = await cds.connect.to('workflow');
  // return await wfService.send('POST', '/v1/workflow-instances', workflowPayload);

  // Mock response for testing:
  console.log('WORKFLOW TRIGGERED:', JSON.stringify(workflowPayload, null, 2));
  return { id: `wf-${Date.now()}`, status: 'RUNNING' };
}
```

---

**Part B: CDS Definition Updates (10 minutes)**

```cds
// srv/purchasing-service.cds — add workflow-related fields and actions
entity PurchaseOrders as projection on db.PurchaseOrders
  with draft
  actions {
    action submit() returns { status: String; message: String; };
    action workflowCallback(
      decision: String;
      approverName: String;
      approverComment: String;
      approvedAt: DateTime
    ) returns { status: String; message: String; };
    function checkApprovalStatus() returns {
      status: String; workflowId: String; message: String;
    };
  };
```

Add to schema:
```cds
// db/schema.cds — add workflow tracking field
entity PurchaseOrders : cuid, managed {
  // ... existing fields ...
  workflowInstanceId : String(50);  // Track which workflow instance handles this PO
  approvedBy         : String(100);
  approverComment    : String(500);
  approvedAt         : DateTime;
}
```

---

**Part C: Test the Flow (30 minutes)**

```http
### 1. Create a PO
POST http://localhost:4004/purchasing/PurchaseOrders
Content-Type: application/json
Authorization: Basic bWFuYWdlcjptYW5hZ2Vy

{
  "poNumber": "PO-WF-001",
  "totalAmount": 7500,
  "supplierName": "TechCorp",
  "priority": "High",
  "category": "IT",
  "status": "Draft"
}

### 2. Submit (triggers workflow)
POST http://localhost:4004/purchasing/PurchaseOrders(PO-ID)/PurchasingService.submit
Content-Type: application/json
Authorization: Basic bWFuYWdlcjptYW5hZ2Vy

{}

### 3. Check status (while waiting for approval)
GET http://localhost:4004/purchasing/PurchaseOrders(PO-ID)/PurchasingService.checkApprovalStatus()
Authorization: Basic bWFuYWdlcjptYW5hZ2Vy

### 4. Simulate workflow callback (in real system, workflow calls this)
POST http://localhost:4004/purchasing/PurchaseOrders(PO-ID)/PurchasingService.workflowCallback
Content-Type: application/json
Authorization: Basic YWRtaW46YWRtaW4=

{
  "decision": "approve",
  "approverName": "director@company.com",
  "approverComment": "Approved — within IT budget for Q2",
  "approvedAt": "2026-06-05T14:30:00Z"
}

### 5. Verify final status
GET http://localhost:4004/purchasing/PurchaseOrders(PO-ID)?$select=poNumber,status,approvedBy,approverComment
Authorization: Basic bWFuYWdlcjptYW5hZ2Vy
```

---

## Session 6: Real-World Patterns & Best Practices (16:00 - 16:45)

### Best Practices for Workflow Design

| # | Practice | Why |
|---|---------|-----|
| 1 | Keep workflows SHORT (3-7 steps max) | Long workflows are hard to debug and maintain |
| 2 | Use Decisions to REDUCE human tasks | Auto-approve trivial cases (saves everyone time) |
| 3 | Always set DEADLINES | Without them, tasks sit forever |
| 4 | Always handle REJECTION | Let requesters know WHY and give them options |
| 5 | Include ESCALATION for critical items | Priority tasks shouldn't wait 5 days |
| 6 | Store workflow ID in your entity | Enables status tracking and correlation |
| 7 | Handle workflow API failures gracefully | If trigger fails, don't block the user |
| 8 | Use meaningful task SUBJECTS | "Approve PO $5K from Sarah" not "Task #4823" |
| 9 | Test with DIFFERENT amounts | Verify each decision rule triggers correctly |
| 10 | Design for RESUBMISSION | Most rejections need corrections, not cancellation |

---

### Common Enterprise Workflow Scenarios

| Scenario | Steps | Key Features |
|----------|-------|-------------|
| **Purchase Order Approval** | Submit → Decide → Approve (1-3 levels) → Notify | Amount-based routing, multi-level |
| **Leave Request** | Apply → Manager Approve → HR Validate → Deduct Balance | Calendar check, balance validation |
| **Invoice Processing** | Receive → Extract (RPA) → Match to PO → Approve Discrepancy → Post | Automation + human exception handling |
| **Employee Onboarding** | HR Creates → IT Setup (parallel) → Manager Welcome → Training Assigned | Parallel tasks, multiple departments |
| **Budget Change Request** | Request → Finance Review → Director Approve → Update Budget | Validation + approval + system update |
| **Contract Renewal** | Expiry Alert → Legal Review → Negotiation Task → Final Approval → Sign | Time-triggered start, document review |

---

### Anti-Patterns (What NOT to Do)

| Anti-Pattern | Problem | Better Approach |
|-------------|---------|----------------|
| 20+ steps in one workflow | Unmaintainable, fragile | Split into sub-processes |
| No deadlines on tasks | Tasks forgotten forever | Always set SLA + escalation |
| Hardcoded approver emails | Breaks when people leave | Use role-based groups |
| No error handling on API calls | Workflow gets stuck | Add error boundary + retry |
| Workflow for every small change | Over-automation fatigue | Only for significant decisions |
| Same form for all approvers | VP doesn't need details line-staff needs | Customize form per level |

---

## Assessment: MCQ — 15 Questions on Advanced SBPA

**Q1.** A parallel gateway in a workflow means:

- A) Two steps run one after another
- B) Multiple steps run simultaneously (all must complete before continuing)
- C) The workflow runs faster
- D) Only one path is chosen

**Answer: B** — Parallel = simultaneous execution. All branches must complete before the join gateway.

---

**Q2.** In a rejection loop pattern, after rejection the requester can:

- A) Only start a completely new workflow
- B) Fix the data and resubmit within the SAME workflow instance (loop back)
- C) Directly approve it themselves
- D) Delete the workflow

**Answer: B** — The loop allows fixing and resubmitting without creating a new instance.

---

**Q3.** A Decision Table evaluates rules:

- A) Randomly
- B) Top-to-bottom; first matching rule's output is used
- C) Bottom-to-top
- D) All rules simultaneously (outputs merged)

**Answer: B** — First match wins. Order your rules from most specific to most general.

---

**Q4.** If a Decision Table has output `autoApprove = true`, the workflow should:

- A) Still send a task to a human
- B) Skip the approval task entirely and mark as approved (no human involved)
- C) Delete the request
- D) Send to two approvers

**Answer: B** — Auto-approve = skip the human task. The system decides automatically for trivial cases.

---

**Q5.** An "Action" step in a workflow is:

- A) A human task
- B) An automated API call to an external system (like calling your CAP backend)
- C) A form display
- D) A delay/wait step

**Answer: B** — Actions call APIs. Use them for callbacks, data updates, notifications via system integrations.

---

**Q6.** Delegation in a workflow allows:

- A) Rejecting a task
- B) Forwarding a task to another person (when the original approver is unavailable)
- C) Deleting the workflow
- D) Changing the amount

**Answer: B** — Delegation = "I can't handle this, forward to my backup/deputy."

---

**Q7.** Storing `workflowInstanceId` in your PO entity is important because:

- A) It's required by the database
- B) It enables tracking which workflow handles which PO (correlation)
- C) It makes the PO run faster
- D) It's for UI display only

**Answer: B** — Correlation: link your business object to its workflow instance for status tracking and debugging.

---

**Q8.** If the workflow API call fails when submitting a PO, the BEST practice is:

- A) Crash the entire submit operation
- B) Log the error, mark PO as "Pending" anyway, allow manual routing as fallback
- C) Silently ignore it
- D) Delete the PO

**Answer: B** — Graceful degradation. The PO is submitted (status=Pending), but route manually if automation fails.

---

**Q9.** RPA (Robotic Process Automation) is best used for:

- A) Making business decisions
- B) Repetitive screen-based tasks (copy data between systems, read PDFs, fill forms)
- C) Database design
- D) User authentication

**Answer: B** — RPA bots mimic human clicks/typing. Best for repetitive, rule-based screen work.

---

**Q10.** Escalation in a workflow:

- A) Increases the PO amount
- B) Auto-reassigns a task to a higher authority after a deadline passes
- C) Sends a thank-you message
- D) Cancels the workflow

**Answer: B** — Escalation ensures tasks don't sit forever. After X days → bump to next level.

---

**Q11.** For a $300 office supply purchase, the BEST workflow approach is:

- A) 3-level approval (Manager → Director → CFO)
- B) Auto-approve (Decision Table: amount ≤ $500 → no human task needed)
- C) Reject (too small to process)
- D) Parallel approval by 4 departments

**Answer: B** — Use decisions to auto-approve trivial amounts. Don't waste executives' time on $300 orders.

---

**Q12.** The workflow callback to CAP should update:

- A) Only the UI
- B) The PO status, approver info, comment, and timestamp in the database
- C) The xs-security.json
- D) The mta.yaml

**Answer: B** — Callback updates the business entity with the decision result so the Fiori app reflects the new status.

---

**Q13.** Multi-level approval (Manager → Director → CFO) is implemented as:

- A) Three separate workflows
- B) Sequential approval steps within ONE workflow (each waits for the previous)
- C) One approval task with three buttons
- D) Parallel approvals

**Answer: B** — Sequential: Step 1 must complete before Step 2 starts. All in one workflow instance.

---

**Q14.** A workflow form should show:

- A) Every field from the database
- B) Only the relevant information needed to make the approval decision (concise!)
- C) The approver's personal details
- D) Source code

**Answer: B** — Show what matters: amount, supplier, requester, reason, items. Don't overwhelm with irrelevant data.

---

**Q15.** The relationship between CAP and SBPA is:

- A) They're competitors
- B) CAP handles data/logic; SBPA handles process orchestration + human decisions. They complement each other.
- C) SBPA replaces CAP
- D) They can't work together

**Answer: B** — Perfect partnership: CAP = data + APIs + business logic. SBPA = routing + approvals + tracking.

---

## Assignment: Design Complete Multi-Department Workflow

### Task

Design and document a comprehensive PO approval workflow that handles a real enterprise scenario.

### Scenario Requirements

Your company has these rules:
1. **All POs under $200** → auto-approved (no human)
2. **$200 - $5,000** → Department Manager approves
3. **$5,000 - $25,000** → Department Manager + Finance Controller (sequential)
4. **Over $25,000** → Department Manager + Finance Controller + CEO (sequential)
5. **Category = "IT" AND Amount > $1,000** → IT Manager ALSO approves (parallel with Dept Manager)
6. **Priority = "Urgent"** → SLA reduced by 50% at each level
7. **If rejected at any level** → requester gets a task to fix and resubmit OR cancel
8. **If not acted on within SLA** → escalate to the next level automatically

### Deliverables

1. **Flow diagram** (draw all paths, branches, and loops)
2. **Decision table** (complete with all rules and outputs)
3. **Form design** (what the approver sees — list the fields)
4. **Escalation table** (who escalates to whom, after how long)
5. **CAP integration code** (trigger function + callback handler)
6. **Test matrix** (what amount triggers which path — verify 8+ scenarios)

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Multi-level approval | Sequential steps: Manager → Director → CFO |
| Parallel approval | Simultaneous tasks that ALL must complete |
| Rejection loop | Fix → resubmit within same workflow instance |
| Decision tables | Complex rules: amount × category × priority → routing |
| Actions (API calls) | Workflow calls CAP backend to update status |
| Automations (RPA) | Bots for repetitive screen tasks (PDF reading, data entry) |
| Full round-trip | CAP triggers → Workflow decides → Approver acts → Callback updates CAP |
| Graceful failure | If workflow trigger fails, don't block the user |
| Escalation | Auto-reassign after deadline (don't let tasks rot) |
| Delegation | Forward task when unavailable |

### The Master Integration Pattern

```
USER → Fiori App → CAP (submit) → Workflow API → Decision → Task → Approve
                                                                      │
USER ← Fiori App ← CAP (status updated) ← Callback (Action step) ←──┘
```

### The One-Liner

> **Advanced SBPA turns complex approval chains into visual, trackable, escalating processes — while your CAP app stays focused on data and business logic.**

---

### Looking Ahead: Day 40

Tomorrow: **Week 8 Review & Practice Day**
- Complete review of Weeks 7-8 (Deploy, S/4HANA, Process Automation)
- Weekly quiz (30 questions)
- Integration mini-project: Deploy + Integrate + Automate

---

*End of Day 39 — You can now design enterprise-grade approval workflows! 🏗️*
