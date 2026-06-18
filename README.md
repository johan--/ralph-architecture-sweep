# ralph-architecture-sweep

[![Skills](https://skills.sh/b/Aijo24/ralph-architecture-sweep)](https://skills.sh/Aijo24/ralph-architecture-sweep)
[![npm version](https://img.shields.io/npm/v/ralph-architecture-sweep.svg)](https://www.npmjs.com/package/ralph-architecture-sweep)
[![license: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A [Claude Code](https://docs.claude.com/en/docs/claude-code) **skill** that drives [**ralph**](https://github.com/frankbria/ralph-claude-code) through an **architecture-deepening sweep** and files the findings as **vertical-slice issues** — over your whole codebase or a chosen part.

It runs the "deepening" methodology — deep vs shallow modules, the **deletion test**, seams — as an **analysis-only** pass, **delta-aware** (skips already-filed/shipped seams), and writes ready-to-grab issues under `.scratch/`. Optionally chain ralph to implement them.

## Requires

- **ralph** — [frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) (MIT), installed in your repo (`ralph-claude-code/`, `.ralphrc`, `.ralph/`).
- **Claude Code**.

## Install

**With the `skills` CLI (recommended)** — run from your project root:

```bash
npx skills add Aijo24/ralph-architecture-sweep
```

Installs to `.claude/skills/ralph-architecture-sweep/`, so you invoke it as `/ralph-architecture-sweep`. This is the install path tracked by [skills.sh](https://skills.sh/Aijo24/ralph-architecture-sweep).

**As a Claude Code plugin** — if you want `/plugin` version tracking + updates:

```text
/plugin marketplace add Aijo24/ralph-architecture-sweep
/plugin install ralph-architecture-sweep@aijo24
```

Plugin-installed skills are **namespaced by the plugin**, so you invoke this one as `/ralph-architecture-sweep:ralph-architecture-sweep` (vs the plain `/ralph-architecture-sweep` from every other path). The trade-off buys you `/plugin update` and marketplace discoverability.

**Fallback: npx installer** — for setups without the `skills` CLI or plugin support:

```bash
npx ralph-architecture-sweep            # → ./.claude/skills/ralph-architecture-sweep
npx ralph-architecture-sweep --global   # → ~/.claude/skills/ (all projects)
npx github:Aijo24/ralph-architecture-sweep   # straight from GitHub, no npm account needed
```

**Manual** — copy the skill folder yourself:

```bash
cp -R skills/ralph-architecture-sweep <your-repo>/.claude/skills/
```

Reload the session so the skill is picked up, then invoke `/ralph-architecture-sweep` (or `/ralph-architecture-sweep:ralph-architecture-sweep` if you went the plugin route).

## Use

`/ralph-architecture-sweep` (or `/ralph-architecture-sweep <path-or-subsystem>` to skip the scope question) — it:

1. **resolves the scope** — the whole codebase or a chosen part (a subsystem / package / directory); asks only if ambiguous, defaults to the whole codebase unattended,
2. forks a `ralph/*` **worktree off your remote default branch** (so it sees shipped refactors and won't re-find them) — or **resumes** an interrupted sweep from its per-area checkpoints,
3. **sweeps** for deepening candidates (analysis-only, delta-aware) — robustly, via one sub-agent per area with a one-retry policy,
4. **verifies every candidate independently** — replays the evidence, re-argues the deletion test, dedupes seams across areas; unverifiable candidates are dropped, not filed,
5. writes **vertical-slice issues** + a per-area PRD under `.scratch/`, lint-checked against the issue template, committing per area.

Nothing is pushed — you review the issues before merging or implementing.

See [`SKILL.md`](skills/ralph-architecture-sweep/SKILL.md) for the full pipeline + guardrails, and [`EXTENDING.md`](skills/ralph-architecture-sweep/EXTENDING.md) for the optional, project-specific implement & deploy phases.

## Configure for your project

The skill uses generic placeholders — adapt:

- your **remote default branch** (e.g. `main`),
- your **area split** (subsystems / packages / directories — one ralph loop each),
- your **gates** (typecheck / test / lint) for the optional implement phase,
- your **decision record** — a `CONTEXT.md` and/or ADR log of decisions the sweep must not re-propose (no record yet? Matt Pocock's `domain-modeling` skill writes one inline; until you have one the guardrail is a no-op).

Keep infrastructure (servers, SSH, secrets, domains, deploy commands) **out of the skill** — those live in your own project.

## How it works

- **Deletion test** — imagine deleting a module: if complexity vanishes it was a pass-through (don't extract it); if complexity reappears across N callers it earns its keep (candidate).
- **Seam** — a place you can change behaviour without editing there; a *real* one is one rule across N call sites, or 2+ adapters over the same data (a single adapter is only a hypothetical seam).
- **Vertical-slice issue** — the deep module + every call site repointed + tests at the new interface + the old copies deleted, as one independently-grabbable ticket.
- **Robust driving** — headless ralph commits only at the end of an iteration, so a long analysis call that drops loses everything; this skill drives the sweep via short sub-agents instead.
- **Propose / verify split** — a fresh verifier (that never sees the proposer's reasoning, only its claims) replays each candidate's evidence and re-argues the deletion test; "Strong" must be earned (≥ 3 independent call sites, or 2+ adapters over the same data).

## Credits

- **ralph** by Frank Bria — [frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) (MIT): the autonomous loop this skill drives.
- **Matt Pocock's skills** — [mattpocock/skills](https://github.com/mattpocock/skills): the methodology this pipeline runs. The deepening vocabulary (deep vs shallow modules, the deletion test, seams) is from **`codebase-design`**; the vertical-slice issue format is from **`to-issues`**; the scan-and-deepen workflow is the headless analogue of **`improve-codebase-architecture`**; and the ADR / `CONTEXT.md` discipline behind the "respect your ADRs" guardrail mirrors **`domain-modeling`**. These upstream skills are now interactive and **user-invoked** (they present reports and grill you, and would hang under headless `claude`), so this pipeline **inlines their methodology and runs it headlessly via ralph rather than calling them**.
- **John Ousterhout**, *A Philosophy of Software Design* — the underlying ideas (deep modules, the deletion test) the deepening vocabulary builds on.

This repo is original packaging that orchestrates ralph through that methodology; it does not redistribute either project's code.

## License

MIT — see [LICENSE](LICENSE).
