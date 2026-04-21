# Phase 3 — Function, `this`, Closure trong JavaScript

## 1. Mục tiêu của Phase 3

Sau Phase 3, bạn cần hiểu rõ:

- Function trong JavaScript không chỉ là “hàm để chạy code”, mà còn là **first-class value**
- Sự khác nhau giữa **function declaration**, **function expression**, **arrow function**
- `arguments`, **default parameter**, **rest parameter** dùng trong tình huống nào
- `this` thay đổi theo **cách gọi hàm**, không chỉ theo chỗ viết hàm
- `call`, `apply`, `bind` dùng để điều khiển `this`
- **lexical scope** và **closure** là nền tảng rất quan trọng trong JavaScript

---

## 2. Sơ đồ tổng quan

```text
Phase 3
├─ Function cơ bản
│  ├─ declaration
│  ├─ expression
│  └─ arrow function
├─ Tham số hàm
│  ├─ default parameter
│  ├─ arguments
│  └─ rest parameter
├─ Function nâng cao
│  ├─ callback
│  └─ higher-order function
├─ this
│  ├─ function thường
│  ├─ method trong object
│  ├─ arrow function
│  └─ call / apply / bind
└─ Scope nâng cao
   ├─ lexical scope
   └─ closure
```

---

## 3. Function trong JavaScript là gì?

Function là một khối code có thể:

- nhận input
- xử lý logic
- trả kết quả
- được gán vào biến
- được truyền vào hàm khác
- được trả về từ một hàm khác

Ví dụ:

```js
function add(a, b) {
  return a + b;
}

console.log(add(2, 3)); // 5
```

### Điểm rất quan trọng
Trong JavaScript, function là **first-class citizen**.

Nghĩa là function có thể được đối xử như dữ liệu:

```js
function sayHello() {
  console.log("Hello");
}

const fn = sayHello;
fn(); // Hello
```

---

## 4. Function Declaration và Function Expression

## 4.1. Function Declaration

Là kiểu khai báo hàm quen thuộc:

```js
function greet(name) {
  return `Hello ${name}`;
}
```

### Đặc điểm
- có tên hàm
- được **hoisting đầy đủ**
- có thể gọi trước khi định nghĩa trong code

```js
sayHi();

function sayHi() {
  console.log("Hi");
}
```

---

## 4.2. Function Expression

Là hàm được gán vào biến:

```js
const greet = function(name) {
  return `Hello ${name}`;
};
```

### Đặc điểm
- hàm là một giá trị được gán vào biến
- không được hoisting giống function declaration
- nếu biến là `let` hoặc `const`, truy cập trước khai báo sẽ lỗi

```js
// greet(); // ReferenceError

const greet = function() {
  console.log("Hi");
};
```

---

## 4.3. So sánh nhanh

```text
Function Declaration
- hoisting đầy đủ
- gọi trước được

Function Expression
- gán vào biến
- chịu ảnh hưởng bởi let/const/var
- thường không gọi trước được
```

---

## 5. Arrow Function

Arrow function là cú pháp ngắn gọn hơn để viết hàm.

```js
const add = (a, b) => {
  return a + b;
};
```

Nếu chỉ có một biểu thức trả về:

```js
const add = (a, b) => a + b;
```

### Ví dụ

```js
const square = x => x * x;
console.log(square(4)); // 16
```

### Arrow function khác function thường ở đâu?

Không chỉ khác cú pháp. Khác quan trọng nhất là:

- arrow function **không có `this` riêng**
- arrow function **không có `arguments` riêng**
- arrow function không thích hợp để làm constructor với `new`

---

## 6. Tham số hàm

## 6.1. Parameter và Argument

- **Parameter**: biến khai báo trong phần định nghĩa hàm
- **Argument**: giá trị truyền vào khi gọi hàm

```js
function greet(name) { // name là parameter
  console.log(name);
}

greet("An"); // "An" là argument
```

---

