# 🎯 KẾ HOẠCH CHUẨN BỊ PHỎNG VẤN FULLSTACK DEVELOPER (Node.js / React)

> **Mục tiêu:** Nắm vững và hiểu sâu tất cả các yêu cầu trong JD để tự tin trả lời phỏng vấn.
> **Thời gian đề xuất:** 8-10 tuần (có thể điều chỉnh tùy nền tảng hiện tại)

---

## TỔNG QUAN CÁC CHỦ ĐỀ

| # | Chủ đề | Mức độ ưu tiên | Tuần |
|---|--------|----------------|------|
| 1 | TypeScript & JavaScript (ES6+) | 🔴 Cao | 1 |
| 2 | Node.js Core Concepts | 🔴 Cao | 1-2 |
| 3 | NestJS Framework | 🔴 Cao | 2-3 |
| 4 | Express.js | 🟡 Trung bình | 3 |
| 5 | TypeORM & Database (PostgreSQL, MongoDB, Redis) | 🔴 Cao | 4-5 |
| 6 | RESTful API Design & Swagger | 🔴 Cao | 5-6 |
| 7 | Authentication & Security (JWT, OAuth2) | 🔴 Cao | 6 |
| 8 | ReactJS Frontend | 🟡 Trung bình | 7 |
| 9 | Git & Git Flow | 🟡 Trung bình | 7 |
| 10 | Docker & Kubernetes | 🟡 Trung bình | 8 |
| 11 | System Design & Phân tích yêu cầu | 🔴 Cao | 9 |
| 12 | Mock Interview & Tổng ôn | 🔴 Cao | 10 |

---

## TUẦN 1: TYPESCRIPT & JAVASCRIPT (ES6+)

### 1.1 JavaScript ES6+ (Nền tảng)
**Mục tiêu:** Hiểu sâu, giải thích được cơ chế hoạt động, không chỉ biết dùng.

#### Các khái niệm CỐT LÕI phải nắm:
- [ ] **Event Loop & Asynchronous**
  - Call Stack, Task Queue, Microtask Queue
  - Thứ tự thực thi: `setTimeout` vs `Promise` vs `queueMicrotask`
  - Câu hỏi thường gặp: *"Giải thích output của đoạn code này"*
  
- [ ] **Closures & Scope**
  - Lexical Scope, Block Scope (`let/const` vs `var`)
  - Ứng dụng thực tế: data privacy, function factory
  - Câu hỏi: *"Closure là gì? Cho ví dụ thực tế"*

- [ ] **Prototypal Inheritance**
  - Prototype chain, `__proto__` vs `prototype`
  - `Object.create()`, `class` syntax (syntactic sugar)

- [ ] **ES6+ Features**
  - Destructuring, Spread/Rest operators
  - Template literals, Arrow functions (và sự khác biệt `this`)
  - `Map`, `Set`, `WeakMap`, `WeakRef`
  - Optional chaining `?.`, Nullish coalescing `??`
  - `Symbol`, `Iterator`, `Generator`

- [ ] **Promise & Async/Await**
  - Promise states: pending, fulfilled, rejected
  - `Promise.all()`, `Promise.allSettled()`, `Promise.race()`, `Promise.any()`
  - Error handling: `try/catch` trong async/await
  - Anti-patterns: callback hell, unhandled rejections

- [ ] **Module System**
  - CommonJS (`require/module.exports`) vs ES Modules (`import/export`)
  - Dynamic import `import()`
  - Circular dependency issues

### 1.2 TypeScript
**Mục tiêu:** Sử dụng thành thạo, hiểu type system sâu.

- [ ] **Kiểu dữ liệu cơ bản & nâng cao**
  - Primitive types, Union, Intersection
  - `interface` vs `type` (khi nào dùng gì?)
  - Generics: `<T>`, constraints, default types
  - Utility types: `Partial<T>`, `Required<T>`, `Pick<T>`, `Omit<T>`, `Record<K,V>`
  - Conditional types: `T extends U ? X : Y`
  - Mapped types, Template literal types

- [ ] **Decorators** (Quan trọng cho NestJS)
  - Class decorators, Method decorators, Property decorators, Parameter decorators
  - `reflect-metadata`
  - Decorator factory pattern

