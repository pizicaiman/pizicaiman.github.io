# 基于Nacos服务发现的网关项目框架梳理

## 项目架构概述

本项目是一个基于Nacos服务发现的微服务网关，用于统一管理微服务的路由、负载均衡、安全认证等功能。网关作为系统的入口点，负责将请求路由到相应的后端服务，并提供统一的安全控制、限流、监控等能力。

## 技术栈

- **网关框架**：Spring Cloud Gateway
- **服务发现**：Nacos Discovery
- **配置管理**：Nacos Config
- **负载均衡**：Spring Cloud LoadBalancer
- **安全框架**：Spring Security + OAuth2
- **监控**：Spring Boot Actuator + Micrometer
- **构建工具**：Maven
- **编程语言**：Java 11+

## 项目结构

```
nacos-gateway/
├── pom.xml
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/gateway/
│       │       ├── GatewayApplication.java
│       │       ├── config/
│       │       │   ├── GatewayConfig.java
│       │       │   ├── NacosConfig.java
│       │       │   ├── SecurityConfig.java
│       │       │   └── RateLimiterConfig.java
│       │       ├── filter/
│       │       │   ├── AuthGlobalFilter.java
│       │       │   ├── LoggingGlobalFilter.java
│       │       │   └── ResponseGlobalFilter.java
│       │       ├── resolver/
│       │       │   └── CustomRouteLocator.java
│       │       ├── handler/
│       │       │   └── HystrixHandler.java
│       │       └── util/
│       │           └── JwtUtil.java
│       └── resources/
│           ├── application.yml
│           ├── bootstrap.yml
│           └── static/
└── README.md
```

## 核心组件详解

### 1. 主应用类 (GatewayApplication.java)

```java
package com.example.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

### 2. 网关配置类 (GatewayConfig.java)

```java
package com.example.gateway.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("user-service", r -> r.path("/api/user/**")
                        .uri("lb://user-service"))
                .route("order-service", r -> r.path("/api/order/**")
                        .uri("lb://order-service"))
                .route("product-service", r -> r.path("/api/product/**")
                        .uri("lb://product-service"))
                .build();
    }
}
```

### 3. Nacos配置类 (NacosConfig.java)

```java
package com.example.gateway.config;

import com.alibaba.cloud.nacos.NacosConfigManager;
import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.api.exception.NacosException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class NacosConfig {

    @Autowired
    private NacosConfigManager nacosConfigManager;

    @Bean
    public ConfigService configService() throws NacosException {
        return nacosConfigManager.getConfigService();
    }
}
```

### 4. 安全配置类 (SecurityConfig.java)

```java
package com.example.gateway.config;

import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        http.authorizeExchange()
                .pathMatchers("/actuator/**").permitAll()
                .pathMatchers("/auth/login").permitAll()
                .anyExchange().authenticated()
                .and().oauth2ResourceServer().jwt();
        return http.build();
    }
}
```

### 5. 限流配置类 (RateLimiterConfig.java)

```java
package com.example.gateway.config;

import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import reactor.core.publisher.Mono;

@Configuration
public class RateLimiterConfig {

    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
    }
}
```

### 6. 全局认证过滤器 (AuthGlobalFilter.java)

```java
package com.example.gateway.filter;

