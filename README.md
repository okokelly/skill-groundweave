# Groundweave

Research groundwork for long-form writing. Produces a three-document knowledge loom — Glossary, Key People, and Reading List — woven together through cross-reference discovery. Subagent farms do the research; the interactive agent integrates, judges, and delivers. The reinforce stage runs as a true self-driving loop you can leave unattended.

## What it does

Before drafting any substantive article, Groundweave runs a subagent farm to research every source, person, and concept you've flagged. It then weaves the findings into three interconnected documents:

- **Glossary** — terms with definitions, origin attribution, common misconceptions, mapped to article sections
- **Key People** — individuals with bios, relevance, must-read articles, concepts they coined
- **Reading List** — articles with verified URLs, section mapping, and "how to use" annotations

These three documents cross-reference each other. Every glossary term links to its originator and canonical article. Every person links to their must-read article. Every article cites its authors and key terms. The result is a research package where you can enter from any entry and navigate the entire knowledge graph.

After initial research, Groundweave runs reinforce passes — subagents dispatched to mine each document for clues about the other two, iterating until no new connections are found. The reinforce stage is a self-driving loop: the interactive agent dispatches each pass via background delegation, is re-woken automatically when it completes, counts what's new, and dispatches the next pass — with no manual step in between. The loop runs under a contract (budget ceilings, exit conditions, a circuit breaker) so an unattended run stops cleanly instead of burning your search quota overnight.

## Quick Start

