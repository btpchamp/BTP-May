# Day 38: SAP Build Process Automation

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Day 37 Recap & Quick Fire | 15 min |
| 09:15 - 10:30 | Session 1: What is SAP Build Process Automation? | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Workflow Design — Forms, Decisions & Automations | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Subscribe & Create Your First Workflow | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Integration with CAP Applications | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Hands-on — Build PO Approval Workflow | 60 min |
| 16:00 - 16:45 | Session 6: Monitoring & Testing Workflows | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain what SAP Build Process Automation is and its role in business processes
- Differentiate between workflows, automations, and decisions
- Design a multi-step approval workflow visually
- Create approval forms that users interact with
- Configure decision logic (auto-approve vs manual approval)
- Understand how to trigger workflows from a CAP application
- Monitor running workflow instances and check their status
- Design a complete PO approval process

---

## Day 37 Recap — Quick Fire (09:00 - 09:15)

1. The 4 integration patterns are _____? → _____
2. Pass-through pattern means _____? → _____
3. Mashup pattern combines _____ data with _____ data? → _____
4. The N+1 problem is solved by _____? → _____
5. If S/4HANA is down during enrichment, you should _____? → _____
6. Virtual fields are stored in _____? → _____
7. In-memory caching with TTL is best for _____? → _____
8. Cloud Connector is needed for _____? → _____

<details>
<summary>Answers</summary>

