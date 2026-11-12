# DeepTutor 技术框架与快速学习路线

> 目标：先知道“这个项目用了哪些框架”，再给出“按目标快速上手”的路线。  
> 版本以当前仓库的 `pyproject.toml`、`requirements/`、`web/package.json`、`apps/desktop/package.json` 为准。

---

## 1. 框架总览（按层）

| 层 | 主要框架/库 | 作用 |
|---|---|---|
| **后端语言/运行时** | Python 3.11–3.14、asyncio、contextvars | 异步 turn 执行、流式事件、租约/协调 |
| **Web API** | FastAPI、Uvicorn、Pydantic v2、WebSocket | HTTP API、`/ws` 命令协议、请求校验 |
| **CLI** | Typer、Rich、prompt_toolkit、pyte | 命令入口、REPL、终端渲染 |
| **LLM 客户端** | OpenAI SDK、Anthropic SDK、DashScope、Perplexity、MCP SDK | 多供应商模型调用、工具协议、OAuth |
| **会话/存储** | SQLite（aiosqlite）、PocketBase、Redis | turn 持久化、事件 journal、租约/命令、可选后端 |
| **RAG/检索** | LlamaIndex、PageIndex、BM25S、FAISS、LightRAG、GraphRAG、sentence-transformers | 知识库索引、检索、重排、图检索 |
| **文档解析** | Docling、MarkItDown、PyMuPDF4LLM、LiteParse、MinerU、Tika | PDF/Office/EPUB/网页解析 |
| **媒体** | Manim、Remotion、FFmpeg、LaTeX、edge-tts | 数学动画、视频课、音频概览 |
| **IM/Partner** | Telegram、Lark/飞书、钉钉、Slack、QQ Bot、WeCom、Zulip、Matrix 等 SDK | 伙伴通道 |
| **前端** | Next.js 16、React 19、TypeScript 5、Tailwind CSS 3、i18next | Web 工作台 |
| **前端富内容** | BlockNote、PDF.js、epub.js、Chart.js、Mermaid、KaTeX、Recogito、Cytoscape、Framer Motion | 笔记、阅读器、图表、标注、图谱、动画 |
| **桌面** | Electron 45、Node 22、@electron/packager | 桌面壳与打包 |
| **测试/质量** | pytest、pytest-asyncio、Ruff、import-linter、mypy、bandit、detect-secrets、pre-commit；Vitest、Testing Library、Playwright、ESLint、dependency-cruiser | Python/前端测试、边界与安全检查 |
| **构建/部署** | setuptools、模块构建脚本、Docker、Compose、GitHub Actions、PyPI wheels、NSIS/DMG | 发行与部署 |

---

## 2. 后端框架

### 2.1 Web / API

| 框架 | 版本/位置 | 在项目中的用途 |
|---|---|---|
| **FastAPI** | `requirements/modules/server.txt` `fastapi>=0.100.0` | `deeptutor/api/main.py` 应用；50 个 router；依赖注入、鉴权、WebSocket |
| **Uvicorn** | `uvicorn[standard]>=0.24.0` | `deeptutor serve` / launcher 启动 API |
| **Pydantic v2** | `pydantic>=2.0.0`、`pydantic-settings>=2.0.0` | `TurnRequest`、能力配置模型、设置模型、请求校验 |
| **WebSocket** | `websockets>=12.0` | `/ws` 统一 turn 命令协议、流式事件、回放 |
| **python-multipart** | `>=0.0.6` | 文件上传 |
| **python-jose / bcrypt** | `>=3.3.0` / `>=4.0.0` | 鉴权/token/密码 |

**关键文件**：`deeptutor/api/main.py`、`api/routers/unified_ws.py`、`api/router_loader.py`、`app/contracts.py`、`runtime/request_contracts.py`。

### 2.2 CLI

| 框架 | 版本 | 用途 |
|---|---|---|
| **Typer** | `>=0.9.0` | `deeptutor` 命令树：`run`、`chat`、`kb`、`module`、`plugin`、`serve`、`start` |
| **Rich** | `>=13.0.0` | 终端 Markdown、表格、状态提示 |
| **prompt_toolkit** | `>=3.0.36` | REPL 输入/补全 |
| **pyte** | `>=0.8.1` | 终端模拟/输出处理 |

**关键文件**：`deeptutor_cli/main.py`、`deeptutor_cli/chat.py`、`deeptutor_cli/common.py`。

