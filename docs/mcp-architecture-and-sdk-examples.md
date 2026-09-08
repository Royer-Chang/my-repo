# MCP 架構圖解版與 MCP SDK 實作範例

## 1. MCP 架構圖解版

```text
┌──────────────────────────────────────────────────────────┐
│                        AI Client                         │
│  ChatGPT / Claude / Copilot / 自建 Agent / AI App        │
└──────────────────────────────────────────────────────────┘
                           │
                           │ MCP 協定請求
                           ▼
┌──────────────────────────────────────────────────────────┐
│                      MCP Protocol                        │
│  Tool Discovery / Resources / Prompts / Context Exchange │
└──────────────────────────────────────────────────────────┘
                │                 │                 │
                │                 │                 │
                ▼                 ▼                 ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│    MCP Server A  │  │    MCP Server B  │  │    MCP Server C  │
│   files / docs   │  │   database / SQL │  │   git / issue    │
└──────────────────┘  └──────────────────┘  └──────────────────┘
        │                    │                    │
        ▼                    ▼                    ▼
   Google Drive        PostgreSQL DB            GitHub
```

### 分層理解

- **AI Client**
  - 發起任務
  - 判斷是否需要外部工具
  - 呼叫 MCP Server
  - 整合工具結果

- **MCP Protocol**
  - 定義工具描述與請求格式
  - 支援能力發現
  - 標準化 AI 與外部系統的互動

- **MCP Server**
  - 包裝底層能力
  - 暴露 tools / resources / prompts
  - 將 MCP 請求轉成實際操作

- **底層系統**
  - API
  - DB
  - 文件系統
  - SaaS 平台

---

## 2. MCP 與 API 的關係

- **API** 是底層能力介面
- **MCP** 是 AI/Agent 更友善的標準接入層

簡單說：
- API 解決「服務怎麼被呼叫」
- MCP 解決「AI 怎麼標準化使用工具與上下文」

---

## 3. 真正的 MCP SDK 實作版

> 以下示範的是 MCP SDK 的典型寫法：  
> - Node.js 使用 `@modelcontextprotocol/sdk`
> - Python 使用官方 Python SDK

---

# 4. Node.js MCP Server 實作

## 4.1 安裝

```bash
npm install @modelcontextprotocol/sdk zod
```

## 4.2 最小可運行範例

```typescript name=node-mcp-server.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "demo-mcp-server",
  version: "1.0.0",
});

// Tool 1: 取得目前時間
server.tool(
  "get_current_time",
  "Get the current UTC time",
  {},
  async () => {
    return {
      content: [
        {
          type: "text",
          text: new Date().toISOString(),
        },
      ],
    };
  }
);

// Tool 2: 查詢文件
server.tool(
  "search_docs",
  "Search internal documentation by keyword",
  {
    query: z.string().describe("Search keyword"),
  },
  async ({ query }) => {
    const results = [
      { title: "MCP Intro", snippet: `Matched keyword: ${query}` },
      { title: "API Guide", snippet: "Related documentation result" },
    ];

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(results, null, 2),
        },
      ],
    };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

## 4.3 說明

這個範例包含：
- `McpServer`：建立 MCP server
- `server.tool(...)`：註冊工具
- `StdioServerTransport`：使用 stdio 傳輸，適合本地 AI client 或桌面整合
- 回傳格式使用 `content: [{ type: "text", text: "..." }]`

---

## 4.4 Node.js 串接 PostgreSQL 的工具範例

```typescript name=node-mcp-postgres.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { Pool } from "pg";
import { z } from "zod";

const pool = new Pool({
  host: "localhost",
  port: 5432,
  user: "postgres",
  password: "password",
  database: "demo",
});

const server = new McpServer({
  name: "postgres-mcp-server",
  version: "1.0.0",
});

server.tool(
  "get_user_stats",
  "Get user stats from PostgreSQL",
  {
    userId: z.number().describe("User ID"),
  },
  async ({ userId }) => {
    const result = await pool.query(
      "SELECT id, name, last_login FROM users WHERE id = $1",
      [userId]
    );

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(result.rows, null, 2),
        },
      ],
    };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

# 5. Python MCP Server 實作

## 5.1 安裝

```bash
pip install mcp
```

