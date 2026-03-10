# PHẦN 5: DATABASE (TypeORM, PostgreSQL, MongoDB, Redis) - GIẢI THÍCH CHI TIẾT

> **Mức độ ưu tiên: CAO**
> Mọi ứng dụng đều cần database. Trong JD yêu cầu: TypeORM, PostgreSQL, MongoDB, Redis.
> Đây là phần rất thực tế, interviewer hay hỏi về query optimization, schema design, 
> và khi nào dùng SQL vs NoSQL.

---

## 1. TYPEORM - ORM CHO TYPESCRIPT

### ORM là gì?

**ORM (Object-Relational Mapping)** = Mapping giữa objects trong code và tables trong database.
Thay vì viết SQL raw, bạn làm việc với objects và TypeORM tự generate SQL.

```
TypeScript Object    ←→    Database Table
─────────────────          ─────────────────
class User {               CREATE TABLE users (
  id: number                 id SERIAL PRIMARY KEY,
  name: string               name VARCHAR,
  email: string              email VARCHAR,
  createdAt: Date            created_at TIMESTAMP
}                          )
```

### Entity (Bảng trong database):

```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, 
         UpdateDateColumn, DeleteDateColumn } from 'typeorm';

@Entity('users')  // Tên bảng: 'users'
export class User {
  @PrimaryGeneratedColumn('uuid')  // Auto-generate UUID
  id: string;
  
  // Hoặc: @PrimaryGeneratedColumn()  → auto-increment integer
  
  @Column({ type: 'varchar', length: 100 })
  name: string;
  
  @Column({ unique: true })  // Unique constraint
  email: string;
  
  @Column({ type: 'varchar', nullable: true })  // Cho phép NULL
  avatar: string | null;
  
  @Column({ type: 'enum', enum: ['active', 'inactive'], default: 'active' })
  status: string;
  
  @Column({ type: 'jsonb', nullable: true })  // PostgreSQL JSONB
  metadata: Record<string, any>;
  
  @Column({ select: false })  // Không trả về khi query (cho password)
  password: string;
  
  @CreateDateColumn()  // Tự gán khi INSERT
  createdAt: Date;
  
  @UpdateDateColumn()  // Tự cập nhật khi UPDATE
  updatedAt: Date;
  
  @DeleteDateColumn()  // Soft delete (đánh dấu xóa, không xóa thật)
  deletedAt: Date | null;
}
```

### Relations (Quan hệ giữa các bảng):

```typescript
// ════════════════════════════════════
// ONE-TO-ONE: 1 User có 1 Profile
// ════════════════════════════════════

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true,     // Khi save User → tự save Profile
    eager: false,       // KHÔNG tự load Profile khi query User
  })
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  bio: string;
  
  @OneToOne(() => User, user => user.profile)
  @JoinColumn()  // ← BÊN NÀO có @JoinColumn → bảng đó có foreign key
  user: User;
}

// ════════════════════════════════════
// ONE-TO-MANY / MANY-TO-ONE: 1 User có nhiều Posts
// ════════════════════════════════════

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @OneToMany(() => Post, post => post.author)
  posts: Post[];  // 1 User → nhiều Posts
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  title: string;
  
  @ManyToOne(() => User, user => user.posts, {
    onDelete: 'CASCADE',  // Xóa User → xóa luôn Posts
  })
  @JoinColumn({ name: 'author_id' })  // FK column name
  author: User;
  
  @Column()
  authorId: number;  // Explicit FK column (để query dễ hơn)
}

// ════════════════════════════════════
// MANY-TO-MANY: Posts có nhiều Tags, Tags thuộc nhiều Posts
// ════════════════════════════════════

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;
  
  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable({  // Tạo bảng trung gian: post_tags
    name: 'post_tags',
    joinColumn: { name: 'post_id' },
    inverseJoinColumn: { name: 'tag_id' },
  })
  tags: Tag[];
}

@Entity()
export class Tag {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @ManyToMany(() => Post, post => post.tags)
  posts: Post[];
}
```

