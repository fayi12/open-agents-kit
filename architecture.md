# Open Agents Kit（崽崽云）技术架构

> 文档版本：v0.1 | 制定时间：2026-03-25 | 负责崽：coder崽
> 项目定位：轻量级多 Agent 协作系统，适合个人开发者和小企业

---

## 1. 系统架构总览

```
┌─────────────────────────────────────────────────────────────┐
│                      Open Agents Kit                        │
│                    （崽崽云 · 5崽协作）                      │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐                                             │
│  │  用户     │  ←── 消息通道：微信 / Telegram / Discord   │
│  └────┬─────┘                                             │
│       │                                                    │
│  ┌────▼──────────────────────────────┐                    │
│  │      Gateway（崽崽调度中心）         │                    │
│  │  ┌─────────────────────────────┐  │                    │
│  │  │  消息路由 / 权限控制 / 通道管理 │  │                    │
│  │  └─────────────────────────────┘  │                    │
│  └──────────┬───────────────────────┘                    │
│             │                                                │
│  ┌──────────▼───────────────────────────────────────────┐   │
│  │              崽崽协作层（5崽）                        │   │
│  │                                                      │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │
│  │  │Researcher│ │ Coder   │ │Scheduler│ │Organizer│ │Silijian │ │
│  │  │ (研究崽) │ │ (码农崽) │ │(管家崽) │ │(整理崽) │ │(事儿妈)  │ │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ │
│  └───────│───────────│───────────│───────────│───────────│─────┘   │
│          │           │           │           │           │         │
│  ┌───────▼───────────▼───────────▼───────────▼───────────▼─────┐ │
│  │              共享数据层                                      │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │ │
│  │  │  任务队列  │  │  共享存储  │  │  配置中心  │  │  日志中心  │     │ │
│  │  │(SQLite)  │  │(Workspace)│  │ (JSON)   │  │ (JSONL)  │     │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │ │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**架构原则**
- 每个崽是独立进程，通过共享 SQLite 任务队列协作
- Gateway 负责消息通道接入（类 OpenClaw），不处理具体业务
- 崽崽之间无直接依赖，可独立扩缩容
- 所有数据默认本地存储，支持 S3/OSS 备份

---

## 2. 目录结构

```
open-agents-kit/
├── README.md
├── architecture.md          ← 本文档
├── deploy_guide.md          ← 部署指南（scheduler崽负责）
├── workflow.md              ← 协作流程（organizer崽负责）
│
├── gateway/                 # 崽崽调度中心（消息网关）
│   ├── index.js             # Gateway 主入口
│   ├── router.js            # 消息路由：根据 intent 分发到对应崽
│   ├── channel/             # 通道适配器
│   │   ├── telegram.js
│   │   ├── discord.js
│   │   ├── wechat.js        # 企业微信 / 个人微信（可选）
│   │   └── webchat.js       # Web UI
│   ├── middleware/
│   │   ├── auth.js          # 身份认证 / 权限控制
│   │   ├── rateLimit.js     # 限流保护
│   │   └── sanitize.js      # 输入清洗 / 防注入
│   └── package.json
│
├── agents/                  # 崽崽目录（每个崽独立子模块）
│   ├── researcher/          # 研究崽
│   │   ├── agent.py         # 主体逻辑
│   │   ├── skills/
│   │   │   ├── web_search.js
│   │   │   ├── github_api.js
│   │   │   └── report_writer.js
│   │   ├── memory/         #崽崽私有记忆
│   │   ├── SOUL.md
│   │   └── AGENTS.md
│   │
│   ├── coder/               # 码农崽
│   │   ├── agent.py
│   │   ├── skills/
│   │   │   ├── code_review.js
│   │   │   ├── file_writer.js
│   │   │   └── git_ops.js
│   │   ├── memory/
│   │   ├── SOUL.md
│   │   └── AGENTS.md
│   │
│   ├── scheduler/           # 管家崽
│   │   ├── agent.py
│   │   ├── skills/
│   │   │   ├── cron_scheduler.js
│   │   │   ├── task_queue.js
│   │   │   └── notifier.js
│   │   ├── memory/
│   │   ├── SOUL.md
│   │   └── AGENTS.md
│   │
│   ├── organizer/           # 整理崽
│   │   ├── agent.py
│   │   ├── skills/
│   │   │   ├── data_parser.js
│   │   │   ├── formatter.js
│   │   │   └── file_organizer.js
│   │   ├── memory/
│   │   ├── SOUL.md
│   │   └── AGENTS.md
│   │
│   └── silijian/            # 事儿妈崽（质量把关）
│       ├── agent.py
│       ├── skills/
│       │   ├── quality_gate.js
│       │   ├── human_in_loop.js
│       │   └── escalation.js
│       ├── memory/
│       ├── SOUL.md
│       └── AGENTS.md
│
├── shared/                  # 崽崽共享资源
│   ├── task_queue.py        # SQLite 任务队列封装
│   ├── db/
│   │   └── tasks.db         # SQLite 数据库
│   ├── config/
│   │   └── agents.json      # 崽崽配置（技能列表、权限）
│   ├── logs/                # 集中日志
│   │   ├── researcher.log
│   │   ├── scheduler.log
│   │   └── ...
│   ├── artifacts/            #崽崽产出物（报告/代码等）
│   │   ├── reports/
│   │   └── code/
│   └── libs/
│       ├── logger.js
│       ├── db.js            # SQLite 连接池
│       └── ipc.js           # 进程间通信（Unix Socket / HTTP）
│
├── scripts/
│   ├── install.sh           # 一键安装脚本
│   ├── start.sh             # 启动所有崽
│   ├── stop.sh              # 停止所有崽
│   ├── status.sh            # 查看崽崽状态
│   └── backup.sh            # 数据备份
│
├── docker/
│   ├── Dockerfile.all       # All-in-One 单容器
│   ├── Dockerfile.gateway
│   ├── Dockerfile.agent
│   └── docker-compose.yml   # 多容器编排
│
├── .env.example             # 环境变量模板
├── package.json             # Node.js 依赖（gateway + 工具）
└── requirements.txt         # Python 依赖（各崽 Python runtime）
```

---

## 3. 核心脚本

### 3.1 Gateway 入口（gateway/index.js）

```javascript
// 消息入口：接收用户消息 → 解析意图 → 分发到对应崽
// 依赖：ws, express, better-sqlite3, dotenv
// 端口：18789（默认），支持 Telegram Bot / Discord Webhook / HTTP

