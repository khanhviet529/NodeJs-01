# Phase 5: Prototype, Class, và Kế thừa trong JavaScript

## 1. Mục tiêu của phase này

Ở phase này, mục tiêu là hiểu bản chất object-oriented trong JavaScript.

Nhiều người mới học thường nghĩ JavaScript kế thừa giống Java, C++, C# theo kiểu class thuần túy. Nhưng thực tế:

- JavaScript kế thừa chủ yếu dựa trên **prototype**
- `class` trong JavaScript chỉ là **syntactic sugar**
- bản chất phía dưới vẫn là **prototype chain**

Nếu không hiểu phase này, bạn sẽ rất dễ:

- nhầm giữa `prototype` và `__proto__`
- không hiểu vì sao object lại gọi được method không nằm trực tiếp trên nó
- dùng `class` nhưng không hiểu cơ chế thật
- khó đọc các đoạn code constructor function cũ

---

## 2. Tư duy cốt lõi cần nắm

JavaScript không copy method vào từng object một cách ngây thơ.

Thay vào đó:

- object có thể liên kết tới một object khác
- khi không tìm thấy thuộc tính trên chính nó, JS sẽ đi tìm ở object liên kết đó
- chuỗi tìm kiếm này gọi là **prototype chain**

Hình dung:

```text
obj
 ↓
prototype 1
 ↓
prototype 2
 ↓
Object.prototype
 ↓
null
```

Khi bạn gọi:

```js
obj.toString()
```

JS sẽ tìm theo thứ tự:

1. trên chính `obj`
2. nếu không có, tìm trên prototype của `obj`
3. nếu vẫn không có, tìm tiếp trên prototype phía trên
4. đến khi gặp `null` thì dừng

---

## 3. Prototype là gì?

**Prototype** là object mà object khác có thể dùng để kế thừa thuộc tính và phương thức.

Nói dễ hiểu:

- một object có thể “tra cứu lên cha”
- “cha” đó chính là prototype

Ví dụ:

```js
const animal = {
  eat() {
    console.log("eating...")
  }
}

const dog = {
  bark() {
    console.log("gâu gâu")
  }
}

Object.setPrototypeOf(dog, animal)

dog.bark() // gâu gâu
dog.eat()  // eating...
```

Ở đây:

- `dog` không có method `eat`
- nhưng prototype của `dog` là `animal`
- nên JS đi lên `animal` để tìm

---

## 4. `__proto__` là gì?

`__proto__` là cách truy cập tới **prototype nội bộ** của một object.

Ví dụ:

```js
const user = { name: "An" }

console.log(user.__proto__)
```

Nó thường được hiểu là:

- object này đang kế thừa từ đâu

Nhưng cần nhớ:

- `__proto__` là cách truy cập mức runtime
- hiện nay thường **không nên lạm dụng** trong code thực tế
- thay vào đó nên dùng:
  - `Object.getPrototypeOf(obj)`
  - `Object.setPrototypeOf(obj, proto)`

Ví dụ:

```js
const user = { name: "An" }

console.log(Object.getPrototypeOf(user) === user.__proto__) // true
```

---

## 5. `prototype` khác `__proto__` như thế nào?

Đây là chỗ cực kỳ hay nhầm.

### 5.1. `prototype`
`prototype` là thuộc tính của **function constructor**.

Ví dụ:

```js
function Person() {}
console.log(Person.prototype)
```

### 5.2. `__proto__`
`__proto__` là thuộc tính truy cập prototype của **một object cụ thể**.

Ví dụ:

```js
const p = new Person()
console.log(p.__proto__ === Person.prototype) // true
```

### 5.3. Câu nhớ nhanh

```text
prototype   -> nằm trên function constructor
__proto__   -> nằm trên object instance
```

### 5.4. Ví dụ dễ hiểu

```js
function Person(name) {
  this.name = name
}

Person.prototype.sayHi = function () {
  console.log("Hi, tôi là " + this.name)
}

const p1 = new Person("An")

console.log(p1.__proto__ === Person.prototype) // true
```

Giải thích:

- `Person.prototype` là nơi chứa method dùng chung
- `p1.__proto__` trỏ tới `Person.prototype`
- nên `p1` gọi được `sayHi()`

---

## 6. Constructor function là gì?

Trước khi `class` xuất hiện, JavaScript thường tạo object bằng **constructor function**.

Ví dụ:

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

const p1 = new Person("An", 20)
const p2 = new Person("Bình", 22)

