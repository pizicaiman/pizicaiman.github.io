
# CPU故障诊断分析指南

## 1. 概述

本指南旨在帮助系统管理员和运维人员快速定位和解决CPU相关的性能问题和故障。

## 2. 常见CPU故障类型

### 2.1 CPU使用率过高
- 应用程序占用过多CPU资源
- 死循环或无限循环
- 频繁的上下文切换
- 系统进程异常

### 2.2 CPU负载不均衡
- CPU亲和性配置不当
- NUMA架构问题
- 中断分布不均

### 2.3 CPU性能下降
- CPU降频（thermal throttling）
- 硬件故障
- 微码问题

## 3. 诊断工具

### 3.1 基础监控工具

#### top/htop
# 实时查看CPU使用情况
top

# 按CPU使用率排序
top -o %CPU

# htop提供更友好的界面
htop

#### mpstat
# 查看所有CPU核心的使用情况
mpstat -P ALL 1

# 输出示例解读
# %usr: 用户空间CPU使用率
# %sys: 内核空间CPU使用率
# %iowait: IO等待时间
# %idle: 空闲时间

#### vmstat
# 查看系统整体性能
vmstat 1

# 关注的指标
# r: 运行队列长度
# b: 不可中断的睡眠进程数
# us: 用户CPU时间
# sy: 系统CPU时间
# wa: IO等待时间

### 3.2 高级分析工具

#### perf
# 采样CPU性能数据
perf top

# 记录性能数据
perf record -a -g sleep 10

# 分析记录的数据
perf report

# CPU火焰图生成
perf record -F 99 -a -g -- sleep 60
perf script > out.perf

#### pidstat
# 监控进程CPU使用情况
pidstat 1

# 监控特定进程
pidstat -p <PID> 1

# 显示线程级别的CPU使用
pidstat -t -p <PID> 1

#### sar
# 查看历史CPU使用数据
sar -u 1 10

# 查看所有CPU核心
sar -P ALL 1 10

# 查看历史数据
sar -u -f /var/log/sa/sa<day>

## 4. 诊断流程

### 4.1 快速诊断步骤

**步骤1：确认问题**
# 查看CPU整体负载
uptime

# 查看实时CPU使用
top

**步骤2：定位高CPU进程**
# 找出CPU使用最高的进程
ps aux --sort=-%cpu | head -10

# 或使用top并按P键排序
top

**步骤3：分析进程详情**
# 查看进程的线程
top -H -p <PID>

# 查看进程状态
cat /proc/<PID>/status

# 查看进程打开的文件
lsof -p <PID>

**步骤4：分析调用栈**
# 查看进程调用栈
pstack <PID>

# 或使用gdb
gdb -p <PID>
(gdb) thread apply all bt

### 4.2 深度分析

#### 4.2.1 CPU使用率高分析

# 1. 使用perf分析热点函数
perf top -p <PID>

# 2. 生成火焰图
perf record -F 99 -p <PID> -g -- sleep 60
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg

# 3. 查看系统调用
strace -c -p <PID>

# 4. 分析上下文切换
pidstat -w -p <PID> 1

#### 4.2.2 系统CPU高分析

# 查看中断情况
cat /proc/interrupts

# 查看软中断
cat /proc/softirqs

# 监控中断
watch -n 1 cat /proc/interrupts

# 分析内核函数调用
perf top -g

#### 4.2.3 iowait高分析

# 查看IO统计
iostat -x 1

# 查看哪个进程在等待IO
iotop

# 查看进程IO详情
pidstat -d 1

## 5. 常见问题及解决方案

### 5.1 应用程序CPU使用率过高

**诊断：**
# 定位高CPU线程
top -H -p <PID>

# 将线程ID转换为16进制
printf "%x\n" <TID>

# 查看对应线程的堆栈
jstack <PID> | grep <hex_TID> -A 30

**解决方案：**
- 优化代码逻辑，消除死循环
- 增加缓存减少计算
- 使用异步处理
- 增加CPU资源

### 5.2 上下文切换过多

**诊断：**
# 查看上下文切换情况
vmstat 1

# 查看进程级别的上下文切换
pidstat -w 1

# cswch/s: 自愿上下文切换（等待资源）
# nvcswch/s: 非自愿上下文切换（时间片到期）

**解决方案：**
- 减少线程数量
- 优化锁的使用
- 调整CPU亲和性
- 增大时间片

