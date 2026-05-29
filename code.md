## Session 3: Hands-on — Create Your First CAP Project (13:30 - 14:30)

### Step 1: Create the Project

```bash
# Navigate to your training folder:
cd ~/cap-training

# Create a new CAP project:
cds init my-first-cap

# Enter the project:
cd my-first-cap

```

**What just happened?** `cds init` created a complete project structure with everything you need!

---

### Step 2: Explore What Was Created

```bash
# See the project structure:
ls -la
```

You should see:

```
my-first-cap/
├── app/              ← Frontend/UI code goes here (empty for now)
├── db/               ← Database models (CDS files) go here
├── srv/              ← Service definitions and logic go here
├── package.json      ← Project config (Node.js)
└── README.md         ← Project documentation
```

---

### Step 3: Look at package.json

```bash
cat package.json
```

```json
{
  "name": "my-first-cap",
  "version": "1.0.0",
  "description": "A simple CAP project.",
  "repository": "<Add your repository here>",
  "license": "UNLICENSED",
  "private": true,
  "dependencies": {
    "@sap/cds": "^7",
  },
  "devDependencies": {
    "@sap/cds-dk": "^7"
  },
  "scripts": {
    "start": "cds-serve",
    "watch": "cds watch"
  }
}
```

**Key observations:**
- `@sap/cds` is a dependency (the runtime)
- `@sap/cds-dk` is a devDependency (development tools)
- Scripts: `start` for production, `watch` for development

---

### Step 4: Install Dependencies

```bash
npm install
```

This downloads `@sap/cds`, `express`, and all their sub-dependencies into `node_modules/`.

---

### Step 5: Create Your First Data Model

Create a new file `db/schema.cds`:

```cds
namespace my.bookshop;

entity Books {
  key ID    : Integer;
  title     : String(100);
  author    : String(100);
  price     : Decimal(10,2);
  stock     : Integer;
  category  : String(50);
}

entity Authors {
  key ID    : Integer;
  name      : String(100);
  country   : String(50);
}
```

---

### Step 6: Create Your First Service

Create a new file `srv/catalog-service.cds`:

```cds
using my.bookshop from '../db/schema';

service CatalogService {
  entity Books as projection on bookshop.Books;
  entity Authors as projection on bookshop.Authors;
}
```

**That's it! Three lines to create a full REST API with CRUD for Books and Authors!**

---

### Step 7: Add Sample Data

Create a file `db/data/my.bookshop-Books.csv`:

```csv
ID,title,author,price,stock,category
1,Clean Code,Robert Martin,450,20,Programming
2,The Pragmatic Programmer,David Thomas,550,15,Programming
3,Sapiens,Yuval Noah Harari,400,30,History
4,Atomic Habits,James Clear,350,50,Self-Help
5,Node.js Design Patterns,Mario Casciaro,600,10,Programming
```

Create a file `db/data/my.bookshop-Authors.csv`:

```csv
ID,name,country
1,Robert Martin,USA
2,David Thomas,USA
3,Yuval Noah Harari,Israel
4,James Clear,USA
5,Mario Casciaro,Italy
```

**Naming convention:** `<namespace>-<EntityName>.csv` — CAP auto-loads these!

---

### Step 8: Run It!

```bash
cds watch
```

**Expected output:**

```
[cds] - loaded model from 2 file(s):

  srv/catalog-service.cds
  db/schema.cds

[cds] - connect to db > sqlite { database: ':memory:' }
  > init from db/data/my.bookshop-Books.csv
  > init from db/data/my.bookshop-Authors.csv
[cds] - serving CatalogService { path: '/catalog' }

[cds] - server listening on { url: 'http://localhost:4004' }
[cds] - [ terminate with ^C ]
```

---

### Step 9: Test It!

Open your browser: **http://localhost:4004**

You'll see a welcome page listing available services and entities:

```
Welcome to cds.services

Service Endpoints:
  /catalog - CatalogService
    /catalog/Books
    /catalog/Authors
```

**Try these URLs:**
## Create a new file `test.http` in the  project root folder

| URL | What It Returns |
|-----|----------------|
| `http://localhost:4004/odata/v4/catalog/Books` | All books (JSON) |
| `http://localhost:4004/odata/v4/catalog/Books?$filter=price gt 400` | Books over ₹400 |
| `http://localhost:4004/odata/v4/catalog/Books?$orderby=price desc` | Sorted by price |
| `http://localhost:4004/odata/v4/catalog/Books?$top=3` | First 3 books |
| `http://localhost:4004/odata/v4/catalog/Books?$select=title,price` | Only title and price |
| `http://localhost:4004/odata/v4/catalog/Authors` | All authors |
| `http://localhost:4004/odata/v4/catalog/$metadata` | OData metadata (XML) |

**All of this from ~15 lines of CDS code!** No Express routes, no SQL queries, no manual parsing.

---

### Step 10: Test with REST Client

Create `test.http` in your project root:

```http
### Get all books
GET http://localhost:4004/odata/v4/catalog/Books

### Get books filtered by category
GET http://localhost:4004/odata/v4/catalog/Books?$filter=category eq 'Programming'

### Get a specific book
GET http://localhost:4004/odata/v4/catalog/Books(1)

### Create a new book
POST http://localhost:4004/odata/v4/catalog/Books
Content-Type: application/json

{
  "ID": 6,
  "title": "Learning SAP CAP",
  "author": "SAP Team",
  "price": 800,
  "stock": 100,
  "category": "Programming"
}

### Update a book
PATCH http://localhost:4004/odata/v4/catalog/Books(1)
Content-Type: application/json

{
  "price": 499,
  "stock": 25
}

### Delete a book
DELETE http://localhost:4004/odata/v4/catalog/Books(6)

### Get all authors
GET http://localhost:4004/odata/v4/catalog/Authors
```

Click "Send Request" on each — they ALL work!

---
