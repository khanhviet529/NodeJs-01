# PHẦN 10: SYSTEM DESIGN & PHÂN TÍCH YÊU CẦU - GIẢI THÍCH CHI TIẾT

> **Mức độ ưu tiên: TRUNG BÌNH-CAO**
> JD yêu cầu "Kỹ năng phân tích yêu cầu và thiết kế hệ thống tốt".
> Đây là câu hỏi phỏng vấn senior-level, nhưng hiểu được fundamentals sẽ ghi điểm lớn.

---

## 1. FRAMEWORK TRẢ LỜI SYSTEM DESIGN

### Template cho mọi câu hỏi System Design:

```
Bước 1: CLARIFY REQUIREMENTS (3-5 phút)
         ↓
Bước 2: ESTIMATE SCALE (2-3 phút)
         ↓
Bước 3: HIGH-LEVEL DESIGN (5-10 phút)
         ↓
Bước 4: DETAILED DESIGN (10-15 phút)
         ↓
Bước 5: BOTTLENECKS & TRADE-OFFS (5 phút)
```

### Bước 1: Clarify Requirements

```
Hỏi interviewer để hiểu rõ scope:

FUNCTIONAL Requirements (Hệ thống làm gì?):
- Users có thể đăng ký, đăng nhập?
- Users tạo/đọc/sửa/xóa dữ liệu?
- Real-time hay async?
- Search functionality?

NON-FUNCTIONAL Requirements (Hệ thống hoạt động thế nào?):
- Bao nhiêu users? (100? 1M? 1B?)
- Availability vs Consistency? (CAP theorem)
- Latency requirement? (<100ms? <1s?)
- Data durability? (có thể mất data không?)
- Security requirements?

Ví dụ câu hỏi: "Thiết kế URL Shortener"
→ Hỏi: Bao nhiêu URL/ngày? URL expire không? 
         Custom alias? Analytics? 
         Read:Write ratio?
```

### Bước 2: Estimate Scale

```
Back-of-envelope estimation:

Số liệu cần nhớ:
- 1 ngày = 86,400 giây ≈ 100,000 giây
- 1 tháng ≈ 2.5 triệu giây
- 1 char = 1 byte, 1 KB = 1000 bytes
- 1 MB = 1000 KB, 1 GB = 1000 MB, 1 TB = 1000 GB

Ví dụ URL Shortener:
- 100M URL mới/ngày
- Read:Write = 10:1 → 1B reads/ngày
- Write QPS = 100M / 86400 ≈ 1200/s
- Read QPS = 12,000/s
- Peak QPS = 2x → 24,000/s read
- Storage: 100M * 500 bytes/URL ≈ 50 GB/ngày
- 5 năm ≈ 50 * 365 * 5 ≈ 90 TB
```

---

## 2. SCALABILITY (KHẢ NĂNG MỞ RỘNG)

### Vertical vs Horizontal Scaling:

```
VERTICAL SCALING (Scale Up):
- Nâng cấp server: thêm RAM, CPU, SSD
- Đơn giản, không cần thay đổi code
- Giới hạn: 1 server có max hardware

  Before:         After:
  ┌─────┐         ┌─────────┐
  │ 4GB │         │  64GB   │
  │ 2CPU│    →    │  32CPU  │
  │     │         │         │
  └─────┘         └─────────┘

HORIZONTAL SCALING (Scale Out):
- Thêm NHIỀU servers
- Phức tạp hơn: cần Load Balancer, Session management
- Không giới hạn (thêm bao nhiêu cũng được)
- Fault tolerant: 1 server chết → others vẫn chạy

  Before:         After:
  ┌─────┐         ┌─────┐ ┌─────┐ ┌─────┐
  │ App │         │ App │ │ App │ │ App │
  │     │    →    │  1  │ │  2  │ │  3  │
  └─────┘         └─────┘ └─────┘ └─────┘
                      ↑       ↑       ↑
                  ┌───┴───────┴───────┴───┐
                  │     Load Balancer      │
                  └────────────────────────┘
```

---

## 3. LOAD BALANCER

```
Client requests → Load Balancer → Phân phối đều → Servers

Thuật toán phân tải:
- Round Robin:       Lần lượt Server 1 → 2 → 3 → 1 → 2 → 3
- Weighted RR:       Server mạnh nhận nhiều hơn
- Least Connections: Gửi đến server ÍT connections nhất
- IP Hash:           Cùng IP → cùng server (session sticky)

Ở đâu?
- Layer 4 (Transport): TCP/UDP level, NHANH (HAProxy, AWS NLB)
- Layer 7 (Application): HTTP level, thông minh hơn, route theo URL/header (Nginx, AWS ALB)
```

