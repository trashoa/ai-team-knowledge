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
- 代码公开后风险极大

## 错误示例

```javascript
// ❌ 错误：硬编码 API Key
const API_KEY = "sk-1234567890abcdef";
const DB_PASSWORD = "admin123";
const SECRET_TOKEN = "my-secret-token";
```

```python
# ❌ 错误：硬编码密码
DB_PASSWORD = "admin123"
API_SECRET = "abc123xyz"
```

## 正确做法

```javascript
// ✅ 正确：使用环境变量
const API_KEY = process.env.API_KEY;
const DB_PASSWORD = process.env.DB_PASSWORD;

// ✅ 正确：使用配置文件
const config = require('./config');
const key = config.get('apiKey');
```

```python
# ✅ 正确：使用环境变量
import os

DB_PASSWORD = os.getenv('DB_PASSWORD')
API_SECRET = os.getenv('API_SECRET')
```

## 检测方法

### 1. 关键词搜索

搜索以下关键词：
- password, passwd, pwd
- token, api_key, apikey
- secret, private_key
- credential, auth

### 2. 工具检测

```bash
# ESLint 插件
npm install eslint-plugin-no-secrets

# TruffleHog（Git 历史）
trufflehog git file://./ --only-verified
```

### 3. 人工审查

在代码审查时特别关注配置文件、初始化代码。

## 修复方案

### 1. 使用环境变量

```bash
# .env 文件（不提交到 Git）
API_KEY=your-api-key-here
DB_PASSWORD=your-password-here
```

```bash
# .gitignore
.env
.env.local
```

### 2. 使用密钥管理服务

- AWS Secrets Manager
- Azure Key Vault
- HashiCorp Vault

### 3. 使用配置中心

- Consul
- etcd
- Spring Cloud Config

## 参考资料

- [OWASP Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [12-Factor App - Config](https://12factor.net/config)
- [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning)
