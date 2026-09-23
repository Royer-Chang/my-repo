# Claude Code 專案初始化模板

這份文件可以直接複製到專案中，作為一份可保存的 Markdown 說明檔案，方便你後續建立 Claude Code 專案結構與 AI 協作規範。

---

## 1. 目錄結構

```text
your-project/
├─ CLAUDE.md
├─ CLAUDE.local.md
├─ AGENTS.md
├─ .mcp.json
├─ .gitignore
├─ .pre-commit-config.yaml
├─ pyproject.toml
├─ requirements.txt
├─ .env.example
├─ README.md
├─ .claude/
│  ├─ settings.json
│  ├─ settings.local.json
│  ├─ rules/
│  │  ├─ code-style.md
│  │  ├─ testing.md
│  │  └─ api-conventions.md
│  ├─ skills/
│  │  └─ deploy/
│  │     ├─ SKILL.md
│  │     └─ deploy-config.md
│  ├─ agents/
│  │  ├─ code-reviewer.md
│  │  └─ security-auditor.md
│  └─ hooks/
│     └─ validate-bash.sh
├─ app/
│  ├─ __init__.py
│  ├─ main.py
│  ├─ api/
│  │  ├─ __init__.py
│  │  ├─ deps.py
│  │  └─ routes/
│  │     └─ v1/
│  │        ├─ __init__.py
│  │        └─ users.py
│  ├─ core/
│  │  ├─ __init__.py
│  │  ├─ config.py
│  │  └─ security.py
│  ├─ db/
│  │  ├─ __init__.py
│  │  └─ session.py
│  ├─ models/
│  │  ├─ __init__.py
│  │  └─ user.py
│  ├─ schemas/
│  │  ├─ __init__.py
│  │  └─ user.py
│  ├─ services/
│  │  ├─ __init__.py
│  │  └─ user_service.py
│  └─ tests/
│     ├─ __init__.py
│     ├─ test_health.py
│     └─ test_users.py
└─ worktrees/
```

---

## 2. 檔案用途說明

### CLAUDE.md
專案總規範，Claude 啟動時優先讀取，用來告訴 AI 這個專案的技術棧、開發方法、命令與安全規範。

### CLAUDE.local.md
本機開發者的覆寫設定，適合放個人化設定，不要放正式敏感資訊。

### AGENTS.md
給 AI Agent 使用的操作指南，說明「怎麼做、不能做什麼、如何驗證」。

### .mcp.json
如果專案要串接 GitHub / Jira / PostgreSQL / Filesystem 等外部工具，可以統一放在這裡。

### .claude/
這個資料夾放 Claude 專屬設定與規則，包括：
- settings.json
- rules/
- skills/
- agents/
- hooks/

---

## 3. 內容模板

### 3.1 CLAUDE.md

```md
# FastAPI Project Overview

This project is a Python FastAPI application designed for maintainable, testable, and safe backend development.

## Stack
- Python: 3.11+
- Framework: FastAPI
- ASGI server: uvicorn
- ORM: SQLAlchemy 2.x
- Database: PostgreSQL
- Validation: Pydantic v2
- Testing: pytest
- Linting: ruff
- Type checking: mypy

## Common Commands
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
pytest
pytest -q
ruff check .
mypy app
```

## Project Goals
- Keep API endpoints small and explicit
- Prefer typed models and DTOs
- Validate input early and consistently
- Keep business logic in services
- Treat persistence and API concerns separately

## Coding Conventions
- Use async functions for I/O operations
- Keep route handlers thin
- Validate request bodies with Pydantic schemas
- Keep secrets in environment variables
- Avoid hidden side effects in services
- Write tests for new endpoints and bug fixes

## API Conventions
- Use RESTful resource paths
- Return consistent JSON response structures
- Keep HTTP status codes explicit
- Prefer 400/404/409/422 for validation and domain errors

## Safety Rules
- Never commit secrets
- Do not run destructive DB commands without confirmation
- Validate each change with relevant tests
- Prefer minimal, reviewable edits

## Local Overrides
- `CLAUDE.local.md` can be used for local-only developer settings.
```

---

### 3.2 CLAUDE.local.md

```md
# Local Overrides

This file contains local-only settings and personal developer preferences.
Do not commit secrets or machine-specific values.

