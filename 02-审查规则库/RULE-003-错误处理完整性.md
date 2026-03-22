---
id: RULE-003
category: 代码审查规则
severity: medium
tags: [错误处理, 异常, 防御性编程, 健壮性]
created: 2026-03-22
author: 光枢
related_issues: []
status: active
---

# 错误处理完整性规则

## 规则说明

所有可能失败的操作都必须有完整的错误处理。

## 错误示例

```javascript
// ❌ 错误：没有错误处理
const data = JSON.parse(userInput);

// ❌ 错误：吞掉错误
try {
  doSomething();
} catch (e) {
  // 什么都不做
}

// ❌ 错误：只记录不停留
fetch(url)
  .then(data => process(data))
  .catch(err => console.log(err)); // 没有抛出或返回错误状态
```

## 正确做法

```javascript
// ✅ 正确：try-catch 并处理
let data;
try {
  data = JSON.parse(userInput);
} catch (e) {
  return res.status(400).json({ error: 'Invalid JSON' });
}

// ✅ 正确：明确错误处理
async function fetchData(url) {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    logger.error(`Fetch failed: ${error.message}`);
    throw error; // 重新抛出或返回错误状态
  }
}

// ✅ 正确：使用 Result 模式
function safeParse(json) {
  try {
    return { success: true, data: JSON.parse(json) };
  } catch (e) {
    return { success: false, error: e.message };
  }
}
```

## 审查检查点

1. [ ] 所有 async 函数是否有 try-catch
2. [ ] JSON.parse 是否有错误处理
3. [ ] 网络请求是否检查 response.ok
4. [ ] 错误是否有适当的日志记录
5. [ ] 错误是否返回适当的 HTTP 状态码
6. [ ] 错误消息是否对用户友好

## HTTP 状态码参考

| 状态码 | 场景 |
|--------|------|
| 400 | 请求参数错误 |
| 401 | 未认证 |
| 403 | 未授权 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |
| 503 | 服务不可用 |

## 参考资料

- [MDN Error Handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Statements)
- [Node.js Error Handling](https://nodejs.org/api/errors.html)
