# Hexie Skills

Hexie Skills 是一组用于中文经济学、管理学论文**开篇论证**的 Claude/Codex skills。

这里的“开篇论证”不是单独指文献综述，而是指一篇论文开头部分的完整论证链：

```text
现实背景 -> 研究问题 -> 文献群落 -> 研究缺口 -> 本文贡献
```

它覆盖引言、导论、现实背景、研究背景、理论铺垫、文献综述、研究问题、研究缺口、边际贡献和贡献段。文献综述只是其中一环；Hexie 的目标是让这些部分之间形成清楚、克制、可核验的学术推进。

## Published Skills

本仓库发布三个目录：

```text
hexie-writer/
hexie-check/
hexie-shared-resource-library/
```

- `hexie-writer`：写作入口，负责写作、改写、选文献、整合现实背景、吸收检查报告并产出终稿。
- `hexie-check`：检查入口，只输出质量检查报告，不直接改文。
- `hexie-shared-resource-library`：共享资源库，保存模板、语料、素材分层和 writer/check 共用规则。

本仓库不包含 `hxpdf`。

## What Hexie Helps With

Hexie 适合处理论文开头的论证组织问题，例如：

- 引言只有现实背景，没有自然引出研究问题。
- 现实背景写成政策、年份、数据、事件堆叠。
- 文献综述写成作者清单，缺少文献群落和关系。
- 研究缺口不能从现实背景和已有文献自然推出。
- 边际贡献写成空泛套话。
- 引用方向、文献发现或作者归属不可靠。
- 语言过于 AI 化，不像中文经管论文的表达。

Hexie 不适合套用到：

- 数据处理
- 模型设定
- 实证结果
- 稳健性检验
- 机制检验
- 异质性分析
- 结论
- 政策建议

这些任务可以普通协助，但不套用 Hexie 的开篇论证模板。

## `hexie-writer`

`hexie-writer` 是日常入口。适合用户提出“改引言”“改文献综述”“重写研究缺口”“补边际贡献”“让开篇更像中文经管论文”等请求。

它的工作方式来自已发布的 `SKILL.md`：

1. 定位任务属于引言、文献综述、理论铺垫、现实背景、研究缺口、贡献段，还是多个部分组成的开篇论证链。
2. 前置检查并区分三类内容：背景事实、文献事实、作者论断。
3. 诊断论证链：现实背景 -> 研究问题 -> 文献群落 -> 缺口 -> 本文贡献。
4. 必要时给出多个开篇组织方案，等待用户选择。
5. 按素材 A/B/C 分层读取模板和语料。
6. 逐句改写，并核验背景事实、文献事实和作者论断。
7. 自动运行 `hexie-check`。
8. 根据 check 修正清单定点修改，再次检查，直到通过或遇到事实阻塞。

`hexie-writer` 的正常终点只有一个：`hexie-check` 全部通过。  
事实无法核验、材料缺失、引用无法确认或背景来源不足时，writer 必须暂停并列出阻塞项。

## `hexie-check`

`hexie-check` 是审稿式检查 skill。它只检查，不改文，不生成终稿，也不生成 `.docx`。

它检查的不是“文字是否顺”，而是开篇论证是否可信、连贯、像中文经管论文：

- 结构：引用位置、段首写法、转折后判断先行、连接词密度、收束句。
- 语言：标点、叙述主体、AI 腔、事实堆叠、语病、句间节奏。
- 忠实：删引用后逻辑是否连贯，词汇是否符合中文经管论文习惯。
- 现实背景：政策、年份、事件、数据、行业事实、企业案例、制度背景、国际形势。
- 文献引用：方向一致、内容一致、归属正确。

判定规则：

- 13 项检查、现实背景核验、文献引用准确性全部通过，才可判定“通过”。
- 任一检查项不通过，判定“不通过”。
- 任一现实背景事实或引用为“待验证”，判定“待验证”，并暂停 writer 的终稿输出。

## Writer-Check Loop

Hexie 的自动闭环由 `hexie-writer` 控制，不由 `hexie-check` 自己循环。

```text
用户调用 hexie-writer
        ↓
writer 前置检查：背景事实 / 文献事实 / 作者论断
        ↓
writer 第一次写作或改写
        ↓
writer 进入 hexie-check
        ↓
check 输出单轮报告：通过 / 不通过 / 待验证 + 修正清单
        ↓
writer 按修正清单定点修改
        ↓
writer 再次进入 hexie-check
        ↓
重复，直到 check 全部通过或事实阻塞
```

`hexie-check` 单独调用时只输出一次检查报告；它不改文，也不自动发起第二轮。第二轮及之后的检查仍由 `hexie-check` 执行，但“再次检查”这个动作由 `hexie-writer` 发起。

因此：

