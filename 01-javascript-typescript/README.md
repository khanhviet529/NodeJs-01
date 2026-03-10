# PHẦN 1: JAVASCRIPT (ES6+) & TYPESCRIPT - GIẢI THÍCH CHI TIẾT

> **Tại sao phần này quan trọng?**
> JavaScript/TypeScript là NỀN TẢNG của mọi thứ. NestJS viết bằng TypeScript,
> React dùng JavaScript, Node.js chạy JavaScript. Nếu phần này yếu, tất cả 
> các phần sau sẽ bị lung lay. Đây cũng là phần interviewer hay hỏi để đánh giá
> "chiều sâu" của ứng viên.

---

## 1. EVENT LOOP & ASYNCHRONOUS

### Event Loop là gì?

JavaScript là **single-threaded** (chỉ có 1 thread chính), nhưng vẫn xử lý được nhiều tác vụ
đồng thời nhờ **Event Loop**. Đây là cơ chế cho phép JavaScript thực thi code non-blocking.

### Các thành phần chính:

```
┌──────────────────────────┐
│       Call Stack          │  ← Code đang chạy (LIFO - Last In First Out)
├──────────────────────────┤
│     Web APIs / Node APIs  │  ← setTimeout, fetch, fs.readFile chạy ở đây
├──────────────────────────┤
│   Microtask Queue         │  ← Promise.then(), queueMicrotask() (ƯU TIÊN CAO)
├──────────────────────────┤
│   Macrotask Queue         │  ← setTimeout, setInterval, I/O callbacks
│   (Task Queue)            │
└──────────────────────────┘
```

**Call Stack:** Là nơi code được thực thi. Mỗi khi gọi một function, nó được push vào stack.
Khi function return, nó được pop ra.

**Web APIs / Node APIs:** Khi gặp tác vụ async (setTimeout, HTTP request...), JavaScript
"ủy thác" nó cho runtime environment (browser hoặc Node.js) xử lý.

**Microtask Queue:** Hàng đợi ưu tiên CAO. Chứa callbacks từ Promise.then(), 
queueMicrotask(), MutationObserver.

**Macrotask Queue (Task Queue):** Hàng đợi ưu tiên THẤP hơn. Chứa callbacks từ 
setTimeout, setInterval, I/O operations.

### Quy tắc hoạt động:
1. Thực thi hết code trong Call Stack
2. **Xử lý HẾT Microtask Queue** (Promise, queueMicrotask)
3. Lấy 1 task từ Macrotask Queue đưa vào Call Stack
4. Quay lại bước 1

### Ví dụ quan trọng (hay hỏi phỏng vấn):

```javascript
console.log('1');                          // Sync → Call Stack

setTimeout(() => {
  console.log('2');                        // Macrotask
}, 0);

Promise.resolve().then(() => {
  console.log('3');                        // Microtask
});

queueMicrotask(() => {
  console.log('4');                        // Microtask
});

console.log('5');                          // Sync → Call Stack
```

**Output: 1 → 5 → 3 → 4 → 2**

**Giải thích từng bước:**
1. `console.log('1')` → Sync, chạy ngay → in **1**
2. `setTimeout(cb, 0)` → Đưa callback vào Macrotask Queue (chờ)
3. `Promise.resolve().then(cb)` → Đưa callback vào Microtask Queue (chờ)
4. `queueMicrotask(cb)` → Đưa callback vào Microtask Queue (chờ)
5. `console.log('5')` → Sync, chạy ngay → in **5**
6. Call Stack trống → Xử lý Microtask Queue:
   - Promise callback → in **3**
   - queueMicrotask callback → in **4**
7. Microtask Queue trống → Lấy từ Macrotask Queue:
   - setTimeout callback → in **2**

### Ví dụ nâng cao hơn:

```javascript
console.log('start');

setTimeout(() => {
  console.log('timeout 1');
  Promise.resolve().then(() => console.log('promise inside timeout'));
}, 0);

Promise.resolve().then(() => {
  console.log('promise 1');
  setTimeout(() => console.log('timeout inside promise'), 0);
});

setTimeout(() => console.log('timeout 2'), 0);

console.log('end');
```