1. **Pass-Through**, **Mashup**, **Replicate**, **Event-Driven**
2. Forwarding/delegating requests **directly to S/4HANA** (proxy)
3. **Local** data with **remote** (S/4HANA) data
4. **Batch lookup** — collect IDs → one `WHERE IN` call
5. **Return local data with placeholder** (graceful degradation, don't crash)
6. **Nowhere** (not in DB) — computed at runtime in handlers
7. **Frequently accessed reference data** that changes rarely
8. Connecting to **on-premise** systems behind a corporate firewall

</details>

---

## Session 1: What is SAP Build Process Automation? (09:15 - 10:30)

### The Problem: Manual Business Processes

In real companies, many processes require **human decisions** — approvals, reviews, exceptions:

```
❌ WITHOUT PROCESS AUTOMATION:

  Employee creates a PO:
    → Sends email to manager: "Please approve PO-001"
    → Manager forgets (email buried in inbox)
    → Employee sends reminder after 3 days
    → Manager finally approves via reply-all email
    → Employee manually updates the PO status
    → Nobody knows who approved what or when

  Problems:
  • No tracking (where is my request?)
  • No deadlines (sits forever)
  • No audit trail (who approved?)
  • Error-prone (manual status updates)
  • Not scalable (what if 100 POs/day?)
```

```
✅ WITH PROCESS AUTOMATION:

  Employee creates a PO:
    → System automatically sends approval task to manager
    → Manager sees it in their task inbox (My Inbox)
    → Manager reviews details, clicks [Approve] or [Reject]
    → System automatically updates PO status
    → Employee gets notified instantly
    → Full audit trail logged

  Benefits:
  • Automatic routing (right person gets the task)
  • Deadlines & escalations (auto-escalate after 3 days)
  • Full audit trail (who, what, when — everything tracked)
  • Consistent process (same steps every time)
  • Scalable (handles 1 or 10,000 requests)
```

---

### What is SAP Build Process Automation?

**SAP Build Process Automation (SBPA)** is a low-code/no-code tool on SAP BTP for creating business workflows, automations, and decisions.

```
┌─────────────────────────────────────────────────────────────┐
│       SAP BUILD PROCESS AUTOMATION                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  THREE CAPABILITIES:                                        │
│                                                             │
│  1️⃣  WORKFLOWS                                              │
│     Multi-step processes with human tasks                   │
│     "PO needs manager approval → then finance review"      │
│     ┌──────┐   ┌────────┐   ┌──────────┐   ┌──────┐      │
│     │Submit│──►│Approve │──►│ Finance  │──►│ Done │      │
│     │      │   │(Manager)│   │ (Review) │   │      │      │
│     └──────┘   └────────┘   └──────────┘   └──────┘      │
│                                                             │
│  2️⃣  DECISIONS (Business Rules)                             │
│     Automated logic without human intervention              │
│     "If amount < $1000 → auto-approve"                     │
│     "If amount > $5000 → requires VP approval"             │
│                                                             │
│  3️⃣  AUTOMATIONS (RPA — Robotic Process Automation)         │
│     Bots that automate repetitive tasks                    │
│     "Copy data from PDF invoice into SAP system"           │
│     "Download report, format it, email to team"            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Workflows vs Automations vs Decisions

| Component | What It Does | Human Involved? | Example |
|-----------|-------------|----------------|---------|
| **Workflow** | Multi-step process with tasks assigned to people | YES (approval tasks) | PO approval, leave request |
| **Decision** | Rules engine that evaluates conditions automatically | NO (fully automated logic) | Route PO to right approver |
| **Automation** | Bot that mimics human actions on screens/systems | NO (bot does the work) | Copy data between systems |

**Today's focus:** Workflows + Decisions (most relevant for CAP applications).

---

### How Workflow Relates to Your CAP App

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  YOUR CAP APP                    SAP BUILD PROCESS AUTOMATION│
│                                                             │
│  User clicks "Submit PO"         Workflow triggered:         │
│  ┌─────────────────────┐        ┌──────────────────────┐   │
│  │ PO: PO-001          │──API──►│ Start                │   │
│  │ Amount: $5,000      │        │   │                   │   │
│  │ Supplier: TechCorp  │        │   ▼                   │   │
│  │ [Submit] ← clicked! │        │ Decision:             │   │
│  └─────────────────────┘        │  Amount > $1000?      │   │
│                                  │   │ YES               │   │
│                                  │   ▼                   │   │
│                                  │ Task: Manager Approval │   │
│                                  │   │ Approved ✅       │   │
│  PO Status → "Approved"         │   ▼                   │   │
│  ┌─────────────────────┐  ◄─API─│ Callback: approved!   │   │
│  │ PO: PO-001          │        │   │                   │   │
│  │ Status: ✅ Approved  │        │   ▼                   │   │
│  │ Approved by: Manager │        │ End                   │   │
│  └─────────────────────┘        └──────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Key Workflow Concepts

| Concept | Meaning | Analogy |
|---------|---------|---------|
| **Process** | The complete workflow definition (template) | A recipe |
| **Instance** | A running execution of the process | Cooking that recipe right now |
| **Task** | A step assigned to a human | "Review and approve this PO" |
| **Form** | The UI that the approver sees | The approval form with details + buttons |
| **Trigger** | What starts the workflow | API call from your CAP app |
| **Context** | Data passed into the workflow | PO number, amount, supplier name |
| **Decision** | Automated routing/logic step | "If amount > $5000, send to VP" |
| **Notification** | Alert sent to the assignee | Email/inbox notification |
| **My Inbox** | Where users see their pending tasks | The approver's task list |

---

### The Approval Workflow Pattern (Most Common)

```
┌───────┐     ┌───────────┐     ┌──────────┐     ┌────────┐
│ START │────►│  DECISION │────►│  APPROVE │────►│  END   │
│       │     │  (Rules)  │     │  (Task)  │     │        │
└───────┘     └─────┬─────┘     └────┬─────┘     └────────┘
                    │                 │
                    │ Auto-approve    │ Rejected
                    │ (amount < $1K)  │
                    ▼                 ▼
              ┌────────┐       ┌──────────┐
              │  END   │       │ NOTIFY   │──► Requester informed
              │(auto-OK)│      │ (Reject) │
              └────────┘       └──────────┘
```

**Steps:**
1. **Start** → Workflow triggered (data received from CAP app)
2. **Decision** → Check rules (amount threshold, category, etc.)
3. **Approve** → Manager gets a task in their inbox
4. **End** → Status updated, requester notified

---

## Session 2: Workflow Design — Forms, Decisions & Automations (10:45 - 12:00)

### The Process Builder (Visual Editor)

SAP Build Process Automation provides a **drag-and-drop visual editor** for designing workflows:

```
┌──────────────────────────────────────────────────────────────┐
│  SAP Build Process Automation — Process Builder               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │                                                  │        │
│  │  ○ Start ──► □ Decision ──► □ Approval ──► ○ End│        │
│  │              "Route by       "Manager             │        │
│  │               amount"        approves"            │        │
│  │                                                  │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  TOOLBOX (drag onto canvas):                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ Approval │ │ Decision │ │ Mail     │ │ Automation│       │
│  │ Form     │ │ Table    │ │ (Notify) │ │ (Bot)    │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                     │
│  │ Condition│ │ Parallel │ │ API Call │                     │
│  │ (If/Else)│ │ Gateway  │ │          │                     │
│  └──────────┘ └──────────┘ └──────────┘                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### Workflow Components Deep Dive

#### 1. Trigger (Start Event)

How the workflow begins:

```
Trigger Types:
├── API Trigger      ← Most common with CAP! (POST request starts it)
├── Form Trigger     ← User fills a form to start (self-service)
├── Event Trigger    ← Reacts to a business event (S/4HANA event)
└── Schedule Trigger ← Runs at specific times (daily, weekly)
```

For CAP integration, you use the **API Trigger** — your CAP handler calls the workflow API when a PO is submitted.

---

#### 2. Approval Form

The form that the approver sees in their inbox:

```
┌──────────────────────────────────────────────────────────────┐
│  APPROVAL TASK: Purchase Order PO-001                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Request Details:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ PO Number:    PO-2026-001                            │    │
│  │ Requester:    Sarah Johnson                          │    │
│  │ Supplier:     TechCorp International                 │    │
│  │ Amount:       $5,000.00                              │    │
│  │ Priority:     High                                   │    │
│  │ Reason:       Quarterly laptop refresh               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Line Items:                                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Laptop Pro     │ 5  │ $800 each │ $4,000           │    │
│  │ USB-C Hub      │ 5  │ $200 each │ $1,000           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Decision:                                                   │
│  Comment: [________________________________]                 │
│                                                              │
│           [Approve ✅]    [Reject ❌]                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Forms are built visually** — drag-and-drop fields, set them as read-only (display) or editable (input).

---

#### 3. Decision Table (Business Rules)

A decision table defines AUTOMATED routing logic:

```
┌─────────────────────────────────────────────────────────────┐
│  DECISION TABLE: "Determine Approver Level"                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  IF                          THEN                           │
│  ────────────────────────    ────────────────────────       │
│  Amount <= $1,000           → Auto-Approve (no task)       │
│  Amount > $1,000 AND <=5K   → Manager Approval (1 level)   │
│  Amount > $5,000 AND <=50K  → Director Approval (2 levels) │
│  Amount > $50,000           → VP + CFO Approval            │
│                                                             │
│  Priority = "Urgent"        → Skip to Director (escalate)  │
│  Category = "IT Equipment"  → IT Manager must also approve │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Decisions evaluate conditions and set variables that control the workflow path (which branches to take, who gets the task).

---

#### 4. Conditions (Gateways)

Split the workflow into branches based on conditions:

```
                    ┌─── YES ──► Approval Task ──► End (approved)
                    │
○ Start ──► ◆ Decision Gateway: Amount > $1000?
                    │
                    └─── NO ───► Auto-Approve ──► End (auto-approved)
```

---

#### 5. Notifications (Mail/Alert)

Send notifications at any point:
- When task is assigned: "You have a new PO to approve"
- When approved: "Your PO-001 has been approved!"
- When rejected: "Your PO-001 was rejected. Reason: ..."
- When deadline passes: "Reminder: PO-001 awaiting your approval for 3 days"

---

### The My Inbox Experience

Approvers access their tasks through **My Inbox** (a standard SAP Fiori app):

```
┌──────────────────────────────────────────────────────────────┐
│  My Inbox (5 tasks)                              👤 Manager  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ ⏳ Approve PO-001 │ $5,000  │ Sarah Johnson │ 2h ago  │  │
│  │ ⏳ Approve PO-003 │ $12,000 │ Mike Chen     │ 1d ago  │  │
│  │ ⏳ Review Invoice  │ $3,200  │ Lisa Park     │ 3h ago  │  │
│  │ ⏳ Leave Request   │ 5 days  │ John Smith    │ 5h ago  │  │
│  │ ⏳ Budget Change   │ +$10K   │ HR Dept       │ 2d ago  │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Click a task → see details form → Approve/Reject            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Session 3: Hands-on — Subscribe & Create Your First Workflow (12:00 - 13:00)

### Step 1: Subscribe to SAP Build Process Automation

1. **BTP Cockpit** → Service Marketplace
2. Search: "SAP Build Process Automation"
3. Click **Create** → Plan: **Free** (trial) or **Standard**
4. Wait for subscription to activate

---

### Step 2: Assign Roles

1. **Security** → Role Collections
2. Find and assign to your user:
   - `ProcessAutomationAdmin` — full admin access
   - `ProcessAutomationDeveloper` — create/edit workflows
   - `ProcessAutomationParticipant` — receive and complete tasks

---

### Step 3: Open the Application

1. BTP Cockpit → Instances and Subscriptions
2. Find "SAP Build Process Automation" → **"Go to Application"**
3. The **Lobby** opens — this is your process design environment

---

### Step 4: Create a New Project

1. In the Lobby → Click **"Create"** → **"Business Process"**
2. Fill in:
   - Project Name: `PO Approval Workflow`
   - Description: `Approval workflow for purchase orders`
3. Click **Create**

---

### Step 5: Design the Process

1. The **Process Builder** opens with a Start and End node
2. Click the **"+"** between Start and End
3. Add: **"Approval"** → Name it: "Manager Approval"
4. Click on the Approval step to configure:
   - Subject: `Approve PO: ${context.poNumber} - $${context.amount}`
   - Recipients: (configure approver — for testing, use your own email)
   - Due Date: 3 days

---

### Step 6: Configure the Trigger (API)

1. Click on the **Start** node
2. Set Trigger Type: **API**
3. Define **Input Variables** (the data your CAP app will send):

| Variable Name | Type | Description |
|--------------|------|-------------|
| `poNumber` | String | PO Number |
| `amount` | Number | Total PO Amount |
| `supplierName` | String | Supplier Name |
| `requesterName` | String | Who submitted |
| `requesterEmail` | String | Email for notifications |
| `priority` | String | Low/Medium/High |
| `items` | String | JSON of line items |

---

### Step 7: Design the Approval Form

1. Click on "Manager Approval" step
2. Click **"Open Form Editor"**
3. Add display fields:
   - PO Number (read-only, mapped to `poNumber`)
   - Amount (read-only, mapped to `amount`)
   - Supplier (read-only, mapped to `supplierName`)
   - Requester (read-only, mapped to `requesterName`)
   - Priority (read-only, mapped to `priority`)
4. Add decision field:
   - Comment (text area, editable — approver can add notes)

---

### Step 8: Save & Deploy

1. Click **"Save"**
2. Click **"Release"** (version it)
3. Click **"Deploy"** (make it available for execution)

Your workflow is now LIVE and can receive API calls!

---

## Session 4: Integration with CAP Applications (13:45 - 14:45)

### Triggering a Workflow from CAP

When a user clicks "Submit" on a PO in your CAP app, you call the workflow API:

```javascript
// srv/purchasing-service.js

const cds = require('@sap/cds');

module.exports = async function () {

  this.on('submit', 'PurchaseOrders', async (req) => {
    const { ID } = req.params[0];
    const { PurchaseOrders } = cds.entities;

    const po = await SELECT.one.from(PurchaseOrders).where({ ID });
    if (!po) req.reject(404, 'PO not found');
    if (po.status !== 'Draft') req.reject(400, 'Only Draft POs can be submitted');

    // Update PO status
    await UPDATE(PurchaseOrders).set({ status: 'Pending' }).where({ ID });

    // Trigger the workflow!
    try {
      await triggerApprovalWorkflow(po, req.user.id);
    } catch (error) {
      console.error('Failed to trigger workflow:', error.message);
      // Don't fail the submit — workflow can be triggered manually if needed
    }

    return { status: 'Pending', message: `PO ${po.poNumber} submitted for approval.` };
  });
};

// Helper function to call the Workflow API
async function triggerApprovalWorkflow(po, requesterId) {
  const workflowApi = await cds.connect.to('workflow');

  // Start a new workflow instance
  await workflowApi.send('POST', '/v1/workflow-instances', {
    definitionId: 'po_approval_workflow',  // Your workflow's definition ID
    context: {
      poNumber: po.poNumber,
      amount: po.totalAmount,
      supplierName: po.supplierName || 'Unknown',
      requesterName: requesterId,
      requesterEmail: requesterId,
      priority: po.priority || 'Medium',
      poId: po.ID
    }
  });
}
```

---

### Configuring the Workflow Service Connection

**In `package.json`:**

```json
{
  "cds": {
    "requires": {
      "workflow": {
        "kind": "rest",
        "[production]": {
          "credentials": {
            "destination": "WORKFLOW_API",
            "path": "/workflow-service/rest"
          }
        },
        "[development]": {
          "credentials": {
            "url": "https://spa-api-gateway-bpi-us-prod.cfapps.us10.hana.ondemand.com/workflow-service/rest",
            "authentication": "OAuth2",
            "tokenUrl": "https://your-subdomain.authentication.us10.hana.ondemand.com/oauth/token",
            "clientId": "your-client-id",
            "clientSecret": "your-client-secret"
          }
        }
      }
    }
  }
}
```

---

### Receiving the Approval Result Back

After the manager approves or rejects, the workflow needs to UPDATE your PO. Two approaches:

#### Approach 1: Workflow Calls Your CAP API (Callback)

Configure the workflow to call your CAP API after approval:

```
Workflow End Event → API Call Step:
  URL: https://po-management-srv.cfapps.../purchasing/PurchaseOrders({poId})/approve
  Method: POST
  Body: { "comment": "${approvalComment}" }
```

#### Approach 2: Poll Workflow Status (Simpler)

Your CAP app periodically checks workflow status:

```javascript
// Check workflow status (could be triggered by user refresh)
this.on('checkApprovalStatus', 'PurchaseOrders', async (req) => {
  const { ID } = req.params[0];
  const po = await SELECT.one.from(PurchaseOrders).where({ ID });

  if (po.status !== 'Pending') return { status: po.status };

  // Check workflow instance status
  const workflowApi = await cds.connect.to('workflow');
  const instances = await workflowApi.send('GET',
    `/v1/workflow-instances?businessKey=${po.poNumber}&status=COMPLETED`
  );

  if (instances.length > 0) {
    const result = instances[0].context.approvalResult;
    const newStatus = result === 'approve' ? 'Approved' : 'Rejected';
    await UPDATE(PurchaseOrders).set({ status: newStatus }).where({ ID });
    return { status: newStatus };
  }

  return { status: 'Pending', message: 'Awaiting approval' };
});
```

---

### The Complete Integration Flow

```
1. User creates PO in Fiori UI (CAP app)
2. User clicks [Submit]
3. CAP handler:
   ├── Updates status → "Pending"
   └── Calls Workflow API → starts workflow instance

4. Workflow evaluates Decision:
   ├── Amount < $1000 → auto-approve → callback to CAP → status = "Approved"
   └── Amount > $1000 → create approval task

5. Manager opens My Inbox → sees the task
6. Manager reviews form → clicks [Approve] or [Reject]

7. Workflow:
   ├── If approved → callback to CAP API → PO status = "Approved"
   └── If rejected → callback to CAP API → PO status = "Rejected"

8. Requester sees updated status in Fiori app! ✅
```

---

## Session 5: Hands-on — Build PO Approval Workflow (15:00 - 16:00)

### Exercise: Design the Complete PO Approval Process

Build this workflow in the Process Builder:

```
┌───────┐   ┌──────────────┐   ┌────────────────┐   ┌──────────┐   ┌─────┐
│ Start │──►│ Decision:    │──►│ Manager        │──►│ Callback │──►│ End │
│ (API) │   │ Amount>1000? │   │ Approval Task  │   │ to CAP   │   │     │
└───────┘   └──────┬───────┘   └───────┬────────┘   └──────────┘   └─────┘
                   │                    │
                   │ NO (auto)          │ Rejected
                   ▼                    ▼
            ┌────────────┐      ┌─────────────┐
            │ Auto-Approve│      │ Notify      │──► End (rejected)
            │ + Callback │      │ Requester   │
            └────────────┘      └─────────────┘
```

---

### Step-by-Step in Process Builder

**1. Start Event (API Trigger)**
- Input: poNumber, amount, supplierName, requesterName, priority

**2. Decision (Condition)**
- Condition: `amount > 1000`
- YES branch → go to Approval Task
- NO branch → go to Auto-Approve step

**3. Auto-Approve Branch**
- Set `approvalResult = "approved"`
- Set `approvedBy = "System (Auto)"`
- Go to End

**4. Approval Task (YES branch)**
- Type: Approval Form
- Subject: `Approve PO ${poNumber} - $${amount}`
- Recipient: Manager (configure group or specific user)
- Form shows: PO details, items, requester info
- Outcomes: Approve / Reject

**5. After Approval**
- If Approved: Set `approvalResult = "approved"`, `approvedBy = {task assignee}`
- If Rejected: Set `approvalResult = "rejected"`, Set `rejectionReason = {comment}`

**6. End**
- Workflow complete

---

### Testing the Workflow

**Manual Test:**
1. In the Process Automation Lobby → your project
2. Click **"Run"** (or use the API test tool)
3. Provide test data:
   ```json
   {
     "poNumber": "PO-TEST-001",
     "amount": 5000,
     "supplierName": "TechCorp",
     "requesterName": "test@trial.com",
     "priority": "High"
   }
   ```
4. Open **My Inbox** → you should see the approval task!
5. Click on it → review the form → click Approve
6. Check the workflow instance → should be "Completed"

---

## Session 6: Monitoring & Testing Workflows (16:00 - 16:45)

### The Monitor View

SAP Build Process Automation provides monitoring:

```
┌──────────────────────────────────────────────────────────────┐
│  Monitor — Process and Workflow Instances                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ Instance ID  │ Definition    │ Status    │ Started      │  │
│  │ wf-001      │ PO Approval   │ ✅ Complete│ Jun 5, 10am │  │
│  │ wf-002      │ PO Approval   │ ⏳ Running │ Jun 5, 11am │  │
│  │ wf-003      │ PO Approval   │ ❌ Canceled│ Jun 4, 3pm  │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Click an instance → see:                                    │
│  • Context data (what was passed in)                        │
│  • Execution log (which steps ran, when)                     │
│  • Current step (where is it stuck?)                         │
│  • Task details (who has the pending task?)                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

### Workflow Statuses

| Status | Meaning |
|--------|---------|
| **Running** | Workflow is active, waiting for a human task or automation |
| **Completed** | All steps finished successfully |
| **Canceled** | Workflow was stopped/canceled |
| **Erroneous** | An error occurred (API call failed, invalid data) |
| **Suspended** | Paused (can be resumed) |

---

### Debugging a Stuck Workflow

```
Workflow stuck? Follow this checklist:

1. Check the Monitor → Is the instance in "Running" state?
2. Look at the Execution Log → Which step is it on?
3. If on an Approval Task:
   → Is the recipient assigned correctly?
   → Has the user checked My Inbox?
   → Is the recipient role assigned in BTP?
4. If on an API Call:
   → Check the URL is reachable
   → Check authentication credentials
   → Check the response (4xx? 5xx?)
5. If "Erroneous":
   → Read the error message in the instance details
   → Fix the issue → Retry or cancel and restart
```

---

### Accessing My Inbox

Users access their tasks via:
- **SAP Build Work Zone** → My Inbox tile
- **Direct URL:** `https://<subdomain>.sap-process-automation.cfapps.<region>.hana.ondemand.com/comsapaboraborequests.html`
- **BTP Cockpit** → SAP Build Process Automation → My Inbox

---

## Assessment: MCQ — 15 Questions on Build Process Automation

**Q1.** SAP Build Process Automation provides:

- A) Only database management
- B) Workflows (human tasks), Decisions (business rules), and Automations (bots)
- C) Only code deployment
- D) Only monitoring

