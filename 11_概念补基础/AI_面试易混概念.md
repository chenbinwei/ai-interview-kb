# AI 面试易混概念补基础

这个文件专门记录面试准备过程中卡住的基础概念。目标不是背百科定义，而是能用自己的项目说清楚。

## Few-shot 示例

### 是什么

Few-shot 就是在 Prompt 里先给模型几个输入输出例子，让模型模仿这个模式继续完成新任务。

最简单的理解：

```text
Zero-shot：不给例子，直接让模型做。
One-shot：给 1 个例子。
Few-shot：给几个例子。
```

例子：

```text
任务：把中文转换成英文。

例 1：
输入：你好
输出：Hello

例 2：
输入：谢谢
输出：Thank you

现在请转换：
输入：再见
输出：
```

模型看到前面的格式后，就更容易输出：

```text
Goodbye
```

### 为什么有用

Few-shot 的作用不是把新知识“训练进模型”，而是在当前上下文里告诉模型：

- 任务应该怎么理解
- 输出格式应该长什么样
- 哪些表达是正确示范
- 哪些边界情况应该怎么处理

它适合用来提升格式稳定性和任务一致性。

### 在 HSBC 项目里怎么说

在 HSBC HKSL Gloss 项目里，Few-shot 可以用于 Prompt 里放几组“原始文本 -> HKSL Gloss JSON”的示例，让模型学习输出风格和结构。

例如：

```text
输入：我今天去银行开户。
输出：
{
  "gloss": ["TODAY", "ME", "BANK", "ACCOUNT_OPEN"]
}
```

这样模型不是凭空生成，而是参考示例里的语序、术语和 JSON 格式。

### 面试一句话

> Few-shot 是在 Prompt 里给模型几个输入输出示例，让它模仿任务格式和输出风格。在我的项目里，它主要用于稳定 HKSL Gloss JSON 的格式和语序表达。

### 容易踩坑

不要说 Few-shot 是微调。Few-shot 不会更新模型参数，只是在当前上下文里给示例。

## Gloss

### 是什么

Gloss 可以理解为普通文本和手语动画之间的中间表示。它不是自然语言句子，而是用一串标准化标签表示手语动作、语序和关键概念。

例子：

```json
{
  "gloss": ["TODAY", "ME", "BANK", "ACCOUNT_OPEN"]
}
```

### 为什么有用

下游动画系统不适合直接吃普通中文或粤语，因为自然语言里有很多歧义。Gloss 更结构化，方便后续映射到手语动作或动画片段。

### 在 HSBC 项目里怎么说

> 我做的不是直接生成视频，而是把普通文本转换成后续手语动画系统能使用的 HKSL Gloss JSON。Gloss 是中间层，负责承接语言理解和动画生成。

### 容易踩坑

不要把 Gloss 说成普通英文翻译。它更像手语系统里的标准动作/概念标签。

## BLEU

### 是什么

BLEU 是机器翻译里常见的自动评估指标，主要看模型输出和参考答案之间的词或词组重合度。

简单理解：

```text
输出和标准答案越像，BLEU 越高。
```

### 为什么有用

它可以快速批量比较模型输出和标准答案，适合有固定参考答案的翻译任务。

### 局限

BLEU 主要看字面重合，不真正理解语义。意思相同但表达不同，分数可能低；词重合很多但意思错了，分数也可能看起来不低。

### 在 HSBC 项目里怎么说

> BLEU 可以作为文本相似度参考，但不是核心指标。因为 HKSL Gloss 可能有多种合理表达，项目更关心 JSON 是否合法、术语是否正确、语序是否合理，以及专家是否认为可用。

## ROUGE

### 是什么

ROUGE 是摘要任务里常见的自动评估指标，主要看参考答案里的内容有没有被模型输出覆盖到。

常见类型：

```text
ROUGE-1：单个词重合
ROUGE-2：连续两个词重合
ROUGE-L：最长公共子序列
```

### 和 BLEU 的区别

可以先这么记：

```text
BLEU：生成内容有多像参考答案。
ROUGE：参考答案里的内容被覆盖了多少。
```

### 在 HSBC 项目里怎么说

> ROUGE 更适合摘要这类内容覆盖任务。在我的项目里，它可以辅助观察输出和参考 Gloss 的重合度，但不能替代专家评估和任务指标。

## 当前项目更核心的评估指标

HSBC HKSL Gloss 项目更适合强调这些指标：

- JSON 合法率
- 核心术语正确率
- HKSL 语序合理性
- 检索命中率
- 专家可用率
- 人工修改量

面试总回答：

> BLEU 和 ROUGE 属于传统文本重合指标，可以作为参考，但我的项目更关注输出能不能被下游手语动画系统使用。所以核心评估会看 JSON 格式正确、金融术语正确、HKSL 语序合理，以及专家反馈后的可用率。

## 普通 RAG、复合 RAG、Agentic RAG

### RAG 能不能复合？

可以。RAG 不一定只能是“检索一次、生成一次”。工程里经常会做成复合流程，比如：

```text
Query Rewrite -> 多路检索 -> Rerank -> Context Compression -> Generate
```