### Eager vs Lazy Loading:

```typescript
// EAGER Loading: Tự động load relations khi query
@OneToMany(() => Post, post => post.author, { eager: true })
posts: Post[];

const user = await userRepo.findOne({ where: { id: 1 } });
console.log(user.posts);  // ✅ Đã có sẵn (auto JOIN)

// LAZY Loading: Chỉ load khi access (cần Promise)
@OneToMany(() => Post, post => post.author, { eager: false })
posts: Promise<Post[]>;  // ← Phải là Promise

const user = await userRepo.findOne({ where: { id: 1 } });
const posts = await user.posts;  // ← Query thêm lúc này

// EXPLICIT Loading (recommended): Chỉ load khi cần, qua find options
const user = await userRepo.findOne({ 
  where: { id: 1 },
  relations: ['posts', 'posts.comments'],  // Load relations cụ thể
});
```

### Repository Pattern - CRUD Operations:

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepo: Repository<User>,
  ) {}
  
  // ═══ FIND ═══
  async findAll(query: QueryDto) {
    return this.userRepo.find({
      where: { status: 'active' },
      relations: ['profile'],
      select: ['id', 'name', 'email'],   // Chỉ lấy các cột cần
      order: { createdAt: 'DESC' },
      skip: (query.page - 1) * query.limit,  // Pagination
      take: query.limit,
    });
  }
  
  async findOne(id: number) {
    const user = await this.userRepo.findOne({
      where: { id },
      relations: ['profile', 'posts'],
    });
    if (!user) throw new NotFoundException(`User #${id} not found`);
    return user;
  }
  
  // ═══ CREATE ═══
  async create(dto: CreateUserDto) {
    const user = this.userRepo.create(dto);  // Tạo instance (chưa save)
    return this.userRepo.save(user);          // INSERT vào DB
  }
  
  // ═══ UPDATE ═══
  async update(id: number, dto: UpdateUserDto) {
    // Cách 1: Load → modify → save (triggers hooks, cascade)
    const user = await this.findOne(id);
    Object.assign(user, dto);
    return this.userRepo.save(user);
    
    // Cách 2: Direct update (nhanh hơn, không trigger hooks)
    // await this.userRepo.update(id, dto);
  }
  
  // ═══ DELETE ═══
  async remove(id: number) {
    const user = await this.findOne(id);
    return this.userRepo.remove(user);
    
    // Soft delete (nếu dùng @DeleteDateColumn)
    // return this.userRepo.softRemove(user);
    // hoặc: this.userRepo.softDelete(id);
  }
}
```

### QueryBuilder (Complex Queries):

```typescript
// Khi find() không đủ → dùng QueryBuilder

// Ví dụ: Search + Filter + Pagination + Sorting
async searchUsers(query: SearchQueryDto) {
  const qb = this.userRepo.createQueryBuilder('user')
    .leftJoinAndSelect('user.profile', 'profile')
    .leftJoin('user.posts', 'post');
  
  // Dynamic filters
  if (query.search) {
    qb.where('user.name ILIKE :search OR user.email ILIKE :search', {
      search: `%${query.search}%`,
    });
  }
  
  if (query.status) {
    qb.andWhere('user.status = :status', { status: query.status });
  }
  
  if (query.minPosts) {
    qb.addGroupBy('user.id')
       .having('COUNT(post.id) >= :minPosts', { minPosts: query.minPosts });
  }
  
  // Sorting
  qb.orderBy(`user.${query.sortBy || 'createdAt'}`, query.sortOrder || 'DESC');
  
  // Pagination
  qb.skip((query.page - 1) * query.limit).take(query.limit);
  
  const [users, total] = await qb.getManyAndCount();
  
  return {
    data: users,
    meta: {
      page: query.page,
      limit: query.limit,
      total,
      totalPages: Math.ceil(total / query.limit),
    },
  };
}

