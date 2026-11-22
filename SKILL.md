---
name: route
description: >-
  Sort a task into easy, mid, or hard and outsource it with route-task.
  Use when the user says /route, "route this", "route as recommended", or asks
  to auto-pick a cheap executor. route-task drives OpenCode, claude-ds Flash,
  OpenRouter free, and Grok CLI.
metadata:
  author: cemini23
  version: "2026-11-22.2"
---

# /route — task router

The parent is this Claude desktop session. Implementation runs through `route-task` and Grok CLI.

DeepSeek V4.1 Flash is the only paid tier the router uses. Its API id is `deepseek-flash`. DeepSeek retired `deepseek-v4-flash` and `deepseek-v4-pro` on 2026-09-10 and 2026-09-14. Both aliases now resolve to `deepseek-flash`. The gateway serves it through a local proxy port. There is no Pro tier, so the picker offers one entry: DeepSeek V4.1 Flash. Do not plan work around a Pro/Flash split.

**Zen model preference.** The cheap chain's OpenCode step ranks the live Zen free catalog and prefers the `deepseek-v4.1` family — `opencode/deepseek-v4.1-flash-free` while OpenCode serves it free. The preference is resolved against the live catalog on every pick, so it is not a pin: when the family leaves the catalog or stops being free, the ranking takes over with no edit.

- `ROUTE_ZEN_PREFER=""` disables the preference. `ROUTE_ZEN_PREFER="longcat,jev"` changes the order. `ROUTE_OPENCODE_MODEL=opencode/<id>` is the hard pin.
- The offline fallback id is `opencode/x-preview-f-free`.
- **Trade-off:** the capability ranking had put two unreleased previews (`longcat-2.5-preview-free`, `step-5-preview-free`) above DeepSeek, because `preview` scores as frontier while `flash` does not. Preferring DeepSeek buys a known-good model and cost certainty, not necessarily the strongest free one. If a cheap task fails twice under DeepSeek, retry with `ROUTE_ZEN_PREFER=""` before blaming the handoff.
- **Never put a credential in this lane.** An earlier audit found a harness leaking its own provider key into a DeepSeek Flash run. The Zen free tier needs no key.

## Contract

- Shell entry: `route-task` on PATH. Profile: `-Profile <name>`, set to the profile configured on your machine.
- Kit: `ROUTE_KIT` points at your `agent-toolkit` checkout.
- Always-approve is already on. Leave it on. Pass `-NoApprove` only when the operator asks for ask mode.
- `-h` is not help. It binds to `-HandoffPath`. Use `-DryClassify` for a local lane check. That flag exits before any lane runs.
- Quote `-WorkDir` when the path contains spaces. Default WorkDir is the current project directory.
- Empty `/route`: ask what the task is.
- Put no secrets in a handoff. Send no secrets to OpenRouter or OpenCode.
- A LIVE Discord change or a live money flip needs the words LIVE OK from the operator.

## Lanes

Announce the lane and one reason. Then outsource. Do not type the implementation in this session.

| Lane | Meaning | Action |
|------|---------|--------|
| easy | Wording, drafts, wiki notes. No tools. | `route-task -Profile <name> -WorkDir "<dir>" "<task>"` |
| mid | One plan, then cheap tool work. | Same command. Grok CLI plans. Cheap execute follows. |
| hard | Code, tests, or multi-file edits. | Write the Plan here. Then `route-task` so Grok CLI implements. |
| ambiguous | Unclear. | Ask: easy, mid, or hard? |

## Executor chain (matches `route-task.ps1`)

- **Easy** — OpenRouter free → OpenCode Zen free → claude-ds Flash (`deepseek-flash`) → best live OpenRouter free. The first step runs only when `OPENROUTER_API_KEY` is set; its model is `openrouter/free` unless `OPENROUTER_EASY_MODEL` overrides it. The last step runs only when the key is set and the text holds no key material.
- **Mid** — Grok CLI plan, then the same cheap chain. When Grok usage is out, the parent writes the plan.
- **Hard** — Plan in the parent. Grok CLI implements through `handoff-to-grok.ps1`. If Grok fails and the Plan is usable, the backup chain is claude-ds Flash, then OpenCode.
- **OpenCode** — the model id is the live Zen free pick. Default preference: `opencode/deepseek-v4.1-flash-free`. OpenCode on the Zen free tier needs no OpenRouter key. The `orfree/` provider is not used by `route-task`.
- **claude-ds** — always `deepseek-flash`. Every alias, including `flash`, `pro`, and `vision`, resolves to it.
- **OpenRouter** — `openrouter/free` for easy and for the cheap-chain fallback. The premium model default is `z-ai/glm-5.2`; it is not part of the cheap chain.

