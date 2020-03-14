---
title: lshort阅读笔记
date: 2018-03-22 21:26:24
tags: latex
---

# lshort

## 源代码结构
源代码以一个 `\documentclass` 作为开头，它规定了文档使用的文档类
<!--more-->
```tex
\documentclass{...}
```
接着可以用 `\usepackage{...}` 调用宏包。
接着，需要用以下的一对命令来标记正文内容的开始位置和结束位置，而将正文内容写入其中：
```tex
\begin{document} 
\end{document}
```
在 `\documentclass` 和 `\begin{document}` 之间的位置称为导言区，除了使用 `\usepackage` 之外，一些对文档的全局设置命令也在这里使用。
必须清楚的几个概念:

- **引擎**: 全称排版引擎，是读入源代码并编译生成文档的程序，如 pdfTEX, XETEX。也叫编译器。
- **格式**: 是定义了一组命令的代码集。LATEX 就是一个格式，高德纳本人还编写了一个简单的 plain TEX 的格式，没有定义诸如 `\documentclass` 和 `\section` 等。
- **命令**: 是引擎和格式二者的结合体。如下文要用到的 pdflatex 命令是结合 pdfTEX 引擎和 LATEX 格式的一个命令，用于编译类似源代码并输出 PDF。(在命令行窗口输入的命令)

## 文档类
```tex
\documentclass[<options>]{<class-name>}
```

- `<class-name>` 为文档类的名称，如 article, book, report，在其基础上派生的一些文档类如支持中文排版的 ctexart、ctexbook、ctexrep，或者有其它功能的一些文档类。
- `<options>` 为文档类设置选项，以全局地影响文档布局的参数，如字号、纸张大小、单双面等等。

如下开头的文档将排版为文章，纸张为 A4 大小，基本字号为 11pt：
```tex
\documentclass[11pt,twoside,a4paper]{article}
```

## 宏包
当基础功能无法满足需求的时候, 就需要扩展的功能, 这些就叫做宏包。
```tex
\usepackage[<options>]{<package-name>}
```
`\usepackage` 的参数里可以使用不止一个宏包，多个宏包用逗号隔开。这种用法一般不要加选项。
每个宏包（包括前面所说的文档类）都定义了许多命令和环境，或者修改了 LATEX 已有的命令和环境。
使用方法是在 Windows 命令提示符或者 Linux 终端下输入命令：
```tex
texdoc <pkg-name>
```
## 大文档的组织
有大规模的源码时，可以将源代码分成若干个文件:
```tex
\include{<filename>}
```
如果编译的主文件不在一个目录中，则要加上相对路径。
可以不带扩展名，此时默认为 `.tex`。
值得注意的是 `\include` 在读入 `<filename>` 之前会另起一页。有的时候并不需要这样，而是用 `\input`，它纯粹是把文件里的内容插入：
```tex
\input{<filename>}
```
`\includeonly` 来组织文件，用于导言区，指定只载入某些文件：
```tex
\includeonly{<filename1>,<filename2>,...}
```
最后介绍实用的工具宏包 syntonly。
在导言区使用 `\syntaxonly` ，可令 LATEX 编译后不生成 DVI 或者 PDF 文档，只排查错误，编译速度会快不少：
```tex
\usepackage{syntonly} 
\syntaxonly
```

# 用 LATEX 排版文字
**ASCII 编码**：
计算机的基本存储单位是字节（byte），每个字节为八位（8-bit），能够代表 0-255 这 256 个数。ASCII （美国通用信息交换码）覆盖前 128 个数 0-127，也就是 7-bit。

**扩展编码**：
各种语言都发展出来了自己的编码，比如西欧语言的 Latin-1，日本的 ShiftJIS，中国大陆的 GB2312-80 和 GBK 等。
它们之中的绝大多数都向下兼容 ASCII，因此无论是在哪种编码下，TEX 以及 LATEX 的命令和符号都能用。
西欧（拉丁字母）、俄语系（西里尔字母）等的编码较容易处理，因为它们刚好能够利用 128255 这个编码范围。

> latex 命令及 pdflatex 命令下，这些编码的支持由 inputenc 宏包提供．
比如将文档保存为 Latin-1 编码，并在导言区使用：

```tex
\usepackage[latin1]{inputenc}
```
其它一些语言也可以用 inputenc 宏包配合 babel 宏包排版。

- GBK 等编码是多字节编码，ASCII 字符为一个字节，而非 ASCII 字符为两个字节，这就需要借助一些宏包进行较为复杂的判断和处理。
- CJK 宏包就是处理汉语等使用的宏包。但 CJK 宏包不方便，不推荐直接使用。


**UTF-8 编码**:
Unicode 是一个多国语言文字的集合，覆盖了几乎全球范围内的语言文字。
UTF-8 是 Unicode 的一套编码方案，一个字符可以由一个到四个（现在用到三个）字节编码，其中单字节范围仍然是 ASCII 字符。

## 排版中文
**XELATEX (编译器)** 

> 支持直接使用系统安装的 TrueType (.ttf) / OpenType(.otf) 等格式的字体，加上对 UTF-8 编码的原生支持，免去了预处理字体的麻烦。
在此基础上的 xeCJK 宏包更进一步完善了排版中文的一些细节，比如中英文之间插入空隙、中文行尾的回车不引入空格、标点符号不出现在行首。

支持用简单的命令配置中文字体, 源代码须保存为 UTF-8 编码:
```tex
\documentclass{article}
\usepackage{xeCJK}
\setCJKmainfont{SimSun}
\begin{document} 
 中文LaTeX排版。 
\end{document}
```

**ctex 宏包和文档类**
ctex 宏包和文档类是在 CJK 和 xeCJK 基础上的进一步封装。

- ctex 文档类包括 ctexart / ctexrep / ctexbook，是对 LATEX 的三个标准文档类的封装，对 LATEX 的排版样式做了许多调整， 以切合中文排版风格。
- 最新版本的 ctex 宏包/文档类甚至支持自动配置字体。

比如上述例子可进一步简化为:
```tex
\documentclass{ctexart} 
\begin{document} 
中文LaTeX排版。
\end{document}
```
ctex 宏包/文档类支持源代码保存为 UTF-8 和 GBK 编码，用 latex + dvipdfmx 命令、pdflatex 或 xelatex 命令（只支持 UTF-8 编码）都能够编译。
建议总是将源代码保存为 UTF-8 编码，用 xelatex 命令编译。


## LATEX 中的符号

- 空格键和 Tab 键输入的空白字符视为“空格”。连续的若干个空白字符视为一个空格。一行开头的空格忽略不计。
- 行末的回车视为一个空格；但连续两个回车，也就是空行，会将文字分段。多个空行被视为一个空行。也可以在行末使用 `\par` 分段. 
- 以下字符特殊用途，如 `%` 表示注释，`$`、`^`、`_` 等用于数学公式，`&` 用于排版表格。直接输入这些字符得不到对应的符号，需要使用反斜杠转义。
- `^` 和 `~` 两个命令的形式比较特殊，如果不加一对括号，就和后面的字符形成了重音效果. 
- `\\` 被直接定义成了手动换行的命令， 输入反斜杠就只好用  `\textbackslash`。


**连字**: 
西文排版中经常会出现连字（Ligatures），常见的有 ff、fi、fl、 ffi、ffl。
```tex
It's difficult to find \ldots .\\           %有连字
It's dif{}f{}icult to f{}ind \ldots .       %用空的分组取消连字
```
上述效果需要在 tex 环境才可以看出。

**连字号和破折号**: 
有三种长度的“横线”可用：连字号、短破折号（en-dash）和长破折号（em-dash）。
不同的用途：

- 连字号(-)用来组成复合词；
- 短破折号(--)将数字连接表示范围；
- 长破折号(---)作为破折号使用。

```tex
daughter-in-law, X-rated\\ 
pages 13--67\\ 
yes---or no?
```

**省略号**: 
LATEX 提供了命令 `\ldots` 来生成省略号，相对于直接输入三个点的方式更为合理。
`\ldots` 和 `\dots` 是两个等效的。


**波浪号**: 
`~` 命令，可以用来输入波浪号，但位置靠顶端。
可以用公式里的 `\sim` 符号来代替：
```tex
a\~{}z \qquad a$\sim$z
```


## 文字强调
`\underline` 为文字添加下划线：
```tex
An \underline{underlined} text.
```
`\underline` 会出现不同单词可能生成高低各异的下划线，并且无法换行。
`ulem` 提供的 `\uline` 能够生成自动换行的下划线。
`\emph` 用来将文字变为斜体以示强调。如果在本身已经用 `\emph `命令强调的文字内部嵌套使用 `\emph`，内部则使用正常字体的文字。

## 断行和断页
断行的位置尽可能选取在两个单词之间，也就是输入到源文件中的“空格”。
一般情况下，这个“空格”生成一个间距，它会根据行宽和文字自动调整，文字密一些的地方，单词间距就略窄，反之略宽。(**也就是说普通的空格会因为文字的疏密调整间距, 会因为断行调整空格的间距**)
可以使用字符 `~` 在合适的位置插入一个不会断行的空格（高德纳称之为 tie，“带子”), 通常用在英文人名、图表名称等场景：
```tex
Fig.~2a \\ 
Donale~E. Knuth
```


**手动断行和断页**:
手动断行：
```tex
\\ or \newline
```
`\\` 也在表格、公式等地方用于分行，`\newline` 只用于文本段落中。
断页的命令有两个：
```tex
\newpage or \clearpage
```
两个断页的区别：

- 一是在双栏排版中 `\newpage` 只起到另起一栏的作用；
- 二是涉及到浮动体的排版上行为不同

当不满足于 LATEX 默认的断行和断页位置，需要进行微调：
```tex
\linebreak[<n>]      \nolinebreak[<n>]
\pagebreak[<n>]      \nopagebreak[<n>]
```
用数字 `<n>` 代表适合、不适合的程度，取值范围为 0-4，缺省为 4。

- 比如 `\linebreak[3]` 意味着此处在断行时优先考虑；
- `\nopagebreak` 或 `\nopagebreak[4]` 意味着禁止在此处断页。

## 断词 
对于绝大部分单词，LATEX 能够找到合适的断词位置，在断开的行尾加上连字符 `-`。
如果一些没能自动断词，可以在单词内使用 `\-` 指定位置：
```tex
I think this is: su\-per\-cal\-% 
i\-frag\-i\-lis\-tic\-ex\-pi\-% 
al\-i\-do\-cious
效果:
I think this is: supercalifragilisticexpialido-
cious.
```

# 文档元素

用以划分章节、生成章节标题并自动编号：
```tex
\chapter{<title>}        \section{<title>}
\subsection{<title>}     \subsubsection{<title>}
\paragraph{<title>}      \subparagraph{<title>}
```
其中 `\chapter` 只在 book 和 report 文档类有定义。`\part` 命令用以将整个文档分割为大的分块。
上述命令除了生成带编号的标题之外，还向目录中添加条目，并影响页眉页脚的内容。每个命令有两种变体：

- 带可选参数的变体：`\section[<short title>]{<title>}` 标题使用 `<title>` 参数，在目录和页眉页脚中使用 `<short title>` 参数；
- 带星号的变体：`\section*{<title>}` 标题不带编号，也不生成目录项和页眉页脚。

较低层次如 `\paragraph` 和 `\subparagraph` 即使不用带星号的变体，生成的标题默认也不带编号，事实上，除 `\part` 外：

- article 文档类带编号的层级为 `\section`、`\subsection`、`\subsubsection` 三级；
- report、book 文档类带编号的层级为 `\chapter`、`\section`、`\subsection` 三级。


**目录**:
```tex
\tableofcontents
```
生成单独的一个章节，标题默认为 “Contents”。
`\chapter*` 或 `\section*` 这样不生成目录项的章节标题命令，而又想手动生成该章节的目录项，可以在标题命令后面使用：
```tex
\addcontentsline{toc}{<level>}{<title>}
```
其中 `<level>` 为章节层次 chapter 或 section 等，`<title>` 为出现于目录项的章节标题.


**文档结构的划分**
所有标准文档类都提供了一个 `\appendix` 将正文和附录分开，最高一级章节改为使用拉丁字母编号，从 A 开始。
book 文档类还提供了前言、正文、后记结构的划分命令：

