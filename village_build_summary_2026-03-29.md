# 崽崽小镇搭建总结

**日期**: 2026-03-29
**重要性**: ⭐⭐⭐⭐⭐
**标签**: OpenClaw | VM部署 | 调度系统

---

## 一、系统架构

### 1.1 整体拓扑
`
用户(Windows) → VM(172.30.127.229) → OpenClaw Gateway → 5个Chief Agent
`

### 1.2 关键端口
| 服务 | 地址 |
|------|------|
| VM SSH | 172.30.127.229:22 |
| Windows SSH | 127.0.0.1:2222 |
| OpenClaw Gateway | ws://127.0.0.1:18789 |

---

## 二、崽崽团队配置

### 5个Chief Agent
| Agent | 模型 | 职责 |
|-------|------|------|
| researcher_chief | M2.7 | 研究调研 |
| coder_chief | M2.7 | 代码开发 |
| silijian_chief | M2.7 | 调度协调 |
| organizer_chief | M2.5 | 知识整理 |
| reviewer_chief | M2.7 | 审核质量 |

---

## 三、工作流程

`
用户 → silijian_chief(调度) → researcher → organizer → reviewer → 存档
`

### 回调机制
VM→Windows: ssh -p 2222 28726@172.30.112.1 cmd /c echo <msg> >> C:\\Users\\28726\\vm_callback.txt

---

## 四、关键技术点

### Agent调用格式
`ash
bash /home/fayi/oc agent --agent <chief名> --message '<英语任务>'
`

### 目录结构
`
/home/fayi/
├── oc                          # OpenClaw wrapper
├── silijian_chief/            # 调度崽
├── researcher_chief/           # 研究崽
├── coder_chief/               # 编码崽
├── organizer_chief/           # 知识崽
├── reviewer_chief/            # 审核崽
└── huitiao/
    ├── ok/                    # 完成的任务
    └── no/                    # 未完成的任务
`

---

## 五、注意事项 ⚠️

- ❌ 禁止spawn subagent方式
- ✅ 只发给silijian_chief调度
- ✅ 用bash /home/fayi/oc agent调用
- ✅ 手动监督每步执行

---

## 六、Skills分配

| Agent | Skills |
|-------|--------|
| researcher | multi-search, deep-research, baidu-search |
| coder | github, codeconductor, browser-use |
| silijian | automation-workflows, proactive-agent |
| organizer | ontology, memory-manager |
| reviewer | skill-vetter, self-reflection |

---

## 七、后续优化

1. 实现崽崽自动回调
2. 配置Signal/Telegram通知
3. 优化调度流程
4. 自动归档到GitHub

---

*本文档由mypc_brain于2026-03-29自动生成*
