# Phase 6: Async JavaScript và Runtime

## 1. Mục tiêu của phase này

Phase 6 giúp bạn hiểu:

- JavaScript xử lý **bất đồng bộ** như thế nào
- vì sao `setTimeout(..., 0)` không chạy ngay
- Promise khác callback ở đâu
- `async/await` thực chất hoạt động dựa trên cái gì
- event loop là gì
- call stack, Web APIs, task queue, microtask queue hoạt động ra sao

Đây là một trong những phase quan trọng nhất, vì khi đi làm bạn sẽ gặp liên tục:

- gọi API
- đọc file
- timer
- xử lý click / input / event
- Promise
- `async/await`
- bug liên quan đến thứ tự log

---

## 2. Synchronous và Asynchronous

## 2.1. Synchronous là gì?

**Synchronous** nghĩa là code chạy **tuần tự, dòng sau chờ dòng trước xong mới chạy**.

Ví dụ:

```js
console.log("A")
console.log("B")
console.log("C")
```

Kết quả:

```js
A
B
C
```

Luồng chạy rất đơn giản:

```text
A xong -> B xong -> C xong
```

---

## 2.2. Asynchronous là gì?

**Asynchronous** nghĩa là có những tác vụ không cần chặn toàn bộ chương trình để chờ nó hoàn thành.

Ví dụ:

```js
console.log("A")
setTimeout(() => {
  console.log("B")
}, 1000)
console.log("C")
```

Kết quả:

```js
A
C
B
```

Vì:

- `setTimeout` chỉ **đăng ký một callback để chạy sau**
- chương trình không đứng yên đợi 1 giây
- nên `console.log("C")` chạy trước

---

## 2.3. Vì sao JavaScript cần asynchronous?

JavaScript ban đầu chạy chủ yếu trên trình duyệt.

Nếu mọi tác vụ đều chặn luồng chính, thì khi:

- gọi API
- chờ timer
- chờ người dùng click
- đọc file

ứng dụng sẽ bị đơ.

Bất đồng bộ giúp JS:

- không bị block trong nhiều tác vụ chờ
- giữ UI phản hồi tốt hơn
- xử lý được các tác vụ như network, timer, event

---

## 3. Callback

## 3.1. Callback là gì?

**Callback** là một hàm được truyền vào hàm khác để được gọi lại sau.

Ví dụ:

```js
function greet(name, callback) {
  console.log("Xin chào", name)
  callback()
}

greet("An", function () {
  console.log("Đã chào xong")
})
```

Kết quả:

```js
Xin chào An
Đã chào xong
```

---

## 3.2. Callback trong bất đồng bộ

Ví dụ:

```js
setTimeout(() => {
  console.log("Chạy sau 1 giây")
}, 1000)
```

Hàm arrow truyền vào `setTimeout` chính là callback.

---

## 3.3. Callback hell

Khi callback lồng quá nhiều tầng, code rất khó đọc:

```js
login(user, function () {
  getProfile(function () {
    getPosts(function () {
      getComments(function () {
        console.log("xong")
      })
    })
  })
})
```

Vấn đề:

- khó đọc
- khó debug
- khó xử lý lỗi
- khó bảo trì

Đây là một trong những lý do Promise ra đời.

---

## 4. Promise

## 4.1. Promise là gì?

**Promise** là một object đại diện cho kết quả của một tác vụ bất đồng bộ, có thể:

- chưa xong
- thành công
- thất bại

Có 3 trạng thái:

- `pending`: đang chờ
- `fulfilled`: thành công
- `rejected`: thất bại

---

## 4.2. Cú pháp cơ bản

```js
const promise = new Promise((resolve, reject) => {
  const success = true

  if (success) {
    resolve("Thành công")
  } else {
    reject("Thất bại")
  }
})
```

- `resolve(value)` => báo thành công
- `reject(error)` => báo thất bại

---

## 4.3. Dùng `.then()` và `.catch()`

```js
promise
  .then((result) => {
    console.log(result)
  })
  .catch((error) => {
    console.log(error)
  })
```

