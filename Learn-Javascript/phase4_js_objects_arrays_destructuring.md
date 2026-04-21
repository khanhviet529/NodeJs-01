# Phase 4 — Object, Array, Destructuring, Spread, Rest, Shallow Copy và Deep Copy trong JavaScript

## 1. Mục tiêu của Phase 4

Sau Phase 4, bạn cần nắm chắc cách JavaScript làm việc với dữ liệu kiểu object và array, vì đây là phần xuất hiện gần như hằng ngày khi đọc và viết code.

Sau khi học xong phase này, bạn cần hiểu rõ:

- object và array là gì
- cách truy cập, thêm, sửa, xóa dữ liệu trong object và array
- object và array là **reference type**
- cách duyệt dữ liệu bằng `Object.keys()`, `Object.values()`, `Object.entries()`
- các array method quan trọng như `map`, `filter`, `find`, `reduce`, `some`, `every`, `forEach`
- destructuring là gì và dùng để làm gì
- spread syntax và rest syntax khác nhau ở đâu
- shallow copy và deep copy khác nhau như thế nào
- vì sao copy object bằng spread chưa chắc đã an toàn với dữ liệu lồng nhau

---

## 2. Sơ đồ tổng quan

```text
Phase 4
├─ Object
│  ├─ tạo object
│  ├─ truy cập thuộc tính
│  ├─ thêm / sửa / xóa
│  └─ duyệt object
├─ Array
│  ├─ truy cập phần tử
│  ├─ thêm / sửa / xóa
│  └─ array methods
├─ Destructuring
│  ├─ object destructuring
│  └─ array destructuring
├─ Spread / Rest
└─ Copy dữ liệu
   ├─ shallow copy
   └─ deep copy
```

---

## 3. Object là gì?

Object là kiểu dữ liệu dùng để nhóm nhiều giá trị lại với nhau theo dạng **key - value**.

Ví dụ:

```js
const user = {
  name: "An",
  age: 20,
  city: "Hanoi"
};
```

Ở đây:

- `name`, `age`, `city` là **key**
- `"An"`, `20`, `"Hanoi"` là **value**

### Khi nào dùng object?
Dùng khi bạn muốn mô tả một thực thể có nhiều thuộc tính.

Ví dụ:

- người dùng
- sản phẩm
- đơn hàng
- cấu hình

---

## 4. Truy cập thuộc tính của object

Có 2 cách phổ biến:

### 4.1. Dot notation

```js
const user = { name: "An", age: 20 };

console.log(user.name); // An
console.log(user.age);  // 20
```

### 4.2. Bracket notation

```js
const user = { name: "An", age: 20 };

console.log(user["name"]); // An
console.log(user["age"]);  // 20
```

### Khi nào dùng bracket notation?
Khi key:

- là biến động
- có khoảng trắng
- có ký tự đặc biệt

Ví dụ:

```js
const key = "name";
const user = { name: "An" };

console.log(user[key]); // An
```

---

## 5. Thêm, sửa, xóa thuộc tính trong object

### Thêm thuộc tính

```js
const user = { name: "An" };
user.age = 20;

console.log(user); // { name: "An", age: 20 }
```

### Sửa thuộc tính

```js
const user = { name: "An", age: 20 };
user.age = 21;

console.log(user.age); // 21
```

### Xóa thuộc tính

```js
const user = { name: "An", age: 20 };
delete user.age;

console.log(user); // { name: "An" }
```

---

## 6. Kiểm tra thuộc tính trong object

### Dùng `in`

```js
const user = { name: "An" };

console.log("name" in user); // true
console.log("age" in user);  // false
```

### Dùng `hasOwnProperty`

```js
const user = { name: "An" };

console.log(user.hasOwnProperty("name")); // true
```

`hasOwnProperty()` chỉ kiểm tra thuộc tính nằm trực tiếp trên object đó, không tính thuộc tính đến từ prototype chain.

---

## 7. Duyệt object

### `Object.keys()`
Trả về mảng chứa các key.

