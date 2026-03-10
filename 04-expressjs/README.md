# PHẦN 4: EXPRESS.JS - GIẢI THÍCH CHI TIẾT

> **Mức độ ưu tiên: TRUNG BÌNH**
> Express xuất hiện trong JD nhưng không phải trọng tâm. NestJS dùng Express bên dưới,
> nên hiểu Express giúp bạn hiểu NestJS tốt hơn. Interviewer có thể hỏi so sánh.

---

## 1. EXPRESS LÀ GÌ?

Express = **Minimal, unopinionated web framework** cho Node.js.
- "Minimal": Chỉ cung cấp core features (routing, middleware)
- "Unopinionated": Không ép bạn theo cấu trúc nào, tự quyết hết

```javascript
const express = require('express');
const app = express();

// Middleware
app.use(express.json());  // Parse JSON body

// Route
app.get('/api/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

// Start server
app.listen(3000, () => console.log('Server running on port 3000'));
```

---

## 2. MIDDLEWARE PATTERN (CỐT LÕI CỦA EXPRESS)

### Middleware là gì?

Middleware = function có access đến `request`, `response`, và `next`.
Mọi thứ trong Express đều là middleware. Requests đi qua một CHUỖI middleware.

```
Request → Middleware 1 → Middleware 2 → Middleware 3 → Route Handler → Response
              ↓              ↓              ↓
          (logging)      (auth check)   (parse body)
```

### Cách middleware hoạt động:

```javascript
// Mỗi middleware PHẢI gọi next() hoặc gửi response
// Nếu không → request bị "treo" mãi mãi

// Middleware chạy TUẦN TỰ theo thứ tự khai báo
app.use((req, res, next) => {
  console.log('Middleware 1: START');
  next();  // Chuyển sang middleware tiếp theo
  console.log('Middleware 1: END');  // Chạy sau khi các middleware tiếp theo hoàn thành
});

app.use((req, res, next) => {
  console.log('Middleware 2');
  next();
});

app.get('/test', (req, res) => {
  console.log('Handler');
  res.send('OK');
});

// Request GET /test:
// Middleware 1: START
// Middleware 2
// Handler
// Middleware 1: END
```

### Các loại middleware:

```javascript
// 1. Application-level middleware
app.use(express.json());                // Mọi route
app.use('/api', authMiddleware);        // Chỉ routes bắt đầu /api

// 2. Router-level middleware
const router = express.Router();
router.use(authMiddleware);             // Chỉ routes trong router này
router.get('/users', getUsers);

// 3. Error-handling middleware (4 tham số)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

// 4. Third-party middleware
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

app.use(cors());                // CORS headers
app.use(helmet());              // Security headers
app.use(morgan('combined'));    // HTTP logging
```

### Error Handling Pattern:

```javascript
// Express error handling là QUAN TRỌNG và hay hỏi phỏng vấn

// Synchronous errors: Express tự bắt
app.get('/sync-error', (req, res) => {
  throw new Error('Sync error');  // Express bắt tự động
});

// Asynchronous errors: PHẢI tự pass vào next()
app.get('/async-error', async (req, res, next) => {
  try {
    const user = await findUser(req.params.id);
    res.json(user);
  } catch (err) {
    next(err);  // Pass error tới error handler
  }
});

// Wrapper để không phải viết try/catch mỗi route
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await findUser(req.params.id);
  if (!user) throw new NotFoundError('User not found');
  res.json(user);
}));

// Error handler (PHẢI đặt SAU tất cả routes)
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    success: false,
    error: {
      message: err.message,
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
    }
  });
});
```

---

## 3. ROUTING

```javascript
const express = require('express');
const router = express.Router();

// CRUD routes
router.get('/users',          getUsers);      // GET    /api/users
router.get('/users/:id',     getUser);       // GET    /api/users/123
router.post('/users',         createUser);    // POST   /api/users
router.put('/users/:id',     updateUser);    // PUT    /api/users/123
router.patch('/users/:id',   patchUser);     // PATCH  /api/users/123
router.delete('/users/:id',  deleteUser);    // DELETE /api/users/123

// Route parameters
router.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;  // { userId: '123', postId: '456' }
});

// Query strings
router.get('/users', (req, res) => {
  const { page, limit, sort } = req.query;  // ?page=1&limit=10&sort=name
});

// Mount router
app.use('/api', router);  // Prefix: /api/users, /api/users/:id
```

---

## 4. EXPRESS VS NESTJS - SO SÁNH CHI TIẾT

| Tiêu chí | Express | NestJS |
|-----------|---------|--------|
| **Kiến trúc** | Không có cấu trúc bắt buộc | Module-based, có cấu trúc rõ ràng |
| **Language** | JavaScript (TypeScript optional) | TypeScript (native) |
| **DI** | Không có | Built-in IoC Container |
| **Validation** | Tự cài (express-validator, Joi) | Built-in (class-validator) |
| **Swagger** | Tự cài (swagger-jsdoc) | Built-in (@nestjs/swagger) |
| **Testing** | Tự setup | Built-in testing utilities |
| **Microservices** | Tự implement | Built-in support |
| **WebSocket** | Tự cài (socket.io) | Built-in (@nestjs/websockets) |
| **Learning Curve** | Thấp | Trung bình-Cao |
| **Boilerplate** | Ít | Nhiều hơn |
| **Cộng đồng** | Rất lớn, mature | Đang phát triển nhanh |
| **Performance** | Nhanh hơn một chút | Overhead nhỏ từ abstraction layer |
| **Phù hợp** | Prototyping, API nhỏ-vừa | Enterprise, dự án lớn, team lớn |

### Khi nào chọn Express?
- Prototype nhanh, MVP
- API đơn giản, ít logic
- Team nhỏ, quen Express
- Cần performance tối đa

### Khi nào chọn NestJS?
- Dự án enterprise, long-term
- Team lớn, cần convention thống nhất
- Cần microservices, WebSocket, GraphQL
- Cần testing dễ dàng (DI)
- TypeScript-first

---

## TÓM TẮT PHẦN 4

### Câu hỏi phỏng vấn:

**1. "Middleware trong Express hoạt động thế nào?"**
→ Middleware là function(req, res, next). Requests đi qua chuỗi middleware tuần tự.
Mỗi middleware phải gọi next() hoặc gửi response. Giống pipeline pattern.

**2. "Express error handling?"**
→ Async errors cần try/catch + next(err). Error handler middleware có 4 tham số 
(err, req, res, next) và đặt SAU tất cả routes.

**3. "So sánh Express vs NestJS?"**
→ Express: minimal, linh hoạt, learning curve thấp, phù hợp project nhỏ.
NestJS: structured, DI, TypeScript-first, phù hợp enterprise.
NestJS dùng Express (hoặc Fastify) bên dưới.
