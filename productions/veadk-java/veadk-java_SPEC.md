# VeADK Java 文档规范

本规范适用于 `productions/veadk-java/preview/{zh,en}` 下的 VeADK Java 文档。根目录 `SPEC.md` 是上位规范；若本文件与根规范冲突，以根规范为准。本文件补充 Java SDK 文档的目录、事实来源、示例和验证要求。

## 内容范围

- 只把 `preview` 作为当前版本来源；`archives` 仅用于已发布版本事实修正。
- VeADK Java 文档面向使用 Java SDK 构建、运行、部署和观测智能体应用的开发者。
- 文档覆盖公开 Maven 坐标、JDK 要求、`Agent.builder()`、`Runner`、模型配置、工具、技能、短期记忆、长期记忆、知识库、可观测、A2A、AgentKit Runtime、Web UI 和项目示例。
- 不写源码走读，不记录内部文件名、内部函数、私有机制、测试辅助类型或未发布能力。
- 中文为默认语言，英文并列；`zh` 与 `en` 路径、页面层级、参数、示例、限制和警告必须等价。

## 版本线与触发规则

`veadk-java` 当前按单基线维护文档：

| 来源 | 目标目录 | 说明 |
| --- | --- | --- |
| `main` 分支 | `productions/veadk-java/preview/{zh,en}` | 默认文档入口，反映当前开发态。 |
| release tag，例如 `v0.0.2` | `productions/veadk-java/archives/<version>/{zh,en}` | 已发布版本快照，目录版本名去掉 tag 前缀 `v`。 |

生成 Preview PR 时必须维护 `.docs-autogen/veadk-java-preview.json`，记录本次 Preview 对应的 `main` commit。生成 Archive 时，必须先由 runtime 脚本确认 tag commit 属于 `main`，不能交给生成文档的 agent 自行判断；只有当该元数据的 `source_head_sha` 与 tag commit 完全一致时，才可以复制 Preview；不一致时应等待 Preview 合入，超时后基于 tag 源码生成 archive，不能复制过旧或过新的 Preview。

## 事实来源

更新 VeADK Java 文档时，按以下优先级确认事实：

1. `veadk-java` 目标分支或目标 tag 的公开 Java API、`pom.xml`、公开示例和 README。
2. `core/src/main/java/**`、`example/src/main/java/**`、`docs/*.md` 和公开测试中体现的可运行用法。
3. ADK Java、A2A Java、OpenTelemetry、Maven Central 等上游公开文档；引用上游行为时必须区分哪些由上游提供，哪些由 VeADK Java 封装。
4. 当前 docs 仓中的 VeADK Python 和 VeADK Go 文档只作为信息架构和文风参考，不能直接迁移 import、包名、参数名或默认值。

不得根据 Python SDK、Go SDK 或旧页面推断 Java SDK 的公开 API。无法从 Java 仓库确认的默认值、环境变量、参数和支持范围，不得写成确定事实。

## 目录与导航

`preview` 目录固定为双语结构：

