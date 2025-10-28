# Kubernetes多集群资源管理项目框架梳理

## 项目架构概述

本项目是一个基于Go语言开发的Kubernetes多集群资源管理系统，旨在提供统一的界面和API来管理多个Kubernetes集群中的资源。该项目支持跨集群资源部署、监控、运维操作，提供资源同步、策略管理、统一认证等功能，帮助企业实现多云和混合云环境下的Kubernetes资源统一管理。

## 技术栈

- **核心语言**：Golang 1.19+
- **Web框架**：Gin
- **Kubernetes客户端**：client-go
- **数据库**：PostgreSQL/MySQL
- **缓存**：Redis
- **消息队列**：Kafka/RabbitMQ
- **配置管理**：Viper
- **日志系统**：Zap
- **监控系统**：Prometheus + Grafana
- **认证授权**：RBAC + JWT
- **部署平台**：Kubernetes
- **构建工具**：Go Modules

## 项目结构

```
kubernetes-multi-cluster-manager/
├── cmd/
│   └── manager/
│       └── main.go
├── internal/
│   ├── app/
│   │   └── manager/
│   │       ├── app.go
│   │       └── options.go
│   ├── pkg/
│   │   ├── config/
│   │   │   ├── config.go
│   │   │   └── types.go
│   │   ├── cluster/
│   │   │   ├── manager.go
│   │   │   ├── client.go
│   │   │   ├── registry.go
│   │   │   └── health.go
│   │   ├── resource/
│   │   │   ├── controller.go
│   │   │   ├── sync.go
│   │   │   ├── manager.go
│   │   │   └── cache.go
│   │   ├── auth/
│   │   │   ├── jwt.go
│   │   │   ├── rbac.go
│   │   │   └── middleware.go
│   │   ├── api/
│   │   │   ├── v1/
│   │   │   │   ├── cluster.go
│   │   │   │   ├── namespace.go
│   │   │   │   ├── deployment.go
│   │   │   │   ├── service.go
│   │   │   │   └── middleware.go
│   │   │   └── server.go
│   │   ├── policy/
│   │   │   ├── engine.go
│   │   │   ├── validator.go
│   │   │   └── enforcer.go
│   │   ├── notification/
│   │   │   ├── dispatcher.go
│   │   │   └── channel.go
│   │   ├── metrics/
│   │   │   ├── collector.go
│   │   │   └── exporter.go
│   │   └── utils/
│   │       ├── crypto.go
│   │       ├── converter.go
│   │       └── validator.go
│   └── test/
│       └── integration/
├── configs/
│   ├── config.yaml
│   └── rbac-policies.csv
├── deploy/
│   ├── kubernetes/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   └── rbac.yaml
│   └── docker/
│       └── Dockerfile
├── docs/
├── scripts/
├── go.mod
├── go.sum
└── README.md
```

## 核心组件详解

### 1. 主程序入口 (cmd/manager/main.go)

```go
package main

import (
	"os"
	"os/signal"
	"syscall"

	"k8s-multi-cluster-manager/internal/app/manager"
)

func main() {
	// 创建应用实例
	app := manager.NewManagerApp()
	
	// 运行应用
	if err := app.Run(); err != nil {
		panic(err)
	}
	
	// 等待中断信号以优雅地关闭应用
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit
	
	// 关闭应用
	if err := app.Stop(); err != nil {
		panic(err)
	}
}
```

### 2. 应用配置 (internal/pkg/config/)

#### 配置类型定义 (types.go)

