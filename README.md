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
- `hexie-shared-resource-library`：共享资源库，保存句子卡片、段落结构语料和 writer/check 共用规则。

本仓库不包含 `hxpdf`。

当前发布版以 `hexie-shared-resource-library/index.md` 和 `how-to-use.md` 为资源入口。旧版的 `examples.md`、`workflow-rules.md`、`material-tier-index.md` 以及 `tier-a/tier-b/tier-c` 目录不再作为运行结构；如果本地仍看到这些名称，通常表示安装目录还没有同步到最新版。

## Current Design

新版 Hexie 的核心不是“找一篇模板照着写”，而是把开篇论证拆成三个层次：

```text
事实可核验 -> 段落论证能推进 -> 句子读起来像中文经管论文
```

对应到资源和检查流程：

- 第一轮看事实：现实背景和文献事实必须能回到原文、PDF、MD、用户材料或可靠来源。
- 第二轮看段落：用 B 档段落结构语料辅助判断论证链、文献关系、缺口和收束。
- 第三轮看句子：用 A 档句子卡片辅助处理词感、节奏、收束力、关联词位置和 AI 腔。

因此，A/B 不是论文质量评价，也不是写法优劣评价。A 是句子级技法卡片，B 是段落级结构语料；两者都不能替代事实核验。

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
5. 判断文献是否充分；不足时按国内五大、国外顶刊、NBER/CEPR/SSRN 优先级补文献。
6. 第二轮动笔前读取 B 档段落结构语料，学习论证骨架。
7. 第三轮逐句修时读取 A 档句子卡片，学习词感、节奏和收束力。
8. 自动运行 `hexie-check` 三轮检查，并根据每轮修正清单定点修改，直到三轮全部通过或遇到事实阻塞。

`hexie-writer` 的正常终点只有一个：`hexie-check` 全部通过。  
事实无法核验、材料缺失、引用无法确认或背景来源不足时，writer 必须暂停并列出阻塞项。

## `hexie-check`

`hexie-check` 是审稿式检查 skill。它只检查，不改文，不生成终稿，也不生成 `.docx`。

它检查的不是“文字是否顺”，而是开篇论证是否可信、连贯、像中文经管论文。新版 `hexie-check` 拆成三轮顺序检查：

- 第一轮：事实与忠实。检查引用方向、引用归属、事实核验、无编造无夸大、素材是否误用。
- 第二轮：结构与论证。检查文献关系是否入文、逻辑链是否完整、缺口是否自然推出、段首判断、引用位置、收束句和文献充分性。
- 第三轮：语言与节奏。检查长短差、收束力、破折号、转折后判断先行、连接词方式、AI 腔、事实堆叠、语病和删引用后逻辑。

判定规则：

- 第一轮不过，不进入结构和语言修改。
- 第二轮不过，不进入语言润色。
- 第三轮通过后，才可判定终稿通过。
- 任一现实背景事实或引用为“待验证”，判定“待验证”，并暂停 writer 的终稿输出。

## Writer-Check Loop

Hexie 的自动闭环由 `hexie-writer` 控制，不由 `hexie-check` 自己循环。三轮顺序固定，不能跳。

```text
用户调用 hexie-writer
        ↓
writer 前置检查：背景事实 / 文献事实 / 作者论断
        ↓
writer 输出文献关系表：并列 / 递进 / 冲突 / 补充 / 机制承接 / 情境差异
        ↓
若现有文献不足，按国内五大、国外顶刊、NBER/CEPR/SSRN 优先级补文献
        ↓
writer 写作或改写
        ↓
hexie-check 第一轮：事实与忠实
        ↓
writer 定点修正，再重跑第一轮，直到通过
        ↓
hexie-check 第二轮：结构与论证
        ↓
writer 定点修正，再重跑第二轮，直到通过
        ↓
hexie-check 第三轮：语言与节奏
        ↓
writer 定点修正，再重跑第三轮，直到通过
        ↓
终稿
```

`hexie-check` 单独调用时只输出当前轮检查报告；它不改文，也不生成终稿。writer 每轮修改后主动调用 `hexie-check`，check 只负责检查当前维度。

凡涉及 2 篇及以上文献，第一阶段必须先产出文献关系表，并在第二轮结构与论证检查中确认关系已经通过句间逻辑进入正文。若文献不足却没有补文献，不能进入语言润色，也不能判定通过。

因此：

```text
测 hexie-check = 测单轮审稿
测 hexie-writer = 测完整写-查-改-再查闭环
```

## `hexie-shared-resource-library`

`hexie-shared-resource-library` 是 writer/check 的共享资源库，不是入口 skill。

```text
hexie-shared-resource-library/
├── index.md
├── how-to-use.md
├── resource-index.md
├── a-sentence-cards/
└── b-structure-corpus/
```

其中：

- `index.md` 说明 A/B 两层资源分工。
- `how-to-use.md` 保存组织方式、前置输出、三轮检查、事实来源和选文献规则。
- `resource-index.md` 是兼容索引，便于旧调用找到资源说明。
- `a-sentence-cards/` 是 A 档句子卡片，按收束力、长短落差、关联词位置、判断先行、无标签推进、气口与词感分类。
- `b-structure-corpus/` 是 B 档段落结构语料，按缺口驱动、机制推导、概念论证块、主题线分组、对比型、政策脉络、概念辨析分类，并保留 raw-papers 作为原始论文入口。

## Material Layers

素材库采用两层结构：

| Layer | Name | Used for | Not used for |
|------|------|----------|--------------|
| A | 句子卡片 | 第三轮逐句修文本时学习词感、节奏、收束力、关联词位置和无标签推进 | 事实依据、照搬原文、替代引用核验 |
| B | 段落结构语料 | 第二轮结构与论证时学习判断链、段落推进和论证骨架 | 事实依据、直接照搬内容 |

简单说：

```text
A = 学句子，解决语言气口和节奏
B = 学段落，解决论证推进和结构
```

无论 A 档还是 B 档，都不能作为当前论文的事实依据。政策、年份、事件、数据、企业案例、制度背景、国际形势和文献发现，必须回到用户材料、原文、PDF、MD 或可靠来源核验。

## Organization Patterns

B 档段落结构语料包含七种开篇组织方式：

1. 缺口驱动
2. 机制推导
3. 概念论证块
4. 主题线分组
5. 对比型
6. 政策脉络
7. 概念辨析

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

## Maintainer Notes

如果你要维护或发布新版 skill，建议遵守这几条规则：

- 先改维护源，再同步到运行目录和 GitHub。
- `.claude/skills` 和 `.agents/skills` 是安装/运行目录，不建议在里面执行 `git pull`、`git stash`、`git reset`、`git clean` 等 Git 操作。
- 排查版本时，优先读取实际文件内容：`hexie-writer/SKILL.md`、`hexie-check/SKILL.md`、`hexie-shared-resource-library/index.md`、`hexie-shared-resource-library/how-to-use.md`。
- 如果运行目录版本可疑，重新复制或同步三个发布目录，然后重启 Claude/Codex 会话。
- GitHub 发布包只需要包含 `hexie-writer/`、`hexie-check/`、`hexie-shared-resource-library/`、`README.md` 和必要的 `.gitignore`。
