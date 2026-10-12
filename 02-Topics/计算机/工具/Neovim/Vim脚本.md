>[Vim 脚本指南 | SpaceVim](https://spacevim.org/cn/development/vim-script/#:~:text=Vim%20%E8%84%9A%E6%9C%AC%E6%98%AF%20Vim%2FNeovim,%E7%BC%96%E8%BE%91%E5%99%A8%E5%86%85%E7%BD%AE%E7%9A%84%E8%84%9A%E6%9C%AC%E8%AF%AD%E8%A8%80%E3%80%82%20%E5%8F%AF%E4%BB%A5%E7%94%A8%E4%BA%8E%E9%85%8D%E7%BD%AE%20Vim%2FNeovim%20%E5%8F%8A%E5%BC%80%E5%8F%91%E6%8F%92%E4%BB%B6%E3%80%82)
# Vim 命令行
>[Site Unreachable](https://zhuanlan.zhihu.com/p/76531156)
- `:e`: 打打开一个新的文件，同时关闭当前文件, 和 `sp` 新建一个窗口不一样
- `[v]split`: 打开水平/垂直窗口,
	- **`Ctrl + w, h/j/k/l`**: 在分割的窗口之间切换
	- **`:close`**: 关闭当前窗口
- `:ctrl r`: 插入模式,从不同的寄存器中插入文本。
	- **`Ctrl+r Ctrl+w`**：将光标下的单词（Word）粘贴到命令行中。这里的单词是指以空格、制表符或其他空白字符为边界的词。
	- **`Ctrl+r Ctrl+a`**：将光标下的 WORD（大写的WORD）粘贴到命令行中。这里的WORD是指任意字符组成的单词，包括符号，且不受空格等字符的影响。
	- **`Ctrl+r %`**：将当前文件的文件名插入到命令行中。
	- **`Ctrl+r "`**：将默认寄存器（通常是最近删除或复制的文本）粘贴到命令行中。
	- **`Ctrl+r 1`、`2`、…、`9`**：这些数字是指特定的寄存器，Vim允许用户在多个寄存器之间存储不同的文本。
- `:ctrl u`: 撤销命令行行为
- `:s/old/new/g`: 在当前行中替换`old`为`new`（使用`g`表示全局替换）
- `:%s/old/new/g`: 在整个文件中替换
- `:help 命令`
- `:help command-line-window`命令，可以查看命令行窗口的更多帮助信息。
- `:%s/<Ctrl+r><Ctrl+w>//g`:将光标下的单词插入到命令行中，而不需要手工输入替换的文字
`:set i` 之后，按下Tab或`Ctrl+D`键，将显示以“i”开头的set命令；继续按Tab键，则可以在这些命令列表间移动，按下回车键就可以执行该命令。
- 在命令行模式下，使用CTRL-F快捷键打开命令行窗口，并显示命令历史纪录；  
    请注意，您可以使用`:set cedit`命令，更改此快捷键。
- 在常规模式下，使用`q:`命令打开命令行窗口，并显示命令历史纪录；
- 在常规模式下，使用`q/`命令打开命令行窗口，并显示向前查找（search forward）的历史纪录；
- 在常规模式下，使用`q?`命令打开命令行窗口，并显示向后查找（search backward）的历史纪录；
可以将命令行窗口，视为常规的[缓冲区 (Buffer)](https://link.zhihu.com/?target=http%3A//yyq123.github.io/learn-vim/learn-vi-13-MultiBuffers.html)来操作。使用k和j键，可以在命令历史纪录中上下移动；也可以使用`/`命令查找命令历史纪录，并在此基础上进行修改，然后点击`<Enter>`键来执行命令（命令行窗口也将同时关闭）。
如果同时打开多个缓冲区 (Buffer)，那么可以在一个缓冲区的命令行窗口使用 `yy` 命令复制一条命令，然后在另一个缓冲区的命令行窗口中粘贴并执行该命令，或者在命令行中使用 `:@"<CR>` 来执行复制的命令。也就是说，你可以很方便地在多个缓冲区中，重复执行命令（比如相同的:%s/old/new/g替换操作），而不必多次手工输入命令。
## `:set`
在Vim中，`:set` 是一个非常重要的命令，用于设置和修改Vim的各种选项和行为。通过这个命令，用户可以调整Vim的配置，使其更加符合个人的使用习惯。下面是对 `:set` 命令的详细解释和常用选项的介绍。
##### 1. 基本用法
- **命令格式**：
  ```vim
  :set option
  ```
  - `option` 是你想要设置的选项名称，可以是开启某个功能或者修改某个配置。
##### 2. 常用选项
这里列出一些常用的 `:set` 选项及其说明：
- **行号/相对行号**：
  - `:set number`：显示行号。
  - `:set relativenumber`：显示相对行号，当前行是0，上方和下方的行号相对显示。
- **语法高亮**：
  - `:set syntax on`：启用语法高亮。
- **自动换行**：
  - `:set wrap`：启用自动换行。
  - `:set nowrap`：禁用自动换行。
- **制表符设置**：
  - `:set tabstop=4`：设置制表符宽度为4个空格。
  - `:set expandtab`：将Tab转为空格（每次按Tab插入空格）。
  - `:set shiftwidth=4`：设置自动缩进时的空格数。
- **搜索设置**：
  - `:set ignorecase`：在搜索时忽略字母大小写。
  - `:set smartcase`：如果搜索中包含大写字母，则搜索时区分大小写。
  - `:set hlsearch`：高亮显示搜索结果。
  - `:set incsearch`：实时高亮搜索结果。
- **历史记录和缓存**：
  - `:set history=1000`：设置命令历史记录的条目数。
##### 3. 查看当前选项
要查看当前的选项及其值，可以在命令模式下输入：
```vim
:set
```
这将列出所有设置的选项及其当前值。
##### 4. 保存设置
- 你可以将 `:set` 命令及其选项添加到 `~/.vimrc` 文件中，以便每次启动Vim时自动加载这些设置。例如：
  ```vim
  set number
  set relativenumber
  set tabstop=4
  set expandtab
  ```
##### 5. 通过 `:set` 取消选项
如果需要取消某个选项，可以使用 `no` 关键词。例如：
- `:set nowrap` 取消自动换行。
- `:set nohlsearch` 取消高亮搜索结果。

# 语法
>[Vim脚本语言入门：打造你的编辑器-CSDN博客](https://blog.csdn.net/fudaihb/article/details/137335437)
Vim脚本中的条件判断语句使用 `if`、`else` 和 `endif` 关键字：
```shell
let variable_name = value
- 字符串：使用单引号或双引号表示，如 `'hello'` 或 `"world"`
- 数字：整数或浮点数，如 `42` 或 `3.14`
- 列表：用方括号括起来的一系列值，如 `[1, 2, 3]`
- 字典：用大括号括起来的键值对，如 `{'name': 'Alice', 'age': 30}`
if condition
    " 条件为真时执行的代码
elseif another_condition
    " 另一个条件为真时执行的代码
else
    " 所有条件都不满足时执行的代码
endif
for variable in range
    " 循环体
endfor
#while
while condition
    " 循环体
endwhile
# 函数
function MyFunction(argument1, argument2)
    " 函数体
endfunction
call MyFunction(value1, value2)
# 使用 `function!` 允许你定义一个函数，即使同名函数已经存在。在这种情况下，新定义的函数会覆盖旧的函数定义。
function Add(a, b)
    return a + b
endfunction
let result = Add(3, 5)
```
# Vim 脚本的执行
## 命令行模式执行
```shell
:source filename.vim
```
## 自动命令
Vim脚本可以通过设置自动命令来在特定事件发生时自动执行相应的操作。自动命令可以在Vim启动时执行、文件读取或保存时执行、光标位置改变时执行等。
例如，以下命令会在Vim启动时执行 `MyScript.vim` 脚本：
```shell
autocmd VimEnter * source MyScript.vim
```
## 映射键位
通过映射键位，可以将Vim命令或者自定义函数与按键绑定，从而实现快捷操作。Vim脚本中的映射键位可以在脚本文件中设置，也可以直接在Vim的命令行模式下临时设置。
```vim
" 在脚本文件中设置键位映射
nnoremap <F5> :call MyFunction()<CR>
" 在命令行模式下设置临时键位映射
:nnoremap <F5> :call MyFunction()<CR>
```
## 创建自定义命令
假设你经常需要查找文本中的特定模式，并进行一些处理。你可以编写一个自定义命令来简化这个过程：
```vim	
command! -nargs=1 MySearch :%s/<args>/replacement/g
```
将以上代码保存到一个脚本文件（比如 `custom_commands.vim`），然后在Vim中执行：
```vim
:source custom_commands.vim
```
现在，你可以在Vim中使用 `:MySearch pattern` 命令来查找并替换文本中的特定模式了。
## 高级主题
缓冲区、窗口和标签页
- 缓冲区（Buffer）： 在Vim中，每个打开的文件都会被加载到一个缓冲区中。Vim允许同时在多个缓冲区之间切换，执行各种编辑操作。
- 窗口（Window）： 窗口是Vim界面中用于显示缓冲区内容的部分。Vim支持分割窗口，并且可以在不同的窗口中同时编辑不同的缓冲区。
- 标签页（Tab Page）： 标签页是用于组织窗口的一种方式。每个标签页可以包含多个窗口，使得用户可以方便地在不同的文件之间切换。

## 其它
- ### 调试和错误处理
- #### Vim脚本的异步执行