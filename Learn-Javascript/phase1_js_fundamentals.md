# Phase 1 JavaScript - Kiểu dữ liệu và ép kiểu

## Mục tiêu của Phase 1

Phase 1 tập trung vào nền tảng quan trọng nhất của JavaScript:
- Kiểu dữ liệu nguyên thủy và kiểu tham chiếu
- Truthy và falsy
- `null` và `undefined`
- `typeof`
- Ép kiểu trong JavaScript
- `==` và `===`
- `NaN`
- `isNaN()` và `Number.isNaN()`
- `Number()`, `parseInt()`, `parseFloat()`
- `||` và `??`
- `value == null` và `value === null`
- Vì sao `typeof null === "object"`

---

# 1. Primitive và Reference

## 1.1. Primitive types là gì?

Primitive là các kiểu dữ liệu nguyên thủy trong JavaScript. Đây là các giá trị được lưu theo kiểu giá trị độc lập.

JavaScript hiện đại có **7 primitive types**:
- `string`
- `number`
- `boolean`
- `undefined`
- `null`
- `symbol`
- `bigint`

Ví dụ:

```js
let a = 10
let b = a
b = 20

console.log(a) // 10
console.log(b) // 20
```

Ở đây `b = a` là **copy giá trị**.

## 1.2. Reference type là gì?

Reference type là kiểu dữ liệu được lưu theo kiểu tham chiếu.

Trong JavaScript, `object` thuộc nhóm reference type.
Ví dụ:
- object thường
- array
- function

```js
let user1 = { name: "An" }
let user2 = user1

user2.name = "Bình"

console.log(user1.name) // "Bình"
console.log(user2.name) // "Bình"
```

Ở đây `user2 = user1` không tạo object mới, mà chỉ copy tham chiếu.

## 1.3. Khác nhau giữa Primitive và Reference

### Primitive
- lưu giá trị trực tiếp
- copy là copy giá trị
- thay đổi biến này không làm đổi biến kia

### Reference
- biến giữ tham chiếu tới object
- copy là copy tham chiếu
- nhiều biến có thể cùng trỏ đến một object

Tóm tắt:

```text
Primitive -> copy giá trị
Reference -> copy tham chiếu
```

---

# 2. Truthy và Falsy

## 2.1. Truthy là gì?

Truthy là những giá trị khi đưa vào ngữ cảnh boolean, JavaScript sẽ ép thành `true`.

Ví dụ:

```js
if ("hello") {
  console.log("chạy")
}
```

Chuỗi `"hello"` là truthy nên khối `if` chạy.

## 2.2. Falsy là gì?

Falsy là những giá trị khi đưa vào ngữ cảnh boolean, JavaScript sẽ ép thành `false`.

Các giá trị falsy quan trọng:
- `false`
- `0`
- `-0`
- `0n`
- `""`
- `null`
- `undefined`
- `NaN`

Ngoài các giá trị falsy trên, phần lớn còn lại là truthy.

## 2.3. Vì sao `{}` và `[]` vẫn là truthy?

Vì trong JavaScript:
- `{}` là object
- `[]` cũng là object
- mà object là truthy

```js
console.log(Boolean({})) // true
console.log(Boolean([])) // true
```

## 2.4. Những ví dụ hay nhầm

```js
console.log(Boolean("0"))    // true
console.log(Boolean(0))      // false
console.log(Boolean([]))     // true
console.log(Boolean({}))     // true
console.log(Boolean(""))     // false
console.log(Boolean(" "))    // true
```

Giải thích:
- `"0"` là chuỗi không rỗng nên truthy
- `0` là số 0 nên falsy
- `[]` là object nên truthy
- `""` là chuỗi rỗng nên falsy

---

# 3. `null` và `undefined`

## 3.1. `undefined` là gì?

`undefined` thường mang ý nghĩa:
> chưa được gán giá trị

Hay gặp trong các trường hợp:
- khai báo biến nhưng chưa gán
- truy cập thuộc tính không tồn tại
- function không `return`
- tham số không được truyền vào

```js
let a
console.log(a) // undefined

const user = {}
console.log(user.name) // undefined

function test() {}
console.log(test()) // undefined
```

## 3.2. `null` là gì?

`null` thường mang ý nghĩa:
> cố ý không có giá trị

