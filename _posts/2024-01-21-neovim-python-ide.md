---
title: Turn Neovim into a modern Python IDE with Ruff and Pyright
date: 2024-01-21 00:00:01 +0000
categories: [neovim, python]
tags: [programming, ide, python, neovim, nvim, ruff]
img_path: /assets/img/posts/
---

Learn how to transform [Neovim](https://neovim.io) into a powerful Python IDE using Ruff for fast linting and Pyright for type checking.

## Update!

Dear reader, this post has become outdated over time.
I've found a better way to convert [Neovim](https://neovim.io/) into a multi-language IDE in a much simpler and more modular way, which makes maintenance easier and reduces the time spent on maintenance and configuration.
I've also added more and better plugins that will increase your productivity, so I recommend you go directly to that article: [How to build a modern Neovim IDE in 2026 A modular Lua guide](https://rubenhortas.github.io/posts/moden-neovim-ide-lua-guide-2026/#basic-ide-configuration)

## Why?

I have to admit it, [Neovim](https://neovim.io) is my editor for everything.
I started using [Vim](https://www.vim.org) in college, and we have been together since then, but, for a while now, now as its fork [Neovim](https://neovim.io).
Need to edit a file? [Neovim](https://neovim.io).
Need to do a bash script? [Neovim](https://neovim.io).
Need to do a little python script? [Neovim](https://neovim.io).

[Vim](https://www.vim.org) and [Neovim](https://neovim.io) are very lightweight and very powerful.
[Vim](https://www.vim.org) comes installed in (almost) every linux distro, and they are very convenient to use via ssh.

Although for medium or large python projects my favorite IDE is [Pycharm](https://www.jetbrains.com/pycharm/) (I really love [Pycharm](https://www.jetbrains.com/pycharm/)), for small and fast (or not-so-fast scripts) I preffer neovim or vim, depends on which one is installed.

The point is that, while I was coding some python scripts, especially using new libraries, I missed some features provided by a IDE.
Features I was used to in [Pycharm](https://www.jetbrains.com/pycharm/), as autocomplete and linting (analyzing source code to flag programming errors, bugs, stylistic errors, etc.).
So, I decided it was time to configure [Neovim](https://neovim.io) to improve my python experience.

## Neovim base configuration

This is my universal configuration that I use across all my machines, regardless of the task.
This configuration will be the base to which I'll add the python configuration.

`~/.config/nvim/init.vim`:

```vim
" ==============================================================================
" CORE FUNCTIONALITY AND FILETYPE
" ==============================================================================
syntax on                               " Enable syntax highlighting
filetype plugin indent on               " Enable filetype detection, plugins, and smart indentation
set encoding=utf-8                      " Set file and terminal encoding to UTF-8

" ==============================================================================
" INDENTATION AND TABS (Using 4-space soft tabs)
" ==============================================================================
set autoindent                          " Copy indentation from the previous line
set smartindent                         " Enable smarter automatic indentation
set expandtab                           " Use spaces instead of actual tabs
set tabstop=4                           " A Tab character is rendered as 4 spaces wide
set shiftwidth=4                        " Auto-indent commands (e.g., >>) use 4 spaces
set softtabstop=4                       " Tab/Backtab keys use 4 spaces when inserting

" Forzar 4 espacios específicamente en C/C++ para evitar overrides
augroup FileTypeSettings
    autocmd!
    autocmd FileType c,cpp setlocal tabstop=4 shiftwidth=4 softtabstop=4
augroup END

" ==============================================================================
" UI AND APPEARANCE
" ==============================================================================
set number                              " Show absolute line number
set showmatch                           " Briefly show the matching bracket/parenthesis
set wildmenu                            " Enhanced command-line completion menu
set mouse=a                             " Enable mouse support
set updatetime=250                      " Sets the delay (ms) for showing diagnostics and tooltips (important for LSP)

" ==============================================================================
" SEARCH
" ==============================================================================
set path+=** " Allow searching for files recursively (e.g., :find filename)
set incsearch                           " Show results as you type the search pattern (incremental search)
set hlsearch                            " Highlight all matches of the last search pattern
set ignorecase                          " Ignore case when searching
set smartcase                           " Override ignorecase if the search pattern contains uppercase letters

" ==============================================================================
" BEHAVIOR AND SYSTEM INTEGRATION
" ==============================================================================
set backspace=indent,eol,start          " Ensures backspace works as expected
set clipboard=unnamedplus               " Integrate with system clipboard for yank/put (requires external tool like xclip/wl-copy)
set noswapfile                          " Disable swap files to prevent clutter
set undofile                            " Enable persistent undo history

" Specify a directory for undo files and create it if it doesn't exist
let s:undo_dir = expand('~/.config/nvim/undodir')
if !isdirectory(s:undo_dir)
    call mkdir(s:undo_dir, "p")
endif
let &undodir = s:undo_dir

" ==============================================================================
" LEADER KEY
" ==============================================================================
let mapleader = " "                     " Sets the Leader key to <Space> (used for custom keybinds like <leader>ca)
```

In my base configuration I usually add the [Vim Better Whitespace Plugin](https://github.com/ntpeters/vim-better-whitespace) (I'll talk about it later), but I keep the option "list" handy (but commented).
The "list" option, by default, show tabs as ">", trailing spaces as "-" and non-breakable space characters as "+".
This default configuration works for me, but can be customized.

## pyright

Pyright is a static type checker for Python created by Microsoft.
Pyright goes beyond just checking for syntax errors.
Its primary goal is to catch type-related errors in your Python code before you even run it.

Pyright helps us write more maintainable, understandable, and bug-free Python code by leveraging static type checking.

`pipx install pyright`

## Language Server Protocol (LSP): Ruff

In order to use [nvim-cmp](https://github.com/hrsh7th/nvim-cmp) and [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig), we need to install a Language Server Protocol (LSP), in this case, instead of using the heavier `python-lsp-server`, we will use [Ruff](https://docs.astral.sh/ruff/). 
[Ruff](https://docs.astral.sh/ruff/) is an extremely fast Python linter and code formatter, written in Rust. 
It can replace dozens of individual tools (like `Flake8`, `isort`, and `Black`) and provides its own LSP.

`pipx install ruff`

Add this to the end of your `~/.config/nvim/init.vim`:

```lua
lua <<EOF
  local lspconfig = require('lspconfig')
  local capabilities = require('cmp_nvim_lsp').default_capabilities()

  -- --- LSP UTILITY FUNCTION: ON_ATTACH ---
  local on_attach = function(client, bufnr)
    local opts = { noremap=true, silent=true, buffer=bufnr }

    -- Navigation and Information
    vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)      -- Go To Definition
    vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)            -- Show Hover Documentation
    vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)      -- List References
    vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, opts)    -- Go To Declaration

    -- Actions
    vim.keymap.set('n', '<leader>ca', vim.lsp.buf.code_action, opts) -- Code Actions
    vim.keymap.set('n', '<leader>rn', vim.lsp.buf.rename, opts)      -- Rename Symbol
    vim.keymap.set('n', '<leader>cf', function() vim.lsp.buf.format { async = true } end, opts) -- Format Code

    -- AUTO-FORMAT ON SAVE
    if client.server_capabilities.documentFormattingProvider then
      vim.api.nvim_create_autocmd("BufWritePre", {
        buffer = bufnr,
        callback = function()
          if client.name == "ruff" then
            vim.lsp.buf.code_action({
              context = { only = { "source.organizeImports" } },
              apply = true,
            })
            vim.wait(50)
          end

          vim.lsp.buf.format({ bufnr = bufnr, async = false })
        end,
      })
    end

    -- Diagnostics Configuration
    vim.diagnostic.config({
        virtual_text = true,
        signs = true,
        update_in_insert = false,
        float = { border = "rounded" },
    })
  end

  -- ==================================================
  -- LSP SERVER SETUP
  -- ==================================================

  -- Setup RUFF
  lspconfig.ruff.setup {
    on_attach = on_attach,
    capabilities = capabilities,
  }

  -- Setup PYRIGHT
  lspconfig.pyright.setup {
    on_attach = function(client, bufnr)
      client.server_capabilities.documentFormattingProvider = false
      on_attach(client, bufnr)
    end,
    capabilities = capabilities,
    settings = {
      pyright = {
        disableOrganizeImports = true,
      },
      python = {
        analysis = {
          typeCheckingMode = "strict", -- or "basic"
          autoSearchPaths = true,
          useLibraryCodeForTypes = true,
        }
      }
    }
  }
EOF
```

## Neovim plugin manager

As plugin manager, my choice is [vim-plug](https://github.com/junegunn/vim-plug#neovim), and its installation is very straightforward:

Unix/linux:

`sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'`

## Plugins

I install three plugins:

* [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)
  Configs for the Nvim LSP client.

* [nvim-cmp](https://github.com/hrsh7th/nvim-cmp)
  A completion engine plugin.

* [Vim Better Whitespace](https://github.com/ntpeters/vim-better-whitespace)
  This plugin highlights all trailing whitespaces.
  It's completely optional and can be replaced by the option "list" in the `vim.init` file, but I like it.

We need to create the directory where we will install the plugins.
I install my plugins in `~/.config/nvim/plugins`, so:

`mkdir `~/.config/nvim/plugins``

### nvim-lspconfig

To install it, we add the plugin to our `init.vim` file, into the `call plug#begin('~/.config/nvim/plugins')` section, below all the lines:

```vim
call plug#begin('~/.config/nvim/plugins')
...

Plug 'neovim/nvim-lspconfig'

call plug#end()
```

### nvim-cmp

A completion engine plugin for neovim written in Lua.
Completion sources are installed from external repositories and "sourced".
To install it we need to add the following to our `init.vim` file (respecting the blocks, if exists):

```vim
call plug#begin('~/.config/nvim/plugins')

...

Plug 'hrsh7th/cmp-nvim-lsp'
Plug 'hrsh7th/cmp-buffer'
Plug 'hrsh7th/cmp-path'
Plug 'hrsh7th/cmp-cmdline'
Plug 'hrsh7th/nvim-cmp'

" For vsnip users.
Plug 'hrsh7th/cmp-vsnip'
Plug 'hrsh7th/vim-vsnip'

call plug#end()
```

```lua
lua <<EOF
  -- ==============================================================================
  -- AUTOCOMPLETION CONFIGURATION: nvim-cmp
  -- ==============================================================================
  local cmp = require'cmp'

  cmp.setup({
    snippet = {
      -- REQUIRED - The expand function for vsnip
      expand = function(args)
        vim.fn["vsnip#anonymous"](args.body) -- For `vsnip` users.
      end,
    },
    mapping = cmp.mapping.preset.insert({
      ['<C-b>'] = cmp.mapping.scroll_docs(-4),
      ['<C-f>'] = cmp.mapping.scroll_docs(4),
      ['<C-Space>'] = cmp.mapping.complete(), -- Trigger completion menu
      ['<C-e>'] = cmp.mapping.abort(),
      ['<CR>'] = cmp.mapping.confirm({ select = true }), -- Accept selected item
    }),
    sources = cmp.config.sources({
      { name = 'nvim_lsp' },  -- Suggestions from LSP servers (clangd, rust_analyzer, etc.)
      { name = 'vsnip' },     -- Suggestions from the vsnip snippet engine
    }, {
      { name = 'buffer' },    -- Suggestions from the current file buffer
    })
  })

  -- Command-line mode completion setup (uses path and cmdline sources)
  cmp.setup.cmdline(':', {
    mapping = cmp.mapping.preset.cmdline(),
    sources = cmp.config.sources({
      { name = 'path' }
    }, {
      { name = 'cmdline' }
    })
  })

  -- Search mode completion setup (uses buffer source)
  cmp.setup.cmdline({ '/', '?' }, {
    mapping = cmp.mapping.preset.cmdline(),
    sources = {
      { name = 'buffer' }
    }
  })
EOF
```

### Configuring Ruff

Ruff follows a specific hierarchy to resolve settings. It searches for a configuration file in the current directory and continues up the parent directories until it finds one.

#### Global Configuration (User-wide)

To apply rules to every Python project on your machine (as a fallback when no project-specific file exists), use the following directory: `~/.config/ruff/` with one of the following filenames:

  * `settings.toml` (Recommended)
  * `ruff.toml`
  * `.ruff.toml`

>Do not use the [tool.ruff] header. Write properties directly at the root of the file.
{: .prompt-info }

#### Project Configuration

To apply rules to a specific project, place the file in the project's root directory:

  * `pyproject.toml` (must use [tool.ruff] header and sections)
  * `ruff.toml` or `.ruff.toml` (direct properties, no headers)

#### Folder-Specific Configuration (Nested)

To override or extend rules for a specific subdirectory (e.g., allowing assert only in /tests or changing line lengths in /scripts), place a file inside that folder.
   * `ruff.toml` or `.ruff.toml`

A `pyproject.toml` example:


```toml
[tool.ruff]
line-length = 120

[tool.ruff.lint]
# "E" (pycodestyle), "F" (Pyflakes), "I" (isort)
select = ["E", "F", "I"]
ignore = []

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

>This is [my pyproject.toml file](https://gist.github.com/rubenhortas/57febeace65b746bda535f5f4c5f087a)
{: .prompt-info }


### Vim Better Whitespace

This plugin causes all trailing whitespace characters to be highlighted.
Whitespace for the current line will not be highlighted while in insert mode.
It is possible to disable current line highlighting while in other modes as well.
A helper function `:StripWhitespace` is also provided to make whitespace cleaning painless.

This plugin is optional, and can be replaced by the "lines" option in the `vim.init` file.
But, I like it, and the `:StripWhitespace` function is very useful.

To install it we add the plugin to our `init.vim` file, into the `call plug#begin('~/.config/nvim/plugins')` section, below all the lines:

```vim
call plug#begin('~/.config/nvim/plugins')
...

Plug 'ntpeters/vim-better-whitespace'

call plug#end()
```

## Installing the plugins

To install the plugins we open nvim and run:

`:PlugInstall`

And, that's all. Now you can start using [Neovim](https://neovim.io) as your Python IDE!

## Unified full init.vim file:

```vim
" ==============================================================================
" CORE FUNCTIONALITY AND FILETYPE
" ==============================================================================
syntax on                               " Enable syntax highlighting
filetype plugin indent on               " Enable filetype detection, plugins, and smart indentation
set encoding=utf-8                      " Set file and terminal encoding to UTF-8

" ==============================================================================
" INDENTATION AND TABS (Using 4-space soft tabs)
" ==============================================================================
set autoindent                          " Copy indentation from the previous line
set smartindent                         " Enable smarter automatic indentation
set expandtab                           " Use spaces instead of actual tabs
set tabstop=4                           " A Tab character is rendered as 4 spaces wide
set shiftwidth=4                        " Auto-indent commands (e.g., >>) use 4 spaces
set softtabstop=4                       " Tab/Backtab keys use 4 spaces when inserting

" Forzar 4 espacios específicamente en C/C++ para evitar overrides
augroup FileTypeSettings
    autocmd!
    autocmd FileType c,cpp setlocal tabstop=4 shiftwidth=4 softtabstop=4
augroup END

" ==============================================================================
" UI AND APPEARANCE
" ==============================================================================
set number                              " Show absolute line number
set showmatch                           " Briefly show the matching bracket/parenthesis
set wildmenu                            " Enhanced command-line completion menu
set mouse=a                             " Enable mouse support
set updatetime=250                      " Sets the delay (ms) for showing diagnostics and tooltips (important for LSP)

" ==============================================================================
" SEARCH
" ==============================================================================
set path+=** " Allow searching for files recursively (e.g., :find filename)
set incsearch                           " Show results as you type the search pattern (incremental search)
set hlsearch                            " Highlight all matches of the last search pattern
set ignorecase                          " Ignore case when searching
set smartcase                           " Override ignorecase if the search pattern contains uppercase letters

" ==============================================================================
" BEHAVIOR AND SYSTEM INTEGRATION
" ==============================================================================
set backspace=indent,eol,start          " Ensures backspace works as expected
set clipboard=unnamedplus               " Integrate with system clipboard for yank/put (requires external tool like xclip/wl-copy)
set noswapfile                          " Disable swap files to prevent clutter
set undofile                            " Enable persistent undo history

" Specify a directory for undo files and create it if it doesn't exist
let s:undo_dir = expand('~/.config/nvim/undodir')
if !isdirectory(s:undo_dir)
    call mkdir(s:undo_dir, "p")
endif
let &undodir = s:undo_dir

" ==============================================================================
" LEADER KEY
" ==============================================================================
let mapleader = " "                      " Sets the Leader key to <Space> (used for custom keybinds like <leader>ca)

" ==============================================================================
" PLUGIN MANAGEMENT (vim-plug)
" ==============================================================================
call plug#begin('~/.config/nvim/plugins')

" LSP Configuration
Plug 'neovim/nvim-lspconfig'            " Configs for the Nvim LSP client

" Completion Engine
Plug 'hrsh7th/cmp-nvim-lsp'             " LSP source for nvim-cmp
Plug 'hrsh7th/cmp-buffer'               " Buffer source for nvim-cmp
Plug 'hrsh7th/cmp-path'                 " Path source for nvim-cmp
Plug 'hrsh7th/cmp-cmdline'              " Cmdline source for nvim-cmp
Plug 'hrsh7th/nvim-cmp'                 " The completion engine plugin

" Snippets (Required for nvim-cmp)
Plug 'hrsh7th/cmp-vsnip'
Plug 'hrsh7th/vim-vsnip'

" Utilities
Plug 'ntpeters/vim-better-whitespace'   " This plugin highlights all trailing whitespaces

call plug#end()

" ==============================================================================
" LUA CONFIGURATION (LSP, AUTO-FORMAT, AND AUTOCOMPLETE)
" ==============================================================================
lua <<EOF
  local lspconfig = require('lspconfig')
  local cmp = require('cmp')
  local capabilities = require('cmp_nvim_lsp').default_capabilities()

  -- --- LSP UTILITY FUNCTION: ON_ATTACH ---
  local on_attach = function(client, bufnr)
    local opts = { noremap=true, silent=true, buffer=bufnr }

    -- Navigation and Information
    vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)      -- Go To Definition
    vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)            -- Show Hover Documentation
    vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)      -- List References
    vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, opts)    -- Go To Declaration

    -- Actions
    vim.keymap.set('n', '<leader>ca', vim.lsp.buf.code_action, opts) -- Code Actions
    vim.keymap.set('n', '<leader>rn', vim.lsp.buf.rename, opts)      -- Rename Symbol
    vim.keymap.set('n', '<leader>cf', function() vim.lsp.buf.format { async = true } end, opts) -- Format Code

    -- AUTO-FORMAT ON SAVE
    if client.server_capabilities.documentFormattingProvider then
      vim.api.nvim_create_autocmd("BufWritePre", {
        buffer = bufnr,
        callback = function()
          if client.name == "ruff" then
            vim.lsp.buf.code_action({
              context = { only = { "source.organizeImports" } },
              apply = true,
            })
            vim.wait(50)
          end
          vim.lsp.buf.format({ bufnr = bufnr, async = false })
        end,
      })
    end

    -- Diagnostics Configuration
    vim.diagnostic.config({
        virtual_text = true,
        signs = true,
        update_in_insert = false,
        float = { border = "rounded" },
    })
  end

  -- ==================================================
  -- LSP SERVER SETUP
  -- ==================================================

  -- Setup RUFF
  lspconfig.ruff.setup {
    on_attach = on_attach,
    capabilities = capabilities,
  }

  -- Setup PYRIGHT
  lspconfig.pyright.setup {
    on_attach = function(client, bufnr)
      client.server_capabilities.documentFormattingProvider = false
      on_attach(client, bufnr)
    end,
    capabilities = capabilities,
    settings = {
      pyright = {
        disableOrganizeImports = true,
      },
      python = {
        analysis = {
          typeCheckingMode = "strict", -- or "basic"
          autoSearchPaths = true,
          useLibraryCodeForTypes = true,
        }
      }
    }
  }

  -- ==============================================================================
  -- AUTOCOMPLETION CONFIGURATION: nvim-cmp
  -- ==============================================================================
  cmp.setup({
    snippet = {
      -- REQUIRED - The expand function for vsnip
      expand = function(args)
        vim.fn["vsnip#anonymous"](args.body) -- For `vsnip` users.
      end,
    },
    mapping = cmp.mapping.preset.insert({
      ['<C-b>'] = cmp.mapping.scroll_docs(-4),
      ['<C-f>'] = cmp.mapping.scroll_docs(4),
      ['<C-Space>'] = cmp.mapping.complete(), -- Trigger completion menu
      ['<C-e>'] = cmp.mapping.abort(),
      ['<CR>'] = cmp.mapping.confirm({ select = true }), -- Accept selected item
    }),
    sources = cmp.config.sources({
      { name = 'nvim_lsp' },  -- Suggestions from LSP servers
      { name = 'vsnip' },     -- Suggestions from the vsnip snippet engine
    }, {
      { name = 'buffer' },    -- Suggestions from the current file buffer
    })
  })

  -- Command-line mode completion setup (uses path and cmdline sources)
  cmp.setup.cmdline(':', {
    mapping = cmp.mapping.preset.cmdline(),
    sources = cmp.config.sources({
      { name = 'path' }
    }, {
      { name = 'cmdline' }
    })
  })

  -- Search mode completion setup (uses buffer source)
  cmp.setup.cmdline({ '/', '?' }, {
    mapping = cmp.mapping.preset.cmdline(),
    sources = {
      { name = 'buffer' }
    }
  })
EOF
```

## Screenshots

![Example of autocompletion](vim_python_ide_1.png)
*Example of autocompletion*

![Example of linting errors and trailing whitespaces](vim_python_ide_2.png)
*Example of linting errors and trailing whitespaces*

![Example of hover documentation](vim_python_ide_3.png)
*Example of hover documentation*

*Thanks for reading! ;)*
