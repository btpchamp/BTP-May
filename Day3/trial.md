### Step-by-Step: Create Your SAP BTP Trial Account

#### Step 1: Go to SAP BTP Trial Page

1. Open your browser
2. Go to: **https://www.sap.com/products/technology-platform/trial.html**
3. Click **"Start your free trial"** button

```
What you'll see:
+--------------------------------------------------+
|  SAP Business Technology Platform                |
|                                                  |
|  [  Start your free trial  ]  ← Click this!     |
|                                                  |
|  ✓ No credit card required                       |
|  ✓ 90-day access                                 |
|  ✓ Full BTP experience                           |
+--------------------------------------------------+
```

---

#### Step 2: Create SAP Account (if you don't have one)

1. Click **"Register"** if you don't have an SAP account
2. Fill in:
   - First Name
   - Last Name
   - Email address (use personal email for trial)
   - Password (must include uppercase, lowercase, number, special char)
   - Country: India
3. Agree to terms and conditions
4. Click **"Register"**
5. Check your email for verification link — **click it!**

```
📧 Check your inbox:
+--------------------------------------------------+
| From: SAP                                        |
| Subject: Please verify your email                |
|                                                  |
| Click the link below to verify your account:     |
| [Verify Email Address]  ← Click this!           |
+--------------------------------------------------+
```

---

#### Step 3: Access BTP Trial Cockpit

1. Go to: **https://cockpit.hanatrial.ondemand.com/**
2. Log in with your SAP account credentials
3. First time? You'll see a **Welcome** screen

```
What you'll see:
+--------------------------------------------------+
|  Welcome to SAP BTP Trial                        |
|                                                  |
|  Select your region:                             |
|  ○ US East (VA) - AWS                           |
|  ○ Europe (Frankfurt) - AWS                      |
|  ● Singapore - AWS  ← Select closest to you!    |
|                                                  |
|  [  Enter Your Trial Account  ]                  |
+--------------------------------------------------+
```

**Important:** Choose the region closest to you (for India, choose Singapore or us10 if Singapore isn't available).

---

#### Step 4: Wait for Trial Setup

After clicking "Enter Your Trial Account," SAP will:
1. Create your Global Account
2. Create a Subaccount called "trial"
3. Set up Cloud Foundry environment
4. Create an Organization and Space

**This takes 1-2 minutes. Be patient!**

```
Setting up your trial...
[████████████░░░░░░░] 60%

Creating global account...  ✓
Creating subaccount...      ✓
Enabling Cloud Foundry...   ⏳
Creating org and space...   ⏳
```

---

#### Step 5: You're In! 🎉

Once setup is complete, you'll see the **BTP Cockpit** — your control center!

```
+--------------------------------------------------+
|  SAP BTP Cockpit                                 |
|--------------------------------------------------|
|  Global Account: <your-id>trial                  |
|                                                  |
|  Subaccounts:                                    |
|  +--------------------------------------------+  |
|  | 📁 trial                                   |  |
|  | Region: ap21 (Singapore)                   |  |
|  | Environment: Cloud Foundry                 |  |
|  | Status: ● Active                           |  |
|  +--------------------------------------------+  |
|                                                  |
+--------------------------------------------------+
```

---

#### Step 6: Note Down Your Details

Write these down — you'll need them later in the course:

| Detail | Your Value |
|--------|-----------|
| Global Account Name | |
| Subaccount Name | |
| Region | |
| CF API Endpoint | |
| Org Name | |
| Space Name | |

**How to find CF API Endpoint:**
1. Click on your subaccount → "Overview" tab
2. Look for "Cloud Foundry" section
3. API Endpoint will look like: `https://api.cf.ap21.hana.ondemand.com`

---

### Troubleshooting Common Issues

| Problem | Solution |
|---------|----------|
| "Email already registered" | Try logging in — you may already have an SAP account |
| Verification email not received | Check spam folder, wait 5 minutes, try resend |
| Trial setup stuck | Refresh the page after 3 minutes |
| Region selection not showing | Some regions may be full — try a different one |
| "Trial account expired" | You can extend once, or create a new account with different email |

---


