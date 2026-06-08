### Defining an Unbound Action in CDS

```cds
// srv/analytics-service.cds
using { com.epm as db } from '../db/schema';

service AnalyticsService @(path: '/analytics') {

  // Unbound action — belongs to the service, not an entity
  action GenerateReport(
    reportType : String(20);     // Input parameter
    startDate  : Date;           // Input parameter
    endDate    : Date            // Input parameter
  ) returns {                    // Output
    reportId   : UUID;
    status     : String(20);
    message    : String(200);
  };

  // Another unbound action — no parameters
  action PingHealth() returns {
    status    : String(10);
    timestamp : DateTime;
    version   : String(20);
  };

  @readonly entity ProductCatalog as projection on db.ProductCatalog;
}
```

### Implementing Unbound Action Handlers

```javascript
// srv/analytics-service.js
const cds = require('@sap/cds');

module.exports = function () {

  // Handler for the GenerateReport action
  this.on('GenerateReport', async (req) => {
    const { reportType, startDate, endDate } = req.data;

    // Validate inputs
    if (!reportType) {
      req.reject(400, 'Report type is required');
    }

    const validTypes = ['Sales', 'Inventory', 'Customers'];
    if (!validTypes.includes(reportType)) {
      req.reject(400, `Invalid report type. Must be: ${validTypes.join(', ')}`);
    }

    if (startDate > endDate) {
      req.reject(400, 'Start date must be before end date');
    }

    // Simulate report generation
    const reportId = cds.utils.uuid();

    console.log(`Generating ${reportType} report from ${startDate} to ${endDate}...`);

    // In real app: query data, generate PDF, store somewhere
    return {
      reportId: reportId,
      status: 'Generated',
      message: `${reportType} report generated successfully for ${startDate} to ${endDate}`
    };
  });

  // Handler for PingHealth
  this.on('PingHealth', (req) => {
    return {
      status: 'healthy',
      timestamp: new Date().toISOString(),
      version: '1.0.0'
    };
  });

};
```



### Calling an Unbound Action (REST Client)

```http
### Call unbound action: GenerateReport
POST http://localhost:4004/analytics/GenerateReport
Content-Type: application/json

{
  "reportType": "Sales",
  "startDate": "2026-01-01",
  "endDate": "2026-05-31"
}
```

**Response (200 OK):**
```json
{
  "reportId": "a1b2c3d4-e5f6-7890-...",
  "status": "Generated",
  "message": "Sales report generated successfully for 2026-01-01 to 2026-05-31"
}
```









**Syntax breakdown:**

```cds
action <ActionName>(
  <param1> : <Type>;       // Input parameters (optional)
  <param2> : <Type>
) returns {                // Return type (optional)
  <field1> : <Type>;
  <field2> : <Type>;
};
```
