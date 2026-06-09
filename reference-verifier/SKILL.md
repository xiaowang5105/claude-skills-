---
name: reference-verifier
description: Verify academic citations by searching online — confirm existence, check metadata, compare abstract direction against text claims. Use for 引用核验, citation verification, 查引用, 核对文献, or as the fact-checking layer called by hexie-writer Phase 2.
---

# 引用联网核验

## 角色

`reference-verifier` 只做一件事：联网核验引用。它不改文、不润色、不判断论文整体质量。输入引用列表和正文中的引用表述，输出核验表。

硬规则：只核验事实——文献是否存在、作者/年份/题名/期刊是否准确、摘要方向是否支持正文概括。不虚构检索结果，查不到就是查不到。

## 输入

1. 引用列表（作者、年份、标题、期刊/来源、DOI 如有）
2. 正文中对该引用的表述（如"X 发现 Y""X 研究了 Z"）

## 核验流程

对每条引用，按以下顺序操作：

1. **搜索**：用作者 + 年份 + 标题关键词，使用当前环境可用的联网搜索、浏览、DOI 解析或网页读取工具（如 Google Scholar、Semantic Scholar、Crossref、CNKI、出版商网站等），优先选择能返回摘要或元数据的渠道
2. **确认存在**：核对题名、作者、年份、期刊/来源是否匹配
3. **获取摘要**：从搜索结果或出版商页面获取摘要或正文片段
4. **方向比对**：将摘要/片段中的核心发现与正文对该引用的表述进行比对
5. **判定**：输出八种状态之一

## 判定状态

| 状态 | 含义 | 示例 |
|------|------|------|
| `METADATA_VERIFIED` | 文献存在，作者/年份/题名/期刊基本对得上 | 搜到 MDPI 页面，题名作者年份卷期完全匹配 |
| `ABSTRACT_DIRECTION_VERIFIED` | 摘要在方向上支持正文概括的大类（如"该文确实研究了供应链韧性"） | 摘要讨论 supply chain resilience，正文说"该文研究了供应链韧性"——方向对 |
| `CLAIM_SUPPORTED` | 正文的具体说法能被摘要或可获取的全文片段明确支持 | 摘要说"补贴显著提升了韧性"，正文也引了同样的发现 |
| `CLAIM_TOO_STRONG` | 文献存在，但正文说法比原文更确定/更广 | 原文说"可能/部分"，正文引成"已证明/全部" |
| `MISMATCH` | 文献存在且方向对得上，但正文引用方向、归属或内容有明显偏差 | 原文研究供应链金融（supply chain finance），正文引成"产业政策中的财税工具" |
| `NOT_FOUND` | 用作者+年份+标题关键词搜不到该文献 | Lee (2024) 在半导体政策文献中搜不到 |
| `AMBIGUOUS` | 命中多个可能匹配，无法确定是哪一篇 | 同姓作者同年发表多篇，标题不精确 |
| `NO_ACCESS` | 文献存在（有题录），但无公开摘要或全文可查 | 某些付费墙后的中文期刊，只有题名作者年份 |
| `NETWORK_UNAVAILABLE` | 当前环境联网工具不可用或搜索持续失败，无法完成核验 | 网络不可达、搜索接口全部超时、或环境限制无法发起联网请求 |

**关键区分**：`METADATA_VERIFIED` 只说明文献存在，不等于引用方向正确。

`ABSTRACT_DIRECTION_VERIFIED` 的使用边界：
- 正文只说"该文研究了 X 主题""该文讨论了 Y 现象" → `ABSTRACT_DIRECTION_VERIFIED` 可视为通过
- 正文说"该文发现 X 显著提升 Y""该文证明了机制 Z""数据显示 A 导致 B" → 必须有 `CLAIM_SUPPORTED`；摘要方向支持不等于具体发现支持

`NOT_FOUND` / `AMBIGUOUS` / `NO_ACCESS` / `NETWORK_UNAVAILABLE` / `CLAIM_TOO_STRONG` / `MISMATCH` 均不得判定为引用事实通过。