Groundweave runs as a skill on [Hermes Agent](https://github.com/NousResearch/hermes-agent). It requires Brave Search API access for web research.

```bash
# Install the skill
cp SKILL.md ~/.hermes/skills/groundweave/SKILL.md

# Or install via Hermes skills hub
hermes skills install groundweave
```

In a Hermes session, load the skill and start with a locked article outline:

```
skill_view('groundweave')

"I want to groundweave an article about [topic].
 Project folder: ~/writings/2026.06 My Article/
 Article plan is locked at article-plan-v1.md.
 Must-include: [names, sources, frameworks].
 Language: en."
```

Groundweave runs in two modes:
- **Auto-run** — the reinforce loop dispatches and continues itself, hands-off, until it hits an exit condition or budget ceiling. You review at gates.
- **Manual gate** — you approve each pass before it continues.

Either way, set a **loop budget** at pre-flight (max passes, wall-clock, search queries, consecutive failures). Auto-run won't start without one.

## Architecture

```
Phase -1: Pre-launch (plan audit → pre-flight + loop budget → config → launch)
    ↓
Launch 1: Must-Include Farm (offloaded /background dispatch, ~15-20 subagents)
    ↓
Integration 1: Raw outputs → glossary/people/reading-list v1 (interactive)
    ↓
Gate 1: User reviews + decides
    ↓
Reinforce Loop: in-session background passes → count new → budget/exit check
                → continue or STOP (exhaust / diminish / budget / circuit breaker)
    ↓
Final Integration: independent verifier fact-check → quality gates → v2 → delivery
```

Two dispatch paths. **Launch 1** (the heavy must-include farm) is offloaded to a separate `/background` agent that signals via a file — keeping the interactive context clean. **Reinforce passes** are dispatched in-session by the interactive agent via background delegation, so their completion re-enters the conversation automatically and the loop self-drives. In both cases: subagents research, the interactive agent integrates and judges, and a fresh-context verifier (not the integrator) does the final fact-check.

## Dependencies

- [Hermes Agent](https://github.com/NousResearch/hermes-agent)
- Brave Search API key (free tier: 2,000 queries/month)
- The `search.py` script from Hermes's web-search skill: `~/.hermes/skills/research/web-search/scripts/search.py`

## Configuration

```yaml
# Required in ~/.hermes/config.yaml
agent:
  max_turns: 400
approvals:
  mode: smart
delegation:
  max_concurrent_children: 3   # subagents running at once; flat fan-out queues past this
  max_async_children: 3        # concurrent background (reinforce) batches
  max_spawn_depth: 2
```

## Latest Update — v2.0.0 (2026-06-30)

This release turns the reinforce stage from a manually-stepped sequence into a genuine self-driving loop, and hardens it for unattended runs. Framed against the Prompt → Context → Harness → **Loop** progression.

- **True auto-run loop.** Reinforce passes are now dispatched in-session via `delegate_task(background=true)`; the completion re-enters the conversation automatically, so the loop continues itself with no manual `/background` between passes. (Verified against the Hermes delegation API. On the stateless HTTP/WebUI it auto-falls-back to synchronous — still no manual step.) The misleading "auto-run still needs you to launch each pass" caveat is gone.
- **Loop Contract.** The reinforce loop now runs under all six dimensions — TRIGGER / SCOPE / ACTION / **BUDGET** / STOP / REPORT. A loop budget (max passes, wall-clock, search queries, consecutive failures) is declared at pre-flight and tracked each pass from real completion-event signals (`api_calls`, `duration_seconds`).
- **Circuit breaker.** Consecutive failed passes (`status != ok` / `exit_reason` timeout|error, or no completion) trip the loop: it keeps the last-good documents and escalates to you instead of silently retrying until the quota is gone.
- **Maker/verifier separation.** The final glossary fact-check now runs in a fresh-context verifier subagent — not the integrator self-reviewing its own work.
- **Subagent cap.** Launch 1 is capped at ~15-20 subagents (group related sources), since `max_concurrent_children` defaults to 3 and a flat fan-out otherwise runs in slow, expensive serial waves.
- **Security hardening.** Argument-injection fix on URL/domain checks (`--` + `--proto '=https'`), path-traversal slugify for subagent filenames, and a Security pitfalls section covering prompt injection and SSRF — all research input is treated as untrusted data, never instructions.

## Roadmap

The reinforce flywheel today is **cross-document** (each doc enriches the other two) and discovers entries by following references *outward* from what it already has. That mechanism has structural blind spots. Open directions:

### 1. Anti-bubble — surface dissent, not just consensus

Following citations and references is preferential attachment plus homophily: well-connected, mutually-agreeing voices reinforce each other, so the package drifts toward the consensus view and systematically under-samples outliers, critics, and adjacent-field outsiders. Directions:

- **A contrarian loop (Loop D)** orthogonal to A/B/C: for each key person/term, explicitly dispatch subagents to find the *strongest opposing view / sharpest critic* — "who publicly disagrees and why," not "who else agrees."
- **Diversity sampling** across communities (academic / practitioner / regulator / journalist / skeptic), instead of expanding into the densest citation cluster.
- **Outlier weighting by argument-relevance, not frequency.** A minority view should be weighted by whether the outline's thesis is *exposed if it's omitted* — never by citation count, since that is exactly what builds the bubble. Add a `contested?` / minority-view field to terms and people.
- Note: the `<5 new → diminishing` exit condition is itself a frequency cutoff that can kill outlier discovery early; outliers are rare by definition.

### 2. Intra-document reinforcement

Reinforce is currently *only* cross-document. But each document has internal, self-reinforcing structure worth mining — and worth emitting as a deliverable in its own right:

- **Key People → intellectual/social network** (mentors, collaborators, rivals, co-authors, institutions). Movements are network phenomena; expanding a person's network surfaces more relevant people.
- **Reading List → citation graph + corpus-gap detection** (a debate represented by 3 "for" and 0 "against" is a visible hole).
- **Glossary → conceptual lineage (思想的脉络)**: term B is a reaction to A; C synthesizes A and B. The genealogy is both a discovery axis and the article's narrative spine.

These could produce a **fourth artifact** — a structure map (network / citation graph / concept genealogy) — not just more entries.

### The tension between 1 and 2

Intra-document reinforcement *amplifies* the bubble: network-following stays inside a cluster, citation-following is preferential attachment. So #1 and #2 must be **co-designed** — every network/lineage expansion needs a diversity counterweight, or the result is a dense, beautiful map of a single tribe. Treat them as one workstream, not two.

### Further directions

- **Circular-sourcing detection** — trace claims to a primary source; flag when N "independent" sources all derive from one origin. Anti-bubble and fact-check in a single mechanism.
- **Relevance-weighted budget** — spend reinforce budget where outline coverage or contestedness is *thin*, not uniformly. Directly funds outlier discovery instead of over-saturating the consensus.
- **Verifier-as-grader** — have the fact-check verifier emit calibrated confidence per entry and a coverage score against the outline, turning "done" into a measurable bar.
- **Recency axis** — systematic re-verification of time-sensitive entries (current roles, latest work), beyond the one-off "wrong current status" check.
- **Cross-project entity memory** — warm-start new runs in a known domain from a persistent people/terms/sources store, instead of starting cold each time.

## License

MIT