```go
package config

type Config struct {
	Server      ServerConfig      `mapstructure:"server"`
	Database    DatabaseConfig    `mapstructure:"database"`
	Redis       RedisConfig       `mapstructure:"redis"`
	Kubernetes  KubernetesConfig  `mapstructure:"kubernetes"`
	Auth        AuthConfig        `mapstructure:"auth"`
	Metrics     MetricsConfig     `mapstructure:"metrics"`
	Log         LogConfig         `mapstructure:"log"`
	Clusters    []ClusterConfig   `mapstructure:"clusters"`
}

type ServerConfig struct {
	Addr         string `mapstructure:"addr"`
	ReadTimeout  int    `mapstructure:"read_timeout"`
	WriteTimeout int    `mapstructure:"write_timeout"`
	IdleTimeout  int    `mapstructure:"idle_timeout"`
}

type DatabaseConfig struct {
	Driver   string `mapstructure:"driver"`
	Host     string `mapstructure:"host"`
	Port     int    `mapstructure:"port"`
	Username string `mapstructure:"username"`
	Password string `mapstructure:"password"`
	Database string `mapstructure:"database"`
}

type RedisConfig struct {
	Addr     string `mapstructure:"addr"`
	Password string `mapstructure:"password"`
	DB       int    `mapstructure:"db"`
}

type KubernetesConfig struct {
	KubeConfig string `mapstructure:"kubeconfig"`
	MasterURL  string `mapstructure:"master_url"`
	ResyncPeriod int  `mapstructure:"resync_period"`
}

type AuthConfig struct {
	JWTSecret     string `mapstructure:"jwt_secret"`
	TokenLifetime int    `mapstructure:"token_lifetime"`
}

type MetricsConfig struct {
	Enabled bool   `mapstructure:"enabled"`
	Addr    string `mapstructure:"addr"`
}

type LogConfig struct {
	Level      string `mapstructure:"level"`
	Format     string `mapstructure:"format"`
	FilePath   string `mapstructure:"file_path"`
	MaxSize    int    `mapstructure:"max_size"`
	MaxBackups int    `mapstructure:"max_backups"`
	MaxAge     int    `mapstructure:"max_age"`
}

type ClusterConfig struct {
	Name        string `mapstructure:"name"`
	DisplayName string `mapstructure:"display_name"`
	KubeConfig  string `mapstructure:"kubeconfig"`
	MasterURL   string `mapstructure:"master_url"`
	Enabled     bool   `mapstructure:"enabled"`
}
```

#### 配置加载 (config.go)

```go
package config

import (
	"github.com/spf13/viper"
)

func LoadConfig(configPath string) (*Config, error) {
	viper.SetConfigFile(configPath)
	
	if err := viper.ReadInConfig(); err != nil {
		return nil, err
	}
	
	var config Config
	if err := viper.Unmarshal(&config); err != nil {
		return nil, err
	}
	
	return &config, nil
}
```

### 3. 集群管理 (internal/pkg/cluster/)

#### 集群管理器 (manager.go)

```go
package cluster

import (
	"context"
	"sync"
	"time"

	"go.uber.org/zap"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/rest"
	"k8s.io/client-go/tools/clientcmd"
)

type Manager struct {
	clusters map[string]*Cluster
	mutex    sync.RWMutex
	logger   *zap.Logger
}

type Cluster struct {
	Name        string
	DisplayName string
	Client      kubernetes.Interface
	Config      *rest.Config
	Enabled     bool
	LastCheck   time.Time
	Healthy     bool
}

func NewManager(logger *zap.Logger) *Manager {
	return &Manager{
		clusters: make(map[string]*Cluster),
		logger:   logger,
	}
}

func (m *Manager) AddCluster(name, displayName, kubeconfig string, enabled bool) error {
	m.mutex.Lock()
	defer m.mutex.Unlock()
	
	config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
	if err != nil {
		return err
	}
	
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		return err
	}
	
	cluster := &Cluster{
		Name:        name,
		DisplayName: displayName,
		Client:      clientset,
		Config:      config,
		Enabled:     enabled,
		LastCheck:   time.Now(),
		Healthy:     true,
	}
	
	m.clusters[name] = cluster
	m.logger.Info("Cluster added", zap.String("name", name))
	
	return nil
}

func (m *Manager) RemoveCluster(name string) {
	m.mutex.Lock()
	defer m.mutex.Unlock()
	
	delete(m.clusters, name)
	m.logger.Info("Cluster removed", zap.String("name", name))
}

func (m *Manager) GetCluster(name string) (*Cluster, error) {
	m.mutex.RLock()
	defer m.mutex.RUnlock()
	
	cluster, exists := m.clusters[name]
	if !exists {
		return nil, ErrClusterNotFound
	}
	
	return cluster, nil
}

func (m *Manager) ListClusters() []*Cluster {
	m.mutex.RLock()
	defer m.mutex.RUnlock()
	
	var clusters []*Cluster
	for _, cluster := range m.clusters {
		clusters = append(clusters, cluster)
	}
	
	return clusters
}

func (m *Manager) HealthCheck() {
	m.mutex.RLock()
	defer m.mutex.RUnlock()
	
	for _, cluster := range m.clusters {
		if !cluster.Enabled {
			continue
		}
		
		go m.checkClusterHealth(cluster)
	}
}

func (m *Manager) checkClusterHealth(cluster *Cluster) {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	
	_, err := cluster.Client.CoreV1().Namespaces().List(ctx, metav1.ListOptions{Limit: 1})
	
	cluster.LastCheck = time.Now()
	if err != nil {
		cluster.Healthy = false
		m.logger.Error("Cluster health check failed", 
			zap.String("name", cluster.Name), 
			zap.Error(err))
	} else {
		cluster.Healthy = true
		m.logger.Info("Cluster health check passed", 
			zap.String("name", cluster.Name))
	}
}
```

