# PHẦN 2: NODE.JS CORE CONCEPTS - GIẢI THÍCH CHI TIẾT

> **Tại sao phần này quan trọng?**
> Node.js là runtime environment. Hiểu cách Node.js hoạt động bên dưới giúp bạn viết code
> hiệu quả hơn, debug dễ hơn, và trả lời được câu hỏi "tại sao" chứ không chỉ "cách nào".

---

## 1. KIẾN TRÚC NODE.JS

### Node.js = V8 Engine + libuv + Core Modules

```
┌─────────────────────────────────────────────┐
│               YOUR APPLICATION               │
│          (JavaScript / TypeScript)            │
├─────────────────────────────────────────────┤
│              NODE.JS BINDINGS                │
│         (C++ layer kết nối JS ↔ C)          │
├───────────────────┬─────────────────────────┤
│    V8 ENGINE      │       LIBUV             │
│  (Google Chrome)  │  (Async I/O Library)    │
│                   │                         │
│  - Parse JS       │  - Event Loop           │
│  - Compile to     │  - Thread Pool (4)      │
│    machine code   │  - Async TCP/UDP        │
│  - Execute        │  - File System ops      │
│  - Garbage        │  - DNS                  │
│    Collection     │  - Signal handling      │
├───────────────────┴─────────────────────────┤
│            OPERATING SYSTEM                  │
│     (Linux, macOS, Windows)                  │
└─────────────────────────────────────────────┘
```

### V8 Engine:
- Biên dịch JavaScript thành machine code (không interpret)
- JIT (Just-In-Time) compilation
- Garbage Collection (Mark-and-Sweep, Generational GC)
- Memory: Heap (objects) + Stack (primitives, references)

### libuv:
- Thư viện C cung cấp async I/O
- Quản lý Event Loop
- **Thread Pool** (mặc định 4 threads) cho các tác vụ blocking:
  - File system operations (fs)
  - DNS lookups
  - Crypto operations
  - Compression (zlib)

### Tại sao Node.js nhanh dù single-threaded?

```
Traditional Server (Multi-threaded):
Request 1 → Thread 1 → [DB query 100ms] → Response
Request 2 → Thread 2 → [DB query 100ms] → Response
Request 3 → Thread 3 → [DB query 100ms] → Response
→ Mỗi request = 1 thread, 10000 requests = 10000 threads = TỐN RAM

Node.js (Single-threaded + Event Loop):
Request 1 → Main Thread → delegate to I/O → [không chờ, xử lý tiếp]
Request 2 → Main Thread → delegate to I/O → [không chờ, xử lý tiếp]
Request 3 → Main Thread → delegate to I/O → [không chờ, xử lý tiếp]
                                    ↓
                     I/O hoàn thành → Callback → Main Thread xử lý
→ 1 thread xử lý TẤT CẢ, nhờ non-blocking I/O
```

**Khi nào Node.js KHÔNG phù hợp?**
- CPU-intensive tasks (image processing, video encoding, heavy computation)
- Vì main thread bị block → tất cả requests bị chờ

**Giải pháp cho CPU-intensive:**
- Worker Threads
- Child Processes
- Message Queue (offload sang service khác)

---

## 2. SINGLE-THREADED, NHƯNG CONCURRENT

### Worker Threads vs Child Processes vs Cluster

```
┌──────────────────────────────────────────────────────────┐
│                    MAIN PROCESS                           │
│                  (Single Thread)                          │
│                                                          │
│  Xử lý: HTTP requests, business logic, routing          │
│  KHÔNG nên: heavy computation, blocking operations       │
└──────────────────────────────────────────────────────────┘

Khi cần parallelism:

1. WORKER THREADS (node:worker_threads)
   ┌──────────┐  ┌──────────┐  ┌──────────┐
   │ Worker 1 │  │ Worker 2 │  │ Worker 3 │
   │ (thread) │  │ (thread) │  │ (thread) │
   └──────────┘  └──────────┘  └──────────┘
   - Chia sẻ MEMORY (SharedArrayBuffer)
   - Nhẹ hơn child process
   - Use case: CPU-intensive tasks (crypto, parsing)

2. CHILD PROCESSES (node:child_process)
   ┌──────────┐  ┌──────────┐  ┌──────────┐
   │ Process 1│  │ Process 2│  │ Process 3│
   │ (riêng)  │  │ (riêng)  │  │ (riêng)  │
   └──────────┘  └──────────┘  └──────────┘
   - Mỗi process có MEMORY RIÊNG
   - Giao tiếp qua IPC (Inter-Process Communication)
   - Use case: chạy script khác, shell commands

3. CLUSTER MODULE (node:cluster)
   ┌──────────┐
   │  Master  │
   │ Process  │
   └────┬─────┘
        ├── Worker Process 1 (port 3000)
        ├── Worker Process 2 (port 3000)
        ├── Worker Process 3 (port 3000)
        └── Worker Process 4 (port 3000)
   - Fork multiple processes lắng nghe CÙNG port
   - Load balancing tự động (round-robin)
   - Use case: tận dụng multi-core CPU cho HTTP server
```