**Answer: B** — Three capabilities: Workflows for human decisions, Decisions for automated logic, Automations for bot tasks.

---

**Q2.** A "Workflow" is:

- A) A database query
- B) A multi-step business process that includes human tasks (approvals, reviews)
- C) A deployment script
- D) A UI component

**Answer: B** — Workflows model multi-step processes where humans need to make decisions at certain points.

---

**Q3.** A "Decision Table" in SBPA is used for:

- A) Deciding which database to use
- B) Automated routing logic (if amount > $5000 → VP approval, else → manager)
- C) User interface design
- D) Code compilation

**Answer: B** — Decisions evaluate conditions automatically without human intervention.

---

**Q4.** To trigger a workflow from your CAP application, you:

- A) Send an email to SAP
- B) Call the Workflow REST API with the process definition ID and context data
- C) Click a button in BTP Cockpit
- D) Modify the database directly

**Answer: B** — POST to `/v1/workflow-instances` with the definition ID and input data.

---

**Q5.** "My Inbox" is where:

- A) You receive emails
- B) Approvers see and act on their pending workflow tasks (Approve/Reject)
- C) Developers write code
- D) Logs are stored

**Answer: B** — My Inbox shows all pending tasks assigned to a user. They can review details and take action.

---

**Q6.** The workflow "context" contains:

