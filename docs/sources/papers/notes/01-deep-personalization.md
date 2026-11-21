# 01 · Deep Personalization: Continuous Context Observation for Local Proactive Agents 精读笔记

> arXiv:2611.13457v1 [cs.LG]，2026-11-10 提交（HTML 头部：`arXiv:2611.13457v1 [cs.LG] 10 Nov 2026`）
> 本地存档：`docs/sources/papers/2611.13457-deep-personalization.html`（arXiv HTML 全文，354,270 字节 / 2,376 行）
> 配套代码：`docs/sources/repos/deep-personalization/`（主仓，论文脚注 1 指向 `https://github.com/NVlabs/deep-personalization`）、`docs/sources/repos/coco/`（论文脚注 2 指向 `https://github.com/collaborative-agents/coco`）
> 摘要对照：`docs/sources/papers-abstracts.md` §1（与 HTML 摘要逐句一致）
> **阅读级别：全文（正文 §1–§8 + 附录 A–F 全部读完，含全部表格与 prompt 附录）**
> 来源标注约定：【摘要】【正文 §n】【附录 X】【代码】【代码推断】【未找到】

---

## 1. 论文速览

| 项目 | 内容 | 来源 |
|---|---|---|
| 标题 | Deep Personalization: Continuous Context Observation for Local Proactive Agents | 【摘要】 |
| 系统名 | **Coco**（co-ntinuous co-ntext observation 的缩写，正文 §3 开篇明示） | 【正文 §3】 |
| 作者 | Yijia Shao, Song Bian, Junxian Guo, Jindong Jiang, Zhen Dong, Huizi Mao, Diyi Yang, Song Han | 【摘要页】 |
| 机构 | NVIDIA、Stanford University、Massachusetts Institute of Technology、OpenAI（首页作者块并列给出） | 【摘要页】 |
| 版本/日期 | v1，2026-11-10，分类 cs.LG | 【摘要页】 |
| Venue | **未在全文中找到**会议/期刊标注——HTML 头部只有 arXiv 行与 `License: arXiv.org perpetual non-exclusive license`，无 comments/venue 字段 | 【未找到】 |
| 页数 | **未在全文中找到**页码（arXiv HTML 版无分页标记）；结构上为正文 8 节 + 附录 A–F + 参考文献 | 【未找到】 |
| 许可 | arXiv.org perpetual non-exclusive license（论文）；代码仓库 **Apache License 2.0**（两仓 `LICENSE` + README License 段） | 【摘要页】【代码】 |
| 代码链接 1（主仓） | `https://github.com/NVlabs/deep-personalization` —— 脚注 1：「Deep Personalization code and OSWorld-Persona data are available at ...」 | 【正文 §1 脚注 1】 |
| 代码链接 2（集成） | `https://github.com/collaborative-agents/coco` —— 脚注 2：「Desktop proactive agent integration is available at ...」 | 【正文 §1 脚注 2】 |
| 数据 | HuggingFace 数据集 `collaborative-agents/OSWorld-Persona`（README 给出 `hf download` 命令） | 【代码】 |
| 演示视频 | Figure 1 图注：Observer Agent 跑 Qwen3.5-35B-A3B-FP8 on NVIDIA DGX Spark，`https://youtu.be/fe7MSjoJqZM` | 【正文 Fig.1 图注】 |
| 博客/官网 | `https://nvlabs.github.io/deep-personalization/`（coco README 顶部 Blog 链接；主仓 `website/index.html` 内同时链到两个 GitHub 仓） | 【代码】 |

**官方代码 vs 论文声称（逐项核对）：**

| 论文声称的贡献 | 代码落点 | 核对结论 |
|---|---|---|
| OSWorld-Persona benchmark + 五阶段数据合成管线 | `benchmarks/osworld_persona/`（含 `synthesis/`） | ✅ 完整实现，含 prompt、CLI、测试 |
| 三种适配方法（Vanilla / Induced Persona / Oracle Persona / Coco-PE / Coco-LoRA） | `methods/prompting.py`、`induced_persona.py`、`coco_pe.py`、`coco_lora.py` | ✅ 完整实现，超参与附录 E 一一对应 |
| 推理/训练后端 | `inference_backends/{openai_compatible,llamafactory_server,tinker}.py`、`training_backends/{llamafactory,tinker}.py` | ✅ |
| on-device 部署策略（change-aware frame selection + observation-first scheduling） | 主仓**无**运行时实现；MSE 选帧逻辑在 `coco/lib/sensing/sensing/streamer.py`（`threshold: float = 8000.0`）与 `deep-personalization/.../synthesis/frames.py`（数据合成用）；调度在 `coco/desktop/src/main/services/personalization-scheduler.ts` | ⚠️ 分散在 coco 仓 + 外部 `mit-han-lab/cti-engine`，主仓不含 |
| 记忆基准对比（Table 3：LoCoMo / LongMemEval / BEAM-100K / BEAM-1M） | **未在两仓中找到**对应评测脚本 | ❌ 未见复现代码 |
| 一周用户研究（10 人） | 桌面端实现在 coco 仓（`desktop/`、`lib/personalization/`）；**研究数据不发布**（Ethics Statement 声明只收集聚合计数） | ⚠️ 有实现、无数据 |
| Observer Agent 提示词（附录 B.1） | 三个版本并存：论文 B.1 / benchmark `assets/observer_prompt.txt` / 部署版 `coco/lib/sensing/sensing/prompts_everyday/observer.txt` | ⚠️ 三者内容不同（见 §5、§6） |

---

## 2. 全文结构（章节地图）

按 HTML 行号定位（`2611.13457-deep-personalization.html`）：

| 章节 | 行号 | 要点 |
|---|---|---|
| Abstract | L226–231 | deep personalization 定义、Coco、OSWorld-Persona、0.12-0.19→0.35-0.48（2.4–3.0×）、用户研究 0.199→0.515 |
| Figure 1（系统总览） | L234–238 | 闭环：观察 → 长期用户建模 → 从隐式反馈自适应 |
| §1 Introduction | L240–290 | 三条技术贡献 + 三条 list 贡献 + 两个代码脚注 |
| §2 Learning Personalized Agents from Continuous Observation | L292–310 | 形式化：策略 π、式 (1) 学习目标；连续观察作为 s_t 与 r_t 的共同来源 |
| §3 Method | L312–373 | §3.1 Observer Agent（式 2、3）；§3.2 长期记忆（二部图 + 检索）；§3.3 隐式反馈自适应 |
| §4 Efficient On-device Deployment | L374–392 | change-aware frame selection、observation-first scheduling、DFlash 投机解码 |
| §5 Experiment | L394–432 | §5.1 OSWorld-Persona 五阶段管线；§5.2 模型/基线/指标 |
| §6 Results | L434–861 | §6.1 主结果（Table 1/2/3）；§6.2 用户研究（Figure 4） |
| §7 Related Work | L863–876 | Proactive computing / 个性化数据源 / 个性化适配 三段（略读） |
| §8 Conclusion | L878–882 | — |
| Acknowledgments / AI Use / Ethics | L885–913 | 隐私张力、同意与可撤销、不发布用户数据 |
| References | L915–1449 | 50 条（`ltx_bibitem` 计数 = 50） |
| **附录 A** Limitations and Future Work | L1451–1484 | 四小节：用户控制 / 增强而非替代 / 隐私 / 能耗 |
| **附录 B** Prompts Used in Coco Observer Agent | L1486–1965 | B.1 Observer prompt；B.2 记忆三 prompt；B.3 隐式信号三 prompt；B.4 Coco-PE 三 prompt |
| **附录 C** OSWorld-Persona Construction Details | L1967–2167 | Table 4 统计 + 五阶段细节 + 两阶段标注 prompt |
| **附录 D** Profiling On-device Deployment | L2169–2258 | Table 5 延迟/KV 命中；CTI 引擎与两项优化 |
| **附录 E** Experiment Details | L2260–2277 | Coco-PE / Coco-LoRA 全部超参 |
| **附录 F** Coco Desktop Interface | L2279–2304 | 渐进披露、三入口、活动/个性化视图 |

---

## 3. 方法精读

### 3.1 问题形式化【正文 §2】

状态 $s_t$（环境）、内部状态 $h_t$、策略 $\pi(a_t \mid s_t, h_t)$、反馈 $r_t=\mathcal{R}(a_t)$（由用户 $u$ 给出），目标是找学习算法 $\mathcal{L}$。原文公式（alttext 提取，式 1）：

