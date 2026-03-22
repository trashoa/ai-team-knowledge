---
id: DEPLOY-002
category: 部署运维Checklist
severity: high
tags: [监控, 告警, 系统健康, 运维]
created: 2026-03-22
author: 光枢
related_issues: []
status: active
---

# 系统监控 Checklist

## 日常监控项

### 1. 基础资源

| 指标 | 阈值 | 检查命令 |
|------|------|---------|
| CPU 使用率 | > 80% | `top -bn1 \| head -5` |
| 内存使用率 | > 80% | `free -h` |
| 磁盘使用率 | > 85% | `df -h` |
| 网络连接 | 正常 | `netstat -tuln \| grep LISTEN` |

### 2. Docker 容器

```bash
# 检查所有容器状态
docker ps -a

# 检查容器资源使用
docker stats --no-stream
```

| 容器 | 期望状态 | 检查 |
|------|---------|------|
| ai-team-dashboard | Running | `docker ps \| grep dashboard` |
| ai-team-qianwen | Running | `docker ps \| grep qianwen` |
| ai-team-kimi | Running | `docker ps \| grep kimi` |

### 3. 核心服务

| 服务 | 端口 | 健康检查 |
|------|------|---------|
| Dashboard | 3800 | `curl -s http://localhost:3800/api/health` |
| OpenClaw Gateway | 18789 | `curl -s http://127.0.0.1:18789/health` |
| Redis | 6379 | `redis-cli ping` |

## 自动监控脚本

### 内存监控脚本

```bash
# 文件位置: /root/ai-team/dashboard/monitor-memory.sh
# 频率: 每 10 分钟执行
# 阈值: 内存 > 80%

#!/bin/bash
MEMORY_USAGE=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100}')

if [ "$MEMORY_USAGE" -gt 80 ]; then
  # 发送告警到飞书
  curl -s -X POST "${FEISHU_WEBHOOK}" \
    -H "Content-Type: application/json" \
    -d "{\"msg_type\": \"text\", \"content\": {\"text\": \"🔴 内存告警: ${MEMORY_USAGE}%\"}}"
  
  # 检查 Dashboard 是否宕机
  if ! curl -s http://localhost:3800/api/health > /dev/null; then
    # 重启 Dashboard
    cd /root/ai-team/dashboard && pm2 restart all
  fi
fi
```

### Worker 心跳监控

```bash
# 检查 Worker 最后心跳时间
# 位置: /root/ai-team/bots/{name}/last_heartbeat
# 超时阈值: 5 分钟
```

## 告警规则

| 告警类型 | 触发条件 | 通知方式 | 严重程度 |
|----------|---------|---------|---------|
| CPU 告警 | > 80% 持续 5 分钟 | 飞书 | Medium |
| 内存告警 | > 80% | 飞书 + 自动重启 | High |
| 磁盘告警 | > 85% | 飞书 | High |
| Worker 离线 | > 5 分钟无心跳 | 飞书 | High |
| 服务不可用 | API 无响应 | 飞书 + 自动重启 | Critical |

## 监控频率

| 监控项 | 频率 |
|--------|------|
| 基础资源 | 每 10 分钟 |
| Docker 容器 | 每 5 分钟 |
| Worker 心跳 | 每 5 分钟 |
| API 健康 | 每 30 秒 |
| 日志分析 | 每日一次 |

## 相关文件

- 内存监控: `/root/ai-team/dashboard/monitor-memory.sh`
- PM2 配置: `/root/ai-team/ecosystem.config.js`
