# Pandora 精读笔记 — Words Caught in the Heart: Simulating Person-Specific Expression for Self-Opaque Clients in Psychotherapy

> 源文件：`docs/sources/papers/2611.00617-pandora.html`（arXiv HTML 全文，约 370KB，LaTeXML 转换）
> 对照摘要：`docs/sources/papers-abstracts.md` §2（arXiv API 抓取）
> 核查方式：全文 grep `github|code is|code at|anonymous|project page|available at` + 全量 `href` 提取 + arXiv abs 页复核；公式取自 `alttext=` 属性原文。
> 所有数字均为论文自报，未复现。

---

## 1. 论文速览

| 项 | 内容 |
|---|---|
| 题目 | Words Caught in the Heart: Simulating Person-Specific Expression for Self-Opaque Clients in Psychotherapy（框架名 **Pandora**） |
| 作者 | Mengxi Xiao, Jingjing Chen, Ziyan Kuang, Jiarong Song, Wenhao Zhao, Zuchen Han, Yihuai Xu, Xiyang Huang, Jie Gong, Yuechen Jiang, Xueqing Peng, Yankai Chen, Xiaoyan Qu, Min Peng, Qianqian Xie（15 人；HTML 未列机构，未在全文中找到） |
| Venue | **arXiv preprint，无会议/期刊 venue**。arXiv:2611.00617v1 [cs.HC]，submitted 31 Oct 2026（HTML 首页日期 October 23, 2026）；abs 页无 Comments/journal-ref 字段，全文未见 "under review"/录用信息 |
| 代码情况 | **全文未见官方代码发布**（详见 §6）。注意：Appendix A.1/I.3/I.4 三处文字声称 "the repository listed on the first page"，但首页与 abs 页均无任何仓库链接 —— 属悬空声明 |
| 一句话定位 | 把「临床组织的个案表征 ≠ 来访者本人的表达」这一 gap 形式化为「心理经验 → 情境化言语」的四段式管线：状态更新 → 主观归因 → 内容/策略规划 → **独立的 expression planner**，并用 115 案例 + 45 条目四评者人评证明表达规划能同时提高 expression similarity、降低 AI-like 判定率 |
| 资助 | NSFC Key U23A20316、General 62676290、CCF-Tencent Rhino-Bird RAGR20250115、武汉自然科学基金 2026040301020029 等（Acknowledgments） |

核心主张（摘要 + §4.3）：Pandora 在 115 案例上取得最高 expression similarity 与最高 overall disclosure-alignment（GDS）；四评者 45 条目研究中 AI-like 判定率 **7.2%**，基线 43.3–67.2%，去掉 expression planning 后升至 35.6%。

---

## 2. 全文结构

| 部分 | 内容 | 备注 |
|---|---|---|
| Abstract / §1 Introduction | gap 陈述（clinician 的 case formulation 会诱导「比来访者当前理解更清晰」的解释）、Figure 1 示例、3 条贡献 | Figure 1：治疗师漏掉 emotional reflection → 感知安全下降 → Pandora 输出简短试探性猜测 |
| §2 Related Work | 两段：Language Expression in User Simulation；LLM-based Psychotherapy Client Simulation | 可略读；点名 Patient-Ψ、MindVoyager、PatientAct、ConsistentMI、AnnaAgent、Eeyore、PsyCLIENT、Interactive Agents |
| §3 Method | 3.1 状态更新 / 3.2 主观归因 / 3.3 内容与策略 / 3.4 表达规划 / 3.5 生成执行 | 公式 (1)(2) |
| §4 Experiment | 4.1 数据、4.2 度量（RQ3 内容 + RQ1/2 风格）、4.3 结果（RQ1–RQ3 分段讨论） | Table 1/2/3、Figure 2；公式 (3)–(10) |
| §5 Conclusion / Limitations / Ethical Statement | 局限与伦理 | 局限写得相当实（见 §5 本笔记） |
| Appendix A | Implementation Details：A.1 预处理与角色蒸馏、A.2 状态表示（12 维）、A.3 状态更新数值、A.4 长度规划、A.5 内容定向与 claim verification、A.6 算力与预算 | **必读** |
| Appendix B | Running Example（Figure 1 的逐阶段拆解） | **必读** |
| Appendix C | 案例检索与语义评审（5,754→2,084→509→115 漏斗 + 评分门槛） | **必读** |
| Appendix D | 度量定义：D.1 Expression Fidelity（TF-IDF/masking/matched-block）、D.2 Memory Disclosure（LDDA/DS/BFE/BSE/GDS） | **必读**，公式 (3)–(9) |
| Appendix E | 人评协议：E.1 matched disclosure 呈现、E.2 评者资质与界面、E.3 聚合与不确定性（公式 10） | **必读** |
| Appendix F | 补充分层时序结果（一句话） | |
| Appendix G | Binary Annotation Guidelines（AI 味判定 rubric 全文） | **必读** |
| Appendix H | 人评案例研究（Table 4，case 45 的 8 候选 + 逐条投票） | **必读** |
| Appendix I | 数据 provenance/许可/发布策略（Table 5）、I.4 研究范围与伦理 | **必读**；代码声明在此 |

---

## 3. 方法精读