```latex
\pi_{u}^{*}=\mathcal{L}(\tau^{(u)})\propto\mathbb{E}_{\tau^{(u)}\sim\pi}\left[\Sigma_{t=1}^{T}\gamma^{t-1}r_{t}\right],
```

其中 $\tau^{(u)}=(s_{1},h_{1},a_{1},...,s_{T},h_{T},a_{T})$，$\gamma\in(0,1]$。

关键论点：**连续上下文观察同时提供 $s_t$ 与 $r_t$ 的来源**——屏幕截图+交互历史给出状态，用户对干预的后续反应（编辑草稿/采纳/忽略/另行求助）给出监督，构成 closed-loop deep personalization【正文 §2 末段】。

### 3.2 Observer Agent：输入、输出与紧凑上下文【正文 §3.1 + 附录 B.1】

**核心公式**（alttext 提取）：

```latex
(y_{t},h_{t})=\mathcal{O}(s_{t},h_{t-1}),          % 式 (2)
h_{t}=\operatorname{Retrieve}(\mathcal{M},o_{t},t). % 式 (3)
```

- $s_t$ = 桌面截图 + 时间戳；$y_t\in\{0,1\}$ 是否主动协助；$y_t=1$ 时唤起下游 Execution Agent，$y_t=0$ 继续被动观察【正文 §3.1】。
- $\mathcal{O}$ 参数化为**多模态语言模型**（至少 text+image），单次 prompt 输出结构化四字段 $o_t$：
  1. `observation` —— 从近期电脑活动中蒸馏的**紧凑增量更新**（compact incremental memory update）；
  2. `user_intent` —— 推断的即时目标；
  3. `rationale` —— 基于旧内部状态+新截图的干预决策简述；
  4. $y_t$ —— 二值决策。
  每个 $o_t$ 写入长期记忆 $\mathcal{M}$，再由检索构造下一时刻 $h_t$【正文 §3.1】。
- $y_t=1$ 时 Execution Agent 以 $h_t$ 为初始上下文，并可把 $\mathcal{M}$ 当作**可搜索工具**做多轮检索（论文比喻：文件系统 vs 编码 agent）【正文 §3.2 Context Management】。

**Observer prompt 的输入块（附录 B.1，逐块）**——这就是「Observer 特征」的实际形态（**未在全文中找到独立的特征表**，输入块即特征清单）：

| 输入块 | 内容 | 备注 |
|---|---|---|
| `<memory>` | 跨会话积累的长期个性化上下文：偏好、可用 AI 工具、以往有效的做法、重复任务；常为 `(no memory yet)` | 明确「不是当前屏幕的描述」 |
| `<screenshots>` | 按时间排序的历史帧，每帧带时间戳与 monitor ID；动作触发序列以 before 帧开始、after 帧结束；**被丢弃帧上的键鼠动作前移到下一保留帧** | 直接对应 §4 的 change-aware frame selection |
| `<recent_observations>` | 最近几次 observation 及用户对气泡的反应：`ACCEPTED / DISMISSED / NEGATIVE rating / ignored / no feedback` | 会话开始可缺省 |
| `<recent_propositions>` | 长期记忆对近期活动的更高层总结，时间跨度更长 | 由 GUM 合成 |
| `<relevant_propositions>` | **最多 3 条**按相关度排序的长期记忆摘要（与 explicit user input 和最新 observer state 相关） | 「是检索提示，不是当前屏幕证据」 |

职责四条：理解时间线（多屏连续工作流、monitor 归属）、描述当前屏状态（**不要把最后一张图当作唯一当前屏**）、推断意图、检测支持机会（`yes` 的标准：收益明显大于打断成本）。输出 JSON schema 见 B.1（`observation/user_intent/rationale/need_support`）。

### 3.3 长期记忆：命题-观察二部图【正文 §3.2 + 附录 B.2】

- $\mathcal{M}$ 是**二部图**：一端是带时间戳的 Observer 输出 $\{o_1,\dots,o_T\}$，一端是高层用户命题 $\{p_1,\dots,p_N\}$；命题用 **General User Model (GUM, ref [11])** 由**同一个 $\mathcal{O}$ 模型**定期合成【正文 §3.2】。
- 新命题生成时，检索 **top-k 最相似既有命题**判断「新建 vs 修订」，修订时同步更新支撑观察集合【正文 §3.2】。
- **上下文构造规则**：产出 $o_t$ 后，取 **k 条最近 Observer 输出** + **与 $o_t$ 语义最相关的 k 条命题**，拼成 $h_t$。**k 的具体取值未在全文中找到**（正文只写符号 k）【未找到】。
- 三个 B.2 prompt 的字段细节（代码级可直接抄）：
  - **Proposition Synthesis**：每条 observation 带 `[observation_id=…]`；返回 `{"propositions":[{"proposition","reasoning","confidence","decay","observation_ids"}]}`；`confidence` 1–10 = 证据强度，`decay` 1 = 短暂上下文 / 10 = 持久上下文；要求只引用直接支撑的 observation id；「不要把可见内容当作持久偏好的证据」【附录 B.2】。
  - **New Proposition Classification**：三分类 `IDENTICAL | SIMILAR | UNRELATED` + `target_ids`【附录 B.2】。
  - **Proposition Consolidation**：把相似簇重写/合并/拆分为 ≤ `max_propositions` 条非重叠命题【附录 B.2】。

### 3.4 隐式信号 → 标签的具体规则【正文 §3.3 + 附录 B.3】

**直接信号**：$y_t=1$ 的时刻，用户参与 → $r_t=1$；拒绝 → $r_t=0$【正文 §3.3】。

**退化风险与补救（论文明确论证）**：只用这两个信号会让 $\mathcal{O}$ 学到「少干预 → 更少信号」的退化策略。于是用**同一个 on-device 模型回溯扫描 $\{h_1,\dots,h_T\}$**，按启发式（重复同一操作 / 卡住 / 显式调用 AI）找「用户事后表现出需要帮助证据」的时刻；若这种证据出现在 $y_t=0$ 之后，则给该决策打**负反馈（missed intervention opportunity）**。论文自陈：这是「潜在支持机会」而非「已确认偏好」，但提供探索信号，后续相似情形的真实反馈会继续校准【正文 §3.3】。

**回溯三段式 prompt（附录 B.3）**：

1. *Prompt to Discover Missed Support Opportunities* —— 在**整条时间线**上（而非逐条标注）找高层可复用机会，类别枚举：`repetitive_work | stuck | anticipate_need`。硬规则：机会必须「在后继行为之前就可推断」；**事后手动打开 ChatGPT/agent 是正面证据**（要在更早的触发点介入，而不是在等价帮助已激活之后打断）；不得把每次 AI 使用都算机会；必须省下大量时间/防重大错误/解锁任务；**不得仅凭沉默推断偏好**；`evidence_observation_ids` 至少 3 个**互不相同**的 id；禁止「修一个 SSH 报错/改一条命令/标一个 typo」这类窄机会；证据可以分布在时间上，不接受一串几乎相同的相邻屏。
2. *Prompt to Ground Missed Support To Observation* —— 把已验证机会落到具体触发点：从 `candidate_no_intervention=true` 的观察里选**最早的、需求已可推断且值得打断**的时刻；置信度 ≥0.75 仅当触发点证据充分；rationale 不得把未来行为写成当时可见。
3. *Self-reflection Prompt to Produce Revised Observation* —— 极性被用户行为推翻后改写标注：`need_support=yes` 则澄清具体需求；`=no` 则删除无支撑的挣扎/错误/低效表述；**禁止在输出中提及 label、feedback 或修正过程**；保留可记忆的具体细节（应用/文件名/命令/报错/工件/工作流状态），避免偶然或敏感细节；输出仅 `observation` + `user_intent`（<15 词）。

得到 $\mathcal{D}=\{(s_{t},h_{t-1},o_{t}^{\prime})\}$（alttext 提取自 §6.2/附录 B.3），用于更新 $\mathcal{O}$。

**两种低资源适配策略【正文 §3.3】**：

- **Coco-LoRA**：$\mathcal{D}$ 定期 shuffle 后更新多模态模型的 LoRA adapter。
- **Coco-PE**（following ACE, ref [18]）：维护**紧凑 bullet-point instruction memory**，按 Generate–Reflect–Curate 迭代。每个 batch 用当前 memory 预测，错例 + **时间相邻的正确例**（对比证据）送 reflection，再 curation 提炼行为规则更新 prompt。

### 3.5 干预决策模型的完整链路

