# 高频必背 Top 30

使用方法：

- 第一遍：只背 15 秒版，保证每题能开口。
- 第二遍：补 60 秒版，能讲清机制。
- 第三遍：全部接回 HSBC Agentic RAG 项目。
- 模拟面试后，把薄弱题标成 `未背` 或 `能说`。

掌握状态：

```text
未背：看到题会卡
能说：能讲 30-60 秒，但追问会虚
熟练：能接项目、能答追问
```

## 题目清单

| 编号 | 问题 | 模块 | 状态 |
|---|---|---|---|
| 01 | Agent 是什么？ | Agent | 未背 |
| 02 | Agent 和 RAG 区别是什么？ | Agent/RAG | 未背 |
| 03 | Agentic Loop 是什么？ | Agent | 未背 |
| 04 | ReAct 是什么？ | Agent | 未背 |
| 05 | Plan-and-Execute 是什么？ | Agent | 未背 |
| 06 | Agent 如何防止死循环？ | Agent 工程化 | 未背 |
| 07 | Agent 如何减少幻觉？ | Agent/评估 | 未背 |
| 08 | Memory 如何设计？ | Memory | 未背 |
| 09 | Function Calling 流程是什么？ | Tool Calling | 未背 |
| 10 | Tool Calling 如何提高准确率？ | Tool Calling | 未背 |
| 11 | MCP 是什么？ | MCP/Skill | 未背 |
| 12 | MCP 和 Function Calling 区别是什么？ | MCP/Tool | 未背 |
| 13 | Skill 和 Tool 区别是什么？ | Skill/Tool | 未背 |
| 14 | RAG 完整流程是什么？ | RAG | 未背 |
| 15 | Chunk 策略怎么设计？ | RAG | 未背 |
| 16 | Embedding 是什么？ | RAG/LLM 基础 | 未背 |
| 17 | 向量数据库是什么？ | RAG | 未背 |
| 18 | BM25 是什么？ | RAG | 未背 |
| 19 | Hybrid Search 是什么？ | RAG | 未背 |
| 20 | Rerank 是什么？ | RAG | 未背 |
| 21 | Query Rewrite 是什么？ | RAG | 未背 |
| 22 | Lost in the Middle 怎么解决？ | RAG/上下文 | 未背 |
| 23 | Prompt 基本结构是什么？ | Prompt | 未背 |
| 24 | JSON 结构化输出怎么保证？ | Prompt/工程化 | 未背 |
| 25 | LangChain 和 LangGraph 区别是什么？ | 框架 | 未背 |
| 26 | FastAPI / async 在 AI 应用里有什么用？ | 工程化 | 未背 |
| 27 | Redis / Docker 在 AI 应用里怎么用？ | 工程化 | 未背 |
| 28 | 线程、进程、协程区别是什么？ | Python 基础 | 未背 |
| 29 | KV Cache 是什么？ | LLM 基础 | 未背 |
| 30 | 你的 Agent 项目架构和难点是什么？ | 项目表达 | 未背 |

## 第一批优先打磨题

先打磨这 5 题，因为它们最容易和你的简历主项目连起来：

1. 你的 Agent 项目架构和难点是什么？
2. Agent 和 RAG 区别是什么？
3. RAG 完整流程是什么？
4. Agent 如何减少幻觉？
5. Tool Calling 如何提高准确率？

## 答题卡模板

```text
问题：
所属模块：
高频等级：

15 秒版：

60 秒版：

3 分钟深挖版：

结合 HSBC 项目怎么说：

可能追问：
1.
2.
3.

容易踩坑：

掌握状态：
```

## 示例：Agent 和 RAG 区别是什么？

15 秒版：

> RAG 主要解决“查知识再回答”，Agent 解决“根据目标自主决策并调用工具完成任务”。RAG 是 Agent 可以使用的一种能力，但 Agent 不等于 RAG。

60 秒版：

> RAG 的核心流程是先从知识库检索相关上下文，再让 LLM 基于上下文回答，重点是提升知识准确性、减少幻觉和支持私有知识。Agent 更强调任务执行能力，它会根据目标进行规划、调用工具、观察结果，并可能多轮循环直到完成任务。在我的 HSBC 项目里，RAG 负责检索手语规则和术语短语，Agentic Workflow 负责任务拆解，把修正、缩减、检索、翻译和校验串起来。

可能追问：

1. 为什么你的项目不是普通 RAG？
2. Agentic Workflow 和 Agent 有什么区别？
3. Agent 怎么防止循环或错误工具调用？

