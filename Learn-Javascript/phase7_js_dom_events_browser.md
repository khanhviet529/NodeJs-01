# Phase 7: DOM, Event và Browser APIs

## 1. Mục tiêu của phase này

Phase 7 giúp bạn hiểu:

- DOM là gì và tại sao JavaScript có thể sửa giao diện trang web
- cách chọn phần tử trong trang
- cách đọc và sửa text, HTML, attribute, class, style
- event là gì và cách lắng nghe event
- event bubbling, event capturing, event delegation khác nhau ra sao
- `preventDefault()` và `stopPropagation()` dùng để làm gì
- một số Browser APIs rất hay gặp trong thực tế

Đây là phase cực kỳ quan trọng nếu bạn học JavaScript cho frontend, vì gần như toàn bộ tương tác với trình duyệt đều đi qua DOM và event.

---

## 2. DOM là gì?

## 2.1. Định nghĩa

**DOM** là viết tắt của **Document Object Model**.

Bạn có thể hiểu DOM là:

> cách trình duyệt biểu diễn tài liệu HTML thành một cây object để JavaScript có thể truy cập và thay đổi.

Ví dụ HTML:

```html
<body>
  <h1 id="title">Xin chào</h1>
  <button>Click me</button>
</body>
```

Trình duyệt sẽ không chỉ coi đây là text thô, mà sẽ biến nó thành các node trong một cây DOM.

Hình dung:

```text
Document
└─ html
   └─ body
      ├─ h1
      └─ button
```

---

## 2.2. Vì sao DOM quan trọng?

Vì JavaScript muốn tương tác với giao diện thì phải thông qua DOM.

Ví dụ:

- đổi nội dung chữ
- thêm / xoá phần tử
- đổi màu chữ
- ẩn / hiện popup
- đọc giá trị người dùng nhập
- bắt sự kiện click, input, submit

---

## 2.3. DOM không phải HTML

Đây là điểm rất hay nhầm.

- **HTML** là mã nguồn đánh dấu
- **DOM** là object tree mà trình duyệt tạo ra từ HTML

Nghĩa là JavaScript không sửa trực tiếp file HTML gốc, mà sửa cây DOM đang được trình duyệt render.

---

## 3. Chọn phần tử trong DOM

## 3.1. `getElementById`

Dùng để lấy phần tử theo `id`.

```js
const title = document.getElementById("title")
```

Ví dụ:

```html
<h1 id="title">Hello</h1>
```

```js
const title = document.getElementById("title")
console.log(title)
```

---

## 3.2. `querySelector`

Dùng để lấy **phần tử đầu tiên** khớp với CSS selector.

```js
const btn = document.querySelector("button")
const title = document.querySelector("#title")
const box = document.querySelector(".box")
```

Ưu điểm:

- rất linh hoạt
- dùng được selector giống CSS

---

## 3.3. `querySelectorAll`

Dùng để lấy **tất cả phần tử** khớp selector.

```js
const items = document.querySelectorAll("li")
```

Kết quả thường là một **NodeList**.

Ví dụ:

```js
const items = document.querySelectorAll("li")
items.forEach((item) => {
  console.log(item.textContent)
})
```

---

## 3.4. So sánh nhanh

```text
getElementById   -> tìm theo id
querySelector    -> lấy phần tử đầu tiên khớp selector
querySelectorAll -> lấy tất cả phần tử khớp selector
```

---

## 4. Đọc và sửa nội dung phần tử

## 4.1. `textContent`

Dùng để đọc hoặc gán **text thuần**.

```js
const title = document.getElementById("title")
console.log(title.textContent)

title.textContent = "Xin chào JavaScript"
```

Ưu điểm:

- an toàn hơn nếu chỉ cần text
- không parse HTML bên trong

---

## 4.2. `innerHTML`

Dùng để đọc hoặc gán **HTML bên trong phần tử**.

```js
const box = document.querySelector(".box")
box.innerHTML = "<strong>Hello</strong>"
```

Lưu ý:

- `innerHTML` sẽ parse chuỗi thành HTML
- nếu dùng với dữ liệu người dùng nhập mà không kiểm soát, có thể gây vấn đề bảo mật như XSS