- `.then()` xử lý khi thành công
- `.catch()` xử lý khi lỗi

---

## 4.4. Ví dụ Promise với timer

```js
const p = new Promise((resolve) => {
  setTimeout(() => {
    resolve("Đã xong")
  }, 1000)
})

p.then((value) => {
  console.log(value)
})
```

Sau 1 giây sẽ in:

```js
Đã xong
```

---

## 4.5. `.finally()`

`finally()` chạy sau cùng, bất kể thành công hay thất bại.

```js
fetchData()
  .then((data) => {
    console.log(data)
  })
  .catch((err) => {
    console.log(err)
  })
  .finally(() => {
    console.log("Luôn chạy")
  })
```

---

## 4.6. Promise chain

```js
getUser()
  .then((user) => getPosts(user.id))
  .then((posts) => getComments(posts[0].id))
  .then((comments) => console.log(comments))
  .catch((err) => console.log(err))
```

Ưu điểm so với callback hell:

- phẳng hơn
- dễ đọc hơn
- xử lý lỗi tập trung hơn

---

## 4.7. Điểm dễ nhầm về Promise

### Promise không có nghĩa là chạy song song

Promise chỉ là cách biểu diễn và xử lý kết quả bất đồng bộ.

Nó **không tự động** làm mọi thứ thành đa luồng.

---

### `.then()` không chạy ngay lập tức trên call stack hiện tại

Callback của `.then()` được đưa vào **microtask queue**.

Đây là lý do Promise thường có thứ tự ưu tiên khác với `setTimeout`.

---

## 5. async / await

## 5.1. `async` là gì?

Khi đặt `async` trước một function, function đó **luôn trả về Promise**.

Ví dụ:

```js
async function test() {
  return 123
}
```

Thực chất giống gần như:

```js
function test() {
  return Promise.resolve(123)
}
```

---

## 5.2. `await` là gì?

`await` dùng để chờ một Promise hoàn thành bên trong hàm `async`.

Ví dụ:

```js
function delay() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("xong"), 1000)
  })
}

async function run() {
  const result = await delay()
  console.log(result)
}

run()
```

Sau 1 giây sẽ in:

```js
xong
```

---

## 5.3. `async/await` có phải bỏ được Promise không?

Không.

`async/await` chỉ là **cú pháp dễ đọc hơn để làm việc với Promise**.

Bản chất bên dưới vẫn là Promise.

---

## 5.4. Xử lý lỗi với `try...catch`

```js
async function run() {
  try {
    const data = await fetchData()
    console.log(data)
  } catch (error) {
    console.log("Lỗi:", error)
  }
}
```

Đây là lý do nhiều người thích `async/await` hơn `.then().catch()` khi luồng dài.

---

## 5.5. Chạy tuần tự và song song

### Tuần tự

```js
const a = await task1()
const b = await task2()
```

Ở đây `task2()` chờ `task1()` xong.

### Song song

```js
const [a, b] = await Promise.all([task1(), task2()])
```

Ở đây cả hai chạy cùng lúc rồi chờ kết quả chung.

---

## 5.6. Điểm dễ nhầm

### `await` không chặn cả chương trình

`await` chỉ tạm dừng **bên trong hàm async đó**, không đóng băng toàn bộ JS runtime.

---

### Không phải chỗ nào cũng dùng được `await`

Thông thường `await` phải nằm trong:

- `async function`
- hoặc môi trường hỗ trợ top-level await

---

## 6. setTimeout và setInterval

## 6.1. `setTimeout`

Dùng để chạy callback **một lần** sau một khoảng thời gian.

```js
setTimeout(() => {
  console.log("Chạy sau 2 giây")
}, 2000)
```

---

## 6.2. `setInterval`

Dùng để chạy callback **lặp lại liên tục** sau mỗi khoảng thời gian.

```js
const id = setInterval(() => {
  console.log("Lặp lại")
}, 1000)
```

Muốn dừng:

```js
clearInterval(id)
```

---

