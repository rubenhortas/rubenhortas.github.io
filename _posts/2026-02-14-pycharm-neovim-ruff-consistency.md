---
title: Unify Python Linting. Syncing Ruff between PyCharm and Neovim
date: 2026-02-14 00:00:01 +0000
categories: [python, linitng]
tags: [python, linitng, ruff, pycharm, neovim]
---

Configure PyCharm and Neovim to ensure your develpment environment is consistent and your code style matches, regardless of which editor your choose.

## Why?

A linter is a tool used in software development to analyze source code for errors, bugs, and stylistic issues.
It helps improve code quality by flagging problems before the code is executed.
Linters are essential for maintaining coding standards and ensuring that code is clean and maintainable.

When we program in python, although IDEs claim to follow PEP8, the reality is that PEP8 is a style guide, not a single software configuration.

Since I usually use PyCharm and Neovim as python IDEs, I, sometimes, encounter minor formatting style differences.
These small differences slightly break the consistency between files/files projects and end up costing me some time to fix.

## Ruff

PyCharm saves its configuration in the `.idea/` folder or in its global settings.
Neovim looks for standard configuration files in the project root.
For consistency between linters, ideally both should read the same source of truth.

`Ruff` is currently the de facto standard because it replaces almost a dozen tools (flake8, isort, pydocstyle, etc.) and is faster because it's written in Rust.

The plan is to keep `Pyright` for type analysis (where it excels) and add `Ruff` for linting and automatic formatting.

First off, let's install `Ruff`:

`pipx install ruff`

Now, we uninstall `python-lsp-server`

`pipx uninstall python-lsp-server`

## Create a common python standard

As we mentioned before, to ensure both IDEs behave the same, they should read the same font.
To force both IDEs to follow the same rules, we need to have this file in the root of our projects:

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

## Configure Neovim

Instead of pylsp (which is heavier), the ideal solution is to use the new ruff server.

Let's add the following sections to our `init.vim`:

```lua
local lspconfig = require('lspconfig')

-- Configuración de Ruff
lspconfig.ruff.setup({
  on_attach = function(client, bufnr)
    -- Deshabilitar el hover de Ruff si prefieres el de Pyright
    client.server_capabilities.hoverProvider = false
  end,
})

-- Mantén Pyright para el tipado, pero deja que Ruff limpie el código
lspconfig.pyright.setup({
  settings = {
    pyright = {
      -- Usamos Ruff para organizar imports
      disableOrganizeImports = true,
    },
    python = {
      analysis = {
        ignore = { '*' }, -- Opcional: deja que Ruff maneje el linting
      },
    },
  },
})
``` 

## Configure PyCharm

Starting in 2025.3, JetBrains included Ruff as a "Core" tool (native) through the Python LSP plugin (enabled by default).

To configure `Ruff`: Go to Settings (Ctrl+Alt+S) > Python > Tools > Ruff:

  * Select the `Enable`checkbox to start configuring `Ruff` settings.

  * Interpreter mode: PyCharm searches for a `Ruff` executable installed in your interpreter. 
    To install the `Ruff` package for the selected interpreter, click `Install Ruff`.

  * Path mode: PyCharm searches for a `Ruff` executable in `$PATH`. 
    If the executable is not found, you can specify the path by clicking the `Browse...` icon.

  * Select which `Ruff` options should be enabled.

### JetBrains doc:

* [Python tools support](https://www.jetbrains.com/help/pycharm/lsp-tools.html)
* [Configure Ruff](https://www.jetbrains.com/help/pycharm/lsp-tools.html#configure-ruff)

*Thanks for reading! :)*