- `\frontmatter` 前言部分，页码为小写罗马字母格式；其后的 `\chapter` 不编号。
- `\mainmatter` 正文部分，页码为阿拉伯数字格式，从 1 开始计数；其后的章节编号正常。
- `\backmatter` 后记部分，页码格式不变，继续正常计数；其后的 `\chapter` 不编号。


**标题页**:
首先需要给定标题和作者等信息：
```tex
\title{<title>}  \author{<author>}   \date{<date>}
```
其中前两个是必须的（不用 `\title` 会报错；不用 `\author` 会警告），`\date` 可选。
`\today` 自动生成当前日期，`\date` 默认使用 `\today`。
在信息给定后，就可以使用:
```tex
\maketitle
```
book 文档类的文档结构示例:
```tex
\documentclass[...]{book}
% 导言区，加载宏包和各项设置
\usepackage{...} % 此处示意对参考文献和索引的设置
\usepackage{makeidx}
\makeindex
\bibliographystyle{...}

\begin{document}
\frontmatter
\maketitle % 标题页
\include{preface} % 前言章节 preface.tex
\tableofcontents

\mainmatter
\include{chapter1} % 第一章 chapter1.tex 
\include{chapter2} % 第二章 chapter2.tex 
...
\appendix
\include{appendixA} % 附录 A appendixA.tex 
...

\backmatter
\include{prologue} % 后记 prologue.tex
\bibliography{...} % 利用 BibTeX 工具生成参考文献 \printindex
% 利用 makeindex 工具生成索引 
\end{document}
```


**交叉引用**:
在能够被交叉引用的地方，如章节、公式、图表、定理等位置使用 `\label`：
```tex
\label{<label-name>}
```
之后可以在别处使用 `\ref` 或 `\pageref`，分别生成交叉引用的编号和页码：
```tex
\ref{<label-name>}   \pageref{<label-name>}
```
比如:
```tex
A reference to this subsection 
\label{sec:this} looks like: 
‘‘see section~\ref{sec:this} on 
page~\pageref{sec:this}.’’

效果:
A reference to this subsection looks like: “see section `3.3` on page `20`.”
```
在使用不记编号的命令形式（`\section*`、`\caption*`、带可选参数的 `\item` 命令等）时不要使用 `\label`，否则生成的引用编号不正确。


**脚注**:
脚注所在的右上角会产生一个脚注的数字, 同时在每页底部产生脚注的内容. 
```tex
“天地玄黄，宇宙洪荒。日月盈昃，辰宿列张。”\footnote{出自《千字文》。}
```
有些情况下（比如在表格环境、各种盒子内）使用 `\footnote` 并不能正确生成脚注, 可以分两步进行，

- 先使用 `\footnotemark `为脚注计数，
- 再在合适的位置用` \footnotetext` 生成脚注。

比如：
```tex
\begin{tabular}{l} 
\hline 
“天地玄黄，宇宙洪荒。日月盈昃，辰宿列张。”\footnotemark \\ 
\hline
\end{tabular} 
\footnotetext{表格里的名句出自《千字文》。}
```
![](/images/tex_1.png)


**列表**:
基本的有序和无序列表环境 enumerate 和 itemize，

- 用法很类似，都用 `\item` 标明每个列表项。
- enumerate 环境会自动对列表项编号。

```tex
\begin{enumerate} 
\item 
\end{enumerate}
```
`\item` 可带一个可选参数，将计数或符号替换成自定义的。列表可以嵌套使用，最多四层。
```tex
\begin{enumerate} 
    \item An item. 
    \begin{enumerate} 
        \item A nested item. 
        \item[*] A starred item. 
        \item Another item. \label{itref} 
    \end{enumerate} 
    \item Go back to upper level. 
    \item Reference(\ref{itref}).
\end{enumerate}

效果:
1. An item.
    (a) A nested item. 
    * A starred item.
    (b) Another item.
2. Go back to upper level. 
3. Reference(1b).
```
itemize 的用法类似, 无序列表的序号为实心圆点和 `-` (子列表中)
description 的用法与以上两者类似，不同的是 `\item` 后的可选参数用来写关键字，以粗体显示，一般是必填的：
```tex
\begin{description} 
\item[Enumerate] Numbered list. 
\item[Itemize] Non-numbered list. 
\end{description}
```
默认的列表间距比较宽，要用到 `enumitem` 宏包定制各种列表间距，还提供了对列表标签、引用等的定制。


**对齐环境**:
center、flushleft 和 flushright 环境分别用于生成居中、左对齐和右对齐的文本环境。
```tex
\begin{center} . . .        \end{center} 
\begin{flushleft} . . .     \end{flushleft} 
\begin{flushright} . . .    \end{flushright}
```
还可以用以下直接改变文字的对齐方式：
```tex
\centering
\raggedright
\raggedleft 
```
上述两种用法的区别: 
`center` 等会在上下文产生一个额外间距，而 `\centering` 等不产生。比如在浮动体环境 table 或 figure 内实现居中对齐，用 `\centering` 即可。


**引用环境**:
两种引用的环境：

- `quote` 用于引用较短的文字，首行不缩进；
- `quotation` 用于引用若干段文字，首行缩进。引用环境较一般文字有额外的左右缩进。

```tex
Francis Bacon says:
\begin{quote} 
Knowledge is power.
\end{quote}
```
verse 用于排版诗歌，与 quotation 恰好相反，verse 是首行悬挂缩进的。(**悬挂缩进，段落第二行和后继行的缩进量要大于首行**)


**摘要环境**:
abstract 默认只在标准文档类中的 article 和 report 可用，一般用于紧跟 `\maketitle` 之后介绍文档的摘要。

- 如果指定了 titlepage 选项，则单独成页；
- 反之，单栏排版时相当于一个居中的小标题加一个 quotation 环境，双栏排版时相当于 `\section*` 定义的一节。


**代码环境**:
它以等宽字体排版代码，回车和空格也分别起到换行和空位的作用; `*` 版本进一步将空格以特殊符号的形式显示出来: 
```tex
\begin{verbatim}
\end{verbatim}

\begin{verbatim*}
\end{verbatim*}
```
要排版简短的代码或关键字，可使用 `\verb`：
```tex
\verb<delim><code><delim>
```
`<delim>` 标明代码的分界位置，前后必须一致，除字母、空格或星号外，可任意选择使得不与代码本身冲突，习惯上使用 `|` 符号。
同 verbatim 环境，`\verb` 后也可以带一个星号，以显示空格：
```tex
\verb|\LaTeX| \\ 
\verb+(a || b)+ \verb*+(a || b)+
```

- verbatim 宏包优化了 verbatim 环境的内部命令，并提供了 `\verbatiminput` 用来直接读入文件生成代码环境。
- fancyvrb 宏包提供了可定制格式的 Verbatim 环境；
- listings 宏包更进一步，可生成关键字高亮的代码环境，支持各种程序设计语言的语法和关键字。

## 表格
排版表格最基本的 tabular 环境用法为：
```tex
\begin{tabular}{<column-spec>}
<item1> & <item2> & . . . \\
\hline 
<item1> & <item2> & . . . \\
\end{tabular}
```
 
- `<column-spec>` 是列格式标记；
- `&` 用来分隔单元格；`\\` 用来换行；
- `\hline` 用来在行与行之间绘制横线。

直接使用 tabular 环境的话，会和周围的文字混排。tabular 环境可带一个可选参数控制垂直对齐（默认是垂直居中）：
```tex
\begin{tabular}{|c|} 
center-\\ aligned \\ 
\end{tabular},

\begin{tabular}[t]{|c|} 
top-\\ aligned \\ 
\end{tabular},
%可选参数 t 代表top
\begin{tabular}[b]{|c|} 
bottom-\\ aligned\\
\end{tabular} tabulars.
%可选参数 b 代表 bottom
```
效果:

![](/images/tabular.png)

但是通常情况下不这么用，tabular 一般会放置在 table 浮动体环境中，并用 `\caption` 加标题。
表格中基本的列格式(`<column-spec>`)如下表：

|列格式 | 说明 |
|---|---|
| `l/c/r` | 单元格内容左对齐/居中/右对齐，不折行|
| `p{<width>}` | 单元格宽度固定为 `<width>`，可自动折行|
| &#124;  |  绘制竖线  |
|  `@{<string>}`    | 自定义内容 `<string>`    |

```tex
\begin{tabular}{lcr|p{6em}} 
\hline 
left    & center    & right 
    & par box with fixed width\\ 
L       & C         & R     & P \\
\hline
\end{tabular}
```
效果: 

![](/images/tabular2.png)

- 表格中每行的单元格数目不能多于列格式里 `l/c/r/p` 的总数（可以少于这个总数），否则出错。
- `@` 格式可在单元格前后插入任意的文本，但同时它也消除了单元格前后额外添加的间距。`@` 格式可以适当使用以充当“竖线”。
- 特别地，`@{}` 可直接用来消除单元格前后的间距 (`@{}` 不算做一个单宇格的元素)：

```tex
% @{} 消除了表格的边沿到元素的距离, 要以一列占位最大的元素为基准. 
\begin{tabular}{ @{} r @{:} l r @{} }
    \hline 1 & 1 & one \\ 
    11 & 3 & eleven \\ 
    \hline
\end{tabular}
```
![](/images/tabular3.png)

将格式参数重复的写法 `*{<n>}{<column-spec>}`，比如以下两种是等效的：
```tex
\begin{tabular}{|c|c|c|c|c|p{4em}|p{4em}|} 
\begin{tabular}{|*{5}{c|}*{2}{p{4em}|}}
```
有时需要为整列修饰格式，比如整列改变为粗体，每个单元格都加上 `\bfseries` 会比较麻烦。array 宏包提供了辅助格式 `>` 和 `<`，用于给列格式前后加上修饰命令：

- `>{\itshape}r`, `>` 表示在左边作用, 

- `<` 表示在右边作用
    - `<{*}` biao

```tex
\begin{tabular}{>{\itshape}r <{*} l} 
\hline 
    italic & normal \\
    column & column \\ 
\hline
\end{tabular}
%第一列的 italic 和 column 就变成了斜体. 
```
辅助格式甚至支持插入 `\centering` 等改变 p 列格式的对齐方式，一般还要加额外的 `\arraybackslash` 以免出错:
```tex
\begin{tabular}
{>{\centering\arraybackslash}p{9em}} 
    \hline 
    Some center-aligned long text. \\ 
    \hline
\end{tabular}
% 效果为表格中只有一个自动换行的句子. 
```


**列宽** :
LATEX 表格有着明显的不足：

- `l/c/r` 格式的列宽是由文字内容的自然宽度决定的，
- 而 `p` 格式给定了列宽却不好控制对齐（可用 `array 宏包`的辅助格式），
- 更何况列与列之间通常还有间距，所以直接生成给定总宽度的表格并不容易。

`tabular*` 用来排版定宽表格，但是不太方便使用，

- 比如要用到 `@` 格式插入额外命令，
- 令单元格之间的间距为 `\fill`，

但即使这样仍有瑕疵：
```tex
\begin{tabular*}{14em}%
{@{\extracolsep{\fill}}|c|c|c|c|} 
    \hline A & B & C & D \\ 
    \hline a & b & c & d \\ 
    \hline
\end{tabular*}
% width is too big
```
![](/images/tabular4.png)

tabularx 宏包提供了方便的解决方案。

- 它引入了一个 `X` 格式，类似 `p` 格式，
- 不过会根据表格宽度自动计算列宽，多个 X 格式平均分配列宽。
- `X` 格式也可以用 array 里的辅助格式修饰对齐方式：

```tex
\begin{tabularx}{14em}%
{|*{4}{>{\centering\arraybackslash}X|}}
    \hline A & B & C & D \\ 
    \hline a & b & c & d \\ 
    \hline 
\end{tabularx}
% 效果正常, 每列宽相等, 而且每个元素居中. 
拆解代码:
| 代表表格的右边缘; 
*{4} 代表 4 个重复元素; 
>{\centering\arraybackslash} 代表修改整列的属性;
X| 代表重复的元素
```


