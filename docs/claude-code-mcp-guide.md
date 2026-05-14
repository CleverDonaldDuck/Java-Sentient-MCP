# Claude Code 接入 Java Sentient MCP 指南

本文档介绍如何在 Claude Code 中接入本项目提供的 Java MCP Server，让 Claude Code 可以通过 Eclipse JDT.LS 获取 Java 项目的 IDE 级代码智能能力。

## 适用场景

接入后，Claude Code 可以使用本 MCP Server 完成以下任务：

- 启动和管理 Java Language Server。
- 加载 Maven 项目并建立符号索引。
- 搜索 Java 类、方法、字段等符号。
- 跳转定义、查找引用、读取 Hover 信息。
- 获取 Java 文件或工作区的编译错误和警告。
- 读取外部依赖或 JDK 类内容。

## 前置条件

请先准备以下环境：

| 依赖 | 要求 |
| --- | --- |
| Claude Code | 已安装并可在命令行执行 `claude` |
| Node.js | 18.0.0 或更高版本 |
| JDK | Java 17 或更高版本，推荐 Java 21+ |
| Eclipse JDT.LS | 已下载并解压到本地目录 |
| 本项目 | 已执行 `npm install` 和 `npm run build` |

在本项目根目录执行构建：

```powershell
cd D:/idea_projects/Java-Sentient-MCP
npm install
npm run build
```

构建完成后，MCP Server 入口文件应位于：

```text
D:/idea_projects/Java-Sentient-MCP/dist/index.js
```

## 推荐接入方式：使用 claude mcp add

本项目是本地 stdio MCP Server，Claude Code 推荐使用 `claude mcp add` 接入。

PowerShell 示例：

```powershell
claude mcp add --transport stdio --scope user `
  --env JDTLS_HOME="D:/software/jdt-language-server-latest" `
  --env JDTLS_JAVA_HOME="D:/software/java/jdk-21" `
  --env JDTLS_JAVA_RUNTIMES="[{`"name`":`"JavaSE-17`",`"path`":`"D:/software/jdk-17`"}]" `
  java-jdtls `
  -- node D:/idea_projects/Java-Sentient-MCP/dist/index.js
```

macOS / Linux 示例：

```bash
claude mcp add --transport stdio --scope user \
  --env JDTLS_HOME="/opt/jdtls" \
  --env JDTLS_JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home" \
  --env JDTLS_JAVA_RUNTIMES='[{"name":"JavaSE-17","path":"/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home"}]' \
  java-jdtls \
  -- node /absolute/path/to/Java-Sentient-MCP/dist/index.js
```

命令说明：

| 参数 | 说明 |
| --- | --- |
| `--transport stdio` | 表示 Claude Code 通过本地进程 stdio 与 MCP Server 通信。 |
| `--scope user` | 当前用户所有项目可用。也可以改成 `local` 或 `project`。 |
| `java-jdtls` | MCP Server 名称，可自行修改，但建议保持简短明确。 |
| `-- node .../dist/index.js` | `--` 后面是真正启动 MCP Server 的命令。 |

## Scope 选择建议

Claude Code 支持多个 MCP 配置范围：

| Scope | 适用场景 | 配置位置 |
| --- | --- | --- |
| `local` | 只给当前项目当前用户使用 | `~/.claude.json` |
| `project` | 团队共享，适合提交到项目仓库 | 项目根目录 `.mcp.json` |
| `user` | 当前用户所有项目可用 | `~/.claude.json` |

如果你只是自己使用，推荐 `--scope user`。

如果团队都要使用这套 Java MCP 工具，推荐 `--scope project`，但不要把带有个人本地绝对路径或敏感信息的配置直接提交。

## 项目级 .mcp.json 配置

也可以在 Java 项目根目录创建 `.mcp.json`：

```json
{
  "mcpServers": {
    "java-jdtls": {
      "type": "stdio",
      "command": "node",
      "args": [
        "D:/idea_projects/Java-Sentient-MCP/dist/index.js"
      ],
      "env": {
        "JDTLS_HOME": "D:/software/jdt-language-server-latest",
        "JDTLS_JAVA_HOME": "D:/software/java/jdk-21",
        "JDTLS_JAVA_RUNTIMES": "[{\"name\":\"JavaSE-17\",\"path\":\"D:/software/jdk-17\"}]",
        "JDTLS_MAVEN_USER_SETTINGS": "D:/software/apache-maven/conf/settings.xml",
        "JDTLS_MAVEN_OFFLINE": "false"
      }
    }
  }
}
```

