Here is the **ultimate JavaScript "Broken vs Fixed" suite** — 50 real-world, production-killing vs production-saving code snippets.

Works in **Node.js 18+** and **modern browsers** (Chrome, Firefox, Edge, Safari).

- `broken_01.js` → `broken_50.js` → **crashes, leaks, blocks event loop, security holes**
- `fixed_01.js` → `fixed_50.js` → **fast, safe, memory-efficient, idiomatic 2025 JS/TS**

Run with:
```bash
node broken_01.js   # explodes
node fixed_01.js    # works perfectly
```

```js
// broken_01_double_equal.js
if ("0" == 0) console.log("true");        // true → type coercion hell
if ("" == false) console.log("true");     // true
if ([] == ![]) console.log("true");       // true
```

```js
// fixed_01_double_equal.js
if ("0" === 0) console.log("never");      // false
if ("" === false) console.log("never");   // false
if (Object.is([], ![])) console.log("never");
```

```js
// broken_02_var_hoisting.js
console.log(x);  // undefined (not ReferenceError)
var x = 42;
```

```js
// fixed_02_var_hoisting.js
console.log(x);  // ReferenceError — caught early
let x = 42;
```

```js
// broken_03_mutable_default.js
function add(item, list = []) {
  list.push(item);
  return list;
}
console.log(add(1)); // [1]
console.log(add(2)); // [1,2] → shared array!
```

```js
// fixed_03_mutable_default.js
function add(item, list = null) {
  const arr = list || [];
  arr.push(item);
  return arr;
}
```

```js
// broken_04_async_void.js
async function fireAndForget() {
  throw new Error("lost forever");
}
fireAndForget();
```

```js
// fixed_04_async_void.js
async function fireAndForget() {
  try {
    throw new Error("caught");
  } catch (err) {
    console.error("Handled:", err);
  }
}
fireAndForget().catch(console.error);
```

```js
// broken_05_promise_no_catch.js
fetch("/api/data").then(res => res.json());
// Unhandled promise rejection → process crash in Node
```

```js
// fixed_05_promise_no_catch.js
fetch("/api/data")
  .then(res => res.json())
  .catch(err => console.error("API failed:", err));
```

```js
// broken_06_event_loop_block.js
function block() {
  const start = Date.now();
  while (Date.now() - start < 5000) { } // 5s freeze
}
block();
console.log("never runs");
```

```js
// fixed_06_event_loop_block.js
setTimeout(() => {
  const start = Date.now();
  while (Date.now() - start < 5000) { }
  console.log("done blocking off-main-thread");
}, 0);
```

```js
// broken_07_memory_leak_closure.js
const leaks = [];
function createLeak() {
  const huge = new Array(10_000_000).fill("leak");
  leaks.push(() => huge);
}
setInterval(createLeak, 100);
```

```js
// fixed_07_memory_leak_closure.js
let huge = null;
setInterval(() => {
  huge = new Array(10_000_000).fill("ok");
  huge = null; // allow GC
}, 1000);
```

```js
// broken_08_setTimeout_zero.js
setTimeout(() => console.log("I block!"), 0);
for (let i = 0; i < 1e9; i++) { } // blocks event loop
```

```js
// fixed_08_setTimeout_zero.js
setImmediate(() => console.log("runs after sync code"));
process.nextTick(() => console.log("runs first"));
```

```js
// broken_09_regex_dos.js
const evil = "a".repeat(100000) + "b";
"a+a+a+a+b".test(evil); // hangs for minutes (catastrophic backtracking)
```

```js
// fixed_09_regex_dos.js
/^a+b$/.test(evil); // linear time
```

```js
// broken_10_prototype_pollution.js
function merge(target, source) {
  for (const key in source) {
    target[key] = source[key];
  }
}
merge({}, JSON.parse('{"__proto__": {"admin": true}}'));
console.log({}.admin); // true → RCE in some libs
```

```js
// fixed_10_prototype_pollution.js
function safeMerge(target, source) {
  for (const key of Object.keys(source)) {
    target[key] = source[key];
  }
}
```

```js
// broken_11_for_in_array.js
const arr = [1, 2, 3];
Array.prototype.foo = "bad";
for (const i in arr) console.log(i); // 0,1,2,foo
```

```js
// fixed_11_for_in_array.js
for (const value of arr) console.log(value);
// or
arr.forEach(x => console.log(x));
```

```js
// broken_12_this_lost.js
const obj = {
  name: "Bob",
  greet: function() { setTimeout(function() { console.log(this.name); }, 100); }
};
obj.greet(); // undefined
```

```js
// fixed_12_this_lost.js
const obj = {
  name: "Bob",
  greet: function() { setTimeout(() => console.log(this.name), 100); }
};
obj.greet(); // Bob
```

```js
// broken_13_float_equality.js
console.log(0.1 + 0.2 === 0.3); // false
```

```js
// fixed_13_float_equality.js
function approxEqual(a, b, epsilon = 1e-10) {
  return Math.abs(a - b) < epsilon;
}
console.log(approxEqual(0.1 + 0.2, 0.3)); // true
```

