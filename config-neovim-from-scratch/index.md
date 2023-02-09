# Transform Your Neovim into a IDE: A Step-by-Step Guide


## Versions info

I use a Macbook pro-2020 Intel Edition with macOS 13.2. This is my `Nvim` edition:

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

## Why Neovim

After using `Vim` for one year, I find myself having trouble in configure `~/.vimrc`. The syntax of Vimscript is not my liking, leading me to switch `Neovim(Nvim)`. **Rather than** migrating my old `~/.vimrc`. I decided to start from scratch and take this opportunity to re-evaluate my previous Vim configuration. I aim to replace my plugins with the latest SOTA(State-of-the-art) alternatives. ~~It's been some time since I last edited my `~/.vimrc`~~

In my opinion, it's essential to understand the meaning behind each option and statement in the configuration file. That's the approach I took in this post. My goal is to make the configuration files self-contained and easily understandable. To achieve this, I aim to provide clear explanations for each setting and include comments to enhance readability.

> 💡 Please note that I may have missed some options. However, as a reminder, you can always access the help docs in the `Nvim` by typing `:h <name>` to get more information

> 💡 This post **assumes** that you have a basic understanding of Vim

## The basics

### Lua 

In my `Nvim` configuration, I will use the Lua programming language as much as possible. Thus, it's recommended that the reader familiarize themselves with Lua. Take a look at [Learn Lua in Y minutes](https://learnxinyminutes.com/docs/lua/)

### Configurations files paths

The configuration directory for `Nvim` is located at `~/.config/nvim`. On Linux/Mac, `Nvim` will read `~/.config/nvim/init.lua` when it starts up. **Theoretically**, we can put everything inside this single file. It's a bad practice though. To keep things organized, I prefer to break it down into smaller, more manageable parts.

If you follow this post to configure your `Nvim`, your `~/.config/nvim` should look like this⬇️

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

**The explanations**
- `init.lua` is the entry point. **We will "import" other `*.lua` files in `init.lua`**
    - `colorscheme.lua` for the theme
    - `keymaps.lua` for key mappings
    - `lsp.lua` for the LSP support
    - `options.lua` for some global options
    - `plugins.lua` for third-party plugins
- Put the configurations of third-party plugins in this `config` folder. *For example, `nvim-cmp.lua` for the `nvim-cmp` plugin*
- `lua` folder. When we call `require` to import a module in Lua, it will search this folder.
    - **Replace the path separator `/` with `.`, and remove the suffix - `.lua`**. That's how you get the parameter of `require`
    - *For example, to import `nvim-cmp.lua`, you should write `require('config.nvim-cmp')`*

### Options

We mainly use these: `vim.g`, `vim.opt`, and `vim.cmd`. I made a cheatsheet below:

| In `Vim`          | In `Nvim`                 | Note                             |
| ----------------- | ------------------------- | -------------------------------- |
| `let g:foo = bar` | `vim.g.foo = bar`         |                                  |
| `set foo = bar`   | `vim.opt.foo = bar`       | `set foo` = `vim.opt.foo = true` |
| `some_vimscript`  | `vim.cmd(some_vimscript)` |                                  |

### key mappings

The syntax of key binding in `Nvim`:

```lua
vim.keymap.set(<mode>, <key>, <action>, <opts>)
```

For a detailed explanation, please refer to `:h vim.keymap.set`

## Configure Nvim from scratch

Now we can configure `Nvim` step by step :)

### Install Neovim

I am a Mac user, so I use Homebrew to install `Nvim`[^1]

```sh
$ brew install neovim 
```

After completing the installation, If the `~/.config/nvim/` directory doesn't exist, you should create the folder and `init.lua` file

```sh
$ mkdir ~/.config/nvim
$ mkdir ~/.config/nvim/lua
$ touch ~/.config/nvim/lua/init.lua
```

> 💡 **Please note that after making any modifications to the `*.lua` files, you need to restart the `Nvim` to see the changes take effect. I will assume that you restart your `Nvim` after each section**