console.log(p1.name) // An
console.log(p2.age)  // 22
```

### 6.1. `new` làm gì?

Khi gọi:

```js
new Person("An", 20)
```

JS làm gần như các bước sau:

1. tạo object rỗng mới
2. gán prototype của object đó thành `Person.prototype`
3. bind `this` trong `Person` vào object mới
4. chạy function `Person`
5. trả về object mới

Hình dung:

```text
new Person()
=> tạo object mới
=> object.__proto__ = Person.prototype
=> this trỏ vào object mới
=> return object
```

---

## 7. Vì sao không nên khai báo method trực tiếp trong constructor?

Ví dụ:

```js
function Person(name) {
  this.name = name
  this.sayHi = function () {
    console.log("Hi, tôi là " + this.name)
  }
}
```

Cách này chạy được, nhưng có vấn đề:

- mỗi lần tạo instance mới, JS lại tạo **một function mới**
- tốn bộ nhớ hơn
- không tối ưu

Ví dụ:

```js
const p1 = new Person("An")
const p2 = new Person("Bình")

console.log(p1.sayHi === p2.sayHi) // false
```

Vì 2 function khác nhau.

### Cách tốt hơn

Đưa method lên `prototype`:

```js
function Person(name) {
  this.name = name
}

Person.prototype.sayHi = function () {
  console.log("Hi, tôi là " + this.name)
}

const p1 = new Person("An")
const p2 = new Person("Bình")

console.log(p1.sayHi === p2.sayHi) // true
```

Vì cả hai dùng chung một method.

---

## 8. Prototype chain là gì?

Prototype chain là chuỗi liên kết prototype mà JS dùng để tra cứu thuộc tính.

Ví dụ:

```js
const grandParent = {
  speak() {
    console.log("grand parent speaks")
  }
}

const parent = {
  walk() {
    console.log("parent walks")
  }
}

const child = {
  run() {
    console.log("child runs")
  }
}

Object.setPrototypeOf(parent, grandParent)
Object.setPrototypeOf(child, parent)

child.run()   // child runs
child.walk()  // parent walks
child.speak() // grand parent speaks
```

Tra cứu diễn ra như sau:

```text
child.run   -> có ngay trên child
child.walk  -> không có trên child, tìm trên parent
child.speak -> không có trên child, không có trên parent, tìm trên grandParent
```

Nếu không tìm thấy ở đâu:

```js
console.log(child.fly) // undefined
```

---

## 9. Property shadowing là gì?

Là khi object con có thuộc tính cùng tên với thuộc tính ở prototype, thì thuộc tính ở object con sẽ “che” thuộc tính phía trên.

Ví dụ:

```js
const animal = {
  sound: "some sound"
}

const dog = {
  sound: "gâu gâu"
}

Object.setPrototypeOf(dog, animal)

console.log(dog.sound) // gâu gâu
```

JS sẽ lấy `dog.sound` trước, không cần đi lên `animal.sound` nữa.

---

## 10. `hasOwnProperty` dùng để làm gì?

Dùng để kiểm tra một thuộc tính có nằm **trực tiếp trên object** hay không.

Ví dụ:

```js
const animal = { eat() {} }
const dog = { bark() {} }
Object.setPrototypeOf(dog, animal)

console.log(dog.hasOwnProperty("bark")) // true
console.log(dog.hasOwnProperty("eat"))  // false
```

Vì:

- `bark` nằm trực tiếp trên `dog`
- `eat` nằm ở prototype

Đây là công cụ rất hữu ích khi duyệt object.

---

## 11. `class` trong JavaScript là gì?

`class` là cú pháp đẹp hơn để làm việc với constructor function và prototype.

Nó không tạo ra cơ chế kế thừa hoàn toàn mới.

Ví dụ:

```js
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
  }

  sayHi() {
    console.log(`Hi, tôi là ${this.name}`)
  }
}

const p1 = new Person("An", 20)
p1.sayHi()
```

Bản chất gần giống:

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

Person.prototype.sayHi = function () {
  console.log("Hi, tôi là " + this.name)
}
```

### Câu nhớ nhanh

```text
class trong JS = cú pháp dễ đọc hơn cho constructor + prototype
```

---

## 12. Constructor trong class là gì?

Trong `class`, hàm `constructor()` là nơi khởi tạo object mới.

Ví dụ:

```js
class User {
  constructor(name) {
    this.name = name
  }
}

const u = new User("An")
console.log(u.name) // An
```

Tương tự constructor function cũ:

```js
function User(name) {
  this.name = name
}
```

---

## 13. Method trong class được lưu ở đâu?

Method khai báo trong class **không nằm trực tiếp trên instance**, mà nằm trên `prototype`.

Ví dụ:

```js
class Person {
  constructor(name) {
    this.name = name
  }

  sayHi() {
    console.log("Hi")
  }
}

const p1 = new Person("An")

console.log(p1.hasOwnProperty("name"))  // true
console.log(p1.hasOwnProperty("sayHi")) // false
```

