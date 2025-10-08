# sys-rag

一个面向生产的 RAG（Retrieval-Augmented Generation）系统与管理平台：统一管理数据源接入、文档切分清洗、向量化索引、检索与重排、模型调用、评估与观测，支持多向量库与多大模型适配，提供标准化 API，便于扩展与编排。

> 当前仓库处于初始化阶段：本 README 提供完整的架构与实现规划、配置项与 API 草案，便于快速落地代码与协作开发。


## 功能特性

- 数据接入与清洗
	- 支持本地文件夹/单文件、HTTP/网页爬取、Git 仓库、对象存储（规划）
	- 常见文档格式：Markdown、TXT、PDF、HTML（更多格式可通过适配器扩展）
	- 文档切分策略：长度窗口、语义切分、分层目录切分等

- 向量化与索引
	- 嵌入模型：OpenAI、Azure OpenAI、Qwen-Embedding、bge、E5、Jina 等（可插拔）
	- 向量库：FAISS、Chroma、Milvus（优先支持 FAISS/Chroma，Milvus 作为企业级选项）
	- 元数据索引：存储源信息、页码、标题、分区等，便于高召回与可追溯

- 检索与重排
	- 检索策略：Dense/向量检索、BM25、Hybrid、MMR、多路召回
	- 重排模型：bge-reranker、Cohere-Rerank 等（可选）
	- 上下文组织：去重、聚类、引用标注、片段合并

- 生成与模板
	- 模型适配：OpenAI、Azure OpenAI、Ollama（本地 LLM）、阿里通义/Qwen（规划）
	- 提示词模板：可参数化、可版本化，支持系统/用户/工具多段模板
	- 可选工具：函数调用、结构化输出（JSON Schema）

- 评估与观测
	- 检索评估：Recall@k、MRR、覆盖率、冗余率
	- 生成评估：答案忠实度、事实一致性、引用完整性（集成 RAGAS 规划）
	- 观测追踪：OpenTelemetry/Langfuse（可选），请求-检索-重排-生成全链路 trace

- 管理与编排
	- 管道定义：以 JSON/YAML 声明式配置，快速复用与迁移
	- 任务管理：异步任务、进度与日志、重试与审计
	- 插件机制：Loader/Embedder/VectorStore/Retriever/Reranker/LLM Adapter 可插拔


## 架构总览

Ingestion -> Chunking/Cleaning -> Embedding -> Indexing -> Retrieval -> Reranking -> Prompting -> Generation -> Evaluation -> Observability

数据流核心：

1) 数据接入（Loader）→ 2) 文档切分与清洗（Chunker/Cleaner）→ 3) 向量化（Embedder）→ 4) 写入向量库（Vector Store）→ 5) 查询检索（Retriever）→ 6) 重排（Reranker）→ 7) 模型生成（LLM Adapter + Prompt）→ 8) 结果与引用返回 → 9) 评估与观测


## 目录结构（规划草案）

本仓库将逐步演进到下述结构，便于清晰分层与可扩展：

```
sys-rag/
├─ backend/
│  ├─ pyproject.toml or requirements.txt
│  ├─ src/
│  │  ├─ app/
│  │  │  ├─ main.py                # FastAPI 入口，路由注册
│  │  │  ├─ deps.py                # 依赖注入、配置、日志
│  │  │  └─ routers/               # 路由模块（ingest/index/query/eval/health）
│  │  ├─ core/
│  │  │  ├─ config.py              # 配置模型与校验
│  │  │  ├─ logging.py             # 日志与追踪
│  │  │  └─ tasks.py               # 异步任务抽象（RQ/Celery/Arq 其一）
│  │  ├─ datasources/              # Loader 适配器
│  │  ├─ chunking/                 # 切分策略
│  │  ├─ embeddings/               # 嵌入模型适配
│  │  ├─ vectorstores/             # 向量库适配（faiss/chroma/milvus）
│  │  ├─ retrievers/               # 检索策略（dense/bm25/hybrid/mmr）
│  │  ├─ rerankers/                # 重排模型
│  │  ├─ llms/                     # LLM 适配（openai/azure/ollama/...）
│  │  ├─ pipelines/                # 管道定义与执行器
│  │  ├─ evaluators/               # 评估指标与数据集
│  │  └─ utils/                    # 通用工具
│  ├─ tests/                       # 单元与集成测试
│  └─ scripts/                     # 维护脚本（构建、数据准备、基准）
├─ docs/                           # 设计文档、API 说明、用例
├─ examples/                       # 示例数据与示例 pipeline
├─ docker/                         # Dockerfile、docker-compose.yml（规划）
├─ .env.example                    # 环境变量示例（规划）
└─ README.md
```


