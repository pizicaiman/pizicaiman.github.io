# Bookstore 项目框架梳理

## 项目架构概述

本项目采用分层架构设计，遵循 MVC 模式，包含以下核心组件：

## 1. Model 类 - 数据模型层

定义用户、书籍信息等数据结构，负责数据实体的定义和验证。

### 代码实现示例：

```go
// model/user.go
package model

import (
    "time"
    "gorm.io/gorm"
)

type User struct {
    ID        uint           `json:"id" gorm:"primaryKey"`
    Username  string         `json:"username" gorm:"uniqueIndex;not null"`
    Email     string         `json:"email" gorm:"uniqueIndex;not null"`
    Password  string         `json:"-" gorm:"not null"` // 密码不返回给前端
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `json:"-" gorm:"index"`
}

// model/book.go
type Book struct {
    ID          uint           `json:"id" gorm:"primaryKey"`
    Title       string         `json:"title" gorm:"not null"`
    Author      string         `json:"author" gorm:"not null"`
    ISBN        string         `json:"isbn" gorm:"uniqueIndex"`
    Price       float64        `json:"price" gorm:"not null"`
    Stock       int            `json:"stock" gorm:"default:0"`
    Description string         `json:"description"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
    DeletedAt   gorm.DeletedAt `json:"-" gorm:"index"`
}
```

## 2. Config 类 - 配置管理

定义中间件的 YAML 配置文件，读取配置并生成服务器配置信息。

### 代码实现示例：

```go
// config/config.go
package config

import (
    "gopkg.in/yaml.v2"
    "io/ioutil"
    "log"
)

type Config struct {
    Server   ServerConfig   `yaml:"server"`
    Database DatabaseConfig `yaml:"database"`
    Redis    RedisConfig    `yaml:"redis"`
    JWT      JWTConfig      `yaml:"jwt"`
    RabbitMQ RabbitMQConfig `yaml:"rabbitmq"`
}

type ServerConfig struct {
    Port string `yaml:"port"`
    Host string `yaml:"host"`
}

type DatabaseConfig struct {
    Host     string `yaml:"host"`
    Port     int    `yaml:"port"`
    Username string `yaml:"username"`
    Password string `yaml:"password"`
    DBName   string `yaml:"dbname"`
}

type RedisConfig struct {
    Host     string `yaml:"host"`
    Port     int    `yaml:"port"`
    Password string `yaml:"password"`
    DB       int    `yaml:"db"`
}

type JWTConfig struct {
    SecretKey   string `yaml:"secret_key"`
    ExpireHours int    `yaml:"expire_hours"`
}

type RabbitMQConfig struct {
    Host     string `yaml:"host"`
    Port     int    `yaml:"port"`
    Username string `yaml:"username"`
    Password string `yaml:"password"`
}

func LoadConfig(configPath string) (*Config, error) {
    data, err := ioutil.ReadFile(configPath)
    if err != nil {
        return nil, err
    }
    
    var config Config
    err = yaml.Unmarshal(data, &config)
    if err != nil {
        return nil, err
    }
    
    return &config, nil
}
```

### 配置文件示例 (config.yaml)：

```yaml
server:
  port: "8080"
  host: "localhost"

database:
  host: "localhost"
  port: 3306
  username: "root"
  password: "password"
  dbname: "bookstore"

redis:
  host: "localhost"
  port: 6379
  password: ""
  db: 0

jwt:
  secret_key: "your-secret-key"
  expire_hours: 24

rabbitmq:
  host: "localhost"
  port: 5672
  username: "guest"
  password: "guest"
```

## 3. Init 类 - 初始化模块

创建数据库连接、Redis 连接、Session、日志等实例化操作。

### 代码实现示例：

```go
// init/init.go
package init

import (
    "bookstore/config"
    "bookstore/model"
    "context"
    "fmt"
    "log"
    
    "github.com/redis/go-redis/v9"
    "github.com/streadway/amqp"
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
)

var (
    DB      *gorm.DB
    RDB     *redis.Client
    MQConn  *amqp.Connection
    Conf    *config.Config
    ctx     = context.Background()
)

func InitDatabase() error {
    dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=utf8mb4&parseTime=True&loc=Local",
        Conf.Database.Username,
        Conf.Database.Password,
        Conf.Database.Host,
        Conf.Database.Port,
        Conf.Database.DBName,
    )
    
    var err error
    DB, err = gorm.Open(mysql.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
    })
    
    if err != nil {
        return fmt.Errorf("failed to connect database: %v", err)
    }
    
    // 自动迁移
    err = DB.AutoMigrate(&model.User{}, &model.Book{})
    if err != nil {
        return fmt.Errorf("failed to migrate database: %v", err)
    }
    
    log.Println("Database connected successfully")
    return nil
}

