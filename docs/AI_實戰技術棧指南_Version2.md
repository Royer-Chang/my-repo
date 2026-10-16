# AI 工程師實戰技術棧指南

> 版本：2026-10-16  
> 目的：整理目前 AI 工程領域中最具實戰價值的工具與架構，涵蓋模型、Agent、工作流、數據與 UI，適合工程師快速做 PoC、原型或產品化落地。

---

## 1. 目錄

- [1. 背景與定位](#1-背景與定位)
- [2. AI 開發工具分類總覽](#2-ai-開發工具分類總覽)
- [3. 工程師實戰工具表（含 GitHub）](#3-工程師實戰工具表含-github)
- [4. 可複製的實戰技術棧](#4-可複製的實戰技術棧)
  - [4.1 方案 A：本地私有 MVP](#41-方案-a本地私有-mvp)
  - [4.2 方案 B：團隊 AI 應用](#42-方案-b團隊-ai-應用)
  - [4.3 方案 C：企業級 Agent / 工作流](#43-方案-c企業級-agent--工作流)
- [5. 實戰選型建議](#5-實戰選型建議)
- [6. 推薦組合](#6-推薦組合)
- [7. GitHub 連結彙整](#7-github-連結彙整)

---

## 1. 背景與定位

AI 開發不只是「呼叫 LLM」這麼簡單，真正工程化的 AI 系統，通常需要同時涵蓋以下幾層：

- 模型層：LLM runtime、推理與部署
- Agent 層：工具調用、長流程任務、代理協作
- 工作流層：流程編排、API 串接、自動化任務
- 數據層：RAG、知識庫建置、資料抓取與清洗
- UI 層：聊天介面、內部入口、使用者交互

這份文件的目的，是幫助工程師快速建立一套「可實際落地」的 AI 技術棧，從 PoC 到產品化都能沿用。

---

## 2. AI 開發工具分類總覽

### 2.1 模型（Model）
- Ollama
  - GitHub: https://github.com/ollama/ollama
  - 用途：在本機執行開源模型，適合本地開發、隱私場景與低成本原型。

### 2.2 Agent
- OpenClaw
  - GitHub: https://github.com/openclaw/openclaw
  - 用途：裝置端 AI 助手，適合個人化與私有化場景。

- Hermes Agent
  - GitHub: https://github.com/NousResearch/hermes-agent
  - 用途：自我改進型 Agent，適合研究 Agent 能力與任務進化。

- OpenCode
  - GitHub: https://github.com/anomalyco/opencode
  - 用途：終端機上的開源編碼 Agent，適合 CLI 工作流。

- Agent Skills
  - GitHub: https://github.com/anthropics/skills
  - 用途：將模型技能模組化，讓 AI 更容易完成特定任務。

- Claude Code
  - GitHub: https://github.com/anthropics/claude-code
  - 用途：跨整個 codebase 的 Agentic coding。

- LangChain
  - GitHub: https://github.com/langchain-ai/langchain
  - 用途：構建 Agent、工具調用、記憶與鏈式流程的核心框架。

### 2.3 工作流（Workflow）
- n8n
  - GitHub: https://github.com/n8n-io/n8n
  - 用途：視覺化工作流、自動化與 AI 節點編排。

- Dify
  - GitHub: https://github.com/langgenius/dify
  - 用途：Agent workflow + RAG 一站式平台，適合快速交付 AI 產品。

- Langflow
  - GitHub: https://github.com/langflow-ai/langflow
  - 用途：拖拉式 AI 流程建構器，便於 PoC 和原型驗證。

### 2.4 數據（Data）
- Firecrawl
  - GitHub: https://github.com/firecrawl/firecrawl
  - 用途：將網站內容轉成 LLM 可用資料，適合做 RAG 和知識庫建置。

### 2.5 UI
- Open WebUI
  - GitHub: https://github.com/open-webui/open-webui
  - 用途：可自架的 ChatGPT 風格多模型聊天介面。

---

## 3. 工程師實戰工具表（含 GitHub）

> 依開發流程整理：模型層 → Agent 層 → 工作流層 → 數據層 → UI 層

### 3.1 模型層（Model Runtime）

| 工具 | GitHub | 實戰用途 | 適用場景 |
|---|---|---|---|
| Ollama | [ollama/ollama](https://github.com/ollama/ollama) | 本機執行與管理開源模型 | 本地開發、隱私需求、離線測試 |

### 3.2 Agent 層（Agent / Coding Agent）

| 工具 | GitHub | 實戰用途 | 適用場景 |
|---|---|---|---|
| OpenClaw | [openclaw/openclaw](https://github.com/openclaw/openclaw) | 個人 AI 助手 | 私有化助理、個人知識工作 |
| Hermes Agent | [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | 自我改進型 Agent | Agent 能力研究、自動化任務演進 |
| OpenCode | [anomalyco/opencode](https://github.com/anomalyco/opencode) | 終端機 coding agent | CLI 工作流、快速改碼 |
| Agent Skills | [anthropics/skills](https://github.com/anthropics/skills) | 能力模組化 | 任務模板化、技能擴充 |
| Claude Code | [anthropics/claude-code](https://github.com/anthropics/claude-code) | 跨 codebase 的 Agentic coding | 大型專案、多檔案重構 |
| LangChain | [langchain-ai/langchain](https://github.com/langchain-ai/langchain) | Agent、工具調用、記憶與鏈式流程 | 高可擴充 AI 應用架構 |

### 3.3 工作流層（Workflow / Orchestration）

| 工具 | GitHub | 實戰用途 | 適用場景 |
|---|---|---|---|
| n8n | [n8n-io/n8n](https://github.com/n8n-io/n8n) | 視覺化自動化與 AI 節點編排 | API 串接、營運自動化 |
| Dify | [langgenius/dify](https://github.com/langgenius/dify) | 一站式 Agent workflow + RAG | 快速交付 AI 產品 |
| Langflow | [langflow-ai/langflow](https://github.com/langflow-ai/langflow) | 拖拉式 AI 流程建構 | PoC、低門檻流程設計 |

### 3.4 數據層（Data / RAG）

| 工具 | GitHub | 實戰用途 | 適用場景 |
|---|---|---|---|
| Firecrawl | [firecrawl/firecrawl](https://github.com/firecrawl/firecrawl) | 網頁資料轉為 LLM 可用資料 | 知識庫建置、RAG 前處理 |

### 3.5 UI 層（Chat Interface）

| 工具 | GitHub | 實戰用途 | 適用場景 |
|---|---|---|---|
| Open WebUI | [open-webui/open-webui](https://github.com/open-webui/open-webui) | 自架 ChatGPT 風格聊天界面 | 內部 AI 入口、團隊共享 |

---

## 4. 可複製的實戰技術棧

### 4.1 方案 A：本地私有 MVP
適合：個人開發、內部工具、隱私優先、低成本快速驗證。

#### 技術組成
- Ollama
- Open WebUI
- Qdrant
- FastAPI
- Firecrawl

#### Docker Compose 範例
```yaml
version: "3.9"

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama:/root/.ollama

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    depends_on:
      - ollama

  qdrant:
    image: qdrant/qdrant:latest
    container_name: qdrant
    ports:
      - "6333:6333"
    volumes:
      - qdrant:/qdrant/storage

volumes:
  ollama:
  qdrant:
  
優點
成本低
可本地化部署
適合快速做 RAG 與知識問答原型
4.2 方案 B：團隊 AI 應用
適合：產品原型、多人協作、RAG 應用落地。

技術組成
模型：Ollama 或雲端 LLM
Agent：LangChain
工作流：Dify
向量資料庫：Qdrant
後端：FastAPI
前端：Next.js
觀測：Langfuse
資料抓取：Firecrawl
入口 UI：Open WebUI
專案目錄示例
Text
ai-app/
├─ frontend/
│  └─ next-app/
├─ backend/
│  ├─ app/
│  ├─ agents/
│  ├─ vector/
│  └─ main.py
├─ docker-compose.yml
├─ .env
├─ requirements.txt
└─ README.md
優點
適合做 Demo 與 MVP
容易擴展成見得人的 AI 產品
工程與產品驗收效率高
4.3 方案 C：企業級 Agent / 工作流
適合：多流程自動化、企業內部整合、長流程任務執行。

技術組成
Agent：LangChain
Coding Agent：Claude Code / OpenCode
Workflow：n8n
資料庫：PostgreSQL
向量資料庫：Qdrant
任務隊列：Celery + Redis
API 後端：FastAPI
前端：Next.js
監控：Prometheus + Grafana
遙測：OpenTelemetry
Docker Compose 示意
YAML
version: "3.9"

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: ai_app
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"

  n8n:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - GENERIC_TIMEZONE=Asia/Taipei

  api:
    build: ./backend
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - qdrant
      - redis
優點
可治理
可持續演進
兼顧自動化、工作流與 Agent 能力
5. 實戰選型建議
5.1 如果你要快速上手（1–3 天）
選：

Ollama
Open WebUI
FastAPI
Qdrant
Firecrawl
5.2 如果你要做產品雛形（1–2 週）
選：

LangChain
Dify
Open WebUI
Qdrant
Next.js
5.3 如果你要做企業級系統
選：

LangChain
n8n
PostgreSQL
Qdrant
FastAPI
Redis
Grafana / Prometheus
6. 推薦組合
最推薦的工程師實戰組合：

Text
Ollama + Open WebUI + FastAPI + Qdrant + Firecrawl + LangChain + Next.js
這套組合能同時滿足：

本地模型部署
知識庫 RAG
Agent 能力
前後端完整產品展示
後續逐步升級成企業級系統
7. GitHub 連結彙整
Ollama: https://github.com/ollama/ollama
Open WebUI: https://github.com/open-webui/open-webui
Firecrawl: https://github.com/firecrawl/firecrawl
OpenClaw: https://github.com/openclaw/openclaw
Hermes Agent: https://github.com/NousResearch/hermes-agent
OpenCode: https://github.com/anomalyco/opencode
Agent Skills: https://github.com/anthropics/skills
Claude Code: https://github.com/anthropics/claude-code
LangChain: https://github.com/langchain-ai/langchain
n8n: https://github.com/n8n-io/n8n
Dify: https://github.com/langgenius/dify
Langflow: https://github.com/langflow-ai/langflow
Qdrant: https://github.com/qdrant/qdrant
FastAPI: https://github.com/fastapi/fastapi
Next.js: https://github.com/vercel/next.js
Langfuse: https://github.com/langfuse/langfuse
PostgreSQL: https://github.com/postgres/postgres
8. 結語
AI 工程師真正需要掌握的，不只是「單一模型」或「單一工具」，而是建立完整的工程化技術棧。
從本地 MVP 到企業級落地，核心仍然是：

模型可用
Agent 可調用
工作流可編排
知識可檢索
產品可展示
這份文件提供了一個可直接實作的起點，讓工程師可以依照需求選擇最適合的技術棧。  