### 3.0 理论前提与输入输出

- **理论骨架**：Levelt et al. (1999) 的 speech production 两阶段区分 —— *conceptual preparation*（概念准备）vs *linguistic formulation*（语言形式化）。Pandora 把客户模拟也切成这两段，并显式计划 "what a client communicates" 与 "how that client says it"（§1、§3 开头）。
- **输入**（§3 第 2 段，符号为 alttext 原文）：turn $t$ 的 therapist utterance $x_{t}$、前文 $H_{t-1}$、上一心理状态 $s_{t-1}$、case material $M_{t}$、个体语言 profile $\rho$。$M_{t}$ 按事件发生时间**剔除未来事件**，但「所有心理深度」（L1–L3）对规划可用 —— 这句的关键在于：**案例信息的可得性与来访者当前的理解/表达方式被刻意分离**（"This separates the case information available to the simulator from the client's evolving understanding and manner of expressing it."）。

### 3.1 组件一：Therapist Intervention Evaluation and State Update（§3.1 + Appendix A.2/A.3）

- **干预评估**：*rule-based* evaluator 识别 therapist utterance 中 clarification / questioning / emotional reflection 三类机会及其**实际使用情况**，输出 intervention signal —— 关键设计是把「**该做而没做**」（opportunities left unaddressed，如 Figure 1 中漏掉的情感反映）也编码进信号。这直接对应"治疗师失误 → 来访者防御上升"的因果链。
- **状态表示**（Appendix A.2）：**12 个数值维度**：distress、emotional arousal、defensiveness、avoidance tendency、trust、perceived safety、disclosure readiness、self-awareness、insight、mentalization、core-belief activation、problem clarity；外加文本字段：dominant emotions and needs、presenting concerns、interpersonal patterns、beliefs、current self-explanation。
- **四组维度划分**（§3.1）：affective / relational / defensive / reflective，使 **willingness to disclose（披露意愿）与 capacity to articulate（表达能力）可以独立变化** —— 「来访者可以对问题更清楚，同时仍不愿谈」。
- **更新规则**（Appendix A.3）：纯规则、有界、**不需要单独的 LLM**（原文："This update is rule-based and does not require a separate LLM to infer a new psychological state."）。干预类型缩放系数：practical exchanges **0.10**、factual questions **0.40**、reflective statements **0.50**、challenges **0.80**、invalidation **1.00**；trust 与 perceived safety 额外用加权融合：旧值 **0.75** + 提议有界值 **0.25**，再做方向调整；另有 field-specific step limits 与 final value bounds。

### 3.2 组件二：Subjective Appraisal and Attribution（§3.2）

- **Appraisal $a_{t}$**：这次经验对来访者**意味着什么** —— emotional significance、perceived threat、implications for the self and relationships、对其解释的 confidence。
- **Attribution $b_{t}$**：来访者**当前如何解释**该经验。二者由 updated state + case material + intervention signal 构造。
- **核心设计**：state-conditioned attribution 是一条**光谱** —— immediate behavior and circumstances → tentative relational interpretations → connections with underlying beliefs；intervention signal 进一步调节「这份解释被表述得多清楚」。
- 效果：**同一份 case material 随交互进程支撑不同的主观叙述，即使来访者开始看清问题也保留不确定性**（"preserving uncertainty even when the client begins to understand a concern more clearly"）—— 这是反「过度阐释」的机制核心。

### 3.3 组件三：Content Target, Client Stance, Response Strategy（§3.3）

公式 (1)（alttext 原文）：

```latex
C_{t}=(a_{t},b_{t},d_{t},r_{t}).
```

- **Content plan $d_{t}$**：memory identifiers + stance。stance 枚举：hesitant / defensive / confused / emotionally overwhelmed / open；由 updated state 与 intervention signal 决定，把案例内容与当前互动参与度绑定。
- **Response-act plan $r_{t}$**：即时会话目的，按 therapist utterance 类型适配 —— factual questions → **answer first, limited introspection**；practical requests → brevity；reflections → confirmation or refinement；challenges or invalidation → resistance or clarification。
- **分离 stance 与 purpose 的意义**：允许「hesitant 的来访者直接回答问题、同时对心理含义表达不确定」。$C_t$ 即表达规划的语义输入。

### 3.4 组件四：Individual Expression Planning（§3.4）—— 表达规划器

公式 (2)（alttext 原文）：

```latex
L_{t}=F_{\mathrm{realize}}(s_{t},C_{t},H_{t-1},\rho).
```

**（a）Offline utterance style distillation（语言模式从来访者话语蒸馏）**

- $\rho$ 是**离线蒸馏、跨轮复用**的个体语言 profile，摘要五类习惯：habitual **discourse markers**、**hedging**、**emotional explicitness**、**utterance length distributions**（中心趋势 + 分布区间）、**sentence counts**。
- 蒸馏的工程来源（Appendix A.1）：对每条对话做 speaker role / message index 归一化 → *client-grounded memory extraction*（从来访者话语中抽取第一人称自传性事实，校验 source language、first-person form、有效 client-turn 索引、去重）→ 从对应 speaker turns 分别蒸馏 **client package** 与 **therapist package**。client package = case content + 观测到的句长分布 + 习惯性 discourse markers；therapist package = 源角色指令 + 提问方式/回复长度/代表话语/会话推进总结，配 keyword-based intent categories 做源例检索。
- 无个体统计时的回退：用心理与会话输入做长度规划（Appendix A.4）。