// Subquery example
const qb = this.userRepo.createQueryBuilder('user')
  .where(qb => {
    const subQuery = qb.subQuery()
      .select('post.authorId')
      .from(Post, 'post')
      .where('post.published = :published', { published: true })
      .getQuery();
    return `user.id IN ${subQuery}`;
  });
```

### Transactions:

```typescript
// Transaction = Nhóm nhiều operations, TẤT CẢ thành công hoặc TẤT CẢ rollback

// Cách 1: QueryRunner (recommended - full control)
async transferMoney(fromId: number, toId: number, amount: number) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    // Trừ tiền người gửi
    await queryRunner.manager.decrement(Account, { id: fromId }, 'balance', amount);
    
    // Cộng tiền người nhận
    await queryRunner.manager.increment(Account, { id: toId }, 'balance', amount);
    
    // Tạo transaction record
    await queryRunner.manager.save(Transaction, { fromId, toId, amount });
    
    await queryRunner.commitTransaction();
  } catch (err) {
    await queryRunner.rollbackTransaction();  // Rollback NẾU có lỗi
    throw err;
  } finally {
    await queryRunner.release();
  }
}

// Cách 2: dataSource.transaction (đơn giản hơn)
async createOrderWithItems(orderDto: CreateOrderDto) {
  return this.dataSource.transaction(async (manager) => {
    const order = manager.create(Order, orderDto);
    await manager.save(order);
    
    const items = orderDto.items.map(item => 
      manager.create(OrderItem, { ...item, orderId: order.id })
    );
    await manager.save(items);
    
    return order;
  });
}
```

### Migrations:

```bash
# Tại sao cần migrations thay vì synchronize?
# synchronize: true → Tự động sync schema (CHỈ dùng dev)
# Production: PHẢI dùng migrations (an toàn, có version control)

# Generate migration từ thay đổi entity
npx typeorm migration:generate src/database/migrations/AddUserAvatar -d src/data-source.ts

# Tạo migration empty
npx typeorm migration:create src/database/migrations/SeedDefaultRoles

# Run migrations
npx typeorm migration:run -d src/data-source.ts

# Revert migration gần nhất
npx typeorm migration:revert -d src/data-source.ts
```

```typescript
// Migration file
export class AddUserAvatar1234567890 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn('users', new TableColumn({
      name: 'avatar',
      type: 'varchar',
      isNullable: true,
    }));
    
    // Hoặc raw SQL
    await queryRunner.query(`
      CREATE INDEX idx_users_email ON users(email);
    `);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('users', 'avatar');
    await queryRunner.query(`DROP INDEX idx_users_email;`);
  }
}
```

### N+1 Problem (HAY HỎI PHỎNG VẤN):

```typescript
// ❌ N+1 PROBLEM:
const users = await userRepo.find();  // 1 query: SELECT * FROM users
for (const user of users) {
  const posts = await postRepo.find({ where: { authorId: user.id } });
  // N queries: SELECT * FROM posts WHERE author_id = ?
}
// Tổng: 1 + N queries (N = số users) → RẤT CHẬM

// ✅ FIX 1: Eager loading / relations
const users = await userRepo.find({ relations: ['posts'] });
// 1 query với JOIN hoặc 2 queries (1 SELECT users + 1 SELECT posts WHERE id IN (...))

// ✅ FIX 2: QueryBuilder với JOIN
const users = await userRepo.createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// 1 query: SELECT ... FROM users LEFT JOIN posts ...

// ✅ FIX 3: DataLoader pattern (cho GraphQL)
```

---

## 2. POSTGRESQL

### Tại sao PostgreSQL?

PostgreSQL = database quan hệ **mạnh nhất** trong open source:
- ACID compliant (reliability)
- JSON/JSONB support (linh hoạt như NoSQL)
- Full-text search
- Window functions
- Materialized views
- Extensions (PostGIS cho geo, pg_trgm cho fuzzy search)

### SQL Fundamentals cần nắm vững:

```sql
-- ═══ JOINs (CỰC QUAN TRỌNG) ═══