import com.example.gateway.util.JwtUtil;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class AuthGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");
        
        // 放行不需要认证的路径
        String path = exchange.getRequest().getURI().getPath();
        if (path.contains("/actuator") || path.contains("/auth/login")) {
            return chain.filter(exchange);
        }
        
        // 验证token
        if (token == null || !JwtUtil.validateToken(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

### 7. 全局日志过滤器 (LoggingGlobalFilter.java)

```java
package com.example.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class LoggingGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        long startTime = System.currentTimeMillis();
        return chain.filter(exchange).then(
                Mono.fromRunnable(() -> {
                    long endTime = System.currentTimeMillis();
                    String path = exchange.getRequest().getURI().getPath();
                    String method = exchange.getRequest().getMethodValue();
                    int status = exchange.getResponse().getStatusCode().value();
                    System.out.println(String.format("请求日志: %s %s %d 耗时: %d ms",
                            method, path, status, endTime - startTime));
                })
        );
    }

    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```

### 8. 响应处理过滤器 (ResponseGlobalFilter.java)

```java
package com.example.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.nio.charset.StandardCharsets;

@Component
public class ResponseGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpResponse response = exchange.getResponse();
        response.getHeaders().add("X-Gateway-Version", "1.0.0");
        return chain.filter(exchange);
    }

    private Mono<Void> writeResponse(ServerHttpResponse response, String message) {
        response.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
        byte[] datas = message.getBytes(StandardCharsets.UTF_8);
        DataBuffer buffer = response.bufferFactory().wrap(datas);
        return response.writeWith(Mono.just(buffer));
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

### 9. JWT工具类 (JwtUtil.java)

```java
package com.example.gateway.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class JwtUtil {

    private static final String SECRET = "mySecretKey";
    private static final long EXPIRATION = 86400L; // 24小时

    // 生成JWT token
    public static String generateToken(String username) {
        Map<String, Object> claims = new HashMap<>();
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(username)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION * 1000))
                .signWith(SignatureAlgorithm.HS512, SECRET)
                .compact();
    }

    // 验证JWT token
    public static Boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(SECRET).parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // 从token中获取用户名
    public static String getUsernameFromToken(String token) {
        try {
            Claims claims = Jwts.parser().setSigningKey(SECRET).parseClaimsJws(token).getBody();
            return claims.getSubject();
        } catch (Exception e) {
            return null;
        }
    }
}
```

### 10. 自定义路由定位器 (CustomRouteLocator.java)

```java
package com.example.gateway.resolver;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CustomRouteLocator {

    @Bean
    public RouteLocator dynamicRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("dynamic_route_1", r -> r.path("/dynamic/**")
                        .filters(f -> f.stripPrefix(1))
                        .uri("lb://dynamic-service"))
                .build();
    }
}
```

### 11. 熔断处理器 (HystrixHandler.java)

```java
package com.example.gateway.handler;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
public class HystrixHandler {

    @GetMapping("/fallback")
    public Mono<String> fallback() {
        return Mono.just("服务暂时不可用，请稍后再试！");
    }
}
```

## 配置文件详解

### 1. 主配置文件 (application.yml)

```yaml
server:
  port: 8080