**（b）Online expression planning（规划输出的槽位）**

- 规划器逐轮指定：**speech style、pronoun focus、certainty、emotional explicitness、hedging、length 与 sentence-count targets**。
- 文献依据：Pennebaker et al. (2003)（心理学意义词使用）、Mairesse & Walker (2011)（可控风格生成）、Smith & Clark (1993)（不确定性下的 hedging）、Clark & Fox Tree (2002)（filled pauses 的交际功能）。
- **条件化规则**（谁决定什么）：client stance → base speech style；attribution → self / others / situation 的 focus；response strategy → 适配当前交换（answer-first → 直白措辞 + 低 emotional explicitness；resistance 或不确定解释 → tentative 表达）；**个体 profile 单独条件化 habitual hedging**，使一个直接回答仍保留熟悉的 "maybe"。
- 与 A.4 的衔接：长度目标按任务缩放 —— 来访者句长中位数（无中位数用均值）× **0.78（factual answers）/ 0.55（practical replies）**，句数统计作附加目标；近期话语约束「不重复解释、不用几乎相同的句子」，而熟悉的 filler/hedge **允许自然复现**。

**（c）规划与实现的分工**

- §3.5：client LLM 的 prompt 组合 = psychological context + appraisal/attribution + content & response strategy plans + linguistic realization plan + client profile + interaction instructions + 当前 therapist utterance + dialogue history；**明令 internal identifiers 与数值状态字段不得出现在话语中**。
- 分工原则一句话：**"Explicit rules organize the intermediate plans, and the LLM supplies their linguistic realization."** —— 规划由显式规则/槽位组织，LLM 只负责把结构化计划兑现成自然语言。
- 生成后闭环：content targeting + claim verification（Appendix A.5）跟踪案例材料的表达进度，accepted utterance 与 coverage record 提交后进入下一轮；case memory bank 固定，心理状态与已接受历史向前滚动。

### 3.5 「self-opaque / 自我阐释过度」的理论立场

- **术语本身只出现在标题**：grep 证实 "self-opaque/Self-Opaque" 在正文中零次定义 —— 全文没有给这个概念下操作性定义（诚实记录：「未在全文中找到」其定义）。
- 实质立场分散在三处：
  1. **摘要**：LLM 客户模拟器依赖 clinically organized case descriptions，会诱导 "responses explaining the client's difficulties **more clearly than their current understanding warrants**"。
  2. **§1**：case formulation 是「clinician 的解释性账户」，来访者可能 "only partially recognize or articulate" 它；用它驱动生成会产生 "greater explanatory coherence than the client's current understanding warrants"；配合 Gendlin (1961)：觉察**不必**立刻成为连贯解释，可能只以具体情境片段出现（"Whenever someone praises me, I feel uncomfortable…"）。
  3. **§2**："an explanatory pattern available to the simulator may remain only partly understood by the client."
- **「自我阐释过度」的操作化落点**在 Appendix G 的判定 rubric：AI 味线索 = *formulaic summaries、excessive completeness、generic psychological explanations、亲密披露与异常权威的解释声之间的不匹配*。也就是说，论文把「过度完整的自我解释」直接等同于需要被抑制的 AI-like 症状 —— 这是其反 over-interpretation 立场的可测量版本。
- 方法论回应链条：不把 formulation 交给生成器 → state（有界规则更新）→ appraisal/attribution（保留不确定的光谱）→ content/stance/strategy → realization（个体化但受 profile 与交互条件约束）。

---

## 4. 实验精读

### 4.1 研究问题与数据

- **RQ1**：Pandora 是否更好复现个体表达模式并降低感知 AI 味？**RQ2**：expression planner 在心理/内容规划之外贡献什么？**RQ3**：表达增益是否伴随忠实的披露时序与结构？（§4 开头）
- **115 案例构成**（§4.1 + Table 1）：97 中文 + 18 英文 = **97 PsyDial-D4 + 11 AnnoMI + 5 Eeyore Full + 2 ConsistentMI**；表名链到各官方 release。
- **构造漏斗**（Appendix C）：5,754 standardized records → 结构门槛（≥24 counselor–client exchanges、≥500 client characters、speaker-alternation ratio 0.70）→ 2,084 → 检索扩充（depth/causal/elicitation 线索 + 全部结构合格的 Eeyore Full/ConsistentMI/AnnoMI + top-100 其余候选）→ 509 进入语义评估 → **115**。
- **评审门槛**（Appendix C）：evidence-constrained rubric，深度判断必须有 client utterance 的 indexed evidence（治疗师解释与症状严重度词汇不算）；纳入需同时满足 minimum consensus scores：psychological-pattern depth **8/15**、deeper-driver depth **10/20**、causal links **7/15**、counselor elicitation **7/15**、progressive disclosure **7/15**，且无 disqualifying flags；**第二次模型辅助评估对第一次的分数/证据/排名/入选状态全盲**，base-score 分歧 >12 分触发第三次盲评；无 source/language/topic/diagnosis 配额，无人工 override。