**横线**:
`\cline{<i>-<j>}` 绘制跨越部分单元格的横线：
```tex
\begin{tabular}{|c|c|c|} 
    \hline
    4 & 9 & 2 \\ \cline{2-3} 
    % 跨越 2 和 3 列的横线
    3 & 5 & 7 \\ \cline{1-1}
    % 第一列的横线
    8 & 1 & 6 \\ 
    \hline
\end{tabular}
```
![](/images/tabular5.png)

在科技论文排版中广泛应用的表格形式是三线表。

> 三线表由 booktabs 宏包支持，提供了 `\toprule`、`\midrule` 和 `\bottomrule` 用以排版三线表的三条线，以及和 `\cline` 对应的 `\cmidrule`。

除此之外，最好不要用其它横线以及竖线：
```tex
\begin{tabular}{cccc} 
    \toprule
        & \multicolumn{3}{c}{Numbers} \\ 
        \cmidrule{2-4} 
            & 1 & 2 & 3 \\
    \midrule
        Alphabet & A & B & C \\ 
        Roman & I & II& III \\ 
    \bottomrule 
\end{tabular}
```
![](/images/tabular6.png)


**合并单元格**: 
横向合并单元格较为容易 : 
```tex
\multicolumn{<n>}{<column-spec>}{<item>}
```
其中 `<n>` 为要合并的列数，`<column-spec>` 为合并单元格后的列格式，只允许出现一个 `l/c/r` 或 `p`。
如果合并前的单元格前后带表格线 `|`，合并后的列格式也要带 `|` 以使得表格的竖线一致。
```tex
\begin{tabular}{|c|c|c|} 
    \hline 
    1 & 2 & Center \\ 
    \hline 
    \multicolumn{2}{|c|}{3} & \multicolumn{1}{r|}{Right} \\
    % 注意第二元素不需要加左竖线, 因为第一个元素已经在两边加了竖线
    \hline 
    4 & \multicolumn{2}{c|}{C} \\ 
    \hline
\end{tabular}
```
![](/images/tabular7.png)

上面的例子还体现了，形如 `\multicolumn{1}{<column-spec>}{<item>}` 的命令可以用来修改某一个单元格的列格式。
纵向合并单元格需要用到 multirow 宏包提供的 `\multirow` 命令：
```tex
\multirow{<n>}{<width>}{<item>}
```
`<width>` 为合并后单元格的宽度，可以填 `*` 以使用自然宽度。
```tex
\begin{tabular}{ccc} 
    \hline
        \multirow{2}{*}{Item} & \multicolumn{2}{c}{Value} \\ 
    \cline{2-3} 
        & First & Second \\ 
    \hline
    A & 1 & 2 \\ 
    \hline
\end{tabular}
```
![](/images/tabular8.png)


**嵌套表格** :
在单元格中嵌套一个小表格可以起到“拆分单元格”的效果. 

- 注意要用 `\multicolumn` 命令配合 `@{}` 格式把单元格的额外边距去掉，使得嵌套的表格线能和外层的表格线正确相连;
- 额外边距指的是内嵌表格的三横线与其他的元素会有一段空白的左右间距, `@{}` 就会使得这个边距变为 0 , 同时设置 `|` 能够使表格有右边框. 

```tex
\begin{tabular}{|c|c|c|} 
    \hline
    a & b & c \\
    \hline 
    a &     \multicolumn{1}{@{}c@{}|} 
            {\begin{tabular}{c|c} 
            e & f \\
            \hline 
            e & f \\
            \end{tabular}} 
    & c \\ 
    \hline
    a & b & c \\ 
    \hline 
\end{tabular}
```


**行距控制** : 
改参数 `\arraystretch` 可以得到行距更加宽松的表格:
```tex
\renewcommand\arraystretch{1.8}
\begin{tabular}{|c|}
```
另一种办法是给换行命令增加可选参数, 在这一行下面加额外的间距，适合用于在行间不加横线的表格：
```tex
\\[6pt]
```

- 但是这种换行方式会导致: 表格的首个单元格不能直接使用中括号 `[]`，
    - 否则 `\\` 往往会将下一行的中括号当作自己的可选参数，因而出错。
- 如果要使用中括号，应当放在花括号 `{}` 里面。
- 或者也可以选择将换行命令写成 `\\[0pt]`。

## 图片
LATEX 本身不支持插图功能，需要由 graphicx 宏包辅助支持。

- 使用 latex + dvipdfmx 编译命令时，调用 graphicx 宏包时要指定 dvipdfmx 选项 6；
- 而使用 pdflatex 或 xelatex 命令编译时不需要. 

调用 graphicx 宏包, 可以用 `\includegraphics` 加载图片：
```tex
\includegraphics[options]{filename}
```
graphicx 宏包还提供了 `\graphicspath`，用于声明一个或多个图片文件存放的目录，这些目录里的图片时可不用写路径, 其他则要写出相对或者绝对路径. 
option 支持 key = value 形式赋值: 

| 参数 | 含义 |
|-----|-----|
| `width=width` | 将图片缩放到宽度为 `<width>`|
| `height=height` | 将图片缩放到高度为 `<height>`|
| `scale=scale` |将图片相对于原尺寸缩放 `<scale>` 倍|
| `angle=angle` | 令图片逆时针旋转 `<angle>` 度|

## 盒子
```tex
\mbox{...}
\makebox[<width>][<align>]{...}
```

- `\mbox` 生成一个基本的水平盒子，内容只有一行（除非嵌套下文介绍的垂直盒子，或者其它内容），不允许分段。
- `\mbox` 的内容与正常的文本无二，不过断行时文字不会从盒子里断开。
- `\makebox` 更进一步:
    - 可以加上可选参数用于控制盒子的宽度 `<width>`，
    - 以及内容的对齐方式 <align>，
    - 可选居中 c（默认值）、左对齐 l、右对齐 r 和分散对齐 s.

```tex
|\mbox{Test some words.}|\\
|\makebox[10em][l]{Test some words.}|\\ %生成一个固定长度的左对齐的文本.
```
普通的盒子与普通的文本没有太大的差别, 只是规定了文本的位置和间距. 

**带框的水平盒子** :
```tex
\fbox{...} 
\framebox[<width>][<align>]{...}
```
可以通过 `\setlength` 调节边框的宽度 `\fboxrule` 和内边距 `\fboxsep`：
```tex
\setlength{\fboxrule}{1.6pt}
\setlength{\fboxsep}{1em} 
\framebox[10em][r]{Test box}
```

**垂直盒子** :
如果需要排版一个文字可以换行的盒子，两种方式：
```tex
% method 1
\parbox[<align>][<height>][<inner-align>]{<width>}{...} 
% method 2
\begin{minipage}[<align>][<height>][<inner-align>]{<width>}
........
\end{minipage}
```

- `<align>` 为盒子和周围文字的对齐情况（类似 tabular 环境）；
- `<height>` 和 `<inner-align>` 设置盒子的高度和内容的对齐方式，类似水平盒子 `\makebox` 的设置，不过 `<inner-align>` 接受的参数是顶部 t、底部 b、居中 c 和分散对齐 s。
- 如果在` minipage` 里使用` \footnote` 命令，生成的脚注会出现在盒子底部，编号是独立的， 并且使用小写字母编号。这也是 minipage 环境之被称为“迷你页”（Mini-page）的原因。
- 而在 `\parbox` 里无法正常使用` \footnote` 命令，只能在盒子里使用 `\footnotemark`，在盒子外使用 `\footnotetext`。

```tex
\fbox{\begin{minipage}{15em}% 这是一个垂直盒子的测试。
    \footnote{脚注来自 minipage。} 
\end{minipage}}
```

![](/images/tabular9.png)


**标尺盒子** :
`\rule` 用来画一个实心的矩形盒子，也可适当调整以用来画线（标尺）:
```tex
Black \rule{12pt}{4pt} box. 

Upper \rule[4pt]{6pt}{8pt} and 
lower \rule[-4pt]{6pt}{8pt} box. 

A \rule[-.4pt]{3em}{.4pt} line.
```
![](/images/tabular10.png)

## 浮动体
图片和表格太大, 导致分页困难. 因此需要使用浮动体。预定义了两类浮动体环境 figure 和 table. 
以 table 环境的用法举例，figure 同理：
```tex
\begin{table}[<placement>]
\end{table}
```
`<placement>` 提供了一些符号用来表示浮动体允许排版的位置，如 hbp 允许浮动体排版在当前位置、底部或者单独成页。table 和 figure 浮动体的默认设置为 tbp。

|  字母 |   含义 |
|----|---|
|h  | 当前位置（代码所处的上下文）|
|t  | 顶部 |
|b   |底部 |
|p|   单独成页 |
|!|   在决定位置时忽视限制|


**浮动体的标题** :
图表等浮动体提供了 `\caption` 加标题，并且自动给浮动体编号：

- `\caption` 的用法类似 `\section`，
- `\caption*` 生成不带编号的标题，
- 也可以使用带可选参数的形式 `\caption[...]{...}`，使得在目录里使用短标题。 
- `\caption` 之后还可以紧跟 `\label` 标记交叉引用。
- `\caption` 生成的标题形如 “Figure 1: . . . ”（figure 环境）或 “Table 1: . . . ”（table 环境）。可通过修改 `\figurename` 和 `\tablename` 的内容来修改标题的前缀


**并排和子图表**
常有在一个浮动体里面放置多张图的用法。最简单就是直接并排放置，也可以通过分段或者换行命令 `\\` 排版多行多列的图片。
```tex
\begin{figure}[htbp] 
    \centering
    \includegraphics[width=...]{...}
    \qquad
    \includegraphics[width=...]{...} 
    \\[..pt] 
    \includegraphics[width=...]{...} 
    \caption{...}
\end{figure}
```
由于标题是`横跨一行`的，用 `\caption` 为每个图片 (浮动体内有两张以上的图片)单独生成标题就需要借助前文提到的 `\parbox` 或者 `minipage`，将标题限制在盒子内。
```tex
\begin{figure}[htbp] 
    \centering
    \begin{minipage}{...} 
        \centering
        \includegraphics[width=...]{...} 
        \caption{...}
    \end{minipage} 
    \qquad
.......
```
当需要更进一步，给每个图片定义小标题时，就要用到 subfig 宏包的功能了。比如：
```tex
\begin{figure}[htbp]
    \centering
    \subfloat[...]{\label{sub-fig-1}% 为子图加交叉引用 \begin{minipage}{...} \centering
    \includegraphics[width=...]{...} \end{minipage}
```
小标题与单独标题不同之处在于小标题还有一个大标题 (两幅图 3 个标题), 而单独标题是互相平行的 (两幅图 2 个标题). 


# 排版数学公式
AMS 宏集宏集合是美国数学学会 (American Mathematical Society) 提供的对 LATEX 原生的数学公式排版的扩展，其核心是 amsmath 宏包，对多行公式的排版提供了有力的支持。此外，amsfonts 宏包以及基于它的 amssymb 宏包提供了丰富的数学符号；amsthm 宏包扩展了 LATEX 定理证明格式。

## 公式排版基础
行内公式由一对 `$` 符号包裹.
单独成行的行间公式在 LATEX 里由 equation 环境包裹。equation 环境为公式自动生成一个编号，这个编号可以用` \label` 和 `\ref` 生成交叉引用，amsmath 的 `\eqref `命令甚至为引用自动加上圆括号；还可以用` \tag` 命令手动修改公式的编号，或者用 `\notag `命令取消为公式编号（与之基本等效的命令是 `\nonumber`）。
```tex
\begin{equation}
    a^2 + b^2 = c^2          % 默认自动生成编号, 编号为 (4-1)的形式
\end{equation}

\begin{equation} 
    E = mc^2 \label{clever} % 生成自动编号, 同时给编号起名字叫做 clever, 方便以后引用
\end{equation}
This is a reference to \eqref{clever}. %引用clever的编号

\begin{equation} 
    1 + 1 = 3 \tag{dumb}    % 生成 (dumb) 编号, dumb就是一个单词
\end{equation}
```

- `\[` 用于生成不带编号的行间公式，
- displaymath 环境也可以生成不带编号的。
- 有的人更喜欢 `equation*` 环境，体现了带星号和不带星号的环境之间的区别。

