# 04 · GraphMemix 精读笔记

> GraphMemix: Query-Aware Evidence Forests for Long-Term Multimodal Agent Memory
> arXiv:2608.26983v1 [cs.AI]，2026-08-27；本地全文：`docs/sources/papers/2608.26983-graphmemix.html`（318KB HTML）；配套代码：`docs/sources/repos/graphmemix/`（github.com/ligeng0197/graphmemix）；摘要对照：`docs/sources/papers-abstracts.md` §4。

---

## 1. 论文速览

| 项 | 内容 | 出处 |
|---|---|---|
| 标题 / 作者 | GraphMemix；Geng Li, Yuhao Wang, Dong Li, Jianye Hao, Yuxin Peng（北大王选计算机研究所 × MemoraX AI，通讯 Yuxin Peng） | 首页作者块 |
| venue | **仅 arXiv 预印本**（arXiv:2608.26983v1 [cs.AI]，27 Aug 2026）；全文与 CITATION.cff 均**未标注任何会议/期刊录用信息**（CITATION 的 `journal` 字段就是 "arXiv"） | HTML 水印行；`CITATION.cff` |
| 关键词 | multimodal agent memory, evidence retrieval, graph optimization, long-term memory | 首页 Keywords |
| 代码 | github.com/ligeng0197/graphmemix（pyproject 另给 Gitee 镜像 gitee.com/oking9/graph-memix）；数据包 HuggingFace `oking0197/graphmemix-benchmarks` | `pyproject.toml`、README |
| 许可 | **MIT**（`pyproject.toml: license = {text = "MIT"}`；仓库根目录**没有独立 LICENSE 文件**，以 pyproject 声明为准）；原始 benchmark 资产沿用各自上游许可，不随仓库再分发 | `pyproject.toml`、README「Data and Artifacts」 |
| 版本 / 日期 | 软件 CITATION `date-released: 2026-08-27`，version 0.1.0，Python ≥3.10 | `CITATION.cff`、`pyproject.toml` |
| 一句话定位 | 把「长期多模态记忆的组织」在**问题到达之后**重新表述为一个**带预算约束的组合优化（证据森林）问题**：三个组件 = 候选图构造 / 证据效用与激活成本 / 森林优化 | 摘要、§1 |

---

## 2. 全文结构

| 章节 | 内容 | 关键落点 |
|---|---|---|
| Abstract | 两条既有路线的问题（question-agnostic 离线摘要贵；naive embedding 相似度不完整且冗余）+ 三组件 + Pareto 主张 | 摘要原文 |
| §1 Introduction | 两条路线综述（MIRIX/SGM/A-MEM/AUGUSTUS/M²A vs MuRAG/Pensieve）；GraphMemix 把节点效用与「以锚点为条件的边增量价值」分开建模；固定节点集下最优最大权森林由 **Kruskal 精确求得**；Qwen3-VL 上四数据集宏平均 49.80%→61.55%（+11.75pp） | §1 末段 |
| §2 Related Work | 2.1 长期多模态 agent 记忆；2.2 查询条件证据检索（MemGuide、MemReranker、RankGPT、RagVL、MEG-RAG——「候选-候选关系不是显式决策变量」） | §2 |
| §3 Method | 3.1 问题形式化 → 3.2 证据森林目标 → 3.3 候选图构造 → 3.4 效用与成本 → 3.5 森林优化（两阶段求解） | 公式 (1)–(13) |
| §4 Experiments | 4.1 设置（表 3–5 放在章首）；4.2 主结果（表 1、2）；4.3 效率/生命周期（图 3）；4.4 消融（表 6、7 + 4 个子节）；4.5 定性（图 4） | 表 1–7 |
| §5 Conclusion | 重述：有界候选图 + 双验证器 + 自适应大小森林；「每个被考虑的节点集都取到精确最大权森林，冻结 proposal 内取到精确最优子集」 | §5 |
| §6 References | — | — |
| Appendix A | A.1 数据与预处理（表 8）；A.2 模型与固定参数（式 14、15）；A.3 全生命周期 wall-clock（图 5）；A.4 提示与结构化输出；A.5 证据序列化；A.6 受控证据集选择（表 9、10，式 16）；A.7 评测校验 | 表 8–10 |

> 排版注意：HTML 中表 1、表 2（主结果）物理上出现在 §3 的 DOM 里，但按 4.2 的引用语义它们是主结果表；本笔记按语义归入 §4。

---

## 3. 方法精读

### 3.1 问题形式化（§3.1）

多模态记忆档案 `Ṽ = {e_i}_{i=1}^N`，查询 `q = (q^text, q^vis)`，选择器 `Π` 在**证据预算 K** 下选出有序证据序列：

```latex
(S_{q},O_{q}) = \Pi(q,\mathcal{V};K),\qquad S_{q}\subseteq\mathcal{V},\quad|S_{q}|\leq K,
\qquad \hat{y} = \mathcal{M}(q,O_{q}).            % 式(1)
```

`ℳ` 是冻结的多模态生成器（reader）。**预算约束的形态就是基数约束 `|S_q| ≤ K`**（实验中 K=10，见 §4.1），不是 token 预算；token 级预算只体现在 reader 上下文参数（`max_model_len=32768`、`reader_max_output_tokens=1000`，见代码配置）。

### 3.2 三组件的形式化定义

#### 组件一：候选图构造（§3.3）

**(a) 多视图种子打分**——同一记忆可经原图/字幕/OCR/视频帧等多视图 `U_i` 被检回，取最大相似度（只提供「多条进入候选集的通路」，不做决策）：

```latex
s_{i}(q)=\max_{u\in\mathcal{U}_{i}}\operatorname{sim}\!\left(h(q),h(u)\right)   % 式(3)
```

**(b) 两类关系 + 发现图**（schema 关系来自可观察交互结构；语义关系为互相 kNN）：

```latex
E_{\mathrm{schema}}=\{(i,j):\exists r\in\mathcal{T}_{\mathrm{schema}},\ \phi_{r}(e_{i},e_{j})=1\},
\qquad
E_{\mathrm{sem}}=\{(i,j):i\in\mathcal{N}_{k}(j),\ j\in\mathcal{N}_{k}(i)\}     % 式(4)
G^{\mathrm{disc}}=(\mathcal{V},E_{\mathrm{schema}}\cup E_{\mathrm{sem}})
```

**(c) 有界候选集**——top-L 种子 + ≤H 跳扩展，再按「检回顺序 + 关系扩展顺序」截到 M：

