# BLOG_PLAN.md — OIA 调研博客（paper-reading-blog v5 合并计划文档）

> 合并 ALIGNMENT / CONSENSUS / DESIGN / TASK 四份文档为一份。生成于 2026-11-21。

## 1. 需求理解（原 ALIGNMENT）

- **主题**：Omni-Interaction-Agent（OIA）立项调研——8 个记忆项目 + 3 个社区插件 + 5 篇论文 + DSH/Pi 基座的深度学习博客。
- **交付形态**：**多页网站**（用户明确：一切汇报只以可视化网站形式，非网站信息一律不看）。
- **用途定位**：只用于学习与自我研究；用户声明所有许可协议均可使用——但博客仍逐处标注代码出处与许可（rolling-memory 为 PolyForm NC，教学摘录）。
- **知识来源**：本地已实读的 7 仓库源码（`docs/sources/repos/`）、13 份文档快照、DSH 内核文档、5 篇 arXiv 论文摘要原文、`report.html` 总报告与 `docs/code-level-notes.md`。
- **继承偏好**：封面 C. 中国水墨 + Hero 模板 A（见 `project_memory.md`）。

## 2. 技术方案与约束（原 CONSENSUS）

- **站点结构**：`blog/index.html`（门户首页）+ `blog/posts/*.html`（6 个内容页）+ `blog/assets/css/style.css`（共享设计系统）。
- **设计 DNA**：skill 的 lavender-coral-mint 调色板 × 水墨气质（墨黑标题、宣纸底、泥金点缀）；Noto Serif SC 标题 / Noto Sans SC 正文（含本地字体回退栈）。
- **Hero**：模板 A（浅色渐变 + 封面卡片），徽章显示系列位置与真实 venue/许可（无作者名，居中）。
- **公式**：手写 HTML/CSS（离线），不用 MathJax CDN——覆盖 skill 默认项，理由记录于 project_memory.md。
- **深度门控**：每页 ≥4 章节；每章 ≥2 深度元素（设计 rationale / 数学直觉 / 架构洞察 / 替代方案 / 跨文连接 / 批判评估）。
- **可视化**：每页 ≥2 个自绘内联 SVG（全站 ≥12，远超 skill 的 5 个下限）；全部中文标签 + 图号 caption。
- **代码**：每页页尾「代码附录」，≥2 个代码块，标注 `[源码原文]`（附文件路径）/ `[示意性实现]`，`white-space: pre-wrap`。
- **深度洞察框**：每页 ≥3 个 `.insight-box`（绿色）。
- **自主科研问题**：每页页尾 1–2 个 `.research-question`（橙标签）；首页汇总 5 个完整版问题。
- **反幻觉**：不引入任何本地存档之外的「事实」；数字一律标「自报」；不虚构论文图表（无 PDF 就不裁图，全部自绘 SVG）。

## 3. 章节大纲与设计（原 DESIGN）

| # | 页面 | 核心叙事 | 招牌 SVG（自绘） | 招牌代码（出处） |
|---|------|---------|------------------|------------------|
| 0 | `index.html` 首页 | 系列总览 · 阅读路线 · 自主科研问题 5 问 | 全景地图（七页关系）+ 水墨封面 | —（导航页无代码附录） |
| 1 | `01-tencentdb-memory.html` | 分层金字塔 × 符号画布：「下层证据，上层结构」 | ① L0–L3 语义金字塔与下钻链 ② offload→node_id→refs 管线 | L1 抽取提示词原文、去重动作集、sqlite schema（`tencent-memory-main/src/…`） |
| 2 | `02-rolling-memory.html` | 零成本词法检索：信息量即权重 | ① 自信息权重标尺（汉字10bit vs 字母2bit） ② burst vs dispersed 权重曲线 | `RuneShannonBits` / `familyEffectiveWeight` / `StringShannonBits` 阻尼（`rolling-memory/internal/…`） |
| 3 | `03-hy-graphmemix.html` | 双过程认知 × diff-chain × 证据森林 | ① System1/System2 节律图 ② thread/round 版本链 ③ 森林预算示意 | `chain_store.py` 表结构与预算常量、意图 schema（`hy_memory/…`） |
| 4 | `04-memorax-community.html` | 四类记忆边界 × 人审治理 × 用户主权 | ① 四边界象限 ② 待审核状态机 ③ 三路召回链 | `review_policy.py` 48h、`recall.py` 管线、`dsh-adapter/package.json` 打包清单 |
| 5 | `05-omni-papers.html` | 主动性与个性化四论文：何时出声、何时沉默、怎么说 | ① 响应/沉默/追问三态机 ② Coco 观察-适应闭环 ③ 表达规划分层 | InterventionPolicy 伪代码（示意性实现） |
| 6 | `06-dsh-pi-oia.html` | DSH×Pi 基座与 OIA 总体设计（隔离红线） | ① 能力 seam 架构图 ② 双 DSH 隔离图 | `cordis.patch.yml` + `@omni/*` bundle 清单模板（xuyi 为范本） |

每页统一骨架：Hero（徽章+标题+tagline+封面 SVG 卡）→ 导航条 → 章节（含洞察框/SVG/表格）→ 代码附录 → 延伸问题 → 页脚（系列导航+诚实声明）。

## 4. 验收标准 checklist（原 TASK，轻量）

- [ ] 7 个 HTML 全部可 `file://` 直接打开，零外部网络依赖（无 CDN/字体外链/图片外链）
- [ ] Hero：无作者名、徽章在、居中、封面 SVG 有语义
- [ ] 每页：章节 ≥4、深度元素 ≥2/章、洞察框 ≥3、自绘 SVG ≥2、代码块 ≥2（含可信度标注与出处路径）
- [ ] 每页页尾：`.research-question` ≥1；首页 5 问
- [ ] 数字全部带「自报」或出处；无本地存档外的编造事实
- [ ] 代码块 `pre-wrap` 不横向溢出；导航链到其余 6 页 + 首页
- [ ] Edge headless 截图验证通过（每页至少 hero+中段+页脚三段）
- [ ] 收尾：README 更新、ACCEPTANCE_*.md、FINAL_*.md、门户卡片（见 report.html 或 README 链接）