func InitRedis() error {
    RDB = redis.NewClient(&redis.Options{
        Addr:     fmt.Sprintf("%s:%d", Conf.Redis.Host, Conf.Redis.Port),
        Password: Conf.Redis.Password,
        DB:       Conf.Redis.DB,
    })
    
    // 测试连接
    _, err := RDB.Ping(ctx).Result()
    if err != nil {
        return fmt.Errorf("failed to connect redis: %v", err)
    }
    
    log.Println("Redis connected successfully")
    return nil
}

func InitRabbitMQ() error {
    var err error
    connStr := fmt.Sprintf("amqp://%s:%s@%s:%d/",
        Conf.RabbitMQ.Username,
        Conf.RabbitMQ.Password,
        Conf.RabbitMQ.Host,
        Conf.RabbitMQ.Port,
    )
    
    MQConn, err = amqp.Dial(connStr)
    if err != nil {
        return fmt.Errorf("failed to connect to RabbitMQ: %v", err)
    }
    
    log.Println("RabbitMQ connected successfully")
    return nil
}

func InitAll(configPath string) error {
    var err error
    
    // 加载配置
    Conf, err = config.LoadConfig(configPath)
    if err != nil {
        return fmt.Errorf("failed to load config: %v", err)
    }
    
    // 初始化数据库
    if err = InitDatabase(); err != nil {
        return err
    }
    
    // 初始化 Redis
    if err = InitRedis(); err != nil {
        return err
    }
    
    // 初始化 RabbitMQ
    if err = InitRabbitMQ(); err != nil {
        log.Printf("Warning: failed to initialize RabbitMQ: %v", err)
    }
    
    return nil
}

func Close() {
    if MQConn != nil {
        MQConn.Close()
    }
    // 其他资源的关闭操作
}
```

## 4. Repository/DAO 类 - 数据访问层

定义数据库操作，调用 Model 类中的方法进行数据持久化。

### 代码实现示例：

``go
// repository/user_repository.go
package repository

import (
    "bookstore/model"
    "gorm.io/gorm"
)

type UserRepository interface {
    Create(user *model.User) error
    GetByID(id uint) (*model.User, error)
    GetByUsername(username string) (*model.User, error)
    GetByEmail(email string) (*model.User, error)
    Update(user *model.User) error
    Delete(id uint) error
    List(offset, limit int) ([]*model.User, error)
}

type userRepository struct {
    db *gorm.DB
}

func NewUserRepository(db *gorm.DB) UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(user *model.User) error {
    return r.db.Create(user).Error
}

func (r *userRepository) GetByID(id uint) (*model.User, error) {
    var user model.User
    err := r.db.First(&user, id).Error
    if err != nil {
        return nil, err
    }
    return &user, nil
}

func (r *userRepository) GetByUsername(username string) (*model.User, error) {
    var user model.User
    err := r.db.Where("username = ?", username).First(&user).Error
    if err != nil {
        return nil, err
    }
    return &user, nil
}

func (r *userRepository) GetByEmail(email string) (*model.User, error) {
    var user model.User
    err := r.db.Where("email = ?", email).First(&user).Error
    if err != nil {
        return nil, err
    }
    return &user, nil
}

func (r *userRepository) Update(user *model.User) error {
    return r.db.Save(user).Error
}

func (r *userRepository) Delete(id uint) error {
    return r.db.Delete(&model.User{}, id).Error
}

func (r *userRepository) List(offset, limit int) ([]*model.User, error) {
    var users []*model.User
    err := r.db.Offset(offset).Limit(limit).Find(&users).Error
    return users, err
}
```

## 5. Service 类 - 业务逻辑层

定义业务逻辑，调用 Repository 类中的方法，同时定义请求体接口的参数结构体。

### 代码实现示例：

``go
// service/user_service.go
package service

import (
    "bookstore/exception"
    "bookstore/model"
    "bookstore/repository"
    "bookstore/util"
    "context"
    "encoding/json"
    "time"
    
    "github.com/streadway/amqp"
    "golang.org/x/crypto/bcrypt"
    "github.com/golang-jwt/jwt/v5"
)

type UserService interface {
    Register(req *RegisterRequest) (*model.User, error)
    Login(req *LoginRequest) (*LoginResponse, error)
    Logout(tokenString string) error
    GetUserByID(id uint) (*model.User, error)
    UpdateUser(req *UpdateUserRequest) (*model.User, error)
    DeleteUser(id uint) error
    ListUsers(req *ListUsersRequest) (*ListUsersResponse, error)
}

// 请求体结构体定义
type RegisterRequest struct {
    Username string `json:"username" binding:"required,min=3,max=20"`
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required,min=6"`
}

