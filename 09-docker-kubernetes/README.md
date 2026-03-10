# PHẦN 9: DOCKER & KUBERNETES - GIẢI THÍCH CHI TIẾT

> **Mức độ ưu tiên: TRUNG BÌNH**
> JD yêu cầu "Hiểu biết về container hóa và công cụ điều phối như Docker và Kubernetes".
> Không cần expert, nhưng phải hiểu concepts và dùng được trong thực tế.

---

## 1. DOCKER LÀ GÌ?

### Vấn đề Docker giải quyết:

```
❌ KHÔNG có Docker:
Developer A: "Code chạy ngon trên máy tôi!"
Developer B: "Ở máy tôi lỗi! Node version khác, thiếu dependencies..."
Deploy server: "Production crash! OS khác, config khác..."
→ "Works on my machine" problem

✅ CÓ Docker:
Đóng gói app + dependencies + environment vào CONTAINER
→ Chạy GIỐNG NHAU trên mọi máy (dev, staging, production)
→ Isolated, portable, reproducible
```

### Container vs Virtual Machine:

```
Virtual Machine:                     Container:
┌───────────────────┐               ┌───────────────────┐
│ App A  │  App B   │               │ App A  │  App B   │
├────────┼──────────┤               ├────────┼──────────┤
│ Libs A │  Libs B  │               │ Libs A │  Libs B  │
├────────┼──────────┤               ├────────┴──────────┤
│ Guest  │  Guest   │               │   Container       │
│  OS    │   OS     │               │   Runtime         │
├────────┴──────────┤               │   (Docker Engine)  │
│   Hypervisor      │               ├───────────────────┤
├───────────────────┤               │   Host OS         │
│   Host OS         │               ├───────────────────┤
├───────────────────┤               │   Hardware        │
│   Hardware        │               └───────────────────┘
└───────────────────┘               
                                    
NẶNG: GB, boot phút                NHẸ: MB, boot giây
Mỗi VM = full OS                   Share Host OS kernel
Isolate hoàn toàn                   Process-level isolation
```

### Image vs Container:

```
IMAGE = "Bản thiết kế" (template, read-only)
- Chứa: OS base, dependencies, app code, config
- Build 1 lần, dùng nhiều lần
- Lưu trong Registry (Docker Hub, AWS ECR)

CONTAINER = "Instance chạy" từ Image
- Container = Image + read-write layer
- Có thể tạo nhiều containers từ 1 image
- Isolated environment

Tương tự:
Image = Class (blueprint)
Container = Object (instance)
```

---

## 2. DOCKERFILE

### Dockerfile = "Công thức" để build Image:

```dockerfile
# ═══ BASIC DOCKERFILE cho NestJS ═══

# 1. Base image (bắt đầu từ đâu)
FROM node:18-alpine
#      ↑          ↑
#   image name   tag (alpine = nhẹ, ~50MB thay vì ~900MB)

# 2. Working directory trong container
WORKDIR /app

# 3. Copy package files TRƯỚC (tận dụng layer caching)
COPY package*.json ./

# 4. Install dependencies
RUN npm ci
# ci = clean install (nhanh hơn, deterministic, dùng cho CI/CD)

# 5. Copy source code
COPY . .

# 6. Build
RUN npm run build

# 7. Expose port (documentation, không thật sự mở port)
EXPOSE 3000

# 8. Command chạy khi container start
CMD ["node", "dist/main.js"]
```

### Multi-stage Build (QUAN TRỌNG):

```dockerfile
# Tại sao Multi-stage?
# Build cần: TypeScript compiler, devDependencies, source code
# Production chỉ cần: Node.js + compiled JS + production dependencies
# → Multi-stage: build ở stage 1, copy KẾT QUẢ sang stage 2 (nhẹ hơn nhiều)

# ═══ Stage 1: BUILD ═══
FROM node:18-alpine AS builder
WORKDIR /app

# Copy & install ALL dependencies (including devDependencies)
COPY package*.json ./
RUN npm ci

# Copy source & build
COPY . .
RUN npm run build

# ═══ Stage 2: PRODUCTION ═══
FROM node:18-alpine AS production
WORKDIR /app

# Chỉ copy production dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy compiled code từ builder stage
COPY --from=builder /app/dist ./dist

# Security: Không chạy bằng root
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nestjs -u 1001
USER nestjs

EXPOSE 3000
CMD ["node", "dist/main.js"]

# Image size comparison:
# Không multi-stage: ~800MB (có TypeScript, devDeps, source code)
# Multi-stage:       ~200MB (chỉ runtime cần thiết)
```

