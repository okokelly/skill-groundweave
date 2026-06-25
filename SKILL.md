---
name: groundweave
description: "Research groundwork for long-form writing — three-document knowledge loom (Glossary ↔ Key People ↔ Reading List) woven by subagent farms + interactive integration. Background agents dispatch. Interactive agent integrates."
version: 3.0.0
---

# Groundweave: Research Groundwork Before Drafting

Groundweave produces the three-document research foundation every substantive article needs: a **Glossary**, a **Key People** map, and a **Reading List** — woven together through cross-reference discovery. It is designed for the user to hand off research to background subagent farms and review at key gates, rather than sit in real-time iterative conversation.

## When to Use

- Writing a substantive article or series (3,000+ words)
- Entering a new domain and needing structured foundational research
- Want to hand off research to run asynchronously and review at gates
- Have a locked outline and a list of must-include sources/people/angles

## What It Produces

Three documents, all mapped to the article outline's sections:

**Glossary:** Terms with definition, why it matters to specific sections, common misconception, origin attribution. Organized by article sections. Each term links to the key person who coined/popularized it and the canonical article about it.

**Key People:** Individuals with current role, relevance to article sections, must-read article, concepts they originated. Organized by category (Scholar, Practitioner, Analyst, Regulator).

**Reading List:** Articles/papers with exact URL, verification status (✅/⚠️/❌), specific section mapping, and a "How to Use" column stating exactly how each article serves the argument.

These three documents cross-reference each other. Every glossary term points to a key person and a reading list entry. Every person points to a reading list entry. Every reading list entry cites authors and terms. This embedded cross-referencing is the foundation — equivalent to one pass of the three-document reinforce flywheel.

---

## Architecture

```
╔══════════════════════════════════════════════════════════╗
║ PHASE -1: PRE-LAUNCH (interactive, ~15min)             ║
║ Plan audit → pre-flight → config → marker → auto-run?  ║
║ Write Launch 1 prompt to {project}/launch-1.txt        ║
║ User runs: /background Read {project}/launch-1.txt     ║
╚══════════════════════════════════════════════════════════╝
                         │
                         ▼
╔══════════════════════════════════════════════════════════╗
║ LAUNCH 1: MUST-INCLUDE FARM (background, 1-3h)         ║
║ Dispatch subagents for every must-include source.       ║
║ Writes raw outputs to subagent/. No integration.        ║
║ On completion: writes SIGNAL file {project}/.launch-1-done ║
╚══════════════════════════════════════════════════════════╝
                         │
           (interactive agent polls SIGNAL file)
                         ▼
╔══════════════════════════════════════════════════════════╗
║ INTEGRATION 1 (interactive, ~20-30min)                 ║
║ Read subagent outputs → build v1 → URL verify →         ║
║ DECISIONS_NEEDED.md → Gate 1 delivery (inline)          ║
╚══════════════════════════════════════════════════════════╝
                         │
                         ▼
              🚦 GATE 1: User reviews + decides
                         │
                         ▼
╔══════════════════════════════════════════════════════════╗
║ HANDOFF (interactive)                                  ║
║ Recover context → parse decisions → gate-1-decisions.md ║
╚══════════════════════════════════════════════════════════╝
                         │
                         ▼
╔══════════════════════════════════════════════════════════╗
║ REINFORCE LOOP (auto-run or manual-gate)               ║
║                                                        ║
║  ┌─ Launch N: Reinforce Pass (background, 1-2h)        ║
║  │  Dispatch CATEGORY-GROUPED subagents.                ║
║  │  Write SIGNAL: {project}/.launch-{N}-done             ║
║  └───→ Interactive: count new entries, report           ║
║         If exhaust or diminish → break                  ║
║         If auto-run → loop to next pass                 ║
║         If manual → ask user → loop if yes              ║
╚══════════════════════════════════════════════════════════╝
                         │
                         ▼
╔══════════════════════════════════════════════════════════╗
║ FINAL INTEGRATION (interactive, ~15-20min)             ║
║ Aggregate reinforce outputs → fact-check → quality      ║
║ gates → v1→v2 → Gate N delivery (inline)               ║
╚══════════════════════════════════════════════════════════╝
```

