# Resco Voice Pilot — Executive Briefing

**Date:** April 2026 | **Classification:** CONFIDENTIAL | **Prepared for:** Resco CEO

---

## The Problem Worth Solving

Field technicians servicing heavy equipment, utility infrastructure, and industrial systems carry the knowledge they need — buried in 3,000-page service manuals and enterprise systems they cannot access with their hands full. A gloved tech troubleshooting a hydraulic fault on a Komatsu excavator cannot thumb through a PDF. A utility worker in a substation cannot type into a form while managing live equipment. An oil field technician forty miles from the nearest cell tower gets no help from a cloud AI assistant.

The result is predictable: technicians guess, call the office, make errors, and complete fewer jobs per day than they should. First-time fix rates suffer. Labor hours accumulate on avoidable rework. Equipment downtime extends past its necessary minimum.

This is not a workflow problem. It is a fundamental mismatch between how enterprise software is built — for office workers with two free hands and a reliable internet connection — and where the actual work happens.

---

## The Opportunity

**Resco is the only enterprise field service platform positioned to solve this.** The reason is architectural: Resco's offline-first mobile platform is the only enterprise system that can host a locally-running AI model and deliver results without internet connectivity. Every competing AI solution — Microsoft Copilot for Field Service, Salesforce Einstein, ServiceNow AI — requires an active cloud connection. In the environments where Resco's customers work, that connection often does not exist.

The field service management market is valued at $4.4 billion today and growing at 13.3% CAGR toward $11.8 billion by 2030. That is the narrow view. The full addressable market for offline-capable AI in regulated and disconnected workforces — which includes defense and military maintenance (ITAR-restricted), law enforcement (CJIS-restricted), healthcare field workers (HIPAA-restricted), social services (HIPAA), and field science — is estimated at $35 billion by 2030. What unites every one of these verticals is a single forcing function: their data cannot leave the device, or their device cannot reach the cloud.

Resco Voice Pilot is the product that captures this market. The technology to build it exists today.

---

## The Recommended Solution

**Resco Voice Pilot** embeds Google's Gemma 4 E2B — a 2-billion-parameter AI model with native audio understanding — directly into the Resco mobile application. The model runs entirely on the device. No cloud call is made at inference time. No data leaves the device during a query.

A field technician activates it with a tap and speaks naturally:

> *"What's the torque spec for the final drive mounting bolts?"*

Within 2.5 seconds, the answer is spoken back from the indexed service manual. The technician did not touch the screen. Did not remove a glove. Did not call the office.

> *"Log hydraulic pressure 2,800 PSI, status nominal."*

The Resco form field is populated. The record queues for sync. When the truck returns to yard Wi-Fi, it uploads automatically.

The solution ships in two deployment configurations: a **Power Platform** track for organizations using Microsoft 365 without full D365 Field Service licensing, and a **D365 Field Service** track for the premium tier with Work Order, Service Task, and Booking entity integration. The underlying AI core is identical — only the data model mapping differs.

Three demo devices prove the breadth of deployment:
- **Google Pixel 9 Pro:** Sub-1-second response on Google's own hardware stack
- **Apple iPhone 17 Pro:** Full iOS support for cross-platform coverage
- **Samsung Galaxy Tab Active5:** IP68, MIL-STD-810H rated, for utility and industrial environments

---

## The Competitive Position

Microsoft's Copilot for Field Service is the most obvious competitor. It requires internet connectivity, processes data through Azure OpenAI (data leaves the device), and does not support service manual ingestion. It costs approximately $30/user/month as an add-on. Resco Voice Pilot does not have these constraints because it does not have Microsoft's architecture.

This is not a feature gap that Microsoft can close with a software update. Changing from a cloud-inference model to on-device inference requires rebuilding the product from the ground up. Resco has the time and the platform advantage to make this market its own before competitors can respond.

---

## The Business Case

**Pricing (recommended):**
- **OOB Cloud-Assisted Tier:** $45/user/month — cloud AI enhanced, internet required, entry-level
- **On-Device AI Tier:** $125/user/month — fully offline, on-device Gemma, data sovereign

**Year 1 target (existing Resco base, 15% attach rate):**
- 24,000 users × $125/month = **$36M ARR** at full On-Device tier
- Blended (50/50 tier split): **~$20M ARR**

**Customer ROI justification at $125/month:**
- 30 minutes saved per technician per day on manual lookup and form entry
- 250 working days × 0.5 hours × $40 fully-loaded hourly rate = **$5,000 saved per technician per year**
- Annual cost of Voice Pilot: **$1,500 per technician**
- **Customer ROI: 3.3:1 in Year 1**

This is a product that pays for itself in under four months. That is a fundable, sellable number.

---

## The Ask

Fund a **10-week proof-of-concept** to deliver a working demo on three devices against a real Resco environment and real service manual corpus. This is not a research project — the AI stack (Gemma, MediaPipe, llama.cpp, SQLite-vec) is open-source and production-ready today. The investment is engineering time to integrate it into Resco's existing plugin architecture.

A working demo answers the only question that matters before a full product commitment: *Does it feel fast enough to be useful in the field?* Based on the hardware capabilities of the target devices and the published performance characteristics of Gemma 4 E2B, the answer is yes. The proof-of-concept proves it on Resco iron, with Resco data.

**The 10-week investment de-risks the full product commitment.** Build the proof-of-concept. Show it to three field service customers. Get their reaction. Then make the funding decision for V1 with real evidence in hand.

---

*Detailed Functional & Technical Design Specification and V1 Deployment Plan available as companion documents.*
