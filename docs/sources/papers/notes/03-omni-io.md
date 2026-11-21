# 03 · Omni-IO Skills: Harnessing Your Agent Omni-Native 精读笔记

> 输入：arXiv HTML 全文 `docs/sources/papers/2609.31847-omni-io.html`（331,299 字节）+ 开源仓库镜像 `docs/sources/repos/omni-io-skill/` + 摘要存档 `docs/sources/papers-abstracts.md` §3。
> 阅读方法：grep 章节标题/`alttext=` 定位后分段 read；代码仓库按 SKILL.md → orchestration/ → mcp/ → formats/ → 代表技能逐个读。
> 所有实验数字均为论文自报，未复现。

---

## 1. 论文速览

| 项 | 内容 | 出处 |
|---|---|---|
| 标题 | Omni-IO Skills: Harnessing Your Agent Omni-Native | HTML `<h1>` |
| 作者 | Yanlin Li, Mingyang Hao, Shengqiong Wu, Hao Fei, Mong-Li Lee, Wynne Hsu | HTML L213–223 |
| 单位 | National University of Singapore；University of Oxford | HTML L225 |
| 标识 | arXiv:2609.31847v1（2026-09-25） | 摘要存档 §3 |
| 类型/venue | arXiv preprint（技术报告体例，含代码链接与 BibTeX）。**会议/期刊标注：未在全文中找到** | 全文检索无 venue 字段 |
| 论文许可 | arXiv.org perpetual non-exclusive license | HTML L201–202 |
| 代码 | https://github.com/any2any-mllm/Omni-IO-Skill （正文首页给出，HTML L247） | 论文 p.1 |
| 仓库许可 | **未在仓库中找到**（镜像内无 LICENSE 文件，README 亦未声明） | 仓库文件清单 |
| README 定位 | "Harnessing your coding agent to become omni-native"；徽章：Agent Skills compatible、MCP stdio server、Python 3.10+、arXiv paper | `README.md` L8–18 |
| BibTeX | `@article{li2026omniio, ... journal={arXiv preprint arXiv:2609.31847}, year={2026}}` | `README.md` L380–387 |
| 一句话主张 | 不改宿主 agent 的推理核心，用「分层 Skills + 标准化多模态执行接口 + 依赖感知编排 + 持久 Asset Registry」把通用 agent 变成 omni-native | 摘要 |

---

## 2. 全文结构

正文 7 节 + 参考文献 + 2 个附录（LaTeXML 渲染，部分残缺）：

| 章节 | 内容 | 关键落点 |
|---|---|---|
| §1 Introduction | 动机（模态碎片化 vs 模型扩展成本）、核心问题「plug-and-play harness 能否让现有 agent omni-native 且不动推理核心」、四条贡献 | L255–317；19 Atomic + 2 Expert + 6 Scenario = 27 Skills（L282） |
| §2 Related Work | 2.1 Omni Foundation Models（自回归统一 vs 混合离散-连续）；2.2 Agent Harnesses and Skills（ReAct、MCP、SkillsBench、CUA-Skill、MMSkills） | L319–361 |
| §3 System Capabilities and Task Coverage | 7 种工件模态（text/image/video/audio/document/3D/code）+ 四个能力族（understanding/generation/reasoning/retrieval）+ 任务组合 | L363–443；**Figure 2 渲染失败（HTML L390 显示 `{forest}`）** |
| §4 Layered Architecture | 4.1 四层总览；4.2 Skill Entry（4.2.1 声明式表示与递归展开、4.2.2 三级技能）；4.3 MCP Tool Service；4.4 Provider and Configuration；4.5 Asset Registry | 式(1)(2)、Table 1 |
| §5 Dependency-Aware Orchestration | 5.1 运行时双流；5.2 DEG 表示与构造；5.3 图验证 / Wave 调度 / 失败处理；5.4 工具路由与参数注入；5.5 资产传播与跨轮复用 | 式(3)、Table 2（四 Wave 示例） |
| §6 Experiments | 6.1 设置与指标；6.2 主结果（Table 3）；6.3 两个定性案例 | L1315–1419 |
| §7 Conclusion | 重述 τ=100%、rel SQCS +47.95/+49.96、StS 100.00/99.78 | L1421–1430 |
| Appendix A | 27 技能完整目录（ID/Skill/Description 三列表格） | L1835–2060，**完整可读** |
| Appendix B | 代表性真实任务覆盖（Task / Skill ID(s) / 输入输出模态） | L2062–2109，**只有表头、表体为空** |

**HTML 渲染缺陷（影响取证，务必记牢）**
1. 「Appendix Contents」节为空（L1830–1833）。
2. §3 与 §6 中指向附录的交叉引用是 `ltx_missing_label`（L1146、L1330）——**UniM-90 的 90 条实例选择协议（selection protocol）附录在本 HTML 中不存在**。
3. Appendix B 表体缺失 → **「38 representative tasks」的任务清单：未在全文中找到**。
4. Figure 2（模态覆盖任务图）渲染为 `{forest}` 错误块。

图表清单：Fig.1 teaser、Fig.2 任务覆盖（坏）、Fig.3 四层架构、Fig.4 技能层级、Fig.5 运行时工作流、Fig.6 案例研究；Table 1 技能组合、Table 2 四 Wave DEG、Table 3 主结果。

---

## 3. 方法精读（四要素）

### 3.1 分层 Skills（§4.2）

**三级按「任务粒度/组合范围」而非模态划分**（§4.2 开篇）：

- **Atomic Skill**：最小可独立调用单元；单步请求可直接调用；需要外部多模态服务的走 MCP，code/Markdown 等由宿主原生执行（§4.2.2）。
- **Expert Skill**：面向**单一最终交付物**的完整专业生产流程（需求分析→任务规划→资产生成→最终组装→质量检查→局部修订）；缺陷只重做受影响组件，已验证中间资产复用（§4.2.2）。
- **Scenario Skill**：面向显式应用场景的**一组相互关联交付物**，给出 trigger 边界、典型输入、交付物选择规则与预期输出，**不规定固定媒体组合或固定步骤**；欠定时按场景惯例并向用户澄清（§4.2.2）。

**声明式表示（式 1，§4.2.1）**

```latex
s=\langle c_{s},I_{s},P_{s},O_{s},H_{s}\rangle,
```

其中 `c_s` 适用条件、`I_s` 所需输入、`P_s` 执行过程（可执行操作或调用其他 Skills 的工作流）、`O_s` 预期输出、`H_s` 与其他 Skills 的关系（用于向下展开）。要点：**输入/输出按语义描述，不绑定 provider**；宿主 agent 无需加载底层服务实现即可检查、选择、组合技能。

**选择与递归展开（§4.2.1 p2）**：请求先定任务上下文与交付物 → 命中场景加载 Scenario、单一专业交付物直取 Expert、自包含操作直接调 Atomic → 高层技能**递归展开直到步骤可执行**；共享输入/中间结果只表示一次；**终端任务（含宿主原生执行的任务）成为 DEG 节点**。

**Table 1 的组合关系（从 HTML SVG 单元格提取，行→原子技能）**

