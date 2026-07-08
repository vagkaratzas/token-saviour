# Benchmark evidence behind the tool picks

Source: [token-consumption-benchmark](https://github.com/vagkaratzas/token-consumption-benchmark)
(9 tools vs a no-tool baseline; 8 comprehension tasks + 1 code-gen task on a realistic ~30-file
Python app; tokenizer `tiktoken o200k_base`; 7 tools run for real, Ponytail + lazy-cat modeled).
Re-run 2026-07-08.

## Aggregate — comprehension suite (input + output tokens)

| Scenario | Total | Δ vs baseline |
|----------|------:|:-------------:|
| baseline (no tool) | 8,564 | — |
| **serena** | 2,906 | **−66.1%** |
| graphify | 2,948 | −65.6% |
| CodeGraph | 3,932 | −54.1% |
| archex | 9,894 | **+15.5%** (bundle overhead > small repo) |
| **rtk** | 8,171 | −4.6% (−36% on command tasks) |
| Pare | 8,580 | +0.2% (−72% on the test run alone) |
| **caveman** | 8,505 | −0.7% (output-only; understated) |
| **stacked: serena + rtk + caveman** | 2,602 | **−69.6%** |
| stacked-pare: serena + Pare + caveman | 2,847 | −66.8% |

## Per task (Δ total vs baseline)

| Task | serena | graphify | CodeGraph | archex | rtk | Pare |
|------|:------:|:--------:|:---------:|:------:|:---:|:----:|
| A1 list a class's methods | **−71%** | −45% | −39% | −6% | +3% | 0% |
| A2 find all callers | −38% | −63% | **−78%** | +128% | −1% | +3% |
| A3 trace call path API→DB | −62% | **−89%** | −68% | +25% | −2% | 0% |
| B1 run test suite | 0% | 0% | 0% | 0% | −65% | **−72%** |
| B2 list project structure | 0% | 0% | 0% | 0% | **−38%** | +32% |
| B3 grep deprecated + TODOs | 0% | 0% | 0% | 0% | **−1%** | +11% |
| C1 explain architecture | **−85%** | −64% | −45% | −24% | −2% | 0% |
| C2 describe feature end-to-end | −65% | **−81%** | −80% | +70% | −3% | 0% |

## Code generation (D1: add a due-date field end-to-end)

Output component alone (684-token verbose baseline implementation):

- **Ponytail: 684 → 400 tokens (−41.5%)** — deleted the unrequested `is_overdue` property,
  `overdue_tasks`/`list_overdue` helpers, defensive parsing.
- lazy-cat: 684 → 373 tokens (−45.6%) — same deletions; the difference vs Ponytail is
  single-sample LLM noise, not signal.
- caveman: −0.4% (leaves code blocks intact — wrong layer).
- Best stack: serena input + Ponytail output = **−63.6%** total.

⚠️ Ponytail/lazy-cat are *modeled* (no headless CLI): their real rule texts applied to a fixed
verbose baseline via the `claude` CLI, identical framing for both. Direction is solid; the
magnitude is authoring-dependent (a "preserve behaviour" framing gave only −1.6%).

## Why each layer's winner won

- **serena (code-read)**: input was ~88% of the baseline bill; symbol-level retrieval is the
  only lever that moves it. graphify/CodeGraph split individual events (call paths / callers)
  but serena wins aggregate *and* is the only one that also edits. archex never breaks even on
  a small repo — its per-query bundle (file tree + ranked chunks) is built for monorepos.
- **rtk (command-output)**: wins 2 of 3 command tasks and never loses one. Pare's structured
  `pytest` is the best single command cell (5 tokens) but its `find`/`search` are *more*
  verbose than terse Unix output on a clean repo.
- **caveman (prose output)**: only tool in its layer; small on concise text, grows with
  chattiness; can *add* tokens on already-terse list output.
- **Ponytail (code output)**: tied with lazy-cat on tokens; picked for intensity levels
  (`lite|full|ultra`) and explicit "when NOT to be lazy" guardrails (trust boundaries, error
  handling, security).

## Limitations

- Token usage computed from real tool artifacts, not metered from a live LLM session.
- One small, clean codebase: understates rtk and caveman; noisier repos widen their wins.
- Pare replaces commands only (its scenario keeps baseline file reads); read its per-task B
  cells, not the aggregate.
- One retrieval per task; single run; one tokenizer (ratios stable across tokenizers).