## 6.3. `setTimeout(..., 0)` có chạy ngay không?

Không.

`0` không có nghĩa là chạy ngay lập tức.

Nó chỉ có nghĩa là:

- callback sẵn sàng được xếp hàng **sớm nhất có thể**
- nhưng vẫn phải chờ call stack hiện tại trống
- và còn phụ thuộc queue nào đang chờ

Ví dụ:

```js
console.log("A")
setTimeout(() => {
  console.log("B")
}, 0)
console.log("C")
```

Kết quả:

```js
A
C
B
```

---

## 6.4. Vì sao `setInterval` dễ gây bug?

Vì nếu callback chạy lâu hơn interval, có thể xảy ra:

- chồng lịch
- khó kiểm soát nhịp thực tế
- khó dừng đúng lúc

Trong nhiều trường hợp thực tế, người ta dùng `setTimeout` lặp thủ công thay vì `setInterval` để kiểm soát tốt hơn.

---

## 7. Runtime của JavaScript

Đây là phần cốt lõi của phase này.

JavaScript engine không tự mình làm tất cả mọi việc. Trong môi trường như browser hoặc Node.js, JS làm việc cùng runtime.

Các khái niệm quan trọng:

- call stack
- Web APIs / host APIs
- callback queue / task queue
- microtask queue
- event loop

---

## 8. Call Stack

## 8.1. Call stack là gì?

Call stack là nơi theo dõi **hàm nào đang chạy**.

Ví dụ:

```js
function a() {
  b()
}

function b() {
  c()
}

function c() {
  console.log("Hello")
}

a()
```

Luồng stack:

```text
push a
push b
push c
c xong -> pop c
b xong -> pop b
a xong -> pop a
```

---

## 8.2. Stack overflow

Nếu hàm gọi đệ quy vô hạn, stack sẽ đầy.

```js
function test() {
  test()
}

test()
```

Sẽ gây lỗi kiểu:

```text
Maximum call stack size exceeded
```

---

## 9. Web APIs / Host APIs

Trong browser, những thứ như:

- `setTimeout`
- DOM events
- `fetch`

không phải do JavaScript engine thuần tự xử lý hết.

Chúng được môi trường cung cấp.

Có thể hình dung:

```text
JS engine + browser runtime
```

Khi bạn gọi `setTimeout`, callback không nằm chờ trong call stack suốt 1 giây. Nó được giao cho môi trường theo dõi thời gian.

---

## 10. Task Queue (Macrotask Queue)

Task queue chứa các callback chờ được chạy khi call stack rỗng.

Ví dụ thường vào task queue:

- `setTimeout`
- `setInterval`
- một số event callback

Ví dụ:

```js
setTimeout(() => {
  console.log("timeout")
}, 0)
```

Callback này sẽ vào task queue khi tới thời điểm thích hợp.

---

## 11. Microtask Queue

Microtask queue có mức ưu tiên cao hơn task queue.

Ví dụ thường vào microtask queue:

- `.then()` / `.catch()` / `.finally()` của Promise
- `queueMicrotask()`
- phần tiếp tục của `await`

Ví dụ:

```js
Promise.resolve().then(() => {
  console.log("microtask")
})
```

---

## 11.1. Microtask ưu tiên hơn task queue

Ví dụ:

```js
console.log("A")

setTimeout(() => {
  console.log("timeout")
}, 0)

Promise.resolve().then(() => {
  console.log("promise")
})

console.log("B")
```

Kết quả:

```js
A
B
promise
timeout
```

Giải thích:

1. `A` chạy ngay
2. `setTimeout` đăng ký callback vào task queue
3. Promise `.then()` vào microtask queue
4. `B` chạy ngay
5. call stack rỗng
6. event loop ưu tiên xử lý microtask trước
7. in `promise`
8. rồi mới tới task queue => `timeout`

---

## 12. Event Loop

## 12.1. Event loop là gì?

Event loop là cơ chế liên tục kiểm tra:

- call stack có rỗng chưa
- có microtask nào đang chờ không
- có task nào đang chờ không

