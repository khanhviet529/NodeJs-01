# PHẦN 3: NESTJS FRAMEWORK - GIẢI THÍCH CHI TIẾT

> **Tại sao phần này QUAN TRỌNG NHẤT?**
> NestJS xuất hiện nhiều nhất trong JD. Đây là framework chính mà bạn sẽ dùng hàng ngày.
> Interviewer sẽ hỏi sâu về NestJS: architecture, DI, request lifecycle, modules, testing.
> Nắm vững NestJS = 50% vượt qua phỏng vấn.

---

## 1. NESTJS LÀ GÌ? TẠI SAO CHỌN NESTJS?

### Định nghĩa:
NestJS là một **progressive Node.js framework** xây dựng trên TypeScript, lấy cảm hứng từ 
**Angular** (DI, Modules, Decorators). Bên dưới nó dùng **Express** (mặc định) hoặc **Fastify**.

### So sánh với Express:

```
Express:
- Minimalist, unopinionated (không ép cấu trúc)
- Bạn tự quyết định cấu trúc thư mục, patterns
- Nhanh cho prototype, nhưng code base lớn → khó maintain
- Không có built-in DI, validation, swagger...

NestJS:
- Opinionated (có cấu trúc rõ ràng)
- Module-based architecture (giống Angular)
- Built-in Dependency Injection
- Decorators (TypeScript)
- Built-in support: validation, swagger, testing, microservices, WebSocket...
- Enterprise-ready
```

### Khi nào chọn NestJS thay vì Express?
- Project lớn, team nhiều người → Cần cấu trúc chuẩn
- Cần microservices, WebSocket, GraphQL → NestJS có sẵn
- Cần testing dễ dàng → DI giúp mock dễ
- Long-term maintenance → Code có tổ chức

---

## 2. KIẾN TRÚC NESTJS

### Module System:

```
AppModule (Root)
├── UserModule
│   ├── UserController     → Xử lý HTTP requests
│   ├── UserService        → Business logic
│   └── UserRepository     → Data access
├── ProductModule
│   ├── ProductController
│   ├── ProductService
│   └── ProductRepository
├── AuthModule
│   ├── AuthController
│   ├── AuthService
│   └── JwtStrategy
└── SharedModule (shared utilities)
    ├── LoggerService
    └── CacheService
```

### @Module() Decorator:

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])],  // Modules/features cần dùng
  controllers: [UserController],                 // Xử lý HTTP routes
  providers: [UserService],                      // Các service (injectable)
  exports: [UserService],                        // Cho phép modules khác dùng
})
export class UserModule {}
```

**Giải thích từng phần:**

- **imports**: Import modules khác để dùng providers của chúng. 
  Ví dụ: import `TypeOrmModule` để dùng Repository.

- **controllers**: Đăng ký controllers xử lý incoming requests.

- **providers**: Đăng ký services, repositories, factories... 
  Mọi thứ có thể được inject đều là provider.

- **exports**: "Xuất khẩu" providers để modules khác import và sử dụng.
  Nếu không export, provider chỉ dùng được trong module hiện tại.

```
UserModule exports [UserService]
                 ↓
OrderModule imports [UserModule]
                 ↓
OrderService có thể inject UserService ← vì nó được export
```

---

## 3. DEPENDENCY INJECTION (DI) - KHÁI NIỆM CỐT LÕI

### DI là gì?

**Dependency Injection** = thay vì tự tạo dependencies, bạn **khai báo** cần gì, 
và framework **tự động cung cấp** (inject) cho bạn.

```typescript
// ❌ KHÔNG dùng DI: Tự tạo dependency
class OrderService {
  private userService: UserService;
  private emailService: EmailService;
  
  constructor() {
    this.userService = new UserService();     // Tự tạo → tight coupling
    this.emailService = new EmailService();   // Khó test, khó thay đổi
  }
}

