---
id: turo-booking-agent
name: Turo Booking Agent
version: 0.8.0
description: Turo booking requests parsed from Gmail, evaluated against host rules, with Slack notifications and one-tap confirm deep links.
category: reflex
requires: [credential-gateway]
secrets:
  - name: CLAWVISOR_URL
    description: ClawVisor gateway URL (Option A — recommended, handles OAuth for you)
    where: https://clawvisor.com — create an agent, activate Gmail service
  - name: CLAWVISOR_AGENT_TOKEN
    description: ClawVisor agent token (Option A)
    where: https://clawvisor.com — agent settings, copy the agent token
  - name: GOOGLE_CLIENT_ID
    description: Google OAuth2 client ID (Option B — direct Gmail API access)
    where: https://console.cloud.google.com/apis/credentials — create OAuth 2.0 Client ID
  - name: GOOGLE_CLIENT_SECRET
    description: Google OAuth2 client secret (Option B)
    where: https://console.cloud.google.com/apis/credentials — same page as client ID
  - name: SLACK_WEBHOOK_URL
    description: Slack incoming webhook URL for booking notifications
    where: https://api.slack.com/apps — create app, add Incoming Webhook, copy URL
health_checks:
  - "[ -n \"$CLAWVISOR_URL\" ] && curl -sf $CLAWVISOR_URL/health > /dev/null && echo 'ClawVisor: OK' || [ -n \"$GOOGLE_CLIENT_ID\" ] && echo 'Google OAuth: configured' || echo 'No email auth configured'"
  - "[ -n \"$SLACK_WEBHOOK_URL\" ] && curl -sf -X POST -H 'Content-type: application/json' -d '{\"text\":\"health check\"}' $SLACK_WEBHOOK_URL > /dev/null && echo 'Slack: OK' || echo 'Slack: not configured'"
setup_time: 25 min
cost_estimate: "$0 (Gmail API free, Slack free tier)"
---

# Turo Booking Agent: Never Miss a Booking Request Again

Turo booking requests arrive by email. This agent parses them instantly, evaluates
the guest against your rules (rating, trip length, lead time), and sends a Slack
notification with the verdict and a deep link to confirm in the Turo app. You go
from "check email, open app, scroll, tap confirm" to "glance at Slack, tap link, done."

## IMPORTANT: Instructions for the Agent

**You are the installer.** Follow these steps precisely.

**Why this matters:** Turo booking requests expire. A host running 5+ vehicles gets
multiple requests per day. Missing one means lost revenue. Slow confirmation means
lower search ranking. This agent turns a 2-minute manual flow into a 5-second
glance-and-tap.

**The core pattern: code for data, LLMs for judgment.**

1. DETERMINISTIC: code pulls Turo notification emails from Gmail, parses structured
   booking data (guest name, rating, dates, vehicle, earnings). This never fails.
   Deep links are always correct. Timestamps are always accurate.
2. LATENT: you (the agent) evaluate edge cases. Guest has no reviews? Trip is 30+
   days? Vehicle has maintenance scheduled? The rules engine handles the obvious
   cases; you handle the ambiguous ones.

**Do not try to confirm bookings programmatically.** Turo has no public API for
booking confirmation. The agent detects and evaluates — the host confirms via the
Turo app using the deep link. This is intentional: the host stays in the loop for
the money decision.

**Sequential execution matters:**
- Step 1 validates Gmail access. Without it, no emails.
- Step 2 validates Slack. Without it, no notifications.
- Step 3 builds the collector. Without it, no parsing.
- Step 4 does the first collection. Without data, no rules.
- Step 5 configures host rules. Without rules, every booking is "review manually."

## Architecture

