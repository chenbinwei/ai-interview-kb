# LLM 算法岗速记

这个文件服务于“大语言模型算法实习生 / LLM 算法 / 轻量化 / 推理优化”面试。重点不是把自己包装成已经做过底层训练的人，而是把 CV 里的经历讲成：我做过真实数据到模型输入、模型输出评估、bad case 复盘和方案迭代。

## 岗位匹配怎么说

如果问“你为什么投这个岗位”，可以说：

> 我投这个岗位不是因为我已经完整做过大模型量化或剪枝，而是因为我过去的经历和它有几个连接点。第一，我做过文本到结构化表示的转换，里面涉及信息保留、冗余压缩和输出约束；第二，我做过不同 LLM、Embedding 和深度学习模型的效果对比；第三，我比较习惯用固定 case、指标和 bad case 去复盘模型效果。这个岗位做 LLM 信息编码、文本压缩、推理优化，我觉得和我之前做过的文本转换、序列重构和评测迭代是顺着的。

如果问“你没做过量化/剪枝怎么办”，可以说：

> 我不会把量化和剪枝说成已有完整项目经验。更准确地说，我目前的优势在实验复现、数据构造、模型对比和结果复盘。如果进入这个方向，我会先从已有方法复现、指标对齐、bad case 分析和实验报告做起，再逐步深入到具体压缩算法和推理优化实现。

## Transformer 是什么

### 15 秒版

> Transformer 是一种基于 Attention 的神经网络结构。它的核心是让每个 token 根据注意力机制去关注序列中其他 token，从而建模上下文关系。现在主流 LLM 大多是 decoder-only Transformer，用前面的 token 预测下一个 token。

### 60 秒版

> Transformer 主要由词向量、位置编码、多头自注意力、前馈网络、残差连接和 LayerNorm 组成。和 RNN 不同，它不按时间一步步处理序列，而是用 Self-Attention 计算 token 之间的相关性，所以更适合并行训练。对于 GPT 这类自回归模型，会用 masked self-attention，只允许当前位置看到前面的 token，再预测下一个 token。

### 深挖追问

#### 为什么 Transformer 比 RNN 更适合大模型？

> RNN 是按时间步顺序处理序列的，长距离依赖和并行训练都比较困难。Transformer 用 Self-Attention 一次性计算序列中 token 之间的关系，训练并行度更高，也更容易扩展到大模型和长上下文。

#### Encoder-only / Decoder-only / Encoder-Decoder 区别

| 架构 | 代表 | 特点 | 适合任务 |
|---|---|---|---|
| Encoder-only | BERT | 双向看上下文 | 分类、抽取、理解 |
| Decoder-only | GPT / LLaMA | 只能看前文，自回归生成 | 对话、续写、代码生成 |
| Encoder-Decoder | T5 | Encoder 理解输入，Decoder 生成输出 | 翻译、摘要、结构化生成 |

面试一句话：

> 现在主流 LLM 多是 decoder-only，因为它天然适合 next token prediction 和开放式生成。

#### Pre-LN 和 Post-LN

> LayerNorm 放在子层前面叫 Pre-LN，放在残差后面叫 Post-LN。大模型训练里 Pre-LN 更常见，因为梯度更稳定，深层网络更容易训练。

#### LayerNorm 和 RMSNorm

> LayerNorm 会做均值和方差归一化；RMSNorm 只按均方根缩放，不减均值，计算更简单。很多 LLM 使用 RMSNorm，是为了降低计算开销并保持训练稳定。

#### FFN / MLP / SwiGLU

> Transformer 里的 FFN 也叫 MLP，负责对每个 token 的表示做非线性变换。很多现代 LLM 会用 SwiGLU 这类门控激活，比传统 ReLU/GELU 表达能力更强。

#### RoPE 是什么

> RoPE 是 Rotary Position Embedding，旋转位置编码。它不是简单把位置向量加到 embedding 上，而是在 Q/K 空间里用旋转方式注入相对位置信息。很多 LLaMA 系模型使用 RoPE，因为它对长度外推和相对位置建模比较友好。