// ✅ DÙng DI: Framework inject dependency
@Injectable()
class OrderService {
  constructor(
    private readonly userService: UserService,      // ← NestJS tự inject
    private readonly emailService: EmailService,    // ← NestJS tự inject
  ) {}
}
```

### Tại sao DI quan trọng?

1. **Loose Coupling**: Classes không phụ thuộc trực tiếp vào implementation cụ thể
2. **Easy Testing**: Có thể inject mock/fake dependencies khi test
3. **Flexibility**: Dễ thay đổi implementation mà không sửa code dùng nó
4. **Single Responsibility**: Mỗi class chỉ lo logic của nó

### IoC Container (Inversion of Control):

```
NestJS IoC Container = "kho chứa" tất cả providers

Khi app khởi động:
1. NestJS scan tất cả @Module, @Injectable, @Controller
2. Tạo instance của mỗi provider
3. Resolve dependencies (nếu A cần B, tạo B trước)
4. Inject dependencies vào constructor

┌─────────────────────────────────────┐
│           IoC Container              │
│                                     │
│  UserService (singleton) ──────┐    │
│  ProductService (singleton) ──┐│    │
│  EmailService (singleton)    ││    │
│  OrderService ◄──────────────┘│    │
│       ↑                       │    │
│       └───────────────────────┘    │
└─────────────────────────────────────┘
```

### Custom Providers:

```typescript
// 1. useClass - Thay thế implementation
@Module({
  providers: [
    {
      provide: UserService,          // Token (AI cung cấp?)
      useClass: MockUserService,     // Implementation thật (cung CẤP gì?)
    },
  ],
})
// Khi inject UserService → nhận MockUserService instance

// 2. useValue - Cung cấp giá trị cố định
@Module({
  providers: [
    {
      provide: 'API_KEY',
      useValue: 'my-secret-key-123',
    },
  ],
})
// Inject: @Inject('API_KEY') private apiKey: string

// 3. useFactory - Tạo provider dynamic (có thể async)
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        const dbConfig = configService.get('database');
        return await createConnection(dbConfig);
      },
      inject: [ConfigService],  // Dependencies cho factory
    },
  ],
})

// 4. useExisting - Alias cho provider khác
@Module({
  providers: [
    UserService,
    {
      provide: 'AliasedUserService',
      useExisting: UserService,  // Cùng instance
    },
  ],
})
```

### Provider Scopes:

```typescript
// DEFAULT (Singleton) - 1 instance cho TOÀN APP (mặc định)
@Injectable()  // hoặc @Injectable({ scope: Scope.DEFAULT })
class UserService {
  // Tạo 1 lần, dùng mãi
  // ✅ Dùng cho: phần lớn services
}

// REQUEST - 1 instance mới cho MỖI REQUEST
@Injectable({ scope: Scope.REQUEST })
class RequestLogger {
  constructor(@Inject(REQUEST) private request: Request) {
    // Mỗi HTTP request tạo 1 instance mới
    // Có access vào request object hiện tại
  }
  // ✅ Dùng cho: logging per-request, multi-tenancy
  // ⚠️ Chậm hơn Singleton (tạo mới mỗi request)
}

// TRANSIENT - 1 instance mới cho MỖI INJECTION
@Injectable({ scope: Scope.TRANSIENT })
class HelperService {
  // Mỗi nơi inject → nhận instance KHÁC NHAU
  // ✅ Dùng cho: stateful services cần isolation
}
```

---

## 4. REQUEST LIFECYCLE (CỰC KỲ QUAN TRỌNG - HAY HỎI)

```
Incoming HTTP Request
        │
        ▼