也可以有：

- Hybrid Search：BM25 + 向量检索
- RAG-Fusion：把一个问题改写成多个 query，再融合检索结果
- Multi-hop RAG：第一轮检索结果决定下一轮检索方向
- GraphRAG：用知识图谱补强多跳关系推理

所以，不能简单说“只要多检索几次就是 Agentic RAG”。

### 普通 RAG 是什么

最基础的 RAG 通常是固定流程：

```text
用户问题 -> 检索 Top-K 文档 -> 拼进 Prompt -> LLM 生成
```

它的特点是流程比较固定，系统不会太多判断“现在的信息够不够”“下一步要不要换检索词”“是否要调用别的工具”。

### 复合 RAG 是什么

复合 RAG 是在普通 RAG 上加优化模块，但流程大多还是工程上预设好的。

例如：

```text
先 query rewrite，再检索，再 rerank，再生成。
```

这比普通 RAG 强，但它不一定有 Agent 的动态决策。

### Agentic RAG 是什么

Agentic RAG 的重点是：系统会根据中间结果决定下一步动作。

例如：

```text
第一次检索结果不够 -> 改写短语再检索
术语没命中 -> 查别名或 fallback retrieval
JSON 校验失败 -> 重新生成或走修复节点
术语冲突 -> 优先采用术语库结果
```

它更像一个带控制流的工作流，而不是一次固定检索。

### 在 HSBC 项目里怎么说

> 我理解 RAG 本身可以做得很复杂，比如 query rewrite、rerank、hybrid search 都属于 RAG 优化。但我把这个项目称为 Agentic RAG，是因为它不是固定的一次检索生成，而是把 HKSL Gloss 生成拆成多步：预处理、语法转换、术语检索、fallback retrieval、函数/代码节点处理、聚合和 JSON 校验。系统会根据检索结果和校验结果决定是否继续检索、改写短语或重新生成，所以更接近 Agentic Workflow + RAG。

### 面试一句话

> 普通 RAG 更像固定管道，复合 RAG 是把检索链路做得更强，而 Agentic RAG 的关键是有动态决策：系统会判断当前信息够不够、下一步查什么、是否重试或调用工具。

## Agent、Tool、Workflow、设计模式

### 这些是不是术语？

是的，但它们不是同一层级的概念。

```text
Agent = 系统
Tool = 单个可调用能力
Workflow = 固定流程
ReAct / Plan-and-Execute / Reflection = Agent 的运行模式或设计模式
```

### 怎么区分

| 概念 | 一句话 |
|---|---|
| Agent | 围绕目标做决策和行动的系统 |
| Tool | 被 Agent 或 Workflow 调用的函数、API、检索器 |
| Workflow | 人提前编排好的固定步骤 |
| ReAct | Agent 边想、边调用工具、边观察结果 |
| Plan-and-Execute | Agent 先规划完整步骤，再逐步执行 |
| Reflection | Agent 执行后反思哪里不好，再修正 |
| Multi-Agent | 多个 Agent 分角色协作 |

### 面试一句话

> Agent 是系统形态，Tool 是原子能力，Workflow 是固定编排，ReAct、Plan-and-Execute、Reflection 这些是 Agent 的运行模式。我的 HSBC 项目更接近 Agentic Workflow，因为主流程固定，但有 fallback、重试、校验和人工审核这些动态控制。

## Query Rewrite

### 是什么

Query Rewrite 就是查询改写：把用户原始问题改写成更适合检索系统理解的表达。

它改的是“检索用的问题”，不是最终答案。

例子：

```text
用户原话：我想开户口
检索 query：开立银行账户 / account opening / 银行账户申请
```

再比如多轮对话里：

```text
用户上一轮：信用卡怎么申请？
用户这一轮：那需要什么材料？

直接检索：那需要什么材料？
改写后检索：申请信用卡需要什么材料？
```

### 为什么有用

用户说话经常不标准，但知识库里的文档通常是标准表达。Query Rewrite 的作用就是把口语、缩写、省略、同义说法，改成更容易命中知识库的表达。

常见改写方式：

- 口语改标准词
- 省略句补完整
- 中文/粤语/英文术语互转
- 一个问题扩展成多个相近 query
- 把复杂问题拆成几个子问题

### 在 HSBC 项目里怎么说

> 在 HSBC 项目里，用户输入可能是普通话、粤语或者比较口语化的金融表达。比如“开户口”不一定和术语库里的“开立银行账户”字面一致，所以可以用 query rewrite 把它改写成更标准的检索词，再去查术语库或知识库，提高召回率。

### 面试一句话

> Query Rewrite 是把用户原始表达改写成更适合检索的 query，解决用户说法和知识库标准表达不一致的问题。

### 容易踩坑

不要说 Query Rewrite 是“让模型重新回答”。它发生在检索前，目标是提高检索命中率。

## Hybrid Search

### 是什么

Hybrid Search 就是混合检索，通常指把关键词检索和向量检索结合起来。

最常见组合：

```text
BM25 / 关键词检索 + 向量检索 / 语义检索
```