Và quyết định callback nào được đẩy vào stack để chạy tiếp.

---

## 12.2. Hình dung đơn giản

```text
1. Chạy code đồng bộ trên call stack
2. Nếu stack rỗng:
   - xử lý hết microtask queue trước
3. Sau đó mới lấy task từ task queue
4. Lặp lại liên tục
```

---

## 12.3. Hình dung trực quan

```text
Code sync -> Call Stack
Timer/API  -> Web APIs
Promise    -> Microtask Queue
Timeout    -> Task Queue
Event Loop -> điều phối callback quay lại Call Stack
```

---

## 13. Ví dụ kinh điển về thứ tự log

## 13.1. Ví dụ 1

```js
console.log(1)

setTimeout(() => {
  console.log(2)
}, 0)

Promise.resolve().then(() => {
  console.log(3)
})

console.log(4)
```

Kết quả:

```js
1
4
3
2
```

Giải thích:

- `1` sync
- đăng ký timeout
- Promise `.then()` vào microtask
- `4` sync
- stack rỗng -> xử lý microtask `3`
- sau đó task queue -> `2`

---

## 13.2. Ví dụ 2

```js
setTimeout(() => {
  console.log("timeout 1")

  Promise.resolve().then(() => {
    console.log("promise in timeout")
  })
}, 0)

setTimeout(() => {
  console.log("timeout 2")
}, 0)
```

Kết quả thường là:

```js
timeout 1
promise in timeout
timeout 2
```

Vì:

- chạy `timeout 1`
- xong callback đó, microtask bên trong nó được xử lý trước
- rồi mới tới task tiếp theo là `timeout 2`

---

## 14. `async/await` với event loop

Ví dụ:

```js
console.log("A")

async function test() {
  console.log("B")
  await Promise.resolve()
  console.log("C")
}

test()
console.log("D")
```

Kết quả:

```js
A
B
D
C
```

Giải thích:

- `A` sync
- gọi `test()`
- `B` sync trong hàm async
- gặp `await`, phần sau `await` được tách ra như một microtask
- quay lại ngoài hàm
- `D` sync
- stack rỗng -> microtask chạy -> `C`

---

## 15. Promise.all, Promise.allSettled, Promise.race, Promise.any

## 15.1. `Promise.all()`

Chờ tất cả promise thành công.

- nếu 1 promise reject -> reject ngay

```js
const [a, b] = await Promise.all([task1(), task2()])
```

---

## 15.2. `Promise.allSettled()`

Chờ tất cả hoàn thành, bất kể thành công hay thất bại.

Phù hợp khi bạn muốn biết kết quả toàn bộ.

---

## 15.3. `Promise.race()`

Promise nào xong trước thì lấy kết quả của promise đó.

---

## 15.4. `Promise.any()`

Promise nào **fulfilled** đầu tiên thì lấy.

Nếu tất cả reject thì mới lỗi.

---

## 16. Những điểm rất hay nhầm

## 16.1. JavaScript là single-threaded có nghĩa gì?

Nghĩa là call stack chính của JS chạy từng việc một.

Nhưng không có nghĩa toàn bộ môi trường chỉ làm được một việc duy nhất.

Môi trường runtime vẫn có thể hỗ trợ timer, I/O, network, event...

---

## 16.2. `setTimeout` không bảo đảm thời gian chính xác tuyệt đối

`setTimeout(fn, 1000)` không có nghĩa đúng 1000ms là callback chạy ngay.

Nó chỉ có nghĩa là:

- sau ít nhất khoảng đó, callback đủ điều kiện vào queue
- còn phải chờ stack rỗng và các queue khác

---

## 16.3. Promise nhanh hơn `setTimeout`

Không phải vì Promise “mạnh hơn” timer, mà vì callback của Promise vào **microtask queue**, mà microtask được ưu tiên trước task queue.

---

## 16.4. `await` không biến code thành đồng bộ hoàn toàn

`await` chỉ làm code trông giống đồng bộ hơn.