### Dockerfile Instructions:

```dockerfile
FROM       # Base image
WORKDIR    # Set working directory
COPY       # Copy files từ host → container
ADD        # Giống COPY + tự extract tar, download URL (ít dùng)
RUN        # Chạy command lúc BUILD (install deps, compile)
CMD        # Command chạy lúc CONTAINER START (1 CMD cuối cùng có hiệu lực)
ENTRYPOINT # Giống CMD nhưng KHÔNG bị override bởi docker run args
EXPOSE     # Document port (không thật sự mở)
ENV        # Set environment variable
ARG        # Build-time variable (chỉ dùng trong Dockerfile)
VOLUME     # Mount point cho persistent data
```

### CMD vs ENTRYPOINT:

```dockerfile
# CMD: Có thể bị OVERRIDE khi docker run
CMD ["node", "dist/main.js"]
# docker run myapp             → node dist/main.js
# docker run myapp npm test    → npm test (OVERRIDE CMD)

# ENTRYPOINT: KHÔNG bị override (args được append)
ENTRYPOINT ["node"]
CMD ["dist/main.js"]
# docker run myapp                  → node dist/main.js
# docker run myapp dist/worker.js   → node dist/worker.js (CMD bị override, ENTRYPOINT giữ)

# Khi nào dùng gì?
# CMD: linh hoạt, cho phép override (phần lớn trường hợp)
# ENTRYPOINT: khi container = 1 executable cố định
```

### .dockerignore:

```
# Giống .gitignore - bỏ qua files không cần copy vào image
node_modules
dist
.git
.env
*.md
.vscode
coverage
```

### Layer Caching (QUAN TRỌNG cho build speed):

```dockerfile
# Docker cache mỗi layer (instruction). Nếu layer không đổi → dùng cache.
# Nếu 1 layer thay đổi → tất cả layers SAU đó phải rebuild.

# ❌ BAD: Mỗi lần sửa code → install lại dependencies
COPY . .
RUN npm ci

# ✅ GOOD: Dependencies chỉ install lại khi package.json thay đổi
COPY package*.json ./     # Layer 1: package files (ít thay đổi)
RUN npm ci                # Layer 2: install (cached nếu Layer 1 không đổi)
COPY . .                  # Layer 3: source code (thay đổi thường xuyên)
RUN npm run build         # Layer 4: build (rebuild khi code thay đổi)

# Kết quả:
# Sửa code → chỉ rebuild Layer 3+4, SKIP Layer 1+2 (tiết kiệm phút → giây)
```

---

## 3. DOCKER COMPOSE

### Docker Compose = Orchestrate MULTIPLE containers:

```yaml
# docker-compose.yml
# Một command (docker compose up) → start toàn bộ stack

version: '3.8'

services:
  # ═══ NestJS Application ═══
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"           # host:container
    environment:
      - NODE_ENV=production
      - DB_HOST=db            # Tên service = hostname trong Docker network
      - DB_PORT=5432
      - DB_DATABASE=mydb
      - DB_USERNAME=postgres
      - DB_PASSWORD=secret
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      db:
        condition: service_healthy  # Chờ DB healthy mới start app
      redis:
        condition: service_started
    restart: unless-stopped       # Auto restart nếu crash

  # ═══ PostgreSQL Database ═══
  db:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data  # Persistent data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ═══ Redis Cache ═══
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data

  # ═══ pgAdmin (optional - DB management UI) ═══
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin

# Named volumes (persistent data, survive container restart)
volumes:
  pgdata:
  redisdata:
```

### Docker Compose Commands:

```bash
docker compose up                  # Start all services
docker compose up -d               # Start in background (detached)
docker compose up --build          # Rebuild images & start
docker compose down                # Stop & remove containers
docker compose down -v             # Stop + remove volumes (LOSE DATA)
docker compose logs app            # Xem logs của service 'app'
docker compose logs -f             # Follow logs (real-time)
docker compose exec app sh         # Shell vào container 'app'
docker compose ps                  # Liệt kê containers
docker compose restart app         # Restart 1 service
```

---

## 4. DOCKER COMMANDS CẦN NHỚ