## 快速开始（预览）

代码初始化即将开始。你可以先准备基础环境：

- Python 3.11+
- 包管理：pip/uv/Poetry 其一
- 可选：Ollama（本地模型）、Milvus/Chroma/FAISS 运行环境

待代码落地后，将提供以下便捷命令：

```bash
# 安装依赖（示例，具体以实际落地为准）
pip install -r backend/requirements.txt

# 启动 API（开发模式）
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 运行测试
pytest -q
```


## 配置

在项目根目录创建 `.env`（或使用系统环境变量）。以下为推荐字段：

```dotenv
# 基本信息
ENV=dev
LOG_LEVEL=INFO

# LLM 提供商（按需其一或多选）
LLM_PROVIDER=openai            # openai | azure | ollama | qwen（规划）
OPENAI_API_KEY=
AZURE_OPENAI_API_KEY=
AZURE_OPENAI_ENDPOINT=
OLLAMA_BASE_URL=http://localhost:11434

# 嵌入模型
EMBEDDING_MODEL=openai:text-embedding-3-large
EMBEDDING_DIM=3072

# 向量库
VECTOR_STORE=faiss             # faiss | chroma | milvus
VECTOR_DIR=./storage/vector
CHROMA_PERSIST_DIR=./storage/chroma
MILVUS_URI=

# 数据与缓存
DATA_DIR=./data
CACHE_DIR=./.cache

# 观测与追踪（可选）
ENABLE_TRACING=false
OTEL_EXPORTER_OTLP_ENDPOINT=
LANGFUSE_PUBLIC_KEY=
LANGFUSE_SECRET_KEY=
LANGFUSE_HOST=
```


## API 设计草案

统一前缀：`/api/v1`

1) 数据接入（异步任务）

- POST `/ingest`
	- 请求体：
		```json
		{
			"source_type": "local|web|git|s3",
			"uri": "./examples/docs",
			"loader_options": {"glob": "**/*.md"},
			"dataset_name": "docs",
			"metadata": {"project": "sys-rag"}
		}
		```
	- 响应：`{"task_id":"...","dataset_id":"..."}`

2) 构建索引（异步任务）

- POST `/index`
	- 请求体：
		```json
		{
			"dataset_id": "...",
			"chunking": {"strategy": "window", "size": 1000, "overlap": 200},
			"embedding": {"model": "openai:text-embedding-3-large"},
			"vector_store": {"type": "faiss", "path": "./storage/vector/docs"}
		}
		```
	- 响应：`{"task_id":"...","index_id":"..."}`

3) 查询（同步）

- POST `/query`
	- 请求体：
		```json
		{
			"query": "sys-rag 的核心功能有哪些？",
			"index_id": "...",
			"retrieval": {"top_k": 8, "strategy": "hybrid", "mmr": true},
			"reranker": {"name": "bge-reranker", "top_n": 5},
			"generation": {"provider": "openai", "model": "gpt-4o-mini"},
			"prompt": {"template": "你是知识库助手...", "variables": {"style": "concise"}}
		}
		```
	- 响应示例（精简）：
		```json
		{
			"answer": "sys-rag 提供数据接入、切分清洗、向量索引、检索重排、生成、评估与观测等能力...",
			"contexts": [
				{"text": "...片段1...", "score": 0.89, "metadata": {"path": "..."}},
				{"text": "...片段2...", "score": 0.85, "metadata": {"path": "..."}}
			],
			"trace_id": "..."
		}
		```

4) 任务状态

- GET `/tasks/{task_id}` → `{"status":"queued|running|succeeded|failed","progress":0.42}`

5) 评估（异步/同步混合）

- POST `/eval`
	- 请求体：
		```json
		{
			"dataset_id": "...",
			"eval_set": [{"query": "...", "ground_truth": "..."}],
			"metrics": ["recall@5", "faithfulness"],
			"config": {"llm": {"provider": "openai", "model": "gpt-4o-mini"}}
		}
		```
	- 响应：
		```json
		{
			"report": {"recall@5": 0.78, "faithfulness": 0.92},
			"runs": [
				{"query": "...", "recall@5": 1, "faithfulness": 0.9}
			]
		}
		```

6) 健康与版本

