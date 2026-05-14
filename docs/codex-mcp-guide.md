# Codex 接入 Java Sentient MCP 指南

本文档介绍如何在 OpenAI Codex 中接入本项目提供的 Java MCP Server，让 Codex 可以调用 Eclipse JDT.LS 的 Java 代码智能能力。

Codex 的 MCP 配置在 CLI 和 IDE 扩展之间共享。你可以使用 `codex mcp add` 命令添加，也可以直接编辑 `~/.codex/config.toml`。

## 适用场景

接入后，Codex 可以通过本 MCP Server 完成以下 Java 工程任务：

- 启动 Java Language Server。
- 加载 Maven 项目。
- 搜索类、方法、字段等符号。
- 获取定义、引用、Hover 信息。
- 获取 Java 编译错误和警告。
- 读取 JDK 或外部依赖类内容。

## 前置条件

请先准备以下环境：

| 依赖 | 要求 |
| --- | --- |
| Codex CLI | 已安装并可执行 `codex` |
| Node.js | 18.0.0 或更高版本 |
| JDK | Java 17 或更高版本，推荐 Java 21+ |
| Eclipse JDT.LS | 已下载并解压到本地目录 |
| 本项目 | 已执行 `npm install` 和 `npm run build` |

在本项目根目录构建：

```powershell
cd D:/idea_projects/Java-Sentient-MCP
npm install
npm run build
```

构建完成后，MCP Server 入口文件应位于：

```text
D:/idea_projects/Java-Sentient-MCP/dist/index.js
```

## 推荐接入方式：使用 codex mcp add

本项目是本地 stdio MCP Server，可以使用 `codex mcp add <NAME> -- <COMMAND>` 添加。

PowerShell 示例：

```powershell
codex mcp add java-jdtls `
  --env JDTLS_HOME="D:/software/jdt-language-server-latest" `
  --env JDTLS_JAVA_HOME="D:/software/java/jdk-21" `
  --env JDTLS_JAVA_RUNTIMES="[{`"name`":`"JavaSE-17`",`"path`":`"D:/software/jdk-17`"}]" `
  -- node D:/idea_projects/Java-Sentient-MCP/dist/index.js
```

macOS / Linux 示例：

```bash
codex mcp add java-jdtls \
  --env JDTLS_HOME="/opt/jdtls" \
  --env JDTLS_JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home" \
  --env JDTLS_JAVA_RUNTIMES='[{"name":"JavaSE-17","path":"/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home"}]' \
  -- node /absolute/path/to/Java-Sentient-MCP/dist/index.js
```

命令说明：

| 参数 | 说明 |
| --- | --- |
| `java-jdtls` | MCP Server 名称，建议保持简短明确。 |
| `--env KEY=VALUE` | 传给 MCP Server 进程的环境变量。 |
| `--` | 分隔 Codex 参数和 MCP Server 启动命令。 |
| `node .../dist/index.js` | 启动本项目 MCP Server 的实际命令。 |

## 验证接入

查看 MCP Server 列表：

```powershell
codex mcp list
```

查看指定 MCP Server 详情：

```powershell
codex mcp get java-jdtls
```

如果需要移除配置：

```powershell
codex mcp remove java-jdtls
```

## 直接编辑 config.toml

也可以直接编辑 Codex 配置文件：

```text
~/.codex/config.toml
```

Windows 通常位于：

```text
C:/Users/<你的用户名>/.codex/config.toml
```

示例配置：

```toml
[mcp_servers.java-jdtls]
command = "node"
args = ["D:/idea_projects/Java-Sentient-MCP/dist/index.js"]

[mcp_servers.java-jdtls.env]
JDTLS_HOME = "D:/software/jdt-language-server-latest"
JDTLS_JAVA_HOME = "D:/software/java/jdk-21"
JDTLS_JAVA_RUNTIMES = "[{\"name\":\"JavaSE-17\",\"path\":\"D:/software/jdk-17\"}]"
JDTLS_MAVEN_USER_SETTINGS = "D:/software/apache-maven/conf/settings.xml"
JDTLS_MAVEN_OFFLINE = "false"
```

macOS / Linux 示例：

```toml
[mcp_servers.java-jdtls]
command = "node"
args = ["/absolute/path/to/Java-Sentient-MCP/dist/index.js"]