---

## 4.3. `innerText`

`innerText` cũng trả về text nhưng bị ảnh hưởng nhiều hơn bởi cách hiển thị CSS.

Khi mới học, bạn chỉ cần nhớ:

- thường dùng `textContent` để thao tác text
- dùng `innerHTML` khi thật sự muốn chèn HTML

---

## 5. Đọc và sửa attribute

## 5.1. `getAttribute`

```js
const link = document.querySelector("a")
console.log(link.getAttribute("href"))
```

---

## 5.2. `setAttribute`

```js
link.setAttribute("href", "https://example.com")
```

---

## 5.3. Truy cập trực tiếp một số thuộc tính phổ biến

Nhiều thuộc tính có thể truy cập trực tiếp:

```js
const input = document.querySelector("input")
console.log(input.value)

input.value = "Hello"
```

Ví dụ khác:

```js
const image = document.querySelector("img")
image.src = "photo.jpg"
image.alt = "Ảnh minh hoạ"
```

---

## 6. Class và style

## 6.1. `classList`

`classList` rất tiện để thêm / xoá / kiểm tra class.

```js
const box = document.querySelector(".box")

box.classList.add("active")
box.classList.remove("hidden")
box.classList.toggle("open")
console.log(box.classList.contains("active"))
```

---

## 6.2. Sửa style trực tiếp

```js
const title = document.querySelector("h1")
title.style.color = "red"
title.style.fontSize = "32px"
```

Lưu ý:

- trong JS, CSS kiểu `font-size` sẽ thành `fontSize`
- style trực tiếp chỉ ảnh hưởng inline style

Trong thực tế, thường ưu tiên thêm / xoá class hơn là sửa quá nhiều style trực tiếp.

---

## 7. Tạo, thêm, xoá phần tử

## 7.1. Tạo phần tử

```js
const li = document.createElement("li")
li.textContent = "Học DOM"
```

---

## 7.2. Thêm phần tử vào DOM

```js
const list = document.querySelector("ul")
list.appendChild(li)
```

Ngoài ra còn có:

```js
list.append(li)
```

---

## 7.3. Xoá phần tử

```js
li.remove()
```

Hoặc:

```js
const item = document.querySelector("li")
item.parentNode.removeChild(item)
```

---

## 7.4. Ví dụ hoàn chỉnh

```js
const list = document.querySelector("ul")
const li = document.createElement("li")
li.textContent = "Mục mới"
list.appendChild(li)
```

---

## 8. Event là gì?

## 8.1. Định nghĩa

**Event** là một sự kiện xảy ra trong trình duyệt.

Ví dụ:

- người dùng click chuột
- nhập text
- submit form
- rê chuột
- nhấn phím
- trang load xong

Khi event xảy ra, JavaScript có thể lắng nghe và xử lý.

---

## 8.2. `addEventListener`

Cách phổ biến nhất để bắt event:

```js
const button = document.querySelector("button")

button.addEventListener("click", function () {
  console.log("Đã click")
})
```

Hoặc dùng arrow function:

```js
button.addEventListener("click", () => {
  console.log("Đã click")
})
```

---

## 8.3. Các event phổ biến

- `click`
- `input`
- `change`
- `submit`
- `keydown`
- `keyup`
- `mouseover`
- `mouseout`
- `load`

---

## 9. Event object

Khi event xảy ra, callback thường nhận được một object mô tả sự kiện.

```js
button.addEventListener("click", (event) => {
  console.log(event)
})
```

Một số thuộc tính hay dùng:

- `event.target`: phần tử thực sự phát sinh event
- `event.currentTarget`: phần tử đang gắn listener
- `event.type`: loại event
- `event.key`: phím được nhấn trong keyboard event

Ví dụ:

```js
input.addEventListener("keydown", (event) => {
  console.log(event.key)
})
```

---

## 10. `this` trong event listener

Với function thường:

```js
button.addEventListener("click", function () {
  console.log(this)
})
```

Trong trường hợp này, `this` thường là chính phần tử được gắn listener.

Nhưng với arrow function:

```js
button.addEventListener("click", () => {
  console.log(this)
})
```

