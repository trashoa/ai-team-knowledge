---
id: SEC-003
category: 安全漏洞
severity: high
tags: [WebSocket, 认证, 鉴权, 未授权访问]
created: 2026-03-22
author: 光枢
related_issues: [18]
status: active
---

# WebSocket 连接未鉴权

## 问题描述

WebSocket 连接没有进行身份验证，任何人都可以连接并接收数据。

## 影响范围

- 敏感数据泄露
- 恶意用户可获取实时数据
- 可能被用于 DDOS 攻击

## 错误示例

```javascript
// ❌ 错误：WebSocket 无鉴权
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  // 任何人都能连接
  ws.send(JSON.stringify(sensitiveData));
});
```

## 正确做法

```javascript
// ✅ 正确：Token 鉴权
const wss = new WebSocket.Server({
  port: 8080,
  verifyClient: (info, cb) => {
    const token = info.req.headers['sec-websocket-protocol'];
    if (isValidToken(token)) {
      cb(true);
    } else {
      cb(false, 401, 'Unauthorized');
    }
  }
});

wss.on('connection', (ws, req) => {
  const user = authenticate(req);
  ws.user = user;
  ws.send(JSON.stringify(getUserData(user)));
});
```

## 检测方法

1. 无 Token 直接连接测试
2. 使用无效 Token 连接测试
3. 检查是否有 verifyClient 配置

## 修复方案

1. **Token 鉴权** - 连接时验证 Token
2. **Origin 检查** - 验证请求来源
3. **速率限制** - 限制连接频率

## 参考资料

- [WebSocket Security](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [OWASP WebSocket Security](https://owasp.org/www-community/vulnerabilities/)
