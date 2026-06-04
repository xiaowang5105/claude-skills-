---
name: hexie-check
description: Audit Chinese economics and management paper openings for structure, background fact verification, citation faithfulness, academic prose quality, non-AI wording, and readiness after hexie-writer revisions. Use for 检查引言, 检查导论, 检查开篇论证, 检查现实背景, 检查文献综述, 检查研究缺口, 检查边际贡献, 查 AI 腔, 检查引用准确性, 质量检查, 审稿式检查, or as the embedded checker in the hexie-writer loop.
---

# 中文经管论文开篇论证质量检查

## 角色

`hexie-check` 只检查，不改文，不生成终稿文件，不生成 `.docx`。它输出通过/不通过/待验证、违规详情、现实背景核验、引用准确性表和修正清单。`hexie-writer` 读取修正清单后定点修改，并再次调用本检查流程。

硬规则：任何现实背景事实或文献事实无法验证、方向错误、内容夸大或归属错误，判定不通过或待验证。不得用 A/B/C 素材目录替代事实依据。

## 适用边界

适用：检查引言、导论、文献综述、理论铺垫、现实背景、研究背景、研究问题、研究缺口、边际贡献、贡献段，以及这些部分之间的开篇论证链。

不适用：数据处理、模型设定、实证结果、稳健性、机制检验、异质性、结论、政策建议。

## 资源读取

先找到 `hexie` 共享资源库根目录，按顺序尝试：

1. `../hexie-shared-resource-library/`
2. `~/.agents/skills/hexie-shared-resource-library/`
3. `~/.claude/skills/hexie-shared-resource-library/`
4. `../hexie/`（旧路径兜底）
5. `~/.agents/skills/hexie/`（旧路径兜底）
6. `~/.claude/skills/hexie/`（旧路径兜底）

必须按需读取：

- `resource-index.md`：确认 `tier-a-complete-structure-templates`、`tier-b-skeleton-structure-templates`、`tier-c-style-corpus` 的用途和事实边界。
- `material-tier-index.md`：按 A/B/C 档判断素材权重。目录只表示素材入口，不等于等级；具体等级以该索引为准。B/C 档不得作为事实通过的依据。
- `workflow-rules.md`：需要适用范围、事实来源、13 条检查清单、现实背景核验、文献引用准确性、报告格式或判定规则时读取对应章节。

不要整篇读取大文件。检查 AI 腔或用词时，先搜索 `tier-c-style-corpus` 或 `tier-c-style-corpus/raw-ocr`，再截取 1-3 段相近语料作风格参照。语料只能帮助判断词感，不能证明事实。

## 检查流程

1. 定位被检查文本属于引言、文献综述、理论铺垫、现实背景、研究缺口、贡献段，还是多个部分组成的开篇论证链；适用边界见 `workflow-rules.md`。
2. 区分背景事实、文献事实、作者论断。
3. 识别组织方式：主题线分组、概念论证块、缺口驱动型、机制推导型、对比型、政策脉络型、时间演进型或概念辨析型。
4. 按 `material-tier-index.md` 查找对应组织方式的素材：若为 A，比较完整结构；若为 B，删除或忽略作者、年份、政策、数据、案例和具体事实后比较判断链与句式；若为 C，只作词感参照，不用于结构通过判定。
5. 必要时搜索 `tier-c-style-corpus` 或 `tier-c-style-corpus/raw-ocr`，只取 1-3 段检查 AI 腔、用词和中文经管论文气口。
6. 按 `workflow-rules.md` 执行 13 条结构/语言/忠实检查。
7. 逐条核对现实背景事实：政策、年份、事件、数据、行业事实、企业案例、制度背景或国际形势。
8. 逐条核对文献引用准确性：方向一致、内容一致、归属正确。
9. 检查逻辑链是否完整：现实背景 -> 研究问题 -> 文献群落 -> 缺口 -> 本文贡献。
10. 按 `workflow-rules.md` 的报告格式输出检查报告；报告必须标注问题权重和验证层级，不通过时附可直接带回 `hexie-writer` 的修正清单。

## 检查覆盖

- 结构：引用位置、段首写法、转折后判断先行、连接词密度、收束句。
- 语言：标点、叙述主体、AI 腔、事实堆叠、语病、句间节奏。
- 忠实：词汇适配、删引用后逻辑连贯性。
- 现实背景：政策、年份、事件、数据、行业事实、企业案例、制度背景、国际形势。
- 文献引用：方向一致、内容一致、归属正确。

## 事实核验最低要求

- 背景事实优先用政府官网、正式政策文本、监管公告、权威数据库、国际组织、顶刊论文、NBER/CEPR working paper 或权威研究报告核验。
- 文献事实优先用用户提供的原文、PDF、MD、参考文献材料或可靠检索结果核验。
- 无法核验时填“待验证”，并写入阻塞项；不得用素材库猜。
- 已提供 PDF/MD/原文时优先本地核验；新增文献或新增背景事实才要求可靠检索或用户补充材料。

## 判定

- 13 项检查、现实背景核验、文献引用准确性全部通过，才可判定“通过”。
- 任一检查项不通过，判定“不通过”。
- 任一现实背景事实或引用为“待验证”，判定“待验证”，并暂停 writer 的终稿输出。
- 通过后不保存文件；由 `hexie-writer` 保存 `原文件名_hexie修改.md`。

## 输出责任

报告必须包含：检查范围、组织方式、检查结论、检查清单、问题权重、验证层级、现实背景核验、文献引用准确性、语料参照、违规详情、修正清单、阻塞项和被检查文本。

若存在阻塞项，写清楚阻塞事实、需要核对的问题、已有依据和建议用户补充的材料；不得用素材库猜测或替用户补齐事实。
