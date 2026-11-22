---
name: route
description: >-
  Sort a task into easy, mid, or hard and outsource it with route-task.
  Use when the user says /route, "route this", "route as recommended", or asks
  to auto-pick a cheap executor. Until about 2026-10-14 the parent is the Claude
  desktop app on DeepSeek. Cursor stays closed. Fable stays unused on this gateway.
metadata.author: cemini23
metadata.version: "2026-10-14.1"
---

# /route — Claude desktop window (until about 2026-10-14)

The parent is this Claude desktop session. The picker label for every lane is DeepSeek V4.1 Flash. Implementation runs through `route-task` and Grok CLI.

Cursor quota is unavailable until about 2026-10-14. Leave Cursor closed. Fable and Opus run on Anthropic. This gateway serves DeepSeek V4.1 Flash through a local proxy port. Claude Desktop drops a raw `deepseek-*` id from the model list. Leave those ids off the picker.

**Model note (2026-10-01).** DeepSeek retired `deepseek-v4-flash` and phased out `deepseek-v4-pro` on 2026-09-10 and 2026-09-14. Both aliases now serve V4.1-Flash, whose API id is `deepseek-flash`. The proxy maps every route to it. There is no separate Pro tier, so the picker offers **one** entry, DeepSeek V4.1 Flash. Do not plan work around a Pro/Flash split.

When Cursor quota returns, stop using this file as the parent doctrine. The long chain stays in `route-task`. The Cursor skill file stays untouched.

**Zen model preference (2026-10-15).** The cheap chain's OpenCode step ranks the live Zen free catalog and now **prefers the `deepseek-v4.1` family** — `opencode/deepseek-v4.1-flash-free` while OpenCode serves it free. The preference is resolved against the live catalog on every pick, so it is **not a pin**: when the family leaves the catalog or stops being free, the ranking takes over with no edit.

- `ROUTE_ZEN_PREFER=""` disables the preference. `ROUTE_ZEN_PREFER="longcat,jev"` changes the order. `ROUTE_OPENCODE_MODEL=opencode/<id>` is still the hard pin.
- **Trade-off, state it plainly:** the capability ranking had put two unreleased previews (`longcat-2.5-preview-free`, `step-5-preview-free`) above DeepSeek, because `preview` scores as frontier while `flash` does not. Preferring DeepSeek buys a **known-good model and cost certainty, not necessarily the strongest free one.** If a cheap task fails twice under DeepSeek, retry with `ROUTE_ZEN_PREFER=""` before blaming the handoff.
- **Never put a credential in this lane.** An earlier audit found a harness leaking its own provider key into a DeepSeek Flash run; the Zen free tier needs no key, and that is a reason to keep it that way.

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
| hard | Code, tests, or multi-file edits. | Write the Plan here on DeepSeek V4.1 Flash. Then `route-task` so Grok CLI implements. |
| ~~money~~ | **Removed 2026-10-14.** Scoring, payments, Greeks, P&L, LIVE, ship. | Routes to **hard**. A live flip still needs LIVE OK. |
| ambiguous | Unclear. | Ask: easy, mid, or hard? |

`route-task` owns the executor chain. Easy runs OpenRouter free, then OpenCode Zen free, then `claude-ds` Flash. Mid runs a Grok CLI plan, then that cheap chain. When Grok usage is out, Flash plans a mid task. There is no Pro tier; every lane uses the same model. Hard implementation is Grok CLI.

## Timeout cascade (2026-11-22)

Each lane has a three-stage budget. Verify is enforced today by
`Invoke-RouteVerifyGate -CommandTimeoutSec`; Plan and Execute are policy targets, and
Grok and claude-ds keep their own hang watchdogs.

| Lane | Plan | Execute | Verify | Total |
|------|------|---------|--------|-------|
| easy | 120s | 180s | 120s | 420s |
| mid | 300s | 600s | 300s | 1200s |
| hard | 900s | 1800s | 600s | 3300s |

The budgets are conservative defaults, not a derivation from the internal benchmark. That
benchmark recorded a ~33% GPU-hour reduction with no accuracy loss; no measurement artifact
is stored in this repository, so cite it as an internal finding.

## Non-empty harness rule

`exit=0` does not prove a test ran. `npm test` with no tests, `mocha --passWithNoTests`,
and `go test` with no test files all exit 0. The Verify gate fails any test-runner command
that reports zero executed tests: pytest, jest, vitest, mocha, ava, tap, go test, cargo
test, dotnet test, node --test, unittest, and npm/pnpm/yarn test wrappers. A runner that
prints neither a count nor an empty marker is decided by exit code alone.

Do not weaken this rule to make a run green. An empty suite is a failed check.

## Hard plan

1. The picker already serves DeepSeek V4.1 Flash. It is the only tier.
2. Write a SIP handoff. Required sections: WorkDir, Success criteria, Verify, NEVER, and Plan.
3. Run `route-task -Profile <name> -WorkDir "<dir>" -HandoffPath "<path>" "<task>"`.
4. When Grok fails and the Plan is usable, run `route-task -SkipGrokPlan -HandoffPath "<path>" "<task>"`.
5. When the script says Grok auth failed, tell the operator to run `grok login`. That is not a usage-out.

A usable Plan is a filled Plan section. An empty placeholder is not a Plan.

**Lane comes from repository inspection, not prompt vocabulary (2026-10-14).** `route-task` reads
`WorkDir` and the named target files first. A documentation repository (`wiki/index.md`, no code
manifest) caps at **mid**, or **easy** for one or two markdown targets. A software repository is
**hard** at three or more executable source files, a schema or build signal, and **mid** below that.
The `money` lane was deleted; those tasks route to **hard**.

## Close

Report the lane, the executor chain, the verify evidence, and the residual risk. Run logs live under `$ROUTE_KIT/briefs/handoffs/_route_runs/`. A status line alone is not done.

Local lane check (no executor):

```bash
route-task -DryClassify -Profile <name> -WorkDir "<project dir>" "easy: rewrite this sentence to be shorter"
```

## $0-First Executor Hierarchy (2026-11-16)

- **Easy** — OpenRouter free → OpenCode free (`orfree/openrouter/free`) → `claude-ds` Flash (`claude-haiku-4-5`).
- **Mid** — Plan by Grok CLI or the parent. Implement: OpenCode free → OpenRouter free → `dsh` (`deepseek-chat`) or `claude-ds` (`claude-sonnet-4-5`). Markdown-only targets cap at mid.
- **Hard** — Plan in the parent on `claude-opus-4-6` (DeepSeek Reasoner), or `pack-ctx` → paid Gemini browser → `run-handoff`. Implement: Grok CLI (`grok-4.7`).
- **Verify gate** — a `## Verify` block that only runs `git status`, `git diff`, `git log`, or `echo` is rejected with exit 3: "SIP Gate Error: ## Verify cannot be git-only. Include a real test, linter, or syntax check." A free executor that fails Verify falls through to the next executor.
- **No secrets to free tiers** — the handoff secret check stays active for OpenRouter Free and OpenCode Free.

OpenCode free reads `OPENROUTER_API_KEY` from the environment. Restart the service after a config change:

```bash
opencode service restart
opencode run -m orfree/openrouter/free "Reply with the single word OK"
```
