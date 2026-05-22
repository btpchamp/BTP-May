```javascript
const appName = "CAP Training";

function greet() {
  let greeting = "Hello";
  
  if (true) {
    let blockVar = "I'm trapped here";
    var functionVar = "I escape to function scope!";
    
    console.log(appName);     
    console.log(greeting);    
    console.log(blockVar);    
  }
  
  console.log(greeting);    
  console.log(functionVar);  
  console.log(blockVar);    
}

greet();
console.log(appName);        
console.log(greeting);       
```
