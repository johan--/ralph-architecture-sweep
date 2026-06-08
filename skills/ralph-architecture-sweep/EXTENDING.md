# Extending — implement & deploy (optional, project-specific)

The sweep (Phases 0–2) is portable. Implementing the filed issues and shipping them is **specific to your stack** — wire these with your own commands. The shapes below are sketches, not recipes.

> **Do not hardcode infrastructure into a shared skill** — no server addresses, SSH details, secrets, internal domains, ports, or auth-token recipes. Keep all of that in your own (private) project config.

## Implement loop

After you review the filed issues, you can chain ralph to implement them:

- Fork an impl worktree off the sweep branch (so it has the issues), bootstrap your deps.
- Write an implement `.ralph/PROMPT.md` / `fix_plan.md`: **one ticket per loop**, **Strong-first**, behaviour-preserving (deletion test), each a vertical slice (the deep module + every call site repointed + tests at the new interface + the old inline copies deleted), then **your gates**, a changelog entry, and a conventional commit staging only changed files (never `git add -A`).
- Launch `ralph-claude-code/ralph_loop.sh --live --verbose --auto-reset-circuit`; monitor per-ticket commits (`git log` / `.ralph/live.log`).
- **Per-call timeout (`CLAUDE_TIMEOUT_MINUTES` in `.ralphrc`):** ralph is not strictly one-ticket-per-loop — a single session can power through several tickets and get **cut mid-edit** on a later one (the iteration times out; this is NOT a crash or a drop, and `status.json` may still read `running`). **Finish the cut tail inline**, re-gate, commit — do NOT relaunch into a dirty tree.

## Gates

Run **your** project's gates (typecheck / tests / lint / type-of-build) and treat any failure as blocking — even one outside the current ticket's scope. **Pin any environment-dependent baseline noise** (tests that fail for reasons unrelated to the change, e.g. a shared/dirty test DB) so you can tell a real regression from the baseline rather than blaming the refactor.

## Review & deploy

Optionally run a code-review pass and apply the safe fixes, then re-gate. Deploy through **your** pipeline (your CI, your environments). Keep every environment detail project-side and out of this skill. A good default guardrail: never push to a production branch without an explicit human go-ahead.