┌───────────────────┐
│    MIDDLEWARE      │  → Chạy TRƯỚC mọi thứ
│  (logging, cors)  │  → Giống Express middleware
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│      GUARDS       │  → Quyết định: request có được đi tiếp không?
│  (auth, roles)    │  → Return true/false
│                   │  → Nếu false → 403 Forbidden
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│   INTERCEPTORS    │  → BEFORE handler
│   (pre-process)   │  → Transform request, logging, caching
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│      PIPES        │  → Validation & Transformation
│  (validate DTOs)  │  → Nếu invalid → 400 Bad Request
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│  ROUTE HANDLER    │  → Controller method (business logic)
│  (controller)     │  → Gọi service → return response
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│   INTERCEPTORS    │  → AFTER handler
│  (post-process)   │  → Transform response, timing
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ EXCEPTION FILTERS │  → Bắt và xử lý errors
│  (error handling) │  → Format error response
└───────┬───────────┘
        │
        ▼
   HTTP Response
```

### Chi tiết từng thành phần:

### 4.1 Middleware

```typescript
// Middleware chạy ĐẦU TIÊN, giống Express middleware
// Use case: logging, CORS, rate limiting, body parsing

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();
    console.log(`[${req.method}] ${req.url} - Start`);
    
    // Khi response kết thúc
    res.on('finish', () => {
      const duration = Date.now() - start;
      console.log(`[${req.method}] ${req.url} - ${res.statusCode} - ${duration}ms`);
    });
    
    next();  // PHẢI gọi next() để tiếp tục
  }
}

// Đăng ký middleware
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .exclude({ path: 'health', method: RequestMethod.GET })  // Loại trừ
      .forRoutes('*');  // Áp dụng cho tất cả routes
  }
}
```

### 4.2 Guards

```typescript
// Guard = "Bảo vệ" → Quyết định request có được xử lý không
// Return true → tiếp tục, Return false → 403 Forbidden
// Use case: Authentication, Authorization, Role-based access

// Auth Guard (kiểm tra JWT token)
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext): boolean {
    // AuthGuard tự động verify JWT token
    return super.canActivate(context);
  }
}

// Roles Guard (kiểm tra quyền)
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    // Lấy roles từ decorator metadata
    const requiredRoles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!requiredRoles) return true;  // Không yêu cầu role → cho qua
    
    const request = context.switchToHttp().getRequest();
    const user = request.user;  // Từ JWT payload
    
    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// Custom decorator cho Roles
const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// Sử dụng
@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard)  // Áp dụng cho cả controller
export class UserController {
  
  @Delete(':id')
  @Roles('admin')  // Chỉ admin mới xóa được
  deleteUser(@Param('id') id: string) {
    return this.userService.delete(id);
  }
  
  @Get('profile')
  // Không có @Roles → mọi authenticated user đều access được
  getProfile(@Request() req) {
    return req.user;
  }
}
```

### 4.3 Interceptors

```typescript
// Interceptor = "Chặn" request/response để xử lý thêm
// Chạy TRƯỚC và SAU handler
// Use case: response transformation, logging, caching, timeout

// Response Transform Interceptor
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data: data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}
// Mọi response sẽ có format: { success: true, data: ..., timestamp: ... }

// Logging Interceptor (đo thời gian xử lý)
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const now = Date.now();
    const request = context.switchToHttp().getRequest();
    
    console.log(`[Request] ${request.method} ${request.url}`);
    
    return next.handle().pipe(
      tap(() => {
        console.log(`[Response] ${request.method} ${request.url} - ${Date.now() - now}ms`);
      }),
    );
  }
}

// Timeout Interceptor
@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),  // 5 giây timeout
      catchError(err => {
        if (err instanceof TimeoutError) {
          throw new RequestTimeoutException();
        }
        throw err;
      }),
    );
  }
}

// Cache Interceptor
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(private cacheService: CacheService) {}
  
  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = request.url;
    
    const cached = await this.cacheService.get(cacheKey);
    if (cached) {
      return of(cached);  // Trả về từ cache, KHÔNG chạy handler
    }
    
    return next.handle().pipe(
      tap(data => this.cacheService.set(cacheKey, data, 60)),  // Cache 60s
    );
  }
}
```

### 4.4 Pipes

```typescript
// Pipe = Validation & Transformation
// Chạy TRƯỚC handler, transform/validate input data
// Nếu validation fail → throw BadRequestException → 400