### Options configuration

The features:
- Use the system's clipboard 
- Use the mouse in `Nvim`
- Tab and whitespace
- UI configuration
- *Smart* search

Create `~/.config/nvim/lua/options.lua` file and edit:


```lua
-- Hint: use `:h <option>` to figure out the meaning if needed
vim.opt.clipboard = 'unnamedplus'   -- use system clipboard 
vim.opt.completeopt = {'menu', 'menuone', 'noselect'}
vim.opt.mouse = 'a'                 -- allow the mouse to be used in Nvim

-- Tab
vim.opt.tabstop = 4                 -- number of visual spaces per TAB
vim.opt.softtabstop = 4             -- number of spacesin tab when editing
vim.opt.shiftwidth = 4              -- insert 4 spaces on a tab
vim.opt.expandtab = true            -- tabs are spaces, mainly because of python

-- UI config
vim.opt.number = true               -- show absolute number
vim.opt.relativenumber = true       -- add numbers to each line on the left side
vim.opt.cursorline = true           -- highlight cursor line underneath the cursor horizontally
vim.opt.splitbelow = true           -- open new vertical split bottom
vim.opt.splitright = true           -- open new horizontal splits right
-- vim.opt.termguicolors = true        -- enabl 24-bit RGB color in the TUI
vim.opt.showmode = false            -- we are experienced, wo don't need the "-- INSERT --" mode hint

-- Searching
vim.opt.incsearch = true            -- search as characters are entered
vim.opt.hlsearch = false            -- do not highlight matches
vim.opt.ignorecase = true           -- ignore case in searches by default
vim.opt.smartcase = true            -- but make it case sensitive if an uppercase is entered
```

Then edit the `init.lua` file, use `require` to import `options.lua` file

```lua
require('options')        
```

### Key mappings configuration

The features:
- Use `<C-h/j/k/l>` to move the cursor among windows
- Use `Ctrl` + arrow keys to resize windows
- In select mode, we can use `Tab` or `Shift-Tab` to change the indentation repeatedly

Create `~/.config/nvim/lua/keymaps.lua` and edit:

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

Edit `init.lua` and import `keymaps.lua`

```lua
...
require('keymaps')        
```

> 💡 `...` means that we omit other lines(in order to save the length of the post)

### Install package manager