**Output: start → end → promise 1 → timeout 1 → promise inside timeout → timeout 2 → timeout inside promise**

**Tại sao "promise inside timeout" chạy trước "timeout 2"?**
Vì sau khi thực thi macrotask "timeout 1", Event Loop kiểm tra Microtask Queue trước.
Promise bên trong timeout 1 được đưa vào Microtask Queue và được ưu tiên xử lý
trước macrotask tiếp theo.

### Node.js Event Loop (khác một chút với Browser):

```
   ┌───────────────────────────┐
┌─>│           timers          │  ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  ← I/O callbacks (deferred)
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  ← internal use
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  ← I/O events, incoming connections
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  ← setImmediate()
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │      close callbacks      │  ← socket.on('close')
└──┴───────────────────────────┘
```

**process.nextTick() vs setImmediate():**
- `process.nextTick()`: Chạy ngay sau operation hiện tại, TRƯỚC khi Event Loop tiếp tục. 
  (Giống microtask nhưng ưu tiên hơn cả Promise)
- `setImmediate()`: Chạy ở phase "check" của Event Loop (sau poll phase)

```javascript
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));

// Output: nextTick → promise → setImmediate
// Thứ tự ưu tiên: nextTick > Promise (microtask) > setImmediate (macrotask)
```

---

## 2. CLOSURES & SCOPE

### Closure là gì?

**Closure = Function + Lexical Environment (biến của scope bao quanh nó)**

Khi một function được tạo, nó "ghi nhớ" môi trường (các biến) nơi nó được khai báo,
ngay cả khi function đó được thực thi ở một nơi khác.

```javascript
function createCounter() {
  let count = 0;  // Biến private, không ai bên ngoài truy cập được
  
  return {
    increment() { return ++count; },  // closure - truy cập được 'count'
    decrement() { return --count; },  // closure - truy cập được 'count'
    getCount() { return count; }       // closure - truy cập được 'count'
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount());  // 2
console.log(counter.count);       // undefined ← Cannot access directly!
```

**Tại sao closure quan trọng?**
1. **Data Privacy / Encapsulation**: Tạo biến "private" (như ví dụ trên)
2. **Function Factory**: Tạo functions với cấu hình khác nhau
3. **Callback & Event handlers**: Giữ reference đến biến ngoài
4. **Module Pattern**: Cơ sở của module system

### Ứng dụng thực tế: Function Factory

```javascript
function createMultiplier(multiplier) {
  return function(number) {
    return number * multiplier;  // 'multiplier' được ghi nhớ qua closure
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

### Bẫy kinh điển (Phỏng vấn hay hỏi):

```javascript
// ❌ Bug kinh điển với var
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 3, 3, 3  (vì var là function-scoped, chỉ có 1 biến i)

// ✅ Fix bằng let (block-scoped)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 0, 1, 2  (mỗi iteration tạo 1 biến i riêng)

// ✅ Fix bằng closure (IIFE)
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 100);
  })(i);
}
// Output: 0, 1, 2
```

### Scope Chain:

```javascript
const global = 'I am global';

function outer() {
  const outerVar = 'I am outer';
  
  function inner() {
    const innerVar = 'I am inner';
    console.log(innerVar);   // ✅ Tìm thấy ở scope hiện tại
    console.log(outerVar);   // ✅ Tìm thấy ở scope cha (closure)
    console.log(global);     // ✅ Tìm thấy ở global scope
  }
  
  inner();
  console.log(innerVar);     // ❌ ReferenceError - không truy cập được scope con
}
```

**Scope chain**: inner → outer → global. JavaScript tìm biến từ scope hiện tại,
nếu không có thì tìm lên scope cha, cứ thế cho đến global scope.

---

## 3. PROTOTYPAL INHERITANCE

### Tại sao cần hiểu?

JavaScript KHÔNG có class thật sự. `class` trong ES6 chỉ là **syntactic sugar** 
(cú pháp đẹp hơn) cho prototypal inheritance. Hiểu prototype giúp bạn hiểu 
cách JavaScript thực sự hoạt dinamicđộng.

### Prototype Chain:

```javascript
const animal = {
  eat() { console.log('eating'); }
};