`this` không phải button, vì arrow function không có `this` riêng.

Khi làm việc với DOM event, nếu cần `this` là element hiện tại thì function thường thường dễ hiểu hơn.

---

## 11. Form và input

## 11.1. Bắt giá trị input

```js
const input = document.querySelector("input")

input.addEventListener("input", (event) => {
  console.log(event.target.value)
})
```

---

## 11.2. Bắt submit form

```js
const form = document.querySelector("form")

form.addEventListener("submit", (event) => {
  event.preventDefault()
  console.log("Form được submit")
})
```

Nếu không dùng `preventDefault()`, form có thể reload trang theo hành vi mặc định của trình duyệt.

---

## 12. `preventDefault()` là gì?

Dùng để **chặn hành vi mặc định** của trình duyệt.

Ví dụ với link:

```js
const link = document.querySelector("a")

link.addEventListener("click", (event) => {
  event.preventDefault()
  console.log("Không chuyển trang")
})
```

Ví dụ với form:

```js
form.addEventListener("submit", (event) => {
  event.preventDefault()
})
```

---

## 13. Event propagation

Khi một event xảy ra, nó không chỉ liên quan tới một phần tử duy nhất.

Nó có thể đi qua nhiều tầng trong cây DOM.

Quá trình này gọi là **event propagation**.

Có 3 giai đoạn chính:

1. capturing phase
2. target phase
3. bubbling phase

---

## 14. Event bubbling

## 14.1. Bubbling là gì?

Khi bạn click vào một phần tử con, event sau đó có thể nổi dần lên các phần tử cha.

Ví dụ HTML:

```html
<div id="parent">
  <button id="child">Click</button>
</div>
```

JS:

```js
const parent = document.getElementById("parent")
const child = document.getElementById("child")

parent.addEventListener("click", () => {
  console.log("parent")
})

child.addEventListener("click", () => {
  console.log("child")
})
```

Khi click button, kết quả thường là:

```js
child
parent
```

Vì event xuất phát ở button rồi nổi lên `div` cha.

---

## 14.2. Ý nghĩa của bubbling

Nhờ bubbling, bạn có thể tận dụng event delegation để xử lý nhiều phần tử con bằng một listener đặt ở phần tử cha.

---

## 15. Event capturing

Capturing là giai đoạn event đi từ ngoài vào trong.

Mặc định, `addEventListener` thường bắt ở bubbling phase.

Nếu muốn bắt ở capturing phase:

```js
parent.addEventListener(
  "click",
  () => {
    console.log("parent capture")
  },
  true
)
```

Hoặc rõ ràng hơn:

```js
parent.addEventListener("click", handler, { capture: true })
```

Khi mới học, bạn không cần quá ám ảnh capturing. Chỉ cần nhớ:

- mặc định hay gặp là bubbling
- capturing là chiều đi từ cha xuống con trước khi tới target

---

## 16. `stopPropagation()` là gì?

Dùng để **chặn event tiếp tục lan truyền** lên trên hoặc qua các bước tiếp theo.

Ví dụ:

```js
child.addEventListener("click", (event) => {
  event.stopPropagation()
  console.log("child")
})
```

Lúc này click vào child có thể chỉ in:

```js
child
```

và không nổi lên parent nữa.

---

## 17. Event delegation

## 17.1. Định nghĩa

**Event delegation** là kỹ thuật gắn event listener ở phần tử cha, sau đó dựa vào `event.target` để xử lý các phần tử con.

Ví dụ:

```html
<ul id="list">
  <li>Item 1</li>
  <li>Item 2</li>
  <li>Item 3</li>
</ul>
```

```js
const list = document.getElementById("list")

list.addEventListener("click", (event) => {
  if (event.target.tagName === "LI") {
    console.log("Bạn click vào:", event.target.textContent)
  }
})
```

---

## 17.2. Vì sao event delegation hữu ích?

Vì bạn không cần gắn listener cho từng `li` riêng lẻ.

Nó hữu ích khi:

- có rất nhiều phần tử con
- danh sách được render động
- phần tử mới được thêm sau này vẫn cần bắt event

---

## 17.3. Ví dụ thực tế