### 2.3 数据/存储

| 框架 | 版本 | 用途 |
|---|---|---|
| **aiosqlite** | `>=0.19.0` | 默认 SessionStore、turn/event/message/usage 持久化 |
| **PocketBase** | `>=0.12.0` | 可选会话后端 |
| **Redis** | `>=5.0.0,<7.0.0` | 多 worker 协调：lease、fencing、command、event journal、leader |
| **SQLite** | Python 内置/aiosqlite | Lite 模式/默认本地存储 |

**关键文件**：`deeptutor/services/session/sqlite_store.py`、`services/session/pocketbase_store.py`、`runtime/coordination/memory.py`、`runtime/coordination/redis.py`。

### 2.4 LLM / Agent 基础

| 框架 | 版本 | 用途 |
|---|---|---|
| **OpenAI SDK** | `openai>=1.30.0` | OpenAI/OpenAI-compatible provider |
| **Anthropic SDK** | `anthropic>=0.30.0`（providers 模块） | Anthropic provider |
| **DashScope** | `>=1.14.0`（providers 模块） | 阿里云模型 |
| **Perplexity** | `>=0.1.0`（providers 模块） | Perplexity provider |
| **MCP Python SDK** | `mcp>=1.26.0,<2.0.0` | MCP 工具发现/调用 |
| **Claude Agent SDK** | optional `claude-agent-sdk>=0.2.174,<0.3` | Claude Code/agent connector |
| **CodeBuddy Agent SDK** | optional `codebuddy-agent-sdk>=0.3,<0.4` | CodeBuddy connector |
| **tenacity** | `>=8.0.0` | 重试 |
| **tiktoken** | `>=0.5.0` | token 计数 |
| **json-repair** | `>=0.57.0,<1.0.0` | 模型 JSON 修复 |

**关键文件**：`services/llm/provider_factory.py`、`services/llm/provider_core/`、`runtime/agentic/`、`services/subagent/`。

### 2.5 RAG / 检索

| 框架 | 版本/位置 | 用途 |
|---|---|---|
| **LlamaIndex** | `llama-index>=0.14.12` | 索引/检索基础 |
| **PageIndex** | `pageindex>=0.2.10,<0.3.0` | reasoning-as-retrieval |
| **BM25S** | `bm25s>=0.3.14,<0.4.0` | 关键词检索 |
| **FAISS** | `faiss-cpu>=1.8/1.12` | 向量检索 |
| **LightRAG** | optional `lightrag-hku==1.5.7` | 图/实体检索 |
| **GraphRAG** | optional `graphrag>=3.0.1,<4.0.0`（Python <3.14） | 图检索 |
| **sentence-transformers** | optional `>=3.0.0,<6.0.0`（Python <3.14） | 本地 embedding/rerank |

**关键文件**：`services/rag/factory.py`、`services/rag/pipelines/`、`services/rag/service.py`、`knowledge/indexing_run.py`。

### 2.6 文档解析

| 引擎 | 位置 | 用途 |
|---|---|---|
| **Docling** | `services/parsing/engines/docling/` | 版面/结构化解析 |
| **MarkItDown** | `services/parsing/engines/markitdown/` | 轻量转 Markdown |
| **PyMuPDF4LLM** | `services/parsing/engines/pymupdf4llm/` | PDF → Markdown |
| **LiteParse** | `services/parsing/engines/liteparse/` | 轻量解析 |
| **MinerU** | `services/parsing/engines/mineru/` | 试卷/复杂版面 |
| **Tika** | `services/parsing/engines/tika/` | 通用文档 |
| **Text-only** | `services/parsing/engines/text_only/` | 纯文本 |

**关键文件**：`services/parsing/service.py`、`services/parsing/engines/`、`runtime/isolated_worker.py`。

### 2.7 媒体/生成

| 框架/工具 | 用途 |
|---|---|
| **Manim** | `math_animator` 数学动画 |
| **Remotion**（Node） | `video_lesson` 浏览器视频课 |
| **FFmpeg** | 视频编码 |
| **LaTeX** | 数学公式渲染 |
| **edge-tts** | 音频概览/朗读 |
| **快速模型 provider** | imagegen/videogen/voice |

**关键文件**：`agents/math_animator/`、`agents/video_lesson/`、`capabilities/audio_overview/`、`services/voice/`。

