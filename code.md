## Session 3: Hands-on — Create Destinations & Configure App (12:00 - 13:00)

### Exercise 1: Create a Destination in BTP Cockpit (20 minutes)

1. **Navigate:** BTP Cockpit → Subaccount → Connectivity → Destinations
2. Click **"New Destination"**
3. Fill in:

```
Name:               po-management-api
Type:               HTTP
URL:                https://po-management-srv.cfapps.us10.hana.ondemand.com
Proxy Type:         Internet
Authentication:     NoAuthentication

Additional Properties (click "New Property"):
  HTML5.ForwardAuthToken = true
  HTML5.DynamicDestination = true
  WebIDEUsage = odata_gen
  WebIDEEnabled = true
```

4. Click **"Save"**
5. Click **"Check Connection"** → should show `200 OK`

---

### Exercise 2: Verify Destination Works (10 minutes)

After creating the destination:

1. Click **"Check Connection"** button
2. Expected result: `Connection to "po-management-api" established. Response: 200 OK`
3. If it fails:
   - Check URL is correct (copy from `cf app po-management-srv | grep routes`)
   - Ensure backend app is running (`cf apps`)
   - Verify no typos in destination name