Tức là do lập trình viên chủ động gán.

```js
let selectedUser = null
```

Biến có tồn tại, nhưng hiện tại được đặt là chưa có dữ liệu.

## 3.3. Giống và khác nhau

### Giống nhau
- đều là primitive
- đều thường dùng để biểu thị thiếu giá trị
- đều là falsy

### Khác nhau
- `undefined` thường là trạng thái mặc định do JS tạo ra
- `null` thường là trạng thái có chủ đích do dev gán

Tóm tắt:

```text
undefined -> chưa được gán giá trị
null -> cố ý gán là không có gì
```

## 3.4. Ví dụ so sánh

```js
let a = null
let b
let c = ""

console.log(a) // null
console.log(b) // undefined
console.log(c) // ""
```

Lưu ý:
- `null` và `undefined` biểu thị thiếu giá trị
- `""` vẫn là một giá trị string, chỉ là chuỗi rỗng

---

# 4. `typeof`

## 4.1. `typeof` dùng để làm gì?

`typeof` dùng để kiểm tra kiểu dữ liệu và trả về kết quả dưới dạng chuỗi.

Ví dụ:

```js
typeof "hello"      // "string"
typeof 123          // "number"
typeof true         // "boolean"
typeof undefined    // "undefined"
typeof function(){} // "function"
```

## 4.2. Những kết quả quan trọng cần nhớ

```js
typeof "abc"        // "string"
typeof 10           // "number"
typeof true         // "boolean"
typeof undefined    // "undefined"
typeof function(){} // "function"
typeof null         // "object"
typeof []           // "object"
typeof {}           // "object"
```

## 4.3. Điểm dễ gây nhầm

### `typeof null`

```js
typeof null // "object"
```

Nhưng `null` **không phải object**. Nó là primitive.
Đây là bug lịch sử của JavaScript.

### `typeof []`

```js
typeof [] // "object"
```

Array là một dạng đặc biệt của object, nên `typeof` không trả về `"array"`.

Muốn kiểm tra array, dùng:

```js
Array.isArray([]) // true
```

---

# 5. `==` và `===`

## 5.1. `==` là gì?

`==` là so sánh lỏng.
Nó **có ép kiểu** trước khi so sánh.

```js
0 == false          // true
"" == false         // true
"5" == 5            // true
null == undefined   // true
```

## 5.2. `===` là gì?

`===` là so sánh nghiêm ngặt.
Nó **không ép kiểu**.

Chỉ khi cùng kiểu và cùng giá trị thì mới là `true`.

```js
0 === false         // false
"" === false        // false
"5" === 5           // false
null === undefined  // false
```

## 5.3. Vì sao nên ưu tiên `===`

Vì `==` dễ gây các kết quả khó đoán do ép kiểu ngầm.

Ví dụ:

```js
"" == 0        // true
0 == false     // true
[] == false    // true
```

Trong code thực tế, thường nên ưu tiên `===` để dễ đọc và ít bug hơn.

## 5.4. Ví dụ đặc biệt: `[] == false`

```js
[] == false   // true
[] === false  // false
```

Với `==`, JavaScript ép kiểu:

```text
[] -> "" -> 0
false -> 0
=> 0 == 0
=> true
```

Còn `===` không ép kiểu:
- `[]` là object
- `false` là boolean
- khác kiểu nên kết quả là `false`

---

# 6. Type Coercion

## 6.1. Type coercion là gì?

Type coercion là hiện tượng JavaScript chuyển một giá trị từ kiểu dữ liệu này sang kiểu khác để thực hiện phép toán hoặc so sánh.

## 6.2. Hai loại ép kiểu

### Ép kiểu ngầm (implicit coercion)
JS tự ép kiểu.

```js
"5" + 1    // "51"
"5" - 1    // 4
0 == false  // true
```

### Ép kiểu tường minh (explicit coercion)
Lập trình viên chủ động ép kiểu.

```js
Number("123")
String(456)
Boolean(0)
```

## 6.3. Giải thích các ví dụ hay gặp

### `"5" + 1`

```js
"5" + 1 // "51"
```

Toán tử `+` nếu có string tham gia thì thường ưu tiên nối chuỗi.

```text
"5" + 1
= "5" + "1"
= "51"
```