简单理解：

```text
BM25：看字面关键词有没有出现。
向量检索：看语义是否相近。
Hybrid Search：两个都查，再把结果融合。
```

### 为什么需要

单靠关键词检索的问题：

```text
用户说“开户口”，文档写“开立银行账户”，字面不一致，可能查不到。
```

单靠向量检索的问题：

```text
金融术语、产品名、缩写、人名、型号这类专有名词，需要精确命中。
```

所以混合检索的思路是：

```text
关键词检索负责精确命中；
向量检索负责语义召回；
最后把两边结果合并排序。
```

### 怎么融合结果

常见方法有两种：

```text
加权分数：Final Score = 0.7 * Vector + 0.3 * BM25
RRF：不直接比分数，而是看不同检索结果里的排名，再融合排序
```

面试不一定要展开公式，知道 RRF 是一种常见融合排序方法就够了。

### 在 HSBC 项目里怎么说

谨慎口径：

> 我们项目阶段主要依赖 Dify 内置知识库、术语别名和 fallback retrieval。Hybrid Search 可以作为后续优化方向，尤其适合金融术语场景：关键词检索保证 HSBC、DPO、信用卡、账户这类专名能精确命中；向量检索负责处理“开户口”“开立账户”这类语义相近但字面不同的表达。

如果被问“为什么金融场景适合 Hybrid Search”：

> 因为金融术语既需要精确性，也有大量口语化表达。只用向量检索可能把相似但不等价的术语混在一起，只用关键词检索又容易漏掉同义表达，所以混合检索更稳。

### 面试一句话

> Hybrid Search 是把 BM25 关键词检索和向量语义检索结合起来，既保证专有名词精确命中，又提高口语化表达和同义表达的召回率。

### 容易踩坑

不要把 Hybrid Search 和 Rerank 混成一件事。

```text
Hybrid Search：召回阶段，多路检索，把候选找全。
Rerank：排序阶段，对候选结果重新打分，把最相关的排前面。
```

## 知识库检索、关键词检索、向量检索、Embedding

### 它们是不是一回事？

不是一回事。可以按层级理解：

```text
知识库：存放文档、术语、规则的地方
知识库检索：从知识库里找相关内容这个动作
关键词检索：知识库检索的一种方法，按字面词匹配
向量检索：知识库检索的一种方法，按语义相似度匹配
Embedding：向量检索用到的文本数字表示
```

类比：

```text
知识库 = 图书馆
知识库检索 = 去图书馆找资料
关键词检索 = 按书名/关键词找
向量检索 = 按“意思相近”找
Embedding = 把每本书和你的问题变成坐标，方便比较距离
```

### 关键词检索是不是知识库检索？

关键词检索可以是知识库检索的一种方式，但两者不能画等号。

关键词检索常见实现是 BM25 或倒排索引，核心是看查询词有没有在文档里出现，以及出现得重不重要。

适合：

- 专有名词
- 产品名
- 缩写
- 金融术语
- 固定 Gloss

例子：

```text
query：HSBC 信用卡
关键词检索会优先找包含 HSBC、信用卡 的知识库内容
```

### 向量检索是不是 Embedding？

更准确地说：向量检索依赖 Embedding，但向量检索不等于 Embedding。

流程是：

```text
文档入库：
文档/术语/规则 -> 切分 chunk -> 用 Embedding 模型转成向量 -> 存入向量库

用户查询：
用户 query -> 用同一个 Embedding 模型转成向量 -> 和文档向量比较相似度 -> 返回最相近的内容
```

Embedding 是“把文本变成一串数字”的步骤；向量检索是“拿这些数字去找相似内容”的过程。

例子：

```text
“开户口”
“开立银行账户”
“account opening”
```

这几个字面不完全一样，但语义接近，所以向量检索可能把它们找出来。

### Query 是怎么被修改的？

Query Rewrite 通常不是改用户原文，也不是直接改最终答案，而是生成一个更适合检索的内部 query。

常见方式有三种：

1. 术语表/别名表改写

```text
开户口 -> 开立银行账户
户口 -> 银行账户
信用卡申请 -> 申请信用卡
```

这种最稳，因为来自人工维护的术语库或别名表。

2. LLM 改写

用 Prompt 要求模型只做检索改写，例如：

```text
请把用户输入改写成适合检索金融术语库的标准查询。
不要回答问题，只输出 3 个检索 query。

用户输入：我想开户口
输出：
1. 开立银行账户
2. 申请银行账户
3. account opening
```

3. 多轮上下文补全

```text
上一轮：信用卡怎么申请？
这一轮：那需要什么材料？

改写后：申请信用卡需要什么材料？
```

### 在 HSBC 项目里怎么说

> 我会把知识库检索理解成一个大概念，里面可以用关键词检索、向量检索或混合检索。关键词检索适合金融术语和固定 Gloss 的精确匹配，向量检索依赖 Embedding，适合处理“开户口”和“开立银行账户”这种语义相近但字面不同的表达。Query rewrite 则是在检索前把用户的口语表达改成更标准的检索 query，提高命中率。

