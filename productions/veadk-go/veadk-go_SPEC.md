# VeADK Go 文档规范

本规范适用于 `productions/veadk-go/preview/{zh,en}` 下的 VeADK Go 文档。根目录 `SPEC.md` 是上位规范；若本文件与根规范冲突，以根规范为准。本文件补充 Go SDK 文档的目录、事实来源、示例和验证要求。

## 内容范围

- 只把 `preview` 作为当前版本来源；`archives` 仅用于已发布版本事实修正。
- VeADK Go 文档面向使用 Go SDK 构建、运行、编排、部署和观测智能体应用的开发者。
- 当前主线以 `github.com/volcengine/veadk-go/v2` 模块为默认说明对象；V1 只作为兼容性或迁移说明出现，不能让新项目示例默认使用 V1 import 路径。
- 文档覆盖公开模块、配置、安装、模型、智能体、Runner、Workflow Agent、工具、会话、记忆、知识库、可观测、A2A、应用封装与部署。
- 不写源码走读，不记录内部文件名、内部函数、私有机制、测试辅助类型或未发布能力。
- 中文为默认语言，英文并列；`zh` 与 `en` 路径、页面层级、参数、示例、限制和警告必须等价。

## 版本线与触发规则

`veadk-go` 是 V1/V2 双基线项目，文档生成和归档必须按分支与 tag 来源分流：

| 来源 | 版本线 | 目标目录 | 模块路径 | 说明 |
| --- | --- | --- | --- | --- |
| `main` 分支 | V2 Preview | `productions/veadk-go/preview/{zh,en}` | `github.com/volcengine/veadk-go/v2` | 默认文档入口，反映当前 V2 开发态；在 V2 正式 tag 前不生成 V2 archive。 |
| `release/v1` 分支上的 `v1.x` tag | V1 Archive | `productions/veadk-go/archives/<version>/{zh,en}` | `github.com/volcengine/veadk-go` | 面向存量 V1 用户，内容必须以对应 tag 的代码和示例为准。 |
| `main` 分支上的 `v2.x` tag | V2 Archive | `productions/veadk-go/archives/<version>/{zh,en}` | `github.com/volcengine/veadk-go/v2` | 仅在 V2 稳定 tag 发布后生成，内容固定到该 tag。 |

生成 Preview 时只读取 `main` 分支事实，不得把 `release/v1` 的 V1 API 写成新项目默认用法。生成 Preview PR 时必须维护 `.docs-autogen/veadk-go-preview.json`，记录本次 Preview 对应的 `main` commit。

生成 Archive 时必须先由 runtime 脚本确认 tag commit 属于对应版本线，不能交给生成文档的 agent 自行判断：`v1.x` tag 应来自 `release/v1` 或 `release-v1`，`v2.x` tag 应来自 `main`；无法确认来源时应停止生成并给出明确错误。V1 archive 不得复用 V2 Preview 文本后只替换 import 路径，必须按 V1 tag 重新核对 API、示例、Go 版本和依赖。V2 archive 可以在 `.docs-autogen/veadk-go-preview.json` 的 `source_head_sha` 与 tag commit 完全一致时复制 Preview；不一致时应等待 Preview 合入，超时后基于 tag 源码生成 archive，不能复制过旧或过新的 Preview。

## 事实来源

更新 VeADK Go 文档时，按以下优先级确认事实：

1. `veadk-go` 目标分支或目标 tag 的公开 Go API、`go.mod`、公开配置结构和公开示例。
2. `README.md`、`README_zh.md`、`examples/**`、公开测试中体现的可运行用法。
3. ADK Go、A2A Go、OpenTelemetry 等上游公开文档；引用上游行为时必须区分哪些由上游提供，哪些由 VeADK Go 封装。
4. 当前 docs 仓中的 VeADK Python 文档只作为信息架构和文风参考，不能直接迁移 Python import、包名、参数名或默认值。

不得根据 Python SDK、Java SDK 或旧 GitBook 页面推断 Go SDK 的公开 API。无法从 Go 仓库确认的默认值、环境变量、参数和支持范围，不得写成确定事实。

## 目录与导航

`preview` 目录固定为双语结构：

```text
productions/veadk-go/preview/
├── zh/
│   ├── index.mdx
│   ├── changelog.mdx
│   ├── get-started/
│   ├── components/
│   ├── apps/
│   └── deploy/
└── en/
    ├── index.mdx
    ├── changelog.mdx
    ├── get-started/
    ├── components/
    ├── apps/
    └── deploy/
```