#### 集群客户端 (client.go)

```go
package cluster

import (
	"context"
	"time"

	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
)

type Client struct {
	clientset kubernetes.Interface
	timeout   time.Duration
}

func NewClient(clientset kubernetes.Interface) *Client {
	return &Client{
		clientset: clientset,
		timeout:   30 * time.Second,
	}
}

func (c *Client) ListNamespaces() (*corev1.NamespaceList, error) {
	ctx, cancel := context.WithTimeout(context.Background(), c.timeout)
	defer cancel()
	
	return c.clientset.CoreV1().Namespaces().List(ctx, metav1.ListOptions{})
}

func (c *Client) ListDeployments(namespace string) (*appsv1.DeploymentList, error) {
	ctx, cancel := context.WithTimeout(context.Background(), c.timeout)
	defer cancel()
	
	return c.clientset.AppsV1().Deployments(namespace).List(ctx, metav1.ListOptions{})
}

func (c *Client) ListServices(namespace string) (*corev1.ServiceList, error) {
	ctx, cancel := context.WithTimeout(context.Background(), c.timeout)
	defer cancel()
	
	return c.clientset.CoreV1().Services(namespace).List(ctx, metav1.ListOptions{})
}

func (c *Client) GetNodeMetrics() (*metav1.APIResourceList, error) {
	// 这里需要metrics API客户端
	// 实现获取节点指标的逻辑
	return nil, nil
}
```

### 4. 资源管理 (internal/pkg/resource/)

#### 资源控制器 (controller.go)