### 4.2 五基线名单（§4.3）

共 **seven implementations** = 5 基线 + Pandora + 消融：

| 系统 | 适配方式（§4.3 原文） |
|---|---|
| **OpenClaw** | direct role-play（**全文无引用、无链接**，仅此一处描述） |
| **Interactive Agents** | role-card context（expert-written role cards；Qiu and Lan, 2026） |
| **Patient Phi**（正文其他处写作 **Patient-Ψ**，两处拼写不一致） | cognitive-model context with style guidance（CBT case formulation + conversational styles） |
| **MindVoyager** | cognitive and relational conditioning（metacognition/openness/rapport） |
| **Consistent MI** | state-conditioned action selection（readiness/action selection/generation 三段分离） |
| Pandora w/o EP | 保留上游各阶段，**移除整个 expression module**（消融把语言 profile 与其交互条件化作为整体评估） |
| Pandora | 完整系统 |

### 4.3 度量定义

**（A）Expression fidelity（§4.2.2 + Appendix D.1）**

- 表征：character **4-gram TF-IDF，最多 4,000 features** + cosine（沿用 Eval4Sim，Bao et al. 2026 的 stylometric 路线，但**不做**其训练式 authorship verifier 与 PAN aggregate score，改为成对比较）。
- 两个视图：**raw**（$S_{\mathrm{raw}}$）与 **content-reduced**（$S_{\mathrm{style}}$，topic masking，Stamatatos 2017 原则）：确定性变换保留**中文** function/stance/filler lexicon、标点、符号、轮边界，其余字符 run 替换为通用 mask。
- TF-IDF 权重（公式 3，alttext 原文）：

```latex
w_{dk}=c_{dk}\left[\log\frac{1+D}{1+\mathrm{df}_{k}}+1\right],\quad\mathbf{v}_{d}=\frac{\mathbf{w}_{d}}{\|\mathbf{w}_{d}\|_{2}},
```

- matched-block 采样：每条 transcript **B=200** 个有放回连续块，**seed 42**，块长上限 **500** 原始字符（按最短 transcript 收缩）；词表与 IDF 只在 reference blocks 上拟合、同案例内所有方法共享；零向量记 0 相似度。匹配分（公式 4）：

```latex
S_{i,v}^{(a)}=\frac{1}{B}\sum_{b=1}^{B}\cos\left(\mathbf{v}_{\phi_{v}(A_{i,b}^{(a)})},\mathbf{v}_{\phi_{v}(R_{i,b})}\right),
```

- **重要限定**（§4.2.2 原文）：masking lexicon **偏中文**，拉丁词被压缩为通用 mask，故 content-reduced 视图对英文案例证据更粗；两个视图**都不**建立 psychological authenticity 或 clinical equivalence，其绝对差也不被解释为测得的内容效应；本评测不做 cross-client retrieval 分数。

**（B）Disclosure alignment（§4.2.1 + Appendix D.2）**

- 共享 memory-card codebook，层级 **L1–L3**，比较每张卡的 **first disclosure turn**；披露判定来自 evidence-grounded **claim verification**（generator 自报深度**不作**观测）。
- 位置归一化（公式 5）：$p_{m}^{x}=\frac{\tau_{m}^{x}-1}{T_{x}-1},\qquad x\in\{r,g\}$（参考中出现而生成中缺失的卡记 $p_{m}^{g}=1$）。
- **LDDA**（披露时序，公式 6）：

```latex
\mathrm{LDDA}=\frac{1}{|\mathcal{L}|}\sum_{\ell\in\mathcal{L}}\left(1-\frac{\sum_{m\in\mathcal{M}_{\ell}^{r}}|p_{m}^{g}-p_{m}^{r}|}{|\mathcal{M}_{\ell}^{r}|}\right),
```

（按卡身份匹配防止不同时间的记忆互相补偿；按层平均防止大量表层记忆主导。）
- **DS**（前置满足，公式 7）：

```latex
\mathrm{DS}=\frac{1}{|\mathcal{H}|}\sum_{\ell\in\mathcal{H}}\frac{1}{|\mathcal{B}_{\ell}|}\sum_{b\in\mathcal{B}_{\ell}}\frac{\sum_{a\in\mathcal{P}(b)}I_{g}(a,b)}{|\mathcal{P}(b)|},
```

$I_{g}(a,b)=1$ 仅当两卡均已披露且 $\tau_{a}^{g}<\tau_{b}^{g}$（缺失卡与同轮披露不算满足前置）。前置关系来自显式 reference 约束，否则由参考中每张 L2/L3 卡回溯各浅层最近的更早卡。
- **Bundling**（公式 8/9）：$\mathrm{BF}_{x}$（一次披露 ≥2 张卡的轮次占比）、$\mathrm{BS}_{x}$（超额张数均值）→ $\mathrm{BFE}=|\mathrm{BF}_{g}-\mathrm{BF}_{r}|$、$\mathrm{BSE}=|\mathrm{BS}_{g}-\mathrm{BS}_{r}|$；$\mathrm{BFid}=\max\{0,1-(\mathrm{BFE}+\mathrm{BSE})/2\}$，总分 **$\mathrm{GDS}=(\mathrm{LDDA}+\mathrm{DS}+\mathrm{BFid})/3$**（三者缺一不算 GDS）。