首版 bootstrap 可以只生成 `index`、`changelog`、`get-started/installation` 和 `get-started/quickstart`，随后按公开能力逐步扩展。扩展时优先使用以下分区：

- `get-started/`：安装、版本支持、配置模型、快速开始。
- `components/agent/`：`llmagent`、远程智能体、子智能体和 Workflow Agent。
- `components/runner/`：`runner.Run`、`runner.RunText`、多轮会话、多模态输入、事件流和自动保存。
- `components/tools/`：函数工具、MCP 工具集、Web Search、Web Fetch、Link Reader、代码执行、图像、视频、语音和火山引擎工具能力。
- `components/session/`：短期会话和 SQLite、MySQL、PostgreSQL 等会话存储。
- `components/memory/`：长期记忆和 Redis、OpenSearch、VikingDB、Mem0 等后端。
- `components/knowledge/`：本地、Redis、Viking、OpenSearch 等知识库后端。
- `components/observability/`：OpenTelemetry、OTLP、APMPlus、Cozeloop、文件和 stdout 导出。
- `apps/`：Simple App、A2A App、AgentKit Server App 等应用封装。
- `deploy/`：与 AgentKit、A2A 服务或其他运行环境相关的部署说明。

新增、删除或移动页面时，必须同步更新 `docs.json` 中 VeADK Go Preview 导航和中英文页面。

## 页面类型

### 入口页

入口页包括 `index.mdx` 和各组件目录的 `index.mdx`。这类页面应：

- 用首段说明 VeADK Go 的能力范围、适用场景和默认版本线。
- 用表格列出核心模块、Go import 路径和使用场景。
- 明确 V2 与 V1 的关系：V2 是新项目默认路径，V1 是维护线。
- 链接使用站点根路径，不带 `.mdx` 后缀。
- 不展开全部参数；详细参数应放到对应子页面。

### 安装与快速开始

安装页应写清：

- Go 版本要求，以 `go.mod` 和 README 为准。
- V2 模块安装命令和 import 路径。
- V1 维护分支和 V1 import 路径，仅作为存量项目说明。
- 配置模型所需的 `config.yaml`、环境变量或凭证来源。

快速开始应覆盖：

1. 创建 Go module；
2. 安装 VeADK Go V2；
3. 配置模型；
4. 编写最小智能体；
5. 通过命令行或 Web UI 运行并验证结果。

代码示例必须使用 V2 import 路径。V1 示例只放在兼容性说明中，且必须明确适用范围。

### 组件说明

组件页面应先回答“是什么、何时使用”，再给出最小 Go 示例，最后列出公开参数和限制。推荐结构：

1. 首段说明能力。
2. `## 何时使用` 或 `## 功能说明`。
3. `## 依赖`，仅当需要额外服务、凭证或上游包时出现。
4. `## 使用示例`。
5. `## 参数` 或 `## 配置`，覆盖公开构造参数、配置项、环境变量和默认值。
6. 必要的 `<Note>`、`<Tip>`、`<Warning>`。

不要把 Go 源码中的内部 helper、私有字段或测试替身写入正文。用户需要直接 import、调用或配置的公开包、类型、函数、结构字段和环境变量可以保留并写清。

### 后端页面

会话、记忆和知识库后端页面必须独立完整，不能依赖读者去其他后端页面补齐参数。每个后端页面至少包括：

- 标题使用「使用 xxx 存储」或现有产品化标题。
- 首段说明该后端使用的服务或本地资源、是否适合生产、关键能力。
- `## 何时使用`：列出适用场景。
- `## 依赖`：列出 Go 包、外部服务、网络和凭证要求。
- `## 使用示例`：给出最小可运行 Go 示例；如支持 `config.yaml`，再给一个配置示例。
- `## 参数`：按“构造参数”“连接配置”“环境变量配置”等分节列出。
- 表格列必须包含 `参数/配置项`、`类型`、`默认值`、`说明`；环境变量表包含 `环境变量`、`默认值`、`说明`。

默认值必须从 Go 代码、测试或 README 核对，不能根据字段名猜测。

### 工具页面

工具页面应覆盖：

- 工具用途和适用场景。
- 安装、外部服务或凭证要求。
- 注册到 `Agent` 的方式。
- 最小可运行 Go 示例。
- 输入输出结构、上下文参数、长时任务、多模态能力或工具确认等特殊行为。
- 安全边界，例如网络请求、代码执行、文件读取、外部服务访问和凭证传递。

