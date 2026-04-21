# Resco Voice Pilot — Functional & Technical Design Specification

**Version:** 1.0  
**Date:** April 2026  
**Classification:** CONFIDENTIAL  
**Status:** Draft for CEO Review

---

## 1. Executive Summary

Resco Voice Pilot is an on-device AI assistant embedded in the Resco mobile platform that enables field workers to interact with enterprise systems and technical documentation entirely through voice — with no internet connection required. Powered by Google's Gemma 4 E2B running locally on the device, it provides sub-3-second response times, full offline operation, and data sovereignty that cloud AI cannot match.

The solution ships in two deployment tracks:

- **Track 1 — Power Platform:** For organizations using Microsoft Power Platform + Dataverse without D365 Field Service
- **Track 2 — D365 Field Service:** For existing D365 FS customers with Work Orders, Service Tasks, and Bookings

Both tracks share a single AI core, manual ingestion pipeline, and Resco plugin architecture. Only the action schema (what Gemma outputs) differs between tracks.

---

## 2. Problem Statement

Field workers servicing complex equipment — heavy machinery, utility infrastructure, industrial systems — face three compounding problems:

| Problem | Real-World Impact |
|---|---|
| Hands occupied during work | Typing requires removing gloves, setting down tools, interrupting workflow |
| Technical manuals are inaccessible in the field | 3,000-page PDFs unusable with dirty hands or in confined spaces |
| Failure code diagnosis is slow | Technicians call the office or guess — average 15–30 min diagnostic delay |
| Cloud AI requires internet | Remote job sites, underground, industrial environments have no reliable signal |

The result: technicians make errors, miss specs, and complete fewer jobs per day than they should.

---

## 3. Solution Overview

### 3.1 What Resco Voice Pilot Does

A field technician wearing work gloves, holding a tool, standing next to a Komatsu excavator can:

1. **Ask a question:** *"What's the torque spec for the final drive mounting bolts?"* → Answer spoken aloud from the indexed service manual within 2.5 seconds
2. **Log a reading:** *"Hydraulic pressure 2,800 PSI, status nominal"* → Resco form field populated automatically
3. **Look up a failure code:** *"Failure code E03, WA200-6"* → Troubleshooting procedure read aloud from Section 40 of the manual
4. **Complete a task:** *"Mark service task 3 complete, took 45 minutes"* → D365 Work Order updated
5. **Request a part:** *"Add part KOM-6732, quantity 2 to this work order"* → Inventory record created

All of the above works with zero internet connectivity.

### 3.2 The Offline Moat

Every competing AI assistant — Microsoft Copilot for Field Service, ServiceNow AI, Salesforce Einstein — requires an active internet connection. Resco Voice Pilot does not. This is not a minor differentiator; it is a category-defining capability that competitors cannot replicate without restructuring their entire cloud architecture.

---

## 4. Functional Requirements

### 4.1 Voice Interface

| ID | Requirement | Priority |
|---|---|---|
| VR-01 | System shall accept continuous voice input via device microphone | MUST |
| VR-02 | System shall process voice input using Gemma 4 E2B native audio (no separate STT required) | MUST |
| VR-03 | System shall classify intent as RESCO ACTION or MANUAL QUERY within 800ms | MUST |
| VR-04 | System shall provide spoken audio response for MANUAL QUERY results | MUST |
| VR-05 | System shall operate without any network connection | MUST |
| VR-06 | System shall support glove-touch activation (tap to speak) | MUST |
| VR-07 | System shall support wake-word activation (Phase 2) | SHOULD |
| VR-08 | System shall handle ambient noise environments (construction, machinery) | MUST |
| VR-09 | System shall support English; additional languages in Phase 2 | MUST |

### 4.2 Manual RAG (Retrieval-Augmented Generation)

| ID | Requirement | Priority |
|---|---|---|
| MR-01 | System shall index up to 3,000 pages of PDF service manuals per machine model | MUST |
| MR-02 | System shall chunk documents into 512-token segments with 50-token overlap | MUST |
| MR-03 | System shall store embeddings in SQLite-vec on-device (target: <30MB per 3,000 pages) | MUST |
| MR-04 | System shall retrieve top 3 most relevant chunks per query | MUST |
| MR-05 | System shall return RAG results within 100ms on target hardware | MUST |
| MR-06 | System shall support delta sync of manual index updates over Wi-Fi | MUST |
| MR-07 | Tablet UI shall display the source manual section alongside voice response | SHOULD |

### 4.3 Track 1 — Power Platform Actions

| ID | Requirement | Priority |
|---|---|---|
| PP-01 | System shall write to custom Dataverse tables via structured JSON commands | MUST |
| PP-02 | System shall support field-level updates (text, number, choice, boolean) | MUST |
| PP-03 | System shall queue writes when offline and sync on Wi-Fi reconnection | MUST |
| PP-04 | System shall integrate with Resco Forms+ for Power Platform | MUST |
| PP-05 | System shall trigger Power Automate flows on form completion | SHOULD |

### 4.4 Track 2 — D365 Field Service Actions

