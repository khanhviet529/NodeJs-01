# PHẦN 6: RESTful API DESIGN & SWAGGER - GIẢI THÍCH CHI TIẾT

> **Mức độ ưu tiên: CAO**
> API design là kỹ năng hàng ngày. Interviewer sẽ hỏi về naming conventions,
> HTTP methods, status codes, pagination, versioning, và error handling.

---

## 1. REST LÀ GÌ?

**REST (Representational State Transfer)** = Kiến trúc thiết kế API dựa trên HTTP.
Không phải giao thức, mà là tập hợp **nguyên tắc/ràng buộc**.

### 6 Ràng buộc của REST:

```
1. CLIENT-SERVER
   → Client và Server tách biệt, giao tiếp qua HTTP
   → Client không biết server lưu data thế nào
   → Server không biết client hiển thị thế nào

2. STATELESS
   → Server KHÔNG lưu state của client giữa các requests
   → Mỗi request phải chứa ĐẦY ĐỦ thông tin (auth token, context)
   → Dễ scale (bất kỳ server nào cũng xử lý được)

3. CACHEABLE
   → Response phải đánh dấu cacheable hay không
   → Giảm tải server, tăng performance

4. UNIFORM INTERFACE
   → Resource được identify bằng URI: /users, /products
   → Manipulation qua HTTP methods: GET, POST, PUT, DELETE
   → Self-descriptive messages (headers, content-type)

5. LAYERED SYSTEM
   → Client không biết connect trực tiếp đến server hay qua proxy/load balancer
   → Cho phép thêm layers: CDN, API Gateway, Security

6. CODE ON DEMAND (optional)
   → Server có thể gửi executable code (JavaScript) cho client
```

---

## 2. HTTP METHODS

```
┌──────────┬──────────────────────────────────────────────┬───────────┬────────────┐
│ Method   │ Ý nghĩa                                      │ Idempotent│ Safe       │
├──────────┼──────────────────────────────────────────────┼───────────┼────────────┤
│ GET      │ Lấy data (read)                              │ ✅ Có     │ ✅ Có     │
│ POST     │ Tạo resource mới (create)                    │ ❌ Không  │ ❌ Không  │
│ PUT      │ Thay thế TOÀN BỘ resource (full update)      │ ✅ Có     │ ❌ Không  │
│ PATCH    │ Cập nhật MỘT PHẦN resource (partial update)  │ ❌ Không* │ ❌ Không  │
│ DELETE   │ Xóa resource                                 │ ✅ Có     │ ❌ Không  │
└──────────┴──────────────────────────────────────────────┴───────────┴────────────┘

* PATCH có thể idempotent tùy implementation

Idempotent = Gọi nhiều lần cho CÙNG kết quả
  PUT /users/1 { name: "Alice" } → gọi 100 lần vẫn cùng kết quả
  DELETE /users/1 → gọi 100 lần, user vẫn bị xóa (không lỗi)
  POST /users { name: "Alice" } → gọi 100 lần = tạo 100 users ← KHÔNG idempotent

Safe = Không thay đổi state trên server
  GET /users → chỉ đọc, không thay đổi gì
```

### PUT vs PATCH (HAY HỎI):

```javascript
// User hiện tại: { id: 1, name: "Alice", email: "alice@old.com", age: 25 }

// PUT /users/1 → THAY THẾ TOÀN BỘ (phải gửi tất cả fields)
// Body: { name: "Alice", email: "alice@new.com", age: 25 }
// → Nếu thiếu field → field đó bị NULL/removed

// PATCH /users/1 → CẬP NHẬT MỘT PHẦN (chỉ gửi fields cần update)
// Body: { email: "alice@new.com" }
// → Chỉ email thay đổi, name và age giữ nguyên

// Trong thực tế: phần lớn dùng PATCH (hoặc PUT nhưng handle như PATCH)
```

---

## 3. RESOURCE NAMING CONVENTIONS

```
✅ ĐÚNG:
GET    /api/v1/users              → Lấy danh sách users
GET    /api/v1/users/123          → Lấy user với id=123
POST   /api/v1/users              → Tạo user mới
PATCH  /api/v1/users/123          → Update user 123
DELETE /api/v1/users/123          → Xóa user 123

GET    /api/v1/users/123/posts    → Lấy posts của user 123 (nested resource)
POST   /api/v1/users/123/posts    → Tạo post cho user 123

❌ SAI:
GET    /api/v1/getUsers           → Đừng dùng động từ (HTTP method đã là động từ)
GET    /api/v1/user               → Dùng PLURAL (users, không phải user)
POST   /api/v1/createUser         → Đừng dùng "create" (POST đã = create)
GET    /api/v1/Users              → Dùng lowercase + kebab-case
GET    /api/v1/user_posts         → Dùng kebab-case: /user-posts hoặc nested

NAMING RULES:
1. Dùng danh từ SỐ NHIỀU: /users, /products, /orders
2. Lowercase + kebab-case: /user-profiles, /order-items
3. Nested resources cho quan hệ: /users/123/posts
4. KHÔNG quá 3 levels nested: /users/123/posts/456 (OK)
   /users/123/posts/456/comments/789/likes (QUÁ SÂU → dùng query params)
```