| 层 | 技能 | 展开到的 Atomic（ID） |
|---|---|---|
| E1 | Poster Design | A1, A6, A16 |
| E2 | Complex Video Production | A2, A3, A6, A7, A8, A9, A10 |
| S1 | Social-Media Post | A1–A10 |
| S2 | Office Documents | A3, A4, A12, A13, A14, A15 |
| S3 | Job Application | A4, A10, A12, A13 |
| S4 | Education Sharing | A1–A14 |
| S5 | Event Material | A6, A7, A8, A9, A10 |
| S6 | Game Asset | A6, A7, A11, A16 |

Atomic 目录（Appendix A，19 条）：A1 Image Understanding、A2 Video Understanding、A3 Audio Understanding、A4 Document Understanding、A5 3D Understanding、A6 Image Generation、A7 Video Generation、A8 Music Generation、A9 Sound-Effect Generation、A10 Speech Generation、A11 3D Generation、A12 PPT、A13 Word、A14 PDF、A15 Excel、A16 Code Generation、A17 Markdown Generation、A18 Web Search、A19 Web Browsing；E1 Poster Design、E2 Complex Video Production；S1 Social-Media Post、S2 Office Documents、S3 Job Application、S4 Education Sharing、S5 Event Material、S6 Game Asset。**19+2+6=27，与摘要一致。**

### 3.2 标准化多模态执行接口（§4.3、§4.4）

论文**没有给接口编号 schema 表**，只有文字契约（具体 JSON schema 在仓库 `formats/`，见 §5）：

- **上行（每个外部执行的 DEG 节点）**：`task type` + `self-contained prompt` + 可选 `params` + **依赖已解析的资产输入**；用 task type 选工具接口，Skill 定义不暴露 provider API 细节（§4.3 p1）。
- **下行（provider 响应 → 公共结果）**：`output type` + `local path` + `description` + `effective parameters`，再转交 Asset Registry（§4.3 p2）。
- **执行边界**：依赖专业多模态模型的走 MCP；code/Markdown 等由宿主原生完成，**两条路线遵守同一 DEG 依赖语义、输出同样被规范化为系统资产**（§4.3 p2；§5.4 p2）。
- **Provider 层（§4.4）**：每个可外执行 task type 绑定 `tool / provider / model / credential reference / default parameters / fallback policy`；Skill 只引用语义能力（如 image generation），换 provider 不改 Skill 描述与 DEG 结构；**有效参数 = provider 默认 + 节点级覆盖 + 前驱输出注入**（§5.4 p1）。

### 3.3 Declare Execution Graph：形式化定义（§5.2–5.3）

**图定义**（§5.2 p1）：`G=(V,E)`；节点 `v∈V` 是可执行任务**或已存在资产源**；有向边 `(u,v)∈E` 表示 v 依赖 u。**节点元组（式 3）**

```latex
v=\langle\texttt{id},\texttt{type},\texttt{prompt},\texttt{params},\texttt{depends\_on}\rangle.
```

- `id` 唯一标识节点；`type` 所需能力；`prompt` **自包含**操作描述；`params` 特化执行；`depends_on` 记录上游节点或资产引用。
- **一条边同时是控制依赖（前驱未完成下游不能启动）和数据依赖（前驱输出被下游消费）**（§5.2 p1 末）。
- 构造：由层级技能展开的终端任务构成；共享中间结果**只表示一次**并被所有消费者引用；历史资产用 `asset_source` 节点（含 `asset_ref`），**引用解析后即视为完成、不发任何生成调用**（§5.2 p2）。

**图验证（§5.3 p1）**：两条检查——① 每个依赖引用都能解析到某个节点或已注册资产；② 诱导子图无环。任一失败视为**规划错误，整图重建**而非带病提交。

**Wave 调度算法（§5.3 p2，原文语义逐条）**
1. 「pending 且所有前驱已完成」的节点构成下一个执行 Wave；
2. 同一 Wave 内节点互相独立，**并发派发**；
3. 无未解析前驱的节点进入**最早** Wave；其余节点进入「其所有直接前驱都已完成」的**第一个更晚** Wave；
4. **调度由声明的依赖决定，与模态无关**：互不消费的图像与音频生成可并发，图像条件化的视频节点必须等参考图。

**失败处理（§5.3 p3）**：节点失败 → 其 pending 后代被**取消**（输入不可得），**独立分支继续执行**；记录每个节点终态，**不回滚已完成节点的输出**。

**工具路由与参数注入（§5.4）**：按 `type` 选工具 → Provider 层解析 provider/model/凭据/默认参数/fallback → 有效参数合并（默认 + 节点覆盖 + 前驱输出）。上游完成后解析其输出本地路径按关系注入：**image→video 注入首帧，image→3D 注入参考图**；因为输入由图注入，**下游 prompt 不需要写路径**（§5.4 p2）。

**资产传播与跨轮复用（§5.5）**：节点成功 → 写盘 → 注册（MCP 输出自动注册，宿主原生文件显式注册）；轮内下游按 `asset_id` 解析；**跨轮**用 `asset_ref` 指代历史资产，其记录被**物化为新 DEG 中已完成的源节点**，立即对后代可用；**修订创建新记录并保留 source 引用，绝不原地更新**（§5.5 p2、p3）。注册只在写盘成功后发生；写入原子且用文件锁串行化（§5.5 p3）。

**Table 2（产品推广案例的四 Wave，§5.5 p4 + Table 2）**

| Wave | 节点 | 依赖与产出 |
|---|---|---|
| Wave 0 | Analyze product reference images | 抽取产品外观与视觉约束 |
| Wave 1 | Generate poster assets | 由产品分析生成海报视觉素材 |
| Wave 1（并行） | Generate video and sound effects | 并行生成视频与音效资产 |
| Wave 2 | Generate promotional video and poster | 汇合资产产出宣传视频与海报 |
| Wave 3 | Generate landing web page | 由宣传产出组装落地页 |

跨轮示例：用户再要「换落地页风格」→ 产品分析与宣传资产被物化为已完成 `asset_source` 节点，**只有落地页任务重跑**（§5.5 p4）。

### 3.4 Asset Registry 数据模型（§4.5）

**资产元组（式 2）**

```latex
a=\langle\texttt{asset\_id},\texttt{type},\texttt{subtype},\texttt{path},\texttt{description},\texttt{params},\texttt{turn\_id},\texttt{source\_asset_id}\rangle.
```

字段语义（§4.5 p1）：`asset_id` **全局唯一、与物理路径解耦的稳定引用**；`type` 大类、`subtype` 细类；`description`+`params` 保留生成上下文；`turn_id` 关联交互轮次；`source_asset_id` 记录修订/派生的**来源关系（provenance）**。

存储与并发（§4.5 p2）：对 MCP 工具产出与宿主原生产出**统一表示**；**append-only JSON registry，不原地更新**（多版本可追溯）；**写入原子化并用文件锁串行化**，并行节点/会话不会互相覆盖；通过 lookup/registration 接口把消费者与具体工作区路径解耦。

---

## 4. 实验精读：UniM-90

### 4.1 任务构成与设置（§6.1）

