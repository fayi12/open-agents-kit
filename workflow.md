# 工作流文档

**更新时间**: 2026-03-29

---

## 一、完整任务流程

```
用户(mypc_brain)
    ↓ 发送任务
silijian_chief (调度崽)
    ↓ 分解任务
researcher_chief / coder_chief (执行崽)
    ↓ 完成后
organizer_chief (整理崽)
    ↓ 整理后
reviewer_chief (审核崽)
    ↓ 审核通过
organizer_chief (存档)
    ↓
回调通知用户
```

---

## 二、Agent调用规范

### 正确调用方式
```bash
# 使用wrapper脚本调用Chief
bash /home/fayi/oc agent --agent researcher_chief --message '<英语任务>'

# 禁止使用spawn subagent方式
```

---

## 三、回调机制

### VM→Windows回调
```bash
ssh -p 2222 28726@172.30.112.1 cmd /c echo <消息> >> C:\\Users\\28726\\vm_callback.txt
```

---

## 四、任务文件

- 完成: `/home/fayi/huitiao/ok/<任务名>_等待检查.md`
- 未完成: `/home/fayi/huitiao/no/<任务名>_等待检查.md`

---

## 五、手动监督模式

当自动回调不可靠时：
1. 发送任务给silijian_chief
2. 等待1-2分钟后检查文件
3. 确认每步完成后再发下一步

---

*本文档由mypc_brain于2026-03-29自动生成*
