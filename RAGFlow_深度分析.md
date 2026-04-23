# RAGFlow 深度架构分析

> 版本: v0.23.1  
> 分析日期: 2026年1月4日  
> 项目地址: https://github.com/infiniflow/ragflow

## 目录

- [1. 项目概述](#1-项目概述)
- [2. 系统架构](#2-系统架构)
- [3. 核心模块详解](#3-核心模块详解)
  - [3.1 API 层](#31-api-层)
  - [3.2 RAG 核心引擎](#32-rag-核心引擎)
  - [3.3 DeepDoc 文档解析](#33-deepdoc-文档解析)
  - [3.4 Agent 智能体系统](#34-agent-智能体系统)
  - [3.5 GraphRAG 知识图谱](#35-graphrag-知识图谱)
- [4. 数据存储层](#4-数据存储层)
- [5. LLM 集成层](#5-llm-集成层)
- [6. 工作流引擎](#6-工作流引擎)
- [7. 部署架构](#7-部署架构)
- [8. 技术亮点](#8-技术亮点)
- [9. 扩展性设计](#9-扩展性设计)
- [10. 总结](#10-总结)

---

## 1. 项目概述

### 1.1 什么是 RAGFlow

RAGFlow 是一个开源的 **检索增强生成（RAG）引擎**，基于深度文档理解构建。它将先进的 RAG 技术与 Agent 能力相融合，为大语言模型（LLM）创建了一个卓越的上下文层。

### 1.2 核心定位

```
+------------------+     +------------------+     +------------------+
|   Document       | --> |   RAG Engine     | --> |   LLM Response   |
|   (PDF/Word/...) |     |   (Chunking +    |     |   (Grounded      |
|                  |     |   Retrieval)     |     |   Citations)     |
+------------------+     +------------------+     +------------------+
```

### 1.3 关键特性

| 特性 | 描述 |
|------|------|
| **深度文档理解** | 基于视觉的 OCR、布局识别、表格结构识别 |
| **模板化分块** | 智能且可解释的多种分块模板 |
| **可追溯引用** | 减少幻觉，支持快速查看关键引用来源 |
| **异构数据源** | 支持 Word、PPT、Excel、PDF、图片、网页等 |
| **自动化 RAG 工作流** | 从个人到企业级的流畅 RAG 编排 |
| **GraphRAG 支持** | 知识图谱构建与图谱检索 |
| **Agent 工作流** | 可编排的智能体工作流画布 |
| **MCP 支持** | Model Context Protocol 集成 |

---

## 2. 系统架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Frontend (React/TypeScript)                     │
│                                   UmiJS + Ant Design                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              API Layer (Quart/Flask)                         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │ kb_app  │ │dialog_app│ │canvas_app│ │ doc_app │ │ file_app│ │ llm_app │   │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
              ┌─────────────────────────┼─────────────────────────┐
              ▼                         ▼                         ▼
┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
│      DeepDoc        │   │     RAG Engine      │   │   Agent/Canvas      │
│  ┌───────────────┐  │   │  ┌───────────────┐  │   │  ┌───────────────┐  │
│  │ PDF Parser    │  │   │  │   Chunking    │  │   │  │  Components   │  │
│  │ OCR           │  │   │  │   Retrieval   │  │   │  │  Tools        │  │
│  │ Layout Recog  │  │   │  │   Reranking   │  │   │  │  Workflow     │  │
│  │ TSR           │  │   │  │   Generation  │  │   │  │  Graph DSL    │  │
│  └───────────────┘  │   │  └───────────────┘  │   │  └───────────────┘  │
└─────────────────────┘   └─────────────────────┘   └─────────────────────┘
              │                         │                         │
              └─────────────────────────┼─────────────────────────┘
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              LLM Integration Layer                           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ ChatModel│ │EmbedModel│ │RerankModel│ │ TTSModel │ │ OCRModel │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│  支持: OpenAI, Anthropic, Ollama, DeepSeek, Gemini, 通义千问, 智谱 等 30+    │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Data Storage Layer                              │
│  ┌──────────┐ ┌────────────────┐ ┌──────────┐ ┌──────────┐                 │
│  │  MySQL   │ │ Elasticsearch  │ │  Redis   │ │  MinIO   │                 │
│  │(Postgres)│ │  (Infinity)    │ │          │ │ (S3/OSS) │                 │
│  └──────────┘ └────────────────┘ └──────────┘ └──────────┘                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 目录结构

```
ragflow/
├── api/                    # 后端 API 服务
│   ├── apps/              # Flask Blueprint 模块
│   │   ├── kb_app.py      # 知识库管理
│   │   ├── dialog_app.py  # 对话/聊天处理
│   │   ├── canvas_app.py  # Agent 工作流画布
│   │   ├── document_app.py # 文档处理
│   │   └── sdk/           # SDK API 接口
│   ├── db/                # 数据库模型和服务
│   │   ├── db_models.py   # ORM 模型定义
│   │   └── services/      # 业务逻辑服务
│   └── ragflow_server.py  # 主服务入口
├── rag/                    # RAG 核心逻辑
│   ├── llm/               # LLM 抽象层
│   ├── nlp/               # NLP 处理（分词、搜索）
│   ├── flow/              # 分块、解析、管道
│   ├── prompts/           # Prompt 模板
│   └── svr/               # 后台任务执行器
├── deepdoc/               # 深度文档解析
│   ├── parser/            # 各类文档解析器
│   └── vision/            # 视觉处理（OCR、布局）
├── agent/                  # Agent 智能体系统
│   ├── canvas.py          # 工作流画布引擎
│   ├── component/         # 组件库
│   ├── tools/             # 工具集成
│   └── templates/         # 预置工作流模板
├── graphrag/              # GraphRAG 知识图谱
│   ├── general/           # 通用图谱提取
│   ├── light/             # 轻量图谱
│   └── search.py          # 图谱检索
├── memory/                # 对话记忆管理
├── mcp/                   # MCP 协议支持
├── plugin/                # 插件系统
├── sdk/                   # Python SDK
└── web/                   # 前端应用 (React)
```

---

## 3. 核心模块详解

### 3.1 API 层

#### 3.1.1 服务入口

RAGFlow 使用 **Quart**（Flask 的异步版本）作为 Web 框架：

```python
# api/ragflow_server.py
from quart import Quart
from quart_cors import cors

app = Quart(__name__)
app = cors(app, allow_origin="*")

# 配置 Swagger API 文档
swagger = Swagger(app, config=swagger_config)

# 请求超时配置（适应慢速 LLM 响应）
app.config["RESPONSE_TIMEOUT"] = 600  # 10分钟
app.config["BODY_TIMEOUT"] = 600
```

#### 3.1.2 API 蓝图模块

| 模块 | 路径 | 功能 |
|------|------|------|
| `kb_app` | `/v1/kb` | 知识库 CRUD、权限管理 |
| `dialog_app` | `/v1/dialog` | 对话管理、聊天配置 |
| `canvas_app` | `/v1/canvas` | Agent 工作流画布 |
| `document_app` | `/v1/document` | 文档上传、解析状态 |
| `chunk_app` | `/v1/chunk` | 文本块管理 |
| `conversation_app` | `/v1/conversation` | 会话历史 |
| `llm_app` | `/v1/llm` | LLM 配置管理 |
| `file_app` | `/v1/file` | 文件存储管理 |

#### 3.1.3 认证机制

```python
# 基于 JWT 的认证
def _load_user():
    jwt = Serializer(secret_key=settings.SECRET_KEY)
    authorization = request.headers.get("Authorization")
    
    access_token = str(jwt.loads(authorization))
    user = UserService.query(access_token=access_token, status=StatusEnum.VALID.value)
    
    # 支持 API Token 认证
    if not user and len(authorization.split()) == 2:
        objs = APIToken.query(token=authorization.split()[1])
        if objs:
            user = UserService.query(id=objs[0].tenant_id)
    
    return user[0] if user else None
```

### 3.2 RAG 核心引擎

#### 3.2.1 检索流程

```
用户查询
    │
    ▼
┌─────────────────┐
│  Query 预处理   │  ← 分词、同义词扩展、关键词提取
└─────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│              混合检索 (Hybrid Search)        │
│  ┌─────────────┐      ┌─────────────┐       │
│  │  全文检索   │      │  向量检索   │       │
│  │ (BM25/TF-IDF)│     │ (Embedding) │       │
│  └─────────────┘      └─────────────┘       │
│            └──────┬──────┘                   │
│                   ▼                          │
│         Weighted Sum Fusion                  │
│         (权重: 0.05 : 0.95)                  │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────┐
│   Reranking     │  ← 重排序模型精排
└─────────────────┘
    │
    ▼
┌─────────────────┐
│   LLM 生成      │  ← 带引用的回答生成
└─────────────────┘
```

#### 3.2.2 搜索实现

```python
# rag/nlp/search.py
class Dealer:
    def search(self, req, idx_names, kb_ids, emb_mdl=None, highlight=None):
        # 1. 全文检索表达式
        matchText, keywords = self.qryr.question(qst, min_match=0.3)
        
        # 2. 向量检索表达式
        matchDense = self.get_vector(qst, emb_mdl, topk, similarity=0.1)
        
        # 3. 融合检索
        fusionExpr = FusionExpr("weighted_sum", topk, {"weights": "0.05,0.95"})
        matchExprs = [matchText, matchDense, fusionExpr]
        
        # 4. 执行搜索
        res = self.dataStore.search(src, highlightFields, filters, 
                                    matchExprs, orderBy, offset, limit,
                                    idx_names, kb_ids)
        return res
```

#### 3.2.3 分块策略

RAGFlow 支持多种智能分块模板：

| 模板 | 适用场景 | 特点 |
|------|----------|------|
| `naive` | 通用文档 | 基于段落/句子的简单分块 |
| `book` | 书籍 | 基于章节结构 |
| `paper` | 论文 | 识别摘要、引用、图表 |
| `laws` | 法律文档 | 条款层级识别 |
| `resume` | 简历 | 结构化字段提取 |
| `manual` | 技术手册 | 步骤和指南识别 |
| `qa` | 问答对 | 自动提取 Q&A 对 |
| `table` | 表格数据 | 表格转文本描述 |

### 3.3 DeepDoc 文档解析

#### 3.3.1 视觉处理管线

```
PDF/图像输入
     │
     ▼
┌─────────────────┐
│      OCR        │  ← PaddleOCR/自研模型
│  文字识别       │
└─────────────────┘
     │
     ▼
┌─────────────────┐
│   布局识别      │  ← 10种布局元素
│ Layout Recog    │     Text, Title, Figure, Table,
│                 │     Header, Footer, Equation...
└─────────────────┘
     │
     ▼
┌─────────────────┐
│   表格结构识别  │  ← TSR (Table Structure Recognition)
│      TSR        │     Column, Row, Header, Spanning Cell
└─────────────────┘
     │
     ▼
┌─────────────────┐
│   语义组装      │  ← 将视觉结果转换为结构化文本
│  Content Merge  │
└─────────────────┘
```

#### 3.3.2 布局识别类别

```python
# 10种基本布局组件
LAYOUT_COMPONENTS = [
    "Text",           # 正文
    "Title",          # 标题
    "Figure",         # 图片
    "Figure caption", # 图片说明
    "Table",          # 表格
    "Table caption",  # 表格说明
    "Header",         # 页眉
    "Footer",         # 页脚
    "Reference",      # 参考文献
    "Equation"        # 公式
]
```

#### 3.3.3 支持的解析器

| 解析器 | 文件类型 | 特殊能力 |
|--------|----------|----------|
| `pdf_parser` | PDF | 多模态理解、OCR |
| `docx_parser` | Word | 样式保留 |
| `ppt_parser` | PowerPoint | 幻灯片结构 |
| `excel_parser` | Excel | 多 Sheet 处理 |
| `html_parser` | HTML | 结构提取 |
| `markdown_parser` | Markdown | 语法解析 |
| `mineru_parser` | PDF | MinerU 深度解析 |
| `docling_parser` | 多格式 | Docling 集成 |

### 3.4 Agent 智能体系统

#### 3.4.1 工作流画布引擎

RAGFlow 的 Agent 系统基于 **Graph DSL** 设计：

```python
# agent/canvas.py
class Canvas(Graph):
    """
    DSL 结构示例:
    {
        "components": {
            "begin": {
                "obj": {"component_name": "Begin", "params": {}},
                "downstream": ["retrieval_0"],
                "upstream": [],
            },
            "retrieval_0": {
                "obj": {"component_name": "Retrieval", "params": {}},
                "downstream": ["generate_0"],
                "upstream": ["begin"],
            },
            "generate_0": {
                "obj": {"component_name": "Generate", "params": {}},
                "downstream": ["message_0"],
                "upstream": ["retrieval_0"],
            }
        },
        "history": [],
        "path": ["begin"],
        "globals": {
            "sys.query": "",
            "sys.user_id": tenant_id,
            "sys.conversation_turns": 0,
            "sys.files": []
        }
    }
    """
    
    async def run(self, **kwargs):
        # 异步执行工作流
        for cpn_id in self.path:
            cpn = self.get_component_obj(cpn_id)
            await cpn.invoke_async(**kwargs)
            
            # 事件流式输出
            yield {"event": "node_started", "component_id": cpn_id}
            yield {"event": "message", "content": cpn.output("content")}
            yield {"event": "node_finished", "component_id": cpn_id}
```

#### 3.4.2 组件库

| 组件类别 | 组件名 | 功能 |
|----------|--------|------|
| **流程控制** | Begin, Switch, Categorize | 工作流入口、条件分支 |
| **迭代** | Iteration, Loop, ExitLoop | 循环处理 |
| **LLM** | LLM, AgentWithTools | 语言模型调用 |
| **数据操作** | VariableAssigner, DataOperations | 变量赋值、数据转换 |
| **输出** | Message, DocsGenerator | 消息输出、文档生成 |
| **特殊** | Fillup, Invoke | 用户输入、外部调用 |

#### 3.4.3 工具集成

```python
# agent/tools/ 目录下的工具
AVAILABLE_TOOLS = [
    "retrieval",    # 知识库检索
    "tavily",       # Tavily 搜索
    "duckduckgo",   # DuckDuckGo 搜索
    "google",       # Google 搜索
    "wikipedia",    # 维基百科
    "arxiv",        # arXiv 论文
    "pubmed",       # PubMed 医学文献
    "github",       # GitHub API
    "exesql",       # SQL 执行
    "code_exec",    # 代码执行 (沙箱)
    "crawler",      # 网页爬取
    "email",        # 邮件发送
    "yahoofinance", # 雅虎财经
    "akshare",      # A 股数据
]
```

### 3.5 GraphRAG 知识图谱

#### 3.5.1 知识图谱构建

```python
# graphrag/general/graph_extractor.py
# 实体和关系提取流程

1. 文本分块 → 2. LLM 实体提取 → 3. 实体消歧 → 4. 关系抽取
                                      ↓
                              5. Leiden 社区发现
                                      ↓
                              6. 社区报告生成
```

#### 3.5.2 图谱检索

```python
# graphrag/search.py
class KGSearch(Dealer):
    async def retrieval(self, question, tenant_ids, kb_ids, emb_mdl, llm):
        # 1. 查询重写 - 提取实体类型和关键词
        ty_kwds, ents = await self.query_rewrite(llm, question, idxnms, kb_ids)
        
        # 2. 实体检索 - 基于关键词
        ents_from_query = self.get_relevant_ents_by_keywords(ents, filters, ...)
        
        # 3. 类型检索 - 基于实体类型
        ents_from_types = self.get_relevant_ents_by_types(ty_kwds, filters, ...)
        
        # 4. 关系检索 - 基于文本相似度
        rels_from_txt = self.get_relevant_relations_by_txt(question, ...)
        
        # 5. N-hop 路径扩展
        nhop_pathes = self._expand_nhop_paths(ents_from_query)
        
        # 6. 综合排序 P(E|Q) = P(E) * P(Q|E) = pagerank * similarity
        ranked_entities = sorted(ents, key=lambda x: x["sim"] * x["pagerank"])
        
        # 7. 社区报告检索
        community_context = self._community_retrieval_(entities, ...)
        
        return {"content": entities + relations + community_context}
```

---

## 4. 数据存储层

### 4.1 存储引擎选择

```yaml
# docker/service_conf.yaml.template
mysql:                    # 关系数据 (用户、知识库、文档元数据)
  host: mysql
  port: 3306

es:                       # 全文+向量检索 (默认)
  hosts: 'http://es01:9200'

infinity:                 # 可选替代 ES (Infinity 向量数据库)
  uri: 'infinity:23817'

redis:                    # 缓存、会话、分布式锁
  host: 'redis:6379'

minio:                    # 对象存储 (文档原文件)
  host: 'minio:9000'
```

### 4.2 数据模型

```python
# api/db/db_models.py - 核心实体

class User(BaseModel):
    id, email, nickname, password, access_token, status, ...

class Tenant(BaseModel):
    id, name, llm_id, embd_id, asr_id, img2txt_id, rerank_id, ...

class Knowledgebase(BaseModel):
    id, tenant_id, name, description, embd_id, parser_id, ...

class Document(BaseModel):
    id, kb_id, name, type, location, size, token_num, chunk_num, 
    parser_id, parser_config, progress, status, ...

class Dialog(BaseModel):
    id, tenant_id, name, llm_id, prompt_type, prompt_config, 
    kb_ids, similarity_threshold, top_k, ...

class Conversation(BaseModel):
    id, dialog_id, name, messages, ...

class Canvas(BaseModel):  # Agent 工作流
    id, tenant_id, title, dsl, description, ...
```

### 4.3 文档存储索引结构

```python
# Elasticsearch/Infinity 索引字段
INDEX_FIELDS = {
    "docnm_kwd": "keyword",        # 文档名
    "kb_id": "keyword",            # 知识库 ID
    "doc_id": "keyword",           # 文档 ID
    "content_ltks": "text",        # 分词内容 (全文检索)
    "content_with_weight": "text", # 带权重内容
    "title_tks": "text",           # 标题分词
    "q_{dim}_vec": "dense_vector", # 向量字段 (动态维度)
    "page_num_int": "integer",     # 页码
    "position_int": "integer",     # 位置
    "important_kwd": "keyword",    # 重要关键词
    "knowledge_graph_kwd": "keyword", # 图谱类型标记
    "entity_kwd": "keyword",       # 实体名
}
```

---

## 5. LLM 集成层

### 5.1 模型抽象

RAGFlow 通过统一抽象支持 **30+ LLM 提供商**：

```python
# rag/llm/__init__.py

# 模型类型枚举
class ModelType:
    ChatModel       # 对话模型
    EmbeddingModel  # 嵌入模型
    RerankModel     # 重排模型
    TTSModel        # 语音合成
    Seq2txtModel    # 语音识别
    CvModel         # 视觉模型
    OcrModel        # OCR 模型

# 支持的提供商
SUPPORTED_PROVIDERS = [
    "OpenAI", "Azure-OpenAI", "Anthropic", "Gemini",
    "DeepSeek", "Ollama", "Groq", "Cohere",
    "Tongyi-Qianwen", "ZHIPU-AI", "MiniMax",
    "Moonshot", "SILICONFLOW", "TogetherAI",
    "NVIDIA", "xAI", "Bedrock", ...
]
```

### 5.2 LiteLLM 集成

```python
# 通过 LiteLLM 统一接口调用
LITELLM_PROVIDER_PREFIX = {
    "OpenAI": "openai/",
    "Anthropic": "",  # 原生支持
    "Ollama": "ollama_chat/",
    "Gemini": "gemini/",
    "DeepSeek": "deepseek/",
    "Azure-OpenAI": "azure/",
    ...
}

# 默认 Base URL
FACTORY_DEFAULT_BASE_URL = {
    "OpenAI": "https://api.openai.com/v1",
    "DeepSeek": "https://api.deepseek.com/v1",
    "Moonshot": "https://api.moonshot.cn/v1",
    "ZHIPU-AI": "https://open.bigmodel.cn/api/paas/v4",
    ...
}
```

### 5.3 模型服务封装

```python
# api/db/services/llm_service.py
class LLMBundle:
    """LLM 服务封装，统一管理模型调用"""
    
    def __init__(self, tenant_id, llm_type, llm_name=None):
        self.tenant_id = tenant_id
        self.llm_type = llm_type
        self.llm_name = llm_name or self._get_default_model()
        
    def chat(self, system, history, gen_conf):
        """同步对话"""
        return self._model.chat(system, history, gen_conf)
    
    async def async_chat(self, system, history, gen_conf):
        """异步对话"""
        return await self._model.async_chat(system, history, gen_conf)
    
    def encode(self, texts):
        """文本嵌入"""
        return self._model.encode(texts)
    
    def rerank(self, query, candidates):
        """重排序"""
        return self._model.rerank(query, candidates)
```

---

## 6. 工作流引擎

### 6.1 执行模型

RAGFlow 的工作流采用 **事件驱动的异步执行模型**：

```python
# agent/canvas.py
async def run(self, **kwargs):
    # 1. 初始化
    yield {"event": "workflow_started", "inputs": kwargs}
    
    # 2. 遍历执行路径
    while idx < len(self.path):
        cpn_id = self.path[idx]
        cpn_obj = self.get_component_obj(cpn_id)
        
        # 3. 节点开始事件
        yield {"event": "node_started", "component_id": cpn_id}
        
        # 4. 异步执行组件
        await cpn_obj.invoke_async(**kwargs)
        
        # 5. 流式消息输出
        if cpn_obj.component_name == "message":
            async for chunk in cpn_obj.output("content"):
                yield {"event": "message", "content": chunk}
        
        # 6. 节点完成事件
        yield {"event": "node_finished", "outputs": cpn_obj.output()}
        
        # 7. 确定下一节点 (条件分支)
        if cpn_obj.component_name in ["categorize", "switch"]:
            self.path.extend(cpn_obj.output("_next"))
        else:
            self.path.extend(cpn["downstream"])
    
    # 8. 工作流完成
    yield {"event": "workflow_finished", "outputs": final_output}
```

### 6.2 组件基类

```python
# agent/component/base.py
class ComponentBase(ABC):
    component_name: str
    
    def __init__(self, canvas, id, param):
        self._canvas = canvas
        self._id = id
        self._param = param
    
    def invoke(self, **kwargs) -> dict[str, Any]:
        """同步执行"""
        self.set_output("_created_time", time.perf_counter())
        try:
            self._invoke(**kwargs)
        except Exception as e:
            self.set_output("_ERROR", str(e))
        return self.output()
    
    async def invoke_async(self, **kwargs):
        """异步执行"""
        if asyncio.iscoroutinefunction(self._invoke):
            await self._invoke(**kwargs)
        else:
            await asyncio.to_thread(self._invoke, **kwargs)
    
    @abstractmethod
    def _invoke(self, **kwargs):
        """具体执行逻辑，子类实现"""
        raise NotImplementedError()
    
    # 变量引用解析
    def get_input(self, key=None):
        for var, o in self.get_input_elements().items():
            v = self.get_param(var)
            if self._canvas.is_reff(v):
                self.set_input_value(var, self._canvas.get_variable_value(v))
```

### 6.3 变量系统

```python
# 变量引用语法: {{component_id@variable_name}} 或 {{sys.xxx}}

# 系统变量
SYSTEM_VARIABLES = {
    "sys.query": "用户输入",
    "sys.user_id": "用户 ID",
    "sys.conversation_turns": "对话轮次",
    "sys.files": "上传文件列表",
}

# 组件输出引用
# {{retrieval_0@content}} → 引用 retrieval_0 组件的 content 输出
# {{llm_0@response}} → 引用 llm_0 组件的 response 输出
```

---

## 7. 部署架构

### 7.1 Docker Compose 部署

```yaml
# docker/docker-compose.yml (简化)
services:
  ragflow:
    image: infiniflow/ragflow:v0.23.1
    ports:
      - "80:80"
      - "9380:9380"
    depends_on:
      - mysql
      - es01
      - redis
      - minio
    
  mysql:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql
    
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.3
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=true
    volumes:
      - es_data:/usr/share/elasticsearch/data
    
  redis:
    image: valkey/valkey:8
    
  minio:
    image: quay.io/minio/minio:RELEASE.2024-11-07
    command: server /data --console-address ":9001"
```

### 7.2 资源要求

| 组件 | 最小配置 | 推荐配置 |
|------|----------|----------|
| CPU | 4 核 | 8+ 核 |
| RAM | 16 GB | 32+ GB |
| Disk | 50 GB | 200+ GB |
| Docker | 24.0+ | 24.0+ |
| Docker Compose | v2.26+ | v2.26+ |

### 7.3 扩展部署选项

```bash
# 使用 GPU 加速 DeepDoc 任务
sed -i '1i DEVICE=gpu' .env
docker compose up -d

# 切换到 Infinity 向量数据库
echo "DOC_ENGINE=infinity" >> .env
docker compose down -v  # 注意：会清除数据
docker compose up -d

# Helm 部署 (Kubernetes)
helm install ragflow ./helm -f values.yaml
```

---

## 8. 技术亮点

### 8.1 深度文档理解

RAGFlow 的核心竞争力在于其 **深度文档理解** 能力：

1. **视觉优先**：将文档渲染为图像后进行分析，不依赖 PDF 原生文本
2. **布局感知**：理解页面布局，正确处理多栏、表格、图表
3. **表格智能化**：TSR 识别复杂表格结构，转换为 LLM 可理解的文本
4. **多模态融合**：结合视觉和文本信息进行理解

### 8.2 可解释的检索

```python
# 检索结果带引用标记
response = """
根据文档分析，RAGFlow 的主要特点包括：
1. 深度文档理解能力 [ID: 42]
2. 模板化的智能分块 [ID: 78]
3. 可追溯的引用来源 [ID: 103]

[ID: 42] 来源: ragflow_overview.pdf, 第3页
[ID: 78] 来源: feature_guide.docx, 章节2.1
[ID: 103] 来源: user_manual.md
"""
```

### 8.3 灵活的工作流

- **可视化编排**：通过画布拖拽构建复杂工作流
- **条件分支**：支持 Switch/Categorize 条件判断
- **循环迭代**：Iteration/Loop 组件处理批量数据
- **工具调用**：集成 20+ 外部工具
- **流式输出**：实时输出生成内容

### 8.4 企业级特性

- **多租户**：完整的租户隔离和权限管理
- **OAuth/OIDC**：企业 SSO 集成
- **API 完备**：RESTful API + Python SDK
- **可观测性**：Langfuse 集成
- **插件系统**：可扩展的插件架构

---

## 9. 扩展性设计

### 9.1 插件系统

```python
# plugin/plugin_manager.py
class GlobalPluginManager:
    @staticmethod
    def load_plugins():
        """加载所有插件"""
        # 1. 内置插件
        load_embedded_plugins()
        
        # 2. 外部插件目录
        for plugin_path in get_plugin_paths():
            load_plugin(plugin_path)
    
    @staticmethod
    def register_tool(tool_class):
        """注册自定义工具"""
        TOOL_REGISTRY[tool_class.name] = tool_class
```

### 9.2 自定义解析器

```python
# 继承基类实现自定义解析器
class CustomParser:
    def __call__(self, file_path, **kwargs):
        # 解析文档
        content = self.extract_text(file_path)
        
        # 返回分块结果
        return [
            {"content": chunk, "metadata": {...}}
            for chunk in self.split(content)
        ]
```

### 9.3 自定义 LLM 提供商

```python
# 实现新的 LLM 提供商
class CustomChatModel(Base):
    _FACTORY_NAME = "CustomProvider"
    
    def __init__(self, api_key, base_url, model_name, **kwargs):
        self.api_key = api_key
        self.base_url = base_url
        self.model_name = model_name
    
    def chat(self, system, history, gen_conf):
        # 实现对话逻辑
        response = self._call_api(system, history)
        return response
```

---

## 10. 总结

### 10.1 RAGFlow 的核心优势

| 维度 | 优势 |
|------|------|
| **文档理解** | 业界领先的深度文档解析，视觉+文本多模态 |
| **检索质量** | 混合检索 + 重排序，可追溯引用 |
| **灵活编排** | 可视化 Agent 工作流画布 |
| **生态集成** | 30+ LLM、20+ 工具、GraphRAG、MCP |
| **企业就绪** | 多租户、权限、SSO、API 完备 |
| **开源社区** | Apache 2.0 许可，活跃开发 |

### 10.2 适用场景

1. **知识库问答**：企业内部文档检索和智能问答
2. **文档分析**：复杂 PDF/Word/Excel 的深度理解
3. **智能助手**：构建定制化的 AI 助手工作流
4. **研究分析**：论文、专利、法律文档的智能检索
5. **数据集成**：多源异构数据的统一 RAG 入口

### 10.3 技术栈总结

```
Frontend:  React + TypeScript + UmiJS + Ant Design
Backend:   Python + Quart/Flask + Peewee ORM
Search:    Elasticsearch / Infinity
Storage:   MySQL/PostgreSQL + MinIO/S3 + Redis
AI/ML:     PyTorch + ONNX + LiteLLM
Deploy:    Docker + Kubernetes (Helm)
```

---

> 本文档基于 RAGFlow v0.23.1 版本分析，项目持续更新中，请参考官方文档获取最新信息。
>
> 官方文档: https://ragflow.io/docs/dev/  
> GitHub: https://github.com/infiniflow/ragflow

