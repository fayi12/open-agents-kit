# 🚀 Open Agents Kit（崽崽云）部署指南

> 文档版本：v1.0 | 更新日期：2026-03-25  
> 适用系统：Linux / macOS / Windows (WSL2)  
> 部署难度：⭐⭐ ☆☆☆（初中级）

---

## 一、前置条件（Prerequisites）

### 1.1 硬件与系统

| 项目 | 最低要求 | 推荐配置 |
|------|----------|----------|
| CPU | 2 核 | 4 核+ |
| 内存 | 4 GB | 8 GB+ |
| 磁盘 | 10 GB 可用 | 20 GB+ SSD |
| 系统 | Ubuntu 20.04+ / macOS 12+ / Windows WSL2 | 同左 |
| 网络 | 能访问 GitHub / PyPI / 模型API | 同左，需科学上网 |

### 1.2 软件依赖

```
✅ Python 3.10 或更高
✅ Git
✅ pip（Python包管理器）
✅ OpenAI API Key（或其他兼容的大模型API）
✅ 消息通道（可选， Telegram / 微信 / Discord 三选一）
```

### 1.3 检查你的环境

```bash
# 检查 Python 版本（需 >= 3.10）
python3 --version
# 输出类似：Python 3.11.6 ✅

# 检查 Git
git --version
# 输出类似：git version 2.39.0 ✅

# 检查 pip
pip --version
# 输出类似：pip 23.2.1 from /usr/lib/python3... ✅
```

> ⚠️ 如果版本低于要求，请先升级：
> ```bash
> # Ubuntu/Debian
> sudo apt update && sudo apt install python3.11 python3-pip
>
> # macOS
> brew install python@3.11
> ```

---

## 二、分步安装（Step-by-Step Install）

### 第一步：克隆项目

```bash
cd ~                        # 进入 home 目录
git clone https://github.com/your-repo/open-agents-kit.git
cd open-agents-kit
```

> 💡 如果你没有 Git，可以用以下方式下载zip包：
> ```bash
> wget https://github.com/your-repo/open-agents-kit/archive/refs/heads/main.zip
> unzip main.zip && cd open-agents-kit
> ```

---

### 第二步：创建 Python 虚拟环境

```bash
# 创建虚拟环境（推荐）
python3 -m venv venv

# 激活虚拟环境
# Linux/macOS:
source venv/bin/activate
# Windows (PowerShell):
# venv\Scripts\Activate.ps1

# 激活成功后，终端会显示 (venv) 前缀
```

---

### 第三步：安装依赖

```bash
# 方式A：自动安装（推荐）
pip install -r requirements.txt

# 方式B：手动安装核心依赖
pip install openai python-dotenv schedule requests
```

> ⏱️ 安装耗时约 1~3 分钟，取决于网络状态。

---

### 第四步：配置环境变量

```bash
# 复制示例配置文件
cp .env.example .env

# 编辑配置文件
nano .env        # Linux/macOS
# 或用记事本打开 .env 文件（Windows）
```

填入以下必要信息：

```env
# ===== 必填 =====
OPENAI_API_KEY=sk-your-api-key-here

# ===== 可选 =====
TELEGRAM_BOT_TOKEN=your-telegram-bot-token
DISCORD_WEBHOOK_URL=your-discord-webhook-url
LOG_LEVEL=INFO
DEFAULT_MODEL=gpt-4o-mini
MAX_TOKENS=2048
```

> 🔑 **如何获取 OpenAI API Key？**
> 1. 访问 https://platform.openai.com/api-keys
> 2. 登录账号（如果没有先注册）
> 3. 点击 "Create new secret key"
> 4. 复制生成的 Key（格式：`sk-xxxx...`）
>
> ⚠️ 不要泄露给他人！不要提交到 Git！

---

### 第五步：验证安装

```bash
# 在项目根目录执行
python3 -c "from agents import researcher, coder, organizer; print('✅ 所有崽崽就位！')"

# 如果看到 ✅ 开头的中文输出，说明安装成功
```

---

### 第六步：启动崽崽系统

```bash
# 方式A：启动全部崽崽（后台运行）
bash start.sh

# 方式B：启动单个崽崽（调试用）
python3 -m agents.organizer
python3 -m agents.researcher
python3 -m agents.coder

# 方式C：Docker 部署（推荐生产环境）
docker-compose up -d
```

---

## 三、崽崽角色说明

| 崽名 | 职责 | 配置文件 |
|------|------|----------|
| 🗂️ **存档崽**（organizer） | 文件整理、归档、清理、目录维护 | `agents/organizer.py` |
| 🔬 **研究崽**（researcher） | 竞品调研、数据采集、趋势分析 | `agents/researcher.py` |
| 💻 **编码崽**（coder） | 代码开发、脚本编写、技术方案实现 | `agents/coder.py` |
| 📅 **调度崽**（scheduler） | 定时任务编排、触发器管理 | `agents/scheduler.py` |
| 📝 **事件崽**（silijian） | 事件记录、日志审计、异常告警 | `agents/silijian.py` |

---

## 四、快速启动清单（Quick Start Checklist）

在开始之前，逐项打勾 ✅：

