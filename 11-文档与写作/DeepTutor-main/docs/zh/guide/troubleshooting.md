# 常见问题

快速解决常见问题。

## 启动问题

| 问题 | 解决方案 |
|:--------|:---------|
| 后端启动失败 | 检查 Python ≥ 3.11，验证 `.env` 配置 |
| `npm: command not found` | 安装 Node.js: `conda install -c conda-forge nodejs` |
| 端口已被占用 | 终止进程：`lsof -i :8001` → `kill -9 <PID>` |

## 连接问题

| 问题 | 解决方案 |
|:--------|:---------|
| 前端无法连接后端 | 确认后端运行在 <http://localhost:8001/docs> |
| WebSocket 连接失败 | 检查防火墙，确认 `ws://localhost:8001/api/v1/...` 格式 |
| 远程访问失败 | 在 `.env` 中设置 `NEXT_PUBLIC_API_BASE=http://your-ip:8001` |

## Docker 问题

| 问题 | 解决方案 |
|:--------|:---------|
| 云端前端无法连接 | 设置 `NEXT_PUBLIC_API_BASE_EXTERNAL=https://your-server:8001` |
| 架构不匹配 | 使用 `uname -m` 检查：AMD64 用 `:latest`，ARM 用 `:latest-arm64` |

## 知识库问题

| 问题 | 解决方案 |
|:--------|:---------|
| 处理卡住 | 检查终端日志，验证 API 密钥 |
| `uvloop.Loop` 错误 | 该编号条目提取流程已移除（llamaindex-only），无需额外处理 |

## 终止后台进程

```bash
# macOS/Linux
lsof -i :8001 && kill -9 <PID>

# Windows
netstat -ano | findstr :8001
taskkill /PID <PID> /F
```

---

📖 **完整 FAQ**: [GitHub README](https://github.com/HKUDS/DeepTutor#-faq)
