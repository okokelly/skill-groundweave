# Groundweave

Pre-writing research flywheel for long-form articles. Give it an outline — it researches, cross-references, and weaves everything into three documents: Glossary, Key People, and Reading List. Runs in the background, you review at checkpoints. The name: groundwork research, woven together.

## What it does

Before drafting any substantive article, Groundweave runs a subagent farm to research every source, person, and concept you've flagged. It then weaves the findings into three interconnected documents:

- **Glossary** — terms with definitions, origin attribution, common misconceptions, mapped to article sections
- **Key People** — individuals with bios, relevance, must-read articles, concepts they coined
- **Reading List** — articles with verified URLs, section mapping, and "how to use" annotations

These three documents cross-reference each other. Every glossary term links to its originator and canonical article. Every person links to their must-read article. Every article cites its authors and key terms. The result is a research package where you can enter from any entry and navigate the entire knowledge graph.

After initial research, Groundweave runs reinforce passes — subagents dispatched to mine each document for clues about the other two, iterating until no new connections are found.

## Quick Start

Groundweave runs as a skill on [Hermes Agent](https://github.com/NousResearch/hermes-agent). It requires Brave Search API access for web research.

```bash
# Install the skill
cp -r groundweave/ ~/.hermes/skills/groundweave/
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
- **Auto-run** — reinforce passes run automatically. You review at gates.
- **Manual gate** — you approve each pass before it continues.

## Architecture

```
Phase -1: Pre-launch (plan audit → pre-flight → config → launch)
    ↓
Launch 1: Must-Include Farm (background subagent dispatch, 1-3h)
    ↓
Integration 1: Raw outputs → glossary/people/reading-list v1 (interactive)
    ↓
Gate 1: User reviews + decides
    ↓
Reinforce Loop: Subagent passes → count new → exhaust/diminish/continue
    ↓
Final Integration: Fact-check → quality gates → v2 → delivery
```

Background agents dispatch subagents. Interactive agents integrate, judge, and deliver. Never cross the streams.

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
  max_concurrent_children: 3
  max_spawn_depth: 2
```

## License

MIT