---

## 4. CACHING STRATEGIES

### Tại sao cần Cache?

```
Không cache: User → App → Database (100ms)
Có cache:    User → App → Redis (1ms) ✅
                        ↘ Database (cache miss)

Cache giảm:
- Database load (ít queries)
- Response time (ms thay vì 100ms)
- Network calls
```

### Caching Patterns:

```
1. CACHE-ASIDE (Lazy Loading):
   ┌──────┐    1.Read     ┌───────┐
   │      │──────────────→│ Cache  │
   │ App  │    2.Miss     │(Redis) │
   │      │←──────────────│        │
   │      │    3.Read DB  └───────┘
   │      │──────────────→┌───────┐
   │      │    4.Response │  DB   │
   │      │←──────────────│       │
   │      │    5.Write    └───────┘
   │      │     Cache
   │      │──────────────→ Cache
   └──────┘

   Pro: Chỉ cache data thực sự cần
   Con: Cache miss = 3 bước (chậm lần đầu)

2. WRITE-THROUGH:
   Write → Cache → DB (đồng thời)
   
   Pro: Cache luôn consistent với DB
   Con: Chậm write (phải write 2 nơi)
   
3. WRITE-BEHIND (Write-Back):
   Write → Cache → (async) → DB
   
   Pro: Write NHANH
   Con: Data loss risk nếu cache crash trước khi sync DB

4. READ-THROUGH:
   Read → Cache (tự fetch DB nếu miss)
   
   Giống Cache-aside nhưng cache library tự manage
```

### Cache Invalidation:

```
Vấn đề: Data trong DB thay đổi → Cache cũ (stale)

Strategies:
1. TTL (Time To Live): Cache tự hết hạn sau X giây
   - Simple, nhưng có thể stale trong TTL window
   
2. Event-based: Khi data đổi → xóa/update cache
   - Consistent hơn, phức tạp hơn
   
3. Cache versioning: Key chứa version number
   - product:v2:123 thay vì product:123

Ví dụ NestJS:
// TTL-based
await this.cacheManager.set('user:123', user, 300); // 5 phút

// Event-based invalidation
async updateUser(id: string, data: UpdateUserDto) {
  const user = await this.userRepo.save({ id, ...data });
  await this.cacheManager.del(`user:${id}`);       // Invalidate
  await this.cacheManager.del('users:list');         // Related cache
  return user;
}
```

---

## 5. MESSAGE QUEUES

### Tại sao cần Message Queue?

```
❌ Synchronous (không queue):
User → API → Send Email → Process Image → Update DB → Response
                ↑              ↑
            3 giây         5 giây
→ User chờ 8+ giây!

✅ Asynchronous (có queue):
User → API → Push to Queue → Response (instant!)
              ↓
         Background workers:
         Worker 1: Send Email (3s, user không chờ)
         Worker 2: Process Image (5s, user không chờ)
```

### Mô hình:

```
PRODUCER → MESSAGE QUEUE → CONSUMER

┌───────┐   enqueue   ┌─────────────────┐   dequeue   ┌──────────┐
│  API  │────────────→ │  [M3][M2][M1]   │────────────→│  Worker  │
│Server │              │  Message Queue   │             │          │
└───────┘              │  (RabbitMQ/Bull) │             └──────────┘
                       └─────────────────┘

Use cases:
- Email/SMS notifications
- Image/video processing
- Report generation
- Order processing
- Webhook delivery
```

### Bull Queue với NestJS:

```typescript
// Cài đặt
// npm install @nestjs/bull bull

// ═══ Module ═══
@Module({
  imports: [
    BullModule.forRoot({
      redis: { host: 'localhost', port: 6379 },
    }),
    BullModule.registerQueue({ name: 'email' }),
  ],
})
export class AppModule {}

// ═══ Producer: Đẩy job vào queue ═══
@Injectable()
export class EmailService {
  constructor(@InjectQueue('email') private emailQueue: Queue) {}

  async sendWelcomeEmail(userId: string) {
    await this.emailQueue.add('welcome', {
      userId,
      template: 'welcome',
    }, {
      attempts: 3,         // Retry 3 lần nếu fail
      backoff: 5000,       // Chờ 5s giữa mỗi retry
      removeOnComplete: true,
    });
    // Return ngay lập tức, không chờ email gửi xong!
  }
}

// ═══ Consumer: Xử lý job ═══
@Processor('email')
export class EmailProcessor {
  @Process('welcome')
  async handleWelcome(job: Job) {
    const { userId, template } = job.data;
    // Gửi email thật sự ở đây
    await this.mailer.send(userId, template);
  }

  @OnQueueFailed()
  onFailed(job: Job, error: Error) {
    console.error(`Job ${job.id} failed:`, error.message);
  }
}
```

