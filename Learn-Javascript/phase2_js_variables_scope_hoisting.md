# Phase 2: Biến, Scope, Hoisting và TDZ trong JavaScript

## 1. Mục tiêu của Phase 2

Ở phase này, bạn cần nắm chắc cách JavaScript xử lý biến và phạm vi hoạt động của biến.

Sau khi học xong phase này, bạn cần hiểu rõ:

- `var`, `let`, `const` khác nhau như thế nào
- scope là gì
- global scope, function scope, block scope khác nhau ra sao
- hoisting là gì
- vì sao `var` truy cập trước khai báo được còn `let` và `const` thì lỗi
- TDZ là gì
- khi nào nên dùng `const`, khi nào dùng `let`, và vì sao nên hạn chế `var`

---

## 2. Tổng quan

Có thể hình dung Phase 2 như sau:

```text
Phase 2
├─ var / let / const
├─ Scope
│  ├─ Global scope
│  ├─ Function scope
│  └─ Block scope
├─ Hoisting
└─ TDZ (Temporal Dead Zone)
```

---

## 3. Biến trong JavaScript là gì?

Biến là nơi dùng để lưu giá trị để sau đó có thể sử dụng lại.

Ví dụ:

```js
let name = "An"
const age = 20
var city = "Hanoi"
```

Ở đây:

- `name` lưu chuỗi `"An"`
- `age` lưu số `20`
- `city` lưu chuỗi `"Hanoi"`

JavaScript cho phép khai báo biến bằng 3 cách chính:

- `var`
- `let`
- `const`

Đây là phần rất quan trọng vì mỗi cách có hành vi khác nhau.

---

## 4. `var`, `let`, `const` là gì?

## 4.1. `var`

`var` là cách khai báo biến cũ trong JavaScript.

Đặc điểm chính:

- có **function scope**
- có hoisting
- được khởi tạo mặc định là `undefined`
- cho phép khai báo lại
- cho phép gán lại giá trị
- dễ gây bug hơn `let` và `const`

Ví dụ:

```js
var a = 10
var a = 20
console.log(a) // 20
```

`var` cho phép khai báo lại cùng tên biến trong cùng scope.

---

## 4.2. `let`

`let` là cách khai báo biến hiện đại hơn.

Đặc điểm chính:

- có **block scope**
- có hoisting nhưng không khởi tạo usable ngay
- nằm trong TDZ trước khi tới dòng khai báo
- không cho khai báo lại trong cùng scope
- cho phép gán lại giá trị

Ví dụ:

```js
let count = 1
count = 2
console.log(count) // 2
```

Nhưng:

```js
let x = 1
let x = 2 // lỗi
```

---

## 4.3. `const`

`const` cũng là cách khai báo hiện đại.

Đặc điểm chính:

- có **block scope**
- có hoisting nhưng nằm trong TDZ trước dòng khai báo
- không cho khai báo lại trong cùng scope
- không cho gán lại biến sau khi đã gán lần đầu
- bắt buộc phải gán giá trị ngay khi khai báo

Ví dụ:

```js
const PI = 3.14
console.log(PI)
```

Sai:

```js
const a
// lỗi vì const phải gán giá trị ngay
```

Sai tiếp:

```js
const age = 20
age = 21 // lỗi
```

---

## 5. So sánh nhanh `var`, `let`, `const`

| Đặc điểm | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function scope | Block scope | Block scope |
| Hoisting | Có | Có | Có |
| Truy cập trước khai báo | Được, ra `undefined` | Lỗi | Lỗi |
| TDZ | Không | Có | Có |
| Gán lại giá trị | Có | Có | Không |
| Khai báo lại | Có | Không | Không |
| Nên dùng hiện nay | Hạn chế | Có | Ưu tiên nhất |

---

## 6. Scope là gì?

**Scope** là phạm vi mà trong đó một biến có thể được truy cập.

Nói đơn giản:

> Biến được sinh ra ở đâu thì nó chỉ dùng được trong phạm vi phù hợp với nơi nó được sinh ra.