### `"5" - 1`

```js
"5" - 1 // 4
```

Toán tử `-` chỉ dùng cho số, nên JS ép `"5"` thành number.

```text
"5" - 1
= 5 - 1
= 4
```

---

# 7. `NaN`

## 7.1. `NaN` là gì?

`NaN` là viết tắt của `Not-a-Number`.
Nhưng trong JavaScript, `NaN` vẫn là một giá trị đặc biệt thuộc kiểu `number`.

```js
console.log(typeof NaN) // "number"
```

## 7.2. Khi nào xuất hiện `NaN`?

```js
Number("abc")   // NaN
0 / 0           // NaN
```

## 7.3. Vì sao `NaN === NaN` là `false`?

Theo quy tắc số học IEEE 754:
> `NaN` không bằng bất kỳ giá trị nào, kể cả chính nó

```js
NaN === NaN // false
NaN == NaN  // false
```

## 7.4. Kiểm tra `NaN` thế nào?

Không nên dùng:

```js
value === NaN
```

Vì luôn là `false`.

Nên dùng:

```js
Number.isNaN(value)
```

---

# 8. `isNaN()` và `Number.isNaN()`

## 8.1. `isNaN()`

`isNaN()` sẽ **ép kiểu trước rồi mới kiểm tra**.

```js
isNaN("abc") // true
```

Vì:
- `"abc"` bị ép sang number
- kết quả là `NaN`
- nên trả về `true`

## 8.2. `Number.isNaN()`

`Number.isNaN()` **không ép kiểu**.
Nó chỉ trả về `true` khi giá trị thật sự là `NaN`.

```js
Number.isNaN("abc") // false
Number.isNaN(NaN)    // true
```

## 8.3. So sánh nhanh

```js
isNaN("abc")             // true
Number.isNaN("abc")      // false

isNaN(NaN)                // true
Number.isNaN(NaN)         // true

isNaN(undefined)          // true
Number.isNaN(undefined)   // false
```

Tóm tắt:

```text
isNaN() -> ép kiểu rồi kiểm tra
Number.isNaN() -> kiểm tra nghiêm ngặt
```

Trong thực tế, thường nên ưu tiên `Number.isNaN()`.

---

# 9. `Number()`, `parseInt()`, `parseFloat()`

## 9.1. `Number()`

Dùng để ép **toàn bộ giá trị** sang number.
Nếu toàn bộ chuỗi không hợp lệ thì dễ ra `NaN`.

```js
Number("123")     // 123
Number("12.5")    // 12.5
Number("123abc")  // NaN
Number("")        // 0
```

## 9.2. `parseInt()`

Dùng để đọc **số nguyên từ đầu chuỗi**.
Đọc được đến đâu thì lấy đến đó, gặp ký tự không hợp lệ thì dừng.

```js
parseInt("123abc") // 123
parseInt("12.5")   // 12
parseInt("abc123") // NaN
```

## 9.3. `parseFloat()`

Gần giống `parseInt()`, nhưng cho phép lấy cả phần thập phân.

```js
parseFloat("123abc") // 123
parseFloat("12.5")   // 12.5
parseFloat("abc12.5") // NaN
```

## 9.4. So sánh nhanh

### Với `"123abc"`

```js
Number("123abc")     // NaN
parseInt("123abc")   // 123
parseFloat("123abc") // 123
```

### Với `"12.5"`

```js
Number("12.5")       // 12.5
parseInt("12.5")     // 12
parseFloat("12.5")   // 12.5
```

## 9.5. Khi nào dễ bị `NaN`?

### `Number()`
- nếu toàn bộ giá trị không ép được thành số hợp lệ

### `parseInt()` / `parseFloat()`
- nếu ngay ký tự đầu tiên đã không đọc được số

---

# 10. `||` và `??`

## 10.1. `||`

`||` trả về vế phải nếu vế trái là **falsy**.

```js
0 || 100          // 100
"" || "default"   // "default"
false || "A"     // "A"
```

## 10.2. `??`

`??` trả về vế phải **chỉ khi** vế trái là:
- `null`
- `undefined`

```js
0 ?? 100          // 0
"" ?? "default"   // ""
false ?? "A"     // false
```

## 10.3. Khác nhau cốt lõi