---

## 6. DESIGN PATTERNS QUAN TRỌNG

### Repository Pattern:

```typescript
// Tách biệt business logic khỏi data access

// Repository (Data Access Layer)
@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(User)
    private repo: Repository<User>,
  ) {}

  findByEmail(email: string): Promise<User | null> {
    return this.repo.findOne({ where: { email } });
  }

  findActive(): Promise<User[]> {
    return this.repo.find({ where: { isActive: true } });
  }
}

// Service (Business Logic Layer)
@Injectable()
export class UserService {
  constructor(private userRepo: UserRepository) {}

  async register(dto: CreateUserDto) {
    const existing = await this.userRepo.findByEmail(dto.email);
    if (existing) throw new ConflictException('Email exists');
    // Business logic here...
  }
}

// Lợi ích:
// - Service không biết dùng DB gì (có thể swap TypeORM → Prisma)
// - Dễ mock khi test (mock UserRepository)
// - Single Responsibility: Repo = data, Service = logic
```

### Strategy Pattern:

```typescript
// Chọn algorithm/behavior lúc runtime

interface PaymentStrategy {
  pay(amount: number): Promise<PaymentResult>;
}

class StripePayment implements PaymentStrategy {
  async pay(amount: number) { /* Stripe API */ }
}

class PayPalPayment implements PaymentStrategy {
  async pay(amount: number) { /* PayPal API */ }
}

class MoMoPayment implements PaymentStrategy {
  async pay(amount: number) { /* MoMo API */ }
}

// Service dùng strategy
@Injectable()
class PaymentService {
  private strategies: Map<string, PaymentStrategy>;

  constructor(
    private stripe: StripePayment,
    private paypal: PayPalPayment,
    private momo: MoMoPayment,
  ) {
    this.strategies = new Map([
      ['stripe', stripe],
      ['paypal', paypal],
      ['momo', momo],
    ]);
  }

  async processPayment(method: string, amount: number) {
    const strategy = this.strategies.get(method);
    if (!strategy) throw new Error('Unsupported payment method');
    return strategy.pay(amount);
  }
}
```

### Observer Pattern (Event-driven):

```typescript
// NestJS EventEmitter
// npm install @nestjs/event-emitter

// ═══ Emit Event ═══
@Injectable()
class OrderService {
  constructor(private eventEmitter: EventEmitter2) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);
    
    // Emit event → KHÔNG quan tâm ai xử lý
    this.eventEmitter.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
    });
    
    return order;
  }
}

// ═══ Listeners (không phụ thuộc OrderService) ═══
@Injectable()
class EmailListener {
  @OnEvent('order.created')
  async sendConfirmation(payload: OrderCreatedEvent) {
    await this.mailer.send(payload.userId, 'Order confirmed!');
  }
}

@Injectable()
class InventoryListener {
  @OnEvent('order.created')
  async updateStock(payload: OrderCreatedEvent) {
    await this.inventory.decreaseStock(payload.items);
  }
}

@Injectable()
class AnalyticsListener {
  @OnEvent('order.created')
  async trackOrder(payload: OrderCreatedEvent) {
    await this.analytics.track('purchase', payload);
  }
}

// Lợi ích: OrderService simple, thêm behavior = thêm listener
// KHÔNG sửa OrderService
```

---

## 7. ARCHITECTURE PATTERNS

### Monolith vs Microservices:

```
MONOLITH (Khối đơn):
┌─────────────────────────────────────┐
│           SINGLE APPLICATION         │
│                                     │
│  ┌─────┐ ┌──────┐ ┌────────┐       │
│  │Users│ │Orders│ │Products│       │
│  └──┬──┘ └──┬───┘ └───┬────┘       │
│     └────────┼─────────┘            │
│           ┌──┴──┐                   │
│           │ DB  │                   │
│           └─────┘                   │
└─────────────────────────────────────┘

Pro: Đơn giản, dễ develop/test/deploy ban đầu
Con: Scale khó (scale tất cả khi chỉ 1 phần cần), 
     deploy chậm (thay đổi nhỏ = deploy tất cả),
     tech stack locked

MICROSERVICES:
┌────────┐  ┌────────┐  ┌──────────┐
│ User   │  │ Order  │  │ Product  │
│Service │  │Service │  │ Service  │
│  ┌──┐  │  │  ┌──┐  │  │  ┌──┐   │
│  │DB│  │  │  │DB│  │  │  │DB│   │
│  └──┘  │  │  └──┘  │  │  └──┘   │
└───┬────┘  └───┬────┘  └────┬────┘
    │           │             │
    └───────────┴─────────────┘
          Message Queue / API Gateway

Pro: Scale riêng từng service, deploy independently,
     team autonomy, tech diversity
Con: Complex (network latency, distributed transactions, 
     service discovery, monitoring)

→ ĐỀ XUẤT: Start MONOLITH, chuyển sang microservices khi CẦN
   "Premature optimization is the root of all evil"
```