```
Gmail (Turo notification emails)
  ↓ (ClawVisor E2E encrypted gateway OR direct Google OAuth)
Booking Collector (deterministic Node.js script)
  ↓ Parses Turo emails:
  ├── data/bookings/{YYYY-MM-DD}.json     (structured booking data)
  ├── data/notifications/{YYYY-MM-DD}.md  (markdown digest for agent)
  └── data/state.json                     (known booking IDs, last check)
  ↓
Rules Engine (deterministic)
  ↓ Evaluates each booking:
  ├── AUTO-APPROVE: guest rating ≥ 4.5, trip 1-14 days, lead time ≥ 24h
  ├── REVIEW: anything outside auto-approve thresholds
  └── FLAG: no reviews, 30+ day trips, same-day pickup
  ↓
Slack Notification
  ↓ Sends:
  ├── Booking summary (guest, dates, vehicle, earnings)
  ├── Verdict (AUTO-APPROVE / REVIEW / FLAG) with reasoning
  └── Deep link: turo://trips/booking/{id} (one-tap confirm)
  ↓
Brain Update (optional, if gbrain configured)
  ├── Timeline entry on vehicle page
  └── Guest page creation (if repeat guest)
```

## Opinionated Defaults

**Turo email patterns (deterministic detection):**
- Subject: "New trip request" or "Booking request" or "wants to book"
- From: `noreply@turo.com` or `trips@turo.com`
- Contains: guest name, guest rating (stars), trip dates, vehicle name, earnings estimate

**Default host rules (configurable in `config.json`):**

| Rule | Auto-Approve | Review | Flag |
|------|-------------|--------|------|
| Guest rating | ≥ 4.5 stars | 4.0–4.4 stars | < 4.0 or no rating |
| Trip length | 1–14 days | 15–29 days | 30+ days |
| Lead time | ≥ 24 hours | 12–23 hours | < 12 hours (same-day) |
| Guest trips | ≥ 3 prior trips | 1–2 prior trips | 0 trips (new guest) |

A booking must pass ALL auto-approve thresholds to get AUTO-APPROVE.
Any single FLAG triggers FLAG. Everything else is REVIEW.

**Slack message format:**
```
🚗 New Turo Booking Request

Guest:    Sarah M. (⭐ 4.8, 12 trips)
Vehicle:  2023 Tesla Model 3
Dates:    Apr 20 – Apr 23 (3 days)
Earnings: $245

Verdict:  ✅ AUTO-APPROVE
Reason:   Rating 4.8 ≥ 4.5, 3 days in range, 72h lead time, 12 prior trips

👉 Confirm in Turo: [Open App](turo://trips)
📧 View email: [Open Gmail](https://mail.google.com/...)
```

**Deep links:**
- Turo app (iOS/Android): `turo://trips` opens the trips tab
- Turo web fallback: `https://turo.com/us/en/trips`

## Prerequisites

1. **Gmail access** via ClawVisor (recommended) or Google OAuth
2. **Slack workspace** with an incoming webhook
3. **Node.js 18+** (for the collector script)
4. **GBrain** (optional, for brain page updates)

## Setup Flow

### Step 1: Validate Gmail Access

Follow the same credential setup as the email-to-brain recipe:

#### Option A: ClawVisor (recommended)

Tell the user:
"I need your ClawVisor URL and agent token. Set up Gmail access:
1. Go to https://clawvisor.com
2. Create an agent (or use existing)
3. Activate the Gmail service
4. Create a standing task with purpose: 'Monitor Turo booking request emails
   and extract booking details for fleet management'
5. Copy the gateway URL and agent token"

Validate:
```bash
curl -sf "$CLAWVISOR_URL/health" && echo "PASS: ClawVisor reachable" || echo "FAIL"
```

**STOP until ClawVisor validates.**

#### Option B: Google OAuth2

Tell the user:
"I need Google OAuth2 credentials. Follow the same steps as the email-to-brain
recipe (create OAuth client, enable Gmail API, run OAuth flow)."

**STOP until OAuth tokens are stored.**

### Step 2: Set Up Slack Webhook

