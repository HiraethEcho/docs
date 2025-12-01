---
title: 一些nvim的配置和使用技巧
date: 2025-08-06
tags:
  - nvim
  - geek
  - tools
categories: handbook
---

# 一些nvim的配置和使用技巧

## cmd

Lua 输出：

```nvim
:lua =vim.opt.runtimepath:get()
```

`:setl ma! ma?` 可以切换一个buffer的只读状态。
注意其他编辑器的只读一般是“你可以修改但不能保存“，这个是你根本不能修改，比如说根本就不能进入插入模式。
可以设一个顺手的keymap，比如说我是nnoremap <silent> ;m :setl ma! ma?<cr> 。

另外，:setl xxx! xxx? 这个是一个常用切换option开关的模式，比如说:nnoremap <silent> <space>w<space> <CMD>setl wrap! wrap?<cr>设keymap来切换自动换行，并在状态栏显示当前实际状态。

```vimscript
xnoremap ,x <ESC>`.`gvp`P
abbrev ,f <C-R>=expand("%:")<left><left>
abbrev ,c <C-R>=getcwd()<cr>
```

下面的keymap可以直接执行一行vimscript或者lua （取决于当前文件类型）。用来调整或者测试vim配置非常方便，改完一行就执行一下，不需要source整个vimrc文件了。

```vimscript
command! -range ExecVimL call execute(getline(<line1>, <line2>), '')
command! -range ExecLua call execute('lua ' . join(getline(<line1>, <line2>), " \n "), '')
nnoremap <silent> <expr> ,r ((&ft=='lua' ? ':ExecLua' : ':ExecVimL') . '<cr>:<c-u>echo "Current line executed"<cr>')
xnoremap <silent> <expr> ,r ((&ft=='lua' ? ':ExecLua' : ':ExecVimL') . '<cr>:<c-u>echo "Selected range executed"<cr>')
```

链接：https://www.zhihu.com/question/636018229/answer/3386613023
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

用`.!`（当前行），`%!`（整个buff），`'<,'>!` (选中）可以把文本作为stdin传给一个外部shell命令，并用其stdout输出内容替换相应文本，用来调外部的格式化命令很方便。另外，如果传给bash或zsh的话就相当于把文本作为shell命令执行。

「修改 1」→「撤销」→「修改 2」，这时如果想退回到「修改 1」，按 `u` 是回不去的。

可以按 `g-`（反方向是 `g+`）

实际上等同于 `:earlier` 和 `:later` 。这两个命令后边是可以跟「次数」而不是「时间」的顺便一说，不写默认是 1 次。

test 2

## start up of nvim

[nvim doc](https://neovim.io/doc/user/starting.html#_initialization)

## lazy.nvim

[ref](https://docs.astronvim.com/configuration/customizing_plugins/) `lazy.nvim` 配置的表合并情况。例如假设`treesitter`的默认配置为

```lua
return {
  "nvim-treesitter/nvim-treesitter",
  opts = {
    ensure_installed = { "lua", "vim" },
    highlight = {
      enable = true,
    },
  },
}
```

手动配置

```lua
return {
  "nvim-treesitter/nvim-treesitter",
  opts = {
    ensure_installed = { "python" },
    highlight = {
      enable = false,
    },
    indent = {
      enable = false,
    },
  },
}
```

那么最后会得到`ensure_installed = { "python" }`。如何要拓展表，则要用

```lua
table.insert(opts.ensure_installed, "python")
```