### 2.8 Partner / IM

| SDK | 通道 |
|---|---|
| `python-telegram-bot` | Telegram |
| `lark-oapi` | 飞书/Lark |
| `dingtalk-stream` | 钉钉 |
| `slack-sdk` | Slack |
| `qq-botpy` | QQ Bot |
| `wecom-aibot-sdk` | 企业微信 |
| `zulip` | Zulip |
| `matrix-nio`（optional） | Matrix |
| `python-socketio` / `websocket-client` | 其他长连接 |

**关键文件**：`partners/channels/`、`services/partners/`、`capabilities/partner_*`。

---

## 3. 前端框架

| 框架/库 | 版本 | 用途 |
|---|---|---|
| **Next.js** | `^16.2.3` | App Router、路由、proxy、standalone build |
| **React** | `^19.0.0` | UI |
| **TypeScript** | `^5` | 类型 |
| **Tailwind CSS** | `^3.4.17` | 样式 |
| **i18next / react-i18next** | `^25.8.0` / `^16.5.3` | 多语言 |
| **BlockNote** | `0.56.0` | 块编辑器/笔记 |
| **PDF.js** | `^6.2.108` | PDF 阅读器 |
| **epub.js** | `^0.3.93` | EPUB 阅读器 |
| **Chart.js** | `^4.5.1` | 图表 |
| **Mermaid** | `^11.14.0` | 流程图/图 |
| **KaTeX** | `0.16.27` | 数学公式 |
| **Recogito** | `4.2.5` | 文本标注 |
| **Cytoscape** | `^3.33.1` | 图谱 |
| **Framer Motion** | `^12.24.0` | 动画 |
| **docx-preview / exceljs** | — | Office 预览 |
| **jsPDF / html2canvas** | — | 导出 |
| **react-markdown + remark/rehype** | — | Markdown/公式/raw HTML |
| **lucide-react** | — | 图标 |
| **clsx / tailwind-merge** | — | className 组合 |

**测试/质量**：Vitest 4、Testing Library、Playwright 1.53、jsdom、ESLint 9、dependency-cruiser 17、openapi-typescript、json-schema-to-typescript。

**关键目录**：`web/app/`、`web/features/`、`web/shared/`、`web/lib/`、`web/contracts/`、`web/proxy.ts`。

---

## 4. 桌面 / 构建 / 测试

| 工具 | 用途 |
|---|---|
| **Electron 45** | 桌面壳 |
| **@electron/packager 20.3** | 打包 |
| **Node 22 LTS** | 桌面/构建 |
| **NSIS** | Windows installer |
| **DMG / codesign / notarize** | macOS 打包 |
| **pytest / pytest-asyncio** | Python 测试 |
| **Ruff** | lint/format |
| **import-linter** | 分层依赖边界 |
| **mypy / bandit / detect-secrets** | 类型/安全/密钥扫描 |
| **pre-commit** | 提交门禁 |
| **Vitest / Playwright** | 前端单元/E2E |
| **Docker / Compose** | 部署 |
| **GitHub Actions** | CI/发布 |

**关键文件**：`apps/desktop/README.md`、`apps/desktop/package.json`、`pyproject.toml`、`.pre-commit-config.yaml`、`.github/workflows/tests.yml`。

---

## 5. 如何快速了解：按目标选路线

不要一次学完所有框架。框架知识应该“按需、沿代码链路”学习。

### 路线 A：想快速理解后端 turn（最推荐）

适合：想改 turn runtime、加 capability/tool、排查流式/回放问题。

1. **Python 异步基础**
   - `async/await`、`asyncio.create_task`、`asyncio.Queue`、`contextvars`。
   - 重点：为什么 DeepTutor 用 `StreamBus` 而不是直接 yield。
2. **FastAPI + Pydantic v2 + WebSocket**
   - 读 `api/main.py`、`api/routers/unified_ws.py`、`app/contracts.py`。
   - 理解 `TurnRequest`、`protocol_version`、`command_ack`。
3. **Turn runtime**
   - `app/service.py` → `services/session/turns/request_preparer.py` → `executor.py` → `lifecycle.py`。
   - 理解 `TurnLease`、`fencing_token`、`seq`、`DONE`、`session_meta`。
4. **Orchestrator + Capability**
   - `runtime/turn_engine.py` → `runtime/orchestrator.py` → `core/capability_protocol.py`。
