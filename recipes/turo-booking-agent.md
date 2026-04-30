---
id: turo-booking-agent
name: Turo Booking Confirmation Agent
version: 0.1.0
description: Auto-confirm Turo bookings with guardrails. Polls for new trip requests, vets guests, checks calendar conflicts, confirms or escalates.
category: act
requires: []
secrets:
  - name: TURO_EMAIL
    description: Email address for your Turo host account
    where: Your Turo account settings — the email you log in with
  - name: TURO_PASSWORD
    description: Turo account password (stored in local keychain only)
    where: Your Turo account credentials
health_checks:
  - "echo 'Turo agent: check browser session validity'"
setup_time: 15 min
cost_estimate: "$0 (uses existing Turo account)"
vehicle: Tesla (single vehicle, Tesla-Turo integration active since 2026-03-06)
notification_channel: iMessage
---

# Turo Booking Confirmation Agent

Booking requests arrive. The agent vets the guest, checks for conflicts,
and auto-confirms good bookings. Borderline cases escalate to Pierre via
iMessage. Every booking is logged.

## IMPORTANT: Instructions for the Agent

**You are the booking manager.** Follow this workflow precisely.

**The core pattern: code for data, LLMs for judgment.**
1. DETERMINISTIC: Poll Turo dashboard for new trip requests, parse booking
   details (guest name, dates, price, guest rating, trip count).
   Timestamps and prices are always accurate.
2. LATENT: You (the agent) make judgment calls. Is this guest trustworthy?
   Are the dates clear? Is the pricing acceptable? Confirm or escalate.

**Do not guess booking details.** Always read them from the Turo dashboard.
If the page doesn't load or data is missing, escalate to Pierre. Never
confirm a booking you can't fully parse.

## Architecture

```
Signal Sources (check both):
  ├── Outlook (skippy@nukasoft.com) — Turo notification emails
  └── Turo Dashboard (turo.com/hosting) — trip requests tab
  ↓
Booking Parser (deterministic)
  ↓ Extracts:
  ├── guest_name, guest_rating, guest_trip_count
  ├── trip_dates (start, end)
  ├── vehicle (confirm it's the right one)
  ├── trip_price, earnings_estimate
  └── guest_profile_url
  ↓
Agent reads parsed booking
  ↓ JUDGMENT CALLS:
  ├── Guest vetting (rating, trip count, red flags)
  ├── Calendar conflict check (is the car available?)
  ├── Pricing sanity check (is the price reasonable?)
  ├── Date proximity check (is pickup imminent? > urgency)
  └── Decision: CONFIRM / DECLINE / ESCALATE
  ↓
Action Layer
  ├── Confirm on Turo (click confirm button via browser)
  ├── Notify Pierre via iMessage (summary of action taken)
  └── Log booking (local record for tracking)
```

## Confirmation Policy: Confirm with Guardrails

### Auto-Confirm When ALL of These Are True

- Guest rating >= 4.0 stars (or new guest with no rating BUT verified ID)
- Guest has completed at least 1 prior trip (OR is ID-verified new guest)
- No calendar conflict for the requested dates
- Trip price is within normal range for the vehicle
- Pickup is not within the next 2 hours (too rushed = escalate)
- Guest profile has a profile photo
- No red flags in guest messages (see Red Flags below)

### Escalate to Pierre When ANY of These Are True

- Guest rating below 4.0 stars
- Guest has negative reviews mentioning damage, cleanliness, or late returns
- Calendar shows a potential conflict (overlapping or back-to-back bookings)
- Trip is unusually long (> 14 days)
- Trip is unusually cheap (> 30% below normal daily rate)
- Pickup is within 2 hours (not enough prep time)
- Guest has no profile photo AND no prior trips
- Guest message contains red flags

### Decline (Rare — Usually Escalate Instead)

- Obvious scam patterns (requests to communicate off-platform, etc.)
- Dates that are clearly impossible (past dates, nonsensical ranges)

### Red Flags in Guest Messages

- Asks to communicate off-platform (phone, WhatsApp, email)
- Mentions someone else will be driving
- Asks about smoking in the vehicle
- Mentions using the car for commercial purposes (Uber, Lyft, delivery)
- Requests to extend the trip before it starts (fishing for discounts)
- Vague about pickup location or drop-off plans

## Step-by-Step Workflow

### Step 1: Detect New Booking Request

**Option A — Email Detection (preferred, faster):**
```
Search Outlook for new emails to skippy@nukasoft.com from Turo:
  sender contains "turo.com"
  subject contains "trip request" OR "booking" OR "wants to book"
  received within last 15 minutes
  unread
```

