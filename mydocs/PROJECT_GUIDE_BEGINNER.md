# DeepTutor 项目解析（小白版）

> 这份文档不讲复杂代码，只回答三个问题：
>
> 1. DeepTutor 到底是什么？
> 2. 它里面有哪些模块，每个模块大概做什么？
> 3. 一条用户消息发出去以后，项目里发生了什么？

---

## 0. 一句话认识 DeepTutor

**DeepTutor 不是“一个会聊天的 AI”，而是一个“AI 学习工作台”。**

它把很多学习场景放在同一个系统里：

- 问问题、聊天
- 解数学题、做深度推理
- 生成题目、做练习
- 做深度研究、查论文
- 画图、做动画、做视频课
- 制定学习路径、一步步带你学
- 打开文档/书籍，边读边问
- 管理知识库、记忆、笔记、日历

你可以把它理解成：

> **一个“AI 老师 + 学习资料库 + 学习计划工具 + 笔记工具 + 可视化工具”的组合体。**

---

## 1. 它是什么，不是什么

### 它是什么

- 一个可以本地运行、也可以打包成桌面 App 的学习产品
- 一个支持多种 AI 模型、多个知识库、多种学习模式的后端平台
- 一个可以让第三方扩展“工具、能力、插件”的框架体系

### 它不是

- 不是单纯的 ChatGPT 套壳
- 不是只支持一种学习模式
- 不是把所有功能堆在一个巨大提示词里
- 不是一个只有前端、没有后端的演示项目

---

## 2. 你打开它会看到什么

前端是一个 Next.js 网页应用，主要页面可以分成几类：

| 区域 | 你会看到什么 |
|---|---|
| **Home / Space** | 首页、最近会话、学习入口、快捷操作 |
| **Chat** | 和 AI 对话、选择模式、附带文件/知识库/人格 |
| **Learning** | 学习路径、阅读、书籍、练习、观看视频 |
| **Reading** | 文档/EPUB/PDF/视频阅读器，选中文字提问 |
| **Books** | AI 生成的书、章节、页面、导出 |
| **Mastery** | 掌握式学习路径：目标 → 模块 → 知识点 → 练习 |
| **Knowledge Center** | 知识库管理、文档上传、检索配置 |
| **Memory** | L1/L2/L3 记忆，学习偏好与长期信息 |
| **Notebook / Co-Writer** | 笔记、题库、协作写作 |
| **Calendar / Task Board** | 学习计划、任务、提醒 |
| **Partners** | 飞书/Telegram/微信等 IM 里的 AI 伙伴 |
| **Settings** | 模型、供应商、工具、语言、界面等设置 |

---

## 3. 它有哪些学习模式

这些模式在项目里叫 **Capability（能力）**。

| 模式 | 人话解释 |
|---|---|
| `chat` | 普通聊天：可以带工具、知识库、附件、人格 |
| `ask_questions` | 先反问你几个关键问题，再继续回答 |
| `deep_solve` | 深度解题：先规划，再推理，再给答案 |
| `deep_question` | 出题：先构思，再生成题目 |
| `deep_research` | 深度研究：改写问题 → 拆解 → 检索 → 写报告 |
| `visualize` | 生成 SVG / Chart.js / Mermaid / HTML 可视化 |
| `math_animator` | 生成数学动画（Manim） |
| `video_lesson` | 生成浏览器里播放的视频课 |
| `audio_overview` | 生成音频概览（类似播客摘要） |
| `mastery_path` | 掌握式学习：按学习路径一步步教 |
| `immersive_reading` | 沉浸阅读：打开文档/视频，边看边问 |

---

## 4. 一条消息的旅程

假设你在 Chat 里输入：

> “解释一下傅里叶变换，并画个图。”

它大致会经历这些步骤：

