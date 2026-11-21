# 06 · M3-Agent 精读笔记

> Seeing, Listening, Remembering, and Reasoning: A Multimodal Agent with Long-Term Memory
> arXiv:2508.09736v1 [cs.CV]，2025-08-12；本地全文：`docs/sources/papers/2508.09736-m3-agent.html`（484KB / 7474 行 HTML，LaTeXML 导出，含 15 个正文+附录章节）；配套代码：`docs/sources/repos/m3-agent/`（github.com/bytedance-seed/m3-agent，无 `.git`，为快照）；摘要对照：`docs/sources/papers-abstracts.md` §6。

---

## 1. 论文速览

| 项 | 内容 | 出处 |
|---|---|---|
| 标题 / 作者 | M3-Agent；Lin Long, Yichen He, Wentao Ye, Yiyuan Pan, Yuan Lin, Hang Li, Junbo Zhao, Wei Li | 首页作者块 |
| 机构 | ByteDance Seed（1）、浙江大学（2）、上海交通大学（3）；通讯 `linyuan.0@bytedance.com` | 首页作者块 |
| venue | **arXiv HTML v1 未标注任何会议**（首页日期 2025-08-12）；**ICLR 2026 仅见于 GitHub README 标题行**（摘要存档 §6 也注明「据其 GitHub README」） | HTML 首页；`README.md:5` |
| 代码 | github.com/bytedance-seed/m3-agent；项目页 m3-agent.github.io；HF 模型 `ByteDance-Seed/M3-Agent-Memorization`、`ByteDance-Seed/M3-Agent-Control`；HF 数据 `ByteDance-Seed/M3-Bench` | `README.md:7-11` |
| 训练代码 | **不在本仓库**，外链两个仓库：`github.com/hyc2026/sft-qwen2.5-omni-thinker`（memorization）、`github.com/hyc2026/M3-Agent-Training`（control） | `README.md:190-193` |
| 许可 | **Apache License 2.0**（仓库根 `LICENSE`，代码头 `Copyright (2025) Bytedance Ltd.`） | `LICENSE` |
| 一句话定位 | 双进程（memorization / control）多模态长期记忆 agent：30s 切片在线生成情节+语义记忆并挂到**实体中心多模态记忆图**上，control 用 RL 训出的多轮检索-推理策略读图答题 | §1、§4 |
| 摘要自报主结果 | 比最强基线 Gemini-1.5-Pro(memorization)+GPT-4o(control) 混合 agent 高 **6.7 / 7.7 / 5.3**（robot / web / VideoMME-long） | HTML 摘要、§1 |

> ⚠️ 版本差异：本地 HTML 是 **v1**（摘要写 6.7%、929 个 web 视频、robot 1,344 QA），而 `papers-abstracts.md` §6 与 `README.md:15` 的摘要写 **8.2%、920 个 web 视频**。仓库结果图 `figs/exp_result.png` 的 **M3-Agent 行与 v1 表 5 逐格一致**（30.7 / 48.9 / 61.8），但个别基线格子不同（MA-LMM 的 VideoMME-Long：HTML **7.2** vs 图中 **17.3**；MovieChat **19.4** vs **19.5**），因此 **8.2% 这个数在本地任何材料里都对不上**，详见 §8。

---

## 2. 全文结构

| 章节 | 内容 | 关键落点 |
|---|---|---|
| Abstract | 双进程 + 情节/语义记忆 + 实体中心组织 + M3-Bench + RL 主结果 | 摘要 |
| §1 Introduction | 三能力（感知/记忆/推理）；episodic vs semantic 例子（Alice 咖啡、绿色垃圾桶）；memorization 的两个挑战：**Infinite information processing**、**World knowledge construction**；三条贡献 | §1 |
| §2 Related Work | 2.1 AI agent 长期记忆（轨迹拼接 / 摘要 / latent embedding；指出多模态记忆要保持一致性并**内化世界知识**）；2.2 在线视频理解（长上下文不可扩展 → 特征记忆模块缺一致性 → Socratic 语言记忆仍缺一致性） | §2 |
| §3 Datasets | M3-Bench 构造（3.1 robot、3.2 web）、表 2 与 7 个 LVQA 基准对比、3.3 GPT-4o 自动评测（与人工 96% 一致） | 表 1、表 2 |
| §4 Approach | 4.1 长期记忆（表 3 节点属性、表 4 检索接口、权重投票）；4.2 memorization（一致实体表示、记忆生成）；4.3 control（算法 1、三类 prompt）；4.4 Training（双模型初始化、SFT 与 DAPO RL、式 1–3） | 表 3、表 4、算法 1、式 1–3 |
| §5 Experiments | 5.1 基线（Socratic / 在线视频理解 / Agent 三类，H=5、k=2）；5.2 数据与评测；5.3 主结果表 5；5.4 消融表 6、7；5.5 案例与硬案例 | 表 5–7 |
| §6 Conclusion | 重述 + 两个未来方向：语义记忆的选择性注意、更丰富高效的**视觉记忆**（快照） | §6 |
| §7 Acknowledgment | — | — |
| §8 M3-Bench-robot 附录 | 脚本/ QA 标注规范、两轮质检、标注员信息、脚本样例（表 8） | §8 |
| §9 M3-Bench-web 附录 | 13 条标注红线、三阶段质检（合格率 ≥90%）、10 名标注员 | §9 |
| §10 Tools 实现细节 | 人脸（5fps、InsightFace buffalo_l、HDBSCAN）、说话人（Gemini-1.5-Pro 分割+ASR、ERes2NetV2、<2s 过滤）、检索（MIPS、阈值 0.3/0.6、k=2、t=0.5）、语音处理 prompt（表 9） | §10 |
| §11 Memorization 演示数据合成 | 11.1 情节记忆（GPT-4o→Gemini 混合，表 11）；11.2 实体 ID 关系（meta-clip + 算法 2 投票字典）；11.3 语义记忆（表 12）；11.4 合成数据质量 | 表 10–12、算法 2 |
| §12 Evaluation of Memorization | 200 条留出验证集、AutoDQ + 等价关系 P/R/F1、表 13 选 3 epoch ckpt | 表 13 |
| §13 RL Training Details | 13.1 DAPO 超参（表 14）与训练曲线（图 4）；13.2 GRPO 目标（式 4、5，ε=0.2、β=0.01） | 表 14、式 4–5 |
| §14 Case Study | 表 15/16 记忆生成对比；表 17 完整 control 轨迹（Tomasz 想象力问题） | 表 15–17 |
| §15 Prompt Templates | 15.1 GPT-4o 评测 prompt（表 18）；15.2 Socratic prompt（表 19）；15.3 M3-Agent 与基线的 memorization/control prompt（表 20–22） | 表 18–22 |

---

## 3. 方法精读

### 3.1 记忆图 schema（§4.1，表 3、表 4）

**论文口径**：长期记忆是外置数据库，组织成 memory graph，每条记忆一个节点；节点属性（表 3）：

| 属性 | 论文描述（表 3） |
|---|---|
| `id` | 节点唯一标识 |
| `type` | 模态类型（text / image / audio；自然语言记忆→text，人脸→image，语音→audio） |
| `content` | 原始内容：纯文本、base64 图像、base64 音频 |
| `embedding` | 内容向量，用于相似度检索 |
| `weight` | 数值，表示该节点的**置信度** |
| `extra_data` | JSON 元数据（如时间戳） |

- 节点之间用**无向边**连接，边是「把相关记忆串起来的线」（§4.1 p1）。
- **冲突消解 = 权重投票**：反复被强化的条目权重升高，压过弱强化的冲突条目（§4.1 p2；§4.2 举例 `<voice_3>↔<face_0>` 随时间胜出）。
- 两类检索接口（表 4）：`search_node(query)` → top-k 相关节点，支持**文本/图像/音频多模态查询**与按模态过滤；`search_clip(text_query)` → top-k 相关视频片段的情节+语义记忆。