### 面试一句话

> 知识库检索是从知识库找资料，关键词检索和向量检索是两种找法；Embedding 是向量检索背后的文本向量表示；Query Rewrite 是检索前把用户问题改写成更容易命中的查询。

## 父子块 / Parent-Child Chunking

### 是什么

父子块是一种知识库切分方式，也可以叫分层切分或 Hierarchical Chunking。

简单理解：

```text
父块：大块，保留完整上下文
子块：小块，方便精准检索
```

更口语一点：

```text
子块负责“被搜到”；
父块负责“讲完整”。
```

### 为什么需要

普通 chunk 切分会遇到两个问题：

```text
切太大：检索不够准，很多无关内容混在一起。
切太小：检索准了，但上下文断了，模型不知道规则完整含义。
```

父子块就是折中：

```text
用小的子块做检索，提高命中率；
命中后带出对应父块，让模型看到完整规则。
```

### 在规则库里怎么做

规则类知识特别适合父子块，因为一条规则通常需要上下文。

父块可以是一条完整规则：

```text
rule_id：R001
规则：时间信息优先
适用条件：句子里出现今天、明天、下周等时间表达
转换方式：时间 -> 人物/地点 -> 动作
例子：我今天去银行开户 -> 今天 我 银行 开户
反例：没有时间信息时不强行补时间
```

子块可以拆成：

```text
子块 1：今天 / 明天 / 下周 / 时间表达
子块 2：我今天去银行开户 -> 今天 我 银行 开户
子块 3：时间信息在 HKSL 语序中优先
```

用户输入“我今天去银行开户”时，可能先命中子块 1 或子块 2；系统再把父块 R001 的完整规则召回给模型。

### 在 HSBC 项目里怎么说

> 在规则知识库里，我做过父子块。因为 HKSL 语法规则不是单个词能说明白的，一条规则通常有适用条件、例子和边界。如果直接切成很小的 chunk，模型可能只看到一个例句，缺少完整规则；如果整条规则作为一个大 chunk，又可能检索不够精准。所以我会用子块承接关键词、触发条件和例句，用父块保留完整规则上下文。检索时小块更容易命中，生成时再把父块上下文给模型。

### 面试一句话

> 父子块的核心是“小块检索，大块提供上下文”。在我的规则知识库里，子块用于命中触发词和例句，父块用于保留完整 HKSL 语法规则，避免模型只拿到碎片化知识。

### 容易踩坑

不要说父子块能自动保证规则正确。它只是组织和召回知识的方式，规则正确性仍然来自专家确认、测试集验证和 bad case 回流。

## Top-K

### 是什么

Top-K 指检索系统返回最相关的前 K 条结果。

简单理解：

```text
K = 3  -> 返回最相关的前 3 条
K = 5  -> 返回最相关的前 5 条
K = 10 -> 返回最相关的前 10 条
```

例子：

```text
query：开户口

Top-3：
1. 开立银行账户
2. 银行账户申请
3. 账户管理
```

这里的 Top-K 不是说这 K 条一定都正确，而是检索系统认为它们最相关。

### 为什么要设置 K

因为知识库里可能有很多内容，不能全部塞给模型。Top-K 就是在控制：

```text
从知识库里拿多少条候选内容给后面的 LLM 使用。
```

K 太小：

```text
可能漏掉真正有用的内容。
```

K 太大：

```text
会带入很多噪音，占用上下文，让模型更容易混乱。
```

所以 K 是召回率和准确率之间的平衡：

```text
K 大一点：更不容易漏，但噪音变多。
K 小一点：更干净，但可能漏掉关键术语。
```

### Top-K 和 Fallback 的关系

第一次检索时，如果 Top-K 结果为空、分数低，或者前几条没有命中关键术语，就说明第一次检索可能不够好。

这时可以触发 fallback：

```text
原始 query：开户口
第一次 Top-K：没有命中“开立银行账户”

fallback：
1. query rewrite：开户口 -> 开立银行账户
2. 扩大 Top-K：从 Top-3 扩到 Top-10
3. 必要时换检索方式：向量检索 -> 关键词 + 向量
```

### 在 HSBC 项目里怎么说

> Top-K 是检索阶段返回的前 K 个候选结果。比如输入“开户口”，系统会从术语库或知识库里返回最相关的几个候选。如果 Top-K 为空，或者前几条没有命中“开立银行账户 / ACCOUNT_OPEN”这类核心术语，就说明第一次检索质量不够，可以触发 query rewrite、扩大 Top-K 或 fallback retrieval。

### 面试一句话

> Top-K 就是检索系统返回最相关的前 K 条结果。K 太小容易漏召回，K 太大容易引入噪音，所以通常要结合命中率、上下文长度和下游生成效果来调。

### 容易踩坑

不要把检索里的 Top-K 和模型生成参数里的 top-k 混成一件事。

```text
RAG 检索 Top-K：从知识库拿前 K 条相关内容。
模型采样 top-k：生成下一个 token 时，只在概率最高的 K 个 token 里选。
```

面试中如果上下文是 RAG，Top-K 通常指检索返回多少条候选。