```
前置检查
[ ] Python 3.10+ 已安装          → python3 --version
[ ] Git 已安装                   → git --version
[ ] OpenAI API Key 已获取         → 访问 platform.openai.com
[ ] 网络可访问 GitHub 和 PyPI     → ping github.com

安装步骤
[ ] 项目已克隆到本地              → ls open-agents-kit/
[ ] 虚拟环境已创建并激活          → 终端显示 (venv)
[ ] 依赖已安装                    → pip list | grep openai
[ ] .env 配置文件已创建           → cat .env | grep OPENAI_API_KEY
[ ] API Key 已填入 .env          → 不是空的 sk-xxx...

验证
[ ] 安装验证通过                  → python3 -c "import ..." 无报错
[ ] 崽崽系统可以启动              → bash start.sh 无报错

可选：消息通道
[ ] Telegram Bot Token 已配置     → .env 中已填入
[ ] 或 Discord Webhook 已配置      → .env 中已填入
```

---

## 五、故障排除（Troubleshooting FAQ）

### Q1：安装依赖时报错 `SSL: CERTIFICATE_VERIFY_FAILED`

**原因：** 网络问题或代理配置异常。

**解决方法：**
```bash
# 方法A：临时跳过 SSL 验证（不推荐生产环境）
pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org -r requirements.txt

# 方法B：配置公司/地区代理
export HTTP_PROXY=http://your-proxy:port
export HTTPS_PROXY=http://your-proxy:port
pip install -r requirements.txt
```

---

### Q2：启动时报错 `ModuleNotFoundError: No module named 'openai'`

**原因：** 虚拟环境未激活，或依赖未安装成功。

**解决方法：**
```bash
# 确认虚拟环境激活（终端应有 (venv) 前缀）
source venv/bin/activate

# 重新安装依赖
pip install -r requirements.txt

# 确认包已安装
pip list | grep openai
```

---

### Q3：报错 `openai.AuthenticationError: Incorrect API key provided`

**原因：** `.env` 文件中 API Key 填写错误或未生效。

**解决方法：**
```bash
# 检查 .env 文件内容
cat .env | grep OPENAI_API_KEY
# 确认格式是 sk-... 开头，不是 "sk-..." 带引号

# 重启终端让 .env 生效，或手动导出
export OPENAI_API_KEY=sk-your-actual-key
```

---

### Q4：崽崽启动后没反应，日志显示 `Waiting for tasks...`

**原因：** 这是正常状态，说明崽崽在待机，等待调度器分配任务。

**解决方法：**
```bash
# 触发一次手动测试任务
python3 -c "
from agents.scheduler import add_task
add_task('researcher', '分析 GitHub Trending Python 榜单')
print('✅ 任务已加入队列')
"

# 查看调度崽是否正常
python3 -m agents.scheduler
```

---

### Q5：Telegram/Discord 消息收不到

**原因：** Token/Webhook 未配置，或 Bot 没有正确权限。

**解决方法：**
```bash
# Telegram：确认 Bot Token 正确
# 1. 在 Telegram 找 @BotFather
# 2. 发送 /newbot 创建新 Bot
# 3. 复制 Bot Token 到 .env

# Discord：确认 Webhook URL 格式正确
# 格式：https://discord.com/api/webhooks/你的ID/你的token

# 重启崽崽系统
bash restart.sh
```

---

### Q6：服务器重启后，崽崽不自动运行

**原因：** 没有配置开机自启或 systemd 服务。

**解决方法（systemd 服务）：**
```bash
# 创建 systemd 服务文件
sudo nano /etc/systemd/system/zaibaby.service
```

写入以下内容：
```ini
[Unit]
Description=Open Agents Kit - 崽崽云
After=network.target

[Service]
Type=simple
User=你的用户名
WorkingDirectory=/home/你的用户名/open-agents-kit
ExecStart=/home/你的用户名/open-agents-kit/venv/bin/python3 -m agents.scheduler
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# 启用开机自启
sudo systemctl daemon-reload
sudo systemctl enable zaibaby
sudo systemctl start zaibaby
```

---

### Q7：日志文件太大，占满磁盘

**解决方法：**
```bash
# 查看日志目录大小
du -sh ~/open-agents-kit/logs/

# 启用日志轮转（logrotate）
sudo nano /etc/logrotate.d/open-agents-kit
```

写入：
```
~/open-agents-kit/logs/*.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
```

---

## 六、升级与维护

### 升级到新版本

```bash
cd open-agents-kit
git pull origin main
source venv/bin/activate
pip install -r requirements.txt  # 更新依赖
bash restart.sh                   # 重启服务
```

### 查看崽崽状态

```bash
# 查看运行中的崽崽进程
ps aux | grep python

# 查看实时日志
tail -f logs/崽崽名.log

# 查看调度队列状态
python3 -c "from agents.scheduler import show_queue; show_queue()"
```

---

## 七、一键部署脚本（Docker）

如果你有 Docker 环境，推荐使用 docker-compose 部署：

```bash
# 1. 复制环境配置
cp .env.example .env
nano .env   # 填入你的 API Key

# 2. 一键启动
docker-compose up -d

# 3. 查看崽崽状态
docker-compose ps

# 4. 查看日志
docker-compose logs -f
```

> 🐳 **Docker 方式的优势**：环境隔离、不污染系统环境、一条命令搞定所有依赖。

---

*📖 文档由存档崽生成 | 混乱是效率的天敌，整洁是创造的前提。*  
*项目地址：https://github.com/your-repo/open-agents-kit*