**（C）Human AIR（Appendix E.3 公式 10）**

```latex
\mathrm{AIR}_{s}=\frac{1}{45R}\sum_{i=1}^{45}\sum_{r=1}^{R}y_{isr},
```

$R=4$ → 每个 source **180** 判定、8 个 source 共 **1,440** 判定；$y_{isr}=1$ 表示 AI-like。

### 4.4 主结果（Table 2，Core115 案例均值）

| Method | LDDA↑ | DS↑ | BFE↓ | BSE↓ | GDS↑ | $S_{\mathrm{style}}$↑ | $S_{\mathrm{raw}}$↑ |
|---|---|---|---|---|---|---|---|
| OpenClaw | 0.658 | 0.362 | 0.115 | 0.270 | 0.609 | 0.397 | 0.280 |
| Patient Phi | 0.667 | 0.373 | 0.108 | 0.200 | 0.629 | 0.438 | 0.310 |
| MindVoyager | 0.672 | 0.378 | 0.105 | 0.190 | 0.634 | 0.408 | 0.290 |
| Consistent MI | 0.690 | 0.417 | 0.100 | 0.128 | 0.664 | 0.512 | 0.352 |
| Interactive Agents | 0.681 | 0.364 | 0.099 | 0.133 | 0.643 | 0.501 | 0.342 |
| Pandora w/o EP | 0.811 | 0.795 | 0.107 | 0.140 | 0.828 | 0.458 | 0.311 |
| **Pandora** | **0.852** | **0.875** | 0.102 | **0.127** | **0.871** | **0.566** | **0.385** |

分段解读（§4.3 各 RQ 小节）：

- **RQ1**：最强基线 Consistent MI 为 0.512/0.352；Pandora **0.566/0.385**，超出 **0.053/0.032**。反例式发现：心理状态条件化并不自动带来表达保真 —— MindVoyager 的 content-reduced 0.408 **低于** role-card 的 Interactive Agents 0.501。**97 个中文子集**上排序保持：Pandora 0.489/0.422、Consistent MI 0.427/0.385、消融 0.362/0.337。
- **RQ2（消融）**：无表达规划时 GDS 仍 0.828（> 最强基线 0.664），但 content-reduced 掉到 **0.458 < Consistent MI 的 0.512**；恢复表达模块：$S_{\mathrm{style}}$ 0.458→0.566（**+0.108**）、$S_{\mathrm{raw}}$ 0.311→0.385（**+0.074**），同时 LDDA/DS/GDS 也 +0.042/+0.079/+0.043（表达规划反过来改善披露时序，因为长度/节奏目标影响披露分布）。
- **RQ3**：分层时序 L1/L2/L3 = **0.843 / 0.854 / 0.866**（Figure 2，各层 n = 113/115/88），每层都超过全部基线与消融（Appendix F 同结论）。bundling 是 tradeoff：Interactive Agents 的 BFE 最低（0.099 vs Pandora 0.102），Pandora 的 BSE 最低（0.127 vs Consistent MI 0.128）。

### 4.5 AI 味评者研究（完整数字）

**设计**（§4.2.2 Human evaluation + Appendix E/G/H）：

- **45 个 matched disclosure items**，来自不同案例，**每个 memory level（L1–L3）各 15 个**（15×3=45，§4.2.2 原文 "15 items at each memory level (L1–L3)"）。
- 每条目 **8 个候选**：reference、5 基线、Pandora、Pandora w/o EP；A–H **随机排序**，移动端界面逐案例呈现，逐候选独立二元判断；摘录为**完整单条来访者话语**（剔除咨询师文本以避免风格归因错误）。
- **四评者**：2 名 counselors（受过正式心理咨询训练、有专业资格、**各 >1,000 小时**咨询经验）+ 2 名 synthetic-data researchers（各至少一篇 synthetic data 的顶会一作）；**四人全部是本文共同作者**（Appendix E.2 明示）；均在评估前未看过原始数据集（Ethical Statement）。
- Rubric（Appendix G）：问题 = "Based only on the client's language and expression, does this utterance sound AI-generated?"；1 = AI-like（有充分风格证据），0 = 未识别为 AI-like（**不等于认证为人类所写**）；判定维度 = register、phrasing、organization、certainty、self-explanation、声音是否像描述自身经验的来访者；负面线索 = formulaic summaries、excessive completeness、generic psychological explanations、亲密披露与权威解释声不匹配；允许纵向比较但须独立赋标，禁止猜测系统身份或使用外部检测器。
- **信度**：全部 360 个候选上，mean pairwise agreement **62.9%**；AI-like 的 positive agreement **53.7%**（95% CI **49.4–58.1%**）。

**结果（Table 3，AI votes / 180）**：

| Source | AI votes | AIR↓ |
|---|---|---|
| OpenClaw | 111 | 61.7% |
| Patient Phi | 121 | 67.2% |
| MindVoyager | 98 | 54.4% |
| Consistent MI | 85 | 47.2% |
| Interactive Agents | 78 | 43.3% |
| Pandora w/o EP | 64 | 35.6% |
| **Pandora** | **13** | **7.2%** |
| Reference | 8 | 4.4% |