5. **Agent Loop + Tool**
   - `agents/loop/pipeline.py` → `agents/loop/agent_loop.py` → `core/tool_protocol.py` → `agents/_shared/tool_composition.py`。
6. **存储/协调**
   - `services/session/sqlite_store.py`、`runtime/coordination/`。

**时间感**：有 Python/异步基础的人，沿着一条 chat turn 读 2–3 天能建立主干理解；深入恢复/回放/多 worker 需要更长。

### 路线 B：想快速理解前端

适合：改页面、聊天 UI、阅读器、设置、i18n。

1. **React 19 + Next.js 16 App Router**
   - `web/app/` 路由组、layout、page。
   - 注意 React 19/Next 16 的 Server/Client Component 边界。
2. **TypeScript + Tailwind**
   - `web/features/` 的类型、`web/shared/` 原语。
3. **i18next**
   - `web/i18n/`、`web/locales/`、`npm run i18n:check`。
4. **聊天链路**
   - `web/features/chat/transport/`、`web/lib/reconnecting-websocket.ts`、`web/lib/stream.ts`、`web/lib/session-events.ts`、`web/lib/turn-reconcile.ts`。
5. **契约**
   - `web/contracts/` 是生成产物；改后端 schema 后跑 `npm run contracts:generate`。
6. **富内容组件**
   - BlockNote、PDF.js、epub.js、Chart.js、Mermaid、KaTeX、Recogito、Cytoscape；按页面需要学，不要全学。

**时间感**：有 React/TS 基础的人，1–2 天可以跑通前端；聊天/阅读器链路需要 3–5 天。

### 路线 C：想理解 RAG / AI

适合：改知识库、检索、embedding、rerank、GraphRAG。

1. **Embedding / 向量检索基础**
   - chunk、embedding、cosine similarity、top-k、BM25、rerank。
2. **LlamaIndex**
   - 索引、retriever、vector store、query engine；读 `services/rag/pipelines/`。
3. **PageIndex**
   - reasoning-as-retrieval；注意它不走 `RAGService.search()`。
4. **LightRAG / GraphRAG**
   - 实体/关系抽取、图检索、社区摘要；读对应 optional pipeline。
5. **DeepTutor 封装**
   - `services/rag/factory.py`、`service.py`、`knowledge/indexing_run.py`、`progress_tracker.py`。
6. **评测**
   - `services/rag/eval/`、`tests/knowledge/`。

**时间感**：有检索基础的人，2–3 天理解单 pipeline；GraphRAG/LightRAG 需要额外 1–2 周。

### 路线 D：想开发 Tool / Capability

适合：加一个工具或一个教学模式。

1. **Tool 协议**
   - `core/tool_protocol.py`、`tools/builtin_specs.py`、`tools/builtin/__init__.py`。
2. **挂载规则**
   - `agents/_shared/tool_composition.py`：`USER_TOGGLEABLE_TOOL_NAMES`、`CONFIGURABLE_BUILTIN_TOOL_NAMES`、`ToolMountFlags`。
3. **Capability 协议**
   - `core/capability_protocol.py`、`agents/_shared/capability_result.py`。
4. **注册**
   - `modules/manifests/*.json`、`runtime/bootstrap/builtin_capabilities.py`、`runtime/registry/`。
5. **测试**
   - `tests/tools/`、`tests/capabilities/`；覆盖 metadata/implementation parity、missing module、权限、取消、结果 envelope。

### 路线 E：想改桌面 / 打包

1. **Electron 基础**：main/renderer、IPC、窗口、进程生命周期。
2. **Node 22**：构建脚本、子进程。
3. **`apps/desktop/README.md`**：editions、core bundle、seal、签名、DMG/NSIS。
4. **打包流程**：`build:runtime`（`git archive HEAD`）→ `smoke` → `package`。

---

## 6. 推荐学习顺序（最短路径）