```js
const user = { name: "An", age: 20 };
console.log(Object.keys(user)); // ["name", "age"]
```

### `Object.values()`
Trả về mảng chứa các value.

```js
const user = { name: "An", age: 20 };
console.log(Object.values(user)); // ["An", 20]
```

### `Object.entries()`
Trả về mảng chứa từng cặp `[key, value]`.

```js
const user = { name: "An", age: 20 };
console.log(Object.entries(user));
// [["name", "An"], ["age", 20]]
```

### Duyệt bằng `for...in`

```js
const user = { name: "An", age: 20 };

for (const key in user) {
  console.log(key, user[key]);
}
```

---

## 8. Array là gì?

Array là kiểu dữ liệu dùng để lưu **nhiều giá trị có thứ tự**.

Ví dụ:

```js
const fruits = ["apple", "banana", "orange"];
```

### Đặc điểm của array

- có thứ tự
- truy cập bằng index
- index bắt đầu từ `0`
- cũng là một dạng đặc biệt của object

Ví dụ:

```js
console.log(typeof []); // "object"
Array.isArray([]);      // true
```

---

## 9. Truy cập, thêm, sửa, xóa phần tử trong array

### Truy cập phần tử

```js
const fruits = ["apple", "banana", "orange"];

console.log(fruits[0]); // apple
console.log(fruits[1]); // banana
```

### Sửa phần tử

```js
const fruits = ["apple", "banana", "orange"];
fruits[1] = "mango";

console.log(fruits); // ["apple", "mango", "orange"]
```

### Thêm phần tử cuối mảng

```js
const nums = [1, 2];
nums.push(3);

console.log(nums); // [1, 2, 3]
```

### Xóa phần tử cuối mảng

```js
const nums = [1, 2, 3];
nums.pop();

console.log(nums); // [1, 2]
```

### Thêm phần tử đầu mảng

```js
const nums = [2, 3];
nums.unshift(1);

console.log(nums); // [1, 2, 3]
```

### Xóa phần tử đầu mảng

```js
const nums = [1, 2, 3];
nums.shift();

console.log(nums); // [2, 3]
```

---

## 10. Object và Array là reference type

Đây là phần cực kỳ quan trọng.

Khi bạn gán object hoặc array sang biến khác, JavaScript **không copy toàn bộ dữ liệu**, mà chỉ copy **tham chiếu**.

### Ví dụ với object

```js
const user = { name: "An" };
const admin = user;

admin.name = "Bình";

console.log(user.name);  // Bình
console.log(admin.name); // Bình
console.log(user === admin); // true
```

### Ví dụ với array

```js
const arr1 = [1, 2, 3];
const arr2 = arr1;

arr2.push(4);

console.log(arr1); // [1, 2, 3, 4]
console.log(arr2); // [1, 2, 3, 4]
console.log(arr1 === arr2); // true
```

### Cách nhớ nhanh

```text
Primitive  -> copy giá trị
Reference  -> copy tham chiếu
```

---

## 11. Các array method quan trọng

## 11.1. `forEach()`
Dùng để duyệt từng phần tử, nhưng **không tạo mảng mới**.

```js
const nums = [1, 2, 3];

nums.forEach((num) => {
  console.log(num);
});
```

### Khi dùng?
Khi bạn chỉ muốn làm gì đó với từng phần tử, ví dụ log, gọi API, cập nhật giao diện.

---

## 11.2. `map()`
Dùng để biến đổi từng phần tử và trả về **mảng mới**.

```js
const nums = [1, 2, 3];
const doubled = nums.map((num) => num * 2);

console.log(doubled); // [2, 4, 6]
```

### Khi dùng?
Khi bạn muốn chuyển đổi dữ liệu.

Ví dụ:

- lấy danh sách tên
- đổi định dạng dữ liệu
- render danh sách mới

---

## 11.3. `filter()`
Dùng để lọc phần tử theo điều kiện và trả về **mảng mới**.

