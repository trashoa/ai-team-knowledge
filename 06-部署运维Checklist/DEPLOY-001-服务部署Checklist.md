---
id: DEPLOY-001
category: 部署运维Checklist
severity: high
tags: [部署, 运维, Docker, 检查清单]
created: 2026-03-22
author: 光枢
related_issues: []
status: active
---

# 服务部署 Checklist

## 部署前检查

- [ ] 代码已提交到 Git 仓库
- [ ] 所有测试通过
- [ ] 无硬编码密码/Token
- [ ] 环境变量配置完成
- [ ] 防火墙端口已开放
- [ ] 磁盘空间充足 (>20%)

## Docker 部署

### 1. 构建镜像

```bash
# 构建最新镜像
docker build -t ai-team-{name}:latest .

# 验证镜像
docker images | grep ai-team-{name}
```

### 2. 运行容器

```bash
# 后台运行
docker run -d \
  --name ai-team-{name} \
  -p {port}:{port} \
  -v /root/ai-team/.env:/app/.env \
  ai-team-{name}:latest

# 检查运行状态
docker ps | grep ai-team-{name}
```

### 3. 日志检查

```bash
# 查看实时日志
docker logs -f ai-team-{name}

# 查看最近 100 行
docker logs --tail 100 ai-team-{name}
```

## 服务验证

### 1. 健康检查

```bash
# API 健康检查
curl http://localhost:{port}/health

# 依赖服务检查
redis-cli ping
docker ps | grep {service}
```

### 2. 功能测试

- [ ] 核心 API 响应正常
- [ ] 文件读写权限正确
- [ ] 日志正常输出
- [ ] 定时任务正常运行

## 部署后验证

- [ ] 服务启动成功（无报错）
- [ ] 端口监听正常
- [ ] 日志无异常
- [ ] 外部可访问
- [ ] 通知渠道正常

## 回滚方案

```bash
# 查看历史版本
docker images | grep ai-team-{name}

# 回滚到上一个版本
docker stop ai-team-{name}
docker rm ai-team-{name}
docker run -d \
  --name ai-team-{name} \
  -p {port}:{port} \
  ai-team-{name}:previous-tag
```

## 常用命令

| 操作 | 命令 |
|------|------|
| 查看日志 | `docker logs -f ai-team-{name}` |
| 重启服务 | `docker restart ai-team-{name}` |
| 查看状态 | `docker ps \| grep {name}` |
| 进入容器 | `docker exec -it {name} bash` |
| 查看资源 | `docker stats {name}` |
