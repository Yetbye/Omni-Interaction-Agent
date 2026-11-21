# ACCEPTANCE_oia-blog.md — OIA 调研博客验收清单

> 任务：以 paper-reading-blog skill v5 为 Omni-Interaction-Agent 项目制作多页学习博客。
> 验收日期：2026-11-21 · 执行者：root（6 个内容页由并行 subagent 撰写，root 撰写首页与设计系统）

## 一、Skill 硬性门控（Phase 4/6）

- [x] **布局唯一**：七页各具独立章节结构（无模板复制）；共享的只有设计 DNA（调色板/字体/组件）
- [x] **Hero 模板**：全部采用模板 A（浅色渐变 + 图片卡片）——与封面风格 C（中国水墨）匹配（skill 2.7 选择规则）
- [x] **封面风格档案**：`project_memory.md` 已记录「C. 中国水墨 + Hero 模板 A」，后续博客自动继承
- [x] **Hero 规范**：无作者名、徽章在（系列位置 + venue/许可）、居中、滚动提示、封面为**自绘语义 SVG**（7 张各异：金字塔三柱 / 信息熵日志 / 双过程版本链 / 人审盖章 / 静默涟漪 / 插件树 / 首页四象限）
- [x] **图表裁剪**：本系列无 PDF 图可裁（综述无全文、论文仅摘要），**全部改用自绘 inline SVG**——未使用任何整页截图作为图
- [x] **自绘 SVG ≥5**：全站 **30+**（首页 2 + 各内容页 3–5 + 封面 7），全部中文标签 + `<figcaption><b>图 N</b>`
- [x] **代码块 ≥2/页 且标注可信度**：内容页共 **29 个代码块**（25 个 `[源码原文]` 附精确文件路径 chip + 4 个 `[示意性实现]`），`white-space: pre-wrap` 由共享 CSS 强制
- [x] **代码准确性自检（3.7）**：subagent 逐块核对公式与源码系数（如 p2 与 entropy.go/cluster_state_aggregates.go 逐项核对；p4 修复 SVG 内 `<sup>` 不渲染问题）
- [x] **深度质量门控**：每页 6–7 章、每章 ≥2 深度元素；`.insight-box` 每页 4–7 个（全站 30+）+ `.critique-box` 批判框
- [x] **自主科研问题**：首页 5 个完整版（理论分析 + 研究方向）+ 各内容页页尾 1–2 个延伸问题卡（橙标签），全站 **11 个**
- [x] **深度洞察框**：绿色卡片样式按 skill 5.4 实现，全站密布关键分析点
- [x] **数学公式**：手写 HTML/CSS（`.math` + sub/sup/frac），**不引 MathJax CDN**——覆盖 skill 默认项，理由记录于 project_memory.md（离线可读优先），已截图验证渲染正确
- [x] **反幻觉审计（6.4）**：所有论文数字对到 `papers-abstracts.md` 摘要原文；所有项目数字对到本地 README/源码快照；全部标 `<span class="self-report">自报·未复现</span>`（全站 40+ 处）；综述《Native Real-Time Interactive Omni Intelligence》**零引用**
- [x] **BLOG_PLAN.md 已生成**（合并四文档，v5 规范）
- [x] **封面图语义**：每张封面编码该页核心隐喻（非装饰色块）

## 二、交付物清单

| 文件 | 说明 |
|---|---|
| `blog/index.html` | 导读首页：水墨封面、七页全景地图 SVG、导览卡、四主题速览、**自主科研问题 5 问**、诚实声明 |
| `blog/posts/01-tencentdb-memory.html` | 腾讯云记忆：金字塔 × 符号画布（7 章 / 4 SVG / 4 代码） |
| `blog/posts/02-rolling-memory.html` | rolling-memory：自信息权重 × 突发折叠（6 章 / 4 SVG / 5 代码 / 9 公式） |
| `blog/posts/03-hy-graphmemix.html` | Hy × GraphMemix：双过程 × diff-chain × 证据森林（6 章 / 5 SVG / 5 代码） |
| `blog/posts/04-memorax-community.html` | MemoraX × 社区：四边界 × 人审 × 主权（6 章 / 5 SVG / 5 代码） |
| `blog/posts/05-omni-papers.html` | 四篇论文：出声 · 沉默 · 表达（7 章 / 5 SVG / 2 代码 / 2 问） |
| `blog/posts/06-dsh-pi-oia.html` | DSH × Pi × OIA：基座与总体设计（6 章 / 3 SVG / 4 代码） |
| `blog/assets/css/style.css` | 共享设计系统（水墨 × lavender-coral-mint，唯一权威样式源） |
| `blog/BLOG_PLAN.md` / `project_memory.md` | 计划文档 / 封面风格偏好档案 |
| `blog/docs/verify/` | 7 张全页验证截图 + 2 张关键区域抽查图（证据留存） |

## 三、技术验证记录

1. **标签平衡**：7 页全量脚本校验，table/div/svg/h2/pre/section/tr 开闭全等，0 错误。
2. **链接完整性**：全站 href 提取 → 目标文件存在性检查，**零断链**（7 页互链 + CSS）。
3. **零外部请求**：7 页 `https?://` 引用均为 0（仅本地 `../assets/css/style.css` 与 SVG 内部 `url(#id)`）。
4. **class 一致性**：各页所用 class 全部存在于 style.css（subagent 自查 + root 复核）。
5. **浏览器渲染验证**（Edge headless + CDP，全页截图）：
   - 首页：Hero 渐变题字/封面 SVG/统计卡 ✓、全景地图 SVG ✓、科研问题卡 ✓、暗色页脚 post-nav 网格 ✓
   - p2：表格/洞察框/批判框 ✓、分式公式（effDF/weight/satDecay）✓、信息量标尺与 burst 曲线 SVG ✓、代码附录（chips + 深色高亮 + pre-wrap）✓、延伸问题卡 ✓
   - p4：章节结构/治理三接口表 ✓、洞察与批判框 ✓、自报标注 ✓
   - 排障记录：① 临时副本置于子目录导致 CSS 相对路径失效（渲染无样式）→ 改同级副本；② `captureBeyondViewport` + sticky 导航导致全页截图内容重复 → 注入静态导航后复测，y=17300 与 DOM 真值（代码块 y=17488）精确吻合。
6. **DOM 地面真值**（CDP Runtime.evaluate）：p2 scrollH=20362、7 个 h2、3 个 svg-figure、5 个 code-block、5 insight、1 RQ —— 与截图逐一对位。

## 四、遗留事项

- [ ] 综述 PDF 入库后回填其框架（当前零引用，已在首页与页脚声明）
- [ ] rolling-memory PolyForm NC：站内摘录仅供教学，若未来商用分发需逐块复核许可（已在页脚声明）