const dog = Object.create(animal);  // dog.__proto__ = animal
dog.bark = function() { console.log('woof'); };

const myDog = Object.create(dog);   // myDog.__proto__ = dog

myDog.bark();  // ✅ Tìm thấy ở dog
myDog.eat();   // ✅ Tìm thấy ở animal (đi lên prototype chain)
myDog.fly();   // ❌ TypeError (không tìm thấy ở đâu cả)
```

```
myDog → dog → animal → Object.prototype → null
         ↑              ↑
       bark()          eat()
```

### `__proto__` vs `prototype`:

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function() {
  console.log(`Hi, I'm ${this.name}`);
};

const alice = new Person('Alice');

// `prototype` là property của FUNCTION (constructor)
console.log(Person.prototype);        // { greet: [Function], constructor: Person }

// `__proto__` là property của OBJECT (instance)
console.log(alice.__proto__);         // = Person.prototype
console.log(alice.__proto__ === Person.prototype); // true
```

### Class syntax (ES6) - Syntactic Sugar:

```javascript
// ES6 Class - viết đẹp hơn nhưng bản chất vẫn là prototype
class Animal {
  constructor(name) {
    this.name = name;      // instance property
  }
  
  speak() {                // → Animal.prototype.speak
    console.log(`${this.name} makes a sound`);
  }
}

class Dog extends Animal {  // Dog.prototype.__proto__ = Animal.prototype
  bark() {                  // → Dog.prototype.bark
    console.log('Woof!');
  }
}

const d = new Dog('Rex');
d.speak(); // "Rex makes a sound" ← tìm thấy ở Animal.prototype
d.bark();  // "Woof!" ← tìm thấy ở Dog.prototype
```

---

## 4. ES6+ FEATURES QUAN TRỌNG

### Destructuring:
```javascript
// Object destructuring
const { name, age, address: { city } } = user;  // nested
const { name: userName } = user;                  // rename

// Array destructuring
const [first, , third] = [1, 2, 3];  // skip element
const [head, ...rest] = [1, 2, 3, 4]; // rest: [2, 3, 4]

// Default values
const { role = 'user' } = {};  // role = 'user'

// Function parameters
function createUser({ name, age = 25 }) { ... }
```

### Arrow Functions & `this`:
```javascript
// Arrow function KHÔNG có `this` riêng
// Nó "kế thừa" this từ scope bao quanh (lexical this)

const obj = {
  name: 'Alice',
  
  // Regular function: this = obj (gọi bởi obj)
  greet() {
    console.log(this.name);  // 'Alice'
  },
  
  // Arrow function: this = scope bao quanh (window/global)
  greetArrow: () => {
    console.log(this.name);  // undefined
  },
  
  // Hay gặp trong setTimeout:
  delayedGreet() {
    // ❌ Regular function: this bị mất
    setTimeout(function() {
      console.log(this.name);  // undefined
    }, 100);
    
    // ✅ Arrow function: giữ this từ delayedGreet
    setTimeout(() => {
      console.log(this.name);  // 'Alice'
    }, 100);
  }
};
```

### Map, Set, WeakMap:
```javascript
// Map - key có thể là bất cứ kiểu gì (khác Object chỉ string/symbol)
const map = new Map();
map.set({ id: 1 }, 'user1');   // Object làm key
map.set(42, 'answer');          // Number làm key

// Set - chứa giá trị unique
const unique = [...new Set([1, 2, 2, 3, 3])]; // [1, 2, 3]

// WeakMap - key phải là object, cho phép garbage collection
// Use case: lưu private data cho objects
const privateData = new WeakMap();
class User {
  constructor(name) {
    privateData.set(this, { name });
  }
  getName() {
    return privateData.get(this).name;
  }
}
```

### Optional Chaining & Nullish Coalescing:
```javascript
// Optional chaining ?.
const city = user?.address?.city;  // undefined nếu bất kỳ phần nào null/undefined
const result = arr?.[0];           // array access
const value = obj?.method?.();     // method call