## Timeout cascade (2026-11-22)

All three stages are enforced. `Get-RouteLaneTimeouts` supplies the budget for the classified lane.

| Lane | Plan | Execute | Verify | Total |
|------|------|---------|--------|-------|
| easy | 120s | 180s | 120s | 420s |
| mid | 300s | 600s | 300s | 1200s |
| hard | 900s | 1800s | 600s | 3300s |

- Plan and Execute pass `-MaxSeconds` to `Invoke-PwshScriptNamedArgs`. A stage that exceeds its budget is killed by process tree and returns `TimedOut = $true`.
- Verify passes `-CommandTimeoutSec` to `Invoke-RouteVerifyGate`.
- `ROUTE_CLAUDE_DS_MAX_SECONDS` (default 2700s) is an outer ceiling. It clamps a stage budget; it does not replace it. Set it explicitly to cap a long hard run.
- A timeout cascades. A Plan timeout falls to the Flash planner. An Execute timeout falls to the next executor in the chain. A stall kill (`KillReason = STALL`) is still a hang and hands off to the parent.
- Each stage appends `{ts, lane, stage, executor, duration_s, budget_s, outcome}` to `$ROUTE_KIT/briefs/handoffs/_route_runs/stages-YYYY-MM-DD.jsonl`. These are measurement artifacts.

The budgets are conservative defaults, not a derivation from the internal benchmark. That benchmark recorded a ~33% GPU-hour reduction with no accuracy loss; no measurement artifact is stored in this repository, so cite it as an internal finding.

## Non-empty harness rule

`exit=0` does not prove a test ran. `npm test` with no tests, `mocha --passWithNoTests`, and `go test` with no test files all exit 0. The Verify gate fails any test-runner command that reports zero executed tests: pytest, jest, vitest, mocha, ava, tap, go test, cargo test, dotnet test, node --test, unittest, and npm/pnpm/yarn test wrappers. A runner that prints neither a count nor an empty marker is decided by exit code alone.

Counts are summed across multi-block output. `cargo test` with 5 unit tests and a doc-test block of `running 0 tests` reports 5 executed and passes.

Do not weaken this rule to make a run green. An empty suite is a failed check.

## Hard plan

1. Write a SIP handoff. Required sections: WorkDir, Success criteria, Verify, NEVER, and Plan.
2. Run `route-task -Profile <name> -WorkDir "<dir>" -HandoffPath "<path>" "<task>"`.
3. When Grok fails and the Plan is usable, run `route-task -SkipGrokPlan -HandoffPath "<path>" "<task>"`.
4. When the script says Grok auth failed, tell the operator to run `grok login`. That is not a usage-out.

A usable Plan is a filled Plan section. An empty placeholder is not a Plan.

**Lane comes from repository inspection, not prompt vocabulary.** `route-task` reads `WorkDir` and the named target files first. A documentation repository (`wiki/index.md`, no code manifest) caps at **mid**, or **easy** for one or two markdown targets. A software repository is **hard** at three or more executable source files, a schema or build signal, and **mid** below that.

## Verify gate

A `## Verify` block that only runs `git status`, `git diff`, `git log`, or `echo` is rejected with exit 3: "SIP Gate Error: ## Verify cannot be git-only. Include a real test, linter, or syntax check." A free executor that fails Verify falls through to the next executor.

A runner whose output format the gate does not know is decided by exit code alone. An empty suite is not.

## Close

Report the lane, the executor chain, the verify evidence, and the residual risk. Run logs live under `$ROUTE_KIT/briefs/handoffs/_route_runs/`. A status line alone is not done.

Local lane check (no executor):

```bash
route-task -DryClassify -Profile <name> -WorkDir "<project dir>" "easy: rewrite this sentence to be shorter"
```