Tell the user:
"I need a Slack incoming webhook for booking notifications.
1. Go to https://api.slack.com/apps
2. Create a new app (or use existing)
3. Enable 'Incoming Webhooks'
4. Add a webhook to the channel you want notifications in (e.g., #turo-bookings)
5. Copy the webhook URL"

Validate:
```bash
curl -sf -X POST -H 'Content-type: application/json' \
  -d '{"text":"Turo Booking Agent: webhook test ✓"}' \
  "$SLACK_WEBHOOK_URL" && echo "PASS: Slack connected" || echo "FAIL"
```

**STOP until the test message appears in Slack.**

### Step 3: Build the Booking Collector

```bash
mkdir -p turo-booking-agent/data/{bookings,notifications}
cd turo-booking-agent
npm init -y
```

The collector script needs these capabilities:

1. **collect** — pull emails from Gmail matching Turo booking patterns, deduplicate
   by booking ID extracted from email body
2. **parse** — extract structured data from each Turo email:
   - Guest name, guest rating (stars), guest trip count
   - Vehicle name
   - Trip start date, trip end date, trip duration
   - Estimated earnings
   - Booking/request ID (from email body or subject)
3. **evaluate** — apply host rules to each parsed booking, assign verdict
4. **notify** — send Slack notification with summary, verdict, and deep links
5. **state tracking** — remember processed booking IDs to avoid duplicate notifications

Key design rules for the collector:
- Gmail query: `from:turo.com subject:(booking OR "trip request" OR "wants to book")`
- Parse with regex first, fall back to LLM extraction for unusual formats
- Deep links generated by CODE, not by the LLM
- All state persisted to `data/state.json`
- Output both JSON (machine-readable) and markdown digest (agent-readable)

### Step 4: Run First Collection

```bash
node turo-booking-agent.mjs collect
```

Verify:
- `ls data/bookings/` should show JSON files
- `ls data/notifications/` should show today's digest
- Slack channel should have received notifications

If no Turo emails exist yet, tell the user: "No Turo booking emails found. The
agent is configured and will detect new bookings as they arrive. You can test by
forwarding an old Turo booking email to yourself."

### Step 5: Configure Host Rules

Create `config.json` with the host's preferences:

```json
{
  "rules": {
    "min_guest_rating": 4.5,
    "min_guest_trips": 3,
    "max_trip_days": 14,
    "min_lead_time_hours": 24
  },
  "vehicles": [
    {"name": "2023 Tesla Model 3", "active": true},
    {"name": "2022 Toyota Camry", "active": true}
  ],
  "notifications": {
    "slack_webhook": "$SLACK_WEBHOOK_URL",
    "notify_auto_approve": true,
    "notify_review": true,
    "notify_flag": true
  }
}
```

Ask the user: "What are your vehicles on Turo, and do you have any booking rules?
For example: minimum guest rating, maximum trip length, or vehicles you want to
exclude from auto-approve?"

Customize `config.json` based on the user's answers.

### Step 6: Set Up Cron

Check for new bookings every 5 minutes (booking requests expire):

```bash
*/5 * * * * cd /path/to/turo-booking-agent && node turo-booking-agent.mjs collect >> /tmp/turo-booking-agent.log 2>&1
```

**Why 5 minutes:** Turo booking requests can expire in as little as 4 hours for
Instant Book, 24 hours for standard requests. 5-minute polling ensures the host
sees requests within minutes.

### Step 7: Brain Integration (Optional)

If gbrain is configured, update brain pages on each booking:

1. **Vehicle pages:** append timeline entry:
   `- YYYY-MM-DD | Booking request: {guest} for {dates} ({verdict}) [Source: Turo email]`
2. **Repeat guests:** if a guest has booked before (check brain), create/update
   a guest page with booking history
3. **Sync:** `gbrain sync --no-pull --no-embed`

### Step 8: Log Setup Completion

```bash
mkdir -p ~/.gbrain/integrations/turo-booking-agent
echo '{"ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","event":"setup_complete","source_version":"0.8.0","status":"ok"}' >> ~/.gbrain/integrations/turo-booking-agent/heartbeat.jsonl
```

Tell the user: "Turo Booking Agent is live. Every booking request gets parsed,
evaluated against your rules, and sent to Slack with a deep link to confirm.
Checking every 5 minutes. You'll never miss a booking again."

## Implementation Guide

These are production-tested patterns for parsing Turo emails.

### Turo Email Detection (Deterministic)

```
TURO_SENDERS = ['noreply@turo.com', 'trips@turo.com', 'no-reply@turo.com']
BOOKING_SUBJECTS = [/new trip request/i, /booking request/i, /wants to book/i,
                    /trip request from/i, /new booking/i]

is_turo_booking(email):
  from = email.from.toLowerCase()
  subject = email.subject || ''
  sender_match = TURO_SENDERS.some(s => from.includes(s))
  subject_match = BOOKING_SUBJECTS.some(p => p.test(subject))
  return sender_match && subject_match
```

Test BOTH sender AND subject. Turo sends many email types (trip reminders,
receipts, marketing). Only booking requests trigger the agent.

### Booking Data Extraction

```
parse_turo_booking(email_body):
  // Turo emails have a consistent HTML structure.
  // Extract from the plain text fallback for reliability.

  guest_name = match(body, /from\s+([A-Z][a-z]+(?:\s[A-Z]\.?)?)/)
  guest_rating = match(body, /([\d.]+)\s*(?:stars?|⭐|★)/)
  guest_trips = match(body, /(\d+)\s*trips?/)

  // Date parsing: "Apr 20 – Apr 23" or "April 20, 2026 - April 23, 2026"
  date_range = match(body, /(\w+\s+\d{1,2}(?:,?\s*\d{4})?)\s*[–\-]\s*(\w+\s+\d{1,2}(?:,?\s*\d{4})?)/)
  start_date = parse_date(date_range[1])
  end_date = parse_date(date_range[2])
  trip_days = diff_days(start_date, end_date)

  vehicle = match(body, /(?:your|for)\s+((?:\d{4}\s+)?[A-Z][\w\s]+?)(?:\s*\n|\s*<)/)
  earnings = match(body, /\$\s*([\d,]+(?:\.\d{2})?)/)

  return { guest_name, guest_rating, guest_trips, start_date, end_date,
           trip_days, vehicle, earnings }
```

**Non-obvious:** Turo's email format changes occasionally. The regex patterns above
cover the most common formats as of 2026. If parsing fails on a specific email,
fall back to LLM extraction:

```
llm_fallback(email_body):
  prompt = `Extract from this Turo booking email:
    - guest_name, guest_rating, guest_trips
    - start_date, end_date (ISO 8601)
    - vehicle, earnings (number only)
    Return JSON.`
  return call_llm(prompt, email_body)
```

### Rules Engine (Deterministic)

```
evaluate_booking(booking, config):
  flags = []
  reviews = []

  r = config.rules

  if !booking.guest_rating:
    flags.push('No guest rating')
  else if booking.guest_rating < 4.0:
    flags.push(f'Low rating: {booking.guest_rating}')
  else if booking.guest_rating < r.min_guest_rating:
    reviews.push(f'Rating {booking.guest_rating} below {r.min_guest_rating} threshold')

  if booking.guest_trips == 0:
    flags.push('First-time guest (0 prior trips)')
  else if booking.guest_trips < r.min_guest_trips:
    reviews.push(f'{booking.guest_trips} trips below {r.min_guest_trips} threshold')

  if booking.trip_days > 29:
    flags.push(f'Long trip: {booking.trip_days} days')
  else if booking.trip_days > r.max_trip_days:
    reviews.push(f'{booking.trip_days} days exceeds {r.max_trip_days}-day max')

  lead_hours = hours_until(booking.start_date)
  if lead_hours < 12:
    flags.push(f'Same-day pickup ({lead_hours}h lead time)')
  else if lead_hours < r.min_lead_time_hours:
    reviews.push(f'{lead_hours}h lead time below {r.min_lead_time_hours}h minimum')

  if flags.length > 0: return { verdict: 'FLAG', reasons: flags }
  if reviews.length > 0: return { verdict: 'REVIEW', reasons: reviews }
  return { verdict: 'AUTO-APPROVE', reasons: ['All thresholds passed'] }
```

### Slack Notification

```
send_slack(booking, evaluation, config):
  emoji = evaluation.verdict == 'AUTO-APPROVE' ? '✅'
        : evaluation.verdict == 'REVIEW' ? '🔍' : '🚩'

  blocks = [
    { type: 'header', text: '🚗 New Turo Booking Request' },
    { type: 'section', fields: [
      f'*Guest:*\n{booking.guest_name} (⭐ {booking.guest_rating}, {booking.guest_trips} trips)',
      f'*Vehicle:*\n{booking.vehicle}',
      f'*Dates:*\n{booking.start_date} – {booking.end_date} ({booking.trip_days} days)',
      f'*Earnings:*\n${booking.earnings}',
    ]},
    { type: 'section', text: f'*Verdict: {emoji} {evaluation.verdict}*\n{evaluation.reasons.join(", ")}' },
    { type: 'actions', elements: [
      { type: 'button', text: 'Open Turo App', url: 'turo://trips' },
      { type: 'button', text: 'Open Turo Web', url: 'https://turo.com/us/en/trips' },
    ]},
  ]

  POST config.notifications.slack_webhook, { blocks }
```

### Deduplication

```
collect():
  state = load_state()
  query = 'from:turo.com subject:(booking OR "trip request") newer_than:1d'

  emails = gmail.list(query, max=20)
  new_bookings = []

  for email in emails:
    booking_id = extract_booking_id(email) || email.id
    if booking_id in state.processed: continue

    parsed = parse_turo_booking(email.body)
    if !parsed: parsed = llm_fallback(email.body)  // regex failed, use LLM

    evaluation = evaluate_booking(parsed, config)
    send_slack(parsed, evaluation, config)

    state.processed[booking_id] = { ...parsed, ...evaluation, ts: now() }
    new_bookings.push(parsed)

  save_state(state)
  return new_bookings
```

### What the Agent Should Test After Setup

1. **Email detection:** Forward an old Turo booking email to yourself. Run collect.
   Verify it's detected as a booking (not filtered as noise).
2. **Data extraction:** Check the parsed JSON in `data/bookings/`. Verify guest name,
   rating, dates, vehicle, and earnings are all correctly extracted.
3. **Rules engine:** Create a test booking with rating 4.2, 5 trips, 3-day trip.
   Verify verdict is REVIEW (rating below 4.5 default).
4. **Slack notification:** Verify the Slack message has all fields, correct verdict,
   and working Turo deep link.
5. **Deduplication:** Run collect twice. Verify no duplicate Slack notifications.
6. **LLM fallback:** Manually corrupt a test email's format. Verify the LLM
   fallback extracts the data correctly.

## Cost Estimate

| Component | Monthly Cost |
|-----------|-------------|
| ClawVisor (free tier) | $0 |
| Gmail API | $0 (within free quota) |
| Slack (free tier) | $0 |
| LLM fallback calls (rare) | ~$0.01/mo |
| **Total** | **$0** |

## Troubleshooting

**No booking emails detected:**
- Verify Gmail query: `from:turo.com subject:(booking OR "trip request")`
- Check that booking notification emails are enabled in Turo settings
- Turo may use a different sender address — check recent Turo emails for the From address

**Parsing fails (empty fields):**
- Turo changed their email format. Check the raw email HTML/text.
- The LLM fallback should handle format changes. If it also fails, update the regex patterns.
- Forward the problematic email to the agent for manual parsing.

**Slack notifications not arriving:**
- Test webhook directly: `curl -X POST -H 'Content-type: application/json' -d '{"text":"test"}' $SLACK_WEBHOOK_URL`
- Check Slack channel permissions (webhook must have access to the channel)
- Check `/tmp/turo-booking-agent.log` for errors

**Deep links don't open Turo app:**
- `turo://trips` requires the Turo app installed on the device
- Fallback to web link: `https://turo.com/us/en/trips`
- On desktop, only the web link works