---

## 4. HTTP STATUS CODES

```
═══ 2xx SUCCESS ═══
200 OK              → GET thành công, PATCH thành công
201 Created         → POST tạo resource thành công (trả về resource + Location header)
204 No Content      → DELETE thành công (không trả body)

═══ 3xx REDIRECT ═══
301 Moved Permanently → URL cũ → URL mới (permanent)
304 Not Modified      → Cache còn valid (không cần gửi data)

═══ 4xx CLIENT ERROR ═══
400 Bad Request       → Input không hợp lệ, JSON syntax error
401 Unauthorized      → CHƯA authentication (chưa login, token hết hạn)
                        ⚠️ Tên "Unauthorized" nhưng ý nghĩa là "Unauthenticated"
403 Forbidden         → ĐÃ authentication nhưng KHÔNG có quyền (authorization)
404 Not Found         → Resource không tồn tại
409 Conflict          → Xung đột (email đã tồn tại, version conflict)
422 Unprocessable Entity → Validation errors (format đúng nhưng logic sai)
429 Too Many Requests → Rate limit exceeded

═══ 5xx SERVER ERROR ═══
500 Internal Server Error → Bug trên server
502 Bad Gateway           → Upstream server lỗi (proxy/load balancer issue)
503 Service Unavailable   → Server quá tải hoặc đang maintenance
504 Gateway Timeout       → Upstream server timeout
```

### 401 vs 403 (HAY NHẦM):

```
401 Unauthorized (thực ra là Unauthenticated):
→ "Bạn là AI? Tôi không biết bạn" → Chưa login, token hết hạn
→ Solution: Login lại, refresh token

403 Forbidden:
→ "Tôi biết bạn là Alice, nhưng Alice không có quyền xóa user"
→ Solution: Dùng account có quyền cao hơn
```

---

## 5. RESPONSE FORMAT

### Standard Success Response:

```json
// GET /api/v1/users/123
{
  "success": true,
  "data": {
    "id": 123,
    "name": "Alice",
    "email": "alice@test.com",
    "createdAt": "2024-01-15T10:30:00.000Z"
  }
}

// GET /api/v1/users (list with pagination)
{
  "success": true,
  "data": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 150,
    "totalPages": 15
  }
}
```

### Standard Error Response:

```json
// 400 Bad Request - Validation Error
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      },
      {
        "field": "password",
        "message": "Must be at least 8 characters"
      }
    ]
  }
}

// 404 Not Found
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "User with ID 999 not found"
  }
}

// 409 Conflict
{
  "success": false,
  "error": {
    "code": "CONFLICT",
    "message": "Email already registered"
  }
}
```

---

## 6. PAGINATION

### Offset-based Pagination:

```
GET /api/v1/users?page=2&limit=10

Cách hoạt động:
- page=1: OFFSET 0, LIMIT 10  → rows 1-10
- page=2: OFFSET 10, LIMIT 10 → rows 11-20
- page=3: OFFSET 20, LIMIT 10 → rows 21-30

SQL: SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 10;

✅ Ưu: Đơn giản, jump to page bất kỳ
❌ Nhược: 
  - OFFSET lớn → CHẬM (DB vẫn phải scan qua rows bị skip)
  - Data thay đổi → kết quả không consistent (duplicate/skip items)
```

### Cursor-based Pagination:

```
GET /api/v1/users?cursor=eyJpZCI6MTB9&limit=10

Cách hoạt động:
- cursor encode thông tin "vị trí cuối cùng" (thường là ID hoặc timestamp)
- Request tiếp: "lấy 10 items SAU cursor này"

SQL: SELECT * FROM users WHERE id > 10 ORDER BY id LIMIT 10;

Response:
{
  "data": [...],
  "meta": {
    "hasNext": true,
    "nextCursor": "eyJpZCI6MjB9"  // Base64 encode { id: 20 }
  }
}

✅ Ưu:
  - Performance ổn định (không OFFSET)
  - Consistent khi data thay đổi
❌ Nhược:
  - Không jump to page bất kỳ
  - Phức tạp hơn
```

### Khi nào dùng gì?

```
Offset-based: Admin panel, reports, khi cần jump to page
Cursor-based: Feed (Facebook, Twitter), infinite scroll, real-time data
```

### Implementation trong NestJS:

```typescript
// pagination.dto.ts
export class PaginationDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit: number = 10;
  
  @IsOptional()
  @IsString()
  sortBy: string = 'createdAt';
  
  @IsOptional()
  @IsEnum(['ASC', 'DESC'])
  sortOrder: 'ASC' | 'DESC' = 'DESC';
}

// users.service.ts
async findAll(query: PaginationDto) {
  const [data, total] = await this.userRepo.findAndCount({
    skip: (query.page - 1) * query.limit,
    take: query.limit,
    order: { [query.sortBy]: query.sortOrder },
  });
  
  return {
    data,
    meta: {
      page: query.page,
      limit: query.limit,
      total,
      totalPages: Math.ceil(total / query.limit),
      hasNext: query.page * query.limit < total,
      hasPrev: query.page > 1,
    }
  };
}
```

---

## 7. API VERSIONING

### 3 Strategies:

```
1. URI Versioning (phổ biến nhất)
   GET /api/v1/users
   GET /api/v2/users
   
   ✅ Rõ ràng, dễ hiểu, dễ route
   ❌ URL dài, nhiều endpoints duplicate

2. Header Versioning
   GET /api/users
   Accept: application/vnd.myapi.v1+json
   
   ✅ URL sạch
   ❌ Khó debug (không thấy version trong URL)

3. Query Parameter
   GET /api/users?version=1
   
   ✅ Dễ implement
   ❌ Không RESTful lắm
```

### NestJS Versioning:

```typescript
// main.ts
app.enableVersioning({
  type: VersioningType.URI,  // /v1/users, /v2/users
  defaultVersion: '1',
});

// controller
@Controller('users')
export class UsersController {
  @Version('1')
  @Get()
  findAllV1() {
    return 'V1: Returns basic user data';
  }

  @Version('2')
  @Get()
  findAllV2() {
    return 'V2: Returns user data with profile';
  }
}
```

---

## 8. SWAGGER / OPENAPI

### Setup trong NestJS:

```typescript
// main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('My API')
  .setDescription('API Documentation')
  .setVersion('1.0')
  .addBearerAuth()                    // JWT auth
  .addTag('users', 'User management') // Tags cho grouping
  .addTag('products')
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api-docs', app, document);  // URL: /api-docs
```

### Decorators cho Swagger:

```typescript
@ApiTags('users')                    // Group trong Swagger UI
@ApiBearerAuth()                     // Yêu cầu JWT token
@Controller('users')
export class UsersController {
  
  @Post()
  @ApiOperation({ summary: 'Create a new user' })  // Mô tả API
  @ApiResponse({ status: 201, description: 'User created successfully' })
  @ApiResponse({ status: 400, description: 'Validation error' })
  @ApiResponse({ status: 409, description: 'Email already exists' })
  createUser(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
  
  @Get()
  @ApiOperation({ summary: 'Get all users' })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiQuery({ name: 'limit', required: false, type: Number })
  @ApiQuery({ name: 'search', required: false, type: String })
  findAll(@Query() query: PaginationDto) {
    return this.usersService.findAll(query);
  }
  
  @Get(':id')
  @ApiParam({ name: 'id', description: 'User ID' })
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.findOne(id);
  }
}

// DTO with Swagger decorators
export class CreateUserDto {
  @ApiProperty({
    description: 'User full name',
    example: 'Alice Johnson',
    minLength: 2,
  })
  @IsString()
  @MinLength(2)
  name: string;

  @ApiProperty({
    description: 'User email address',
    example: 'alice@example.com',
  })
  @IsEmail()
  email: string;

  @ApiPropertyOptional({               // Optional field
    description: 'User role',
    enum: UserRole,
    default: UserRole.USER,
  })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;
}
```

---

## TÓM TẮT PHẦN 6

### Câu hỏi phỏng vấn:

**1. "RESTful API là gì?"**
→ Kiến trúc API dựa trên HTTP, tuân theo các ràng buộc: stateless, client-server,
uniform interface, cacheable. Resources được identify bằng URI, thao tác bằng HTTP methods.

**2. "PUT vs PATCH?"**
→ PUT: thay thế toàn bộ resource (gửi tất cả fields). PATCH: cập nhật một phần 
(chỉ gửi fields thay đổi). Thực tế hay dùng PATCH hơn.

**3. "401 vs 403?"**
→ 401: chưa authentication (chưa login). 403: đã authentication nhưng không có quyền.

**4. "Cursor-based vs Offset-based pagination?"**
→ Offset: đơn giản, jump to page. Cursor: performance tốt hơn, consistent hơn.
Offset cho admin panel. Cursor cho feed/infinite scroll.

**5. "API versioning?"**
→ 3 cách: URI (/v1/users), Header, Query param. URI phổ biến nhất vì rõ ràng.

**6. "Bạn thiết kế error response thế nào?"**
→ Consistent format: { success, error: { code, message, details } }.
Dùng application-level error codes. Validation errors kèm chi tiết từng field.
