
# Dockerfile文件标准

## 1. 基本规范

### 1.1 通用最佳实践

- 使用官方基础镜像
- 最小化层数，合并RUN命令
- 使用多阶段构建减小镜像体积
- 不要在镜像中存储敏感信息
- 使用.dockerignore文件排除不必要的文件
- 指定具体的版本标签，避免使用latest
- 合理使用缓存机制
- 每个RUN指令后清理缓存

### 1.2 标准结构

# 基础镜像
FROM <image>:<tag>

# 维护者信息
LABEL maintainer="your-email@example.com"

# 设置环境变量
ENV KEY=VALUE

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY <source> <dest>

# 安装依赖
RUN <commands>

# 暴露端口
EXPOSE <port>

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s \
  CMD <health-check-command>

# 启动命令
CMD ["executable","param1","param2"]

---

## 2. Java应用

### 2.1 Spring Boot应用（多阶段构建）

# 构建阶段
FROM maven:3.8.6-openjdk-17-slim AS builder

LABEL maintainer="java-team@example.com"
LABEL description="Spring Boot Application"

WORKDIR /build

# 复制pom.xml并下载依赖（利用Docker缓存）
COPY pom.xml .
RUN mvn dependency:go-offline -B

# 复制源代码并构建
COPY src ./src
RUN mvn clean package -DskipTests -B

# 运行阶段
FROM openjdk:17-slim

