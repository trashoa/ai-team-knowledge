---
id: SEC-002
category: 安全漏洞
severity: high
tags: [API, 缓存, 错误处理, 数据一致性, 用户体验]
created: 2026-03-14
author: 大黑手
related_issues: [35]
status: active
---

# API 错误处理不完整 - 返回过期缓存数据

## 问题描述

当 GitHub API 请求失败时，系统只返回缓存数据，用户不知道看到的是过期数据。

## 影响范围

- 用户看到过期数据但不知情
- 决策基于错误信息
- 无法及时发现 API 问题
- 数据一致性问题

## 错误示例

```javascript
// ❌ 错误：API 失败时静默返回缓存
try {
  const data = await fetchGitHubData();
  return data;
} catch (error) {
  // 只返回缓存，没有错误提示
  return cache.get('github-data');
}
```

```javascript
// ❌ 错误：没有标记数据状态
app.get('/api/data', async (req, res) => {
  try {
    const data = await fetchFromGitHub();
    res.json(data);
  } catch (error) {
    // 返回缓存，但用户不知道是缓存
    res.json(cache.get('data'));
  }
});
```

## 正确做法

```javascript
// ✅ 正确：API 失败时标记数据状态
app.get('/api/data', async (req, res) => {
  try {
    const data = await fetchFromGitHub();
    cache.set('data', data);
    res.json({
      data,
      source: 'api',
      cached: false,
      timestamp: Date.now()
    });
  } catch (error) {
    // 记录错误
    logger.error('GitHub API failed:', error);
    
    // 返回缓存，但明确标记
    const cachedData = cache.get('data');
    res.json({
      data: cachedData,
      source: 'cache',
      cached: true,
      timestamp: cachedData?.timestamp,
      error: 'API temporarily unavailable, returning cached data'
    });
  }
});
```

```javascript
// ✅ 正确：添加重试机制
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.ok) return await response.json();
    } catch (error) {
      lastError = error;
      logger.warn(`Attempt ${i + 1} failed:`, error.message);
      await delay(1000 * Math.pow(2, i)); // 指数退避
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}
```

## 检测方法

### 1. 代码审查

检查以下模式：
- `catch` 块中直接返回缓存
- 没有错误日志记录
- 没有重试机制

### 2. 单元测试

```javascript
// 测试 API 失败场景
test('should mark data as cached when API fails', async () => {
  mockGitHubAPI.toThrow(new Error('API Error'));
  
  const result = await fetchData();
  
  expect(result.cached).toBe(true);
  expect(result.source).toBe('cache');
});
```

### 3. 集成测试

- 模拟 API 超时
- 验证返回数据结构
- 检查错误日志

## 修复方案

### 1. 添加数据状态标记

```javascript
// 统一响应格式
function createResponse(data, options = {}) {
  return {
    data,
    source: options.source || 'api',
    cached: options.cached || false,
    timestamp: Date.now(),
    error: options.error || null
  };
}
```

### 2. 添加重试机制

```javascript
// 使用 p-retry 库
const pRetry = require('p-retry');

const data = await pRetry(
  () => fetchGitHubData(),
  {
    retries: 3,
    onFailedAttempt: error => {
      logger.warn(`Attempt ${error.attemptNumber} failed`);
    }
  }
);
```

### 3. 添加详细日志

```javascript
// 记录 API 调用详情
logger.info('GitHub API call', {
  endpoint: '/repos/owner/repo/pulls',
  duration: 234,
  status: 'success',
  cached: false
});

logger.error('GitHub API call failed', {
  endpoint: '/repos/owner/repo/pulls',
  error: error.message,
  stack: error.stack,
  fallback: 'cache'
});
```

## 参考资料

- [Retry Pattern - Microsoft](https://docs.microsoft.com/en-us/azure/architecture/patterns/retry)
- [Circuit Breaker Pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)
- [HTTP Cache](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
