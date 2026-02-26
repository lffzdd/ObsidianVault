Obsidian 里编辑视图/阅读视图用到的类，确实不止一个，你看到的 `cm-s-obsidian`、`markdown-preview-view` 等，其实是不同层级、不同阶段留下来的“历史产物”。

## 大致结构

从外往里看，一个普通笔记面板大概会长这样（简化）：

- `div.workspace-leaf-content`
    - `div.view-content`
        - `div.markdown-source-view`（编辑视图时）
        - `div.markdown-reading-view`（阅读视图时）

这里就已经有两个核心类：

- 编辑视图：`markdown-source-view`
- 预览/阅读视图：`markdown-reading-view`

## 编辑视图（Source / Live Preview）

新编辑引擎基于 CodeMirror 6，但为了兼容历史和主题生态，会加很多“壳”：

典型结构类似：

- `div.markdown-source-view.mod-cm6`
    - `div.cm-editor.cm-s-obsidian`
        - `div.cm-scroller`
            - `div.cm-content`

几个关键类：

- `markdown-source-view`：整个“编辑视图”的容器。
- `mod-cm6`：表示使用的是新版 CM6 引擎（老版会有 `mod-cm5`）。
- `cm-editor`：CodeMirror 编辑器根节点。
- `cm-s-obsidian`：这是“主题名”，来自 CodeMirror 的传统：`cm-s-<theme-name>`，Obsidian 默认主题就叫 `obsidian`。
    - 你如果切换第三方主题，很多主题仍然沿用这个类来挂样式。
- 还有一堆 token 类：`cm-formatting-header`, `cm-strong`, `cm-em`, `cm-inline-code` 等，用来做语法高亮/结构高亮。

也就是说：  
**编辑模式真正“皮肤”的类主要是 `cm-s-obsidian` + 各种 `cm-*` token，再包在 `markdown-source-view` 里。**

## 阅读视图（Preview）

阅读视图不再是 CodeMirror，而是普通 DOM：

- `div.markdown-reading-view`
    - `div.markdown-preview-view`
        - `div.markdown-preview-sizer`
            - 各种渲染后的 HTML，如：
                - `h1, h2, p, ul, code, pre` 等
                - 再配一堆 `markdown-preview-section`, `internal-link` 等类

关键点：

- 容器：`markdown-reading-view`
- 内容：`markdown-preview-view`
- 里面是“渲染后的 Markdown HTML”，不再是 `cm-*` 这一套。

## 为什么你会觉得“好多套”？

1. 历史原因：
    
    - 早期 Obsidian：编辑用 CM5（`cm-s-obsidian`），预览用 `markdown-preview-view`。
    - 0.13 以后：引入 CM6 + 实时预览，编辑视图结构变化，但为了兼容老主题，很多类继续保留。
        
2. 模式叠加：
    
    - “编辑视图”又分：源码模式（Source mode）和实时预览（Live Preview），它们在同一个 `markdown-source-view` 里，用不同的 `cm-*` token 组合来呈现。
        
3. 主题/插件：
    
    - 很多主题仍然对 `cm-s-obsidian`、`.markdown-preview-view` 分别写 CSS，导致你在 DevTools 里看到一堆看似“冲突”的类。

## 实际上你可以怎么理解/写 CSS

如果你要针对两种视图写样式，可以简单粗暴按层次分：

- 只作用于编辑视图（两种编辑模式一起管）：
    
    css
    
    `.markdown-source-view {   /* 编辑视图整体 */ } .markdown-source-view .cm-s-obsidian {   /* 编辑区内部 */ }`
    
- 只作用于阅读/预览视图：
    
    css
    
    `.markdown-reading-view {   /* 阅读视图整体 */ } .markdown-preview-view {   /* 预览内容 */ }`
    
- 精细到某种 Token（编辑模式的语法高亮）：
    
    css
    
    `.cm-s-obsidian .cm-formatting-header {   /* # 标记部分 */ } .cm-s-obsidian .cm-header-1 {   /* H1 正文部分 */ }`

如果你愿意，你可以把“记忆模型”简化为一句话：

> “编辑视图 = `markdown-source-view` + `cm-s-obsidian`；预览视图 = `markdown-reading-view` + `markdown-preview-view`。”

你现在在 DevTools 里按这个层次去看 DOM，会清晰很多。