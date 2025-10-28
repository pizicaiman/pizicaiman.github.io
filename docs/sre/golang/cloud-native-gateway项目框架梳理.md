	<-stopCh
	c.logger.Info("Stopping Service controller")
	
	return nil
}

func (c *ServiceController) enqueueService(obj interface{}) {
	var key string
	var err error
	if key, err = cache.MetaNamespaceKeyFunc(obj); err != nil {
		runtime.HandleError(err)
		return
	}
	c.workqueue.Add(key)
}

func (c *ServiceController) runWorker() {
	for c.processNextWorkItem() {
	}
}

func (c *ServiceController) processNextWorkItem() bool {
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
			runtime.HandleError(nil)
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
		runtime.HandleError(err)
		return true
	}
	
	return true
}

func (c *ServiceController) syncHandler(key string) error {
	namespace, name, err := cache.SplitMetaNamespaceKey(key)
	if err != nil {
		runtime.HandleError(err)
		return nil
	}
	
	service, err := c.clientset.CoreV1().Services(namespace).Get(context.TODO(), name, metav1.GetOptions{})
	if err != nil {
		c.logger.Error("Failed to get Service", zap.String("key", key), zap.Error(err))
		return err
	}
	
	c.eventHandler.OnAdd(service)
	return nil
}
```

### 4. 网关核心功能 (internal/pkg/gateway/)

#### 路由器 (router.go)

```go
package gateway

import (
	"net/http"
	"net/http/httputil"
	"net/url"

	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
	networkingv1 "k8s.io/api/networking/v1"
)

type Router struct {
	engine     *gin.Engine
	routes     map[string]*Route
	logger     *zap.Logger
	proxy      *Proxy
	middleware []gin.HandlerFunc
}

type Route struct {
	Path        string
	Backend     string
	Middlewares []string
	Ingress     *networkingv1.Ingress
}

func NewRouter(logger *zap.Logger) *Router {
	r := &Router{
		engine: gin.New(),
		routes: make(map[string]*Route),
		logger: logger,
		proxy:  NewProxy(logger),
	}
	
	r.setupMiddleware()
	r.setupRoutes()
	
	return r
}

func (r *Router) setupMiddleware() {
	r.engine.Use(gin.Logger())
	r.engine.Use(gin.Recovery())
}

func (r *Router) setupRoutes() {
	// 默认路由
	r.engine.Any("/*path", r.proxyHandler)
}

func (r *Router) proxyHandler(c *gin.Context) {
	path := c.Param("path")
	
	// 查找匹配的路由
	route, ok := r.findRoute(path)
	if !ok {
		c.JSON(http.StatusNotFound, gin.H{"error": "route not found"})
		return
	}
	
	// 解析后端URL
	backendURL, err := url.Parse(route.Backend)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "invalid backend URL"})
		return
	}
	
	// 创建反向代理
	proxy := httputil.NewSingleHostReverseProxy(backendURL)
	
	// 修改请求
	proxy.Director = func(req *http.Request) {
		req.URL.Scheme = backendURL.Scheme
		req.URL.Host = backendURL.Host
		req.Host = backendURL.Host
	}
	
	// 执行代理
	proxy.ServeHTTP(c.Writer, c.Request)
}

func (r *Router) findRoute(path string) (*Route, bool) {
	// 简单的路由匹配逻辑
	for _, route := range r.routes {
		if path == route.Path {
			return route, true
		}
	}
	return nil, false
}

func (r *Router) AddRoute(route *Route) {
	r.routes[route.Path] = route
}

func (r *Router) RemoveRoute(path string) {
	delete(r.routes, path)
}

func (r *Router) Engine() *gin.Engine {
	return r.engine
}
```

#### 反向代理 (proxy.go)

```go
package gateway

import (
	"net/http"
	"net/http/httputil"
	"time"

	"go.uber.org/zap"
)

type Proxy struct {
	logger *zap.Logger
	client *http.Client
}

func NewProxy(logger *zap.Logger) *Proxy {
	return &Proxy{
		logger: logger,
		client: &http.Client{
			Timeout: 30 * time.Second,
		},
	}
}