```js
const nums = [1, 2, 3, 4, 5];
const evenNums = nums.filter((num) => num % 2 === 0);

console.log(evenNums); // [2, 4]
```

### Khi dùng?
Khi bạn muốn lấy ra một nhóm dữ liệu thỏa điều kiện.

---

## 11.4. `find()`
Dùng để tìm **phần tử đầu tiên** thỏa điều kiện.

```js
const users = [
  { id: 1, name: "An" },
  { id: 2, name: "Bình" }
];

const user = users.find((u) => u.id === 2);
console.log(user); // { id: 2, name: "Bình" }
```

Nếu không tìm thấy thì trả về `undefined`.

---

## 11.5. `some()`
Kiểm tra xem **ít nhất một** phần tử thỏa điều kiện hay không.

```js
const nums = [1, 3, 5, 6];
console.log(nums.some((num) => num % 2 === 0)); // true
```

---

## 11.6. `every()`
Kiểm tra xem **tất cả** phần tử có thỏa điều kiện hay không.

```js
const nums = [2, 4, 6];
console.log(nums.every((num) => num % 2 === 0)); // true
```

---

## 11.7. `reduce()`
Dùng để gom mảng thành một giá trị duy nhất.

```js
const nums = [1, 2, 3, 4];
const sum = nums.reduce((total, num) => total + num, 0);

console.log(sum); // 10
```

### `reduce()` thường dùng để làm gì?

- tính tổng
- đếm số lượng
- nhóm dữ liệu
- chuyển mảng thành object

Ví dụ đếm số lần xuất hiện:

```js
const fruits = ["apple", "banana", "apple"];

const result = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});

console.log(result);
// { apple: 2, banana: 1 }
```

---

## 12. So sánh nhanh các array method

| Method | Trả về | Mục đích chính |
|---|---|---|
| `forEach()` | `undefined` | duyệt từng phần tử |
| `map()` | mảng mới | biến đổi dữ liệu |
| `filter()` | mảng mới | lọc dữ liệu |
| `find()` | 1 phần tử hoặc `undefined` | tìm phần tử đầu tiên |
| `some()` | boolean | kiểm tra có ít nhất 1 phần tử đúng |
| `every()` | boolean | kiểm tra tất cả đúng |
| `reduce()` | bất kỳ kiểu gì | gom dữ liệu |

---

## 13. Destructuring là gì?

Destructuring là cú pháp giúp **lấy dữ liệu từ object hoặc array ra thành biến** một cách nhanh gọn.

---

## 14. Object destructuring

```js
const user = {
  name: "An",
  age: 20,
  city: "Hanoi"
};

const { name, age } = user;

console.log(name); // An
console.log(age);  // 20
```

### Đổi tên biến khi destructure

```js
const user = { name: "An", age: 20 };

const { name: userName, age: userAge } = user;

console.log(userName); // An
console.log(userAge);  // 20
```

### Gán giá trị mặc định

```js
const user = { name: "An" };
const { name, age = 18 } = user;

console.log(name); // An
console.log(age);  // 18
```

### Destructure trong parameter

```js
function showUser({ name, age }) {
  console.log(name, age);
}

showUser({ name: "An", age: 20 });
```

---

## 15. Array destructuring

```js
const fruits = ["apple", "banana", "orange"];
const [first, second] = fruits;

console.log(first);  // apple
console.log(second); // banana
```

### Bỏ qua phần tử

```js
const nums = [10, 20, 30];
const [a, , c] = nums;

console.log(a); // 10
console.log(c); // 30
```

### Gán giá trị mặc định

```js
const arr = [1];
const [a, b = 2] = arr;

console.log(a); // 1
console.log(b); // 2
```

### Hoán đổi giá trị

```js
let a = 1;
let b = 2;

[a, b] = [b, a];

console.log(a); // 2
console.log(b); // 1
```

---

## 16. Spread syntax là gì?

Spread syntax dùng dấu `...` để **trải** phần tử của array hoặc thuộc tính của object ra.