```latex
R_{q}^{(L)}=\operatorname{Top}_{L}(\mathcal{V};s_{i}(q)),\qquad
\widetilde{C}_{q}=R_{q}^{(L)}\cup N_{\leq H}^{G^{\mathrm{disc}}}(R_{q}^{(L)}),\qquad
C_{q}=\operatorname{Top}_{M}(\widetilde{C}_{q})                               % 式(5)
```

实验取 **L=24, H=1, k=8, M=48**（§4.1）。这一步「保留局部上下文，同时**为后续所有语义推理封顶**」——M=48 就是下游验证器与优化器的计算上界。

#### 组件二：证据效用与激活成本（§3.4）

**(a) 直接节点效用**——listwise 节点验证器一次调用读入 `q` 与 `C_q`，输出 `v_i(q)`（该记忆**独立**支持问题的程度），与检回先验融合（式 6 左）再求和（式 6 右）：

```latex
p_{i}(q)=\sigma\!\left(\tau[\alpha s_{i}(q)+(1-\alpha)\widetilde{v}_{i}(q)-\delta]\right),
\qquad \mathcal{U}_{\mathrm{node}}(S\mid q)=\sum_{i\in S}p_{i}(q)              % 式(6)
```

固定参数（全数据集、全 backbone 一套，不逐基准重调，A.2）：`(α,τ,δ)=(0.8, 5.2, 0.7)`，`v_i∈[0,5]`、`ṽ_i=v_i/5`，等价实现式 `p_i(q)=σ(4.2 s_i(q)+0.2 v_i(q)-3.6)`（式 14、15）。

**(b) 锚点条件的关系验证**——从**冻结的**高置信锚集 `A_q`（由检回结果确定，先于并独立于验证器与 gold，A.2「Anchor-set construction」）只检查锚↔候选的 schema 边：

```latex
E_{q}^{\mathrm{elig}}=\{(a,i)\in E_{\mathrm{schema}}:a\in A_{q},\ i\in C_{q}\}  % 式(7)
G_{q}^{\mathrm{trust}}=(C_{q},E_{q}^{\mathrm{trust}}),\qquad E_{q}^{\mathrm{trust}}\subseteq E_{q}^{\mathrm{elig}}  % 式(8)
```

Evidence-Chain Verifier（ECV）一次 listwise 调用：为每个候选挑「最能解释其增量角色」的锚，打增量分 `s_ai^inc(q)`，并给出六种角色之一：`new_fact / clarification / corroboration / redundant / conflict / irrelevant`；**只有正分且属于前三种角色的最佳锚边被保留**。边可靠性与不确定成本：

```latex
w_{ai}(q)=w_{ai}^{\mathrm{schema}}\frac{s_{ai}^{\mathrm{inc}}(q)}{s_{\max}},\qquad
c_{ai}(q)=-\log w_{ai}(q),\qquad
\mathcal{C}_{\mathrm{edge}}(F\mid q)=\lambda\sum_{e\in F}c_{e}(q)               % 式(9)
```

`w^schema` 是 schema 类型的可靠性上限（实验取最大 0.99，§4.1）。**角色门控决定边是否携带正增量语义，连续分数决定其额度**——相关但冗余/冲突的记忆拿不到结构奖励。

**(c) 独立链（开链）成本**——`m(S,F)` 为森林连通分量数，每个分量是一条需独立解读的证据链：

```latex
\mathcal{C}_{\mathrm{open}}(S,F)=\kappa\,m(S,F)                               % 式(10), \kappa\ge 0
```

**(d) 总目标**（§3.2）：

```latex
\max_{S,F}\;\mathcal{U}_{\mathrm{node}}(S\mid q)-\mathcal{C}_{\mathrm{edge}}(F\mid q)-\mathcal{C}_{\mathrm{open}}(S,F)  % 式(2)
```

#### 组件三：森林优化（§3.5）

把式(6)–(10) 代入式(2) 得到完整带约束问题：

```latex
(S_{q}^{\star},F_{q}^{\star})=\arg\max_{S,F}\ \sum_{i\in S}p_{i}(q)-\lambda\sum_{e\in F}c_{e}(q)-\kappa m(S,F)
\quad \text{s.t.}\quad S\subseteq C_{q},\ |S|\leq K,\quad F\subseteq E_{q}^{\mathrm{trust}}[S]   % 式(11)
```

**关键化简**：边成本非负、删环不降目标 ⇒ 最优 `F` 可取为森林 ⇒ `m(S,F)=|S|-|F|`；定义结构增益 `B_e(q)=\kappa-\lambda c_e(q)`，目标等价于

```latex
\sum_{i\in S}\bigl(p_{i}(q)-\kappa\bigr)+\sum_{e\in F}B_{e}(q)                  % 式(12)
```

这一步的含义（论文原话级）：**不需要单独的逐节点罚项就得到自适应基数**——孤立记忆贡献 `p_i−κ`，必须能「自证开一条独立链」才保留；若经边 `e` 并入已有分量，则找回一份开链成本，边际贡献变成 `p_i−λ c_e`。**受保护的是「可信的互补证据」，被删的是「低效用的孤立证据」。**

### 3.3 求解算法与复杂度直觉（§3.5「Bounded exact cardinality refinement」）

两阶段、确定性：

1. **阶段一（固定基数 proposal）**：在 `C_q` 上产出 K 节点提案 `S̄_q ⊆ C_q`——以 top-K 节点效用初始化，每次评估一个「已选↔未选」的 1-swap，**对每个提案节点集用 Kruskal 精确重算最大权森林**，终止于确定性的 1-swap 局部最优。
2. **阶段二（冻结 proposal 内的变基数精确解）**：

```latex
F_{S}^{\star}=\arg\max_{F\subseteq E_{q}^{\mathrm{trust}}[S]}\sum_{e\in F}B_{e}(q),
\qquad
(S_{q}^{\star},F_{q}^{\star})=\arg\max_{\varnothing\neq S\subseteq\bar{S}_{q}}
\Bigl[\sum_{i\in S}\bigl(p_{i}(q)-\kappa\bigr)+\sum_{e\in F_{S}^{\star}}B_{e}(q)\Bigr]   % 式(13)
```

对正增益边跑 Kruskal 即得 `F_S*`（精确）；平局先偏好更少记忆、再偏好更早的 proposal 排名。**返回解在 `S̄_q` 的所有非空子集上全局最优。**