- Pandora 相对最低基线 Interactive Agents 降 **36.1 个百分点**；相对消融的配对差 **28.3 点**（exploratory case-bootstrap 95% CI **20.6–36.1 点**）。
- 分评者组（每组 2 人，描述性）：Pandora 在 counselors **10.0%**、researchers **4.4%**；消融为 **40.0% / 31.1%**。
- 分布稳健性：45 个 Pandora 候选中 **35 个得 0 票、7 个得 1 票、3 个得 2 票，无人达到 ≥3/4 的严格多数**。
- **Reference 的 4.4% 必须打折看**：论文自认 reference provenance 含 reconstructed/generated 材料，"is not a uniformly verified human control"，其与 Pandora 接近**不**等于与已验证人类语音等价。
- Appendix H 案例（case 45，L3 自责类披露，英文译文）：Pandora **0/4**、Reference **0/4**、OpenClaw 2/4、Interactive Agents 2/4、Pandora w/o EP **3/4**、MindVoyager 3/4、Consistent MI 3/4、Patient Phi 3/4 —— 短并不自动免疫（Consistent MI 较短仍 3 票），「响应直接性 + 心理解释展开程度 + 句长」共同决定声音。
- 人评动机（Appendix E）：引用 X-TURING（人/LLM 判定分歧）与 MirrorBench（中等相关、对 judge 选择敏感）说明 stylometric 相似度需与直接人评互补。

---

## 5. 附录/细节挖掘

1. **Prompt 情况**：全文**只有 prompt 构成的描述、没有 verbatim prompt 文本** —— 「未在全文中找到」任何完整提示词。可见的描述：§3.5（prompt 组合与"数值字段不得入话"规则）、A.1（保留源 counselor system prompt、client/therapist package 内容）、A.4（长度目标为 "prompt-level guidance"）、Appendix C（评审 rubric 与 prompts「随采集保留、可审查」但未附出）。
2. **A.5 公平性设置**：Pandora 私有 context 最多给 **3 条 memory cues、鼓励 1 条主 cue**；**所有被评方法都拿到同一份 full neutral fact catalog（打乱 id、无 L1–L3 元数据）**；共享 coverage controller 提供 deadline 目标、优先保证核心事实；生成后 claim-verification 不足可触发 **bounded regeneration**。
3. **A.6 算力**：全部 LLM 客户模拟、案例筛选、模型辅助评审用 **GPT-5.4 mini（API id `gpt-5.4-mini`）**，全 API 推理，估算总 API 成本 **$600**。
4. **B Running example**：Figure 1 中离线 profile 提供高频 hedging 与习惯标记 "maybe"；answer-first 策略 → 直白 + 低情感显式度；最终话语 "Maybe he's upset with me, or I said something wrong." —— 回答了问题、传达了即时猜测、保留不确定与习惯措辞；planning 结构只是 planning context，不是必须复述的句子。
5. **D.1 细节**：masking 与 reference-block 采样是本文的 adaptation；TF-IDF 平滑与 L2 归一化在参考块上拟合。
6. **E.1**：候选按 required claim evidence 对齐到同一 case/同一 disclosure item；长度可比性与完整话语保留之间做平衡；每条 reference 每案例匿名出现一次。
7. **I 数据与许可（Table 5）**：PsyDial-D4（ACL 2025 main，**Apache-2.0**，可修改再分发）；AnnoMI（ICASSP 2022 + Future Internet 2023 扩展，**Public Domain**）；ConsistentMI（ACL 2025 main，官方公开仓库、无单独数据许可声明）；Eeyore Full（Findings of ACL 2025，**需审批访问**）。PsyDial 用 RMRR（Retrieve-Mask-Reconstruct-Refine，LLM 重建 client utterance）——即 97/115 的案例本身是**重建语料**。
8. **I.3/I.4 与伦理**：「We **will** maintain a GitHub repository… The repository listed on the first page **currently** provides the preprocessing and modeling code」（两处时态并存）；研究仅用既有数据 + 四位共同作者二元标注，**未获机构伦理审批或豁免认定**（no institutional ethics approval or formal exemption determination）。
9. **局限（Limitations）自陈**：案例偏中文且含重建/生成材料；状态数值是 design choices；消融把表达模块当整体评估，细粒度信号贡献未分离；人评 panel 小、用孤立话语、rater 有差异；**source-reveal 使用未被记录，无法确认完全盲**；是否改善 therapist training/eval 未验证。
10. **Ethical Statement**：内部心理状态是 modeling constructs 而非对真实个体的临床发现；测量不建立 clinical safety 或 treatment efficacy；参考集并非统一验证的真实来访者语音。

---

## 6. 代码情况核查

**结论：全文未见官方代码发布。**

核查过程与证据：

