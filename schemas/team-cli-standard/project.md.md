# 项目级约定（Project.md）

## 项目基本信息



* 项目名称：定时 + 消息消费纯业务型服务

* 技术栈：Golang + GORM (MySQL) + Redis + 定时任务框架 + MQ 消息队列

* 架构说明：无 Kitex RPC 服务，仅包含 biz 核心业务层 + crontab 定时任务模块 + consumer 消息消费模块

* 规范版本：v1.0.0

## 文档说明

本规范基于团队通用 OpenSpec 微服务基准规则扩展适配，**项目无 Kitex RPC 服务、无 Handler/Service 分层**，仅保留**biz 核心业务层**（mysql/redis 操作 + 公共逻辑），搭配**定时任务 crontab + MQ 消息消费 consumer**双启动子模块；

biz/dal 目录规范**完全对齐 Kitex RPC 标准目录规则**，全局规则优先级＞模块扩展规则。

## 全局继承规则

### 语言规范



1. 全部注释、业务文档、需求描述纯中文；仅技术关键字`ctx、db、req、resp、gorm、redis`保留英文

2. 禁止拼音命名、中英文混写、无意义英文缩写。

### 分层依赖规则

#### 合法依赖白名单



```
Logic → DAL、Convert

crontab任务实现 → biz.Logic / biz.DAL

consumer消费实现 → biz.Logic / biz.DAL
```

#### 禁止依赖红线



1. 无 Service/Handler 层，**Logic 直接作为顶层业务入口**

2. Convert 只做模型转换，禁止编写业务逻辑

3. DAL 层 Repo 禁止全局单例，按数据库名分目录

4. Where 查询构造器、GORM Repo、limit+1 分页、分页结构体规则**完全沿用 Kitex RPC 标准模板**

### Golang 编码规范

全项目统一编码约束：



* 结构体聚合初始化

* 变量`:=`自动推导

* 禁用变量遮蔽

* 无限循环使用`for{}`

* JSON 使用反引号字符串

* main 函数无全局状态

* 注释统一使用`//`中文单行

* Lint 告警添加`// NOLINT`标记

## 目录结构规范



```
项目根目录

├── biz/                     # 核心业务层（100%对齐Kitex RPC dal规范）

│   ├── logic/               # 公共业务逻辑（项目顶层业务实现）

│   │   └── xxx\_logic.go     # 业务逻辑实现（crontab/consumer共用）

│   ├── dal/                 # 数据操作层（规范=Kitex RPC标准）

│   │   ├── gormL/           # MySQL操作

│   │   │   ├── {db\_name}/   # 按库隔离

│   │   │   │   ├── repo/    # 数据表操作Repo

│   │   │   │   └── where/   # 查询条件构造器

│   │   │   └── init.go      # MySQL初始化

│   │   └── redis/           # Redis操作

│   │       ├── {db\_name}/   # 按库隔离

│   │       └── init.go      # Redis初始化

│   └── convert/             # 模型转换（结构体/DO/PO互转）

├── crontab/                 # 定时任务独立启动模块

│   ├── main.go              # 定时任务启动入口

│   └── task/                 # 定时任务实现（调用biz.Logic）

├── consumer/                # 消息消费独立启动模块

│   ├── main.go              # 消费服务启动入口

│   └── mq/                   # 消费逻辑实现（调用biz.Logic）

├── config/                  # 全局配置文件

├── go.mod

└── go.sum
```

## 核心模块专属规范

### biz 核心层规范（强制）



1. **dal 层**：目录结构、文件命名、代码写法**完全复刻 Kitex RPC 标准**，不允许自定义修改

2. **logic 层**：所有可复用业务逻辑统一收敛在此，crontab/consumer 禁止重复实现

3. **convert 层**：仅做数据结构转换，无任何业务逻辑、无任何数据操作

### crontab 定时任务规范（强制）



1. 启动模块`crontab/`仅做**任务注册、启动、调度**，核心业务逻辑必须调用`biz.logic`

2. 任务实现统一放在`crontab/task/`，禁止在 main.go 编写业务代码

3. 必须携带`ctx`上下文，支持链路追踪、超时控制

### consumer 消息消费规范（强制）



1. 消费模块`consumer/`仅做**消息接收、重试、异常处理**，核心业务逻辑必须调用`biz.logic`

2. 消费逻辑统一放在`consumer/mq/`，按消息主题 / 队列拆分文件

3. 严格遵循消费幂等、失败重试、死信队列规则

## 禁用约束（绝对红线）



1. 禁止在`crontab/consumer`中直接编写 MySQL/Redis 操作，必须调用`biz.dal`

2. 禁止修改`biz.dal`目录结构，必须保持与 Kitex RPC 标准一致

3. 禁止在`convert`中写业务逻辑、数据查询、外部调用

4. 禁止全局单例 Repo、禁止硬编码配置、禁止跨层违规依赖

> （注：文档部分内容可能由 AI 生成）