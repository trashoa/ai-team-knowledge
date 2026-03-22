---
id: AGENT-001
category: Agent协作规范
severity: high
tags: [Agent, GitHub, Issues, 协作, 工作流]
created: 2026-03-22
author: 光枢
related_issues: []
status: active
---

# GitHub Issues 协作流程

## 背景

AI Team 使用 GitHub Issues 作为任务协作中心。

## 标签体系

### 1. 角色标签 (role:)

| 标签 | 角色 | 说明 |
|------|------|------|
| `role:qwen-worker` | 先锋手 | 编码开发工程师 |
| `role:kimi-worker` | 军师 | 策略分析（已停用） |
| `role:军师` | 大黑手 | 代码审查、技术咨询 |
| `role:cto` | 光枢 | 任务分配、协调 |

### 2. 状态标签 (status:)

| 标签 | 状态 | 说明 |
|------|------|------|
| `status:pending` | 待处理 | 等待 Worker 领取 |
| `status:in-progress` | 进行中 | Worker 正在处理 |
| `status:done` | 已完成 | 任务完成 |
| `status:blocked` | 阻塞 | 需要外部依赖 |
| `status:review` | 待审查 | 等待大黑手审查 |

## 标准工作流程

```
1. CTO 创建 Issue
   ↓
2. 添加标签: role:qwen-worker, status:pending
   ↓
3. Worker 轮询发现任务（每 2 分钟）
   ↓
4. Worker 评论 "开始处理" + 改标签 status:in-progress
   ↓
5. Worker 执行任务
   ↓
6. Worker 提交代码 + 改标签 status:review
   ↓
7. 大黑手审查 + 评分
   ↓
8. 审查通过 → status:done
   审查失败 → status:in-progress（返工）
```

## Issue 模板

```markdown
## 任务描述

[清晰描述任务目标]

## 技术方案

[可选：技术实现建议]

## 验收标准

- [ ] 功能实现
- [ ] 单元测试
- [ ] 文档更新

## 截止时间

[可选：期望完成时间]

## 相关资源

- [相关文档/代码链接]
```

## 注意事项

1. **一个任务一个 Issue** - 避免过于复杂
2. **明确的验收标准** - 便于 Worker 判断是否完成
3. **及时更新标签** - 让 CTO 能追踪进度
4. **审查必须自动化** - Worker 完成后自动创建审查任务

## 相关文件

- 任务仓库: `trashoa/ai-team-tasks`
- 代码仓库: `trashoa/ai-team-fullstack-code`
