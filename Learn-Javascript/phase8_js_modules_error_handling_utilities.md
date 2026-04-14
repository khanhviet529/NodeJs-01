# Phase 8: Module, Error Handling, Optional Chaining, Nullish Coalescing, JSON, Storage, Debounce, Throttle

## 1. Mục tiêu của phase này

Phase 8 giúp bạn nắm những phần rất hay gặp khi đọc code thực tế:

- tách file bằng module
- import / export dữ liệu và hàm
- xử lý lỗi bằng `try...catch`
- ném lỗi bằng `throw`
- dùng `?.` và `??` đúng cách
- chuyển đổi dữ liệu bằng `JSON.stringify()` và `JSON.parse()`
- lưu dữ liệu tạm bằng `localStorage` và `sessionStorage`
- hiểu `debounce` và `throttle`
- phân biệt mutation và immutability

Đây là phase cực kỳ thực dụng vì gần như dự án JavaScript nào cũng đụng tới những phần này.

---

## 2. Module là gì?

Module là cách chia code thành nhiều file nhỏ, mỗi file chịu trách nhiệm cho một phần logic.

Ví dụ:

- file xử lý API
- file chứa utility function
- file chứa constant
- file chứa component

Lợi ích:

- code dễ đọc hơn
- dễ tái sử dụng
- dễ bảo trì
- tránh viết tất cả trong một file quá dài

---

## 3. `export` và `import`

### 3.1. Named export

Dùng khi một file muốn export nhiều giá trị.

```js
// math.js
export const PI = 3.14

export function add(a, b) {
  return a + b
}
```

Import:

```js
import { PI, add } from './math.js'

console.log(PI)
console.log(add(2, 3))
```

### Đặc điểm của named export

- phải import đúng tên
- có thể export nhiều thứ trong một file
- khi import dùng dấu `{}`

---

### 3.2. Default export

Dùng khi file có một giá trị chính cần export.

```js
// user.js
export default function getUser() {
  return { id: 1, name: 'An' }
}
```

Import:

```js
import getUser from './user.js'

console.log(getUser())
```

### Đặc điểm của default export

- khi import không cần đúng tên gốc
- không dùng dấu `{}`
- mỗi file chỉ có một `default export`

Ví dụ:

```js
import fetchUser from './user.js'
```

Tên `fetchUser` là do người import tự đặt.

---

## 4. Named export và default export khác nhau thế nào?

### Named export

```js
export const a = 1
export const b = 2
```

```js
import { a, b } from './file.js'
```

### Default export

```js
export default function test() {}
```

```js
import test from './file.js'
```

### Chốt nhanh

```text
named export   -> import bằng đúng tên, dùng {}
default export -> import không cần đúng tên, không dùng {}
```

---

## 5. Module scope là gì?

Mỗi module có phạm vi riêng.

Biến, hàm, constant khai báo trong file đó sẽ không tự động lộ ra toàn bộ chương trình như kiểu viết script toàn cục.

Ví dụ:

```js
// a.js
const secret = 123
```

`secret` chỉ dùng trong `a.js` nếu chưa export ra.

Đây là điểm rất quan trọng vì nó giúp tránh đụng tên biến giữa nhiều file.

---

## 6. Error handling là gì?

Error handling là cách chương trình xử lý khi xảy ra lỗi.

Nếu không xử lý lỗi tốt:

- app có thể crash
- UI có thể đứng
- user không biết chuyện gì xảy ra
- debugging khó hơn

Trong JavaScript, phần nền tảng nhất là:

- `try`
- `catch`
- `finally`
- `throw`

---

## 7. `try...catch` hoạt động như thế nào?

### Cú pháp

```js
try {
  // code có thể lỗi
} catch (error) {
  // xử lý lỗi
}
```

Ví dụ:

```js
try {
  const user = JSON.parse('{ invalid json }')
  console.log(user)
} catch (error) {
  console.log('Có lỗi xảy ra:', error.message)
}
```

### Cách hiểu

- `try`: thử chạy đoạn code
- nếu có lỗi: nhảy vào `catch`
- `catch` nhận object lỗi

---

## 8. `finally` là gì?

`finally` là đoạn code luôn chạy, dù có lỗi hay không.

```js
try {
  console.log('Bắt đầu')
} catch (error) {
  console.log('Có lỗi')
} finally {
  console.log('Luôn chạy')
}
```

Rất hay dùng khi:

- đóng loading
- giải phóng tài nguyên
- cleanup

---

