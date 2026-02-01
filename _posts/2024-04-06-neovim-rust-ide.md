---
title: Configuring Neovim as Rust IDE
date: 2024-04-06 00:00:01 +0000
categories: [neovim, rust]
tags: [programming, ide, rust, neovim, nvim]
img_path: /assets/img/posts/
---

If you have read my article [Configuring Neovim as Python IDE](https://rubenhortas.github.io/posts/neovim-python-ide/), you already know that one of mi favorite editors is [Neovim](https://neovim.io).
Between other reasons, when I'm using [neo]vim I feel more focused, I don't know why.

One of my open fronts, in my spare time, is learn Rust, and [Neovim](https://neovim.io) (in this case) seems a great IDE to do it.
I have to say that programming in rust using [Neovim](https://neovim.io) it reminds me of the old days programming in Standard C using [vim](https://www.vim.org/download.php).

Convert [Neovim](https://neovim.io) in a Rust IDE it's very fast and straightforward.
After installing Rust, we only will need `rust-analyzer` and three vim plugins.

## rust-analyzer

`rust-analyzer` is an implementation of Language Server Protocol (LSP) for the Rust programming language.
It provides features like completion and goto definition for many code editors.

`rust-analyzer` needs the sources of the standard library, we can install them via `rustup`:

`rustup component add rust-src`

Now, we can install `rust-analyzer` via `rustup`:

`rustup component add rust-analyzer`

## Neovim base configuration

As always, I'll start from my base configuration.
This is my universal configuration, that I use across all my machines, regardless of the task

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

## Neovim plugin manager

As plugin manager, my choice is [vim-plug](https://github.com/junegunn/vim-plug#neovim), and its installation it's very straightforward:

Unix/linux:

`sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'`

## Plugins

I install four plugins:

* [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)
  Configs for the Neovim LSP client.

* [nvim-cmp](https://github.com/hrsh7th/nvim-cmp)
  A completion engine plugin.

* [rust-lang](https://github.com/rust-lang/rust.vim-plug)
  Provides Rust file detection, syntax highlighting, formatting, Syntastic integration, and more.

* [Vim Better Whitespace](https://github.com/ntpeters/vim-better-whitespace)
  This plugin highlights all trailing whitespaces.
  It's completely optional and can be replaced by the option "lines" in the `vim.init` file, but I like it.

We need to create the directory where we will install the plugins.
I install my plugins in `~/.config/nvim/plugins`, so:

`mkdir ~/.config/nvim/plugins`

### nvim-lspconfig

To install it, we add the plugin to our `init.vim` file, into the `call plug#begin('~/.config/nvim/plugins')` section, below all the lines:

```vim
call plug#begin('~/.config/nvim/plugins')
...

Plug 'neovim/nvim-lspconfig'

call plug#end()
```

We will also pass LSP settings to the server adding within a lua block (if exists):

```vim
lua << EOF
local lspconfig = require'lspconfig'

local on_attach = function(client)
    require'completion'.on_attach(client)
end

lspconfig.rust_analyzer.setup({
    on_attach = on_attach,
    settings = {
        ["rust-analyzer"] = {
            imports = {
                granularity = {
                    group = "module",
                },
                prefix = "self",
            },
            cargo = {
                buildScripts = {
                    enable = true,
                },
            },
            procMacro = {
                enable = true
            },
        }
    }
})
EOF
```

### nvim-cmp

> If you already have installed this plugin, you only need to add the set up lspconfig section for `rust_analyzer`:
```
  require('lspconfig')['rust_analyzer'].setup {
    capabilities = capabilities
  }
```
{: .prompt-info}

A completion engine plugin for neovim written in Lua. Completion sources are installed from external repositories and "sourced".
To install it we need to add the following to our `init.vim` file (respecting the blocks, if exists):

```vim
call plug#begin('~/.config/nvim/plugins')
Plug 'neovim/nvim-lspconfig'
Plug 'hrsh7th/cmp-nvim-lsp'
Plug 'hrsh7th/cmp-buffer'
Plug 'hrsh7th/cmp-path'
Plug 'hrsh7th/cmp-cmdline'
Plug 'hrsh7th/nvim-cmp'

" For vsnip users.
Plug 'hrsh7th/cmp-vsnip'
Plug 'hrsh7th/vim-vsnip'

call plug#end()

lua <<EOF
  -- Set up nvim-cmp.
  local cmp = require'cmp'

  cmp.setup({
    snippet = {
      -- REQUIRED - you must specify a snippet engine
      expand = function(args)
        vim.fn["vsnip#anonymous"](args.body) -- For `vsnip` users.
        -- require('luasnip').lsp_expand(args.body) -- For `luasnip` users.
        -- require('snippy').expand_snippet(args.body) -- For `snippy` users.
        -- vim.fn["UltiSnips#Anon"](args.body) -- For `ultisnips` users.
      end,
    },
    window = {
      -- completion = cmp.config.window.bordered(),
      -- documentation = cmp.config.window.bordered(),
    },
    mapping = cmp.mapping.preset.insert({
      ['<C-b>'] = cmp.mapping.scroll_docs(-4),
      ['<C-f>'] = cmp.mapping.scroll_docs(4),
      ['<C-Space>'] = cmp.mapping.complete(),
      ['<C-e>'] = cmp.mapping.abort(),
      ['<CR>'] = cmp.mapping.confirm({ select = true }), -- Accept currently selected item. Set `select` to `false` to only confirm explicitly selected items.
    }),
    sources = cmp.config.sources({
      { name = 'nvim_lsp' },
      { name = 'vsnip' }, -- For vsnip users.
      -- { name = 'luasnip' }, -- For luasnip users.
      -- { name = 'ultisnips' }, -- For ultisnips users.
      -- { name = 'snippy' }, -- For snippy users.
    }, {
      { name = 'buffer' },
    })
  })

  -- Set configuration for specific filetype.
  cmp.setup.filetype('gitcommit', {
    sources = cmp.config.sources({
      { name = 'git' }, -- You can specify the `git` source if [you have installed it](https://github.com/petertriho/cmp-git).
    }, {
      { name = 'buffer' },
    })
  })

  -- Use buffer source for `/` and `?` (if you enabled `native_menu`, this won't work anymore).
  cmp.setup.cmdline({ '/', '?' }, {
    mapping = cmp.mapping.preset.cmdline(),
    sources = {
      { name = 'buffer' }
    }
  })

  -- Use cmdline & path source for ':' (if you enabled `native_menu`, this won't work anymore).
  cmp.setup.cmdline(':', {
    mapping = cmp.mapping.preset.cmdline(),
    sources = cmp.config.sources({
      { name = 'path' }
    }, {
      { name = 'cmdline' }
    })
  })

  -- Set up lspconfig.
  local capabilities = require('cmp_nvim_lsp').default_capabilities()
  -- Replace <YOUR_LSP_SERVER> with each lsp server you've enabled.
  require('lspconfig')['rust_analyzer'].setup {
    capabilities = capabilities
  }
EOF
```

### rust-lang

To install it, we add the plugin to our `init.vim` file, into the `call plug#begin('~/.config/nvim/plugins')` section, below all the lines:

```vim
call plug#begin('~/.config/nvim/plugins')
...

Plug 'rust-lang/rust.vim'

call plug#end()
```

### Vim Better Whitespace

This plugin causes all trailing whitespace characters to be highlighted.
Whitespace for the current line will not be highlighted while in insert mode.
It is possible to disable current line highlighting while in other modes as well.
A helper function `:StripWhitespace` is also provided to make whitespace cleaning painless.

This plugin is optional, and can be substituded by the "lines" option in the `vim.init` file.
But, I like it, and the `:StripWhitespace` function is very useful.

To install it we add the plugin to our `init.vim` file, into the `call plug#begin('~/.config/nvim/plugins')` section, below all the lines:

```vim
call plug#begin('~/.config/nvim/plugins')
...

Plug 'ntpeters/vim-better-whitespace'

call plug#end()
```

## Install the plugins

To install the plugins we open nvim and run:

`:PlugInstall`

## Screenshots

![Example of autocompletion](vim_rust_ide_1.png)
*Example of autocompletion*

![Example of linting errors and trailing whitespaces](vim_rust_ide_2.png)
*Example of linting errors*

*Enjoy! ;)*