func (p *Proxy) ReverseProxy(target string) *httputil.ReverseProxy {
	targetURL, err := url.Parse(target)
	if err != nil {
		p.logger.Error("Failed to parse target URL", zap.String("target", target), zap.Error(err))
		return nil
	}
	
	return httputil.NewSingleHostReverseProxy(targetURL)
}
```

### 5. 服务发现 (internal/pkg/discovery/)

#### Kubernetes服务发现 (kubernetes.go)

```go
package discovery

import (
	"context"

	"go.uber.org/zap"
	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
)

type KubernetesDiscovery struct {
	clientset kubernetes.Interface
	logger    *zap.Logger
	namespace string
}

func NewKubernetesDiscovery(clientset kubernetes.Interface, namespace string, logger *zap.Logger) *KubernetesDiscovery {
	return &KubernetesDiscovery{
		clientset: clientset,
		logger:    logger,
		namespace: namespace,
	}
}

func (k *KubernetesDiscovery) GetServiceEndpoints(serviceName string) ([]string, error) {
	svc, err := k.clientset.CoreV1().Services(k.namespace).Get(context.TODO(), serviceName, metav1.GetOptions{})
	if err != nil {
		return nil, err
	}
	
	// 获取Endpoints
	endpoints, err := k.clientset.CoreV1().Endpoints(k.namespace).Get(context.TODO(), serviceName, metav1.GetOptions{})
	if err != nil {
		return nil, err
	}
	
	var addresses []string
	for _, subset := range endpoints.Subsets {
		for _, addr := range subset.Addresses {
			for _, port := range subset.Ports {
				address := addr.IP
				if port.Port != 0 {
					address = addr.IP + ":" + string(rune(port.Port))
				}
				addresses = append(addresses, address)
			}
		}
	}
	
	return addresses, nil
}

func (k *KubernetesDiscovery) ListServices() ([]*corev1.Service, error) {
	services, err := k.clientset.CoreV1().Services(k.namespace).List(context.TODO(), metav1.ListOptions{})
	if err != nil {
		return nil, err
	}
	
	return services.Items, nil
}
```

### 6. 插件系统 (internal/pkg/plugin/)

#### 插件管理器 (manager.go)

```go
package plugin

import (
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
)

type PluginManager struct {
	plugins map[string]Plugin
	logger  *zap.Logger
}

func NewPluginManager(logger *zap.Logger) *PluginManager {
	return &PluginManager{
		plugins: make(map[string]Plugin),
		logger:  logger,
	}
}

func (pm *PluginManager) Register(name string, plugin Plugin) {
	pm.plugins[name] = plugin
	pm.logger.Info("Plugin registered", zap.String("name", name))
}

func (pm *PluginManager) Get(name string) (Plugin, bool) {
	plugin, ok := pm.plugins[name]
	return plugin, ok
}

func (pm *PluginManager) List() []string {
	var names []string
	for name := range pm.plugins {
		names = append(names, name)
	}
	return names
}

func (pm *PluginManager) Apply(name string, c *gin.Context) error {
	plugin, ok := pm.plugins[name]
	if !ok {
		return nil // 插件不存在，不执行
	}
	
	return plugin.Execute(c)
}
```

#### 插件接口 (types.go)

```go
package plugin

import "github.com/gin-gonic/gin"

type Plugin interface {
	Name() string
	Execute(c *gin.Context) error
}

type AuthPlugin struct {
	name string
}

func (ap *AuthPlugin) Name() string {
	return ap.name
}

func (ap *AuthPlugin) Execute(c *gin.Context) error {
	// 实现认证逻辑
	token := c.GetHeader("Authorization")
	if token == "" {
		c.AbortWithStatusJSON(401, gin.H{"error": "Authorization required"})
		return nil
	}
	
	// 验证token
	// ...
	
	return nil
}

type RateLimitPlugin struct {
	name string
}

func (rlp *RateLimitPlugin) Name() string {
	return rlp.name
}

func (rlp *RateLimitPlugin) Execute(c *gin.Context) error {
	// 实现限流逻辑
	// ...
	
	return nil
}
```

### 7. 监控指标 (internal/pkg/metrics/)

#### 指标收集器 (collector.go)

```go
package metrics