Bạn có một bảng dữ liệu có hàng trăm dòng.

Thay vì gắn 100 listener cho 100 nút xoá, bạn có thể gắn 1 listener ở table hoặc container cha, sau đó xác định nút nào được bấm bằng `event.target`.

---

## 18. DOMContentLoaded và load

## 18.1. `DOMContentLoaded`

Event này chạy khi HTML đã được parse xong và DOM đã sẵn sàng.

```js
document.addEventListener("DOMContentLoaded", () => {
  console.log("DOM đã sẵn sàng")
})
```

---

## 18.2. `load`

Event `load` thường đợi nhiều tài nguyên hơn, như ảnh, CSS, script phụ thuộc...

```js
window.addEventListener("load", () => {
  console.log("Trang đã load xong")
})
```

---

## 19. Browser APIs là gì?

JavaScript bản thân ngôn ngữ không tự có mọi thứ liên quan trình duyệt.

Những thứ như:

- `document`
- `window`
- `setTimeout`
- `localStorage`
- `fetch`
- `alert`

thường là do **browser environment** cung cấp.

Đó chính là các **Browser APIs**.

---

## 20. Một số Browser APIs hay gặp

## 20.1. `window`

`window` là object toàn cục trong môi trường trình duyệt.

Ví dụ:

```js
console.log(window.innerWidth)
console.log(window.location.href)
```

---

## 20.2. `alert`, `confirm`, `prompt`

```js
alert("Hello")
const ok = confirm("Bạn có chắc không?")
const name = prompt("Tên của bạn là gì?")
```

Hiện nay không phải lúc nào cũng dùng trong app thật, nhưng đây là các API cơ bản của browser.

---

## 20.3. `localStorage`

Dùng để lưu dữ liệu dạng key-value trong trình duyệt.

```js
localStorage.setItem("token", "abc123")
console.log(localStorage.getItem("token"))
localStorage.removeItem("token")
```

Lưu ý:

- dữ liệu lưu dưới dạng string
- thường phải kết hợp `JSON.stringify()` và `JSON.parse()` nếu lưu object

Ví dụ:

```js
const user = { name: "An", age: 20 }
localStorage.setItem("user", JSON.stringify(user))

const savedUser = JSON.parse(localStorage.getItem("user"))
console.log(savedUser.name)
```

---

## 20.4. `fetch`

Dùng để gọi HTTP request trong trình duyệt.

```js
fetch("https://api.example.com/users")
  .then((response) => response.json())
  .then((data) => {
    console.log(data)
  })
```

Hoặc dùng với `async/await`:

```js
async function loadUsers() {
  const response = await fetch("https://api.example.com/users")
  const data = await response.json()
  console.log(data)
}
```

---

## 21. Điểm dễ nhầm trong DOM và event

## 21.1. `querySelectorAll()` không trả về array thật

Nó trả về **NodeList**.

Thường có thể dùng `forEach`, nhưng không phải lúc nào cũng đầy đủ method như array.

Nếu cần, bạn có thể chuyển sang array:

```js
const items = Array.from(document.querySelectorAll("li"))
```

---

## 21.2. `textContent` khác `innerHTML`

```text
textContent -> text thuần
innerHTML   -> HTML string được parse
```

---

## 21.3. `event.target` khác `event.currentTarget`

- `event.target`: nơi event thực sự xảy ra
- `event.currentTarget`: nơi listener đang được gắn

Ví dụ với delegation, hai cái này thường khác nhau.

---

## 21.4. `preventDefault()` không giống `stopPropagation()`

- `preventDefault()` -> chặn hành vi mặc định
- `stopPropagation()` -> chặn lan truyền event

Hai cái này hoàn toàn khác mục đích.

---

## 21.5. Gắn script quá sớm có thể không tìm thấy DOM

Nếu script chạy trước khi DOM được tạo xong, dòng như:

```js
document.getElementById("title")
```

có thể trả về `null`.

Cách xử lý phổ biến:

- đặt script cuối `body`
- hoặc đợi `DOMContentLoaded`

---

## 22. Ví dụ mini: click để đổi nội dung

HTML:

```html
<h1 id="title">Xin chào</h1>
<button id="btn">Đổi nội dung</button>
```

