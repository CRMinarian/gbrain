# Resco Voice Pilot — Back of the Napkin: ROI, Deployment & V1 Plan

**Date:** April 2026 | **Classification:** CONFIDENTIAL

---

## Pricing Tiers

### Tier 1 — OOB Cloud-Assisted

**What it is:** Voice interface using cloud AI (Azure OpenAI / Copilot APIs) integrated into Resco. Internet required at time of query. No on-device model. Faster to build, works on any device with adequate RAM.

**Target customer:** Organizations with reliable field connectivity that want voice and manual Q&A but don't have offline requirements or data sovereignty concerns.

| | |
|---|---|
| **Price** | $45 / user / month |
| **Offline** | No |
| **Data leaves device** | Yes (to Azure OpenAI) |
| **Manual RAG** | Yes (cloud-side vector search) |
| **Device requirement** | Any modern iOS/Android |
| **HIPAA / CJIS / ITAR capable** | No |
| **Positioning** | Entry-level — competes on price with Microsoft Copilot ($30/user) |

---

### Tier 2 — On-Device AI (The Moat)

**What it is:** Gemma 4 E2B running locally on the device. No cloud call at inference time. Full offline operation. Manual RAG stored on-device in SQLite-vec. Data never leaves the device during query.

**Target customer:** Organizations in regulated industries (HIPAA, CJIS, ITAR), remote environments (oil fields, mining, utilities, military), or with data sovereignty requirements.

| | |
|---|---|
| **Price** | $125 / user / month |
| **Offline** | Yes — 100% on-device |
| **Data leaves device** | No |
| **Manual RAG** | Yes (on-device SQLite-vec) |
| **Device requirement** | 8GB+ RAM, modern NPU (see spec doc) |
| **HIPAA / CJIS / ITAR capable** | Yes |
| **Positioning** | Premium — category-defining capability no competitor offers |

---

## Customer ROI Model

### Inputs (conservative estimates)

| Variable | Value |
|---|---|
| Time saved per tech per day (manual lookup + form entry) | 30 minutes |
| Working days per year | 250 |
| Hours saved per tech per year | 125 hours |
| Fully-loaded field tech hourly rate | $40 |
| **Annual value per technician** | **$5,000** |

### Cost vs. Value

| Tier | Annual cost / tech | Annual value / tech | ROI | Payback |
|---|---|---|---|---|
| OOB Cloud ($45/mo) | $540 | $5,000 | **9.3:1** | 5.5 weeks |
| On-Device ($125/mo) | $1,500 | $5,000 | **3.3:1** | 16 weeks |

**Additional value not captured above:**
- Reduced first-call-back rate (fewer errors from guessed specs)
- Fewer diagnostic delays on failure codes
- Compliance auditability (auto-logged voice actions)
- Reduced training time for new technicians

Even at conservative numbers, On-Device pays for itself in one quarter.

---

## Dependencies — What Must Exist Before Building

### External Dependencies

| Dependency | Owner | Status | Blocker? |
|---|---|---|---|
| Resco SDK / plugin development access | Resco engineering | Requires partnership agreement | YES |
| Gemma 4 E2B model weights (GGUF + MediaPipe) | Google / HuggingFace | Publicly available | No |
| SQLite-vec library | Open source (MIT) | Available | No |
| MiniLM-L6-v2 embedding model | HuggingFace | Publicly available | No |
| llama.cpp iOS framework | Open source (MIT) | Available | No |
| MediaPipe LLM Inference API | Google | Available (Android SDK) | No |
| Test devices (Pixel 9 Pro, iPhone 17 Pro, Tab Active5) | Procurement | Purchase required | Soft |
| D365 Field Service sandbox tenant | Microsoft | Dev tenant available | No |
| Power Platform developer tenant | Microsoft | Free tier available | No |
| Sample manual corpus (PDF) | Customer / internal | Komatsu manuals available | No |
| 1–2 pilot customers for field validation | Resco sales | Identify from existing base | YES |

### Internal Dependencies (Resco side)

| Dependency | Notes |
|---|---|
| Resco Woodford (admin UI) | Must expose Voice Pilot plugin config — existing surface |
| Resco Sync Engine | Must support manual index delta sync — likely minor extension |
| Resco Mobile SDK | Plugin host must be stable and documented |
| App Store / Play Store accounts | Required for distribution |
| Legal / compliance review | HIPAA BAA template, data processing agreements |

---

## Staffing Requirements

### Core Team — V1 Build

| Role | Responsibilities | Dev Hours |
|---|---|---|
| **Mobile AI Engineer** (1 FTE) | Gemma integration, MediaPipe (Android), llama.cpp (iOS), SQLite-vec, RAG pipeline | 340 hrs |
| **Mobile Platform Engineer** (1 FTE) | Resco plugin architecture, iOS/Android integration, form action execution, sync | 280 hrs |
| **Backend / Data Engineer** (0.5 FTE) | PDF ingestion pipeline, chunking, embedding server, delta packaging | 120 hrs |
| **QA / Field Validation** (0.5 FTE) | Device testing, latency benchmarking, field scenario validation | 80 hrs |
| **Product / PM** (0.25 FTE) | Requirements, customer coordination, demo preparation | 40 hrs |
| **Total** | | **860 dev hours** |

### Optional / Phase 2