## 输出

核验表格式：

```markdown
## 引用联网核验报告

**核验时间**：[时间]
**核验范围**：[核验了哪些引用]
**总判定**：[所有引用均可核验 / 部分引用存疑 / 存在未找到引用]

### 逐条核验

| # | 引用 | 正文表述 | 搜索结果 | 文献存在 | 摘要方向 | 判定 | 依据 |
|---|------|----------|----------|:--:|:--:|------|------|
| 1 | Farrand (2025) | 欧盟将半导体供应链治理定义为经济安全与国家安全的一部分 | Cambridge/Eur J Risk Regulation, 16(1), 279-293 | ✅ | ✅ | ABSTRACT_DIRECTION_VERIFIED | DOI: 10.1017/err.2024.63; https://www.cambridge.org/core/journals/european-journal-of-risk-regulation/article/... — 摘要讨论 economy-security nexus, strategic autonomy, semiconductor supply chain regulation |
| 2 | Lee (2024) | 韩国加码半导体国家支持 | 无稳定命中 | ❌ | — | NOT_FOUND | 用"Lee semiconductor Korea 2024"在 Google Scholar 及 Semantic Scholar 均未找到匹配文献；无 URL/DOI 可附 |

### 处理建议

| 引用 | 建议 |
|------|------|
| Lee (2024) | 移除或请求用户提供完整文献信息 |
| Liu & Shang (2026) | 文献存在，但研究的是供应链金融而非产业政策财税工具，正文引用需加"间接佐证"限定或更换直接文献 |
```

## 核验策略

- 优先使用当前环境可用的联网搜索工具搜 Google Scholar、Semantic Scholar 等学术搜索引擎
- 有 DOI 的优先打开 DOI 或出版商页面获取摘要
- 中文文献优先搜 CNKI、万方、百度学术等中文数据库
- 同一引用尝试不少于 2 个搜索词组合
- 搜索结果不明确时用 `AMBIGUOUS`，不强行判定
- 查不到时用 `NOT_FOUND`，不编造
- 联网不可用时用 `NETWORK_UNAVAILABLE`，不得强行用本地知识替代联网核验
- 每条 `VERIFIED` 类判定（`METADATA_VERIFIED` / `ABSTRACT_DIRECTION_VERIFIED` / `CLAIM_SUPPORTED`）必须在"依据"栏附 URL、DOI 或出版商/数据库页面链接；没有可附链接的一律降级为 `NO_ACCESS` 或 `AMBIGUOUS`

## 适用边界

适用：核验论文引言、文献综述段中的引用准确性。

不适用：判断论文创新性、评价文献质量、替代 hexie-check 的事实与忠实检查。`reference-verifier` 只做联网存在性和方向比对，最终的引用方向深度判断仍由 hexie-check Gate 1 结合核验报告和用户原文完成。

## 与 hexie 的协作

`hexie-writer` 阶段二（定文献）调用本 skill 核验引用存在性和摘要方向，再基于核验报告完成文献分配表：

```
hexie-writer Phase 2
    │
    ├── 标注文献需求（每个判断位置需要什么类型的文献）
    ├── 选文献（按来源优先级）
    ├── 调用 reference-verifier → 核验表
    │
    ├── METADATA_VERIFIED → 文献存在确认，方向还需自行判断
    ├── ABSTRACT_DIRECTION_VERIFIED / CLAIM_SUPPORTED → 引用事实可视为通过
    ├── NOT_FOUND / MISMATCH / CLAIM_TOO_STRONG → 致命，不得进入文献分配表
    ├── AMBIGUOUS / NO_ACCESS / NETWORK_UNAVAILABLE → 待验证
    │
    └── 输出文献分配表（只含核验通过和待验证的文献）
```

本 skill 可独立调用，也可被 hexie-writer 在阶段二中调用。不自动发起第二轮核验——再次核验由 hexie-writer 或用户主动请求。