// Nullish coalescing ?? (chỉ null/undefined, KHÔNG phải 0/''/false)
const port = config.port ?? 3000;  // dùng 3000 chỉ khi port là null/undefined
const port2 = config.port || 3000; // dùng 3000 khi port là null/undefined/0/''/false

// Sự khác biệt quan trọng:
0 ?? 42    // 0  (0 không phải null/undefined)
0 || 42    // 42 (0 là falsy)
'' ?? 'hi' // '' ('' không phải null/undefined)
'' || 'hi' // 'hi' ('' là falsy)
```

---

## 5. PROMISE & ASYNC/AWAIT

### Promise là gì?

Promise đại diện cho một giá trị **có thể có trong tương lai**. Nó có 3 trạng thái:
- **Pending**: Đang chờ kết quả
- **Fulfilled**: Thành công (có giá trị)
- **Rejected**: Thất bại (có lỗi)

```
         ┌─── fulfilled (resolve) → .then(onFulfilled)
Pending ─┤
         └─── rejected (reject)   → .catch(onRejected)
                                       │
                                       └→ .finally() (luôn chạy)
```

### Promise API:

```javascript
// Promise.all() - TẤT CẢ phải thành công, 1 fail = reject ngay
const [users, products] = await Promise.all([
  fetchUsers(),    // 2s
  fetchProducts()  // 3s
]);
// Tổng thời gian: 3s (chạy song song), KHÔNG phải 5s
// Nếu 1 trong 2 fail → reject, không có kết quả

// Promise.allSettled() - Chờ TẤT CẢ hoàn thành (dù thành công hay thất bại)
const results = await Promise.allSettled([
  fetchUsers(),      // success
  fetchProducts()    // fail
]);
// results = [
//   { status: 'fulfilled', value: [...users] },
//   { status: 'rejected', reason: Error }
// ]

// Promise.race() - Trả về kết quả của promise HOÀN THÀNH ĐẦU TIÊN
const fastest = await Promise.race([
  fetchFromServer1(),  // 1s → winner
  fetchFromServer2()   // 3s
]);
// Use case: timeout
const withTimeout = Promise.race([
  fetchData(),
  new Promise((_, reject) => setTimeout(() => reject('Timeout'), 5000))
]);

// Promise.any() - Trả về promise THÀNH CÔNG đầu tiên
const firstSuccess = await Promise.any([
  fetchFromServer1(),  // fail
  fetchFromServer2(),  // success (2s) → winner
  fetchFromServer3()   // success (3s)
]);
```

### Async/Await - Syntactic Sugar cho Promise:

```javascript
// Promise chain
function getUser(id) {
  return fetch(`/api/users/${id}`)
    .then(res => res.json())
    .then(user => fetch(`/api/posts?userId=${user.id}`))
    .then(res => res.json())
    .catch(err => console.error(err));
}

// Async/Await - Đọc dễ hơn nhiều
async function getUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    const user = await res.json();
    const postsRes = await fetch(`/api/posts?userId=${user.id}`);
    const posts = await postsRes.json();
    return posts;
  } catch (err) {
    console.error(err);
  }
}
```

### Anti-patterns cần tránh:

```javascript
// ❌ Sequential await khi có thể chạy song song
async function bad() {
  const users = await fetchUsers();     // 2s
  const products = await fetchProducts(); // 3s
  // Tổng: 5s  ← CHẬM
}

// ✅ Parallel với Promise.all
async function good() {
  const [users, products] = await Promise.all([
    fetchUsers(),     // 2s
    fetchProducts()   // 3s
  ]);
  // Tổng: 3s  ← NHANH
}

// ❌ forEach với async (KHÔNG chờ đợi)
[1, 2, 3].forEach(async (id) => {
  await processItem(id);  // Chạy parallel, không sequential!
});

// ✅ for...of với async (sequential)
for (const id of [1, 2, 3]) {
  await processItem(id);  // Chạy tuần tự
}