-- INNER JOIN: Chỉ lấy rows có match ở CẢ HAI bảng
SELECT u.name, p.title 
FROM users u 
INNER JOIN posts p ON u.id = p.author_id;
-- User không có post → KHÔNG xuất hiện
-- Post không có author → KHÔNG xuất hiện

-- LEFT JOIN: Lấy TẤT CẢ rows từ bảng TRÁI, match hoặc NULL từ bảng phải
SELECT u.name, p.title 
FROM users u 
LEFT JOIN posts p ON u.id = p.author_id;
-- User không có post → xuất hiện với p.title = NULL

-- RIGHT JOIN: Ngược lại LEFT JOIN
-- FULL JOIN: Tất cả rows từ cả hai bảng

-- ═══ Aggregation ═══
SELECT 
  u.name,
  COUNT(p.id) as post_count,
  MAX(p.created_at) as latest_post
FROM users u
LEFT JOIN posts p ON u.id = p.author_id
GROUP BY u.id, u.name              -- Group theo user
HAVING COUNT(p.id) > 5             -- Lọc groups (>5 posts)
ORDER BY post_count DESC
LIMIT 10;

-- ═══ Subquery ═══
-- Users có nhiều posts nhất
SELECT * FROM users 
WHERE id = (
  SELECT author_id FROM posts 
  GROUP BY author_id 
  ORDER BY COUNT(*) DESC 
  LIMIT 1
);

-- ═══ CTE (Common Table Expression) - WITH clause ═══
-- Đọc dễ hơn subquery, tái sử dụng được
WITH user_post_counts AS (
  SELECT author_id, COUNT(*) as cnt
  FROM posts
  GROUP BY author_id
)
SELECT u.name, upc.cnt
FROM users u
JOIN user_post_counts upc ON u.id = upc.author_id
WHERE upc.cnt > 10;

-- ═══ Window Functions (nâng cao, hay hỏi) ═══
-- ROW_NUMBER: đánh số thứ tự
SELECT name, salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- RANK: đánh rank (có thể trùng)
SELECT name, department, salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;
-- PARTITION BY = GROUP BY cho window function

-- LAG / LEAD: lấy giá trị row trước/sau
SELECT date, revenue,
  revenue - LAG(revenue) OVER (ORDER BY date) as daily_change
FROM daily_sales;
```

### PostgreSQL Specific Features:

```sql
-- ═══ JSONB (rất hay dùng) ═══
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  attributes JSONB  -- Flexible schema trong relational DB
);

INSERT INTO products (name, attributes) VALUES
('iPhone', '{"color": "black", "storage": 128, "specs": {"ram": 6, "cpu": "A15"}}');

-- Query JSONB
SELECT * FROM products WHERE attributes->>'color' = 'black';
SELECT * FROM products WHERE attributes->'specs'->>'ram' = '6';
SELECT * FROM products WHERE attributes @> '{"color": "black"}';  -- Contains

-- Index cho JSONB
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);

-- ═══ Full-text Search ═══
-- Tìm kiếm text hiệu quả hơn LIKE
SELECT * FROM posts 
WHERE to_tsvector('english', title || ' ' || content) 
      @@ to_tsquery('english', 'javascript & framework');

-- ═══ Array Type ═══
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  tags TEXT[]  -- Array column
);
SELECT * FROM users WHERE 'developer' = ANY(tags);
```

### Indexing (QUAN TRỌNG cho performance):

```sql
-- Index = "mục lục" cho database, giúp tìm kiếm nhanh hơn
-- Không có index: Full Table Scan → O(n) - chậm
-- Có index: Index Scan → O(log n) - nhanh

-- B-tree index (mặc định, phổ biến nhất)
CREATE INDEX idx_users_email ON users(email);
-- Tốt cho: =, <, >, <=, >=, BETWEEN, IN, LIKE 'abc%'