### Why Subagent Farms + Interactive Integration

Background agents dispatch subagents reliably. They cannot orchestrate multi-phase research, integrate documents, self-assess quality, or reliably deliver results. Interactive agents excel at integration, judgment, and delivery.

**SIGNAL files replace delivery.** Background launches write `.launch-{N}-done` files. The interactive agent polls these.

---

## Phase -1: Pre-Launch (Interactive)

### Step -1.1: Plan Audit — Q→Section Coverage

Read the article plan. Run the coverage check:

1. List every question the user asked (numbered, as finalized).
2. For each question, list the section(s) in the outline that answer it.
3. If any question has zero sections assigned → **stop and flag the gap.** Do not launch.

### Step -1.2: Pre-Flight Checklist

| # | Item | Format |
|---|------|--------|
| 1 | Project folder | Absolute path, locked article plan inside |
| 2 | Language | `en` / `zh` / `mixed` |
| 3 | Must-include list | People, sources, angles, constraints (empty allowed) |
| 4 | Scope boundaries | 3-5 in-scope, explicit out-of-scope |

### Step -1.3: Reinforce Mode

Ask: "Auto-run reinforce passes to exhaustion, or ask at each gate?"

- **Auto-run (recommended):** Interactive agent runs reinforce loop without pausing. Reports after each pass. **Note:** User must still manually run each `/background Read {project}/launch-{N}.txt`.
- **Manual gate:** User approves each reinforce pass individually.

Default: auto-run unless user explicitly wants manual.

### Step -1.4: Verify Configuration

Check `max_turns` ≥ 400 and `approvals.mode` = "smart" in `~/.hermes/config.yaml`, or use `hermes config show`. Fix if needed.

### Step -1.5: Domain Accessibility Probe

Before writing launch prompts, probe every must-include domain:

```bash
for url in "https://domain1.com" "https://domain2.com"; do
  curl -sL --max-time 10 -o /dev/null -w "HTTP %{http_code} | Time: %{time_total}s\n" "$url"
done
```

Flag 403/blocked domains immediately. Workarounds: Substack API for Substack authors, Brave Search for syndicated copies, ask user for alternatives.

### Step -1.6: Write Pre-Flight + Marker

Write `pre-flight-v1.md` to project folder. Write marker: `echo "[path]" > ~/.hermes/.groundweave-active`.

### Step -1.7: Write Launch Prompt + Launch

Write the Launch 1 prompt to `{project}/launch-1.txt`. Tell user:

```
Run: /background Read and execute the full prompt at {project}/launch-1.txt.
Load skill_view('groundweave') before starting.

Expected: 1-3 hours. I'll check the SIGNAL file and let you know when it's done.
```

---

## Launch 1: Must-Include Farm (Background)

ONE task: dispatch deep-dive subagents. No integration. No reinforce. No quality gates. No delivery.

### Subagent Dispatch Plan

1. One subagent per must-include person/source from pre-flight.
2. One subagent per article plan section needing bootstrap research.
3. Dispatch ALL subagents at once — system's `max_concurrent_children` (default 3) auto-queues them.
4. Every subagent writes to `subagent/<name>-v0.md`.

### Completion Signal

Write empty file `{project}/.launch-1-done`. The interactive agent polls for this.

### Launch 1 Prompt Template

Write to `{project}/launch-1.txt`:

```
Call skill_view('groundweave') before starting.

PROJECT: {absolute path}
Read pre-flight-v1.md for must-include list and scope.
Read article-plan-v1.md for section structure.

YOUR ONLY JOB: Dispatch deep-dive subagents.
- One per must-include person/source from pre-flight
- One per article plan section needing bootstrap research
- Dispatch ALL at once (system handles queueing)
- Write to {project}/subagent/<name>-v0.md

SEARCH: Every subagent MUST use Brave Search API.
python3 ~/.hermes/skills/research/web-search/scripts/search.py "query" --count N
DO NOT use web_search. All consumer engines blocked.

AFTER ALL SUBAGENTS COMPLETE:
Write empty file: {project}/.launch-1-done

DO NOT:
- Integrate outputs into any document
- Run reinforce loops
- Quality-check output
- Write glossary, key-people, or reading-list files
- Deliver any message to the user

Your only output: subagent raw files + the .launch-1-done SIGNAL.
```

