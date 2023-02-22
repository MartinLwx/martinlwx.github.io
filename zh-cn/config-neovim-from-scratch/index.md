# 从零开始配置 Neovim(Nvim)


## 版本信息

我使用的是 Macbook pro 2020 Intel 版本，系统版本为 macOS 13.2。我的 `Nvim` 版本信息如下
```
NVIM v0.8.3
Build type: Release
LuaJIT 2.1.0-beta3
Compiled by brew@Ventura

Features: +acl +iconv +tui
See ":help feature-compile"

   system vimrc file: "$VIM/sysinit.vim"
  fall-back for $VIM: "/usr/local/Cellar/neovim/0.8.3/share/nvim"

Run :checkhealth for more info
```

## 为什么选择 Neovim

在使用 `Vim` 一年多之后，我越发觉得 `Vim` 的配置麻烦，启动加载速度也不尽人意。我也很不喜欢 Vimscript 的写法，这导致我决定使用 `Neovim(Nvim)`。我决定**重新配置** `Nvim`。为什么会想要**重新配置**而不是**迁移配置**呢？因为我想顺便趁着这个机会，**重新审视**我本来 `Vim` 的配置，以及将本来用到的的插件替换为现在的 SOTA(State-of-the-art)。~~我自从看完 MIT 的 Missing semester 的课配置了 `~/.vimrc` 之后就很长时间都没有再编辑 `~/.vimrc` 文件了~~

我认为在配置 `Nvim` 的时候弄清楚每一个选项的意思是很有必要的，因此我在这篇博客中会尽量解释清楚每个选项的含义，同时将解释放在注释里面，即我尽量让我自己的配置文件是 **self-contained** 而且可读性强的

> 💡 当然，这难免有疏漏。别忘了我们永远可以在 `Nvim` 里面输入 `:h <name>` 来看到更为详细的解释

> 💡 该篇博客**假定**你对 `Vim` 有基本了解

## Nvim 配置基础知识

### Lua 语言