## Local Setup
- Use `.env.local` for local overrides
- Start database with Docker Compose if needed
- Prefer `uvicorn app.main:app --reload --host 0.0.0.0 --port 8000`

## Developer Notes
- Keep local test data isolated
- Use a local DB connection string from `.env.local`
- Do not hardcode tokens or credentials into repo files
```

---

### 3.3 AGENTS.md

```md
# Agent Operating Guide

Agents should follow the rules in `CLAUDE.md` and local overrides in `CLAUDE.local.md`.

## Required Behavior
- Read project instructions before editing
- Prefer narrow, domain-specific changes
- Keep endpoints and models consistent with project conventions
- Avoid broad refactors without clear need
- Validate the changed behavior with relevant tests

## Safety Expectations
- Never modify secrets or production credentials
- Ask before executing destructive operations
- Avoid risky DB or migration commands without explicit confirmation
- Keep logs concise and relevant

## Preferred Workflow
1. Read relevant files
2. Identify the root cause or task requirement
3. Make the smallest correct fix
4. Validate with unit or integration tests
5. Summarize assumptions and results clearly
```

---

### 3.4 .mcp.json

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."]
    }
  }
}
```

---

### 3.5 .claude/settings.json

```json
{
  "model": "claude-sonnet-4",
  "permissions": {
    "allow": [
      "read",
      "write",
      "run_in_shell",
      "network_access"
    ],
    "deny": [
      "delete_production_data",
      "modify_git_history",
      "write_sensitive_files"
    ]
  },
  "env": {
    "PYTHON_ENV": "development"
  },
  "hooks": {
    "pre_command": ".claude/hooks/validate-bash.sh"
  },
  "sandbox": {
    "enabled": true,
    "allowed_paths": [
      ".",
      "app",
      "tests",
      "scripts"
    ]
  }
}
```

---

### 3.6 .claude/rules/code-style.md

```md
# Python Code Style

## General Rules
- Keep modules focused and cohesive
- Prefer explicit dependencies over hidden global state
- Use type hints everywhere practical
- Prefer small functions and pure helpers
- Avoid overly clever shortcuts

## FastAPI Conventions
- Keep route functions thin
- Put business logic in services
- Use Pydantic schemas for request/response validation
- Return explicit status codes
- Avoid mixing DB logic directly in route handlers

## Data Layer
- Use repository/service boundaries
- Keep session management centralized
- Prefer explicit transaction scopes
```

---

### 3.7 .claude/rules/testing.md

```md
# Testing Rules

## Required
- Write tests for all new endpoints and services
- Cover both success and failure flows
- Validate response structure and status codes
- Keep tests fast and deterministic

## Test Layout
- Put tests under `app/tests/`
- Prefer clear test naming:
  - `test_create_user_returns_201`
  - `test_get_user_returns_404_for_missing_id`

## Recommended
- Unit test service logic
- Integration test API endpoints
- Use fixtures for DB setup and teardown
```

---

### 3.8 .claude/rules/api-conventions.md

```md
# API Conventions

## HTTP Rules
- Use nouns for resource names
- Keep endpoints stable and predictable
- Use consistent error payloads

## Response Shape
```json
{
  "success": true,
  "data": {},
  "error": null
}
```

## Error Shape
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "INVALID_INPUT",
    "message": "Request body is invalid",
    "details": []
  }
}
```

## Validation
- Return 422 for validation issues
- Return 404 for missing resources
- Return 409 for conflicts
```

---

### 3.9 .claude/agents/code-reviewer.md

```md
# Code Reviewer Agent

## Mission
Review code for correctness, clarity, and maintainability.

## Focus Areas
- API design consistency
- Exception handling
- Validation coverage
- Performance concerns
- Test quality
- Maintainability

## Output Format
- Summary of changes
- Risks identified
- Recommended improvements
```

---

### 3.10 .claude/agents/security-auditor.md

```md
# Security Auditor Agent

## Mission
Review backend code for security risks.

## Focus Areas
- Authentication and authorization
- Input validation
- SQL injection / ORM misuse
- Secret management
- CORS and headers
- Logging sensitive data
- Unsafe shell or filesystem access