-- Composite index (nhiều cột)
CREATE INDEX idx_users_status_created ON users(status, created_at);
-- Tốt cho: WHERE status = 'active' AND created_at > '2024-01-01'
-- ⚠️ Thứ tự cột QUAN TRỌNG: (status, created_at) ≠ (created_at, status)
-- Rule: cột có selectivity cao (ít giá trị unique) đặt trước

-- GIN index (cho JSONB, array, full-text)
CREATE INDEX idx_products_attrs ON products USING GIN(attributes);

-- Partial index (index chỉ 1 phần dữ liệu)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
-- Nhỏ hơn full index, nhanh hơn nếu phần lớn query lọc active

-- ═══ KHI NÀO KHÔNG NÊN INDEX ═══
-- 1. Bảng nhỏ (< vài nghìn rows) → Full scan nhanh hơn
-- 2. Cột ít query → Tốn disk, chậm INSERT/UPDATE
-- 3. Cột có ít giá trị unique (boolean, status) → Ít hiệu quả
-- 4. Bảng INSERT/UPDATE nhiều → Index chậm write operations

-- ═══ EXPLAIN ANALYZE ═══
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@test.com';
-- Seq Scan on users  (cost=0.00..1234.00 rows=1 width=100) (actual time=50ms)
-- → Seq Scan = CHẬM, cần index!

-- Sau khi thêm index:
-- Index Scan using idx_users_email on users  (cost=0.00..8.27 rows=1) (actual time=0.05ms)
-- → Index Scan = NHANH!
```

---

## 3. MONGODB

### MongoDB vs PostgreSQL:

```
PostgreSQL (SQL - Relational):         MongoDB (NoSQL - Document):
─────────────────────────              ────────────────────────
- Bảng (tables)                        - Collections  
- Dòng (rows)                          - Documents (BSON/JSON)
- Schema cố định                       - Schema linh hoạt
- ACID                                 - Eventual consistency (có thể ACID)
- JOINs mạnh                          - Embedding / $lookup
- Mature, reliable                     - Horizontal scaling dễ

USE CASE:                              USE CASE:
- Financial transactions               - Content management
- Complex relationships                - IoT, logging
- Reports, analytics                   - Real-time analytics
- Khi cần ACID reliability            - Schema thay đổi thường xuyên
- Khi cần JOINs phức tạp             - Horizontal scaling
```

### Schema Design Patterns:

```javascript
// ═══ EMBEDDING (Nhúng document con vào document cha) ═══
// Tốt khi: dữ liệu luôn đọc cùng nhau, 1-to-few relationship

// Thay vì 2 collections (users + addresses):
{
  _id: ObjectId("..."),
  name: "Alice",
  email: "alice@test.com",
  addresses: [  // Nhúng trực tiếp
    { street: "123 Main St", city: "HCM", primary: true },
    { street: "456 Oak Ave", city: "HN", primary: false }
  ]
}

// ═══ REFERENCING (Tham chiếu qua ID) ═══
// Tốt khi: many-to-many, dữ liệu lớn, cần update riêng

// Collection: users
{ _id: ObjectId("user1"), name: "Alice" }

// Collection: posts
{ _id: ObjectId("post1"), title: "Hello", authorId: ObjectId("user1") }

// ═══ KHI NÀO EMBEDDING vs REFERENCING? ═══
// Embedding:
//   ✅ Read cùng nhau (1 query)
//   ✅ Atomic updates (1 document)
//   ❌ Document size limit (16MB)
//   ❌ Data duplication nếu share giữa nhiều documents

// Referencing:
//   ✅ Không duplicate data
//   ✅ Không lo document size
//   ❌ Cần $lookup (JOIN) → chậm hơn
//   ❌ Không atomic across documents
```

### Aggregation Pipeline:

```javascript
// Aggregation = pipeline xử lý documents qua nhiều stages
// Giống SELECT... FROM... WHERE... GROUP BY... ORDER BY... trong SQL