type LoginRequest struct {
    Username string `json:"username" binding:"required"`
    Password string `json:"password" binding:"required"`
}

type UpdateUserRequest struct {
    ID       uint   `json:"id" binding:"required"`
    Username string `json:"username" binding:"omitempty,min=3,max=20"`
    Email    string `json:"email" binding:"omitempty,email"`
}

type ListUsersRequest struct {
    Page     int    `form:"page" binding:"min=1"`
    PageSize int    `form:"page_size" binding:"min=1,max=100"`
    Username string `form:"username"`
    Email    string `form:"email"`
}

// 响应体结构体定义
type LoginResponse struct {
    Token string     `json:"token"`
    User  *model.User `json:"user"`
}

type ListUsersResponse struct {
    Users    []*model.User `json:"users"`
    Total    int64         `json:"total"`
    Page     int           `json:"page"`
    PageSize int           `json:"page_size"`
}

type userService struct {
    userRepo    repository.UserRepository
    jwtSecret   string
    mqChannel   *amqp.Channel
    redisClient interface{} // Redis 客户端接口
}

func NewUserService(userRepo repository.UserRepository, jwtSecret string, mqConn *amqp.Connection) UserService {
    // 创建 RabbitMQ 通道
    ch, err := mqConn.Channel()
    if err != nil {
        // 错误处理
        panic(err)
    }
    
    // 声明队列
    _, err = ch.QueueDeclare(
        "user_events", // 队列名称
        true,          // 持久化
        false,         // 自动删除
        false,         // 独占
        false,         // 不等待
        nil,           // 参数
    )
    if err != nil {
        panic(err)
    }
    
    return &userService{
        userRepo:  userRepo,
        jwtSecret: jwtSecret,
        mqChannel: ch,
    }
}

func (s *userService) Register(req *RegisterRequest) (*model.User, error) {
    // 检查用户名是否已存在
    existingUser, _ := s.userRepo.GetByUsername(req.Username)
    if existingUser != nil {
        return nil, exception.ErrUserExists
    }
    
    // 检查邮箱是否已存在
    existingUser, _ = s.userRepo.GetByEmail(req.Email)
    if existingUser != nil {
        return nil, exception.ErrUserExists
    }
    
    // 加密密码
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
    if err != nil {
        return nil, exception.WrapError(err, "密码加密失败")
    }
    
    // 创建用户
    user := &model.User{
        Username: req.Username,
        Email:    req.Email,
        Password: string(hashedPassword),
    }
    
    err = s.userRepo.Create(user)
    if err != nil {
        return nil, exception.WrapError(err, "创建用户失败")
    }
    
    // 发送用户注册事件到消息队列
    event := map[string]interface{}{
        "event":     "user_registered",
        "user_id":   user.ID,
        "username":  user.Username,
        "timestamp": time.Now(),
    }
    
    eventBytes, _ := json.Marshal(event)
    s.mqChannel.Publish(
        "",              // 交换机
        "user_events",   // 路由键
        false,           // 强制
        false,           // 立即
        amqp.Publishing{
            ContentType: "application/json",
            Body:        eventBytes,
        })
    
    return user, nil
}

func (s *userService) Login(req *LoginRequest) (*LoginResponse, error) {
    // 根据用户名查找用户
    user, err := s.userRepo.GetByUsername(req.Username)
    if err != nil {
        return nil, exception.ErrInvalidPassword
    }
    
    // 验证密码
    err = bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(req.Password))
    if err != nil {
        return nil, exception.ErrInvalidPassword
    }
    
    // 生成 JWT Token
    token, err := s.generateJWT(user.ID, user.Username)
    if err != nil {
        return nil, exception.WrapError(err, "生成令牌失败")
    }
    
    // 发送用户登录事件到消息队列
    event := map[string]interface{}{
        "event":     "user_logged_in",
        "user_id":   user.ID,
        "username":  user.Username,
        "timestamp": time.Now(),
    }
    
    eventBytes, _ := json.Marshal(event)
    s.mqChannel.Publish(
        "",              // 交换机
        "user_events",   // 路由键
        false,           // 强制
        false,           // 立即
        amqp.Publishing{
            ContentType: "application/json",
            Body:        eventBytes,
        })
    
    return &LoginResponse{
        Token: token,
        User:  user,
    }, nil
}