**代码口径（`mmagent/videograph.py`，与论文有出入，见 §8）**：

```python
class VideoGraph:
    self.nodes = {}        # node_id -> Node(id, type, embeddings, metadata)
    self.edges = {}        # (id1, id2) -> weight   # 双向各存一份，即无向加权边
    self.text_nodes = []   # 文本节点插入顺序
    self.text_nodes_by_clip = {}        # clip_id -> [text node ids]
    self.event_sequence_by_clip = {}    # clip_id -> [episodic node ids]
    self.character_mappings = {}        # character_i -> ["face_3","voice_7",...]
    self.reverse_character_mappings = {}# "face_3" -> "character_i"
```

- 节点 `type` 只有 4 种：`img` / `voice` / `episodic` / `semantic`（`videograph.py:66`）。
- `Node` 类**没有 `weight` 字段**，只有 `embeddings` 与 `metadata`（`contents`、`timestamp`）；权重实际落在**边上**（`reinforce_node` / `weaken_node` 对所有相连边 ±1，`videograph.py:253/276`）。
- 边的创建只有一处：`memory_processing_qwen.py:195` `add_edge(text_node, entity_node)`，且 `add_edge` 禁止**同类型文本节点互连**（`videograph.py:231`）。即实际边类型 = **text↔img、text↔voice**（+ 语义上允许的 episodic↔semantic）。
- **人脸-语音的等价关系不是 img-voice 边**，而是以一条语义文本节点保存：内容形如 `Equivalence: <face_1>, <voice_2>`；`refresh_equivalences()`（`videograph.py:417`）用**并查集（union-find + 按秩合并 + 路径压缩）**把同一人的 face/voice 合并成 `character_i`，`fix_collisions(mode='eq_only')` 保证**每个 voice 最多映射到一个 face**（同权时随机，`videograph.py:361`）。

### 3.2 双进程管线（§1、§4.2、§4.3）

```
                    ┌──────────── memorization（在线、逐 30s 切片）────────────┐
视频/音频流 ──30s切片──▶ process_video_clip(5fps 帧 + 16kHz wav)
                          ├─ 人脸: InsightFace buffalo_l → HDBSCAN 聚类 → <face_i>
                          ├─ 说话人: Gemini-1.5-Pro 分割+ASR(≥2s) → ERes2NetV2 → 阈值匹配 <voice_j>
                          ├─ memorization 模型生成 episodic + semantic（必须用 ID 指代人物）
                          └─ 写入 VideoGraph：文本节点 + 与实体的边 + 权重投票
                    └────────────────────────────────────────────────────────┘
                                     ▼ pickle: data/memory_graphs/<split>/<video>.pkl
                    ┌──────────────── control（按指令触发）────────────────────┐
指令 q ──▶ π_θ (Qwen3-32B) ──[Search]──▶ search_clip / search_node ──▶ 追加到上下文
              │                 （最多 H=5 轮；最后一轮强制 [Answer]）
              └──[Answer]──▶ 最终答案 ──▶ GPT-4o 评测器给 0/1 奖励
                    └────────────────────────────────────────────────────────┘
```

**Memorization 细节**

1. **30s 切片**：训练数据切 30s（§4.4）；README 的复现脚本用 `ffmpeg -ss $start -t 30 -c copy` 切成 `data/clips/<video>/<i>.mp4`（`README.md:86-102`）。`processing_config.json` 里 `interval_seconds: 30, fps: 5`。
2. **中间输出（工具层，§10 + 代码）**，按切片落盘、可复用（`memorization_intermediate_outputs.py`）：
   - `clip_{i}_faces.json`：`[{frame_id, bounding_box, face_emb, cluster_id, extra_data:{face_type(ortho/side 按长宽比), face_base64, face_detection_score, face_quality_score}}, ...]`（`src/face_extraction.py:61-72`）。质量分 = 原始 embedding 的 L2 范数；过滤阈值 `face_detection_score_threshold=0.85`、`face_quality_score_threshold=22`、每人物最多 `max_faces_per_character=3` 张脸（`face_processing.py:144-147`）。
   - `clip_{i}_voices.json`：`[{start_time, end_time, asr, duration, audio_segment(base64 wav), embedding(192 维归一化), matched_node}, ...]`（`voice_processing.py:159-216`）；说话人分割**由 Gemini-1.5-Pro 完成**（prompt = `prompts.py:14` `prompt_audio_segmentation`，要求只按说话轮次切、不给说话人 ID），`duration < 2s` 直接丢弃（`min_duration_for_audio: 2`）。
   - 人脸抽帧 `process_video_clip(fps=5)` 与论文 §10「5 frames per second」一致（`utils/video_processing.py:82`）。
3. **持久身份**：每个抽到的脸/声先用 `search_img_nodes` / `search_voice_nodes` 与已有节点比对（图像平均余弦相似度阈值 **0.3**、语音 **0.6**，命中则 `update_node` 追加 embedding——每节点最多存 10 个人脸 embedding / 20 个语音 embedding，超出**随机采样**淘汰，`videograph.py:219`），否则新建节点；得到的 `face_id/voice_id` 作为跨切片的持久引用（§4.2）。
4. **记忆生成**：模型输出是**两条列表**——情节记忆（原子句）与语义记忆；人物必须写成 `<face_1>` / `<voice_2>`，禁用代词与编造名字（表 11/12、`prompts.py:308`）。语义记忆中特殊的一类是 `Equivalence: <face_x>, <voice_y>`。
5. **落图规则**（`memory_processing_qwen.py:183-243`）：每条记忆 → 一个文本节点 → 与句中实体建边；语义记忆若其实体集合是某已有语义节点实体的子集，则按相似度投票：**>0.85 → `reinforce_node`（边权 +1）**，**<0 → `weaken_node`（−1）**，否则新建节点。这就是论文说的 weight-based voting 的实现。
6. **训练标签的 meta-clip 算法（§11.2，算法 2）**：把长视频按关键帧切成 ≤5s 短片段；只含 **1 个 face ID 且 1 个 voice ID** 的片段叫 meta-clip（高置信同人样本）；据此建 face↔voice 二部图，边权 = 共现次数，**先删权重=1 的边**，再对每个 face 保留占优边（占比 ≥ p 才保留，否则全删），对每个 voice 只留权重最大边，得到 `voice→face` 字典；用它给任意 30s 子片段自动标注 `Equivalence:` 行。含 voice ID 不在字典中的片段**整条丢弃**；最终 **10,952** 个 30s 片段带有效等价标注，人工抽检 48 条准确率 **95.83%**（§11.2 末）。

**Control 细节（算法 1，§4.3）**

- 策略模型 `π_θ`，输入问题 `q` 与记忆 `ℳ`，最多 **H 轮**（实验统一 **H=5**，§5.1）。
- 三类 prompt：① `system_prompt`（会话开头，含问题）；② `instruction_prompt`（每轮开头追加，除最后一轮）；③ `last_round_prompt`（仅最后一轮，强制作答）。原文见表 22 / `m3_agent/control.py:79-89`。
- 每轮：生成 `Action: [Search]|[Answer]` + `Content`；`[Search]` → 用 content 当 query 检索记忆，把结果 `Searched knowledge: {...}` 追加为下一轮 user 消息；`[Answer]` → 结束。
- 检索是**两级**的：普通查询走 `search_clip`（top-2 片段，阈值 t=0.5，返回 `CLIP_i: [句子...]`）；当 content 含 `"character id"` 时改走 `mem_wise=True` 的 `search_node` 路径（top-20 节点），用于查 `<character_i>` ↔ 人名的映射（`control.py:121-127`；prompt 明确教模型先查 ID 映射、之后用 ID 检索）。
- 每次检索**只返回没见过的片段**（`current_clips` 去重，`retrieve.py:260`），空结果会被显式告知：`(The search result is empty. Please try searching from another perspective.)`（`control.py:130`）。
- **因果截断 `before_clip`**：robot 集每条 QA 带提问时刻 `timestamp` 与其对应 `clip_id`（`before_clip`），control 会 `truncate_memory_by_clip(before_clip)`，只允许检索提问之前的内容（`control.py:117-119`、`videograph.py:866`）。web 集问题时间戳全在视频末尾，故无 `before_clip`（§3.2）。

