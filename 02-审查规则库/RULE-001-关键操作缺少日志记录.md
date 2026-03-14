---
id: RULE-001
category: 审查规则
severity: medium
tags: [日志, 可观测性, 调试, 最佳实践, 运维]
created: 2026-03-14
author: 大黑手
related_issues: [35]
status: active
---

# 关键操作缺少日志记录

## 问题描述

代码中的关键操作（API 调用、缓存操作、WebSocket 连接等）没有日志记录，导致无法排查问题。

## 影响范围

- 无法追踪问题根因
- 难以调试生产环境
- 无法分析系统行为
- 运维困难

## 错误示例

```javascript
// ❌ 错误：没有日志记录
app.post('/api/webhook', (req, res) => {
  const data = req.body;
  processWebhook(data);
  res.json({ success: true });
});

// ❌ 错误：缓存操作无日志
const data = cache.get(key);
if (!data) {
  const newData = await fetchData();
  cache.set(key, newData);
}

// ❌ 错误：WebSocket 无日志
wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    handleMessage(message);
  });
});
```

## 正确做法

```javascript
// ✅ 正确：API 调用日志
const logger = require('./logger');

app.post('/api/webhook', (req, res) => {
  const startTime = Date.now();
  const data = req.body;
  
  logger.info('Webhook received', {
    endpoint: '/api/webhook',
    action: data.action,
    id: data.id,
    ip: req.ip,
    timestamp: new Date().toISOString()
  });
  
  try {
    processWebhook(data);
    
    logger.info('Webhook processed', {
      endpoint: '/api/webhook',
      action: data.action,
      duration: Date.now() - startTime,
      status: 'success'
    });
    
    res.json({ success: true });
  } catch (error) {
    logger.error('Webhook processing failed', {
      endpoint: '/api/webhook',
      action: data.action,
      error: error.message,
      stack: error.stack,
      duration: Date.now() - startTime
    });
    
    res.status(500).json({ error: 'Processing failed' });
  }
});
```

```javascript
// ✅ 正确：缓存操作日志
const data = cache.get(key);

if (data) {
  logger.debug('Cache hit', { key, ttl: cache.getTtl(key) });
} else {
  logger.info('Cache miss', { key });
  
  const newData = await fetchData();
  cache.set(key, newData, 300);
  
  logger.info('Cache set', { key, ttl: 300, size: JSON.stringify(newData).length });
}
```

```javascript
// ✅ 正确：WebSocket 日志
wss.on('connection', (ws, req) => {
  const clientId = generateClientId();
  
  logger.info('WebSocket connected', {
    clientId,
    ip: req.socket.remoteAddress,
    userAgent: req.headers['user-agent'],
    timestamp: new Date().toISOString()
  });
  
  ws.on('message', (message) => {
    logger.debug('WebSocket message received', {
      clientId,
      messageSize: message.length,
      type: typeof message
    });
    
    try {
      handleMessage(message);
    } catch (error) {
      logger.error('WebSocket message handling failed', {
        clientId,
        error: error.message,
        stack: error.stack
      });
    }
  });
  
  ws.on('close', (code, reason) => {
    logger.info('WebSocket disconnected', {
      clientId,
      code,
      reason: reason.toString(),
      duration: Date.now() - connectionStartTime
    });
  });
  
  ws.on('error', (error) => {
    logger.error('WebSocket error', {
      clientId,
      error: error.message,
      stack: error.stack
    });
  });
});
```

## 日志级别规范

| 级别 | 使用场景 | 示例 |
|------|----------|------|
| ERROR | 系统错误，需要立即处理 | API 调用失败、数据库连接失败 |
| WARN | 警告，需要注意但不一定立即处理 | 缓存过期、重试次数过多 |
| INFO | 重要业务事件 | 用户登录、订单创建、API 调用成功 |
| DEBUG | 调试信息 | 函数调用、变量值、缓存命中/未命中 |

## 必须记录的操作

### 1. API 调用

```javascript
{
  level: 'info',
  message: 'API call completed',
  endpoint: '/api/users',
  method: 'GET',
  statusCode: 200,
  duration: 123,
  requestId: 'uuid',
  userId: 'user-123'
}
```

### 2. 缓存操作

```javascript
{
  level: 'debug',
  message: 'Cache operation',
  operation: 'get|set|del',
  key: 'user:123',
  hit: true|false,
  ttl: 300
}
```

### 3. WebSocket 事件

```javascript
{
  level: 'info',
  message: 'WebSocket event',
  event: 'connect|disconnect|message|error',
  clientId: 'client-uuid',
  ip: '192.168.1.1'
}
```

### 4. 外部服务调用

```javascript
{
  level: 'info',
  message: 'External service call',
  service: 'github|slack|feishu',
  endpoint: '/api/v1/messages',
  status: 'success|failed',
  duration: 456
}
```

## 检测方法

### 1. 代码审查检查清单

- [ ] 所有 API 端点有日志
- [ ] 缓存操作有日志
- [ ] WebSocket 事件有日志
- [ ] 外部服务调用有日志
- [ ] 错误处理有日志

### 2. 静态分析工具

```javascript
// ESLint 规则示例
{
  "rules": {
    "no-console": ["warn", { "allow": ["error"] }],
    "require-logger": "error"
  }
}
```

### 3. 运行时检查

```javascript
// 检查日志覆盖率
const winston = require('winston');

winston.add(new winston.transports.Console({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  )
}));

// 验证日志输出
logger.info('Test message', { test: true });
// 检查输出是否包含 timestamp, level, message
```

## 修复方案

### 1. 统一日志库

```javascript
// 使用 winston 或 pino
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'ai-team' },
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

module.exports = logger;
```

### 2. 日志中间件

```javascript
// Express 日志中间件
function requestLogger(req, res, next) {
  const start = Date.now();
  
  res.on('finish', () => {
    logger.info('HTTP Request', {
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration: Date.now() - start,
      ip: req.ip,
      userAgent: req.get('user-agent')
    });
  });
  
  next();
}

app.use(requestLogger);
```

### 3. 日志轮转

```javascript
// 使用 winston-daily-rotate-file
const DailyRotateFile = require('winston-daily-rotate-file');

const transport = new DailyRotateFile({
  filename: 'application-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  zippedArchive: true,
  maxSize: '20m',
  maxFiles: '14d'
});
```

## 参考资料

- [Winston Logger](https://github.com/winstonjs/winston)
- [Pino Logger](https://github.com/pinojs/pino)
- [12-Factor App Logs](https://12factor.net/logs)
- [Structured Logging](https://stackify.com/what-is-structured-logging-and-why-developers-need-it/)
