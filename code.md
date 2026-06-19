## Session 4: Weekly Quiz — Days 26-30 (13:45 - 15:00)

### Instructions
- 30 questions covering Days 26-30 (Git, HANA, Auth, Security)
- Time: 75 minutes
- Passing score: 21/30 (70%)

---

**Q1.** The 3 areas in Git are:
- A) Branch, Merge, Push
- B) Working Directory, Staging Area, Repository
- C) Local, Remote, Cloud
- D) Add, Commit, Deploy

**Answer: B**

---

**Q2.** `git add .` followed by `git commit -m "message"` does what?
- A) Pushes code to GitHub
- B) Stages all changes and saves a permanent snapshot with a description
- C) Creates a new branch
- D) Deletes uncommitted changes

**Answer: B**

---

**Q3.** A feature branch allows you to:
- A) Delete the main branch
- B) Work in isolation without affecting the main/stable code
- C) Speed up Git operations
- D) Skip code review

**Answer: B**

---

**Q4.** `.gitignore` should include all of these EXCEPT:
- A) `node_modules/`
- B) `.env`
- C) `db/schema.cds`
- D) `*.db`

**Answer: C** — `schema.cds` is your source code — it MUST be committed. The others are generated/secret/local.

---

**Q5.** A Pull Request (PR) is used for:
- A) Downloading code
- B) Requesting code review before merging a branch into main
- C) Pulling from multiple remotes
- D) Requesting git access

**Answer: B**

---

**Q6.** `git pull --rebase origin main` is used when:
- A) You want to delete remote changes
- B) Your push was rejected because remote has newer commits
- C) You want to create a new branch
- D) You want to revert to an older version

**Answer: B**

---

**Q7.** SAP HANA stores data in:
- A) Hard disk only
- B) RAM (in-memory) for speed, with persistence to disk
- C) The cloud only (no local storage)
- D) CSV files

**Answer: B**

---

**Q8.** Column store is better than row store for:
- A) Inserting one record at a time
- B) Aggregation queries (SUM, AVG, COUNT) on large datasets
- C) Looking up a single record by primary key
- D) Storing images

**Answer: B**

---

**Q9.** HDI Container provides:
- A) More CPU for your app
- B) Isolation between applications (each app has its own database objects)
- C) A container for Docker images
- D) User interface hosting

**Answer: B**

---

**Q10.** `cds add hana` configures your project to:
- A) Create a HANA Cloud instance
- B) Use HANA in production and SQLite in development (profile-based)
- C) Delete the SQLite database
- D) Generate HTML files

**Answer: B**

---

**Q11.** In HANA SQL, to get the top 5 most expensive products:
- A) `SELECT * FROM Products LIMIT 5 ORDER BY price DESC`
- B) `SELECT TOP 5 * FROM COM_EPM_PRODUCTS ORDER BY PRICE DESC`
- C) `SELECT * FROM Products WHERE ROWNUM <= 5`
- D) `SELECT FIRST 5 FROM Products`

**Answer: B**

---

**Q12.** CDS `String(100)` maps to HANA type:
- A) `VARCHAR(100)`
- B) `NVARCHAR(100)`
- C) `TEXT(100)`
- D) `STRING(100)`

**Answer: B** — HANA uses NVARCHAR (Unicode-capable).

---

**Q13.** If HANA deployment fails with "foreign key constraint violated":
- A) Delete all tables and retry
- B) Parent/reference tables must be loaded before child tables
- C) Change all associations to compositions
- D) Increase HANA memory

**Answer: B**

---

**Q14.** Authentication proves:
- A) What you can do
- B) WHO you are (identity verification)
- C) Where you are located
- D) How much data you can access

**Answer: B**

---

**Q15.** Authorization determines:
- A) Your password strength
- B) WHAT you are allowed to do (permissions)
- C) Your login time
- D) Your network speed

**Answer: B**

---

