# Omni-Interaction-Agent（OIA）· 项目门户

> 面向「实时交互式全模态智能体」的 harness 二次开发线——
> 以开源 **DeepSeek Harness** 为基座（Pi Agent 生态补给），围绕 **记忆 / 委托与主动性 / 一致性 / 个性化 / 全模态 IO** 做隔离式开发。

**🌐 在线阅读（GitHub Pages）**：**https://yetbye.github.io/Omni-Interaction-Agent/**（根路径自动进入博客首页）
**📦 仓库**：https://github.com/Yetbye/Omni-Interaction-Agent

**当前阶段：立项调研 + 设计（暂不开发）。**

## 一句话结论

1. **基座**：DSH（Cordis 插件树、能力 seam、subagent/Teams/Goals/Schedule/会话事件日志）为主；Pi 的 `pi-ai` 统一 LLM API（DSH 已内置 `dsh-llm-pi-ai`）与包扩展哲学为补给。二开形态 = `@omni/*` 组合包 + patch 层，不 fork 内核。
2. **记忆**：OIA-Memory 五组件（Vault 原文库 / Distiller 蒸馏 / Canvas 符号画布 / Recall 分级召回 / Governance 人审治理），证据链硬约束「画像→场景→事实→原话」可下钻。
3. **隔离红线**：独立 DSH_HOME（`~\.omni-dsh`）、独立 profile（`omni`）、独立插件命名空间（`@omni/*`）、独立记忆库与工作区目录；与当前在用 DSH Desktop 及本工作区五个项目门户**零共享、零侵入**。

## 交付物

| 交付物 | 路径 | 说明 |
|---|---|---|
| **调研博客（主交付 · 多页站）** | [blog/index.html](blog/index.html) | 七页离线站：导读首页 + 6 篇内容页（源码级/公式级拆解、自绘图解、代码摘录、科研问题）；封面风格「中国水墨」，验收见 [blog/ACCEPTANCE_oia-blog.md](blog/ACCEPTANCE_oia-blog.md) |
| **设计调研报告** | [report.html](report.html) | 单文件 HTML：逐项目拆解、论文洞见、总体架构（内联 SVG）、集成矩阵、路线图、风险与诚实声明 |
| **代码级借鉴清单** | [docs/code-level-notes.md](docs/code-level-notes.md) | 7 仓库核心源码实读记录：精确到文件/函数/公式/常量的借鉴清单 |
| 一手资料存档 | [docs/sources/](docs/sources/) | 15 份文档快照（含论文摘要原文 papers-abstracts.md）+ `repos/` 内 7 个仓库源码 |

## 报告看点速览

- **§1 边界与隔离**：五层隔离红线 + 五个现有门户（code2music / shenglv / savepoint / innerpond / pitch）逐一冲突检查，均零冲突；含 M0 隔离验收清单。
- **§4 记忆拆解**：TencentDB-Agent-Memory（分层金字塔 + Mermaid 符号画布）、rolling-memory（verbatim 词法基线 + 采纳审计）、MemoraX（四类边界 + 意图式保存）、Hy-Memory（双过程 + diff-chain）、GraphMemix（证据森林）、续忆 / 琥珀之心 / Anima Brain（人审、纠正、抑制、召回透明）——每项给出「借鉴 / 不借鉴 / 落点」。
- **§5 委托与主动性**：DSH 原生机制盘点 + ALIVE 三态决策（响应/沉默/反馈寻求）+ Coco 隐式反馈环 + Pitch 三缺陷反面清单 → 干预策略器设计定稿。
- **§6 个性化**：Coco 观察闭环 + Pandora 表达规划器 + 三社区插件共识 → UserModel 四层渲染管线。
- **§7 Omni-IO Skills**：与 DSH skills/workflow/attachments 的逐项对位。
- **§9 路线图**：M0（本次）→ M5 评测硬化，均带完成判据。

## 待办（下一轮）

- R1：把《Native Real-Time Interactive Omni Intelligence》综述 PDF 放入 `docs/sources/`，精读后回填报告 §5。
- R2：M1 开工——独立 DSH_HOME 与 `@omni/memory-vault` 骨架。
- R3：逐仓库复核 LICENSE（rolling-memory 为 PolyForm 非商用：思路可用、代码不并入）。
- R4：设计 20 题式记忆盲测集（题先写、失败公开）。

## 隔离声明

本门户**不读写**其他五个项目门户的任何文件；不向当前 `~\.dsh`（日常 Desktop profile）安装或启用任何插件；所有 OIA 运行数据只落在 `~\.omni-dsh\`。

## 版本历史

| 版本 | 日期 | 变更 |
|---|---|---|
| v0.1 | 2026-11-21 | 立项调研：report.html + code-level-notes.md + 15 份资料存档 |
| v0.2 | 2026-11-21 | 7 仓库源码实读，产出代码级借鉴清单（精确到文件/函数/公式） |
| v1.0 | 2026-11-21 | **调研博客建成**（blog/ 七页离线站，paper-reading-blog skill v5 规范，全部验收门控通过） |
| v1.1 | 2026-11-21 | 推送 GitHub 公开仓库 `Yetbye/Omni-Interaction-Agent` 并部署 GitHub Pages（根重定向 → blog/；第三方克隆 `docs/sources/repos/` 不入库） |
| v1.2 | 2026-11-21 | **六篇论文全文精读 + 代码研读**（5 篇 arXiv HTML + ALIVE PDF 提取；6 个论文代码仓克隆）；逐篇笔记 `docs/sources/papers/notes/01–06.md`；博客 03/05/06 页按全文修订（GraphMemix 目标函数与消融、新增 M3-Agent 章、ALIVE/Coco/Pandora/Omni-IO 全部升级并修正摘要级错误）；报告 §5 增补精读状态 |