import { createServer } from 'http';
import { Router } from './router.js';
import { TaskQueue } from '../shared/task_queue.js';

const router = new Router(new TaskQueue('../shared/db/tasks.db'));

const server = createServer(async (req, res) => {
  if (req.method === 'POST' && req.url === '/webhook') {
    const body = [];
    req.on('data', chunk => body.push(chunk));
    req.on('end', async () => {
      const msg = JSON.parse(Buffer.concat(body).toString());
      await router.dispatch(msg);
      res.end('ok');
    });
  }
});

server.listen(18789, () => console.log('🦞 Gateway running on :18789'));
```

### 3.2 崽崽基类（agents/base_agent.py）

```python
# 所有崽崽的父类，封装公共能力
# 依赖：openai, sqlite3, pydantic, croniter, aiohttp

import sqlite3, json, threading, logging
from pathlib import Path
from datetime import datetime

class BaseAgent:
    def __init__(self, name: str, db_path: str = "../shared/db/tasks.db"):
        self.name = name
        self.db = sqlite3.connect(db_path, check_same_thread=False)
        self.logger = logging.getLogger(name)
        self._setup_db()

    def _setup_db(self):
        self.db.execute("""
            CREATE TABLE IF NOT EXISTS task_queue (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                task_type TEXT, source TEXT, payload TEXT,
                status TEXT DEFAULT 'pending',
                created_at TEXT, updated_at TEXT, assigned_to TEXT
            )
        """)
        self.db.commit()

    def enqueue(self, task_type: str, payload: dict, source: str = "gateway"):
        self.db.execute(
            "INSERT INTO task_queue (task_type, source, payload, created_at) VALUES (?, ?, ?, ?)",
            (task_type, source, json.dumps(payload), datetime.now().isoformat())
        )
        self.db.commit()

    def dequeue(self, task_type: str) -> dict | None:
        row = self.db.execute(
            "SELECT * FROM task_queue WHERE task_type=? AND status='pending' ORDER BY created_at ASC LIMIT 1",
            (task_type,)
        ).fetchone()
        if row:
            self.db.execute("UPDATE task_queue SET status='running', assigned_to=? WHERE id=?", (self.name, row[0]))
            self.db.commit()
            return {"id": row[0], "payload": json.loads(row[3])}
        return None

    def complete(self, task_id: int, result: dict):
        self.db.execute(
            "UPDATE task_queue SET status='done', updated_at=?, result=? WHERE id=?",
            (datetime.now().isoformat(), json.dumps(result), task_id)
        )
        self.db.commit()