A powerful `Nvim` should be augmented with third-party plugins. I have selected [Packer.nvim](https://github.com/wbthomason/packer.nvim) as my plugin manager, which has several amazing features including:
- Support for dependencies
- Expressive configuration and lazy-loading options
- Post-install/update hooks
- ...

> 💡 **The syntax of adding a third-party package is `use ...`**

Create `~/.config/nvim/lua/plugins.lua` and paste the following code. At the moment, I **haven't added any third-party packages**. The template code will do these for us:
1. Install `Packer.nvim` if not installed
2. After modifying the `plugins.lua` file and saving it, Packer.nvim will automatically update and configure the plugins. You should see **a popped window on the right side of the `Nvim`** indicating the status of the plugin updates.


> 💡 You do not need to memorize all the commands available in Packer.nvim, as the template will handle the majority of the work for you. It's worth mentioning that if you failed to update and configure because of the network issue, you can press `<R>` in the popped window to re-sync. Once the Packer.nvim syncs successfully, you can **restart** your `Nvim` to see the changes.


```lua
-- Install Packer automatically if it's not installed(Bootstraping)
-- Hint: string concatenation is done by `..`
local ensure_packer = function()
    local fn = vim.fn
    local install_path = fn.stdpath('data')..'/site/pack/packer/start/packer.nvim'
    if fn.empty(fn.glob(install_path)) > 0 then
        fn.system({'git', 'clone', '--depth', '1', 'https://github.com/wbthomason/packer.nvim', install_path})
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

Again, import `plugins.lua` in `init.lua`

```lua
...
require('plugins')        
```

If you see a black window with no content when opening `Nvim`, just wait for a moment as Packer.nvim is in the process of installing itself☕️


### Colorscheme

My favorite theme - [monokai](https://github.com/tanvirtin/monokai.nvim). Add this plugin in `plugins.lua`

```lua
...
use 'tanvirtin/monokai.nvim'
...
```

Save the changes and wait for Packer.nvim to finish installing. Create `~/.config/nvim/colorscheme.lua` and edit:

```lua
-- define your colorscheme here
local colorscheme = 'monokai_pro'

local is_ok, _ = pcall(vim.cmd, "colorscheme " .. colorscheme)
if not is_ok then
    vim.notify('colorscheme ' .. colorscheme .. ' not found!')
    return
end
```

The `pcall` here refers to a protected call in Lua, which will return a boolean value to indicate its successful execution(a similar approach can be found in Go with the use of `err`). By using `pcall` instead of `vim.cmd('colorscheme monokai_pro')`, we can avoid some annoying error messages in case the colorscheme is not installed[^3]

Again, import `colorscheme.lua` in `init.lua`

```lua
...
require('colorscheme')
```

### Auto-completion 

It can be quite complicated to configure auto-completion manually, which is why we use some fantastic plugins to ease the burden. Now I will discuss **a simpler solution I have found**.

First, use this plugin [nvim-cmp](https://github.com/hrsh7th/nvim-cmp), which can manage many completion sources for us. It can also let us customize the completion menu etc.

Create `~/.config/nvim/lua/config/nvim-cmp.lua` and edit

> 💡 Let's first write the configurations of `nvim-cmp` and then modify the `plugins.lua` file. It assures we won't get an annoying error message when the `nvim-cmp` tries to read the missing `nvim-cmp.lua` file. **The code below may seem a little complicated. Don't worry, I will show you how it works.**


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
            elseif luasnip.expand_or_locally_jumpable() then
                -- You could replace the expand_or_jumpable() calls with expand_or_locally_jumpable()
                -- they way you will only jump inside the snippet region
                luasnip.expand_or_jump()
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

Then we modify `plugins.lua` file to add the plugins needed:

```lua
...
use { 'neovim/nvim-lspconfig' }
use { 'hrsh7th/nvim-cmp', config = [[require('config.nvim-cmp')]] }    
use { 'hrsh7th/cmp-nvim-lsp', after = 'nvim-cmp' } 
use { 'hrsh7th/cmp-buffer', after = 'nvim-cmp' }        -- buffer auto-completion
use { 'hrsh7th/cmp-path', after = 'nvim-cmp' }          -- path auto-completion
use { 'hrsh7th/cmp-cmdline', after = 'nvim-cmp' }       -- cmdline auto-completion
use 'L3MON4D3/LuaSnip'
use 'saadparwaiz1/cmp_luasnip'
...
```

**Explanations**:
- `cmp.setup` function accepts a Lua table, which defines some options for customization. You will find that plenty of plugins follow this API design. **It's a common practice.**
- The `nvim-cmp` is the main plugin we care about. All other plugins begin with `cmp-` is the completion sources helper used by `nvim-cmp`
- `LuaSnip` is a code snippet engine. The `nvim-cmp` says that we should pick a code snippet engine **at least**. Just ignore this if you don't need this
- We can use `config = ...` in Packer.nvim to specify the code to run after the plugin is loaded. *So `config = [[require('config.nvim-cmp')]]` will execute the `nvim-cmp.lua` file. I found this idea on[^2]*

#### Key mappings in nvim-cmp

Use `mapping = ...`. The syntax is `['<key-binding>'] = cmp.mapping.xxx,`. Different `cmp.mapping.xxx` options can be found in the manual. **If you want to set a different key-binding, just change the `[...]`**

My key mappings:
1. Use `<C-k/j>` to move among completion items
2. Use `<C-b/f>` to scroll among the doc of completion item
3. Use `<CR>` to confirm completion
4. "A super tab", we can press `Tab` and move among the completion items or placeholders in a code snippet


#### Completion menu in nvim-cmp

Use `formatting = ...`:
- Use `fields` to specify the appearance of each completion item
- Use `format = function(...)` to set the text for each completion source. You can specify completion sources in `sources = ...`

> 🎙️ You can use basic completion now ~

#### LSP

To turn `Nvim` into an IDE, it is necessary to rely on LSP[^4]. It is cumbersome to install and configure LSP one by one manually, as different LSPs have different installation steps, and it is inconvenient for future management. That's where tools like [mason](https://github.com/williamboman/mason.nvim) and [mason-lspconfig](https://github .com/williamboman/mason-lspconfig.nvim) come in to make our lives easier🥰

> ❗️ Note that the order of `mason.nvim`, `mason-lspconfig.nvim` and the `nvim-lspconfig` is crucial. **There is a specific ordering requirement between these three plugins and their configurations**. So it's recommended to follow the code provided


Modify the `plugins.lua` file:

```lua
...
use { 'williamboman/mason.nvim' }
use { 'williamboman/mason-lspconfig.nvim'}
-- use { 'neovim/nvim-lspconfig' }            -- previous installed
...
```

Create a `~/.config/nvim/lua/lsp.lua` file to manage it. Let's configure `mason` and `mason-lspconfig` first

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
    ensure_installed = { 'pylsp', 'gopls', 'sumneko_lua', 'rust_analyzer' },
})
```

> 💡 **Add whatever LSP you like in the `ensure_installed`**, the complete list can be found in [server_configurations](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations. md). *I personally use the three programming languages `python/go/rust`, and because we use Lua to configure `Nvim`, we also added `sumneko_lua` here*

After restarting `Nvim`, you should be able to see in the status bar below that Mason is installing the LSP we specified above (**Note that `Nvim` cannot be closed at this time**). By typing `:Mason` in `Nvim` we can check the installation progress


After successfully installing LSP, we should use `nvim-lspconfig` plug-in to configure (because the configuration code is relatively long, the configuration of `pylsp` is only shown below, and the configuration of other languages is similar). Add the following code to the `nvim/lua/lsp.lua` file


```lua
...
-- Set different settings for different languages' LSP
-- LSP list: https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md
-- How to use setup({}): https://github.com/neovim/nvim-lspconfig/wiki/Understanding-setup-%7B%7D
--     - the settings table is sent to the LSP
--     - on_attach: a lua callback function to run after LSP attaches to a given buffer
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

Append a line in `init.lua`

```lua
...
require('lsp')
```

The key-binding here is quite similar to what we did in `nvim-cmp`. Refer to the manual as you wish.

Now we got a lightweight IDE🎉🎉🎉

## Wrapping up

With this configuration, we successfully turned `Nvim` into a lightweight IDE, which supports code highlighting, code completion, syntax checking, and other functionalities. And it is **completely open source and free**🥰

I realized that even after trying different code editors and IDEs, I always found myself searching for Vim support. So I chose to turn `Nvim` into an IDE, and host the configuration files on my [martinlwx/dotfiles](https://github.com/MartinLwx/dotfiles). In this way, I can easily clone my configuration files to any new machine and **have a consistent programming experience across machines.**

Polishing tools requires effort and time. In order to understand the purpose of each option, I had to search for various materials. However, despite the challenges, **I firmly believe that it's worth it**. Understanding your tools allows you to further extend and customize them. This article has aimed to present a simple and straightforward configuration, but there are still many beautification and customization things that can be done, including many excellent third-party plug-ins that have not been mentioned yet. The exploration and discovery are left to the readers


## Refs

[^1]: [Installing-Neovim](https://github.com/neovim/neovim/wiki/Installing-Neovim)
[^2]: [jdhao/nvim-config](https://github.com/jdhao/nvim-config)
[^3]: [Adding a colorscheme/theme](https://www.youtube.com/watch?v=Ow03haHO1NE&list=RDCMUCS97tchJDq17Qms3cux8wcA&index=4)
[^4]: [Language Server Protocol - Wiki](https://en.wikipedia.org/wiki/Language_Server_Protocol)