#### MHA / MQA / GQA

| 名称 | 含义 | 作用 |
|---|---|---|
| MHA | Multi-Head Attention | 每个头都有自己的 Q/K/V |
| MQA | Multi-Query Attention | 多个 Q 头共享一组 K/V |
| GQA | Grouped-Query Attention | 多组 Q 共享较少组 K/V |

面试一句话：

> MQA/GQA 的核心是减少 K/V 头的数量，降低 KV Cache 显存和带宽压力，让推理更高效。GQA 是 MHA 和 MQA 之间的折中。

## Attention 和 Q/K/V

### “Token 互相看”是什么意思

> “互相看”不是字面意思，而是每个 token 都会和其他 token 计算向量相关性，判断哪些上下文对自己更重要，再把重要 token 的信息汇总进自己的新表示里。

可以按这条链理解：

```text
token 向量
-> Q/K 向量关系
-> 相关性分数
-> softmax 权重
-> 加权汇总 V
-> 融合上下文后的 token 表示
```

比如句子是：

```text
我 今天 去 银行 办理 业务
```

处理“银行”这个 token 时，它可能会更关注“办理”“业务”，因为这些词能帮助模型判断这里的“银行”是金融机构，而不是河岸。

### Attention 公式

$$
\mathrm{Attention}(Q, K, V)
=
\mathrm{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

面试先不用推导，先记住一句话：

> Attention 先用 Q 和 K 算出 token 之间的相关性分数，再用 softmax 把分数变成注意力权重，最后用这个权重去加权汇总 V。

每一项的含义：

```text
Q = Query：当前 token 想找什么信息
K = Key：每个 token 能被匹配的特征
V = Value：真正被加权汇总的信息内容
QK^T = 当前 token 和其他 token 的相关性分数
softmax = 把分数变成权重
乘 V = 按权重汇总上下文信息
```

### 从 X 到 Q/K/V

假设一句话有 `n` 个 token，每个 token 的 embedding 维度是 `d_model`，那么输入矩阵是：

$$
X \in \mathbb{R}^{n \times d_{model}}
$$

其中每一行 `x_i` 是第 `i` 个 token 的向量表示。

模型会用三组可学习参数，把同一个输入 `X` 投影成 Q、K、V：

$$
Q = XW_Q,\quad K = XW_K,\quad V = XW_V
$$

维度是：

$$
W_Q \in \mathbb{R}^{d_{model} \times d_k},\quad
W_K \in \mathbb{R}^{d_{model} \times d_k},\quad
W_V \in \mathbb{R}^{d_{model} \times d_v}
$$

$$
Q \in \mathbb{R}^{n \times d_k},\quad
K \in \mathbb{R}^{n \times d_k},\quad
V \in \mathbb{R}^{n \times d_v}
$$

`W_Q/W_K/W_V` 不是人工规则，而是训练时学出来的参数矩阵。

### QK^T 在算什么

$$
S = QK^T
$$

维度是：

$$
(n \times d_k)(d_k \times n)=n \times n
$$

所以 `S` 是一个注意力分数矩阵：

$$
s_{ij}=q_i k_j^T
$$

意思是：

```text
第 i 个 token 的 Query
和第 j 个 token 的 Key
有多相关
```

这里的 `T` 是 transpose，转置。因为 `K` 原本是 `n x d_k`，转置后变成 `d_k x n`，才能和 `Q` 相乘，得到 `n x n` 的两两相关性矩阵。

单个 token 写法和矩阵写法是同一件事：

```text
原始单个分数：s_ij = q_i · k_j
原始矩阵分数：S = QK^T
```

缩放之后可以写成：

$$
R=\frac{S}{\sqrt{d_k}}=\frac{QK^T}{\sqrt{d_k}}
$$

其中：

$$
r_{ij}=\frac{s_{ij}}{\sqrt{d_k}}
$$

### Softmax 之后得到什么

$$
A=\mathrm{softmax}(R)
=
\mathrm{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)
$$

其中：

$$
A \in \mathbb{R}^{n \times n}
$$

`A` 是注意力权重矩阵，里面的每个元素可以写成：

$$
\alpha_{ij}
$$

意思是：

```text
第 i 个 token
应该关注第 j 个 token 多少
```

每一行权重加起来等于 1：

$$
\sum_j \alpha_{ij}=1
$$

### 最后为什么要乘 V

$$
O = AV
$$

维度是：

$$
(n \times n)(n \times d_v)=n \times d_v
$$

第 `i` 个输出向量是：

$$
o_i=\sum_{j=1}^{n}\alpha_{ij}v_j
$$

意思是：

> 第 i 个 token 会根据注意力权重，把所有 token 的 Value 信息加权汇总，得到融合上下文后的新表示。

### 口语解释

> Q/K/V 可以理解成一次检索。Query 是“我现在需要什么”，Key 是“每个位置有什么特征可以被匹配”，Value 是“真正拿来汇总的信息”。模型先用 Q 和 K 算相似度，再用这个相似度对 V 加权求和，得到当前 token 的上下文表示。

### Softmax 是什么

> Softmax 是把一组分数转成一组权重的函数。它会把所有值变成正数，并让它们加起来等于 1。

公式：

$$
\mathrm{softmax}(r_i)
=
\frac{e^{r_i}}{\sum_j e^{r_j}}
$$

这里的 `r_i` 只是泛指 softmax 输入里的第 `i` 个分数。放到 Attention 里，更建议写成 `r_ij`，也就是第 `i` 个 Query 和第 `j` 个 Key 的缩放后相关性分数，避免和最终输出向量混在一起。

在 Attention 里，softmax 的作用是：

```text
把缩放后的相关性分数 R
变成每个 token 应该被关注多少的注意力权重
```

注意：

> Softmax 不只是普通归一化。因为它用了指数函数，会放大高分和低分的差距，让模型更明确地关注高相关 token。

### 为什么除以 sqrt(d_k)

> `QK^T` 是向量点积。维度 `d_k` 越大，点积结果可能越大。如果分数太大，softmax 会变得过于尖锐，某一个 token 权重接近 1，其他接近 0，训练会不稳定。

所以要做缩放：

$$
\frac{QK^T}{\sqrt{d_k}}
$$

面试一句话：

> 除以 `sqrt(d_k)` 是为了避免 Q/K 点积过大导致 softmax 饱和、梯度变小，从而让训练更稳定。

### Self-Attention 复杂度

> Self-Attention 的缺点是每个 token 都要和所有 token 算相关性。序列长度是 `n` 时，注意力矩阵大小是 `n x n`，所以时间和显存压力大致是 `O(n^2)`。

面试一句话：

> Transformer 更容易并行，但长上下文会贵，因为 Self-Attention 对序列长度是平方复杂度。

### Multi-Head Attention

> Multi-Head Attention 是把注意力拆成多个头，每个头在不同子空间里看关系。有的头可能更关注语法关系，有的头更关注长距离依赖。最后把多个头的结果拼起来，再经过线性层融合。

### Attention 符号表

| 符号 | 含义 | 常见维度 |
|---|---|---|
| `n` | token 数量 | 标量 |
| `d_model` | 输入 embedding / hidden 维度 | 标量 |
| `d_k` | Query / Key 维度 | 标量 |
| `d_v` | Value 维度 | 标量 |
| `X` | 输入 token 表示矩阵 | `n x d_model` |
| `W_Q` | Query 投影矩阵 | `d_model x d_k` |
| `W_K` | Key 投影矩阵 | `d_model x d_k` |
| `W_V` | Value 投影矩阵 | `d_model x d_v` |
| `Q` | Query 矩阵 | `n x d_k` |
| `K` | Key 矩阵 | `n x d_k` |
| `V` | Value 矩阵 | `n x d_v` |
| `S` | 注意力原始分数矩阵 | `n x n` |
| `R` | 缩放后的注意力分数矩阵 | `n x n` |
| `A` | softmax 后的注意力权重矩阵 | `n x n` |
| `O` | Attention 输出 | `n x d_v` |

最短记忆链：

```text
X -> Q/K/V
QK^T -> 分数矩阵 S
S / sqrt(d_k) -> 缩放分数矩阵 R
softmax(R) -> 权重矩阵 A
A V -> 融合上下文后的输出 O
```

## Masked Self-Attention

> GPT 这类模型生成文本时不能偷看未来 token，所以会加 mask。第 t 个 token 只能关注第 1 到 t 个位置，不能看 t 之后的内容。这样训练目标才和生成时一致，都是根据前文预测下一个 token。

## KV Cache

### 是什么

> KV Cache 是推理时缓存历史 token 的 Key 和 Value。生成新 token 时，不需要每一步都重新计算所有历史 token 的 K/V，只需要计算当前 token 的 Q，并复用之前缓存的 K/V。

### 为什么能加速

不用 KV Cache：

```text
每生成一个 token，都重新算整段上下文的 Attention。
```

用 KV Cache：

```text
历史 K/V 已经缓存，只算当前 token，再和历史 K/V 做注意力。
```

面试一句话：

> KV Cache 是用显存换推理速度。它减少重复计算，但上下文越长，缓存的 K/V 越多，显存压力也越大。

## Token

> Token 是模型处理文本的基本单位，不一定等于一个字或一个词。中文里一个字可能是一个 token，也可能多个字合成一个 token；英文单词也可能被拆成词根或子词。模型看到的不是原始字符，而是 token id。

## Temperature / Top-k / Top-p

| 参数 | 含义 | 影响 |
|---|---|---|
| Temperature | 控制分布平滑程度 | 越高越发散，越低越保守 |
| Top-k | 只在概率最高的 k 个 token 里采样 | 限制候选范围 |
| Top-p | 只在累计概率达到 p 的候选集合里采样 | 动态控制候选范围 |

面试一句话：

> Temperature 控制随机性，Top-k 和 Top-p 控制采样候选范围。严肃任务一般用低温，让输出更稳定；创意任务可以提高温度。

## SFT

> SFT 是 Supervised Fine-Tuning，监督微调。它用人工标注或整理好的 instruction-response 数据，让预训练模型学会按照指令回答。预训练学的是语言规律，SFT 让模型更像一个能听懂任务的助手。

## LoRA

### 是什么

> LoRA 是一种参数高效微调方法。它不直接更新大模型原始权重，而是冻结原模型，在部分线性层旁边加一个低秩矩阵更新，只训练这小部分参数。

可以记成：

```text
原来要更新 W
LoRA 改成：W + ΔW
其中 ΔW = B A，A 和 B 是低秩小矩阵
```

### 为什么省

> 因为训练参数少很多，显存和训练成本都更低。多个任务还可以保存多个 LoRA adapter，需要哪个任务就加载哪个 adapter。

### 容易踩坑

不要说：

```text
LoRA 是压缩模型。
```

更准确：

```text
LoRA 是参数高效微调。它减少训练时需要更新的参数量，但不等同于量化或剪枝。
```

### 深挖追问

#### LoRA 一般加在哪里？

> LoRA 通常加在 Transformer 的线性层上，最常见是 Attention 里的 q_proj、v_proj，有时也会加 k_proj、o_proj 或 MLP 层。加在哪里取决于任务和显存预算。

面试稳妥说法：

> 如果只做轻量适配，常见做法是先加 q_proj 和 v_proj；如果任务差异更大，可以扩展到更多投影层或 MLP 层，但训练参数和显存也会上升。

#### rank r 是什么？

> rank r 控制 LoRA 低秩矩阵的容量。r 越大，可训练参数越多，表达能力更强，但显存和过拟合风险也更高。r 太小可能学不动任务，r 太大又接近完整微调的成本。

#### alpha 是什么？

> alpha 是 LoRA 更新量的缩放系数，常和 rank 一起决定 LoRA 对原模型的影响强度。可以粗略理解为：rank 控制容量，alpha 控制 LoRA 更新对原模型输出的影响幅度。

#### LoRA 为什么能减少灾难性遗忘？

> 因为原模型权重被冻结，没有直接被新任务数据覆盖。LoRA 只训练额外 adapter，所以原模型通用能力更容易保留。需要不同任务时，也可以切换不同 adapter。

#### LoRA 和全量微调区别

| 维度 | 全量微调 | LoRA |
|---|---|---|
| 更新参数 | 更新全部或大部分权重 | 冻结原模型，只训练 adapter |
| 显存成本 | 高 | 低 |
| 训练速度 | 慢 | 快 |
| 多任务管理 | 每个任务一套模型成本高 | 多个 adapter 可切换 |
| 上限 | 通常更高 | 取决于 rank 和任务差异 |

#### LoRA 和 QLoRA

> QLoRA 是在量化后的基础模型上训练 LoRA。常见做法是把 base model 量化到 4-bit，冻结它，然后训练 LoRA adapter。这样可以大幅降低微调显存。

不要说：

```text
QLoRA 是把 LoRA 量化。
```

更准确：

```text
QLoRA = 4-bit 量化 base model + LoRA adapter 微调。
```

#### LoRA 可以合并回模型吗？

> 可以。训练完成后，可以把 LoRA 的增量权重 merge 回原模型权重，推理时就不需要单独加载 adapter。但如果要频繁切换多个任务，也可以保留 adapter 形式。

#### LoRA 的局限

```text
1. 如果任务和原模型能力差距很大，LoRA 可能不够。
2. rank 太小会欠拟合，太大又增加成本。
3. 数据质量仍然很关键，LoRA 不能解决脏数据问题。
4. 它主要降低微调成本，不直接解决推理压缩问题。
```

## 量化 Quantization

> 量化是把模型权重或激活从高精度表示，比如 FP16，转换成更低精度，比如 INT8、INT4。这样可以减少显存占用和内存带宽压力，提高推理速度，但可能带来精度损失。

常见两类：

| 方法 | 含义 |
|---|---|
| PTQ | Post-Training Quantization，训练后量化，用校准数据统计分布后量化 |
| QAT | Quantization-Aware Training，训练时模拟量化误差，让模型适应低精度 |

面试一句话：

> 量化的核心 trade-off 是效率和精度。INT8 通常比较稳，INT4 压缩更狠但更容易掉点，需要校准、分组量化或更复杂的方法来控制误差。

### Q4 / Q8 / Q4_K_M 是什么意思

注意：这里的 Q 不是 Attention 里的 Query，而是 Quantization。

```text
Q8 = 8-bit 量化
Q4 = 4-bit 量化
```

本地模型里经常看到：

```text
Q8_0
Q6_K
Q5_K_M
Q4_K_M
Q4_0
Q4_1
```

可以先这样记：

```text
数字越大，一般质量越稳，占用越大。
数字越小，一般越省显存，质量风险越高。
```

#### K 是什么

> 在 Q4_K_M 里，K 不是 Q/K/V 的 Key，而是 GGUF / llama.cpp 里的 K-quants 量化格式，可以理解成一类改进的分组量化方案，通常比老的 Q4_0、Q4_1 效果更好。

#### M 是什么

> M 通常表示 Medium，是质量和大小之间的中等折中版本。常见还有 S = Small，L = Large。

所以：

```text
Q4_K_M = 4-bit + K-quants 改进量化方案 + Medium 折中版本
```

面试稳妥说法：

> 这些后缀不是所有框架的通用标准，主要常见于 GGUF / llama.cpp 本地模型。面试里可以说清大意：Q4 表示 4-bit，K 表示一类改进分组量化，M 表示中等折中版本。

### W4A16 / W8A8

```text
W = Weight，权重
A = Activation，激活
```

| 格式 | 含义 | 特点 |
|---|---|---|
| W4A16 | 权重 4-bit，激活 16-bit | 比较常见，显存省，精度较稳 |
| W8A8 | 权重 8-bit，激活 8-bit | 加速潜力更大，但实现和精度控制更难 |
| W4A8 | 权重 4-bit，激活 8-bit | 更激进，对 kernel 和校准要求更高 |

### Scale / Zero Point

量化通常不是随便四舍五入，而是把浮点数映射到整数：

```text
real_value ≈ scale * (quantized_value - zero_point)
```

```text
scale：缩放比例
zero_point：零点偏移
dequantization：反量化，把低精度值近似还原到浮点计算空间
```

### Per-tensor / Per-channel / Group-wise

| 粒度 | 含义 | 特点 |
|---|---|---|
| Per-tensor | 整个张量共用一套 scale | 简单，但误差可能大 |
| Per-channel | 每个通道单独 scale | 更准，复杂一些 |
| Group-wise | 分组量化，每组一套 scale | LLM 里常见，折中效果好 |

面试一句话：

> 量化粒度越细，通常精度越好，但元数据和实现复杂度也更高。

### Calibration 和 Outlier

> Calibration 是用一小批校准数据统计权重或激活分布，决定量化 scale。校准数据如果和真实任务差异很大，量化后效果可能会掉。

> Outlier 是异常大的激活或权重通道。直接全局量化时，outlier 会拉大量化范围，让普通值表示得很粗，导致精度损失。所以很多方法会用 per-channel、group-wise 或特殊 outlier 处理。

### GPTQ / AWQ

> GPTQ 和 AWQ 都是 LLM 里常见的 INT4 权重量化方法。GPTQ 更偏数学优化，逐层最小化量化误差；AWQ 会根据激活分布找出重要权重并保护，量化更快，工程上更实用。

### 量化怎么评估

不要只看模型大小，要同时看：

```text
显存下降多少
推理吞吐 tokens/s
首 token 延迟和总延迟
perplexity 是否变差
下游任务准确率
数学、代码、长文本、专业术语 bad case 是否增加
```

面试一句话：

> 量化不是压得越狠越好，而是在显存、吞吐、延迟和任务质量之间找平衡。

## 剪枝 Pruning

> 剪枝是把模型里不重要的权重、神经元、通道或 Attention Head 去掉，减少计算和参数。它的难点是判断哪些部分“不重要”，以及剪完之后怎么恢复精度。

一句话：

> 量化是降低数值精度，剪枝是减少模型结构或参数数量。

## 蒸馏 Distillation

> 蒸馏是用大模型当 teacher，小模型当 student，让小模型学习大模型的输出分布或中间表示。目标是在保留尽量多能力的同时，让模型更小、更快、更便宜。

## Prompt 压缩 / 文本压缩

> Prompt 压缩不是简单把文本变短，而是保留对任务有用的信息，删掉冗余内容，减少 token 消耗和推理成本。好的压缩要看下游任务效果，而不是只看压缩率。

结合 HSBC 项目可以说：

> 在手语项目里，原始中文或粤语文本里有些修饰语对手语 Gloss 生成帮助不大，所以需要先做句子修正和内容缩减。但不能乱删，金融术语、时间、人物、动作这些关键信息必须保留。这和 LLM 文本压缩里的核心问题类似：减少冗余，同时保住任务相关信息。

## 面试最后速记

```text
Transformer：Attention 架构，用上下文预测下一个 token。
Q/K/V：Query 找信息，Key 被匹配，Value 被汇总。
Attention：softmax(QK^T / sqrt(d_k))V。
Masked Attention：生成时不能看未来。
KV Cache：缓存历史 K/V，用显存换速度。
Token：模型处理文本的基本单位。
LoRA：冻结原模型，只训练低秩 adapter。
LoRA rank：控制 adapter 容量；alpha：控制更新强度。
QLoRA：4-bit base model + LoRA 微调。
量化：降低数值精度，省显存和带宽。
Q4_K_M：4-bit + K-quants + Medium 折中格式。
剪枝：删掉不重要结构或参数。
蒸馏：大模型教小模型。
Prompt 压缩：少 token，但保留任务关键信息。
```
