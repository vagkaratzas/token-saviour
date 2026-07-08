# token-saviour — spend tokens where they matter

Always-on rules for agents that read `AGENTS.md` (Codex CLI/app, VS Code Codex extension, and
other instruction-only hosts). Full playbook with commands and evidence:
`skills/token-saviour/SKILL.md`.

Token cost has **four independent layers**; a different tool owns each. Before acting, ask:
"which layer is this task spending tokens on?" — then use that layer's tool if it is installed.
Never pretend a tool ran; if it's missing, fall back to plain Read/Grep/Bash (installs:
`skills/token-saviour/references/tool_links.md`).

| When you're about to… | Use instead | Layer |
|---|---|---|
| `cat`/read several files to find a symbol, callers, a call path, or explain a module | **serena** MCP tools: `get_symbols_overview`, `find_symbol`, `find_referencing_symbols` (it also edits: `replace_symbol_body`, `rename_symbol`) | code-read input |
| run a noisy command — tests, build, lint, `git`, `grep`, `find`, big listings | **rtk** proxies: `rtk test <cmd>`, `rtk grep`, `rtk read`, `rtk find`, `rtk git …` | command-output input |
| write a long, chatty prose reply | **caveman** terse mode (keep code, commands, errors, ordered/irreversible steps verbatim) | prose output |
| generate/edit a chunk of code | **Ponytail** rules: YAGNI, stdlib first, no unrequested abstractions, delete over add, one runnable check | code output |
| a tiny one-file/one-line lookup | plain Read/Grep/Bash — tool overhead beats the benefit | — |

Rules of thumb (all benchmark-measured; see `skills/token-saviour/references/benchmark_results.md`):

- Code reads dominate the bill (~88%); switching them to serena is the highest-value move
  (−66% alone; the full stack measured −70%).
- One tool per layer. Never run two code-read tools; never use rtk for comprehension (~0% there);
  never use caveman on code or Ponytail on prose (each ~0% outside its lane).
- If a loop is dominated by test runs and Pare's MCP servers are wired in, its structured
  `pytest` beats rtk on that one command (−72%); rtk still owns the layer overall.
- Announce what you used, one line: `🪙 token-saviour: serena + rtk + caveman` (or
  `plain tools (fallback)`).