**复杂度直觉**（论文未给显式大 O，以下为按算法描述的推算，标注为推算）：
- 搜索空间被两次封顶：候选 `M=48` → 提案 `K=10` → 阶段二枚举 `2^K−1 = 1023` 个非空子集；
- 每个子集/每次 swap 的森林子问题 = 对 `E_q^trust[S]`（至多 `|A_q|·K` 量级的锚-候选边）排序 + Kruskal，`O(E log E)`；
- 阶段一最坏 `O(K·(M−K))` 次交换评估；实证（图 2，6,592 个评测问题）**前几次被接受的 swap 就贡献了绝大部分均值目标改进，6 次之后变化可忽略**——「有界 proposal 只需少量交换即可被修正，而每次交换都用精确 Kruskal 评估其最优诱导森林」。
- 论文没有给出运行时间上界、近似比（approximation ratio）或收敛性定理——**只声称「冻结 proposal 内精确」「每个被考虑节点集的森林精确」**（§5）。

**序列化**（A.5）：每个连通分量取效用最高节点为根，按根效用排序分量，分量内 BFS；平局按节点效用、再按原始检回顺序——保证「直接锚点先于其可信关系恢复出的上下文」，reader 至多收到 K 条规范多模态记忆。

### 3.4 「查询感知」如何避免问题无关摘要

- **时机**：一切语义推理都在 `q` 到达之后发生；写入侧不做 question-agnostic 压缩（§1 明确批评 MIRIX/SGM/A-MEM/AUGUSTUS/M²A 的冷启动与持续更新成本，以及「压缩掉后来变得决定性的视觉属性/状态迁移/局部上下文」）。
- **三层查询条件化**：① 种子 `s_i(q)` 与候选集 `C_q` 由 `q` 决定（式 3、5）；② 节点效用 `p_i(q)` 由 `q` 条件的验证器分数校准（式 6）；③ **边只在锚 `a∈A_q` 与候选之间检查，且由 ECV 按 `q` 打增量分**（式 7–9）。关系本身（schema 图）是 query-independent 的，但**哪些关系被信任、值多少成本是 query-dependent**——这正是「关系是显式决策变量」与 RankGPT/MemGuide 类「只输出 query–记忆相关性分数」的分野（§2.2）。
- **冗余/冲突抑制**：角色门控把 `redundant/conflict/irrelevant` 直接排除出 `E_q^trust`，因此「主题相近但暗示不同答案」的记忆拿不到结构奖励（§3.4）。
- **无生成式历史处理**：摘要类方法的 lifecycle 成本来自「对全历史做生成式预处理」；GraphMemix 的语义推理被限制在 ≤M=48 的查询相关子图上（§4.3）。

---

## 4. 实验精读

### 4.1 设置（§4.1、A.1、A.2）

**四个基准**（表 8，实验样本数）：

| 基准 | contexts | memories | media | 实验题数 | caption 来源 | schema 关系 |
|---|---:|---:|---|---:|---|---|
| ATM-Bench | 1 | 11,034（6,742 邮件 + 3,759 图 + 533 视频） | 3,759 I + 533 V | 1,044（1,013 default + 31 hard） | 发布的派生字段（Qwen3-VL-2B-Instruct） | `event_bridge` / `same_record` / `record_grounding` |
| Mem-Gallery | 20（240 sessions） | 7,944 | 1,490 I | 1,711（其中 184 题无 gold 标注，Recall/Hit 只在 1,527 题上算） | 原生字幕 | `same_round` / `consecutive_turn` |
| MemEye | 16 | 3,392 | 442 I（438 历史 + 4 查询） | 1,855 | 原生记忆字幕 | `same_round` / `consecutive_turn` |
| H2HMem | 25 | 7,078 turns | 1,300 I | 1,982（固定 track；发布集 2,236 题） | **Qwen3-VL-8B 生成的冻结 sidecar**（对所有方法共享） | 仅 `consecutive_turn` |

四基准分别强调：个人多媒体档案 / 跨会话交互 / 细粒度视觉记忆 / 多方交互历史（§4.1 Datasets）。

**指标**：主指标 LLM-as-a-Judge 的 **Judge Accuracy（GPT-5-mini，快照 gpt-5-mini-2025-08-07，temp 0，协议 mmmb-llm-judge-1.2，**不计入方法成本**）**；各基准另报一个原生指标——ATM/Mem-Gallery 用归一化 EM，MemEye 用四个选项位置轮换平均的 MCQ EM，H2HMem 用去停用词的词面 Recall；证据诊断用 Recall@K、Hit@K。人工审计 200 条分层样本，与 GPT-5-mini 判断一致 194/200 = **97.0%**（A.7）。

**基线**：文本/结构化记忆 A-MEM、VimRAG、LightMem；多模态检索 UniversalRAG、MemGuide——所有方法用同一 reader。

**实现参数**（§4.1）：冻结编码器 `gme-Qwen2-VL-2B-Instruct`（1,536 维）；L=24、H=1、k=8、M=48、K=10；schema 可靠性上限 0.99；ECV 增量分归一到 [0,1]；**λ=0.1**；proposal 阶段 **κ_prop=0.2**，精确基数细化阶段 **κ=0.12**；节点验证器与 ECV 各一次 listwise 调用、并行执行。三阶段（节点验证/ECV/reader）同 backbone：Qwen3-VL 设定全用 `Qwen/Qwen3-VL-8B-Instruct`，Gemma 设定全用 `google/gemma-4-12B-it`（A.2 明确「不偷偷给 GraphMemix 更强的验证器」）。

### 4.2 主结果（表 1、表 2；§4.2）

**表 1 · Qwen3-VL-8B-Instruct，GPT-5-mini 判分（J.Acc./原生指标，%）**

| Method | Venue | ATM J.Acc | ATM EM | Gallery J.Acc | Gallery EM | MemEye J.Acc | MemEye M.EM | H2H J.Acc | H2H Recall | Avg |
|---|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| A-MEM | NeurIPS'25 | 41.86 | 40.80 | 52.89 | 32.61 | 37.09 | 40.57 | 39.94 | 40.55 | 42.94 |
| UniversalRAG | ACL'26 | 43.10 | 41.19 | 63.76 | 32.50 | 47.98 | 49.33 | 44.36 | 45.04 | 49.80 |
| MemGuide | AAAI'26 | 48.47 | 47.80 | 48.10 | 32.73 | 41.40 | 44.61 | 39.94 | 35.81 | 44.48 |
| LightMem | ICLR'26 | 22.70 | 24.52 | 38.28 | 23.20 | 36.23 | 39.76 | 31.04 | 27.82 | 32.06 |
| VimRAG | ICML'26 | 33.81 | 32.76 | 48.33 | 16.13 | 28.25 | 28.23 | 17.89 | 17.01 | 32.07 |
| **GraphMemix** | Ours | **55.27** | **53.83** | **76.33** | **36.76** | **53.64** | **54.25** | **60.96** | **54.70** | **61.55** |
| Gain over 2nd | – | +6.80 | +6.03 | +12.57 | +4.03 | +5.66 | +4.92 | +16.60 | +9.66 | **+11.75** |

