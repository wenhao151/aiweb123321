# 模型接入验证证据

> 采集时间：2026-11-15 15:52（北京时间）
> 采集方式：本机重启后端后调用公开接口，响应原样记录。
> **本文件不含任何 API Key、请求头、手机号或个人信息**，可安全放入技术文档与演示视频。

## 1. 健康检查 · `GET /api/health`

```json
{
  "status": "ok",
  "name": "知实 AI",
  "version": "0.1.0",
  "env": "dev",
  "demo_mode": false,
  "llm_configured": true,
  "llm_model": "deepseek-chat",
  "embed_provider": "hash",
  "embed_model": "bge-m3",
  "embed_dim": 1024,
  "knowledge_base": {
    "status": "ready",
    "documents": 4,
    "chunks": 15
  },
  "server_time": "2026-11-15T07:52:13.891486Z"
}
```

**关键结论**：`demo_mode:false`、`llm_configured:true` —— 已退出演示模式，真实大模型接入生效；知识库 4 篇文档 / 15 个片段就绪。（`server_time` 为 UTC。）

## 2. 真实问答 · `POST /api/teaching/hint`

请求体：

```json
{
  "question": "TypeError: can only concatenate str (not \"int\") to str 怎么改？",
  "target_level": 1
}
```

| 指标 | 实测结果 |
|---|---|
| HTTP 状态 | `200` |
| 响应耗时 | 4.86 秒 |
| `llm_status` | `"model"` |
| 问题分类 | 编程报错（`category: error`，置信度 0.95） |
| 返回层级 | L1 知识点提示 |

`llm_status:"model"` 表明该次分层提示由真实模型生成；若为 `"template"` 表示仍走模板，`"fallback"` 表示模型调用失败后自动回退模板。项目内置的失败回退机制保证现场 API 异常时页面仍可用。

## 3. 复现命令（Git Bash）

```bash
curl -s http://127.0.0.1:8000/api/health

curl -s -X POST http://127.0.0.1:8000/api/teaching/hint \
  -H "Content-Type: application/json" \
  -d '{"question":"TypeError: can only concatenate str (not \"int\") to str 怎么改？","target_level":1}'
```

## 4. 口径说明

- `embed_dim:1024` 是配置文件为将来接语义 Embedding 预留的默认值；当前实际使用的离线哈希向量为 **512 维**（见 `backend/app/library/embeddings.py`），检索不依赖 GPU 或外网。
- 密钥仅存于本机 `.env`，已被 `.gitignore` 排除，未进入仓库；`/api/health` 只返回布尔状态，不回显密钥。
- 现场若 API 超时或欠费，把 `.env` 的 `DEMO_MODE` 设为 `on` 并重启后端即可一键回退演示模式，检索、引用与分层策略不受影响。
