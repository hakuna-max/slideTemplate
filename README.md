# DSA Slides

## Prerequisite

为了确保文档的顺利编译和优化的编辑环境，请遵循以下推荐的设置和安装指南。

### 推荐的安装和配置

#### TeX 发行版

- 强烈推荐使用 TeX Live 或者 MacTeX 。它是一个广泛使用的、跨平台的 LaTeX 发行版，包含了编译 LaTeX 文档所需的所有程序和宏包。

#### 代码高亮支持

- 为了使用 `minted` 宏包高亮显示源代码，请确保已经通过`pip install Pygments`安装了 Pygments。`minted` 依赖于 Pygments 进行代码的语法高亮，这会显著提升您代码块的视觉效果。
- 在编译带有 `minted` 宏包的文档时，确保您的编译命令包含了`-shell-escape` 选项。这允许 LaTeX 运行外部程序 Pygments，是 minted 正常工作所必需的。

#### 字体安装

- 根据文档设置（请查看 `preamble.tex`）的需要，安装必要的字体。请点击以下链接下载相关字体
  - [Source Han Serif SC](https://github.com/adobe-fonts/source-han-serif/releases/download/2.002R/09_SourceHanSerifSC.zip)
  - [Source Han Sans CN](https://github.com/adobe-fonts/source-han-sans/releases/download/2.004R/SourceHanSansCN.zip)
  - [Source Han Mono SC](https://github.com/adobe-fonts/source-han-mono/releases/download/1.002/SourceHanMono.ttc)
  - [Source Code Pro](https://github.com/adobe-fonts/source-code-pro/releases/download/2.042R-u%2F1.062R-i%2F1.026R-vf/OTF-source-code-pro-2.042R-u_1.062R-i.zip)
  - [Fira Mono](https://fonts.google.com/specimen/Fira+Mono?query=Fira+Mono)
  - [Fira Sans](https://fonts.google.com/specimen/Fira+Sans)

#### 编辑器推荐

- 建议使用 Visual Studio Code 配合 LaTeX Workshop 扩展进行编辑。

#### svg图片支持

- 本文档使用了svg包来加载svg图片，该包需要inkscape，可以通过以下方式安装

```bash
# mac
brew install --cask inkscape

# windows
scoop install extras/inkscape
```

- 由于已安装inkscape，可以通过inkscape将svg图片转换为位图，比如pdf格式，然后在文档中使用

## 执行脚本来编译每个章节

### 准备 LaTeX 文档

确保`main.tex`文件能够接受外部定义的日期和子标题。修改`main.tex`文件如下

```tex
\documentclass[10pt]{beamer}

\input{preamble.tex}

\title{数据结构与算法}
\subtitle{\mysubtitle} % 使用外部定义的子标题
\date{\mydate}        % 使用外部定义的日期

\author{张露平（zlp@upc.edu.cn）}
\institute{经济管理学院-中国石油大学（华东）}

\begin{document}

\maketitle

\input{ch/ch01.tex} % 假设每个章节都在单独的文件中

\end{document}
```

### Bash 脚本

在项目目录中创建一个`Bash`脚本（例如 `compile_chapters.sh`），并添加以下内容：

```bash
#!/bin/bash

# 定义所有日期的数组
dates=("April 22, 2024" "April 29, 2024" "May 06, 2024" "May 13, 2024" "May 20, 2024" "May 27, 2024" "June 03, 2024" "June 10, 2024")

# 定义所有子标题的数组
subtitles=("引言" "抽象数据类型与Python类" "线性表" "字符串" "栈与队列" "二叉树和树" "图" "字典和集合" "排序")

# 确保数组长度匹配
i=0
for chapter in ch/*.tex; do
    filename=$(basename "$chapter" .tex)

    # 获取对应的日期和子标题
    date=${dates[$i]}
    subtitle=${subtitles[$i]}

    # 编译单独的章节
    xelatex -shell-escape -jobname="output_$filename" "\\def\\mydate{$date}\\def\\mysubtitle{$subtitle}\\input{main.tex}"

    # 移动输出文件（如果需要）
    # mv "output_$filename.pdf" path/to/your/output/directory

    ((i++))
done
```

### 使脚本可执行

在终端中，切换到脚本所在的目录，并运行以下命令使脚本可执行：

```bash
chmod +x compile_chapters.sh
```

这个脚本会遍历 `ch` 目录下的所有 `.tex` 文件，为每个文件设置特定的日期和子标题，并编译成单独的 PDF 文件。

### 注意事项

确保 `dates` 和 `subtitles` 数组中的元素数量与章节文件的数量相匹配。
如果章节文件不在 `ch`目录下，或者有不同的命名规则，请相应地调整脚本中的路径和文件名。
这个脚本假设 LaTeX 环境已配置`xelatex`并支持`-shell-escape` 选项。

#### 运行脚本

执行脚本来编译每个章节：

```bash
./compile_chapters.sh
```

## Problem

### `[fragile]`

在 Beamer 中，当在 frame 环境中使用`minted`环境或其他不兼容的环境时，如

```tex
    \begin{minted}{python}
        def sqrt(x):
            y = 1.0
            e = 1e-6
            while abs(y * y - x) > e:
                y = (y + x/y)/2
            return y
    \end{minted}
```

需要在`\begin{frame}`后添加`[fragile]`选项。这是因为这些环境在背后使用了“敏感”命令，这些命令可能会被 Beamer 的帧解析机制破坏。默认情况下，Beamer 对帧内容的处理方式假定内容不包含“脆弱”命令或环境。如果不加`[fragile]`选项，当 Beamer 遇到`minted`环境这样的代码块时，它可能无法正确处理，导致编译错误。`[fragile]`选项告诉 Beamer 更改其处理此帧内容的方式，以避免破坏那些敏感的命令。具体来说，它允许帧内容包含原样文本（`verbatim` text）或执行文件写入操作，这两者都是`minted`环境在运行时需要执行的操作。因此，当使用 minted 或类似的包（如 `verbatim`、`lstlisting` 等）来展示源代码时，需要加上`[fragile]`选项来确保 Beamer 可以正确处理你的代码块。这样做的代价是稍微增加了编译时间，因为 Beamer 必须采用一种更安全的方式来处理这些帧。

### 在 latex workshop internal viewer 中使用正反向搜索

为了使正反向搜索功能按预期工作，应该保留编译时生成的`*.synctex.gz`文件，否则不起效果。

### 在 minted 环境中使用 overlay 的解决方案

```tex
    \begin{overprint}
        \onslide<1>
        \begin{minted}{python}
           class Toy(object):
                def __init__(self):
                    self._elems = []
                def add(self, new_elems):
                    """new_elems is a list"""
                    self._elems += new_elems
                def __len__(self):
                    return len(self._elems)
                def __add__(self, other):
                    new_toy = Toy()
                    new_toy._elems = self._elems + other._elems
                    return new_toy
        \end{minted}

        \onslide<2>
        \begin{minted}[firstnumber=last]{python}
                def __eq__(self, other):
                    return self._elems == other._elems
                def __str__(self):
                    return str(self._elems)
                def __hash__(self):
                    return id(self)
        \end{minted}

    \end{overprint}
```