**表 2 · Gemma 4 12B Unified（google/gemma-4-12B-it）**

| Method | ATM J.Acc | ATM EM | Gallery J.Acc | Gallery EM | MemEye J.Acc | MemEye M.EM | H2H J.Acc | H2H Recall | Avg |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| A-MEM | 42.43 | 41.95 | 50.03 | 28.40 | 24.42 | 30.93 | 44.20 | 39.97 | 40.27 |
| UniversalRAG | 49.04 | 48.18 | 64.82 | 32.14 | 58.17 | 38.07 | 48.34 | 41.31 | 55.09 |
| MemGuide | 54.98 | 53.07 | 48.33 | 22.85 | 30.40 | 36.66 | 47.98 | 41.97 | 45.43 |
| LightMem | 24.33 | 25.48 | 33.72 | 17.01 | 20.70 | 30.59 | 32.80 | 27.79 | 27.89 |
| VimRAG | 22.51 | 21.36 | 46.99 | 14.14 | 31.81 | 34.64 | 39.00 | 33.67 | 35.08 |
| **GraphMemix** | **58.05** | **57.47** | **81.47** | **35.65** | **66.68** | **43.06** | **63.47** | **49.91** | **67.42** |
| Gain over 2nd | +3.07 | +4.41 | +16.66 | +3.51 | +8.52 | +4.99 | +15.14 | +7.94 | **+12.33** |

要点：两个 backbone 上四基准 J.Acc 全胜；宏平均 61.55 / 67.42。Gemma 设定下四个原生指标也均高于各表中的最强基线（Gallery EM 35.65 > 32.14、MemEye M.EM 43.06 > 38.07、H2H Recall 49.91 > 41.97、ATM EM 57.47 > 53.07），但**原生指标的绝对值随 backbone 波动较大**（MemEye M.EM 43.06 vs Qwen 设定 54.25；H2H Recall 49.91 vs 54.70），论文主推 J.Acc。

### 4.3 效率 / 生命周期成本（§4.3、A.3）

**度量方式**：定义一个完整 lifecycle = 「把全部历史 digest 一遍 + 把全部问题各答一遍」的估计 wall-clock（estimated time to digest all memories once and answer all questions once），并把该时间与 Judge Accuracy 一起画成 (a) 成本分解、(b) 精度-成本散点（图 3、图 5）。

- ATM-Bench：11,034 条记忆 + 1,044 题（图 3 图注）；GraphMemix 相对 **A-MEM 1.78×、VimRAG 4.27×、LightMem 4.74× 缩短完整 lifecycle**（§4.3）。
- 另外三基准（A.3）：GraphMemix 估计 **2.19 h（Mem-Gallery, 7,944 记忆/1,711 题）、2.71 h（MemEye, 3,392/1,855）、3.44 h（H2HMem, 7,078/1,982）**，且各自 J.Acc 最高。
- **Pareto 结论**：四个基准上 GraphMemix 都落在**经验 Pareto 前沿**——「没有任何被比较系统以更短 lifecycle 达到同等或更高精度」（§4.3、A.3）。
- 诚实限定：这是**估计 wall-clock**；**全文未找到硬件/端点/并发配置的说明**（未在全文中找到）；Pareto 只相对「被比较的这几个方法」成立。

### 4.4 消融

**(a) 增量消融（表 3，J.Acc %，同一 reader、同一候选预算）**

| Configuration | MV | Node | ECV | Opt. | ATM | Gallery | MemEye | H2H | Avg |
|---|:-:|:-:|:-:|:-:|--:|--:|--:|--:|--:|
| (a) Embedding Top-K | | | | | 42.80 | 62.50 | 46.70 | 44.80 | 49.20 |
| (b) + Multi-view Retrieval | ✓ | | | | 47.40 | 67.90 | 49.10 | 50.00 | 53.60 |
| (c) + Node Verifier | ✓ | ✓ | | | 53.20 | 73.80 | 52.20 | 56.00 | 58.80 |
| (d) + ECV Reranking | ✓ | ✓ | ✓ | | 54.00 | 74.60 | 52.80 | 57.40 | 59.70 |
| (e) + Forest Optimization | ✓ | ✓ | ✓ | ✓ | **55.27** | **76.33** | **53.64** | **60.96** | **61.55** |

增量（宏平均）：多视图检索 **+4.40**；节点验证器 **+5.20**（最大单项）；ECV 作为「逐点 bonus」**+0.90**；**集合级森林优化再 +1.85（59.70→61.55）**。论文强调最后一段差值「隔离出把证据作为结构化集合来选择、而非给单个候选加关系分数」的收益（§4.4）。