1. 对 HTML 全文 grep `github|code is|code at|anonymous|project page|available at`：命中仅两类 —— arXiv/LaTeXML 页面自带的 footer 链接（`github.com/arXiv/html_feedback/issues`、`brucemiller/LaTeXML/...`，与论文无关），以及 Table 1 中链向**基线/数据集**的三个仓库（见下表）。
2. 提取全文**全部** `href` 外链（100+ 条，去重后逐条核对）：除上述三个 GitHub 仓库外，无任何指向 Pandora 自身代码/项目页/demo 的链接。
3. 复核 arXiv abs 页（`arxiv.org/abs/2611.00617`）：无 Comments 字段、无 Code/Data 链接、无 journal-ref。
4. **悬空声明**（重要）：论文三处声称首页有仓库 —— Appendix A.1 "available in the code repository listed on the first page"、I.3 "The repository listed on the first page currently provides the preprocessing and modeling code"、I.4 "code listed on the first page"；但首页（HTML 标题/摘要区）与 abs 页均**无该链接**。同类悬空：Appendix G 提到 "An online demonstration lets readers inspect all 45 cases"，全文亦无该 demo 链接。可能是 PDF 首页有链接而 HTML 转换丢失，也可能是链接本身缺失 —— **无法从现有材料确认，按「未在全文中找到官方代码」记录**。

**文中引用的基线/数据集仓库清单**：

| 链接 | 用途 |
|---|---|
| `https://github.com/IzzetYoung/ConsistentMIClientSimulator` | 基线 **Consistent MI**（state-conditioned action selection；ACL 2025）的官方仓库；其 profiles/dialogues 派生自 AnnoMI，附加标注与生成输出单列 |
| `https://github.com/MichiganNLP/Eeyore` | 基线/数据集 **Eeyore Full**（Findings of ACL 2025，抑郁相关表达学习）官方仓库；指向 gated 释放 |
| `https://github.com/uccollab/AnnoMI` | 数据集 **AnnoMI**（ICASSP 2022）官方仓库；公开域声明的转录集 |
| `https://huggingface.co/datasets/qiuhuachuan/PsyDial-D4` | 数据集 **PsyDial-D4**（ACL 2025，Apache-2.0）官方 dataset card —— 97/115 案例来源 |
| `https://huggingface.co/datasets/liusiyang/eeyore_profile` | Eeyore 的 gated profile 释放 |
| `https://doi.org/10.3390/fi15030110` | AnnoMI 扩展论文（数据许可依据） |

另：基线 **OpenClaw**（direct role-play）与 **Interactive Agents**、**Patient Phi/Ψ**、**MindVoyager** 在 Table 2/3 中被评，但论文未给它们的代码链接（OpenClaw 连引用都没有）。

---

## 7. 摘要级理解的修正

1. **「expression planning 是核心贡献」需要精确到侧**：消融显示上游（状态 + 归因 + 内容策略）已经拿到 GDS 0.828 vs 最强基线 0.664 —— **披露结构的大幅领先主要来自上游规划**；表达规划的净贡献集中在表达侧（$S_{\mathrm{style}}$ +0.108）与 AIR（−28.3 点），对披露指标是小幅正溢出（+0.042/+0.079/+0.043）。
2. **「最高 expression similarity」的基线强度**：最强表达基线是 Consistent MI（0.512/0.352），而**感知 AI 味的最强基线是 Interactive Agents（43.3%）** —— 文本相似度与人眼 AI 味**排序不一致**（论文自认二者互补），摘要的双冠军叙事掩盖了这一错位。
3. **Reference 并非干净的人类对照**：reference AIR 4.4%（8/180），且含重建/生成材料 —— Pandora 的 7.2% 是「接近一个不完全真实的参照」，不是「接近已验证人类语音」。
4. **评者全部是共同作者**（2 counselor + 2 researcher），且 source-reveal 未记录、盲法无法完全确认、无伦理审批 —— 摘要的 "four-evaluator study" 不含这些限定。
5. **数据极度偏科**：97/115 来自 PsyDial-D4（且为 RMRR 重建语料），115 中仅 18 条英文；content-reduced 度量的 masking 词表偏中文，英文子集证据更粗 —— 摘要的 "four datasets" 实际是 97/11/5/2 的悬殊分布。
6. **基线是「适配版」**：论文原话是 "evaluated baseline adaptations"，各家实现保真度未知；OpenClaw 无引用无链接。
7. **方法细节摘要未提**：状态更新是**纯规则、无独立 LLM**（12 维数值 + 干预缩放系数 0.10–1.00）；expression planner 的输入是 $(s_t, C_t, H_{t-1}, \rho)$，输出 6 类槽位；生成层被明令禁止把数值状态/内部 id 写进话语。
8. **代码**：摘要与全文都让人以为「首页有仓库」，实际**无任何可访问链接**（见 §6）。
9. 摘要与 `papers-abstracts.md` §2 存档**逐字一致**（含 43.3–67.2%、7.2%、35.6% 等），无需修正存档本身；修正的是「摘要隐含的对照与外推边界」。

---

## 8. 对 OIA 的借鉴更新

> 三条既有线索（表达规划分层、StyleLexicon 蒸馏、克制检查）在全文细节下的具体落点。以下为**建议**，需与 OIA 现有实现对表后落地。

### 8.1 表达规划分层（把单跳生成拆成「语义计划 → realization 计划 → 执行」）