func (s *userService) Logout(tokenString string) error {
    // 在实际应用中，可以将 token 加入 Redis 黑名单
    // 这里简化处理，只发送登出事件
    
    // 解析 token 获取用户信息
    claims, err := util.ValidateJWT(tokenString, s.jwtSecret)
    if err != nil {
        return exception.ErrInvalidToken
    }
    
    // 发送用户登出事件到消息队列
    event := map[string]interface{}{
        "event":     "user_logged_out",
        "user_id":   claims["user_id"],
        "username":  claims["username"],
        "timestamp": time.Now(),
    }
    
    eventBytes, _ := json.Marshal(event)
    s.mqChannel.Publish(
        "",              // 交换机
        "user_events",   // 路由键
        false,           // 强制
        false,           // 立即
        amqp.Publishing{
            ContentType: "application/json",
            Body:        eventBytes,
        })
    
    return nil
}

func (s *userService) generateJWT(userID uint, username string) (string, error) {
    claims := jwt.MapClaims{
        "user_id":  userID,
        "username": username,
        "exp":      time.Now().Add(time.Hour * time.Duration(util.GetJWTExpireHours())).Unix(),
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString([]byte(s.jwtSecret))
}

func (s *userService) GetUserByID(id uint) (*model.User, error) {
    return s.userRepo.GetByID(id)
}

func (s *userService) UpdateUser(req *UpdateUserRequest) (*model.User, error) {
    user, err := s.userRepo.GetByID(req.ID)
    if err != nil {
        return nil, exception.ErrUserNotFound
    }
    
    if req.Username != "" {
        user.Username = req.Username
    }
    if req.Email != "" {
        user.Email = req.Email
    }
    
    err = s.userRepo.Update(user)
    if err != nil {
        return nil, exception.WrapError(err, "更新用户失败")
    }
    
    return user, nil
}

func (s *userService) DeleteUser(id uint) error {
    return s.userRepo.Delete(id)
}

func (s *userService) ListUsers(req *ListUsersRequest) (*ListUsersResponse, error) {
    // 设置默认分页参数
    if req.Page <= 0 {
        req.Page = 1
    }
    if req.PageSize <= 0 {
        req.PageSize = 10
    }
    
    offset := (req.Page - 1) * req.PageSize
    
    users, err := s.userRepo.List(offset, req.PageSize)
    if err != nil {
        return nil, exception.WrapError(err, "获取用户列表失败")
    }
    
    // 这里可以添加总数查询逻辑
    total := int64(len(users)) // 简化处理
    
    return &ListUsersResponse{
        Users:    users,
        Total:    total,
        Page:     req.Page,
        PageSize: req.PageSize,
    }, nil
}
```

## 6. Controller 类 - 控制器层

定义业务逻辑，调用 Service 类中的方法，处理 HTTP 请求和响应。

### 代码实现示例：

``go
// controller/user_controller.go
package controller

import (
    "bookstore/exception"
    "bookstore/service"
    "net/http"
    "strconv"
    
    "github.com/gin-gonic/gin"
)

type UserController struct {
    userService service.UserService
}

func NewUserController(userService service.UserService) *UserController {
    return &UserController{
        userService: userService,
    }
}

// Register 用户注册
func (c *UserController) Register(ctx *gin.Context) {
    var req service.RegisterRequest
    if err := ctx.ShouldBindJSON(&req); err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": "参数错误: " + err.Error(),
        })
        return
    }
    
    user, err := c.userService.Register(&req)
    if err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": err.Error(),
        })
        return
    }
    
    ctx.JSON(http.StatusCreated, gin.H{
        "message": "注册成功",
        "data":    user,
    })
}

// Login 用户登录
func (c *UserController) Login(ctx *gin.Context) {
    var req service.LoginRequest
    if err := ctx.ShouldBindJSON(&req); err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": "参数错误: " + err.Error(),
        })
        return
    }
    
    response, err := c.userService.Login(&req)
    if err != nil {
        ctx.JSON(http.StatusUnauthorized, gin.H{
            "error": err.Error(),
        })
        return
    }
    
    ctx.JSON(http.StatusOK, gin.H{
        "message": "登录成功",
        "data":    response,
    })
}

// Logout 用户登出
func (c *UserController) Logout(ctx *gin.Context) {
    // 从 Header 中获取 token
    token := ctx.GetHeader("Authorization")
    if token == "" {
        ctx.JSON(http.StatusUnauthorized, gin.H{
            "error": "缺少认证令牌",
        })
        return
    }
    
    // 移除 "Bearer " 前缀
    if len(token) > 7 && token[:7] == "Bearer " {
        token = token[7:]
    }
    
    err := c.userService.Logout(token)
    if err != nil {
        ctx.JSON(http.StatusUnauthorized, gin.H{
            "error": err.Error(),
        })
        return
    }
    
    ctx.JSON(http.StatusOK, gin.H{
        "message": "登出成功",
    })
}

// GetUser 获取用户信息
func (c *UserController) GetUser(ctx *gin.Context) {
    idStr := ctx.Param("id")
    id, err := strconv.ParseUint(idStr, 10, 32)
    if err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": "无效的用户ID",
        })
        return
    }
    
    user, err := c.userService.GetUserByID(uint(id))
    if err != nil {
        ctx.JSON(http.StatusNotFound, gin.H{
            "error": "用户不存在",
        })
        return
    }
    
    ctx.JSON(http.StatusOK, gin.H{
        "data": user,
    })
}

// UpdateUser 更新用户信息
func (c *UserController) UpdateUser(ctx *gin.Context) {
    var req service.UpdateUserRequest
    if err := ctx.ShouldBindJSON(&req); err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": "参数错误: " + err.Error(),
        })
        return
    }
    
    user, err := c.userService.UpdateUser(&req)
    if err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": err.Error(),
        })
        return
    }
    
    ctx.JSON(http.StatusOK, gin.H{
        "message": "更新成功",
        "data":    user,
    })
}

// DeleteUser 删除用户
func (c *UserController) DeleteUser(ctx *gin.Context) {
    idStr := ctx.Param("id")
    id, err := strconv.ParseUint(idStr, 10, 32)
    if err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": "无效的用户ID",
        })
        return
    }
    
    err = c.userService.DeleteUser(uint(id))
    if err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": err.Error(),
        })
        return
    }
    
    ctx.JSON(http.StatusOK, gin.H{
        "message": "删除成功",
    })
}

// ListUsers 获取用户列表
func (c *UserController) ListUsers(ctx *gin.Context) {
    var req service.ListUsersRequest
    if err := ctx.ShouldBindQuery(&req); err != nil {
        ctx.JSON(http.StatusBadRequest, gin.H{
            "error": "参数错误: " + err.Error(),
        })
        return
    }
    
    response, err := c.userService.ListUsers(&req)
    if err != nil {
        ctx.JSON(http.StatusInternalServerError, gin.H{
            "error": err.Error(),
        })
        return
    }
    
    ctx.JSON(http.StatusOK, gin.H{
        "data": response,
    })
}
```

## 7. Router 类 - 路由层

定义路由规则，统一不同分组或业务逻辑，调用 Controller 类中的方法。

### 代码实现示例：

``go
// router/router.go
package router

import (
    "bookstore/controller"
    "bookstore/middleware"
    "bookstore/service"
    "bookstore/repository"
    "bookstore/init"
    
    "github.com/gin-gonic/gin"
)

func SetupRouter() *gin.Engine {
    r := gin.Default()
    
    // 全局中间件
    r.Use(middleware.Logger())
    r.Use(middleware.Recovery())
    r.Use(middleware.CORS())
    
    // 初始化依赖
    userRepo := repository.NewUserRepository(init.DB)
    userService := service.NewUserService(userRepo, init.Conf.JWT.SecretKey, init.MQConn)
    userController := controller.NewUserController(userService)
    
    // API 路由组
    api := r.Group("/api/v1")
    {
        // 公开路由（无需认证）
        public := api.Group("/auth")
        {
            public.POST("/register", userController.Register)
            public.POST("/login", userController.Login)
        }
        
        // 需要认证的路由
        protected := api.Group("/")
        protected.Use(middleware.JWTAuth(init.Conf.JWT.SecretKey))
        {
            // 用户登出路由
            protected.POST("/auth/logout", userController.Logout)
            
            // 用户相关路由
            users := protected.Group("/users")
            {
                users.GET("/", userController.ListUsers)
                users.GET("/:id", userController.GetUser)
                users.PUT("/:id", userController.UpdateUser)
                users.DELETE("/:id", userController.DeleteUser)
            }
            
            // 书籍相关路由（可以继续添加）
            // books := protected.Group("/books")
            // {
            //     books.GET("/", bookController.ListBooks)
            //     books.POST("/", bookController.CreateBook)
            //     books.GET("/:id", bookController.GetBook)
            //     books.PUT("/:id", bookController.UpdateBook)
            //     books.DELETE("/:id", bookController.DeleteBook)
            // }
        }
    }
    
    return r
}
```

## 8. Middleware 类 - 中间件层

定义中间件，如日志、权限验证、CORS 等。

### 代码实现示例：

``go
// middleware/middleware.go
package middleware

import (
    "bookstore/exception"
    "bookstore/util"
    "fmt"
    "net/http"
    "time"
    
    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v5"
)

// Logger 日志中间件
func Logger() gin.HandlerFunc {
    return gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {
        return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
            param.ClientIP,
            param.TimeStamp.Format(time.RFC1123),
            param.Method,
            param.Path,
            param.Request.Proto,
            param.StatusCode,
            param.Latency,
            param.Request.UserAgent(),
            param.ErrorMessage,
        )
    })
}

// Recovery 恢复中间件
func Recovery() gin.HandlerFunc {
    return gin.Recovery()
}

// CORS 跨域中间件
func CORS() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("Access-Control-Allow-Origin", "*")
        c.Header("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
        c.Header("Access-Control-Allow-Headers", "Origin, Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization")
        
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(204)
            return
        }
        
        c.Next()
    }
}

// JWTAuth JWT 认证中间件
func JWTAuth(jwtSecret string) gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.JSON(http.StatusUnauthorized, gin.H{
                "error": exception.ErrUnauthorized.Error(),
            })
            c.Abort()
            return
        }
        
        // 移除 "Bearer " 前缀
        if len(token) > 7 && token[:7] == "Bearer " {
            token = token[7:]
        }
        
        // 验证 JWT
        claims, err := util.ValidateJWT(token, jwtSecret)
        if err != nil {
            c.JSON(http.StatusUnauthorized, gin.H{
                "error": exception.ErrInvalidToken.Error(),
            })
            c.Abort()
            return
        }
        
        // 将用户信息存储到上下文中
        c.Set("user_id", claims["user_id"])
        c.Set("username", claims["username"])
        
        c.Next()
    }
}
```

## 9. Util 类 - 工具类

定义工具类，如字符串处理、时间处理、JWT 处理等。

### 代码实现示例：

``go
// util/jwt.go
package util

import (
    "bookstore/init"
    "errors"
    "time"
    
    "github.com/golang-jwt/jwt/v5"
)

func ValidateJWT(tokenString string, jwtSecret string) (jwt.MapClaims, error) {
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, errors.New("unexpected signing method")
        }
        return []byte(jwtSecret), nil
    })
    
    if err != nil {
        return nil, err
    }
    
    if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
        // 检查 token 是否过期
        if exp, ok := claims["exp"].(float64); ok {
            if time.Now().Unix() > int64(exp) {
                return nil, errors.New("token expired")
            }
        }
        return claims, nil
    }
    
    return nil, errors.New("invalid token")
}