```go
package resource

import (
	"context"
	"time"

	"go.uber.org/zap"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/watch"
	"k8s.io/client-go/tools/cache"
	"k8s.io/client-go/util/workqueue"
)

type Controller struct {
	name       string
	client     Interface
	informer   cache.SharedIndexInformer
	workqueue  workqueue.RateLimitingInterface
	handler    ResourceHandler
	logger     *zap.Logger
}

type ResourceHandler interface {
	OnAdd(obj interface{})
	OnUpdate(oldObj, newObj interface{})
	OnDelete(obj interface{})
}

func NewController(
	name string,
	client Interface,
	objType runtime.Object,
	listFunc cache.ListFunc,
	watchFunc cache.WatchFunc,
	handler ResourceHandler,
	logger *zap.Logger,
) *Controller {
	
	informer := cache.NewSharedIndexInformer(
		&cache.ListWatch{
			ListFunc: listFunc,
			WatchFunc: watchFunc,
		},
		objType,
		time.Second*30,
		cache.Indexers{},
	)
	
	controller := &Controller{
		name:      name,
		client:    client,
		informer:  informer,
		workqueue: workqueue.NewNamedRateLimitingQueue(workqueue.DefaultControllerRateLimiter(), name),
		handler:   handler,
		logger:    logger,
	}
	
	informer.AddEventHandler(cache.ResourceEventHandlerFuncs{
		AddFunc: controller.enqueueObject,
		UpdateFunc: func(old, new interface{}) {
			controller.enqueueObject(new)
		},
		DeleteFunc: controller.enqueueObject,
	})
	
	return controller
}

func (c *Controller) Run(stopCh <-chan struct{}) error {
	defer c.workqueue.ShutDown()
	
	c.logger.Info("Starting controller", zap.String("name", c.name))
	
	go c.informer.Run(stopCh)
	
	if !cache.WaitForCacheSync(stopCh, c.informer.HasSynced) {
		return nil
	}
	
	go c.runWorker()
	
	<-stopCh
	c.logger.Info("Stopping controller", zap.String("name", c.name))
	
	return nil
}

func (c *Controller) enqueueObject(obj interface{}) {
	var key string
	var err error
	if key, err = cache.MetaNamespaceKeyFunc(obj); err != nil {
		c.logger.Error("Failed to get key for object", zap.Error(err))
		return
	}
	c.workqueue.Add(key)
}

func (c *Controller) runWorker() {
	for c.processNextWorkItem() {
	}
}

func (c *Controller) processNextWorkItem() bool {
	obj, shutdown := c.workqueue.Get()
	
	if shutdown {
		return false
	}
	
	err := func(obj interface{}) error {
		defer c.workqueue.Done(obj)
		var key string
		var ok bool
		
		if key, ok = obj.(string); !ok {
			c.workqueue.Forget(obj)
			return nil
		}
		
		if err := c.syncHandler(key); err != nil {
			c.workqueue.AddRateLimited(key)
			return err
		}
		
		c.workqueue.Forget(obj)
		return nil
	}(obj)
	
	if err != nil {
		c.logger.Error("Failed to process work item", zap.Error(err))
		return true
	}
	
	return true
}

func (c *Controller) syncHandler(key string) error {
	obj, exists, err := c.informer.GetIndexer().GetByKey(key)
	if err != nil {
		return err
	}
	
	if !exists {
		c.handler.OnDelete(key)
	} else {
		c.handler.OnAdd(obj)
	}
	
	return nil
}
```

#### 资源同步器 (sync.go)

```go
package resource

import (
	"context"
	"sync"
	"time"

	"go.uber.org/zap"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
)

type SyncManager struct {
	clusters map[string]kubernetes.Interface
	resources map[string]ResourceSyncer
	mutex sync.RWMutex
	logger *zap.Logger
	interval time.Duration
}

type ResourceSyncer interface {
	Sync(ctx context.Context) error
	GetResourceType() string
}

func NewSyncManager(logger *zap.Logger, interval time.Duration) *SyncManager {
	return &SyncManager{
		clusters: make(map[string]kubernetes.Interface),
		resources: make(map[string]ResourceSyncer),
		logger: logger,
		interval: interval,
	}
}

func (sm *SyncManager) AddCluster(name string, client kubernetes.Interface) {
	sm.mutex.Lock()
	defer sm.mutex.Unlock()
	
	sm.clusters[name] = client
}

func (sm *SyncManager) RemoveCluster(name string) {
	sm.mutex.Lock()
	defer sm.mutex.Unlock()
	
	delete(sm.clusters, name)
}

func (sm *SyncManager) AddResourceSyncer(name string, syncer ResourceSyncer) {
	sm.mutex.Lock()
	defer sm.mutex.Unlock()
	
	sm.resources[name] = syncer
}

func (sm *SyncManager) Start(ctx context.Context) {
	ticker := time.NewTicker(sm.interval)
	defer ticker.Stop()
	
	for {
		select {
		case <-ticker.C:
			sm.syncAllResources()
		case <-ctx.Done():
			return
		}
	}
}

func (sm *SyncManager) syncAllResources() {
	sm.mutex.RLock()
	defer sm.mutex.RUnlock()
	
	ctx := context.Background()
	
	for name, syncer := range sm.resources {
		go func(name string, syncer ResourceSyncer) {
			sm.logger.Info("Starting resource sync", zap.String("resource", name))
			
			if err := syncer.Sync(ctx); err != nil {
				sm.logger.Error("Failed to sync resource", 
					zap.String("resource", name), 
					zap.Error(err))
			} else {
				sm.logger.Info("Resource sync completed", zap.String("resource", name))
			}
		}(name, syncer)
	}
}
```