### Với array

```js
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4];

console.log(arr2); // [1, 2, 3, 4]
```

### Với object

```js
const user = { name: "An", age: 20 };
const admin = { ...user, role: "admin" };

console.log(admin);
// { name: "An", age: 20, role: "admin" }
```

### Dùng để clone nhanh

```js
const arr1 = [1, 2, 3];
const arr2 = [...arr1];

const user1 = { name: "An" };
const user2 = { ...user1 };
```

---

## 17. Rest syntax là gì?

Rest syntax cũng dùng dấu `...`, nhưng mục đích là **gom phần còn lại** thành một nhóm.

### Với array destructuring

```js
const nums = [1, 2, 3, 4];
const [first, ...rest] = nums;

console.log(first); // 1
console.log(rest);  // [2, 3, 4]
```

### Với object destructuring

```js
const user = { name: "An", age: 20, city: "Hanoi" };
const { name, ...others } = user;

console.log(name);   // An
console.log(others); // { age: 20, city: "Hanoi" }
```

### Với function parameter

```js
function sum(...nums) {
  return nums.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4)); // 10
```

---

## 18. Spread và Rest khác nhau như thế nào?

Cùng dùng `...` nhưng ý nghĩa khác nhau.

### Spread
Là **trải ra**.

```js
const arr = [1, 2, 3];
console.log(...arr); // 1 2 3
```

### Rest
Là **gom lại**.

```js
const [first, ...rest] = [1, 2, 3];
console.log(rest); // [2, 3]
```

### Cách nhớ

```text
Spread -> trải ra
Rest   -> gom lại phần còn lại
```

---

## 19. Shallow copy là gì?

Shallow copy là **copy nông**.

Nghĩa là:

- tầng đầu tiên được copy ra biến mới
- nhưng nếu bên trong còn object hoặc array lồng nhau, thì phần lồng nhau vẫn dùng chung tham chiếu

### Ví dụ

```js
const user = {
  name: "An",
  address: {
    city: "Hanoi"
  }
};

const copy = { ...user };
copy.address.city = "Danang";

console.log(user.address.city); // Danang
console.log(copy.address.city); // Danang
console.log(user === copy); // false
console.log(user.address === copy.address); // true
```

### Giải thích

- `user` và `copy` là 2 object khác nhau
- nhưng `address` bên trong vẫn là cùng 1 object

Đây chính là **shallow copy**.

---

## 20. Deep copy là gì?

Deep copy là **copy sâu**.

Nghĩa là:

- tạo ra dữ liệu mới hoàn toàn
- cả các object lồng nhau cũng được copy riêng
- sửa bên copy không làm ảnh hưởng dữ liệu gốc

### Ví dụ dùng `structuredClone()`

```js
const user = {
  name: "An",
  address: {
    city: "Hanoi"
  }
};

const copy = structuredClone(user);
copy.address.city = "Danang";

console.log(user.address.city); // Hanoi
console.log(copy.address.city); // Danang
```

---

## 21. Một số cách deep copy phổ biến

### 21.1. `structuredClone()`
Đây là cách hiện đại, rõ ràng hơn trong nhiều trường hợp.

```js
const newObj = structuredClone(oldObj);
```

### 21.2. `JSON.parse(JSON.stringify(obj))`
Cách cũ, dùng được trong trường hợp đơn giản.

```js
const copy = JSON.parse(JSON.stringify(obj));
```

### Hạn chế của cách JSON
Có thể mất dữ liệu với:

- `undefined`
- function
- `Date`
- `Map`
- `Set`
- `Symbol`

Nên không phải lúc nào cũng phù hợp.

---

## 22. So sánh nhanh shallow copy và deep copy

| Loại copy | Tầng đầu tiên | Dữ liệu lồng nhau |
|---|---|---|
| Shallow copy | tạo mới | vẫn có thể dùng chung tham chiếu |
| Deep copy | tạo mới | cũng được copy riêng hoàn toàn |

---