// Built-in Pipes:
// - ValidationPipe     → Validate DTOs (class-validator)
// - ParseIntPipe       → String → Number
// - ParseBoolPipe      → String → Boolean
// - ParseUUIDPipe      → Validate UUID format
// - DefaultValuePipe   → Cung cấp default value

// Sử dụng built-in pipes:
@Get(':id')
getUser(@Param('id', ParseIntPipe) id: number) {
  // id đã được convert từ string → number
  // Nếu không convert được → 400 Bad Request
  return this.userService.findOne(id);
}

@Get()
getUsers(@Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number) {
  // page = 1 nếu không truyền, convert sang number
}

// DTO Validation với class-validator:
// install: npm i class-validator class-transformer

// create-user.dto.ts
import { IsEmail, IsString, MinLength, IsOptional, IsEnum } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2, { message: 'Name must be at least 2 characters' })
  name: string;
  
  @IsEmail({}, { message: 'Invalid email format' })
  email: string;
  
  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  password: string;
  
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;
}

// Controller sử dụng DTO
@Post()
createUser(@Body() createUserDto: CreateUserDto) {
  // NestJS tự validate createUserDto trước khi chạy method này
  // Nếu invalid → 400 Bad Request với chi tiết lỗi
  return this.userService.create(createUserDto);
}

// Global ValidationPipe (trong main.ts)
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,       // Loại bỏ fields không có trong DTO
  forbidNonWhitelisted: true,  // Throw error nếu có fields thừa
  transform: true,       // Tự động transform types
}));
```

### 4.5 Exception Filters

```typescript
// Exception Filter = bắt và format error responses
// Chạy khi có exception được throw

// NestJS built-in exceptions:
throw new BadRequestException('Invalid input');          // 400
throw new UnauthorizedException('Please login');         // 401
throw new ForbiddenException('Access denied');           // 403
throw new NotFoundException('User not found');           // 404
throw new ConflictException('Email already exists');     // 409
throw new InternalServerErrorException('Server error');  // 500

// Custom Exception Filter
@Catch()  // Catch ALL exceptions
export class AllExceptionsFilter implements ExceptionFilter {
  constructor(private logger: LoggerService) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    let errorCode = 'INTERNAL_ERROR';
    
    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      message = typeof exceptionResponse === 'string' 
        ? exceptionResponse 
        : (exceptionResponse as any).message;
      errorCode = this.getErrorCode(status);
    }
    
    // Log error
    this.logger.error(`[${request.method}] ${request.url}`, {
      status,
      message,
      stack: exception instanceof Error ? exception.stack : undefined,
    });
    
    // Format response
    response.status(status).json({
      success: false,
      error: {
        code: errorCode,
        message: message,
        timestamp: new Date().toISOString(),
        path: request.url,
      },
    });
  }
}

// Đăng ký global
app.useGlobalFilters(new AllExceptionsFilter(logger));
```

---

## 5. DYNAMIC MODULES

### Tại sao cần Dynamic Modules?

Static modules luôn cùng cấu hình. Dynamic modules cho phép **cấu hình khác nhau** 
tùy context.

```typescript
// Static Module - cấu hình cố định
@Module({
  providers: [DatabaseService],  // Luôn connect cùng 1 database
})
export class DatabaseModule {}

// Dynamic Module - cấu hình linh hoạt
@Module({})
export class DatabaseModule {
  static forRoot(options: DatabaseOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'DATABASE_OPTIONS',
          useValue: options,
        },
        DatabaseService,
      ],
      exports: [DatabaseService],
      global: true,  // Có thể dùng ở mọi nơi
    };
  }
  
  // forFeature: đăng ký entities cho module cụ thể
  static forFeature(entities: any[]): DynamicModule {
    return {
      module: DatabaseModule,
      providers: entities.map(entity => ({
        provide: `${entity.name}_REPOSITORY`,
        useFactory: (connection: Connection) => connection.getRepository(entity),
        inject: ['DATABASE_CONNECTION'],
      })),
      exports: entities.map(entity => `${entity.name}_REPOSITORY`),
    };
  }
}