### 5. 认证授权 (internal/pkg/auth/)

#### JWT认证 (jwt.go)

```go
package auth

import (
	"time"

	"github.com/golang-jwt/jwt/v4"
)

type Claims struct {
	Username string   `json:"username"`
	Roles    []string `json:"roles"`
	jwt.RegisteredClaims
}

type JWTManager struct {
	secretKey     string
	tokenDuration time.Duration
}

func NewJWTManager(secretKey string, tokenDuration time.Duration) *JWTManager {
	return &JWTManager{
		secretKey:     secretKey,
		tokenDuration: tokenDuration,
	}
}

func (jm *JWTManager) GenerateToken(username string, roles []string) (string, error) {
	claims := Claims{
		Username: username,
		Roles:    roles,
		RegisteredClaims: jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(jm.tokenDuration)),
			IssuedAt:  jwt.NewNumericDate(time.Now()),
		},
	}
	
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString([]byte(jm.secretKey))
}

func (jm *JWTManager) VerifyToken(tokenString string) (*Claims, error) {
	token, err := jwt.ParseWithClaims(
		tokenString,
		&Claims{},
		func(token *jwt.Token) (interface{}, error) {
			return []byte(jm.secretKey), nil
		},
	)
	
	if err != nil {
		return nil, err
	}
	
	claims, ok := token.Claims.(*Claims)
	if !ok {
		return nil, err
	}
	
	return claims, nil
}
```

#### RBAC权限控制 (rbac.go)

```go
package auth

import (
	"github.com/casbin/casbin/v2"
	"go.uber.org/zap"
)

type RBACManager struct {
	enforcer *casbin.Enforcer
	logger   *zap.Logger
}

func NewRBACManager(modelPath, policyPath string, logger *zap.Logger) (*RBACManager, error) {
	enforcer, err := casbin.NewEnforcer(modelPath, policyPath)
	if err != nil {
		return nil, err
	}
	
	return &RBACManager{
		enforcer: enforcer,
		logger:   logger,
	}, nil
}

func (rm *RBACManager) CheckPermission(user, resource, action string) bool {
	allowed, err := rm.enforcer.Enforce(user, resource, action)
	if err != nil {
		rm.logger.Error("Failed to enforce policy", zap.Error(err))
		return false
	}
	
	return allowed
}

func (rm *RBACManager) AddPolicy(user, resource, action string) error {
	_, err := rm.enforcer.AddPolicy(user, resource, action)
	return err
}

func (rm *RBACManager) RemovePolicy(user, resource, action string) error {
	_, err := rm.enforcer.RemovePolicy(user, resource, action)
	return err
}
```

### 6. API接口 (internal/pkg/api/v1/)

#### 集群管理API (cluster.go)

