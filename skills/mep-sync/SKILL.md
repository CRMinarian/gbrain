# MEP-Brain Sync Skill

Sync MEP Protocol handoffs with GBrain. What agents learn across machines
compounds in the brain. What the brain knows flows into the next handoff.

## Why This Matters

MEP Protocol keeps agents in sync across machines via handoff files. GBrain keeps
knowledge compounding via persistent pages. Without this skill, these are separate
systems — agents forget cross-session learnings, and the brain doesn't benefit from
distributed work. With it, every MEP session makes every future session smarter.

## Workflow

### On Session End (`/mep end` or handoff push)

1. **Read the handoff file.** Parse the current handoff.md for structured content:
   - Decisions made (what was chosen and why)
   - Entities mentioned (people, companies, repos, services)
   - Blockers hit (problems encountered, workarounds found)
   - Outcomes (what was accomplished, what's pending)

2. **Write to brain.** For each extracted signal:
   - **Entities:** check if brain page exists (`gbrain search "<entity>"`).
     If yes, append timeline entry. If new and notable, create page.
   - **Decisions:** append to the relevant project/concept page as a timeline entry:
     `- YYYY-MM-DD | Decision: {what} because {why} [Source: MEP session, {machine}]`
   - **Blockers:** append to relevant page with resolution:
     `- YYYY-MM-DD | Blocker: {problem}. Resolution: {fix} [Source: MEP session]`
   - **Outcomes:** update compiled truth on project pages if status changed

3. **Sync brain.** `gbrain sync --no-pull --no-embed` to index changes.

4. **Log to heartbeat.**
   ```bash
   echo '{"ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","event":"mep_session_end","entities":'$COUNT'}' \
     >> ~/.gbrain/integrations/mep-sync/heartbeat.jsonl
   ```

### On Session Start (`/mep start` or handoff pull)

1. **Read the handoff file.** Identify the current task/project context.

2. **Query brain for relevant context:**
   - `gbrain search "<project name>"` — recent timeline entries
   - `gbrain query "<current task description>"` — related knowledge
   - `gbrain get <project-slug>` — full project page if it exists

3. **Inject brain context into handoff.** Add a `## Brain Context` section at the
   top of the handoff file (below the header, above existing entries):
   ```markdown
   ## Brain Context (auto-injected by mep-sync)
   - **Project status:** [compiled truth summary from brain]
   - **Recent decisions:** [last 3 decisions from timeline]
   - **Known blockers:** [unresolved blockers from timeline]
   - **Related entities:** [people/companies involved, with brain slugs]
   ```

4. **Commit and push.** The enriched handoff is now available to the next agent.

### During Dream Cycle (Nightly)

1. **Consolidate cross-session learnings.** Read all MEP-sourced timeline entries
   from the last 24 hours. Rewrite compiled truth on affected project pages.
2. **Detect stale handoffs.** If a handoff file hasn't been updated in 7+ days,
   flag the project as potentially abandoned.
3. **Cross-reference.** Link entities mentioned across different MEP sessions.
   If the same person appears in 3 different project handoffs, their brain page
   should reflect all three contexts.

## Integration Points

**MEP Protocol side:** The MEP_RELAY.md skill in the mep-protocol repo should
call gbrain at two points:
- After `/mep end`: trigger brain write (this skill's "Session End" workflow)
- Before `/mep start`: trigger brain read (this skill's "Session Start" workflow)

**GBrain side:** This skill is a standard gbrain skill. It uses existing operations:
- `gbrain search` / `gbrain query` for reads
- `gbrain put` / `gbrain sync` for writes
- Timeline entries for structured event data

## Handoff Parsing Rules

The handoff file is structured markdown. Parse deterministically:

```
SECTION_HEADERS = [/^##\s+(.+)/]  // H2 headers delimit sections
DECISION_PATTERNS = [/decided|chose|picked|going with|ruling/i]
BLOCKER_PATTERNS = [/blocked|stuck|failed|broken|can't|workaround/i]
ENTITY_PATTERNS = [/[A-Z][a-z]+(?:\s+[A-Z][a-z]+)+/]  // Proper nouns
OUTCOME_PATTERNS = [/completed|shipped|merged|deployed|done|finished/i]
```

Use regex for detection, LLM for extraction. If a line matches a pattern,
the agent reads surrounding context to extract the structured data.

## Handoff Quality Tracking

Each MEP session teaches the system what the next handoff needs.

### On Session Start: Rate the Handoff

After reading the handoff file, the agent self-rates:
- **Sufficient:** had enough context to start immediately (no re-discovery needed)
- **Partial:** some context was missing, had to re-read brain or re-explore code
- **Insufficient:** handoff was stale or missing critical context, significant ramp-up

Log the rating:
```bash
echo '{"ts":"...","event":"handoff_quality","rating":"sufficient|partial|insufficient","missing":"[what was missing]"}' \
  >> ~/.gbrain/integrations/mep-sync/heartbeat.jsonl
```

### Weekly: Adapt the Template

During the weekly session-patterns analysis, read handoff quality ratings:
- If 3+ "partial" or "insufficient" ratings mention the same missing context,
  add that section to the handoff template automatically.
- If a handoff section is never referenced (no brain writes from it, no decisions
  traced to it), suggest removing it from the template.
- Track template evolution in brain: `references/mep-handoff-template` page with
  timeline entries for each adaptation.

The handoff template evolves toward optimal information density — sections the agent
needs are always present, sections it ignores are pruned.

## Quality Rules

- Never overwrite human-written handoff content. Brain context is injected as a
  clearly marked section that the agent can update.
- Timeline entries from MEP sessions are tagged `[Source: MEP session]` for
  provenance tracking.
- Don't create brain pages for trivial entities (variable names, file paths).
  Only track people, companies, projects, and concepts.
- If the handoff file mentions the same entity the brain already tracks, use the
  brain's canonical slug (don't create duplicates).

## Tools Used

- Keyword search gbrain (search)
- Hybrid search gbrain (query)
- Read a page from gbrain (get_page)
- Store/update a page in gbrain (put_page)
- Add a timeline entry in gbrain (add_timeline_entry)
- Link entities in gbrain (add_link)
- Sync changes (sync)