Có 3 loại scope quan trọng:

- Global scope
- Function scope
- Block scope

---

## 7. Global scope

Biến ở global scope có thể được truy cập ở nhiều nơi trong chương trình.

Ví dụ:

```js
let appName = "My App"

function showName() {
  console.log(appName)
}

showName() // My App
```

Ở đây, `appName` nằm ngoài function nên thuộc global scope.

### Lưu ý
Dùng quá nhiều biến global sẽ dễ:

- đụng tên biến
- khó kiểm soát
- khó debug

---

## 8. Function scope

Biến khai báo bằng `var` bên trong function sẽ chỉ dùng được trong function đó.

Ví dụ:

```js
function test() {
  var x = 10
  console.log(x) // 10
}

test()
console.log(x) // lỗi
```

`x` chỉ tồn tại trong function `test`.

Đây là lý do người ta nói `var` có **function scope**.

---

## 9. Block scope

Block là vùng nằm trong cặp ngoặc nhọn `{}` như:

- `if`
- `for`
- `while`
- block tự tạo

Biến khai báo bằng `let` hoặc `const` sẽ chỉ dùng được trong block đó.

Ví dụ:

```js
if (true) {
  let a = 10
  const b = 20
  console.log(a) // 10
  console.log(b) // 20
}

console.log(a) // lỗi
console.log(b) // lỗi
```

---

## 10. Vì sao `var` dễ gây nhầm trong block?

`var` không bị giới hạn bởi block.

Ví dụ:

```js
if (true) {
  var x = 100
}

console.log(x) // 100
```

Nhiều người mới học sẽ nghĩ `x` phải chỉ tồn tại trong `if`, nhưng không phải.

Vì `var` không có block scope.

---

## 11. Ví dụ so sánh `var` với `let`

```js
if (true) {
  var a = 1
  let b = 2
}

console.log(a) // 1
console.log(b) // lỗi
```

Kết luận:

- `var` thoát ra ngoài block được
- `let` thì không

---

## 12. Hoisting là gì?

**Hoisting** là cơ chế JavaScript đưa phần khai báo lên đầu scope trước khi code được thực thi.

Cần hiểu đúng:

- không phải code bị di chuyển thật sự trong file
- mà JavaScript xử lý như thể phần khai báo đã được biết trước

---

## 13. Hoisting với `var`

Ví dụ:

```js
console.log(a)
var a = 10
```

Kết quả:

```js
undefined
```

Có thể hình dung JavaScript xử lý gần như:

```js
var a
console.log(a)
a = 10
```

Nghĩa là:

- `var a` được hoisting
- và được khởi tạo luôn bằng `undefined`

---

## 14. Hoisting với `let`

Ví dụ:

```js
console.log(b)
let b = 20
```

Kết quả:

```js
ReferenceError
```

`let` cũng có hoisting, nhưng không usable trước dòng khai báo.

Nó nằm trong **TDZ**.

---

## 15. Hoisting với `const`

Ví dụ:

```js
console.log(c)
const c = 30
```

Kết quả:

```js
ReferenceError
```

`const` cũng giống `let` ở điểm này:

- có hoisting
- nhưng nằm trong TDZ trước khi tới dòng khai báo

---

## 16. TDZ là gì?

**TDZ** là viết tắt của **Temporal Dead Zone**.

Hiểu đơn giản:

> Đây là khoảng thời gian từ lúc scope bắt đầu cho tới trước dòng khai báo `let` hoặc `const`, trong đó biến đã tồn tại về mặt scope nhưng chưa được phép truy cập.

Ví dụ:

```js
{
  console.log(a) // lỗi
  let a = 10
}
```

Ở đây:

- block đã bắt đầu
- JavaScript đã biết có biến `a`
- nhưng trước dòng `let a = 10`, biến `a` đang ở TDZ
- nên truy cập vào là lỗi

---

## 17. Hình dung TDZ

