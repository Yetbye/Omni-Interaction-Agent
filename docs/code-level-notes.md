# OIA 代码级借鉴清单（源码实读记录）

> 2026-11-21 · 配套 report.html §4/§5/§6 的代码级支撑。以下所有结论来自**实读源码**，
> 本地存档：`docs/sources/repos/`（7 仓库浅克隆/tarball 解包，rolling-memory 25MB、tencent-memory、hy-memory、memorax-code、xuyi-memory、amber-heart、anima-brain；**该目录不入库**——体量与上游许可归各项目，见 `.gitignore`，仓库内保留全部文档快照 `docs/sources/*.md`）。
> 精确到文件/函数/公式/常量。许可注意：rolling-memory 为 PolyForm 非商用，**只学思路不搬代码**。

---

## 1. rolling-memory（Go）— Recall T0 词法检索内核

### 1.1 自信息权重（可整体移植的思想与公式）
- `internal/text/entropy.go` — `RuneShannonBits(r)`：按 Unicode 文块给先验香农信息量（**汉字 10 bit**、字母 2、数字 3.32、控制/空白 0、组合符 0.5）。
- `StringShannonBits(s)`：字符串自信息 = Σ字符 bit，叠加四种抑制：
  ① 无字母数字→0；② 一阶马尔可夫重复阻尼（同字符连续 k≥3 乘 0.25^(k-2)）；
  ③ 滑窗周期 p∈1..4 串联重复折叠；④ 空格分隔重复洪泛折叠。
- **借鉴**：用「信息 bit 数」做检索权重与预算单位（query coverage 以 nats 计），比 TF-IDF 更抗日志重复刷屏。OIA `@omni/memory-recall` 的 T0 层直接采用此权重模型（自实现，不搬代码）。

### 1.2 突发克隆折叠 / 离散增益
- `internal/core/cluster_state_aggregates.go` — `familyEffectiveWeight(df, multiDocCount, clusters, total, entropy, clusterN)`：
  `effDF = 1 + (df-1) · ((clusters-1)/(df-1))^(1/e) · disp`；`weight = ln(1 + (clusterN+1)/(effDF+1))`
  其中 `familyDispersion = min(1, H/ln(df))`（H=搭配词分布熵）。
  → **同一搭配反复刷（burst）熵低→权重高；分散出现（每个文档不同搭配）→按独立事件衰减**。
- `internal/core/unit_relations.go` — `burstFreeQuota = 3.0`：事件内重复出现前 3 次免费，
  之后 `satDecay = exp(-max(0, microOcc-3)·e)` 饱和衰减。
- **借鉴**：记忆去重与「重要性」打分要用**出现分布**而非次数；常数 3/e 可作为初版参数。

### 1.3 时间聚类（会话分段）
- `internal/core/pipeline_state.go` L2540：`tau_episode = e^(mu+sigma) - 1`（mu/sigma=对数间隔的均值/方差），
  间隔严格超出 tau 才切新 episode；同 burst 跨 UTC 午夜仍同 episode（`episodic_component_test.go`）。
- **借鉴**：把散乱消息聚成「事件/情境」的确定性规则，给 Distiller 的 L2 场景切分做零成本预切分（LLM 只需确认/命名）。

### 1.4 两跳扩散 + 预算
- `internal/core/pipeline.go`：`transmissionPerHop = 1/e`；hop1 用 `Hop1Budget`、hop2 用 `1/e²` 与 `Hop2Budget`；`spread()` 独立成可测函数。
- `internal/core/pipeline_select.go` L346：`compBound = (1/e)² · lex · (1-exp(-lex/total))` 结构补偿上界。
- **借鉴**：召回候选扩展（同实体/同事件邻接）必须带**跳数预算与质量上界**，防止相关但无用的证据淹没注入窗口。

### 1.5 其他工程细节
- `internal/memory/localvault/facade local_vault.go` — `foldRecentNoeticDerivatives`：L3 衍生句柄（由 recall 产生）折叠到原始记忆之后，防止衍生品刷屏。
- 查询词对齐（`internal/text/spacetime.go`：路径折叠、日期跨形态）；`ids.go NormalizeWithMap` NFKC+casefold。
- 效能账本：`internal/efficacy/`（采纳审计）、`internal/eval/`（ATM/LoCoMo 离线复算）→ **评测方法论直接抄作业**：题集 pin SHA-256、逐题 JSONL 可复算。