| ID | Requirement | Priority |
|---|---|---|
| FS-01 | System shall update `msdyn_workorder` entity (status, notes, completion) | MUST |
| FS-02 | System shall update `msdyn_workorderservicetask` (completion %, duration, notes) | MUST |
| FS-03 | System shall create `msdyn_workorderproduct` records (parts consumed) | MUST |
| FS-04 | System shall create `msdyn_workorderservice` records (labor/time entries) | MUST |
| FS-05 | System shall read `msdyn_customerasset` for equipment context | MUST |
| FS-06 | System shall sync all offline writes to D365 on reconnection | MUST |

---

## 5. Technical Architecture

### 5.1 Three-Layer Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    RESCO MOBILE APP                         │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              VOICE PILOT PLUGIN                     │   │
│  │                                                     │   │
│  │  [Mic Input] → Gemma 4 E2B → Intent Classifier     │   │
│  │                      │                             │   │
│  │          ┌───────────┴───────────┐                 │   │
│  │          ▼                       ▼                 │   │
│  │   RESCO ACTION            MANUAL QUERY             │   │
│  │   JSON function call      SQLite-vec retrieval     │   │
│  │   → Resco Rules Engine    → Top 3 chunks           │   │
│  │   → Dataverse / D365 FS   → Gemma response        │   │
│  │                           → Spoken aloud           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Resco Sync Engine ──────────────────→ Cloud on Wi-Fi      │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 AI Layer — Gemma 4 E2B

| Property | Value |
|---|---|
| Model | Google Gemma 4 E2B (Edge 2B) |
| Parameters | 2 billion |
| Audio support | Native (no separate STT required) |
| Quantization | 4-bit (Q4_K_M) |
| Model size on disk | 3.2 GB |
| Android runtime | MediaPipe LLM Inference API |
| iOS runtime | llama.cpp with Metal backend (GGUF format) |
| Context window | 128K tokens |
| Function calling | Supported natively |

### 5.3 Knowledge Layer — Local RAG Pipeline

```
Server-Side (one-time ingestion):
PDF Files → Text Extraction (OCR) → Chunking (512 tokens, 50 overlap)
→ Embedding (MiniLM-L6-v2, 384 dimensions) → SQLite-vec database

On-Device (runtime):
Voice query → Embed query → Cosine similarity search → Top 3 chunks
→ Inject into Gemma context → Generate response
```

| Property | Value |
|---|---|
| Embedding model | all-MiniLM-L6-v2 (80 MB) |
| Vector dimensions | 384 |
| Index size (3,000 pages) | ~25 MB |
| Retrieval latency | <50ms on-device |
| Vector store | SQLite-vec (SQLite extension, zero infrastructure) |
| Chunk size | 512 tokens with 50-token overlap |
| Estimated chunks (3,000 pages) | ~15,000 chunks |

### 5.4 Action Layer — Resco Integration

**Track 1 — Power Platform action output schema:**
```json
{
  "action": "update_record",
  "table": "cr8a2_inspection_items",
  "record_id": "current",
  "fields": {
    "cr8a2_pressure_reading": "2800",
    "cr8a2_unit": "PSI",
    "cr8a2_status": "Nominal"
  }
}
```

**Track 2 — D365 Field Service action output schema:**
```json
{
  "action": "update_service_task",
  "work_order_id": "WO-2024-1847",
  "service_task_id": "ST-003",
  "fields": {
    "msdyn_percentcomplete": 100,
    "msdyn_actualduration": 45,
    "notes": "Hydraulic pressure 2800 PSI, within spec"
  }
}
```

Gemma outputs structured JSON via function calling. The Resco plugin receives this JSON and executes the write via Resco's Rules Engine. All writes are queued locally and synced when connectivity is restored.

---

## 6. Device Specifications

### 6.1 Google Pixel 9 Pro (Demo Device — Performance)

| Spec | Value |
|---|---|
| Chip | Google Tensor G4 |
| RAM | 16 GB |
| AI Runtime | MediaPipe LLM Inference API (Tensor G4 TPU path) |
| Gemma headroom | 3.2 GB model / 12.8 GB free |
| Target latency | ~900ms end-to-end |
| Platform | Android 15 |

**Why this wins the demo:** Google made Gemma, Tensor G4, and MediaPipe. All three are co-optimized at silicon level. Gemini Nano already ships on this device — Gemma 4 E2B is the same model family, higher capability tier.

### 6.2 Apple iPhone 17 Pro (Demo Device — Cross-Platform)

| Spec | Value |
|---|---|
| Chip | Apple A19 Pro |
| Neural Engine | 16-core |
| AI Runtime | llama.cpp with Metal backend |
| Model format | GGUF 4-bit |
| Target latency | ~1.3 seconds end-to-end |
| Platform | iOS 19 |

**iOS-specific implementation notes:**
- Model stored in app's Documents directory (not iCloud synced — privacy + offline)
- Swift bridge wraps llama.cpp C API for Resco iOS plugin
- App Store compliant — local LLM inference permitted (precedent: LLM Farm, Enchanted)
- No conflict with Apple Intelligence on-device models

