# 崽崽云架构文档

**更新时间**: 2026-03-29

---

## 一、系统架构

### 1.1 整体拓扑
```
用户(控制端)
    ↓ SSH
VM: <你的VM IP>:<SSH端口>
    ↓
OpenClaw Gateway (ws://127.0.0.1:18789)
    ↓
5个Chief Agent (崽崽)
    ↓ 调度 (silijian_chief)
任务队列 → 执行 → 存档
```

### 1.2 关键组件

| 组件 | 地址/路径 | 说明 |
|------|-----------|------|
| VM SSH | 用户自定义 | 控制端→VM |
| OpenClaw Gateway | ws://127.0.0.1:18789 | Agent通信 |
| 任务存档 | ~/huitiao/ok/ | 完成的任务 |
| 崽崽workspace | ~/<chief>_chief/ | 各崽崽工作目录 |

---

## 二、崽崽团队 (5 Chief Agents)

| Agent | 模型 | 职责 | Skills |
|-------|------|------|--------|
| researcher_chief | M2.7 | 研究调研 | multi-search, deep-research, baidu-search, news-summary |
| coder_chief | M2.7 | 代码开发 | github, codeconductor, browser-use, agent-browser |
| silijian_chief | M2.7 | 调度协调 | automation-workflows, proactive-agent |
| organizer_chief | M2.5 | 知识整理 | ontology, memory-manager, self-improving |
| reviewer_chief | M2.7 | 审核质量 | skill-vetter, self-reflection, humanizer |

---

## 三、工作流程

### 3.1 完整任务流程
```
用户 → 主控Agent → silijian_chief(调度崽)
    ↓ 分解任务
researcher_chief / coder_chief (执行崽)
    ↓ 完成后
organizer_chief (整理崽)
    ↓ 整理后
reviewer_chief (审核崽)
    ↓ 审核通过
存档到 ~/huitiao/ok/<任务名>_等待检查.md
```

### 3.2 Agent调用格式
```bash
bash ~/oc agent --agent <chief名> --message '<英语任务>'
```

### 3.3 回调机制
```bash
# VM→控制端回调（需要配置反向SSH）
ssh -p <端口> <用户>@<控制端IP> cmd /c echo <消息> >> <回调文件>
```

---

## 四、目录结构

```
~/
├── oc                              # OpenClaw wrapper脚本
├── npm-global/lib/node_modules/openclaw/
├── silijian_chief/               # 调度崽 workspace
├── researcher_chief/              # 研究崽 workspace
├── coder_chief/                  # 编码崽 workspace
├── organizer_chief/              # 知识崽 workspace
├── reviewer_chief/               # 审核崽 workspace
├── huitiao/
│   ├── ok/                      # 完成的任务
│   └── no/                      # 未完成的任务
└── work_db/崽崽村/              # 崽崽共享数据
```

---

## 五、Skills系统

### 5.1 SkillHub安装
```bash
~/.local/bin/skillhub install <skill名>
```

### 5.2 Skills复制到Chief
```bash
cp -r ~/.openclaw/skills/<skill名> ~/<chief名>/.openclaw/skills/
```

---

## 六、部署要求

### 6.1 VM要求
- Ubuntu 20.04+ / Debian
- OpenClaw 2026.3.24+
- Node.js 18+
- 4GB+ RAM
- 20GB+ 磁盘

### 6.2 网络要求
- SSH访问
- 内网互通（或公网可访问）

---

## 七、注意事项 ⚠️

### 7.1 严禁事项
- ❌ 禁止spawn subagent方式 (会创建非法agent)
- ❌ 不要直接发给多个崽崽 (打乱调度流程)
- ❌ 不要让崽崽自己spawn subagent
- ❌ 不要提交Token/密码等到代码库

### 7.2 正确做法
- ✅ 只发给silijian_chief调度
- ✅ 用bash ~/oc agent调用
- ✅ 敏感信息放在环境变量或独立配置中

---

## 八、后续优化

1. 实现崽崽自动回调
2. 配置Signal/Telegram通知
3. 优化调度流程
4. 自动归档到GitHub

---

*本文档由mypc_brain于2026-03-29自动生成*