## 23. Những điểm rất hay nhầm

## 23.1. `const` object không có nghĩa là object bất biến

```js
const user = { name: "An" };
user.name = "Bình"; // vẫn được
```

`const` chỉ có nghĩa là **không gán lại biến sang object khác**, chứ không khóa nội dung bên trong object.

---

## 23.2. Spread copy object chưa chắc là deep copy

```js
const a = { info: { age: 20 } };
const b = { ...a };

b.info.age = 30;
console.log(a.info.age); // 30
```

Nhiều người tưởng `{ ...a }` là copy hoàn toàn, nhưng thực ra chỉ là **copy nông**.

---

## 23.3. `map()` khác `forEach()`

- `forEach()` dùng để duyệt, không trả mảng mới hữu ích
- `map()` trả về mảng mới

```js
const nums = [1, 2, 3];

const a = nums.forEach((n) => n * 2);
const b = nums.map((n) => n * 2);

console.log(a); // undefined
console.log(b); // [2, 4, 6]
```

---

## 23.4. `find()` khác `filter()`

- `find()` trả về **1 phần tử đầu tiên**
- `filter()` trả về **mảng các phần tử**

```js
const nums = [1, 2, 3, 4];

console.log(nums.find((n) => n > 2));   // 3
console.log(nums.filter((n) => n > 2)); // [3, 4]
```

---

## 24. Khi đi làm thực tế, Phase 4 thường được dùng ở đâu?

Rất nhiều nơi:

- đọc dữ liệu API
- render danh sách giao diện
- lọc và tìm kiếm dữ liệu
- clone state trong React/Vue
- transform dữ liệu trước khi gửi API
- lấy nhanh thuộc tính từ object bằng destructuring
- merge cấu hình bằng spread

Ví dụ rất thực tế:

```js
const response = {
  data: {
    user: {
      name: "An",
      age: 20
    }
  }
};

const {
  data: {
    user: { name, age }
  }
} = response;

console.log(name, age); // An 20
```

---

## 25. Tóm tắt nhanh toàn bộ Phase 4

```text
Object
- nhóm dữ liệu theo key-value
- truy cập bằng dot hoặc bracket

Array
- dữ liệu có thứ tự
- truy cập bằng index

Reference
- object và array copy theo tham chiếu

Array methods
- forEach: duyệt
- map: biến đổi
- filter: lọc
- find: tìm phần tử đầu tiên
- some: ít nhất một đúng
- every: tất cả đúng
- reduce: gom dữ liệu

Destructuring
- lấy dữ liệu từ object/array ra biến nhanh hơn

Spread
- trải dữ liệu ra

Rest
- gom phần còn lại

Shallow copy
- copy tầng ngoài
- nested object vẫn dùng chung tham chiếu

Deep copy
- copy toàn bộ sâu bên trong
```

---

## 26. Checklist tự kiểm tra sau khi học xong Phase 4

Bạn nên tự trả lời được các câu sau:

- Vì sao object và array là reference type?
- `map()` khác `forEach()` ở đâu?
- `find()` khác `filter()` ở đâu?
- `{ ...obj }` là shallow copy hay deep copy?
- Vì sao sửa nested object trong bản copy lại làm object gốc đổi theo?
- Spread và rest đều dùng `...`, nhưng mục đích khác nhau thế nào?
- Destructuring giúp code gọn hơn ra sao?

---

## 27. Kết luận

Phase 4 là một trong những phase quan trọng nhất của JavaScript thực tế.

Nếu Phase 1 và Phase 2 giúp bạn hiểu nền tảng của biến, kiểu dữ liệu và ép kiểu, còn Phase 3 giúp bạn hiểu function, `this`, closure, thì Phase 4 giúp bạn thao tác với dữ liệu hằng ngày.

Nắm chắc phase này sẽ giúp bạn:

- đọc code dự án dễ hơn
- xử lý API tốt hơn
- hiểu state management tốt hơn
- tránh được nhiều bug liên quan đến copy tham chiếu

