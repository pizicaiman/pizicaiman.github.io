# APISIX 二次开发实战指南

APISIX 是一款云原生、高性能、可扩展的开源 API 网关。其极致的插件化机制和灵活的架构，极适合自定义与二次开发。本文聚焦二次开发的典型场景、主流程解读、插件开发实战与调试建议。

---

## 1. APISIX 核心架构简述

- **核心组件**：
  - control plane（管理面, etcd 配置同步）
  - data plane（数据面, nginx+lua 实现）
  - 强插件化架构：所有流量能力（认证、限流、路由、协议转换等）均为插件实现
- **开发语言**：Lua（OpenResty）为主，部分配置/控制支持 Go

---

## 2. 二次开发典型场景

- 自定义认证/限流/流量染色/协议转换/安全插件等
- 扩展数据平面钩子，比如请求打标、灰度发布、服务编排
- 集成自有微服务注册中心、配置更灵活的路由方式

---

## 3. 插件开发流程与最佳实践

### 3.1 插件骨架快速创建

1. 在 `apisix/plugins/` 新建 `your-plugin.lua`  
2. 参考内置插件如 `example-plugin.lua` 填充基础注册信息：

```lua
local plugin_name = "your-plugin"
local core = require("apisix.core")

local schema = {type = "object", properties = {}, required = {}}

local _M = {
    version = 0.1,
    priority = 1000,
    name = plugin_name,
    schema = schema,
}

function _M.check_schema(conf)
    return core.schema.check(schema, conf)
end

function _M.access(conf, ctx)
    -- 实现请求拦截、身份校验、流量处理等
    core.log.warn("Your plugin triggered!")
    -- 支持返回各类自定义响应: return 401, { message = "unauthorized" }
end

return _M
```

### 3.2 生命周期钩子说明

- `access`: 最常用，请求到达阶段处理
- `rewrite`, `header_filter`, `body_filter`, `log`: 可根据需要实现完整钩子

### 3.3 插件注册与热加载

- 注册插件需修改 `config.yaml` 里的 `plugins: [...]` 列表
- APISIX 支持热加载插件代码，无需重启 nginx，开发效率高

---

## 4. 实战开发建议

- **调试日志**：利用 `core.log.info|warn|error` 打印调试信息
- **单元与集成测试**：建议参考内置 `t/plugin/*.t` 进行自动化 Lua 测试
- **高可维护性**：复用 `core`/`apisix` 工具模块，少造轮子
- **安全评审**：编码时严防外部输入注入/信息泄露

---

## 5. 参考资料与进阶

- [APISIX Plugin 开发官方文档](https://apisix.apache.org/zh/docs/apisix/plugin-develop/)
- [核心实现源码 (GitHub)](https://github.com/apache/apisix/tree/master/apisix/plugins)
- [基于 OpenResty 的 OpenAPI 实践](https://zhuanlan.zhihu.com/p/349162460)

---

> 总结：APISIX 插件机制高度解耦，极易按需扩展。掌握典型插件开发、源码调试与热插拔流程，可助你定制企业级流量网关能力。如果有复杂认证、动态流控、安全合规等场景，二次开发插件是首选方案。

如需具体插件模板、定制流程或集群化部署实战，欢迎留言交流！