- A) CSS styles
- B) The business data passed when the workflow starts (PO number, amount, supplier)
- C) Database credentials
- D) HTML templates

**Answer: B** — Context = the data payload. Your CAP app sends PO details as context when triggering.

---

**Q7.** After a manager approves in the workflow, how does your CAP app know?

- A) Magic
- B) Callback: workflow calls your CAP API to update status, OR you poll the workflow status
- C) The database automatically updates
- D) An email arrives

**Answer: B** — Either the workflow calls back via API, or your app polls the workflow API for completion status.

---

**Q8.** A decision like "Auto-approve POs under $1000" removes the need for:

- A) The entire workflow
- B) Human intervention for small amounts (saves manager's time for important decisions)
- C) Authentication
- D) The CAP application

**Answer: B** — Decision tables automate trivial cases. Managers only handle significant amounts.

---

**Q9.** The main benefit of using a workflow vs custom code for approvals:

- A) Faster execution
- B) Visual design, built-in tracking/monitoring, My Inbox, deadlines, escalations — without coding
- C) Lower memory usage
- D) Better security

**Answer: B** — Workflows provide infrastructure (inbox, forms, audit trail, escalations) that would take months to code manually.

---

**Q10.** Workflow instances can be monitored to see:

- A) Only the final result
- B) Which step is currently active, who has the task, execution log, context data, and history
- C) Only errors
- D) Only completed instances