| Role | Purpose | Hours |
|---|---|---|
| Security / compliance engineer | HIPAA BAA, CJIS, ITAR certification packaging | 80 hrs |
| UX designer | Voice UI feedback design, tablet split-screen layout | 60 hrs |
| DevOps | Manual ingestion pipeline CI/CD, index packaging automation | 40 hrs |

---

## Timeline — Dev Hours to V1

All durations expressed in dev hours. Calendar time assumes 2.5 FTE average across the build.

```
PHASE 1 — SHARED AI CORE                              [240 hours]
├─ Week 1–2 (approx)
├─ Gemma 4 E2B on Android (MediaPipe)                  80 hrs
├─ Gemma 4 E2B on iOS (llama.cpp + Metal)              80 hrs
├─ SQLite-vec + MiniLM RAG pipeline                    60 hrs
└─ Voice capture + intent routing                      20 hrs
   MILESTONE: "Hello World" voice → RAG answer on Pixel 9 Pro

PHASE 2A — TRACK 1: POWER PLATFORM                    [160 hours]
├─ Week 3–4 (approx), parallel with 2B
├─ Dataverse action schema + Resco plugin              80 hrs
├─ Offline queue + sync integration                    40 hrs
└─ Device testing (Pixel, iPhone, Tab)                 40 hrs
   MILESTONE: Voice → Dataverse form fill, offline, all 3 devices

PHASE 2B — TRACK 2: D365 FIELD SERVICE                [160 hours]
├─ Week 3–4 (approx), parallel with 2A
├─ D365 FS entity mapping (Work Order, Service Task)   80 hrs
├─ Parts + labor voice commands                        40 hrs
└─ Device testing (Pixel, iPhone, Tab)                 40 hrs
   MILESTONE: Voice → Work Order update, offline, all 3 devices

PHASE 3 — MANUAL INGESTION PIPELINE                   [160 hours]
├─ Week 5–6 (approx)
├─ PDF → OCR → chunker → embed → SQLite-vec            80 hrs
├─ Delta sync packaging via Resco Sync Engine          40 hrs
└─ Admin UI (Woodford) for model assignment            40 hrs
   MILESTONE: 3,000-page Komatsu manual indexed, queried offline

PHASE 4 — INTEGRATION + POLISH                        [140 hours]
├─ Week 7–8 (approx)
├─ End-to-end latency optimization                     60 hrs
├─ Cold start optimization (<8 sec)                    30 hrs
├─ Battery drain testing                               20 hrs
└─ App Store / Play Store packaging                    30 hrs
   MILESTONE: All acceptance criteria met on all 3 devices

TOTAL: 860 dev hours
```

### Calendar Estimate

| Scenario | Calendar Time |
|---|---|
| 2.5 FTE (recommended) | ~10–11 weeks |
| 2 FTE (lean) | ~13–14 weeks |
| 4 FTE (accelerated) | ~7 weeks |

---

## Revenue Projections

### Year 1 — Resco Existing Base (160,000 users)

| Scenario | Attach Rate | Users | Tier Mix | Monthly | ARR |
|---|---|---|---|---|---|
| Conservative | 10% | 16,000 | 70% OOB / 30% On-Device | — | **$13.1M** |
| Base | 15% | 24,000 | 50/50 mix | — | **$24.0M** |
| Optimistic | 25% | 40,000 | 40% OOB / 60% On-Device | — | **$47.4M** |

### Year 3 — Vertical Expansion (Defense, LE, Healthcare, Field Science)

| Market | Addressable Users | Realistic Capture | ARR at $125 |
|---|---|---|---|
| D365 FS + Power Platform base (expanded) | 500,000 | 5% | $37.5M |
| Defense / ITAR maintenance | 200,000 | 2% | $6.0M |
| Law enforcement (CJIS) | 150,000 | 2% | $4.5M |
| Healthcare field workers | 100,000 | 1.5% | $2.25M |
| **Year 3 Total** | | | **~$50M+ ARR** |

---

## Proof-of-Concept Investment (Phase 1 Only)

To de-risk before full V1 commitment, fund Phase 1 alone:

| | |
|---|---|
| **Scope** | Gemma on-device + basic RAG + one voice → form fill demo |
| **Dev hours** | 240 hours |
| **Calendar time** | ~3 weeks at 2 FTE |
| **Deliverable** | Working demo: voice query answered from Komatsu manual on Pixel 9 Pro, offline |
| **Decision gate** | Latency < 2s? Accuracy acceptable? → proceed to full V1 |

**This is the right first check.** A 240-hour investment answers the technical question before committing 620 more hours to the full build.

---

## Risk Register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Gemma E2B latency too slow on Tab Active5 | Medium | High | Fall back to OOB cloud tier for lower-spec devices |
| Resco SDK doesn't expose required plugin hooks | Low | High | Engage Resco engineering early; this is a partnership requirement |
| iOS App Store rejects local LLM app | Low | High | Precedent: LLM Farm, Enchanted already approved. Comply with privacy labels |
| Manual OCR quality poor for older PDFs | Medium | Medium | Use Azure Form Recognizer for complex layouts; budget 10% cleanup overhead |
| Customer pilot partner delays field validation | Medium | Low | Recruit 2 pilots to ensure redundancy |
| Battery drain exceeds 15% threshold | Low | Medium | Implement model unload after 60s idle; lazy inference warm |