```text
productions/veadk-java/preview/
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

- `get-started/`：安装、Maven 依赖、模型凭证、快速开始。
- `components/agent/`：`Agent.builder()`、模型、工具、子智能体、metadata 和 Python 能力边界。
- `components/runner/`：`Runner.run(...)`、用户与会话、`RunConfig`、多模态输入和最终文本提取。
- `components/tools/`：函数工具、MCP Router、Skills Sandbox、Web Search、Web Fetch、媒体生成、Lark、Vanna、代码执行等内置工具。
- `components/session/`：短期记忆和本地、SQLite、MySQL、PostgreSQL 等会话存储。
- `components/memory/`：长期记忆、Mem0、Viking Memory 和自动保存。
- `components/knowledge/`：Viking、OpenSearch 等知识库后端。
- `components/observability/`：OpenTelemetry、APMPlus、TLS 和相关导出器。
- `components/security/`：模型凭证、身份授权、LLM Shield、工具凭证和安全边界。
- `apps/`：ADK Web、AgentKit Runtime、AgentKit Server、A2A Server 和示例应用。
- `deploy/`：容器镜像、AgentKit Runtime 和线上运行说明。

新增、删除或移动页面时，必须同步更新 `docs.json` 中 VeADK Java Preview 导航和中英文页面。

## 页面类型

### 入口页

入口页包括 `index.mdx` 和各组件目录的 `index.mdx`。这类页面应：

- 用首段说明 VeADK Java 的能力范围、适用场景和当前 Maven 坐标。
- 用表格列出核心模块、公开类或包名和使用场景。
- 明确当前功能范围与 Python 侧能力差异，不把未支持的 Python 能力写成 Java 已支持。
- 链接使用站点根路径，不带 `.mdx` 后缀。
- 不展开全部参数；详细参数应放到对应子页面。

### 安装与快速开始

安装页应写清：

- JDK 版本要求，以根 `pom.xml` 和 README 为准。
- Maven 坐标、当前版本号和源码构建方式。
- 模型凭证的解析顺序、必需环境变量和 AK/SK fallback 条件。
- 示例模块的构建方式和运行入口。

快速开始应覆盖：

1. 创建 Maven 项目并添加依赖；
2. 配置模型凭证；
3. 使用 `Agent.builder()` 创建最小智能体；
4. 使用 `new Runner(agent).run(...)` 执行一次对话；
5. 运行仓库示例或 Web UI。

示例必须使用当前公开包名和类名，例如 `com.volcengine.veadk.Agent`、`com.volcengine.veadk.Runner`。如果使用仓库示例入口，必须以源码当前包路径为准。

### 组件说明

组件页面应先回答“是什么、何时使用”，再给出最小 Java 示例，最后列出公开参数和限制。推荐结构：

1. 首段说明能力。
2. `## 何时使用` 或 `## 功能说明`。
3. `## 依赖`，仅当需要额外服务、凭证或上游包时出现。
4. `## 使用示例`。
5. `## 参数` 或 `## 配置`，覆盖公开 builder 方法、构造参数、环境变量和默认值。
6. 必要的 `<Note>`、`<Tip>`、`<Warning>`。

不要把 Java 源码中的内部 helper、私有字段或测试替身写入正文。用户需要直接 import、调用或配置的公开类、构造函数、builder 方法、环境变量和 Maven 坐标可以保留并写清。

### 后端页面

会话、记忆和知识库后端页面必须独立完整，不能依赖读者去其他后端页面补齐参数。每个后端页面至少包括：

- 标题使用「使用 xxx 存储」或现有产品化标题。
- 首段说明该后端使用的服务或本地资源、是否适合生产、关键能力。
- `## 何时使用`：列出适用场景。
- `## 依赖`：列出 Maven 依赖、外部服务、网络和凭证要求。
- `## 使用示例`：给出最小可运行 Java 示例；如支持 Spring 或配置文件，再给一个配置示例。
- `## 参数`：按“构造参数”“builder 方法”“环境变量配置”等分节列出。
- 表格列必须包含 `参数/配置项`、`类型`、`默认值`、`说明`；环境变量表包含 `环境变量`、`默认值`、`说明`。

默认值必须从 Java 代码、测试或 README 核对，不能根据字段名猜测。

### 工具与技能页面

工具页面应覆盖：

- 工具用途和适用场景。
- Maven 依赖、外部服务或凭证要求。
- 注册到 `Agent` 的方式。
- 最小可运行 Java 示例。
- 输入输出结构、异步任务、多模态能力或工具确认等特殊行为。
- 安全边界，例如网络请求、代码执行、文件读取、外部服务访问和凭证传递。

技能页面必须区分本地 skills、classpath skills、远程 skill source 和 Skills Sandbox。写清 `SKILL.md` frontmatter 要求、路径处理方式、同名 skill 覆盖规则和远端刷新失败的行为。

### 应用与部署页面

应用封装和部署页面应采用操作指南结构：

- 写清前置条件、账号权限、运行环境和必要环境变量。
- 给出可复制 Maven 命令、容器构建命令和配置。
- 说明启动、访问、部署和验证方式。
- 涉及公网访问、A2A 服务、OAuth、云资源、IAM Role、环境变量透传和用户数据时，必须在操作前说明风险和影响。