如果要让团队共享配置，可以使用环境变量占位：

```json
{
  "mcpServers": {
    "java-jdtls": {
      "type": "stdio",
      "command": "node",
      "args": [
        "${JAVA_SENTIENT_MCP_HOME}/dist/index.js"
      ],
      "env": {
        "JDTLS_HOME": "${JDTLS_HOME}",
        "JDTLS_JAVA_HOME": "${JDTLS_JAVA_HOME}",
        "JDTLS_JAVA_RUNTIMES": "${JDTLS_JAVA_RUNTIMES:-[]}"
      }
    }
  }
}
```

使用这种方式时，每个开发者需要在自己的系统环境变量中配置：

```powershell
$env:JAVA_SENTIENT_MCP_HOME="D:/idea_projects/Java-Sentient-MCP"
$env:JDTLS_HOME="D:/software/jdt-language-server-latest"
$env:JDTLS_JAVA_HOME="D:/software/java/jdk-21"
$env:JDTLS_JAVA_RUNTIMES='[{"name":"JavaSE-17","path":"D:/software/jdk-17"}]'
```

## 使用 npx 启动

如果你希望使用 npm 包方式，而不是本地源码路径，也可以使用：

```powershell
claude mcp add --transport stdio --scope user `
  --env JDTLS_HOME="D:/software/jdt-language-server-latest" `
  --env JDTLS_JAVA_HOME="D:/software/java/jdk-21" `
  java-jdtls `
  -- npx -y @cleverdonaldduck/java-jdtls-mcp-server@latest
```

本地开发和调试本项目时，仍推荐使用 `node dist/index.js`，这样能确保 Claude Code 使用的是你当前工作区构建出的版本。

## 验证接入

查看已配置的 MCP Server：

```powershell
claude mcp list
```

查看某个 MCP Server 详情：

```powershell
claude mcp get java-jdtls
```

进入 Claude Code 后查看 MCP 状态：

```text
/mcp
```

如果显示 `java-jdtls` 已连接，并能看到 Java 相关工具，说明接入成功。

## 推荐使用提示词

在 Claude Code 中可以这样要求 Agent 使用本 MCP：

```text
请使用 java-jdtls MCP 检查 Java Language Server 状态，必要时启动它，然后加载当前 Maven 项目并分析 UserService 的引用关系。
```

修改 Java 代码后，可以这样要求：

```text
修改完成后，请通过 java_open_file 同步文件，并使用 java_get_diagnostics 检查是否存在编译错误。
```

## 推荐项目规则

可以在 Java 项目中添加 `CLAUDE.md`，让 Claude Code 更稳定地使用这个 MCP：

```md
# Claude Code Java MCP Rules

处理 Java 项目时，优先使用 `java-jdtls` MCP。

工作流要求：

1. 先调用 `java_get_status` 检查 Java Language Server 状态。
2. 如未启动，调用 `java_start`，并等待状态变为 `READY`。
3. 对 Maven 项目，优先调用 `java_load_maven_project` 建立索引。
4. 查找类、方法、字段时，优先使用 `java_search_symbols`。
5. 修改 Java 文件后，必须调用 `java_open_file` 同步内容。
6. 同步后调用 `java_get_diagnostics` 检查错误和警告。
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

### 1. `claude` 命令不存在

说明 Claude Code 没有安装，或者命令没有加入 `PATH`。请先确认：

```powershell
claude --version
```

### 2. MCP Server 显示未连接

可按顺序检查：

1. `dist/index.js` 是否存在。
2. `node D:/idea_projects/Java-Sentient-MCP/dist/index.js` 是否可以启动。
3. `JDTLS_HOME` 是否指向 Eclipse JDT.LS 根目录。
4. Claude Code 是否需要重启。
5. 在 Claude Code 内执行 `/mcp` 查看详细状态。

### 3. JDT.LS 启动失败

重点检查：

1. `JDTLS_HOME` 下是否存在 `plugins/` 目录。
2. JDK 版本是否为 17+。
3. `JDTLS_JAVA_HOME` 是否包含 `bin/java` 或 `bin/java.exe`。
4. Windows 路径建议使用 `/`，例如 `D:/software/jdk-21`。

### 4. Java 工具可见但调用时报初始化中

JDT.LS 启动和索引大型项目需要时间。先调用：

```text
java_get_status
```

等待状态变为 `READY` 后再继续使用符号搜索、定义跳转或诊断工具。

## 参考资料

- Claude Code MCP 文档: https://code.claude.com/docs/en/mcp
- MCP 官方文档: https://modelcontextprotocol.io/