- **基准**：UniM-90 = 从 UniM（bib [12]，*UniM: a unified any-to-any interleaved multimodal benchmark*，bib L1549）中**固定抽取的 90 条实例**，覆盖 text/image/audio/video/document/code/3D 七模态及其代表性交错组合；子集抽取**独立于任何被测 agent 的原生模态能力**。
- **选择协议**：论文称「detailed selection protocol is provided in Appendix …」，但该附录交叉引用为 `ltx_missing_label`——**具体抽取协议：未在全文中找到**。
- **基线/被测**：GPT-5.6 Sol、Claude Sonnet 5 两个宿主 × 两种配置：
  - *Base Agent*：默认环境，保留系统内置 skills/tools，排除额外第三方扩展与任务定制；
  - *Agent + Omni-IO Skills*：再加载 Omni-IO 技能与其 MCP 工具服务，**其余内置技能、工具配置、prompt、执行预算全部不变**（§6.1 p1）。
  - 论文明确**只解释 within-agent 增益，不把跨 agent 分差当作模型能力排名**（§6.1 p1 末）。

### 4.2 指标定义（§6.1 p2）

- `τ` = **input-support rate**：agent 能完整接收并处理全部输入模态的实例比例。
- 绝对分 `X^abs`：只在**受支持子集**上计算；
- 相对分 `X^rel = τ · X^abs`：反映**完整测试集**上的表现（式为 HTML `alttext` 原文：`\mathcal{X}^{\mathrm{rel}}=\tau\mathcal{X}^{\mathrm{abs}}`）。
- 四个指标：**SQCS**（Semantic–Quality Coupled Score，语义正确性与生成质量联合）、**ICS**（Interleaved Coherence Score，交错多模态响应的整体连贯性）、**StS**（Strict Structure Score，严格输出结构一致性）、**LeS**（Lenient Structure Score，模态级覆盖）。
- 失败计分规则：**输入可处理但所需输出模态无法生成/目标产出无法完成的失败，仍计入指标**（§6.1 p2 末）。
- **「SSC」这一指标名：未在全文中找到**（本篇只有 SQCS/ICS/StS/LeS；若别篇笔记出现 SSC，需另行核对）。
- 各指标的具体计算公式：**未在全文中找到**（论文只说「adopt the UniM Evaluation Suite」，公式在 UniM 原文）。

### 4.3 主结果全表（Table 3，§6.2；Higher is better）

| Base Agent | τ | abs SQCS | abs ICS | abs StS | abs LeS | rel SQCS | rel ICS | rel StS | rel LeS |
|---|---|---|---|---|---|---|---|---|---|
| GPT-5.6 Sol | 40% | 67.49 | 86.53 | 47.84 | 72.22 | 26.99 | 34.61 | 19.14 | 28.89 |
| + Omni-IO Skills | **100%** | 74.94 | 93.98 | **100.00** | **100.00** | 74.94 | 93.98 | 100.00 | 100.00 |
| Claude Sonnet 5 | 38.89% | 71.53 | 82.29 | 52.21 | 68.57 | 27.82 | 32.00 | 20.30 | 26.67 |
| + Omni-IO Skills | **100%** | 77.78 | 83.28 | **99.78** | **100.00** | 77.78 | 83.28 | 99.78 | 100.00 |

**派生读数（论文自报的增益表述）**
- τ：40.00% → 100%、38.89% → 100%（摘要、§1、§6.2、§7）。
- rel SQCS：26.99 → 74.94（**+47.95**）、27.82 → 77.78（**+49.96**）（§1 p4、§7）。
- rel ICS：34.61 → 93.98、32.00 → 83.28（§6.2）。
- abs SQCS：67.49 → 74.94、71.53 → 77.78；论文特别指出**加载 harness 后全 90 条上的绝对分（74.94/77.78）高于 Base Agent 在其更窄受支持子集上测得的 67.49/71.53**（§1 p4）。
- 结构指标：GPT-5.6 Sol StS/LeS = 100.00/100.00；Claude Sonnet 5 = 99.78/100.00（§6.2）。
- 因为 τ=100%，**加 harness 后 abs 与 rel 两列完全相同**（Table 3 中可见）。

### 4.4 消融与其它实验

- **消融实验：未在全文中找到**（全文 grep `ablat`/`Ablation` 零命中；§6 仅 6.1 setup / 6.2 main / 6.3 case studies 三小节，无 ablation 表）。
- 也没有：显著性检验/多次运行方差、token 或费用成本、延迟/吞吐、人类偏好评审、与其它 harness（如纯 prompt 编排、无 Registry 变体）的对照——**均未在全文中找到**。
- **§6.3 定性案例（2 个）**：
  1. *Art tutorial generation*：调用 S4 Education Sharing 规划、A2 Video Understanding 抽绘画步骤、A3 Audio Understanding 恢复讲解与步骤顺序、A6 Image Generation 产出 **11 张图**教程；
  2. *Product promotion*：A1 Image Understanding 抽产品属性 → E1 Poster Design 设计海报 → E2 Complex Video Production 制作宣传片 → A16 Code Generation 建落地页。
  两个宿主都能产出全部请求输出并保持教程步骤/产品身份/交付物一致性（§6.3 p1）。

---

## 5. 代码研读（本篇重点：harness 工程）

仓库镜像：`docs/sources/repos/omni-io-skill/`（案例媒体已裁剪，剩 txt/js/html/json 提示词与配置）。

### 5.1 目录结构（README L145–158 + 实际文件清单）

```text
Omni-IO-Skills/
├── SKILL.md            # 入口与路由索引（Agent Skills 规范的 frontmatter + 分层索引）
├── skill.yaml          # 技能与 MCP 声明（stdio server + env 透传 + config 路径）
├── skills/             # 19 个原子技能
│   ├── understand/{image,video,audio,document,3d}.md
│   ├── generate/{image,video,3d,code,markdown}.md
│   ├── generate/audio/{music,sfx,speech}.md
│   ├── generate/document/{ppt,word,pdf,excel}.md
│   └── utility/{search,browse}.md
├── expert/             # 2 个专家技能
│   ├── poster-design/poster-design.md
│   └── video-producer/video-producer.md
├── scenarios/          # 6 个场景技能 + _template.md（模板不可触发）
│   ├── social_media_post.md / office.md / job_application.md
│   ├── education_sharing.md / event_materials.md / game_assets.md
├── orchestration/      # 编排协议（5 个 md）
│   ├── declare_protocol.md / dependency_rules.md / asset_registry.md
│   ├── expert_protocol.md / interruption.md
├── mcp/                # stdio MCP 服务器（Python 3.10+）
│   ├── server.py(485 行, FastMCP) / generate.py(910 行) / understand.py(599 行)
│   ├── utility.py / registry.py(119 行) / requirements.txt
├── formats/            # 内部任务声明格式
│   ├── declare_format.md / input_format.md / output_format.md
├── config/             # config.yaml(161 行, provider 绑定) + .env.example
├── setup/              # quickstart.md / api_guide.md / feature_list.md
├── examples/           # 20 个案例（PROMPT.txt + 部分 outputs）
└── assets/             # README 图片/视频
```

**技能数核对**：`skills/` 19 + `expert/` 2 + `scenarios/` 6（`_template.md` 不计）= **27，与论文 §1 一致**。

