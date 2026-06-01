# Hexie — 文献综述全流程写作工具

一套 Claude Code skill，用于中文经济学/管理学论文的文献综述写作、修改和质量检查。

## 功能

| Skill | 用途 |
|-------|------|
| **hexie** | 全流程：选文献 → 判关系 → 范例找模板 → 套模板改文 → 联网验证 → 自动质检 |
| **hexie-check** | 四维度质量检查 + 原文含义核对 + 逻辑链检验，零差距后自动输出 md + docx |

## 安装

将 `hexie/` 和 `hexie-check/` 两个文件夹放到 `~\.claude\skills\` 下：

```
~\.claude\skills\
  hexie\
    SKILL.md
    examples.md
  hexie-check\
    SKILL.md
```

重启 Claude Code 即可。

## 使用

在对话中输入：

- "帮我改一下这段文献综述"
- "写一下这段文献综述，不要罗列文献"

hexie 会自动进入前置检查（确认大纲/文献/关系后才会动笔），改完后自动调 hexie-check 质检，零差距后交回最终文本。

## 更新

覆盖 `~\.claude\skills\hexie\` 和 `~\.claude\skills\hexie-check\` 文件夹即可。
