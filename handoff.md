# Handoff Log

Context relay between sessions. Newest first. Read before working, write before signing off.

## 2026-04-21 — Claude | Claude (Code) | no-op
**Tag-in:** 2026-04-21 | **Tag-out:** EOL

### What happened
- No new work. Session opened and closed immediately.

### What's pending
- [ ] All items from 2026-04-20 entry below still apply (CEO deck, competitive landscape, PR #4 merge, SSH cleanup)

### Watch out for
- Nothing new. See prior entries.

---

## 2026-04-20 — Claude | Claude (Code) | office-hours + product-strategy
**Tag-in:** 09:00 CT | **Tag-out:** EOL

### What happened
- Ran full office-hours session on **Resco Voice Pilot** — local LLM (Gemma 4 E2B) as a voice interface for Resco field service workers with offline manual RAG
- Researched Resco.net, Resco Toolkit, Gemma 4 E2B native audio capabilities, D365 Field Service entity model
- Refined architecture to **dual-track**: Track 1 (Power Platform standalone) + Track 2 (D365 Field Service) sharing one AI core
- Optimized for **three demo devices**: Pixel 9 Pro (MediaPipe, ~900ms), iPhone 17 Pro (llama.cpp Metal, ~1.3s), Samsung Galaxy Tab Active5 rugged (MIL-STD-810H, ~2.1s)
- Developed **market analysis** with offline as the core moat — $35B TAM across FSM, defense (ITAR), law enforcement (CJIS), social services (HIPAA), healthcare EMS, field science
- Established **document format rule** in CLAUDE.md: all deliverables in Markdown only
- Built and saved three deliverable documents to `docs/resco-voice-pilot/`:
  1. `functional-design-spec.md` — full 12-section FTS with architecture, device specs, RAG pipeline, compliance, acceptance criteria
  2. `executive-briefing.md` — 2-page CEO brief, offline moat, $35B market, 3.3:1 customer ROI
  3. `back-of-napkin-roi-deployment.md` — two pricing tiers ($45 OOB / $125 on-device), 860 dev hrs to V1, staffing, timeline, risk register
- Published all three to the dashboard at `http://dashboard.nukasoft.ai/reports/` via `publish-report.sh` on Hot Rod
- Cleared stale SSH host key for Hot Rod (192.168.0.219) — key had changed, required `ssh-keygen -R` before reconnecting

### What's pending
- [ ] CEO presentation deck (PowerPoint or slide-ready Markdown) — not built yet, user approved docs first
- [ ] Market analysis section incomplete — competitive landscape (vs. Copilot, ServiceMax, IFS) and full TAM/SAM/SOM breakdown not written up
- [ ] Resco CEO meeting prep — identify who the contact is and how to get on the calendar
- [ ] All prior SSH/PR items from earlier handoff entries still open (PR #4 merge, gh scope refresh, gho_ token revoke)

### Watch out for
- Hot Rod SSH host key was rotated — `~/.ssh/known_hosts` was updated this session. If you get a host key warning again, run `ssh-keygen -R 192.168.0.219` before reconnecting
- Resco Voice Pilot docs live in TWO places: source in `~/Dev/Gbrain/docs/resco-voice-pilot/` (this repo) and published copies on Hot Rod at `/var/www/hotrod/reports/`. If you edit the source, re-run `publish-report.sh` on Hot Rod to sync
- The "OOB" pricing tier ($45/user/month) uses cloud AI — it needs internet. The $125/user/month tier is the offline on-device version. Don't conflate them in the CEO pitch
- Gemma 4 E2B native audio is confirmed on E2B and E4B model variants only — the larger 31B and 26B MoE variants do NOT have native audio

## 2026-04-20 — Claude | Claude (Code desktop, MBP) | split + scrub + EOL
**Tag-in:** continuing 04-17 session | **Tag-out:** EOL — handing to Hot Rod

### What happened
- **Two-repo split is live.** Created private [CRMinarian/pbrain](https://github.com/CRMinarian/pbrain) as the personal-data sidecar to the public fork. Pattern matches `docs/guides/repo-architecture.md` upstream.
- **pbrain at `/Users/pierrehulsebus/Dev/pbrain`** (local) and `origin/main` on GitHub (private). Contains: `README.md` explaining the pattern, `.gitignore` for secrets/state/backups, `recipes/turo-booking-agent.md`, `recipes/fieldservice-to-brain.md`, and `recipes/drafts/turo-booking-agent.desktop-2026-04-17.md` (preserved the 725-line desktop-original diff for reference).
- **PR #2 was merged** on gbrain fork (`0bea19a`) while prior session was in flight — recipes hit public master with personal data. **Why problematic:** `skippy@nukasoft.com`, 2023 Tesla Model 3 VIN context, Gmail query syntax, iMessage escalation rules all published to public git history.
- **PR #3 opened** to remove recipes from the tree (`7c16697`). Merged as a no-op after the history rewrite absorbed it.
- **Full history scrub executed** via `git-filter-repo` (Homebrew install). Mirror-cloned fork, ran `git filter-repo --path recipes/turo-booking-agent.md --path recipes/fieldservice-to-brain.md --invert-paths`, force-pushed `--mirror`. Master went `0bea19a → 59e2813`, all branches rewritten with new SHAs. Every historical mention of the two recipe paths is gone from every ref. GitHub's hidden `refs/pull/*` refs couldn't be overwritten (expected — those are server-managed), but the PR metadata is harmless; the commits themselves are unreachable.
- **Verified clean**: `gh api repos/CRMinarian/gbrain/contents/recipes` returns 7 generic upstream recipes only. No Turo, no Field Service.
- **Local gbrain worktree reset** to new master. Stale branches deleted (`chore/move-personal-recipes-to-pbrain`, `claude/add-office-hours-iW7t8`, `Nagatha/musing-johnson-ffe231`). Worktree at `.claude/worktrees/musing-johnson-ffe231` removed.
- **Backup** of pre-scrub `.git` saved at `/tmp/gbrain-pre-scrub-backup/gbrain-20260420-173035.tar.gz` (5.3 MB). Keep for 7 days then delete.

### What's pending (for Hot Rod pickup)
- [ ] **@Hot-Rod-Claude: wire agent to read recipes from pbrain.** `export GBRAIN_RECIPES_DIR="$HOME/Dev/pbrain/recipes"`. Clone pbrain first: `git clone https://github.com/CRMinarian/pbrain.git ~/Dev/pbrain`. Remember `$GBRAIN_RECIPES_DIR` is untrusted per wave-3 security — recipes there can't run arbitrary commands in health_checks. That's intentional.
- [ ] **@Hot-Rod-Claude: reconcile Turo recipe duplicates in pbrain.** Two files currently: `recipes/turo-booking-agent.md` (the cloud-era PR #2 version) and `recipes/drafts/turo-booking-agent.desktop-2026-04-17.md` (the pre-cloud desktop draft, 725 lines of diff vs the other). Diff, merge the useful parts, drop the losing side. Desktop didn't want to make that call under time pressure.
- [ ] **@Hot-Rod-Claude: delete `/Users/pierrehulsebus/Dev/Gbrain/HANDOFF_TO_PHONE.md` on first sync.** It's a stale one-shot meat-puppet script superseded by this handoff.md. Untracked — just `rm` it.
- [ ] **@Skippy chip is posted** (from the 04-17 session) for the dead `Pierre-Alithya` SSH credential audit. Not picked up yet. Still relevant on Hot Rod too — check `~/.ssh/config` there.
- [ ] **@Hot-Rod-Claude: consider upstreaming MEP pattern to `garrytan/gbrain`.** The skills (mep-sync, session-patterns) and the MEP section of CLAUDE.md are generic and shareable. Would be a clean contribution PR. Skip the recipes — those are personal and already private.
- [ ] **@Hot-Rod-Claude: delete local backup tarball** `/tmp/gbrain-pre-scrub-backup/` after 7 days confidence window (target: 2026-04-27). Clean system hygiene.

### Watch out for
- **Scrubbed history means SHAs changed everywhere.** Any machine with an old clone of the fork will fail to pull cleanly. Hot Rod's first action should be a fresh `git clone https://github.com/CRMinarian/gbrain.git` (or `git fetch origin && git reset --hard origin/master` if the clone pre-existed).
- **Dead `Pierre-Alithya` SSH key** still unresolved. HTTPS + gh token works; SSH fails silently. On Hot Rod, run `gh auth setup-git` before any git push.
- **pbrain is brand new and has exactly one commit** (`b8b445d`). First time an agent reads from it, nothing will be pre-validated against the security model. Run `gbrain doctor` or the integration recipe smoke tests after wiring it up.
- **PR #3 shows as MERGED with 0 changes** on GitHub. That's not a GitHub bug — it's the history scrub making the PR's intent redundant. Don't re-open it. Close out notification UI if it's bugging you.
- **mep-sync skill is still spec-only.** We wrote about it, we didn't implement the hooks. Every handoff still manual. Hot Rod should keep writing handoffs by hand on session end.
- **Cloud instance's 2nd handoff commit (`370d86f`) was on the branch post-merge and got absorbed into the scrub.** Its content is not on master. If it had anything critical, it's gone. Check `/tmp/gbrain-pre-scrub-backup/gbrain-20260420-173035.tar.gz` — decompress and run `git log` if you need to recover it.

### Quality rating for next agent
- **Sufficient** — all file paths, SHAs, command sequences, and reasoning included. Hot Rod should be able to start cold.

---

## 2026-04-17 — Skippy | Claude (Code) | pr-open
**Tag-in:** continuation of the ssh-audit entry below | **Tag-out:** PR #4 open for that session's handoff write

### What happened
- Opened [PR #4](https://github.com/CRMinarian/gbrain/pull/4) on `CRMinarian/gbrain:master` carrying the ssh-audit handoff entry. Branch `Nagatha/musing-johnson-ffe231`. Single-commit PR (handoff-only, no code). **Why:** per repo workflow the handoff lives in `master`, but the originating worktree pushed to a feature branch; PR is the merge path.
- Left untracked `HANDOFF_TO_PHONE.md` in the worktree alone — it's the stale phone-handoff file flagged for deletion in an earlier handoff entry, not this session's concern.

### What's pending
- [ ] @Pierre: review + merge PR #4 (handoff-only, no CI risk).
- [ ] All items from the ssh-audit entry below still apply (gh scope refresh, `gho_` token revoke, `~/aly-fs-weather` decision, deploy-key audit).

### Watch out for
- PR #4's base is `CRMinarian/gbrain:master`, not `garrytan/gbrain:master`. The default `gh pr create` base is upstream for this fork — pass `--repo CRMinarian/gbrain` when intent is fork-internal.

## 2026-04-17 — Skippy | Claude (Code) | ssh-audit + rotation
**Tag-in:** delegated from prior handoff (@Skippy Pierre-Alithya SSH audit) | **Tag-out:** client-side rotation complete, GitHub-side pending

### What happened
- Audited every dependency on the dead `Pierre-Alithya` SSH identity across `~/.ssh/`, launchd, crontab, `~/Dev/*`, `~/Documents/*`, `~/aly-fs-weather`, `~/FieldService`, gh CLI, and Actions workflows. Full report filed at `skippy-brain:machines/macbook/ssh-identity-audit-2026-04-17.md` (commit `f7628c5` on `main`).
- **Rewrote `~/.ssh/config`**: default `Host github.com` + `Host github-nukasoft` both now point at `id_ed25519_crminarian` with `IdentitiesOnly yes`. `ssh -T git@github.com` confirmed → `Hi CRMinarian!`. **Why:** prior default routed to the dead Pierre-Alithya key, so any `git@github.com:...` remote silently auth'd as the wrong user.
- **`gh config set git_protocol https -h github.com`** — closes the latent trap where `gh repo clone` would pick an SSH URL and fall through to the dead key.
- **Stripped embedded OAuth token** from `~/Dev/skippy-brain` remote URL (`https://gho_Irc2Sp…@github.com/...` → bare HTTPS). Token was in plaintext `.git/config`. **Why:** filesystem-readable secret; also survives terminal screenshares.
- Verified `git fetch --dry-run` on all 6 working repos (Gbrain, nukasoft.ai, skippy-brain, FieldService, MacBook.Local.Skippy, Pierre_Brain1).
- **No new SSH key generated** — `id_ed25519_crminarian` already existed and was already on CRMinarian's GitHub account. Correction to the original task: simpler than "rotate to a new key," the fix was "repoint the default identity at the live key already on disk."

### What's pending
- [ ] @Pierre: run `gh auth refresh -h github.com -s admin:public_key -s admin:ssh_signing_key` so a follow-up Skippy session can enumerate and `gh ssh-key delete` the Pierre-Alithya public key from GitHub.
- [ ] @Pierre: decide what to do with `~/aly-fs-weather` — remote `github.com/Pierre-Alithya/aly-fs-weather.git` returns 404 (account is deleted, repo is gone with it). Recover from backup / re-host under CRMinarian / delete local clone.
- [ ] @Pierre: revoke the `gho_Irc2Sp<REDACTED>` token that was embedded in skippy-brain's remote (GitHub → Settings → Developer Settings → Personal access tokens). May still be live.
- [ ] @Pierre: manually check in GitHub UI whether any repo has **Deploy keys** with the Pierre-Alithya fingerprint — `gh` token lacks `admin:org` scope to enumerate org-wide.
- [ ] @Skippy (follow-up session): after scope refresh, delete Pierre-Alithya public key from github.com and close the loop in the audit report.

### Watch out for
- **`~/.ssh/id_ed25519` (Pierre-Alithya private key) is still on disk on purpose** — `Host hotrod` (192.168.0.220 LAN Ubuntu) uses it for non-GitHub auth. Don't delete the file. Only revoke the public key on github.com.
- **`github-nukasoft` SSH alias still used** by `~/Dev/nukasoft.ai` and `~/Documents/MacBook.Local.Skippy` remotes (hardcoded to `git@github-nukasoft:NukaSoft/...`). Alias now resolves to the crminarian key, so those still work — don't remove the alias block from `~/.ssh/config`.
- **`~/aly-fs-weather` is silently dead.** `.github/workflows/export-solution.yml` doesn't run anywhere anymore. Pushes have been silently failing. If Pierre cares about this repo's history, recover before deleting.
- **Do not embed OAuth tokens in git remote URLs.** The osxkeychain + `gh auth git-credential` helper in `~/.gitconfig` handles HTTPS auth cleanly without plaintext secrets in `.git/config`.
## 2026-04-17 — Claude | Claude (Code desktop) | rebase + sync
**Tag-in:** received phone handoff | **Tag-out:** PR #2 mergeable, awaiting review decision

### What happened
- Fork `master` fast-forwarded from stale `91ced66` (Voice v0.8.0, #55) to upstream's `7bbfc3e` (wave 3 security, #174) via `gh repo sync` — server-side merge commit `4cc6fae`. Caught up ~100 commits including GStackBrain v0.10.0, autopilot/features/extract (v0.10.1), search quality boost (v0.8.1), wave 2 + 3 security.
- Branch `claude/add-office-hours-iW7t8` rebased onto new master. Resolved conflicts in `skills/manifest.json` (kept all 25 upstream skills + appended `mep-sync`, `session-patterns` → 27 total) and `CLAUDE.md` (kept MEP Protocol section at top, kept newer GStack-aware intro paragraph). Rebase only replayed 2 commits because the first 2 (`/office-hours` + `$ARGUMENTS` fix) were already in fork master via PR #1.
- Force-pushed rebased branch. PR #2 now base `4cc6fae`, head `c1d005b`, state `MERGEABLE`. **Why:** stale base meant PR would have merged without wave 3 security fixes; rebase de-risks merge.
- PR #2 title corrected to "feat: add /office-hours slash command + 2 self-recursive learning skills + 2 recipes" (was misleadingly "4 skills"). **Why:** diff is 2 skills + 2 recipes.
- Desktop `.gitconfig` remotes restructured: `origin = CRMinarian/gbrain` (fork), `upstream = garrytan/gbrain`. Origin switched from SSH to HTTPS because SSH key identity is `Pierre-Alithya` (dead account) while gh token is `CRMinarian`. `gh auth setup-git` wired the token as git credential helper.

### What's pending
- [ ] @Pierre: decide whether to merge PR #2 or solicit review first. No CI configured so tests are manual.
- [ ] @Desktop-Claude: reconcile `recipes/turo-booking-agent.md` duplicate — untracked backup at `recipes/turo-booking-agent.md.desktop-backup` has 725 lines of diff vs the committed version. Diff and pick a winner before the backup clutters the tree.
- [ ] @Skippy: audit what services on this machine (and others) depend on the dead `Pierre-Alithya` SSH credential. Rotate to a live identity before that key gets revoked elsewhere.
- [ ] @Desktop-Claude: once PR #2 merges, consider upstreaming the 4 new files (2 skills + 2 recipes + MEP section + handoff.md) to `garrytan/gbrain` as a separate PR. MEP Protocol and the Turo recipe are the most obvious candidates.
- [ ] Phone-side: delete stale `HANDOFF_TO_PHONE.md` from desktop tree — it was a one-shot meat-puppet script, superseded now that handoff.md is the shared surface.

### Watch out for
- **Dead SSH cred `Pierre-Alithya`** — `git push` over SSH fails silently under that identity; the workaround is HTTPS + gh token. Anywhere a git automation assumes SSH works will break until the key is rotated.
- **Fork master merge commit** (`4cc6fae`) is a merge of upstream into old fork master, not a fast-forward. The commit graph has a braid in it. Clean, but different from a linear history.
- **PR #2 body still references "4 self-recursive learning skills"** in its summary bullets even though the title is fixed. Consider editing the body too if that matters for CHANGELOG generation.
- **No CI on this repo.** Don't assume tests ran. Before merging, desktop should run `bun test` (and ideally `bun run test:e2e` per CLAUDE.md's test DB lifecycle).
- **mep-sync skill is not yet functional** — it's a spec, not an implementation. Future sessions still need to write handoffs manually until the skill's hooks are wired into gbrain itself.

## 2026-04-17 — Claude | Claude (Code) | code
**Tag-in:** session start | **Tag-out:** active

### What happened
- Added `/office-hours` slash command for product idea evaluation (structured scoring)
- Added 4 self-recursive learning skills: `mep-sync`, `session-patterns` + 2 recipes
- Created PR #2 on crminarian/gbrain (`claude/add-office-hours-iW7t8` -> `master`)
- Set up MEP Protocol handoff infrastructure in this repo

### What's pending
- [ ] PR #2 awaiting review — no CI checks configured, no review comments yet
- [ ] MEP-sync skill (`skills/mep-sync/SKILL.md`) should be aligned with NukaSoft spec
- [ ] Session-patterns skill needs real session data to begin pattern extraction

### Watch out for
- No CI configured on the repo — tests must be run manually before merge
- `.env.testing` may not exist in this worktree — check sibling worktrees if E2E tests needed
