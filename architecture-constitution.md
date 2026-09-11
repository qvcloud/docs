# 前后端技术架构宪法 (General Technical Architecture Constitution)

> 本文档为通用软件工程前后端技术架构标准规范，旨在确立可维护、可替换、高内聚低耦合、安全且具备高度可观测性的系统工程准则。适用于现代微服务/模块化单体、前后端解耦的通用软件系统。

---

## 目录

- [第一章 核心治理原则](#第一章-核心治理原则-core-principles)
  - [原则 I. 清晰边界与可替换依赖](#原则-i-清晰边界与可替换依赖-explicit-boundaries--replaceable-dependencies)
  - [原则 II. 分层架构与强类型契约](#原则-ii-分层架构与强类型契约-layered-architecture--typed-contracts)
  - [原则 III. 可观测的防御性错误处理](#原则-iii-可观测的防御性错误处理-observable-failure-handling)
  - [原则 IV. 环境配置隔离与凭据数据安全](#原则-iv-环境配置隔离与凭据数据安全-configuration--credential-security)
  - [原则 V. 全链路自动化质量门禁](#原则-v-全链路自动化质量门禁-automated-quality-gates)
- [第二章 前端技术架构规范](#第二章-前端技术架构规范-frontend-architecture)
  - [2.1 物理目录与职责边界](#21-物理目录与职责边界)
  - [2.2 分层架构与状态管理](#22-分层架构与状态管理)
  - [2.3 渲染策略与多语言/SEO 架构](#23-渲染策略与多语言seo-架构)
- [第三章 Go 后端技术架构规范](#第三章-go-后端技术架构规范-go-backend-architecture)
  - [3.1 标准目录结构布局 (Directory Layout)](#31-标准目录结构布局-directory-layout)
  - [3.2 核心类库与技术栈选型标准](#32-核心类库与技术栈选型标准)
  - [3.3 架构分层与依赖注入 (DI & Fx)](#33-架构分层与依赖注入-di--fx)
  - [3.4 统一响应与链路追踪规范](#34-统一响应与链路追踪规范)
  - [3.5 异步任务与长耗时计算解耦](#35-异步任务与长耗时计算解耦)
  - [3.6 基础质量门禁与 Makefile 标准 (Quality Gates)](#36-基础质量门禁与-makefile-标准-quality-gates)
- [第四章 接口、数据与兼容性规范](#第四章-接口数据与兼容性规范-contract--compatibility)
  - [4.1 跨端契约演进与向后兼容](#41-跨端契约演进与向后兼容)
  - [4.2 敏感数据保护与最小权限原则](#42-敏感数据保护与最小权限原则)
  - [4.3 可重现构建与单一真相来源](#43-可重现构建与单一真相来源)
- [第五章 开发、审查与发布流程](#第五章-开发审查与发布流程-development--delivery-lifecycle)
  - [5.1 规范驱动开发 (Spec-Driven)](#51-规范驱动开发-spec-driven)
  - [5.2 代码审查门禁标准](#52-代码审查门禁标准)
  - [5.3 基于 Git Tag 的唯一发布真相](#53-基于-git-tag-的唯一发布真相)
  - [5.4 保留历史的平滑回滚机制](#54-保留历史的平滑回滚机制)
- [第六章 宪法效力与修订治理](#第六章-宪法效力与修订治理-governance--amendments)

---

## 第一章 核心治理原则 (Core Principles)

### 原则 I. 清晰边界与可替换依赖 (Explicit Boundaries & Replaceable Dependencies)
1. **职责单一与适配层封装**：
   - 系统中每个模块或组件必须拥有清晰且不可重叠的职责边界。
   - 所有第三方基础服务（存储、消息队列、缓存、外部开放接口、云厂商专有服务）必须封装在适配层（Adapter/Client Wrapper）或抽象接口（Interface）之后。
2. **拒绝专有协议泄露**：
   - 核心业务逻辑严禁直接导入或依赖任何特定服务商的 SDK、专有数据结构、协议头或专有错误码。
   - 严禁将外部供应商名称、网络凭据或内部专用路由暴露到面向最终用户的公共接口、日志、UI 提示或外部文档中。
3. **保证依赖可测试与可替换**：
   - 必须确保所有外部依赖均能在离线状态下通过 Mock、Stub 或本地模拟容器进行全功能验证，防止因第三方服务不可控阻断研发与测试。

### 原则 II. 分层架构与强类型契约 (Layered Architecture & Typed Contracts)
1. **严格单向依赖**：
   - 系统依赖关系必须单向流动：**传输接入层 -> 业务编排/应用层 -> 领域核心层 -> 基础设施/持久化层**。
   - 传输接入层（如 HTTP Controller、RPC Handler、CLI 入口）不得越级绕过业务逻辑直接操作数据库或持久化层。
   - 领域核心层与核心业务逻辑严禁依赖任何具体的传输协议、网络框架、HTTP 请求上下文或 CLI 上下文。
2. **严禁弱类型与非结构化传递**：
   - 跨模块、跨服务以及前后端之间的数据交互，必须使用结构明确、具名且可执行验证的强类型 DTO（Data Transfer Object）或 Schema。
   - 核心业务实体严禁使用自由字典（无约束 Map/Dictionary/Any）进行无校验的透传与业务运算。

### 原则 III. 可观测的防御性错误处理 (Observable Failure Handling)
1. **统一公共错误语义**：
   - 所有公开接入端点必须提供统一、规范且稳定的一致性响应结构。
   - 可预期的业务失败必须映射为明确的标准化错误码与客户端可理解的安全消息；不可预期的内部异常必须经过遮蔽处理。
2. **严禁内部实现泄露**：
   - 生产环境中，严禁直接向客户端/调用方输出未经处理的堆栈信息、数据库连接或查询错误、服务器内部敏感路径、访问令牌或上游供应商报错原文。
3. **全链路追踪与结构化日志**：
   - 每个接入请求、异步任务或批处理作业在进入系统时，必须生成或沿用全局唯一的请求标识（`Trace ID` / `Request ID`）。
   - 该标识必须在进程内上下文、RPC 调用、异步事件及消息队列边界中无缝透传。
   - 系统日志必须采用结构化（JSON/键值对）格式输出，强制包含追踪标识、执行上下文、操作类型和耗时等审计要素。

### 原则 IV. 环境配置隔离与凭据数据安全 (Configuration & Credential Security)
1. **零硬编码机密**：
   - 源代码、版本控制文件、测试用例夹具、示例模板及公开文档中，绝对禁止硬编码真实访问密钥、私钥、数据库密码、内部私有 IP/域名或用户隐私数据。
2. **外部化与版本化配置**：
   - 所有环境相关的端点、阈值、容量配额及功能开关，必须统一通过版本化配置定义，并在运行时通过环境变量或受控密钥管理服务注入。
3. **数据生命周期与权限控制**：
   - 所有涉及用户输入、动态生成内容或敏感属性的模块，必须严格定义最小保留期限（TTL）、访问授权规则及显式失效/物理清理机制。

### 原则 V. 全链路自动化质量门禁 (Automated Quality Gates)
1. **风险匹配的自动化测试**：
   - 任何代码变更必须具备相匹配的自动化测试：
     - 核心业务逻辑与边界异常分支必须配备确定性且可重复执行的**单元测试**；
     - 涉及跨模块契约、存储变更、外部集成接口的核心主流程必须提供**集成测试/契约测试**。
2. **测试环境无污染与确定性**：
   - 自动化测试不得依赖未受控的外部生产网络环境、可变公网接口或非确定性物理时钟。
3. **不可逾越的 CI 门禁**：
   - 代码提交与合并前，必须强制在本地与 CI 中执行全量基础质量检查（Go 端 `make fmt`, `make lint`, `make test`, `go vet`；前端 `npm run lint`, `npm run typecheck`, `npm run test:unit`）；任何一项未通过均不得合入主分支。

---

## 第二章 前端技术架构规范 (Frontend Architecture)

### 2.1 物理目录与职责边界
- 仓库代码保持物理隔离，前端工程独立归属于 `frontend/` 根目录。
- 严禁以相对路径（`../backend/`）直接引用后端实现，两端交互必须完全基于显式版本化的 HTTP/RPC 接口契约。

### 2.2 分层架构与状态管理
1. **组件四层架构**：
   - **表现层（Presentational UI）**：纯函数式无状态组件，严格依赖 Props 渲染，禁止包含网络请求副作用；
   - **容器/视图层（Containers / Pages）**：负责页面路由级编排、读取上下文与分发动作；
   - **业务与状态层（Hooks / Stores）**：负责客户端业务状态机流转、本地缓存管理；
   - **数据接入层（Services / API Clients）**：统一封装 HTTP 请求、入参组装、响应 DTO 解析与错误捕获拦截。
2. **防御性强类型保障**：
   - 必须启用 Strict Mode 模式，严禁使用 `any` 绕过静态检查；
   - 接口返回数据在进入核心状态前需经由 DTO/Schema 校验，防止后端脏数据导致前端白屏。

### 2.3 渲染策略与多语言/SEO 架构
1. **动静渲染边界**：
   - 静态/索引内容优先采用服务端预渲染（SSG/SSR）以确保搜索引擎抓取与极致首屏性能（LCP）；
   - 用户私密会话与高频交互模块显式声明为客户端渲染（Client Components），服务端上下文严禁调用浏览器独有 API。
2. **文案多语言全隔离**：
   - 界面可见自然语言全部抽取至独立语言字典，禁止业务代码硬编码字符串；全语种字典键值结构严格镜像对称。

---

## 第三章 Go 后端技术架构规范 (Go Backend Architecture)

### 3.1 标准目录结构布局 (Directory Layout)

Go 后端工程遵循现代化标准化工程结构，实现高内聚、低耦合、按职责严格分层：

```text
backend/
├── cmd/                         # 应用程序各入口（Main 入口）
│   ├── http/                    # HTTP API 服务入口
│   │   └── main.go
│   ├── worker/                  # 异步消费与后台 Worker 服务入口
│   │   └── main.go
│   ├── mcp-admin/               # MCP (Model Context Protocol) 插件/服务入口
│   └── <cli-tool>/              # 维护、数据迁移、批处理等独立命令行工具
│       └── main.go
├── config/                      # 配置文件模板与静态配置规范
│   └── config.example.yaml
├── internal/                    # 内部私有代码（Go 编译器保护，外部包不可导入）
│   ├── core/                    # 领域核心模型与领域服务接口定义
│   ├── logic/                   # 业务逻辑编排与领域用例实现 (Service Layer)
│   ├── http/                    # HTTP 传输接入层
│   │   ├── handler/             # 控制器处理函数（参数绑定、DTO 校验、响应输出）
│   │   ├── middleware/          # 中间件（Trace ID、JWT 鉴权、CORS、Zap 日志、Recover）
│   │   └── router.go            # 路由定义与路由组装配
│   ├── persistence/             # 数据持久化基础设施层
│   │   ├── entity/              # 数据库 ORM 实体结构体与索引定义
│   │   └── repository/          # 数据访问仓储层实现（Repository Pattern）
│   ├── dto/                     # 跨层及前后端传输数据传输对象 (Data Transfer Objects)
│   ├── event/                   # 异步领域事件发布与消费消息载荷定义
│   ├── di/                      # 依赖注入容器装配 (Uber Fx 模块与 Providers)
│   │   ├── provider/            # 基础设施单例提供者（DB、Redis、Logger、Config）
│   │   └── module/              # 业务模块依赖集合
│   └── migrations/              # 数据库 Schema 迁移脚本与自动迁移逻辑
├── pkg/                         # 可复用的通用基础工具库（无业务属性，可跨项目导出）
│   ├── response/                # 标准化统一 HTTP 响应封装结构体与方法
│   ├── reqctx/                  # Context 链路上下文、Request ID 提取与透传
│   ├── jwt/                     # JWT 令牌签发、解析与验证封装
│   ├── scheduler/               # 定时任务调度器封装
│   ├── version/                 # 构建元信息注入（Commit Hash、Tag、Build Time）
│   └── upstream/                # 通用外部第三方客户端适配封装
├── scripts/                     # 构建、版本计算、环境联通运维脚本
├── Dockerfile                   # 多阶段轻量化生产镜像构建定义
├── Makefile                     # 自动化质量门禁与本地开发常用指令编排
├── go.mod                       # Go 模块依赖定义
└── go.sum                       # 依赖哈希校验锁
```

### 3.2 核心类库与技术栈选型标准

后端架构技术选型以**极致性能、高稳定性、低内存占用及强企业级支持**为基准，严格遵循如下类库清单：

| 分类 | 核心类库 | 选型用途与架构规范 |
| :--- | :--- | :--- |
| **Web 引擎** | `github.com/gin-gonic/gin` | 高性能 HTTP 路由树、灵活中间件链、参数结构体校验。 |
| **依赖注入** | `go.uber.org/fx` | 应用程序生命周期编排、模块控制反转（IoC），消除全局隐式单例。 |
| **日志框架** | `go.uber.org/zap` | 极致性能零分配结构化日志引擎，全系统统一输出 JSON 格式日志。 |
| **ORM / 持久化** | `gorm.io/gorm`<br>`gorm.io/driver/postgres`<br>`gorm.io/driver/sqlite` | 统一数据持久化抽象层，生产使用 PostgreSQL，本地/轻量环境支持 SQLite。 |
| **缓存与分布式** | `github.com/redis/go-redis/v9` | 分布式缓存、分布式锁、原子计数器及消息队列消费组。 |
| **配置管理** | `github.com/spf13/viper`<br>`github.com/subosito/gotenv` | 统一管理 YAML 配置解析与 `.env` 环境变量覆盖注入。 |
| **CLI 脚手架** | `github.com/spf13/cobra` | 命令行子命令路由管理与参数解析，用于维护工具构建。 |
| **定时调度** | `github.com/robfig/cron/v3` | 纳秒级高精度内存定时任务驱动器。 |
| **鉴权与加密** | `github.com/golang-jwt/jwt/v5`<br>`golang.org/x/crypto` | 无状态 JWT Token 颁发/验证，行业标准密码学散列加密。 |
| **唯一 ID** | `github.com/google/uuid` | 生成 RFC 4122 标准全局唯一标识符（Trace ID、Task ID、实体主键）。 |
| **文档与契约** | `github.com/swaggo/swag`<br>`github.com/swaggo/gin-swagger` | 注释驱动生成 OpenAPI / Swagger 规范交互式接口文档。 |
| **Agent / MCP** | `github.com/mark3labs/mcp-go` | 统一实现 Model Context Protocol 标准协议，提供智能体工具接口。 |
| **测试与 Mock** | `github.com/stretchr/testify`<br>`github.com/DATA-DOG/go-sqlmock` | 单元测试断言库，与离线 SQL 查询拦截模拟，杜绝测试污染生产。 |

### 3.3 架构分层与依赖注入 (DI & Fx)

1. **生命周期受控反转 (IoC)**：
   - 所有的数据库连接池、Redis 客户端、日志器、业务 Service 与 Handler 必须通过 `provider` 提供，并在 `di/module` 中组合声明。
   - 严禁在业务函数中使用 `init()` 隐式初始化或直接引用全局变量；组件间依赖必须通过结构体构造函数显式入参传递。
2. **平滑启停 (Graceful Shutdown)**：
   - 主程序（HTTP / Worker）必须监听系统终止信号（`SIGINT`, `SIGTERM`），通过 `fx.Hook` 或 `signal.NotifyContext` 在指定超时阈值（如 5~10 秒）内完成在途请求与持久化刷盘，保障零丢单。

### 3.4 统一响应与链路追踪规范

1. **统一 API 封装结构 (`pkg/response`)**：
   所有 HTTP 响应体必须统一格式输出，严禁各 Handler 自由定义零散顶层结构：
   ```go
   type Envelope struct {
       Code    int    `json:"code"`             // 业务响应状态码：0 代表成功，非 0 代表标准错误码
       Message string `json:"message"`          // 用户可理解的提示消息
       Data    any    `json:"data,omitempty"`   // 业务负载 DTO（失败或无返回时忽略）
   }
   ```
2. **全链路 Context 与 Request ID 透传 (`pkg/reqctx`)**：
   - 入口中间件检查请求头 `X-Request-ID`，若不存在则使用 `uuid.NewString()` 生成；
   - 必须通过 `reqctx.WithRequestID(ctx, id)` 写入 `context.Context`，并在响应头回写 `X-Request-ID`；
   - 所有 I/O、SQL 查询、外部 HTTP 请求及 Zap 日志记录，强制携带该 Context，确保全链路日志可通过唯一标识穿透检索。

### 3.5 异步任务与长耗时计算解耦

1. **物理进程与职责隔离**：
   - 用户交互 API 进程（`cmd/http`）与后台长耗时计算进程（`cmd/worker`）代码同仓组织，但部署时严格独立运行、独立扩缩容。
2. **瘦消息（Thin Message）流转机制**：
   - HTTP 接口仅负责创建任务记录、写入状态为 `PENDING`，并将包含 `task_id` 的瘦消息推入队列；
   - Worker 通过消费组接收消息，根据 `task_id` 读取持久化数据执行计算，并在状态机中流转状态（`RUNNING` -> `COMPLETED` / `FAILED`）。

### 3.6 基础质量门禁与 Makefile 标准 (Quality Gates)

Go 后端工程统一采用 `Makefile` 固化所有工程化门禁指令。**任何代码在提交、提 PR 及触发发布前，必须 100% 通过以下基础质量检查：**

```makefile
# 统一工程化质量管理指令标准
.PHONY: test fmt lint docs install help

# 1. 安装开发与质量工具链（锁定具体语义化版本，防止版本漂移）
install:
	go mod tidy
	go mod download
	go install github.com/golangci/golangci-lint/cmd/golangci-lint@v1.64.8
	go install github.com/swaggo/swag/cmd/swag@v1.16.6

# 2. 代码格式化检查：必须符合官方格式化规范，自动排除生成目录
fmt:
	gofmt -w $$(find . -name '*.go' -not -path './generated/*')

# 3. 静态代码分析与质量扫描：通过 golangci-lint 与 go vet 消除隐患
lint:
	golangci-lint run
	go vet ./...

# 4. 单元与集成测试：强制 --count=1 禁用测试结果缓存，全量验证断言
test:
	go test ./... --count=1

# 5. API 契约文档自动同步：代码修改公共接口后必须重新生成 Swagger 契约
docs:
	swag init -g cmd/http/main.go --parseInternal -o ./generated/docs
```

#### 质量门禁执行准则：
1. **零警告/零报错通过 `make lint`**：
   - 严禁忽略未处理的 `error` 返回值；
   - 严禁引入 Goroutine 泄漏、Context 未传递、数据竞争（Data Race）或无用冗余变量；
   - 必须通过 `go vet ./...` 检查潜在可疑结构与格式化占位符不匹配问题。
2. **零缓存通过 `make test`**：
   - 执行测试时强制附加 `--count=1` 参数，杜绝误信本地 Cache；
   - 核心领域服务、Handler 入参校验及 DTO 转换的单测覆盖率必须满足发布要求；
   - 测试运行必须使用本地隔离环境或 Mock 机制（如 `go-sqlmock`），严禁在单测中依赖外网真实服务或生产数据库。
3. **接口契约同步性检查 (`make docs`)**：
   - 凡涉及 `cmd/http` 暴露的路由、请求结构体、响应 DTO 增删改，必须运行 `make docs` 更新 `generated/docs/` 下的 Swagger 规格定义；
   - 禁止出现代码实现与 Swagger 文档不一致的现象。

---

## 第四章 接口、数据与兼容性规范 (Contract & Compatibility)

### 4.1 跨端契约演进与向后兼容
1. **契约向后兼容性**：
   - 公开接口或跨端契约的演进必须保持向后兼容（Backward Compatibility）：
     - 允许向响应 DTO 添加非破坏性字段；
     - 严禁在原有接口中直接删除现有字段、重命名现有字段或修改现有字段的数据类型及语义。
2. **破坏性变更版本化**：
   - 当无法保持向后兼容时，必须创建新版本路由（如 `/v2/`）或全新接口，并制定清晰的弃用（Deprecation）周期、迁移指南与双版本并行过渡方案。

### 4.2 敏感数据保护与最小权限原则
1. **脱敏与最小暴露**：
   - 返回给前端客户端的数据结构，严禁包含后端内部管理字段（如内部加密盐值、哈希密码、未脱敏第三方凭据、逻辑删除标记等）。
   - 数据脱敏必须在后端基础设施与传输层完成，前端不得接收未脱敏敏感数据再行本地过滤。
2. **接口访问控制**：
   - 运营控制类端点、管理类 API 必须与面向普通用户的公开 API 保持严格物理路由隔离与独立鉴权链路（如专属 API Key 校验）。

### 4.3 可重现构建与单一真相来源
1. **构建输出可重现**：
   - 所有的生成产物（二进制、静态资源、容器镜像）必须完全可通过受版本控制的源代码与工程命令机械重现。
2. **严禁手工篡改产物**：
   - 严禁手工修改打包产物、生产服务器文件或生成物，任何变更必须从源码仓库出发，通过标准流水线进行构建发布。源码必须是系统行为的唯一真相来源。

---

## 第五章 开发、审查与发布流程 (Development & Delivery Lifecycle)

### 5.1 规范驱动开发 (Spec-Driven)
1. **先设计后编码**：
   - 复杂功能或重大技术重构在动手编码之前，必须先产出规格设计说明（Spec / RFC），明确功能边界、数据流转、接口契约与验收测试准则。
2. **全生命周期可追溯**：
   - 需求拆解、实现代码、单元测试与发布记录必须与架构设计规范保持一一对应的可追溯关系。

### 5.2 代码审查门禁标准
代码合并前，审查人员必须重点核验以下项目：
- [ ] 是否通过了 Go 后端全量质量门禁（`make fmt && make lint && make test`）？
- [ ] 是否通过了前端全量质量门禁（`npm run lint && npm run typecheck && npm run test:unit`）？
- [ ] 是否违反了分层隔离与单向依赖？
- [ ] 是否存在未封装的第三方专有依赖？
- [ ] 是否在源码、测试或配置文件中硬编码了机密或私有地址？
- [ ] 错误处理是否安全？是否暴露了未捕获的内部堆栈或敏感信息？
- [ ] 新增或修改的逻辑是否具备充分且可重复的自动化测试验证？
- [ ] 接口变更是否保持了向后兼容？是否同步执行了 `make docs`？

### 5.3 基于 Git Tag 的唯一发布真相
1. **标签即版本**：
   - 系统的正式发布必须严格以代码仓库中受版本控制的 **Git Tag**（遵循 `vMAJOR.MINOR.PATCH` 语义化版本规则）作为触发与分发的唯一真相来源。
   - 严禁在 CI/CD 流水线中使用临时分支名称、动态时间戳或随机构建号作为生产部署标识。
2. **镜像与配置一致性**：
   - 构建产物、部署清单（Manifest）、容器镜像标签与发布日志中引用的版本号必须完全一致，确保可追溯性与环境一致性。

### 5.4 保留历史的平滑回滚机制
1. **快速安全回退**：
   - 生产发布必须具备确定的回滚机制。当检测到严重缺陷时，优先通过重新部署上一稳定版本的 Git Tag 产物实现平滑恢复。
2. **严禁破坏 Git 历史**：
   - 代码仓库的主干分支（`main` / `master`）严禁使用强行推送（Force Push / Reset）来抹除问题提交，必须采用正向修复或回退提交（`git revert`）的方式保留完整的历史可追溯记录。

---

## 第六章 宪法效力与修订治理 (Governance & Amendments)

### 最高效力原则 (Supremacy)
1. 本技术架构宪法高于项目内部的一切口头约定、团队临时习惯及随意性实现偏好。
2. 任何工程规划、代码审查、技术方案设计及交付验收，均必须将本宪法要求作为基础核验标准。
3. 若确需设立技术债务或临时特例（Exemption），必须在方案设计中显式记录偏离条款、风险评估、应急措施、批准人及修复期限。

### 修订治理规则 (Amendment Policy)
1. **修订方式**：对本宪法的任何修改，必须以 Pull Request 形式提交修改提案，并由核心架构维护团队一致审查批准后方可合入。
2. **语义化版本演进**：
   - **MAJOR（主版本号）**：当废除既有核心治理原则、或对系统基础架构设计造成颠覆性调整时递增；
   - **MINOR（次版本号）**：当新增架构原则、补充重要约束或完善工程流程规则时递增；
   - **PATCH（修订号）**：仅用于错别字订正、排版美化或非实质语义的表述澄清时递增。
