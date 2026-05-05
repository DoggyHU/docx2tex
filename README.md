# docx2tex

> 将 `.docx` 文件（Word / WPS）转换为 LaTeX `.tex` 项目

## 核心特性

- **OpenXML 解析**：基于 OpenXML 标准解析 Word/WPS 文档，无需 Office 或 WPS 运行时
- **表格保真**：使用 `tabularray` 的 `longtblr` 环境，保留合并单元格、网格线、跨页表头
- **图片提取**：自动从文档中提取图片并配图注
- **中文章节识别**：支持 WPS 自定义样式 + 文本模式兜底（`一、`、`（一）`、`1.` 等）
- **格式独立**：`format.sty` 与 `main.tex` 分离，便于接入已有 LaTeX 项目

## 环境要求

- [.NET SDK](https://dotnet.microsoft.com/download) **8.0+**（10.0 亦可）
- LaTeX 发行版（需 `xelatex`、`ctex`、`tabularray`）

## 快速开始

### 安装依赖

```bash
cd ~/.openclaw/skills/docx2tex/scripts/docx2tex
dotnet restore
```

### 转换文档

```bash
dotnet run -- /path/to/your-document.docx
```

转换后在同目录生成 `{文件名}-tex/` 文件夹：

```
{文件名}-tex/
├── main.tex          # 单文件，含有全部内容
├── format.sty        # 独立格式文件
├── .latexmkrc        # xelatex 配置
├── .vscode/settings.json
├── figures/          # 提取的图片
└── ref.bib
```

### 编译 PDF

```bash
cd {文件名}-tex/
latexmk -xelatex main.tex
# 或用 VSCode 打开 main.tex 按 Ctrl+Alt+B
```

## 接入已有 LaTeX 项目

若要使用项目统一的 `format.sty`，参考 SKILL.md 中的详细步骤，主要包括：

1. 将 `report` 类替换为 `ctexrep`
2. `\usepackage{format}` 替换为 `\input{../../format.sty}`
3. 根据章节编号调整文档结构

## 表格溢出修复

多列表格（10+ 列）可能出现单元格内容溢出，参考 SKILL.md 中的修复优先级：

1. 列间距压缩至 2pt
2. 表格字号 11pt → 9pt
3. 调整列权重
4. 横向排版
5. 数字内断行

## 项目结构

```
docx2tex/
├── SKILL.md                    # OpenClaw Skill 说明文档
├── README.md                   # 本文件
├── scripts/
│   └── docx2tex/
│       ├── Docx2Tex.csproj    # .NET 项目文件
│       └── Program.cs          # 核心解析逻辑
```

## 致谢

本项目的实现思路受 [MiniMax docx skill](https://github.com/minimax-open/x-crawl) 开源项目的启发。在 OpenClaw 生态下，如何将结构化文档内容转换为目标格式是一个常见需求，本工具尝试从 OpenXML 层面提供一种可定制的中文 LaTeX 转换方案。

## License

MIT