### 3.3 训练细节（§4.4、§13）

| | Memorization | Control |
|---|---|---|
| 基座 | **Qwen2.5-Omni-7B**（视觉+音频） | **Qwen3-32B**（消融另有 8B/14B） |
| 方式 | **仅模仿学习（SFT）** → `memory-7b-sft` | **RL：DAPO**（消融对比 GRPO），从 `control-32b-prompt` 初始化 |
| 数据 | 自有视频，标注规范同 M3-Bench-web：**500 个长视频 / 26,943 个 30s 切片 / 2,736 条 QA**（§4.4 p2） | 同一批 QA；先用 `memory-7b-sft` 为每个视频生成记忆，问答只允许在**该视频的记忆**里搜（§4.4 p5） |
| 演示合成 | 三阶段：①情节（GPT-4o 0.5fps 帧描述作先验 → Gemini-1.5-Pro 出终稿）②meta-clip 等价关系 ③语义（GPT-4o 起草 → Gemini 精修）；共 **10,752 训练样本 + 200 验证样本**（§11 开头） | — |
| 超参 | 3 epochs，lr **1e-5**，batch **16**，**16×80GB GPU**（§4.4 p4） | 表 14：batch 32、**16×80GB（32B 用 32 卡）**、rollout 并行 1/1/2、lr **1e-6**、**H=5**、组大小 **G=4**、**180 步**、`ε_low=0.2`、`ε_high=0.28` |
| 选点 | 200 条留出集 AutoDQ + Eq-F1，取 **3 epoch**（表 13） | 训练曲线单调上升（图 4，EMA 平滑权重 0.9） |

> 摘要说「M3-Agent, trained via reinforcement learning」，但**真正走 RL 的只有 control 模型**；memorization 是纯 SFT（§4.4 明写 imitation learning）。

**奖励与目标（式 1–3，§4.4）**

```latex
R_i = 1  if gpt4o_evaluator(q, a, y_i) = True  else 0                       % 式(1) 二值奖励，用 §3.3 的 GPT-4o 评测器
\hat{A}_{i,t} = \frac{R_i - \mathrm{mean}(\{R_i\}_{i=1}^G)}{\mathrm{std}(\{R_i\}_{i=1}^G)}  % 式(2) 组内归一化优势（GRPO 式同形）
```

```latex
\mathcal{J}_{\mathrm{DAPO}}(\theta)=
\mathbb{E}\Big[\tfrac{1}{\sum_i\sum_t \mathbb{I}(\tau_{i,t})}
\sum_i\sum_t \mathbb{I}(\tau_{i,t})\cdot
\min\big(r_t\hat{A}_{i,t},\ \mathrm{clip}(r_t,1-\epsilon_{\mathrm{low}},1+\epsilon_{\mathrm{high}})\hat{A}_{i,t}\big)\Big]
\ \ \text{s.t.}\ 0<\sum_{i=1}^{G}R_i<G                              % 式(3)
```

- `𝕀(τ_{i,t})=1` 当且仅当该 token 是 **LLM 生成的 token**（不算 prompt/检索结果）。
- 约束 `0 < ΣR_i < G` = **过滤全对/全错组**（DAPO 的 dynamic sampling 思路），是它与 GRPO 的关键差别之一。
- GRPO 版本（式 4、5，§13.2）额外带 `−β D_KL[π_θ‖π_ref]`，`ε=0.2`、`β=0.01`，其余超参沿用 DAPO。

### 3.4 记忆质量如何支撑控制（§4.2 的设计动机）

- 一致性不靠文字描述（"a man with a beard" 会漂移），而靠**保留原始多模态特征**（人脸图、声纹）作为持久身份锚点。
- 语义记忆不只是内容，还是**额外检索线索**（§4.2 p1）。
- 跨模态等价关系一旦确立，检索时同一人的 face/voice 节点被统一到共享 `<character_id>`（`retrieve.py:35-73` 的 `translate` / `back_translate`：返回记忆时把 `<face_3>` 换成 `character_4`，查询时把 `character_4` 展开成所有成员 ID 的多个查询变体）。

---

## 4. 实验精读

### 4.1 M3-Bench 构造（§3、附录 §8–9）

**总体**（表 2）：

| Benchmark | #Videos | 平均时长(s) | #QAs | 标注 | 形式 | Agent Present | Cross-Modal | Human | Knowledge |
|---|---|---|---|---|---|---|---|---|---|
| M3-Bench-robot | 100 | **2,039.9** | **1,276** | 人工 | 开放 | ✓ | ✓ | ✓ | ✓ |
| M3-Bench-web | **929** | 1,630.7 | **5,037** | 人工 | 开放 | ✗ | ✓ | ✓ | ✓ |
| （对比）EgoSchema / LongVideoBench / HourVideo / MVBench / Video-MME / MLVU | 5,063 / 3,763 / 500 / 3,641 / 900 / 1,730 | 180 / 473 / 2,742 / 16 / 1,018 / 930 | 5,063 / 6,678 / 12,976 / 4,000 / 2,700 / 3,102 | 混合 | 选择/混合 | ✗ | ✗ | ✗ | ✗ |

> §1 又说 robot **1,344** 条 QA（与表 2 的 1,276 不一致，见 §8）；本地 `data/annotations/robot.json` 实测 **100 视频 / 1,276 QA**，与表 2 一致。

**五类问题（表 1）**

| 类型 | 说明 + 例子 |
|---|---|
| Multi-detail Reasoning | 聚合分散在视频多处的信息。例：五件藏品中哪件起拍价最高？ |
| Multi-hop Reasoning | 跨片段逐步推理。例：他们去过 Ding Cha 之后又去了哪家奶茶店？ |
| Cross-modal Reasoning | 跨视觉/音频推理。例：红色文件夹放机密、白色放普通 → 机密放哪个？ |
| Human Understanding | 身份/情绪/性格/关系。例：Lucas 烹饪水平如何？需聚合多次做菜行为 |
| General Knowledge Extraction | 从具体事件抽通用知识。例：蔬菜该放冰箱哪一层？ |

**M3-Bench-robot（§3.1、附录 §8）**
- 7 个日常场景（客厅/厨房/卧室/书房/办公室/会议室/健身房），每脚本 1 机器人 + 2–4 人；每场景多主题变体。
- 硬性要求：**≥15 题**且每题标注所需 reference events；每题至少属一类；**每脚本 ≥70 个事件以保证 ≥30 分钟**（§3.1 与 §8.1）。
- **拍摄是人演机器人**：1 名演员戴头戴设备（iPhone 16 Pro / Xiaomi 14 Ultra / GoPro HERO13）拍第一人称；**67 名演员、51 个地点、每地 ≤3 条**；两路音频（头戴原始环境声 + 每人领夹麦）。
- QA：保留/改写脚本题并标 `timestamp`，**时间戳必须早于机器人回应/动作以免泄题**；不足则补题到 ≥12（§3.1 p9；附录 §8.2 写 ≥10）；另做带说话人身份与时间戳的字幕。
- 质检：两轮（先 3 条试标→作者反馈定资格；正式每批 5 条抽 1 条，>1 条无效则整批重做）；**5 名标注员答题，最终人工准确率 90.7%**，最常见错因是**计数类**（§8.3）。标注员 11 人写脚本、67 人拍摄、5 人做 QA（§8.4）。

