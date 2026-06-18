## Session 3: Hands-on — Create HANA Cloud Instance in BTP Trial (12:00 - 13:00)

### Prerequisites

- SAP BTP Trial account (https://account.hanatrial.ondemand.com)
- Access to SAP BTP Cockpit

---

### Step 1: Access BTP Cockpit

1. Log in to https://cockpit.hanatrial.ondemand.com
2. Click on your **Subaccount** (e.g., "trial")
3. You should see the subaccount overview

---

### Step 2: Create HANA Cloud Instance

1. In the left menu: **SAP HANA Cloud** → Click **"Create Instance"**
   (Or: go to **Service Marketplace** → search "SAP HANA Cloud" → Create)

2. Fill in the form:
   - **Instance Name:** `hana-trial` (or any name)
   - **Administrator Password:** Choose a strong password (save it!)
   - **Memory:** 30 GB (trial default)
   - **Allowed Connections:** Select "Allow all IP addresses" for trial

3. Click **"Create Instance"**

4. Wait 5-10 minutes for provisioning (grab a coffee ☕)

5. Status should change to: **"Running"** ✅

---

### Step 3: Verify HANA Cloud is Running

1. In BTP Cockpit → SAP HANA Cloud → Your instance
2. Status: **Running** (green indicator)
3. Note the **SQL Endpoint** — you'll need this later:
   ```
   abc123.hana.trial-us10.hanacloud.ondemand.com:443
   ```
https://developers.sap.com/group.hana-cloud-get-started-1-trial.html
---

### Step 4: Open Database Explorer

1. Click **"Open in SAP HANA Database Explorer"** (from instance actions)
2. Or go to: SAP HANA Database Explorer (from BTP tools)
3. Add your database connection:
   - Host: (your SQL endpoint)
   - Port: 443
   - User: DBADMIN
   - Password: (the one you set)
4. You should see the database catalog (empty for now — no tables yet!)

---

### Important Notes for Trial

| Note | Details |
|------|---------|
| **Auto-stop** | Trial HANA stops after inactivity. Restart from BTP Cockpit. |
| **Memory limit** | 30 GB in trial (plenty for learning!) |
| **Cost** | Free in trial. In production: pay-per-use based on memory. |
| **Region** | Choose a region close to you for lower latency |
| **Allowed IPs** | In trial, allow all. In production, restrict! |

---
