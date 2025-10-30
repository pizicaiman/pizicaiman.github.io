# 基于Go实现容器应用项目

本项目旨在通过Go语言从零实现一个简单的容器，帮助大家理解容器的基本原理和实现方式。

## 项目目标

- 理解Linux命名空间（Namespace）和控制组（CGroup）原理
- 实现简单的容器隔离和资源限制
- 自行实现容器的`run`、`init`子命令

---

## 1. 环境准备

- 安装Go语言（推荐1.18及以上版本）
- Linux操作系统（建议Ubuntu 20.04+）
- `root`权限（部分Namespace和CGroup操作需要）

---

## 2. 项目结构

```
go-container/
├── main.go
├── run.go
├── container/
│   ├── namespace.go
│   └── cgroup.go
└── README.md
```

---

## 3. 基本实现思想

### 3.1 创建新进程与Namespace隔离

通过`clone`系统调用（Go中可用`syscall.SysProcAttr`）创建子进程时指定新的Namespace：  
- `UTS`（主机名隔离）
- `PID`（进程隔离）
- `Mount`（挂载点隔离）
- `Network`（网络隔离）

### 3.2 使用CGroup限制资源

控制进程可用的CPU、内存等资源，通过操作`/sys/fs/cgroup`文件系统实现。

---

## 4. 示例代码

### 4.1 启动容器主进程（main.go）

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    if len(os.Args) < 2 {
        fmt.Println("expected 'run' or 'init' subcommands")
        os.Exit(1)
    }
    switch os.Args[1] {
    case "run":
        Run()
    case "init":
        Init()
    default:
        fmt.Println("invalid command")
        os.Exit(1)
    }
}
```

### 4.2 容器Run逻辑（run.go）

```go
import (
    "os/exec"
    "syscall"
)

func Run() {
    cmd := exec.Command("/proc/self/exe", "init")
    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS |
                    syscall.CLONE_NEWPID |
                    syscall.CLONE_NEWNS |
                    syscall.CLONE_NEWNET,
    }
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr

    if err := cmd.Run(); err != nil {
        fmt.Println("Run error:", err)
    }
}
```

### 4.3 容器Init逻辑（run.go）

```go
func Init() {
    fmt.Printf("Container process (PID: %d)\n", os.Getpid())
    cmd := exec.Command("/bin/bash")
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
    cmd.Run()
}
```

---

## 5. CGroup资源限制基础（container/cgroup.go）

```go
import (
    "os"
    "strconv"
)

func LimitMemory(pid int, limit string) error {
    cgroupMem := "/sys/fs/cgroup/memory/mycontainer/"
    os.Mkdir(cgroupMem, 0755)
    os.WriteFile(cgroupMem+"memory.limit_in_bytes", []byte(limit), 0700)
    os.WriteFile(cgroupMem+"tasks", []byte(strconv.Itoa(pid)), 0700)
    return nil
}
```

---

## 6. 运行效果

```
sudo go run main.go run
```

会进入新命名空间，并限制资源。

---

## 7. 总结

通过Go语言，实现了容器进程隔离、资源限制的最小原型。进一步可以学习挂载文件系统、网络隔离等高级特性。

---

> **推荐参考：**
> - [Linux命名空间与CGroup官方文档](https://man7.org/linux/man-pages/man7/namespaces.7.html)
> - [runc源码](https://github.com/opencontainers/runc)
> - [《自己动手写Docker》](https://github.com/yeasy/docker_practice/tree/master/init/container)