JS:

```js
const title = document.getElementById("title")
const btn = document.getElementById("btn")

btn.addEventListener("click", () => {
  title.textContent = "Bạn vừa click nút"
})
```

---

## 23. Ví dụ mini: thêm item vào danh sách

HTML:

```html
<input id="taskInput" />
<button id="addBtn">Thêm</button>
<ul id="taskList"></ul>
```

JS:

```js
const taskInput = document.getElementById("taskInput")
const addBtn = document.getElementById("addBtn")
const taskList = document.getElementById("taskList")

addBtn.addEventListener("click", () => {
  const value = taskInput.value.trim()

  if (!value) return

  const li = document.createElement("li")
  li.textContent = value
  taskList.appendChild(li)

  taskInput.value = ""
})
```

---

## 24. Ví dụ mini: event delegation

HTML:

```html
<ul id="menu">
  <li data-id="1">Trang chủ</li>
  <li data-id="2">Sản phẩm</li>
  <li data-id="3">Liên hệ</li>
</ul>
```

JS:

```js
const menu = document.getElementById("menu")

menu.addEventListener("click", (event) => {
  if (event.target.tagName === "LI") {
    console.log("Bạn chọn mục:", event.target.dataset.id)
  }
})
```

---

## 25. Tóm tắt nhanh toàn phase

## 25.1. DOM

- DOM là cây object đại diện cho tài liệu HTML
- JavaScript thao tác giao diện thông qua DOM
- chọn phần tử bằng `getElementById`, `querySelector`, `querySelectorAll`
- đọc / sửa nội dung bằng `textContent`, `innerHTML`
- sửa thuộc tính bằng `getAttribute`, `setAttribute`
- sửa class bằng `classList`
- tạo phần tử bằng `createElement`, thêm bằng `appendChild`, xoá bằng `remove`

---

## 25.2. Event

- event là sự kiện như click, input, submit, keydown...
- bắt sự kiện bằng `addEventListener`
- callback nhận `event object`
- `preventDefault()` chặn hành vi mặc định
- `stopPropagation()` chặn lan truyền event
- bubbling là event nổi từ con lên cha
- delegation là gắn listener ở cha để xử lý các con

---

## 25.3. Browser APIs

- `window`, `document`, `localStorage`, `fetch`, `alert`, `setTimeout`...
- đây là các API do môi trường trình duyệt cung cấp
- JavaScript chạy trong browser sẽ dùng chúng để tương tác với trang web và người dùng

---

## 26. Bản đồ tư duy rút gọn

```text
Phase 7
├─ DOM
│  ├─ DOM là cây object của HTML
│  ├─ Chọn phần tử
│  ├─ Sửa nội dung / thuộc tính / style / class
│  └─ Tạo / thêm / xoá phần tử
├─ Event
│  ├─ addEventListener
│  ├─ event object
│  ├─ bubbling / capturing
│  ├─ preventDefault
│  ├─ stopPropagation
│  └─ delegation
└─ Browser APIs
   ├─ window
   ├─ localStorage
   ├─ fetch
   └─ các API do trình duyệt cung cấp
```

---

## 27. Những câu bạn nên tự trả lời được sau phase này

1. DOM là gì?
2. `querySelector` khác `querySelectorAll` thế nào?
3. `textContent` khác `innerHTML` thế nào?
4. Event là gì? `addEventListener` dùng làm gì?
5. `event.target` là gì?
6. Bubbling là gì?
7. `preventDefault()` và `stopPropagation()` khác nhau thế nào?
8. Event delegation dùng để làm gì?
9. `localStorage` lưu dữ liệu thế nào?
10. `fetch` dùng để làm gì?

---

## 28. Kết luận

Sau Phase 7, bạn đã hiểu phần quan trọng nhất của JavaScript phía trình duyệt:

- cách JS nhìn thấy giao diện
- cách JS can thiệp vào giao diện
- cách JS lắng nghe tương tác của người dùng
- cách JS dùng các Browser APIs để lưu dữ liệu, gọi API và điều khiển trang

Đây là nền tảng bắt buộc trước khi đi sâu vào framework như Vue, React hay Angular.