spring:
  application:
    name: nacos-gateway
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yaml
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/user/**
          filters:
            - StripPrefix=2
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/order/**
          filters:
            - StripPrefix=2
        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/product/**
          filters:
            - StripPrefix=2
      default-filters:
        - name: Hystrix
          args:
            name: default
            fallbackUri: forward:/fallback
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOriginPatterns: "*"
            allowedMethods: "*"
            allowedHeaders: "*"
            allowCredentials: true
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8081/auth/realms/myrealm

management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    gateway:
      enabled: true

logging:
  level:
    org.springframework.cloud.gateway: DEBUG
```

### 2. Bootstrap配置文件 (bootstrap.yml)

```yaml
spring:
  application:
    name: nacos-gateway
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yaml
        group: GATEWAY_GROUP
        namespace: public
```

## Nacos配置管理

### 1. 网关路由配置 (nacos-gateway.yaml)

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/user/**
          filters:
            - StripPrefix=2
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/order/**
          filters:
            - StripPrefix=2
        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/product/**
          filters:
            - StripPrefix=2
```

### 2. 限流配置 (rate-limiter-config.yaml)

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: rate-limit-route
          uri: lb://user-service
          predicates:
            - Path=/api/rate/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

## 服务调用流程

### 1. 正常请求流程

```
客户端 → 网关 → Nacos服务发现 → 目标服务
   ↑                               ↓
   ←────── 响应返回 ←───────────────┘
```

### 2. 认证流程

```
客户端 → AuthGlobalFilter → 验证Token → 路由到目标服务
   ↑                                          ↓
   ←──────────── 响应返回 ←───────────────────┘
```

### 3. 限流流程

```
客户端 → 限流过滤器 → 检查请求频率 → 路由到目标服务
   ↑                                      ↓
   ←────────── 响应返回 ←─────────────────┘
```

## 部署架构

### 1. 单体部署架构

```
┌─────────────────────────────────────────────────────────────┐
│                        Load Balancer                        │
└─────────────────────────────┬───────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼───────┐     ┌───────▼───────┐     ┌───────▼───────┐
│  Gateway-1    │     │  Gateway-2    │     │  Gateway-N    │
│ (Nacos Client)│     │ (Nacos Client)│     │ (Nacos Client)│
└───────┬───────┘     └───────┬───────┘     └───────┬───────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼───────┐     ┌───────▼───────┐     ┌───────▼───────┐
│ User Service  │     │ Order Service │     │ Product Serv. │
│ (Nacos Client)│     │ (Nacos Client)│     │ (Nacos Client)│
└───────────────┘     └───────────────┘     └───────────────┘
```

### 2. 高可用部署架构

```
┌─────────────────────────────────────────────────────────────┐
│                        VIP/SLB                            │
└─────────────────────────────┬───────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼───────┐     ┌───────▼───────┐     ┌───────▼───────┐
│   Nacos-1     │     │   Nacos-2     │     │   Nacos-3     │
│   Cluster     │◄───►│   Cluster     │◄───►│   Cluster     │
└───────┬───────┘     └───────┬───────┘     └───────┬───────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼───────┐     ┌───────▼───────┐     ┌───────▼───────┐
│  Gateway-1    │     │  Gateway-2    │     │  Gateway-N    │
│ (Nacos Client)│     │ (Nacos Client)│     │ (Nacos Client)│
└───────┬───────┘     └───────┬───────┘     └───────┬───────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼───────┐     ┌───────▼───────┐     ┌───────▼───────┐
│ User Service  │     │ Order Service │     │ Product Serv. │
│ (Nacos Client)│     │ (Nacos Client)│     │ (Nacos Client)│
└───────────────┘     └───────────────┘     └───────────────┘
```

## 监控与运维

### 1. 健康检查端点

- `/actuator/health` - 健康检查
- `/actuator/gateway/routes` - 路由信息
- `/actuator/metrics` - 指标监控

### 2. 日志配置

```yaml
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    reactor.netty.http.client: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"
```

### 3. 性能监控指标

- 路由请求数量
- 响应时间
- 错误率
- 限流触发次数

## 安全机制

### 1. 认证机制

- JWT Token验证
- OAuth2集成
- API密钥验证

### 2. 授权机制

- 基于角色的访问控制(RBAC)
- 路径级别的权限控制

### 3. 数据安全

- HTTPS加密传输
- 敏感信息脱敏
- 请求/响应日志记录

## 扩展性设计

### 1. 动态路由

- 支持运行时动态添加/修改路由
- 通过Nacos配置中心实现配置热更新

### 2. 插件化过滤器

- 自定义全局过滤器
- 自定义路由级别过滤器

### 3. 多租户支持

- 基于路径或Header的租户隔离
- 租户级别的配置管理

## 总结

基于Nacos服务发现的网关项目具有以下优势：

1. **服务自动发现**：与Nacos集成，自动发现和路由到后端服务
2. **配置动态更新**：通过Nacos配置中心实现配置的动态更新
3. **高可用性**：支持集群部署，保证网关的高可用性
4. **安全控制**：集成认证授权机制，保障服务安全
5. **监控运维**：提供丰富的监控指标和运维接口
6. **扩展性强**：支持动态路由和插件化扩展

这种架构设计能够很好地满足微服务架构下的统一入口需求，提供稳定、安全、高性能的API网关服务。