## 扩大 Top-K、Rerank、最后怎么用

### 先记一句话

```text
扩大 Top-K：先把可能正确的候选捞进来，解决漏召回。
Rerank：再把候选重新排序，解决排序不准。
最终取 Top-N：只把最有用的少量结果交给 LLM。
```

### 为什么正确答案排第 6 时要扩大 K

如果系统只取 Top-5：

```text
Top-1：错
Top-2：相关但不准确
Top-3：错
Top-4：错
Top-5：错
Top-6：正确
```

那正确答案不会进入 Prompt，LLM 根本看不到它。

这说明：

```text
Recall@5 失败。
Recall@10 可能成功。
```

所以扩大 K 的目的不是把更多内容都塞给 LLM，而是先扩大候选池：

```text
原来：向量检索 Top-5
改成：向量检索 Top-20
```

这样正确答案如果在第 6、第 8、第 10，就有机会先被召回。

### Top-20 会全部给 LLM 吗

一般不会。Top-20 噪音太多，直接塞给 LLM 会让上下文变乱。

更常见的做法是：

```text
第一步：粗召回
向量检索 / 关键词检索召回 Top-20 或 Top-50

第二步：重排序
Rerank 对候选重新打分

第三步：精选上下文
取重排后的 Top-3 或 Top-5 给 LLM
```

所以完整逻辑是：

```text
扩大 K 不是最终答案，只是扩大候选池。
Rerank 之后再筛掉噪音。
```

### Rerank 的规则是什么

Rerank 可以理解为“第二轮更认真地排序”。

常见有三种方式。

第一种：模型 reranker。

它会把 `query + 候选内容` 放在一起判断相关性，而不是只看两个向量距离。

例子：

```text
query：开户口

候选 A：账户余额查询      0.41
候选 B：信用卡申请        0.35
候选 C：开立银行账户      0.92
候选 D：银行网点地址      0.20
```

重排后：

```text
Top-1：开立银行账户
Top-2：账户余额查询
Top-3：信用卡申请
```

第二种：业务规则 rerank。

在金融或术语库场景里，可以给某些候选加分或降分：

```text
命中术语库标准词：加分
命中别名表：加分
有固定 Gloss：加分
和输入业务场景一致：加分
只是字面相近但业务含义不同：降分
缺少必要 metadata：降分
```

比如用户输入“开户口”：

```text
开立银行账户：命中别名 + 有固定 Gloss -> 加分
账户余额查询：也有“账户”，但不是开户 -> 降分
```

第三种：Hybrid Search 的融合排序。

如果同时用了关键词检索和向量检索，可以融合两个排序结果。

```text
某候选在关键词检索里排第 1
在向量检索里排第 6
```

它可能值得被提到前面，因为关键词命中了关键术语。

### 最后怎么解决

完整链路可以这样说：

```text
query：开户口
  ↓
扩大召回：向量检索 Top-20 / 关键词检索 Top-20
  ↓
Rerank：模型重排 + 业务规则加权
  ↓
精选上下文：取重排后的 Top-3 / Top-5
  ↓
LLM 生成：使用召回到的标准术语和 Gloss
  ↓
输出校验：检查最终是否用了 ACCOUNT_OPEN，JSON 是否合法
```

所以不是“扩大 K 就解决”，而是：

```text
扩大 K 解决漏召回；
Rerank 解决排序不准；
最终校验解决有没有被正确采用。
```

### 句子级 Top-K 还是词语级 Top-K

看任务粒度。

普通 RAG 问答：

```text
用整个 query / 整个问题去检索文档 chunk。
```

术语匹配或 Gloss 项目：

```text
整句检索 + 关键短语/实体检索结合。
```

比如：

```text
输入：我想开户口并申请信用卡

整句检索：
找整体场景和语法规则

短语检索：
开户口 -> ACCOUNT_OPEN
信用卡 -> CREDIT_CARD
```

不要机械地对每个分词都做 Top-K。普通分词太细，会带来大量噪音。

面试一句话：

> Top-K 的对象要和任务粒度一致。普通 RAG 用整个 query 检索文档 chunk；术语匹配可以先抽取关键短语或实体，再分别检索术语库。扩大 K 是为了提高召回，Rerank 是为了把真正相关的候选排到前面，最后只把少量高质量候选交给 LLM。

## RAG 检索链路速记

### 一条完整链路

可以先把这些概念串成下面这条线：

```text
用户输入
  ↓
Query Rewrite
把口语/省略/别名改成更适合检索的 query
  ↓
知识库检索
从术语库、规则库、文档库里找相关内容
  ↓
Top-K
返回最相关的前 K 条候选
  ↓
Hybrid Search（可选）
关键词检索 + 向量检索一起找，减少漏检
  ↓
Fallback Retrieval
如果第一次结果不好，就改写 query、扩大 Top-K 或换检索方式再查
  ↓
校验
看术语是否命中、语义是否相关、JSON 是否合法
```

### 用 HSBC 例子串起来

