# Session Patterns Skill

Detect recurring patterns across MEP sessions and surface actionable insights.
Turns repetition into automation by feeding the skill development cycle.

## Why This Matters

When agents work across multiple machines via MEP, patterns emerge: the same
blockers recur, the same entities appear, the same decisions get re-made. Without
detection, this repetition is invisible. With it, recurring patterns become new
skills — the system literally learns what to automate next.

## Workflow

### 1. Collect Handoff History

Read the last N handoff files (default: last 14 days or 20 sessions, whichever
is smaller). Source data:
- Handoff files from the MEP repo (git log for handoff.md changes)
- MEP-sourced timeline entries from gbrain (tagged `[Source: MEP session]`)
- Heartbeat events from `~/.gbrain/integrations/mep-sync/heartbeat.jsonl`

### 2. Extract Structured Signals

For each session, extract:
- **Entities mentioned** (people, companies, projects, tools)
- **Blockers encountered** (problems, errors, failures)
- **Decisions made** (choices, trade-offs, direction changes)
- **Tools/commands used** (what the agent actually did)
- **Time spent** (session duration from git timestamps)

### 3. Detect Patterns

Run frequency analysis across extracted signals:

| Pattern Type | Detection Rule | Threshold |
|-------------|---------------|-----------|
| Recurring blocker | Same error/problem in 3+ sessions | 3 occurrences in 14 days |
| Hot entity | Same person/company in 5+ sessions | 5 mentions in 14 days |
| Repeated decision | Same choice made 3+ times | 3 identical decisions |
| Tool repetition | Same command sequence in 4+ sessions | 4 occurrences |
| Session clustering | 3+ sessions on same project in 1 week | 3 sessions in 7 days |

### 4. Surface Insights

For each detected pattern, generate an actionable insight:

**Recurring blocker → Suggest a fix or skill:**
"You've hit [API rate limiting] in 4 of your last 7 sessions. Consider: (a) creating
a rate-limit-retry utility, (b) caching API responses, or (c) adding a skill that
handles this automatically."

**Hot entity → Suggest a brain page or enrichment:**
"[Dynamics 365 Connected Field Service] appeared in 6 sessions this week. The brain
page was last updated 12 days ago. Consider running enrichment to refresh it."

**Repeated decision → Suggest codifying as a default:**
"You've chosen [TypeScript over Python] for API scripts 3 times. Consider adding
this as a project default in CLAUDE.md so agents don't re-decide it."

**Tool repetition → Suggest a skill or alias:**
"You run [docker stop && docker rm && docker run] in the same sequence every session.
Consider a `reset-test-db` script."

**Session clustering → Suggest a dedicated skill:**
"3+ sessions this week on [Field Service Integration]. This is a recurring workstream.
Consider creating a dedicated skill with project-specific context."

### 5. Write Pattern Report

Output a structured report to the user:

```
## Session Pattern Report — [date range]

### Sessions Analyzed: N
### Patterns Detected: M

**RECURRING BLOCKERS**
- [blocker] — N occurrences — Suggestion: [action]

**HOT ENTITIES**
- [entity] — N mentions — Brain page status: [fresh/stale/missing]

**REPEATED DECISIONS**
- [decision] — N times — Suggestion: codify as default

**AUTOMATION CANDIDATES**
- [command sequence] — N occurrences — Suggestion: create script/skill
```

### 6. Feed Skill Development Cycle

For any pattern that triggers the "suggest a skill" action:
- Create a draft skill concept (Step 1 of the 5-step cycle)
- Add to TODOS.md as a candidate skill
- Track whether the user acts on the suggestion

Over time, the system learns which pattern types the user acts on and
prioritizes those in future reports.

## Schedule

Run weekly (Sunday night, as part of dream cycle):
```
0 3 * * 0 # 3 AM Sunday — session pattern analysis
```

Can also be triggered manually: "Run session pattern analysis"

## Quality Rules

- Only surface patterns with 3+ occurrences. Isolated events are noise.
- Don't suggest skills for one-off projects. Only recurring workstreams.
- Insights must be actionable. "You mentioned X a lot" is not actionable.
  "X appeared 5 times — here's what to do about it" is.
- Respect MECE: if an existing skill already covers a detected pattern,
  reference it instead of suggesting a new one.
- Keep the report under 500 words. Density over comprehensiveness.

## Tools Used

- Keyword search gbrain (search)
- Hybrid search gbrain (query)
- Read a page from gbrain (get_page)
- List pages in gbrain by type (list_pages)
- View timeline entries in gbrain (get_timeline)