**(b) 恢复低排名关系证据（4.4.1，表 4/表 5）**
- 表 4 gold 证据可及性（Direct top-10 / Recoverable（top-10 没有但有界候选图里有）/ No access，%）：ATM 68.50/15.14/16.35；Mem-Gallery 56.67/37.87/5.46；MemEye 42.08/39.70/18.21；H2HMem 15.61/33.93/**50.46**。
- 表 5（Gemma 4，问题级恢复）：R.Q.（候选图中可恢复 gold 的题数）/ C.Q.（GraphMemix 至少恢复一条 gold 的题数）/ 率 / 占全部保留 gold 对的份额——ATM 165/113/68.48%/12.01%；Gallery 1,142/629/55.08%/19.58%；MemEye 1,256/621/49.44%/19.17%；H2H 1,956/784/40.08%/19.01%。即**在合格问题上恢复 40.08–68.48%，这些记忆占保留 gold 的 12.01–19.58%**。

**(c) 优化行为（4.4.2，图 2）**：6,592 题上审计两层求解器；外层 1-swap 前几次接受的更新即贡献大部分均值目标改进，**6 次后变化可忽略**；每次接受的交换都用精确 Kruskal 评估。

**(d) 查询条件边 vs 固定边（4.4.3，表 6）**：fixed 变体给每条合格显式 schema 边统一 0.99 的查询无关可靠性（无角色门控、无增量分）。ECV 把每题选中边数从约 6–7 条压到 **0.25–1.47** 条，同时 Hit@10 全面提升：ATM 84.20→87.74（+3.54）、Gallery 94.76→96.59（+1.83）、MemEye 85.34→88.89（+3.56）、H2H 89.46→95.06（+5.60）；分量数（Fixed/ECV）3.77/9.34、2.66/8.53、3.93/9.75、3.25/9.00（每题边数与分量数见表 6）。

**(e) 双验证器分工（4.4.4，表 7，随机 100 题/基准 held-out）**：单次调用同时预测两种判断（严格 keyed JSON，两实现都做到完整结构化输出）vs 分离双调用——**分离版宏平均 Acc 61.25 vs 59.25（+2.00），R@10 55.94 vs 52.94（+3.00）**；两验证器无串行依赖、可并行，故分离不引入额外串行成本。

**(f) 受控证据集选择（A.6，表 9/表 10）**——冻结 M=48 候选、校准后节点效用、ECV 输出与 K=10 预算，所有选择器都恰好返回 10 条（Recall@10 %）：

| Selector | ATM | Gallery | MemEye | H2H | Avg |
|---|--:|--:|--:|--:|--:|
| Node Utility Top-K | 82.27 | 68.99 | 58.19 | 18.04 | 56.87 |
| MMR (β=0.05) | 82.27 | 69.39 | 57.94 | 17.76 | 56.84 |
| Rel.–Red. (β=0.02) | 82.27 | 69.12 | 58.00 | 17.87 | 56.82 |
| DPP ((γ,σ)=(8,0.5)) | 76.73 | 67.12 | 49.38 | 17.52 | 52.69 |
| Facility Location (β=0.25) | 82.27 | 68.99 | 57.80 | 18.14 | 56.80 |
| Anchor–Neighbor (η=1.6) | 82.27 | 70.43 | 58.19 | 18.45 | 57.33 |
| ECV Pointwise (η=0.4) | 82.27 | 70.24 | 58.29 | 18.25 | 57.26 |
| Greedy Forest | 82.27 | 70.31 | 58.09 | 18.50 | 57.29 |
| **Forest Proposal (ours)** | 82.27 | **70.58** | **58.49** | **19.63** | **57.74** |

结论：**通用多样性/覆盖（MMR、Rel-Red、DPP、facility）相对纯节点选择没有提升**（最好 56.84 vs 56.87）；ECV 逐点 bonus 57.26；**联合节点+边 57.74**；同一目标换成前向贪心只有 57.29（**把目标与搜索分开**：确定性 refinement 优于前向贪心）。

净低排名 gold 恢复（式 16 `ΔG_q(S)=|S_q∩G_q|−|S_q^{node}∩G_q|`，可恢复问题数 ATM/Gallery/MemEye/H2H = 12/62/66/98）：Node 0、MMR −6、Rel–Red −4、DPP −36、facility −2、Anchor–Neighbor +14、ECV Pointwise +12、Greedy Forest +18、**Forest Proposal +43（H2H 单独 +31）**——「森林不只是通用多样性先验，它把已验证的关系路径转化为可测量的低排名证据恢复」（A.6）。

**(g) 定性（4.5，图 4，Mem-Gallery）**：独立排序只取到部分答案历史；GraphMemix 做 5 次改进性 1-swap，把孤立对话轮换成成对轮次；随后 Kruskal 更新把目标从 **1.340 → 2.133**，gold 覆盖 **3/4 → 4/4**，最终给出两只狗（Lumi 马尔济斯、Coco 玩具贵宾）的完整答案。

---

## 5. 代码研读

仓库根：`docs/sources/repos/graphmemix/`（对应 github.com/ligeng0197/graphmemix，version 0.1.0）。

### 5.1 结构

```text
graphmemix/
├── README.md                      # 主结果表、MobileMem 榜、安装/运行、ATM-Hard 扩展
├── pyproject.toml                 # MIT；可选 extras [methods]；入口 graphmemix/mmmb = mm_memory_bench.cli
├── CITATION.cff                   # date-released 2026-08-27，preferred-citation 为 arXiv 条目
├── MANIFEST.sha256 / release/repository_allowlist.txt / scripts/audit_graphmemix_release.py
├── config/                        # datasets.json / evaluation.json / main_track.json（通用默认）
├── configs/release/
│   ├── graphmemix.json            # ★ 冻结的端到端执行契约（见 5.2）
│   ├── calibration/shared_dev.json# ★ 节点效用的机器精确 logistic 标定系数
│   ├── atm_hard_graph_proof.json  # ATM-Bench-Hard 扩展契约
│   └── prompts/atm_hard_evidence_graph_v3_compact.txt
├── schema/mmmb-1.0.schema.json    # 统一记忆 JSON schema
├── docs/                          # graphmemix_release.md / unified_format.md / llm_judge.md / h2h_evidence_track.md
├── release/                       # benchmark_results.json（冻结结果清单）、mobile_mem_leaderboard.json、atm_hard_price_performance.json
├── scripts/                       # ★ 流水线各阶段 + 求解器 + 审计（见 5.3）
├── src/mm_memory_bench/           # 统一评测 harness、4 个基准转换器、基线方法适配器
│   ├── core.py harness.py judge.py oracle.py reader.py cli.py ...
│   ├── converters/{atm,mem_gallery,memeye,h2hmem}.py
│   └── methods/{base,backends,vector_index,gme,media,universalrag,
│                concrete_amem,concrete_lightmem,concrete_memguide,
│                concrete_memix,concrete_universalrag,concrete_vimrag}.py
└── tests/                         # pytest -q tests（README 明确不要裸跑 pytest）
```

### 5.2 冻结配置（`configs/release/graphmemix.json`，与论文 §4.1 一一对应）

`source_top_k: 200`（检回深度）、`source_top_l: 24`（=L）、`candidate_limit: 48`（=M）、`semantic_k: 8`、`anchor_count: 10`、`reader_k: 10`（=K）、`edge_weight: 0.1`（=λ）、`proposal_root_cost: 0.2`（=κ_prop）、`pruning_root_cost: 0.12`（=κ）、`edge_power: 1.0`、`snippet_chars: 420`、`verifier_ocr_chars: 120`、`reader_ocr_chars: 320`、`max_model_len: 32768`、judge `gpt-5-mini` 且 `excluded_from_method_cost: true`；`node_utility_summary` 明写 `alpha 0.8 / tau 5.2 / delta 0.7` 并注明是「human-readable approximation；标定文件里才是机器精确系数」。

`configs/release/calibration/shared_dev.json`：协议 `graphmemix-shared-logistic-devfold-1.0`，特征 `[atomic_score, verifier_score]`，**机器精确系数 a=4.054056101800787、b=0.22537987366704051、c=−3.711336391577528**（L2=0.001，62,988 候选 / 8,491 正例 / 患病率 13.48%），并带 warning：「gold evidence 只用在 SHA256 fold 0 的开发问题上」——即**式(14)(15) 的参数来自 dev fold 上的 gold 标定**（论文 A.2 只说「不按基准重调」，未披露标定用 gold；这是代码比论文更透明的一处）。

### 5.3 关键实现文件（按流水线阶段）

| 阶段（`run_graphmemix_release.py` 的 stages） | 文件 | 做什么 |
|---|---|---|
| `checkpoint` | `scripts/build_memix_checkpoints.py` | 统一快照检查点 |
| `priors`（检回先验） | `scripts/run_memix_main_track_retrieval.py`、`run_memix_retrieval_only.py` | 冻结 Atomic 检回，产出 `source_priors/{ds}.jsonl`（retrieval_ids/scores） |
| `relations`（★图构造） | `scripts/build_graphmemix_relation_store.py` | `explicit_edges()`（同轮/相邻轮等显式关系）、`spatiotemporal_edges()`（含 `haversine_meters` 的 event_bridge：1 分钟–1 小时、≤5km、每记忆最多 2 个局部邻居）、`mutual_neighbors()`（互为 kNN 的语义边）、`merge_explicit_edges()`，产出 `relation_store/{ds}.jsonl` |
| `node`（节点验证器） | `scripts/score_graphmemix_verifier.py` | listwise 打 `v_i∈[0,5]`，5-fold 抽样、严格 JSON 解析、模态标签 |
| `ecv`（关系验证器） | `scripts/score_graphmemix_ecv.py` | 一次 listwise 读锚-候选对，输出 `edge_scores{role, incremental_support}`；六角色常量与 `POSITIVE_ROLES={"new_fact","clarification","corroboration"}` 一致 |
| `select`（★森林选择） | `scripts/evaluate_graphmemix_ecv.py` | `conditioned_adjacency()`：`similarity = 0.99 * (incremental/5)^power`、只留正角色边；`prizes()`：`expit(a·atomic + b·verifier + c)`；再调求解器（`--edge-weight 0.1 --root-cost 0.2 --k 10`，power 试 1.0/2.0） |
| 求解器（核心） | `scripts/probe_graphmemix_pcsf.py` | **两个精确/近似求解器**：① `solve_forest()`——用 `scipy.optimize.milp` 解** prize-collecting Steiner forest 形式的 MILP**（变量：x 选点 / r 根弧 / y 有向边 / 流量守恒；`Σx = k` 固定基数；`time_limit=5.0, mip_rel_gap=0.0`），返回目标值、边数、分量数、求解秒数；② `solve_forest_greedy()`——**确定性 1-swap 搜索，每个节点集用 Kruskal 精确求森林**（`benefit = root_cost − edge_weight·(−log sim)`，只收 benefit>0 的边），即论文 §3.5 描述的算法，返回 `local_search_iterations` 与 `exact: False` |
| `prune`（★变基数精确细化） | `scripts/evaluate_graphmemix_exact_pruning.py` | `exact_subset()`：**枚举全部非空子集（n≤20，`2^n−1`），每个子集内 Kruskal 精确森林**，`value = Σ(prize − node_cost − root_cost) + edge_benefit`；平局「先少节点、再早排名」；元数据诚实标注 `exact: True / exact_scope: "frozen_input_topk" / glob​ally_exact_over_candidate_pool: false` |
| `reader` / `judge` | `scripts/run_memix_answers_from_retrieval.py`、`judge_atm_official_prompt.py`、`compute_native_metrics.py`、`score_*` | 检索行哈希绑定的 reader 恢复、官方 judge prompt、原生指标 |
| 审计 | `scripts/audit_graphmemix_release.py`、`export_graphmemix_release.py` | 按 `release/repository_allowlist.txt` 校验可分发树 |
| ATM-Hard 扩展 | `scripts/select_atm_hard_graphmemix_forest.py`（`structural_coverage` / `edge_incidence_scores` / `graph_obligation_ledger` / `select_question`）、`run_atm_hard_graph_proof_release.py`、`plot_atm_hard_price_performance.py` | v3 节点 prize = 图置信度 + 逆 hub 结构覆盖 + 有界语义 source-prior 排名奖励 + 关系无关的边入射项；只按任务类型（list_recall/number/open_end）路由冻结预算与权重，**选择器从不读问题文本/答案/gold** |

**评测脚本**：`scripts/evaluate_atm_retrieval.py`（Recall@K/Hit@K、gold 对齐）、`scripts/evaluate_graphmemix_exact_pruning.py`（消融式精确剪枝）、`tests/test_graphmemix_*.py`（core/pcsf/relation_store/release 各有单测）。

### 5.4 代码与论文的差异 / 可借鉴要点

1. **MILP 求解器是论文没写的**：`probe_graphmemix_pcsf.py:solve_forest()` 给出了完整 prize-collecting Steiner forest 的 MILP（固定 K、根弧、单商品流），而论文只讲「1-swap + Kruskal」。两者数学同源（`−log sim` 边成本、root_cost、固定基数），MILP 可作为小规模精确基准/离线对照。
2. **精确性范围被代码明确标注**：`exact_subset` 只在「frozen top-K」上精确，`globally_exact_over_candidate_pool: false`——比论文 §3.5 的表述更保守、更准确，写论文式引用时应引用代码口径。
3. **边权变换是可调的形状参数**：`similarity = 0.99·(incremental/5)^power`，power ∈ {1.0, 2.0} 做过扫描（`--edge-power` 可重复传入）——把「角色门控 + 连续增量分」压成一个 [0,0.99] 的伪相似度，再取 `−log` 当成本，是把 LLM 判断塞进图算法的干净接口。
4. **确定性与可复现工程**：所有阶段产物记录输入契约哈希，不兼容行拒绝 resume（`docs/graphmemix_release.md`）；judge 行存 requested model、prompt 协议版本与 prompt SHA-256（A.2）；结果冻结在 `release/benchmark_results.json`。
5. **reader 上下文即「预算」的落地形态**：`reader_k=10` 条 × `snippet_chars=420`（OCR 另有 120/320 字符预算），序列化按「分量根→BFS」——这是一个**按条数计的预算**，不是按 token 计；token 级只约束 `max_model_len/reader_max_output_tokens`。
6. **仓库还含论文没有的结果**：README 的 MobileMem 榜（MobileMem-Omni 16 用户/7,415 题，GraphMemix LLM-Judge 34.79 vs 次优 LightMem 27.08，F1 40.32；MobileMem-Struct 54.72 vs MemOS 24.17，注明只复现 10 个 persona 中的 9 个）；以及 ATM-Bench-Hard 扩展 **57.95 QS / $3.61**（31 题，`release/benchmark_results.json` 自己标注 `development-set extension; not part of the held-out benchmark suite`，`docs/graphmemix_release.md` 也承认「在同一 31 道公开 hard 题上调过 aggregate evidence-recall，不是独立 held-out」）。这些数字**不能**当作论文主结果引用。

---

## 6. 摘要级理解的修正

摘要（及 `papers-abstracts.md` §4）只给了「三组件 + 预算 + Pareto」的骨架；全文补上了以下摘要里没有或含糊的要点：

1. **目标函数的具体形态**：摘要说「jointly selects ... under a maximum evidence budget」，全文给出的是式(2)/(11)：**效用 − 边激活成本 − 开链成本**，且经式(12) 化简为 `Σ(p_i−κ) + Σ B_e`，`B_e = κ − λ c_e`——「预算」在目标里其实是 **K 的基数硬约束 + κ 的软开链成本** 双重机制；**自适应基数来自 κ，而非单独的 cardinality penalty**（§3.5 明说）。摘要完全没提 `m(S,F)`（连通分量数）这个项。
2. **求解不是「一个优化器」而是两阶段确定性管线**：固定基数 1-swap proposal（每步精确 Kruskal）→ 冻结 proposal 内 `2^K−1` 子集枚举的**全局精确**细化；摘要的「forest optimization」看不出「哪一层精确、哪一层启发式」。论文还给了实证收敛（图 2：6 次 swap 后饱和）。
3. **成本的度量方式是 lifecycle wall-clock**：摘要的「lifecycle cost」= 「digest 全部记忆一次 + 回答全部问题一次」的**估计耗时**，ATM 上相对 A-MEM/VimRAG/LightMem 缩短 1.78×/4.27×/4.74×，另三基准 2.19/2.71/3.44 h，四基准经验 Pareto；**不是 FLOPs、不是 token 计数、硬件未披露**。
4. **消融的量级排序**（摘要只说「significant improvements」）：节点验证器 +5.20 > 多视图检索 +4.40 > 森林优化 +1.85 > ECV 逐点 bonus +0.90（表 3）——**最大单项是节点验证器，不是图优化**；图优化的价值要靠「受控选择 + 净 gold 恢复 +43」这类细粒度实验才能看清（A.6 表 9/10）。
5. **ECV 的真实作用是「剪边」而非「加边」**：固定边时每题选 6–7 条边，ECV 后只剩 0.25–1.47 条，同时 Hit@10 提升（表 6）——「抑制无条件传播」比「增加结构」更贴近数据。
6. **参数是跨基准冻结的**：`(α,τ,δ)=(0.8,5.2,0.7)`、λ=0.1、κ=0.2/0.12、L=24/H=1/k=8/M=48/K=10 全套固定（A.2「We do not retune these values by benchmark or reader backbone」）；代码进一步显示标定系数来自 dev fold 的 gold logistic 拟合。
7. **验证器分工是被单独验证的设计**（4.4.4）：分离双调用 +2.00 Acc / +3.00 R@10，且可并行不加串行成本——摘要的「decouples direct memory support from anchor-conditioned relation verification」背后有对照实验支撑。

---

## 7. 对 OIA 的借鉴更新

> OIA 侧语境（来自 `omni-interaction-agent/README.md`）：OIA-Memory 五组件 = Vault 原文库 / Distiller 蒸馏 / Canvas 符号画布 / **Recall 分级召回** / Governance 人审治理，证据链硬约束「画像→场景→事实→原话」。以下为把 GraphMemix 的算法落到「Recall 预算精排器」与「DSH compaction 预算」的设计建议（**未改动 OIA 任何代码，属于方案层建议**）。

### 7.1 Recall 预算精排器的具体算法落点

**落点 1 — 目标函数照抄式(12)，按条数预算 K。**
在 Recall 的候选池（M 条）上定义：`p_i = σ(a·retrieval_score + b·verifier_score + c)`（复用其标定思路：先在 dev fold 上做一次 logistic 标定，把异构分数压到 [0,1] 同一尺度），边 `B_e = κ − λ·(−log w_e)`，优化 `Σ_{i∈S}(p_i − κ) + Σ_{e∈F} B_e`，约束 `|S| ≤ K`。要点：
- **κ 是「开一条新证据链的成本」**——正好对应 OIA「画像→场景→事实→原话」链条里**每条独立链都要占用 reader 预算**的事实；`B_e > 0` 的边（`w_e > e^{−κ/λ}`）才允许把两个候选并进同一条链，天然产出「链内紧凑、链间付费」的结构。
- 代码口径可直接搬：`benefit = root_cost − edge_weight·(−log max(sim,1e-6))`，只收 benefit>0 的边，Kruskal 按 benefit 降序连边（`probe_graphmemix_pcsf.py:solve_forest_greedy` / `evaluate_graphmemix_exact_pruning.py:exact_subset`，约 60 行无依赖实现）。

**落点 2 — 两阶段求解器照抄 §3.5，复杂度可控且确定性。**
- 阶段一：top-K 初始化 + 1-swap 局部搜索，每个提案集用 Kruskal 精确森林打分（`O(K·(M−K)·E log E)`，M=48、K=10 时毫秒级）；图 2 的经验「6 次 swap 后饱和」可以直接作为早停阈值。
- 阶段二：在冻结的 K 条 proposal 上枚举 `2^K−1`（K=10 → 1023）子集取全局最优，平局「先少条、再早排名」。**这条平局规则对 OIA 很重要**：预算紧时宁可少给一条不确定证据。
- 需要严格最优时可上 `scipy.optimize.milp` 的 PCSF 形式（`solve_forest()`），但注意它 `time_limit=5.0`，只适合离线/小规模。

**落点 3 — 双验证器 + 角色门控（式 7–9 的工程化）。**
- 节点验证器问「这条记忆**单独**能不能答」；关系验证器（ECV）问「在锚 a 条件下它**增量**了什么」，并给出六角色；**只有正增量的前三角色进 `E_q^trust`**。OIA 的 Recall 可以把「冗余/冲突」显式判掉——这正是抑制重复召回的机制，比 MMR 的相似度惩罚更可控（表 9：MMR/Rel-Red/DPP/facility 相对纯 top-K **零增益甚至负增益**）。
- 角色门控 + 连续分压成 `[0, w_max]` 伪相似度的接口（`0.99·(inc/5)^power`）可以直接复用，`power` 是免费的锐化旋钮。
- **锚集必须在验证器之前冻结**（A.2）：由检回 top-L 去重保序产生，不用验证器分数、不用 gold——避免循环选择。OIA 里对应「锚来自 Vault 的原文命中，不来自蒸馏后的判断」。

**落点 4 — 序列化协议（A.5）直接对应 OIA 证据链下钻。**
分量按根效用排序、分量内 BFS、平局按效用→检回顺序：保证 reader 先看到直接锚点，再看到经可信关系恢复的上下文。这与「画像→场景→事实→原话」的下钻顺序同构，可作为 Recall 输出的固定序列化规则（确定性、可测、可回归测试）。

### 7.2 与 DSH compaction 预算的联动

- **把 compaction 的 token 预算 K_token 映射成条数预算 K**：GraphMemix 的 `K=10 × 420 字符 snippet（OCR 另计 120/320）` 是「先按条数选、再按字符截」的两级预算。对 DSH compaction，可用同样两级：先用式(12) 在候选记忆上解出森林（条数预算 K = floor(B_token / avg_snippet_chars)），再对每条做字符/OCR 预算分配（`memory_snippet()` 的「语义 2/3 + OCR 1/3 保留」策略可直接抄，`scripts/graphmemix_core.py`）。
- **κ 与 λ 是预算的「影子价格」**：`κ` 越大 → 分量越少、证据链越集中（压缩后段落更少更长）；`λ` 越大 → 越不敢用关系边扩展（更保守）。**调 DSH compaction 的紧凑度时，优先调 κ 而不是直接砍 K**——因为 κ 同时控制「条数」与「链数」，且式(12) 让它自动变成自适应基数（低效用孤立证据自己掉出去），避免「固定 top-K 把刚够用的证据挤掉」的老问题。
- **lifecycle 视角**：GraphMemix 的 Pareto 论证是「把生成式处理从全历史挪到查询时的有界子图」。对应到 DSH：**compaction 触发时不要对整段历史重跑生成式摘要，而是只对「当前查询相关的候选子图」做证据重排**——即 compaction 的代价应随查询局部化（论文 §4.3 的 1.78×–4.74× 缩短正是来自这一点）。
- **评测口径可借鉴**：把「compaction 后回答质量 vs compaction 耗时」画成与图 3(b) 同构的精度-成本 Pareto 图，并用「冻结候选 + 冻结预算」的受控选择实验（A.6 表 9 的做法：所有选择器返回同样的 K，只比选择策略）来隔离精排器本身的贡献——否则预算变化会污染归因。

### 7.3 不建议照搬的部分

- **不要为了「图」而引入 LLM 全量关系推理**：GraphMemix 的关系是 query-independent 的便宜结构（同轮/相邻轮/时空规则/互为 kNN），昂贵的只有两次 listwise 调用；OIA 若把关系也交给生成式推理，会把摘要类方法的 lifecycle 成本又带回来。
- **K=10、M=48、λ=0.1、κ=0.12/0.2 是该论文四个多模态基准上的取值**，不是普适常数；OIA 需在自己的 dev 集上按 A.6 的受控协议重标。

---

## 8. 诚实边界

1. **venue 未定**：仅为 arXiv 预印本（2608.26983v1，2026-08-27），全文与 CITATION 均无会议/期刊录用信息；所有 baseline 的 venue 标注（NeurIPS'25/ACL'26/AAAI'26/ICLR'26/ICML'26）是**论文自报**，未独立核实。
2. **无 Limitations 章节**：全文没有独立的局限性讨论（grep 只命中 §1/§5 里指代「两条限制」的 limitation 一词）。
3. **生命周期成本是「估计 wall-clock」**：硬件、端点、并发、缓存策略**未在全文中找到**；Pareto 前沿只相对被比较方法成立，不能外推到「所有记忆系统」。
4. **主指标是 LLM-as-a-Judge**（GPT-5-mini）；人工一致性 194/200=97.0%（A.7）但样本仅 200。原生指标随 backbone 波动（如 MemEye MCQ EM 在 Gemma 设定下 43.06 vs Qwen 设定 54.25）。
5. **标定用过 gold**：节点效用 logistic 系数来自 dev fold（SHA256 fold 0）上的 gold 标定（代码 `shared_dev.json` 的 warning；论文 A.2 只说「不按基准重调」）。A.1 同时声明 gold 不进入检索/验证/生成——两者不矛盾，但**标定阶段接触 gold** 这点论文正文没有明说。
6. **精确性的范围**：全局最优只在「冻结的 K 条 proposal 子集」内（代码 `exact_scope: "frozen_input_topk"`、`globally_exact_over_candidate_pool: false`）；候选图之外、以及阶段一的 1-swap 搜索都不精确，无近似比保证。
7. **恢复率的分母口径**：表 5 的「40.08–68.48%」是**合格（recoverable）问题**上的比例，份额 12.01–19.58% 是占全部保留 gold 对的比例——不是端到端 recall。
8. **代码仓库含非论文结果**：MobileMem 两榜与 ATM-Bench-Hard 57.95 QS/$3.61 均为**仓库自报**，其中 ATM-Hard 明确是 31 题开发集结果（`development-set extension`、`not an independent held-out result`），**不可与论文四基准主结果混引**。
9. **本笔记的复杂度分析**（§3.3 的大 O 直觉）是按算法描述的推算，论文未给出显式复杂度、近似比或收敛定理——标注为推算，不是论文原文。
10. **OIA/DSH 借鉴部分是方案建议**：未在 OIA 代码库中验证，未做任何实验；与 DSH compaction 的联动属于设计推演。

---

### 自查记录

- 公式均取自 HTML `alttext=` 原文并标注式号（式 1–16）；章节出处（§/表/图/A.x）逐条对应。
- 四基准数字、表 1/2/3/4/5/6/7/9/10 全部自报数字已录入；未找到的（venue 录用、lifecycle 硬件配置、复杂度定理）明确写「未在全文中找到」。
- 代码路径均为仓库内实际文件（`scripts/graphmemix_core.py`、`scripts/probe_graphmemix_pcsf.py`、`scripts/evaluate_graphmemix_exact_pruning.py`、`scripts/build_graphmemix_relation_store.py`、`scripts/score_graphmemix_ecv.py`、`scripts/evaluate_graphmemix_ecv.py`、`configs/release/graphmemix.json`、`configs/release/calibration/shared_dev.json`、`release/benchmark_results.json` 等）。