**M3-Bench-web（§3.2、附录 §9）**
- YouTube，**问题驱动选片**（能出 ≥5 题才收）；类别清单强调信息密度与 agent 相关性，每类 ≤20 条，最终 **46 类**；同收集者同时出 ≥5 题。
- 20–40 分钟视频出 5 题，>40 分钟出 10 题；每视频**至少 2 题人物属性 + 2 题常识推理**；视觉信息 ≤720p 可辨；所需上下文 **>10 秒**；不能全靠音频、要保持一定视觉题比例；Yes/No 平衡；禁止把台词直接改写成问题。
- **所有问题时间戳 = 视频末尾**（§3.2 p4）。
- 质检三阶段（试标 → 10 条批抽 2 → 30 条批抽 5，合格率 <90% 复检），10 名标注员（§9.2、9.3）。

**评测（§3.3）**：GPT-4o 当判官（prompt = 表 18 / `prompts.py:992`，判断「agent 答案能否在题境下**蕴含**参考答案」，只输出 Yes/No）；**100 条三元组、3 名作者独立标注、多数票对齐，一致率 96%**。

### 4.2 基线与设置（§5.1）

- **Socratic Models**：多模态模型描述 30s 切片当记忆 → GPT-4o 做单轮 RAG（`search_clip` 一次）。记忆生成器 4 种：Gemini-1.5-Pro（整段视频+音频）、GPT-4o（0.5fps 帧 + ASR 文本）、Qwen2.5-Omni-7b（整段）、Qwen2.5-VL-7b（帧+ASR）。
- **在线视频理解**：MovieChat、MA-LMM（1fps + 100 帧）、Flash-VStream（1fps 异步压缩），用官方权重。
- **Agent**：`Gemini-Agent`（memorization 与 control 都用 Gemini-1.5-Pro，输入含人脸/说话人识别结果）；`Gemini-GPT4o-Hybrid`（memorization 仍 Gemini、control 换 GPT-4o）。
- 统一 **H=5**；`search_clip` 返回 **top-2** 片段（找不到返回空）（§5.1 p8）。

### 4.3 主结果（表 5，全部自报数字）

| Method | robot MD | MH | CM | HU | GK | **All** | web MD | MH | CM | HU | GK | **All** | VideoMME-Long |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Socratic: Qwen2.5-Omni-7b | 2.1 | 1.4 | 1.5 | 1.5 | 2.1 | **2.0** | 8.9 | 8.8 | 13.7 | 10.8 | 14.1 | **11.3** | 42.2 |
| Socratic: Qwen2.5-VL-7b | 2.9 | 3.8 | 3.6 | 4.6 | 3.4 | **3.4** | 11.9 | 10.5 | 13.4 | 14.0 | 20.9 | **14.9** | 46.9 |
| Socratic: Gemini-1.5-Pro | 6.5 | 7.5 | 8.0 | 9.7 | 7.6 | **8.0** | 18.0 | 17.9 | 23.8 | 23.1 | 28.7 | **23.2** | 38.0 |
| Socratic: GPT-4o | 9.3 | 9.0 | 8.4 | 10.2 | 7.3 | **8.5** | 21.3 | 21.9 | 30.9 | 27.1 | 39.6 | **28.7** | 38.8 |
| MovieChat | 13.3 | 9.8 | 12.2 | 15.7 | 7.0 | **11.2** | 12.2 | 6.6 | 12.5 | 17.4 | 11.1 | **12.6** | 19.4 |
| MA-LMM | 25.6 | 23.4 | 22.7 | 39.1 | 14.4 | **24.4** | 26.8 | 10.5 | 22.4 | 39.3 | 15.8 | **24.3** | 7.2 |
| Flash-VStream | 21.6 | 19.4 | 19.3 | 24.3 | 14.1 | **19.4** | 24.5 | 10.3 | 24.6 | 32.5 | 20.2 | **23.6** | 25.0 |
| Gemini-Agent | 15.8 | 17.1 | 15.3 | 20.0 | 15.5 | **16.9** | 29.3 | 20.9 | 33.8 | 34.6 | 45.0 | **34.1** | 55.1 |
| Gemini-GPT4o-Hybrid | 21.3 | 25.5 | 22.7 | 28.8 | 23.1 | **24.0** | 35.9 | 26.2 | 37.6 | 43.8 | 52.2 | **41.2** | 56.5 |
| **M3-Agent** | **32.8** | **29.4** | **31.2** | **43.3** | 19.1 | **30.7** | **45.9** | **28.4** | **44.3** | **59.3** | **53.9** | **48.9** | **61.8** |

（列缩写：MD 多细节 / MH 多跳 / CM 跨模态 / HU 人类理解 / GK 通用知识；仓库结果图用 ME/PU 命名。）

**论文自报的增量（§5.3、§1）**
- vs 最强 agent 基线 Gemini-GPT4o-Hybrid：robot **+6.7**（30.7−24.0）、web **+7.7**（48.9−41.2）、VideoMME-long **+5.3**（61.8−56.5）。
- §5.3 正文另有一句：robot 上比「最强基线 **MA-LMM**」高 **6.3**（30.7−24.4）——同一段里两个「最强基线」口径不同。
- 分题型：robot 上比 MA-LMM 高 **HU +4.2、CM +8.5**；web 上比 Hybrid 高 **HU +15.5、CM +6.7**。
- 值得注意：**M3-Agent 在 robot 的 GK（通用知识）列不是最高**——19.1，反而低于 Gemini-GPT4o-Hybrid 的 **23.1**（该列最高）；它的优势集中在 HU / CM / MD。
- Socratic 系普遍崩（robot ≤8.5）；MA-LMM 在 VideoMME-long 只有 **7.2**（几乎失效；仓库结果图同一格是 17.3，属版本差异）。

### 4.4 消融

**表 6：memorization 模型（control 固定）**

| Memorization | robot | web | VideoMME-Long |
|---|---|---|---|
| memory-gemini-prompt | 28.7 | 46.3 | 52.7 |
| memory-7b-prompt | 25.3 | 39.9 | 50.8 |
| **memory-7b-sft（=M3-Agent）** | **30.7** | **48.9** | **61.8** |
| memory-7b-sft w/o equivalence | 19.5 | 39.7 | 52.1 |
| memory-7b-sft w/o semantic memory | 13.6 | 29.7 | 48.7 |

- 换成 Gemini 记忆 → **−2.0 / −2.6 / −9.1**；换成未微调的 7b prompt → **−5.4 / −9.0 / −11.0**（§5.4）。
- **去掉语义记忆 → −17.1 / −19.2 / −13.1**（摘要引用的就是这组）；**去掉身份等价 → −11.2 / −9.2 / −9.7**（30.7→19.5 等，由表 6 相减得到，论文正文只说「significantly degrades」）。

**表 7：control 模型（memorization 固定为 memory-7b-sft）**

| Control | robot | web | VideoMME-Long |
|---|---|---|---|
| control-32b-grpo | 30.0 | 47.7 | 58.7 |
| control-8b-prompt | 16.4 | 35.7 | 45.3 |
| control-8b-rl | 24.6 | 40.5 | 50.8 |
| control-14b-prompt | 18.3 | 36.9 | 49.1 |
| control-14b-rl | 28.2 | 46.9 | 56.0 |
| control-32b-prompt | 20.7 | 40.9 | 52.5 |
| **control-32b-rl（=M3-Agent）** | **30.7** | **48.9** | **61.8** |
| control-32b-prompt w/o inter-turn instruction | 12.8 | 32.3 | 48.3 |
| control-32b-rl w/o inter-turn instruction | 20.2 | 43.1 | 55.9 |
| control-32b-rl w/o reasoning | 19.0 | 40.1 | 52.3 |