- GET `/health` → `{"status":"ok"}`
- GET `/version` → `{"version":"0.1.0"}`


## cURL 示例（参考）

```bash
# 1) 接入数据
curl -X POST http://localhost:8000/api/v1/ingest \
	-H "Content-Type: application/json" \
	-d '{
		"source_type": "local",
		"uri": "./examples/docs",
		"loader_options": {"glob": "**/*.md"},
		"dataset_name": "docs"
	}'

# 2) 构建索引
curl -X POST http://localhost:8000/api/v1/index \
	-H "Content-Type: application/json" \
	-d '{
		"dataset_id": "<dataset-id>",
		"chunking": {"strategy": "window", "size": 1000, "overlap": 200},
		"embedding": {"model": "openai:text-embedding-3-large"},
		"vector_store": {"type": "faiss", "path": "./storage/vector/docs"}
	}'

# 3) 查询
curl -X POST http://localhost:8000/api/v1/query \
	-H "Content-Type: application/json" \
	-d '{
		"query": "sys-rag 的核心功能有哪些？",
		"index_id": "<index-id>",
		"retrieval": {"top_k": 8, "strategy": "hybrid", "mmr": true},
		"generation": {"provider": "openai", "model": "gpt-4o-mini"}
	}'
```


## 开发与测试（规划）

- 语言与框架：Python 3.11+、FastAPI、Pydantic、Uvicorn
- 数据处理：sentence-transformers、rank-bm25、ragas（评估）、unstructured（解析可选）
- 向量库：faiss-cpu、chromadb、pymilvus（按需）
- 观测：opentelemetry-sdk、langfuse（可选）
- 测试：pytest、httpx

CI 方向（后续补充）：
- Lint/Type：ruff 或 flake8 + mypy
- Test：pytest -q，覆盖核心路径
- 构建：Docker 镜像（多阶段），缓存模型与索引目录


## Roadmap

- [ ] v0.1 MVP：
	- [ ] FastAPI 基础骨架与路由：`/health`、`/ingest`、`/index`、`/query`
	- [ ] 本地文件 Loader + 简单切分（window）
	- [ ] OpenAI 嵌入 + FAISS 索引
	- [ ] Dense/Hybrid 检索 + 简单 MMR
	- [ ] OpenAI 生成 + 基础 Prompt 模板
	- [ ] 任务状态轮询 + 简单日志

- [ ] v0.2 可观测与评估：
	- [ ] RAGAS 集成（faithfulness、answer correctness）
	- [ ] Langfuse/OTel Trace 打通

- [ ] v0.3 扩展生态：
	- [ ] Reranker（bge-reranker、Cohere-Rerank）
	- [ ] Chroma/Milvus 适配与数据迁移
	- [ ] Ollama 本地 LLM 适配

- [ ] v0.4 管道与前端：
	- [ ] Pipeline JSON/YAML 配置与可视化
	- [ ] 轻量 Web 控制台（检索观察、配置与评估）


## 贡献指南

欢迎 Issue/PR！建议：

- 讨论前先查看 Roadmap 与开放议题
- 采用小步 PR，附上复现步骤与结果说明
- 代码风格与检查（后续在仓库中提供 ruff/mypy 配置）
- 提交信息建议：`feat: ...`、`fix: ...`、`docs: ...`、`refactor: ...`


## 许可协议

MIT License（拟）。在首次可用版本发布前保留变更权利。


## 常见问题 FAQ（进行中）

- 支持哪些文件格式？
	- 初期：md/txt/pdf/html，更多格式通过适配器扩展
- 是否必须联网？
	- 取决于所选模型与向量库；本地方案可选：Ollama + FAISS/Chroma
- 如何提升召回质量？
	- 合理切分、Hybrid 检索、MMR、Reranker、元数据过滤
- 如何降低成本？
	- 预归一与去重、缓存与重用、检索阈值调整、上下文压缩


## 致谢与生态

本项目理念与实现将参考并兼容以下生态：
- LangChain / LlamaIndex（管道与适配器思想）
- sentence-transformers / Jina / bge / E5（嵌入与重排）
- FAISS / Chroma / Milvus（向量库）
- RAGAS（评估）
- OpenTelemetry / Langfuse（可观测）


---

下一步建议：
- 初始化 `backend/` 与 FastAPI 骨架，提交 `.env.example` 与基础依赖
- 落地 `/health`、`/ingest`、`/index`、`/query` 四个路由的最小实现
- 提供 `examples/` 演示数据与端到端示例脚本