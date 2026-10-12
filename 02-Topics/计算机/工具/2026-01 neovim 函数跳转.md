在 LazyVim 里，跳转到函数或变量定义最常用的是 **`gd`**，配合 LSP 使用。[deepwiki+1](https://deepwiki.com/LazyVim/LazyVim/4.1-lsp-keymaps-and-configuration&rut=b1170bed8156cef645dfa1b5a76f273b3a13f0e182153f24475cb382fc899888)​

## 基本跳转操作

- 把光标放在函数名 / 变量名上，按：  
    **`gd`** → 跳转到定义（Go to Definition）。[lazyvim+1](http://www.lazyvim.org/configuration/keymaps)​
    
- 跳回刚才的位置：  
    **`<C-o>`**（Ctrl + o，后退），需求反向则用 `<C-i>`（前进）。[neovim](https://neovim.io/doc/user/lsp.html)​
    

## 其它相关跳转

- **`gD`**：跳到声明（Declaration），有些语言里和 `gd` 不同。[deepwiki](https://deepwiki.com/LazyVim/LazyVim/4.1-lsp-keymaps-and-configuration&rut=b1170bed8156cef645dfa1b5a76f273b3a13f0e182153f24475cb382fc899888)​
    
- **`gr`**：查看所有引用（References），会弹出列表，选中后跳转。[deepwiki](https://deepwiki.com/LazyVim/LazyVim/4.1-lsp-keymaps-and-configuration&rut=b1170bed8156cef645dfa1b5a76f273b3a13f0e182153f24475cb382fc899888)​
    
- **`gI`**：跳到实现（Implementation），比如接口的具体实现类。[deepwiki](https://deepwiki.com/LazyVim/LazyVim/4.1-lsp-keymaps-and-configuration&rut=b1170bed8156cef645dfa1b5a76f273b3a13f0e182153f24475cb382fc899888)​
    

如果 `gd` 没反应，先在当前文件执行 `:LspInfo`，确认对应语言的 LSP 已经 attached；没有的话需要先把语言服务器配置好。
