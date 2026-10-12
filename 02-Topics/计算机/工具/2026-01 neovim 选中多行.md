在 Vim / LazyVim 里，“选中多行再 Tab”对应的是 **可视模式 + 缩进命令**。

## 选中多行

1. 普通模式下，把光标放到要缩进的第一行或最后一行。
2. 按 **`V`**（大写 v）进入“行可视模式”。[neovim+1](https://neovim.io/doc/user/visual.html)​
3. 用 **`j`/`k`** 把光标移动到另一端，整块行就被选中了。

## 缩进 / 反缩进

- 选中后，按 **`>`**：整块向右缩进一个 `shiftwidth`（通常 2 或 4 空格）。
- 按 **`<`**：整块向左缩进一个层级。[stackoverflow+1](https://stackoverflow.com/questions/7452301/how-to-indent-multiple-lines-on-vim)​

如果你非常想用 Tab / Shift-Tab 来做这个，也可以加映射（可选）：

text

`vnoremap <Tab>   >gv vnoremap <S-Tab> <gv`

这样在可视模式选中多行后按 Tab / Shift-Tab 就能左右缩进，并且保持选区继续高亮，方便多次缩进。[stackoverflow+1](https://stackoverflow.com/questions/235839/indent-multiple-lines-quickly-in-vi)​