- [ ] **Enums, Namespaces, Declaration files (.d.ts)**

- [ ] **Strict mode & tsconfig.json**
  - `strict`, `strictNullChecks`, `noImplicitAny`
  - `paths`, `baseUrl`, module resolution

#### 📝 Câu hỏi phỏng vấn thường gặp:
```
1. Sự khác biệt giữa `interface` và `type` trong TypeScript?
2. Generics là gì? Cho ví dụ thực tế khi bạn sử dụng generics?
3. Giải thích cách hoạt động của Decorator trong TypeScript?
4. `any` vs `unknown` vs `never` khác nhau thế nào?
5. Giải thích Event Loop trong Node.js? Output của đoạn code sau là gì?
6. Promise.all() vs Promise.allSettled() khác nhau thế nào?
```

#### 🛠 Bài tập thực hành:
```
- Viết một generic function `deepClone<T>(obj: T): T`
- Implement Promise.all() từ đầu
- Viết decorator @Log() để log method calls
- Giải các bài tập Event Loop output prediction
```

---

## TUẦN 2-3: NODE.JS CORE & NESTJS FRAMEWORK

### 2.1 Node.js Core Concepts
- [ ] **Architecture**
  - V8 Engine, libuv, Event-driven non-blocking I/O
  - Single-threaded nhưng xử lý concurrent thế nào?
  - Worker Threads vs Child Processes vs Cluster

- [ ] **Streams & Buffers**
  - Readable, Writable, Duplex, Transform streams
  - Pipe pattern, backpressure
  - Khi nào dùng streams? (upload file lớn, real-time data)

- [ ] **Error Handling trong Node.js**
  - Unhandled rejections, uncaught exceptions
  - Error-first callback pattern
  - Graceful shutdown

- [ ] **Performance**
  - Memory leaks detection
  - Profiling, benchmarking
  - `process.nextTick()` vs `setImmediate()`

### 2.2 NestJS Framework (TRỌNG TÂM)
**Mục tiêu:** Đây là trọng tâm chính, cần nắm rất vững.

- [ ] **Kiến trúc & Concepts cốt lõi**
  - Module system (`@Module()`, `imports`, `exports`, `providers`)
  - Dependency Injection (DI) & IoC Container
  - Controllers, Services, Repositories pattern
  - Lifecycle hooks (`OnModuleInit`, `OnApplicationBootstrap`, etc.)

- [ ] **Modules nâng cao**
  - Dynamic modules (`forRoot()`, `forRootAsync()`, `forFeature()`)
  - Global modules (`@Global()`)
  - Lazy-loaded modules

- [ ] **Providers & DI sâu**
  - Custom providers: `useClass`, `useValue`, `useFactory`, `useExisting`
  - Scope: `DEFAULT` (Singleton), `REQUEST`, `TRANSIENT`
  - Circular dependency resolution (`forwardRef()`)

- [ ] **Middleware, Guards, Interceptors, Pipes, Filters**
  ```
  Request → Middleware → Guards → Interceptors (before) → Pipes → Handler 
  → Interceptors (after) → Exception Filters → Response
  ```
  - Middleware: logging, cors, rate limiting
  - Guards: authentication, authorization, role-based access
  - Interceptors: response transformation, caching, logging, timeout
  - Pipes: validation (`ValidationPipe`), transformation (`ParseIntPipe`)
  - Exception Filters: custom error responses, global error handling

- [ ] **Validation**
  - `class-validator` + `class-transformer`
  - DTOs (Data Transfer Objects) pattern
  - Custom validation decorators
  - Whitelist, transform options

- [ ] **Configuration**
  - `@nestjs/config`, `.env` files
  - Configuration namespaces
  - Validation schema (Joi/class-validator)

- [ ] **Testing trong NestJS**
  - Unit testing: `Test.createTestingModule()`
  - E2E testing: `@nestjs/testing`, supertest
  - Mocking providers

- [ ] **Microservices basics** (điểm cộng)
  - Transport layers: TCP, Redis, gRPC, Kafka
  - Message patterns, Event patterns