**Option B — Dashboard Polling (fallback):**
```
Navigate to turo.com/us/en/trips (or turo.com/hosting)
Look for trips with status "Pending" or "Requested"
Parse the trip details from the page
```

Run both. Email is the trigger; dashboard is the source of truth.

### Step 2: Parse Booking Details

Extract from the Turo trip page:
```
guest_name:       [from trip details]
guest_rating:     [stars, e.g., 4.8]
guest_trip_count: [number of completed trips]
guest_joined:     [when they joined Turo]
guest_photo:      [yes/no]
guest_verified:   [ID verified yes/no]
trip_start:       [date and time]
trip_end:         [date and time]
trip_duration:    [calculated days]
trip_price:       [total earnings]
daily_rate:       [calculated]
vehicle:          [confirm it matches your listing]
pickup_location:  [if specified]
guest_message:    [any message from the guest]
```

### Step 3: Apply Guardrails

Run through the confirmation policy above. Score the booking:

```
PASS conditions met:  [list which passed]
FAIL conditions:      [list any that failed]
RED FLAGS found:      [list any from guest message]
DECISION:             CONFIRM / ESCALATE / DECLINE
REASON:               [one-line explanation]
```

### Step 4: Take Action

**If CONFIRM:**
1. Navigate to the trip request on Turo
2. Click "Confirm" / "Accept" button
3. Send iMessage to Pierre:
   "Turo: Confirmed booking for [guest_name] ([rating] stars,
   [trip_count] trips). [start_date] to [end_date] ([duration] days).
   Earning $[price]. Guest is [one-line assessment]."

**If ESCALATE:**
1. Do NOT confirm or decline
2. Send iMessage to Pierre:
   "Turo: New booking needs your review.
   Guest: [guest_name] ([rating] stars, [trip_count] trips)
   Dates: [start_date] to [end_date]
   Price: $[price]
   Issue: [reason for escalation]
   Action needed: Confirm or decline at [turo_trip_url]"

**If DECLINE:**
1. Do NOT auto-decline (let Pierre review first)
2. Send iMessage to Pierre with recommendation to decline and why

### Step 5: Log the Booking

Create a local log entry:
```
[timestamp] | [CONFIRMED/ESCALATED/DECLINED] | [guest_name] |
[dates] | $[price] | [reason]
```

## Execution

Since this runs locally, use a simple polling loop:

```python
while True:
    try:
        check_for_new_bookings()
        time.sleep(1800)  # 30 minutes
    except Exception as e:
        send_imessage(f"Turo agent crashed: {e}")
        time.sleep(30)
```

Runs as a persistent local process via scheduled task, polls every
30 minutes, and self-recovers on crash with an iMessage alert.

## Opinionated Defaults

- **Speed matters.** Turo gives hosts a window to respond. Fast confirmation
  = higher ranking in search. Poll every 30 minutes minimum.
- **Err toward confirming.** A 4.2-star guest with 5 trips is fine. Don't
  over-screen. The guardrails catch actual problems.
- **Never decline without Pierre's input.** Escalate, don't decline.
  Pierre can always decline himself.
- **Tesla integration handles logistics.** The car locks/unlocks via Turo
  automatically. No key handoff needed. This simplifies everything.
- **Calendar is the hard constraint.** If dates conflict, always escalate.
  Don't try to resolve scheduling conflicts autonomously.

## Failure Modes

- **Turo session expires:** Re-authenticate. If 2FA is required, escalate
  to Pierre with "Turo session expired, please log in and re-authorize."
- **Email notifications not arriving:** Fall back to dashboard polling.
  The dashboard is the source of truth.
- **Guest has no rating (brand new):** Check if they're ID-verified. If yes,
  treat as 4.0 stars. If no, escalate.
- **Browser automation fails (button not found):** Screenshot the page,
  send to Pierre via iMessage with context. Never silently fail.
- **Multiple bookings arrive simultaneously:** Process sequentially. Check
  for date conflicts between the new bookings themselves, not just
  existing bookings.

## How to Verify

1. Check Turo for any pending trip requests. If one exists, run the full
   workflow manually and verify the decision matches the policy.
2. Send a test iMessage to Pierre confirming the notification format works.
3. Verify calendar conflict detection by checking a date range that has
   existing events.
4. Verify the escalation path works by simulating a low-rated guest scenario.
5. Check that the log file is being written correctly after each action.

---

*GBrain Recipe — Turo Booking Confirmation Agent v0.1.0*