```
截图流 → [MSE 选帧 + 动作前移] → <screenshots> + <memory>+<recent_*> 块
      → 多模态 LLM（单次 prompt）→ o_t = {observation, user_intent, rationale, y_t}
      → y_t=1:  Execution Agent（初始上下文 h_t，可多轮检索 M）
      → 用户反应 r_t / 60s 内主动求助 / 回溯扫描 → 标签
      → Coco-PE（改 prompt memory）或 Coco-LoRA（改 adapter）→ 更新 O
```

### 3.6 on-device 策略【正文 §4 + 附录 D】

1. **Change-aware frame selection**：先把键鼠动作与每帧对齐；保留首帧与末帧，中间帧仅当其与最近保留帧的 **RGB 均方误差 > τ** 才保留；被丢弃帧的动作前移到下一保留帧（保语义、省视觉 token）。DGX Spark 重放负载上把 **KV cache 命中率从 26.7% 提升到 45.2%**【正文 §4】（附录 D 给出双配置：RTX 5090 27.1%→45.2%，DGX Spark 26.7%→45.2%，均 >1.6×，TTFT 改善 >1.7×；重放实验 **τ=8,000**，数据合成的关键帧提取 **τ=2,000**【附录 C、D】）。
2. **Observation-first scheduling**：观察与用户可见推理是前台负载；个性化在后台机会式执行——**用户空闲 1 分钟后**才允许轻量 revision 作业，**更长空闲或显式 sleep mode** 才做全量个性化；个性化是**有界、带 checkpoint、低优先级**进程，交互推理一来即被抢占，之后从最近 checkpoint 续跑。分负载还带来 cache 局部性：连续 Observer 请求共享稳定 system prompt 前缀【正文 §4】。
3. **投机解码**：block diffusion prediction（ref [19]）；附录 D 具体为 Qwen3.5 **DFlash，draft block size = 8**，并用 **radix prefix cache（16-token 页粒度）**，在共享前缀边界 checkpoint Qwen3.5 recurrent state 与 DFlash draft state【附录 D】。
4. **引擎**：**CTI**（`github.com/mit-han-lab/cti-engine`，基于 Mini-SGLang），单 GPU 同实例既推理又微调，避免 vLLM/SGLang 那样维护两套系统【附录 D】。

---

## 4. 实验精读

### 4.1 OSWorld-Persona 构造细节【正文 §5.1 + 附录 C】

五阶段管线（Figure 3）：

1. **收集录屏/轨迹**：源数据 **OSWorld 2.0**（ref [20]），**108 个**长时程任务，覆盖个人与职业场景。
2. **关键帧提取**：沿用 §4 的选帧策略，**τ = 2,000**。
3. **构造输入实例**：把关键帧按**滚动上下文窗口（最多 5 帧）**与采样到的 persona 配对。
4. **两阶段标注**：标注 VLM = **Kimi-K2.6**（脚注 5 给出链接）。第一阶段 persona 无关地生成 observation + user intent；第二阶段做 persona 条件的二值 `need_support` 决策。
5. **按时间切分**：**每个 persona 按时间顺序 70% train / 30% test**（附录 C），避免泄漏。

**质量控制**：self-consistency 在 **temperature 0.4** 下每个决策阶段**再跑 2 次**，只有 2 次都复现原标签才保留；再做**人工校验**——人工复核 `need_support=true` 的案例并标注支持主动协助的上下文，找不到上下文就**删除该样本**【正文 §5.1、附录 C】。

**规模**：**6,558 个关键帧、3,279 条主动协助标注、108 个会话**【正文 §5.1】。

**Table 4 统计（附录 C）**——列 = 示例总数 / Support: Yes / Support: No：

| Persona | 简述 | Split | 总数 | Yes | No |
|---|---|---|---|---|---|
| Automation First | Offloads repetitive/multi-step work to agents; hands-off for trivial one-off actions | Train | 521 | 202 | 319 |
| | | Test | 239 | 100 | 139 |
| Eager Hands-on | Delegates whole sub-tasks and reviews results; prefers doing over explaining | Train | 509 | 390 | 119 |
| | | Test | 237 | 192 | 45 |
| Learning-oriented | Prefers guidance and explanation over silent takeover; welcomes nudges when stuck | Train | 660 | 37 | 623 |
| | | Test | 285 | 10 | 275 |
| Reserved Drafts | Prefers AI-generated drafts or partial artifacts to review and edit independently | Train | 589 | 60 | 529 |
| | | Test | 239 | 30 | 209 |
| **Total** | | Train | **2,279** | 689 | 1,590 |
| | | Test | **1,000** | 332 | 668 |

（正负比本身即 persona 的可分性证据：Eager test 正例 192/237 ≈ 81%，Learning-oriented 仅 10/285 ≈ 3.5%。2,279+1,000 = 3,279 与 §5.1 的标注总数吻合。）

**两阶段标注 prompt（附录 C）**：
- Stage 1 *Perceive*：输入 Screen History（带 `[t=mm:ss] Screenshot i of N` 时间戳）+ 可选 Recent Observation；输出 `{"observation": "<2--4 sentences>", "user_intent": "..."}`；明确「只描述屏幕，不决定是否帮忙」。
- Stage 2 *Decide*：输入 User Preference Memory + **Available Agent Profile** + observation + intent + 可选 Recent Observation；要求保守（收益过线才 `yes`，稳步推进/动作琐碎/打断不值当则 `no`）；输出 `{"need_support","rationale","suggestion_type","suggestion"}`，其中 `suggestion_type ∈ {none, direct_message, prompt_to_agent}`。

### 4.2 评测设置【正文 §5.2】

- **三个 backbone（本地可部署消费级硬件/AI PC）**：① Qwen-3.5-9B（dense，9B 激活）；② Qwen-3.5-35B-A3B（MoE，3B 激活）；③ Nemotron-3-Nano-Omni（MoE omni，3B 激活）。
- **基线**：Vanilla（直接二值决策提示）、Induced Persona（从训练数据归纳紧凑画像，ref [21]）、Oracle Persona（注入合成用的真值 persona，**部署中不可得**）。上下文管理对照：仅用模型 context、**Mem0**（ref [22]）。Coco 与 Mem0 均用 **Qwen-3.5-9B 构建记忆**，answer model 决定检索查询，**检索调用上限 5 次**。
- **指标**（following Lu et al. [8]）：Recall、Precision、Accuracy、**False-Alarm**（在无需求时给出干预的比例）、F1。记忆质量用 LoCoMo / LongMemEval / BEAM-100K / BEAM-1M，**LLM-as-a-judge Accuracy，judge = gpt-5.5**，沿用 Mem0 的 judging protocol（脚注 3 指向 `mem0ai/memory-benchmarks`）。
- **Table 1 是三个随机种子的平均**（Table 1 图注）。

### 4.3 主结果：Table 1 完整数字（§6.1，macro 与 per-persona F1）

列：Precision / Recall / Acc / FA / F1（macro average），随后 4 个 persona 的 F1（Automation First / Eager Hands-on / Learning-oriented / Reserved Draft）。

| Model | Method | P | R | Acc | FA | **F1** | AF | EH | LO | RD |
|---|---|---|---|---|---|---|---|---|---|---|
| Qwen-3.5-9B | Vanilla | 0.4231 | 0.1739 | 0.6498 | 0.0468 | **0.1861** | 0.0371 | 0.1387 | 0.3919 | 0.1765 |
| | Induced Persona | 0.3729 | 0.2221 | 0.6380 | 0.1055 | 0.2094 | 0.0785 | 0.1993 | 0.2583 | 0.3014 |
| | Oracle Persona | 0.4758 | 0.4345 | 0.7295 | 0.1599 | 0.3932 | 0.2096 | 0.7291 | 0.3099 | 0.3243 |
| | **Coco-PE** | 0.4845 | 0.4933 | 0.7759 | 0.2492 | **0.4832** | 0.5233 | 0.8088 | 0.3888 | 0.2121 |
| | Coco-LoRA | 0.5052 | 0.4334 | 0.7753 | 0.2343 | 0.4551 | 0.3126 | 0.8408 | 0.4264 | 0.2404 |
| Qwen-3.5-35B-A3B | Vanilla | 0.3884 | 0.1863 | 0.6350 | 0.0857 | **0.1827** | 0.1020 | 0.1507 | 0.1900 | 0.2878 |
| | Induced Persona | 0.4373 | 0.2886 | 0.6490 | 0.0864 | 0.2408 | 0.1184 | 0.2158 | 0.3440 | 0.2850 |
| | Oracle Persona | 0.4448 | 0.3192 | 0.6527 | 0.1188 | 0.2713 | 0.2671 | 0.3810 | 0.1769 | 0.2602 |
| | **Coco-PE** | 0.5678 | 0.3785 | 0.7846 | 0.1298 | **0.4334** | 0.4514 | 0.7369 | 0.2603 | 0.2850 |
| | Coco-LoRA | 0.4087 | 0.3335 | 0.6784 | 0.1697 | 0.3135 | 0.1911 | 0.5385 | 0.2563 | 0.2682 |
| Nemotron-3-Nano-Omni | Vanilla | 0.4050 | 0.0815 | 0.6485 | 0.0359 | **0.1175** | 0.0362 | 0.0951 | 0.1525 | 0.1863 |
| | Induced Persona | 0.4260 | 0.1965 | 0.6494 | 0.0714 | 0.2192 | 0.0657 | 0.1991 | 0.3043 | 0.3075 |
| | Oracle Persona | 0.4222 | 0.2218 | 0.6927 | 0.1285 | 0.2810 | 0.1721 | 0.5247 | 0.1515 | 0.2759 |
| | Coco-PE | 0.4784 | 0.3025 | 0.6922 | 0.1233 | 0.3211 | 0.3229 | 0.4737 | 0.1928 | 0.2949 |
| | **Coco-LoRA** | 0.4923 | 0.3152 | 0.7857 | 0.2237 | **0.3485** | 0.1771 | 0.8609 | 0.1635 | 0.1923 |

