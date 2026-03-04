---
paths:
  - "lua/**/*.lua"
  - "plugin/**/*.vim"
---

# Lua / Vim Script Conventions

## Quality Gate (must pass before commit)
```
make check   # luacheck + stylua
```

## Tooling
- **luacheck**: `.luacheckrc` — Lua 5.1, vim globals, no line length limit
- **stylua**: `.stylua.toml` — 120 char width, 2-space indent, Unix newlines

## Rules
- Main logic in Lua (`lua/marp.lua`), Vim script (`plugin/marp.vim`) is minimal bridge only
- Error feedback via `vim.notify()` with appropriate log levels
- All marp CLI invocations go through `/bin/sh -c` via `vim.fn.jobstart`
- Use `pcall` around `vim.fn.jobstop` calls