## 9. `throw` là gì?

`throw` dùng để chủ động ném ra lỗi.

Ví dụ:

```js
function divide(a, b) {
  if (b === 0) {
    throw new Error('Không thể chia cho 0')
  }

  return a / b
}
```

Nếu gọi:

```js
console.log(divide(10, 0))
```

thì sẽ có lỗi.

Nếu muốn xử lý an toàn:

```js
try {
  console.log(divide(10, 0))
} catch (error) {
  console.log(error.message)
}
```

---

## 10. Custom error là gì?

Ngoài `Error`, bạn có thể tạo lỗi với message riêng để code dễ hiểu hơn.

Ví dụ đơn giản:

```js
function registerUser(email) {
  if (!email) {
    throw new Error('Email là bắt buộc')
  }

  return 'Đăng ký thành công'
}
```

Mục tiêu của custom error:

- báo lỗi rõ ràng hơn
- dễ debug hơn
- dễ hiển thị message cho user hơn

---

## 11. Optional chaining `?.` là gì?

Optional chaining giúp truy cập thuộc tính an toàn khi giá trị trước đó có thể là `null` hoặc `undefined`.

Ví dụ không dùng `?.`:

```js
const user = null
console.log(user.address.city)
```

Đoạn này lỗi vì `user` là `null`.

Dùng `?.`:

```js
const user = null
console.log(user?.address?.city)
```

Kết quả:

```js
undefined
```

### Cách hiểu

Nếu bên trái là `null` hoặc `undefined`, JS dừng lại và trả về `undefined`, thay vì ném lỗi.

---

## 12. Khi nào nên dùng `?.`

Dùng khi:

- dữ liệu từ API có thể thiếu field
- object lồng nhiều tầng
- truy cập callback có thể không tồn tại
- code UI đọc dữ liệu chưa chắc đã có

Ví dụ:

```js
const user = {
  profile: {
    name: 'An'
  }
}

console.log(user?.profile?.name) // An
console.log(user?.profile?.age)  // undefined
```

---

## 13. Nullish coalescing `??` là gì?

`??` trả về vế phải khi vế trái là:

- `null`
- `undefined`

Ví dụ:

```js
const name = null ?? 'Guest'
console.log(name) // Guest
```

```js
const count = 0 ?? 100
console.log(count) // 0
```

Điểm quan trọng:

`??` không coi `0`, `false`, `""` là thiếu giá trị.

---

## 14. `||` và `??` khác nhau thế nào?

### `||`
Trả về vế phải nếu vế trái là falsy.

```js
console.log(0 || 100)        // 100
console.log('' || 'default') // default
```

### `??`
Trả về vế phải chỉ khi vế trái là `null` hoặc `undefined`.

```js
console.log(0 ?? 100)        // 0
console.log('' ?? 'default') // ''
```

### Chốt nhanh

```text
||  -> fallback khi falsy
??  -> fallback khi null hoặc undefined
```

---

## 15. Kết hợp `?.` và `??`

Đây là pattern rất hay gặp trong code thực tế.

```js
const city = user?.address?.city ?? 'Chưa có thành phố'
```

Cách hiểu:

- thử đọc `user.address.city`
- nếu giữa đường có `null`/`undefined` thì trả `undefined`
- nếu kết quả cuối là `null`/`undefined` thì lấy chuỗi mặc định

Pattern này cực kỳ phổ biến khi xử lý dữ liệu từ API.

---

## 16. JSON là gì?

JSON là định dạng dữ liệu dạng text, thường dùng để trao đổi dữ liệu giữa:

- frontend và backend
- client và server
- file cấu hình

Ví dụ JSON:

```json
{
  "name": "An",
  "age": 20
}
```

Trong JavaScript, hay dùng:

- `JSON.stringify()`
- `JSON.parse()`

---

## 17. `JSON.stringify()` là gì?

Dùng để chuyển JavaScript value thành chuỗi JSON.

Ví dụ:

```js
const user = { name: 'An', age: 20 }
const text = JSON.stringify(user)

console.log(text)
// '{"name":"An","age":20}'
```

Hay dùng khi:

- gửi dữ liệu lên server
- lưu object vào storage
- debug dữ liệu

---

## 18. `JSON.parse()` là gì?

Dùng để chuyển chuỗi JSON thành JavaScript value.

Ví dụ:

```js
const text = '{"name":"An","age":20}'
const user = JSON.parse(text)

console.log(user.name) // An
```

---

## 19. Lỗi hay gặp với JSON

