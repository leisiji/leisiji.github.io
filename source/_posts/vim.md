---
title: vim 基础
date: 2020-03-14 21:26:24
tags: vim
---

使用 vim 已经有半年了，总结 vim 的一些比较有用的知识

<!--more-->

# 移动

## 基本移动

vim 基本命令：

| 按键 | 作用							   |
| ---  | ---							   |
| u    | 撤销							   |
| x    | 删除当前光标的字符				   |
| p    | 在光标后粘贴					   |
| P    | 在光标前粘贴					   |
| y    | yarn							   |
| Y    | 相当于 yy						   |
| d    | delete							   |
| dd   | 删除行							   |
| D    | 从光标处删除至行末，相当 `d$`	   |
| c    | change;						   |
| C    | 从光标处删除至行末，再进入 insert |
| cc   | 和 S 相同						   |
| a    | append;						   |
| A    | 在行末 append					   |
| i    | insert							   |
| I    | 在行首非 blank 字符 insert		   |
| f    | 向后查找						   |
| F    | 向前查找						   |
| U    | visual 下改为大写				   |
| u    | visual 下改为小写				   |
| ~    | 大小写转换						   |
| s    | subsitute a character			   |
| S    | subsitue line					   |
| o    | 在该行前插入新行				   |
| O    | 在该行后插入新行				   |
| J    | Join, 删除行未的换行符，连接两行  |
| #    | 搜索光标下的单词				   |
| .    | 在 normal 下重复上一次的动作	   |
| `<<` | 减小缩进						   |

单词和字符串移动（单词默认是以空格分隔的，字符串以标点作为分隔）：

| 键位 | 含义                       |
| ---  | ---                        |
| w    | 到下一个单词的开头         |
| e    | 到下一个单词的结尾         |
| W    | 到下一个字符串的开头       |
| E    | 到下一个字符串的结尾       |
| B    | 到前一个字符串的首字符上   |
| b    | 到前一个单词的首个字符上   |
| ge   | 向后移动到上一个单词的末尾 |

快速移动（blank 字符就是空格、tab、换行、回车等）：

| 键位 | 含义                              |
| ---  | ---                               |
| `0`  | 数字零，到行头                    |
| `^`  | 到本行第一个不是 blank 字符的位置 |
| `$`  | 到本行行尾                        |
| `g_` | 到本行最后一个不是blank字符的位置 |
| `%`  | 到光标所在这对括号的另外一个      |
| `gg` | 首行                              |
| `G`  | 最后一行                          |

# ctrl 对应的键位

normal 模式下的 ctrl 快捷键：

| 键位			 | 含义									   |
| ---			 | ---									   |
| ctrl+d, ctrl+u | page down / page up (半个屏幕)		   |
| ctrl+f, ctrl+b | page forward / page backward (一个屏幕) |
| ctrl+y/ctrl+e  | 向上/向下移动一行，但是光标不移动	   |
| ctrl+g		 | 显示当前文件相对工程的路径			   |

insert 模式下的 ctrl 快捷键

| 键位			 | 含义					   |
| ---			 | ---					   |
| ctrl+w		 | 删除一个单词			   |
| ctrl+u		 | 从光标位置删除到行首    |
| ctrl+r		 | redo，和 u 的 undo 相反 |
| ctrl+o		 | 进入 normal mode		   |
| ctrl+u		 | 重新编辑本行			   |
| crtl+d, ctrl+t | 左右缩进				   |

# buffer/window/tab

## buffer

vim 启动时打开多个 buffer：
```bash
vim -e file.txt file2.txt
```
命令：

- `:e[dit]`：打开某个文件到缓冲区
- `:bd[elete]`：移出缓冲区
- `Ctrl+^`（无需 Shift）来切换 window 的当前和上一个缓冲区

还有其他的命令：
```vim
:ls, :buffers       "列出所有缓冲区
:bn[ext]            "下一个缓冲区
:bp[revious]        "上一个缓冲区
:b {number, expression}     "跳转到指定缓冲区

:b2					"将会跳转到编号为2的缓冲区
:b exa				"将会跳转到最匹配exa的文件名

" 分屏
:sb 3               "分屏并打开编号为3的Buffer
:vertical sb 3      "同上，垂直分屏
```