Vì:

- `name` là property của instance
- `sayHi` nằm trên `Person.prototype`

Kiểm tra:

```js
console.log(p1.__proto__ === Person.prototype) // true
```

---

## 14. `extends` là gì?

`extends` dùng để tạo class con kế thừa từ class cha.

Ví dụ:

```js
class Animal {
  eat() {
    console.log("eating...")
  }
}

class Dog extends Animal {
  bark() {
    console.log("gâu gâu")
  }
}

const d = new Dog()
d.bark() // gâu gâu
d.eat()  // eating...
```

Ở đây:

- `Dog` kế thừa từ `Animal`
- instance của `Dog` dùng được cả method riêng và method cha

---

## 15. `super` là gì?

`super` dùng để gọi logic từ class cha.

Có 2 cách gặp phổ biến:

### 15.1. Gọi constructor của cha

```js
class Animal {
  constructor(name) {
    this.name = name
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name)
    this.breed = breed
  }
}

const d = new Dog("Milu", "Poodle")
console.log(d.name)  // Milu
console.log(d.breed) // Poodle
```

### 15.2. Gọi method của cha

```js
class Animal {
  speak() {
    console.log("animal speaks")
  }
}

class Dog extends Animal {
  speak() {
    super.speak()
    console.log("dog barks")
  }
}

const d = new Dog()
d.speak()
```

Kết quả:

```text
animal speaks
dog barks
```

---

## 16. Vì sao trong class con phải gọi `super()` trước khi dùng `this`?

Trong class kế thừa, nếu có `constructor`, bạn phải gọi `super()` trước khi dùng `this`.

Sai:

```js
class Animal {
  constructor(name) {
    this.name = name
  }
}

class Dog extends Animal {
  constructor(name) {
    this.type = "dog" // lỗi
    super(name)
  }
}
```

Đúng:

```js
class Dog extends Animal {
  constructor(name) {
    super(name)
    this.type = "dog"
  }
}
```

Vì:

- `super()` dùng để khởi tạo phần cha trước
- sau đó `this` mới sẵn sàng để dùng

---

## 17. Static method là gì?

Static method là method gắn vào chính class, không gắn vào instance.

Ví dụ:

```js
class MathHelper {
  static add(a, b) {
    return a + b
  }
}

console.log(MathHelper.add(2, 3)) // 5
```

Sai nếu gọi qua instance:

```js
const m = new MathHelper()
// m.add(2, 3) // lỗi
```

### Cách nhớ

```text
static method -> gọi bằng ClassName.method()
instance method -> gọi bằng instance.method()
```

---

## 18. Getter và Setter là gì?

JavaScript class hỗ trợ getter/setter.

Ví dụ:

```js
class User {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }

  get fullName() {
    return `${this.firstName} ${this.lastName}`
  }

  set fullName(value) {
    const [firstName, lastName] = value.split(" ")
    this.firstName = firstName
    this.lastName = lastName
  }
}

const u = new User("Nguyen", "An")
console.log(u.fullName) // Nguyen An

u.fullName = "Tran Binh"
console.log(u.firstName) // Tran
console.log(u.lastName)  // Binh
```

Điểm hay là dùng như property thường:

```js
u.fullName
u.fullName = "Tran Binh"
```

chứ không cần gọi như function.

---

## 19. Class expression là gì?

Ngoài class declaration, còn có class expression.

Ví dụ:

```js
const Person = class {
  constructor(name) {
    this.name = name
  }
}

const p = new Person("An")
```

Giống như function, class cũng có thể được gán vào biến.

---

## 20. Hoisting của class

Class có hoisting theo nghĩa JS biết tới nó trong scope, nhưng không dùng được trước khai báo do TDZ.

Ví dụ:

```js
const p = new Person("An") // lỗi

class Person {
  constructor(name) {
    this.name = name
  }
}
```

Điều này khá giống `let` và `const`.

---

## 21. So sánh constructor function và class

### Constructor function

```js
function Person(name) {
  this.name = name
}

Person.prototype.sayHi = function () {
  console.log("Hi")
}
```

### Class

```js
class Person {
  constructor(name) {
    this.name = name
  }

  sayHi() {
    console.log("Hi")
  }
}
```

### Bản chất

- đều tạo instance bằng `new`
- đều dùng prototype để chia sẻ method
- `class` chỉ dễ đọc hơn

---

## 22. Kiểm tra quan hệ kế thừa

### 22.1. `instanceof`

Dùng để kiểm tra một object có nằm trong chuỗi prototype của constructor nào không.

Ví dụ:

```js
class Animal {}
class Dog extends Animal {}

const d = new Dog()

console.log(d instanceof Dog)    // true
console.log(d instanceof Animal) // true
console.log(d instanceof Object) // true
```

### 22.2. `isPrototypeOf`

```js
console.log(Animal.prototype.isPrototypeOf(d)) // true
console.log(Dog.prototype.isPrototypeOf(d))    // true
```

---

## 23. Các điểm rất hay nhầm

### 23.1. `prototype` không phải của mọi object
Nhiều người nghĩ object nào cũng có `.prototype`.

Sai.

```js
const user = {}
console.log(user.prototype) // undefined
```

Vì `.prototype` thường nằm trên function constructor.

---

### 23.2. `__proto__` và `prototype` không giống nhau

```text
Person.prototype  -> prototype dùng cho instance tạo từ Person
p1.__proto__      -> prototype thực tế của object p1
```

Thường:

```js
p1.__proto__ === Person.prototype // true
```

---

### 23.3. `class` không thay thế prototype
`class` không xóa bỏ prototype.
Nó chỉ bọc cú pháp prototype lại cho dễ dùng hơn.

---

### 23.4. Method trong class không copy vào từng instance
Method vẫn nằm trên prototype, không nằm trực tiếp trên từng object instance.

---

### 23.5. Arrow function trong method class có hành vi `this` khác
Nếu dùng arrow function ở field instance, nó không giống method prototype thông thường.
Đây là phần nâng cao nhưng rất hay gặp trong React hoặc code hiện đại.

Ví dụ:

```js
class Counter {
  count = 0

  increase = () => {
    this.count++
  }
}
```

Cách này tạo function trên từng instance, không nằm trên prototype như method class chuẩn.

---

## 24. Ví dụ tổng hợp toàn bộ

```js
class Animal {
  constructor(name) {
    this.name = name
  }

  eat() {
    console.log(`${this.name} is eating`)
  }

  speak() {
    console.log(`${this.name} makes a sound`)
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name)
    this.breed = breed
  }

  speak() {
    console.log(`${this.name} barks`)
  }

  get info() {
    return `${this.name} - ${this.breed}`
  }

  static isDog(obj) {
    return obj instanceof Dog
  }
}

const d = new Dog("Milu", "Poodle")

d.eat()              // Milu is eating
d.speak()            // Milu barks
console.log(d.info)  // Milu - Poodle
console.log(Dog.isDog(d)) // true
```

Bạn có thể đọc ví dụ này để thấy cùng lúc:

- `extends`
- `super`
- override method
- getter
- static method
- inheritance
- instance check

---

## 25. Tóm tắt nhanh toàn phase

### 25.1. Các ý chính

- JavaScript kế thừa qua **prototype**
- object tìm thuộc tính theo **prototype chain**
- `prototype` nằm trên function constructor
- `__proto__` là prototype thực tế của object instance
- `class` chỉ là cú pháp đẹp hơn cho constructor + prototype
- `extends` dùng để kế thừa
- `super` dùng để gọi cha
- method trong class nằm trên prototype

### 25.2. Bảng nhớ nhanh

```text
prototype   -> thuộc về function constructor
__proto__   -> thuộc về object instance
class       -> syntactic sugar
extends     -> kế thừa class cha
super       -> gọi constructor/method cha
instanceof  -> kiểm tra quan hệ prototype chain
```

---

## 26. Mini checklist tự kiểm tra

Bạn nên tự trả lời được các câu này:

1. Prototype là gì?
2. `prototype` và `__proto__` khác nhau ra sao?
3. `new` làm những gì?
4. Vì sao nên đưa method lên `prototype`?
5. Prototype chain hoạt động như thế nào?
6. `class` có phải cơ chế mới hoàn toàn không?
7. `extends` và `super` dùng để làm gì?
8. Vì sao `sayHi()` trong class thường không nằm trực tiếp trên instance?
9. `instanceof` kiểm tra cái gì?
10. Vì sao `typeof class Something {}` vẫn là `function`?

---

## 27. Kết luận phase 5

Nếu Phase 1 giúp bạn hiểu dữ liệu, Phase 2 giúp bạn hiểu biến và scope, Phase 3 giúp bạn hiểu function và closure, Phase 4 giúp bạn thao tác dữ liệu, thì **Phase 5 giúp bạn hiểu bản chất hướng đối tượng của JavaScript**.

Sau phase này, bạn sẽ:

- đọc code `class` dễ hơn
- hiểu constructor function cũ
- đỡ nhầm giữa `prototype` và `__proto__`
- hiểu vì sao object vẫn truy cập được method không nằm trực tiếp trên nó

Đây là một phase rất quan trọng để bạn chuyển từ mức “biết dùng JS” sang mức “bắt đầu hiểu sâu JS”.
