---
title: Configuring Neovim as Rust IDE
date: 2024-04-06 00:00:01 +0000
categories: [programming, ide]
tags: [programming, ide, rust, neovim, nvim]
img_path: /assets/img/posts/
---

If you read my article [Configuring Neovim as Python IDE](https://rubenhortas.github.io/posts/configuring-neovim-as-python-ide/), you already know that one of mi favorite editors is [neo]vim.
Between other reasons, when I'm using [neo]vim I feel more focused, I don't know why.

One of my open fronts, in my spare time, is learn Rust, and neovim (in this case) seems a great IDE to do it.
I have to say that programming in rust using neovim it reminds me of the old days programming in (ANSI) C using vim.

Convert neovim in a Rust IDE it's very fast and very straightforward.
After installing Rust, we only will need `rust-analyzer` and three vim plugins.

## rust-analyzer

`rust-analyzer` is an implementation of Language Server Protocol (LSP) for the Rust programming language.
It provides features like completion and goto definition for many code editors.

`rust-analyzer` needs the sources of the standard library, we can install them via `rustup`:

`rustup component add rust-src`

Now, we can install `rust-analyzer` via `rustup`:

`rustup component add rust-analyzer`

## [Neo]vim base configuration

As allways, I'll part from my base configuration.
This is my configuration for all, in all my computers, no matter the purpose.

`~/.config/nvim/init.vim`:

```lua
syntax on                       "syntax highlighting
filetype plugin indent on       "file type detection
set number                      "display line number
set path+=**                    "improves searching
set noswapfile                  "disable use of swap files
set wildmenu                    "completion menu
set backspace=indent,eol,start  "ensure proper backspace functionality
set incsearch                   "see results while search is being typed, see :help incsearch
set smartindent                 "auto indent on new lines, see :help smartindent
set expandtab                   "expanding tab to spaces
set tabstop=4                   "setting tab to 4 columns
set shiftwidth=4                "setting tab to 4 columns
set softtabstop=4               "setting tab to 4 columns
set showmatch                   "display matching bracket or parenthesis
set hlsearch incsearch          "highlight all pervious search pattern with incsearch
"set list                        "show all whitespaces (uncomment without Better Whitespace plugin)
```

## [Neo]vim plugin manager

As plugin manager, my choice is [vim-plug](https://github.com/junegunn/vim-plug#neovim), and its installation it's very straightforward:

Unix/linux:

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

## Plugins

I install four plugins:

* [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)
  Configs for the Nvim LSP client.
  
* [nvim-cmp](https://github.com/hrsh7th/nvim-cmp)
  A completion engine plugin.
  
* [rust-lang](https://github.com/rust-lang/rust.vim-plug)
  Provides Rust file detection, syntax highlighting, formatting, Syntastic integration, and more.
  
* [Vim Better Whitespace](https://github.com/ntpeters/vim-better-whitespace)
  This plugin highlights all trailing whitespaces. 
  It's totally optional and can be substituted by the option "lines" in the `vim.init` file, but I like it.

### nvim-lspconfig

To install it, we add the plugin to our `init.vim` file, into the `call plug#begin('/home/rubenhortas/.config/nvim/plugins')` section, below all the lines:

```lua
call plug#begin('/home/rubenhortas/.config/nvim/plugins')
...

Plug 'neovim/nvim-lspconfig'

call plug#end()
```

We will also pass LSP settings to the server adding:

```
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
To install it we need to add the following to our `init.vim` file:

```lua
call plug#begin('/home/rubenhortas/.config/nvim/plugins')
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
      { name = 'git' }, -- You can specify the `git` source if [you were installed it](https://github.com/petertriho/cmp-git).
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

To install it, we add the plugin to our `init.vim` file, into the `call plug#begin('/home/rubenhortas/.config/nvim/plugins')` section, below all the lines:

```lua
call plug#begin('/home/rubenhortas/.config/nvim/plugins')
...

Plug 'rust-lang/rust.vim'

call plug#end()
```

### Vim Better Whitespace

This plugin causes all trailing whitespace characters to be highlighted.
Whitespace for the current line will not be highlighted while in insert mode.
It is possible to disable current line highlighting while in other modes as well.
A helper function :StripWhitespace is also provided to make whitespace cleaning painless.

This plugin is optional, and can be substituded by the "lines" option in the `vim.init` file.
But, I like it, and the `:StripWhitespace` function is very useful.

To install it we add the plugin to our `init.vim` file, into the `call plug#begin('/home/rubenhortas/.config/nvim/plugins')` section, below all the lines:

```lua
call plug#begin('/home/rubenhortas/.config/nvim/plugins')
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