```text
你点发送
  ↓
前端把消息、模式、附件、知识库等打包
  ↓
后端 API / WebSocket 收到请求
  ↓
TurnApplicationService 判断：这是不是一个重复请求？要不要新建 turn？
  ↓
TurnRuntimeManager 准备上下文：历史、记忆、附件、知识库、人格……
  ↓
TurnEngine 把这次 turn 交给 ChatOrchestrator
  ↓
ChatOrchestrator 选择 chat 这个能力
  ↓
Agent Loop 开始循环：
  想 → 调工具（比如 RAG、web_search、visualize）→ 看结果 → 再想
  ↓
每产生一段文字/工具调用/阶段变化，就发一个 StreamEvent
  ↓
事件被保存到会话存储，同时通过 WebSocket 流式推给前端
  ↓
前端边收边渲染
  ↓
turn 结束：completed / failed / cancelled
  ↓
如果你刷新页面，后端能从保存的事件里把这次 turn 回放出来
```

用餐厅打比方：

| 角色 | 项目里的东西 |
|---|---|
| 前台接待 | `api/` |
| 点单/排号/订单管理 | `app/TurnApplicationService` |
| 后厨调度 | `runtime/TurnEngine` + `ChatOrchestrator` |
| 厨师 | `capabilities/` + `agents/` |
| 厨具 | `tools/` |
| 食材仓库 | `services/`（模型、RAG、记忆、存储……） |
| 订单小票和账本 | `services/session/` + 事件 journal |

---

## 5. 模块地图（人话版）

| 目录/模块 | 你可以把它理解成 | 它负责什么 |
|---|---|---|
| `deeptutor/core/` | 通用语言和规则 | 定义“上下文”“事件”“能力”“工具”等基础协议 |
| `deeptutor/app/` | 总调度台 | 接收 turn、去重、启动、取消、订阅、回放 |
| `deeptutor/runtime/` | 发动机和流水线 | 执行 turn、选能力、发事件、管租约、恢复 worker |
| `deeptutor/api/` | 前台/电话接线员 | HTTP API、WebSocket、鉴权、路由 |
| `deeptutor/agents/` | 各位“老师/专家”的大脑 | 不同模式的 agent 实现、通用 agent loop |
| `deeptutor/capabilities/` | 不同的教学模式 | 每种模式如何组织阶段、提示词、工具 |
| `deeptutor/tools/` | 工具箱 | 模型可以调用的单步工具：搜索、RAG、执行代码、画图等 |
| `deeptutor/services/` | 后勤部门 | 模型调用、RAG、会话存储、记忆、工作区、MCP、沙箱等 |
| `deeptutor/knowledge/` | 图书馆/知识库 | 文档管理、索引、进度、检索准备 |
| `deeptutor/learning/` | 教务处 | 学习路径、掌握式学习、评估、复习策略 |
| `deeptutor/reading/` | 阅读器后台 | 文档导入、目录、页面渲染、朗读、翻译、引用 |
| `deeptutor/book/` | 写书/编书 | 书的结构、章节、页面、导出、进度 |
| `deeptutor/partners/` | IM 伙伴 | 飞书/Telegram/微信等通道、消息收发 |
| `deeptutor/plugins/` | 插件市场后台 | 第三方插件的安装、审批、运行 |
| `deeptutor/multi_user/` | 用户与权限 | 账户、租户/工作区、角色、资源访问 |
| `deeptutor/notes/` | 笔记后台 | 笔记块、复习、Anki 导出 |
| `deeptutor/co_writer/` | 协作写作 | 文档编辑 agent、DOCX 转换 |
| `deeptutor/visualizers/` | 可视化框架 | 可视化协议、注册、存储、工具 |
| `deeptutor/video_learning/` | 视频学习 | 视频标记、笔记、账号、迁移 |
| `deeptutor/textbook_struct/` | 教材结构 | 章节重建、页眉、栏处理 |
| `web/` | 前端界面 | Next.js 页面、组件、状态、WebSocket 连接 |
| `apps/desktop/` | 桌面 App | Electron 外壳、打包、签名、启动后端 |
| `packages/chat-engine/` | 实验性引擎 | TypeScript 版 chat engine（默认不用） |
| `scripts/` | 工具间 | 构建模块、生成契约、检查架构、检查仓库 |
| `tests/` | 质检部门 | Python 测试 |
| `docs-for-dev/` | 开发者手册 | 架构、测试、模块、插件、I18N 等说明 |