### 6.3 Samsung Galaxy Tab Active5 (Demo Device — Rugged / Utility)

| Spec | Value |
|---|---|
| Chip | Samsung Exynos 1380 (5nm) |
| RAM | 8 GB |
| AI Runtime | MediaPipe LLM Inference API |
| IP Rating | IP68 |
| Drop Rating | MIL-STD-810H (1.5m to concrete) |
| Battery | 10,090 mAh + hot-swap capable |
| Display | 8" outdoor-readable, glove touch mode |
| Target latency | ~2.1 seconds end-to-end |
| Platform | Android 14 |

**Tablet UX — Split-Screen Mode:**

The 8" display enables a split view unavailable on phones:
- Left pane: relevant manual section, auto-highlighted on RAG match
- Right pane: active Resco form / Work Order
- Bottom bar: voice status indicator

### 6.4 Minimum Hardware Requirements

| Tier | Minimum | Recommended |
|---|---|---|
| RAM | 8 GB | 12 GB |
| Storage free | 5 GB | 8 GB |
| Chip | Snapdragon 7 Gen 2 / Exynos 1380 / A17 | Snapdragon 8 Gen 3+ / Tensor G4 / A19 |
| OS | Android 13 / iOS 17 | Android 15 / iOS 19 |

---

## 7. System Latency Budget

| Step | Pixel 9 Pro | iPhone 17 Pro | Tab Active5 |
|---|---|---|---|
| Audio capture + model warm | 150ms | 200ms | 300ms |
| Intent classification (Gemma) | 500ms | 800ms | 1,200ms |
| RAG retrieval (if manual query) | 40ms | 45ms | 50ms |
| Response generation | 210ms | 255ms | 550ms |
| **Total** | **~900ms** | **~1.3s** | **~2.1s** |

All targets are within the 3-second UX threshold for conversational interfaces.

---

## 8. Security & Compliance Architecture

| Concern | Approach |
|---|---|
| Data sovereignty | All inference on-device — no data leaves the device at query time |
| HIPAA | No PHI transmitted to cloud; local storage encrypted via device OS |
| CJIS | Criminal justice data never touches commercial cloud AI |
| ITAR | Defense/manufacturing manuals processed entirely on-device |
| Model integrity | Gemma model hash verified at install and on each app launch |
| Sync encryption | All Resco sync traffic encrypted in transit (TLS 1.3) |
| Manual index | Pre-processed server-side; index only (no raw text) stored on device |

---

## 9. Manual Ingestion Pipeline

### 9.1 Server-Side Processing (one-time per manual revision)

```
Raw PDFs → OCR (Azure Form Recognizer or Tesseract)
        → Text normalization (section headers preserved)
        → Chunking (512 tokens, 50-token overlap, section-aware)
        → Embedding (MiniLM-L6-v2)
        → SQLite-vec .db file packaged per machine model
        → Delta generation for updates
        → Delivered via Resco Sync Engine
```

### 9.2 Device-Side Storage

| Component | Size |
|---|---|
| Gemma 4 E2B (4-bit) | 3.2 GB |
| MiniLM embedding model | 80 MB |
| Manual index (3,000 pages, 1 machine model) | ~25 MB |
| Manual index (full fleet, 10 models) | ~250 MB |
| **Total (single model)** | **~3.3 GB** |
| **Total (full fleet)** | **~3.5 GB** |

---

## 10. Deployment Architecture

### 10.1 Resco Plugin Package

The Voice Pilot ships as a single Resco plugin package containing:
- Gemma 4 E2B runtime (Android: MediaPipe; iOS: llama.cpp)
- MiniLM embedding model
- SQLite-vec library
- Voice Pilot business logic layer
- Action schema definitions (Track 1 or Track 2 variant)

### 10.2 Admin Configuration

Administrators configure via Resco Woodford (no-code admin UI):
- Which Dataverse tables/fields are voice-writable
- Which machine models' manual indexes are deployed to which users
- Voice action allow/deny lists
- Sync schedule for manual index updates

### 10.3 Update Model

- **Model updates:** Delivered as app updates (Gemma model version pinned per release)
- **Manual index updates:** Delta sync via Resco Sync Engine on Wi-Fi
- **Action schema updates:** Delivered via Resco Woodford configuration push

---

## 11. Acceptance Criteria — V1

| Criterion | Target |
|---|---|
| End-to-end latency (Pixel 9 Pro) | < 1.5 seconds |
| End-to-end latency (Tab Active5) | < 3 seconds |
| Manual Q&A accuracy (top-3 retrieval) | > 85% relevant response |
| Form fill accuracy | > 95% correct field mapping |
| Offline reliability | 100% — no cloud dependency at query time |
| Battery impact | < 15% additional drain per 8-hour shift |
| Cold start time (model load) | < 8 seconds |
| App store compliance | Apple App Store + Google Play approved |

---

## 12. Out of Scope — V1

- Wake word detection (Phase 2)
- Multi-turn conversation memory across sessions
- Diagram/schematic rendering from manuals
- Video content in manuals
- Custom model fine-tuning
- Languages other than English
- Windows/web deployment