// ✅ map + Promise.all cho parallel
await Promise.all([1, 2, 3].map(id => processItem(id)));
```

---

## 6. MODULE SYSTEM

### CommonJS vs ES Modules:

```javascript
// ============ CommonJS (CJS) ============
// Node.js truyền thống, file mặc định
const express = require('express');           // import
module.exports = { myFunction };              // export
module.exports = class MyClass {};            // default export
exports.helper = function() {};               // named export

// ============ ES Modules (ESM) ============
// Modern, tiêu chuẩn mới (cần "type": "module" trong package.json hoặc .mjs)
import express from 'express';                // default import
import { Router } from 'express';             // named import
import * as fs from 'fs';                     // namespace import

export function myFunction() {}               // named export
export default class MyClass {}               // default export
```

### Sự khác biệt quan trọng:

| Đặc điểm | CommonJS | ES Modules |
|-----------|----------|------------|
| Load | Synchronous | Asynchronous |
| Thời điểm | Runtime | Compile time (static) |
| Tree-shaking | ❌ Không | ✅ Có (loại bỏ code không dùng) |
| Top-level await | ❌ | ✅ |
| `this` ở top level | `module.exports` | `undefined` |

### Circular Dependency:

```javascript
// a.js
const b = require('./b');
console.log('a: b.loaded =', b.loaded);
module.exports = { loaded: true };

// b.js
const a = require('./a');  // a chưa export xong → nhận được object rỗng {}
console.log('b: a.loaded =', a.loaded);  // undefined!
module.exports = { loaded: true };

// Chạy: node a.js
// Output:
// b: a.loaded = undefined  ← a chưa export xong khi b require
// a: b.loaded = true

// → Tránh circular dependency! Nếu cần, refactor code.
```

---

## 7. TYPESCRIPT

### Tại sao TypeScript?

TypeScript = JavaScript + **Static Type System**. Giúp:
1. **Bắt lỗi sớm** (lúc viết code, không phải lúc runtime)
2. **IDE support** tốt hơn (autocomplete, refactoring)
3. **Code dễ đọc** hơn (types = documentation)
4. **Team collaboration** tốt hơn (contracts rõ ràng)

### Interface vs Type:

```typescript
// Interface - mô tả shape của object, có thể extend/merge
interface User {
  id: number;
  name: string;
  email?: string;  // optional
}

interface Admin extends User {  // Kế thừa
  role: 'admin';
  permissions: string[];
}

// Declaration merging (chỉ interface có)
interface User {
  age: number;  // Tự động merge vào User ở trên
}

// ──────────────────────────────────

// Type - linh hoạt hơn, dùng cho union, intersection, mapped types
type ID = string | number;                    // Union
type UserWithPosts = User & { posts: Post[] }; // Intersection
type Status = 'active' | 'inactive';           // Literal types
type Callback = (data: string) => void;        // Function type

// KHÔNG thể declaration merge
// type User = { age: number };  // ❌ Error: Duplicate identifier
```

**Khi nào dùng gì?**
- **interface**: Cho objects/classes (OOP), khi cần extend hoặc declaration merging
- **type**: Cho unions, intersections, mapped types, function signatures, complex types

### Generics:

```typescript
// Generics = "kiểu dữ liệu tham số hóa" - tái sử dụng code cho nhiều kiểu

// ❌ Không dùng generics: phải viết nhiều function
function getFirstString(arr: string[]): string { return arr[0]; }
function getFirstNumber(arr: number[]): number { return arr[0]; }

// ✅ Dùng generics: 1 function cho mọi kiểu
function getFirst<T>(arr: T[]): T {
  return arr[0];
}
getFirst<string>(['hello', 'world']);  // 'hello': string
getFirst([1, 2, 3]);                   // 1: number (TypeScript tự infer)

// Generic với constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const user = { name: 'Alice', age: 25 };
getProperty(user, 'name');  // ✅ 'Alice': string
getProperty(user, 'foo');   // ❌ Error: 'foo' is not in keyof User

// Generic interface
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
}