```

### 3.3 Scheduler 管家崽定时调度（agents/scheduler/agent.py）

```python
# 定时任务调度器，扫描 crontab → 插入任务队列
# 依赖：croniter, apscheduler, pytz

from base_agent import BaseAgent
from datetime import datetime
import croniter

class SchedulerAgent(BaseAgent):
    CRON_TASKS = [
        {"name": "github_trending", "cron": "0 8 * * *", "agent": "researcher", "task_type": "research"},
        {"name": "daily_report",    "cron": "0 9 * * *", "agent": "organizer", "task_type": "organize"},
        {"name": "health_check",    "cron": "*/30 * * * *", "agent": "silijian", "task_type": "quality"},
    ]

    def run(self):
        now = datetime.now()
        for task in self.CRON_TASKS:
            cron = croniter.croniter(task["cron"], now)
            next_run = cron.get_next(datetime)
            if (next_run - now).seconds < 60:
                self.enqueue(task["task_type"], {"trigger": task["name"]}, source="scheduler")
                self.logger.info(f"触发了定时任务: {task['name']}")

    def start(self):
        # 每分钟检查一次
        while True:
            self.run()
            time.sleep(60)
```

---

## 4. 依赖清单

### 4.1 Python 运行时（每个崽）

```txt
# requirements.txt
openai>=1.30.0          # LLM API 调用
requests>=2.31.0         # HTTP 客户端
sqlite3                  # 内置，无需安装
pydantic>=2.0            # 数据校验
python-dotenv>=1.0.0    # 环境变量
croniter>=4.0.0          # Cron 解析
python-dateutil>=2.8.0   # 日期工具
loguru>=0.7.0            # 日志（比 logging 更美观）
httpx>=0.27.0            # 异步 HTTP
```

### 4.2 Node.js 运行时（Gateway）

```json
// package.json（gateway/）
{
  "dependencies": {
    "ws": "^8.18.0",
    "express": "^4.19.0",
    "dotenv": "^16.4.0",
    "better-sqlite3": "^11.0.0",
    "telegraf": "^4.16.0",
    "discord.js": "^14.14.0",
    "axios": "^1.7.0"
  }
}
```

### 4.3 系统依赖

```bash
# Ubuntu/Debian
apt install -y python3.11 python3-pip nodejs npm redis-server sqlite3 git

# macOS
brew install python@3.11 node redis sqlite3 git
```

---

## 5. 安装步骤（Setup Steps）

### 5.1 标准安装（Ubuntu/Debian 单机）

```bash
# 1. 克隆项目
git clone https://github.com/your-org/open-agents-kit.git
cd open-agents-kit

# 2. 安装系统依赖
sudo apt update && sudo apt install -y python3.11 python3-pip nodejs npm sqlite3 redis-server git

# 3. 安装 Python 依赖
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 4. 安装 Node.js 依赖
cd gateway && npm install && cd ..

# 5. 配置环境变量
cp .env.example .env
# 编辑 .env 填入 OPENAI_API_KEY / TELEGRAM_BOT_TOKEN 等

# 6. 初始化数据库
python3 -c "import shared.task_queue; print('DB initialized')"

# 7. 启动 Redis（任务队列用）
redis-server --daemonize yes