### 更新日志

`changelog.mdx` 使用表格组织版本变更，列为 `模块 / 变更细节 / 链接`。要求：

- Preview 中尚未对应已发布 tag 的变更统一放在 `未发布` / `Unreleased` 分节。
- 如果 Java 仓库没有 tag，应明确当前基线来自 `main` 分支和 `pom.xml` 版本，不把源码基线写成已发布 tag。
- 每条变更对应用户可感知能力、行为变化或兼容性变化。
- 链接指向 Preview 中对应说明页。
- 不写内部提交、内部任务编号或源码文件。
- 中英文更新日志的版本顺序、条目和链接等价。

## 示例与代码

- Java 示例使用 `java` 代码块；Maven XML 使用 `xml`；命令使用 `bash`；YAML 使用 `yaml`。
- 代码块可标注用户自建文件名，例如 `Main.java` 或 `pom.xml`，但不要暴露内部源码路径。
- 示例应最小、完整、可运行。包名、类名、builder 方法、环境变量和 Maven 坐标必须与公开 API 一致。
- 凭证使用环境变量、占位符或产品支持的安全配置，不把 API Key、AK/SK、Token 写入正文。
- 涉及火山方舟、VikingDB、Redis、OpenSearch、Mem0、MySQL、PostgreSQL、APMPlus、Cozeloop、TOS 等外部服务时，必须说明连接配置、区域、凭证和网络可达性。
- 关键示例发布前应运行 Maven 编译、单测或至少用公开测试与 README 示例核对签名。无法执行时，在交付说明中说明验证范围。

## 排版与组件

- YAML frontmatter 提供 `title`，保留并维护语言切换使用的 `en_link` / `zh_link`，不添加 `description` 或页面副标题。
- 标题使用描述性短语，不使用数字序号。
- 产品名、包名、类名、构造参数、builder 方法、配置键、环境变量、模型名和协议名使用反引号或原文。
- 优先用普通 Markdown 表格承载参数、依赖、后端能力矩阵和环境变量。
- Mintlify 组件使用规则：
  - `<Note>`：补充默认行为、兼容性、前置说明。
  - `<Tip>`：推荐实践或便捷路径。
  - `<Warning>`：安全风险、费用、删除、覆盖、废弃和兼容性破坏。
  - `<Steps>` / `<Step>`：必须顺序执行的操作。
  - `<Tabs>` / `<Tab>`：互斥配置方式、模型提供方差异或运行方式差异。
- 不添加空泛总结和“下一步”章节，除非该页确实需要引导读者进入下一项任务。

## 术语与命名

- 中文正文中的 “agent” 写作「智能体」；公开包名、类名和配置键保持原文。
- `runner` 按语境写作 `Runner` 或「Runner」；`runtime` 译为「运行时」；`session` 译为「会话」；`memory` 按语境译为「记忆」或「长期记忆」；`knowledge base` 译为「知识库」。
- ADK Java、A2A、MCP、OpenTelemetry、OTLP、APMPlus、Cozeloop、VikingDB、OpenSearch、Redis、Mem0、TOS、API Key、Maven、JDK 等技术标识保留。
- 中文标题不要写括号译名，例如不写「A2A（Agent-to-Agent）」。

## 更新检查

更新 VeADK Java 文档前后至少检查：

- 公开 API、构造参数、builder 方法、默认值、配置键、环境变量和 Maven 依赖与目标版本一致。
- Maven 坐标和版本号没有与源码 `pom.xml` 冲突。
- 已检查 README、examples、公开测试、`docs/*.md` 和 `pom.xml`。
- 新增能力已说明安装、配置、示例、限制和安全影响。
- 后端页面参数覆盖完整，并包含可运行示例。
- Changelog 新增条目的目标页面实际包含对应能力。
- 中英文路径和内容等价。
- 新页面已加入 `docs.json` Preview 导航。
- 内部实现细节、源码路径、内部异常类型、测试辅助类型和未发布能力未出现在正文。
- 关键 Java 示例已经运行或按公开签名核对。
