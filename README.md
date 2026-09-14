# AI 题境 —— AI 智能刷题平台

基于 Spring Boot 的 AI 刷题平台，支持 **AI 生成题目与解答**、题库管理、分词检索、在线刷题与刷题记录查看，并集成多层性能优化与安全防护方案。

## ✨ 核心亮点

### 1. AI 生成题目链路优化
- 接入火山方舟（DeepSeek V3）实现 AI 自动生成题目与解答
- 基于 MyBatis Plus batch 分批插入 + 线程池隔离 + `CompletableFuture` 异步编排，AI 生成 10 道题目并入库的耗时降低约 1 倍

### 2. 高性能设计
- **Redis BitMap** 实现用户年度刷题记录，相比数据库行式存储节省数百倍空间
- 本地缓存 + 位运算优化，减少网络请求与接口传输体积
- **Elasticsearch** 替代 MySQL 模糊查询，IK 分词器 + 自定义词典实现灵活检索，并实现 ES 故障降级策略（宕机时回落数据库/缓存）
- 京东 **HotKey** 热点探测，自动发现并本地缓存热点题目

### 3. 高可用与流控
- **Sentinel** 热点参数限流（单 IP 题目获取流控）+ 熔断降级（题库列表接口熔断时返回本地缓存），规则通过 Push 模式持久化
- 增量/全量定时任务同步 MySQL 与 Elasticsearch 数据

### 4. 安全防护
- **Sa-Token** 同端登录冲突检测：懒惰式策略通知多端登录，避免轮询压力
- 分级反爬虫：滑动窗口频率统计 + Redis + Lua 脚本保证原子性，超限自动告警与封禁
- **WebFilter + BloomFilter** IP 黑名单拦截，结合 Nacos 配置中心动态更新
- 自定义注解封装 HotKey 探测、分布式锁、反爬校验，消除冗余代码

## 🛠 技术栈

Spring Boot 2.7 / MyBatis Plus / MySQL / Redis + Redisson / Elasticsearch / Sentinel / Nacos / HotKey / Sa-Token / 火山方舟（DeepSeek V3）/ 腾讯云 COS / Knife4j

## 🚀 快速启动

```bash
# 1. 准备 MySQL / Redis / Elasticsearch / Nacos 环境
# 2. 在 application-local.yml 中配置 AI apikey 等本地凭证（不入库）
# 3. 启动
mvn spring-boot:run
```

## 📂 模块结构

```
src/main/java/com/xiaochen
├── aop/           # 权限 / 反爬 / 分布式锁 / 热点探测拦截器
├── blackfilter/   # IP 黑名单过滤器（Nacos 动态规则）
├── controller/    # 用户 / 题目 / 题库 / 点赞收藏 / 微信公众号
├── esdao/         # ES 数据访问层
├── job/cycle/     # 增量数据同步任务
├── manager/       # AI 调用管理器（AiManager）
├── sentinel/      # 限流熔断规则管理
└── ThreadPool/    # 线程池隔离（AI / 数据库）
```
