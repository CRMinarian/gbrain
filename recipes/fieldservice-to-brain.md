---
id: fieldservice-to-brain
name: FieldService-to-Brain
version: 0.8.0
description: Import FieldService knowledge repo (lexicon, frameworks, diagrams) into brain pages. Agent becomes a domain expert.
category: sense
requires: []
secrets: []
health_checks:
  - "[ -d \"$FIELDSERVICE_REPO_PATH\" ] && echo 'FieldService repo: found' || echo 'FieldService repo: not found (set FIELDSERVICE_REPO_PATH)'"
setup_time: 10 min
cost_estimate: "$0"
---

# FieldService-to-Brain: Domain Expertise That Compounds

Your FieldService repo is a curated knowledge library — lexicon entries, architectural
frameworks, diagrams, and reference material for AI + Field Service + Copilot. This
recipe imports it into GBrain so your agent becomes a field service domain expert.
When you update the repo, the brain re-syncs automatically.

## IMPORTANT: Instructions for the Agent

**You are the installer.** Follow these steps precisely.

**Why this is high-value:** The FieldService repo contains canonical definitions and
frameworks that the agent needs to answer domain questions correctly. Without this
recipe, the agent guesses at terminology. With it, the agent speaks your vocabulary
and references your frameworks.

**The core pattern: code for data, LLMs for judgment.**

1. DETERMINISTIC: git pull the FieldService repo, list markdown files, detect changes
   since last sync (git diff), convert to brain page format with correct types and slugs.
2. LATENT: you (the agent) read imported pages, detect cross-references between
   lexicon entries and frameworks, create links, and update compiled truth when
   multiple sources inform the same concept.

**Sequential execution matters:**
- Step 1 clones or locates the repo. Without it, no source data.
- Step 2 maps repo structure to brain page types. Without it, wrong slugs.
- Step 3 does the first import. Without data, no enrichment.
- Step 4 sets up live sync. Without it, the brain falls behind.

## Architecture

```
FieldService Repo (GitHub: CRMinarian/FieldService)
  ├── /lexicon/*.md      → brain type: concept
  ├── /frameworks/*.md   → brain type: framework
  ├── /diagrams/*.md     → brain type: reference
  ├── /decks/*.md        → brain type: reference
  └── /references/*.md   → brain type: reference
  ↓
Import Script (deterministic)
  ↓ For each markdown file:
  ├── Parse frontmatter (if any)
  ├── Assign brain type from directory
  ├── Generate slug: {type}/{filename-without-ext}
  ├── Check if brain page exists (idempotent)
  └── Write brain page with source attribution
  ↓
GBrain Import
  ↓ `gbrain import brain/fieldservice/ --no-embed && gbrain embed --stale`
  ↓
Agent Cross-Linking
  ↓ Read imported pages, detect relationships:
  ├── Lexicon entry references framework → add link
  ├── Framework references diagram → add link
  └── Multiple entries define same concept → merge into compiled truth
```

## Opinionated Defaults

**Directory-to-type mapping:**

| Repo Directory | Brain Type | Slug Pattern | Example |
|---------------|-----------|-------------|---------|
| `/lexicon/` | concept | `concepts/{slug}` | `concepts/copilot-architecture` |
| `/frameworks/` | framework | `frameworks/{slug}` | `frameworks/field-service-operating-model` |
| `/diagrams/` | reference | `references/diagrams/{slug}` | `references/diagrams/iot-flow` |
| `/decks/` | reference | `references/decks/{slug}` | `references/decks/2026-04-field-service-ai` |
| `/references/` | reference | `references/{slug}` | `references/dynamics-365-api` |

**Brain page format for imported content:**
```markdown
---
type: concept
source: fieldservice-repo
source_path: lexicon/copilot-architecture.md
last_synced: 2026-04-17
tags: [fieldservice, lexicon, copilot]
---

# Copilot Architecture

[Content from the original markdown file]

---

## Source
Imported from [FieldService/lexicon/copilot-architecture.md](https://github.com/CRMinarian/FieldService/blob/main/lexicon/copilot-architecture.md)
```

**Idempotent by source_path:** If a brain page with the same `source_path` already
exists, update it instead of creating a duplicate.

## Prerequisites

1. **GBrain installed and configured** (`gbrain doctor` passes)
2. **Git** (for cloning and tracking changes)
3. **FieldService repo accessible** (public repo, no auth needed)

## Setup Flow

### Step 1: Clone or Locate the FieldService Repo

Ask the user: "Where is your FieldService repo? Is it already cloned locally,
or should I clone it?"

If not cloned:
```bash
git clone https://github.com/CRMinarian/FieldService.git /path/to/fieldservice
export FIELDSERVICE_REPO_PATH=/path/to/fieldservice
```

If already cloned, just set the path:
```bash
export FIELDSERVICE_REPO_PATH=/path/to/existing/fieldservice
```