```text
用户输入：我想开户口

Query Rewrite：
开户口 -> 开立银行账户 / 申请银行账户 / account opening

关键词检索：
找包含“银行账户”“开户”“account opening”的条目

向量检索：
找语义上接近“开立银行账户”的条目

Top-K：
返回最相关的前几条候选

Fallback：
如果没有命中 ACCOUNT_OPEN，就继续查别名、扩大 Top-K 或换检索方式

校验：
确认最终 Gloss 使用 ACCOUNT_OPEN，JSON 字段完整
```

### 一句话总串

> Query Rewrite 负责把问题改好，关键词检索和向量检索负责从知识库里找资料，Top-K 决定拿前几条候选，Hybrid Search 是两种检索一起用，Fallback Retrieval 是第一次没查好时换一种方式再查，最后用术语和 JSON 校验判断结果能不能用。

## JSON 校验、JSON Schema、结构化输出

### JSON 是什么

JSON 可以理解为一种固定格式的数据表述方式，常用于系统之间传数据。

例子：

```json
{
  "gloss": ["TODAY", "ME", "BANK", "ACCOUNT_OPEN"],
  "confidence": 0.86,
  "unmatched_phrases": []
}
```

它比普通文本更适合给下游系统用，因为字段清楚、机器容易解析。

### 只会肉眼看 JSON 的问题

肉眼只能大概看出“像不像”，但系统需要自动判断：

```text
1. 这是不是合法 JSON？
2. 必填字段有没有缺？
3. 字段类型对不对？
4. 字段值是否符合业务规则？
```

比如下面这个就不是合法 JSON，因为少了引号：

```text
{gloss: ["TODAY", "BANK"]}
```

下面这个虽然是合法 JSON，但不符合业务要求，因为 `gloss` 应该是数组，却变成了字符串：

```json
{
  "gloss": "TODAY ME BANK",
  "confidence": 0.86
}
```

### JSON 合法 vs Schema 合法

这两个不是一回事。

```text
JSON 合法：能不能被 JSON parser 解析。
Schema 合法：字段名、字段类型、必填项是否符合业务定义。
```

例子：

```json
{
  "gloss": ["TODAY", "ME", "BANK"],
  "confidence": 0.86
}
```

如果业务要求还必须有：

```text
source_text
language
unmatched_phrases
```

那上面的 JSON 语法没错，但 Schema 校验仍然失败，因为缺字段。

### JSON Mode、Structured Outputs、Pydantic

先按面试够用程度理解：

```text
JSON Mode：保证模型输出合法 JSON，但不一定字段都对。
Structured Outputs / JSON Schema：要求输出符合指定字段和类型。
Pydantic：Python 里常用的 Schema 和类型校验工具。
```

在工程里，常见流程是：

```text
LLM 输出
  ↓
JSON parser 检查语法
  ↓
JSON Schema / Pydantic 检查字段和类型
  ↓
业务规则检查
```

### 校验失败怎么办

常见处理方式：

```text
1. 简单格式问题：自动修复或让模型按错误信息重生成。
2. 缺字段/类型错：把校验错误反馈给模型，让它只修正 JSON。
3. 术语不确定：标记 low confidence，进入人工/专家审核。
4. 连续失败：停止重试，记录日志，避免死循环。
```

重试次数一般要有限制，比如 2-3 次。不能无限让模型重试。

### 在 HSBC 项目里怎么说

> 在 HSBC 项目里，JSON 校验不是靠肉眼看，而是分两层：第一层检查模型输出是不是合法 JSON，第二层检查它是否符合我们定义的字段和类型，比如 gloss 是否是数组、unmatched_phrases 是否存在、confidence 是否是数字。校验失败时，可以把错误信息反馈给模型重新生成，或者由 Code Node 做简单修复；如果连续失败，就标记为低置信，记录 case，进入人工或专家反馈。

### 面试一句话

> JSON 校验分语法校验和 Schema 校验。语法校验看能不能解析，Schema 校验看字段、类型和必填项是否符合业务要求；失败后有限重试，仍失败就低置信或转人工，不能让错误 JSON 直接进入下游动画系统。

## Parser、Schema、Code Node

### 先用一句话理解

```text
Parser：把模型输出的字符串读成机器能处理的数据。
Schema：规定这个数据必须长什么样。
Code Node：在工作流里用代码做确定性处理和校验。
```

它们通常是连在一起的：

```text
LLM 输出文本
  ↓
Parser 解析
  ↓
Schema 校验
  ↓
Code Node 修复 / 拼接 / 判断是否重试
```

### Parser 是什么

Parser 就是解析器。它负责把一段字符串转换成程序能处理的数据结构。

比如模型输出：

```json
{
  "gloss": ["TODAY", "ME", "BANK", "ACCOUNT_OPEN"],
  "confidence": 0.86
}
```

在 Python 里可以用 `json.loads()` 把它从字符串变成字典；在 JavaScript 里可以用 `JSON.parse()` 把它变成对象。

如果模型输出成这样：

```text
好的，下面是结果：
{"gloss": ["TODAY", "BANK"]}
```

或者这样：

```text
{gloss: ["TODAY", "BANK"]}
```