## tabe

用多个标签页启动 Vim
```bash
vim -p main.cpp my-oj-toolkit.h /private/etc/hosts
```
常用命令：
```vim
:tabp      "前一个
:tabn      "后一个
:tabr      "第一个
:tablast   "最后一个

" 打开和关闭 tab
:tabe[dit] {file}   "edit specified file in a new tab
:tabf[ind] {file}   "open a new tab with filename given, searching the 'path' to find it
:tabc[lose]         "close current tab
:tabc[lose] {i}     "close i-th tab
:tabo[nly]          "close all other tabs (show only the current tab)
```
normal 跳转前后 tabe 命令：gt, gT
```vim
nnoremap <M-1> 1gt
noremap <M-l> gt	"go to next tab
noremap <M-h> gT	"go to previous tab
```

## window

分屏打开多个文件
```bash
vim -O main.cpp my-oj-toolkit.h
# -o 可以水平分屏
```
vim 的优势在于在不同的 window 下打开同一个文件，可以同时编辑文件的不同地方，常用命令：
```vim
" 打开/关闭窗口
:sp[lit] {file}     " 水平分屏
:new {file}         " 水平分屏
:sv[iew] {file}     " 水平分屏，以只读方式打开
:vs[plit] {file}    " 垂直分屏
:clo[se]            " 关闭当前窗口
```
打开关闭快捷键（有共同的前缀：`Ctrl+w`），`:help window`

- `Ctrl+w s`：水平分割当前窗口
- `Ctrl+w v`：垂直分割当前窗口
- `Ctrl+w q`：关闭当前窗口
- `Ctrl+w n`：打开一个新窗口（空文件）
- `Ctrl+w o`：关闭出当前窗口之外的所有窗口
- `Ctrl+w T`：当前窗口移动到新标签页
- `Ctrl+w ]`：跳转定义在新窗口打开
- `Ctrl+w z`: 关闭当前打开的 preview window

切换 window：

- `Ctrl+w h/j/k/l`（左、下、上、右切换）
- `Ctrl+w w`（遍历切换窗口）
- `Ctrl+w t` 切到最上方的窗口
- `Ctrl+w b` 切到最下方的窗口

调整 window 的大小，分窗口是水平分隔还是垂直分隔：

- 水平分隔：`:nwinc +/-` 把当前激活窗口高度增加、减少 n 个字符高度，如 `:5winc +`
- 垂直分隔：`:nwinc >/<` 把当前激活窗口宽度增加、减少 n 个字符宽度，如 `:5winc >`

# 标记

标记 (mark) 可以实现 vim 的精确定位

vim 用单一的字符表示

- 大小写字母 (A-Za-z) 都可以做为标记的名字，这些标志的位置可由用户设置
- 而数字标记0-9，以及一些标点符号，是 vim 内置的特殊标记

标记规则：

- 小写字母标记局限于缓冲区，各缓冲区间的小写字母标记彼此不干扰
- 大写字母标记是全局的，它在文件间都有效
- 数字标记和标点符号标记可查阅 `:help mark-motions`

标记命令：

- 设定标记：m{a-zA-Z}
- 跳转到指定的标记 \`{a-zA-Z} 或 '{a-zA-Z}
- 跳转时不改变 jumplist：g'{mark} 或 g\`{mark}
- `:delmarks` 删除指定标记
- `:marks` 列出所有的标记

特殊的标记：

| 命令 | 跳转至						  |
| ---  | ---						  |
| `[`  | 上一次修改或复制字符		  |
| `<`  | 上一次在可视模式下选取的字符 |
| `'`  | 上一次跳转之前的光标位置	  |
| `^`  | 上一次插入字符后的光标位置   |
| `(`  | 当前句子的开头				  |
| `{`  | 当前段落的开头				  |
| `}`  | 当前段落的结尾				  |
| `.`  | 最后一次修改的地方			  |