func GetJWTExpireHours() int {
    if init.Conf != nil && init.Conf.JWT.ExpireHours > 0 {
        return init.Conf.JWT.ExpireHours
    }
    return 24 // 默认24小时
}

// util/response.go
package util

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

type Response struct {
    Code    int         `json:"code"`
    Message string      `json:"message"`
    Data    interface{} `json:"data,omitempty"`
}

func SuccessResponse(c *gin.Context, data interface{}) {
    c.JSON(http.StatusOK, Response{
        Code:    200,
        Message: "success",
        Data:    data,
    })
}

func ErrorResponse(c *gin.Context, code int, message string) {
    c.JSON(code, Response{
        Code:    code,
        Message: message,
    })
}

// util/validator.go
package util

import (
    "regexp"
    "strings"
)

func IsValidEmail(email string) bool {
    pattern := `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`
    matched, _ := regexp.MatchString(pattern, email)
    return matched
}

func IsValidUsername(username string) bool {
    if len(username) < 3 || len(username) > 20 {
        return false
    }
    pattern := `^[a-zA-Z0-9_]+$`
    matched, _ := regexp.MatchString(pattern, username)
    return matched
}

func TrimString(s string) string {
    return strings.TrimSpace(s)
}
```

## 10. Exception 类 - 异常处理

定义异常类，如参数错误、权限错误等。

### 代码实现示例：

``go
// exception/errors.go
package exception

import (
    "errors"
    "net/http"
)

// 自定义错误类型
type AppError struct {
    Code    int    `json:"code"`
    Message string `json:"message"`
    Err     error  `json:"-"`
}

func (e *AppError) Error() string {
    return e.Message
}

// 预定义错误
var (
    ErrInvalidParams    = &AppError{Code: http.StatusBadRequest, Message: "参数错误"}
    ErrUnauthorized     = &AppError{Code: http.StatusUnauthorized, Message: "未授权"}
    ErrForbidden        = &AppError{Code: http.StatusForbidden, Message: "禁止访问"}
    ErrNotFound         = &AppError{Code: http.StatusNotFound, Message: "资源不存在"}
    ErrInternalError    = &AppError{Code: http.StatusInternalServerError, Message: "内部服务器错误"}
    ErrUserExists       = &AppError{Code: http.StatusConflict, Message: "用户已存在"}
    ErrUserNotFound     = &AppError{Code: http.StatusNotFound, Message: "用户不存在"}
    ErrInvalidPassword  = &AppError{Code: http.StatusUnauthorized, Message: "用户名或密码错误"}
    ErrTokenExpired     = &AppError{Code: http.StatusUnauthorized, Message: "令牌已过期"}
    ErrInvalidToken     = &AppError{Code: http.StatusUnauthorized, Message: "无效的令牌"}
)

// 创建自定义错误
func NewAppError(code int, message string, err error) *AppError {
    return &AppError{
        Code:    code,
        Message: message,
        Err:     err,
    }
}

// 包装错误
func WrapError(err error, message string) *AppError {
    return &AppError{
        Code:    http.StatusInternalServerError,
        Message: message,
        Err:     err,
    }
}
```

