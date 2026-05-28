### Setting Up Express

```bash
# Create project:
mkdir book-api && cd book-api
npm init -y
npm install express
```

Create `server.js`:

```javascript
const express = require('express');
const app = express();

// Middleware: Parse JSON request bodies
app.use(express.json());

// Route: GET /
app.get('/', (req, res) => {
  res.json({ message: "Welcome to the Book API! 📚" });
});

// Start server:
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running at http://localhost:${PORT}`);
});
```

Run: `node server.js` → Open `http://localhost:3000`