import (
	"github.com/prometheus/client_golang/prometheus"
)

type Collector struct {
	requestTotal    *prometheus.CounterVec
	requestDuration *prometheus.HistogramVec
	requestSize     *prometheus.HistogramVec
	responseSize    *prometheus.HistogramVec
}

func NewCollector() *Collector {
	c := &Collector{
		requestTotal: prometheus.NewCounterVec(
			prometheus.CounterOpts{
				Name: "http_requests_total",
				Help: "Total number of HTTP requests made.",
			},
			[]string{"method", "endpoint", "status"},
		),
		requestDuration: prometheus.NewHistogramVec(
			prometheus.HistogramOpts{
				Name:    "http_request_duration_seconds",
				Help:    "HTTP request latencies in seconds.",
				Buckets: prometheus.DefBuckets,
			},
			[]string{"method", "endpoint"},
		),
		requestSize: prometheus.NewHistogramVec(
			prometheus.HistogramOpts{
				Name:    "http_request_size_bytes",
				Help:    "HTTP request sizes in bytes.",
				Buckets: prometheus.ExponentialBuckets(100, 10, 8),
			},
			[]string{"method", "endpoint"},
		),
		responseSize: prometheus.NewHistogramVec(
			prometheus.HistogramOpts{
				Name:    "http_response_size_bytes",
				Help:    "HTTP response sizes in bytes.",
				Buckets: prometheus.ExponentialBuckets(100, 10, 8),
			},
			[]string{"method", "endpoint"},
		),
	}
	
	prometheus.MustRegister(c.requestTotal)
	prometheus.MustRegister(c.requestDuration)
	prometheus.MustRegister(c.requestSize)
	prometheus.MustRegister(c.responseSize)
	
	return c
}

func (c *Collector) Describe(ch chan<- *prometheus.Desc) {
	c.requestTotal.Describe(ch)
	c.requestDuration.Describe(ch)
	c.requestSize.Describe(ch)
	c.responseSize.Describe(ch)
}