## 11. MQ 消费者 - 消息队列处理

处理来自消息队列的事件。

### 代码实现示例：

```go
// consumer/user_consumer.go
package consumer

import (
    "bookstore/init"
    "encoding/json"
    "log"
    
    "github.com/streadway/amqp"
)

type UserEvent struct {
    Event     string      `json:"event"`
    UserID    interface{} `json:"user_id"`
    Username  string      `json:"username"`
    Timestamp string      `json:"timestamp"`
}

func StartUserEventConsumer() {
    ch, err := init.MQConn.Channel()
    if err != nil {
        log.Fatal(err)
    }
    defer ch.Close()
    
    q, err := ch.QueueDeclare(
        "user_events", // 队列名称
        true,          // 持久化
        false,         // 自动删除
        false,         // 独占
        false,         // 不等待
        nil,           // 参数
    )
    if err != nil {
        log.Fatal(err)
    }
    
    msgs, err := ch.Consume(
        q.Name, // 队列
        "",     // 消费者
        true,   // 自动应答
        false,  // 独占
        false,  // 不等待
        false,  // 参数
        nil,    // 参数
    )
    if err != nil {
        log.Fatal(err)
    }
    
    forever := make(chan bool)
    
    go func() {
        for d := range msgs {
            // 处理用户事件
            var event UserEvent
            err := json.Unmarshal(d.Body, &event)
            if err != nil {
                log.Printf("Error unmarshalling event: %v", err)
                continue
            }
            
            switch event.Event {
            case "user_registered":
                log.Printf("用户注册事件: 用户ID=%v, 用户名=%s", event.UserID, event.Username)
                // 可以在这里添加发送欢迎邮件等业务逻辑
                
            case "user_logged_in":
                log.Printf("用户登录事件: 用户ID=%v, 用户名=%s", event.UserID, event.Username)
                // 可以在这里添加登录日志记录等业务逻辑
                
            case "user_logged_out":
                log.Printf("用户登出事件: 用户ID=%v, 用户名=%s", event.UserID, event.Username)
                // 可以在这里添加登出日志记录等业务逻辑
                
            default:
                log.Printf("未知用户事件: %s", event.Event)
            }
        }
    }()
    
    log.Printf("等待用户事件消息。按 CTRL+C 退出")
    <-chan bool)
    
    <-forever
}

```

