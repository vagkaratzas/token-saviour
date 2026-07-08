# token-saviour

**Spend tokens where they matter.** An agent skill that routes every coding task to the most
token-efficient tool for the layer it stresses — instead of reflexively reading whole files,
dumping raw command output into context, or writing more code than asked.

Token cost has **four independent layers**, and a different tool owns each. token-saviour picks
one winner per layer — the best measured combination from a
[9-tool benchmark](https://github.com/vagkaratzas/token-consumption-benchmark) on a realistic
Python codebase:

| Layer | Tool | Measured |
|-------|------|----------|
| Code-read **input** (symbols, callers, call paths, architecture) | [serena](https://github.com/oraios/serena) | **−66%** alone — the dominant cost |
| Command-output **input** (tests, builds, git, grep, listings) | [rtk](https://github.com/rtk-ai/rtk) | −65% on a test run, −36% on command tasks |
| Generated **prose output** (chatty replies, write-ups) | [caveman](https://github.com/juliusbrussee/caveman) | −6% on prose answers (grows with chattiness) |
| Generated **code output** (implementations you write) | [Ponytail](https://github.com/DietrichGebert/ponytail) | ~−40% on a verbose implementation (illustrative) |

Stacked (serena + rtk + caveman): **−69.6% total tokens** on the comprehension suite. For
code-generation work, swap caveman → Ponytail (≈ −64%).

The picks were re-validated (2026-07) against newer entrants — archex, Pare, lazy-cat — and every
incumbent held its layer. The runner-ups, the niches where they flip (e.g. Pare's structured
`pytest` for test-dominated loops), and the full evidence live in
[`skills/token-saviour/references/`](skills/token-saviour/references/).

## Install

### Claude Code (plugin)

```
/plugin marketplace add vagkaratzas/token-saviour
/plugin install token-saviour@token-saviour
```

Or as a plain personal skill, no plugin system involved:

```bash
git clone https://github.com/vagkaratzas/token-saviour
mkdir -p ~/.claude/skills
cp -r token-saviour/skills/token-saviour ~/.claude/skills/
```

### Codex

```bash
codex plugin marketplace add vagkaratzas/token-saviour
codex plugin add token-saviour@token-saviour
```

In Codex the skill is invoked with `@token-saviour`. The VS Code Codex extension and the Codex
app read `AGENTS.md`, which this repo ships — so it also works from the repo root with no
setup, and `cp AGENTS.md ~/.codex/AGENTS.md` makes the always-on rules global.

### Any other agent

Copy `AGENTS.md` (compact, always-on ruleset) or `skills/token-saviour/SKILL.md` (full
playbook) into whatever instruction file your agent reads.

## What the skill does

1. **Classifies the task** by the layer it stresses: reading code, reading command output,
   writing prose, or writing code.
2. **Routes to that layer's tool** with concrete commands (serena MCP calls, `rtk` proxies,
   caveman terse mode, Ponytail rules) — and degrades gracefully to plain Read/Grep/Bash when a
   tool isn't installed (install/verify commands in
   [`references/tool_links.md`](skills/token-saviour/references/tool_links.md)).
3. **Refuses anti-patterns**: two code-read tools at once, rtk for comprehension, caveman on
   code, Ponytail on prose, tooling-up trivial one-line lookups.
4. **Announces what it used**: `🪙 token-saviour: serena + rtk + caveman`.

## The tools it routes to (install the smallest set you need)

- **serena** — `uv tool install -p 3.13 serena-agent` — the one code-read tool (LSP symbols;
  also does semantic edits/renames).
- **rtk** — `brew install rtk` or `cargo install --git https://github.com/rtk-ai/rtk` — add
  only for noisy command loops.
- **caveman** — see [repo](https://github.com/juliusbrussee/caveman) — add only when prose
  brevity is the bottleneck.
- **Ponytail** — `/plugin marketplace add DietrichGebert/ponytail` — add only when
  code-generation work is the bottleneck.

## Evidence

All numbers come from [token-consumption-benchmark](https://github.com/vagkaratzas/token-consumption-benchmark):
real tools, real artifacts, one tokenizer (`tiktoken o200k_base`), per-task tables, and honest
limitations (modeled-not-metered, small clean codebase, single run). Summary:
[`references/benchmark_results.md`](skills/token-saviour/references/benchmark_results.md).

## License

MIT — see [LICENSE](LICENSE).