```go
package v1

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"k8s-multi-cluster-manager/internal/pkg/cluster"
)

type ClusterAPI struct {
	manager *cluster.Manager
}

func NewClusterAPI(manager *cluster.Manager) *ClusterAPI {
	return &ClusterAPI{
		manager: manager,
	}
}

func (ca *ClusterAPI) ListClusters(c *gin.Context) {
	clusters := ca.manager.ListClusters()
	
	// 转换为API响应格式
	var response []map[string]interface{}
	for _, cluster := range clusters {
		response = append(response, map[string]interface{}{
			"name":         cluster.Name,
			"display_name": cluster.DisplayName,
			"enabled":      cluster.Enabled,
			"healthy":      cluster.Healthy,
			"last_check":   cluster.LastCheck,
		})
	}
	
	c.JSON(http.StatusOK, gin.H{
		"clusters": response,
	})
}

func (ca *ClusterAPI) GetCluster(c *gin.Context) {
	name := c.Param("name")
	
	cluster, err := ca.manager.GetCluster(name)
	if err != nil {
		c.JSON(http.StatusNotFound, gin.H{
			"error": "cluster not found",
		})
		return
	}
	
	response := map[string]interface{}{
		"name":         cluster.Name,
		"display_name": cluster.DisplayName,
		"enabled":      cluster.Enabled,
		"healthy":      cluster.Healthy,
		"last_check":   cluster.LastCheck,
	}
	
	c.JSON(http.StatusOK, response)
}

func (ca *ClusterAPI) AddCluster(c *gin.Context) {
	var req struct {
		Name        string `json:"name" binding:"required"`
		DisplayName string `json:"display_name"`
		KubeConfig  string `json:"kubeconfig" binding:"required"`
		Enabled     bool   `json:"enabled"`
	}
	
	if err := c.ShouldBindJSON(&req); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{
			"error": err.Error(),
		})
		return
	}
	
	err := ca.manager.AddCluster(req.Name, req.DisplayName, req.KubeConfig, req.Enabled)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{
			"error": err.Error(),
		})
		return
	}
	
	c.JSON(http.StatusCreated, gin.H{
		"message": "cluster added successfully",
	})
}

func (ca *ClusterAPI) RemoveCluster(c *gin.Context) {
	name := c.Param("name")
	
	ca.manager.RemoveCluster(name)
	
	c.JSON(http.StatusOK, gin.H{
		"message": "cluster removed successfully",
	})
}
```

## 配置文件详解

### 1. 主配置文件 (configs/config.yaml)

```yaml
server:
  addr: ":8080"
  read_timeout: 30
  write_timeout: 30
  idle_timeout: 60

database:
  driver: "postgres"
  host: "localhost"
  port: 5432
  username: "admin"
  password: "password"
  database: "k8s_manager"

redis:
  addr: "localhost:6379"
  password: ""
  db: 0

kubernetes:
  kubeconfig: ""
  master_url: ""
  resync_period: 30

auth:
  jwt_secret: "your-jwt-secret-key"
  token_lifetime: 86400

metrics:
  enabled: true
  addr: ":9090"

log:
  level: "info"
  format: "json"
  file_path: "/var/log/manager.log"
  max_size: 100
  max_backups: 3
  max_age: 7

clusters:
  - name: "cluster1"
    display_name: "Production Cluster"
    kubeconfig: "/etc/kubeconfig/cluster1.yaml"
    enabled: true
  - name: "cluster2"
    display_name: "Staging Cluster"
    kubeconfig: "/etc/kubeconfig/cluster2.yaml"
    enabled: true
```

### 2. RBAC策略文件 (configs/rbac-policies.csv)

```csv
p, admin, *, *
p, developer, deployments, read
p, developer, services, read
p, operator, deployments, *
p, operator, services, *
p, operator, pods, read
```

## 部署配置

### 1. Kubernetes部署文件 (deploy/kubernetes/deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-multi-cluster-manager
  labels:
    app: k8s-multi-cluster-manager
spec:
  replicas: 2
  selector:
    matchLabels:
      app: k8s-multi-cluster-manager
  template:
    metadata:
      labels:
        app: k8s-multi-cluster-manager
    spec:
      serviceAccountName: manager-service-account
      containers:
      - name: manager
        image: k8s-multi-cluster-manager:latest
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 9090
          name: metrics
        volumeMounts:
        - name: config
          mountPath: /etc/manager
        - name: kubeconfig
          mountPath: /etc/kubeconfig
        env:
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 20
      volumes:
      - name: config
        configMap:
          name: manager-config
      - name: kubeconfig
        secret:
          secretName: kubeconfig-secrets
