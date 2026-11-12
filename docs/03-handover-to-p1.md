# 03 — Handover to P1

**Reference:** [handover-intelligence-portfolio](https://github.com/luizfroes-automation/handover-intelligence-portfolio) — the public README and `app/schema.py` at `cf738f2` were used for this contract.

## What P1 does, and what it does not do

The P1 service (`handover-intelligence-portfolio`) receives a transcript and returns a validated `HandoverPackage`. It does **not** execute a cancellation, queue a case, route a conversation, store the transcript or own the customer record. The P1 README is explicit that the caller chooses the review queue, keeps the conversation and deduplicates with its own identifier.

That division is the basis of this handover design:

- **P6 owns the journey:** channel state, identity, policy evidence, cancellation execution and customer communication.
- **P1 owns the structured assessment:** whether a human is needed, why, what was attempted, urgency, suggested team, confidence and flags.
- **The channel orchestration owns the transfer:** it calls P1, interprets the package and hands the case to the contact-centre routing layer.
- **The human owns the exception decision:** policy interpretation, identity exceptions and any action the automation could not complete.

## The P1 contract used here

The current schema in the public repository defines these `HandoverPackage` fields:

| Field | Type / allowed values | Meaning |
|---|---|---|
| `handoff_needed` | boolean | Whether the conversation needs a human |
| `reason` | non-empty string | Specific reason for the transfer |
| `summary` | string | What the bot already attempted |
| `urgency` | `low`, `medium`, `high` | Priority advice for the human; not a routing decision on its own |
| `suggested_team` | `billing`, `technical`, `escalations` | Suggested queue; the receiving agent may correct it |
| `confidence` | float `0.0`–`1.0` | The model’s self-declared confidence; not calibrated |
| `flags` | list of strings | Additional flags, e.g. `repeat_contact` |

The request is a `TranscriptRequest` with `messages`, each message having `role` (`bot` or `customer`) and `content`. The public contract limits content to 2,000 characters per message, 100 messages and 20,000 characters in total.

### What P1 does not support

These are **not** P1 fields, and this design does not pretend otherwise:

- `booking_reference`
- `channel`
- `cancellation_reference`
- `identity_status`
- `policy_snapshot`
- an `agent` or `system` role in the transcript

Those values stay in the channel’s case record. Some can be represented indirectly in `reason`, `summary` or `flags`, but a free-text flag is not a substitute for a structured field. The channel must not overload `reason` with a booking reference and then expect a consumer to parse it.

## Synthetic example — ambiguous cancellation policy

The guest has verified identity, the booking displays “Flexible”, the rate code is `SAVER-PROMO`, and a partner promotion says “free cancellation”. The automation has not cancelled and has not promised a charge.

### Transcript sent to P1

```json
{
  "messages": [
    {
      "role": "customer",
      "content": "I need to cancel booking FF-4821-QM."
    },
    {
      "role": "bot",
      "content": "I can help. I have sent a code to the contact on the booking; please enter it so I can verify identity."
    },
    {
      "role": "customer",
      "content": "My verification code is [verification code redacted]."
    },
    {
      "role": "bot",
      "content": "Identity verified. I can see Seabrook Cabin, 14–16 March 2027. The booking says Flexible, but the rate code is SAVER-PROMO and the partner promotion says free cancellation. I cannot decide which rule applies and I will not promise a refund or a charge."
    },
    {
      "role": "customer",
      "content": "So can you cancel it?"
    },
    {
      "role": "bot",
      "content": "I can transfer you to the cancellations policy team to review the rate code and promotion. Nothing has been cancelled yet."
    }
  ]
}
```

Before the transcript leaves the channel, verification codes, secure-link URLs, one-time tokens and any contact detail not needed for the assessment are replaced. P1 receives the **result** — “identity verified” — not the credential or the link. The channel keeps the verification evidence and the original conversation.

### Package returned by P1

```json
{
  "handoff_needed": true,
  "reason": "cancellation_policy_ambiguity",
  "summary": "Guest asked to cancel FF-4821-QM. Identity was verified. The booking displays Flexible, the rate code is SAVER-PROMO and a partner promotion says free cancellation. The automation did not cancel or promise a charge.",
  "urgency": "medium",
  "suggested_team": "escalations",
  "confidence": 0.78,
  "flags": [
    "policy_ambiguity",
    "identity_verified",
    "cancellation_not_executed"
  ]
}
```

Why these values:

- `handoff_needed: true` means the conversation needs a human. It does **not** mean the booking is cancelled.
- `reason` names the ambiguity; it is a free string in the schema, not an enum.
- `summary` records what the bot attempted, not the whole transcript.
- `urgency: medium` is advice, not a measured business priority.
- `suggested_team: escalations` is the closest queue in the current P1 enum. There is no `cancellations` team in the schema. The receiving agent may correct it.
- `confidence: 0.78` is the model’s self-declared number and is not calibrated; it must not be used as a gate for the cancellation decision.
- The flags are readable by a human but are not a typed contract; the channel keeps the structured policy facts separately.

## What the human receives

The P1 package is one part of the handover. The channel orchestration sends a case context alongside it:

| Handover item | Source | Why it matters |
|---|---|---|
| Case identifier | Channel | Deduplication and audit |
| Booking reference | Channel booking lookup | The human can act without asking again |
| Channel and conversation link | Channel | The original chat, voice recording/transcript or email thread |
| Identity status and method | Identity service | Whether the guest is verified, and how |
| Policy evidence | Booking system | Displayed rate, rate code, promotion note |
| Actions already taken | Channel orchestration | What the automation attempted, including failed calls |
| Cancellation status | Booking system | `not_requested`, `unknown`, `failed` or `confirmed` with reference; `unknown` is reconciled before any retry |
| Notification status | Channel | `pending`, `sent` or `failed`; a failed notification does not undo a confirmed cancellation |
| P1 `HandoverPackage` | P1 | The structured assessment |
| Customer-facing messages sent | Channel | What the guest has already been told |

The human should not ask the guest to repeat the booking reference, dates, contact details or the reason for the request. They may ask a question that genuinely depends on the exception, such as which rule the policy team decides to apply.

## Where the original conversation remains

| Channel | System of record | What P1 receives |
|---|---|---|
| Chat | Chat transcript store | A transcript snapshot in `TranscriptRequest` form |
| Voice | Voice platform recording and transcript | A normalised transcript; the bot role represents the voice agent, the customer role represents the caller |
| Email | Email thread and case system | A transcript assembled from the thread; the bot role represents automated replies, the customer role represents the guest |

P1 does not store the conversation. The channel keeps the original and gives the human a link or case reference. If P1 returns a package, it is attached to the channel case; it does not replace the original record.

## Transfer precedence

The journey’s transfer decision has precedence over P1:

1. The **channel automation** reaches a transfer condition or the guest asks for a human. That decision is final for the channel.
2. The transfer proceeds even if P1 is unavailable or returns `handoff_needed: false`. P1 enriches the case; it does not veto a transfer the journey requires.
3. If a package is wanted, the **channel orchestration** calls P1 with the transcript. The transfer does not wait indefinitely for the package.
4. The channel uses the package, when present, plus the case context to select a queue and priority. `suggested_team` and `urgency` are advice; the routing layer performs the actual transfer.
5. The **human agent** receives the case, the original conversation and the package, if it arrived.
6. P1 does not queue, transfer, store or cancel anything.

A guest who asks for a human is transferred even if P1 returns `handoff_needed: false`. If that happens, the package is attached with a note that the journey required the transfer; the package is not used to overrule the guest. Conversely, a `handoff_needed: true` package does not cancel a booking or replace the booking-system state.

## When P1 is unavailable

The cancellation decision does not depend on P1 being available. The channel must preserve the honest state:

| P1 outcome | What the channel does |
|---|---|
| `200` with `HandoverPackage` | Attach the package and route using the case context |
| `200` with `handoff_needed: false` | Do **not** use it to cancel a journey-required transfer; attach it as advice if the case is transferred |
| `503 provider_unavailable` | No package; transfer with channel context or defer; do not claim cancellation |
| `503 provider_rate_limited` | No package; the body’s `fallback` is not a `HandoverPackage`; route for human review; do not claim cancellation |
| `502 invalid_llm_response` | No package; transfer or defer; do not claim cancellation |
| `502 content_blocked` | No package; the body’s `fallback` is a triage signal, not a `HandoverPackage`; route for human review; do not claim cancellation |
| `500 proxy_error` / `llm_configuration_error` | No package; transfer or defer; do not claim cancellation |
| `422` input rejection | Fix the transcript or escalate the channel defect; do not treat the request as a cancellation |

Both the `provider_rate_limited` and `content_blocked` branches carry a `fallback` object. For `provider_rate_limited`:

```json
{
  "assessment": "unavailable",
  "error": "provider_rate_limited",
  "urgency": "high"
}
```

For `content_blocked`:

```json
{
  "assessment": "unavailable",
  "error": "content_blocked",
  "urgency": "high"
}
```

These are triage signals for a human review; they are not a `HandoverPackage` and must not be read as a model assessment. The `invalid_llm_response` branch does not carry a fallback.

## Setting the boundary for review

The connection is deliberately narrow:

- P6 decides **when** a conversation needs a human and **what** the human needs to continue.
- P1 produces the **structured assessment** of that conversation.
- The channel keeps the **original conversation** and the **case state**.
- The human makes the **policy or exception decision**.

This is the bridge to the P1 model: the journey supplies the context and the state; P1 supplies the structured handover package. Neither substitutes for the other.
