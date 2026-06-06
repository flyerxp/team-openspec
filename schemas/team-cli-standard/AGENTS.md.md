# Agent 开发指南（AGENTS.md）

## 项目概述

这是一个无 RPC 的纯业务型 Golang 服务，包含核心业务逻辑层、定时任务模块和消息消费模块，biz/dal 层完全对齐团队 Kitex RPC 标准规范。

## 开发环境



* 语言：Golang 1.20+

* 依赖管理：Go Modules

* 数据库：MySQL 8.0+、Redis 6.0+

* 中间件：MQ 消息队列、定时任务框架

## 常用命令



```
\# 依赖下载

go mod download

\# 代码编译

go build -o bin/crontab ./crontab

go build -o bin/consumer ./consumer

\# 代码检查

golangci-lint run
```

## 代码库结构



```
项目根目录

├── biz/                     # 核心业务层（禁止修改dal目录结构）

│   ├── logic/               # 公共业务逻辑，所有业务实现收敛于此

│   ├── dal/                 # 数据操作层，完全对齐Kitex标准

│   └── convert/             # 模型转换，仅做结构转换

├── crontab/                 # 定时任务模块，仅做调度，业务调用biz

│   ├── main.go              # 启动入口

│   └── task/                # 任务实现

├── consumer/                # 消费模块，仅做消息处理，业务调用biz

│   ├── main.go              # 启动入口

│   └── mq/                  # 消费逻辑

├── config/                  # 配置文件

├── AGENTS.md                # AI代理开发指南

└── openspec/                # OpenSpec规范目录

&#x20;   ├── config.yaml          # OpenSpec全局配置

&#x20;   ├── project.md           # 项目级约定

&#x20;   └── specs/

&#x20;       └── core/

&#x20;           └── spec.md      # 核心业务模块规范
```

## 编码规范

### 语言规范



* 所有注释、文档必须使用中文，仅技术关键字保留英文

* 禁止拼音命名、中英文混写，使用有意义的英文命名

* 禁止无意义的缩写，命名要清晰易懂

### 分层依赖规则



* **Logic 层**：可以依赖 DAL 层（所有数据操作）和 Convert 层，禁止反向依赖

* **crontab 模块**：只能调用 biz.Logic 或 biz.DAL，禁止自己实现业务逻辑或数据操作

* **consumer 模块**：只能调用 biz.Logic 或 biz.DAL，禁止自己实现业务逻辑或数据操作

* **Convert 层**：只能做结构体转换，禁止任何业务逻辑或数据操作

### Golang 编码规则



* 结构体必须使用聚合初始化，禁止使用指针初始化后赋值

* 变量优先使用`:=`自动推导类型，禁止显式声明不必要的类型

* 禁止变量遮蔽，避免同名变量覆盖外层变量

* 无限循环必须使用`for{}`，禁止使用`for true {}`

* JSON 标签必须使用反引号字符串，禁止使用双引号

* main 函数禁止定义全局状态，所有初始化必须在函数内完成

* 所有注释必须使用`//`开头的单行注释，禁止使用块注释

* 对于无法避免的 Lint 告警，必须添加`// NOLINT`标记并说明原因

## 禁止操作



1. 禁止在 crontab 或 consumer 模块中直接编写 MySQL/Redis 操作，必须通过 biz.dal 层

2. 禁止修改 biz.dal 的目录结构，必须保持与 Kitex RPC 标准完全一致

3. 禁止在 convert 层编写业务逻辑、数据查询或外部调用

4. 禁止创建全局单例的 Repo 对象，必须按请求初始化

5. 禁止硬编码配置，所有配置必须通过 config 模块加载

6. 禁止跨层违规依赖，比如 Logic 层不能反向依赖上层模块

## 开发流程



1. 新增业务逻辑时，必须先在 biz.logic 层实现，确保 crontab 和 consumer 可以复用

2. 新增定时任务时，在 crontab/task/ 下创建任务文件，调用 biz 的逻辑

3. 新增消费逻辑时，在 consumer/mq/ 下创建消费文件，调用 biz 的逻辑

4. 新增数据操作时，在 biz.dal 层实现，遵循 Kitex 的 dal 规范

5. 所有代码必须符合 OpenSpec 规范，提交前会自动校验

> （注：文档部分内容可能由 AI 生成）