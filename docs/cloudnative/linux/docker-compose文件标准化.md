
# Docker Compose 文件标准化

## 概述

Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。为了提高可维护性、可读性和团队协作效率，需要对 Docker Compose 文件进行标准化。

## 文件命名规范

- 默认文件名：`docker-compose.yml`
- 环境特定文件：`docker-compose.{env}.yml`（如 `docker-compose.dev.yml`、`docker-compose.prod.yml`）
- 覆盖文件：`docker-compose.override.yml`

## 文件结构标准

### 基本结构

version: '3.8'  # 使用稳定的 Compose 文件版本

services:
  # 服务定义

volumes:
  # 数据卷定义

networks:
  # 网络定义

configs:
  # 配置定义（仅 v3.3+）

secrets:
  # 密钥定义（仅 v3.1+）

### 服务定义标准

services:
  service-name:
    # 1. 镜像信息
    image: repository/image:tag
    # 或使用 build
    build:
      context: ./path
      dockerfile: Dockerfile
      args:
        - BUILD_ARG=value
    
    # 2. 容器名称
    container_name: project-service-name
    
    # 3. 重启策略
    restart: unless-stopped
    
    # 4. 环境变量
    environment:
      - ENV_VAR=value
      - ANOTHER_VAR=value
    # 或使用 env_file
    env_file:
      - .env
      - .env.local
    
    # 5. 端口映射
    ports:
      - "host_port:container_port"
    
    # 6. 数据卷挂载
    volumes:
      - type: bind
        source: ./local/path
        target: /container/path
      - named-volume:/container/path
    
    # 7. 网络配置
    networks:
      - network-name
    
    # 8. 依赖关系
    depends_on:
      - dependency-service
    
    # 9. 健康检查
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # 10. 资源限制
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    
    # 11. 日志配置
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

## 命名规范

### 服务命名

- 使用小写字母和连字符
- 格式：`{project}-{service}-{role}`
- 示例：`myapp-web-frontend`、`myapp-db-postgres`

### 容器命名

- 格式：`{project}_{service}_{instance}`
- 示例：`myapp_web_1`、`myapp_db_1`

### 网络命名

- 格式：`{project}_{network_purpose}`
- 示例：`myapp_frontend`、`myapp_backend`

### 数据卷命名

- 格式：`{project}_{service}_{data_type}`
- 示例：`myapp_db_data`、`myapp_redis_data`

## 最佳实践

### 1. 版本控制

# 推荐使用 3.x 版本
version: '3.8'

### 2. 环境变量管理

# 使用 .env 文件管理敏感信息
services:
  web:
    env_file:
      - .env
    environment:
      - NODE_ENV=production
      - API_URL=${API_BASE_URL}/api

### 3. 多环境配置

# 开发环境
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# 生产环境
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up

### 4. 健康检查

healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost/ || exit 1"]
  interval: 1m
  timeout: 10s
  retries: 3
  start_period: 40s

### 5. 资源限制

deploy:
  resources:
    limits:
      cpus: '1'
      memory: 1G
    reservations:
      cpus: '0.5'
      memory: 512M

### 6. 日志管理

logging:
  driver: "json-file"
  options:
    max-size: "50m"
    max-file: "5"
    labels: "production_status"

### 7. 依赖顺序

services:
  web:
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

## 完整示例

version: '3.8'

services:
  # Web 应用服务
  web:
    image: nginx:alpine
    container_name: myapp_web
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./app:/usr/share/nginx/html:ro
      - web-logs:/var/log/nginx
    networks:
      - frontend
      - backend
    depends_on:
      - app
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 5s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # 应用服务
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
    container_name: myapp_app
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - REDIS_HOST=redis
    volumes:
      - ./app:/app
      - app-modules:/app/node_modules
    networks:
      - backend
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M

  # 数据库服务
  db:
    image: postgres:14-alpine
    container_name: myapp_db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d:ro
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # 缓存服务
  redis:
    image: redis:7-alpine
    container_name: myapp_redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  web-logs:
    driver: local
  app-modules:
    driver: local
  db-data:
    driver: local
  redis-data:
    driver: local

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true

## 安全建议

1. **不要在 docker-compose.yml 中硬编码敏感信息**
   - 使用环境变量或 secrets
   - 将 `.env` 文件添加到 `.gitignore`

2. **限制网络访问**
   networks:
     backend:
       internal: true  # 内部网络，不允许外部访问

3. **使用只读挂载**
   volumes:
     - ./config:/app/config:ro

4. **设置合理的资源限制**
   deploy:
     resources:
       limits:
         memory: 512M

## 常用命令

# 启动服务
docker-compose up -d

# 停止服务
docker-compose down

# 查看日志
docker-compose logs -f [service]

# 重启服务
docker-compose restart [service]

# 查看服务状态
docker-compose ps

# 执行命令
docker-compose exec [service] [command]

# 验证配置文件
docker-compose config

# 拉取镜像
docker-compose pull

## 检查清单

- [ ] 使用稳定的 Compose 版本（3.8）
- [ ] 服务名称遵循命名规范
- [ ] 配置了合适的重启策略
- [ ] 敏感信息使用环境变量
- [ ] 配置了健康检查
- [ ] 设置了资源限制
- [ ] 配置了日志管理
- [ ] 正确定义服务依赖关系
- [ ] 使用命名卷持久化数据
- [ ] 网络配置合理
- [ ] 添加了必要的注释