```text
Bắt đầu scope
   ↓
Biến let/const đã được biết tới
   ↓
Nhưng chưa được khởi tạo usable
   ↓
Nếu truy cập tại đây => lỗi
   ↓
Tới dòng khai báo
   ↓
Biến bắt đầu dùng được
```

---

## 18. Ví dụ rất hay gặp về TDZ

```js
let a = 5

function test() {
  console.log(a)
  let a = 10
}

test()
```

Nhiều người nghĩ sẽ in `5`, nhưng thực tế là lỗi.

Vì sao?

Vì bên trong `test`, JavaScript thấy có:

```js
let a = 10
```

nên tạo ra một biến `a` mới trong scope của function `test`.

Biến `a` mới này che mất biến `a` bên ngoài.

Nhưng trước dòng khai báo, nó đang nằm trong TDZ.

Nên:

```js
console.log(a)
```

sẽ lỗi.

---

## 19. Khai báo lại và gán lại khác nhau thế nào?

Đây là chỗ rất nhiều người học dễ nhầm.

### 19.1. Khai báo lại

Tức là dùng lại từ khóa khai báo cho cùng tên biến trong cùng scope.

Ví dụ:

```js
let x = 1
let x = 2 // lỗi
```

Ở đây là **khai báo lại**.

---

### 19.2. Gán lại

Tức là biến đã tồn tại rồi, chỉ thay đổi giá trị.

Ví dụ:

```js
let x = 1
x = 2
console.log(x) // 2
```

Ở đây là **gán lại**, và `let` cho phép.

---

## 20. `const` có thật sự bất biến không?

Cần hiểu rất đúng chỗ này.

`const` không cho gán lại **biến**, nhưng nếu giá trị là object hoặc array thì vẫn có thể sửa **nội dung bên trong**.

Ví dụ:

```js
const user = {
  name: "An"
}

user.name = "Binh"
console.log(user.name) // Binh
```

Điều này vẫn hợp lệ.

Sai là:

```js
const user = { name: "An" }
user = { name: "Binh" } // lỗi
```

### Kết luận

- `const` không cho đổi tham chiếu của biến
- nhưng không đồng nghĩa toàn bộ dữ liệu bên trong là immutable

---

## 21. `var` trong vòng lặp và lỗi hay gặp

Ví dụ:

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i)
  }, 0)
}
```

Kết quả:

```js
3
3
3
```

### Vì sao?

Vì:

- `var` chỉ có một biến `i` dùng chung cho cả vòng lặp
- khi callback chạy thì vòng lặp đã kết thúc
- lúc đó `i` đã bằng `3`

---

## 22. `let` trong vòng lặp

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i)
  }, 0)
}
```

Kết quả:

```js
0
1
2
```

### Vì sao?

Vì với `let`, mỗi vòng lặp sẽ có một binding `i` riêng.

Nên mỗi callback giữ được đúng giá trị của vòng lặp tương ứng.

---

## 23. Khi nào nên dùng `var`, `let`, `const`?

## 23.1. Ưu tiên `const` mặc định

Nếu bạn không cần gán lại biến, hãy dùng `const`.

Ví dụ:

```js
const API_URL = "https://example.com"
const user = { name: "An" }
```

Điều này giúp code:

- rõ ý hơn
- ít bị thay đổi nhầm
- dễ đọc hơn

---

## 23.2. Dùng `let` khi cần gán lại

Ví dụ:

```js
let count = 0
count++
```

Các trường hợp như:

- biến đếm
- giá trị thay đổi trong quá trình xử lý
- giá trị input thay đổi

thì dùng `let` hợp lý.

---

## 23.3. Hạn chế `var`

Hiện nay hầu hết code hiện đại đều tránh dùng `var`, vì:

- không có block scope
- cho phép khai báo lại
- dễ gây bug trong vòng lặp và block
- khó kiểm soát hơn

---

## 24. Các lỗi điển hình trong Phase 2

## 24.1. Nghĩ rằng `var` là global scope