- DAPO vs GRPO（32B）：**+0.7 / +1.2 / +3.1**（30.7 vs 30.0 等）。
- **RL 增益（32B）：+10.0 / +8.0 / +9.3**（20.7→30.7、40.9→48.9、52.5→61.8）；8B +8.2/+4.8/+5.5；14B +9.9/+10.0/+6.9。
- **去掉 inter-turn instruction（每轮追加的 instruction_prompt）：−10.5 / −5.8 / −5.9**；**去掉 reasoning（禁 `<think>`）：−11.7 / −8.8 / −9.5**（摘要引用值）。
- 消融规模：4 个模型尺寸 × 2 阶段 + 3 个设计开关，共 10 行，**每格一个数，无方差/多次运行报告**。

**表 13：memorization 模型本身的评测（200 条留出集）**

| Model | AutoDQ-P | R | F1 | Eq-P | Eq-R | Eq-F1 |
|---|---|---|---|---|---|---|
| memory-gemini-prompt | 0.692 | 0.539 | 0.606 | 0.472 | 0.805 | 0.595 |
| memory-7b-prompt | 0.495 | 0.355 | 0.414 | 0.117 | 0.192 | 0.145 |
| memory-7b-sft (1ep) | 0.634 | 0.596 | 0.616 | 0.742 | 0.817 | 0.778 |
| (2ep) | 0.628 | 0.610 | 0.619 | 0.845 | 0.810 | 0.827 |
| **(3ep，采用)** | **0.635** | **0.620** | **0.627** | 0.836 | 0.856 | **0.846** |
| (4ep) | 0.616 | 0.618 | 0.617 | 0.825 | 0.839 | 0.832 |
| (5ep) | 0.609 | 0.621 | 0.615 | 0.813 | 0.840 | 0.827 |

**合成数据质量（§11.4）**：情节记忆平均 **245.7 词** vs Gemini-1.5-Pro 151.3；语义 **276.2** vs 81.4；抽 10 个切片共 **353** 条记忆人工核对，准确率 **95.5%**，主要错因是说话人识别（背景噪声/叠话）。

### 4.5 案例与硬案例（§5.5、§14）

- 记忆对比（表 15/16）：`memory-7b-sft` 比 `memory-gemini-prompt` 更细（场景、动作、表情、逐字对白）、能稳定给出 `Equivalence:` 行、语义记忆更主动（人物属性、关系、体裁、常识）。
- control 完整轨迹（表 17）：问题「Tomasz 是有丰富想象力的人吗」→ 第 1 轮查 `What is the character id of Tomasz` → 拿到 `<character_4>` → 第 2 轮查其想象力人格 → **空结果** → 第 3 轮换角度查「创造性解题方法」→ 命中「对扩张无人机技术感兴趣，创新进取」→ 第 4 轮作答。
- 两类硬案例：①**细粒度细节**（"谁想吃火腿肠"、"帽子该挂高架还是矮架"）→ 需要选择性记忆/注意，全存会认知过载；②**空间推理**（"零食在哪"、"杯子在第几层"）→ **文字记忆不如视觉记忆**，未来应存**快照**（§5.5、§6）。

---

## 5. 代码研读

### 5.1 目录与职责

```
m3-agent/
├── README.md                     # 复现步骤、HF 链接、外链训练仓库
├── LICENSE                       # Apache-2.0
├── visualization.py              # 记忆图 pkl 的可读化输出（按 clip 打印文本/人脸/语音节点）
├── configs/
│   ├── api_config.json           # gpt-4o-2024-11-20 / gemini-1.5-pro-002 / text-embedding-3-large
│   ├── processing_config.json    # 30s、5fps、阈值、topk=2、total_round=5、ckpt 等
│   └── memory_config.json        # max_img_embeddings=10 / max_audio=20 / 0.3 / 0.6
├── m3_agent/                     # 三个可执行入口
│   ├── memorization_intermediate_outputs.py   # 只跑人脸+说话人，落 clip_{i}_faces/voices.json
│   ├── memorization_memory_graphs.py          # 读中间输出 → 模型生成记忆 → 组图 → pickle
│   └── control.py                             # vLLM 跑 M3-Agent-Control，多轮检索，GPT-4o 判分
├── mmagent/                      # 核心库
│   ├── videograph.py             # VideoGraph（951 行）：节点/边/等价/检索/截断
│   ├── retrieve.py               # search / translate / back_translate / answer_with_retrieval
│   ├── memory_processing_qwen.py # 记忆生成调用 + 落图 + 权重投票
│   ├── memory_processing.py      # parse_video_caption（实体抽取）等
│   ├── face_processing.py / voice_processing.py
│   ├── prompts.py                # 1300+ 行，40+ 个 prompt（论文表 9/11/12/18–22 的源）
│   ├── src/face_extraction.py, src/face_clustering.py   # InsightFace + HDBSCAN
│   └── utils/chat_api.py(Azure OpenAI), chat_qwen.py(Qwen2.5-Omni 本地), general.py, video_processing.py
├── data/annotations/{robot,web}.json   # 评测标注（robot 100 视频/1,276 QA；web 920 视频/3,214 QA）
└── figs/                         # illustration / m3-agent / exp_result / m3-bench-* 等
```

### 5.2 三个入口的真实流程

**① `m3_agent/memorization_intermediate_outputs.py`（93 行）** — 逐切片 `process_voices(preprocessing=["voice"])` + `process_faces(preprocessing=["face"])`，只写 JSON 不建图。中间产物已存在则直接读（幂等缓存）。

**② `m3_agent/memorization_memory_graphs.py`（113 行）** — 关键 30 行：

```python
def process_segment(video_graph, base64_video, base64_frames, base64_audio, clip_id, sample, clip_path):
    id2voices = process_voices(...)      # → <voice_j> + matched_node
    id2faces  = process_faces(...)       # → <face_i> + matched_node
    episodic_memories, semantic_memories = generate_memories(base64_frames, id2faces, id2voices, clip_path)
    process_memories(video_graph, episodic_memories, clip_id, type="episodic")
    process_memories(video_graph, semantic_memories, clip_id, type="semantic")

# 循环全部切片后：
video_graph.refresh_equivalences()                 # 并查集合并 → character_i
with open(sample["mem_path"], "wb") as f:
    pickle.dump(video_graph, f)                    # 整个 VideoGraph 对象 pickle
```

**③ `m3_agent/control.py`（211 行）** — 真正的双进程「control 侧」实现：
- vLLM 加载 `models/M3-Agent-Control`（`tensor_parallel_size=2`），采样 `temperature=0.6, top_p=0.95, top_k=20, max_tokens=1024`，`enable_thinking=True`；
- 用正则 `r"Action: \[(.*)\].*Content: (.*)"` 解析（`control.py:98`），**解析失败默认按 Search 处理**（`control.py:109`）；
- 每轮批量生成 → `multiprocessing.Pool().map(consumer, ...)` 执行检索；
- 检索分支：`"character id" in content` → `mem_wise=True, topk=20`（查 ID↔名字映射），否则 `threshold=0.5, topk=processing_config["topk"]`(=2)；
- 最后一轮追加 `(The Action of this round must be [Answer]. If there is insufficient information, you can make reasonable guesses.)`；
- 结果写 `data/results/<dataset>.jsonl`，每条带 `gpt_eval` 布尔（Azure GPT-4o 判官，`temperature=0, max_tokens=2048`，失败重试 20 次、每次 sleep 20s）。