### API Gateway Pattern:

```
Không dùng Gateway:                 Dùng Gateway:
Client biết tất cả services         Client chỉ biết 1 endpoint

Client → User Service               Client → API Gateway → Route
Client → Order Service                                    → Auth
Client → Product Service                                  → Rate limit
                                                          → User Service
                                                          → Order Service
                                                          → Product Service
```

### CQRS (Command Query Responsibility Segregation):

```
Tách READ và WRITE thành 2 models riêng:

TRUYỀN THỐNG:
┌────────┐
│  API   │──→ Same Model ──→ DB
└────────┘    (Read+Write)

CQRS:
                ┌─────────────────┐
                │   COMMAND       │ (Create, Update, Delete)
     Write ────→│   Write Model   │──→ Write DB
                └─────────────────┘
                ┌─────────────────┐
                │   QUERY         │ (Read, Search, Report)
     Read ─────→│   Read Model    │──→ Read DB (optimized)
                └─────────────────┘

Khi nào dùng?
- Read >> Write (e-commerce: users browse 100x, order 1x)
- Read model phức tạp (reports, aggregations)
- Cần scale read/write independently
```

---

## 8. PHÂN TÍCH YÊU CẦU (REQUIREMENTS ANALYSIS)

### Quy trình phân tích:

```
1. Thu thập yêu cầu:
   - Stakeholder interviews
   - User stories: "As a [role], I want [feature], so that [benefit]"
   - Use case diagrams

2. Phân loại:
   FUNCTIONAL:     HỆ THỐNG LÀM GÌ
   - User đăng ký bằng email
   - User tạo order
   - Admin quản lý products

   NON-FUNCTIONAL: HỆ THỐNG HOẠT ĐỘNG THẾ NÀO
   - Performance: Response < 200ms
   - Availability: 99.9% uptime
   - Security: Data encryption at rest & transit
   - Scalability: Support 10K concurrent users

3. Ưu tiên (MoSCoW):
   MUST have:    Đăng ký/Đăng nhập, CRUD cơ bản
   SHOULD have:  Search, Filter, Pagination
   COULD have:   Real-time notifications
   WON'T have:   AI recommendation (future)

4. Validate & Document:
   - API specification (Swagger/OpenAPI)
   - Database schema (ERD)
   - Sequence diagrams (complex flows)
```

---

## 9. PRACTICE: THIẾT KẾ URL SHORTENER

### Bước 1: Requirements

```
Functional:
- Tạo short URL từ long URL
- Redirect short URL → long URL
- Custom alias (optional)
- Set expiration time
- Analytics: click count

Non-functional:
- 100M URLs/ngày (write-heavy)
- 10:1 read/write ratio
- URL length: 7 characters
- Latency < 100ms
- 99.99% availability
```

### Bước 2: Estimation

```
Write: 100M/day ≈ 1200/s
Read: 1B/day ≈ 12,000/s
Storage: 100M × 500 bytes ≈ 50GB/day → 90TB in 5 years
Bandwidth: Read = 12,000 × 500 bytes = 6MB/s
Cache: 20% hot URLs = 0.2 × 1B × 500 bytes ≈ 100GB RAM
```

### Bước 3: High-Level Design

```
                    ┌─────────────────┐
   Browser ────────→│  Load Balancer  │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ↓              ↓              ↓
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ App Sv 1 │  │ App Sv 2 │  │ App Sv 3 │
        └────┬─────┘  └────┬─────┘  └────┬─────┘
             │              │              │
             └──────────────┼──────────────┘
                    ┌───────┴───────┐
                    ↓               ↓
              ┌──────────┐   ┌──────────┐
              │  Redis   │   │ Database │
              │  Cache   │   │ (NoSQL)  │
              └──────────┘   └──────────┘
```

### Bước 4: Detailed Design