---

## Integration 1: Raw Outputs → v1 (Interactive)

### Step 1: Verify Completion

Check `.launch-1-done` exists. Count subagent/ files. Accept partial output — flag missing sources.

### Step 2: Read All Subagent Outputs

Read every file in subagent/. Extract: terms → glossary, people → key-people, articles → reading list.

### Step 3: Build v1 Documents

Write three documents with cross-references embedded:

**glossary-v1.md:**
```markdown
### Term Name (English Original)
**Definition:** One paragraph.
**Why it matters (§X.X):** One sentence mapping to article section.
**Common misconception:** One sentence.
**Origin:** Who coined/popularized (linked to Key People entry name).
**Canonical source:** Title — URL
```

Organize by category groups (5-8 categories of 3-5 terms each). If a category has only 1 term, merge into a broader one.

**key-people-v1.md:**
```markdown
### Name
**Now:** Current role. Past notable roles.
**Why matters (§X.X):** One paragraph mapping to article section.
**One must-read:** "Exact Article Title" — URL
**Terms originated:** [term name], [term name]
```

Organize by category: Scholar, Practitioner, Analyst, Regulator.

**reading-list-v1.md:**
```markdown
| # | Article | Author | URL | Status | How to Use |
|---|---------|--------|-----|--------|------------|
| N | Exact Title | Name | URL | ✅ | Supports argument X in §Y.Y / Provides data for Z / Contrast view for W |
```

Organize by article section. Every entry has a non-empty "How to Use" column.

### Step 4: URL Verify

```bash
xargs -I {} -P 10 curl -sL --max-time 10 -o /dev/null -w "{}: %{http_code}\n" {} < /tmp/urls.txt
```

Mark ✅ (200), ⚠️ (403 geoblocked), ❌ (dead). Reject homepage entries — every entry must be a specific article URL.

### Step 5: DECISIONS_NEEDED.md

```markdown
# Decisions Needed — Gate 1

## 🔴 Blockers (must resolve before next pass, ≤5)
- [ ] **Decision 1:** [Question]. Options: A / B. **My recommendation:** [X] because [reason].

## 🟡 Uncertainties (my best guess)
- **Assumption:** I assumed [X]. If wrong → sections [Y, Z] need revision.

## 🟢 Auto-Resolved (for awareness)

## 📊 Progress Snapshot
- Glossary: N terms | Key People: N | Reading List: N articles (X verified)
- Exit condition: [TBD]

## 🔍 Gaps I Can't Fill
- **[Name/Source]** — searched: Brave, Wikipedia, domain probing. NEED HUMAN.
```

Rules: state impact on specific sections, always include a recommendation, keep blockers under 5.

### Step 6: Gate 1 Delivery

Deliver INLINE (not file paths): full DECISIONS_NEEDED.md content + glossary top 10 + key-people top 10 + reading list top 15 with verification status. File paths as backup.

### Step 7: Prepare Reinforce

Write `reinforce-state.md`: baseline list of all v1 entries for new-entry counting.

```markdown
# Reinforce State — Pass 0 (baseline)

## Glossary terms: [count] — [list term names]
## Key people: [count] — [list names]
## Reading list: [count] — [list titles]
```

---

## Gate 1 + Handoff (Interactive)

User reviews → replies with decisions → agent recovers context → reads DECISIONS_NEEDED.md → parses decisions → detects contradictions → writes gate-1-decisions.md → confirms with user.

Three-tier context recovery:
1. `session_search(query="Groundweave Gate 1", limit=3)` — find delivery message with project path.
2. `read_file('~/.hermes/.groundweave-active')` — marker file.
3. `search_files(pattern='DECISIONS_NEEDED.md', target='files', path='~/')` — most recently modified.

After confirmation: prepare Launch 2 (first reinforce pass).

---

## Reinforce Passes

### The Reinforce Flywheel

Each pass MUST run three cross-reference loops:

- **Loop A: Glossary → Key People + Reading List.** Who coined each term? (→ add to Key People). What's the canonical article? (→ add to Reading List with verified URL).
- **Loop B: Key People → Reading List + Glossary.** Most relevant article? (→ add to Reading List). Terms originated? (→ add to Glossary).
- **Loop C: Reading List → Key People + Glossary.** Who wrote this? (→ add to Key People). Specialized terms? (→ add to Glossary).

Subagents are **grouped by document category**, not one-per-entry (~15 per pass, not ~100).

### Pass 1 (Launch 2): All current entries.

### Pass 2+ (Launch 3+): ONLY entries newly discovered in the previous pass. Read from `reinforce-targets.md`:

```markdown
# Reinforce Targets — Pass {N}

## Glossary (new terms to research)
- [Term name] — need: origin/coiner, canonical article URL

## Key People (new people to research)
- [Name] — need: most relevant article, network/institution

## Reading List (new articles needing author bios)
- [Article title] by [author if known] — need: author background
```

### Reinforce Launch Templates

Write to `{project}/launch-{N}.txt`.

**Pass 1 version:**
```
Call skill_view('groundweave') before starting.

PROJECT: {absolute path}
PASS: 1 (reinforce)

YOUR ONLY JOB: Dispatch CATEGORY-GROUPED research subagents.
For each category in glossary-v1.md: one subagent to find origin/coiner +
canonical article for all terms in that category.
For each category in key-people-v1.md: one subagent to find most relevant
article + network for all people in that category.
For each section in reading-list-v1.md: one subagent to find author bios
for entries lacking them.

Dispatch ALL subagents at once. Write to {project}/subagent/reinforce-p1/<category>-v0.md.

SEARCH: Brave Search API. Every subagent context MUST include the command.

AFTER ALL SUBAGENTS COMPLETE: Write empty file {project}/.launch-2-done.
DO NOT: integrate, count, judge, or deliver messages.
```

**Pass N version (N ≥ 2):**
```
Call skill_view('groundweave') before starting.

PROJECT: {absolute path}
PASS: {N} (reinforce)

Read {project}/reinforce-targets.md for the EXACT list of entries to research.
ONLY dispatch subagents for those entries. Group by category.

Dispatch ALL subagents at once. Write to {project}/subagent/reinforce-p{N}/<name>-v0.md.

SEARCH: Brave Search API. Every subagent context MUST include the command.

AFTER ALL SUBAGENTS COMPLETE: Write empty file {project}/.launch-{N}-done.
DO NOT: integrate, count, judge, deliver messages, or research non-target entries.
```

### Reinforce Gate (Interactive)

After `.launch-{N}-done` appears:

1. Read all outputs in `subagent/reinforce-p{N}/`.
2. Compare against `reinforce-state.md`.
3. Count genuinely NEW entries per document.
4. Update `reinforce-state.md`.
5. Report: "Pass {N}: +X glossary, +Y people, +Z articles."

**Exit conditions:**
- ALL THREE zero new → EXHAUSTED. Break.
- Total new across all three < 5 → DIMINISHING. Break.
- 6 passes completed → DIMINISHING. Break.

**Auto-run:** write `reinforce-targets.md`, write `launch-{N+1}.txt`, tell user to run it.
**Manual:** ask user before continuing.

---

## Final Integration (Interactive)

1. Aggregate all reinforce pass outputs into v1 documents.
2. **Glossary fact-check** (see below).
3. Quality gates: sequential numbering (verify 1–N, renumber if collisions), internal consistency (cross-check contradictions), how-to-use column (every entry mapped).
4. Copy v1 → v2. Move v1 to `_arc/`.
5. Write STATUS.md, DECISIONS_NEEDED.md v2 (final).
6. Gate N delivery inline.
7. Delete `~/.hermes/.groundweave-active` on user approval.

### Glossary Fact-Check

Run on every entry before final delivery.