[mcp_servers.java-jdtls.env]
JDTLS_HOME = "/opt/jdtls"
JDTLS_JAVA_HOME = "/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home"
JDTLS_JAVA_RUNTIMES = "[{\"name\":\"JavaSE-17\",\"path\":\"/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home\"}]"
```

修改 `config.toml` 后，建议重启 Codex CLI 或 IDE 扩展。

## 使用 npx 启动

如果希望使用 npm 包，而不是本地源码构建产物，可以配置为：

```powershell
codex mcp add java-jdtls `
  --env JDTLS_HOME="D:/software/jdt-language-server-latest" `
  --env JDTLS_JAVA_HOME="D:/software/java/jdk-21" `
  -- npx -y @cleverdonaldduck/java-jdtls-mcp-server@latest
```

对应 `config.toml` 写法：

```toml
[mcp_servers.java-jdtls]
command = "npx"
args = ["-y", "@cleverdonaldduck/java-jdtls-mcp-server@latest"]

[mcp_servers.java-jdtls.env]
JDTLS_HOME = "D:/software/jdt-language-server-latest"
JDTLS_JAVA_HOME = "D:/software/java/jdk-21"
```

本地开发和调试本项目时，推荐使用 `node dist/index.js`，这样 Codex 使用的是当前仓库构建出的版本。

## 推荐 AGENTS.md 规则

为了让 Codex 更稳定地使用本 MCP，可以在 Java 项目根目录添加或更新 `AGENTS.md`：

```md
# Java MCP Usage

处理 Java 项目时，优先使用 `java-jdtls` MCP。

工作流要求：

1. 先调用 `java_get_status` 检查 Java Language Server 状态。
2. 如未启动，调用 `java_start` 并等待状态变为 `READY`。
3. 对 Maven 项目，优先调用 `java_load_maven_project` 建立索引。
4. 查找类、方法、字段时，优先使用 `java_search_symbols`。
5. 需要理解符号时，使用 `java_get_definition`、`java_get_references` 或 `java_get_hover`。
6. 修改 Java 文件后，必须调用 `java_open_file` 同步内容。
7. 同步后调用 `java_get_diagnostics` 检查错误和警告。
```

## 推荐使用提示词

在 Codex 中可以这样使用：

```text
请使用 java-jdtls MCP 检查 Java Language Server 状态，必要时启动它，然后加载当前 Maven 项目。
```

分析代码：

```text
请使用 java-jdtls MCP 搜索 UserService，查看它的定义和所有引用，并总结调用关系。
```

修改后验证：

```text
修改 Java 文件后，请通过 java_open_file 同步最新内容，再调用 java_get_diagnostics 检查编译错误。
```

## 常用 MCP 工具

| 工具 | 说明 |
| --- | --- |
| `java_get_status` | 查看 JDT.LS 当前状态。 |
| `java_start` | 启动 JDT.LS。 |
| `java_restart` | 重启 JDT.LS。 |
| `java_load_maven_project` | 加载 Maven 项目。 |
| `java_search_symbols` | 全局符号搜索。 |
| `java_get_file_symbols` | 获取文件内符号。 |
| `java_get_definition` | 跳转定义。 |
| `java_get_references` | 查找引用。 |
| `java_get_hover` | 获取类型和文档信息。 |
| `java_open_file` | 同步文件内容。 |
| `java_get_diagnostics` | 获取文件诊断。 |
| `java_get_workspace_diagnostics` | 获取工作区诊断。 |
| `read_java_content` | 读取本地文件或外部 Java 内容。 |

## 常见问题

### 1. `codex mcp list` 看不到配置

请确认添加命令是否成功执行：

```powershell
codex mcp add --help
codex mcp list
```

也可以直接检查：

```text
~/.codex/config.toml
```

### 2. MCP Server 启动失败

优先检查：

1. `dist/index.js` 是否存在。
2. `node D:/idea_projects/Java-Sentient-MCP/dist/index.js` 是否能启动。
3. `JDTLS_HOME` 是否指向 Eclipse JDT.LS 根目录。
4. `JDTLS_JAVA_HOME` 是否指向 JDK 根目录。
5. Windows 路径建议使用 `/`，例如 `D:/software/jdk-21`。

### 3. Java 工具调用时提示服务未启动

先让 Codex 调用：

```text
java_get_status
```

如果状态为 `STOPPED`，再调用：

```text
java_start
```

启动后等待状态变为 `READY`，再进行符号搜索、定义跳转或诊断。

### 4. Maven 项目符号搜索结果不完整

大型项目需要先加载和索引。请让 Codex 调用：

```text
java_load_maven_project
```

并等待 JDT.LS 完成初始化和索引。

## 参考资料

- Codex MCP 配置说明: https://developers.openai.com/learn/docs-mcp
- MCP 官方文档: https://modelcontextprotocol.io/
