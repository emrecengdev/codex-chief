# Rationale — every decision with its evidence

Sources: Artificial Analysis (AA Intelligence Index cost matrix, Coding Agent
Index), DeepSWE v1.1 leaderboard, OpenAI MRCR v2 long-context evals, the
official Codex rate card and docs (learn.chatgpt.com), and community testing
shared on X. All figures as of **July 2026**.

## The credit economics (official rate card)

Credits per 1M tokens on ChatGPT plans:

| Model | Input | Cached input | Output |
|---|---|---|---|
| gpt-5.6-sol | 125 | 12.5 | 750 |
| gpt-5.6-terra | 62.5 | 6.25 | 375 |
| gpt-5.6-luna | 25 | 2.5 | 150 |
| gpt-5.4-mini | 18.75 | 1.875 | 113 |

Two structural facts drive everything:

- **Reasoning effort does not change the per-token rate — it multiplies
  token volume.** Sol Low vs Sol Max is purely a volume difference.
- **Cached input is 10× cheaper.** Stable prompts and long-lived sessions
  are a direct quota saving.

## Why each tier

**Main agent = Terra Medium.** Official positioning ("everyday workhorse") +
AA matrix: Terra Medium scores 46 at 3.5× normalized cost vs Luna High's 46
at 4.0× — same score, cheaper in total, despite Luna's lower token price.
The main thread is also the long-lived context, which rules Luna out (see
below). Its weak DeepSWE score (35%) doesn't matter in this architecture
because implementation is delegated.

**Implementation = Luna XHigh.** DeepSWE v1.1: 57% completion at $1.54/task,
45K output tokens. The community's revised recommendation (same performance
as Terra High, 2.5× cheaper, 1.3× faster) matches the data. Extra reasoning
helps small models disproportionately.

**Judgment = Sol High.** The cost/quality knee: 69% DeepSWE at $3.47 and 37
steps (vs Sol Max: 73% at $8.39 and 61 steps). "High" is where planning,
hard debugging, and risky review get reliable.

**Escalation ceiling = Sol XHigh, not Max.** DeepSWE: 71% at $4.70 vs Max's
73% at $8.39 — ~97% of the quality at 56% of the cost. (Single-source
figure; AA doesn't publish an xhigh/max split. Flagged honestly.)

**Long context = Terra, never Luna.** MRCR v2 8-needle at 256K–512K: Luna
recall collapses to 41.3% while Terra holds 89.6% and Sol 91.5%. Shrinking
Luna's context window doesn't fix this — the task defines the context need;
capping the window just forces compaction churn and more steps.

## Why the "avoid" list

- **Ultra effort**: officially "goes beyond a single-agent run — it uses
  subagents." Children inherit the parent's model and effort, so Ultra on an
  expensive model multiplies itself. Community reports: 2 hours of Sol Ultra
  consumed 5 hours' worth of a 20x plan window. The "4 parallel subagents"
  figure circulating is NOT official — the mechanism and the quota burn are.
- **Sol Max**: +1 AA point over xhigh for ~2–3× the token volume.
- **Terra High/XHigh/Max**: dominated on the AA matrix — Sol at a lower
  effort beats each on both score and cost (e.g. Terra High 49 @ 7.2× vs
  Sol Low 49 @ 5.1×). Nuance: Terra Max matches Claude Fable 5's DeepSWE
  score at a quarter of its cost, so it's not useless — but for
  Pareto-optimal quota spend, a Sol+Luna mix beats Terra's upper efforts.
- **Luna Low/Medium for code**: 2% / 11% DeepSWE completion. Scouting only.
- **Fast mode**: 1.5× speed for 2–2.5× credits. For instant text iteration,
  `gpt-5.3-codex-spark` has its own separate Pro quota instead.
- **"Cheap model ≠ cheap task"**: Luna Max looks cheap per token but scored
  51 at 12.7× suite cost (102 steps/task) — step count, retries, and volume
  decide total cost, not the price sheet.

## The delegation economics

Official docs are explicit: subagent workflows consume MORE tokens than an
equivalent single-agent run. Delegation pays in exactly three cases:

1. **Genuine parallelism** — independent workstreams.
2. **Context hygiene** — each subagent gets a disposable context window
   (CLI ≥0.107.0), keeping bulk reads out of the expensive long-lived main
   thread and protecting its cache hits.
3. **Capability mismatch** — the task exceeds the main model's tier: a
   bounded implementation on Luna XHigh (57%) vs Terra Medium (35%) — the
   failure-retry loop of the weaker model costs more than the delegation
   overhead.

Everything else stays in the main thread.

## Prompting facts worth knowing

- OpenAI's internal evals: leaner system prompts (each instruction stated
  exactly once) scored 10–15% better while cutting tokens 41–66% and cost
  33–67%.
- Repeated approval instructions ("ask first", "do not mutate") cause
  unnecessary approval requests for safe actions — state the autonomy policy
  once.
- GPT-5.6 is more concise by default; legacy "be brief" rules may now
  over-trim.
- The routing policy has no config field — `[agents]` only carries
  thread/depth/runtime settings. Routing lives in AGENTS.md.

## Known uncertainties

- The Sol xhigh≈max claim rests on one leaderboard (DeepSWE).
- Benchmarks mix harnesses and workloads; none measures *your* task mix.
  See [MEASUREMENT.md](MEASUREMENT.md) for a fair way to test that.
- GPT-5.6 Sol had the highest measured reward-hacking rate in METR's
  predeployment evals and a higher hallucination rate than GPT-5.5 —
  hence the "evidence, not optimism" rules baked into the agent
  instructions. Verify critical changes yourself.