#### 📝 Câu hỏi phỏng vấn thường gặp:
```
1. Giải thích Dependency Injection trong NestJS? Tại sao nó quan trọng?
2. Sự khác biệt giữa Middleware, Guard, Interceptor và Pipe? 
   Cho ví dụ use case của từng loại?
3. Request lifecycle trong NestJS diễn ra thế nào?
4. Giải thích Dynamic Module? Khi nào cần dùng?
5. Provider scopes trong NestJS? Khi nào dùng REQUEST scope?
6. Bạn xử lý validation input thế nào trong NestJS?
7. Cách bạn tổ chức code trong một dự án NestJS lớn?
```

#### 🛠 Project thực hành:
```
Xây dựng REST API hoàn chỉnh với NestJS:
- User module (CRUD + Auth)
- Product module (CRUD + Search + Pagination)
- Order module (relationship với User & Product)
- Sử dụng Guards cho Auth, Pipes cho Validation
- Global Exception Filter
- Swagger documentation
- Unit tests cho Services
```

---

## TUẦN 3: EXPRESS.JS

### 3.1 Express.js Fundamentals
- [ ] **Core Concepts**
  - Middleware chain pattern
  - Request/Response lifecycle
  - Error handling middleware
  - Router, route parameters, query strings

- [ ] **So sánh Express vs NestJS**
  - Khi nào chọn Express? Khi nào chọn NestJS?
  - Performance comparison
  - Express: lightweight, flexible, unopinionated
  - NestJS: structured, opinionated, enterprise-ready

- [ ] **Common Middleware**
  - `cors`, `helmet`, `morgan`, `compression`
  - `express-rate-limit`, `express-validator`
  - Custom middleware patterns

#### 📝 Câu hỏi phỏng vấn:
```
1. Middleware trong Express hoạt động thế nào?
2. Cách xử lý error trong Express?
3. So sánh Express và NestJS?
```

---

## TUẦN 4-5: DATABASES (TypeORM, PostgreSQL, MongoDB, Redis)

### 4.1 TypeORM (TRỌNG TÂM)
- [ ] **Entity & Decorators**
  - `@Entity()`, `@Column()`, `@PrimaryGeneratedColumn()`
  - Column types, nullable, default values
  - `@CreateDateColumn()`, `@UpdateDateColumn()`, `@DeleteDateColumn()`

- [ ] **Relations**
  - `@OneToOne`, `@OneToMany`/`@ManyToOne`, `@ManyToMany`
  - Eager vs Lazy loading
  - `@JoinColumn()`, `@JoinTable()`
  - Cascade options: insert, update, remove, soft-remove

- [ ] **Query Builder & Repository pattern**
  - Repository API: `find()`, `findOne()`, `save()`, `remove()`
  - Find options: `where`, `order`, `relations`, `select`, `skip`, `take`
  - QueryBuilder: complex queries, subqueries, joins
  - Raw queries khi cần

- [ ] **Migrations**
  - Generate, run, revert migrations
  - Migration best practices
  - Schema synchronization vs Migrations (khi nào dùng gì)

- [ ] **Performance**
  - N+1 problem & cách giải quyết
  - Indexing strategies
  - Query optimization
  - Connection pooling
  - Transactions (`QueryRunner`, `@Transaction()`)

### 4.2 PostgreSQL
- [ ] **SQL Fundamentals nâng cao**
  - JOINs: INNER, LEFT, RIGHT, FULL, CROSS
  - Subqueries, CTEs (WITH clause)
  - Window functions: `ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`
  - Aggregate functions, GROUP BY, HAVING

- [ ] **PostgreSQL Specific**
  - JSONB columns & operators
  - Array types
  - Full-text search (`tsvector`, `tsquery`)
  - Indexing: B-tree, GIN, GiST, BRIN
  - EXPLAIN ANALYZE (query planning)
  - Partitioning

- [ ] **Database Design**
  - Normalization (1NF, 2NF, 3NF, BCNF)
  - Denormalization strategies
  - Index design principles

### 4.3 MongoDB
- [ ] **Core Concepts**
  - Document model, Collections, BSON
  - Schema design patterns (embedding vs referencing)
  - CRUD operations, aggregation pipeline
  - Indexing (compound, text, geospatial)