函数工具必须说明 Go 函数签名、参数结构、返回结构、错误处理，以及注册到智能体的方式。MCP 工具集必须说明 stdio、SSE 或 streamable HTTP 等传输方式的支持范围。

### 应用与部署页面

应用封装和部署页面应采用操作指南结构：

- 写清前置条件、账号权限、运行环境和必要环境变量。
- 给出可复制命令和配置。
- 说明启动、访问、部署和验证方式。
- 涉及公网访问、A2A 服务、OAuth、云资源、IAM Role、环境变量透传和用户数据时，必须在操作前说明风险和影响。

### 更新日志

`changelog.mdx` 使用表格组织版本变更，列为 `模块 / 变更细节 / 链接`。要求：

- Preview 中尚未对应已发布 tag 的变更统一放在 `未发布` / `Unreleased` 分节。
- Go 仓库存在 V1/V2 分支差异时，变更条目必须标明影响的版本线或模块路径。
- 每条变更对应用户可感知能力、行为变化或兼容性变化。
- 链接指向 Preview 中对应说明页。
- 不写内部提交、内部任务编号或源码文件。
- 中英文更新日志的版本顺序、条目和链接等价。

## 示例与代码

- Go 示例使用 `go` 代码块；命令使用 `bash`；配置使用 `yaml`。
- 代码块可标注用户自建文件名，例如 `main.go` 或 `config.yaml`，但不要暴露内部源码路径。
- 示例应最小、完整、可运行。import 路径、包别名、类型名、字段名和函数名必须与公开 API 一致。
- 新项目示例默认使用 `github.com/volcengine/veadk-go/v2` 和 `google.golang.org/adk/v2`。
- 凭证使用环境变量、`config.yaml` 占位符或产品支持的安全配置，不把 API Key、AK/SK、Token 写入正文。
- 涉及火山方舟、VikingDB、Redis、OpenSearch、Mem0、MySQL、PostgreSQL、APMPlus、Cozeloop、TOS 等外部服务时，必须说明连接配置、区域、凭证和网络可达性。
- 关键示例发布前应运行 `go test`、`go run` 或至少用 `go test ./...`、公开测试与 README 示例核对签名。无法执行时，在交付说明中说明验证范围。

## 排版与组件

- YAML frontmatter 提供 `title`，保留并维护语言切换使用的 `en_link` / `zh_link`，不添加 `description` 或页面副标题。
- 标题使用描述性短语，不使用数字序号。
- 产品名、包名、类型名、构造参数、结构字段、配置键、环境变量、模型名和协议名使用反引号或原文。
- 优先用普通 Markdown 表格承载参数、依赖、后端能力矩阵和环境变量。
- Mintlify 组件使用规则：
  - `<Note>`：补充默认行为、兼容性、前置说明。
  - `<Tip>`：推荐实践或便捷路径。
  - `<Warning>`：安全风险、费用、删除、覆盖、废弃和兼容性破坏。
  - `<Steps>` / `<Step>`：必须顺序执行的操作。
  - `<Tabs>` / `<Tab>`：互斥配置方式、V1/V2 差异或运行方式差异。
- 不添加空泛总结和“下一步”章节，除非该页确实需要引导读者进入下一项任务。

## 术语与命名

- 中文正文中的 “agent” 写作「智能体」；公开包名、类型名和配置键保持原文。
- `runner` 按语境写作 `Runner` 或「Runner」；`runtime` 译为「运行时」；`session` 译为「会话」；`memory` 按语境译为「记忆」或「长期记忆」；`knowledge base` 译为「知识库」。
- ADK、A2A、MCP、OpenTelemetry、OTLP、APMPlus、Cozeloop、VikingDB、OpenSearch、Redis、Mem0、TOS、API Key 等技术标识保留。
- 中文标题不要写括号译名，例如不写「A2A（Agent-to-Agent）」。

## 更新检查

更新 VeADK Go 文档前后至少检查：

- 公开 API、构造参数、结构字段、默认值、配置键、环境变量和依赖与目标版本一致。
- V2 / V1 模块路径没有混用；新项目示例默认使用 V2。
- 已检查 README、examples、公开测试和 `go.mod`。
- 新增能力已说明安装、配置、示例、限制和安全影响。
- 后端页面参数覆盖完整，并包含可运行示例。
- Changelog 新增条目的目标页面实际包含对应能力。
- 中英文路径和内容等价。
- 新页面已加入 `docs.json` Preview 导航。
- 内部实现细节、源码路径、内部异常类型、测试辅助类型和未发布能力未出现在正文。
- 关键 Go 示例已经运行或按公开签名核对。
