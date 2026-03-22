---
id: FIX-001
category: 性能优化
severity: medium
tags: [Redis, 缓存, 性能, TTL]
created: 2026-03-22
author: 光枢
related_issues: []
status: active
---

# Redis 缓存集成方案

## 背景

Dashboard API 响应慢（3.5秒），需要集成 Redis 缓存提升性能。

## 优化效果

- **优化前**: 3.5 秒
- **优化后**: 0.04 秒
- **提升**: 96 倍

## 实现步骤

### 1. 安装依赖

```bash
npm install ioredis
```

### 2. Redis 连接配置

```javascript
const Redis = require('ioredis');

const redis = new Redis({
  host: '127.0.0.1',
  port: 6379,
  password: process.env.REDIS_PASSWORD || undefined,
  retryDelayOnFailover: 100,
  maxRetriesPerRequest: 3
});
```

### 3. 缓存键命名规范

```
dashboard:status      - 团队状态
dashboard:monitor    - 监控数据
dashboard:conversation - 对话历史
dashboard:bot:{id}   - Bot 详情
```

### 4. 缓存读写模式

```javascript
async function getCachedData(key, fetchFn, ttl = 300) {
  // 1. 尝试从缓存获取
  const cached = await redis.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  // 2. 缓存未命中，从数据源获取
  const data = await fetchFn();

  // 3. 写入缓存
  await redis.setex(key, ttl, JSON.stringify(data));

  return data;
}
```

### 5. TTL 推荐值

| 数据类型 | TTL | 说明 |
|----------|-----|------|
| 实时状态 | 30秒 | 需要最新数据 |
| 监控数据 | 60秒 | 允许一定延迟 |
| 业务数据 | 300秒 | 5分钟更新 |
| 配置信息 | 3600秒 | 1小时更新 |

### 6. 缓存失效策略

```javascript
// 主动失效
async function invalidateCache(pattern) {
  const keys = await redis.keys(pattern);
  if (keys.length > 0) {
    await redis.del(...keys);
  }
}

// 更新后调用
await invalidateCache('dashboard:*');
```

## 注意事项

1. **连接失败降级**: Redis 连不上时降级到直接查询
2. **内存监控**: 定期检查 Redis 内存使用
3. **键过期**: 设置 TTL 防止无限增长
4. **序列化**: 大对象压缩后再存储

## 相关文件

- Dashboard: `/root/ai-team/dashboard/server.js`
