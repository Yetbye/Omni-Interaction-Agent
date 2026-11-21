# 论文摘要原文存档（arXiv API 抓取，2026-11-21）

> 来源：`http://export.arxiv.org/api/query`（id_list 批量查询），原文照录，供博客引用。
> 所有引用数字均为论文自报，未复现。
>
> **⚠️ 2026-11-21 更新：六篇已全部升级为「全文级」精读**——
> 全文 HTML/PDF 与逐篇精读笔记见 `papers/`（HTML 原文、ALIVE 的 PDF+提取文本）与 `papers/notes/01–06.md`（含公式、完整实验数字、代码研读、与摘要级理解的修正）。
> **本文件降级为「摘要级历史存档」**；凡与笔记冲突处，以笔记为准。已知冲突：
> - **M3-Agent（§6）**：本存档（据 GitHub README）写 8.2% / 920 视频；arXiv **v1 全文写 6.7%**、robot QA 表 2 为 1,276（摘要处 1,344）；仓库实测 annotations 为 robot 100 视频/1,276 QA、web 920 视频/3,214 QA（论文正文写 5,037）——三处口径不一，见 `notes/06-m3-agent.md`。
> - ALIVE（§5）：全文级数字（IPIBench、FSI EDP/FDP、消融）见 `notes/05-alive.md`，本摘要数字不再单独引用。

---

## 1. Deep Personalization: Continuous Context Observation for Local Proactive Agents
- arXiv: 2611.13457v1（2026-11-10）
- 作者：Yijia Shao, Song Bian, Junxian Guo, Jindong Jiang, Zhen Dong, Huizi Mao, Diyi Yang, Song Han

> As AI agents become increasingly capable, users must still recognize when and how to ask for help, leaving many opportunities for useful support unrealized. Proactive agents hold the promise to close this gap, but effective proactivity requires understanding users' real-time activities and personalizing intervention timing so that assistance is offered when it is likely to be helpful rather than disruptive. We introduce deep personalization, a closed-loop process that connects continuous context observation with continual agent adaptation: an agent observes user activity, selectively retains long-term context, and adapts intervention decisions from implicit feedback. We initiate deep personalization in Coco, a proactive desktop agent, by having a separate Observer Agent that monitors desktop activity, maintains compact long-term user modeling context, and learns when assistance is desirable from implicit signals such as engagement, dismissal, explicit requests, and repeated work patterns. We further develop efficient on-device strategies to enable the Observer Agent to run on local AI compute platforms without streaming raw activity data off-device. For evaluation at scale without collecting sensitive real-user activity, we introduce a data-synthesis pipeline that transforms existing computer-use trajectories into OSWorld-Persona, a benchmark with persona-conditioned intervention labels. On OSWorld-Persona, deep personalization improves macro F1 across three model backbones from 0.12-0.19 under direct prompting to 0.35-0.48 after adaptation, a 2.4-3.0x relative gain. In a week-long user study, we observe that Coco improves its macro-averaged F1 score from 0.199 to 0.515. These results suggest that on-device deep personalization can make proactive agents more timely and increasingly aligned with each user's everyday work.

---

## 2. Words Caught in the Heart: Simulating Person-Specific Expression for Self-Opaque Clients in Psychotherapy
- arXiv: 2611.00617v1（2026-10-31）
- 作者：Mengxi Xiao, Jingjing Chen, Ziyan Kuang, Jiarong Song, Wenhao Zhao, Zuchen Han, Yihuai Xu, Xiyang Huang, Jie Gong, Yuechen Jiang, Xueqing Peng, Yankai Chen, Xiaoyan Qu, Min Peng, Qianqian Xie

> Psychotherapy clients may experience distress before they can coherently describe it, and their expression changes with emotion, perceived safety, and trust. Large language model-based client simulators often rely on clinically organized case descriptions, which can encourage responses explaining the client's difficulties more clearly than their current understanding warrants. We propose Pandora, a psychotherapy client simulation framework that models how psychological experience becomes situated, individual expression. Pandora updates psychological and relational states in response to therapist interventions, constructs subjective appraisals and attributions, and plans response content and strategy. A separate expression planner combines these decisions with language patterns distilled from the client's utterances to guide linguistic realization. We evaluate Pandora against five baselines on 115 cases from four counseling and motivational interviewing datasets. It achieves the highest expression similarity and overall disclosure-alignment score among the compared systems. In a four-evaluator study of 45 matched disclosure items, Pandora receives AI-like judgments in 7.2% of ratings, compared with 43.3--67.2% for the baselines and 35.6% without expression planning. The ablation also reduces expression similarity. These results support explicit expression planning in client simulation to improve alignment with individual language patterns and reduce perceived AI-like expression in the evaluated setting.

---

## 3. Omni-IO Skills: Harnessing Your Agent Omni-Native
- arXiv: 2609.31847v1（2026-09-25）
- 作者：Yanlin Li, Mingyang Hao, Shengqiong Wu, Hao Fei, Mong-Li Lee, Wynne Hsu

