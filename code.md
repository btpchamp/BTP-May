## Node.js Core Modules


### 3. `http` — Create a Web Server

Node.js can create a web server with ZERO external packages:

```javascript
const http = require('http');

const server = http.createServer((request, response) => {
  // 'request' = what the client sent (URL, method, headers)
  // 'response' = what we send back
  
  response.writeHead(200, { 'Content-Type': 'application/json' });
  response.end(JSON.stringify({ message: "Hello from Node.js!", time: new Date() }));
});

server.listen(3000, () => {
  console.log("🚀 Server running at http://localhost:3000");
});