Bản chất nó vẫn là cơ chế bất đồng bộ dựa trên Promise.

---

## 17. So sánh nhanh

## 17.1. Callback vs Promise vs async/await

### Callback

- đơn giản cho case nhỏ
- dễ rơi vào callback hell
- khó xử lý lỗi khi luồng dài

### Promise

- rõ ràng hơn callback
- chain tốt hơn
- phù hợp cho xử lý async hiện đại

### async/await

- dễ đọc nhất
- nhìn gần giống code đồng bộ
- xử lý lỗi đẹp với `try...catch`

---

## 17.2. `setTimeout` vs `setInterval`

### `setTimeout`
- chạy 1 lần
- dễ kiểm soát hơn

### `setInterval`
- chạy lặp lại
- dễ bị chồng nhịp nếu callback nặng

---

## 17.3. Task queue vs Microtask queue

### Task queue
- `setTimeout`
- `setInterval`
- một số event callback

### Microtask queue
- Promise `.then()`
- `await`
- `queueMicrotask()`

**Microtask luôn được ưu tiên trước task queue khi call stack rỗng.**

---

## 18. Các mẫu code quan trọng cần nhớ

## 18.1. Thứ tự log cơ bản

```js
console.log("start")

setTimeout(() => {
  console.log("timeout")
}, 0)

Promise.resolve().then(() => {
  console.log("promise")
})

console.log("end")
```

Kết quả:

```js
start
end
promise
timeout
```

---

## 18.2. `async/await`

```js
async function main() {
  console.log("1")
  await Promise.resolve()
  console.log("2")
}

main()
console.log("3")
```

Kết quả:

```js
1
3
2
```

---

## 18.3. Promise chain

```js
getUser()
  .then((user) => getPosts(user.id))
  .then((posts) => getComments(posts[0].id))
  .catch((err) => console.log(err))
```

---

## 18.4. Chạy song song với Promise.all

```js
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts(),
])
```

---

## 19. Checklist bạn cần hiểu sau phase này

Sau khi xong Phase 6, bạn nên tự giải thích được:

- synchronous và asynchronous khác nhau thế nào
- callback là gì
- Promise là gì
- `resolve`, `reject`, `then`, `catch`, `finally` là gì
- `async/await` hoạt động ra sao
- `setTimeout(..., 0)` có chạy ngay không
- `setInterval` khác `setTimeout` ở đâu
- call stack là gì
- task queue là gì
- microtask queue là gì
- event loop là gì
- vì sao Promise callback thường chạy trước `setTimeout`

---

## 20. Tóm tắt Phase 6

```text
1. JavaScript chạy sync trên call stack
2. Timer, event, network được môi trường runtime hỗ trợ
3. Callback là hàm được gọi lại sau
4. Promise giúp xử lý async tốt hơn callback hell
5. async/await chỉ là cú pháp đẹp hơn của Promise
6. setTimeout không chạy ngay cả khi delay = 0
7. Promise callback vào microtask queue
8. setTimeout callback vào task queue
9. Event loop ưu tiên xử lý microtask trước task queue
```

---

## 21. Những lỗi tư duy rất hay gặp

- tưởng `setTimeout(..., 0)` là chạy ngay
- tưởng `await` chặn cả chương trình
- tưởng Promise là đa luồng
- tưởng `async/await` không còn liên quan Promise
- nhầm task queue và microtask queue
- không hiểu vì sao thứ tự log lại là `1 4 3 2`

---

## 22. Kết luận

Phase 6 là phần giúp bạn chuyển từ việc “viết JS được” sang “đọc và giải thích được JS runtime đang làm gì”.

Nếu chưa chắc phase này, bạn sẽ rất dễ:

- đoán sai thứ tự log
- viết async bị lỗi
- dùng Promise và `await` theo cảm tính
- khó debug bug liên quan timer, API, hoặc event

Nếu chắc phase này, bạn sẽ hiểu sâu hơn gần như toàn bộ phần bất đồng bộ trong JavaScript hiện đại.