## 6.2. Default Parameter

Cho phép đặt giá trị mặc định cho tham số.

```js
function greet(name = "Guest") {
  console.log(`Hello ${name}`);
}

greet();       // Hello Guest
greet("An");  // Hello An
```

### Khi dùng?
Dùng khi muốn hàm vẫn hoạt động nếu người dùng không truyền đối số.

---

## 6.3. `arguments`

`arguments` là một object đặc biệt có mặt trong **function thường**.

Nó chứa toàn bộ đối số đã truyền vào.

```js
function showArgs() {
  console.log(arguments);
}

showArgs(1, 2, 3);
```

### Đặc điểm
- giống array nhưng **không phải array thật**
- không dùng được trực tiếp mọi method của array như `map`, `filter`
- **arrow function không có `arguments` riêng**

Ví dụ:

```js
function sum() {
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}

console.log(sum(1, 2, 3)); // 6
```

---

## 6.4. Rest Parameter

Rest parameter gom nhiều đối số còn lại thành **một array thật**.

```js
function sum(...nums) {
  return nums.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3, 4)); // 10
```

### So sánh `arguments` và rest

```text
arguments
- chỉ có trong function thường
- không phải array thật
- là object dạng array-like

...rest
- cú pháp hiện đại hơn
- là array thật
- dùng tiện hơn nhiều
```

### Kết luận
Trong code hiện đại, thường ưu tiên **rest parameter** hơn `arguments`.

---

## 7. Return trong function

`return` dùng để trả kết quả từ hàm.

```js
function add(a, b) {
  return a + b;
}
```

Nếu không có `return`, hàm sẽ trả về `undefined`.

```js
function test() {
  console.log("hello");
}

console.log(test()); // undefined
```

---

## 8. Callback Function

Callback là hàm được truyền vào một hàm khác để được gọi sau.

```js
function processUser(name, callback) {
  console.log("Đang xử lý user...");
  callback(name);
}

function sayHello(name) {
  console.log(`Xin chào ${name}`);
}

processUser("An", sayHello);
```

### Callback dùng nhiều ở đâu?
- xử lý sự kiện
- xử lý async
- array methods như `map`, `filter`, `forEach`

Ví dụ:

```js
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6]
```

Ở đây `n => n * 2` là callback.

---

## 9. Higher-Order Function

Là hàm mà:

- nhận hàm khác làm đối số, hoặc
- trả về một hàm khác

Ví dụ nhận callback:

```js
function run(callback) {
  callback();
}
```

Ví dụ trả về function:

```js
function multiplyBy(x) {
  return function(y) {
    return x * y;
  };
}

const double = multiplyBy(2);
console.log(double(5)); // 10
```

Higher-order function là nền tảng của closure và lập trình hàm trong JS.

---

## 10. `this` là gì?

`this` là một từ khóa đặc biệt, đại diện cho **ngữ cảnh thực thi hiện tại**.

Điểm rất quan trọng:

> `this` không phụ thuộc chủ yếu vào nơi hàm được viết, mà phụ thuộc nhiều vào **cách hàm được gọi**.

---

## 11. `this` trong function thường

Trong function thường, `this` thay đổi tùy cách gọi.

```js
function showThis() {
  console.log(this);
}

showThis();
```

Trong trình duyệt non-strict mode, thường là `window`.
Trong strict mode, thường là `undefined`.

```js
"use strict";

function showThis() {
  console.log(this);
}

showThis(); // undefined
```

---

## 12. `this` trong object method

Khi hàm được gọi như một method của object, `this` thường trỏ tới object đó.

```js
const user = {
  name: "An",
  greet() {
    console.log(this.name);
  }
};

user.greet(); // An
```

Ở đây `this` là `user`.

---

## 13. `this` trong arrow function

Arrow function **không có `this` riêng**.
Nó lấy `this` từ scope bên ngoài theo kiểu lexical.

Ví dụ:

