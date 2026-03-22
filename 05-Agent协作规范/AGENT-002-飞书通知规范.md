---
id: AGENT-002
category: Agent协作规范
severity: medium
tags: [Agent, 飞书, 通知, 协作]
created: 2026-03-22
author: 光枢
related_issues: []
status: active
---

# 飞书通知规范

## 背景

AI Team 重要事件需要通知飞书群，让 BOSS 实时了解团队状态。

## Webhook 配置

```
URL: https://open.feishu.cn/open-apis/bot/v2/hook/b9f3a575-1a7b-48a2-8f17-f94e6317abf7
```

## 通知类型

### 1. 任务通知

| 类型 | 触发条件 | 格式 |
|------|---------|------|
| ✅ 任务完成 | Worker 完成 Issue | `✅ 任务完成\nIssue: #{id} {title}\n执行者: {name}` |
| ❌ 任务失败 | Worker 执行超时/报错 | `❌ 任务失败\nIssue: #{id}\n错误: {error}` |
| 🔴 Worker 离线 | 超过 5 分钟无心跳 | `🔴 Worker 离线超过 5 分钟\nBot: {name}\n最后心跳: {time}` |

### 2. 审查通知

| 类型 | 触发条件 | 格式 |
|------|---------|------|
| 📋 审查任务创建 | CTO 分配审查 | `📋 新审查任务\nIssue: #{id}\n审查对象: #{target}\n执行者: 大黑手` |
| ⭐ 审查完成 | 大黑手提交评分 | `⭐ 审查完成\nIssue: #{id}\n评分: {score}/5\n问题: {count} 个` |

### 3. 沟通汇总通知

| 类型 | 触发条件 | 格式 |
|------|---------|------|
| 📝 沟通记录汇总 | 对话达成共识 | `【AI Team 沟通记录汇总】\n日期: {date}\n参与: {names}\n消息数: {count}` |

## Shell 脚本示例

```bash
# 任务完成通知
notify_task_done() {
  local issue_id="$1"
  local title="$2"
  local worker="$3"
  
  curl -s -X POST "${FEISHU_WEBHOOK}" \
    -H "Content-Type: application/json" \
    -d "{
      \"msg_type\": \"text\",
      \"content\": {
        \"text\": \"✅ 任务完成\\nIssue: #${issue_id} ${title}\\n执行者: ${worker}\"
      }
    }" > /dev/null
}

# Worker 离线告警
notify_worker_offline() {
  local name="$1"
  local last_heartbeat="$2"
  
  curl -s -X POST "${FEISHU_WEBHOOK}" \
    -H "Content-Type: application/json" \
    -d "{
      \"msg_type\": \"text\",
      \"content\": {
        \"text\": \"🔴 Worker 离线超过 5 分钟\\nBot: ${name}\\n最后心跳: ${last_heartbeat}\"
      }
    }" > /dev/null
}
```

## 汇总脚本

```bash
# 手动触发沟通汇总
python3 /root/.openclaw/workspace/scripts/summarize-communication.py {issue_id}
```

## 注意事项

1. **JSON 转义**: 消息内容中的引号需要转义
2. **静默发送**: curl 输出重定向到 /dev/null
3. **失败重试**: 网络超时时可重试一次
4. **避免刷屏**: 同类通知合并发送

## 相关文件

- Worker 脚本: `/root/ai-team/scripts/poll-tasks.sh`
- 汇总脚本: `/root/.openclaw/workspace/scripts/summarize-communication.py`