```tex
方法一:
\begin{equation*}
    a^2 + b^2 = c^2
\end{equation*}
方法二:
\[ a^2 + b^2 = c^2 \]
方法三:
\begin{displaymath} 
    a^2 + b^2 = c^2
\end{displaymath}
```
行间公式的对齐、编号位置等性质由文档类选项控制，文档类的 `fleqn` 选项令行间公式左对齐；`leqno` 选项令编号放在公式左边。


**数学模式和文本** :
使用 `$` 开启行内公式输入，或是使用 `\[` 命令、equation 环境时，就进入了数学模式，有以下特点：

1. 数学模式中输入的空格全部被忽略。数学符号的间隙默认完全由符号的性质（关系符号、运算符等）决定。需要空格，使用 `\quad` 和 `\qquad`。
2. 不允许有空行（分段），公式也无法自动换行或者用 `\\` 换行。排版多行公式需要用到后续介绍的各种环境。
3. 所有的字母被当作数学公式中的变量处理，字母间距与文本模式不一致，也无法生成单词之间的空格。
4. 如果在数学公式中输入正体，可用 `\mathrm{X}`。或者 amsmath 提供的 `\text{文本}`。

```tex
$x^{2} \geq 0 \qquad 
\text{for \textbf{all} }
x \in \mathbb{R}$
```
效果:
$x^{2} \geq 0 \qquad \text{for \textbf{all} } x\in\mathbb{R}$

## 数学符号

- 省略号有 $\dots$ (`\dots`) 和 $\cdots$ (`\cdots`)。它们的各自用途：
    - $a_1, a_2, \dots, a_n$ 
    - $a_1 + a_2 + \cdots + a_n$
- `\ldots` 和 `\dots` 是完全等效的，它们既能用在公式中，也用来在文本里作为省略号。
- 在矩阵中可能会用到竖排的 (`\vdots`) 和斜排的 (`\ddots`)。


`amsmath` 提供了方便的命令` \dfrac` 和 `\tfrac`，前者在行内使用正常大小的分数, 后者在行内进行极度压缩。
一般的根式使用 `\sqrt{2}`；表示 n 次方根时写成 `\sqrt[n]{2}`. 
特殊的分式形式，如二项式结构，由 amsmath 宏包的 `\binom` 生成：
```tex
\[ \binom{n}{k} =\binom{n-1}{k} + \binom{n-1}{k-1} \]
```
$\binom{n}{k} =\binom{n-1}{k} + \binom{n-1}{k-1}$


**关系符** :

- 不等于 : `\ne` ; 大于等于 : `\ge` ; 小于等于 : `\le` ; 
- 约等于 : `\approx` ; 等价 : $\equiv$ (`\equiv`) ; 
- 正比 : $\propto$ ,`\propto` ; 相似 : `\sim`
- 倾斜的小于等于 : $\leqslant$ , `\leqslant` ; $\le$ (使用 amssymb 宏包).


**算符** :

- 乘号 : `\times` ; 除号 : `\div` ; 
- 点乘 : `\cdot` ; 
- 加减号 : `\pm` 或者 `\mp`(加减号的上下位置相反)
- $\nabla$: `\nabla` ; 
- $\partial$ : `\partial`


**巨算符** :
$ \int $(`\int`) , $\oint$ (`\oint`), $\prod$ (`\prod`) 和 $\sum$ (`\sum`) 称为巨算符. 
巨算符在行内公式和行间公式的大小和形状有区别。 
巨算符的上下标用作其上下限。

- 行间公式中，积分号默认将上下限放在右上角和右下角，求和号默认在上下方；
- 行内公式一律默认在右上角和右下角。
- 可以在巨算符后使用 `\limits` 手动令上下限显示在上下方，`\nolimits` 则相反。
- $\sum\limits\_{i=1}^n$ : `\sum\limits_{i=1}^n `

amsmath 宏包还提供了 `\substack`，能够在下限位置书写多行表达式；
subarray 环境更进一步，令多行表达式可选择居中 (c) 或左对齐 (l)：
```tex
\sum_{\substack{0\le i\le n \\ 
        j\in \mathbb{R}}}
P(i,j) = Q(n)

\sum_{\begin{subarray}{l} 
        0\le i\le n \\ 
        j\in \mathbb{R} 
    \end{subarray}}
```


**数学重音和上下括号**:
数学符号可以像文字一样加重音，

- 比如对时间求导的符号 $\dot{r}$ (`\dot{r}`)，$\ddot{r}$ (`\ddot{r}`), $\vec{r}$ (`\vec{r}`), 
- 表示欧式空间单位向量的 $\hat{\mathbf{e}}$ (`\hat{\mathbf{e}}`)；

也能为多个字符加重音，

- 包括直接画线的 `\overline `和 `\underline` 命令（可叠加使用）、
- 宽重音符号 `\widehat`、
- 表示向量的箭头 $\overrightarrow{AB}$ `\overrightarrow` 等 (箭头的长度与 $\vec{AB}$, `\vec{AB}` 稍有不同)

`\overbrace` 和 `\underbrace` 命令用来生成上/下括号，各自可带一个上/下标公式。
```tex
\underbrace {
    \overbrace{a+b+c}^6 
    \cdot
    \overbrace{d+e+f}^7 } 
_\text{meaning of life} = 42
```
效果:
$$\underbrace{\overbrace{a+b+c}^6 \cdot \overbrace{d+e+f}^7} _\text{meaning of life} = 42$$

**箭头** :
除了作为上下标之外，箭头还用于表示过程。amsmath 的 `\xleftarrow` 和 `\xrightarrow` 命令可以为箭头增加上下标：
```tex
% [] 里面的参数做下标, {} 里面的参数做上标
c \xrightarrow[x<y]{a*b*c}d
```
效果:
$$ c \xrightarrow[ x < y ]{a*b*c}d $$


**括号和定界符** :
`\left` 和 `\right` 可令括号（定界符）的大小可变：
```tex
1 + \left(\frac{1}{1-x^{2}} \right)^3 \qquad 
\left.\frac{\partial f}{\partial t} \right|_{t=0}
```
还可自己调节定界符的大小：

- 可以用 `\big`、`\bigg` 等生成固定大小的定界符。
- 更常用的形式是类似 `\left` 的 `\bigl`、`\biggl` 等，
- 以及类似 `\right` 的 `\bigr`、`\biggr` 等（`\bigl` 和 `\bigr` 不必成对出现）

```tex
\Bigl((x+1)(x-1)\Bigr)^{2}
```
效果为 : $\Bigl((x+1)(x-1)\Bigr)^{2}$

> `\big` 和 `\bigg` 的另外一个好处是：用 `\left` 和 `\right` 分界符包裹的公式块是不允许断行的
下文提到的 array 或者 aligned 等环境视为一个公式块，
所以也不允许在多行公式里跨行使用，而 `\big` 和 `\bigg` 等不受限制。

## 多行公式

**长公式折行** :
amsmath 宏包的 `multline` 环境提供了书写折行长公式的方便环境。它允许用 `\\` 折行，将公式编号放在最后一行。多行公式的首行左对齐，末行右对齐，其余行居中。
```tex
\begin{multline} 
    a + b + c + d + e + f + g + h + i \\ 
        = j + k + l + m + n \\
    = o + p + q + r + s
\end{multline}
```
效果: 
$$\begin{multline}
    a + b + c + d + e + f + g + h + i \\\\
        = j + k + l + m + n \\\\
    = o + p + q + r + s \end{multline}
$$
与表格不同的是，公式的最后一行不写 `\\`，如果写了，反倒会产生一个多余的空行。


**多行公式**:
需要罗列一系列公式，并令其按照等号对齐。
使用 align 环境，它将公式用 `&` 隔为两部分并对齐。
分隔符通常放在等号左边：
```tex
\begin{align} 
    a = & b + c 
        & + f + g \\  % 分隔符放在等号的右边, 使得换行的元素也能够与等号右边对齐
     =  &  d + e
\end{align}
```
可以用 `\notag` 去掉某行的编号。  
align 还能够对齐多组公式，除等号前的 `&` 之外，公式之间也用 `&` 分隔：
```tex
\begin{align} 
    a &=1 & b &=2 & c &=3 \\ 
    d &=-1 & e &=-2 & f &=-5 
\end{align}
```
效果: 
$$\begin{align} 
    a &=1 & b &=2 & c &=3 \\\\
    d &=-1 & e &=-2 & f &=-5 \end{align}
$$
如果不需要按等号对齐，只需罗列数个公式，gather 将是很好用的环境：
```tex
\begin{gather} 
    a = b + c  \\
    h + i = j + k \notag \\ 
    l + m = n 
\end{gather}
```
align 和 gather 有不带编号的版本 `align*` 和 `gather*`。


**公用编号的多行公式** :

> 多行公式公用一个编号，编号位于公式垂直位置居中，amsmath 宏包提供了诸如 `aligned`、`gathered` 等环境，与 equation 环境套用。

用法是相同的。仅以 aligned 举例：
```tex
%　效果是中间的等号对齐
\begin{equation} 
    \begin{aligned} 
        a &= b + c \\
        d &= e + f + g \\
        h + i &= j + k \\ 
        l + m &= n 
    \end{aligned} 
\end{equation}
```
`split` 环境也是类似，也用于和 `equation `环境套用，区别是 `split` 只能将每行的一个公式分两栏，`aligned `允许每行多个公式多栏。

## 数组和矩阵
数组排版使用 array 环境, 用法与 tabular 环境极为类似，需要定义列格式, 并用 `\\` 换行. 数组可作为一个公式块，在外套用 `\left`、`\right` 等定界符：
```tex
 \mathbf{X} 
= \left( 
    \begin{array}{cccc} 
        x_{11} & x_{12} & \ldots & x_{1n}\\ x_{21} & x_{22} & \ldots & x_{2n}\\ \vdots & \vdots & \ddots & \vdots\\ x_{n1} & x_{n2} & \ldots & x_{nn}\\ 
    \end{array} 
\right)
```
效果为：
$$\mathbf{X} = \left( 
    \begin{array}{cccc} 
        x\_{11} & x\_{12} & \ldots & x\_{1n}\\\\ 
        x\_{21} & x\_{22} & \ldots & x\_{2n}\\\\ 
        \vdots & \vdots & \ddots & \vdots\\\\ 
        x\_{n1} & x\_{n2} & \ldots & x\_{nn}\\\\
    \end{array} \right)