在配置 `Nvim` 的时候，我会**尽可能**用 Lua 语言写配置，因此你有必要了解一下 Lua 的基本语法和语义。可以快速浏览一下 [Learn Lua in Y minutes](https://learnxinyminutes.com/docs/lua/) 了解大概

### 配置文件路径

`Nvim` 的配置目录在 `~/.config/nvim` 下。在 Linux/Mac 系统上，`Nvim` 会默认读取 `~/.config/nvim/init.lua` 文件，**理论上**来说可以将所有配置的东西都放在这个文件里面，但这样不是一个好的做法，因此我划分不同的文件和目录来分管不同的配置

首先看下按照本篇教程配置 `Nvim` 之后，目录结构看起来会是怎么样⬇️

```
nvim
├── init.lua
└── lua
    ├── colorscheme.lua
    ├── config
    │   └── nvim-cmp.lua
    ├── keymaps.lua
    ├── lsp.lua
    ├── options.lua
    └── plugins.lua
```

**解释如下**
- `init.lua` 为 `Nvim` 配置的 Entry point，我们主要**用来导入其他 `*.lua` 文件**
    - `colorscheme.lua` 配置主题
    - `keymaps.lua` 配置按键映射
    - `lsp.lua` 配置 LSP
    - `options.lua` 配置选项
    - `plugins.lua` 配置插件
- `config` 用于**存放各种插件自身的配置**，文件名为插件的名字，这样比较好找。*这里的 `nvim-cmp.lua` 就是 `nvim-cmp` 插件的配置文件*
- `lua` 目录。当我们在 Lua 里面调用 `require` 加载模块（文件）的时候，它会自动在 `lua` 文件夹里面进行搜索
    - **将路径分隔符从 `/` 替换为 `.`，然后去掉 `.lua` 后缀就得到了 `require` 的参数格式**
    - *比如要导入上面的 `nvim-cmp.lua` 文件，可以用 `require('config.nvim-cmp')`*

### 选项配置

主要用到的就是 `vim.g`、`vim.opt`、`vim.cmd` 等，我制造了一个快速参照对比的表格

| In `Vim`          | In `Nvim`                 | Note                             |
| ----------------- | ------------------------- | -------------------------------- |
| `let g:foo = bar` | `vim.g.foo = bar`         |                                  |
| `set foo = bar`   | `vim.opt.foo = bar`       | `set foo` = `vim.opt.foo = true` |
| `some_vimscript`  | `vim.cmd(some_vimscript)` |                                  |

### 按键配置

在 `Nvim` 里面进行按键绑定的语法如下，具体的解释可以看 `:h vim.keymap.set`

```lua
vim.keymap.set(<mode>, <key>, <action>, <opts>)
```

## 从零开始配置 Nvim

在阅读了前面一些配置基础之后，现在我们可以从头开始，由简到易一步步配置 `Nvim` 了


### 安装 Nvim

我用的是 Mac，用 Homebrew 安装 `Nvim` 非常容易，只要运行如下命令即可[^1]

```sh
$ brew install neovim 
```

在安装完成之后，如果 `~/.config/nvim` 目录不存在，创建目录并新建 `init.lua` 文件

```sh
$ mkdir ~/.config/nvim
$ mkdir ~/.config/nvim/lua
$ touch ~/.config/nvim/lua/init.lua
```

> 💡 **配置文件编辑保存之后，重启 `Nvim` 就能看到效果，后面默认每次小章节配置完成后就重启**

### 选项配置

选项配置功能一览：
- 默认采用系统剪贴板，同时支持鼠标操控 `Nvim`
- Tab 和空格的换算
- UI 界面
- “智能”搜索

新建 `~/.config/nvim/lua/options.lua` 文件并加入如下内容⬇️

```lua
-- Hint: use `:h <option>` to figure out the meaning if needed
vim.opt.clipboard = 'unnamedplus' -- use system clipboard
vim.opt.completeopt = { 'menu', 'menuone', 'noselect' }
vim.opt.mouse = 'a' -- allow the mouse to be used in Nvim

-- Tab
vim.opt.tabstop = 4 -- number of visual spaces per TAB
vim.opt.softtabstop = 4 -- number of spacesin tab when editing
vim.opt.shiftwidth = 4 -- insert 4 spaces on a tab
vim.opt.expandtab = true -- tabs are spaces, mainly because of python

-- UI config
vim.opt.number = true -- show absolute number
vim.opt.relativenumber = true -- add numbers to each line on the left side
vim.opt.cursorline = true -- highlight cursor line underneath the cursor horizontally
vim.opt.splitbelow = true -- open new vertical split bottom
vim.opt.splitright = true -- open new horizontal splits right
-- vim.opt.termguicolors = true        -- enabl 24-bit RGB color in the TUI
vim.opt.showmode = false -- we are experienced, wo don't need the "-- INSERT --" mode hint

-- Searching
vim.opt.incsearch = true -- search as characters are entered
vim.opt.hlsearch = false -- do not highlight matches
vim.opt.ignorecase = true -- ignore case in searches by default
vim.opt.smartcase = true -- but make it case sensitive if an uppercase is entered
```

然后打开 `init.lua`，用 `require` 导入刚才写的 `options.lua` 文件

```lua
require('options')        
```

### 按键配置

按键功能一览：
- 用 `<C-h/j/k/l>` 快速在多窗口之间移动光标
- 用 `Ctrl` + 方向键进行窗口大小的调整
- 选择模式下可以一直用 `Tab` 或者 `Shift-Tab` 改变缩进

新建 `~/.config/nvim/lua/keymaps.lua` 文件并放入如下内容⬇️

```lua
-- define common options
local opts = {
    noremap = true,      -- non-recursive
    silent = true,       -- do not show message
}

-----------------
-- Normal mode --
-----------------

-- Hint: see `:h vim.map.set()`
-- Better window navigation
vim.keymap.set('n', '<C-h>', '<C-w>h', opts)
vim.keymap.set('n', '<C-j>', '<C-w>j', opts)
vim.keymap.set('n', '<C-k>', '<C-w>k', opts)
vim.keymap.set('n', '<C-l>', '<C-w>l', opts)

-- Resize with arrows
-- delta: 2 lines
vim.keymap.set('n', '<C-Up>', ':resize -2<CR>', opts)
vim.keymap.set('n', '<C-Down>', ':resize +2<CR>', opts)
vim.keymap.set('n', '<C-Left>', ':vertical resize -2<CR>', opts)
vim.keymap.set('n', '<C-Right>', ':vertical resize +2<CR>', opts)

-----------------
-- Visual mode --
-----------------

-- Hint: start visual mode with the same area as the previous area and the same mode
vim.keymap.set('v', '<', '<gv', opts)
vim.keymap.set('v', '>', '>gv', opts)
```

然后在 `init.lua` 文件里面再次加上一行导入这个文件

```lua
...
require('keymaps')        
```

> 💡 `...` 表示我们省略了其他部分的代码（为了节省博客的篇幅）

### 安装插件管理器

一个强大的 `Nvim` 离不开插件的支持。我选用的是当下最为流行，而且完全用 Lua 语言编写的 [Packer.nvim](https://github.com/wbthomason/packer.nvim)。它支持如下许多特性：
- 支持配置第三方插件的依赖
- 支持 Lazy loading
- 支持设置插件安装之后的钩子函数
- ...

> 💡 **在 `Packer.nvim` 配置里面指定第三方插件很简单，用 `use ...` 即可**

新建 `~/.config/nvim/lua/plugins.lua` 文件并放入如下内容。下面的模板只完成了 Packer.vim 自身的安装，**还没有指定其他第三方插件**。这个模板的功能主要是
1. 初次启动的时候自动安装 `Packer.nvim`
2. 当我们保存对这个文件（`plugins.lua`）文件的修改的时候，Packer.nvim 会自动帮我们自动更新插件和帮我们做好配置。你可以**看到`Nvim` 右边看到多了一个窗口显示进度**

> 💡 Packer.nvim 还支持了不少命令，不过你不需要把他们都记住。因为这个模板会**自动帮我们处理好**。值得一提的是**如果因为网络问题安装失败**的话，在它弹出的窗口里面按照提示按下大写的 `R` 就会自动重新下载。在 Packer.nvim 提示全都安装成功后，**重启** `Nvim` 就生效了

```lua
-- Install Packer automatically if it's not installed(Bootstraping)
-- Hint: string concatenation is done by `..`
local ensure_packer = function()
    local fn = vim.fn
    local install_path = fn.stdpath('data') .. '/site/pack/packer/start/packer.nvim'
    if fn.empty(fn.glob(install_path)) > 0 then
        fn.system({ 'git', 'clone', '--depth', '1', 'https://github.com/wbthomason/packer.nvim', install_path })
        vim.cmd [[packadd packer.nvim]]
        return true
    end
    return false
end
local packer_bootstrap = ensure_packer()


-- Reload configurations if we modify plugins.lua
-- Hint
--     <afile> - replaced with the filename of the buffer being manipulated
vim.cmd([[
  augroup packer_user_config
    autocmd!
    autocmd BufWritePost plugins.lua source <afile> | PackerSync
  augroup end
]])


-- Install plugins here - `use ...`
-- Packer.nvim hints
--     after = string or list,           -- Specifies plugins to load before this plugin. See "sequencing" below
--     config = string or function,      -- Specifies code to run after this plugin is loaded
--     requires = string or list,        -- Specifies plugin dependencies. See "dependencies".
--     ft = string or list,              -- Specifies filetypes which load this plugin.
--     run = string, function, or table, -- Specify operations to be run after successful installs/updates of a plugin
return require('packer').startup(function(use)
        -- Packer can manage itself
        use 'wbthomason/packer.nvim'

        ---------------------------------------
        -- NOTE: PUT YOUR THIRD PLUGIN HERE --
        ---------------------------------------

        -- Automatically set up your configuration after cloning packer.nvim
        -- Put this at the end after all plugins
        if packer_bootstrap then
            require('packer').sync()
        end
    end)
```

然后在 `init.lua` 文件里面再次加上一行导入这个文件

```lua
...
require('plugins')        
```

此时你重启 `Nvim` 会发现黑屏没显示，这是因为 Packer.nvim 在安装自己，静待片刻即可☕️

### 主题配置

我喜欢的主题是 [monokai](https://github.com/tanvirtin/monokai.nvim)，在 `plugins.lua` 里面加上

```lua
...
use 'tanvirtin/monokai.nvim'
...
```

保存更改待 Packer.nvim 下载插件完成之后，新建并编辑 `~/.config/nvim/colorscheme.lua` 文件

```lua
-- define your colorscheme here
local colorscheme = 'monokai_pro'

local is_ok, _ = pcall(vim.cmd, "colorscheme " .. colorscheme)
if not is_ok then
    vim.notify('colorscheme ' .. colorscheme .. ' not found!')
    return
end
```

这里用到的 `pcall` 是 Lua 里面的 protected call，会返回一个 `bool` 变量表示是否执行成功（跟 Golang 的 `err` 功能类似）。这里采用 `pcall` 而不是直接在 `init.lua` 文件里面加上 `vim.cmd('colorscheme monokai_pro')` 是为了避免主题没有安装的话打开 `Nvim` 看到一大堆报错信息[^3]

最后在 `init.lua` 文件里面导入就行

```lua
...
require('colorscheme')
```

### 自动补全指南

自己手动配置自动补全功能的话会比较繁琐，所以我们最好是借助于一些插件来帮忙，下面我讲讲**我摸索出来的最简单配置方法**

首先，第一个要用到的插件是 [nvim-cmp](https://github.com/hrsh7th/nvim-cmp)，它可以管理各种补全候选项来源，然后展示在补全菜单里面，还支持我们对外观等进行定制化。

我们先新建 `~/.config/nvim/lua/config/nvim-cmp.lua` 文件配置 `nvim-cmp`

> 💡 这里首先选择写 `nvim-cmp` 的配置文件然后再在 `plugins.lua` 文件里面用 `use` 添加插件。这样可以保证 Packer.nvim 安装 nvim-cmp 的相关插件读取 `nvim-cmp.lua` 配置文件的时候不会报错。**下面的配置文件暂时看不懂也没有关系，我会对其进行解释**

```lua
local has_words_before = function()
    unpack = unpack or table.unpack
    local line, col = unpack(vim.api.nvim_win_get_cursor(0))
    return col ~= 0 and vim.api.nvim_buf_get_lines(0, line - 1, line, true)[1]:sub(col, col):match("%s") == nil
end

local luasnip = require("luasnip")
local cmp = require("cmp")

cmp.setup({
    snippet = {
        -- REQUIRED - you must specify a snippet engine
        expand = function(args)
            require('luasnip').lsp_expand(args.body) -- For `luasnip` users.
        end,
    },
    mapping = cmp.mapping.preset.insert({
        -- Use <C-b/f> to scroll the docs
        ['<C-b>'] = cmp.mapping.scroll_docs( -4),
        ['<C-f>'] = cmp.mapping.scroll_docs(4),
        -- Use <C-k/j> to switch in items
        ['<C-k>'] = cmp.mapping.select_prev_item(),
        ['<C-j>'] = cmp.mapping.select_next_item(),
        -- Use <CR>(Enter) to confirm selection
        -- Accept currently selected item. Set `select` to `false` to only confirm explicitly selected items.
        ['<CR>'] = cmp.mapping.confirm({ select = true }),

        -- A super tab
        -- sourc: https://github.com/hrsh7th/nvim-cmp/wiki/Example-mappings#luasnip
        ["<Tab>"] = cmp.mapping(function(fallback)
            -- Hint: if the completion menu is visible select next one
            if cmp.visible() then
                cmp.select_next_item()
            elseif luasnip.expand_or_locally_jump() then
                -- You could replace the expand_or_jumpable() calls with expand_or_locally_jumpable()
                -- they way you will only jump inside the snippet region
                luasnip.expand_or_jumpable()
            elseif has_words_before() then
                cmp.complete()
            else
                fallback()
            end
        end, { "i", "s" }), -- i - insert mode; s - select mode
        ["<S-Tab>"] = cmp.mapping(function(fallback)
            if cmp.visible() then
                cmp.select_prev_item()
            elseif luasnip.jumpable( -1) then
                luasnip.jump( -1)
            else
                fallback()
            end
        end, { "i", "s" }),
    }),

  -- Let's configure the item's appearance
  -- source: https://github.com/hrsh7th/nvim-cmp/wiki/Menu-Appearance
  formatting = {
      -- Set order from left to right
      -- kind: single letter indicating the type of completion
      -- abbr: abbreviation of "word"; when not empty it is used in the menu instead of "word"
      -- menu: extra text for the popup menu, displayed after "word" or "abbr"
      fields = { 'abbr', 'menu' },

      -- customize the appearance of the completion menu
      format = function(entry, vim_item)
          vim_item.menu = ({
              nvim_lsp = '[Lsp]',
              luasnip = '[Luasnip]',
              buffer = '[File]',
              path = '[Path]',
          })[entry.source.name]
          return vim_item
      end,
  },

  -- Set source precedence
  sources = cmp.config.sources({
    { name = 'nvim_lsp' },    -- For nvim-lsp
    { name = 'luasnip' },     -- For luasnip user
    { name = 'buffer' },      -- For buffer word completion
    { name = 'path' },        -- For path completion
  })
})
```

然后我们修改 `plugins.lua` 文件添加插件

```lua
...
use { 'neovim/nvim-lspconfig' }
use { 'hrsh7th/nvim-cmp', config = [[require('config.nvim-cmp')]] }
use { 'hrsh7th/cmp-nvim-lsp', after = 'nvim-cmp' }
use { 'hrsh7th/cmp-buffer', after = 'nvim-cmp' } -- buffer auto-completion
use { 'hrsh7th/cmp-path', after = 'nvim-cmp' } -- path auto-completion
use { 'hrsh7th/cmp-cmdline', after = 'nvim-cmp' } -- cmdline auto-completion
use 'L3MON4D3/LuaSnip'
use 'saadparwaiz1/cmp_luasnip'
...
```
**解释如下**
- `cmp.setup` 函数的参数是一个 Lua 的 Table，用于设置各个选项（下面会解释）。后面你会发现很多第三方插件都用 `setup` 传入一个 Lua table 的方式进行配置，**这个是 `Nvim` 的 Lua 插件的惯例**
- 不要被上面的这么多插件吓到，`nvim-cmp` 为主，其他 `cmp-...` 的插件是用于在候选项来源和 `nvim-cmp` 之间交互
- `LuaSnip` 是 code snippet 引擎，因为`nvim-cmp` 要求我们必须指定**至少一个** code snippet 引擎来源所以才加上，你暂时用不到当它不存在也没有关系
- Packer.nvim 支持用 `config = ...` 指定对应插件被加载之后要运行的代码，*所以 `config = [[require('config.nvim-cmp')]]` 的意思是导入了 `config` 文件夹里面的 `nvim-cmp.lua` 文件，这个设计参考了[^2]*

#### nvim-cmp 里面的按键映射

按键映射用的是 `mapping = ...` ，每个按键绑定的格式是 `['<key-binding>'] = cmp.mapping.xxx,`，不同的 `cmp.mapping.xxx` 的含义可以用 `:h` 查看。**如果你想要用其他的按键，只要修改 `[...]` 里面的按键即可**

按照我的个人习惯，设置了
1. `<C-k/j>` 在各种候选项里面移动
2. `<C-b/f>` 在候选项的文档里面移动
3. `<CR>` 也就是回车键确定补全
4. "超级 Tab 键"，允许我们按下 `Tab` 的时候在候选项之间、代码片段的占位符位置之间里面跳跃


#### nvim-cmp 里面的补全菜单

补全菜单的定制化用的是 `formatting = ...`
- `fields` 字段规定了每个候选项要显示什么东西
- `format = function(...)` 设置了不同的候选项的来源显示，在 `sources = ...` 里面声明来源

> 🎙️ 到这为止，重新启动 `Nvim` 后应该能够用初步的自动补全功能了～

#### LSP

要把 `Nvim` 变成 IDE 就势必要借助于 LSP[^4]，自己安装和配置 LSP 是比较繁琐的。不同的 LSP 安装方法不同，也不方便后续管理。这就是 [mason](https://github.com/williamboman/mason.nvim) 和配套的 [mason-lspconfig](https://github.com/williamboman/mason-lspconfig.nvim) 这两个插件大放异彩的地方🥰

> ❗️ 注意，`mason.nvim` 和 `mason-lspconfig.nvim` 和前面我们已经添加的 `nvim-lspconfig` **这三个插件之间的安装和配置是有顺序要求的**。推荐直接照搬我下面的就好了

首先修改 `plugins.lua` 文件，增加上面这两个插件

```lua
...
use { 'williamboman/mason.nvim' }
use { 'williamboman/mason-lspconfig.nvim'}
-- use { 'neovim/nvim-lspconfig' }            -- previous installed
...
```

新建一个 `~/.config/nvim/lua/lsp.lua` 文件并编辑，首先配置 `mason` 和 `mason-lspconfig`

```lua
require('mason').setup({
    ui = {
        icons = {
            package_installed = "✓",
            package_pending = "➜",
            package_uninstalled = "✗"
        }
    }
})

require('mason-lspconfig').setup({
    -- A list of servers to automatically install if they're not already installed
    ensure_installed = { 'pylsp', 'gopls', 'lua_ls', 'rust_analyzer' },
})
```

> 💡 **我们想要用什么语言的 LSP 就在 `ensure_installed` 里面加上**，完整的列表可以看 [server_configurations](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md)。*我个人常用的就 `python/go/rust` 这三个编程语言，而因为我们都用 Lua 语言来配置 `Nvim`，所以也加上了 `lua_ls`*

重启 `Nvim` 之后你应该可以在下面的状态栏看到 Mason 正在下载安装上面我们指定的 LSP（**注意此时不能关闭 `Nvim`**），可以输入 `:Mason` 查看安装进度

在成功安装 LSP 之后，我们就可以用 `nvim-lspconfig` 插件进行配置（因为配置的代码比较长，下面只展示了 `pylsp` 的配置，其他语言的配置大同小异），编辑 `~/.config/nvim/lua/lsp.lua` 文件新增如下内容

```lua
...
-- Set different settings for different languages' LSP
-- LSP list: https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md
-- How to use setup({}): https://github.com/neovim/nvim-lspconfig/wiki/Understanding-setup-%7B%7D
--     - the settings table is sent to the LSP
--     - on_attach: a lua callback function to run after LSP atteches to a given buffer
local lspconfig = require('lspconfig')

-- Customized on_attach function
-- See `:help vim.diagnostic.*` for documentation on any of the below functions
local opts = { noremap = true, silent = true }
vim.keymap.set('n', '<space>e', vim.diagnostic.open_float, opts)
vim.keymap.set('n', '[d', vim.diagnostic.goto_prev, opts)
vim.keymap.set('n', ']d', vim.diagnostic.goto_next, opts)
vim.keymap.set('n', '<space>q', vim.diagnostic.setloclist, opts)

-- Use an on_attach function to only map the following keys
-- after the language server attaches to the current buffer
local on_attach = function(client, bufnr)
    -- Enable completion triggered by <c-x><c-o>
    vim.api.nvim_buf_set_option(bufnr, 'omnifunc', 'v:lua.vim.lsp.omnifunc')

    -- See `:help vim.lsp.*` for documentation on any of the below functions
    local bufopts = { noremap = true, silent = true, buffer = bufnr }
    vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, bufopts)
    vim.keymap.set('n', 'gd', vim.lsp.buf.definition, bufopts)
    vim.keymap.set('n', 'K', vim.lsp.buf.hover, bufopts)
    vim.keymap.set('n', 'gi', vim.lsp.buf.implementation, bufopts)
    vim.keymap.set('n', '<C-k>', vim.lsp.buf.signature_help, bufopts)
    vim.keymap.set('n', '<space>wa', vim.lsp.buf.add_workspace_folder, bufopts)
    vim.keymap.set('n', '<space>wr', vim.lsp.buf.remove_workspace_folder, bufopts)
    vim.keymap.set('n', '<space>wl', function()
        print(vim.inspect(vim.lsp.buf.list_workspace_folders()))
    end, bufopts)
    vim.keymap.set('n', '<space>D', vim.lsp.buf.type_definition, bufopts)
    vim.keymap.set('n', '<space>rn', vim.lsp.buf.rename, bufopts)
    vim.keymap.set('n', '<space>ca', vim.lsp.buf.code_action, bufopts)
    vim.keymap.set('n', 'gr', vim.lsp.buf.references, bufopts)
    vim.keymap.set('n', '<space>f', function() vim.lsp.buf.format { async = true } end, bufopts)
end

lspconfig.pylsp.setup({
    on_attach = on_attach,
})
...
```

最后在 `init.lua` 文件里面加上

```lua
...
require('lsp')
```

上面的按键绑定的意思是很直观的，这里就不多解释

大功告成🎉🎉🎉

## 总结

这样配置下来，我们成功把 `Nvim` 变成了一个轻量级的 IDE，它支持代码高亮、代码补全、语法检查等功能，而且是**完全开源免费的**，虽然还有些简陋，但已经是可以用的了🥰

我发现自从学了 Vim 之后，我总在其他各种代码编辑器、IDE 看是不是支持 Vim。大多数情况下，他们对 Vim 的支持都不是很让人满意，还**容易有快捷键冲突等问题**。因此我选择将 `Nvim` 变成 IDE，并将配置文件托管在我的 [martinlwx/dotfiles](https://github.com/MartinLwx/dotfiles) 上。这样在新的机器上只要安装好 `Nvim` 并克隆配置静待片刻之后，就**可以在不同的机器上获得一样的编程体验**

打磨定制化工具是需要付出一定精力的。为了理解每个选项都在干啥，我不得不查找各种资料。但**我仍相信这是值得的**，**理解你的工具利于你做扩展和定制化**。本文已经尽可能采用了比较简单的配置，还有很多美化、私人定制化的内容可以配置，更别提其他很多优秀的第三方插件都还没有提及，这些就留给读者自己去探索发现了

## 参考 

[^1]: [Installing-Neovim](https://github.com/neovim/neovim/wiki/Installing-Neovim)
[^2]: [jdhao/nvim-config](https://github.com/jdhao/nvim-config)
[^3]: [Adding a colorscheme/theme](https://www.youtube.com/watch?v=Ow03haHO1NE&list=RDCMUCS97tchJDq17Qms3cux8wcA&index=4)
[^4]: [Language Server Protocol - Wiki](https://en.wikipedia.org/wiki/Language_Server_Protocol)