### 5.3 关键 prompt 原文（可直接引用）

**① memorization 模型的训练/推理 prompt**（`mmagent/prompts.py:308` `prompt_generate_memory_with_ids_sft`，即 README 中 Qwen2.5-Omni-7B 用的 `prompt_generate_full_memory` 系列之一）：

> "Your task consists of two parts:
> 1. **Video Description**: Generate a detailed and cohesive description of the current video clip. Use the provided feature IDs as references to characters (when applicable). … **Each description should focus on a single atomic event or fact.**
> 2. **High-Level Conclusions**: Generate high-level reasoning-based conclusions that go beyond surface-level observations. … If a face and a voice feature refer to the same character, indicate it using this exact format: **`Equivalence: <face_x>, <voice_y>`**
> Output Format: `{ "video_description": [...], "high_level_conclusions": [...] }` … Please only return the valid JSON object"

（代码里解析键是 `video_descriptions` / `high_level_conclusions`，`memory_processing_qwen.py:153-154`，与 prompt 里写的 `video_description` 单复数不一致，靠 `validate_and_fix_json` 兜底。）

**② control 模型的 prompt**（`m3_agent/control.py:79-89`，对应论文表 22）：

> `system_prompt`：*"You are given a question and some relevant knowledge. Your task is to reason about whether the provided knowledge is sufficient to answer the question. If it is sufficient, output [Answer] followed by the answer. If it is not sufficient, output [Search] and generate a query … Question: {question}"*
>
> `instruction_prompt`：*"Output the answer in the format: Action: [Answer] or [Search] / Content: {content} … The search {content} needs to be different from the previous. You can get the mapping relationship between character ID and name by using search query such as: "What is the name of <character_{i}>" or "What is the character id of {name}". **After obtaining the mapping, it is best to use character ID instead of name for searching.** … Only name can appear in the answer, not character ID like <character_{i}>."*
>
> `last_round_prompt`：*"The Action of this round must be [Answer]. If there is insufficient information, you can make reasonable guesses."*

**对照：基线 agent 的 control prompt 长得多**（`prompts.py:476` `prompt_generate_action_with_plan`，论文表 21），多了 retrieval plan、`CLIP_x` 时序查询语法、"query 用关键词短语、不要用 Retrieve/What/When 等"、"不要与历史查询语义重复"等十几条工程约束。**RL 训出来的 control prompt 反而极简——这是「prompt engineering 被 RL 替代」的直接证据**（结合表 7 中 `control-32b-prompt` 用的是简单 prompt 也仍比带 plan 的 GPT-4o 弱）。

**③ 判官 prompt**（`prompts.py:992` `prompt_agent_verify_answer_referencing`，= 论文表 18）：要求「不要比表面形式，只判断 agent 答案能否在题境下蕴含 ground truth，只回 Yes/No」。

**④ 说话人分割 prompt**（`prompts.py:14` `prompt_audio_segmentation`，= 论文表 9）：ASR + 按说话轮次切段、**不分配说话人 ID**、MM:SS 时间戳、只回 JSON、无语音回 `[]`。

**⑤ 其余**：`prompt_generate_captions_with_ids`（Gemini/GPT-4o 记忆生成）、`prompt_generate_action_qwen3`（含「You need to provide an answer within 5 rounds.」）、`prompt_generate_plan`（Gemini 先出 retrieval plan）、`prompt_autodq`（AutoDQ 三分类 entailment/neutral/contradiction）、`prompt_refine_qa_list`（**中文推理→英文 QA** 的改写，说明部分标注链路在中文侧完成，与 `robot.json` 里中文 `reasoning` 字段吻合）。

### 5.4 记忆图 pkl 的结构与可视化

pkl 就是**整个 `VideoGraph` 对象**（`pickle.dump(video_graph, f)`），字段见 §3.1。加载注意：`m3_agent/control.py:30` 有一行 `sys.modules["videograph"] = mmagent.videograph`，说明历史 pkl 的类路径可能写成顶层 `videograph`，跨环境反序列化要小心模块别名。

```bash
python visualization.py --mem_path data/memory_graphs/robot/bedroom_01.pkl --clip_id 1
```

`visualization.py` 的输出逻辑（`print_clip_full`）：
1. 取 `vg.text_nodes_by_clip[clip_id]` → 逐节点打印 `[episodic|semantic] id=.. | contents`（内容先经 `retrieve.translate()` 把 `<face_3>` 换成 `character_4`）；
2. 用 `vg.get_connected_nodes(nid, type=['img'])` / `['voice']` 找该片段关联的**人脸节点与语音节点**；
3. `vg.print_faces(...)` 用 matplotlib 把 base64 人脸拼成 9 列网格并以 `Node {id}` 标题（`SHOW_FACES=True` 才画，无图形环境可关）；语音节点打印 ASR 内容与相连文本节点。
顶部三个开关：`ONLY`（只看 episodic/semantic）、`MAX_LEN`（截断）、`SHOW_FACES`。

### 5.5 复现方式（README 原样）

```bash
bash setup.sh
pip install git+https://github.com/huggingface/transformers@f742a644...   # 固定 commit
pip install qwen-omni-utils==0.0.4

# 1) 切 30s 片段（ffmpeg 脚本见 README:86-102）
# 2) 写 data/data.jsonl：{"id","video_path","clip_path","mem_path","intermediate_path"}
python m3_agent/memorization_intermediate_outputs.py --data_file data/data.jsonl   # 需 models/pretrained_eres2netv2.ckpt + speakerlab/
python m3_agent/memorization_memory_graphs.py       --data_file data/data.jsonl    # 需 HF ByteDance-Seed/M3-Agent-Memorization
python visualization.py --mem_path data/memory_graphs/robot/bedroom_01.pkl --clip_id 1

# control（需 HF ByteDance-Seed/M3-Agent-Control；transformers==4.51.0, vllm==0.8.4, numpy==1.26.4）
python m3_agent/control.py --data_file data/annotations/robot.json    # 输出 data/results/robot.jsonl
```

也可直接从 HF 下 `intermediate_outputs` 与 `memory_graphs` 跳过前两步；M3-Bench-robot 视频在 HF，web 视频按 `data/annotations/web.json` 的 `video_url` 自己下载。

### 5.6 可借鉴的代码级要点（对 OIA 直接有用）

1. **「中间输出层」与「模型层」拆成两个脚本**：`*_intermediate_outputs.py` 只跑确定性工具（人脸/说话人）并按 `clip_{i}_*.json` 落盘，`*_memory_graphs.py` 只做模型推理+组图；两边都**先查缓存再算**。这让昂贵的感知结果可复用、可单独替换（换人脸识别库不用重跑模型）。
2. **图即代码**：整个记忆是**一个可 pickle 的对象**，包含反向索引（`text_nodes_by_clip`）与派生视图（`character_mappings`）；`refresh_equivalences()` 每次读图前重算派生视图，而不是把派生结果写死。代价是 pkl 与类定义强耦合（`sys.modules` 别名 hack）。
3. **权重投票写成 `reinforce/weaken` 两个原语**，冲突消解只在**读取时**发生（`fix_collisions`、`get_entity_info` 的 `drop_threshold=0.9` 冗余剔除），写入路径保持简单。
4. **`before_clip` 因果截断**：一个整数字段就把「提问时刻之前不能看记忆」贯彻到检索层（`truncate_memory_by_clip` 同时清 `nodes/edges/text_nodes/text_nodes_by_clip/event_sequence_by_clip`）。这是做**在线/流式**评测最省事的机制。
5. **`translate` / `back_translate` 双向改写**：返回侧把实体 ID 归一成 `character_i`，查询侧把 `character_i` 展开成所有成员的查询变体（>100 条随机抽 100）。用最朴素的方式实现「以实体为中心的检索」。
6. **空结果也是信号**：检索为空时给模型一句固定反馈而不是空串（`control.py:130`），配合「query 必须与之前不同」的 prompt 约束，避免原地打转。
7. **批量多轮循环的写法**：所有样本共享轮次循环，`finish` 标记跳过已答样本，`multiprocessing.Pool` 并发执行检索——不引入 agent 框架也能跑多轮 tool-use。
8. **配置集中**：`processing_config.json` 一个文件同时充当「预处理参数 + 训练参数 + 推理参数」（`ckpt`、`model`、`total_round`、`batch_size` 都在里面），简单但耦合。