### Worker Threads Example:

```javascript
// main.js
const { Worker } = require('worker_threads');

function runHeavyTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}

// Không block main thread
app.get('/heavy', async (req, res) => {
  const result = await runHeavyTask({ numbers: [1, 2, 3, 4, 5] });
  res.json(result);
});

// worker.js
const { workerData, parentPort } = require('worker_threads');
// Heavy computation ở đây
const result = workerData.numbers.reduce((sum, n) => sum + fibonacci(n), 0);
parentPort.postMessage(result);
```

### Cluster Example:

```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  console.log(`Master process ${process.pid}`);
  
  // Fork workers = số CPU cores
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, restarting...`);
    cluster.fork();  // Auto-restart
  });
} else {
  // Worker processes chạy HTTP server
  const app = require('./app');
  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started`);
  });
}
```

---

## 3. STREAMS & BUFFERS

### Buffer là gì?

Buffer = vùng nhớ tạm chứa dữ liệu nhị phân (binary data). 
JavaScript ban đầu không xử lý binary data, Buffer sinh ra để giải quyết điều này.

```javascript
// Tạo Buffer
const buf1 = Buffer.from('Hello');           // Từ string
const buf2 = Buffer.alloc(10);               // 10 bytes, fill 0
const buf3 = Buffer.from([72, 101, 108]);    // Từ byte array

console.log(buf1);           // <Buffer 48 65 6c 6c 6f>
console.log(buf1.toString()); // 'Hello'
console.log(buf1.length);    // 5 (bytes, KHÔNG phải characters)
```

### Streams là gì?

Stream = cách xử lý dữ liệu **từng phần (chunk)** thay vì load TẤT CẢ vào memory.

**Tại sao cần Streams?**
```javascript
// ❌ KHÔNG dùng stream: Load toàn bộ file 2GB vào RAM
const data = fs.readFileSync('huge-file.csv'); // 2GB vào RAM → CRASH!

// ✅ Dùng stream: Đọc từng chunk nhỏ
const stream = fs.createReadStream('huge-file.csv');
stream.on('data', (chunk) => {
  // Mỗi chunk chỉ 64KB → RAM luôn ổn định
  processChunk(chunk);
});
```

### 4 loại Stream:

```
1. Readable Stream  → Nguồn dữ liệu (đọc)
   - fs.createReadStream()
   - http.IncomingMessage (request)
   - process.stdin

2. Writable Stream  → Đích dữ liệu (ghi)
   - fs.createWriteStream()
   - http.ServerResponse (response)
   - process.stdout

3. Duplex Stream    → Vừa đọc vừa ghi (2 chiều)
   - net.Socket (TCP)
   - WebSocket

4. Transform Stream → Biến đổi dữ liệu khi đi qua
   - zlib.createGzip() (nén)
   - crypto.createCipher()
```

### Pipe Pattern (quan trọng):

```javascript
// Pipe = kết nối output của stream này vào input của stream kia
// Giống ống nước: nước chảy từ nguồn → qua ống → đến đích

const fs = require('fs');
const zlib = require('zlib');

// Đọc file → Nén → Ghi ra file mới
fs.createReadStream('input.txt')        // Readable
  .pipe(zlib.createGzip())              // Transform (nén)
  .pipe(fs.createWriteStream('input.txt.gz'))  // Writable
  .on('finish', () => console.log('Done!'));

// HTTP response streaming
app.get('/video', (req, res) => {
  const stream = fs.createReadStream('video.mp4');
  stream.pipe(res);  // Stream video trực tiếp, không load vào RAM
});

// Upload file lớn
app.post('/upload', (req, res) => {
  const writeStream = fs.createWriteStream('uploaded-file');
  req.pipe(writeStream);  // Request body → file
  req.on('end', () => res.json({ message: 'Uploaded' }));
});
```

### Backpressure:

```
Khi Readable stream tạo data NHANH hơn Writable stream xử lý:

Readable (nhanh) ──data──data──data──> Writable (chậm)
                                        ↑
                                   Buffer đầy!
                                   
→ Backpressure: Readable tạm DỪNG gửi data cho đến khi Writable xử lý xong.
→ pipe() tự động xử lý backpressure cho bạn.
```

```javascript
// ❌ Không xử lý backpressure → memory leak
readable.on('data', (chunk) => {
  writable.write(chunk);  // Nếu writable chậm, buffer sẽ đầy
});

// ✅ Xử lý backpressure manually
readable.on('data', (chunk) => {
  const canContinue = writable.write(chunk);
  if (!canContinue) {
    readable.pause();  // Tạm dừng đọc
    writable.once('drain', () => readable.resume());  // Tiếp tục khi writable sẵn sàng
  }
});

// ✅ Hoặc đơn giản dùng pipe()
readable.pipe(writable);  // Tự động xử lý backpressure
```

---

## 4. ERROR HANDLING TRONG NODE.JS

### Phân loại errors:

```
1. Operational Errors (lỗi vận hành - EXPECTED)
   - Database connection failed
   - Request timeout
   - File not found
   - Invalid user input
   → XỬ LÝ: try/catch, error responses

2. Programmer Errors (lỗi lập trình - BUGS)
   - TypeError: Cannot read property of undefined
   - ReferenceError: variable is not defined
   - Passing wrong type to function
   → XỬ LÝ: Fix code, không nên catch

3. Unhandled Errors (nguy hiểm nhất)
   - Unhandled Promise Rejection
   - Uncaught Exception
   → XỬ LÝ: Global handlers + graceful shutdown
```

### Patterns xử lý error:

```javascript
// 1. Try/Catch cho async/await
async function getUser(id) {
  try {
    const user = await userRepository.findOne(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`);
    }
    return user;
  } catch (error) {
    if (error instanceof NotFoundException) {
      throw error;  // Re-throw expected errors
    }
    // Log unexpected errors
    logger.error('Unexpected error in getUser:', error);
    throw new InternalServerErrorException('Something went wrong');
  }
}

