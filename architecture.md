# 崽崽云架构文档

**更新时间**: 2026-03-29

## 系统架构

### 整体拓扑
用户(Windows/mypc_brain) → VM(172.30.127.229) → OpenClaw Gateway → 5个Chief Agent

### 关键组件
| 组件 | 地址/路径 | 说明 |
|------|-----------|------|
| VM SSH | 172.30.127.229:22 | Windows→VM |
| Windows SSH | 127.0.0.1:2222 | VM→Windows回调 |
| OpenClaw Gateway | ws://127.0.0.1:18789 | Agent通信 |
| 任务存档 | /home/fayi/huitiao/ok/ | 完成的任务 |

## 崽崽团队 (5 Chief Agents)

| Agent | 模型 | 职责 | Skills |
|-------|------|------|--------|
| researcher_chief | M2.7 | 研究调研 | multi-search, deep-research |
| coder_chief | M2.7 | 代码开发 | github, codeconductor |
| silijian_chief | M2.7 | 调度协调 | automation-workflows |
| organizer_chief | M2.5 | 知识整理 | ontology, memory-manager |
| reviewer_chief | M2.7 | 审核质量 | skill-vetter, self-reflection |

## 工作流程
用户 → silijian_chief → researcher → organizer → reviewer → 存档

## Agent调用
bash /home/fayi/oc agent --agent <chief名> --message '<英语任务>'

## 回调机制
ssh -p 2222 28726@172.30.112.1 cmd /c echo <消息> >> C:\\Users\\28726\\vm_callback.txt

## 注意事项
- 禁止spawn subagent方式
- 只发给silijian_chief调度
- 用bash /home/fayi/oc agent调用