---

## 6. 摘要级理解的修正

初读只有一段摘要（`papers-abstracts.md` §6），全文与代码带来这些**摘要里没有或写反了**的机制细节：

| # | 摘要给人的印象 | 全文/代码的真实情况 | 出处 |
|---|---|---|---|
| 1 | 「M3-Agent, trained via RL」像是整个 agent 都用 RL 训 | **memorization 只做 SFT**（Qwen2.5-Omni-7B，3 epoch/1e-5），**只有 control 做 RL**（Qwen3-32B + DAPO）；且明说「概念上是一个模型，实际训了两个」 | §4.4 |
| 2 | 「entity-centric, multimodal format」不知怎么实现 | 是 **face_id/voice_id 持久锚点 + `Equivalence:` 语义节点 + 并查集合成 `character_i`**；不是预先定义的 schema，也不是 img-voice 边 | §4.2、§11.2、`videograph.py:417` |
| 3 | 「实时视觉听觉输入」多实时？ | 粒度是**固定 30s 切片**，5fps 抽帧，工具层先跑（InsightFace + Gemini 分割说话人），模型只看一个切片 | §4.4、§10、README |
| 4 | 「multi-turn, iterative retrieval」多轮长什么样 | **H=5 轮**、`Action: [Search]/[Answer]` 二值动作、`search_clip` 只回 **top-2** 且去重、最后一轮强制作答；还有一条专门查「ID↔人名」的旁路查询 | 算法 1、§5.1、`control.py` |
| 5 | 不知道训练数据怎么来 | 记忆标签是**三阶段合成**（GPT-4o→Gemini），等价关系靠 **meta-clip 投票字典**自动标；共 10,752 训练 + 200 验证样本，等价标注 10,952 片、抽检 95.83% | §11 |
| 6 | 不知道冲突怎么解决 | **边权投票**：相似度 >0.85 强化 +1、<0 削弱 −1；读取时 `fix_collisions` 每个 voice 只留最高等价边 | §4.1、`memory_processing_qwen.py:213-243` |
| 7 | 不知道 benchmark 怎么防泄漏/作弊 | robot 题目带 **`timestamp` + `before_clip`**，控制进程**只能看到提问前的记忆**；题目时间戳必须早于机器人回应 | §3.1、`control.py:117`、`robot.json` |
| 8 | 不知道评测谁打分 | **GPT-4o 当判官**（与人工 96% 一致），同时它也是 **RL 的 0/1 奖励来源**——训练与评测共用一个判官 | §3.3、式 1 |
| 9 | 摘要数字与本地 HTML 不符 | 摘要存档/README：**8.2%、920 web 视频**；HTML v1 摘要与 §1：**6.7%、929 视频**；robot QA：正文 1,344 vs 表 2 与代码 1,276 | §1、表 2、`README.md:15` |
| 10 | 不知道消融规模 | 记忆侧 5 行 + 控制侧 10 行 + 记忆模型 5 个 checkpoint 的 AutoDQ/Eq-F1 表，**全部单点数值、无方差** | 表 6、7、13 |
| 11 | 「long-term memory」像纯软件结构 | 人脸/声纹是**真实 base64 原始特征**存在图里（每节点最多 10 个脸 embedding / 20 个声 embedding），所以记忆体积随视频线性涨，论文自己承认空间推理需要「存快照」这种更重的视觉记忆 | 表 3、§5.5、§6 |
| 12 | 不知道 prompt 工程的对照 | 基线 GPT-4o control 用的是**带 retrieval plan、十几条 query 规范的长 prompt**，RL 后的 M3-Agent 只用 3 段极简 prompt——**RL 吃掉了 prompt engineering** | 表 21/22、`control.py:79` |

---

## 7. 对 OIA 的借鉴更新

### 7.1 实体中心记忆图：对多模态记忆的价值

- **它解决的是「同一实体跨模态、跨时间的身份漂移」**，而不是「存得更多」。M3-Agent 的做法有三层，可直接映射到 OIA 的多模态记忆：
  1. **持久原始锚点**：人脸图/声纹以 base64 + embedding 存节点，检索靠 0.3/0.6 阈值匹配——比「a man in a blue shirt」这类文本描述稳定得多（§4.2 的原话就是批评这类描述 inherently ambiguous）。
  2. **符号化中间层**：模型只用 `<face_i>/<voice_j>` 指代，**禁止代词和名字**，把「指代消解」从生成任务降级为格式约束（表 11/12 的 Strict Requirements）。
  3. **等价关系作为一等公民**：`Equivalence:` 行 + 并查集，读取时统一成 `character_i`；写入时用权重投票吸收偶发错误。
- **对 OIA 的具体启示**：多模态记忆节点应显式区分「原始观测（可回放）」「实体（跨模态身份）」「派生结论（可被推翻）」三类，且**冲突消解放在读取路径**（投票/阈值），写入路径保持 append-only。M3-Agent 的 `reinforce/weaken + fix_collisions` 是最小可行实现（约 100 行）。
- **代价要认清**：这套机制依赖**能用的外部感知工具**（人脸、说话人）。在人脸不可用的场景（隐私敏感、非人场景、屏幕共享），等价关系会退化——OIA 若迁移到文档/屏幕实体，需要的是「同一对象的跨表示锚点」（例如同一文件的路径、同一 URL 的 favicon、同一人的 email），**机制可迁移、锚点类型必须换**。

### 7.2 后台 memorizer subagent

M3-Agent 的 memorization 天然就是**一个不打断主循环的后台进程**：它有自己的模型（7B MLLM）、自己的输入（切片 + 感知中间结果）、自己的产出（追加到共享图）、**不需要知道当前指令**。这正好是 OIA 里「后台 memorizer subagent」的原型，且论文给了三条可复用设计：

1. **与 control 完全解耦的双模型**：memorization 要多模态理解（Qwen2.5-Omni），control 要推理（Qwen3）；**不要用一个模型兼任**（§4.4 明确说概念上一个、实践上两个）。对 OIA：后台记忆 agent 与前台执行 agent 应允许不同基座、不同上下文预算。
2. **记忆生成的输出契约**：两条列表（atomic episodic / distilled semantic）+ 实体 ID 引用 + 等价行，JSON 出参 + `validate_and_fix_json` 重试。契约里最重要的两条约束是「**一句一个原子事件**」和「**语义记忆不许复述表面观察、不许写推理步骤**」（表 12）——这直接决定记忆可检索性。
3. **成本结构**：后台只跑 7B + 一次 embedding（`text-embedding-3-large`），感知工具是本地的（InsightFace/ERes2NetV2）+ 一次 Gemini 分割调用；**每个 30s 切片只调 1 次生成模型**。这个「切片级一次调用」的预算是可估算的，OIA 设计后台 memorizer 时可以照抄这个粒度。