$$
上一节末尾介绍的 aligned 等环境也可以用定界符包裹。
还可以利用空的定界符排版出这样的效果：
```tex
|x| = \left\{ 
    \begin{array}{rl} 
        -x & \text{if } x < 0,\\ 
        0 & \text{if } x = 0,\\ 
        x & \text{if } x > 0. 
    \end{array} 
    \right.
```
效果为 :
$$\|x\| = \left\{ 
    \begin{array}{rl} 
    -x & \text{if } x < 0,\\\\ 
    0 & \text{if } x = 0,\\\\ 
    x & \text{if } x > 0 
    \end{array} 
    \right.
$$
amsmath 宏包还直接提供了多种排版矩阵的环境，包括不带定界符的 matrix，以及带各种定界符的矩阵 `pmatrix (`, `bmatrix [`, `Bmatrix {`, `vmatrix |`, `Vmatrix ||`.
在矩阵中的元素里排版分式时，一来要用到 `\dfrac` 等命令，二来行与行之间有可能紧贴着，
这时要用到 3.6.6 小节的方法来调节间距. 

## 公式中的间距
在公式中还可能用到的间距包括 `\,`、`\:`、`\;` 以及负间距 `\!`，

- 其中 `\quad` 、`\qquad` 和 `\,` 在文本和数学环境中可用，
- 前三个命令只用于数学环境。

间隔大小 :

![](/images/math1.png)

- 上表常见的用途是修正积分的被积函数 f(x) 和微元 dx 之间的距离。注意微元里的 d 用的是**正体**
- 另一个用途是生成多重积分号。
    - 如果直接连写两个 `\int`，之间的间距将会过宽，此时可以使用负间距 `\!` 修正。
    - 不过 amsmath 提供了多重积分号，如二重积分 `\iint`、三重积分 `\iiint`。

```tex
\int_a^b f(x)\,\mathrm{d}x
```
效果为: $\int_a^b f(x) \, \mathrm{d}x$

## 数学符号的字体控制
LATEX 允许一部分数学符号切换字体，主要是拉丁字母、数字等等。

![](/images/math2.png)


**数学符号的尺寸** :
不懂, 暂留


**加粗的数学符号** :
想得到粗斜体的符号，没有现成的命令：

- `\mathbf` 只能改变拉丁字母，希腊字母就没有用.
- `\boldmath` 令用户可以将整套数学字体切换为粗体版本。但这个命令只能在公式外使用；
- `\boldsymbol` 可以用于公式内。
- 定界符、巨算符等一些符号本身没有粗体版本，`\boldsymbol` 也得不到粗体

`{\boldmath $\mu, M$}` 效果为：$\mu, M$ 这个行内公式得到了加粗。
`\boldsymbol{\mu}` 效果为：$\boldsymbol{\mu}$
bm 宏包可以用 `\bm` 生成“伪粗体”，一定程度上解决了不带粗体版本的符号的问题。

## 定理环境
`\newtheorem` 提供定理环境的定义：
```tex
\newtheorem{<type>}{<title>}[<section-name>]
\newtheorem{<type>}[<counter>]{<title>}
```

- `type` 为定理类型的名称，作为一个环境来使用。
- 定理环境都需要定义，LATEX 里没有现成的 theorem 环境，直接使用很可能会出错。
- `title` 是定理类型的标签（“定理”，“公理”等），排版在序号之前。

定理的序号由两个可选参数之一决定，它们不能同时使用：

- `section name` 为章节名称，这使定理序号成为章节的下一级序号；
- `counter` 为用 `\newcounter` 自定义的计数器名称（详见 8.3 节），定理序号由这个计数器管理。

```tex
\newtheorem{mythm}{My Theorem}[section]  % 先定义一个 mythm 环境
\begin{mythm}\label{thm:light} 
    The light speed in vaccum is $299,792,458\,\mathrm{m/s}$. 
\end{mythm}
```


**amsthm 宏包** :
LATEX 只给了原始的证明环境格式（粗体标签、斜体正文、定理名用小括号包裹）。如果需要修改格式，则要依赖其它的宏包，如 amsthm、ntheorem 等等。
amsthm 提供了 `\theoremstyle` 支持定理格式的切换，在用 `\newtheorem` 命令定义定理环境之前使用。amsthm 预定义了三种格式用于 `\theoremstyle`：

- plain 和 LATEX 原始的格式一致；
- definition 使用粗体标签、正体内容；
- remark 使用斜体标签、正体内容。
- 另外 amsthm 还支持用带星号的 `\newtheorem*` 定义不带序号的定理环境：

```tex
\theoremstyle{definition}   \newtheorem{law}{Law} 
\theoremstyle{plain}        \newtheorem{jury}[law]{Jury}
\theoremstyle{remark}       \newtheorem*{mar}{Margaret}
```
```tex
% 以下例子使用上述所定义的环境
\begin{law}\label{law:box} 
    Don’t hide in the witness box. 
\end{law}

\begin{jury}[The Twelve] 
    It could be you! So beware and see law~\ref{law:box}.
\end{jury} 

\begin{mar}
    No, No, No
\end{mar} 
```
效果:

![](/images/math3.png)

amsthm 还支持使用 `\newtheoremstyle` 自定义定理格式，更为方便是 ntheorem 宏包。


**证明环境和证毕符号** :
amsthm 还提供了一个 proof 环境用于排版定理的证明过程。proof 环境末尾自动加上一个证毕符号 (一个小正方形, 而且证明前会加上一个斜体的 Proof) ：

```tex
\begin{proof} 
    For simplicity, we use
    ....
    That’s it. 
\end{proof}
```
如果行末是一个不带编号的公式，证毕符号会另起一行，可用 `\qedhere` 将证毕符号放在公式末尾：
```tex
\begin{proof} 
    For simplicity, we use 
    \[
        E=mc^2 \qedhere 
    \]
\end{proof}
```
\qedhere 对于 align* 等命令也有效, 证毕符号放在多行公式的最后：
```tex
\begin{align*} 
    E &= \gamma m_0 c^2 \\ 
    p &= \gamma m_0v \qedhere 
\end{align*}
```
如果有使用实心符号作为证毕符号的需求，需要自行用 \renewcommand 命令修改, 可以利用标尺盒子来生成一个适当大小的“实心矩形”：
```tex
\renewcommand{\qedsymbol}% 
            {\rule{1ex}{1.5ex}}

\begin{proof}  
    For simplicity, we use
    \[
        E=mc^2          \qedhere 
    \]
\end{proof}
```

## 符号表
自行查阅书籍


# 排版样式设定
## 字体和字号
LATEX 根据文档的逻辑结构（章节、脚注等）来选择默认的字体样式以及字号。需要更改字体样式或字号的话，可以使用表 5.1 和表 5.2 中列出的命令。
```tex
{\small The small and \textbf{bold} Romans ruled}       %  \small 使用小字体
{\Large all of great big                                % \Large使用大字体
    {\itshape Italy}.}                               % \itshape 使用斜体字, 同时在 \Large 作用域内部 
```


**字体样式** :
LATEX 提供了两组修改字体的命令，见表 5.1。

- 其中诸如 `\bfseries` 形式的命令将会影响之后所有的字符，
    - 如果想要让它在局部生效，需要用花括号分组，即 `{\bfseries sometext}`; 
    - 对应的 `\textbf` 形式带一个参数，只改变参数内部的字体，更为常用。
- 在公式中，直接使用 `\textbf` 等不会起效，甚至报错。


**字号** :
字号命令实际大小依赖于所使用的文档类及其选项。表 5.3 列出了这些命令在标准文档类中的绝对大小，单位为 pt。
使用字号命令的时候，通常也需要用花括号进行分组，如同 `\rmfamily` 那样。
```tex
He likes 
    {\LARGE large 
        and {\small small}  % small 相对于 He likes 要更小
    letters}    
```
LATEX 还提供了一个基础的命令 `\fontsize` 用于设定任意大小的字号：
```tex
\fontsize{size}{base line-skip}
```
`\fontsize` 用到两个参数, 

- size 为字号, 
- 第二个为基础行距. 
- 字号都有对应的基础行距, 大小为字号的 1.2 倍. 
- 如果不是在导言区，`\fontsize` 的设定需要 `\selectfont` 命令才能立即生效。


表 5.1---字体命令:

| 法1 | 法2 | 英文名称|中文名称|
|---|---|---|---|
| `\rmfamily` | `\textrm{...}` | roman | 衬线字体（罗马体）|
| `\sffamily` | `\textsf{...}` | sans serif | 无衬线字体 |
| `\ttfamily` | `\texttt{...}` | typewriter | 等宽字体 |
| `\mdseries` | `\textmd{...}` | medium | 正常粗细（中等）|
| `\bfseries` | `\textbf{...}` | bold face | 粗体 |
| `\upshape` | `\textup{...}` | upright | 直立体 |
| `\itshape` | `\textit{...}` | italic | 意大利斜体 |
| `\slshape` | `\textsl{...}` | slanted | 倾斜体 |
| `\scshape` | `\textsc{...}` | Small Caps |小字母大写|
| `\em` | `\emph{...}` | emphasized | 强调，默认斜体 |
| `\normalfont` | `\textnormal{...}` | normal font | 默认字体 |

![](/images/table_of_fonts.png)

字号排序:
```tex
\tiny  \scriptsize  \normalsize  \small  \footnotesize  \large  \Large  \LARGE  \huge  \Huge
```


**选用字体宏包** :
默认字体为由高德纳设计制作的 Computer Modern 字体。表 5.4 列出了较为常用的字体宏包，其中相当多的宏包还配置了数学字体，或者文本、数学字体兼而有之。
参照 P72. 上网查找字体风格. 


**字体编码** :
字体编码对于 LATEX 用户来讲是一个比较难懂的概念。

- 它指定了一个字体里面包含了哪些符号、符号如何编码等等细节。
- 需要明确一点：字体编码并不与在 2.1.1 等小节叙述的 ASCII 编码等一一对应。

常见的正文字体编码有 OT1 和 T1 等。

- LATEX 默认使用对原始 TEX 兼容的 OT1 编码，
    - 使用起来有诸多限制：高德纳在设计 Computer Modern 字体时认为一些符号，如大于号、小于号等，原则上都应该在公式里出现，
    - 所以在正文字体（`\rmfamily` 或 `\sffamily`）里，这些符号所在的位置被其它符号所占据。
    - 事实上用户输入 `<` 和 `>` 得到的是 `!` 和 `?` 两个倒立的标点符号，
    - 正常的大于号和小于号可用命令 `\textgreater` 和 `\textless` 输入；
    - `\ttfamily` 字体下基本上是正常的。
- 扩展的 T1 编码则对 ASCII 字符的兼容好得多，
    - 不会出现上述的大于号、小于号的问题。
    - T1 编码配合一些字体宏包如 txfonts、lmodern 等，还能够令用户使用 `\textasciitilde` 命令输入位置居中的连字符 a~b，相比数学符号 `$\sim$` 来得合理一些。

切换字体编码要用到 fontenc 宏包：
```tex
\usepackage[T1]{fontenc}
```
fontenc 宏包是用来配合传统的 LATEX 字体的，如表 5.4 中的大部分宏包。如果使用下文的
fontspec 宏包调用 ttf 或 otf 格式字体，就不要再使用 fontenc 宏包。

![](/images/font_package.png)

**使用 fontspec 宏包更改字体 (xelatex)** :
xelatex 编译命令能够支持直接调用系统安装的 `.ttf` 或 `.otf` 格式字体。相比于上一小节有了更多修改字体的余地。
xelatex 下支持用户调用字体的宏包是 fontspec，提供了几个设置全局字体的命令，设置 `\rmfamily` 等对应命令的默认字体：
```tex
\setmainfont[font features]{font name} 
\setsansfont[font features]{font name} 
\setmonofont[font features]{font name}
```

- 其中 font name 使用字体的文件名（带扩展名）或者字体的英文名称。
- font features 用来手动配置对应的粗体或斜体，比如为 Windows 下的无衬线字体 Arial 配置粗体和斜体（通常情况下自动检测并设置对应的粗体和斜体，无需手动指定）：

```tex
\setsansfont [BoldFont={Arial Bold}, ItalicFont={Arial Italic}] {Arial}
```
font features 还能配置字体本身的各种特性，这里不再赘述。
需要注意的是：fontspec 宏包会覆盖数学字体设置：

- 需要调用表 5.4 中列出的一些数学字体宏包时，应当在调用 fontspec 宏包时指定 `no-math` 选项。
- fontspec 宏包可能被其它宏包或文档类（如 xeCJK、ctex 文档类）自动调用时，则在文档开头的 `\documentclass` 里指定 `no-math`选项。



**使用 xeCJK 宏包更改中文字体** :
前文已经介绍过的 xeCJK 宏包使用了和 fontspec 宏包非常类似的语法设置中文字体：
```tex
\setCJKmainfont[font features]{font name}
\setCJKsansfont[font features]{font name}
\setCJKmonofont[font features]{<font name}
```


## 段落格式和间距
长度的数值 length 由数字和单位组成。常用的单位如下：

|符号| 意义|
|----|----|
|pt |点阵宽度，1/72.27in|
|bp |点阵宽度，1/72in |
|in |英寸 |
|cm |厘米 |
|mm |毫米 |
|em| 当前字号下大写字母 M 的宽度，常用于水平距离的设定| 
|ex |当前字号下小写字母 x 的高度，常用于垂直距离的设定|

在一些情况下还会用到可伸缩的“弹性长度”，如 `12pt plus 2pt minus 3pt` 表示基础长度为 12pt，可以伸展到 14pt ，也可以收缩到 9pt。
自定义长度变量：
```tex
\newlength{\length command}
```
长度变量可以用 `\setlength` 赋值，或用 `\addtolength` 增加长度：
```tex
\setlength{\length command}{length}
\addtolength{\length command}<length}
```

**行距** :
前文提到过 `\fontsize` 命令可以为字号设定对应的行距，但很少那么用。更常用的办法是在导言区使用 `\linespread` 命令，
```tex
\linespread{factor}
```

- 这里的 factor 是在基础行距上而不是字号上乘以一个因子。
- 大部分时候，默认的基础行距是 1.2 倍字号大小（参考 `\fontsize` 命令），
- 因此设置 1.5 倍行距的命令 `\linespread{1.5}` 意味着最终行距为 1.8 倍的字号大小。

如果想要局部地改变某个段落的行距，用 `\selectfont` 使 `\linespread` 的改动立即生效：
```tex
{\linespread{2.0} \selectfont 
    The baseline skip is set to be twice the normal baseline skip. Pay attention to the \verb|\par| command at the end. \par
}
% 这一段的行距会增加, 但是下一段的行距会变会正常值
```

> 字号的改变是即时生效的，而行距的改变直到文字分段时才生效。

**段落格式** :
以下长度分别为段落的左缩进、右缩进和首行缩进：
```tex
\setlength{\leftskip}{20pt}
\setlength{\rightskip}{20pt} 
\setlength{\parindent}{2em}
% \setlength 命令同样是要在分组{}里面运行
```
它们和设置行距的命令一样，在分段时生效。

- 如果在某一段不想使用缩进，可使用某一段开头使用 `\noindent`。
- 相反地，`\indent` 命令强制开启一段首行缩进的段落。
- 多个 `\indent` 命令可以累加缩进量。

LATEX 还默认在 `\chapter`、`\section` 等章节标题**命令之后的第一段**不缩进。

**水平间距** :
LATEX 默认为将单词之间的“空格”转化为水平间距。如果需要手动插入额外的水平间距，可使用 `\hspace{length}`：
```tex
This\hspace{1.5cm}is a space of 1.5 cm.
```

- `\stretch{n}` 生成一个特殊弹性长度，参数 n 为权重。
    - 它的基础长度为 0pt，但可以无限延伸，直到占满可用的空间。
    - 如果同一行内出现多个 `\stretch{n}`，这一行的所有可用空间将按每个 `\stretch` 给定的权重 n 进行分配.
- 在正文中用` \hspace` 命令生成水平间距时，往往使用 em 作为单位，生成的间距随字号大小而变。
- 数学公式中的 `\quad` 和 `\qquad` 也可以用于文本中，分别相当于 `\hspace{1em} `和 `\hspace{2em}`


**垂直间距** :
在页面中，段落、章节标题、行间公式、列表、浮动体等元素之间的间距是 LATEX 预设的。比如 `\parskip` ，默认设置为 `0pt plus 1pt`。
如果要人为地增加段落之间的垂直间距，可以在两个段落之间的位置使用：
```tex
\vspace{length}
```
`\vspace` 的间距在一页的顶端或底端可能被“吞掉”，类似 `\hspace` 在一行的开头和末尾那样。

- 而 `\vspace*` 产生不会因断页而消失的垂直间距。
- 在段落内的两行之间增加垂直间距，一般通过给断行命令 `\\ `加可选参数，如 `\\[6pt]` 或 `\\*[6pt]`。

`\vspace` 也可以在段落内使用：
```tex
add \vspace{12pt} some spaces 
between lines in a paragraph.
% 上下两行的垂直间距增大, 但是在同一段中
```
`\bigskip, \medskip, \smallskip` 可以增加预定义长度的垂直间距。
```tex
\parbox[t]{3em}{TeX \par TeX} 
\parbox[t]{3em}{TeX \par \smallskip TeX} 
\parbox[t]{3em}{TeX \par \medskip TeX} 
\parbox[t]{3em}{TeX \par \bigskip TeX}
```

## 页面和分栏
LATEX 允许通过为文档类指定选项来控制纸张的大小（见表 1.2），包括 a4paper、letterpaper 等等，并配合字号设置了适合的页边距。
但是，如果要直接设置页边距等参数，着实是一件麻烦事。根据图 5.1 将各个方向的页边距计算公式给出（以奇数页为例）：
```tex
left-margin = 1in + \hoffset + \oddsidemargin
right-margin = \paperwidth − left-margin − \textwidth 
top-margin = 1in + \voffset + \topmargin + \headheight + \headsep 
bottom-margin = \paperheight − top-margin − \textheight
```
`geometry` 宏包提供了设置页面参数的简便方法，能够完成背后繁杂的计算。


**利用 geometry 宏包设置页面参数** :
geometry 宏包的调用方式类似于 graphicx，

- 在 latex + dvipdfmx 命令下需要指定选项 dvipdfm （注意这里不是 dvipdfmx）；
- pdflatex 和 xelatex 编译命令下不需要。

```tex
\usepackage{geometry}
\geometry{geometry-settings}

% 也可以将参数指定为宏包的选项：
\usepackage[geometry-settings]{geometry}
```
其中 `geometry-settings` 多以 `key=value` 的形式组织。
比如，符合 Microsoft Word 习惯的页面设定是 A4 纸张，上下边距 1 英寸，左右边距 1.25 英寸，于是可以通过如下两种等效的方式之一设定页边距：
```tex
\usepackage[left=1.25in,right=1.25in, top=1in,bottom=1in]{geometry} 
% or like this:
\usepackage[hmargin=1.25in,vmargin=1in]{geometry}
```
又如要设定周围的边距一致为 1.25 英寸，可以用更简单的语法：
```tex
\usepackage[inner=1in, outer=1.25in]{geometry}
```
geometry 宏包本身也能够修改纸张大小、页眉页脚高度、边注宽度等等参数。


**页面内容的垂直对齐** :
LATEX 默认将页面内容在垂直方向分散对齐。
对于有大量图表的文档，许多时候想要做到排版匀称的页面很困难，垂直分散对齐会造成某些页面的垂直间距过宽，还可能报大量的 `Underfull \vbox` 消息。
LATEX 还提供了另一种策略：将页面内容向顶部对齐，给底部留出高度不一的空白。
在导言区或者适合的位置使用以下命令开启顶部对齐的效果：
```tex
\raggedbottom
```
相反地，`\flushbottom` 用于设置成默认的分散对齐。


**分栏** :
标准文档类的全局选项 onecolumn、twocolumn 可控制全文分单栏或双栏排版。LATEX 也提供了切换单/双栏排版的命令：
```tex
\onecolumn 
\twocolumn[one-column top material]
```

- `\twocolumn` 支持带一个可选参数，用于排版双栏之上的一部分单栏内容。
- 切换单/双栏排版时总是会另起一页（`\clearpage`）。
    - 在双栏模式下使用` \newpage` 会换栏而不是换页；
    - `\clearpage` 则能够换页。
- 双栏排版时每一栏的宽度为 `\columnwidth`，它由 `\textwidth` 减去 `\columnsep` 的差除以 2 得到。
    - 两栏之间还有一道竖线，宽度为 `\columnseprule`，默认为零，也就是看不到竖线。
    - 一个比较好用的分栏解决方案是 multicol，它提供了简单的 multicols 环境（注意不要写成 multicol）自动产生分栏，如以下环境将内容分为 3 栏：

```tex
\begin{multicols}{3}
\end{multicols}
```

- multicol 宏包能够在一页之中切换单栏/多栏，也能处理跨页的分栏，且各栏的高度分布平衡。
- 但代价是在 multicols 环境中无法正常使用 `table` 和 `figure` 等浮动体环境，它会直接让浮动体丢失。
- multicols 环境中只能用跨栏的 `table*` 和 `figure*` 环境，或者用 float 宏包提供 的 H 参数固定浮动体的位置。


## 页眉页脚
`\pagestyle` 来修改页眉页脚的样式：
```tex
\pagestyle{page-style}
```
另外一个命令只影响当页的页眉页脚样式：
```tex
\thispagestyle{page-style}
```
page-style 参数为样式的名称，在 LATEX 里预定义了四类样式 :

| 样式名 | 含义 |
|----|----|
|empty   |  页眉页脚为空|
|plain  | 页眉为空，页脚为页码。（article 和 report 文档类默认；book 文档类的每章第一页也为 plain 格式）|
|headings |页眉为章节标题和页码，页脚为空。（book 文档类默认） |
|myheadings |页眉为页码及 `\markboth` 和 `\markright` 命令手动指定的内容，页脚为空。|

其中 headings 的情况较为复杂：

- article 文档类，twoside 选项： 偶数页为页码和节标题，奇数页为小节标题和页码； 
- article 文档类，oneside 选项： 页眉为节标题和页码； 
- book/report 文档类，twoside 选项： 偶数页为页码和章标题，奇数页为节标题和页码；
- book/report 文档类，oneside 选项： 页眉为章标题和页码。


**手动更改页眉页脚的内容** :
对于 headings 或者 myheadings 样式，LATEX 允许用户使用命令手动修改页眉上面的内容，特别是因为使用了 `\chapter*` 等命令而无法自动生成页眉页脚的情况：
```tex
\markright{right-mark} 
\markboth{left-mark}{right-mark}
```
在双面排版、headings / myheadings 页眉页脚样式下，left-mark 和 right-mark 的内容分别预期出现在左页（偶数页）和右页（奇数页）。
事实上 `\chapter`、`\section` 等命令内部也使用 `\markboth` 或者 `\markright` 写页眉。LATEX 默认将页眉的内容都转为大写字母。如果不喜欢这样，可以:
```tex
\renewcommand\chaptermark[1]{% 
    \markboth{Chapter \thechapter\quad #1}{}} 
\renewcommand\sectionmark[1]{% 
    \markright{\thesection\quad #1}}
```


**fancyhdr 宏包** :
fancyhdr 宏包改善了页眉页脚样式的定义方式，允许将内容自由安置在页眉和页脚的左、中、右三个位置，还为页眉和页脚各加了一条横线。
fancyhdr 自定义了样式名称 fancy。使用 fancyhdr 宏包定义页眉页脚之前，通常先用 `\pagestyle{fancy}` 调用这个样式。在 fancyhdr 中定义页眉页脚的命令为：
```tex
\fancyhead[position]{...} 
\fancyfoot[position]{...}
```
position 为 L（左）/C（中）/R（右）以及与 O（奇数页）/E（偶数页）字母的组合。
这段代码可以用于导言区，它的效果为将章节标题放在和 headings 一致的位置，但使用加粗格式；页码都放在页脚正中；修改横线宽度，“去掉” 页脚的横线。比如:
```tex
% 导言区部分
\usepackage{fancyhdr} 
\pagestyle{fancy}
\renewcommand{\chaptermark}[1]{\markboth{#1}{}} 
\renewcommand{\sectionmark}[1]{\markright{\thesection\ #1}}
\fancyhf{}                              % 清空当前的页眉页脚 
\fancyfoot[C]{\bfseries\thepage} 
\fancyhead[LO]{\bfseries\rightmark}
\fancyhead[RE]{\bfseries\leftmark} 
\renewcommand{\headrulewidth}{0.4pt}    % 注意不用 \setlength \renewcommand{\footrulewidth}{0pt}
```

# 特色工具和功能

## 参考文献和 BIBTEX 工具

**基本的参考文献和引用** :
LATEX 提供的参考文献和引用方式比较原始，需要用户自行书写参考文献列表。不同学术论文对参考文献列表的格式要求不一样，自行书写是一件极其头疼的事情。
LATEX 提供了最基本的 \cite 命令用于在正文中引用参考文献：
```tex
\cite{citation}
```

- citation 为引用的参考文献的标签，类似 `\ref` 里的参数；
- `\cite` 带一个可选参数，为引用的编号后加上额外的内容，如 `\cite[page 22]{pa}` 可能得到形如 [13, page 22] 这样的引用。

参考文献由 thebibliography 环境包裹。每条参考文献由 `\bibitem` 开头，其后是参考文献本身的内容：
```tex
\bibitem[item number]{citation}
```
item number 自定义参考文献的序号，如果省略，则按自然排序给定序号。


**BIBTEX 数据库** :
BIBTEX 数据库以 `.bib` 作为扩展名，其内容是若干个文献条目，每个条目的格式为：
```tex
@type{citation, 
    key1    = {value1}, 
    key2    = {value2}, 
    ...
}
```
type 为文献的类别，如 article 为学术论文，book 为书籍，incollection 为论文集中的某一篇，等等。citation 为 \cite 命令使用的文献标签。在 citation 之后为条目里的各个数据项，以 key = {value} 的形式组织。
在此简单列举学术论文里使用较多的 BIBTEX 文献条目类别：

- article 学术论文，必需数据项有 author, title, journal, year; 可选数据项包括 volumn, number, pages, doi 等；
- book 书籍，必需数据项有 author/editor, title, publisher, year; 可选数据项包括 volumn/number, series, address 等；
- incollection 论文集中的一篇，必需数据项有 author, title, booktitle, publisher, year; 可选数 据项包括 editor, volumn/number, chapter, pages, address 等；
- inbook 书中的一章，必需数据项有 author/editor, title, chapter/pages, publisher, year; 可选数 据项包括 volumn/number, series, address 等。

例如 article 类别的参考文献数据条目写法如下：
```tex
@article{Alice13, 
    title = {Demostration of bibliography items}, 
    author = {A. Alice and B. Bob and C. Charlie and D. Danny}, 
    year = {2013}, journal = {Journal of \TeX perts}, 
    volume = {36}, 
    number = {7}, 
    pages = {114-120}}
```
所有类别的文献条目格式请参考[这里](CTAN://biblio/bibtex/base/btxdoc.pdf)
多数时候，无需自己手写 BIBTEX 文献条目。从 Google Scholar 或者期刊/数据库的网站上都能够导出 BIBTEX 文献条目。


**BIBTEX 样式** :
参考文献的写法在不同文献里千差万别，包括作者、标题、年份等各项的顺序和字体样式、文献在列表中的排序规则等。
BIBTEX 用样式（style）来管理参考文献的写法。BIBTEX 提供了几个预定义的样式，如 plain, unsrt, alpha 等。如果使用期刊模板的话，可能会提供自用的样式。样式文件以 `.bst` 为扩展名。
使用样式文件的方法是在源代码内（一般在导言区）使用 `\bibliographystyle`：
```tex
\bibliographystyle{bst-name}
```
bst-name 为 `.bst` 样式文件的名称，不要带 `.bst` 扩展名。
以上一节给出的数据条目为例，使用 `\bibliographystyle` 选择不同的参考文献样式，效果大致如下。

![](/images/math4.png)

**使用 BIBTEX 排版参考文献** :

- 第一步：当然需要一份 BIBTEX 数据库，假设数据库文件名为 books.bib，和 LATEX 源代码一般位于同一个目录下。
- 第二步：在源代码中添加必要的命令。假设源代码名为 demo.tex

    1. 首先需要使用命令 `\bibliographystyle` 设定参考文献的格式。
    2. 其次，在正文中引用参考文献。BIBTEX 程序在生成参考文献列表的时候，通常只列出用了 `\cite` 命令引用的那些。如果需要列出未被引用的文献，则需要 `\nocite{citation}`；而 `\nocite{*}` 则让所有未被引用的文献都列出。
    3. 再次，在要列出参考文献的位置，使用 `\bibliography` 代替 thebibliography 环境：

```tex
\documentclass{article} 
\bibliographystyle{plain} 

\begin{document} 
\section{Some words} 
Some excellent books, for example, 
\cite{citation1} 
and \cite{citation2} \ldots

\bibliography{books} % 参数为是 BIBTEX 数据库的文件名，不要带 .bib 扩展名。
\end{document}
```

- 第三步：写好了以上两个文件之后，就可以开始编译了。
    1. 首先使用 pdflatex 或 xelatex 等命令编译 LATEX 源代码 demo.tex；
    2. 接下来用 bibtex 命令处理 demo.aux 辅助文件记录的参考文献格式、引用条目等信息。 bibtex 命令处理完毕后会生成 demo.bbl 文件，内容就是一个 thebibliography 环境；
    3. 再使用 pdflatex 或 xelatex 等命令把源代码 demo.tex 编译两遍，读入参考文献并正确生成引用

注意：`\bibliographystyle` 和 `\bibliography` 缺一不可，没有这两个，使用 BIBTEX 生成参考文献列表的时候会报错。

**natbib 宏包** :
时下许多学术期刊比较喜欢使用人名——年份的引用方式，形如 (Alice et al., 2013)。natbib 宏包提供了对这种“自然”引用方式的处理。 
除了 `\cite` 之外，natbib 宏包在正文中支持两种引用方式：
```tex
\citep{citation} 
\citet{citation}
```
natbib 宏包同样也支持数字引用，并且支持将引用的序号压缩，例如：
```tex
\usepackage[numbers,sort&compress]{natbib}
```
调用 natbib 宏包时指定以上选项后，连续引用多篇文献时，会生成形如 (3-7) 的引用而不是 (3, 4, 5, 6, 7)。
natbib 宏包还有更多选项和用法，比如默认的引用是用小括号包裹的，可指定 square 选项改为中括号.

## 索引和 makeindex 工具
要使用索引，须经过这么几个步骤:

1. 在 LATEX 源代码的导言区调用 makeidx 宏包，并使用 `\makeindex` 命令开启索引的收集：`\usepackage{makeidx}`, `\makeindex`
2. 在正文中需要索引的地方使用 \index 命令。`\index` 命令的参数写法详见下一小节；并在需要输出索引的地方（如所有章节之后）使用 `\printindex` 命令。
3. 编译过程：第一, 首先用 xelatex 等命令编译源代码 demo.tex。编译过程中产生索引记录文件 demo.idx；第二, 用 makeindex 程序处理 demo.idx，生成用于排版的索引列表文件 demo.ind；第三, 再次编译源代码 demo.tex，正确生成索引列表。


**索引项的写法** :
添加索引项的命令为：
```tex
\index{index entry}
```
其中 index entry 为索引项, 写法由表 6.2 汇总。其中 !、@ 和 | 为特殊符号，如果要向索引项直接输出这些符号，需要加前缀 `"`；而 " 需要输入两个引号` ""` 才能输出到索引项。
![](leanote://file/getImage?fileId=5ab34f72ab64412132000d62)

## 使用颜色
LATEX 原生不支持颜色，它依赖 color 宏包或者 xcolor 宏包.
```tex
\color[color-mode]{code} 
\color{color-name}
```
颜色的表达方式有两种，其一是使用色彩模型和色彩代码，代码用 0 ∼ 1 的数字代表成分的比例。color 宏包支持 rgb、cmyk 和 gray 模型，xcolor 支持更多的模型如 hsb 等。
```tex
% 下面的例子颜色就如文字的描述
\large\sffamily 
{\color[gray]{0.6} 
    60\% 灰色} \\
{\color[rgb]{0,1,1} 
    青色}
```
其二是直接使用名称代表颜色，前提是已经定义了颜色名称（没定义的话会报错）：
```tex
\large\sffamily
{\color{red} 红色} \\ 
{\color{blue} 蓝色}
```
color 宏包仅定义了 8 种颜色名称，xcolor 补充了一些，总共有 19 种, 自行查阅。 
xcolor 还支持将颜色通过表达式混合或互补：
```tex
\large\sffamily 
{\color{red!40} 40\% 红色}\\ 
{\color{blue}蓝色
    \color{blue!50!black}蓝黑 
    \color{black}黑色}\\ 
{\color{-red}红色的互补色}
```
还可以通过命令自定义颜色名称，注意这里的 color-mode 是必选参数：
```tex
\definecolor{color-name}{color-mode}{code}
```
如果调用 color 或 xcolor 宏包时指定 dvipsnames 选项，就有额外的 68 种颜色名称可用。xcolor 宏包还支持通过指定其它选项载入更多颜色名称。


**带颜色的文本和盒子** :
原始的 `\color` 类似于 `\bfseries`，它使之后排版的内容全部变成指定的颜色，所以直接使用时通常要加花括号分组。color / xcolor 宏包都定义了一些方便用户使用的带颜色元素。
输入带颜色的文本可以用类似 `\textbf`：
```tex
\textcolor[color-mode]{code}{text} 
\textcolor{color-name}{text}
```
以下命令构造一个带背景色的盒子，material 为盒子中的内容：
```tex
\colorbox[color-mode]{code}{matrial}
\colorbox{color-name}{material}
```
以下命令构造一个带背景色和有色边框的盒子，fcode 或 fcolor-name 用于设置边框颜色:
```tex
\fcolorbox[color-mode]{fcode}{code}{material}
\fcolorbox{fcolor-name}{color-name}{material}
```
例:
```tex
\sffamily 
文字用\textcolor{red}{红色}强调\\ 
\colorbox[gray]{0.95}{浅灰色背景} \\ 
\fcolorbox{blue}{yellow}
{% 
    \textcolor{blue}{蓝色边框+文字，% 黄色背景}
}
```

## 使用超链接
hyperref 宏包涉及到的链接遍布 LATEX 的每一个角落——目录、引用、脚注、索引、参考文献等等都被封装成链接。但这也使得它与其它宏包的冲突机会大大增加，虽然宏包已经尽力解决各方面的兼容性，但仍不能面面俱到。为减少冲突的可能性，习惯上将 hyperref 宏包**放在其它宏包之后**调用。
与 graphicx 宏包类似，latex + dvipdfmx 命令下调用 hyperref 宏包时，需要指定选项 dvipdfmx；而 pdflatex 或 xelatex 命令下不需要。
```tex
% 提供以下两种方法去初始化
\hypersetup{option1 ,option2={value},...} 
\usepackage[option1 ,option2={value},...]{hyperref}
```
hyperref 宏包提供的参数设置请自行查表. 
hyperref 宏包提供了直接书写超链接的命令，用于在 PDF 中生成 URL：
```tex
\url{url} 
\nolinkurl{url}
```
`\url` 和 `\nolinkurl `都生成可以点击的 URL，区别是前者有彩色，后者没有。在 `\url` 中作为参数的 URL 里，可直接输入如 `%`、`&` 这样的特殊符号。
也可以像网页一样，把一段文字赋予其“超链接”的作用：
```tex
\href{url}{text}
% 比如:
\href{http://wikipedia.org}{Wiki}
```
用户也可对某个 `\label` 命令定义的标签 label 作超链接（注意这里的 label 虽然是可选参数的形式， 但通常是必填的）：
```tex
hyperref{label}{text}
```
默认的超链接在文字外边加上一个带颜色的边框（在打印 PDF 时边框不会打印），可指定 colorlinks 参数修改为将文字本身加上颜色，或修改 pdfborder 参数调整边框宽度以“去掉”边框；hidelinks 参数则令超链接既不变色也不加边框。
```tex
\hypersetup{hidelinks} 
% or:
\hypersetup{pdfborder={0 0 0}}
```


**PDF 书签** :
对于章节命令 `\chapter`、`\section`
等，默认情况下会为 PDF 自动生成书签。和交叉引用、索引等类似，生成书签也需要多次编译源代码，第一次编译将书签记录写入 .out 文件，第二次编译才正确生成书签。
hyperref 还提供了手动生成书签的命令：
```tex
\pdfbookmark[level]{bookmark}{anchor}
```
bookmark 为书签名称，anchor 为书签项使用的锚点（类似交叉引用的标签）。可选参数 level 为书签的层级，默认为 0。

# 绘图功能
现在流行的绘图代码有以下几种：

- **PSTricks** : 以 PostSciprt 语言的功能为基础的绘图宏包，具有优秀的绘图能力。它对老式的 latex + dvips 编译命令支持最好，而现在的几种编译命令下使用起来都不够方便。
- **TikZ & pgf** : 德国的 Till Tantau 在开发著名的 LATEX 幻灯片文档类 beamer 时一并开发了绘图宏包 pgf， 目的是令其能够在 pdflatex 或 xelatex 等不同的编译命令下都能使用。TikZ 是在 pgf 基础上封装的一个宏包，采用了类似 METAPOST 的语法，提供了方便的绘图命令，绘图 能力不输 PSTricks。
-  **METAPOST & Asymptote** : METAPOST 脱胎于高德纳为 TEX 配套开发的字体生成程序 METAFONT，具有优秀的绘图 能力，并能够调用 TEX 引擎向图片中插入文字和公式。Asymptote 在 METAPOST 的基础 上更进一步，具有一定的类似 C 语言的编程能力，支持三维图形的绘制。 它们往往需要把代码写在单独的文件里，用特定的工具去编译，也可以借助特殊的宏包在 LATEX 代码里直接使用

## TikZ 绘图语言
```tex
\tikz[...] tikz code; 
\tikz[...] {tikz code 1 ; tikz code 2; ...} 

\begin{tikzpicture}[...] 
    tikz code 1 ; 
    tikz code 2;
    . . . . . .
\end{tikzpicture}
```

- 前一种用法为 `\tikz` 带单条绘图命令，以分号结束，一般用于在文字之间插入简单的图形；
- 后两种用法较为常见，使用多条绘图命令，可以在 figure 等浮动体中使用。


**TikZ 坐标和路径** :
TikZ 用直角坐标系或者极坐标系描述点的位置, (x,y) 或者 ($\theta$ : r). 
还可以为某个点命名：\coordinate (A) at (coordinate) 然后就可以使用 (A) 作为点的位置了。
```tex
% 以下命令画出了三条直线
\begin{tikzpicture} 
    \draw (0,0) -- (30:1); 
    \draw (1,0) -- (2,1); 
    
    \coordinate (S) at (0,1); 
    \draw (S) -- (1,1); 
\end{tikzpicture}

% 以下命令画出了一个正方形
% 坐标的表示形式还包括“垂足”形式：
\begin{tikzpicture} 
    \coordinate (S) at (2,2); 
    \draw[gray] (0,2) -- (S);
    \draw[gray] (2,0) -- (S); 
    \draw[red] (0,0) -- (0,0 -| S);
    \draw[blue] (0,0) -- (0,0 |- S); 
\end{tikzpicture}
```
TikZ 最基本的路径为两点之间连线，如 (x1,y1) -- (x2,y2)，可以连用表示多个连线（折线）.连续使用连线时，可以使用 cycle 令路径回到起点，生成闭合的路径。
```tex
\begin{tikzpicture} 
    \draw (0,0) -- (1,1) -- (2,0) -- cycle; 
\end{tikzpicture}
```
其它常用的路径还包括：
```tex
矩形、圆和椭圆：
\begin{tikzpicture} 
    \draw (0,0) rectangle (1.5,1); 
    \draw (2.5,0.5) circle [radius=0.5]; 
    \draw (4.5,0.5) ellipse [x radius=1,y radius=0.5];
\end{tikzpicture}

% 直角、圆弧、椭圆弧：
\begin{tikzpicture}
    \draw (0,0) |- (1,1); 
    \draw (1,0) -| (2,1); 
    \draw (4,0) arc (0:135:1); 
    \draw (6,0) arc (0:135:1 and 0.5); 
\end{tikzpicture}
% 其他的线自己查阅
```
网格、函数图像; 网格可用 step 参数控制网格大小，函数图像用 domain 参数控制定义域：
```tex
\begin{tikzpicture} 
    \draw[help lines,step=0.5] (-2,-2) grid (2,2);
    \draw[->] (-2.5,0) -- (2.5,0); 
    \draw[->] (0,-2.5) -- (0,2.5); 
    \draw[domain=-2:2] 
    plot(\x,{\x*\x -2}); 
\end{tikzpicture}
```


**TikZ 绘图命令和参数** :
除了 `\draw` 命令之外，TikZ 还提供了 `\fill `命令用来填充图形，`\filldraw` 命令则同时填充和描边。除了矩形、圆等现成的闭合图形外，`\fill `和 `\filldraw` 命令也能够填充人为构造的闭合路径。
```tex
\draw[...] path; 
\fill[...] path; 
\filldraw[...] path;
```
TikZ 还提供了 scope 环境，令一些绘图参数在局部起效：
```tex
\begin{tikzpicture}
    \draw (0,0) rectangle (2.5, 2.5); 
    \begin{scope}[fill=gray,scale=0.5]
        \filldraw (0,0) rectangle (2.5, 2.5); 
    \end{scope}
\end{tikzpicture}
```
一些常见的绘图参数请自行查阅书籍. 


**TikZ 文字结点** :
文字节点就是该处的坐标会显示 text 的内容; TikZ 用 \node 命令绘制文字结点：
```tex
\node[options] (name) at (coordinate) {text};
% options 可以省略
```
比如: 
```tex
\begin{tikzpicture} 
    \node (A) at (0,0) {A}; 
    \node (B) at (1,0) {B}; 
    \node (C) at (60:1) {C}; 
    \draw (A) -- (B) -- (C) -- (A);
\end{tikzpicture}
```


TikZ 通过 pgffor 功能宏包实现了简单的循环功能，语法为：
```tex
\foreach \a in {list} {commands}
```
上述语法定义了 `\a `为变量，在 `{commands}` 中使用 `\a` 完成循环。
```tex
% 下列例子画出一条尺子
\begin{tikzpicture} 
    \draw (0,0)--(5,0);
    \foreach \i in {0.0,0.1,...,5.0} 
        {\draw[very thin] (\i,0)--(\i,0.15);}
    \foreach \I in {0,1,2,3,4,5} 
        {\draw (\I,0)--(\I,0.25) node[above] {\I};} 
\end{tikzpicture}
```
`\foreach` 还可使用变量对参与循环：
```tex
\begin{tikzpicture} 
    \foreach \n/\t in {0/\alpha,1/\beta,2/\gamma} 
        {\node[circle,fill=lightgray,draw] at (\n,0) {$\t$};} 
\end{tikzpicture}
```

# 自定义 LATEX 命令和功能
## 自定义命令和环境

**定义新命令** :
```tex
\newcommand{\name}[num]{definition}
```
`\name` 的 `\` 是不可以省略的, num 是可选的，用于指定新命令所需的参数数目 (最多 9 个), 如果缺省可选参数，默认就是 0，也就是新建的命令不带任何参数。
例如 :
```tex
% \tnss。这个命令是本手册英文名称 “The Not So Short Introduction to LATEX2ε” 的简写。

\newcommand{\tnss}{The not so Short Introduction to \LaTeXe}
This is ‘‘\tnss’’ \ldots{} ‘‘\tnss’’ % 显示新命令
```
第二个例子演示了如何定义一个带参数的命令。在命令的定义中，标记 #1 代表指定的参数。
如果想使用多个参数，可以依次使用 #2、...、#9 等标记。
```tex
\newcommand{\txsit}[1] 
    {This is the \emph{#1} 
    Short Introduction to \LaTeXe}
% in the document body: 
\begin{itemize} 
    \item \txsit{not so} 
    \item \txsit{very} 
\end{itemize}
```


**定义环境** :
```tex
\newenvironment{name}[num]{before}{after}
```
同样地，`\newenvironment` 有一个可选的参数。在 before 中的内容将在此环境包含的文本之前处理，而在 after 中的内容将在遇到 `\end{name}` 命令时处理。
```tex
\newenvironment{king} 
{\rule{1ex}{1ex}        % before 的内容
    \hspace{\stretch{1}}} 
{\hspace{\stretch{1}}   % after 的内容
\rule{1ex}{1ex}}

\begin{king} 
    My humble subjects \ldots 
\end{king}
```


## 编写自己的宏包和文档类
写一个宏包的基本工作就是将原本在你的文档导言区里很长的内容拷贝到另一个文件中去，这个文件需要以 .sty 作扩展名。还需要在最前面加上 `\ProvidesPackage{package name}`
宏包的一个最简示例：
```tex
% Demo Package by Tobias Oetiker 
\ProvidesPackage{demopack} % 
    \newcommand{\tnss}
        {The not so Short Introduction to \LaTeXe}
    \newcommand{\txsit}[1]
        {The \emph{#1} Short Introduction to \LaTeXe}
    \newenvironment{king}{\begin{quote}}{\end{quote}}
```
在宏包中调用其它宏包：
```tex
\RequirePackage[options]{package name}
```


**编写自己的文档类** :
当更进一步，需要编写自己的文档类，如论文模板等，问题就稍稍麻烦了一些。首先，自己的文档类以 `.cls` 作扩展名，开头使用 `\ProvidesClass`：
```tex
\ProvidesClass{class name}
```

- 但是有了上述命令和之前学到的 `\newcommand` 等，还并不能完成一个文档类的编写，因为诸如 `\chapter`、`\section` 等等许多常用的命令都是在文档类中定义的。
- 事实上，许多时候只需要像调用宏包那样调用一个基本的文档类，省去许多不必要的麻烦。

在文档类中调用其它文档类的命令是 `\LoadClass` ，用法和 `\documentclass` 十分相像：
```tex
\LoadClass[options]{package name}
```

## 计数器
LATEX 对文档元素自动计数的能力：章节符号、列表、图表都是依靠 LATEX 提供的“计数器”完成的。
定义一个计数器的方法为：
```tex
\newcounter{counter name} 
\newcounter{counter name}[parent counter name] % parent name 设置上级计数器, 比如标题可以有两级
```
以下命令修改计数器的数值，

- `\setcounter` 将数值设为 number；
- `\addtocounter` 将数值加上 number；
- `\stepcounter` 将数值加一，并将所有下级计数器归零。3

```tex
\setcounter{counter name}{number} 
\addtocounter{counter name}{number} 
\stepcounter{counter name}
```


**LATEX 中的计数器** :

- 所有章节命令 \chapter、\section 等分别对应计数器 chapter、 section 等等，而且有上下级的关系。而计数器 part 是独立的。
- 有序列表 enumerate 的各级计数器为 enumi, enumii, enumiii, enumiv，也是有上下级的关 系。
- 图表浮动体的计数器就是 table 和 figure；公式的计数器为 equation。这些计数器在 article 文档类中是独立的，而在 book 和 report 中以 chapter 为上级计数器。
- 页码、脚注的计数器分别是 page 和 footnote。

可以利用前面介绍过的命令，修改计数器的样式以达到想要的效果，比如把页码修改成 大写罗马字母，左右加横线，或是给脚注加上方括号：
```tex
\renewcommand\thepage{--~\Roman{page}~--} 
\renewcommand\thefootnote{[\arabic{footnote}]}
```

# 安装和更新宏包
TEX Live 提供了图形界面的宏包管理器 TEX Live Manager 用于安装和更新宏包，而 MikTEX 也提供了管理器 MikTEX Package Manager。
```bash
% TeX Live 命令行工具 tlmgr 的使用示例
tlmgr install <package-name>    % 安装某个宏包
tlmgr remove <package-name>     % 卸载某个宏包 
tlmgr update --all --self       % 更新所有宏包（包括 tlmgr 本身）
tlmgr update --list             % 列出所有可更新的宏包
tlmgr repository set http://.../CTAN/systems/texlive/tlnet
                                % 指定更新源（CTAN）地址

% MikTeX 命令行工具 mpm 的使用示例 
% 建议始终加 --admin 参数使用 
mpm --admin --install <package-name>    % 安装某个宏包 
mpm --admin --uninstall <package-name>  % 卸载某个宏包 
mpm --admin --update                    % 更新所有宏包
mpm --admin --set-repository=http://.../CTAN/systems/win32/miktex/tm/packages 
                                        % 指定更新源（CTAN）地址
mpm --admin --print-package-info <package-name> % 查看宏包信息
```
TEX Live 默认安装所有宏包，而 MikTEX 的安装程序只包含了若干用于 LATEX 的基本宏包。
如非万不得已，尽量不要手动安装宏包。

# 排除错误、寻求帮助
比如说有一个明显出错的例子： 
```tex
\documentclass{article}
\begin{document} Test 
    \LaTEx{} and it’s friends. 
\end{document}
```
编译过程中遇到这个错误将会停顿下来，提示错误，并等待用户输入指令：
```tex
! Undefined control sequence.
l.3 Test \LaTEx 
            {} and it’s friends.
```
这种错误信息分两部分，前一部分提示了错误的信息，后一部分指出了错误发生的行号，以及通过错落的文字告知发生错误的命令所在位置。如上错误显示 `\LaTEx` 位置发生了错误，错误信息是“未定义的控制序列”，意思是 `\LaTEx` 是 TEX 编译器无法识别的一个命令，很显然是把 `\LaTeX` 的大小写写错了。
