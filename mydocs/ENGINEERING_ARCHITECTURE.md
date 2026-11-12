# DeepTutor 工程架构与模块职责

> 本文聚焦 **工程结构、分层与模块职责**。  
> 一次 turn 的数据结构与执行链、设计取舍、黑盒清单见 [`ARCHITECTURE.md`](./ARCHITECTURE.md)。

DeepTutor 是“**一个 Python 后端运行时 + 一个 Next.js 前端 + 多个可独立安装的发行模块**”的 monorepo。可以按五层理解：

```text
入口层     CLI / WebSocket / SDK / Partner IM
             ↓
适配层     api/  (FastAPI routers, /ws)
             ↓
应用层     app/  (TurnApplicationService / DeepTutorApp)
             ↓
运行时     runtime/  (TurnEngine / Orchestrator / StreamBus / Coordination / Registry)
             ↓
业务层     agents/ + capabilities/ + tools/ + 各领域包
             ↓
基础设施   services/  (LLM / RAG / Session / Memory / Workspace / Storage ...)
             ↓
协议层     core/  (UnifiedContext / StreamEvent / Capability/Tool Protocol)
             ↑
装配层     modules/  (manifest catalog / lifecycle)
```

---

## 1. 仓库顶层结构

| 目录 | 职责 |
|---|---|
| `deeptutor/` | Python 后端主包：运行时、能力、工具、领域服务、API |
| `deeptutor_cli/` | Typer CLI；`deeptutor` 命令入口 |
| `deeptutor_web/` | Web 资产打包占位包 |
| `web/` | Next.js 16 + React 19 前端 |
| `apps/desktop/` | Electron 桌面壳 + Mac/Windows 打包 |
| `packages/chat-engine/` | 实验性 TypeScript chat engine |
| `scripts/` | 构建、模块打包、契约生成、架构/仓库检查 |
| `requirements/` | 由模块 manifest 生成的依赖集合 |
| `tests/` | Python 测试 |
| `docs-for-dev/` | 开发手册（架构、测试、模块、I18N 等） |
| `packaging/deeptutor-cli/` | `deeptutor-cli` 兼容元包 |
| `examples/` | 第三方插件示例 |

---

## 2. 后端分层：每层做什么

| 层 | 目录 | 职责 |
|---|---|---|
| **协议层** | `deeptutor/core/` | 语言级协议和值对象：`UnifiedContext`、`StreamEvent`、`TurnCapability`、`BaseTool`、错误/身份/trace |
| **应用层** | `deeptutor/app/` | `TurnApplicationService`、`DeepTutorApp` SDK 门面、依赖容器、turn 请求契约 |
| **运行时层** | `deeptutor/runtime/` | `TurnEngine`、`ChatOrchestrator`、`StreamBus`、协调器、注册表、懒加载、launcher、LLM agentic 原语 |
| **接入层** | `deeptutor/api/` | FastAPI 应用、50 个 router、WebSocket、鉴权、路由懒加载 |
| **业务层** | `deeptutor/agents/`、`capabilities/`、`tools/` | Agent loop、turn capability、模型可选工具 |
| **领域层** | `learning/`、`reading/`、`book/`、`knowledge/`、`partners/`、`notes/`、`co_writer/`、`visualizers/`、`video_learning/` | 学习、阅读、书籍、知识库、伴侣、笔记、协作写作、可视化、视频学习 |
| **基础服务层** | `deeptutor/services/` | LLM、RAG、会话、记忆、工作区、存储、MCP、沙箱、解析、Prompt、i18n 等 |
| **装配层** | `deeptutor/modules/` | 读取 manifest，决定模块路径、依赖、路由、命令、导航、启用/就绪状态 |
| **横切层** | `events/`、`i18n/`、`logging/`、`utils/`、`config/` | 事件总线、国际化元数据、日志、通用工具、设置 |

---

## 3. `deeptutor/` 各包职责