## 12. Main 函数 - 程序入口

调用 Router 类中的方法，启动服务。

### 代码实现示例：

```go
// main.go
package main

import (
    "bookstore/consumer"
    "bookstore/init"
    "bookstore/router"
    "log"
    "os"
    "os/signal"
    "syscall"
)

func main() {
    // 初始化所有组件
    err := init.InitAll("config/config.yaml")
    if err != nil {
        log.Fatalf("初始化失败: %v", err)
    }
    
    // 确保程序退出时关闭连接
    defer init.Close()
    
    // 启动消息队列消费者
    go consumer.StartUserEventConsumer()
    
    // 设置路由
    r := router.SetupRouter()
    
    // 启动服务器
    port := init.Conf.Server.Port
    if port == "" {
        port = "8080"
    }
    
    // 创建一个通道来接收系统信号
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    
    // 在 goroutine 中启动服务器
    go func() {
        log.Printf("服务器启动在端口 %s", port)
        if err := r.Run(":" + port); err != nil {
            log.Fatalf("服务器启动失败: %v", err)
        }
    }()
    
    // 等待中断信号以优雅地关闭服务器
    <-quit
    log.Println("正在关闭服务器...")
}
```

## 调用流程说明

### 1. 用户注册流程

```
客户端请求 → Router → Controller → Service → Repository → Database
                ↓
            响应返回 ← Router ← Controller ← Service ← Repository ← Database
```

**详细步骤：**

1. **客户端** 发送 POST 请求到 `/api/v1/auth/register`
2. **Router** 接收请求，路由到 `UserController.Register`
3. **Controller** 解析 JSON 请求体到 `RegisterRequest` 结构体
4. **Controller** 调用 `UserService.Register` 方法
5. **Service** 验证用户名和邮箱唯一性
6. **Service** 调用 `UserRepository.Create` 创建用户
7. **Repository** 执行数据库插入操作
8. **Service** 发送用户注册事件到消息队列
9. **返回** 创建成功的用户信息