- [ ] **Mongoose (nếu dùng với Node.js)**
  - Schema definition, validation
  - Middleware (pre/post hooks)
  - Population (references)
  - Virtual fields

- [ ] **So sánh SQL vs NoSQL**
  - Khi nào dùng PostgreSQL? Khi nào dùng MongoDB?
  - ACID vs BASE
  - CAP theorem

### 4.4 Redis
- [ ] **Data Structures**
  - Strings, Lists, Sets, Sorted Sets, Hashes
  - Streams, Pub/Sub

- [ ] **Use Cases thường gặp**
  - Caching strategies: Cache-aside, Write-through, Write-behind
  - Session storage
  - Rate limiting
  - Queue (Bull/BullMQ với NestJS)
  - Leaderboard (Sorted Sets)
  - Real-time features (Pub/Sub)

- [ ] **Redis trong NestJS**
  - `@nestjs/cache-manager` hoặc `ioredis`
  - Cache interceptor
  - Bull queues (`@nestjs/bull`)

- [ ] **TTL, Eviction policies, Persistence (RDB, AOF)**

#### 📝 Câu hỏi phỏng vấn thường gặp:
```
1. N+1 problem là gì? Cách giải quyết trong TypeORM?
2. Khi nào dùng SQL, khi nào dùng NoSQL? Cho ví dụ cụ thể?
3. Giải thích các loại JOIN trong SQL?
4. Redis dùng để làm gì trong hệ thống của bạn? 
5. Caching strategy nào bạn thường dùng?
6. Giải thích ACID properties?
7. Indexing hoạt động thế nào? Khi nào không nên đánh index?
8. TypeORM migration workflow của bạn thế nào?
9. Bạn xử lý transaction thế nào?
```

#### 🛠 Bài tập thực hành:
```
- Thiết kế database cho hệ thống E-commerce (ERD)
- Viết complex queries với TypeORM QueryBuilder
- Implement caching layer với Redis trong NestJS
- Implement pagination (cursor-based vs offset-based)
- Viết migration scripts
```

---

## TUẦN 5-6: RESTful API DESIGN & SWAGGER

### 5.1 RESTful API Design Principles
- [ ] **REST Constraints**
  - Client-Server, Stateless, Cacheable
  - Uniform Interface, Layered System
  - HATEOAS (Hypermedia)

- [ ] **API Design Best Practices**
  - Resource naming conventions (plural nouns, kebab-case)
  - HTTP Methods: GET, POST, PUT, PATCH, DELETE
  - `PUT` vs `PATCH` (full update vs partial update)
  - Status codes:
    ```
    200 OK, 201 Created, 204 No Content
    400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict
    422 Unprocessable Entity
    500 Internal Server Error
    ```
  - Response format consistency
    ```json
    {
      "success": true,
      "data": {},
      "meta": { "page": 1, "limit": 10, "total": 100 }
    }
    ```

- [ ] **Pagination**
  - Offset-based: `?page=1&limit=10`
  - Cursor-based: `?cursor=abc123&limit=10`
  - Khi nào dùng loại nào?

- [ ] **Filtering, Sorting, Searching**
  - `?status=active&sort=-createdAt&search=keyword`
  - Nested filtering strategies

- [ ] **API Versioning**
  - URI versioning: `/api/v1/users`
  - Header versioning: `Accept: application/vnd.api.v1+json`
  - Query parameter: `?version=1`
  - Pros/cons của từng approach

- [ ] **Error Handling**
  - Consistent error response format
  - Error codes (application-level)
  - Validation error details
    ```json
    {
      "success": false,
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "Validation failed",
        "details": [
          { "field": "email", "message": "Invalid email format" }
        ]
      }
    }
    ```

- [ ] **Rate Limiting & Throttling**
  - Token bucket, sliding window algorithms
  - NestJS `@nestjs/throttler`

### 5.2 Swagger / OpenAPI
- [ ] **NestJS Swagger Integration**
  - `@nestjs/swagger` setup
  - `@ApiTags()`, `@ApiOperation()`, `@ApiResponse()`
  - `@ApiProperty()` trong DTOs
  - `@ApiBearerAuth()`, `@ApiQuery()`, `@ApiParam()`
  - Multiple swagger documents (admin vs public)