同时要注意 M3-Agent **没有**做的：记忆没有遗忘/压缩/合并策略（只有 `prune_memory_by_node_type` 这种消融用的粗删），没有任务驱动的选择性注意（论文自己把它列为未来工作，§5.5）。OIA 若要长期运行，这三块需要自己补。

### 7.3 M3-Bench 评测方法论（可迁移的部分）

**可迁移的骨架**
1. **把 benchmark 当「agent 输入流」而不是「视频问答集」**：视频 = 感知流，QA = 在指定时刻发出的指令；`timestamp` + `before_clip` 让评测具备**因果性**（不能用未来信息回答过去的问题）。这一条对任何流式/主动式 agent 评测都成立，与具体视频场景无关。
2. **题型按「记忆推理能力」而非「视频内容类别」划分**：multi-detail / multi-hop / cross-modal / human understanding / general knowledge——这是**关于记忆组织方式的分类**，可以直接搬到文档记忆、会议记忆、屏幕操作记忆上。
3. **问题必须绑定 reference events**（robot 脚本要求每题标注依赖哪些事件），这让错误可以归因到「漏看了哪段」而不是笼统的答错。
4. **双向校验**：GPT-4o 判官与人工 96% 一致（100 条三元组、3 人多数票）；同时报告**人类上限**（robot 90.7%，最常见错因是计数）。这两条是低成本但很硬的可信度证据。
5. **红线清单**（附录 §9）可直接抄：不能只靠音频、视觉信息 ≤720p 可辨、所需上下文 >10s、禁止把台词改写成问题、Yes/No 平衡、每视频至少 2 人物属性题 + 2 常识题。
6. **与既有基准的对比表**（表 2）用四个布尔维度（Agent Present / Cross-Modal / Human / Knowledge）说明「为什么需要新基准」——这是很有效的动机呈现方式。

**边界：「视频场景不迁移，但组织方式可迁移」**

- **不迁移的部分**：机器人第一人称 30 分钟剧本、人脸/说话人身份等价、VideoMME-long 上的绝对分数、以及整条「30s 视频切片 + Gemini 分割说话人」的流水线——这些都绑定在**视觉+语音的物理场景**。OIA 处理的是屏幕/文档/工具调用，直接照搬既无数据也无意义；M3-Bench 的 920+100 条视频与 6,000+ QA 也**不能**当成 OIA 的评测集。
- **可迁移的部分**：①因果时间戳机制（`before_clip`）；②按记忆推理能力分型的题型体系；③「实体中心 + 原子情节 + 蒸馏语义 + 等价关系」的记忆组织三件套；④判官与人工对齐的校验流程；⑤红线清单与 reference-event 标注法。
- **迁移时必须替换的锚点**：face/voice → 文件/URL/人名/工具名等 OIA 实体；`Equivalence:` → 「同一实体的多种表示/别名」等价行；ERes2NetV2/InsightFace → 屏幕 OCR、DOM/元素指纹等确定性工具。**机制可复用，感知工具与数据完全不可复用。**

---

## 8. 诚实边界

1. **版本不一致（最重要）**：本地 HTML 是 arXiv **v1（2025-08-12）**，摘要写 **6.7% / 929 视频 / robot 1,344 QA**；`papers-abstracts.md` §6（arXiv API 抓取）与 `README.md:15` 写 **8.2% / 920 视频**。仓库 `figs/exp_result.png` 的 M3-Agent 行与 v1 表 5 逐格一致（30.7/48.9/61.8），但**个别基线格子不同**（MA-LMM VideoMME-Long 7.2 vs 17.3、MovieChat 19.4 vs 19.5），说明图与摘要可能来自较新版本；**8.2% 在本地任何材料中都无法验证**，未在全文中找到支撑。本笔记正文数字一律以 **HTML v1** 为准。
2. **同一段落内口径打架**：§5.3 说 robot 上比「最强基线 MA-LMM」高 6.3，摘要与 §1 说比 Gemini-GPT4o-Hybrid 高 6.7（两者都是真的，只是「最强基线」定义不同）；§5.4 正文写「fixed the control model to **control-7b-rl**」，但表 6 标题写 `control-32b-rl`，且 control 侧并无 7B 模型（只有 8B/14B/32B）——**未在全文中找到 control-7b 的任何其他说明**，判断为笔误。同段还有 `memory-gemini-7b`（应为 memory-gemini-prompt）。
3. **QA 总数三个版本**：§1 说 robot 1,344 / web 5,037；表 2 说 robot 1,276 / web 5,037；本地 `data/annotations/` 实测 robot **1,276**、web **920 视频 / 3,214 QA**。**web 的 5,037 与代码里的 3,214 差 1,823**，未在全文中找到解释（可能是完整版与发布子集之别）。
4. **所有数字均为论文自报，未复现**：主结果、消融、AutoDQ/Eq-F1、95.83%/95.5%/96%/90.7% 全部照录。无方差、无多 seed、无显著性检验；每格一个数。
5. **训练代码不在本仓库**：SFT 与 DAPO/GRPO 实现都在两个外链仓库（`hyc2026/*`），本次未读取；因此训练细节只到论文表 14 的超参粒度，**优化器、warmup、rollout 具体调度未在全文中找到**。
6. **仓库不能开箱即跑**：`configs/api_config.json` 只有 `base_url`/`api_key`，而 `mmagent/utils/chat_api.py:38-43` 与 `m3_agent/control.py:35-39` 都按 **`azure_endpoint` / `api_version`** 读取并构造 `AzureOpenAI`（`chat_api` 用 `try/except: pass` 静默吞掉）——照 README 只填 base_url 会直接 KeyError。另 `memory_processing_qwen.py` 读的键是 `video_descriptions`，prompt 模板写的是 `video_description`。**这两处不一致未验证是否在运行时被兜底逻辑修复。**
7. **论文表 3 的 `weight` 属性在代码中没有对应字段**：`Node` 只有 `id/type/embeddings/metadata`，权重实际是**边权**；`extra_data`（时间戳）在代码里是 `metadata['timestamp']`。schema 表述与实现存在落差。
8. **记忆图 pkl 未随仓库提供**（`.gitignore` 排除 `memory_graphs`/`intermediate_outputs`/`videos`），需从 HuggingFace 下载；**未在本地加载过任何 pkl**，§5.4 的结构描述来自 `videograph.py` 源码与 `visualization.py` 读取路径，不是对实际数据的观察。
9. **评测链路的两个共用风险**：①GPT-4o 既是**判官**又是**RL 奖励函数**，策略可能学会迎合判官的措辞偏好；②判官与被评模型（GPT-4o 基线）同源。论文只报告了判官与人工 96% 一致，**未报告对判官的敏感性分析**。
10. **M3-Bench-robot 不是真机器人**：由人戴头戴设备扮演（§3.1），因此「机器人视角」的第一人称运动、遮挡、操作噪声都被弱化；视频为演员按剧本表演，不是自然采集。web 视频来自 YouTube，**本地只有 URL，未下载、未核验可用性**。
11. **消融的归因有限**：`w/o semantic memory` 是读取时 `prune_memory_by_node_type('semantic')` 级别的粗删（代码里有该函数，但论文未说明消融是在训练时还是推理时做——**未在全文中找到**）；`w/o equivalence` 对应代码 `order_character()`（每个 face/voice 独立成 character），也未在论文中说明实现位置。
12. **本笔记未运行任何代码**：所有代码结论来自静态阅读（`videograph.py` 951 行、`retrieve.py` 445 行、`prompts.py` 1324 行、三个入口脚本与配置），未执行推理、未构造记忆图。

---

*笔记生成：基于 `2508.09736-m3-agent.html`（7,474 行）全文分段精读 + `repos/m3-agent/` 源码静态阅读 + `papers-abstracts.md` §6 对照。所有百分比与分数均为论文自报。*