**正文解读（§6.1）**：
- Vanilla macro recall 仅 **0.082–0.186**、F1 < 0.19（与摘要 0.12–0.19 一致）。
- Induced Persona F1 **0.209–0.241**；Oracle Persona **0.271–0.393**。
- 最佳适配 macro F1：**Qwen-3.5-9B 0.483（2.6×）**、**Qwen-3.5-35B-A3B 0.433（2.4×）**、**Nemotron-3-Nano-Omni 0.349（3.0×）**，**均超过对应 Oracle Persona**。
- 增益不只是「干预更多」：precision 与 accuracy 同时提升；FA 略高于 vanilla（从近乎沉默转向主动干预）。
- 方法偏好：**Qwen 上 Coco-PE 更强，Nemotron 上 Coco-LoRA 更强**。

> ⚠️ **数字待澄清**：引言写「reaches up to 0.77 F1 on one of the personas」（§1），但 Table 1 中单 persona 最高 F1 为 **0.8609**（Nemotron / Coco-LoRA / Eager Hands-on），其次 0.8408、0.8088、0.7369；**未在全文中找到 0.77 对应的单元格**（可能是笔误或另一种聚合口径）。

### 4.4 消融：Table 2（Observer formulation，§6.1）

| Method | Variant | Qwen-3.5-9B | Qwen-3.5-35B-A3B | Nemotron-3-Nano-Omni |
|---|---|---|---|---|
| Coco-PE | Full | **0.4832** | **0.4334** | **0.3211** |
| | No Context（去掉 $h_t$） | 0.3878 | 0.3983 | 0.3040 |
| | Decision Only（只输出决策） | 0.3045 | 0.1993 | 0.2529 |
| Coco-LoRA | Full | **0.4551** | **0.3135** | **0.3485** |
| | No Context | 0.2851 | 0.2951 | 0.3247 |
| | Decision Only | 0.4286 | 0.3204 | 0.2450 |

结论（正文原话口径）：Coco-PE 下去掉上下文，F1 0.483→0.388（9B）、0.433→0.398（35B）、0.3211→0.3040（Nemotron）；Coco-LoRA 下 9B 掉得最狠 0.455→0.285。**累积上下文对个性化是必要的，不只是 prompting 假象**；同时 Decision Only 对比说明「输出 observation 本身」既积累上下文也帮助决策。

### 4.5 记忆基准：Table 3（§6.1）

| Answer Model | Memory | LoCoMo | LongMemEval | BEAM-100K | BEAM-1M |
|---|---|---|---|---|---|
| GPT-5.5 | 无外挂 | 0.9364 | **0.9440** | 0.5811 | 0.5121 |
| | Mem0 | 0.7734 | 0.7220 | 0.6750 | 0.6257 |
| | **Coco** | **0.9513** | 0.8220 | **0.7500** | **0.6786** |
| Claude-Sonnet-4.6 | 无外挂 | 0.9162 | **0.9040** | 0.5394 | 0.3840 |
| | Mem0 | 0.7240 | 0.7180 | 0.7275 | 0.7171 |
| | **Coco** | **0.9448** | 0.7520 | **0.8375** | **0.8329** |
| Kimi-K3 | 无外挂 | 0.9227 | **0.9200** | 0.6010 | 0.3951 |
| | Mem0 | 0.7955 | 0.7260 | 0.5966 | 0.5799 |
| | **Coco** | **0.9565** | **0.8340** | **0.7194** | **0.7253** |

要点：Coco 在**每个基准上都超过 Mem0**，并在 LoCoMo、BEAM-100K、BEAM-1M 取得最好成绩；长上下文优势在 BEAM 最明显（Claude Sonnet 4.6 上 BEAM-100K 0.539→0.838，BEAM-1M 0.384→0.833；GPT-5.5 上 BEAM-1M 0.512→0.679；Kimi-K3 上 0.395→0.725）。**LongMemEval 上 vanilla 长上下文模型最好**（对话能塞进上下文窗口），但记忆变长后就不行了。

### 4.6 一周用户研究（§6.2）

- **设计**：10 名参与者在**一周**内把 Coco 用于日常电脑活动；Observer 跑 **Qwen3.5-9B on 单张 RTX 5090**；适配方法 = **Coco-PE**（按 benchmark 该 backbone 最优选择）。向被试解释主动功能并给**极简指令**（只教 thumb-up/thumb-down 与通过文字/音频求助），其余照常使用。
- **数据边界（隐私设计的核心）**：Observer 输入、输出与整理后的记忆**全部留在本机**；知情同意下只收集三类交互事件：① 对建议 thumb-up → 计为 **TP**；② thumb-down 或「未展开完整内容就关闭」→ 计为 **FP**；③ Coco 沉默时的文字/音频直接请求 → 计为 **FN 代理**（脚注 4 承认 FN 可能被低估，因为用户可能想要帮助却没开口）。
- **数字**：macro-averaged F1 proxy 从 **0.199 → 0.515**；**Day 1 有 6/10 用户 F1 proxy 为 0**（冷启动，Coco 没给出有用建议），适配后这 6 人都开始收到自评有帮助的建议。
- **定性（作者之一的一周部署，Figure 4c）**：Day 2 学到粗线索（屏上出现「TODO」≈ 可以帮忙，技术输出≈保持沉默）；Day 5 细化为「正在填写 TODO 才帮，滚动/通读全文档则沉默」；Day 7 进一步区分「填生成性空缺（欢迎协助）」vs「整合已验证内容（用户自己来）」——体现**情境化偏好**而非静态画像。
- **未在全文中找到**：被试人口学（年龄/职业/技术背景）、分日逐人 F1 数值表、显著性检验【未找到】。

### 4.7 on-device 剖析（附录 D，Table 5）

- **Replay workload**：录制 1 位作者 **30 分钟**电脑使用，按 **15 秒窗口**聚合动作（动作 <2 个的窗口并入下一个），共 **80 个 Observer 请求**；不含累积上下文与已学偏好；比较 τ=8,000 的 change-aware 选帧 vs 全帧基线 → KV 命中 27.1%→45.2%（RTX 5090 / Qwen3.5-9B）、26.7%→45.2%（DGX Spark / Qwen3.5-35B-A3B-FP8），TTFT 改善 >1.7×。
- **Real single-user workload**（含 $h_t$ 与 Coco-PE 学到的偏好）：

| Model | Device | Latency mean/P50/P95 (s) | TTFT mean/P50/P95 (s) | KV hit mean/P50/P95 (%) | MM Proc. hit mean/P50/P95 (%) |
|---|---|---|---|---|---|
| Qwen-3.5-9B | RTX 5090 | 2.84 / 2.42 / 6.97 | 1.22 / 0.94 / 3.68 | 29.23 / 39.74 / 55.97 | 17.57 / 20.00 / 50.00 |
| Qwen-3.5-35B-A3B-FP8 | DGX Spark | 7.52 / 6.63 / 18.68 | 2.83 / 1.70 / 10.49 | 27.92 / 40.50 / 55.87 | 26.29 / 25.00 / 75.00 |