**Answer: B** — Full visibility: current step, assigned user, timing, data, errors — everything about the instance.

---

**Q11.** In the Process Builder, an "Approval Form" defines:

- A) The database schema
- B) The visual form the approver sees (display fields + action buttons like Approve/Reject)
- C) The API specification
- D) The deployment configuration

**Answer: B** — Forms show relevant data (read-only) and provide decision buttons for the approver.

---

**Q12.** Which role is needed to COMPLETE tasks in My Inbox?

- A) ProcessAutomationAdmin only
- B) ProcessAutomationParticipant (allows receiving and completing tasks)
- C) ProcessAutomationDeveloper only
- D) No role needed

**Answer: B** — Participant role = can view and act on tasks in My Inbox.

---

**Q13.** If a workflow step fails (API call error), the instance status becomes:

- A) Completed
- B) Erroneous (can be investigated in Monitor and potentially retried)
- C) Deleted
- D) Running (ignores the error)

**Answer: B** — Errors are captured. Admin can see details in Monitor and decide to retry or cancel.

---

**Q14.** The recommended approach for PO approval in CAP + SBPA:

- A) Code everything in CAP handlers (no workflow)
- B) CAP handles CRUD + Submit action → triggers workflow → workflow handles routing + approval → callback updates CAP
- C) Only use workflow, no CAP needed
- D) Use email for approvals