```bash
# ═══ IMAGE ═══
docker build -t myapp:1.0 .       # Build image từ Dockerfile
docker images                      # Liệt kê images
docker pull node:18-alpine         # Download image từ registry
docker push myrepo/myapp:1.0      # Push image lên registry
docker rmi myapp:1.0              # Xóa image

# ═══ CONTAINER ═══
docker run -d -p 3000:3000 myapp   # Chạy container (detached)
docker run -it myapp sh            # Chạy interactive shell
docker ps                          # Liệt kê running containers
docker ps -a                       # Tất cả containers (kể cả stopped)
docker stop <container_id>         # Stop container
docker start <container_id>        # Start stopped container
docker rm <container_id>           # Xóa container
docker logs <container_id>         # Xem logs
docker exec -it <container_id> sh  # Vào shell container đang chạy

# ═══ VOLUME ═══
docker volume ls                   # Liệt kê volumes
docker volume create mydata        # Tạo volume
docker volume rm mydata            # Xóa volume

# ═══ NETWORK ═══
docker network ls                  # Liệt kê networks
docker network create mynet        # Tạo network
```

---

## 5. KUBERNETES (K8s) - BASICS

### Kubernetes là gì?

Kubernetes = **Container Orchestration Platform**
- Docker: chạy 1 container
- Kubernetes: quản lý HÀNG NGHÌN containers trên NHIỀU servers

```
Kubernetes giải quyết:
1. Scaling: Tự động tăng/giảm số container theo load
2. Load balancing: Phân tải đều giữa containers
3. Self-healing: Container crash → tự restart
4. Rolling updates: Deploy version mới không downtime
5. Service discovery: Containers tìm thấy nhau
```

### Architecture:

```
┌─────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER                     │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │              CONTROL PLANE (Master)                 │  │
│  │                                                    │  │
│  │  API Server    → Giao tiếp (kubectl, dashboard)    │  │
│  │  Scheduler     → Quyết định pod chạy ở node nào   │  │
│  │  Controller    → Đảm bảo desired state = actual    │  │
│  │  etcd          → Database lưu cluster state        │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────┐  ┌──────────────────┐              │
│  │      NODE 1       │  │      NODE 2       │             │
│  │  (Worker Server)  │  │  (Worker Server)  │             │
│  │                   │  │                   │             │
│  │  ┌─────┐ ┌─────┐ │  │  ┌─────┐ ┌─────┐ │             │
│  │  │Pod A│ │Pod B│ │  │  │Pod C│ │Pod D│ │             │
│  │  │     │ │     │ │  │  │     │ │     │ │             │
│  │  │[App]│ │[App]│ │  │  │[App]│ │[DB] │ │             │
│  │  └─────┘ └─────┘ │  │  └─────┘ └─────┘ │             │
│  │                   │  │                   │             │
│  │  kubelet  kube-   │  │  kubelet  kube-   │             │
│  │           proxy   │  │           proxy   │             │
│  └──────────────────┘  └──────────────────┘              │
└─────────────────────────────────────────────────────────┘
```

### Core Concepts:

```
POD
- Đơn vị nhỏ nhất trong K8s
- 1 Pod = 1 hoặc nhiều containers (thường 1)
- Shared network (localhost), shared storage
- Ephemeral: Pod có thể bị xóa/restart bất cứ lúc nào

DEPLOYMENT
- Quản lý pods: replicas, updates, rollbacks
- Đảm bảo luôn có N pods đang chạy
- Rolling update: thay thế pods từng cái một

SERVICE
- Stable endpoint (IP + DNS) để truy cập pods
- Pods thay đổi liên tục → Service cung cấp địa chỉ ổn định
- Load balance traffic giữa pods

  Types:
  - ClusterIP:    Chỉ truy cập NỘI BỘ cluster (default)
  - NodePort:     Expose qua port trên mỗi Node (30000-32767)
  - LoadBalancer: Expose qua cloud Load Balancer (GKE, EKS, AKS)

INGRESS
- HTTP/HTTPS routing vào cluster
- Domain-based routing: api.myapp.com → API service
                         myapp.com → Frontend service
- SSL termination

CONFIGMAP & SECRET
- ConfigMap: Lưu config (non-sensitive): API URLs, feature flags
- Secret:    Lưu sensitive data: passwords, API keys, certificates
- Inject vào pods qua environment variables hoặc mounted files
```

### Kubernetes YAML:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nestjs-app
  labels:
    app: nestjs-app