### 2. 用户登录流程

```
客户端请求 → Router → Controller → Service → Repository → Database
                ↓
            响应返回 ← Router ← Controller ← Service ← Repository ← Database
```

**详细步骤：**

1. **客户端** 发送 POST 请求到 `/api/v1/auth/login`
2. **Router** 接收请求，路由到 `UserController.Login`
3. **Controller** 解析 JSON 请求体到 `LoginRequest` 结构体
4. **Controller** 调用 `UserService.Login` 方法
5. **Service** 根据用户名查找用户
6. **Service** 验证密码
7. **Service** 生成 JWT Token
8. **Service** 发送用户登录事件到消息队列
9. **返回** Token 和用户信息

### 3. 用户登出流程

```
客户端请求 → Router → Middleware → Controller → Service
                ↓
            响应返回 ← Router ← Controller ← Service
```

**详细步骤：**

1. **客户端** 发送 POST 请求到 `/api/v1/auth/logout`，携带 JWT Token
2. **Router** 接收请求，通过 `JWTAuth` 中间件验证 Token
3. **Middleware** 验证 JWT Token 有效性，提取用户信息
4. **Controller** 调用 `UserService.Logout` 方法
5. **Service** 发送用户登出事件到消息队列
6. **返回** 登出成功信息

### 4. 获取用户信息流程（需要认证）

```
客户端请求 → Router → Middleware → Controller → Service → Repository → Database
                ↓
            响应返回 ← Router ← Controller ← Service ← Repository ← Database
```

**详细步骤：**

1. **客户端** 发送 GET 请求到 `/api/v1/users/:id`，携带 JWT Token
2. **Router** 接收请求，通过 `JWTAuth` 中间件验证 Token
3. **Middleware** 验证 JWT Token 有效性，提取用户信息
4. **Controller** 解析路径参数，调用 `UserService.GetUserByID`
5. **Service** 调用 `UserRepository.GetByID` 查询用户
6. **Repository** 执行数据库查询操作
7. **返回** 用户信息

## 项目结构总结

```
bookstore/
├── main.go                 # 程序入口
├── config/
│   ├── config.go          # 配置管理
│   └── config.yaml        # 配置文件
├── init/
│   └── init.go            # 初始化模块
├── model/
│   ├── user.go            # 用户模型
│   └── book.go            # 书籍模型
├── repository/
│   ├── user_repository.go # 用户数据访问层
│   └── book_repository.go # 书籍数据访问层
├── service/
│   ├── user_service.go    # 用户业务逻辑层
│   └── book_service.go    # 书籍业务逻辑层
├── controller/
│   ├── user_controller.go # 用户控制器
│   └── book_controller.go # 书籍控制器
├── router/
│   └── router.go          # 路由配置
├── middleware/
│   └── middleware.go      # 中间件
├── util/
│   ├── jwt.go            # JWT 工具
│   ├── response.go       # 响应工具
│   └── validator.go      # 验证工具
├── consumer/
│   └── user_consumer.go  # 消息队列消费者
└── exception/
    └── errors.go         # 异常定义
```

## 新增功能说明

### JWT 认证机制

1. **登录时生成 Token**：用户登录成功后，系统生成包含用户信息和过期时间的 JWT Token
2. **请求时验证 Token**：通过中间件验证每个受保护路由的 Token 有效性
3. **登出处理**：支持用户主动登出，将 Token 加入黑名单（示例中简化处理）

### 消息队列集成

1. **事件发布**：在用户注册、登录、登出等关键操作时发布事件到 RabbitMQ
2. **事件消费**：通过消费者程序处理用户事件，可用于日志记录、发送邮件等异步任务
3. **解耦设计**：将核心业务逻辑与辅助功能（如日志、通知）解耦，提高系统可扩展性

### 完整的用户功能

1. **用户注册**：包括数据验证、密码加密、唯一性检查
2. **用户登录**：密码验证、Token 生成
3. **用户登出**：Token 处理、事件发布
4. **信息查询**：根据 ID 获取用户信息
5. **信息更新**：修改用户资料
6. **用户删除**：删除用户账号

这种架构设计具有以下优势：

1. **安全性**：使用 JWT 进行身份验证，密码加密存储
2. **可扩展性**：通过消息队列实现异步处理，便于扩展功能
3. **职责分离**：每层都有明确的职责，便于维护和测试
4. **松耦合**：层与层之间通过接口交互，降低耦合度
5. **可测试性**：每层都可以独立进行单元测试
6. **代码复用**：工具类和中间件可以在多个地方复用