parser 就可能失败，因为第一种混了额外解释文字，第二种不是合法 JSON。

面试里可以这样说：

> Parser 解决的是“模型输出能不能被机器读懂”的问题。它先不关心业务对不对，只看格式能不能被解析。

### Parser 有哪些常见写法

最常见的是标准 JSON parser：

```python
import json

data = json.loads(raw_output)
```

或者 JavaScript：

```js
const data = JSON.parse(rawOutput);
```

这类 parser 只检查 JSON 语法是否合法。比如括号、引号、逗号、冒号有没有问题，能不能被程序读成对象或字典。

有时候模型会在 JSON 外面多说一句话，比如：

```text
好的，下面是结果：
{"gloss": ["TODAY", "BANK"]}
```

这时可以先提取 JSON 块，再交给 parser：

```python
import json
import re

match = re.search(r"\{.*\}", raw_output, re.DOTALL)
if not match:
    raise ValueError("未找到 JSON")

data = json.loads(match.group())
```

但这只是补救。更好的做法是在 Prompt、JSON Mode 或 Structured Outputs 里尽量让模型一开始就只输出 JSON。

还有一种叫结构化输出 parser，比如 Pydantic：

```python
from pydantic import BaseModel

class HKSLGlossOutput(BaseModel):
    source_text: str
    language: str
    gloss: list[str]
    unmatched_phrases: list[str]
    confidence: float

data = json.loads(raw_output)
result = HKSLGlossOutput(**data)
```

这里实际上做了两件事：

```text
json.loads：parser，检查 JSON 语法。
HKSLGlossOutput(**data)：Schema 校验，检查字段和类型。
```

### Schema 是什么

Schema 可以理解为数据格式约定，类似一张表格模板。它规定：

```text
必须有哪些字段
每个字段是什么类型
哪些字段可以为空
字段值有没有范围限制
```

比如 HSBC HKSL Gloss 项目里可以有这样的业务约定：

```text
source_text：原始文本，字符串
language：输入语言，字符串
gloss：Gloss 列表，数组
unmatched_phrases：未匹配短语，数组
confidence：置信度，数字
```

下面这个 JSON 语法上是合法的：

```json
{
  "gloss": "TODAY ME BANK",
  "confidence": "high"
}
```

但它不符合 Schema，因为：

```text
gloss 应该是数组，不应该是字符串。
confidence 应该是数字，不应该是 "high"。
source_text、language、unmatched_phrases 可能缺失。
```

面试里可以这样说：

> JSON parser 只检查语法，Schema 检查字段和类型。一个输出可能是合法 JSON，但仍然不符合业务 Schema。

### Schema 是人工检查吗

不是人工一条一条看。更准确地说：

```text
人定义 Schema；
系统按照 Schema 自动检查每次输出。
```

比如人先定义业务规则：

```text
source_text 必须存在，类型是 string
language 必须存在，类型是 string
gloss 必须存在，类型是 list[str]
unmatched_phrases 必须存在，类型是 list[str]
confidence 必须存在，类型是 float，而且最好在 0 到 1 之间
```

然后程序自动检查。用 Pydantic 可以这样写：

```python
from pydantic import BaseModel, Field

class HKSLGlossOutput(BaseModel):
    source_text: str
    language: str
    gloss: list[str]
    unmatched_phrases: list[str] = []
    confidence: float = Field(ge=0, le=1)
```

如果模型输出里 `gloss` 是字符串、`confidence` 是 `"high"`，或者少了必填字段，Schema 校验就会失败。人工主要负责定义规则和处理高风险 bad case，不是每条都靠肉眼检查。

### 规则限制和自动校验的区别

可以把它们分成三层：

```text
生成前约束：Prompt / JSON Mode / Structured Outputs
生成后校验：Parser / Schema / Code Node
业务正确性：术语库 / 规则库 / 专家反馈
```

Prompt 是软约束，告诉模型“你应该这样输出”。但模型仍然可能出错。

Parser 和 Schema 是硬校验，用程序判断“它到底有没有按要求输出”。

业务规则是人定义的，比如：

```text
金融术语优先使用术语库里的标准 Gloss。
没有未匹配短语时，unmatched_phrases 填 []。
confidence 必须是 0 到 1 之间的数字。
不确定的 Gloss 不能直接进入下游动画系统。
```

所以不能说“靠规则限制就够了”。更准确的说法是：

> 规则先由人定义，再通过 Prompt、Schema 和 Code Node 在系统里执行；Prompt 负责引导，Schema 和 Code Node 负责自动校验和兜底。

### Code Node 是什么

Code Node 是 Dify、n8n 这类工作流工具里的“代码节点”。它不是让模型自由发挥，而是用确定性代码处理某一步。

常见用途：

```text
1. 检查 JSON 字段是否完整。
2. 给缺失字段补默认值，比如 unmatched_phrases 补成 []。
3. 把检索结果和模型输出拼接成最终 JSON。
4. 检查 gloss 是否是数组、confidence 是否是数字。
5. 返回错误信息，让上游 LLM 重新生成。
6. 连续失败时标记 low confidence 或 needs review。
```