```
URL Generation strategies:
1. Base62 encoding (a-z, A-Z, 0-9):
   - 7 chars = 62^7 = 3.5 trillion combinations
   
2. Hash + truncate:
   - MD5(long_url) → lấy 7 chars đầu
   - Collision? → append random char, retry

3. Counter-based:
   - Distributed counter (Zookeeper) → base62 encode
   - No collision, predictable

API:
POST /api/shorten
  Body: { "url": "https://very-long-url.com/path", "alias": "custom" }
  Response: { "shortUrl": "https://short.ly/abc1234" }

GET /:shortCode
  → 301 Redirect to original URL

Database Schema:
{
  id: string,           // short code
  originalUrl: string,
  createdAt: Date,
  expiresAt: Date,
  userId: string,
  clickCount: number,
}
```

---

## 10. PRACTICE: THIẾT KẾ E-COMMERCE

### Simplified Design:

```
┌──────────────────────────────────────────────────┐
│                    API Gateway                     │
│            (Auth, Rate Limit, Routing)             │
└──────────┬───────────┬───────────────┬────────────┘
           ↓           ↓               ↓
    ┌────────────┐ ┌──────────┐ ┌──────────────┐
    │    User    │ │ Product  │ │    Order     │
    │  Service   │ │ Service  │ │   Service    │
    │            │ │          │ │              │
    │  ┌──────┐  │ │┌──────┐ │ │  ┌────────┐  │
    │  │PG DB │  │ ││PG DB │ │ │  │ PG DB  │  │
    │  └──────┘  │ │└──────┘ │ │  └────────┘  │
    └────────────┘ │┌──────┐ │ └──────────────┘
                   ││Redis │ │        │
                   │└──────┘ │        │
                   └──────┘──┘        ↓
                                ┌──────────┐
                                │  Payment │
                                │ Service  │
                                └──────────┘

Các vấn đề cần giải quyết:
1. Product catalog: Caching heavy (Redis), search (Elasticsearch)
2. Inventory: Race condition khi nhiều user mua cùng lúc
   → Optimistic locking hoặc Redis atomic decrement
3. Order: Distributed transaction (Saga pattern)
4. Payment: Idempotency key, webhook callbacks
5. Notifications: Message queue (Background workers)
```

---

## TÓM TẮT PHẦN 10

### Câu hỏi phỏng vấn:

**1. "Vertical vs Horizontal scaling?"**
→ Vertical: nâng cấp 1 server (thêm RAM/CPU). Giới hạn hardware.
Horizontal: thêm nhiều servers. Không giới hạn, nhưng cần LB, session management.

**2. "Bạn chọn Monolith hay Microservices?"**
→ Start Monolith (đơn giản, nhanh ship). Chuyển sang Microservices khi team lớn,
cần scale riêng từng phần, deploy frequently. Không premature optimization.

**3. "Cache invalidation strategies?"**
→ TTL (expire time), Event-based (xóa khi data đổi), Versioning.
TTL đơn giản nhất, event-based consistent nhất.

**4. "Khi nào dùng Message Queue?"**
→ Tách async tasks: email, image processing, reports.
Khi user KHÔNG CẦN chờ kết quả ngay (fire-and-forget).

**5. "CQRS là gì?"**
→ Tách Read/Write thành 2 models. Optimize read cho queries phức tạp.
Dùng khi read >> write (e-commerce, reporting).

**6. "Thiết kế hệ thống X cho tôi"**
→ Dùng framework: Clarify → Estimate → High-level → Detailed → Trade-offs.
LUÔN hỏi requirements trước khi vẽ diagram.

---

## TỔNG KẾT TOÀN BỘ KẾ HOẠCH

```
Phần  Chủ đề                    Mức ưu tiên    File
──────────────────────────────────────────────────────────
1     JavaScript & TypeScript    CAO            01-javascript-typescript/
2     Node.js Core               CAO            02-nodejs-core/
3     NestJS Framework           RẤT CAO        03-nestjs/
4     Express.js                 TRUNG BÌNH     04-expressjs/
5     Databases                  CAO            05-databases/
6     RESTful API & Swagger      CAO            06-restful-api/
7     Auth & Security            CAO            07-auth-security/
8     React & Git                TRUNG BÌNH     08-react-git/
9     Docker & Kubernetes        TRUNG BÌNH     09-docker-kubernetes/
10    System Design              TRUNG BÌNH-CAO 10-system-design/

Thứ tự học ĐỀ XUẤT:
1. NestJS (phần 3) ← Core framework trong JD
2. JavaScript/TypeScript (phần 1)
3. Databases (phần 5)
4. Auth & Security (phần 7)
5. RESTful API (phần 6)
6. Node.js Core (phần 2)
7. System Design (phần 10)
8. Docker & K8s (phần 9)
9. React & Git (phần 8)
10. Express.js (phần 4)
```