- 结论口径：Observer 能在**多秒窗口**上工作（而非逐个低层动作响应），因此 1.2–2.8s 的平均 TTFT 与 2.8–7.5s 的端到端延迟「与交互循环兼容」。

---

## 5. 附录 / 细节挖掘

### 5.1 prompt 全清单（附录 B）

| 位置 | Prompt | 关键约束/字段 |
|---|---|---|
| B.1 | Observer Agent Prompt | 5 个输入块；四职责；`need_support: yes` 仅当收益明显大于打断成本；JSON 四字段 |
| B.2 | Proposition Synthesis | `confidence 1–10`、`decay 1–10`、必须 cite observation_ids、`max_propositions` |
| B.2 | New Proposition Classification | `IDENTICAL\|SIMILAR\|UNRELATED` + `target_ids` |
| B.2 | Proposition Consolidation | 重写/合并/拆分为 ≤ `max_propositions` 条 |
| B.3 | Discover Missed Support Opportunities | 3 类机会；≥3 个唯一证据 id；禁止窄机会；「Never infer a preference merely from silence」 |
| B.3 | Ground Missed Support To Observation | 从 `candidate_no_intervention` 中选最早可推断触发点；confidence ≥0.75 门槛 |
| B.3 | Self-reflection to Produce Revised Observation | 只回 `observation` + `user_intent`(<15词)；禁止提及 label/feedback |
| B.4 | Reflector | 输出 `reflection` / `helpful_bullet_ids` / `harmful_bullet_ids` / `proposed_insights`；section ∈ `when_to_support \| when_to_stay_silent \| how_to_support \| general`；**每次最多 2 条 insight**；insight 必须是「情境 X 下该用户想要/不想要 Y 因为 Z」，不得提及样例 id |
| B.4 | Curator | 增量 delta ops：`add / update / delete`；bullet 要原子、具体、可操作（「when X, do/don't Y」）；每条一个决策规则；禁止近重复；**最多 `{max_ops}` 次操作** |
| B.4 | Final Organization | 把 case 级 memory 转成「用户意图 + 协助偏好」的统一 insight；**不要把反复出现的应用/领域/任务当成偏好**；按 bullet id 引用例子、不得发明 id（用户研究期间用于生成可读的可审阅版本） |

### 5.2 超参（附录 E）

**Coco-PE**：每个 persona 从空 textual memory 起步；**batch = 8**，每例**最多 8 张截图**；错例全送 + **最多 2 个时间相邻的正确例**给 reflector；curator **最多 8 次 add/update/delete**；reflection/curation **temperature = 0.2**，输出上限 **8,192 / 4,096 tokens**；memory 上限 **50 条 bullet**（超出删低效条目）；**epoch = 1**。

**Coco-LoRA**：每 backbone 每 persona 独立 adapter，用 **LLaMA-Factory**；**LoRA rank = 32、α = 64、dropout = 0**；**1 epoch、BF16、batch size 8、cosine + 10% warmup**；学习率 **1e-5**（Qwen3.5-9B / 35B-A3B）、**5e-5**（Nemotron-3-Nano-Omni），**thinking 关闭**；**冻结 vision encoder 与 multimodal projector**，LoRA 施加于语言模型所有可用线性层（除 output head）；MoE 骨干额外挂在 expert 与 router/gating 投影上（若暴露为线性层）。

### 5.3 限制（附录 A，四条）

1. **用户控制**：训练信号大多隐式推断；被试 appreciate「不用频繁显式反馈」，但也**想要看见系统学到了什么、参与学习过程**；未来方向 = 轻量解释「为什么给这个建议」、可编辑的偏好表示、可调范围/节奏的控件。
2. **增强而非替代用户**：糟糕的主动协助可能让用户「用理解换便利」；正面案例——Coco 建议她学习新材料时出测验题，她反复采用，Coco 随后几天校准得更好；呼吁评估标准除「建议被采纳」外还要衡量**学习、能动性、持续收益**。
3. **隐私**：架构把观察限制在自包含 on-device 组件，**但 Execution Agent 目前依赖云端大模型，检索出的用户上下文可能离开设备**——「reduces but does not eliminate privacy risks」。
4. **能耗**：always-on 观察/推理/适配有常规助手没有的计算与能耗成本，**本文未系统测量**；未来做 energy-latency-quality 权衡、事件触发感知、自适应观察频率、模型压缩。

### 5.4 界面与控制（附录 F）

- 悬浮 avatar（可移动/隐藏）；**渐进披露**：机会出现时只显示简短描述 + 「Help me with this」按钮，**点开才展示完整建议**；不点 = 负奖励，用户研究里也计为 FP；自动消失。
- 三入口：点 avatar、`Cmd+Shift+Space`（附截图）、唤醒词「Hey Coco」；文字/语音/屏幕接地请求共用会话历史；**Coco 沉默时主动请求 = 该预测的负奖励，研究中计 FN**。
- Activity view（回看观察与建议）与 Personalization view（学习阶段、进度、最近 checkpoint、纳入的反馈数；每条偏好可 keep/edit/reject，**修订需用户批准后才生效**）。**用户研究期间偏好编辑被禁用**，以评估纯自进化产生的偏好。

### 5.5 其它细节

- AI Use Statement：LLM 参与代码开发与数据合成标注，**不用于**研究构思、结果解读、实验设计或性能 claim；写作期间也用 Coco 本身辅助。
- Ethics：明确「观察越多→个性化越好 vs 敏感信息暴露越多」的张力；on-device 是**原则性隐私保障**而非工程优化；残余风险 = 强化不良模式/过拟合短暂行为、以及被滥用为监控机制——要求**显式、知情、可撤销的同意**，清楚披露观察什么、保留什么、如何使用。
- 数据政策：**不发布任何用户活动数据集**；只发合成管线。

---

## 6. 代码研读

### 6.1 仓库 A：`deep-personalization`（主仓，Apache-2.0）

```
deep-personalization/
├── README.md                     # uv sync；HF 下载数据；五方法表；Prepare→Predict→Score 三段管线
├── pyproject.toml                # Python>=3.11；依赖 huggingface-hub/numpy/openai/opencv-headless/tinker；
│                                 # 入口 osworld-persona、osworld-persona-synthesize
├── benchmarks/osworld_persona/
│   ├── README.md                 # 每个方法的完整复现命令（LLaMA-Factory 与 Tinker 双后端）
│   ├── cli.py (42KB)             # validate/export/run/score/induce-persona/evolve-coco-pe/
│   │                             # prepare-coco-lora/train-coco-lora/eval-coco-lora
│   ├── prompts.py                # RECENT_OBSERVATION_LIMIT=5；ORACLE/INDUCED profile block；
│   │                             # LEARNED_MEMORY_BLOCK；reflector/curator system+template；
│   │                             # observer_prompt()/render_user_prompt()/apply_condition()
│   ├── metrics.py                # parse_decision、_binary_metrics、compute_metrics（macro persona 平均）
│   ├── scoring.py / run.py / data.py / schema.py / method_adapter.py
│   ├── assets/observer_prompt.txt      # benchmark 用的共享 Observer system prompt（69 行）
│   ├── assets/oracle_personas.json     # 4 个 persona 的真值画像
│   └── synthesis/                # 五阶段数据合成管线
│       ├── frames.py             # DEFAULT_VIDEO_MSE_THRESHOLD=8000.0、DEFAULT_TRAJECTORY_MSE_THRESHOLD=2000.0
│       │                         # detect_key_frame_times()、key_frames_from_trajectory()（MSE 去重）
│       ├── build.py              # BuildConfig: window_size=5, stride=5, min_window=2, max_windows=8,
│       │                         # recent_observations=2；frame_windows()；balance_records()
│       ├── pipeline.py           # perceive_temperature=0.4、label_temperature=0.4、max_tokens=10240
│       ├── consistency.py        # self-consistency（min_agree，复跑比较 need_support）
│       ├── profiles.py           # 5 个 USER_PROFILES + 5 个 TOOL_PROFILES（见 6.4 差异）
│       ├── prompts.py            # 两阶段标注 prompt（Perceive / Decide）
│       ├── review.py / split.py / cli.py（--mse-threshold 默认 2000.0，--test-ratio 默认 0.2）
├── methods/
│   ├── prompting.py              # Vanilla / Oracle persona
│   ├── induced_persona.py        # 归纳画像（digestion → profile 两段 prompt）
│   ├── coco_pe.py                # PromptMemory/MemoryBullet、CocoPETrainer（见 6.3）
│   └── coco_lora.py              # CocoLoRAResult/CocoLoRATrainer/_revise（revision_batch_size=32）
├── inference_backends/           # openai_compatible.py、llamafactory_server.py、tinker.py
├── training_backends/            # llamafactory.py、tinker.py、configs/qwen3_5_lora.yaml
└── tests/                        # 5 个测试文件（见 §9 实测）
```

