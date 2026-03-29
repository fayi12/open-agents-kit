# 崽崽云架构文档

**更新时间**: 2026-03-29

---

## 一、系统架构

### 1.1 整体拓扑
`
用户(控制端)
    ↓ SSH
VM: <你的VM IP>
    ↓
OpenClaw Gateway (ws://127.0.0.1:18789)
    ↓
5个Chief Agent (崽崽)
    ↓ 调度 (silijian_chief)
任务队列 → 执行 → 存档
`

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
| researcher_chief | 可配置 | 研究调研 | multi-search, deep-research, baidu-search, news-summary |
| coder_chief | 可配置 | 代码开发 | github, codeconductor, browser-use, agent-browser |
| silijian_chief | 可配置 | 调度协调 | automation-workflows, proactive-agent |
| organizer_chief | 可配置 | 知识整理 | ontology, memory-manager, self-improving |
| reviewer_chief | 可配置 | 审核质量 | skill-vetter, self-reflection, humanizer |

---

## 三、工作流程

### 3.1 完整任务流程
`
用户 → 主控Agent → silijian_chief(调度崽)
    ↓ 分解任务
researcher_chief / coder_chief (执行崽)
    ↓ 完成后
organizer_chief (整理崽)
    ↓ 整理后
reviewer_chief (审核崽)
    ↓ 审核通过
存档到 ~/huitiao/ok/<任务名>_等待检查.md
`

### 3.2 Agent调用格式
`ash
bash ~/oc agent --agent <chief名> --message '<英语任务>'
`

---

## 四、目录结构

`
~/
├── oc                              # OpenClaw wrapper脚本
├── silijian_chief/               # 调度崽 workspace
├── researcher_chief/              # 研究崽 workspace
├── coder_chief/                  # 编码崽 workspace
├── organizer_chief/              # 知识崽 workspace
├── reviewer_chief/               # 审核崽 workspace
└── huitiao/
    ├── ok/                      # 完成的任务
    └── no/                      # 未完成的任务
`

---

## 五、部署要求

### 5.1 VM要求
- Ubuntu 20.04+ / Debian
- OpenClaw 2026.3.24+
- Node.js 18+
- 4GB+ RAM

### 5.2 网络要求
- SSH访问
- 内网互通

---

## 六、注意事项 ⚠️

- ❌ 禁止spawn subagent方式
- ✅ 只发给silijian_chief调度
- ✅ 敏感信息放环境变量

---

*本文档由mypc_brain于2026-03-29自动生成*