**Answer: B** — CAP manages data; SBPA manages the approval process. They work together via APIs.

---

**Q15.** Escalation in a workflow means:

- A) Increasing the PO amount
- B) If the task isn't completed within a deadline, automatically reassign to a higher authority
- C) Sending more data
- D) Restarting the server

**Answer: B** — Escalation = auto-reassign after timeout (e.g., if manager doesn't approve in 3 days → escalate to director).

---

## Assignment: Design a PO Approval Workflow

### Task

Design a complete Purchase Order approval workflow with the following requirements:

### Workflow Specification

```
TRIGGER: API call from CAP (when user clicks "Submit PO")

INPUT DATA:
  • poNumber (String)
  • amount (Number)
  • supplierName (String)
  • requesterName (String)
  • requesterEmail (String)
  • priority (String: Low/Medium/High/Urgent)
  • category (String: IT/Office/Travel/Equipment)
  • items (String: JSON of line items)

DECISION RULES:
  • Amount ≤ $500 → Auto-approve (no human needed)
  • Amount > $500 AND ≤ $5,000 → Manager approval (1 level)
  • Amount > $5,000 AND ≤ $50,000 → Manager + Director approval (2 levels)
  • Amount > $50,000 → Manager + Director + CFO (3 levels)
  • Priority = "Urgent" → Skip to Director (bypass manager)
  • Category = "IT" AND Amount > $2,000 → IT Manager must ALSO approve

APPROVAL FORM:
  • Display: PO number, amount, supplier, requester, priority, items
  • Input: Approval comment (optional)
  • Actions: [Approve] [Reject]

AFTER APPROVAL:
  • Notify requester (email): "Your PO has been approved/rejected"
  • Callback to CAP API: Update PO status

AFTER REJECTION:
  • Notify requester with rejection reason
  • PO status → "Rejected" (via callback)

DEADLINES:
  • Manager: 3 business days
  • Director: 5 business days
  • CFO: 7 business days
  • After deadline: Escalate to next level
```

### Deliverables

1. **Process flow diagram** (draw the steps with branches)
2. **Decision table** (rules for routing)
3. **Form design** (list the fields and their types)
4. **CAP trigger code** (JavaScript handler that starts the workflow)
5. **Callback handler** (JavaScript that receives the approval result)

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| SAP Build Process Automation | Low-code tool for workflows, decisions, and automations |
| Workflows | Multi-step processes with human tasks (approvals) |
| Decisions | Automated rules (route by amount, category, priority) |
| Forms | Visual forms approvers interact with in My Inbox |
| API Trigger | CAP starts workflows via REST API call |
| Callbacks | Workflow notifies CAP when complete (status update) |
| My Inbox | Where approvers see and act on pending tasks |
| Monitoring | Track instances, debug stuck workflows, view audit trail |
| Escalation | Auto-reassign after deadline passes |

### The Integration Pattern

```
CAP (Data + Logic) ←→ SBPA (Process + Decisions)

CAP handles: CRUD, validation, business logic, data storage
SBPA handles: Routing, human tasks, deadlines, escalations, audit trail

Together: Complete enterprise workflow! 🤝
```

### The One-Liner

> **SAP Build Process Automation turns your approval steps from "send an email and hope" into tracked, escalated, audited business processes — triggered by a single API call from your CAP app.**

---

### Looking Ahead: Day 39

Tomorrow: **Advanced Topics & Best Practices**
- CAP project structure best practices
- Multi-tenancy concepts
- Performance optimization
- Testing strategies (unit tests, integration tests)
- Preparing for the capstone project

---

*End of Day 38 — You can now automate business processes with zero code! 🤖*
