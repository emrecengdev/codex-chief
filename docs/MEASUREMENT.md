# Measuring on your own workload (optional)

The setup in this repo is a strong, evidence-based default — but "optimal"
is workload-dependent. If you want proof rather than trust, here is a fair
protocol. (For most personal users, the pragmatic alternative is fine:
install, use for a week, watch your 5-hour quota windows, roll back from the
backup if unhappy.)

## Variants

Compare at least three, so you learn whether the router beats both the
expensive old way AND the simple cheap way:

- **old** — your previous setup (often Sol XHigh everywhere)
- **new** — this repo's routed system
- **plain** — Terra Medium with no custom agents and no routing

## Isolation (both kinds)

**Code:** every run starts from a fresh worktree/checkout of the same
commit. A fixed repo can't host the same task twice.

**Config:** a git worktree does NOT isolate `~/.codex` — all variants would
load the same routing and agent overrides. Give each variant its own
`CODEX_HOME`, and derive all three from the same **minimal seed**:

- Copy in: auth files, base config.toml (providers included), the
  plugin/skill surface you normally use.
- Leave out: sessions/history, SQLite telemetry, memories, WAL/SHM files,
  caches, attachments, package binaries. (A full ~/.codex copy is typically
  gigabytes of irrelevant, non-reproducible state — and copying it while
  Codex runs can snapshot SQLite mid-write.)
- Prepare seeds while Codex is fully closed. Restrict ACLs on copied
  credentials; delete the copies when done.
- Between variants, ONLY the experiment variables differ: model/effort,
  the AGENTS.md routing section, agent files, `[agents]` limits.

PowerShell (Windows): `$env:CODEX_HOME = 'C:\codex-ab\new'` per terminal.

## Tasks and budget

Three classes — search (explorer path), routine fix (worker path +
delegation threshold), hard problem (judge escalation).

- **Pilot:** 5 tasks × 3 classes × 3 variants = **45 runs** minimum (plus
  retries). Budget in runs, not tasks.
- **Final:** winners advance to ≥10 tasks per class.
- **Counterbalance the order** — don't run all of one variant back-to-back
  (server load, quota windows, and cache warmth would confound it). Rotate:
  task 1 old→new→plain, task 2 new→plain→old, ...

## Metrics (decide BEFORE measuring, never after)

1. **Quality floor:** per class, success rate may trail the old setup by at
   most **20 percentage points in the pilot, 10 in the final** (absolute
   points, not relative % — and note 5 samples can only resolve 20-point
   steps, which is why the pilot tolerance is wider). A variant below the
   floor is eliminated regardless of cost.
2. **Primary metric** among survivors:

   ```
   total credits spent (INCLUDING failed runs) / successfully completed tasks
   ```

   Failed runs count in the numerator — an error-prone system must not look
   cheap.
3. **Tie-breaker:** wall-clock time.

Measure credits as before/after deltas (`/status`), but first sanity-check
its precision — it reports remaining limits, not guaranteed per-task
accounting. If it's too coarse, compute from per-run token usage in session
logs. No concurrent Codex work during measurement.