### Chuỗi không hợp lệ

```js
JSON.parse('{ name: An }')
```

Đoạn này lỗi vì JSON yêu cầu key và string phải có dấu `"`.

### JSON không lưu được function

```js
const obj = {
  name: 'An',
  sayHi() {
    console.log('Hi')
  }
}

console.log(JSON.stringify(obj))
```

Function sẽ không được giữ lại đúng nghĩa trong JSON.

---

## 20. `localStorage` là gì?

`localStorage` là nơi lưu dữ liệu key-value trong trình duyệt.

Đặc điểm:

- dữ liệu vẫn còn sau khi tắt tab hoặc tắt trình duyệt
- chỉ lưu string
- hay dùng để lưu setting, token, theme, dữ liệu tạm

Ví dụ:

```js
localStorage.setItem('theme', 'dark')
console.log(localStorage.getItem('theme')) // dark
```

---

## 21. `sessionStorage` là gì?

`sessionStorage` cũng lưu key-value trong trình duyệt, nhưng phạm vi ngắn hơn.

Đặc điểm:

- dữ liệu tồn tại trong phiên tab hiện tại
- đóng tab là mất
- cũng chỉ lưu string

Ví dụ:

```js
sessionStorage.setItem('page', 'home')
console.log(sessionStorage.getItem('page')) // home
```

---

## 22. `localStorage` và `sessionStorage` khác nhau thế nào?

### `localStorage`
- lâu dài hơn
- đóng trình duyệt mở lại vẫn còn

### `sessionStorage`
- ngắn hạn hơn
- đóng tab là mất

### Chốt nhanh

```text
localStorage   -> lưu lâu hơn
sessionStorage -> lưu theo tab / session
```

---

## 23. Vì sao storage chỉ lưu string?

Nếu lưu object trực tiếp:

```js
const user = { name: 'An', age: 20 }
localStorage.setItem('user', user)
```

thường sẽ không ra như mong muốn.

Nên phải dùng:

```js
localStorage.setItem('user', JSON.stringify(user))
```

Khi lấy ra:

```js
const user = JSON.parse(localStorage.getItem('user'))
console.log(user.name)
```

---

## 24. Debounce là gì?

Debounce là kỹ thuật trì hoãn việc gọi hàm cho đến khi người dùng ngừng thao tác trong một khoảng thời gian.

Hay dùng trong:

- ô tìm kiếm
- resize window
- autocomplete
- input validation

Ví dụ tình huống:

Người dùng gõ:

```text
h -> he -> hel -> hell -> hello
```

Nếu mỗi ký tự đều gọi API thì rất tốn.

Debounce sẽ đợi người dùng ngừng gõ, ví dụ 300ms, rồi mới gọi API một lần.

---

## 25. Ví dụ debounce

```js
function debounce(fn, delay) {
  let timer

  return function (...args) {
    clearTimeout(timer)

    timer = setTimeout(() => {
      fn(...args)
    }, delay)
  }
}
```

Dùng:

```js
const handleSearch = debounce((keyword) => {
  console.log('Gọi API với:', keyword)
}, 300)
```

### Cách hiểu

- mỗi lần gọi lại, timer cũ bị xóa
- chỉ khi ngừng gọi đủ lâu mới chạy thật

---

## 26. Throttle là gì?

Throttle là kỹ thuật giới hạn số lần gọi hàm trong một khoảng thời gian.

Ví dụ:

- scroll event
- resize event
- mousemove event

Nếu event bắn liên tục, throttle sẽ cho hàm chạy tối đa mỗi khoảng thời gian nhất định.

Ví dụ: 1 giây chỉ chạy 1 lần.

---

## 27. Ví dụ throttle

```js
function throttle(fn, delay) {
  let isRunning = false

  return function (...args) {
    if (isRunning) return

    isRunning = true
    fn(...args)

    setTimeout(() => {
      isRunning = false
    }, delay)
  }
}
```

### Cách hiểu

- chạy ngay lần đầu
- sau đó khóa lại một thời gian
- hết thời gian mới cho chạy tiếp

---

## 28. Debounce và throttle khác nhau thế nào?

### Debounce
- đợi ngừng thao tác rồi mới chạy
- phù hợp search input

### Throttle
- giới hạn tần suất chạy
- phù hợp scroll, resize

### Chốt nhanh

```text
debounce -> chỉ chạy sau khi ngừng thao tác
throttle -> giới hạn chạy mỗi khoảng thời gian
```

---

