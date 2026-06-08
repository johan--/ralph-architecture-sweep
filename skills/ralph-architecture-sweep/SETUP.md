# Setup — scope, worktree, backlog (Phases 0–1)

## Scope → areas

Ask the user: **whole codebase** or a **chosen part**? Map the answer to **areas**, one area per ralph loop, each diffable against an existing epic. Pick areas that are coherent subsystems (a package, a layer, a directory) so each loop has a tractable, self-contained scope.

## Base off the remote default branch (refs lag!)

```
git fetch origin <default-branch>                                    # e.g. main
git rev-parse --short origin/<default-branch>                        # the base
git rev-list --left-right --count origin/<default-branch>...HEAD     # how far local lags
```

Always fork from the **remote** ref, never the local checkout (which may be far behind, missing shipped refactors). A stale-base sweep re-finds seams that are already shipped.

## Detect prior sweeps → delta

```
git ls-tree -r origin/<default-branch> --name-only -- .scratch/ | grep deepening
```

If an area already has a `…-deepening` (± `…-deepening-delta`) epic, run a **delta** sweep: the worktree (off the remote tip) sees the shipped refactors, so it can EXCLUDE them; write to `…-deepening-delta2` (bump the suffix). Tell the user "already swept N times — delta vs fresh re-sweep"; a fresh re-sweep re-finds shipped seams (waste + false candidates).

## Worktree + tooling

```
WT=<repo-parent>/ralph-<slug>-wt
git worktree add "$WT" -b ralph/<slug> origin/<default-branch>
# if ralph's tooling is untracked, copy it into the worktree:
cp -R <repo>/ralph-claude-code "$WT"/ && cp <repo>/.ralphrc "$WT"/
```

If `.ralph/PROMPT.md` + `fix_plan.md` are tracked they come with the worktree — overwrite them for this epic. An analysis-only sweep needs **no** install/deps. Clean up after with `git worktree remove --force "$WT"`.

## Backlog: inline the methodology

Interactive deepening/issue skills hang in headless `claude`, so **inline** their methodology into `.ralph/PROMPT.md` (per-iteration spec) + `.ralph/fix_plan.md` (the checkbox area list).

**Deepening:** deep vs shallow modules; a **seam** is one rule across N call sites, or 2+ adapters over the same data; the **deletion test** — delete the proposed module: complexity *vanishes* = pass-through (reject); *reappears across N callers* = earns its keep (propose). Behaviour-preserving.

**Issues (vertical slice):** one issue per candidate — *What to build* (the deep module/seam + every call site repointed + tests at the new interface + old copies deleted) / *Acceptance criteria* / *Deletion-test verdict* / *Strength* (Strong / Worth-exploring) / *Blocked by*. Plus a per-area `PRD.md` (Problem / Candidates / Out of scope).

`fix_plan.md` = one `- [ ]` per area (ralph logs "Remaining tasks: N" at startup — confirms scoping). Bake the guardrails into PROMPT.md: analysis-only (writes only under `.scratch/<area>-deepening*/`), no push, **respect your ADRs** (never re-propose a settled decision — flag, don't fork), and **re-verify every candidate against the remote tip**.

If your repo already has `.scratch/*-deepening*` epics, use them as the template — they show the proven output shape.