- **落点**：OIA 的 response 生成前增加一个**显式 realization-plan 中间产物**（结构化 JSON/槽位），字段直接对齐 §3.4 的六项：`speech_style / pronoun_focus / certainty / emotional_explicitness / hedging / length_and_sentence_targets`；上游先产出 `content_plan(memory_ids + stance) + response_act(purpose)`（公式 1 的 $C_t$），再由 realization 函数 $F_{\mathrm{realize}}(s_t, C_t, H_{t-1}, \rho)$ 生成 $L_t$（公式 2），最后执行层只做兑现。
- **分工纪律可直接抄**（§3.5 原话的工程版）：**规则组织中间计划，LLM 只负责 linguistic realization**；执行 prompt 中显式禁止把内部 id、数值状态字段写进输出文本。
- **stance 与 purpose 必须分离**（§3.3）：允许「犹豫但直接作答」这类组合；stance 枚举（hesitant/defensive/confused/emotionally overwhelmed/open）与 purpose 枚举（answer-first / brevity / confirm-or-refine / resist-or-clarify）各自条件化。

### 8.2 StyleLexicon 蒸馏（$\rho$ 的落地）

- **蒸馏内容五件套**（§3.4 offline）：habitual discourse markers、hedging 词表、emotional explicitness 基线、**句长分布（中位数 + 分布区间，缺中位数退化为均值）**、句数统计 —— 对应 OIA 每个用户/角色离线构建一次 `StyleLexicon`，跨轮复用。
- **注入方式**：作为 profile 进 prompt，同时给**目标值**而非形容词 —— 长度目标 = 中位句长 × 任务缩放系数（论文给 factual **0.78** / practical **0.55**，可配置）；近期输出加「禁止重复解释/近似复句」约束，但**允许熟悉 filler/hedge 自然复现**（Appendix A.4）。
- **蒸馏管线可参考 A.1**：先做角色与索引归一化 → 抽取有第一人称、有源话语索引支撑的 facts（校验 + 去重）→ 再按说话人分蒸 client/therapist 包；检索侧用 keyword-based intent categories 找源例。
- **回退策略**（A.4）：个体统计缺失时用心理/会话输入兜底 —— OIA 冷启动用户同样需要这条路径。

### 8.3 克制检查（反「自我阐释过度」的两道闸）

- **生成前/生成中的内容克制（A.5）**：私有上下文最多 **3 条 memory cue、默认 1 条主 cue**；共享 coverage controller 提供 deadline、**优先核心事实而非铺陈**；生成后 claim-verification 要求「每条主张有证据支撑」，缺主张只做**有界**重生成 —— 落到 OIA 即：披露/引用受 cue 预算约束 + 强制 grounding 校验 + 有界 retry，防止 agent 把知道的全倒出来。
- **生成后的 AI 味 lint（Appendix G 的 rubric 可直接做成检查项）**：把四条负面线索转成可判定规则 —— ① formulaic summaries（模板化总结）、② **excessive completeness**（过度完整、把话说满）、③ generic psychological explanations（泛泛的心理解释）、④ 亲密内容配权威解释声（身份/语气错配）。命中任一项即降权或重写 —— 这正是论文把「自我阐释过度」操作化为 AI-like 判定的方式。
- **状态层的克制**：所有数值更新**有界 + 步长限制 + 方向调整**（A.3），且 trust/safety 用 0.75 旧值的惯性加权 —— OIA 的用户状态/情绪状态更新可采用同样的 bounded + inertia 设计，避免状态跳变驱动的过激输出。

### 8.4 评测侧借鉴（附带）

- 低成本回归评测组合：**char 4-gram TF-IDF + 掩码（去主题）cosine**（B=200 块、seed 42、块长 ≤500）测表达保真；**memory card first-disclosure + claim verification** 测内容忠实（LDDA/DS/GDS 思路）；**二元 AI-like 盲评**（8 候选随机序、逐条独立、报告 pairwise/positive agreement 与 bootstrap CI）测感知质量。注意论文的三重警告：相似度 ≠ 心理真实性；参考集可能含生成材料；评者是作者时需声明。

---

## 9. 诚实边界

- 本笔记全部数字与引文来自 arXiv HTML 全文（2611.00617v1），**未复现任何实验**；数字均标注章节出处（§/Table/Figure/Appendix）。
- **代码核查限于本地 HTML 与 arXiv abs 页**：结论是「全文未见官方代码发布」+「三处 first-page 仓库声明悬空」；若 PDF 首页带链接（HTML 转换可能丢失），或 abs 页 Comments 后续补充，结论需更新 —— 未在现有材料中找到，故不肯定存在也不肯定不存在，仅记录事实。
- **未在全文中找到**的内容：Pandora 官方仓库 URL、Appendix G 所述在线 demo URL、verbatim prompt 文本、「self-opaque」术语的正文定义、作者机构、venue/录用信息、OpenClaw 的引用与来源。
- 论文自身的效度边界（Limitations/Ethical/I.4 已明示）：案例偏中文且 97/115 来自重建语料 PsyDial-D4；英文子集 18 例、masking 词表偏中文；四评者全为共同作者、panel 小、孤立话语、盲法不完全确认；无机构伦理审批或豁免；状态数值为 design choice；消融只到模块级；测量不涉及临床安全与疗效。
- 对 OIA 的建议（§8）是**设计推论**，不是论文的原话主张，也未经本笔记验证其在 OIA 代码中的可实施性。
