### Quiz Questions

**Day 16 — CAP Services Basics**

**Q1.** What does `as projection on` do in a service definition?
- A) Creates a copy of the database table
- B) Exposes a database entity through the service as an API endpoint
- C) Deletes the entity from the database
- D) Renames the entity

---

**Q2.** Given `service PurchaseService {}`, what URL path is generated?
- A) `/purchase-service`
- B) `/purchaseservice`
- C) `/purchase`
- D) `/PurchaseService`

---

**Q3.** What does `@readonly` do on a service entity?
- A) Makes the entity invisible
- B) Allows only GET requests, blocks POST/PUT/PATCH/DELETE
- C) Allows only POST requests
- D) Encrypts the data

---

**Q4.** Fill in: The `$metadata` endpoint returns an _____ document that describes the API schema.

---

**Q5.** How many services can one CAP project have?
- A) Exactly one
- B) Maximum three
- C) As many as needed
- D) One per entity

---

**Day 17 — OData Query Operations**

**Q6.** Which OData option filters records?
- A) `$search`
- B) `$filter`
- C) `$where`
- D) `$query`

---

**Q7.** To get books with price between 10 and 50, the filter is:
- A) `$filter=price between 10 and 50`
- B) `$filter=price ge 10 and price le 50`
- C) `$filter=price > 10 AND price < 50`
- D) `$filter=price in (10,50)`

---

**Q8.** `$top=10&$skip=20&$count=true` — what does `$count` return?
- A) 10 (the page size)
- B) 20 (the skip value)
- C) Total matching records BEFORE pagination
- D) Number of pages

---

**Q9.** Inside `$expand()`, options are separated by:
- A) Ampersand (`&`)
- B) Comma (`,`)
- C) Semicolon (`;`)
- D) Pipe (`|`)

---

**Q10.** To search case-insensitively for "phone" in product names:
- A) `$filter=contains(productName,'phone')`
- B) `$filter=contains(tolower(productName),'phone')`
- C) `$filter=productName like '%phone%'`
- D) `$search=phone`

---

**Day 18 — CRUD & Custom Logic**

**Q11.** What is the difference between PUT and PATCH?
- A) PUT is for create, PATCH is for update
- B) PUT replaces the entire record, PATCH updates only sent fields
- C) PUT is slower, PATCH is faster
- D) No difference

---

**Q12.** Custom handler files must be named:
- A) `handlers.js`
- B) Same as the CDS file with `.js` extension
- C) Any name you want
- D) `index.js`

---

**Q13.** In a `before('CREATE', 'Books', handler)`, `req.data` contains:
- A) The database record
- B) The incoming request body (JSON payload)
- C) The response data
- D) The error messages

---

**Q14.** `req.error(400, 'Price is required', 'price')` — the third parameter identifies:
- A) The HTTP method
- B) The target field that caused the error
- C) The error code
- D) The entity name

---

**Q15.** TRUE or FALSE: Deep Insert (creating parent + children in one POST) works only with Composition.
- A) TRUE
- B) FALSE

---

**Day 19 — Actions, Functions & Events**

**Q16.** Actions use which HTTP method?
- A) GET
- B) POST
- C) PUT
- D) DELETE

---

**Q17.** Functions use which HTTP method?
- A) GET
- B) POST
- C) PATCH
- D) OPTIONS

---

**Q18.** The key difference: Actions can _____ data; Functions are _____.
- A) read; write-only
- B) modify; read-only
- C) delete; create-only
- D) encrypt; decrypt

---

**Q19.** Bound actions are defined where in CDS?
- A) Directly inside `service { }`
- B) In a separate file
- C) Inside the entity's `actions { }` block
- D) In `db/schema.cds`

---

**Q20.** The URL for calling bound action `approve` on PurchaseOrder with ID `po-123`:
- A) `POST /purchasing/approve?id=po-123`
- B) `POST /purchasing/PurchaseOrders(po-123)/PurchasingService.approve`
- C) `GET /purchasing/PurchaseOrders(po-123)/approve`
- D) `POST /purchasing/PurchaseOrders/po-123/approve`

---

**Q21.** Events in CAP are:
- A) Synchronous — waits for all listeners
- B) Asynchronous — fire and forget
- C) Only for errors
- D) Only available in HANA

---

**Q22.** To emit an event from a handler:
- A) `req.emit('EventName', data)`
- B) `this.emit('EventName', data)`
- C) `cds.fire('EventName', data)`
- D) `event.send('EventName', data)`

---

**Day 20 — Best Practices & Integration**

**Q23.** How should you split services in a CAP project?
- A) One service per file
- B) One service per entity
- C) One service per business domain
- D) Only one service per project

---

**Q24.** Which annotation makes an entity write-once (create but no update/delete)?
- A) `@writeonce`
- B) `@insertonly`
- C) `@createonly`
- D) `@immutable`

---

**Q25.** For status changes in a workflow (New → Confirmed → Shipped), you should use:
- A) PATCH with status field (direct update)
- B) Bound actions (confirm, ship, deliver)
- C) DELETE and recreate with new status
- D) SQL UPDATE statement

---

**Q26.** TRUE or FALSE: Within the same CAP project, one service can directly access another service's database entities.
- A) TRUE
- B) FALSE

---

**Q27.** Which handler phase should you use for input validation?
- A) `on`
- B) `after`
- C) `before`
- D) `init`

---

**Q28.** Fill in: For pagination, the formula for $skip is: `$skip = (_____ - 1) × _____`

---

**Q29.** What happens when you call `req.reject(403, 'Forbidden')`?
- A) Logs a warning but continues
- B) Immediately stops processing and returns error to client
- C) Adds to error collection
- D) Retries the request

---

**Q30.** Which is the correct order of handler execution?
- A) after → on → before
- B) on → before → after
- C) before → on → after
- D) before → after → on
