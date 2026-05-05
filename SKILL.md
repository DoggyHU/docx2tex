---
name: docx2tex
description: >
  将 .docx 文件（Word / WPS）转换为 LaTeX .tex 项目。
  适用场景：(1) 用户要求"docx 转 tex"、"Word 转 LaTeX"、"把文档转成 LaTeX"；
  (2) 需要从 WPS/Word 文档中提取内容用于 LaTeX 写作；
  (3) 处理中文通用报告中的表格密集型文档。
  支持：文本内容、表格（tabularray longtblr）、图片、章节结构（WPS 自定义样式识别）。
  注意：不支持 .doc（二进制旧格式）和 PDF——需要先预处理转换。
triggers:
  - docx 转 tex
  - docx2tex
  - Word 转 LaTeX
  - Word转LaTeX
  - LaTeX
  - latex
  - tex
  - 文档转 LaTeX
  - 转换 docx
  - WPS 转 LaTeX
---

# docx2tex

将 `.docx` 文件（Word / WPS）转换为可编译的 LaTeX `.tex` 项目，包含文本、表格、图片和章节层级。

## 环境要求

- .NET SDK **8.0+**（10.0 亦可）
- LaTeX 发行版（需 `xelatex`、`ctex`、`tabularray`）

## Setup（首次使用）

```bash
cd ~/.openclaw/skills/docx2tex/scripts/docx2tex
dotnet restore
```

## 使用方法

```bash
cd ~/.openclaw/skills/docx2tex/scripts/docx2tex
dotnet run -- <input.docx>
```

在输入文件同目录生成 `{文件名}-tex/` 文件夹：

```
{文件名}-tex/
├── main.tex            # 单文件，含有全部内容
├── format.sty          # 独立格式文件（tabularray + grid lines）
├── .latexmkrc          # xelatex 配置（VSCode LaTeX Workshop 用）
├── .vscode/settings.json
├── figures/            # 提取的图片
└── ref.bib
```

### 编译 PDF

```bash
cd {文件名}-tex/
latexmk -xelatex main.tex
# 或用 VSCode 打开 main.tex 按 Ctrl+Alt+B
```

### 接入已有项目

converter 输出的 `main.tex` 使用 `report` 类 + `\usepackage{format}`。
若要使用项目统一的 `format.sty`（如 `已有项目/format.sty`）：

1. `\documentclass[12pt,a4paper]{report}` → `\documentclass[12pt,a4paper,fontset=windows,AutoFakeBold=3]{ctexrep}`
2. `\usepackage{format}` → `\input{../../format.sty}`
3. `\begin{document}` 后加 `\踵足（chapter=N）`（N = 章节编号）
4. 若有 TIFF 图片：`python -c "from PIL import Image; Image.open('fig.tiff').convert('RGB').save('fig.png')"` 并将 `.tiff}` → `.png}`

## 章节结构识别（两路并行）

### 路径 A：样式优先（styleId）

| Style ID | 映射 | 示例 |
|----------|------|------|
| `1` | `\section{}` | 一、项目概述 |
| `2` | `\subsection{}` | （一）项目背景 |
| `4` | `\subsubsection{}` | 1. 研究意义 |

### 路径 B：文本兜底（无样式时）

- `一、` → `\section{}`
- `（一）` → `\subsection{}`
- `1.` / `1．` → `\subsubsection{}`

**WPS 文档特别重要**：WPS 常使用非标准自定义样式，文本兜底是识别章节的主要依靠。

## 表格处理

所有表格使用 `tabularray` `longtblr`，特性：

### 结构保真
- **合并单元格**：列合并 `GridSpan` → `\SetCell[c=N]{c}`，行合并 `VerticalMerge` → `\SetCell[r=N]{c}`
- **表头跨页**：自动检测 `rowhead`，跨页表格自动重复表头
- **网格线**：`hlines, vlines` 每表独立，忠实还原 Word/WPS 外观
- **跨页英文提示消除**：在 `\AtBeginDocument` 中设置 `\SetTblrTemplate{conthead}{empty}{}` 等，去掉 "Continued" 英文提示

### 视觉保真
- **列宽**：`X[对齐, 权重]` 按比例分配，参考 DOCX `tblGrid` 实际宽度
- **列对齐**：根据单元格段落 `jc`（居左/居中/居右）投票决定
- **数字断点**：`数字~数字`、`数字-数字`、`数字/数字` 中插入 `\allowbreak{}`，防止窄列数字溢出
- **字号**：表格全局统一字号（检测所有单元格 `rPr/sz` 取众数）。≥8 列自动 `\small`，≥12 列自动 `\footnotesize`
- **列间距**：`\SetTblrInner{columns = {colsep=2pt}}` 将默认 6pt 压缩至 2pt，多列表格可腾出约 1.7cm 宽度

## 图片处理

- **XML 直接扫描**：扫描段落 XML 中的 `a:blip` 引用（Word 和 WPS 均可靠）
- **自动配图注**：将 `fig_caption` 文字块与后续图片配对
- **图片提取**：`figures/` 目录下保存为 `figure1.jpg`、`figure2.png` 等

## 表格溢出修复（列宽不够时）

窄列中的长字符串（如 7 位数字 `1557053`）会溢出到相邻单元格。按以下顺序修复：

| 优先级 | 修复 | 命令 |
|--------|------|------|
| 1 | 列间距压缩至 2pt | 在 preamble 加 `\SetTblrInner{columns = {colsep=2pt}}` |
| 2 | 表格字号缩小 | `sed -i 's/fontsize{11pt}/fontsize{9pt}/g' main.tex` |
| 3 | 调整列权重 | 修改 `colspec` 中对应列的权重数字 |
| 4 | 横向排版 | 用 `\begin{landscape}...\end{landscape}` 包裹表格 |
| 5 | 数字内断行 | `1557053` → `1557\allowbreak{}053` |

> **经验**：长表格（10+ 列）场景下，`colsep=2pt` + 字号 11pt→9pt 组合可解决大部分单元格溢出问题。

批量修复脚本：

```bash
# 字号 11pt → 9pt
sed -i 's/font=\\fontsize{11pt}{1.2\\baselineskip}\\selectfont/font=\\fontsize{9pt}{1.1\\baselineskip}\\selectfont/g' main.tex
# 字号 10.5pt → 9pt
sed -i 's/font=\\fontsize{10.5pt}{1.2\\baselineskip}\\selectfont/font=\\fontsize{9pt}{1.1\\baselineskip}\\selectfont/g' main.tex
# 字号 10pt → 9pt
sed -i 's/font=\\fontsize{10pt}{1.2\\baselineskip}\\selectfont/font=\\fontsize{9pt}{1.1\\baselineskip}\\selectfont/g' main.tex
```

## 已知局限

- **窄列数字溢出**：10+ 列表格中长数字串可能溢出，参见上方修复手册
- **SVG 图片**：提取为 `.bin`（XeLaTeX 不支持），需转 PDF/PNG
- **TIFF 图片**：XeLaTeX 无法确定边界框，需先转 PNG
- **`.doc` 旧格式**：不支持，先用 LibreOffice 转为 `.docx`
- **单元格自定义边框**：`tcPr/tcBorders` 暂未读取，统一用 `hlines, vlines` 网格线替代
- **行高**：`trPr/trHeight` 暂未读取