- [ ] **OpenAPI Spec**
  - Schemas, Parameters, Responses
  - Security definitions
  - Auto-generation từ DTOs

#### 📝 Câu hỏi phỏng vấn:
```
1. RESTful API là gì? Các nguyên tắc thiết kế?
2. PUT vs PATCH khác nhau thế nào?
3. Bạn xử lý API versioning thế nào?
4. Bạn thiết kế error response thế nào?
5. Cursor-based vs Offset-based pagination?
6. Bạn viết API documentation thế nào?
```

---

## TUẦN 6: AUTHENTICATION & SECURITY

### 6.1 JWT (JSON Web Token)
- [ ] **Cấu trúc JWT**: Header, Payload, Signature
- [ ] **Access Token vs Refresh Token**
  - Short-lived access token (15-30 min)
  - Long-lived refresh token (7-30 days)
  - Token rotation strategy
- [ ] **Token storage**: HttpOnly cookies vs localStorage
- [ ] **JWT trong NestJS**
  - `@nestjs/jwt`, `@nestjs/passport`
  - Passport strategies: `jwt`, `local`
  - Custom Guards

### 6.2 OAuth2
- [ ] **OAuth2 Flows**
  - Authorization Code Flow (recommended for web)
  - Client Credentials Flow (server-to-server)
  - Resource Owner Password (legacy)
  - PKCE extension
- [ ] **Social login**: Google, Facebook, GitHub
- [ ] **OpenID Connect (OIDC)**

### 6.3 Authorization & Access Control
- [ ] **RBAC** (Role-Based Access Control)
  - Roles, Permissions
  - NestJS: custom `@Roles()` decorator + `RolesGuard`
- [ ] **ABAC** (Attribute-Based Access Control)
- [ ] **CASL** library cho NestJS

### 6.4 Web Security
- [ ] **OWASP Top 10**
  - SQL Injection → parameterized queries
  - XSS → input sanitization, CSP headers
  - CSRF → CSRF tokens, SameSite cookies
  - Broken Authentication
  - Security Misconfiguration
- [ ] **CORS** (Cross-Origin Resource Sharing)
  - Origin, methods, headers, credentials
- [ ] **HTTPS, HSTS**
- [ ] **Password hashing**: bcrypt, argon2
- [ ] **Rate limiting** chống brute force
- [ ] **Input validation & sanitization**
- [ ] **Helmet.js** security headers

#### 📝 Câu hỏi phỏng vấn:
```
1. JWT hoạt động thế nào? Cấu trúc của JWT?
2. Access Token vs Refresh Token? Flow hoạt động?
3. Bạn lưu token ở đâu? Tại sao?
4. OAuth2 là gì? Giải thích Authorization Code Flow?
5. RBAC vs ABAC?
6. Cách bạn phòng chống SQL Injection, XSS, CSRF?
7. CORS là gì? Cách cấu hình?
```

#### 🛠 Bài tập:
```
- Implement auth module hoàn chỉnh trong NestJS:
  + Register, Login, Refresh Token, Logout
  + JWT strategy với Passport
  + Role-based guards
  + Password reset flow
```

---

## TUẦN 7: REACTJS & GIT

### 7.1 ReactJS
- [ ] **Core Concepts**
  - JSX, Components (Functional vs Class)
  - Props, State, Lifecycle
  - Virtual DOM & Reconciliation

- [ ] **Hooks**
  - `useState`, `useEffect`, `useContext`
  - `useReducer`, `useMemo`, `useCallback`, `useRef`
  - Custom hooks
  - Rules of Hooks

- [ ] **State Management**
  - Context API + useReducer
  - Redux Toolkit / Zustand / Jotai
  - Server state: React Query / TanStack Query

- [ ] **Routing**: React Router v6

- [ ] **Performance**
  - `React.memo()`, `useMemo`, `useCallback`
  - Code splitting: `React.lazy()` + `Suspense`
  - Virtualization (react-window)

- [ ] **Forms**: React Hook Form + Zod/Yup

- [ ] **API Integration**
  - Axios / Fetch
  - Error handling, loading states
  - Interceptors

