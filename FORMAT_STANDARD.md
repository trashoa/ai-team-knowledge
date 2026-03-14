# 📚 AI Team 知识库格式标准

> 版本: v1.0  
> 日期: 2026-03-14  
> 维护: 光枢 🦞

---

## 📋 目录结构

```
知识库/
├── 01-安全漏洞库/
│   ├── 硬编码密码/
│   ├── SQL注入/
│   ├── XSS跨站脚本/
│   ├── 未授权访问/
│   └── ...
├── 02-审查规则库/
│   ├── 代码规范/
│   ├── 安全检查/
│   ├── 性能优化/
│   └── ...
├── 03-修复方案库/
│   ├── 前端修复/
│   ├── 后端修复/
│   ├── 安全修复/
│   └── ...
└── 04-常见问题库/
    ├── 配置问题/
    ├── 部署问题/
    └── ...
```

---

## 📝 知识条目模板

### 基础信息（YAML 头）

```yaml
---
id: SEC-001
category: 安全漏洞
severity: critical | high | medium | low
tags: [关键词1, 关键词2, 关键词3]
created: 2026-03-14
updated: 2026-03-14
author: 大黑手
related_issues: [35, 39]
status: active | deprecated
---
```

### 内容结构

```markdown
# 标题（问题名称）

## 问题描述
清晰简洁地描述这个问题是什么。

## 影响范围
这个问题会造成什么影响？

## 错误示例
展示错误的代码示例。

## 正确做法
展示正确的代码示例。

## 检测方法
如何检测这个问题？（工具、方法）

## 修复方案
具体的修复步骤。

## 参考资料
相关的文档、链接。
```

---

## 🏷️ 分类标签

### 按严重程度 (severity)

| 标签 | 说明 |
|------|------|
| critical | 严重，必须立即修复 |
| high | 高危，建议尽快修复 |
| medium | 中等，常规修复 |
| low | 低危，可选修复 |

### 按分类 (category)

| 分类 | 说明 |
|------|------|
| 安全漏洞 | 安全相关问题 |
| 代码规范 | 代码风格、格式 |
| 性能优化 | 性能相关 |
| 架构设计 | 架构问题 |
| 配置问题 | 部署、配置 |
| 常见问题 | 常见错误 |

### 按状态 (status)

| 状态 | 说明 |
|------|------|
| active | 有效 |
| deprecated | 已过时 |

---

## 📌 示例：硬编码密码

```yaml
---
id: SEC-001
category: 安全漏洞
severity: critical
tags: [硬编码, 密码, Token, 认证, API密钥]
created: 2026-03-14
author: 大黑手
related_issues: [35, 39]
status: active
---

# 硬编码密码/Token 问题

## 问题描述
在代码中直接写入密码、API Token、密钥等敏感信息。

## 影响范围
- 泄露敏感凭证
- 可能被恶意利用
- 安全审计不通过

## 错误示例
```javascript
// ❌ 错误
const API_KEY = "sk-1234567890abcdef";
const PASSWORD = "admin123";
```

## 正确做法
```javascript
// ✅ 正确
const API_KEY = process.env.API_KEY;
const PASSWORD = process.env.DB_PASSWORD;

// 或使用配置中心
const config = require('./config');
const key = config.get('apiKey');
```

## 检测方法
1. 搜索关键词：password, token, secret, key, api_key
2. 使用 ESLint 插件：eslint-plugin-no-secrets
3. 人工代码审查

## 修复方案
1. 将敏感信息移到环境变量
2. 使用密钥管理服务
3. 添加 .env 到 .gitignore

## 参考资料
- OWASP Secrets Management
- 12-Factor App
```

---

## 📋 入库检查清单

入库前检查：

- [ ] id 唯一（格式：类型-序号，如 SEC-001）
- [ ] category 正确
- [ ] severity 合理
- [ ] tags 完整（至少3个）
- [ ] created 日期正确
- [ ] related_issues 填写
- [ ] 内容结构完整（问题描述/错误示例/正确做法/检测方法/修复方案）
- [ ] 代码示例已验证

---

## 🔧 工具支持

### 自动检测规则

后续会添加自动化工具检测代码问题。

---

## 📅 更新记录

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-03-14 | v1.0 | 初始版本 |

---

*维护者: 光枢 🦞*