# 8. 启动崽崽们
bash scripts/start.sh
```

### 5.2 Docker 一键部署（推荐）

```bash
# 最简方式：一条命令启动全部
docker compose up -d

# 查看崽崽状态
docker compose ps

# 查看某个崽的日志
docker compose logs -f researcher
```

---

## 6. 一键部署脚本（One-Click Deploy）

### 6.1 install.sh（自动检测系统并安装）

```bash
#!/bin/bash
# scripts/install.sh — 一键安装脚本（支持 Ubuntu/macOS）

set -e

echo "🦞 崽崽云安装器开始工作..."

# 检测系统
OS=$(uname -s)
if [ "$OS" = "Linux" ]; then
    echo "[1/6] 检测到 Linux，安装系统依赖..."
    sudo apt update && sudo apt install -y python3.11 python3-pip nodejs npm sqlite3 redis-server git curl
    redis-server --daemonize yes
elif [ "$OS" = "Darwin" ]; then
    echo "[1/6] 检测到 macOS，安装系统依赖..."
    which brew >/dev/null 2>&1 || /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    brew install python@3.11 node redis sqlite3 git
    brew services start redis
fi

# Python 环境
echo "[2/6] 创建 Python 虚拟环境..."
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Node.js 环境
echo "[3/6] 安装 Node.js 依赖..."
cd gateway && npm install && cd ..

# 配置
echo "[4/6] 生成配置文件..."
if [ ! -f .env ]; then
    cp .env.example .env
    echo "⚠️  请编辑 .env 填入必要的 API Key"
fi

# 数据库初始化
echo "[5/6] 初始化 SQLite 数据库..."
python3 -c "
import sqlite3, os
os.makedirs('shared/db', exist_ok=True)
db = sqlite3.connect('shared/db/tasks.db')
db.execute('''
    CREATE TABLE IF NOT EXISTS task_queue (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        task_type TEXT, source TEXT, payload TEXT,
        status TEXT DEFAULT \"pending\",
        created_at TEXT, updated_at TEXT, assigned_to TEXT, result TEXT
    )
''')
db.execute('''
    CREATE TABLE IF NOT EXISTS task_log (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        agent TEXT, action TEXT, task_id INTEGER,
        created_at TEXT, duration_ms INTEGER
    )
''')
db.commit()
print('✅ 数据库初始化完成')
"

# 权限
echo "[6/6] 设置脚本执行权限..."
chmod +x scripts/*.sh

echo ""
echo "🎉 安装完成！"
echo "   启动：bash scripts/start.sh"
echo "   状态：bash scripts/status.sh"
echo "   Docker：docker compose up -d"
```

### 6.2 start.sh（启动所有崽）

```bash
#!/bin/bash
# scripts/start.sh

source venv/bin/activate

echo "🚀 启动崽崽云..."

# 启动 Gateway
cd gateway && node index.js &
GATEWAY_PID=$!
cd ..

# 启动各崽（后台运行）
python3 agents/researcher/agent.py &
python3 agents/scheduler/agent.py &
python3 agents/organizer/agent.py &
python3 agents/coder/agent.py &
python3 agents/silijian/agent.py &

echo "✅ 崽崽全部上线！"
echo "   Gateway: http://localhost:18789"
echo "   PID: $GATEWAY_PID"
```

---

## 7. 技术选型说明

| 组件 | 选型 | 原因 |
|---|---|---|
| **消息网关** | Node.js + WS | 轻量、高并发、通道适配器生态丰富 |
| **崽 runtime** | Python 3.11+ | LLM API 生态最好（OpenAI/HuggingFace）|
| **任务队列** | SQLite WAL 模式 | 无运维，单文件，支持高并发读 |
| **缓存/锁** | Redis（可选） | 崽间 pub/sub + 分布式锁 |
| **日志** | Loguru（崽）+ JSONL（集中收集）| 搜索友好，结构化 |
| **配置** | JSON + .env | 简单，无额外学习成本 |
| **部署** | Docker Compose | 一键，跨平台，环境隔离 |
| **备份** | S3/OSS（可选）| 共享存储，artifacts 自动同步 |

---

*文档版本：v0.1 | coder崽出品 | 2026-03-25*
