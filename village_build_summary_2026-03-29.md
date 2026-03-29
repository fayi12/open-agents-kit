# 崽崽小镇搭建总结

**日期**: 2026-03-29
**重要性**: ⭐⭐⭐⭐⭐ (核心项目)
**标签**: 崽崽小镇 | OpenClaw | VM部署 | 调度系统

---

## 一、系统架构

### 1.1 整体拓扑
```
用户(Windows)
    ↓ SSH
VM: 172.30.127.229 (fayi/123456)
    ↓
OpenClaw Gateway (ws://127.0.0.1:18789)
    ↓
5个崽崽Chief Agent
    ↓ 调度
huitiao/ok/ & huitiao/no/ (任务结果目录)
```

### 1.2 关键端口
| 服务 | 地址 | 说明 |
|------|------|------|
| VM SSH | 172.30.127.229:22 | Windows→VM |
| Windows SSH | 127.0.0.1:2222 | VM→Windows回调 |
| OpenClaw Gateway | ws://127.0.0.1:18789 | Agent通信 |

---

## 二、崽崽团队配置

### 2.1 5个Chief Agent
| Agent | 模型 | 职责 | Skills |
|-------|------|------|--------|
| researcher_chief | M2.7 | 研究调研 | multi-search, deep-research, baidu-search, news-summary |
| coder_chief | M2.7 | 代码开发 | github, codeconductor, browser-use, agent-browser |
| silijian_chief | M2.7 | 调度协调 | automation-workflows, proactive-agent |
| organizer_chief | M2.5 | 知识整理 | ontology, memory-manager, self-improving |
| reviewer_chief | M2.7 | 审核质量 | skill-vetter, self-reflection, humanizer |

### 2.2 执行命令格式
```bash
# 调用Chief Agent
bash /home/fayi/oc agent --agent <chief名> --message '<英语任务>'

# 回调Windows
ssh -p 2222 28726@172.30.112.1 cmd /c echo <消息> >> C:\\Users\\28726\\vm_callback.txt
```

---

## 三、目录结构

```
/home/fayi/
├── oc                          # OpenClaw wrapper脚本
├── npm-global/lib/node_modules/openclaw/
├── silijian_chief/            # 调度崽 workspace
├── researcher_chief/           # 研究崽 workspace
├── coder_chief/               # 编码崽 workspace
├── organizer_chief/           # 知识崽 workspace
├── reviewer_chief/            # 审核崽 workspace
├── huitiao/
│   ├── ok/                    # 完成的任务
│   └── no/                    # 未完成的任务
└── work_db/崽崽村/           # 崽崽共享数据
```

---

## 四、工作流程

### 4.1 完整任务流程
```
用户 → mypc_brain → silijian_chief(调度)
    ↓ 分解任务
researcher_chief → 执行研究
    ↓ 完成后
organizer_chief → 整理评估
    ↓ 整理后
reviewer_chief → 审核质量
    ↓ 审核通过
存档到 huitiao/ok/<任务名>_等待检查.md
    ↓
回调通知用户
```

### 4.2 回调机制
- VM→Windows: `ssh -p 2222 28726@172.30.112.1 cmd /c echo <消息> >> C:\\Users\\28726\\vm_callback.txt`
- Windows读取文件获取崽崽消息

---

## 五、关键技术点

### 5.1 SSH双向配置
**Windows→VM**: 正常SSH连接
```bash
ssh fayi@172.30.127.229
```

**VM→Windows**: 需要配置authorized_keys
- Key路径: `C:\ProgramData\ssh\administrators_authorized_keys`
- 格式: 一行ssh-ed25519公钥

### 5.2 OpenClaw Agent调用
```bash
# 正确方式 - 使用wrapper脚本
bash /home/fayi/oc agent --agent researcher_chief --message '<任务>'

# 错误方式 - spawn subagent(会创建非法agent)
```

### 5.3 SOUL.md配置
每个崽崽的SOUL.md包含:
- 身份定义
- 职责说明
- 工作流程
- 回调规则

---

## 六、Skills安装

### 6.1 SkillHub CLI
```bash
~/.local/bin/skillhub install <skill名>
```

### 6.2 Skills复制到Chief
```bash
cp -r /home/fayi/.openclaw/skills/<skill名> /home/fayi/<chief名>/.openclaw/skills/
```

---

## 七、注意事项 ⚠️

### 7.1 严禁事项
- ❌ 禁止暴露Token/Key
- ❌ 不要用spawn subagent方式(会创建非法agent)
- ❌ 不要直接发给多个崽崽(打乱调度流程)
- ❌ 不要让崽崽自己spawn subagent

### 7.2 正确做法
- ✅ 只发给silijian_chief调度
- ✅ 用bash /home/fayi/oc agent调用
- ✅ 回调写到vm_callback.txt
- ✅ 手动监督每步执行

### 7.3 常见问题
1. **回调失败**: 检查SSH key格式是否正确(单行)
2. **崽崽不执行**: 确认SOUL.md已更新
3. **命令解析错误**: 使用wrapper脚本 ~/oc

---

## 八、文件同步

### 8.1 记忆备份
- 主位置: E:\work_db\MEMORY_latest.md
- VM副本: /home/fayi/work_db/

### 8.2 崽崽汇报
- 路径: /home/fayi/work_db/崽崽村/共享/汇报.txt

---

## 九、后续优化方向

1. **自动化**: 实现崽崽自动回调(目前半自动)
2. **信号通道**: 配置Signal/Telegram发送通知
3. **调度优化**: 让silijian真正成为调度中心
4. **知识库**: 崽崽研究成果自动归档到GitHub

---

*本文档用于更新崽崽GitHub项目，请勿泄露任何密钥Token*
