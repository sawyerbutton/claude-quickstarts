# 代理

一个使用 Claude API 实现 LLM 代理的最小化教育性实现。

> **注意：** 这不是一个 SDK，而是核心概念的参考实现

## 概述与核心组件

本仓库展示了如何使用 Claude API [构建有效的代理](https://www.anthropic.com/engineering/building-effective-agents)。它展示了复杂的 AI 行为如何从一个简单的基础中涌现：LLM 在循环中使用工具。这个实现不是规范性的 - 核心逻辑不到 300 行代码，并且有意缺少生产功能。请随意将这些模式转换到你的语言和生产技术栈中（[Claude Code](https://docs.claude.com/en/docs/agents-and-tools/claude-code/overview) 可以帮助你！）

它包含三个组件：

- `agent.py`：管理 Claude API 交互和工具执行
- `tools/`：工具实现（包括原生工具和 MCP 工具）
- `utils/`：消息历史和 MCP 服务器连接的实用工具

## 使用方法

```python
from agents.agent import Agent
from agents.tools.think import ThinkTool

# 创建一个同时具有本地工具和 MCP 服务器工具的代理
agent = Agent(
    name="MyAgent",
    system="You are a helpful assistant.",
    tools=[ThinkTool()],  # 本地工具
    mcp_servers=[
        {
            "type": "stdio",
            "command": "python",
            "args": ["-m", "mcp_server"],
        },
    ]
)

# 运行代理
response = agent.run("What should I consider when buying a new laptop?")
```

从这个基础上，你可以添加特定领域的工具、优化性能或实现自定义响应处理。我们故意保持不带偏见 - 这个骨架只是让你从基础开始。

## 要求

- Python 3.8+
- Claude API 密钥（设置为 `ANTHROPIC_API_KEY` 环境变量）
- `anthropic` Python 库
- `mcp` Python 库
