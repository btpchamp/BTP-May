```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout 1");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise 1");
});

process.nextTick(() => {
  console.log("Next Tick");
});
console.log("End");

Question ) What is the output order?





for (var i = 1; i <= 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, i * 100);
}




for (let i = 1; i <= 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, i * 100);
}





Promise.resolve(1)
  .then((x) => {
    console.log(x);
    return x + 1;
  })
  .then((x) => {
    throw new Error("Failed");
  })
  .catch((err) => {
    console.log(err.message);
    return 100;
  })
  .then((x) => {
    console.log(x);
  });






const wait = (ms, value) =>
  new Promise((resolve) =>
    setTimeout(() => resolve(value), ms)
  );

async function run() {
  console.time("time");

  const a = await wait(1000, "A");
  const b = await wait(1000, "B");

  console.log(a, b);

  console.timeEnd("time");
}

run();







async function run() {
  console.time("time");

  const [a, b] = await Promise.all([
    wait(1000, "A"),
    wait(1000, "B")
  ]);

  console.log(a, b);

  console.timeEnd("time");
}







async function test() {
  return Promise.resolve("Hello");
}

console.log(test());








const p = new Promise((resolve, reject) => {
  resolve("First");
  resolve("Second");
  reject("Error");
});

p.then(console.log).catch(console.log);






async function test() {
  try {
    Promise.reject("Failed");
  } catch (e) {
    console.log("Caught");
  }
}

test();

Question) Will "Caught" print?





let balance = 100;

async function deduct(amount) {
  const current = balance;

  await new Promise((r) => setTimeout(r, 100));

  balance = current - amount;
}

async function run() {
  await Promise.all([
    deduct(30),
    deduct(50)
  ]);

  console.log(balance);
}

run();

Question ) What can the final balance be?




setTimeout(() => console.log("timeout"));
setImmediate(() => console.log("immediate"));
Promise.resolve().then(() => console.log("promise"));
process.nextTick(() => console.log("nextTick"));

Question) Predict the exact order.