db.orders.aggregate([
  // Stage 1: Filter (WHERE)
  { $match: { status: "completed", createdAt: { $gte: new Date("2024-01-01") } } },
  
  // Stage 2: Join (LEFT JOIN)
  { $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
  }},
  
  // Stage 3: Unwind array
  { $unwind: "$user" },
  
  // Stage 4: Group (GROUP BY)
  { $group: {
      _id: "$user.name",
      totalOrders: { $sum: 1 },
      totalRevenue: { $sum: "$amount" },
      avgOrderValue: { $avg: "$amount" }
  }},
  
  // Stage 5: Sort (ORDER BY)
  { $sort: { totalRevenue: -1 } },
  
  // Stage 6: Limit
  { $limit: 10 }
]);
```

---

## 4. REDIS

### Redis là gì?

Redis = **In-memory data store**. Dữ liệu lưu trong RAM → CỰC NHANH (microseconds).
Dùng như: Cache, Session store, Message broker, Rate limiter, Leaderboard.

### Data Structures:

```
┌──────────────────────────────────────────────────┐
│                 REDIS DATA TYPES                  │
├──────────────┬───────────────────────────────────┤
│ String       │ "hello" , "42", binary data       │
│              │ Use: cache, counters, sessions     │
├──────────────┼───────────────────────────────────┤
│ List         │ [a, b, c, d]                      │
│              │ Use: message queue, activity feed  │
├──────────────┼───────────────────────────────────┤
│ Set          │ {a, b, c} (unique)                │
│              │ Use: tags, unique visitors          │
├──────────────┼───────────────────────────────────┤
│ Sorted Set   │ {a:1.0, b:2.5, c:3.0} (scored)   │
│ (ZSet)       │ Use: leaderboard, ranking          │
├──────────────┼───────────────────────────────────┤
│ Hash         │ {field1: val1, field2: val2}       │
│              │ Use: object storage, user profiles  │
├──────────────┼───────────────────────────────────┤
│ Stream       │ Append-only log                    │
│              │ Use: event sourcing, message queue  │
└──────────────┴───────────────────────────────────┘
```

### Caching Strategies:

```
═══ CACHE-ASIDE (Lazy Loading) - PHỔ BIẾN NHẤT ═══

  Client → Cache (Redis)
            │
            ├── HIT → Return data (nhanh!)
            │
            └── MISS → Query Database → Store in Cache → Return data

  Flow:
  1. App kiểm tra cache trước
  2. Cache HIT → trả về ngay
  3. Cache MISS → query DB → lưu vào cache → trả về
  
  ✅ Ưu: Chỉ cache data thực sự cần
  ❌ Nhược: Cache miss = chậm hơn (phải query DB rồi write cache)


═══ WRITE-THROUGH ═══

  Client → Write to Cache → Cache writes to Database
  
  1. Mỗi lần write → update cache VÀ database đồng thời
  
  ✅ Ưu: Cache luôn consistent với DB
  ❌ Nhược: Write operations chậm hơn (phải write 2 nơi)


═══ WRITE-BEHIND (Write-Back) ═══

  Client → Write to Cache → (Async) Cache writes to Database later
  
  1. Write chỉ vào cache (nhanh!)
  2. Cache async sync xuống DB theo batch/interval
  
  ✅ Ưu: Write cực nhanh
  ❌ Nhược: Có thể mất data nếu cache crash trước khi sync
```

### Redis trong NestJS:

```typescript
// Cài đặt: npm i @nestjs/cache-manager cache-manager cache-manager-redis-yet

// app.module.ts
@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        store: redisStore,
        host: config.get('REDIS_HOST'),
        port: config.get('REDIS_PORT'),
        ttl: 60,  // 60 seconds default TTL
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}

// Sử dụng trong Service
@Injectable()
export class UsersService {
  constructor(
    @Inject(CACHE_MANAGER) private cacheManager: Cache,
    @InjectRepository(User) private userRepo: Repository<User>,
  ) {}
  