```

### 2. RBAC配置 (deploy/kubernetes/rbac.yaml)

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: manager-service-account

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: manager-cluster-role
rules:
- apiGroups: [""]
  resources: ["namespaces", "pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: manager-cluster-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: manager-cluster-role
subjects:
- kind: ServiceAccount
  name: manager-service-account
  namespace: default
```

## 核心功能实现

### 1. 多集群统一管理

通过集群管理器实现对多个Kubernetes集群的统一管理：

```go
func (m *Manager) AddCluster(name, displayName, kubeconfig string, enabled bool) error {
	// 构建集群配置
	config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
	if err != nil {
		return err
	}
	
	// 创建客户端
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		return err
	}
	
	// 注册集群
	cluster := &Cluster{
		Name:        name,
		DisplayName: displayName,
		Client:      clientset,
		Config:      config,
		Enabled:     enabled,
	}
	
	m.clusters[name] = cluster
	return nil
}
```

### 2. 资源同步机制

实现跨集群资源状态同步：

```go
func (sm *SyncManager) syncAllResources() {
	// 并发同步所有集群的资源状态
	for name, syncer := range sm.resources {
		go func(name string, syncer ResourceSyncer) {
			if err := syncer.Sync(context.Background()); err != nil {
				// 记录错误日志
			}
		}(name, syncer)
	}
}
```

### 3. 统一认证授权

通过JWT和RBAC实现统一的认证授权机制：

```go
func (jm *JWTManager) GenerateToken(username string, roles []string) (string, error) {
	// 生成包含用户信息和角色的JWT令牌
	claims := Claims{
		Username: username,
		Roles:    roles,
		// ... 其他声明
	}
	
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString([]byte(jm.secretKey))
}
```

## 监控与运维

### 1. 指标暴露

通过Prometheus暴露关键指标：

```go
type Collector struct {
	clusterHealth *prometheus.GaugeVec
	resourceCount *prometheus.GaugeVec
	apiRequests   *prometheus.CounterVec
}

func (c *Collector) Collect(ch chan<- prometheus.Metric) {
	// 收集集群健康状态指标
	// 收集资源数量指标
	// 收集API请求指标
}
```

### 2. 健康检查

提供健康检查端点：

```go
func (app *ManagerApp) setupHealthChecks() {
	app.router.GET("/healthz", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"status": "healthy",
		})
	})
	
	app.router.GET("/ready", func(c *gin.Context) {
		if app.isReady() {
			c.JSON(200, gin.H{
				"status": "ready",
			})
		} else {
			c.JSON(503, gin.H{
				"status": "not ready",
			})
		}
	})
}
```

## 扩展性设计

### 1. 插件机制

支持动态加载插件：

```go
type PluginManager struct {
	plugins map[string]Plugin
}

func (pm *PluginManager) LoadPlugin(path string) error {
	// 动态加载插件
	return nil
}
```

### 2. 自定义资源控制器

支持自定义资源的管理：

```go
func NewCustomResourceController(
	client Interface,
	crdType runtime.Object,
	handler ResourceHandler,
) *Controller {
	// 创建自定义资源控制器
	return &Controller{
		// ... 初始化
	}
}
```

## 总结

基于Go语言开发的Kubernetes多集群资源管理项目具有以下优势：

1. **统一管理**：提供统一界面管理多个Kubernetes集群
2. **高性能**：基于Go语言和Gin框架，性能优异
3. **可扩展性**：插件化架构，易于扩展功能
4. **安全性**：集成JWT和RBAC认证授权机制
5. **可观测性**：集成Prometheus监控和日志系统
6. **高可用性**：支持多实例部署，保证服务可用性
7. **资源同步**：支持跨集群资源状态同步
8. **策略管理**：支持统一的资源策略和访问控制

这个框架为构建现代化的多集群Kubernetes管理平台提供了完整的解决方案，可以满足企业级多云和混合云环境下的Kubernetes资源管理需求。