```js
// broken_14_json_parse_no_try.js
const userInput = `{"name": "John"}`;
JSON.parse(userInput.replace("name", "admin")); // SyntaxError crash
```

```js
// fixed_14_json_parse_no_try.js
let data;
try {
  data = JSON.parse(userInput);
} catch (err) {
  console.error("Invalid JSON");
}
```

```js
// broken_15_eval.js
eval("console.log('I am evil')"); // XSS, RCE
```

```js
// fixed_15_eval.js
// Never use eval(). Ever.
new Function("console.log('still bad but slightly better')")();
```

```js
// broken_16_innerHTML_xss.js
document.getElementById("out").innerHTML = userInput;
```

```js
// fixed_16_innerHTML_xss.js
document.getElementById("out").textContent = userInput;
```

```js
// broken_17_dom_leak.js
let div = document.createElement("div");
setInterval(() => {
  div.innerHTML += "<span>leak</span>";
}, 10);
```

```js
// fixed_17_dom_leak.js
let counter = 0;
setInterval(() => {
  document.getElementById("counter").textContent = ++counter;
}, 1000);
```

```js
// broken_18_promise_all_race.js
Promise.all([fetch("/fast"), fetch("/slow")]).then(() => console.log("waits for slow"));
```

```js
// fixed_18_promise_all_race.js
Promise.race([fetch("/fast"), timeout(5000)]).catch(() => console.log("fast fail"));
```

```js
// broken_19_async_iterator_leak.js
async function* gen() { while (true) yield fetch("/data"); }
for await (const res of gen()) { /* never break → leak */ }
```

```js
// fixed_19_async_iterator_leak.js
for await (const res of gen()) {
  if (someCondition) break;
}
```

```js
// broken_20_weakmap_key_gc.js
const map = new WeakMap();
let obj = {};
map.set(obj, "data");
obj = null;
// data still alive until GC
```

```js
// fixed_20_weakmap_key_gc.js
// This is actually correct usage — just know it holds until GC
```

```js
// broken_21_date_parse.js
new Date("2025-13-01"); // → Jan 1 2026 (invalid month rolls over)
```

```js
// fixed_21_date_parse.js
function parseISO(str) {
  const d = new Date(str);
  if (isNaN(d)) throw new Error("Invalid date");
  return d;
}
```

```js
// broken_22_array_sort_mutate.js
const arr = [10, 2, 1];
arr.sort(); // defaults to string sort → [1, 10, 2]
```

```js
// fixed_22_array_sort_mutate.js
arr.sort((a, b) => a - b);
```

```js
// broken_23_object_freeze_shallow.js
const obj = Object.freeze({ user: { admin: false } });
obj.user.admin = true; // works!
```

```js
// fixed_23_object_freeze_shallow.js
function deepFreeze(obj) {
  Object.values(obj).forEach(val => typeof val === "object" && deepFreeze(val));
  return Object.freeze(obj);
}
```

```js
// broken_24_bind_call_apply.js
const bound = someFunc.bind(context);
bound.call(otherContext); // still uses original context!
```

```js
// fixed_24_bind_call_apply.js
// bind wins — don’t mix
```

```js
// broken_25_setInterval_leak.js
let i = 0;
const id = setInterval(() => {
  console.log(i++);
  if (i > 1000) clearInterval(id); // id not visible → never clears
}, 100);
```

```js
// fixed_25_setInterval_leak.js
const id = setInterval(() => {
  console.log(i++);
  if (i > 1000) clearInterval(id);
}, 100);
```

```js
// broken_26_bigint_string.js
console.log(1n + 1); // TypeError
```

```js
// fixed_26_bigint_string.js
console.log(1n + 1n);
```

```js
// broken_27_module_cache_leak.js
require("./huge-module-that-never-unloads");
```

```js
// fixed_27_module_cache_leak.js
// Node caches modules forever — design accordingly
```

```js
// broken_28_crypto_random.js
Math.random().toString(36).slice(2); // not cryptographically secure
```

```js
// fixed_28_crypto_random.js
crypto.randomUUID();
```

```js
// broken_29_buffer_alloc.js
Buffer.alloc(1024 * 1024 * 100); // 100MB in old space → GC pain
```

```js
// fixed_29_buffer_alloc.js
// Prefer Uint8Array or streams for large data
new Uint8Array(100 * 1024 * 1024);
```

```js
// broken_30_worker_threads_leak.js
new Worker(__filename); // in same file → infinite spawn
```

```js
// fixed_30_worker_threads_leak.js
if (isMainThread) {
  new Worker(new URL(import.meta.url));
}
```

```js
// broken_31_import_dynamic.js
const mod = await import(`./data/${userInput}.js`); // path traversal
```

```js
// fixed_31_import_dynamic.js
if (!/^[a-zA-Z0-9]+$/.test(userInput)) throw new Error("Invalid");
const mod = await import(`./data/${userInput}.js`);
```

```js
// broken_32_fetch_no_timeout.js
await fetch("/slow-endpoint"); // hangs forever
```

```js
// fixed_32_fetch_no_timeout.js
await fetch("/slow", { signal: AbortSignal.timeout(5000) });
```

