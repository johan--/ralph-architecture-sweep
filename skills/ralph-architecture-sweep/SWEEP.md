# Sweep — drive analysis-only, robustly (Phase 2)

## Why not just launch headless ralph

Headless ralph (`claude -p`, JSON mode) **commits only at the END of a successful iteration**. A single area's sweep is one long call (minutes); if it drops mid-stream (e.g. an API socket-drop), the whole iteration AND its spend are discarded, and auto-reset just re-burns long calls until the circuit breaker. Short interactive calls don't have this failure mode. **So for an analysis-only sweep, drive it from the interactive session, not headless.**

## Default: fan out sub-agents (one per area)

Run the areas in parallel via the agent tool. Each sub-agent:

- operates in the **worktree** (absolute paths — it's at the remote tip, so it sees the shipped refactors),
- **reads the area's existing epic(s) first** to build the exclusion list,
- applies the **deletion test** + deep/shallow/seam vocab,
- returns ONLY genuinely-new candidates (or an explicit "zero"), each with: title, strength, **evidence** (files + symbols + call-site count, grep-verified), proposed seam, deletion-test verdict, and an exclusion check ("not a re-proposal of `<epic/issue>` because …").

Then **vet each candidate yourself** against the remote tip before writing it up — don't write a candidate a sub-agent couldn't ground. Only keep ones you'd stake the deletion test on.

## Alternative: headless ralph loop

If you want the literal hands-off loop and the API is calm:

```
# reset a stale session FIRST, in a SEPARATE invocation (it exits without looping):
bash ralph-claude-code/ralph_loop.sh --reset-session
# then run:
bash ralph-claude-code/ralph_loop.sh --live --verbose --auto-reset-circuit
```

Monitor `.ralph/live.log` + `.ralph/status.json` (`status:"completed"`). Watch for dropped long calls — stop the loop and switch to sub-agents rather than letting auto-reset churn.

## Output + commit

Write `.scratch/<area>-deepening[-delta2]/PRD.md` + `issues/NN-<slug>.md` in the vertical-slice format. Commit per area, staging ONLY those files (never `git add -A` — the worktree has untracked tooling / runtime state):

```
git -C "$WT" add .scratch/<area>-deepening-delta2
git -C "$WT" commit -m "docs(arch): <delta-vN> sweep — <area> (N new candidate(s))"
```

Mark the area `[x]` in `fix_plan.md`. **Do not push.** Report: per-area counts (Strong / Worth-exploring / zero), the branch, the worktree path, and the `.scratch/` paths — and ask whether the user wants the optional implement phase ([EXTENDING.md](EXTENDING.md)).
