---
name: hxpdf
description: Use when the user wants to check if an md file faithfully reproduces its source PDF, verify no content was lost during PDF-to-MD conversion, or says "检查这篇" "对照一下" "有没有丢内容" "比对pdf和md". If no md exists, auto-convert PDF to md first before checking.
---

# PDF-MD 对比检查

## 目标

确认 md 与 PDF 原文**无差别**——不丢字、不加字、不改字。若 md 不存在，自动用 MinerU 3.x 将 PDF 转为 md 后再进入对照流程。

## 前置依赖

- **MinerU 3.x**（命令 `mineru`），安装参见 [MinerU 官方仓库](https://github.com/opendatalab/MinerU)
- 模型通过 ModelScope 下载：`SET MINERU_MODEL_SOURCE=modelscope && mineru-models-download`

## 双引擎架构

| 引擎 | 角色 | 特点 |
|------|------|------|
| PyMuPDF | 基准参照 | 快速、忠实，读 PDF 内部文字层 |
| MinerU 3.x | PDF→MD 转换 + 辅助补抽 | 处理 PyMuPDF 抽不出的异常页面，高质量 MD 输出 |

## 单篇流程

```
1. 定位 PDF，检查是否存在对应 md：
   - 有 md → 直接进入步骤 2
   - 无 md → mineru -p PDF路径 -o 输出目录，取生成的 .md 放到 PDF 同目录
2. PyMuPDF 全篇抽取（快速、忠实）
3. 逐页异常检测 → 标记可疑页/破损页
4. MinerU 补抽（仅异常页）
5. 逐页交叉验证：
   - 正常页：PyMuPDF 对照 MD
   - 异常页：MinerU 为主、PyMuPDF 为辅，双参照对照 MD
   - 两引擎结果不一致 → 标记高风险区，重点人工核查
6. grep "（ ）" 快速扫描空括号
7. grep " ， " 扫描孤立逗号
8. grep "[a-zA-Z]" 扫描英文/数字残留
9. 对照 PDF 逐处修正差异
10. 再扫描，确认零命中
11. 报告修正数
```

**每一篇都必须走完，禁止跳过。**

## 步骤详解

### 1. PyMuPDF 全篇抽取

```bash
python << 'PYEOF'
import fitz
pdf = fitz.open(r"PDF完整路径")
with open(r"C:\Users\hexiezuel\Desktop\pdf_extract_sample.txt", "w", encoding="utf-8") as f:
    for i in range(pdf.page_count):
        text = pdf[i].get_text()
        char_count = len(text.strip())
        f.write(f"\n===== PAGE {i+1} (chars: {char_count}) =====\n")
        f.write(text)
PYEOF
```

### 2. 逐页异常检测

检查每页抽出的文字量（chars 数）：
- **文字量 < 50 字符** → 标记"可疑页"
- **出现连续乱码**（如 `□□□`、不可打印字符连片）→ 标记"破损页"
- 其余 → 正常页，以 PyMuPDF 为准

### 3. MinerU 补抽（仅异常页）

对异常页范围单独解析：
```bash
mineru -p "PDF完整路径" -o "输出目录" -s 异常起始页 -e 异常结束页
```

### 4. MinerU 批量转 MD（无 md 时的初始化）

```bash
# 单篇
mineru -p "PDF路径" -o "输出目录"

# 批量
SET MINERU_MODEL_SOURCE=modelscope
mineru -p "PDF所在目录" -o "输出目录"
```
输出为 Markdown，取生成结果放到 PDF 同目录即可进入对照流程。

### 5. 交叉验证

- **正常页**：PyMuPDF 提取结果 ↔ MD 对比
- **异常页**：magic-pdf 提取结果（主） + PyMuPDF（辅） ↔ MD 对比
- **两引擎结果不一致处**：标记为高风险区，需对照原始 PDF 页面人工判断

### 6. 快速扫描

```bash
grep "（ ）" md文件    # 空括号 = 内容丢失
grep " ， " md文件    # 孤立逗号 = 前面丢了东西
grep "[a-zA-Z]" md文件  # 英文/数字残留检查
```

## 常见丢失模式

中英文混排处的数字、英文名、技术参数最容易被丢弃：

| 丢失内容 | PDF 原文 | md 输出 |
|----------|----------|---------|
| 引用年份 | "安同良等（2023a）" | "安同良等（ ）" |
| 英文/数字 | "DVD、5G和高铁" | " 、 和高铁" |
| 英文名+年份 | "（Teece，1986）" | "（ ， ）" |
| 技术参数 | "10—22nm，6%，10nm" | "10— ， ， " |
| 统计数据 | "22363家企业" | " 家企业" |
| 表号列号 | "表4列（1）—（6）" | "表 列（ ）—（ ）" |

## 其他差异

| 类型 | 处理 |
|------|------|
| 页眉/页码/期号混入 | 删除 |
| 引用年份折行 | 合并回一行 |
| 段落拆散 | 合并恢复 |
| 表格/公式变形 | 对照 PDF 修正 |
| 句子遗漏/字符误识 | 对照 PDF 补回 |

## 批量处理

逐篇执行单篇流程，全部完成后汇总：总篇数、每篇正常页/异常页/修正数、遗留问题。
