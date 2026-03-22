---
id: FIX-002
category: 系统集成
severity: medium
tags: [飞书, Webhook, 通知, 机器人]
created: 2026-03-22
author: 光枢
related_issues: [14]
status: active
---

# 飞书群机器人 Webhook 通知

## 背景

AI Team 任务完成后需要通知飞书群。

## Webhook 信息

```
URL: https://open.feishu.cn/open-apis/bot/v2/hook/b9f3a575-1a7b-48a2-8f17-f94e6317abf7
```

## 消息格式

### 1. 任务完成通知

```bash
curl -X POST "${WEBHOOK_URL}" \
  -H "Content-Type: application/json" \
  -d '{
    "msg_type": "text",
    "content": {
      "text": "✅ 任务完成\nIssue: #10 Dashboard 任务进度显示\n执行者: 先锋\n状态: 完成 9/9 项"
    }
  }'
```

### 2. 任务失败通知

```bash
curl -X POST "${WEBHOOK_URL}" \
  -H "Content-Type: application/json" \
  -d '{
    "msg_type": "text",
    "content": {
      "text": "❌ 任务失败\nIssue: #11\n错误: 执行超时"
    }
  }'
```

### 3. Worker 离线告警

```bash
curl -X POST "${WEBHOOK_URL}" \
  -H "Content-Type: application/json" \
  -d '{
    "msg_type": "text",
    "content": {
      "text": "🔴 Worker 离线超过 5 分钟\nBot: 先锋\n最后心跳: 2026-03-22 18:00:00"
    }
  }'
```

## 常见问题

### 1. JSON 转义问题

```bash
# ❌ 错误：heredoc 中的变量转义问题
json=$(cat <<EOF
{"msg_type": "text", "content": {"text": "$message"}}
EOF
)

# ✅ 正确：使用纯文本格式
text="任务完成: $issue"
curl -X POST "${WEBHOOK_URL}" \
  -H "Content-Type: application/json" \
  -d "{\"msg_type\": \"text\", \"content\": {\"text\": \"${text}\"}}"
```

### 2. 特殊字符转义

```bash
# 消息中有双引号或换行需要转义
message=$(echo "$message" | sed 's/"/\\"/g' | tr '\n' ' ')
```

## Shell 函数封装

```bash
notify_feishu() {
  local text="$1"
  curl -s -X POST "${FEISHU_WEBHOOK}" \
    -H "Content-Type: application/json" \
    -d "{\"msg_type\": \"text\", \"content\": {\"text\": \"${text}\"}}" \
    > /dev/null
}
```

## 相关文件

- Worker 脚本: `/root/ai-team/scripts/poll-tasks.sh`
- 环境变量: `/root/ai-team/.env`
