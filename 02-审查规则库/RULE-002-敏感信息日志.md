---
id: RULE-002
category: 代码审查规则
severity: high
tags: [日志, 敏感信息, 隐私, 审计]
created: 2026-03-22
author: 光枢
related_issues: []
status: active
---

# 敏感信息日志记录规则

## 规则说明

日志中禁止记录敏感信息，防止信息泄露。

## 敏感信息类型

| 类型 | 示例 |
|------|------|
| 认证凭证 | password, token, API key, session |
| 个人隐私 | 身份证, 手机号, 地址, 银行卡 |
| 业务数据 | 订单号, 金额, 交易明细 |
| 系统信息 | IP, MAC, 服务器路径 |

## 错误示例

```javascript
// ❌ 错误：记录完整用户信息
logger.info(`用户登录: ${user}`); // user 包含 password

// ❌ 错误：记录请求参数
logger.debug(`请求参数: ${JSON.stringify(req.body)}`); // 包含敏感字段

// ❌ 错误：记录错误堆栈（可能包含敏感信息）
logger.error(err); // err.message 可能包含敏感路径
```

## 正确做法

```javascript
// ✅ 正确：日志脱敏
logger.info(`用户登录: ${userId}`);

// ✅ 正确：过滤敏感字段
const safeParams = filterSensitiveFields(req.body);
logger.debug(`请求参数: ${JSON.stringify(safeParams)}`);

// ✅ 正确：自定义错误消息
logger.error(`处理失败: ${err.message}`);
```

## 过滤函数示例

```javascript
const SENSITIVE_FIELDS = ['password', 'token', 'apiKey', 'secret', 'creditCard'];

function filterSensitiveFields(obj) {
  const filtered = { ...obj };
  for (const field of SENSITIVE_FIELDS) {
    if (filtered[field]) {
      filtered[field] = '***';
    }
  }
  return filtered;
}
```

## 审查检查点

1. [ ] 日志中是否记录了 password/token/apiKey
2. [ ] 错误堆栈是否包含敏感路径
3. [ ] 请求/响应日志是否过滤了敏感字段
4. [ ] 数据库查询日志是否脱敏

## 参考资料

- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [PCI DSS Requirement 10](https://www.pcisecuritystandards.org/)