```text
第 1 步：跑起来
  python -m pip install -r requirements/full.txt
  python -m pip install -e . --no-deps
  (cd web && npm ci --legacy-peer-deps)
  deeptutor init --non-interactive
  deeptutor start --dev

第 2 步：跑一条 chat turn
  deeptutor run chat "用一句话解释傅里叶变换" --format json

第 3 步：读主干
  app/service.py
    → services/session/turns/request_preparer.py
    → services/session/turns/executor.py
    → runtime/turn_engine.py
    → runtime/orchestrator.py
    → agents/chat/capability.py
    → agents/loop/pipeline.py

第 4 步：读一个工具
  core/tool_protocol.py
    → agents/_shared/tool_composition.py
    → tools/rag_tool.py（或 web_search.py）

第 5 步：读事件/存储
  core/stream.py
    → runtime/stream_bus.py
    → services/session/turns/lifecycle.py
    → services/session/sqlite_store.py

第 6 步：按需扩展
  前端：web/proxy.ts → web/features/chat/transport
  RAG：services/rag/factory.py → services/rag/pipelines
  桌面：apps/desktop/README.md
```

---

## 7. 官方文档清单

### 后端

- FastAPI: <https://fastapi.tiangolo.com/>
- Pydantic v2: <https://docs.pydantic.dev/latest/>
- Uvicorn: <https://www.uvicorn.org/>
- Typer: <https://typer.tiangolo.com/>
- Rich: <https://rich.readthedocs.io/>
- SQLite: <https://www.sqlite.org/docs.html>
- Redis: <https://redis.io/docs/latest/>
- MCP: <https://modelcontextprotocol.io/>

### AI / RAG

- LlamaIndex: <https://docs.llamaindex.ai/>
- FAISS: <https://github.com/facebookresearch/faiss/wiki>
- LightRAG: <https://github.com/HKUDS/LightRAG>
- GraphRAG: <https://microsoft.github.io/graphrag/>
- OpenAI Python SDK: <https://github.com/openai/openai-python>
- Anthropic Python SDK: <https://github.com/anthropics/anthropic-sdk-python>

### 前端

- Next.js: <https://nextjs.org/docs>
- React: <https://react.dev/>
- TypeScript: <https://www.typescriptlang.org/docs/>
- Tailwind CSS: <https://tailwindcss.com/docs>
- i18next: <https://www.i18next.com/>
- BlockNote: <https://www.blocknotejs.org/>
- PDF.js: <https://mozilla.github.io/pdf.js/>
- Mermaid: <https://mermaid.js.org/>
- KaTeX: <https://katex.org/docs/>
- Vitest: <https://vitest.dev/>
- Playwright: <https://playwright.dev/>

### 桌面 / 构建

- Electron: <https://www.electronjs.org/docs/latest/>
- Docker: <https://docs.docker.com/>
- GitHub Actions: <https://docs.github.com/actions>

---

## 8. 快速学习的避坑建议

1. **不要先学完所有框架**  
   先沿“一条 chat turn”读代码；遇到哪个框架再查哪个文档。

2. **Pydantic 注意 v2**  
   项目用 `pydantic>=2.0.0`；v1 的 `validator`、`Config` 写法不适用。

3. **Next.js 16 / React 19 较新**  
   文档和示例可能滞后；优先看官方 App Router / React 19 文档，注意 Server/Client Component 边界。

4. **RAG 引擎不是同一套 API**  
   LlamaIndex、PageIndex、LightRAG、GraphRAG 的索引/检索语义不同；不要假设一个 pipeline 能直接替换另一个。

5. **requirements 是生成的**  
   `requirements/full.txt`、`requirements/modules/*.txt` 由 `scripts/sync_module_requirements.py` 生成；要改依赖，改模块 manifest，不要手改生成文件。

6. **contracts 是生成的**  
   `web/contracts/` 由后端 schema 生成；跑 `npm run contracts:generate`，不要手改。

7. **DeepTutor 的 turn 协议是自定义的**  
   FastAPI/WebSocket 文档不会告诉你 `TurnLease`、`seq`、`after_seq`、`session_meta` 的语义；这些要看 `app/service.py` 和 `services/session/`。

8. **异步/子进程容易踩坑**  
   `nest_asyncio`、`contextvars`、`asyncio.create_task`、`run_in_isolated_process_sync`、Windows `ProactorEventLoop` 都有项目特定约束；改流式/取消/解析前先读测试。

9. **先跑测试再改**  
   Python: `python -m pytest tests/... -q`；前端: `cd web && npm run check:fast`；架构边界: `lint-imports` + `python scripts/check_architecture.py`。

10. **不要用“框架默认行为”替代项目契约**  
    DeepTutor 对事件顺序、终态、租约、模块可用性有自己的 invariant；框架只是实现手段。