## 29. Mutation và immutability là gì?

### Mutation
Là thay đổi trực tiếp dữ liệu gốc.

Ví dụ:

```js
const user = { name: 'An' }
user.name = 'Bình'
```

Object gốc đã bị sửa.

### Immutability
Là không sửa trực tiếp dữ liệu gốc, mà tạo ra bản mới.

Ví dụ:

```js
const user = { name: 'An' }
const newUser = { ...user, name: 'Bình' }
```

- `user` cũ giữ nguyên
- `newUser` là object mới

---

## 30. Vì sao immutability quan trọng?

Nó giúp:

- dễ debug hơn
- tránh bug do sửa nhầm object cũ
- dễ theo dõi state hơn
- cực kỳ quan trọng trong React, Redux và state management nói chung

---

## 31. Những lỗi hay gặp trong phase này

### 31.1. Nhầm giữa named export và default export

Sai:

```js
import add from './math.js'
```

nếu file chỉ export:

```js
export function add() {}
```

---

### 31.2. Dùng `||` khi đáng ra phải dùng `??`

Ví dụ:

```js
const page = 0 || 1 // ra 1
```

nếu `0` là giá trị hợp lệ thì đây là bug.

Nên dùng:

```js
const page = 0 ?? 1 // ra 0
```

---

### 31.3. Quên `JSON.stringify()` khi lưu object vào storage

Sai:

```js
localStorage.setItem('user', { name: 'An' })
```

Nên:

```js
localStorage.setItem('user', JSON.stringify({ name: 'An' }))
```

---

### 31.4. `JSON.parse()` với chuỗi sai format

```js
JSON.parse('{ name: An }')
```

sẽ lỗi.

---

### 31.5. Nhầm giữa debounce và throttle

- search input thường dùng debounce
- scroll thường dùng throttle

---

## 32. Bảng tóm tắt nhanh

| Chủ đề | Ý chính |
|---|---|
| Module | Chia code thành nhiều file nhỏ |
| Named export | Export nhiều giá trị, import dùng `{}` |
| Default export | Export giá trị chính, import không cần `{}` |
| Module scope | Mỗi file có phạm vi riêng |
| `try...catch` | Bắt và xử lý lỗi |
| `throw` | Chủ động ném lỗi |
| `?.` | Truy cập an toàn khi có thể `null`/`undefined` |
| `??` | Fallback khi giá trị là `null` hoặc `undefined` |
| `JSON.stringify()` | Chuyển object thành chuỗi JSON |
| `JSON.parse()` | Chuyển chuỗi JSON thành object |
| `localStorage` | Lưu dữ liệu lâu hơn trong browser |
| `sessionStorage` | Lưu theo phiên tab |
| Debounce | Đợi ngừng thao tác mới chạy |
| Throttle | Giới hạn tần suất chạy |
| Mutation | Sửa trực tiếp dữ liệu gốc |
| Immutability | Tạo dữ liệu mới thay vì sửa dữ liệu cũ |

---

## 33. Tóm tắt cuối phase

Sau phase này, bạn nên nắm được:

- cách tách code bằng module
- cách import / export đúng
- cách xử lý lỗi cơ bản trong JavaScript
- cách dùng `?.` và `??`
- cách làm việc với JSON
- cách lưu dữ liệu bằng browser storage
- khác nhau giữa debounce và throttle
- vì sao immutability quan trọng

Đây là phase giúp bạn đọc code dự án thực tế đỡ ngợp hơn rất nhiều.

---

## 34. Câu hỏi ôn tập

1. Named export và default export khác nhau thế nào?
2. `try...catch` dùng để làm gì?
3. Khi nào nên dùng `??` thay vì `||`?
4. `user?.profile?.name` hoạt động như thế nào?
5. Vì sao lưu object vào `localStorage` phải dùng `JSON.stringify()`?
6. `localStorage` và `sessionStorage` khác nhau thế nào?
7. Debounce và throttle khác nhau thế nào?
8. Mutation và immutability khác nhau thế nào?

---

## 35. Bản ghi nhớ siêu ngắn

```text
Module        -> chia file, import/export
Error         -> try/catch/finally/throw
?.            -> truy cập an toàn
??            -> fallback cho null/undefined
JSON          -> stringify / parse
Storage       -> localStorage / sessionStorage
Debounce      -> đợi ngừng thao tác
Throttle      -> giới hạn tần suất
Immutability  -> tạo bản mới, không sửa trực tiếp dữ liệu cũ
```