### 5.3 CPU降频问题

**诊断：**
# 查看CPU频率
watch -n 1 "cat /proc/cpuinfo | grep MHz"

# 查看CPU温度
sensors

# 查看调频策略
cpupower frequency-info

# 查看温度限制事件
dmesg | grep -i thermal

**解决方案：**
# 设置性能模式
cpupower frequency-set -g performance

# 检查散热系统
# 清理灰尘、更换硅脂

# 调整BIOS设置
# 关闭节能模式

### 5.4 NUMA问题

**诊断：**
# 查看NUMA配置
numactl --hardware

# 查看进程的NUMA信息
numastat -p <PID>

# 查看NUMA统计
cat /proc/<PID>/numa_maps

**解决方案：**
# 绑定进程到特定NUMA节点
numactl --cpunodebind=0 --membind=0 <command>

# 设置进程的NUMA策略
numactl --interleave=all <command>

## 6. 性能优化建议

### 6.1 应用层优化

1. **代码优化**
   - 减少不必要的计算
   - 使用更高效的算法
   - 避免频繁的内存分配

2. **并发优化**
   - 使用线程池
   - 减少锁竞争
   - 使用无锁数据结构

3. **缓存优化**
   - 提高缓存命中率
   - 减少远程数据访问
   - 注意CPU缓存行对齐

### 6.2 系统层优化

1. **CPU亲和性设置**
# 绑定进程到特定CPU
taskset -cp 0,1,2,3 <PID>

# 启动进程时绑定
taskset -c 0-3 <command>

2. **中断亲和性优化**
# 查看中断分布
cat /proc/interrupts

# 设置网卡中断亲和性
echo 4 > /proc/irq/<IRQ>/smp_affinity

3. **调度策略优化**
# 设置实时调度
chrt -f 99 <PID>

# 设置nice值
renice -n -10 -p <PID>

## 7. 监控和告警

### 7.1 设置监控指标

# 编写监控脚本
#!/bin/bash
CPU_THRESHOLD=80

while true; do
    CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
        echo "Warning: CPU usage is ${CPU_USAGE}%"
        # 发送告警
    fi
    sleep 60
done

### 7.2 日志记录

# 定期记录CPU状态
*/5 * * * * sar -u 1 1 >> /var/log/cpu_monitor.log

# 记录top输出
*/5 * * * * top -bn1 >> /var/log/top_monitor.log

## 8. 故障案例分析

### 案例1：Java应用CPU 100%

**现象：** Java应用CPU持续100%

**诊断过程：**
# 1. 找到进程ID
jps

# 2. 找到高CPU线程
top -H -p <PID>

# 3. 转换线程ID
printf "%x\n" <TID>

# 4. 查看线程堆栈
jstack <PID> | grep <hex_TID> -A 30

**解决：** 发现死循环，修复代码逻辑

### 案例2：系统CPU sy过高

**现象：** 系统CPU sy占用80%+

**诊断过程：**
# 1. 使用perf分析内核函数
perf top

# 2. 发现大量的自旋锁竞争
perf record -a -g -- sleep 10
perf report

# 3. 分析锁竞争
perf lock record -a -- sleep 10
perf lock report

**解决：** 优化内核模块，减少锁竞争

## 9. 预防措施

1. **建立基线**
   - 记录正常情况下的CPU使用模式
   - 建立性能基准测试

2. **容量规划**
   - 预留足够的CPU资源
   - 考虑峰值负载

3. **定期维护**
   - 更新系统补丁
   - 清理不必要的服务
   - 检查硬件状态

4. **自动化监控**
   - 部署监控系统（Prometheus, Grafana）
   - 设置合理的告警阈值
   - 建立应急响应流程

## 10. 参考资源

- Linux Performance Tools: http://www.brendangregg.com/linuxperf.html
- perf Examples: http://www.brendangregg.com/perf.html
- CPU火焰图: http://www.brendangregg.com/flamegraphs.html
- Linux性能优化实战
- 系统性能调优指南

## 11. 总结

CPU故障诊断需要系统化的方法和合适的工具。关键步骤包括：

1. 快速定位问题（top, uptime）
2. 识别高CPU进程/线程
3. 深入分析原因（perf, strace, pstack）
4. 实施解决方案
5. 验证效果
6. 建立预防机制

记住：性能问题的解决不是一蹴而就的，需要持续的监控和优化。