> 若你的環境使用不同套件名稱或版本，請以官方 Python SDK 文件為準。[[2]](https://py.sdk.modelcontextprotocol.io/get-started/)[[3]](https://modelcontextprotocol.io/docs/2026-07-28/sdk)

## 5.2 最小可運行範例

```python name=python-mcp-server.py
from mcp.server.fastmcp import FastMCP
from datetime import datetime

mcp = FastMCP("demo-mcp-server")

@mcp.tool()
def get_current_time() -> str:
    """Get the current UTC time."""
    return datetime.utcnow().isoformat() + "Z"

@mcp.tool()
def search_docs(query: str) -> str:
    """Search internal documentation by keyword."""
    results = [
        {"title": "MCP Intro", "snippet": f"Matched keyword: {query}"},
        {"title": "API Guide", "snippet": "Related documentation result"},
    ]
    return str(results)

if __name__ == "__main__":
    mcp.run()
```

## 5.3 說明

這個範例包含：
- `FastMCP(...)`：快速建立 MCP server
- `@mcp.tool()`：註冊工具
- 直接用 Python 函式表示 tool handler
- `mcp.run()`：啟動服務

---

## 5.4 Python 串接 SQLite 的工具範例

```python name=python-mcp-sqlite.py
from mcp.server.fastmcp import FastMCP
import sqlite3

mcp = FastMCP("sqlite-mcp-server")

@mcp.tool()
def find_todos(keyword: str) -> str:
    """Find todos by keyword."""
    conn = sqlite3.connect("app.db")
    cursor = conn.cursor()

    cursor.execute(
        "SELECT id, title, status FROM todos WHERE title LIKE ?",
        (f"%{keyword}%",)
    )
    rows = cursor.fetchall()
    conn.close()

    results = [
        {"id": row[0], "title": row[1], "status": row[2]}
        for row in rows
    ]
    return str(results)

if __name__ == "__main__":
    mcp.run()
```

---

# 6. 實際應用場景

## 場景 1：企業知識庫問答
- AI Client 問：「這份產品規格最新版在哪裡？」
- MCP Server 連 Google Drive / Notion / SharePoint
- 回傳文件摘要與位置

## 場景 2：資料庫查詢助理
- AI Client 問：「上週活躍用戶數是多少？」
- MCP Server 透過 PostgreSQL tool 執行 SQL
- 回傳查詢結果

## 場景 3：程式碼庫助理
- AI Client 問：「登入流程在哪個檔案？」
- MCP Server 連 GitHub / Git service
- 回傳檔案、PR、issue、commit 上下文

## 場景 4：客服與工單自動化
- AI Client 問：「幫我查這個客戶最近工單並建立追蹤 issue」
- MCP Server 連 CRM 與 GitHub
- 自動完成跨系統任務

## 場景 5：個人工作助理
- AI Client 整合 calendar、notes、drive、task system
- 生成摘要、待辦、會議紀錄

---

# 7. 設計重點與注意事項

- MCP 不是取代 API，而是包在 API 上方的 AI 標準介面
- Tool 的名稱與 description 要清楚，否則模型不容易選對工具
- 輸出要結構化，避免只回傳過於自由格式的文字
- 權限、驗證、審計仍要另外設計
- 若工具太多，會增加 agent 選擇成本

---

# 8. 總結

**MCP 是為 AI/Agent 設計的標準化工具與上下文接入協定；MCP Server 將底層 API、DB、文件系統或 SaaS 能力包裝成一致介面，讓 AI 更容易、安全且可擴充地使用外部系統。**

---

# 9. 參考資料

- [Using MCP servers with the GitHub Copilot SDK](https://docs.github.com/en/copilot/how-tos/copilot-sdk/features/mcp)
- [MCP server debugging guide](https://docs.github.com/en/copilot/how-tos/copilot-sdk/troubleshooting/mcp-debugging)
- [@modelcontextprotocol/sdk - npm](https://www.npmjs.com/package/@modelcontextprotocol/sdk)
- [Get started - MCP Python SDK](https://py.sdk.modelcontextprotocol.io/get-started/)
- [SDKs - Model Context Protocol](https://modelcontextprotocol.io/docs/2026-07-28/sdk)