### 6.2 仓库 B：`coco`（collaborative-agents/coco，Apache-2.0）

```
coco/
├── README.md          # 双 agent 架构：Always-on Observer Agent + Execution Agent（基于 pi）
│                      # 一键本地部署（CTI，RTX 5090 / RTX PRO 6000 / DGX Spark，Linux + driver≥580）
├── lib/sensing/       # 屏幕感知与 Observer 运行时
│   ├── sensing/observer.py                    # Observer 抽象基类（改自 GUM 的 observer.py）
│   ├── sensing/streamer.py                    # calc_diff_scores()（MSE）、segment_threshold=8000.0
│   ├── sensing/screen.py (56KB) / segment_processor.py (57KB) / progress_detector.py (38KB)
│   ├── sensing/support_feedback.py             # 支持反馈事件
│   └── sensing/prompts_everyday/observer.txt   # 部署版 Observer prompt（102 行，输出 10 字段）
├── lib/personalization/                        # ← 论文方法的运行时实现
│   ├── README.md                               # CTI docker 命令 + CLI 用法
│   ├── personalization/signals/user_feedback.py      # FEEDBACK_POLICIES + 30min TTL
│   ├── personalization/signals/missed_opportunities.py # 60s 窗口的「沉默后主动求助」
│   ├── personalization/signals/retrospective.py (66KB) # 三段式回溯（discovery/verification/grounding）
│   ├── personalization/labeling.py             # label_moment()：加权打分 → 二值标签
│   ├── personalization/memory/evolve.py        # SelfEvolvingLearner、EvolveConfig、resume checkpoint
│   ├── personalization/memory/selection.py     # select_evolution_moments()：错例全保留 + 相邻正确锚点
│   └── personalization/cli.py                  # summary/label/revise-labels/self-evolve/build-dataset
├── lib/memory/        # GUM 风格长期记忆（engine/store/context_groups）
├── lib/external_api/  # 各模型后端（litellm、tinker、nv_inference、lm_studio…）
├── desktop/           # Electron 桌面端（avatar、气泡、PersonalizationCenter、通知）
│   └── src/main/services/personalization-scheduler.ts  # 空闲调度 + 抢占 + checkpoint
└── desktop/src/main/services/evening-personalization-scheduler.ts
```

### 6.3 关键函数与论文对应关系

| 论文概念 | 代码位置（文件:行/函数） | 说明 |
|---|---|---|
| 式(2)(3) 的 $\mathcal{O}$ 与 Retrieve | `coco/lib/sensing/sensing/observer.py`（抽象基类）+ `prompts_everyday/observer.txt` | 运行时 Observer；GUM 衍生 |
| 四字段结构化输出 | `deep-personalization/benchmarks/osworld_persona/assets/observer_prompt.txt` | benchmark 版 schema：observation/user_intent/rationale/need_support |
| 输入块（memory/screenshots/recent_*） | `coco/.../prompts_everyday/observer.txt:7-39` | 部署版还多 `<user_name>`、hotkey 截图、`<conversation_history>`、`<user_input>` |
| 隐式信号 → 短窗信号 | `signals/user_feedback.py:FEEDBACK_POLICIES` | abstain/dismiss = negative(conf 1.0, scope task, weight 1.2)；thumbs_up/down = ±0.9 session 1.0；engage→reveal 正向 0.8 observation；TTL 30min |
| 「沉默后主动求助」= FN 代理 | `signals/missed_opportunities.py:derive_missed_opportunity_signals()` | **60s 窗口**（`MISSED_OPPORTUNITY_WINDOW_S=60`），要求最终决策为 no 且 60s 内出现 `trigger=="user_prompt"` |
| 标签合成 | `labeling.py:label_moment()` | `score = Σ signed_score()`；**\|score\| < 0.45 直接丢弃不打标签**；>0 → yes，<0 → no；confidence = \|score\|/Σ\|conf×weight\| |
| 弱负例 | `labeling.py:UNVERIFIED_NO_SUPPORT_SOURCE` | 未经纠正的 no-support 预测按 confidence 0.25 计入（可用 `--no-include-unverified-no-support` 排除） |
| 回溯扫描（B.3 三段式） | `signals/retrospective.py:DISCOVERY_SYSTEM_PROMPT / _EVIDENCE_VERIFICATION_SYSTEM_PROMPT / _TRIGGER_GROUNDING_SYSTEM_PROMPT`；类别 `RETROSPECTIVE_CATEGORIES = {repetitive_work, stuck, anticipate_need}` | 与论文 B.3 完全对应；`_DEFAULT_MAX_INPUT_CHARS=48_000`、`_MODEL_MAX_OUTPUT_TOKENS=8192`、JSON 重试 3 次、可断点续跑 |
| Coco-PE 训练循环 | `methods/coco_pe.py:CocoPETrainer` | 默认 `batch_size=8, max_bullets=50, max_operations=8, reflect_correct=2, reflection_max_tokens=8192, curator_max_tokens=4096, seed=42` —— **与附录 E 完全一致** |
| 样例选择（错例优先） | `coco/.../memory/selection.py:select_evolution_moments()` | 全部 disagreements + unknown 保留；与错例同会话相邻的正确例必留（锚点）；其余按 `correct_sample_rate=0.5`、`seed=42` 确定性采样 |
| Coco-LoRA 修订监督 | `methods/coco_lora.py:CocoLoRAResult/_revise()` | `revision_batch_size=32`；四字段 target schema（`parse_complete_decision` 强制四字段齐全） |
| change-aware frame selection | `coco/.../streamer.py`（`calc_diff_scores`、`segment_threshold=8000.0`）；`deep-personalization/.../frames.py`（`_mse()`、阈值 8000/2000） | 论文 τ=8000（部署剖析）/ 2000（数据合成） |
| observation-first scheduling | `desktop/src/main/services/personalization-scheduler.ts` | `revisionIdleSeconds ?? 60`、`evolveIdleSeconds ?? 10*60`、失败冷却 revision 5min / evolve 15min、`signals_checkpoint.json` + `evolve_checkpoint.json`、sleep mode 可暂停——**与论文「1 分钟轻量、更长空闲全量、抢占+checkpoint」逐条对应** |
| 记忆检索 k | `benchmarks/.../prompts.py:RECENT_OBSERVATION_LIMIT=5`（`accumulated_memory()` 默认 limit=5） | **benchmark 侧的 k=5**；论文正文未给 k 值 |

### 6.4 代码与论文的差异（重要）

1. **persona 池是 5 个，论文只用 4 个**：`synthesis/profiles.py` 的 `USER_PROFILES` 含 `minimal_interruptions`（强打断厌恶型），而 README/论文的 OSWorld-Persona 是 `automation_first, eager_handson, learning_oriented, reserved_drafts` 四个；合成 CLI 示例也只传这 4 个。第 5 个画像可复用但不在已发布数据集中【代码】。
2. **切分比例默认值不一致**：论文附录 C 写 **70/30**，`synthesis/cli.py` 的 `--test-ratio` **默认 0.2**（README 示例也用 0.2）。发布数据集应是按 0.3 构建的（Table 4 test=1,000 ≈ 30%），但仓库默认值不是 0.3【代码 vs 附录 C】。
3. **三个 Observer prompt 版本**：
   - 论文附录 B.1（5 个输入块、四职责、四字段 JSON）；
   - benchmark `assets/observer_prompt.txt`（69 行，正文按「Previous Recent Memory / New Screen History」两类上下文组织，标签 `<previous_recent_memory>`/`<screenshots>` 由 `prompts.py:render_user_prompt()` 拼出；明确「不要产出完整视觉描述」）；
   - 部署版 `prompts_everyday/observer.txt`（10 字段输出：`current_task/task_continuity/repeats_dismissed_support/observation/user_intent/inefficiency_patterns/mistake_made_by_human/task_complete/applying_ai_output/status`，且要求识别 Coco 自身 UI、忽略之；状态标签 `progress|mistake|inefficient|stuck|observing`）。
   → 论文描述的是「研究形态」，桌面端实际跑的是更丰富的形态【代码 vs 附录 B.1】。