---

## 2. TencentDB-Agent-Memory（TS）— Distiller 提示词与 Canvas 管线

### 2.1 L1 抽取提示词（`src/core/prompts/l1-extraction.ts`，可直接改写为 OIA 版）
- 一次 LLM 调用同时做**情境切分 + 记忆提取**；情境命名模板：「我（AI）在和xxx做xxx」30–50 字、全局唯一、可继承。
- 记忆只收三类：`persona`（稳定属性/偏好，priority 80–100 核心 / 50–70 一般 / <50 丢弃）、
  `episodic`（客观事件，句式「用户在[绝对时间]于[地点][做了…]」，可带 activity_start/end_time）、
  `instruction`（长期行为规则，priority -1 为死命令 / 90–100 核心 / <70 丢弃）。
- 「宁缺毋滥 / 独立完整（跳出对话仍成立）/ 归纳合并」三原则 + 明确负面清单（琐碎闲聊、一次性指令、AI 自身输出、纯主观感受）。
- 背景消息仅用于理解（**严禁从中提取**），只从新消息提取——与 rolling-memory「原文档案」互补。

### 2.2 冲突/去重提示词（`src/core/prompts/l1-dedup.ts`）
- 动作集：`store | skip | update | merge`；`target_ids` 数组支持**多对多合并**；跨 type 可合并（persona+episodic 同一事实）。
- `merged_timestamps` = 所有相关时间戳并集（保留完整时间线）；merge 后 priority 酌情提升。
- 批量模式：统一候选池 + 每条新记忆的相关候选 ID，一次调用全局去重。
- **借鉴**：这就是 OIA「意图式保存 + 语义去重 + 取代留痕」的现成提示词骨架；OIA 加一条「旧措辞保留为 superseded 历史」（MemoraX/续忆要求）。

### 2.3 短期卸载 + Mermaid 画布（`src/offload/`）
- 数据形态：`offload.jsonl` 条目 `{tool_call_id, node_id|null|"wait", result_ref → refs/*.md}`；全文卸载到 refs。
- L2 触发：node_id=null 条目 ≥ **4**（`types.ts: l2NullThreshold`）或时间触发；失败留 `"wait"` 态 + 重试秒数；`backfillNodeIds` 回填。
- 注入格式（`mmd-injector.ts`）：`【当前活跃任务的mermaid流程图】` + mermaid 代码块 + **节点索引说明**（node_id → offload.{session}.jsonl → result_ref → 读原文）。
- L2 提示词纪律（`local-llm/prompts/l2-prompt.ts`）：「只记录已发生、未发生不写、每个节点标注 node_id 来源」。
- **借鉴**：Canvas 组件照此实现（DSH 侧落为 compaction 后端 + tool-result pruner 配合）；注入模板与「node_id 下钻」说明直接翻译成 OIA 文案。

### 2.4 存储 schema（`src/core/store/sqlite.ts`）
- `l1_records`（type/priority/timestamp_start/end/scene_name/session_key）+ `l0_conversations`；
  FTS5 虚表（l1_fts/l0_fts）+ sqlite-vec `vec0` 虚表（embedding float[768]）双索引。
- **借鉴**：OIA Vault 表结构照此裁剪（FTS5 必选、vec0 可选），索引列清单直接沿用。

---

## 3. Hy-Memory（Python）— diff-chain 与前瞻意图

### 3.1 diff-chain（`hy_memory/coding/chain_store.py`，924 行，注释即设计文档）
- **定位**：「装饰器不是检索器」——向量层决定给哪几条 fact，本模块只在 fact 下挂「该处代码的全部版本（用户原话 + 真实 diff）」。
- 表：`thread(id, owner, branch, file, symbol, lo, hi, n_rounds)` + `round(id, thread_id, seq, said, patch)`；
  **owner=user|workspace 必须进键**（防跨用户串链）。