**Q16.** XSUAA issues what type of token?
- A) Session cookie
- B) JWT (JSON Web Token) containing identity and scopes
- C) API key
- D) SSL certificate

**Answer: B**

---

**Q17.** In `xs-security.json`, the hierarchy from smallest to largest is:
- A) Role Collection → Role Template → Scope
- B) Scope → Role Template → Role Collection
- C) Role Template → Role Collection → Scope
- D) Scope → Role Collection → Role Template

**Answer: B** — Scopes (atomic permissions) → Role Templates (bundles) → Role Collections (assigned to users).

---

**Q18.** `@requires: 'authenticated-user'` on a service means:
- A) Only admins can access
- B) ANY logged-in user can access (regardless of specific role)
- C) No authentication needed
- D) Only users from a specific company

**Answer: B**

---

**Q19.** `@restrict: [{ grant: 'READ', to: 'Viewer' }, { grant: '*', to: 'Admin' }]` means:
- A) Everyone can read and write
- B) Viewers can ONLY read; Admins can do everything
- C) Nobody can access
- D) Viewers and Admins have the same permissions

**Answer: B**

---

**Q20.** `where: 'createdBy = $user'` in @restrict:
- A) Sets createdBy to the current user
- B) Filters data so users only see records they created (row-level security)
- C) Validates that createdBy is not null
- D) Creates a new user

**Answer: B**

---

**Q21.** HTTP status code 401 means:
- A) Success
- B) Not Found
- C) Not Authenticated (no valid login/token)
- D) Server Error

**Answer: C**

---

**Q22.** HTTP status code 403 means:
- A) Success
- B) Not Found
- C) Authenticated but NOT Authorized (wrong role/permission)
- D) Server Error

**Answer: C**

---

**Q23.** To test security locally in CAP, you configure:
- A) Real XSUAA service
- B) Mocked users with roles in package.json (`kind: "mocked"`)
- C) A separate auth server
- D) Browser cookies

**Answer: B**

---

**Q24.** `req.user.id` in a handler returns:
- A) The user's password
- B) The authenticated user's identity (e.g., email address)
- C) The database user
- D) A random number

**Answer: B**

---

**Q25.** `$user.attr.Department` in a where clause accesses:
- A) A field in the entity
- B) A user attribute configured in xs-security.json (e.g., user's department)
- C) The department table
- D) An environment variable

**Answer: B**

---

**Q26.** Row-level security ensures:
- A) Users can't log in
- B) Users only see the specific RECORDS they're authorized to access
- C) Tables are encrypted
- D) Columns are hidden

**Answer: B**

---

**Q27.** If a new entity is added to a service WITHOUT @restrict:
- A) It's automatically secured
- B) ANY authenticated user can perform ALL operations on it (potential security hole!)
- C) It's invisible
- D) Only admins can access it

**Answer: B** — New entities without restrictions inherit only the service-level @requires (which allows all operations for anyone with that role).

---

**Q28.** To prevent a user from editing ANOTHER user's record:
- A) Use @readonly
- B) `{ grant: 'UPDATE', where: 'createdBy = $user' }` (only update own records)
- C) Remove the UPDATE button from the UI
- D) Don't expose the entity

**Answer: B** — Row-level WHERE clause on UPDATE ensures users can only modify their own data.

---

**Q29.** Which is a security pitfall?
- A) Using `req.user.id` to set createdBy
- B) Trusting client-sent `createdBy` field from the request body
- C) Adding @restrict to every entity
- D) Testing with multiple user profiles

**Answer: B** — Never trust client data for authorization decisions. Always use server-validated `req.user`.

---

**Q30.** The correct order of security checks in CAP is:
- A) Authorization → Authentication → Query
- B) Authentication → Service @requires → Entity @restrict → WHERE filter → Response
- C) Query → Authentication → Response
- D) Response → Authorization → Authentication

**Answer: B** — First verify identity, then check service access, then entity permissions, then row-level filtering.
