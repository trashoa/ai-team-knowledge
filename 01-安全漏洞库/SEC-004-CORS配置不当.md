---
id: SEC-004
category: 安全漏洞
severity: medium
tags: [CORS, 跨域, 资源共享, 安全配置]
created: 2026-03-22
author: 光枢
related_issues: []
status: active
---

# CORS 配置不当

## 问题描述

CORS（跨域资源共享）配置过于宽松，允许任意来源访问。

## 影响范围

- API 被恶意网站调用
- 敏感数据泄露
- CSRF 攻击风险增加

## 错误示例

```javascript
// ❌ 错误：允许所有来源
app.use(cors());

// ❌ 错误：使用通配符
app.use(cors({
  origin: '*'
}));
```

## 正确做法

```javascript
// ✅ 正确：明确指定允许的来源
app.use(cors({
  origin: ['https://example.com', 'https://app.example.com'],
  credentials: true
}));

// ✅ 正确：使用环境变量
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS.split(','),
  credentials: true
}));
```

## Express CORS 中间件配置

```javascript
const corsOptions = {
  origin: function (origin, callback) {
    const allowedOrigins = [
      'https://example.com',
      'https://admin.example.com'
    ];
    // 允许同源请求或白名单来源
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'],
  credentials: true,
  maxAge: 86400 // 24 小时
};

app.use(cors(corsOptions));
```

## 审查检查点

1. [ ] origin 是否明确指定，非 `*`
2. [ ] credentials 是否与 origin 匹配
3. [ ] 是否限制允许的 HTTP 方法
4. [ ] 是否暴露必要的响应头

## 参考资料

- [MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [OWASP CORS Security](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