func (c *Collector) Collect(ch chan<- prometheus.Metric) {
	c.requestTotal.Collect(ch)
	c.requestDuration.Collect(ch)
	c.requestSize.Collect(ch)
	c.responseSize.Collect(ch)
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

kubernetes:
  kubeconfig: ""
  master_url: ""
  namespace: "default"

nginx:
  template_path: "/etc/nginx/template/nginx.conf.tmpl"
  config_path: "/etc/nginx/nginx.conf"
  binary_path: "/usr/sbin/nginx"
  reload_script: "/etc/nginx/scripts/reload.sh"

metrics:
  enabled: true
  addr: ":9090"

log:
  level: "info"
  format: "json"
  file_path: "/var/log/gateway.log"
  max_size: 100
  max_backups: 3
  max_age: 7
```

### 2. Nginx模板文件 (configs/nginx-template.conf)

```nginx
events {
    worker_connections 1024;
}

http {
    upstream backend {
        {{range .Backends}}
        server {{.}};
        {{end}}
    }

    server {
        listen 80;
        
        {{range .Routes}}
        location {{.Path}} {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        {{end}}
    }
}
```

## 部署配置

### 1. Kubernetes部署文件 (deploy/kubernetes/deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloud-native-gateway
  labels:
    app: cloud-native-gateway
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cloud-native-gateway
  template:
    metadata:
      labels:
        app: cloud-native-gateway
    spec:
      serviceAccountName: gateway-service-account
      containers:
      - name: gateway
        image: cloud-native-gateway:latest
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 9090
          name: metrics
        volumeMounts:
        - name: config
          mountPath: /etc/gateway
        env:
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      volumes:
      - name: config
        configMap:
          name: gateway-config
```

### 2. RBAC配置 (deploy/kubernetes/rbac.yaml)

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gateway-service-account

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: gateway-cluster-role
rules:
- apiGroups: [""]
  resources: ["services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: gateway-cluster-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: gateway-cluster-role
subjects:
- kind: ServiceAccount
  name: gateway-service-account
  namespace: default
```

## 核心功能实现

### 1. 动态路由更新

通过监听Kubernetes Ingress资源的变化，动态更新网关路由配置：

```go
func (ic *IngressController) syncHandler(key string) error {
	// 获取Ingress资源
	ingress, err := ic.getIngress(key)
	if err != nil {
		return err
	}
	
	// 解析路由规则
	routes := parseIngressRules(ingress)
	
	// 更新网关路由
	for _, route := range routes {
		ic.gatewayRouter.AddRoute(route)
	}
	
	return nil
}
```

### 2. 服务发现集成

与Kubernetes服务发现机制集成，自动发现后端服务实例：

```go
func (kd *KubernetesDiscovery) WatchServices() {
	// 监听服务变化
	watcher, err := kd.clientset.CoreV1().Services(kd.namespace).Watch(context.TODO(), metav1.ListOptions{})
	if err != nil {
		kd.logger.Error("Failed to create service watcher", zap.Error(err))
		return
	}
	
	for event := range watcher.ResultChan() {
		switch event.Type {
		case watch.Added, watch.Modified:
			service := event.Object.(*corev1.Service)
			kd.updateServiceEndpoints(service.Name)
		case watch.Deleted:
			service := event.Object.(*corev1.Service)
			kd.removeServiceEndpoints(service.Name)
		}
	}
}
```

### 3. 负载均衡

支持多种负载均衡算法：

```go
type LoadBalancer interface {
	Select(endpoints []string) string
}

type RoundRobinLoadBalancer struct {
	index int
}

func (rr *RoundRobinLoadBalancer) Select(endpoints []string) string {
	if len(endpoints) == 0 {
		return ""
	}
	
	selected := endpoints[rr.index%len(endpoints)]
	rr.index++
	
	return selected
}

type RandomLoadBalancer struct{}

func (r *RandomLoadBalancer) Select(endpoints []string) string {
	if len(endpoints) == 0 {
		return ""
	}
	
	return endpoints[rand.Intn(len(endpoints))]
}
```

## 监控与运维

### 1. 指标暴露

通过Prometheus暴露关键指标：

```go
func (c *Collector) Middleware() gin.HandlerFunc {
	return func(ctx *gin.Context) {
		start := time.Now()
		
		ctx.Next()
		
		duration := time.Since(start)
		status := strconv.Itoa(ctx.Writer.Status())
		method := ctx.Request.Method
		endpoint := ctx.Request.URL.Path
		
		c.requestTotal.WithLabelValues(method, endpoint, status).Inc()
		c.requestDuration.WithLabelValues(method, endpoint).Observe(duration.Seconds())
		
		if ctx.Request.ContentLength > 0 {
			c.requestSize.WithLabelValues(method, endpoint).Observe(float64(ctx.Request.ContentLength))
		}
		
		if ctx.Writer.Size() > 0 {
			c.responseSize.WithLabelValues(method, endpoint).Observe(float64(ctx.Writer.Size()))
		}
	}
}
```

### 2. 健康检查

提供健康检查端点：

```go
func (app *GatewayApp) setupHealthChecks() {
	app.router.GET("/healthz", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"status": "healthy",
		})
	})
	
	app.router.GET("/ready", func(c *gin.Context) {
		// 检查依赖服务是否就绪
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
	// ...
	return nil
}
```

### 2. 自定义中间件

支持自定义中间件：

```go
func (r *Router) AddMiddleware(name string, middleware gin.HandlerFunc) {
	r.middlewares[name] = middleware
}
```

## 总结

基于Ingress-Nginx和Higress-Nginx的云原生网关项目具有以下优势：

1. **云原生集成**：与Kubernetes生态系统无缝集成
2. **动态配置**：支持运行时动态更新路由和服务发现
3. **高性能**：基于Go语言和Gin框架，性能优异
4. **可扩展性**：插件化架构，易于扩展功能
5. **可观测性**：集成Prometheus监控和日志系统
6. **安全控制**：支持认证、授权和限流等安全功能
7. **负载均衡**：支持多种负载均衡算法
8. **易于运维**：提供健康检查和配置管理功能

这个框架为构建现代化的云原生API网关提供了完整的解决方案，可以满足企业级微服务架构下的各种需求。
