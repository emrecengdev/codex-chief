<div align="center">

<img src="docs/assets/hero-v3.png" alt="codex-chief routing work from one chief architect to four specialized agents in a warm editorial studio" width="100%" />

# codex-chief

**A budget-disciplined multi-agent setup for OpenAI Codex.**

[![License: MIT](https://img.shields.io/badge/License-MIT-2ea44f.svg)](LICENSE)
![Codex CLI](https://img.shields.io/badge/Codex-CLI%20%7C%20Desktop%20%7C%20IDE-8A2BE2)
![Models](https://img.shields.io/badge/GPT--5.6-Sol%20·%20Terra%20·%20Luna-ff9f1c)
![Install](https://img.shields.io/badge/install-one%20prompt-00b4d8)

*Start cheap. Escalate on judgment. Never the reverse.*

</div>

---

Your Codex Pro quota burns too fast for one reason: **every task — trivial or hard — runs on your most expensive model at high reasoning effort.** And it compounds: Codex subagents inherit the parent's model and effort, so three subagents spawned from a Sol XHigh session are three Sol XHigh instances burning quota in parallel.

codex-chief fixes both with a small, evidence-based architecture — a cheap orchestrator and four tiered subagents, each pinned to the model and reasoning effort its job actually needs.

Built for the GPT-5.6 family (Sol / Terra / Luna) on ChatGPT Plus/Pro plans. Works identically in Codex CLI, the desktop app, and IDE extensions — they share the same config layers.

```
you
 └─ main agent · Terra Medium          triage, coordination, trivial edits
     ├─ explorer     · Luna High       quick read-only lookups, small scope
     ├─ explorer_xl  · Terra Medium    repo-wide / long-context exploration
     ├─ worker       · Luna XHigh      meaningful implementation, tests
     └─ judge        · Sol High        architecture, hard bugs, security, risky reviews
```

## Which model does which job — and why

The GPT-5.6 family gives you 3 model sizes × 6+ effort levels = a lot of wrong combinations. Each seat below was chosen from benchmark data, not vibes. Two structural facts from the official rate card drive everything:

> **Effort doesn't change the per-token price — it multiplies token volume.** And **cached input is 10× cheaper**, so a stable long-lived session is itself a cost optimization.

| Credits / 1M tokens | Input | Cached | Output |
|---|---|---|---|
| `gpt-5.6-sol` | 125 | 12.5 | 750 |
| `gpt-5.6-terra` | 62.5 | 6.25 | 375 |
| `gpt-5.6-luna` | 25 | 2.5 | 150 |

### 🎛️ Main agent — `gpt-5.6-terra` · `medium`

**The job:** triage every request, decide what to delegate, do trivial edits and small-diff reviews directly, and carry the long-lived session context.

**Why Terra:** it's the official "everyday workhorse" at half of Sol's credit rate — and the community's favorite cheap default (Luna High) actually loses to it on total cost: on the Artificial Analysis matrix both score 46, but Terra Medium costs 3.5× (normalized) vs Luna High's 4.0×, because Luna takes more steps to finish the same work. *Cheaper per token ≠ cheaper per task.*

**Why not Luna here:** the main thread is precisely the context that grows all day — and Luna's recall collapses past 256K context (41.3% vs Terra's 89.6% on MRCR v2 8-needle). The orchestrator seat needs long-context stamina more than raw coding power.

**Why not Sol here:** the most common tasks would run on the most expensive model. That's the failure mode this repo exists to fix. Sol is one `--profile heavy` away when you need it.

**Known trade-off:** Terra Medium is weak at end-to-end implementation (35% on DeepSWE). Irrelevant here — implementation is delegated to `worker`.

### 🔎 `explorer` — `gpt-5.6-luna` · `high` · read-only

**The job:** "where is X defined, who calls it?" — quick lookups, file reads, small-scope evidence gathering. Returns a ≤300-word summary with file:line references.

**Why Luna High:** cheapest capable tier (5× cheaper credits than Sol) and reading is exactly what it's good at in small scopes. On Pro plans Luna also has ~3× Sol's message allowance — routing lookups here directly extends your quota window.

**Why read-only sandbox:** an explorer that can write is a worker with bad instructions. The sandbox makes the role contract enforceable, not aspirational.

### 🗺️ `explorer_xl` — `gpt-5.6-terra` · `medium` · read-only

**The job:** repo-wide sweeps, long files, "how does this whole subsystem work?"

**Why a second explorer:** one variable separates the two — context size. Past ~256K context Luna misses what Terra catches (41% vs 90% recall). Capping Luna's context window wouldn't fix this: the task defines how much context is needed; a smaller window just forces compaction churn, re-reads, and more steps. When the reading is big, you pay Terra's rate or you pay twice.

### 🔨 `worker` — `gpt-5.6-luna` · `xhigh`

**The job:** meaningful, *bounded* implementation — features, fixes, test writing — with a minimal diff and verification. Instructed to stop and report back if the task turns out to need architectural judgment.

**Why Luna XHigh:** the cheap-model quality peak. DeepSWE v1.1: **57% task completion at $1.54/task** — vs Luna High's 44%, and vs Terra High which delivers the same class of work at 2.5× the cost and 1.3× slower. Extra reasoning effort helps small models disproportionately; XHigh is where Luna punches above its weight class.

**Why not Luna Medium/Low:** 11% and 2% DeepSWE completion. They write code; it just doesn't work. The retry loop of a too-weak model costs more than the stronger setting — this is the "capability mismatch" rule in the routing policy.

### ⚖️ `judge` — `gpt-5.6-sol` · `high`

**The job:** the 10% that's actually hard — architecture decisions, root-cause debugging, security, complex or high-risk review, ambiguous multi-step work. Returns conclusions with evidence, not working notes.

**Why Sol:** judgment is the one thing the cheap tiers can't fake. Sol leads the family on every reasoning-heavy benchmark (AA Coding Agent Index 80.0, Terminal-Bench 88.8, Agents' Last Exam 53.6).

**Why High, not XHigh/Max:** High is the cost/quality knee — 69% DeepSWE at $3.47 and 37 steps, vs Max's 73% at $8.39 and 61 steps. For the rare hardest problem, escalate a session to Sol XHigh (~97% of Max quality at 56% of its cost). Sol Max buys +1 point for ~2× the tokens. Skip it.

### 🚫 The do-not-use list

| Setting | Why it's out |
|---|---|
| `ultra` effort (any model) | Officially "uses subagents" internally — children inherit your model+effort, multiplying it. Community report: 2h of Sol Ultra ate 5h of a 20x plan window |
| Sol `max` | +1 point over xhigh for ~2–3× the token volume |
| Terra `high`/`xhigh`/`max` | Dominated: Sol at lower effort beats each on both score and cost (e.g. Terra High 49 @ 7.2× vs Sol Low 49 @ 5.1×) |
| Luna on >256K context | Recall cliff: 41% — worse than GPT-5.5 |
| Luna `low`/`medium` for code | 2% / 11% DeepSWE completion — scouting only |
| Fast mode | 1.5× speed for 2–2.5× credits; use `gpt-5.3-codex-spark` (separate Pro quota) for instant iteration instead |

Full evidence, sources, and the honest list of what rests on single benchmarks: **[docs/RATIONALE.md](docs/RATIONALE.md)**.

## When delegation pays (and when it doesn't)

Official docs are blunt: subagent workflows consume **more** tokens than the same work done inline. Delegation pays in exactly three cases, and the routing policy encodes them:

1. **Genuine parallelism** — independent workstreams running at once.
2. **Context hygiene** — each subagent gets a disposable context window, keeping bulk reads out of your expensive, cache-warm main thread.
3. **Capability mismatch** — the task exceeds the main model's tier (implementation → `worker`, judgment → `judge`), where the weaker model's retry loop costs more than the delegation overhead.

Everything else — trivial edits, simple docs, small-diff reviews — stays in the main thread. That's why the main agent had to be cheap.

## What's in the box

| Path | What it is |
|---|---|
| [`INSTALL_PROMPT.md`](INSTALL_PROMPT.md) | A single prompt you paste into any AI agent (Codex, Claude Code, Cursor…) to install everything safely |
| [`config/config.toml`](config/config.toml) | The 5 keys to merge into `~/.codex/config.toml` |
| [`config/agents/`](config/agents/) | The four subagent definitions |
| [`config/profiles/`](config/profiles/) | `budget` (Luna High) and `heavy` (Sol High) session profiles |
| [`AGENTS.md`](AGENTS.md) | Routing policy, delegation economics, autonomy boundaries, engineering standards — appended to `~/.codex/AGENTS.md` |
| [`docs/RATIONALE.md`](docs/RATIONALE.md) | Every decision with its benchmark evidence — including what we're *not* sure about |
| [`docs/MEASUREMENT.md`](docs/MEASUREMENT.md) | Optional: a fair A/B protocol to verify the setup on your own workload |

## Install

### Option A — one prompt (recommended)

Clone this repo, open your AI agent of choice **in the repo directory**, and paste the contents of [`INSTALL_PROMPT.md`](INSTALL_PROMPT.md). The agent backs up your config, merges (never overwrites), and verifies.

### Option B — manual (10 minutes)

1. Back up `~/.codex/config.toml` and `~/.codex/AGENTS.md`.
2. Merge the 5 keys from `config/config.toml` into your `~/.codex/config.toml`. **Root keys must come before any `[table]` header** — a root key written after `[agents]` silently becomes `agents.<key>` and does nothing.
3. Copy `config/agents/*.toml` → `~/.codex/agents/`.
4. Copy `config/profiles/*.toml` → `~/.codex/` (usage: `codex --profile budget`).
5. **Append** this repo's `AGENTS.md` sections to your `~/.codex/AGENTS.md` — do not replace the file.
6. Restart Codex — agents load on startup.

### Verify

New session → *"Where is function X defined and who calls it?"* should route to `explorer`; a small feature request should route to `worker`. Inspect threads with `/agent`.

### Daily driving

| Session | Command |
|---|---|
| Normal day | `codex` (Terra Medium + routing) |
| Architecture day / hard debugging | `codex --profile heavy` (Sol High) |
| Quota running low / small tasks | `codex --profile budget` (Luna High) |
| Rare hardest single problem | picker → Sol XHigh for that session |

## Optional: full autonomy & Windows fixes

Two opt-ins that are **not** part of the default install:

- **No approval prompts:** `approval_policy = "never"` + `sandbox_mode = "danger-full-access"` (commented in [`config/config.toml`](config/config.toml)). Understand the trade first: the only remaining guardrail is the behavioral policy in AGENTS.md — there is no technical barrier left. Approval prompts from subagent threads disappear, which is exactly why people want it.
- **Windows: recurring sandbox/elevation prompts?** The recommended `elevated` native sandbox sometimes fails to install (`codex-windows-sandbox-setup.exe not found` in the logs) and re-prompts every session. Official fallback: `[windows] sandbox = "unelevated"`.

## Extra cost levers

- **Trim MCP tool surfaces.** Allowlisting a 93-tool MCP server down to 3 tools cut per-turn schema overhead from ~55K to ~3K tokens. The single biggest lever most people miss.
- **Keep sessions stable.** Cached input is 10× cheaper — continue in the same thread; don't churn your prompts or AGENTS.md.
- **State each instruction exactly once.** OpenAI's own evals: leaner prompts scored 10–15% better while cutting tokens 41–66%.
- Keep `agents.max_depth = 1` (recursive fan-out is a quota explosion), Fast mode off, Chronicle off.

## Honesty notes

- Benchmark figures: Artificial Analysis, DeepSWE v1.1, MRCR v2, official rate card — as of **July 2026**. Models and pricing move; re-verify before trusting blindly.
- Some verdicts rest on single sources — they're flagged in [RATIONALE](docs/RATIONALE.md).
- "Optimal" is workload-dependent. This is a strong, safe default; [MEASUREMENT.md](docs/MEASUREMENT.md) shows how to prove it on *your* tasks.
- GPT-5.6 Sol scored the highest measured reward-hacking rate in METR's predeployment evals. The "evidence over optimism" rules in the agent instructions exist for a reason — verify critical changes yourself.

## License

[MIT](LICENSE)