---

## 6. 26 个“发行模块”，按家族分

项目里有一组 `deeptutor/modules/manifests/*.json`，它们决定“装什么、开什么、前端显示什么”。

可以按家族理解：

### 基础家族

| 模块 | 作用 |
|---|---|
| `core` | 最核心：turn、chat、ask_questions、CLI/SDK、设置、会话 |
| `server` | 后端 API、鉴权、路由 |
| `web` | 打包好的前端页面 |

### 学习模式家族

| 模块 | 作用 |
|---|---|
| `solve` | 深度解题 |
| `questions` | 出题 |
| `research` | 深度研究 |
| `visualize` | 可视化 |
| `renderer-manim` | 数学动画 |
| `video-lesson` | 视频课 |
| `audio-overview` | 音频概览 |
| `mastery` | 掌握式学习路径 |
| `reading` | 沉浸阅读 |
| `books` | AI 书籍 |

### 知识与解析家族

| 模块 | 作用 |
|---|---|
| `knowledge` | 知识库、RAG、Obsidian/MarginNote/IMA |
| `parsing` | 文档解析基础 |
| `parser-markitdown` | MarkItDown 解析器 |
| `parser-docling` | Docling 解析器 |
| `parser-pymupdf4llm` | PyMuPDF4LLM 解析器 |
| `parser-liteparse` | LiteParse 解析器 |

### 学习工具家族

| 模块 | 作用 |
|---|---|
| `memory` | 长期记忆读写 |
| `notebook` | 笔记/收藏/题库 |
| `co-writer` | 协作写作 |
| `calendar` | 日历/定时任务 |

### 合作与扩展家族

| 模块 | 作用 |
|---|---|
| `partners` | IM 伙伴、通道、群组 |
| `providers` | 可选模型供应商 SDK |

> 注意：**模块 manifest 不等于 Python 目录**。manifest 是“发行和装配”的清单；Python 目录是实现。一个模块可能包含多个目录，一个目录也可能服务于多个模块。

---

## 7. 数据存在哪里

DeepTutor 使用一个叫 `DEEPTUTOR_HOME` 的“运行时目录”。它为什么重要？

- 它拥有 `data/` 目录；
- 设置、会话、知识库、日志、模块启用状态都在里面；
- 换一个 home，就相当于换一个独立的 DeepTutor 实例；
- 项目根目录的 `.env` **不会**自动加载。

常见位置：

| 数据 | 位置 |
|---|---|
| 设置 | `data/user/settings/*.json` |
| 模块启用状态 | `data/system/modules.json` |
| 会话/消息 | 会话存储（默认 SQLite） |
| 知识库 | `data/.../knowledge_bases` |
| 日志 | `data/user/logs/deeptutor.jsonl` |
| 启动日志 | `data/user/runtime/launcher.log` |

---

## 8. 为什么代码这么多

一句话：**因为它不是一个聊天软件，而是一整套学习平台。**

代码量大的原因主要有：

1. **模式多**：11 种学习能力，每个都有自己的提示词、阶段、工具、接口和测试。
2. **模块多**：26 个发行模块，每个都要处理安装、启用、依赖、路由、导航。
3. **供应商多**：很多 AI 模型供应商、不同接口格式、不同认证方式。
4. **知识库多**：多种 RAG 引擎、多种文档解析器。
5. **前端大**：聊天、阅读器、书籍、可视化、设置、记忆、笔记……每个都是完整页面。
6. **可靠性要求高**：断线重连、事件回放、worker 崩溃恢复、取消、幂等。
7. **测试多**：大量测试保护这些边界，测试本身就有几十万行。