- thread 键三级降级：hunk header 带符号→(branch,file,symbol,行区间) → 无符号→(branch,file,行区间聚类，LINE_MERGE=20 行) → 裸 hunk→(branch,file)。
- `seq` = observe() 调用计数，纯物理事实，**不做版本覆盖推断**。
- 注入预算常量（env 可调）：每版 6 行/480 字、原话 800 字、整块 14000 字、SHOW_ALL=8 版、TOPN_THREADS=3 链、TOPN_FACTS=1 条 fact、BODY_MAX_CHARS=900。
- 失败闸门：合成编辑（对话里的 s.replace 型改动）必须校验「改动确实成功」才入链（拒绝"提议未执行/执行失败"）。
- **借鉴**：①「被 revert 的版本只有 diff 链能救」直接进 OIA Vault（挂编辑史，零 LLM）；② 预算常量表作为 OIA 注入预算初值；③ 抽取 prompt 明文「EXACT COORDINATES, NOT PASTED CODE」（~200 字预算）与 diff 链互补的分工。

### 3.2 前瞻意图（`hy_memory/agent/intention_detector.py`）
- schema：`{content, trigger_type: time_based|event_based, trigger_condition, trigger_time, trigger_event_pattern, priority, expiry, is_proactive_care}`；
  还有「关怀型」隐式意图（"考试快到了好紧张" → 考后关心）。
- 三个消费点：System 1 快速入队 / 读路径触发检查 / System 2 深化触发条件。
- **借鉴**：OIA 干预策略器的意图层字段照单全收，`is_proactive_care` 对应 InterventionPolicy 的 SEEK_FEEDBACK/关怀通道。

---

## 4. 琥珀之心（Python/PostgreSQL）— 召回链与人审状态机

### 4.1 召回（`memory_core/recall/recall.py` + `search.py` + `rerank*.py`）
- 主链：`ThreeRouteSearch`（关键词/向量/图谱三路粗排）→ `replace_parents`（父卡替换）→ `add_rerank_evidence` →
  `order_candidates`（精排模型）→ `select_candidates`（LLM 细选）+ `TailContext`（全列表模式）。
- 全程 `stats` 诊断对象随管线流转（retrieval/ordering/selection_stats）→ 这就是「召回透明页」的数据源。
- 时间限定检索：`recall/date_period.py`（昨天/上周…）。
- **借鉴**：OIA Recall 的 T1 阶段照此分层（粗排三路→证据补充→模型精排→LLM 选择），**stats 贯穿**设计直接抄。

### 4.2 人审状态机（`memory_core/cards/`）
- `pending_memories`：status ∈ `pending | structure_pending | structure_failed | approved | rejected | deleted`；
  乐观并发三件套：`version` + `review_key`（防重复审批）+ `policy_revision`（策略变更后重审）。
- `review_policy.py`：`delay_hours = 48`（env `MEMORY_AUTO_APPROVE_AFTER_HOURS`），`auto_approve_enabled` 策略开关，
  `review_ready_at` 到期自动收录；`drafts.expire_materials` 清理过期素材。
- **借鉴**：OIA Governance 的待审核表与状态机照搬字段与 CAS 语义；48h 参数化。

---

## 5. 续忆（DSH 插件）— @omni/* 打包模板与治理接口

### 5.1 DSH bundle 清单（`dsh-adapter/package.json` + `cordis.patch.yml`）——OIA 插件的直接模板
```jsonc
// package.json 关键字段
{
  "name": "dsh-xuyi-memory", "type": "module", "main": "./index.js",
  "exports": { ".": "./index.js", "./client": "./client.js", "./locale/*.json": "./locale/*.json" },
  "dsh": {
    "bundle": { "patch": "./cordis.patch.yml" },
    "client": { "platform": "web", "immediately": true }
  }
}
// cordis.patch.yml
- insert:
    - id: xuyi-memory
      name: dsh-xuyi-memory
```
- 文件布局：`index.js`（Host 侧插件入口）/ `client.js`（Web 客户端插件）/ `logic.js`（业务 62KB）/ `rhythm.js`（节奏）/ `locale/{zh,en}.json` / `market/index.json`（插件市场索引）。
- **借鉴**：`@omni/*` 每个 bundle 按此形状落地（id 前缀 omni-、client 插件带面板）；市场索引可参考 `scripts/make_market_index.py`。