  async findOne(id: number): Promise<User> {
    const cacheKey = `user:${id}`;
    
    // 1. Check cache
    const cached = await this.cacheManager.get<User>(cacheKey);
    if (cached) return cached;  // Cache HIT
    
    // 2. Query DB
    const user = await this.userRepo.findOne({ where: { id } });
    if (!user) throw new NotFoundException();
    
    // 3. Store in cache (TTL: 5 phút)
    await this.cacheManager.set(cacheKey, user, 300);
    
    return user;
  }
  
  async update(id: number, dto: UpdateUserDto) {
    await this.userRepo.update(id, dto);
    
    // INVALIDATE cache khi data thay đổi
    await this.cacheManager.del(`user:${id}`);
    
    return this.findOne(id);
  }
}

// Hoặc dùng Cache Interceptor (tự động cache GET responses)
@Controller('users')
@UseInterceptors(CacheInterceptor)
export class UsersController {
  @Get(':id')
  @CacheTTL(300)  // 5 phút
  @CacheKey('user')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);
  }
}
```

### Redis cho Queue (Bull):

```typescript
// Cài đặt: npm i @nestjs/bull bull
// Use case: async jobs, email sending, image processing

// app.module.ts
@Module({
  imports: [
    BullModule.forRoot({
      redis: { host: 'localhost', port: 6379 },
    }),
    BullModule.registerQueue({ name: 'email' }),
  ],
})

// Producer: thêm job vào queue
@Injectable()
export class NotificationService {
  constructor(@InjectQueue('email') private emailQueue: Queue) {}
  
  async sendWelcomeEmail(user: User) {
    await this.emailQueue.add('welcome', {
      to: user.email,
      name: user.name,
    }, {
      delay: 5000,       // Delay 5s trước khi xử lý
      attempts: 3,        // Retry 3 lần nếu fail
      backoff: { type: 'exponential', delay: 2000 },
    });
  }
}

// Consumer: xử lý jobs từ queue
@Processor('email')
export class EmailProcessor {
  @Process('welcome')
  async handleWelcome(job: Job<{ to: string; name: string }>) {
    const { to, name } = job.data;
    await sendEmail(to, `Welcome ${name}!`);
  }
  
  @OnQueueFailed()
  onFailed(job: Job, error: Error) {
    console.error(`Job ${job.id} failed:`, error);
  }
}
```

---

## TÓM TẮT PHẦN 5 - DATABASE

### Top câu hỏi phỏng vấn:

**1. "N+1 problem là gì? Cách giải quyết?"**
→ Query 1 lấy list, rồi N query cho từng item. Fix: eager loading/JOIN, 
QueryBuilder với leftJoinAndSelect, DataLoader.

**2. "SQL vs NoSQL, khi nào dùng gì?"**
→ SQL: complex relationships, ACID, reports. NoSQL: flexible schema, 
horizontal scaling, real-time. PostgreSQL có JSONB nên khá flexible.

**3. "Redis dùng để làm gì?"**
→ Caching (cache-aside), Session storage, Rate limiting, Queue (Bull), 
Leaderboard (Sorted Set), Pub/Sub.

**4. "ACID là gì?"**
→ Atomicity (tất cả hoặc không), Consistency (dữ liệu luôn valid),
Isolation (transactions không ảnh hưởng nhau), Durability (committed = không mất).

**5. "Indexing hoạt động thế nào?"**
→ Index = mục lục, B-tree structure. Biến O(n) scan → O(log n) lookup.
Trade-off: nhanh READ nhưng chậm WRITE. Dùng EXPLAIN ANALYZE để verify.

**6. "TypeORM migration workflow?"**
→ Sửa entity → generate migration → review SQL → run migration.
Không bao giờ dùng synchronize: true trong production.

**7. "Eager vs Lazy loading?"**
→ Eager: auto load relations (có thể load thừa). Lazy: load khi access (có thể N+1).
Best practice: explicit loading qua find options hoặc QueryBuilder.