- [ ] **Testing**
  - React Testing Library
  - Jest

### 7.2 Git & Git Flow
- [ ] **Git Fundamentals**
  - Working directory, Staging area, Repository
  - `add`, `commit`, `push`, `pull`, `fetch`
  - `merge` vs `rebase` (khi nào dùng gì?)
  - `cherry-pick`, `stash`, `reset`, `revert`
  - `reset --soft` vs `--mixed` vs `--hard`
  - Interactive rebase: `git rebase -i`

- [ ] **Branching Strategies**
  - **Git Flow**: main, develop, feature/*, release/*, hotfix/*
  - **GitHub Flow**: main + feature branches
  - **Trunk-based development**
  - Khi nào dùng strategy nào?

- [ ] **Pull Request Workflow**
  - Code review best practices
  - PR template
  - Conventional commits: `feat:`, `fix:`, `chore:`, `docs:`
  - Squash merge vs Merge commit vs Rebase merge

- [ ] **Conflict Resolution**
  - Merge conflicts: nguyên nhân và cách giải quyết
  - `git mergetool`

- [ ] **Advanced**
  - `.gitignore` patterns
  - Git hooks (husky, lint-staged)
  - Semantic versioning

#### 📝 Câu hỏi phỏng vấn:
```
1. Merge vs Rebase khác nhau thế nào?
2. Git Flow là gì? Mô tả branching strategy bạn dùng?
3. Bạn giải quyết merge conflict thế nào?
4. Reset vs Revert khác nhau thế nào?
5. Giải thích React hooks lifecycle?
6. useMemo vs useCallback?
7. Bạn quản lý state trong React thế nào?
```

---

## TUẦN 8: DOCKER & KUBERNETES

### 8.1 Docker
- [ ] **Core Concepts**
  - Image, Container, Registry
  - Dockerfile instructions: `FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`, `ENTRYPOINT`, `EXPOSE`, `ENV`
  - `CMD` vs `ENTRYPOINT`
  - Build context

- [ ] **Best Practices**
  - Multi-stage builds (giảm image size)
  ```dockerfile
  # Build stage
  FROM node:18-alpine AS builder
  WORKDIR /app
  COPY package*.json ./
  RUN npm ci
  COPY . .
  RUN npm run build

  # Production stage
  FROM node:18-alpine
  WORKDIR /app
  COPY --from=builder /app/dist ./dist
  COPY --from=builder /app/node_modules ./node_modules
  CMD ["node", "dist/main.js"]
  ```
  - `.dockerignore`
  - Layer caching optimization
  - Non-root user
  - Alpine images

- [ ] **Docker Compose**
  - `docker-compose.yml` structure
  - Services, networks, volumes
  - Environment variables
  - Depends_on, healthcheck
  ```yaml
  version: '3.8'
  services:
    app:
      build: .
      ports:
        - "3000:3000"
      depends_on:
        - db
        - redis
    db:
      image: postgres:15
      volumes:
        - pgdata:/var/lib/postgresql/data
      environment:
        POSTGRES_DB: mydb
        POSTGRES_PASSWORD: secret
    redis:
      image: redis:7-alpine
  volumes:
    pgdata:
  ```

- [ ] **Networking**
  - Bridge, Host, None
  - Container-to-container communication

- [ ] **Volumes & Data Persistence**
  - Named volumes, Bind mounts, tmpfs

### 8.2 Kubernetes (Basics)
- [ ] **Core Concepts**
  - Cluster, Node, Pod
  - Deployment, Service, Ingress
  - ConfigMap, Secret
  - Namespace

- [ ] **Workloads**
  - Pod: smallest deployable unit
  - Deployment: manages ReplicaSets
  - StatefulSet: for stateful apps (databases)
  - DaemonSet, Job, CronJob

- [ ] **Networking**
  - Service types: ClusterIP, NodePort, LoadBalancer
  - Ingress controller

- [ ] **Configuration**
  - ConfigMap & Secret
  - Environment variables in pods
  - Resource limits (CPU, memory)

- [ ] **Basic kubectl commands**
  ```bash
  kubectl get pods/services/deployments
  kubectl describe pod <name>
  kubectl logs <pod-name>
  kubectl apply -f deployment.yaml
  kubectl scale deployment <name> --replicas=3
  kubectl port-forward <pod> 3000:3000
  ```

- [ ] **Kubernetes YAML Manifests**
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-app
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: my-app
    template:
      metadata:
        labels:
          app: my-app
      spec:
        containers:
        - name: my-app
          image: my-app:latest
          ports:
          - containerPort: 3000
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
  ```

- [ ] **Health Checks**: liveness probe, readiness probe, startup probe

#### 📝 Câu hỏi phỏng vấn:
```
1. Docker image vs container?
2. CMD vs ENTRYPOINT?
3. Multi-stage build là gì? Tại sao cần?
4. Docker Compose dùng để làm gì?
5. Pod trong Kubernetes là gì?
6. Deployment vs StatefulSet?
7. Service types trong K8s?
8. Bạn deploy ứng dụng Node.js bằng Docker thế nào?
```

#### 🛠 Bài tập:
```
- Dockerize NestJS app + PostgreSQL + Redis
- Viết docker-compose.yml cho full-stack app
- Viết Kubernetes manifests cơ bản
```

---

## TUẦN 9: SYSTEM DESIGN & PHÂN TÍCH YÊU CẦU

### 9.1 System Design Fundamentals
- [ ] **Scalability**
  - Horizontal vs Vertical scaling
  - Load balancing (Round Robin, Least Connections, IP Hash)
  - Database scaling: Read replicas, Sharding, Partitioning

- [ ] **Caching**
  - CDN, Application cache (Redis), Database cache
  - Cache invalidation strategies
  - Cache-aside, Write-through, Write-behind

- [ ] **Message Queues**
  - RabbitMQ, Kafka, Bull (Redis-based)
  - Use cases: async processing, event-driven architecture
  - Dead letter queues

- [ ] **Design Patterns**
  - Repository pattern
  - Service layer pattern
  - Factory pattern
  - Observer/Event pattern
  - Strategy pattern
  - Singleton pattern

- [ ] **Architecture Patterns**
  - Monolith vs Microservices
  - Event-Driven Architecture
  - CQRS (Command Query Responsibility Segregation)
  - Domain-Driven Design (DDD) basics

- [ ] **API Gateway pattern**
- [ ] **Circuit Breaker pattern**
- [ ] **Database per Service pattern**

### 9.2 Phân tích yêu cầu & Lập kế hoạch
- [ ] **Requirements Analysis**
  - Functional vs Non-functional requirements
  - User stories format: "As a [role], I want [feature], so that [benefit]"
  - Acceptance criteria

- [ ] **Estimation**
  - Story points
  - T-shirt sizing
  - Breaking down tasks

- [ ] **Development Planning**
  - Sprint planning
  - Task prioritization (MoSCoW method)
  - Technical debt management

### 9.3 System Design Practice
- [ ] **Thiết kế các hệ thống phổ biến:**
  ```
  1. URL Shortener (like bit.ly)
  2. Chat Application (real-time)
  3. E-commerce System
  4. Social Media Feed
  5. File Upload Service
  6. Notification System
  7. Rate Limiter
  ```

- [ ] **Template trả lời System Design:**
  ```
  1. Clarify requirements & constraints
  2. Estimate scale (users, requests/sec, data size)
  3. High-level design (components diagram)
  4. API design
  5. Database schema
  6. Deep dive into components
  7. Bottlenecks & trade-offs
  ```

#### 📝 Câu hỏi phỏng vấn:
```
1. Monolith vs Microservices? Khi nào chọn gì?
2. Thiết kế hệ thống notification real-time?
3. Bạn xử lý high traffic thế nào?
4. Giải thích caching strategies?
5. Message queue dùng khi nào?
6. Bạn phân tích requirement thế nào khi nhận task mới?
```

---

## TUẦN 10: TỔNG ÔN & MOCK INTERVIEW

### 10.1 Checklist tự đánh giá

#### Backend (Node.js / NestJS):
- [ ] Giải thích Event Loop và output đoạn code bất kỳ
- [ ] Viết NestJS module từ đầu (controller, service, DTO, entity)
- [ ] Implement JWT auth flow hoàn chỉnh
- [ ] Viết TypeORM query phức tạp
- [ ] Thiết kế RESTful API cho một feature
- [ ] Viết Swagger documentation
- [ ] Giải thích NestJS request lifecycle

#### Frontend (React):
- [ ] Giải thích Virtual DOM, Reconciliation
- [ ] Viết custom hook
- [ ] State management approach
- [ ] Performance optimization techniques

#### Database:
- [ ] Thiết kế ERD cho hệ thống phức tạp
- [ ] Viết complex SQL queries
- [ ] Redis use cases và implementation
- [ ] Database optimization (indexing, query planning)

#### DevOps:
- [ ] Dockerize một ứng dụng
- [ ] Viết docker-compose
- [ ] Giải thích K8s concepts

#### Soft Skills:
- [ ] Mô tả quy trình phân tích requirements
- [ ] Git workflow trong team
- [ ] Code review practices
- [ ] Xử lý deadline pressure

### 10.2 Mock Interview Sessions
```
Session 1: Technical Screening (45 min)
- JavaScript/TypeScript fundamentals
- NestJS concepts
- Database design

Session 2: Coding Challenge (60 min)  
- Live coding: build a feature with NestJS
- Algorithm/data structure (nếu có)

Session 3: System Design (45 min)
- Design a system from scratch
- Trade-offs discussion

Session 4: Behavioral (30 min)
- Past experience, teamwork
- Problem-solving approach
- Handling conflicts/deadlines
```

### 10.3 Behavioral Questions Preparation
```
1. Kể về project phức tạp nhất bạn từng làm?
2. Bạn đã giải quyết một bug khó thế nào?
3. Khi có disagreement về technical decision, bạn xử lý thế nào?
4. Bạn học công nghệ mới thế nào?
5. Mô tả cách bạn review code?
```

---

## 📚 TÀI NGUYÊN HỌC TẬP

### Documentation (ưu tiên đọc):
- [NestJS Official Docs](https://docs.nestjs.com/) ⭐⭐⭐
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [TypeORM Documentation](https://typeorm.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Redis Documentation](https://redis.io/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

### Courses & Tutorials:
- NestJS Zero to Hero (Udemy - Ariel Weinberger)
- Docker & Kubernetes: The Complete Guide (Udemy - Stephen Grider)
- React - The Complete Guide (Udemy - Maximilian Schwarzmüller)

### Practice Platforms:
- **Coding**: LeetCode, HackerRank (JavaScript)
- **System Design**: DesignGuru, ByteByteGo
- **SQL**: SQLZoo, LeetCode Database

### GitHub Repos tham khảo:
- `nestjs/nest` - NestJS source code
- `goldbergyoni/nodebestpractices` - Node.js best practices
- `donnemartin/system-design-primer` - System Design
- `yangshun/tech-interview-handbook` - Interview prep

---

## 🎯 DAILY ROUTINE GỢI Ý

```
Sáng (2h):   Đọc docs/theory + ghi chép
Chiều (3h):  Code thực hành (project)
Tối (1h):    Review flashcards + giải câu hỏi phỏng vấn
Cuối tuần:   Mock interview + System design practice
```

---

## 💡 MẸO PHỎNG VẤN

1. **Không chỉ biết dùng, phải hiểu TẠI SAO** - "Tại sao dùng NestJS thay vì Express?"
2. **Luôn cho ví dụ thực tế** - Liên hệ với project bạn đã làm
3. **Thành thật** - Nếu không biết, nói "Tôi chưa có kinh nghiệm với phần này, nhưng..."
4. **Hỏi ngược lại** - Thể hiện tư duy phân tích
5. **Trade-offs** - Mỗi quyết định đều có ưu/nhược, hãy nêu cả hai
6. **STAR method** cho behavioral questions: Situation, Task, Action, Result

---

> **Lưu ý:** Plan này khá comprehensive. Tùy vào nền tảng hiện tại, bạn có thể:
> - Bỏ qua những phần đã vững
> - Tập trung nhiều hơn vào NestJS, TypeORM, API Design (vì đây là core requirements)
> - Điều chỉnh timeline cho phù hợp

**Chúc bạn phỏng vấn thành công! 🚀**