# 创建非root用户
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# 从构建阶段复制jar包
COPY --from=builder /build/target/*.jar app.jar

# 设置JVM参数
ENV JAVA_OPTS="-Xmx512m -Xms256m -XX:+UseG1GC"

# 暴露端口
EXPOSE 8080

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# 切换到非root用户
USER appuser

# 启动应用
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

### 2.2 Gradle构建的Java应用

FROM gradle:7.6-jdk17 AS builder

WORKDIR /build

# 复制Gradle配置文件
COPY build.gradle settings.gradle gradlew ./
COPY gradle ./gradle

# 下载依赖
RUN gradle dependencies --no-daemon

# 复制源代码并构建
COPY src ./src
RUN gradle bootJar --no-daemon

# 运行阶段
FROM openjdk:17-slim

WORKDIR /app

COPY --from=builder /build/build/libs/*.jar app.jar

ENV JAVA_OPTS="-Xmx512m -Xms256m"

EXPOSE 8080

CMD ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

---

## 3. Node.js应用

### 3.1 Express/Koa应用

# 构建阶段
FROM node:18-alpine AS builder

LABEL maintainer="nodejs-team@example.com"

WORKDIR /app

# 复制package文件
COPY package*.json ./

# 安装依赖（生产环境）
RUN npm ci --only=production && \
    npm cache clean --force

# 运行阶段
FROM node:18-alpine

# 安装dumb-init（处理信号）
RUN apk add --no-cache dumb-init

# 创建非root用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# 从构建阶段复制node_modules
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

# 复制应用代码
COPY --chown=nodejs:nodejs . .

# 设置环境变量
ENV NODE_ENV=production \
    PORT=3000

EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD node healthcheck.js

USER nodejs

# 使用dumb-init启动
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "index.js"]

### 3.2 Next.js应用

FROM node:18-alpine AS deps

WORKDIR /app
COPY package*.json ./
RUN npm ci

# 构建阶段
FROM node:18-alpine AS builder

WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1

RUN npm run build

# 运行阶段
FROM node:18-alpine AS runner

WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]

---

## 4. Python应用

### 4.1 Flask/Django应用

# 构建阶段
FROM python:3.11-slim AS builder

LABEL maintainer="python-team@example.com"

WORKDIR /app

# 安装系统依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev && \
    rm -rf /var/lib/apt/lists/*

# 复制requirements文件
COPY requirements.txt .

# 创建虚拟环境并安装依赖
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# 运行阶段
FROM python:3.11-slim

WORKDIR /app

# 只安装运行时需要的系统依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libpq5 && \
    rm -rf /var/lib/apt/lists/*

# 创建非root用户
RUN useradd -m -u 1000 appuser

# 从构建阶段复制虚拟环境
COPY --from=builder /opt/venv /opt/venv

# 复制应用代码
COPY --chown=appuser:appuser . .

# 设置环境变量
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD python -c "import requests; requests.get('http://localhost:8000/health')"

USER appuser

# 使用gunicorn启动（生产环境）
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]

### 4.2 FastAPI应用

FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

ENV PYTHONUNBUFFERED=1

EXPOSE 8000

# 使用uvicorn启动
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]

---

## 5. Go应用

### 5.1 标准Go应用（多阶段构建）

# 构建阶段
FROM golang:1.21-alpine AS builder

LABEL maintainer="go-team@example.com"

# 安装构建工具
RUN apk add --no-cache git make

WORKDIR /build

# 复制go.mod和go.sum
COPY go.mod go.sum ./

# 下载依赖
RUN go mod download

# 复制源代码
COPY . .

# 编译（静态链接）
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -installsuffix cgo \
    -o app .

# 运行阶段（使用最小镜像）
FROM alpine:latest

# 安装ca证书（HTTPS请求需要）
RUN apk --no-cache add ca-certificates tzdata

WORKDIR /app

# 从构建阶段复制二进制文件
COPY --from=builder /build/app .

# 创建非root用户
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser && \
    chown -R appuser:appuser /app

# 设置时区
ENV TZ=Asia/Shanghai

EXPOSE 8080

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

USER appuser

CMD ["./app"]

### 5.2 使用scratch镜像（最小化）

FROM golang:1.21-alpine AS builder

WORKDIR /build

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags='-w -s' -o app .

# 使用scratch空镜像
FROM scratch

# 复制ca证书
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# 复制时区信息
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo

# 复制二进制文件
COPY --from=builder /build/app /app

EXPOSE 8080

ENTRYPOINT ["/app"]

---

## 6. PHP应用

### 6.1 Laravel应用

# 构建阶段
FROM composer:2.6 AS builder

WORKDIR /app

# 复制composer文件
COPY composer.json composer.lock ./

# 安装依赖（不包括开发依赖）
RUN composer install --no-dev --no-scripts --no-autoloader --prefer-dist

COPY . .

RUN composer dump-autoload --optimize --classmap-authoritative

# 运行阶段
FROM php:8.2-fpm-alpine

# 安装PHP扩展
RUN apk add --no-cache \
    libpng-dev \
    libjpeg-turbo-dev \
    libzip-dev \
    zip \
    unzip && \
    docker-php-ext-configure gd --with-jpeg && \
    docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    gd \
    zip \
    opcache

# 安装Redis扩展
RUN pecl install redis && \
    docker-php-ext-enable redis

WORKDIR /var/www/html

# 从构建阶段复制文件
COPY --from=builder /app .

# 设置权限
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache

# 复制PHP配置
COPY php.ini /usr/local/etc/php/conf.d/custom.ini

EXPOSE 9000

CMD ["php-fpm"]

### 6.2 Nginx + PHP-FPM组合

# PHP-FPM容器
FROM php:8.2-fpm-alpine

RUN docker-php-ext-install pdo_mysql opcache

COPY . /var/www/html
WORKDIR /var/www/html

CMD ["php-fpm"]

# Nginx容器
FROM nginx:1.25-alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

---

## 7. .NET应用

### 7.1 ASP.NET Core应用

# 构建阶段
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder

WORKDIR /src

# 复制项目文件并还原依赖
COPY *.csproj ./
RUN dotnet restore

# 复制所有文件并构建
COPY . ./
RUN dotnet publish -c Release -o /app/publish --no-restore

# 运行阶段
FROM mcr.microsoft.com/dotnet/aspnet:8.0

WORKDIR /app

# 从构建阶段复制发布文件
COPY --from=builder /app/publish .

# 设置环境变量
ENV ASPNETCORE_URLS=http://+:5000 \
    ASPNETCORE_ENVIRONMENT=Production

EXPOSE 5000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s \
  CMD curl -f http://localhost:5000/health || exit 1

ENTRYPOINT ["dotnet", "YourApp.dll"]

---

## 8. 前端应用（Nginx部署）

### 8.1 Vue/React/Angular应用

# 构建阶段
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# 运行阶段
FROM nginx:1.25-alpine

# 复制自定义nginx配置
COPY nginx.conf /etc/nginx/nginx.conf

# 从构建阶段复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 添加非root用户
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chmod -R 755 /usr/share/nginx/html

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

CMD ["nginx", "-g", "daemon off;"]

nginx.conf示例：
user nginx;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;
    keepalive_timeout 65;
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    server {
        listen 80;
        server_name _;
        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }

        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
}

---

## 9. 数据库镜像自定义

### 9.1 MySQL自定义镜像

FROM mysql:8.0

LABEL maintainer="dba-team@example.com"

# 复制初始化脚本
COPY init-scripts/ /docker-entrypoint-initdb.d/

# 复制自定义配置
COPY my.cnf /etc/mysql/conf.d/custom.cnf

# 设置时区
ENV TZ=Asia/Shanghai

EXPOSE 3306

# MySQL镜像已有健康检查，这里可选
HEALTHCHECK --interval=10s --timeout=3s --start-period=30s \
  CMD mysqladmin ping -h localhost -u root -p${MYSQL_ROOT_PASSWORD} || exit 1

---

## 10. .dockerignore文件示例

# Git
.git
.gitignore
.gitattributes

# CI/CD
.github
.gitlab-ci.yml
Jenkinsfile

# 文档
*.md
README
docs/
LICENSE

# 日志
*.log
logs/

# 测试
**/test
**/tests
**/__tests__
*.test.js
*.spec.js
coverage/

# 依赖目录（根据语言选择）
node_modules/
vendor/
venv/
__pycache__/
*.pyc

# IDE
.vscode
.idea
*.swp
*.swo

# 环境变量
.env
.env.local
.env.*.local

# 构建产物
dist/
build/
target/
*.o
*.so

# 临时文件
tmp/
temp/
*.tmp

---

## 11. 安全最佳实践

### 11.1 镜像安全扫描

使用工具：
- Trivy
- Clair
- Anchore

### 11.2 关键安全配置

# 使用非root用户运行
USER appuser

# 只读文件系统（需要应用支持）
# docker run --read-only

# 限制capabilities
# docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE

# 不使用特权模式
# 避免 --privileged

# 使用健康检查
HEALTHCHECK CMD <command>

# 签名镜像
# docker trust sign

---

## 12. 性能优化建议

1. **层缓存优化**：将不经常变化的层放在前面
2. **多阶段构建**：减小最终镜像大小
3. **合并RUN命令**：减少层数
4. **使用轻量基础镜像**：alpine, slim, distroless
5. **清理临时文件**：每个RUN后清理缓存
6. **使用构建缓存**：利用BuildKit和缓存挂载

# 使用BuildKit缓存挂载
# syntax=docker/dockerfile:1
FROM node:18-alpine

WORKDIR /app

RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production