| 包 | 主要职责 |
|---|---|
| `app/` | `TurnApplicationService`：turn 持久化 admission、幂等提交、订阅回放、取消、用户回复；`DeepTutorApp` SDK 门面；`container.py` 组装 store/runtime/registry |
| `api/` | `main.py` FastAPI 生命周期；`routers/` 50 个领域路由；`routers/unified_ws.py` `/ws` 命令协议；`router_loader.py` 按 manifest 懒加载路由；`security_guard.py` 请求筛选 |
| `runtime/` | `turn_engine.py` 执行入口；`orchestrator.py` 选 capability；`stream_bus.py` 每 turn 事件扇出；`coordination/` 租约/fencing/恢复；`registry/` capability/tool 注册；`bootstrap/` 内置能力目录；`agentic/` LLM 客户端、label 协议、并行工具派发、usage；`launcher.py` 后端/前端进程管理；`isolated_worker.py` 解析隔离进程 |
| `core/` | `context.py` `UnifiedContext`；`stream.py` `StreamEvent`；`capability_protocol.py` `TurnCapability`；`tool_protocol.py` `BaseTool`/`ToolDefinition`；`turn_request.py`；`trace.py`；`errors.py` |
| `agents/` | `loop/` 通用探索式 agent loop（pipeline、agent_loop、context_budget、prompt_blocks、native adapter）；`chat/` chat 能力绑定；`research/`、`question/`、`visualize/`、`math_animator/`、`video_lesson/`、`notebook/`、`vision_solver/` 各模式 agent；`_shared/` 工具组合、能力结果、workspace prompt |
| `capabilities/` | 独立 turn 能力与 loop 扩展：`ask_questions`、`solve`、`mastery`、`reading`、`audio_overview`、`notes`、`partner_authoring`、`partner_group`、`obsidian`、`marginnote4`、`ima`、`setup`、`creator`、`subagent`、`explore_context`、`handoff`；`protocol.py`/`registry.py` |
| `tools/` | 模型可调用的单步工具：`builtin/`、`builtin_specs.py`、`rag_tool.py`、`web_search.py`、`paper_search_tool.py`、`exec_tool.py`、`workspace.py`、`mastery_tool.py`、`question_bank.py`、`list_notebook.py`、`cron_tool.py`、`github_query.py`、`media_gen_tool.py`、`vision/` 等 |
| `services/` | 领域与基础设施服务；详见下一节 |
| `knowledge/` | KB 管理、索引运行、进度追踪、manifest、staging、命名、文档添加 |
| `learning/` | 掌握式学习：mastery、grading、assessment、question card、review strategy、scheduler、topic generation/materials、visual practice、storage、prompts |
| `reading/` | 沉浸阅读：ingestion、catalog/store、extract、EPUB bilingual、page render、caption、read aloud、quiz、search、translation、vocabulary、annotations |
| `book/` | Book 引擎/编译器、agents、blocks、prompts、streaming、export、progress、learning overlay、KB health |
| `partners/` | Partner/IM 运行时：bus、channels、config、network、transcription、helpers |
| `plugins/` | 第三方插件：manifest、catalog、distribution、environment、frontend、lifecycle、loader、registry、runtime、transactions |
| `multi_user/` | 多用户：审计、book/knowledge/learning/model/partner/skill/tool access、grant、guardian、identity、learner profile、paths、session handoff |
| `notes/` | 笔记块、proposals、Anki 导出、resources、review、threads |
| `co_writer/` | 协作写作：docx 转换、edit agent、prompts、storage |
| `visualizers/` | 可视化协议/注册/存储/工具/loop capability、builtin |
| `video_learning/` | Invidious 账号、marks、notes、reading migration、service |
| `textbook_struct/` | 教材结构：章节重建、栏黑名单、页眉处理 |
| `events/` | 进程内 EventBus |
| `i18n/` | 能力/状态元数据国际化 |
| `logging/` | 日志配置、适配器、上下文、格式化、loguru bridge、进程流、统计 |
| `utils/` | 归档、BibTeX、配置、编码、文档提取/图片/校验、错误、文件类型、JSON 解析、网络、密钥文件 |
| `config/` | 设置基类 |
| `modules/` | manifest 目录、模块生命周期、模块服务 |

---

## 4. `services/` 子系统

`services/` 是最大的基础服务层，按领域分组：