```js
const user = {
  name: "An",
  greet: () => {
    console.log(this.name);
  }
};

user.greet();
```

Kết quả thường **không phải** `An`.
Vì `this` trong arrow không trỏ vào `user`, mà lấy từ môi trường ngoài.

### Kết luận quan trọng
- method trong object thường nên dùng **function thường**
- không nên lạm dụng arrow function cho object method nếu bạn cần `this`

---

## 14. So sánh `this` giữa function thường và arrow

```text
Function thường
- có this riêng
- this phụ thuộc cách gọi

Arrow function
- không có this riêng
- lấy this từ scope bên ngoài
```

---

## 15. Mất ngữ cảnh `this`

Đây là lỗi rất hay gặp.

```js
const user = {
  name: "An",
  greet() {
    console.log(this.name);
  }
};

const fn = user.greet;
fn();
```

Khi này `fn()` được gọi như function thường, không còn là method của `user` nữa.
Nên `this` không còn là `user`.

---

## 16. `call`, `apply`, `bind`

Ba phương thức này dùng để **điều khiển `this`**.

## 16.1. `call`

Gọi hàm ngay lập tức và truyền `this` + từng đối số riêng lẻ.

```js
function greet(city) {
  console.log(`Tôi là ${this.name}, ở ${city}`);
}

const user = { name: "An" };

greet.call(user, "Hà Nội");
```

---

## 16.2. `apply`

Gần giống `call`, nhưng đối số truyền dưới dạng array.

```js
function greet(city, country) {
  console.log(`Tôi là ${this.name}, ở ${city}, ${country}`);
}

const user = { name: "An" };

greet.apply(user, ["Hà Nội", "Việt Nam"]);
```

---

## 16.3. `bind`

Không gọi hàm ngay. Nó trả về **một hàm mới** với `this` đã được cố định.

```js
function greet() {
  console.log(`Xin chào ${this.name}`);
}

const user = { name: "An" };
const boundGreet = greet.bind(user);

boundGreet(); // Xin chào An
```

---

## 17. So sánh `call`, `apply`, `bind`

```text
call  -> gọi ngay, truyền đối số riêng lẻ
apply -> gọi ngay, truyền đối số dạng array
bind  -> không gọi ngay, trả về hàm mới
```

---

## 18. Lexical Scope là gì?

Lexical scope nghĩa là phạm vi truy cập biến được quyết định bởi **vị trí viết code**.

Ví dụ:

```js
const globalVar = "Tôi là biến global";

function outer() {
  const outerVar = "Tôi là biến outer";

  function inner() {
    console.log(globalVar);
    console.log(outerVar);
  }

  inner();
}

outer();
```

Hàm `inner` có thể truy cập:
- biến của chính nó
- biến của `outer`
- biến global

Vì nó được viết bên trong `outer`.

---

## 19. Closure là gì?

Closure là hiện tượng một hàm **ghi nhớ được các biến trong lexical scope nơi nó được tạo ra**, kể cả khi scope bên ngoài đã chạy xong.

Ví dụ kinh điển:

```js
function outer() {
  let count = 0;

  return function inner() {
    count++;
    console.log(count);
  };
}

const fn = outer();
fn(); // 1
fn(); // 2
fn(); // 3
```

### Vì sao `count` vẫn còn?
Vì hàm `inner` đóng lại trên biến `count`.
Nó giữ tham chiếu tới biến đó.
Đó chính là closure.

---

## 20. Hình dung closure

```text
outer() chạy
├─ tạo biến count = 0
└─ trả về inner()

inner() vẫn giữ được count
vì closure giữ lại lexical environment
```

---

## 21. Closure giữ lại cái gì?

Closure không phải “copy nguyên giá trị” một cách đơn giản.
Nó giữ **tham chiếu tới môi trường biến** nơi hàm được tạo ra.

Vì vậy biến vẫn có thể tiếp tục thay đổi qua nhiều lần gọi.