// 2. Error-first Callback Pattern (Node.js truyền thống)
fs.readFile('file.txt', (err, data) => {
  if (err) {
    console.error('Error:', err);
    return;
  }
  console.log(data);
});

// 3. Custom Error Classes
class AppError extends Error {
  constructor(message, statusCode, errorCode) {
    super(message);
    this.statusCode = statusCode;
    this.errorCode = errorCode;
    this.isOperational = true;  // Đánh dấu là operational error
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with ID ${id} not found`, 404, 'NOT_FOUND');
  }
}
```

### Graceful Shutdown:

```javascript
// Khi app gặp lỗi nghiêm trọng hoặc cần tắt, phải shutdown "đẹp":
// 1. Ngừng nhận requests mới
// 2. Hoàn thành requests đang xử lý
// 3. Đóng database connections
// 4. Thoát process

const server = app.listen(3000);

async function gracefulShutdown(signal) {
  console.log(`Received ${signal}. Starting graceful shutdown...`);
  
  // 1. Ngừng nhận connections mới
  server.close(() => {
    console.log('HTTP server closed');
  });
  
  // 2. Đóng database
  await database.disconnect();
  console.log('Database disconnected');
  
  // 3. Đóng Redis
  await redis.quit();
  console.log('Redis disconnected');
  
  // 4. Exit
  process.exit(0);
}

// Lắng nghe signals
process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));  // Docker stop
process.on('SIGINT', () => gracefulShutdown('SIGINT'));    // Ctrl+C

// Global error handlers
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled Rejection:', reason);
  // Log nhưng KHÔNG crash ngay, để hoàn thành requests đang xử lý
});

process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  // Đây là lỗi nghiêm trọng → PHẢI restart
  gracefulShutdown('uncaughtException');
});
```

---

## 5. PERFORMANCE & MEMORY

### Memory trong Node.js:

```
┌─────────────────────────────────────┐
│              V8 HEAP                │
│  ┌──────────────┬────────────────┐  │
│  │  New Space   │   Old Space    │  │
│  │  (Young Gen) │  (Old Gen)     │  │
│  │              │                │  │
│  │ Short-lived  │ Long-lived     │  │
│  │ objects      │ objects        │  │
│  │              │                │  │
│  │ Scavenge GC  │ Mark-Sweep GC  │  │
│  │ (nhanh,      │ (chậm,         │  │
│  │  thường      │  ít thường     │  │
│  │  xuyên)      │  xuyên)        │  │
│  └──────────────┴────────────────┘  │
└─────────────────────────────────────┘
```

### Memory Leaks phổ biến:

```javascript
// 1. ❌ Global variables tích lũy
const cache = {};  // Global object, không bao giờ bị GC
function processRequest(req) {
  cache[req.id] = req.data;  // Ngày càng to → LEAK
}
// ✅ Fix: Dùng LRU cache hoặc TTL
const LRU = require('lru-cache');
const cache = new LRU({ max: 500, ttl: 1000 * 60 * 5 });

// 2. ❌ Event listeners không remove
class UserService {
  constructor() {
    // Mỗi lần tạo instance → thêm 1 listener → LEAK
    eventEmitter.on('userCreated', this.handleUserCreated);
  }
  // ✅ Fix: Remove listener khi destroy
  destroy() {
    eventEmitter.off('userCreated', this.handleUserCreated);
  }
}

// 3. ❌ Closures giữ reference lớn
function processData() {
  const hugeArray = new Array(1000000).fill('data');
  return function() {
    // Closure giữ reference đến hugeArray dù không dùng
    return 'done';
  };
  // ✅ Fix: Nullify sau khi dùng xong
  // hugeArray = null;
}

// 4. ❌ Setinterval không clear
const interval = setInterval(() => {
  // Chạy mãi nếu không clearInterval
}, 1000);
// ✅ Fix
clearInterval(interval);
```

### `process.nextTick()` vs `setImmediate()`:

```javascript
// process.nextTick() 
// - Chạy NGAY SAU operation hiện tại, TRƯỚC khi Event Loop tiếp tục
// - Ưu tiên cao nhất trong async callbacks
// - ⚠️ Cẩn thận: nếu gọi nextTick đệ quy → block Event Loop

process.nextTick(() => console.log('nextTick'));
// Use case: đảm bảo callback chạy sau khi constructor hoàn thành
// nhưng trước bất kỳ I/O nào

class Database {
  constructor() {
    this.connected = false;
    // Đảm bảo 'connected' event emit SAU khi constructor return
    process.nextTick(() => {
      this.connected = true;
      this.emit('connected');  
    });
  }
}

// setImmediate()
// - Chạy ở check phase của Event Loop (sau poll phase)
// - An toàn hơn nextTick cho recursive calls

setImmediate(() => console.log('setImmediate'));
// Use case: không muốn block I/O events

// So sánh
setImmediate(() => console.log('1. setImmediate'));
setTimeout(() => console.log('2. setTimeout'), 0);
process.nextTick(() => console.log('3. nextTick'));
Promise.resolve().then(() => console.log('4. promise'));

// Output (đảm bảo):
// 3. nextTick
// 4. promise
// 1 hoặc 2 (phụ thuộc timing, thường 2 trước 1)
```

---

## 6. NODE.JS BUILT-IN MODULES QUAN TRỌNG

```javascript
// ═══════ fs (File System) ═══════
const fs = require('fs');
const fsPromises = require('fs/promises');  // Promise-based API

// Sync (blocking - tránh dùng trong production)
const data = fs.readFileSync('file.txt', 'utf8');

// Async callback
fs.readFile('file.txt', 'utf8', (err, data) => { });

// Async Promise (recommended)
const data = await fsPromises.readFile('file.txt', 'utf8');
await fsPromises.writeFile('output.txt', 'content');
await fsPromises.mkdir('new-dir', { recursive: true });

// ═══════ path ═══════
const path = require('path');
path.join('/users', 'alice', 'docs');      // '/users/alice/docs'
path.resolve('src', 'index.ts');            // Absolute path
path.extname('file.txt');                   // '.txt'
path.basename('/users/alice/file.txt');     // 'file.txt'
path.dirname('/users/alice/file.txt');      // '/users/alice'

// ═══════ events ═══════
const EventEmitter = require('events');

class OrderService extends EventEmitter {
  createOrder(order) {
    // Business logic...
    this.emit('orderCreated', order);  // Phát sự kiện
  }
}

const orderService = new OrderService();
orderService.on('orderCreated', (order) => {
  sendConfirmationEmail(order);  // Listener
});
orderService.on('orderCreated', (order) => {
  updateInventory(order);         // Another listener
});

// ═══════ crypto ═══════
const crypto = require('crypto');

// Hash password
const hash = crypto.createHash('sha256').update('password').digest('hex');

// Random bytes (for tokens)
const token = crypto.randomBytes(32).toString('hex');
```

---

## TÓM TẮT PHẦN 2 - NODE.JS CORE

### Top câu hỏi phỏng vấn:
1. **"Node.js là single-threaded nhưng xử lý concurrent thế nào?"**
   → Event Loop + non-blocking I/O + libuv thread pool

2. **"Khi nào Node.js không phù hợp?"**
   → CPU-intensive tasks. Giải pháp: Worker Threads, microservices

3. **"Stream dùng khi nào?"**
   → File lớn, video streaming, real-time data processing

4. **"process.nextTick() vs setImmediate()?"**
   → nextTick: trước Event Loop iteration. setImmediate: check phase

5. **"Cách xử lý memory leak?"**
   → Monitor heap, tránh global caching, remove event listeners, profiling

6. **"Cluster module dùng để làm gì?"**
   → Tận dụng multi-core CPU, mỗi core chạy 1 worker process

### Kinh nghiệm thực tế nên chia sẻ:
- "Trong project, tôi dùng streams để xử lý upload file lớn (CSV import)"
- "Tôi dùng cluster/PM2 để tận dụng multi-core trong production"
- "Tôi implement graceful shutdown để không mất data khi deploy"