| 子目录 | 职责 |
|---|---|
| `llm/` | LLM 客户端、provider factory、OpenAI/Azure/Anthropic/本地/云 provider、Claude SDK、模型能力、token 计数、重试、usage、request cache、telemetry、traffic control、图片 caption/replay |
| `model_selection/` | 模型选择与配置解析 |
| `provider_registry.py` | provider 目录与能力声明 |
| `embedding/` | embedding 模型与绑定 |
| `rag/` | RAG 服务、pipeline factory、embedding 绑定、index versioning、query rewrite、retrieval、source visuals、evaluation |
| `parsing/` | 文档解析抽象、引擎注册、隔离 worker、格式路由 |
| `search/` | 搜索 provider、结果融合 |
| `web_source/` | Web 来源抓取/同步 |
| `research/` | 研究工具与 paper/学术搜索支持 |
| `knowledge/`（顶层） | KB 管理、索引运行、进度、staging |
| `session/` | session store、turn runtime、消息、事件 journal、model history、title、usage、迁移 |
| `workspace/` | workspace 上下文、目录、资源、迁移、session 移动、渲染存储 |
| `path_service.py` | 当前账户/workspace 路径解析 |
| `config/` | 运行时 settings、JSON 配置、进程环境覆盖 |
| `settings/` | 设置服务 |
| `auth.py` / `multi_user/` | 认证、账户、tenant/workspace 权限、grants、guardian |
| `memory/` | L1 trace、L2/L3 markdown、记忆存储、召回、consolidator |
| `notebook/` | Notebook 存储与服务 |
| `practice/` | 练习/题库 |
| `calendar/` | 日历 |
| `cron/` | 定时任务 |
| `learning_journal/` | 学习日志 |
| `persona/` | Persona |
| `skill/` | Skill 加载、目录、安装 |
| `prompt/` | PromptManager、YAML 加载、语言选择 |
| `i18n.py` | 服务级 i18n |
| `mcp/` | MCP 配置、OAuth、secrets、manager、session state |
| `cli_apps/` | CLI app 安装/运行/状态 |
| `sandbox/` | 代码/视频沙箱：Docker/Local/Disabled、quota、artifacts |
| `partners/`、`partner_groups/` | Partner 服务与群组 |
| `subagent/` | 本地/远程 agent connector（Claude Code、Codex、Grok 等） |
| `creations/` | 创作/生成物 |
| `task_board.py` | Task Board |
| `storage/` | 文件存储 |
| `file_io.py`、`office_preview.py` | 文件读写、Office 预览 |
| `voice/` | TTS/STT/语音 |
| `imagegen/`、`videogen/` | 图片/视频生成 |
| `generation_http.py` | 生成服务 HTTP 客户端 |
| `keypool.py` | Key 池 |
| `codex_auth/`、`codebuddy_auth.py`、`github_copilot_auth.py` | 各类 OAuth/凭据 |
| `app_update.py`、`doctor.py` | 应用更新、诊断 |
| `base_sync.py`、`github_source/` | 基础同步、GitHub 来源 |
| `suggestions.py` | 建议生成 |
| `singleflight_cache.py` | 单飞缓存 |

---

## 5. 26 个发行模块（`deeptutor/modules/manifests/*.json`）

这些是“**装什么、开什么、暴露什么**”的权威目录，不等同于 Python 包。

| 模块 | 负责内容 |
|---|---|
| `core` | 持久 turn、chat、ask_questions、共享工具、CLI/SDK、设置、会话 |
| `server` | FastAPI API、认证/路由适配 |
| `web` | 打包的 Next.js Web 资产 |
| `solve` | `deep_solve` 能力 + solve planning 工具 |
| `questions` | `deep_question` 能力 + `question_bank` |
| `research` | `deep_research` 能力 + paper/research 工具 |
| `visualize` | `visualize` 能力 + `submit_visualization` |
| `renderer-manim` | `math_animator` + Manim 管线 |
| `video-lesson` | `video_lesson` 能力 |
| `audio-overview` | `audio_overview` 能力 |
| `knowledge` | KB/RAG、Obsidian/MarginNote/IMA loop 与工具 |
| `parsing` | 文档解析基础 |
| `parser-markitdown` | MarkItDown 解析器 |
| `parser-docling` | Docling 解析器 |
| `parser-pymupdf4llm` | PyMuPDF4LLM 解析器 |
| `parser-liteparse` | LiteParse 解析器 |
| `reading` | `immersive_reading` + 阅读工具/路由/导航 |
| `mastery` | `mastery_path` + mastery 工具 |
| `books` | Book 引擎/页面/导出/进度 |
| `memory` | `read_memory` / `write_memory` |
| `notebook` | notebook 工具/路由 |
| `co-writer` | notes loop + note 工具 |
| `calendar` | cron 工具 + 日历路由 |
| `partners` | Partner loop/工具/路由/IM 通道 |
| `providers` | 可选模型供应商 SDK |

