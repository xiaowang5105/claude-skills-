# Hexie Skills

这是面向中文经济学、管理学论文开篇写作与材料核验的一组 Claude/Codex skill。当前版本将写作入口、质检入口和共享素材库拆开：writer 负责写作与循环修正，check 负责质检报告，shared resource library 负责模板、语料和规则。

## Skills

### hexie-writer

用于改写、重组或撰写中文经管论文的开篇论证，包括：

- 引言、导论、文献综述、理论铺垫
- 现实背景、研究背景、研究问题
- 研究缺口、边际贡献、贡献段

核心流程：

1. 识别任务类型和文本位置。
2. 区分背景事实、文献事实和作者论断。
3. 读取共享资源库的资源索引、素材分层和工作规则。
4. 按 A/B/C 规则选用素材：A 学完整结构，B 去引用后学结构，C 只学词感。
5. 改写后自动进入 hexie-check。
6. 若 check 不通过，只针对问题句修正，并继续 writer-check 循环。
7. 事实无法核验时暂停，不输出假通过终稿。

### hexie-check

用于检查中文经管论文开篇论证质量，只输出报告，不直接改文。检查范围包括：

- 结构：引用位置、段首判断、文献关系、缺口收束、贡献定位
- 语言：标点、叙述主体、AI 腔、引用简洁、语病、句间节奏
- 忠实：文献方向、内容、归属、现实背景事实核验

任一现实背景事实或文献事实无法核验，判定为待验证；任一引用方向错、内容错或归属错，判定不通过。

### hexie-shared-resource-library

共享资源库，不是入口 skill。它给 writer/check 提供素材、规则和索引。

```text
hexie-shared-resource-library/
├── resource-index.md
├── material-tier-index.md
├── workflow-rules.md
├── structure-template-library/
├── limited-reference-templates/
└── low-trust-style-corpus/
```

素材分层：

- A：完整结构模板，可以学习整段结构、问题链、文献群落和收束方式。
- B：去引用后结构模板，删除作者、年份、政策、数据、案例和具体事实后，学习剩余判断链和句式。
- C：词感语料，只用于中文经管论文用词、连接方式、气口和去 AI 腔。

素材库永远不能作为当前论文的事实依据。文献发现、政策、年份、事件、数据、企业案例和国际背景必须回到用户材料、原文、PDF、MD、官方来源、权威数据库或可靠检索。

### hxpdf

用于 PDF 与 Markdown 的转换核对。适合检查 MinerU/PDF-to-MD 转换是否丢内容、错页、漏图表或 OCR 污染。

## 安装结构

复制到 Claude 或 Codex 的 skills 目录：

```text
~/.claude/skills/
├── hexie-writer/
├── hexie-check/
├── hexie-shared-resource-library/
└── hxpdf/

~/.agents/skills/
├── hexie-writer/
├── hexie-check/
├── hexie-shared-resource-library/
└── hxpdf/
```

`hexie-writer` 是日常入口；`hexie-check` 可独立检查，也会被 writer 循环调用；`hexie-shared-resource-library` 由 writer/check 按需读取。

## 依赖

- hexie-writer / hexie-check：无固定外部程序依赖，但事实核验需要用户材料、PDF/MD 或可靠检索。
- hxpdf：需要本地 PDF 解析/OCR 环境，例如 MinerU、PyMuPDF 和对应模型缓存。