所以它是“平台级产品”的体量，不是“一个页面 + 一个接口”的体量。

---

## 9. 想读代码，按这个顺序

如果你是第一次读这个项目，建议不要从 `services/` 开始，会迷路。推荐顺序：

```text
1. README.md / AGENTS.md / mydocs/ARCHITECTURE.md
   ↓ 先知道它是什么
2. deeptutor/app/service.py
   ↓ 看 turn 怎么被接收
3. deeptutor/runtime/turn_engine.py
   ↓ 看执行入口
4. deeptutor/runtime/orchestrator.py
   ↓ 看怎么选能力
5. deeptutor/agents/chat/capability.py
   ↓ 看 chat 能力有多薄
6. deeptutor/agents/loop/pipeline.py
   ↓ 看 agent loop 怎么跑
7. deeptutor/agents/_shared/tool_composition.py
   ↓ 看工具怎么被挂载
8. deeptutor/tools/
   ↓ 看具体工具
9. deeptutor/services/llm/ + services/rag/ + services/session/
   ↓ 最后再看模型、检索、存储
```

一句话：**先看“一条消息怎么走”，再看“每个工具箱里有什么”。**

---

## 10. 术语表

| 术语 | 人话解释 |
|---|---|
| **Turn** | 一次“用户发消息 → AI 完成回答”的完整过程 |
| **Capability** | 一种完整的教学模式，拥有整个 turn |
| **Tool** | 模型在一次回答中可以调用的单步操作 |
| **Agent Loop** | AI “想 → 调工具 → 看结果 → 再想”的循环 |
| **RAG** | 先从知识库检索，再让模型基于检索结果回答 |
| **Provider** | 提供 AI 模型的服务商，如 OpenAI、Anthropic、本地 Ollama |
| **Module** | 可独立安装/启用的发行单元，例如 solve、reading、memory |
| **StreamEvent** | 流式事件：文字、思考、工具调用、阶段、错误、结束等 |
| **Session** | 一次持续对话的容器 |
| **Workspace** | 用户的内容工作区，文件、会话、知识库等在其中的作用域 |
| **Runtime Home** | `DEEPTUTOR_HOME`，保存设置、数据、日志的根目录 |
| **Lease** | turn 的“所有权凭证”，防止多进程/多 worker 抢同一个 turn |

---

## 11. 常见误解

1. **“它就是一个 ChatGPT 套壳”**  
   不是。它有自己的 turn 生命周期、能力/工具注册、知识库、记忆、学习路径、阅读器、书籍、桌面端。

2. **“模块就是 Python 目录”**  
   不是。模块是 `modules/manifests` 里的发行清单；Python 目录是实现。

3. **“工具就是能力”**  
   不是。工具是模型按需选择的单步操作；能力是拥有整个 turn 的完整教学流程。

4. **“前端改一下就行，后端随便返回”**  
   不行。前端不变时，后端必须严格保持 API/WS/事件/错误码契约。

5. **“代码多就是写得乱”**  
   不完全是。大部分体量来自产品宽度、可靠性、测试和多供应商支持；但也确实存在兼容层和超大文件的技术债。

---

## 12. 总结

> **DeepTutor 是一个“多模式 AI 学习平台”：前端负责界面，后端负责 turn 生命周期、能力调度、工具执行、模型/知识库/记忆/存储，模块 manifest 负责装配，测试负责守住边界。**

如果你只记一条主线：

```text
前端 → API → TurnApplicationService → TurnEngine → ChatOrchestrator
     → Capability/Agent Loop → Tools
     → LLM + RAG + Session + Memory
     → StreamEvent → 前端流式渲染 / 断线回放
```

更深入的内容见：

- [`ENGINEERING_ARCHITECTURE.md`](./ENGINEERING_ARCHITECTURE.md) — 工程结构与模块职责
- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — 架构阅读报告与一次 turn 的完整链路
