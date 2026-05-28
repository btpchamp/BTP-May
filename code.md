Session 3: Hands-on — Build a Complete REST API
Step 1: Project Setup 

```bash
mkdir ~/cap-training/product-api
cd ~/cap-training/product-api
npm init -y
npm install express
```

Step 2: Create server.js (Use the CRUD code from Session 2)

Copy the complete CRUD example from above into server.js.

Step 3: Start the Server

```bash
node server.js
```

Step 4: Test with REST Client (Create test.http)

Create a file called test.http in VS Code (requires REST Client extension):

```http
### Get all products
GET http://localhost:3000/products

### Get a single product
GET http://localhost:3000/products/1

### Get a non-existent product (should return 404)
GET http://localhost:3000/products/999

### Create a new product
POST http://localhost:3000/products
Content-Type: application/json

{
  "name": "Wireless Mouse",
  "category": "Electronics",
  "price": 1200,
  "stock": 50
}

### Create another product
POST http://localhost:3000/products
Content-Type: application/json

{
  "name": "Mechanical Keyboard",
  "category": "Electronics",
  "price": 3500,
  "stock": 20
}

### Create with missing fields (should return 400)
POST http://localhost:3000/products
Content-Type: application/json

{
  "price": 500
}

### Update a product
PUT http://localhost:3000/products/1
Content-Type: application/json

{
  "name": "Wireless Mouse Pro",
  "category": "Electronics",
  "price": 1500,
  "stock": 40
}

### Delete a product
DELETE http://localhost:3000/products/3

### Verify deletion
GET http://localhost:3000/products

```

Step 5: Add Search & Filter (Enhancement)

Enhance your API by adding filtering, searching, and sorting capabilities.

Update the GET /products route inside server.js

### Test Search Filter

### Search by price
```http
### Sort by Price
GET http://localhost:3000/products?sort=price

###Filter by minimum Price
GET http://localhost:3000/products?minPrice=2000

```