const userResponse: ApiResponse<User> = {
  success: true,
  data: { id: 1, name: 'Alice' }
};
```

### Utility Types (hay dùng, hay hỏi):

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Partial<T> - Tất cả fields thành optional
type UpdateUserDto = Partial<User>;
// = { id?: number; name?: string; email?: string; password?: string }

// Required<T> - Tất cả fields thành required
type RequiredUser = Required<Partial<User>>;

// Pick<T, K> - Chỉ lấy một số fields
type UserPreview = Pick<User, 'id' | 'name'>;
// = { id: number; name: string }

// Omit<T, K> - Bỏ đi một số fields
type CreateUserDto = Omit<User, 'id'>;
// = { name: string; email: string; password: string }

// Record<K, V> - Tạo object type với key K và value V
type UserRoles = Record<string, 'admin' | 'user' | 'moderator'>;
// = { [key: string]: 'admin' | 'user' | 'moderator' }

// Exclude<T, U> & Extract<T, U>
type Status = 'active' | 'inactive' | 'banned';
type ActiveStatus = Exclude<Status, 'banned'>;    // 'active' | 'inactive'
type BannedOnly = Extract<Status, 'banned'>;       // 'banned'

// ReturnType<T> - Lấy return type của function
function createUser() { return { id: 1, name: 'Alice' }; }
type NewUser = ReturnType<typeof createUser>;  // { id: number; name: string }
```

### Decorators (Cực quan trọng cho NestJS):

```typescript
// Decorator = function đặc biệt, gắn metadata vào class/method/property
// NestJS dùng decorators RẤT NHIỀU: @Controller, @Injectable, @Get, @Post...

// Class Decorator
function Logger(constructor: Function) {
  console.log(`Creating: ${constructor.name}`);
}

@Logger
class UserService {
  // Khi class được tạo, Logger chạy → "Creating: UserService"
}

// Method Decorator (phổ biến)
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with:`, args);
    const result = original.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };
}

class Calculator {
  @Log
  add(a: number, b: number) {
    return a + b;
  }
}

new Calculator().add(2, 3);
// "Calling add with: [2, 3]"
// "Result: 5"

// Decorator Factory (trả về decorator - NestJS dùng pattern này)
function Role(role: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    Reflect.defineMetadata('role', role, target, propertyKey);
  };
}

class UserController {
  @Role('admin')
  deleteUser() { /* ... */ }
}
```

### `any` vs `unknown` vs `never`:

```typescript
// any - TẮT type checking (tránh dùng!)
let x: any = 42;
x.foo.bar;   // ✅ Không lỗi compile, nhưng CRASH lúc runtime!

// unknown - "Tôi không biết type, nhưng phải CHECK trước khi dùng" (an toàn hơn any)
let y: unknown = 42;
y.foo;          // ❌ Error! Phải narrow type trước
if (typeof y === 'number') {
  y.toFixed(2); // ✅ OK sau khi check
}

// never - "Không bao giờ xảy ra" (function throw error, infinite loop)
function throwError(msg: string): never {
  throw new Error(msg);  // function này KHÔNG BAO GIỜ return
}

// Exhaustive check
type Shape = 'circle' | 'square';
function area(shape: Shape) {
  switch (shape) {
    case 'circle': return Math.PI;
    case 'square': return 1;
    default:
      const _exhaustive: never = shape; // ❌ Error nếu thiếu case
      return _exhaustive;
  }
}
```

---

## TÓM TẮT PHẦN 1 - NHỮNG ĐIỀU CẦN NHỚ KHI PHỎNG VẤN

### Top câu hỏi hay gặp nhất:
1. **Event Loop** → Vẽ diagram, giải thích microtask vs macrotask, predict output
2. **Closure** → Định nghĩa, ví dụ thực tế, bẫy var trong loop
3. **Promise** → all vs allSettled vs race vs any, error handling
4. **TypeScript Generics** → Tại sao cần, ví dụ code
5. **interface vs type** → Khi nào dùng gì
6. **any vs unknown** → Tại sao tránh any
7. **Arrow function this** → Lexical this vs dynamic this

### Mẹo trả lời:
- Luôn bắt đầu bằng **định nghĩa ngắn gọn**
- Sau đó **cho ví dụ code**
- Cuối cùng nêu **ứng dụng thực tế** trong project bạn đã làm