## Output Format
- Severity
- Vulnerability description
- Affected files
- Suggested remediation
```

---

### 3.11 .claude/hooks/validate-bash.sh

```bash
#!/usr/bin/env bash
set -euo pipefail

cmd="${1:-}"

if [[ -z "$cmd" ]]; then
  exit 0
fi

if echo "$cmd" | grep -Eq '(^|[[:space:]])rm[[:space:]]+-rf[[:space:]]+/?'; then
  echo "[BLOCKED] Dangerous delete command detected: $cmd"
  exit 1
fi

if echo "$cmd" | grep -Eq '(^|[[:space:]])drop[[:space:]]+database'; then
  echo "[BLOCKED] Database drop command detected: $cmd"
  exit 1
fi

if echo "$cmd" | grep -Eq '(^|[[:space:]])sudo[[:space:]]+rm'; then
  echo "[BLOCKED] Risky sudo delete command detected: $cmd"
  exit 1
fi

echo "[OK] Command passed validation"
```

---

## 4. FastAPI 最小骨架範例

### app/main.py

```python
from fastapi import FastAPI

app = FastAPI(title="My FastAPI App", version="0.1.0")


@app.get("/health")
async def health_check() -> dict[str, str]:
    return {"status": "ok"}
```

---

### app/core/config.py

```python
from functools import lru_cache
from pydantic import Field
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    app_name: str = "my-fastapi-app"
    app_env: str = "development"
    debug: bool = True
    database_url: str = Field(default="postgresql+psycopg://postgres:postgres@localhost:5432/app_db")
    secret_key: str = "change-me"
    access_token_expire_minutes: int = 30

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )


@lru_cache
def get_settings() -> Settings:
    return Settings()
```

---

### app/api/routes/v1/users.py

```python
from fastapi import APIRouter, Depends, status
from sqlalchemy.orm import Session

from app.db.session import get_db
from app.schemas.user import UserCreate, UserRead
from app.services.user_service import create_user

router = APIRouter(prefix="/api/v1/users", tags=["users"])


@router.post("", response_model=UserRead, status_code=status.HTTP_201_CREATED)
def create_user_endpoint(
    user_in: UserCreate,
    db: Session = Depends(get_db),
) -> UserRead:
    user = create_user(db, user_in)
    return UserRead.model_validate(user)
```

---

### app/schemas/user.py

```python
from pydantic import BaseModel, ConfigDict, EmailStr


class UserBase(BaseModel):
    email: EmailStr
    full_name: str


class UserCreate(UserBase):
    pass


class UserRead(UserBase):
    id: int

    model_config = ConfigDict(from_attributes=True)
```

---

### app/services/user_service.py

```python
from sqlalchemy.orm import Session

from app.models.user import User
from app.schemas.user import UserCreate


def create_user(db: Session, user_in: UserCreate) -> User:
    user = User(email=user_in.email, full_name=user_in.full_name)
    db.add(user)
    db.commit()
    db.refresh(user)
    return user
```

---

### app/tests/test_health.py

```python
from fastapi.testclient import TestClient

from app.main import app

client = TestClient(app)


def test_health_check() -> None:
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}
```

---

## 5. 一鍵建立指令

如果你想直接產生這份目錄結構，可以使用以下命令：

```bash
mkdir -p app/api/routes/v1 app/core app/db app/models app/schemas app/services app/tests .claude/rules .claude/skills/deploy .claude/agents .claude/hooks worktrees

# 然後將上述各檔案內容依序寫入
```

---

## 6. 建議使用方式

1. 先建立 `CLAUDE.md`
2. 再加入 `.claude/rules/`
3. 接著補上 `app/` 基本架構
4. 最後加上 `hooks` 與 `skills`

這樣 AI 協作的開發流程會更穩定、更安全，也更容易保留一致的程式風格。

---

## 7. 優點總結

- 可直接用於 FastAPI 專案
- AI 較容易理解專案結構與規範
- 可將安全檢查、部署與程式風格統一
- 適合團隊協作與長期維護

如果你要，我下一步可以直接幫你整理成：
- 版本 A：最小可執行版
- 版本 B：生產級版（含 JWT + SQLAlchemy + Docker + Alembic）
- 版本 C：公司內部規範版（含 CI/CD、PR 模板、lint/format 規則）