```text
测 hexie-check = 测单轮审稿
测 hexie-writer = 测完整写-查-改-再查闭环
```

## `hexie-shared-resource-library`

`hexie-shared-resource-library` 是 writer/check 的共享资源库，不是入口 skill。

```text
hexie-shared-resource-library/
├── resource-index.md
├── material-tier-index.md
├── workflow-rules.md
├── tier-a-complete-structure-templates/
├── tier-b-skeleton-structure-templates/
└── tier-c-style-corpus/
```

其中：

- `resource-index.md` 说明资源库的用途和事实边界。
- `material-tier-index.md` 规定“写法类型 × 素材质量”的二维索引。
- `workflow-rules.md` 保存适用范围、事实来源优先级、八种组织方式、writer-check 循环、检查权重、13 条检查清单和报告格式。
- `tier-a-complete-structure-templates/` 是 A 档完整结构模板入口；当前可为空，只放人工筛出的干净段落。
- `tier-b-skeleton-structure-templates/` 是 B 档去事实后结构骨架入口；当前 01/02/03/04/05/06/08 主要在这里。
- `tier-c-style-corpus/` 是 C 档词感语料入口；当前 07、历史大素材和 MinerU/OCR 原始材料在这里。

## Material Tiers

素材库采用二维结构：8 种写法是横向分类，A/B/C 是纵向质量等级。每一种写法理论上都可以有 A、B、C 三档素材；A/B/C 不代表写法本身好坏，只决定“AI 怎么安全使用素材”。

| Tier | Name | Can be used for | Cannot be used for |
|------|------|-----------------|--------------------|
| A | 完整结构模板 | 整段结构、问题链、文献群落、贡献定位、句层节奏、收束方式 | 当前论文事实依据；照搬原文 |
| B | 去引用后结构模板 | 删除作者、年份、政策、数据、案例、具体结论后的判断链、句式节奏、连接方式 | 引用事实、背景事实、整段原样模仿 |
| C | 词感语料 | 词语、短语、连接方式、中文经管论文气口 | 结构模板、引用事实、背景事实、标点和引用格式 |

简单说：

```text
A = 学完整结构
B = 去事实/引用后学结构
C = 只学词语和气口
```

无论素材属于哪一档，都不能作为当前论文的事实依据。政策、年份、事件、数据、企业案例、制度背景、国际形势和文献发现，必须回到用户材料、原文、PDF、MD 或可靠来源核验。

## Organization Patterns

共享规则库包含八种开篇组织方式：

1. 主题线分组
2. 概念论证块
3. 缺口驱动型
4. 机制推导型
5. 对比型
6. 政策脉络型
7. 时间演进型
8. 概念辨析型

这些组织方式服务于开篇论证链，而不是只服务于文献综述。比如政策脉络型可以用于现实背景和研究问题之间的过渡；机制推导型可以用于理论铺垫；缺口驱动型可以用于引言收束和贡献引出。

## Fact Verification

Hexie 的硬规则是：不得无中生有。

现实背景事实优先使用：

- 政府官网、正式政策文本、监管公告
- 国家统计局、海关总署、Wind、CSMAR、CNRDS、WIPO、World Bank、OECD、IMF、UNCTAD 等
- 官方公告、国际组织、顶刊论文、NBER/CEPR working paper、权威研究报告
- 企业公告、年报、监管文件、权威数据库或正式新闻稿

文献事实必须回到用户提供的原文、PDF、MD、参考文献材料，或可靠检索结果。素材库不能替代事实核验。

## Installation

复制三个目录到 Claude 或 Codex 的 skills 目录。

Claude:

```text
~/.claude/skills/
├── hexie-writer/
├── hexie-check/
└── hexie-shared-resource-library/
```

Codex:

```text
~/.agents/skills/
├── hexie-writer/
├── hexie-check/
└── hexie-shared-resource-library/
```

目录名需要保持不变。`hexie-writer` 和 `hexie-check` 会按相对路径查找 `hexie-shared-resource-library`。

## Usage Examples

```text
帮我改这段引言，让现实背景更自然地引出研究问题。
```

```text
这段开篇论证太散了，帮我重组为“现实背景 -> 研究问题 -> 文献群落 -> 缺口 -> 贡献”的结构。
```

```text
帮我检查这段引言里的政策年份、行业事实和文献引用是否可靠。
```

```text
这段文献综述像作者清单，帮我改成有文献关系的开篇论证。
```

```text
帮我重写研究缺口和边际贡献，但不要编造文献发现。
```

## Repository Structure

```text
README.md
.gitignore
hexie-writer/
hexie-check/
hexie-shared-resource-library/
```

`hexie-writer` 是日常写作入口。  
`hexie-check` 可独立检查，也会被 writer 在循环中调用。  
`hexie-shared-resource-library` 由 writer/check 按需读取。