spec:
  replicas: 3                    # Chạy 3 pods
  selector:
    matchLabels:
      app: nestjs-app
  template:
    metadata:
      labels:
        app: nestjs-app
    spec:
      containers:
      - name: nestjs-app
        image: myregistry/nestjs-app:1.0.0
        ports:
        - containerPort: 3000
        
        # Resource limits (quan trọng cho production)
        resources:
          requests:              # Minimum resources
            memory: "128Mi"
            cpu: "250m"          # 0.25 CPU
          limits:                # Maximum resources
            memory: "256Mi"
            cpu: "500m"          # 0.5 CPU
        
        # Health checks
        livenessProbe:           # Container còn sống?
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          
        readinessProbe:          # Container sẵn sàng nhận traffic?
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        
        # Environment from ConfigMap & Secret
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nestjs-service
spec:
  type: LoadBalancer
  selector:
    app: nestjs-app
  ports:
  - port: 80              # Service port
    targetPort: 3000       # Container port
```

### Health Checks:

```
3 loại probe:

LIVENESS PROBE  → "Container còn sống không?"
- Nếu fail → K8s RESTART container
- Check: process crash, deadlock
- Ví dụ: HTTP GET /health → 200 OK

READINESS PROBE → "Container sẵn sàng nhận traffic?"
- Nếu fail → K8s KHÔNG gửi traffic đến pod này
- Check: database connected, cache warmed
- Ví dụ: HTTP GET /ready → 200 OK

STARTUP PROBE   → "Container đã start xong?"
- Dùng cho apps start CHẬM
- Liveness/Readiness probe chờ startup probe pass trước
```

### kubectl Commands:

```bash
# ═══ GET (xem resources) ═══
kubectl get pods                    # Liệt kê pods
kubectl get pods -o wide            # Thêm thông tin (node, IP)
kubectl get services                # Liệt kê services
kubectl get deployments             # Liệt kê deployments
kubectl get all                     # Tất cả resources
kubectl get nodes                   # Cluster nodes

# ═══ DESCRIBE (chi tiết) ═══
kubectl describe pod <name>         # Chi tiết pod (events, errors)

# ═══ LOGS ═══
kubectl logs <pod-name>             # Xem logs
kubectl logs <pod-name> -f          # Follow logs
kubectl logs <pod-name> -c <container>  # Multi-container pod

# ═══ APPLY (deploy) ═══
kubectl apply -f deployment.yaml    # Deploy/update resources
kubectl apply -f k8s/              # Apply tất cả files trong folder

# ═══ SCALE ═══
kubectl scale deployment nestjs-app --replicas=5  # Scale lên 5 pods

# ═══ DEBUG ═══
kubectl exec -it <pod> -- sh        # Shell vào pod
kubectl port-forward <pod> 3000:3000  # Forward port đến local
kubectl top pods                     # CPU/Memory usage
```

---

## TÓM TẮT PHẦN 9

### Câu hỏi phỏng vấn:

**1. "Docker Image vs Container?"**
→ Image = template read-only (class). Container = instance đang chạy (object).
1 image → nhiều containers.

**2. "CMD vs ENTRYPOINT?"**
→ CMD: override được khi docker run. ENTRYPOINT: cố định, args append.
Thường kết hợp: ENTRYPOINT ["node"], CMD ["dist/main.js"].

**3. "Multi-stage build là gì? Tại sao cần?"**
→ Build ở stage 1 (có devDeps, TypeScript), copy kết quả sang stage 2 (chỉ production).
Giảm image size từ ~800MB → ~200MB. An toàn hơn (không có source code, devDeps).

**4. "Pod trong Kubernetes là gì?"**
→ Đơn vị nhỏ nhất, chứa 1+ containers. Ephemeral (có thể restart bất kỳ lúc nào).
Deployment quản lý pods (replicas, updates).

**5. "Service types trong K8s?"**
→ ClusterIP (nội bộ), NodePort (expose port), LoadBalancer (cloud LB).

**6. "Liveness vs Readiness probe?"**
→ Liveness: container sống? (fail → restart). 
Readiness: sẵn sàng nhận traffic? (fail → stop routing).

**7. "Bạn dockerize NestJS app thế nào?"**
→ Multi-stage Dockerfile. docker-compose cho dev (app + postgres + redis).
CI/CD build & push image → K8s deployment cho production.
