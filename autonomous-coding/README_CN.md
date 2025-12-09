# 自主编程代理演示

一个展示使用 Claude Agent SDK 进行长时间运行自主编程的最小化框架。该演示实现了一种双代理模式（初始化代理 + 编程代理），可以在多个会话中构建完整的应用程序。

## 前提条件

**必需：** 安装 Claude Code 和 Claude Agent SDK 的最新版本：

```bash
# 安装 Claude Code CLI（需要最新版本）
npm install -g @anthropic-ai/claude-code

# 安装 Python 依赖项
pip install -r requirements.txt
```

验证你的安装：
```bash
claude --version  # 应该是最新版本
pip show claude-code-sdk  # 检查 SDK 是否已安装
```

**API 密钥：** 设置你的 Anthropic API 密钥：
```bash
export ANTHROPIC_API_KEY='your-api-key-here'
```

## 快速开始

```bash
python autonomous_agent_demo.py --project-dir ./my_project
```

用于有限迭代次数的测试：
```bash
python autonomous_agent_demo.py --project-dir ./my_project --max-iterations 3
```

## 重要的时间预期

> **警告：此演示运行时间很长！**

- **第一次会话（初始化）：** 代理生成包含 200 个测试用例的 `feature_list.json`。这需要几分钟，可能看起来像是卡住了 - 这是正常的。代理正在写出所有功能。

- **后续会话：** 根据复杂程度，每次编程迭代可能需要 **5-15 分钟**。

- **完整应用：** 构建所有 200 个功能通常需要在多个会话中累计 **数小时** 的总运行时间。

**提示：** 提示中的 200 个功能参数是为了全面覆盖而设计的。如果你想要更快的演示，可以修改 `prompts/initializer_prompt.md` 来减少功能数量（例如，20-50 个功能可以更快地演示）。

## 工作原理

### 双代理模式

1. **初始化代理（会话 1）：** 读取 `app_spec.txt`，创建包含 200 个测试用例的 `feature_list.json`，设置项目结构，并初始化 git。

2. **编程代理（会话 2+）：** 从上一个会话中断的地方继续，逐个实现功能，并在 `feature_list.json` 中将它们标记为通过。

### 会话管理

- 每个会话都以全新的上下文窗口运行
- 进度通过 `feature_list.json` 和 git 提交持久化
- 代理在会话之间自动继续（3 秒延迟）
- 按 `Ctrl+C` 暂停；运行相同的命令以恢复

## 安全模型

此演示使用深度防御安全方法（参见 `security.py` 和 `client.py`）：

1. **操作系统级沙箱：** Bash 命令在隔离环境中运行
2. **文件系统限制：** 文件操作仅限于项目目录
3. **Bash 允许列表：** 仅允许特定命令：
   - 文件检查：`ls`、`cat`、`head`、`tail`、`wc`、`grep`
   - Node.js：`npm`、`node`
   - 版本控制：`git`
   - 进程管理：`ps`、`lsof`、`sleep`、`pkill`（仅限开发进程）

不在允许列表中的命令会被安全钩子阻止。

## 项目结构

```
autonomous-coding/
├── autonomous_agent_demo.py  # 主入口点
├── agent.py                  # 代理会话逻辑
├── client.py                 # Claude SDK 客户端配置
├── security.py               # Bash 命令允许列表和验证
├── progress.py               # 进度跟踪实用工具
├── prompts.py                # 提示加载实用工具
├── prompts/
│   ├── app_spec.txt          # 应用程序规范
│   ├── initializer_prompt.md # 第一次会话提示
│   └── coding_prompt.md      # 后续会话提示
└── requirements.txt          # Python 依赖项
```

## 生成的项目结构

运行后，你的项目目录将包含：

```
my_project/
├── feature_list.json         # 测试用例（真实来源）
├── app_spec.txt              # 复制的规范
├── init.sh                   # 环境设置脚本
├── claude-progress.txt       # 会话进度备注
├── .claude_settings.json     # 安全设置
└── [应用程序文件]             # 生成的应用程序代码
```

## 运行生成的应用程序

代理完成（或暂停）后，你可以运行生成的应用程序：

```bash
cd generations/my_project

# 运行代理创建的设置脚本
./init.sh

# 或手动运行（Node.js 应用程序的典型方式）：
npm install
npm run dev
```

应用程序通常在 `http://localhost:3000` 或类似地址可用（查看代理的输出或 `init.sh` 以获取确切的 URL）。

## 命令行选项

| 选项 | 描述 | 默认值 |
|--------|-------------|---------|
| `--project-dir` | 项目目录 | `./autonomous_demo_project` |
| `--max-iterations` | 最大代理迭代次数 | 无限制 |
| `--model` | 使用的 Claude 模型 | `claude-sonnet-4-5-20250929` |

## 自定义

### 更改应用程序

编辑 `prompts/app_spec.txt` 来指定要构建的不同应用程序。

### 调整功能数量

编辑 `prompts/initializer_prompt.md` 并将"200 个功能"的要求更改为较小的数字，以获得更快的演示。

### 修改允许的命令

编辑 `security.py` 以在 `ALLOWED_COMMANDS` 中添加或删除命令。

## 故障排除

**"第一次运行时似乎卡住了"**
这是正常的。初始化代理正在生成 200 个详细的测试用例，这需要相当长的时间。观察 `[Tool: ...]` 输出以确认代理正在工作。

**"命令被安全钩子阻止"**
代理尝试运行不在允许列表中的命令。这是安全系统按预期工作。如果需要，请在 `security.py` 中将该命令添加到 `ALLOWED_COMMANDS`。

**"API 密钥未设置"**
确保在你的 shell 环境中导出了 `ANTHROPIC_API_KEY`。

## 许可证

Anthropic 内部使用。