| Problem | Example | Fix |
|---------|---------|-----|
| Forced provenance | Tracing "reconciliation" to Pacioli 1494 | Remove the stretch or soften language |
| Over-specific dating | "FP&A crystallized mid-20th-century" | Cite source or drop the date |
| Unsourced claims | "Big Four have standardized review hierarchies for decades" | Cite audit methodology source or soften |
| Academic window-dressing | "Agentic AI traces to Bandura (1986)" | Drop unrelated citation. True origin: Andrew Ng (2024) |
| Vague institutional origin | "Emerged organically in VC (SV, 1970s-80s)" | Find specific first use or say "no single documented origin" |
| Company marketing as fact | "First to productize LLM-based bookkeeping" | Attribute: "Self-described as..." |
| Misattributed coinage | Crediting OpenAI with "JSON mode" | Say "popularized by" not "introduced by" |
| Wrong current status | "CFO at PartsTech" when left 18 months ago | Verify via LinkedIn/Twitter/recent articles |
| Strawman misconception | "People think FP&A is just accounting with charts" | Misconception must be something practitioners actually get wrong |

Uncertain entries → flag "⚠️ SELF-CHECKED — verify manually."

---

## Search Strategy

**Primary: Brave Search API.**
```bash
python3 ~/.hermes/skills/research/web-search/scripts/search.py "query" --count N
```
Free tier: 2,000/month. Log every 50 searches. Throttle above 200/pass.

**Subagent context MUST include this command.** `web_search` tool is universally blocked.

**Chinese sources:** Brave finds titles/snippets but sites may return 403 from non-China IPs. Mark ⚠️ GEOBLOCKED. Do not remove.

**Fallback (Brave empty):** Wikipedia API → domain probing → Substack API → SearXNG → flag.

---

## Output Language & Writing Rules

Output glossary, key-people bios, and reading-list entries in the project's language. If `zh` or `mixed`, use Chinese with English originals in parentheses.

Chinese writing rules:
- **No em dashes（破折号）.** Use commas, periods, or restructure sentences.
- **No "不是…而是…" AI pattern.** If you see it, rewrite sentence structure entirely.
- Output should be sharp（有力）over framework-heavy（套路化）.

---

## File Structure

```
{project}/
├── article-plan-v1.md
├── pre-flight-v1.md
├── launch-1.txt, launch-2.txt, ...
├── .launch-1-done, .launch-2-done, ...
├── glossary-v1.md → glossary-v2.md
├── key-people-v1.md → key-people-v2.md
├── reading-list-v1.md → reading-list-v2.md
├── DECISIONS_NEEDED.md
├── gate-1-decisions.md
├── reinforce-state.md
├── reinforce-targets.md
├── STATUS.md
├── _arc/           (v1 → v2 archive)
└── subagent/       (all raw outputs)
    ├── <name>-v0.md
    └── reinforce-p1/, reinforce-p2/, ...
```

---

## Pitfalls

### Architecture

- **Background agents dispatch. Interactive agents integrate.**
- **SIGNAL files, not delivery.** No background agent sends user messages.
- **Launch prompts go to files.** Write to `{project}/launch-{N}.txt`. User runs `/background Read {project}/launch-{N}.txt`.
- **Context loss.** 3-tier recovery: session_search → marker file → filesystem search.
- **One project at a time.** Single marker file.

### Background Execution

- **Flat dispatch.** All subagents in one `delegate_task` call. System queues them.
- **Launch failure.** If SIGNAL file absent after ~2x expected duration, interactive agent takes over — dispatches only missing subagents.
- **Subagent partial failure.** Accept partial output. Flag in DECISIONS_NEEDED.md.
- **Category grouping.** One subagent per category of terms, not per entry.

### Reinforce

- **Loop A→B→C every pass.** The flywheel is cross-document discovery, not gap-filling.
- **reinforce-targets.md for Pass 2+.** Prevents re-researching covered entries.
- **reinforce-state.md tracks baseline.**
- **Exit at <5 total new.**

### Subagent Characteristics

- **Same model, context-starved.** Zero conversation history, no memory, limited tools. Cannot pivot when blocked. Not a capability gap — an information gap.
- **Search-heavy subagents are naturally slow** (5-12 min). Normal, not a stall.
- **Orchestrator pattern (future).** `max_spawn_depth=2` configured. Overseer subagent can handle failures and retries autonomously.

### Search

- **Brave quota.** Log every 50. Throttle above 200/pass.
- **Subagent web_search fails.** #1 cause. Every context MUST include Brave Search command.
- **Domain pre-check.** Probe all must-include domains from interactive side before launch.