> General-purpose agents can plan, reason, and act over long horizons, yet their production capabilities remain fragmented across text, images, audio, video, documents, 3D assets, and code. Extending a foundation model to additional modalities ties capability growth to costly model updates, while assembling specialist models and tools leaves unresolved how procedures, dependencies, intermediate assets, and cross-turn revisions should be coordinated. We present Omni-IO Skills, a plug-and-play Agent Harness that makes existing agents omni-native through hierarchical Skills, a standardized multimodal execution interface, dependency-aware orchestration, and a persistent Asset Registry. Multi-asset workflows are represented as Declare Execution Graphs, which schedule independent operations concurrently and register successful outputs for downstream and cross-turn reuse across replaceable execution backends. Its 27 Skills cover 38 representative tasks spanning seven artifact modalities and four capability families: understanding, generation, reasoning, and retrieval. On UniM-90, the harness raises the input-support rates of GPT-5.6 Sol and Claude Sonnet 5 from 40.00% and 38.89% to 100%, while increasing relative Semantic--Quality Coupled Score from 26.99 to 74.94 and from 27.82 to 77.78, respectively; Strict Structure Score reaches 100.00 and 99.78. These results establish harness-level capability composition as a practical route to broad, evolvable Omni systems without changing the host agent's reasoning core.

---

## 4. GraphMemix: Query-Aware Evidence Forests for Long-Term Multimodal Agent Memory
- arXiv: 2608.26983v1（2026-08-27）
- 作者：Geng Li, Yuhao Wang, Dong Li, Jianye Hao, Yuxin Peng（memorax 技术博客对应）

> Organizing long-term memory for multimodal agents remains challenging because existing methods either suffer from expensive question-agnostic offline summaries or naive embedding similarity matching that introduces incomplete and redundant context. To address this issue, we propose GraphMemix, a combinatorial-optimization graph memory framework that models memory organization as query-aware evidence-forest construction. Specifically, our method consists of three key components: (1) candidate graph construction, which expands multi-view seed memories through schema and semantic relations to obtain query-aware original context; (2) evidence utility and activation costs, which decouples direct memory support from anchor-conditioned relation verification to suppress redundant or conflicting information; and (3) forest optimization, which jointly selects a forest-format memory context under a maximum evidence budget and its reliable relational structure. By organizing memory into a query-relevant subgraph, the method avoids substantial lifecycle cost and recovers low-similarity complementary evidence. Experimental results across four long-term multimodal memory benchmarks demonstrate significant improvements with different foundation models and establish a new Pareto frontier between accuracy and lifecycle cost.

---

## 5. ALIVE: Making Streaming Video Models Interactive
- arXiv: 2611.21366v1（2026-11-16）
- 作者：Jinzhao Li, Yuan Xu, Jinxuan Wu, Yukun Wang, Yijia Lei, Hanyu Ji, Yinuo Chen, Zining Huang, Chi Zhang, Hang Ye, Yiwu Zhong, Honglei Yan, Panwang Pan, Yizhou Wang, Yin Li, Miao Liu

> Streaming video LLMs can continuously observe the world, yet often handle user requests in isolation. We introduce ALIVE, an interactive streaming video model supporting proactive monitoring, streaming task coordination, and feedback-seeking interaction. We construct ALIVE-SFT-288K and ALIVE-RL-3K for two-stage training. Contrastive supervised fine-tuning teaches when and how to respond, while reinforcement learning with Behavior-Aligned Credit Assignment (BACA) combines trajectory-level and local advantages to refine decisions, responses, and silence across multi-turn interactions. We also introduce FSI-Bench to evaluate feedback seeking and subsequent task completion. ALIVE achieves the highest overall scores among the compared models on four public benchmarks and FSI-Bench, demonstrating strong streaming understanding and interactive capabilities.

---

## 6. Seeing, Listening, Remembering, and Reasoning: A Multimodal Agent with Long-Term Memory（M3-Agent）
- arXiv: 2508.09736 · ICLR 2026（据其 GitHub README）

> We introduce M3-Agent, a novel multimodal agent framework equipped with long-term memory. Like humans, M3-Agent can process real-time visual and auditory inputs to build and update its long-term memory. Beyond episodic memory, it also develops semantic memory, enabling it to accumulate world knowledge over time. Its memory is organized in an entity-centric, multimodal format, allowing deeper and more consistent understanding of the environment. Given an instruction, M3-Agent autonomously performs multi-turn, iterative reasoning and retrieves relevant information from memory to accomplish the task. To evaluate memory effectiveness and memory-based reasoning in multimodal agents, we develop M3-Bench, a new long-video question answering benchmark. M3-Bench comprises 100 newly recorded real-world videos captured from a robot's perspective (M3-Bench-robot) and 920 web-sourced videos across diverse scenarios. We annotate question-answer pairs designed to test key capabilities essential for agent applications, including human understanding, general knowledge extraction, and cross-modal reasoning. Experimental results show that M3-Agent, trained via reinforcement learning, outperforms the strongest baseline, a prompting agent using Gemini-1.5-pro and GPT-4o, achieving 8.2%, 7.7%, and 5.3% higher accuracy on M3-Bench-robot, M3-Bench-web and VideoMME-long, respectively.

---

## 附注
- 《Native Real-Time Interactive Omni Intelligence: A Unified Framework》综述：公开渠道（arXiv API 多组检索）**无结果**，仅有用户本地 PDF，未入库；博客中不得引用其具体论断。
- 以上全部数字为论文自报，未独立复现。