### 5.2 治理接口（README + `logic.js`/tests 佐证）
- 「别再提了」= 抑制位（数据保留、不再召回）；「记错了」= 纠正 + 旧值保留；时间限定检索；模型后装（bge-small-zh）+ 词法兜底。
- tests/ 目录 20+ 测试文件（consolidate/decay/diary/audit…）——**先写题后改码**的评测纪律。
- **借鉴**：Governance 三接口（suppress/correct/delete）语义与降级路径直接采用。

---

## 6. Anima Brain（Python）— 打分公式与证据链

- `retrieval.py`：RRF 融合 `1/(60+position+1)`（词法+向量两路）后线性排序：
  **`score = 0.6·relevance + 0.25·exp(-age_days/45) + 0.15·importance/10`**（公式随结果返回，可解释）。
- `evidence_index.py` + `note_evolution.py`：摘要必附引用证据、笔记演化留痕。
- **借鉴**：OIA Recall 精排的初版公式照此（可把 0.6/0.25/0.15 做成配置），「分数随结果返回」进召回透明页。

---

## 7. MemoraX Code（TS）— 系统边界与 DSH 适配器

- `ARCHITECTURE.md`（1633 行）：Hook/检索数据流、自动写回、Repo Memory 协调、权威边界（authority map）、
  「本地-only 数据流」可观测性——**架构文档本身的组织方式**值得 OIA 抄（来源即事实源、可执行架构契约）。
- `packages/ts/memorax-code-dsh-adapter/`：DSH profile 生命周期管理（安装/更新/恢复），证明 DSH 插件生态可以做复杂生命周期。
- **借鉴**：写回的「只发用户指令 + 最终答复（去掉记忆披露尾注）」最小化上传原则；trace 可 metadata-only。

---

## 8. 汇总：OIA 从代码层「直接搬什么」

| 目标组件 | 搬运内容 | 来源（文件级） | 方式 |
|---|---|---|---|
| recall-T0 | 自信息权重 + 突发折叠 + tau_episode 分段 | rolling-memory entropy.go / cluster_state_aggregates.go / pipeline_state.go | 公式自实现（许可约束） |
| recall-T1 | 三路粗排→证据→精排→选择，stats 贯穿 | amber recall.py / rerank*.py | 结构照搬 + 自实现 |
| recall-精排公式 | 0.6·rel + 0.25·exp(-age/45) + 0.15·imp/10 | anima retrieval.py | 参数化采用 |
| distiller-抽取 | 情境切分+三类记忆+priority+负面清单提示词 | tencent prompts/l1-extraction.ts | 改写为 OIA 提示词 |
| distiller-去重 | store/skip/update/merge + target_ids 多对多 | tencent prompts/l1-dedup.ts | 改写 + 加 superseded 留痕 |
| canvas | node_id/result_ref/refs、L2 阈值 4、wait 重试、注入模板 | tencent src/offload/* | 结构照搬 |
| vault-schema | l1_records/l0_conversations + FTS5 + vec0 索引列 | tencent store/sqlite.ts | schema 裁剪 |
| vault-编辑史 | thread/round 表 + 三级 thread 键 + 预算常量 | hy-memory coding/chain_store.py | 自实现（思路+参数） |
| 意图层 | time/event/care 三触发 + expiry | hy-memory intention_detector.py | 字段照搬 |
| governance | pending 状态机 + version/review_key/policy_revision CAS + 48h | amber cards/* | 字段与语义照搬 |
| @omni/* 打包 | dsh.bundle.patch + client 插件 + locale + market | xuyi dsh-adapter/package.json | 模板照搬 |
| 架构文档 | 权威边界/来源即事实源/可执行契约 | memorax ARCHITECTURE.md | 文档方法论 |

**尚未读**：memorax backend 内部实现（云侧核心未开源，开源的是集成层——已确认边界）；
rolling-memory admin/console 前端；amber 图谱召回实现（graph_store.py 14KB 未逐行）；Pi 各包源码（pi-ai 对接面以后按需）。
