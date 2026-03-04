---
paths:
  - "lua/marp.lua"
---

# Architecture — lua/marp.lua

## Key Internal State
- `M.active_processes[bufnr]` — job ID per buffer (watch/server)
- `M.metadata.browser_opened[bufnr]` — prevents duplicate browser launches
- `M.metadata.process_retries[bufnr]` — auto-restart counter (max 3)

## Helper Functions (modify with care)
| Function | Purpose |
|----------|---------|
| `get_marp_cmd()` | Resolves marp executable with NODE_OPTIONS prefix |
| `get_common_options()` | Builds flags applied to ALL invocations (theme-set) |
| `get_export_options(format)` | Builds format-specific flags (pdf-notes, image-scale, etc.) |
| `get_node_env_prefix()` | NODE_OPTIONS env var for Node.js v25+ compat |
| `find_marp_config_dir(path)` | Walks up to find .marprc.yml etc., sets cwd |

## Process Lifecycle
1. `watch()` → initial HTML gen via `vim.fn.system` → `jobstart` with `--watch`
2. Auto-restart on crash (exit_code != 0 and != 143), up to 3 retries
3. `BufDelete`/`BufWipeout` autocmd triggers `stop(bufnr)`
4. `stop()` sets retries=999 to suppress auto-restart, then `jobstop`