### 5.2 技能描述格式

**(a) 入口 `SKILL.md`（frontmatter + 索引 + 全局规则）**

```yaml
---
name: omni-io
description: >
  An omni-modal content generation and understanding skill set. Trigger it automatically
  when the user needs image generation, video production, ... dependency-aware DAG task
  scheduling (executing independent tasks in parallel), and cross-turn asset reuse.
version: 3.0.0
---
```

正文七段式：**First-Use Check**（首个会话多模态任务先 `check_config()`，展示 ✅/❌/🆓 能力清单，一会话只做一次）→ **Workflow**（7 步：`set_turn` 打轮次标签 → 按 `expert_protocol.md` 的路由顺序判定场景/专家/原子 → 专家先用一句话向用户说明所用流程 → 内部构造 `<declare>` 图（**不展示给用户**）→ 按 DAG 并行调度 → 专家交付物必检、**默认最多两轮增量返工** → 实时返回、最终摘要以完整交付物优先）→ **Skill Index**（三层索引，并写死单向调用约束：`Scenario → Expert → Atomic`，Atomic 不得反调）→ **Orchestration Rules**（5 个 orchestration/*.md 永远适用）→ **Global Rules**（9 条：先建图、并行、**生成前先读 registry.json 防重复**、**依赖不确定时宁可多声明**、实时返回、先查 key、结果按 `✅ Image generated: <path>` 格式呈现、专家不得引入新 task type、交付物必检）。

**(b) `skill.yaml`（技能 + MCP 声明）**

```yaml
name: omni-io
version: 3.0.0
entry: SKILL.md
mcp_servers:
  omni-io:
    type: stdio
    command: python
    args: [mcp/server.py]
    env:            # 按能力逐项透传 key，缺 key 不阻塞其它能力
      FAL_API_KEY: "${FAL_API_KEY}"
      WAN_API_KEY: "${WAN_API_KEY}"
      OMNI_OUTPUT_DIR: "${OMNI_OUTPUT_DIR:-~/Documents/OmniIO}"
config: config/config.yaml
```

即：**宿主只需 symlink 技能目录 + 注册一个 stdio MCP**（README Quick Start 的 Codex/Claude Code 命令），这正是论文「plug-and-play、不改宿主推理核心」的工程形态。

**(c) 原子技能文件的固定小节**（以 `skills/generate/image.md` 为例）
`Trigger`（触发条件 = declare 中出现 `type:"image"`）→ `Tool`（默认 fal FLUX.1-schnell / 备选 OpenAI gpt-image-2，指向 `config.yaml → generate.image`）→ `Inputs`（字段/来源/说明三列表）→ `Procedure`（4 步：解析参数→调 API→落盘→写 output_format 并注册）→ `Output Format`（JSON 模板）→ `Defaults` → `Integration with Other Skills`（下游 video/3D 自动注入首帧/参考图）→ `Error Handling`（key 缺失/内容策略/超时三行表）。

理解类技能的差异（`skills/understand/image.md`）：**宿主原生视觉、不调 MCP**；输入来源四行表（上传/URL/本地路径用 Read 工具/本轮 `asset_id` 先 `get_asset`）；「**纯理解请求不输出 `<declare>` 块**」；输出分三形态（问答段落 / 结构化列表 / 可拼进生成 prompt 的 style description）。

**(d) 场景技能模板 `scenarios/_template.md` 的 10 节**：Scenario Description / Trigger Examples / **Trigger Boundary（Trigger when / Do not trigger when / When uncertain）** / Typical Input Assets / **Media and Deliverable Selection Rules（核心：按「用户明说的模态 → 场景惯例 → 每个交付物选 Expert 还是原子」三级判定，不预设固定包）** / Expert-Atomic Selection Rules（表格）/ Non-Tool Outputs（**纯文本不进 declare、不注册**）/ Reference `declare` Block（示例非固定模板）/ Expected Outputs / Extensions。

**(e) 专家技能 `expert/poster-design/poster-design.md`（142 行，专业闭环的范本）**
12 个小节：是否该用本 Expert 的**正反判据**（普通照片/壁纸直接走 `generate/image.md`）→ **locked brief**（原文拼写/日期/价格/CTA 视为锁定，**绝不让图像模型负责文字准确性**）→ internal visual philosophy（决策系统而非给用户看的散文）→ **最小资产集**（能复用就不生成；每个 image 任务要求「无任何文字/伪文字 + 预留排版负空间」）→ **展开为既有原子任务的扁平 fragment**（示例 JSON：`poster_bg` type image + `poster_final` type code，`depends_on: poster_bg`；**禁止 `type:"expert"/"poster"`**）→ 组装（走既有 `code` 路径，确定性排版，中文字体，PNG 默认/PDF 可选，最后 `register_asset(asset_type="image", subtype="poster")`）→ **交付前用原生视觉逐项检查 8 类问题**（文案/层级/版式/可读性/视觉/品牌/输出规格/工艺）→ **只修坏层**（四类失败→最小增量计划表；**全系统无原地更新，重生成 image 必得新 asset_id，重组装前必须把 code 任务的 `depends_on` 与 `get_asset()` 解析路径改指新 id，否则会静默复用旧的失败资产**）→ 最多两轮自主返工 → 返回时**优先呈现最终交付物**，被场景调用时把最终海报作为 Expert 输出端点供其它交付物复用（如视频首帧）。

### 5.3 执行图（orchestration/）

**`orchestration/declare_protocol.md`**（116 行）
- `<declare>` = **宿主 agent 在任何生成任务前构建的内部执行图**，JSON 结构：
  `{"tasks":[{id, type, prompt, params, depends_on, asset_ref}]}`；纯理解请求不建图。
- Wave 调度四步：**Wave 0 = 无 `depends_on` 的任务（含 `asset_ref` 视为已完成）→ Wave N = 直接依赖都在更早 Wave 完成 → Wave 0 的所有 MCP 调用必须在同一个 tool-call turn 内并发发出（依赖宿主的并发工具调用能力）→ 逐波推进**；上游结果的 URL/`asset_id` 自动用于下游参数。
- 例外：`type:"code"/"markdown"` 仍是波内普通节点、照常等依赖，但执行改为**宿主原生 Write/Edit 写文件后调 `register_asset()`**。
- 错误表：单任务失败→标 failed 并继续独立任务、取消其下游；`asset_ref` 不存在→询问是否重生成；自检发现环→告知用户并重规划；key 缺失→标 failed。
- 附一张 ASCII 时序图（intent → 建图 → 拓扑分波 → 并行 → 注册 → 展示）。

**`orchestration/dependency_rules.md`**（175 行，两半：声明原则 + 自动注入）
- **核心原则「保守声明」**：缺依赖 = 下游读到空/旧资产 → 高成本错误；多余依赖 = 只损失并行度 → 低成本。**宁多勿漏。**
- 必须声明的三类：内容引用（"用上图作首帧"）、格式/媒体依赖（image-to-3D 的参考图 URL）、**强语义一致性**（下游内容须贴近上游实际产出，事前写的 prompt 可能与实际输出不一致）。
- **注入表**：`image → video` 注入上游图 URL 作为 `first_frame_url`；`image → 3d` 注入 `reference_image_url`；「**不要在 prompt 里手传 URL 或 asset_id**」；本地路径由 Wan/Kling 内部转 base64 data URI。
- **多依赖数组仅 `code`/`markdown` 合法**（其它类型是固定参数签名的单次 MCP 调用），触发是 AND 逻辑；数组是**调度指令，禁止原样传给 `register_asset()`**（后者 `depends_on` 是单字符串、含义为版本血缘）。
- 上游 failed → 下游全部 `cancelled`，原因写进 `output_format.failed`。
- 4 个图例：线性链 / 扇出（Wave 0: img1, mus1 → Wave 1: vid1, 3d1）/ 混合 / 多依赖汇聚到 code。

**`orchestration/asset_registry.md`**（144 行）
- 注册表 = `$OMNI_OUTPUT_DIR/registry.json`；**所有读写必须走 MCP 工具，宿主绝不直接读写该 JSON**；写入用临时文件 + rename 原子化，macOS/Linux 再加 fcntl 文件锁；**Windows 无 fcntl 跳过锁但保留原子写**。
- 三大用途：跨轮引用（"把上一张图变 3D"）、**生成前查重避免重复生成**、**上下文压缩**（历史资产以 `asset_id` 进入规划，不占上下文）。
- 记录示例（字段比论文式(2) 多了 `url`/`params_used`/`completed_at`/`status`）：
  `{"asset_id":"img_a3f2c1","type":"image","url":"/Users/.../img_a3f2c1.png","description":"...","params_used":{"aspect_ratio":"16:9"},"turn_id":"turn_20250617_1437","completed_at":1749520015}`；音频带 `subtype`（music/sfx/speech）。
- `asset_id` 格式 `{prefix}_{6位hex}`；**生命周期表**：每轮开始 `set_turn("turn_YYYYMMDD_HHmm")` → 生成工具返回即自动注册 → **code/markdown/Expert 成品必须手动 `register_asset()`，否则成孤儿、后续 `asset_ref` 找不到**（且 Expert 成品要按**真实模态**注册：海报用 `image`+`poster`，不是 `code`）→ 历史资产用 `list_assets()`（按完成时间倒序）+ `get_asset()` → 跨轮复用在 declare 里写 `asset_ref`，该节点**不发 API 调用**。

**`orchestration/expert_protocol.md`**（186 行）：单向层级与路由顺序（场景→专家→原子，**不因覆盖某模态就过度触发专家**）；可观测路由（首次生成前用一句话说明流程，这是手工触发测试的主要观测信号）；Expert 输入契约与**展开契约**（只能用 `declare_format.md` 的 12 种类型，禁止 `expert/poster/scene/assemble`；产出**扁平 fragment，由根技能与场景的其它原子任务合并成一张图**；多专家场景用 `poster_bg`/`promo_final` 这类可区分 id）；**组装节点 = 既有 `type:"code"`**（海报走排版、视频走本地 FFmpeg 合 MP4），**本阶段不新增永久脚本/MCP 工具/原子技能**；**审查循环**：交付前必须用对应理解能力检查（海报→原生视觉，视频→`understand_video`），把问题归到最小责任层，**增量重做、复用已验证资产、默认最多两轮**，两轮后把决定权交回用户。

**`orchestration/interruption.md`**（157 行，论文未展开的工程细节）：五类中断及流程——①单任务局部改（按 not started/running/completed 三分支处理，并检查下游是继续等还是询问是否重生成）②全局风格/场景改（全部 running 取消并重写 prompt 重启；已完成的默认不重生成）③加任务（**增量插入现有 DAG**，id 顺延）④取消任务（取消信号、下游标 cancelled、**不写注册表**）⑤换话题（先确认，清空本轮 DAG，**已完成资产保留在注册表**）。附「语义相关性检查」三值判定（受影响/不受影响/不确定→保守继续或询问）与 UX 原则；**本质是对现有 DAG 的动态重规划，而非整体丢弃**。

### 5.4 MCP 接口层（`mcp/`）

`mcp/server.py`：`FastMCP("omni-io", instructions=...)`，启动时 `load_dotenv(config/.env, override=True)`。**共 21 个工具**（`grep -c '@mcp.tool'` = 21：4 理解 + 10 生成 + 2 工具类 + 1 配置自检 + 4 注册表）：

| 组 | 工具 |
|---|---|
| Understanding（4） | `understand_video`、`understand_audio`（Gemini 优先，缺 key 按 subtype 回退 Whisper/Qwen2-Audio）、`understand_document`（Mistral OCR 3，正文 ≤4,000 字 Markdown）、`understand_3d`（trimesh 元数据 + Blender Cycles 渲染，失败回退 matplotlib，并**如实告知用户回退渲染没有材质**） |
| Generation（8） | `generate_image(prompt, aspect_ratio, quality)`、`generate_video(prompt, duration_seconds, aspect_ratio, first_frame_url)`、`generate_music`、`generate_sfx`、`generate_speech(text, voice, language, speed)`、`generate_3d(prompt, reference_image_url)`、`generate_ppt/word/pdf/excel(structure, style)`（**本地库渲染，MCP 不调 LLM，内容由宿主先规划**） |
| Utility（2） | `search`（Brave）、`browse`（Playwright，等 network idle，过短则滚动重试） |
| Config（1） | `check_config()`（读 config.yaml + 环境变量，返回 active_providers/ready/missing/always_free/notes） |
| Registry（4） | `get_asset(asset_id)`、`register_asset(asset_type, url, description, params_used, subtype, depends_on)`、`list_assets()`、`set_turn(turn_id)` |

所有 `generate_*` 都是 `return registry.register(**_to_registry_kwargs(result))`——**生成与注册在同一调用里原子完成**。`register_asset` 的 docstring 明确：Expert 成品（海报/成片）虽内部走 code 任务，**`asset_type` 必须写真实结果模态**；`depends_on` 只记录单个前置版本的血缘，**每次调用都产生新 asset_id（无原地更新）**。

`mcp/registry.py` 关键实现（全文 119 行，可整读）：

```python
asset_id = f"{prefix}_{uuid.uuid4().hex[:6]}"     # prefix_map: image→img, video→vid, 3d→3d;
                                                   # audio: music→mus, sfx→sfx, speech→tts
record = {"asset_id":..., "type":..., "status":"completed", "url":...,
          "description":..., "params_used":..., "turn_id": self._current_turn,
          "completed_at": int(time.time())}
with open(self._lock_path, "w") as lock_file:
    if fcntl: fcntl.flock(lock_file, fcntl.LOCK_EX)   # Windows 跳过锁
    store = self._load(); store[asset_id] = record; self._save(store)   # 锁内重读再写
```

`_save` 用 `tmp + os.replace` 原子替换；`list_all()` 按 `completed_at` 倒序；**全局单例 `registry` 供整个 MCP 进程共享**；文档注释直白承认「**Windows 并发进程可能丢记录**」。

`mcp/generate.py` 的 provider 分发形态（可直接抄的模式）：

```python
async def generate_image(prompt, aspect_ratio="1:1", quality="standard"):
    cfg = _provider_cfg("generate", "image")
    if cfg["provider"] == "openai": return await _image_openai(...)
    if cfg["provider"] == "fal":    return await _image_fal(...)
    raise ValueError(...)
```

配套 `_auto_describe_image()`：**生成完用 gpt-4o-mini 给图写 ≤80 词英文描述**（无 `OPENAI_API_KEY` 时退化为 `"Image generated: <path>"`）——这条 description 正是注册表里供后续轮次语义检索的字段。

`config/config.yaml` 的绑定形态：`understand.{video,speech,audio_analysis,document,3d}` 与 `generate.{image,video,music,sfx,speech,3d,document}` 各有 `provider:` 选中项 + `providers:` 备选项（如 image: `fal|openai`、video: `wan|kling`、music: `elevenlabs|huggingface|replicate`、3d: `tripo|meshy`、speech: `edge_tts|openai|elevenlabs`、document: `local`）+ `defaults:` 段。**注释里直接标了价格与免费额度**（FLUX≈US$0.003/图、gpt-image-2≈US$0.04/图、Edge TTS 完全免费等）。

### 5.5 工件格式定义（`formats/`）

- **`declare_format.md`**：12 种 `type`（image/video/sfx/music/speech/3d/ppt/word/pdf/excel/code/markdown）；id 命名约定（`img1/vid1/sfx1/mus1/tts1/3d1/ppt1/code1/md1`）；**prompt 必须自包含**（并行任务的 prompt 写在同一规划上下文里以天然保持语义一致）；`params` 覆盖 `config.yaml` 默认（aspect_ratio/duration_seconds/voice/language/instrumental）；`depends_on` 单字符串 vs 数组的合法性边界；`asset_ref` 与 `prompt` **互斥**。
- **`input_format.md`**：`{instruction, assets[], history_assets[]}`；asset 字段 `type/asset_id/source/description` + 按模态可选 `subtype/transcript/timeline/content/mesh_info`；`source` 区分 `user upload` 与 `generated in previous turn`；各模态示例（视频给 `timeline` 分段、文档给 Markdown 正文、3D 给 mesh_info）。
- **`output_format.md`**：`{turn_id, assets[], failed[]}`；asset 字段 `task_id/type/asset_id/status/url/description/subtype/params_used/depends_on/completed_at`，**`status ∈ completed|failed|cancelled`，`url` 是 `OMNI_OUTPUT_DIR` 下的绝对本地路径而非 HTTP URL**；结尾说明与注册表的关系：**只有 `completed` 写入注册表，failed/cancelled 不写**。

### 5.6 可借鉴的代码级要点（含与 DSH 的对位）

**直接可抄的工程做法**
1. **入口三件套**：`SKILL.md`（frontmatter + 分层索引 + 9 条全局规则）+ `skill.yaml`（stdio MCP 声明 + 逐项 env 透传 + config 路径）+ `_template.md`（场景技能的作者模板，含 Trigger Boundary 三段式）。
2. **协议与技能分离**：`orchestration/*.md` 是「永远适用」的协议层，技能文件只写本技能差异——协议改一处全局生效。
3. **内部图不外露**：`<declare>` 只给模型自己看，用户只看到一句自然语言的流程说明（专家路由的可观测信号）。
4. **保守依赖原则**：宁可多声明一条边损失并行度，也不漏边产出错误结果——一句话可写进任何编排 prompt。
5. **注入而非口述**：上游产物由调度器注入具名参数（`first_frame_url`/`reference_image_url`），prompt 里禁止出现路径，实现「图驱动的参数解析」。
6. **注册与生成同调用原子完成** + 原生写盘文件**强制手动注册**（并明确「漏注册=孤儿」的后果描述）。
7. **append-only + 原子写 + 文件锁 + 锁内重读**的最小持久化注册表实现（`registry.py` 全文仅 ~119 行）。
8. **无原地更新的版本血缘**：每次修订产生新 id，`source/depends_on` 记血缘；并显式警告「重组装前必须改指新 id，否则静默复用旧资产」。
9. **专家审查闭环**：交付前必检 → 问题归到最小责任层 → 增量重做 → **默认 ≤2 轮**后交还用户。
10. **`check_config` 能力自检**：会话首个任务先报 ready/missing/free，缺 key 只标失败不阻塞其它模态。
11. **动态中断五分类**（局部改/全局改/加任务/取消/换话题）+ 语义相关性三值判定。
12. **本地零成本能力优先**：PPT/Word/PDF/Excel 本地库、Edge TTS、Playwright、宿主原生视觉/代码——`setup/feature_list.md` 把「免 key」与「需 key」两栏列清，降低上手摩擦。

**与 DSH 现有机制的对位**（依据本机 DSH 文档 `dsh-doc/references/packages.md`，未逐一运行验证）

| Omni-IO 概念 | 文件 | DSH 对位 | 关键差异 / 迁移要点 |
|---|---|---|---|
| 技能目录 + 入口索引 | `SKILL.md`、`skills/**`、`expert/**`、`scenarios/**` | 会话技能目录 + `skill` 工具；包 `@deepseek-ai/dsh-skill`（注册表）、`@deepseek-ai/dsh-skill-filesystem`（本地文件系统技能提供者）、`@deepseek-ai/dsh-skill-office`（内置 Office workflow） | DSH 技能是「按需加载的指令包」，同样有触发描述与懒加载；Omni-IO 额外提供了**三层粒度 + 单向调用约束**与「场景模板」，可作为 DSH 技能的组织约定引入 |
| `<declare>` DAG + Wave 调度 | `orchestration/declare_protocol.md`、`dependency_rules.md` | `workflow` 工具（JS 编排：`agent/pipeline/parallel/phase`）；包 `@deepseek-ai/dsh-workflow-ptc`（共享沙箱 Node PTC 运行时中的 workflow 编排） | **执行主体不同**：DSH workflow 由脚本驱动子代理 fan-out；Omni-IO 的 DEG 由**宿主模型自己在 prompt 内**做拓扑分波并依赖宿主的「同一回合并发工具调用」。OIA 若在 DSH 内实现，宜把 DEG 解析成 `parallel(thunks)`/`pipeline(...)` 落差，或保留 prompt 内调度并用 wave 语义校验 |
| Asset Registry | `orchestration/asset_registry.md`、`mcp/registry.py` | 附件存储 `@deepseek-ai/dsh-attachment-local`（**私有、内容寻址、DSH_HOME 存储**）+ 展示插件 `@deepseek-ai/dsh-client-ui-attachment`；交付物侧 `present` 文件卡片 | **方向相反**：DSH attachments 主要管**输入侧**附件的内容寻址与呈现；Omni-IO registry 管**输出侧**资产的业务索引（`turn_id`/`description` 语义检索/`source_asset_id` 血缘/可枚举 `list_assets`）。两者互补，OIA 需要的是后者这类「可枚举、可跨轮引用、带血缘」的生成物注册表 |
| Provider 层 | `config/config.yaml`、`mcp/generate.py` | DSH 的 MCP 连接（配置型 bundle，`references/mcp-bundle.md`）与插件/预设配置 | Omni-IO 的「一个 task type ↔ provider/model/key/默认参数/fallback」绑定表 + `check_config()` 自检，可直接作为 OIA 的 provider 配置 schema |
| 中断处理 | `orchestration/interruption.md` | DSH 会话 steering/取消（停止键 SIGHUP 语义） | Omni-IO 的「增量改 DAG 而非整体重规划」+ 五分类是可直接写进 OIA 编排 prompt 的规则集 |
| 能力自检 | `mcp/server.py → check_config()`、`setup/feature_list.md` | DSH 会话可用工具清单 / 技能目录 `available_skills` | 把「ready / missing(带价格与替代项) / always_free」三栏做成会话首任务的一次性汇报，是低成本的可用性设计 |

---

## 6. 摘要级理解的修正（对照 `papers-abstracts.md` §3）

摘要原文（存档 §3）与全文逐项核对结果：

| # | 摘要级印象 | 全文精读后的修正/补充 |
|---|---|---|
| 1 | 「hierarchical Skills、standardized multimodal execution interface、dependency-aware orchestration、persistent Asset Registry」四件套 | 四件套分别落在**四层架构**上：Skill Entry / MCP Tool Service / Provider and Configuration / Asset Registry（§4.1）。摘要没给层名，也没说**四层的职责切分是「任务知识 / 工具接口 / 服务实现 / 持久产出」**——这是理解「为什么换 provider 不用改技能」的关键 |
| 2 | 「Declare Execution Graph」只是一句名词 | 全文给了**完整形式化**：`G=(V,E)`、节点元组式(3) `v=⟨id,type,prompt,params,depends_on⟩`、`asset_source` 节点（解析即完成、不发调用）、两条验证（引用可解析 + 无环，失败即**整图重建**）、Wave 调度四条规则、失败「取消下游 + 独立分支继续 + 不回滚」（§5.2–5.3）。**摘要完全没提失败语义与验证语义** |
| 3 | 「register successful outputs for downstream and cross-turn reuse」 | 具体机制是：**append-only JSON（不原地更新）+ 原子写 + 文件锁**，式(2) 八字段；跨轮时 `asset_ref` 被**物化为新图中已完成的源节点**；修订产生新记录并留 `source_asset_id` 血缘（§4.5、§5.5）。摘要未提「**只在写盘成功后才注册**」与并发写保护 |
| 4 | 「27 Skills cover 38 representative tasks」 | 27 可在仓库**逐一点数核对**（19 Atomic + 2 Expert + 6 Scenario，附录 A 完整）；**38 个任务的清单：未在全文中找到**——Appendix B 只剩表头，表体为空 |
| 5 | 「UniM-90」 | 摘要没说的是：它是 **UniM 的 90 条固定子集**、覆盖 7 模态 + 交错组合、**抽取独立于被测 agent 原生能力**；而**选择协议附录在 HTML 中缺失**（交叉引用为 `ltx_missing_label`）→ 协议本身**未在全文中找到** |
| 6 | 指标口径 | 摘要只报 SQCS 与 StS。全文还报 **ICS、LeS**，并区分 abs/rel 两列，给出 `X^rel = τ·X^abs`；且明确**输出模态不可生成的失败仍计入**。加 harness 后 τ=100% ⇒ abs 与 rel 两列相等（Table 3 可见） |
| 7 | 摘要数字 | **全部与全文一致**（τ 40.00%/38.89%→100%；rel SQCS 26.99→74.94、27.82→77.78；StS 100.00/99.78）。全文另给 +47.95/+49.96 的增益表述（§1、§7）与 ICS/LeS/abs 全表 |
| 8 | 摘要暗示「做了评测即有消融」 | **消融：未在全文中找到**（无 ablation 章节/表）；亦无显著性、成本、延迟、人评——§6 只有 setup / 主结果 / 两个定性案例 |
| 9 | 「replaceable execution backends」 | 工程上就是 `config/config.yaml` 里每个能力的 `provider:` 单选 + `providers:` 备选（fal/openai、wan/kling、elevenlabs/hf/replicate、tripo/meshy、edge_tts…）+ `check_config()` 自检；换后端只改 YAML（§4.4 的实现形态） |
| 10 | 「plug-and-play」 | 具体动作 = **symlink 技能目录到宿主 skills 路径 + 注册一个 stdio MCP（`skill.yaml` 的 `mcp_servers`）**，不动宿主 prompt/预算（README Quick Start、§6.1） |
| 11 | 摘要的「SSC」？ | 摘要与全文**均无 SSC 指标**（只有 SQCS/ICS/StS/LeS）；任务提示中的「SSC」应是别篇笔记的指标，本篇不适用 |
| 12 | 摘要未提的工程约束 | 单向层级（Scenario→Expert→Atomic，禁止反调/平行互调）、专家**默认最多两轮**增量返工、**全系统无原地更新**（重生成必得新 asset_id，重组装前必须改指新 id）、五类中断处理、`code/markdown` 是唯一合法的多依赖类型——这些都在仓库 `orchestration/*.md`，论文正文只零散提到「不回滚」「≤2 轮」 |

---

## 7. 对 OIA 的借鉴更新

> 前提说明：OIA 仓库当前是「资料 + 博客」形态（`blog/`（posts/01–06 静态页）、`docs/sources/`（论文 HTML、仓库镜像、笔记）），**尚无 harness 运行时代码**。下面的「技能清单」是给 OIA 若自建 harness 时的**首批建设顺序**，「schema 落点」是**建议路径**（按现有结构推断，非已存在文件）。

### 7.1 首批 8–12 个技能清单建议（先零成本、后付费；先单点、后成稿）

按 Omni-IO 的三层结构给建议，**优先级 = 免 key/本地能力 > 高频理解 > 单点生成 > 成稿专家 > 多交付物场景**：

| 优先 | 建议技能（对位 Omni-IO 路径） | 为什么先做 |
|---|---|---|
| P0-1 | `understand/image`（宿主原生视觉，无 MCP） | 零成本、覆盖最广，且是海报/成稿审查的必需能力 |
| P0-2 | `generate/code` + `generate/markdown`（宿主原生写文件 + 强制 `register_asset`） | 零成本；同时把**注册表**这条主线跑通（否则跨轮复用无从谈起） |
| P0-3 | `generate/document/{ppt,word,pdf,excel}`（本地库，MCP 不调 LLM） | 零成本、Office 是 OIA 最高频交付物；接口形态（宿主先规划 `structure` 再渲染）可直接照搬 |
| P0-4 | `utility/search` + `utility/browse`（Brave + Playwright） | 检索族补齐；`browse` 的「等 network idle + 过短滚动重试」是现成实现 |
| P1-5 | `understand/document`（Mistral OCR，正文截 4,000 字 Markdown） | 让 PDF/Word 输入进图；截断上限是控制上下文的好做法 |
| P1-6 | `generate/speech`（默认 Edge TTS 免 key） | 免费的音频出口，立刻支持「配音/讲解」类交付 |
| P1-7 | `understand/audio`（Gemini 优先，缺 key 按 subtype 回退 Whisper/Qwen2-Audio） | 「优先通用、按 subtype 回退」的降级链值得抄 |
| P2-8 | `generate/image`（fal/OpenAI 双 provider） | 付费能力第一条；同时引入 `first_frame_url/reference_image_url` 注入约定 |
| P2-9 | **Expert：`expert/poster-design`**（locked brief → 最小视觉资产 → code 排版 → 原生视觉审查 → 只修坏层 ≤2 轮） | 这是全仓库**最完整的专业闭环范本**，也是「确定性排版 + 图像模型不负责文字」这条关键设计的落点 |
| P2-10 | `understand/video` + `generate/video`（Gemini 分析 / Wan·Kling 生成） | 视频族（费用与时延最高，放在能力面铺开之后） |
| P3-11 | **Scenario：`scenarios/office`**（会议纪要→PPT/Word/Excel 包） | 第一个多交付物场景：验证「场景选专家/原子 + 交付物间复用」的编排是否成立 |
| P3-12 | **Scenario：`scenarios/event_materials` 或 `social_media_post`**（海报+音乐+短视频） | 跨模态扇出 + 专家间资产复用（海报作视频首帧）的最佳演示，对应论文 Table 2 的四 Wave 案例 |

配套的**四个横切协议**（不算技能，但必须与首批技能同批落地）：`declare_format`（12 种 type + 自包含 prompt）、`dependency_rules`（保守声明 + 注入表 + 多依赖仅限 code/markdown）、`asset_registry`（`set_turn`/自动注册/手动注册/`asset_ref`）、`expert_protocol` + `interruption`（≤2 轮返工、五类中断）。

### 7.2 资产注册表 schema 落点与建议

**落点建议（OIA 侧）**
- 协议与字段说明（给人读）：`docs/sources/repos/...` 不合适（那是镜像），建议新建 `docs/asset-registry-spec.md`，或并入现有 `docs/code-level-notes.md`；
- 运行时实现（给代码）：建议 `harness/registry/registry.json` + `harness/registry/registry.py`（路径为建议，OIA 目前无 `harness/` 目录，需新建）；
- 每个工程一个 `OMNI_OUTPUT_DIR` 等价物（Omni-IO 用「不同项目不同输出目录」做项目隔离，README L312）。

**建议 schema（论文式(2) + 仓库实测字段合并版）**

```json
{
  "img_a3f2c1": {
    "asset_id": "img_a3f2c1",
    "type": "image",
    "subtype": "poster",
    "status": "completed",
    "url": "/abs/path/under/OUTPUT_DIR/img_a3f2c1.png",
    "description": "生成时自动写下的完整句子描述（供跨轮语义检索）",
    "params_used": { "aspect_ratio": "3:4", "provider": "fal" },
    "turn_id": "turn_20261121_1419",
    "completed_at": 1795276755,
    "depends_on": "img_9f81aa",
    "task_id": "poster_final"
  }
}
```

字段来源与取舍：
- `asset_id / type / subtype / path(实现为 url) / description / params / turn_id / source_asset_id(实现为 depends_on)` ← 论文式(2)（§4.5）；
- 额外必须加的实现字段（仓库实测）：`status`（只允许 completed 入表，failed/cancelled 不写，`output_format.md` §关系）、`completed_at`（排序与「上一张图」解析）、`task_id`（回指 declare 节点）；
- **不加** HTTP URL：`url` 必须是本地绝对路径（`output_format.md` 明确「not an HTTP URL」）；
- `description` 是**跨轮语义检索的主键式字段**（`list_assets()` 返回后靠 `turn_id` + `type` + description 语义匹配定位），建议沿用「生成时用小模型自动写 ≤80 词」的做法（`generate.py::_auto_describe_image`）。

**实现要点清单（照抄 `mcp/registry.py`）**
1. 单一 JSON 文件为唯一事实源，**读写都直连文件、不用内存缓存**（多进程共享）；
2. `_save` = 写 `.json.tmp` + `os.replace`（**全平台原子**）；
3. macOS/Linux 用 `fcntl.flock(LOCK_EX)` 串行化，**锁内重新 load 再写**；Windows 无 fcntl 则跳过锁（须在文档里如实标注并发丢记录风险）；
4. id = `{prefix}_{uuid4().hex[:6]}`，prefix 表（image→img、video→vid、3d→3d、music→mus、sfx→sfx、speech→tts、否则 subtype）；
5. **append-only、永不原地更新**；修订产生新 id 并记 `depends_on` 血缘；
6. 注册时机：生成工具返回时自动注册；宿主原生写盘（code/markdown/成稿）**必须手动注册**，并把「漏注册 = 孤儿、后续轮次找不到」写进技能文本；
7. 每轮开始 `set_turn("turn_YYYYMMDD_HHmm")`；
8. 跨轮复用：declare 里写 `asset_ref`，该节点视为已完成、不发 API。

---

## 8. 诚实边界

1. **未复现任何实验数字**。§4 全部为论文自报（Table 3），且基准 UniM-90 的实例清单与选择协议附录在本 HTML 中缺失，**无法独立核验抽样是否「独立于被测 agent 能力」**。
2. **本篇无消融、无显著性/方差、无费用与延迟数据、无人评**（全文 grep `ablat` 零命中）。主结果只证明「加了 harness 后 τ=100% 且分数更高」，**不能区分增益来自技能、MCP 工具、provider 换代还是输出格式约束**——这是论文最明显的证据缺口。
3. **HTML 渲染缺陷影响取证**：Figure 2（任务覆盖图）渲染为 `{forest}`；Appendix B 表体为空（38 任务清单缺失）；UniM-90 选择协议与「参考程序附录」的交叉引用为 `ltx_missing_label`。以上均按「未在全文中找到」处理，未做推测性补全。
4. **代码仓库与论文版本的一致性未验证**：仓库 `SKILL.md`/`skill.yaml` 标 `version: 3.0.0`，README News 为 2026-08（代码）/2026-09（论文）；未运行 `mcp/server.py`、未安装依赖、未实际调用任何 provider，**所有代码摘录仅为静态阅读**。
5. **案例媒体已裁剪**：`examples/` 只剩 `PROMPT.txt` 与部分输出（html/json/js/txt/pdf），README 中的 teaser.jpg 与 case*.mp4 在镜像中不存在，**无法核对案例展示效果**；20 个案例与论文 38 任务的对应关系未逐一核对。
6. **指标计算细节不在本篇**：SQCS/ICS/StS/LeS 的公式与评审协议在 UniM 原文（bib [12]），本篇只给口径描述——引用具体分数时须说明「UniM Evaluation Suite 自报」。
7. **Windows 锁降级、`gpt-4o-mini` 自动描述、provider 价格**均为仓库注释/文档的自述，未在本环境验证；价格随时可能变化（README 亦如此提示）。
8. **DSH 对位**基于本机安装文档 `dsh-doc/references/packages.md`（`dsh-skill`/`dsh-skill-filesystem`/`dsh-workflow-ptc`/`dsh-attachment-local`/`dsh-client-ui-attachment`）与本会话可见的工具语义，**未逐一运行验证**；DSH workflow 的实际调度能力（能否直接承载 DEG Wave）需另行实验确认。
9. 本笔记对 Table 1 组合关系的提取依赖 HTML 中 SVG 单元格的行号区间映射（§3.1 表），**与论文印刷版可能存在的排版差**（如箭头链 vs 并列）未与 PDF 交叉核对。