Validate:
```bash
[ -d "$FIELDSERVICE_REPO_PATH/lexicon" ] && echo "PASS: FieldService repo found" || echo "FAIL"
```

**STOP until the repo is accessible.**

### Step 2: Map Repo Structure

List the content to import:
```bash
find $FIELDSERVICE_REPO_PATH -name "*.md" -not -path "*/.github/*" -not -name "README.md" | head -30
```

Confirm the directory structure matches the expected mapping. If the repo has
directories not listed above, ask the user which brain type to assign them.

### Step 3: First Import

Create the brain pages directory and run the import:
```bash
mkdir -p brain/fieldservice/concepts brain/fieldservice/frameworks brain/fieldservice/references
```

For each markdown file in the repo:
1. Read the file content
2. Determine brain type from parent directory
3. Generate slug from filename
4. Write brain page with frontmatter (type, source, source_path, last_synced, tags)
5. Append original content below frontmatter

Then import to GBrain:
```bash
gbrain import brain/fieldservice/ --no-embed
gbrain embed --stale
```

Verify:
```bash
gbrain search "field service" --limit 5
```

Tell the user: "Imported N pages from FieldService repo. Here are the first 5
search results: [list]."

### Step 4: Set Up Live Sync

Create a sync script that pulls changes and re-imports:
```bash
#!/bin/bash
cd $FIELDSERVICE_REPO_PATH
git pull origin main --quiet

# Detect changed files since last sync
CHANGED=$(git diff --name-only HEAD~1 -- '*.md' 2>/dev/null || echo "")
if [ -z "$CHANGED" ]; then
  echo "No changes to sync"
  exit 0
fi

# Re-import changed files
for file in $CHANGED; do
  echo "Syncing: $file"
  # Import logic here (same as Step 3, per-file)
done

gbrain sync --no-pull --no-embed
gbrain embed --stale
```

Add to cron (daily sync):
```bash
0 6 * * * /path/to/fieldservice-sync.sh >> /tmp/fieldservice-sync.log 2>&1
```

### Step 5: Cross-Link Imported Pages

This is YOUR job (the agent). After import:

1. **Read each lexicon entry.** Does it reference other concepts? Create links:
   `gbrain link concepts/copilot-architecture concepts/dynamics-365 references`
2. **Read each framework.** Does it reference lexicon terms? Link them.
3. **Detect clusters.** If 3+ lexicon entries relate to the same topic, consider
   creating a parent concept page that links to all of them.

### Step 6: Log Setup Completion

```bash
mkdir -p ~/.gbrain/integrations/fieldservice-to-brain
echo '{"ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","event":"setup_complete","source_version":"0.8.0","status":"ok","pages_imported":'$COUNT'}' >> ~/.gbrain/integrations/fieldservice-to-brain/heartbeat.jsonl
```

Tell the user: "FieldService knowledge is now in your brain. Your agent can answer
field service questions with your canonical definitions and frameworks. Sync runs
daily at 6 AM to catch repo updates."

## Implementation Guide

### File Discovery (Deterministic)

```
discover_files(repo_path):
  files = []
  for dir in ['lexicon', 'frameworks', 'diagrams', 'decks', 'references']:
    pattern = join(repo_path, dir, '**/*.md')
    files.push(...glob(pattern))
  return files.filter(f => !f.includes('README.md'))
```

### Slug Generation

```
generate_slug(file_path, repo_root):
  relative = file_path.replace(repo_root + '/', '')
  dir = dirname(relative)  // e.g., 'lexicon'
  name = basename(relative, '.md')  // e.g., 'copilot-architecture'
  type = DIR_TO_TYPE[dir]  // e.g., 'concepts'
  return `${type}/${slugify(name)}`
```

### What the Agent Should Test After Setup

1. **Search accuracy:** `gbrain search "copilot"` returns FieldService lexicon entries.
2. **Source attribution:** Read a brain page. Verify `source: fieldservice-repo` and
   `source_path` point to the correct file.
3. **Idempotency:** Run import twice. Verify no duplicate pages created.
4. **Cross-links:** Check that related concepts are linked in the brain.
5. **Live sync:** Update a file in the FieldService repo, push, run sync. Verify
   the brain page reflects the change.

## Cost Estimate

| Component | Monthly Cost |
|-----------|-------------|
| Git operations | $0 |
| GBrain import/embed | $0 (local) or ~$0.01 (OpenAI embeddings) |
| **Total** | **~$0** |

## Troubleshooting

**No files found during import:**
- Check `FIELDSERVICE_REPO_PATH` is set correctly
- Verify the repo has markdown files in the expected directories
- Run `ls $FIELDSERVICE_REPO_PATH/lexicon/` to confirm structure

**Search returns no results after import:**
- Run `gbrain embed --stale` to generate embeddings
- Check `gbrain doctor` for database connectivity issues

**Duplicate pages after re-import:**
- The import should be idempotent by `source_path`
- If duplicates exist, check that frontmatter `source_path` is consistent