4. **Table 3 的记忆基准复现代码不在仓库**（两仓均无 LoCoMo/LongMemEval/BEAM 脚本）【未找到】。
5. **on-device 运行时不在主仓**：CTI 集成、DFlash、radix cache 都在外部 `mit-han-lab/cti-engine`；coco 仓只给 docker 命令与 MSE 分段/调度实现【代码】。

### 6.5 README 自述的复现方式

**主仓（deep-personalization）**：
1. `uv sync`（Python 3.11+，Tinker 推理/训练也需要）。
2. `uv run hf download collaborative-agents/OSWorld-Persona --repo-type dataset --local-dir data/osworld-persona`。
3. 起 LLaMA-Factory API（`llamafactory-cli api examples/inference/qwen3vl.yaml ...`）或用 Tinker 后端。
4. 三段管线 **Prepare → Predict → Score**，CLI 子命令：`validate / export / run / score / induce-persona / evolve-coco-pe / prepare-coco-lora / train-coco-lora / eval-coco-lora`；`--resume` 可续跑 Coco-PE checkpoint；Coco-LoRA 训练用仓库自带 `training_backends/configs/qwen3_5_lora.yaml`（`gradient_accumulation_steps: 8`、cosine、warmup 0.1、bf16），逐 persona 起停 server 评测后用 `jq` 合并 + `score` 汇总。
5. 数据合成：`uv run osworld-persona-synthesize trajectory|video ... --user-profiles ... --tool-profiles ...`，再 `consistency`（`--samples 3`）与 `split`。
6. 「新增方法」四步协议：实现于 `methods/`（依 `methods/types.py` 与共享 predictor/trainer 协议）→ benchmark 专用 prompt 放 `benchmarks/<name>/method_adapter.py` → 暴露到 benchmark CLI → 需要时加 backend adapter。

**Coco 仓**：
1. 发布版直接装（macOS Apple Silicon/Intel、Windows）；源码运行需 `uv sync` + Node.js，然后 `cd desktop && npm install && npm start`。
2. 本地模型一条命令（CTI docker，Qwen3.5-9B / 35B-A3B / 35B-A3B-FP8 / Qwen3.8-27B-NVFP4，配 DFlash 投机模型），在 Settings→Model 指定为 screen understanding model。
3. CLI 个性化：`uv run coco-personalization summary|label|revise-labels|self-evolve|build-dataset`（`--retrospective-model` 开回溯标注，`--resume` 续跑，草稿需批准才进正式 prompt）；`COLLECT_TRAINING_SCREENSHOTS=1` 才保留截图，默认演化后删除。
4. 隐私自述：**无 telemetry/analytics**；设置与活动历史在本机；**屏幕理解会把截图发给你选择的模型提供方——用本地模型才能全程留在本机**（README 明写）。

### 6.6 可借鉴的代码级要点

- **`label_moment()` 的加权投票 + 阈值丢弃**：polarity/confidence/scope/label_weight 四元组策略表 + `|score|<0.45` 不打标签，天然解决「信号稀疏且互相矛盾」的标注问题。
- **`select_evolution_moments()` 的对比采样**：错例全留 + 同会话相邻正确例必留 + 其余降采样，是「用最小样本做反思式适配」的现成实现。
- **`retrospective.py` 的三段式回溯 + 断点续跑 + 输入截断（48k chars）**：把昂贵的 LLM 回溯做成可恢复、可审计（trace 快照）的批处理。
- **`personalization-scheduler.ts` 的空闲分层（60s/600s）+ 冷却（5min/15min）+ 双 checkpoint**：前台永远不被个性化拖慢的最小可行方案。
- **`observer_prompt.txt` 的「compact incremental update + 不重复旧记忆」**：让上下文长度随信息增量而非随时间线性增长。
- **`metrics.py` 的 `strict_accuracy`（分母含 parse_failures）与 `parse_coverage`**：对「模型输出不可解析」这一常见失败模式给出可追踪指标，而不是静默丢样本。

---

## 7. 摘要级理解的修正

对照 `papers-abstracts.md` §1（= HTML 摘要）逐条列出**读全文后才知道/与初读理解不同**的点：

1. **「隐式信号」的完整清单比摘要多一层**：摘要只列 engagement / dismissal / explicit requests / repeated work patterns；正文 §3.3 还有一整套**回溯扫描**（同一 on-device 模型扫 $\{h_1..h_T\}$、`repetitive_work|stuck|anticipate_need` 三类、事后手动调用 AI 视为正面证据），并明确承认这是「机会证据」而非「确认偏好」，且是为**对抗「越不干预越没信号」的退化策略**而设。
2. **适配方法的具体名字与机制**：摘要只说「continual adaptation」；正文给出 **Coco-LoRA 与 Coco-PE（ACE 式 Generate-Reflect-Curate）**两条路，附录 E 给出全部超参（rank32/α64、batch8、50 bullets、temp0.2 等）。
3. **紧凑上下文的具体构造**：摘要说 compact long-term user modeling context；正文 §3.2 给出**命题-观察二部图 + GUM 合成 + top-k 时间/语义双检索**，附录 B.2 给出 confidence/decay 打分与三段式合并 prompt。
4. **k 未给值**：正文只写「k 条最近输出 + k 条最相关命题」，**具体 k 未在全文中找到**（benchmark 代码里是 5）。
5. **on-device 不只是「跑得动」**：摘要只说 efficient strategies；正文/附录 D 给出**双配置实测**（RTX 5090 TTFT 1.22s / DGX Spark 2.83s，KV 命中 29.23%/27.92%）、**CTI 引擎 + DFlash(block=8) + 16-token radix prefix cache**，以及调度细节（1 分钟轻量 revision、长空闲/sleep 才全量、抢占+checkpoint）。
6. **评测的质控细节**：摘要只说 persona-conditioned labels；正文给出 **Kimi-K2.6 两阶段标注 + temp 0.4 自洽（再跑 2 次全票）+ 人工复核正例（找不到上下文就删）+ 按时间 70/30 切分**，以及 6,558 帧 / 3,279 标注 / 108 会话的规模。
7. **基线里有 Oracle Persona 这个「上界」且被超越**：摘要没有；正文 §6.1 的关键卖点之一是**适配后超过 Oracle Persona**（0.483 vs 0.393 等）。
8. **FA 代价被正文坦承**：摘要不提；Table 1 显示适配后 False-Alarm 明显升高（如 9B Coco-PE 0.2492 vs Vanilla 0.0468），正文解释为「从近乎沉默转向主动干预」。
9. **用户研究的口径是「F1 proxy」而非真值 F1**：摘要写 macro-averaged F1 0.199→0.515；正文说明它由三类事件代理（thumb-up=TP、thumb-down/未展开=FP、沉默时主动求助=FN 代理），且**脚注承认 FN 被低估**——这是代理指标，不是人工标注真值。
10. **冷启动数字**：Day 1 有 6/10 用户 F1 proxy 为 0（摘要没有）。
11. **隐私边界比摘要更诚实**：Ethics/附录 A 明说 **Execution Agent 走云端，检索出的上下文可能离开设备**（reduces but does not eliminate）；README 也明写「屏幕理解会把截图发给模型提供方」。
12. **数字口径疑点**：引言「up to 0.77 F1 on one of the personas」与 Table 1 单 persona 最高 0.8609 对不上（未找到 0.77 的出处）。
13. **三 backbone 的形态**：摘要说 three model backbones，正文才说明是 1 dense + 2 MoE（其中 1 个 omni），且都强调「本地消费级硬件可部署」。
14. **代码是两个仓**：摘要/脚注只在正文脚注里区分——主仓管 benchmark+方法，coco 管桌面集成；两者 license 均为 Apache-2.0，论文本身是 arXiv perpetual non-exclusive license。

---

## 8. 对 OIA 的借鉴更新（具体到组件）

> OIA 现状前提（据 `omni-interaction-agent/README.md`）：DSH 基座 + OIA-Memory 五组件（Vault/Distiller/Canvas/Recall/Governance）+ 干预策略器 + UserModel 四层渲染管线，M0 隔离、M5 评测硬化；当前处于调研/设计阶段。

1. **Observer subagent（主动性的感知层）**
   - 直接采用**四字段输出 schema**（`observation/user_intent/rationale/need_support`）作为 OIA Observer subagent 的返回契约，理由：它同时是「记忆写入格式」与「决策格式」，Table 2 已实证 Decision Only 明显更差（0.483→0.3045）。
   - 输入块照抄 B.1 的五块划分（memory / screenshots / recent_observations / recent_propositions / relevant_propositions≤3），并坚持 **compact incremental update + 不重复旧记忆** 的写法，控制 token 增速。
   - 落点：`@omni/*` 新包中的 observer prompt 与输出 schema 校验；与 OIA-Memory 的 Distiller 对齐——Observer 的 `observation` 即 Distiller 的增量输入。

