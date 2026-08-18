# PrivateChef - AI 私人厨师助手

<div align="center">

基于 LangChain + LangGraph 的 AI 私厨助手，支持图片识别食材、智能食谱检索与推荐。

[![Python](https://img.shields.io/badge/Python-3.13+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.141+-green.svg)](https://fastapi.tiangolo.com/)
[![LangChain](https://img.shields.io/badge/LangChain-0.3+-orange.svg)](https://python.langchain.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

## 📖 项目简介

**PrivateChef（私厨）** 是一款基于大语言模型的 AI 私人厨师助手。用户可以上传食材照片或输入食材清单，系统会自动识别食材、搜索合适食谱，并从营养价值和制作难度两个维度进行评分排序，最终输出一份结构清晰的烹饪建议报告。

### 核心功能

- 📸 **图片识别**：上传食材照片，AI 自动识别食材种类与新鲜度
- 🔍 **智能食谱搜索**：调用 Tavily 搜索引擎查找可行菜谱
- ⭐ **多维度评分**：从营养价值与制作难度两个维度量化打分
- 💬 **流式对话**：基于 LangGraph 的流式响应，实时输出内容
- 🧠 **会话记忆**：使用 SQLite 存储对话历史，支持多会话管理
- ☁️ **云端存储**：集成阿里云 OSS，支持图片上传与访问
- 🖥️ **Web 界面**：内置 Next.js 前端，提供友好的交互体验

## 🏗️ 技术栈

| 层级 | 技术选型 |
|------|----------|
| 后端框架 | FastAPI |
| AI 框架 | LangChain 0.3 + LangGraph |
| 大模型 | 通义千问 Qwen3-Omni-Flash（阿里云 DashScope） |
| Web 搜索 | Tavily |
| 会话存储 | SQLite（LangGraph Checkpoint） |
| 对象存储 | 阿里云 OSS |
| 前端 | Next.js（静态资源） |
| 包管理 | uv |

## 📂 项目结构

```
PrivateChef/
├── app/
│   ├── main.py                 # FastAPI 应用入口
│   ├── agents/
│   │   └── personal_chief.py   # AI Agent 核心逻辑
│   ├── api/
│   │   └── v1/
│   │       ├── chat.py         # 对话 API（流式/历史/清空）
│   │       └── oss.py          # OSS 签名上传 API
│   ├── common/
│   │   └── logger.py           # 日志配置
│   ├── models/
│   │   └── schemas.py          # Pydantic 数据模型
│   └── static/                 # 前端静态资源
├── db/
│   └── personal_chief.db       # SQLite 会话数据库
├── .env                        # 环境变量配置
├── pyproject.toml              # 项目依赖声明
├── langgraph.json              # LangGraph 配置
└── README.md
```

## 🚀 快速开始

### 环境要求

- Python 3.13+
- uv 包管理器（推荐）或 pip

### 安装依赖

```bash
# 使用 uv（推荐）
uv sync

# 或使用 pip
pip install -e ".[dev]"
```

### 配置环境变量

复制 `.env` 文件并填入你的 API 密钥：

```bash
# 大模型配置（阿里云 DashScope）
DASHSCOPE_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
DASHSCOPE_API_KEY=your_api_key_here

# Web 搜索（Tavily）
TAVILY_API_KEY=your_tavily_key_here

# 阿里云 OSS
OSS_ACCESS_KEY_ID=your_oss_ak
OSS_ACCESS_KEY_SECRET=your_oss_sk
OSS_BUCKET=your_bucket_name
```

### 启动服务

```bash
# 开发模式启动
python -m app.main

# 或使用 uvicorn
uvicorn app.main:app --host 127.0.0.1 --port 8001 --reload
```

服务启动后访问：`http://127.0.0.1:8001`

## 📡 API 接口

### 流式对话

```http
POST /api/v1/chat/stream
Content-Type: application/json

{
  "message": "用这些食材能做些什么？",
  "image_url": "https://example.com/food.jpg",  // 可选
  "thread_id": "unique-session-id"
}
```

**响应**：`text/event-stream` 流式返回 AI 回复内容

### 获取会话历史

```http
GET /api/v1/chat/messages?thread_id=xxx
```

**响应**：
```json
{
  "messages": [
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."}
  ]
}
```

### 清空会话

```http
DELETE /api/v1/chat/messages?thread_id=xxx
```

### OSS 上传签名

```http
GET /api/v1/oss/presign?filename=food.jpg
```

**响应**：
```json
{
  "uploadUrl": "https://...",
  "contentType": "image/jpeg",
  "accessUrl": "https://bucket.oss-cn-beijing.aliyuncs.com/food.jpg"
}
```

## 🤖 Agent 工作流程

```
用户输入（图片/文本）
    │
    ▼
┌─────────────────┐
│  食材识别与评估  │  ← AI 分析照片，生成"可用食材清单"
└────────┬────────┘
         ▼
┌─────────────────┐
│  智能食谱检索    │  ← Tavily 搜索相关菜谱
└────────┬────────┘
         ▼
┌─────────────────┐
│  多维度评分排序  │  ← 营养价值 + 制作难度 综合打分
└────────┬────────┘
         ▼
┌─────────────────┐
│  结构化方案输出  │  ← 推荐报告（含参考图片）
└─────────────────┘
```

## 📝 依赖说明

主要依赖：

| 包 | 用途 |
|----|------|
| `langchain` | AI 应用框架 |
| `langgraph` | 状态机与对话管理 |
| `langchain-openai` | 兼容 OpenAI 格式的模型调用 |
| `langchain-tavily` | Web 搜索工具 |
| `fastapi` | Web 服务框架 |
| `alibabacloud-oss-v2` | 阿里云对象存储 SDK |
| `python-dotenv` | 环境变量加载 |

完整依赖列表见 [pyproject.toml](pyproject.toml)。

## 🔧 开发指南

### 添加新的 AI 工具

在 [app/agents/personal_chief.py](app/agents/personal_chief.py) 中定义新工具，并将其加入 `agent` 的 `tools` 参数即可。

### 修改系统提示词

编辑 [app/agents/personal_chief.py](app/agents/personal_chief.py) 中的 `system_prompt` 变量。

### 切换大模型

修改 [app/agents/personal_chief.py](app/agents/personal_chief.py) 中的 `model` 参数，支持所有兼容 OpenAI 格式的模型。

## 📄 License

[MIT License](LICENSE)

## 🙏 致谢

- [LangChain](https://python.langchain.com/)
- [LangGraph](https://langchain-ai.github.io/langgraph/)
- [FastAPI](https://fastapi.tiangolo.com/)
- [通义千问](https://tongyi.aliyun.com/)
- [Tavily](https://tavily.com/)
