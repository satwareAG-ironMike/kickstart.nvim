# AGENTS.md — Personal Neovim Configuration (satware® AI Environment)

## Project Type

Personal Neovim configuration for use in the [satware® AI](https://satware.ai/) environment. Based on the kickstart.nvim template — a single-file Lua config for Neovim 0.12+. Not a distribution; it's a starting point for personal customization.

## Architecture

The entire configuration lives in a single `init.lua` (~977 lines), organized into numbered sections:

| Section | Purpose |
|---------|---------|
| 1 | Foundation: options, leader key, basic keymaps, diagnostics config |
| 2 | Plugin manager (`vim.pack`), build hooks (`PackChanged` autocmd) |
| 3 | UI/UX: colorscheme (tokyonight), which-key, mini.nvim modules, statusline |
| 4 | Search & navigation: Telescope (fuzzy finder) + LSP picker keymaps via `LspAttach` |
| 5 | LSP: lspconfig, Mason (tool installer), fidget.nvim, per-language server config |
| 6 | Formatting: conform.nvim |
| 7 | Autocomplete & snippets: blink.cmp + luasnip |
| 8 | Treesitter: parser installation, auto-attach via `FileType` autocmd |
| 9 | Optional example plugins (commented out) |

### Plugin installation pattern

```lua
-- Simple install (uses `gh` helper for GitHub URLs)
vim.pack.add { gh 'lewis6991/gitsigns.nvim' }

-- With version constraint
vim.pack.add { { src = gh 'saghen/blink.cmp', version = vim.version.range '1.*' } }

-- Multiple at once
vim.pack.add { gh 'nvim-lua/plenary.nvim', gh 'nvim-telescope/telescope.nvim' }

-- Post-install setup
require('gitsigns').setup { ... }
```

The `gh(repo)` helper (line 320) expands `repo` to `https://github.com/repo`.

### Plugin directories

- **`lua/kickstart/plugins/`** — Example plugins from the kickstart repo (commented out in init.lua, included one at a time). These are **examples**, not loaded by default.
- **`lua/custom/plugins/`** — Custom plugins. `init.lua` auto-loads any `.lua` file here (except `init.lua` itself). Add new plugins by creating files in this directory.

### Build hooks

`PackChanged` autocmd (line 291) runs build commands after `install`/`update`:
- `telescope-fzf-native.nvim` → `make`
- `LuaSnip` → `make install_jsregexp`
- `nvim-treesitter` → `TSUpdate`

## Commands

### Neovim commands (internal)

| Command | Purpose |
|---------|---------|
| `:Tutor` | Open Neovim tutorial |
| `:checkhealth` | Verify Neovim and plugin health |
| `:lua vim.pack.update()` | Update all plugins |
| `:lua vim.pack.update(nil, { offline = true })` | Inspect plugin state without updating |
| `:Mason` | Mason tool installer UI |
| `:TSInstall <lang>` | Install a treesitter parser |
| `:TSUpdate` | Update all treesitter parsers |
| `<space>sh` | Search help documentation |
| `<space>sf` | Fuzzy find files (Telescope) |
| `<space>ss` | Telescope picker browser |
| `<space>sk` | Fuzzy find keymaps |
| `<space>sg` | Live grep |
| `<space>sw` | Search current word |
| `<space>sd` | Search diagnostics |
| `<space>sc` | Search commands |
| `<space>sn` | Search Neovim config files |
| `<space>s/` | Live grep in open files |
| `<space>/` | Fuzzy search in current buffer |
| `<leader>q` | Open diagnostic quickfix list |
| `<leader>f` | Format buffer |
| `<leader>th` | Toggle inlay hints |
| `<leader>s` | Search group (which-key) |
| `<leader>t` | Toggle group (which-key) |
| `<leader>h` | Git hunk group (which-key) |
| `<C-h/j/k/l>` | Move window focus |
| `<Esc>` | Clear search highlights |
| `<F5>` | Debug: Start/Continue (dap) |
| `<F1>` | Debug: Step Into |
| `<F2>` | Debug: Step Over |
| `<F3>` | Debug: Step Out |
| `<leader>b` | Debug: Toggle Breakpoint |
| `<F7>` | Debug: See last session result |
| `\\` | NeoTree reveal (if enabled) |

### External commands

- `stylua --check .` — Check Lua formatting (per `.stylua.toml`)
- `stylua .` — Format all Lua files
- `nvim-pack-lock.json` — Managed by `vim.pack`; track in version control for forks

## Lua Style

Configured by `.stylua.toml`:

- **Indent**: 2 spaces
- **Line ending**: Unix
- **Column width**: 160
- **Quote style**: AutoPreferSingle
- **Call parentheses**: None
- **Collapse simple statement**: Always

All Lua files should be formatted with Stylua before committing.

## Key Patterns & Gotchas

### Leader key timing
`vim.g.mapleader` **must** be set before plugins are loaded (line 98). Changing it after plugin load causes wrong leader bindings.

### LSP formatting conflict
`lua_ls` has `documentFormattingProvider = false` and `format.enable = false` (lines 705, 731) because formatting is handled by `stylua` via `conform.nvim`, not the LSP. This prevents double-formatting.

### Treesitter auto-attach
The `FileType` autocmd (line 926) auto-installs missing parsers and attaches them. This means treesitter features work immediately for new filetypes without manual `:TSInstall`.

### Telescope LSP keymaps are buffer-scoped
LSP-related Telescope keymaps (`grd`, `grd`, `grr`, `gri`, `grt`, `gO`, `gW`) are set via `LspAttach` autocmd with `buffer = buf` — they only exist for buffers with an attached LSP.

### Plugin version constraints
Some plugins use `vim.version.range` for version constraints (blink.cmp `1.*`, LuaSnip `2.*`). This pins to major versions to avoid breaking changes.

### Nerd font conditional
`vim.g.have_nerd_font = false` (line 102) controls several features: `nvim-web-devicons`, which-key icons, mini.statusline icons. Set to `true` if using a Nerd Font.

### `vim.pack` vs `vim.fn.stdpath`
Use `vim.fn.stdpath 'config'` for the config directory path, not hardcoded `~/.config/nvim`. This respects `NVIM_APPNAME` and XDG paths.

### Mason tool install list
Only `stylua` is in `ensure_installed` (line 754). Other LSP servers (gopls, pyright, etc.) are commented out — uncomment and add to the list to install them.

### Custom plugins auto-load
Any `.lua` file in `lua/custom/plugins/` is auto-required by `lua/custom/plugins/init.lua` (line 11). Create a new file there for new plugins; no need to modify `init.lua`.

### `lua_ls` library config
The `on_init` hook (line 704) extends the Lua workspace library to include Neovim runtime files, excluding the config directory itself (to avoid slow analysis). This is critical for proper Lua LSP completion in the config.

## Testing

Neovim has no traditional test suite. Validation is done via:

- `:checkhealth` — Run in Neovim to verify plugin health
- `:lua vim.pack.update(nil, { offline = true })` — Inspect plugin state
- Manual testing of keymaps and features in Neovim
- `stylua --check .` — Verify Lua formatting before committing

## External Dependencies

Required: `git`, `make`, `unzip`, `rg` (ripgrep)

Optional: `fd-find`, `tree-sitter CLI`, `gcc` (for native extensions), a Nerd Font, language-specific tools (go for delve, npm for typescript, etc.)

## Adding New Plugins

1. Create `lua/custom/plugins/<name>.lua`
2. Add the plugin: `vim.pack.add { gh 'user/repo' }`
3. Call setup: `require('<name>').setup { ... }`
4. Optionally add keymaps via `vim.keymap.set`

## Troubleshooting

- **Plugin not loading**: Run `:checkhealth` — most warnings are about missing optional tools, not broken configs
- **LSP not working**: Ensure the LSP server is installed via `:Mason`, then restart Neovim
- **Formatting not working**: Check `conform.nvim` config in `lua/kickstart/plugins/lint.lua` and ensure the formatter is installed via Mason
- **Treesitter not highlighting**: Run `:TSInstall <lang>` for the language
- **Stylua errors**: Run `stylua .` to auto-format