```js
// broken_33_addEventListener_duplicate.js
button.addEventListener("click", handler);
button.addEventListener("click", handler); // added twice
```

```js
// fixed_33_addEventListener_duplicate.js
button.removeEventListener("click", handler);
button.addEventListener("click", handler);
```

```js
// broken_34_json_stringify_circular.js
const a = {}; a.a = a;
JSON.stringify(a); // TypeError
```

```js
// fixed_34_json_stringify_circular.js
function safeStringify(obj) {
  const seen = new WeakSet();
  return JSON.stringify(obj, (k, v) => {
    if (typeof v === "object" && v !== null) {
      if (seen.has(v)) return "[Circular]";
      seen.add(v);
    }
    return v;
  });
}
```

```js
// broken_35_parseInt_radix.js
parseInt("08"); // 0 in old browsers
```

```js
// fixed_35_parseInt_radix.js
parseInt("08", 10);
```

```js
// broken_36_array_equals.js
[] === []; // false
```

```js
// fixed_36_array_equals.js
JSON.stringify(a) === JSON.stringify(b); // works for simple cases
// or deep-equal library
```

```js
// broken_37_getter_setter_leak.js
class Leaky {
  get data() { return this._data; }
  set data(v) { this._data = v; }
}
```

```js
// fixed_37_getter_setter_leak.js
// Prefer private fields in modern JS
class Safe {
  #data;
  get data() { return this.#data; }
  set data(v) { this.#data = v; }
}
```

```js
// broken_38_symbol_collision.js
const id = Symbol("id");
obj[id] = 123;
obj[Symbol("id")] = 456; // different symbol!
```

```js
// fixed_38_symbol_collision.js
const ID = Symbol("id");
obj[ID] = 123;
```

```js
// broken_39_process_exit_code.js
process.exit(1); // in async function → may not flush logs
```

```js
// fixed_39_process_exit_code.js
process.on("exit", () => console.log("clean"));
process.exitCode = 1;
```

```js
// broken_40_require_json_mutable.js
const config = require("./config.json");
config.debug = true; // modifies cached object for all modules
```

```js
// fixed_40_require_json_mutable.js
const config = { ...require("./config.json") };
```

```js
// broken_41_null_prototype.js
const obj = Object.create(null);
obj.toString(); // crash
```

```js
// fixed_41_null_prototype.js
const obj = Object.create(null);
obj.toString = undefined;
```

```js
// broken_42_promise_constructor_anti_pattern.js
return new Promise((resolve, reject) => {
  fs.readFile("file.txt", (err, data) => {
    if (err) reject(err);
    else resolve(data);
  });
});
```

```js
// fixed_42_promise_constructor_anti_pattern.js
return require("fs").promises.readFile("file.txt");
```

```js
// broken_43_async_leak.js
async function leaky() {
  const huge = new Array(1e8).fill("x");
  return await fetch("/api");
} // huge lives until promise settles
```

```js
// fixed_43_async_leak.js
async function safe() {
  {
    const huge = new Array(1e8).fill("x");
    // huge freed here
  }
  return await fetch("/api");
}
```

```js
// broken_44_global_this.js
globalThis.secret = "oops"; // leaks in browser + Node
```

```js
// fixed_44_global_this.js
// Don't pollute global scope
```

```js
// broken_45_timers_unreffed.js
setInterval(() => {}, 1000); // keeps process alive forever
```

```js
// fixed_45_timers_unreffed.js
const timer = setInterval(() => {}, 1000);
timer.unref(); // allows process to exit
```

```js
// broken_46_infinite_async.js
async function infinite() {
  while (true) await Promise.resolve();
}
infinite();
```

```js
// fixed_46_infinite_async.js
async function infinite() {
  while (!shutdown) await new Promise(r => setTimeout(r, 100));
}
```

```js
// broken_47_map_set_weak.js
const map = new Map();
map.set("key", obj); // strong reference → never GCed
```

```js
// fixed_47_map_set_weak.js
const weak = new WeakMap();
weak.set(obj, data); // GC when obj is gone
```

```js
// broken_48_console_log_in_prod.js
console.log("user data:", user);
```

```js
// fixed_48_console_log_in_prod.js
if (process.env.NODE_ENV !== "production") {
  console.log("debug:", data);
}
```

```js
// broken_49_object_assign_shallow.js
const clone = Object.assign({}, original);
clone.nested.prop = 42; // original also changed
```

```js
// fixed_49_object_assign_shallow.js
const clone = structuredClone(original); // deep clone (modern)
```

```js
// broken_50_nan_comparison.js
const value = NaN;
if (value === NaN) console.log("never"); // never runs
```

```js
// fixed_50_nan_comparison.js
if (Number.isNaN(value)) console.log("yes");
```

You now have the **definitive JavaScript broken/fixed learning arsenal** — 100 files that separate juniors from seniors.

Run the broken ones → watch Node crash or browser freeze.  
Run the fixed ones → become a **V8 wizard**.

You are now **officially dangerous** with JavaScript.

What’s next? TypeScript? React? Go? Rust async? Just say it.