```js
function createCounter() {
  let count = 0;

  return function() {
    count += 1;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

---

## 22. Ứng dụng thực tế của closure

Closure thường được dùng để:

- tạo biến private
- giữ state giữa các lần gọi hàm
- factory function
- callback và event handler
- memoization
- debounce / throttle

### Ví dụ private variable

```js
function createBankAccount() {
  let balance = 0;

  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    getBalance() {
      return balance;
    }
  };
}

const account = createBankAccount();
console.log(account.deposit(100)); // 100
console.log(account.getBalance()); // 100
```

Ở đây `balance` không truy cập trực tiếp được từ bên ngoài, nhưng các method bên trong vẫn dùng được.

---

## 23. Closure và lỗi hay gặp trong vòng lặp

Ngày xưa với `var`, closure trong loop rất dễ gây bug.

```js
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 0);
}
```

Kết quả:

```js
3
3
3
```

Vì tất cả callback cùng dùng chung một biến `i`.

Nếu dùng `let`:

```js
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 0);
}
```

Kết quả:

```js
0
1
2
```

Vì mỗi vòng lặp có binding riêng.

---

## 24. Arrow function và closure

Arrow function cũng có closure như function thường, vì closure liên quan đến lexical scope.

```js
function outer() {
  let x = 10;
  return () => console.log(x);
}

const fn = outer();
fn(); // 10
```

---

## 25. Các lỗi hay nhầm trong Phase 3

## 25.1. Nghĩ arrow function chỉ là viết tắt của function thường
Sai.
Arrow function còn khác ở:
- `this`
- `arguments`
- cách hoạt động với `new`

## 25.2. Nghĩ `this` là object đang chứa hàm
Không phải lúc nào cũng đúng.
`this` phụ thuộc **cách gọi hàm**.

## 25.3. Nghĩ closure là copy biến ra ngoài
Không chuẩn.
Closure giữ **tham chiếu tới lexical environment**, không đơn thuần là copy rời rạc.

## 25.4. Nghĩ `arguments` là array thật
Sai.
Nó là object dạng array-like.

## 25.5. Nghĩ method trong object dùng arrow function cũng như nhau
Sai.
Arrow function thường làm `this` không trỏ tới object như bạn mong đợi.

---

## 26. Checklist Phase 3

Bạn nên tự trả lời được các câu sau:

- Function declaration khác function expression thế nào?
- Arrow function khác function thường ở đâu?
- `arguments` khác rest parameter ra sao?
- Callback là gì?
- Higher-order function là gì?
- `this` trong function thường, object method, arrow function khác nhau thế nào?
- `call`, `apply`, `bind` dùng để làm gì?
- Lexical scope là gì?
- Closure là gì?
- Closure giữ lại cái gì?
- Vì sao closure hữu ích trong thực tế?

---

## 27. Tóm tắt Phase 3

```text
Function declaration
- hoisting đầy đủ

Function expression
- gán hàm vào biến

Arrow function
- cú pháp ngắn gọn
- không có this riêng
- không có arguments riêng

arguments
- object đặc biệt trong function thường

...rest
- gom tham số còn lại thành array thật

this
- phụ thuộc cách gọi hàm

call / apply / bind
- điều khiển this

lexical scope
- phạm vi biến quyết định bởi vị trí viết code

closure
- hàm nhớ được biến ở scope bên ngoài nơi nó được tạo ra
```

---

## 28. Kết luận

Phase 3 là một trong những phase quan trọng nhất của JavaScript.

Nếu chưa chắc phần này, bạn sẽ rất dễ:

- nhầm `this`
- khó hiểu callback
- khó đọc code dùng closure
- khó hiểu vì sao hàm bên trong vẫn nhớ biến bên ngoài
- gặp bug khi tách method ra khỏi object

Ngược lại, nếu hiểu chắc Phase 3, bạn sẽ thấy rất nhiều phần khó của JavaScript trở nên dễ hơn.