// Sử dụng:
@Module({
  imports: [
    // forRoot: cấu hình 1 lần ở AppModule
    DatabaseModule.forRoot({
      host: 'localhost',
      port: 5432,
      database: 'mydb',
    }),
  ],
})
export class AppModule {}

@Module({
  imports: [
    // forFeature: đăng ký entities cho UserModule
    DatabaseModule.forFeature([User, UserProfile]),
  ],
})
export class UserModule {}
```

### forRoot vs forFeature:

```
forRoot()    → Cấu hình TOÀN CỤC, gọi 1 lần ở AppModule
forFeature() → Cấu hình CỤC BỘ cho từng module

Ví dụ TypeORM:
- TypeOrmModule.forRoot({ ... })           → Kết nối database (1 lần)
- TypeOrmModule.forFeature([User, Post])   → Đăng ký entities (mỗi module)
```

### forRootAsync (khi cấu hình cần async):

```typescript
// Khi cần lấy config từ ConfigService (async)
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_NAME'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: false,  // KHÔNG dùng synchronize trong production!
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

---

## 6. CẤU TRÚC PROJECT NESTJS CHUẨN

```
src/
├── main.ts                          # Entry point
├── app.module.ts                    # Root module
├── app.controller.ts                # Root controller (health check)
│
├── config/                          # Configuration
│   ├── configuration.ts             # Config factory
│   ├── database.config.ts           # DB config
│   └── jwt.config.ts                # JWT config
│
├── common/                          # Shared utilities
│   ├── decorators/                  # Custom decorators
│   │   ├── roles.decorator.ts
│   │   └── current-user.decorator.ts
│   ├── guards/                      # Guards
│   │   ├── jwt-auth.guard.ts
│   │   └── roles.guard.ts
│   ├── interceptors/                # Interceptors
│   │   ├── transform.interceptor.ts
│   │   └── logging.interceptor.ts
│   ├── filters/                     # Exception filters
│   │   └── all-exceptions.filter.ts
│   ├── pipes/                       # Custom pipes
│   ├── dto/                         # Shared DTOs
│   │   └── pagination.dto.ts
│   └── interfaces/                  # Shared interfaces
│
├── modules/
│   ├── auth/                        # Auth module
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── strategies/
│   │   │   ├── jwt.strategy.ts
│   │   │   └── local.strategy.ts
│   │   └── dto/
│   │       ├── login.dto.ts
│   │       └── register.dto.ts
│   │
│   ├── users/                       # Users module
│   │   ├── users.module.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.repository.ts      # Custom repository (optional)
│   │   ├── entities/
│   │   │   └── user.entity.ts
│   │   └── dto/
│   │       ├── create-user.dto.ts
│   │       └── update-user.dto.ts
│   │
│   └── products/                    # Products module
│       ├── products.module.ts
│       ├── products.controller.ts
│       ├── products.service.ts
│       ├── entities/
│       │   └── product.entity.ts
│       └── dto/
│           ├── create-product.dto.ts
│           └── query-product.dto.ts
│
├── database/                        # Database
│   ├── migrations/                  # TypeORM migrations
│   └── seeds/                       # Seed data
│
└── test/                            # E2E tests
    ├── app.e2e-spec.ts
    └── jest-e2e.json
```

---

## 7. TESTING TRONG NESTJS

### Unit Test:

```typescript
// users.service.spec.ts
describe('UsersService', () => {
  let service: UsersService;
  let repository: MockType<Repository<User>>;
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useFactory: repositoryMockFactory,
          // Mock repository → KHÔNG cần database thật
        },
      ],
    }).compile();
    
    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
  });
  
  describe('findOne', () => {
    it('should return a user', async () => {
      const user = { id: 1, name: 'Alice', email: 'alice@test.com' };
      repository.findOne.mockReturnValue(user);
      
      expect(await service.findOne(1)).toEqual(user);
      expect(repository.findOne).toHaveBeenCalledWith({ where: { id: 1 } });
    });
    
    it('should throw NotFoundException if user not found', async () => {
      repository.findOne.mockReturnValue(null);
      
      await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
    });
  });
});
```

### E2E Test:

```typescript
// app.e2e-spec.ts
describe('UsersController (e2e)', () => {
  let app: INestApplication;
  
  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();
    
    app = moduleFixture.createNestApplication();
    app.useGlobalPipes(new ValidationPipe());
    await app.init();
  });
  
  it('/users (POST) - should create user', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ name: 'Alice', email: 'alice@test.com', password: '12345678' })
      .expect(201)
      .expect(res => {
        expect(res.body.data).toHaveProperty('id');
        expect(res.body.data.name).toBe('Alice');
      });
  });
  
  it('/users (POST) - should validate input', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ name: '', email: 'invalid' })  // Invalid data
      .expect(400);
  });
  
  afterAll(async () => {
    await app.close();
  });
});
```

---

## 8. LIFECYCLE HOOKS

```typescript
@Injectable()
export class AppService implements OnModuleInit, OnModuleDestroy, OnApplicationBootstrap {
  
  // 1. Chạy khi MODULE được khởi tạo xong
  async onModuleInit() {
    console.log('Module initialized');
    // Use case: kết nối database, load config
    await this.connectDatabase();
  }
  
  // 2. Chạy khi TẤT CẢ modules đã khởi tạo
  async onApplicationBootstrap() {
    console.log('Application bootstrapped');
    // Use case: start background jobs, warm up cache
    await this.warmUpCache();
  }
  
  // 3. Chạy khi MODULE sắp bị destroy
  async onModuleDestroy() {
    console.log('Module destroying');
    // Use case: đóng connections, cleanup resources
    await this.disconnectDatabase();
  }
  
  // 4. Chạy trước khi app shutdown (cần enableShutdownHooks())
  async onApplicationShutdown(signal?: string) {
    console.log(`App shutting down due to ${signal}`);
    // Use case: graceful shutdown
  }
}

// Thứ tự lifecycle:
// onModuleInit → onApplicationBootstrap → [APP RUNNING] → onModuleDestroy → onApplicationShutdown
```

---

## TÓM TẮT PHẦN 3 - NESTJS

### Top câu hỏi phỏng vấn & cách trả lời:

**1. "Giải thích Dependency Injection trong NestJS?"**
→ DI là pattern mà class khai báo dependencies trong constructor, NestJS IoC container 
tự động tạo và inject. Giúp loose coupling, dễ test, dễ maintain.

**2. "Request lifecycle trong NestJS?"**
→ Middleware → Guards → Interceptors (before) → Pipes → Handler → Interceptors (after) 
→ Exception Filters. Vẽ diagram.

**3. "Middleware vs Guard vs Interceptor vs Pipe?"**
→ Middleware: generic (logging, cors). Guard: yes/no access. 
Interceptor: transform request/response. Pipe: validate/transform input.

**4. "forRoot() vs forFeature()?"**
→ forRoot: cấu hình global 1 lần. forFeature: đăng ký cụ thể cho từng module.

**5. "Provider scopes?"**
→ DEFAULT (singleton), REQUEST (mới mỗi request), TRANSIENT (mới mỗi injection).
Phần lớn dùng DEFAULT. REQUEST dùng cho multi-tenancy.

**6. "Bạn tổ chức NestJS project thế nào?"**
→ Module-based: mỗi feature là 1 module (controller + service + DTOs + entities).
Common module cho shared utilities.