```text
|| -> fallback nếu falsy
?? -> fallback nếu null hoặc undefined
```

## 10.4. Ví dụ dễ nhầm

```js
0 || 100     // 100
0 ?? 100     // 0

"" || "default"   // "default"
"" ?? "default"   // ""
```

---

# 11. `value == null` và `value === null`

## 11.1. `value == null`

`value == null` sẽ là `true` nếu:
- `value` là `null`
- `value` là `undefined`

```js
let a = null
let b

console.log(a == null) // true
console.log(b == null) // true
```

## 11.2. `value === null`

`value === null` chỉ là `true` khi giá trị đúng chính xác là `null`.

```js
let a = null
let b

console.log(a === null) // true
console.log(b === null) // false
```

## 11.3. Vì sao `undefined == null` là `true`?

Đây là quy tắc đặc biệt của toán tử `==` trong JavaScript.

```js
undefined == null // true
undefined === null // false
```

Tóm tắt:

```text
value == null  -> kiểm tra null hoặc undefined
value === null -> chỉ kiểm tra null
```

---

# 12. Vì sao `typeof null === "object"`?

Đây là bug lịch sử của JavaScript.

Thời kỳ đầu, JavaScript dùng cách biểu diễn kiểu dữ liệu ở mức thấp bằng type tag. Giá trị `null` bị nhận diện nhầm như object.

Vì vậy:

```js
typeof null // "object"
```

Nhưng cần nhớ:
- `null` không phải object
- `null` là primitive

Lỗi này đã tồn tại quá lâu và được giữ lại để tương thích ngược.

---

# 13. Tổng kết Phase 1

Sau Phase 1, bạn cần nắm chắc các ý sau:

## 13.1. Kiểu dữ liệu
- JavaScript có 7 primitive types
- object là reference type
- primitive copy giá trị
- reference copy tham chiếu

## 13.2. Truthy/Falsy
- falsy chỉ có một nhóm nhỏ giá trị
- object và array rỗng vẫn là truthy
- `"0"` là truthy còn `0` là falsy

## 13.3. `null` và `undefined`
- `undefined` thường là chưa được gán giá trị
- `null` thường là cố ý không có giá trị
- cả hai đều falsy

## 13.4. So sánh
- `==` có ép kiểu
- `===` không ép kiểu
- thường nên ưu tiên `===`

## 13.5. Ép kiểu
- `+` gặp string thường nối chuỗi
- `-`, `*`, `/` thường ép sang number
- JS có cả ép kiểu ngầm và ép kiểu tường minh

## 13.6. `NaN`
- `NaN` là giá trị number đặc biệt
- `typeof NaN === "number"`
- `NaN !== NaN`
- nên dùng `Number.isNaN()` để kiểm tra

## 13.7. `||` và `??`
- `||` fallback theo falsy
- `??` chỉ fallback theo `null` hoặc `undefined`

---

# 14. Bảng nhớ nhanh

```js
// Primitive
string, number, boolean, undefined, null, symbol, bigint

// typeof
typeof "abc"        // "string"
typeof 10           // "number"
typeof true         // "boolean"
typeof undefined    // "undefined"
typeof function(){} // "function"
typeof null         // "object"
typeof []           // "object"

// Truthy / Falsy
Boolean("0")   // true
Boolean(0)     // false
Boolean([])    // true
Boolean({})    // true
Boolean("")    // false

// So sánh
0 == false         // true
0 === false        // false
null == undefined  // true
null === undefined // false

// NaN
typeof NaN         // "number"
NaN === NaN        // false
Number.isNaN(NaN)  // true

// Number / parseInt / parseFloat
Number("123abc")      // NaN
parseInt("123abc")    // 123
parseFloat("123abc")  // 123

// || và ??
0 || 100           // 100
0 ?? 100           // 0
"" || "default"    // "default"
"" ?? "default"    // ""
```

---

# 15. Kết luận

Phase 1 là phần nền rất quan trọng. Nếu nắm chắc phase này, bạn sẽ:
- đọc code JS dễ hơn
- bớt bị nhầm ở các case ép kiểu
- hiểu vì sao nhiều biểu thức trong JS cho kết quả "lạ"
- có nền tốt để học tiếp Phase 2: `var`, `let`, `const`, scope, hoisting, TDZ
