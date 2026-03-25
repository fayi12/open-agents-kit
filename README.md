# Open Agents Kit（崽崽云）

> 轻量级多 Agent 协作系统 · 5个专业化崽替你打工 · 一键部署，开箱即用

[![Stars](https://img.shields.io/github/stars/your-org/open-agents-kit)](https://github.com/your-org/open-agents-kit)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.11+](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![Node.js 18+](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org/)

---

## 一句话定位

**崽崽云：一键部署 5 个专业化 AI 崽，替你 7×24 小时干活——研究、写作、运营、客服全自动，数据留在自己服务器，不用写代码。**

---

## 核心特性

### 🐣 5 个专业崽，分工明确

| 崽 | 职责 | 能做什么 |
|---|---|---|
| **研究崽** Researcher's Agent | 信息采集与调研 | GitHub Trending、竞品动态、行业情报自动抓取 |
| **码农崽** Coder's Agent | 代码与工程任务 | 代码审查、文件写入、Git 操作自动化 |
| **管家崽** Scheduler's Agent | 定时任务调度 | 每日自动任务触发、任务入队出队管理 |
| **整理崽** Organizer's Agent | 数据整理与输出 | 报告生成、内容格式化、文件归档 |
| **事儿妈崽** Silijian's Agent | 质量把控与升级 | 质量门禁、人工复核点、异常上报 |

### 🏗 架构亮点

- **Gateway 消息网关**：Node.js 实现，接收微信 / Telegram / Discord / Web 消息，统一路由分发
- **崽间零依赖**：每个崽是独立进程，通过 SQLite 任务队列协作，可独立扩缩容
- **本地优先**：所有数据默认存在本地（SQLite WAL 模式），支持 S3/OSS 备份
- **可选 Redis**：崽间 pub/sub + 分布式锁，量上来也不怕

### 🚀 开箱即用

- **一条命令启动**：Docker Compose 一键跑全栈
- **或手动部署**：bash 脚本帮你搞定 Python 环境 + Node 依赖 + 数据库初始化
- **多消息通道**：Telegram Bot、Discord Webhook、企业微信、Web UI 按需启用

### 🧩 典型使用场景

1. **AI 内容工厂**：研究崽采集热点 → 写手崽生成文章 → 整理崽格式化 → 事儿妈崽审核发布
2. **竞品监控**：研究崽每日抓取竞品动态 → 整理崽汇总报告 → 推送到微信群
3. **个人 AI 助理团队**：在微信/Telegram 上拥有一个 7×24 小时运转的 AI 小团队

---

## 技术架构

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
│  │  消息路由 · 权限控制 · 通道管理      │                    │
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

**架构原则**：Gateway 只负责消息通道接入（类 OpenClaw），不处理具体业务；崽崽之间通过 SQLite 任务队列协作，无直接依赖。

---

## 快速开始

### 前置依赖

- Python 3.11+ / Node.js 18+ / SQLite3 / Git
- （可选）Docker & Docker Compose

### 方式一：Docker 一键部署（推荐）

```bash
# 克隆项目
git clone https://github.com/your-org/open-agents-kit.git
cd open-agents-kit

# 一条命令启动全部崽崽
docker compose up -d

# 查看崽崽状态
docker compose ps
```

### 方式二：手动安装

```bash
# 1. 克隆项目
git clone https://github.com/your-org/open-agents-kit.git
cd open-agents-kit

# 2. 安装依赖（自动检测 Ubuntu/macOS）
bash scripts/install.sh

# 3. 配置环境变量
cp .env.example .env
# 编辑 .env，填入 OPENAI_API_KEY / TELEGRAM_BOT_TOKEN 等

# 4. 启动崽崽
bash scripts/start.sh
```

启动成功后：
- Gateway 运行在 `http://localhost:18789`
- 查看崽崽状态：`bash scripts/status.sh`

---

## 项目结构

```
open-agents-kit/
├── gateway/                 # 消息网关（Node.js）
│   ├── index.js             # Gateway 主入口
│   ├── router.js            # 消息路由
│   └── channel/             # 通道适配器（Telegram / Discord / WeChat / Web）
│
├── agents/                  # 崽崽目录（每个崽独立子模块）
│   ├── researcher/          # 研究崽
│   ├── coder/               # 码农崽
│   ├── scheduler/           # 管家崽
│   ├── organizer/           # 整理崽
│   └── silijian/            # 事儿妈崽
│
├── shared/                  # 崽崽共享资源
│   ├── task_queue.py        # SQLite 任务队列封装
│   ├── db/tasks.db          # SQLite 数据库
│   ├── config/agents.json   # 崽崽配置
│   └── logs/                 # 集中日志
│
├── scripts/                 # 运维脚本
│   ├── install.sh           # 一键安装
│   ├── start.sh             # 启动全部崽
│   ├── stop.sh              # 停止
│   └── status.sh            # 查看状态
│
└── docker/
    ├── docker-compose.yml   # 多容器编排
    └── Dockerfile.all       # All-in-One 单容器
```

---

## License

MIT License - 崽崽云项目组 · 2026