2. **隐式信号 → 标签（把 DSH 会话事件日志变成监督）**
   - DSH 已有会话事件日志/Goals/Schedule，可直接实现 `FEEDBACK_POLICIES` 四事件表 + 30 分钟 TTL + `scope(observation/task/session)` 分级；**`|score|<0.45` 不打标签**的阈值策略可原样搬，避免噪声标签污染。
   - **missed-opportunity 60s 窗口**（`user_prompt_after`）与回溯三段式 prompt 是「沉默期的 FN 代理」，对 OIA 尤其重要——主动性系统绝大多数时间是沉默的，只靠采纳/拒绝会训练出「永不干预」。
   - 落点：OIA 的 Governance 组件记录 `label_sources + label_rationale`（`labeling.py:label_rationale()` 已给出「取最强 3 条证据拼 rationale」的格式），天然满足「画像→场景→事实→原话」可下钻。

3. **干预时机评测集（M5 评测硬化的核心增量）**
   - 复刻 OSWorld-Persona 五阶段：**选帧（MSE τ）→ 滚动 5 帧窗口 + persona 配对 → 两阶段标注（先 persona 无关感知、后 persona 条件决策）→ 自洽复跑（temp 0.4，2 次全票）→ 人工复核正例 → 按时间切分**。
   - 指标直接用 **macro F1 + False-Alarm + per-persona F1**（FA 是主动性的「打扰率」，比 F1 更能约束产品行为）；报告 `strict_accuracy` 与 `parse_coverage` 防止「解析失败被静默丢弃」。
   - persona 池可扩到代码里的第 5 个 `minimal_interruptions`，做更极端的打扰厌恶压力测试。
   - 落点：`docs/` 下新增 OIA-Intervention-Timing 基准与 `score` 脚本，作为 M5 的完成判据之一。

4. **无需训练的个性化（先 PE 后 LoRA）**
   - **Coco-PE 型 bullet memory** 最适合 OIA：50 条上限、section 分类（`when_to_support / when_to_stay_silent / how_to_support / general`）、curator 的 `add/update/delete` 增量操作、错例+相邻正确例的对比反思——**用户可读、可审阅、可回滚**，与 Governance 的人审治理天然契合。
   - 代码级直接参考：`methods/coco_pe.py:CocoPETrainer`（默认超参即论文值）与 `memory/selection.py:select_evolution_moments()`。
   - 论文 §6.2 的证据可写进设计：PE 在 Qwen 系上优于 LoRA（0.483 vs 0.455），且 PE 的 prompt 可被用户检查（附录 F 的 Personalization view）。

5. **隐私边界（必须写进 OIA 的红线文档）**
   - 论文的诚实结论可直接引用：**Observer/记忆/适配可 on-device，但 Execution Agent 走云端时检索出的上下文会离开设备**——OIA 应在 Governance 中显式建模「哪些记忆块允许被注入到云端推理请求」（分域脱敏/白名单字段），而不是笼统宣称本地隐私。
   - 用户研究的三事件收集法（TP/FN/FP 只留计数、不留内容）是**最小化数据收集的可复用模板**；OIA 的遥测若要做，照这个口径。
   - 「显式、知情、可撤销的同意 + 可查看/编辑/删除已学偏好」应作为 M0/M1 就要有的控件（附录 A 的用户控制诉求 + 附录 F 的 review/edit/reject 流程）。

6. **调度与资源（对 DSH 后台任务的直接启示）**
   - 分层空闲策略：**60s 轻量修订 / 600s 全量演化 / sleep mode 停观察**，失败冷却 5min/15min，双 checkpoint 断点续跑——可直接映射到 DSH 的 background job + schedule 机制。
   - observation-first（前台交互永远抢占后台个性化）是主动性系统不卡顿的底线原则，值得写进干预策略器的非功能需求。

7. **反面清单（OIA 应避免的三件事，来自本文的自我披露）**
   - 不要只用采纳率优化（会退化为沉默；必须有回溯机会信号）；
   - 不要把反复出现的应用/领域当偏好（B.4 Final Organization 明令禁止）；
   - 不要默认「观察越多越好」而不给出域边界（附录 A 隐私 + 能耗两条限制）。

---

## 9. 诚实边界（代码能否跑、依赖、未验证项）

**已实测（本会话）：**

- ✅ **语法**：主仓全部 Python 文件 + `coco/lib/` 共 **174 个 `.py` 全部 `py_compile` 通过**（0 失败；未覆盖 `coco/desktop` 的 TS/JS 与 `desktop/release` 内嵌代码）。
- ✅ **单测可运行**：未执行 `uv sync`（避免联网装依赖），改用本机 Anaconda Python（`D:\CodeModel\python\anaconda\python.exe`）直接 `pytest tests`，所需依赖（pytest/cv2/numpy 等）在该环境已可用：
  - **47 项测试（4 个有效测试文件：14+6+17+10；`tests/test_tinker_runtime_compatibility.py` 收集到 0 项）**；
  - 全套设 `PYTHONUTF8=1` 实测两次，结果一致：**43 通过 / 4 失败**；
  - 不设 UTF-8 时仅对 `tests/test_metrics_and_methods.py` 单独实测：**3 失败 / 14 通过**，比 UTF-8 模式多 2 个失败（GBK 默认编码读 UTF-8 prompt 导致的 `UnicodeDecodeError` 与 hash 不符）；**未跑无 UTF-8 的全套**，故不给出全套数字。
  - **4 个失败全部是 Windows 平台假设，非业务逻辑错误**（已逐个定位）：
    1. `test_llamafactory_server_manages_only_its_process_group` —— `AttributeError: module 'os' has no attribute 'killpg'`（POSIX 进程组）；
    2. `test_coco_lora_revised_data_round_trip` —— `WindowsPath('C:/0.png') != WindowsPath('/0.png')`（POSIX 绝对路径假设）；
    3/4. `test_synthesis.py` 两个选帧测试 —— 临时目录含非 ASCII 用户名（`C:\Users\小黑\...`），**cv2.imwrite 直接写失败**（已复现：`imread -> None`，`np.fromfile` 报 FileNotFoundError），导致帧列表为空。
  - 结论：**在 Linux（论文目标环境）上这套测试大概率全绿，但本会话未在 Linux 验证**。
- ✅ **复现文档完整**：主仓 README + `benchmarks/osworld_persona/README.md` + `synthesis/README.md` 给出从数据下载到五方法评测的完整命令；coco 仓给出 CTI docker 与 CLI 命令。

**未做 / 未验证（不要当作已验证）：**

- ❌ **论文全部数字未复现**（0.12-0.19→0.35-0.48、2.4-3.0×、0.199→0.515、KV 26.7%→45.2%、TTFT 等均为论文自报，摘要存档也标注「所有引用数字均为论文自报，未复现」）。
- ❌ **benchmark 未跑**：需要 `hf download` 数据集（网络）、LLaMA-Factory 或 Tinker 服务、GPU；本会话均未执行。
- ❌ **coco 桌面端未构建/未运行**：需 `uv sync` + `npm install` + Electron；本地模型路径还需 Docker + NVIDIA driver ≥580 + Container Toolkit（本机为 Windows，不满足其 Linux 部署前提）。
- ❌ **Table 3 记忆基准、Table 5 on-device 剖析**：无本地脚本/无对应硬件，未验证。
- ⚠️ **未在全文中找到**的信息（不臆造）：venue/会议、页码、检索超参 k 的取值、用户研究被试人口学与逐日明细、引言「0.77 per-persona F1」的出处、Observer 输入的独立特征表。
- ⚠️ **仓库默认值与论文不一致处**（见 §6.4）：`--test-ratio` 默认 0.2 vs 论文 70/30；persona 池 5 个 vs 论文 4 个；三个版本的 Observer prompt 不同——复现时以论文附录 + 发布数据集为准，不要直接用仓库默认值。
- ⚠️ 本笔记所有公式均为 HTML `<math alttext="...">` 中的 LaTeX 源原样摘录；表格数字为 HTML 表格逐格提取，未经二次计算（除标注的正负比推算）。

---

*笔记写完后已 `read` 自查一遍：结构 9 节齐全，公式为 alttext 原文，所有实验数字标注了出处章节，「未找到」项均明确标注。*