它和 LLM 的区别：

```text
LLM：适合语义理解、改写、生成初稿。
Code Node：适合规则固定、必须稳定的处理。
```

### 为什么需要它们

因为 Prompt 里写“请输出 JSON”不等于工程上真的可靠。模型可能：

```text
多输出解释文字
字段名写错
数组写成字符串
数字写成文字
漏掉必填字段
```

如果下游是手语动画系统，它需要的是稳定的结构化数据，不能靠人工肉眼看。所以要用 parser、Schema 和 Code Node 把模型输出变成可检查、可修复、可追踪的工程流程。

### 前面内容对了，后面为什么还会出问题

这里要分清楚两件事：

```text
内容正确：术语、Gloss、语序大方向是对的。
结构可用：输出能被下游系统稳定解析和使用。
```

前面检索和生成可能已经知道：

```text
开户口 -> ACCOUNT_OPEN
```

但最后输出时仍然可能变成：

```json
{
  "gloss": "ACCOUNT_OPEN"
}
```

这里内容没错，但结构错了，因为 `gloss` 应该是数组：

```json
{
  "gloss": ["ACCOUNT_OPEN"]
}
```

也可能出现这种情况：

```text
下面是转换结果：
{
  "gloss": ["ACCOUNT_OPEN"]
}
```

人看得懂，但机器 parser 可能读不进去，因为 JSON 外面多了说明文字。

还有一种情况是聚合环节出错：

```text
检索结果：ACCOUNT_OPEN
最终输出：BANK_ACCOUNT
```

这说明后置校验不是在“重新做所有内容判断”，而是在发现输出阶段、聚合阶段或前面环节残留的问题。

面试里可以这样说：

> 前面环节负责让内容尽量正确，比如检索术语、生成 Gloss 初稿；后面结构化校验负责确认这些内容有没有被包装成下游系统能使用的稳定 JSON。理想情况下内容问题应该在前面解决，但工程上仍然需要后置校验来发现字段丢失、类型错误、多余文本、术语没有被正确采用等问题。

一句话：

> 前面负责“内容尽量对”，后面负责“输出一定能用”。

### 校验失败怎么避免死循环

不能让模型无限重试。常见做法是：

```text
设置最大重试次数，比如 2-3 次。
每次重试带上具体错误信息。
重试仍失败就停止，标记 low confidence 或 needs review。
记录 bad case，回流到 Prompt、知识库或规则库。
```

重试时不是简单重复问模型，而是把错误说清楚：

```text
错误：gloss 字段应为数组，但当前是字符串。
要求：只修正 JSON 结构，不要改写 Gloss 内容。
```

### 什么能自动修，什么不能自动修

可以自动修的通常是格式类问题：

```text
JSON 外面多了说明文字。
缺少 unmatched_phrases，可以补 []。
gloss 是字符串但明显可以转成数组。
confidence 缺失时可以填默认值或标记 low confidence。
字段顺序不一致。
```

不能硬修的通常是内容类问题：

```text
Gloss 术语不确定。
金融术语没有命中。
HKSL 语序不合理。
检索结果和最终输出冲突。
模型生成了知识库没有确认过的 Gloss。
```

内容类问题要回到前面的环节排查：

```text
检索没召回 -> 调整 query rewrite / fallback / Top-K
知识库缺词 -> 补术语库和别名
Prompt 没约束好 -> 增加规则和 few-shot
专家不认可 -> 记录 bad case，更新规则库或测试集
```

面试里可以这样说：

> 格式类问题可以通过 Schema 报错、Code Node 或有限重试自动修复；内容类问题不能靠自动修复硬补，要回溯到检索、知识库、Prompt 或专家反馈环节处理。

### 在 HSBC 项目里怎么串

可以这样理解：

```text
用户输入：我今天去银行开户
  ↓
LLM / Workflow 生成 HKSL Gloss JSON 字符串
  ↓
Parser 检查这是不是合法 JSON
  ↓
Schema 检查 gloss、confidence、unmatched_phrases 等字段是否符合要求
  ↓
Code Node 做简单修复、字段补齐、结果拼接或返回错误
  ↓
如果通过，交给下游动画系统；如果失败，重试或标记人工审核
```

面试回答：

> 在我的项目里，Prompt 约束只是第一层。模型输出后，还需要 parser 检查它是不是合法 JSON，再用 Schema 检查字段和类型，比如 gloss 必须是数组、confidence 必须是数字、unmatched_phrases 必须存在。Code Node 则负责一些确定性的逻辑，比如补默认字段、拼接检索结果、返回校验错误或触发重试。这样可以避免错误格式直接进入下游手语动画系统。

### 容易踩坑

不要把 Schema 只理解成数据库 Schema。在这里它更多指 JSON 输出结构的约定。

不要把 Code Node 说成 Agent 自己思考。Code Node 更像工作流里的固定程序步骤。

不要说“parser 能判断 Gloss 对不对”。Parser 只能判断格式能不能读；Gloss 对不对还要靠术语库、规则库和专家反馈。
