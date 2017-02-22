---
title: 美团分享总结：系统性能优化之道
date: 2017-02-21 23:11:47
categories:
- coding
tags:
- 方法论
- 美团
---
# 典型性能问题
1. 响应慢：你这个服务, 总是响应超时,尽快解决下!
2. 单机容量低：就这么点量, 还要加机器?
3. 并发能力弱：这个服务并发怎么上不去呢, 查下为啥

# 性能优化方法论
## 数据驱动
## 系统诊断
### 如何选择工具
![性能工具][1]

### 性能诊断层次
- 系统层：OS JVM CPU Memory Network Disk | top jstat iftop iostat dstat...
- 组件层：Jetty DB Driver JSON Lib... | JProfiler Mtrace
- 业务层: 业务逻辑 数据结构 算法 | 日志 Jstack Greys

### 例子：首页超时了
- 排查网关问题
  * 网络延迟数据
- 排查后端服务问题
  * 从接入层(API层)开始检查, 首页调用链各个环节的延迟, 负载指标
  * 假设其中一个环节(比如POI 服务负载高, 响应异常). 检查服务器系统指标, OCTO 性能指标, CAT 监控数据, 日志数据 

# 参考手册
## 单机容量上不去
### CPU
- 如何识别：load、cpu使用率、 CPU.Steal()
- 如何诊断
  * top -bH -p <pid> -n 1 | head -n10
  * stack
  * jstat
  * JProfiler (能够精确定位，可以定位到具体代码消耗多少时间)
### 内存
- 如何识别：mem指标、swap、jvm.gc.count …
- 如何诊断：
  * jstat
  * jmap
- 精确定位：MAT
### 网络
- 如何识别：net.if.*; TcpExt.ListenOverflows ;
- 如何诊断：
  * netstat
  * iftop

## 响应时间慢
### 下游依赖方
db、缓存、服务
### 同步调用
### 逻辑实现
- 循环调用
- 本地方法耗时过长 Greys可以分析耗时

## 并发上不去
- 资源瓶颈：线程池，连接池 （JProfiler检查线程）
- 资源竞争：cpu切换，锁 （线程池并发模型 -> 异步并发模型）

Java 快速诊断性能瓶颈, 首选 JProfiler



  [1]: http://7xpw11.com1.z0.glb.clouddn.com/performance-tool.png