模块状态有三层：**installed**（是否安装 distribution）、**enabled**（`data/system/modules.json` 是否启用）、**ready**（依赖、版本、系统可执行文件是否满足）。

---

## 6. 前端 `web/`

| 目录 | 职责 |
|---|---|
| `web/app/` | Next.js App Router：`(workspace)` chat/learning/reading/books/mastery/co-writer/partners/calendar、`(utility)` space/memory/knowledge-bases/notebooks/agents/settings、`(settings)`、`(auth)`、`(admin)` |
| `web/features/` | 领域切片：chat、settings、notes、calendar、knowledge、co-writer、multi-user、capabilities、runtime-status、tasks、ui-layout |
| `web/components/` | 共享 UI 组件（约 115k 行） |
| `web/lib/` | API client、WebSocket 重连、stream、session events、turn reconcile、markdown、阅读器、文件/附件等 |
| `web/shared/` | `api`、`auth`、`storage`、`ui` 原语 |
| `web/contracts/` | 由后端生成的 API/turn 契约（约 46.8k 行） |
| `web/context/`、`web/hooks/` | React context 与 hooks |
| `web/i18n/`、`web/locales/` | 多语言 |
| `web/proxy.ts` | 把 `/api/*`、`/ws/*` rewrite 到后端；多用户登录门禁 |
| `web/tests/` | Node/Vitest/Playwright 测试 |
| `web/.dependency-cruiser.cjs` | 禁止循环、页面 import、shared 向上依赖、渲染层 import domain/store/transport |

---

## 7. 桌面端与工具链

| 目录 | 职责 |
|---|---|
| `apps/desktop/src/` | Electron shell、启动画面、菜单、运行时监督 |
| `apps/desktop/platforms/macos-arm64/`、`macos-x64/` | 各平台 Python/Node 构建 + DMG 打包 |
| `apps/desktop/platforms/windows-x64/` | Windows zip + NSIS 安装器 |
| `apps/desktop/scripts/` | build-runtime、seal-runtime、smoke、package、notarize |
| `packages/chat-engine/` | 实验性 TS chat engine：scheduler、context、providers、tools、IPC |
| `scripts/` | `build_modules.py`、`sync_module_requirements.py`、`export_frontend_contracts.py`、`check_architecture.py`、`check_repo_hygiene.py`、`install_extras.py` 等 |
| `requirements/` | `core.txt`、`full.txt`、`server.txt`、`partners.txt` 等生成依赖 |
| `tests/` | Python 测试（约 24 万行） |
| `docs-for-dev/` | 开发手册：ARCHITECTURE、TESTING、MODULES、PLUGINS、I18N、RUNTIME_DESIGN 等 |

---

## 8. 依赖方向与架构边界

`.importlinter` + `scripts/check_architecture.py` 强制：

```text
core  → 不能 import app / api / runtime / services
runtime / services / knowledge / learning / multi_user / capabilities / agents
      → 不能 import api
app   → 不能 import api
api   → 最外层适配器，可以 import app/runtime/services
```

装配层 `modules/` 只读 manifest，不 import feature 实现，保证 core 安装不会因为可选依赖缺失而启动失败。`runtime/registry/` 用懒加载 descriptor，避免无关的 provider、parser、feature 包在启动时被 import。

---

## 9. 一句话总结

> **`core/` 定义协议，`app/` 管 turn 生命周期，`runtime/` 执行和调度，`api/` 接外部，`agents/capabilities/tools/` 定义怎么回答，`services/` 提供 LLM/RAG/存储/记忆等能力，`modules/` 决定装什么和开什么，`web/` 是完整前端，`apps/desktop` 是打包壳。**

如果只记一条主线：

```text
CLI / WS / SDK
  → api/ → app/TurnApplicationService
  → runtime/TurnEngine → ChatOrchestrator
  → capabilities/agents → tools/
  → services/llm + rag + session + memory
  → StreamBus + SessionStore + Coordinator
  → 适配器回放
```

更细的模块职责和一次 turn 的完整调用链，可以看 [`ARCHITECTURE.md`](./ARCHITECTURE.md)。
