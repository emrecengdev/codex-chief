# The Install Prompt

Clone this repository, open any capable AI coding agent (Codex, Claude Code,
Cursor, ...) **in the repository directory**, and paste everything below the
line. The agent will install codex-chief safely: backup first, merge — never
overwrite — and verify.

---

```text
You are installing "codex-chief", a budget-disciplined multi-agent
configuration for OpenAI Codex. The source files are in the current
directory (this repository). The target is the user's Codex home directory:
$CODEX_HOME if set, otherwise ~/.codex (on Windows: %USERPROFILE%\.codex).

Follow these steps exactly, in order. Show the user every diff before
saving. Never overwrite an existing file wholesale — merge.

STEP 0 — Backup.
Create a timestamped backup directory next to the codex home (e.g.
~/.codex-backup-<date>/ — on Windows: %USERPROFILE%\.codex-backup-<date>\)
and copy the current config.toml, AGENTS.md, and the agents/ directory into
it (those that exist). This backup directory is the one permitted exception
to the "no writes outside codex-home" constraint below. Tell the user the
backup path. Do not proceed without a successful backup.

STEP 1 — Merge config keys.
Read the target config.toml. Merge in ONLY these keys from ./config/config.toml:
  - model = "gpt-5.6-terra"
  - model_reasoning_effort = "medium"
  - model_auto_compact_token_limit = 200000
  - agents.max_threads = 4
  - agents.max_depth = 1
Rules:
  - The three root keys MUST be placed before any [table] header in the
    file. In TOML, a root key written after a [table] header silently
    becomes a member of that table and will not work. Prefer grouping
    model_auto_compact_token_limit next to the other model_* root keys.
  - "Already exists" means AT ROOT SCOPE — in the region before the first
    [table] header. Keys named model or model_reasoning_effort may also
    appear inside profile tables like [profiles.plan]; those are separate,
    profile-scoped settings — do NOT modify, move, or count them as
    duplicates. Only update the root-scope occurrence, or add the key to
    the root region if absent there.
  - Preserve every other existing key, table, comment, profile, provider,
    and plugin setting exactly as it is.
  - Do NOT add model_verbosity, model_reasoning_summary, or
    tool_output_token_limit — they are already model defaults.

STEP 2 — Install agent definitions.
Copy ./config/agents/explorer.toml, explorer_xl.toml, worker.toml, and
judge.toml into <codex-home>/agents/ (create the directory if needed).
If a file with the same name exists, show the user both versions and ask
before replacing.

STEP 3 — Install profiles.
Copy ./config/profiles/budget.config.toml and heavy.config.toml into
<codex-home>/ (they sit next to config.toml, not inside agents/).
Same conflict rule as step 2.

STEP 4 — Append AGENTS.md sections.
Read the target <codex-home>/AGENTS.md (create it if missing). APPEND the
five sections from ./AGENTS.md of this repo (## Mindset, ## Routing,
## Autonomy and approval, ## Engineering standards, ## Output) to the END
of the existing file. Skip ONLY the leading comment block — the few lines
at the very top of the repo file that begin with a single '#' followed by
a space; the '##' markdown section headings are content and MUST be kept.
Do not delete or modify any existing content in the target file. Conflict
check is by exact heading text: only if the target file already contains a
heading exactly matching one of the five above, show the user both
sections and ask which to keep.

STEP 5 — Verify.
  a. Re-read the final config.toml and confirm: the three root keys appear
     before the first [table] header; [agents] contains max_threads = 4 and
     max_depth = 1; all pre-existing settings are still present.
  b. Confirm the four agent files and two profile files exist at their
     targets.
  c. Print a short summary: what was changed, what was added, the backup
     path, and this reminder: "Restart Codex or open a new session — agents
     load on startup. Test routing with a read-only question (should route
     to explorer) and a small coding task (should route to worker). Use
     'codex --profile heavy' for judgment-heavy sessions and
     'codex --profile budget' when quota runs low."

Constraints for you, the installing agent:
  - If any step's target state is ambiguous, stop and ask; do not guess.
  - Make no changes outside <codex-home> and the backup directory.
  - If the user's config uses a model provider other than OpenAI defaults
    (custom model_provider entries), do not touch those entries; only set
    the five keys listed in step 1.
```