Sai hoàn toàn nếu nói như vậy một cách tổng quát.

Chuẩn hơn:

- `var` có **function scope**
- nếu khai báo ngoài function thì nó có thể xuất hiện ở phạm vi global

---

## 24.2. Nghĩ `let` và `const` không có hoisting

Sai.

Cả hai **đều có hoisting**.

Khác ở chỗ:

- chúng nằm trong TDZ trước khi tới dòng khai báo
- nên truy cập sớm sẽ lỗi

---

## 24.3. Nghĩ `const` là bất biến hoàn toàn

Sai.

`const` chỉ khóa việc gán lại biến, không khóa luôn nội dung của object/array.

---

## 24.4. Nhầm giữa khai báo lại và gán lại

- `let x = 1; let x = 2` → lỗi vì khai báo lại
- `let x = 1; x = 2` → hợp lệ vì gán lại

---

## 25. Bảng ví dụ tổng hợp

### Ví dụ 1: `var`

```js
console.log(a)
var a = 10
```

Kết quả:

```js
undefined
```

---

### Ví dụ 2: `let`

```js
console.log(b)
let b = 20
```

Kết quả:

```js
ReferenceError
```

---

### Ví dụ 3: `const`

```js
const x = 1
x = 2
```

Kết quả:

```js
TypeError
```

---

### Ví dụ 4: block scope

```js
if (true) {
  let a = 1
  var b = 2
}

console.log(a) // lỗi
console.log(b) // 2
```

---

### Ví dụ 5: object với `const`

```js
const user = { name: "An" }
user.name = "Binh"
console.log(user.name) // Binh
```

---

## 26. Tóm tắt toàn bộ Phase 2

```text
1. var có function scope
2. let và const có block scope
3. Cả var, let, const đều có hoisting
4. var được khởi tạo là undefined
5. let và const nằm trong TDZ trước dòng khai báo
6. var cho khai báo lại và gán lại
7. let không cho khai báo lại, nhưng cho gán lại
8. const không cho khai báo lại và không cho gán lại biến
9. const với object/array vẫn có thể sửa nội dung bên trong
10. Trong code hiện đại: ưu tiên const, cần thay đổi thì dùng let, hạn chế var
```

---

## 27. Checklist tự kiểm tra sau Phase 2

Bạn nên tự trả lời được các câu này:

- `var` khác `let` ở đâu?
- Vì sao `var` không an toàn bằng `let`?
- Hoisting là gì?
- Vì sao `console.log(a); let a = 1` lại lỗi?
- TDZ là gì?
- Vì sao `const user = {}` vẫn sửa `user.name` được?
- Vì sao `for (var i...)` với `setTimeout` thường in ra cùng một giá trị?
- Vì sao `for (let i...)` lại khác?

---

## 28. Mini review Phase 2

### Câu 1
JavaScript có mấy cách khai báo biến chính?

### Câu 2
`var`, `let`, `const` khác nhau ở scope như thế nào?

### Câu 3
Hoisting là gì?

### Câu 4
TDZ là gì?

### Câu 5
Vì sao `const` không cho gán lại nhưng vẫn sửa được nội dung object?

### Câu 6
Vì sao `var` trong vòng lặp với `setTimeout` thường gây bug?

---

## 29. Kết luận

Phase 2 là phần nền cực kỳ quan trọng vì nó ảnh hưởng trực tiếp đến cách bạn đọc và viết code JavaScript.

Nếu chưa chắc phần này, bạn sẽ rất dễ:

- nhầm scope
- nhầm hoisting
- không hiểu vì sao code báo lỗi ReferenceError
- không hiểu vì sao `setTimeout` trong vòng lặp ra kết quả lạ

Ngược lại, nếu nắm chắc Phase 2, bạn sẽ đọc code thực tế dễ hơn rất nhiều và chuẩn bị tốt để sang các phase tiếp theo như:

- Function
- `this`
- Closure
- Destructuring
- Optional Chaining
- Async JavaScript

