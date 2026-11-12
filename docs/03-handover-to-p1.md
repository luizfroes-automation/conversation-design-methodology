# 03 — Handover to P1

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
      "content": "Code 418205."
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
| Cancellation status | Booking system | `not_requested`, `unknown`, `failed` or `confirmed` with reference |
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

## Who executes the transfer

1. The **channel automation** reaches a transfer condition or the guest asks for a human.
2. The **channel orchestration** calls P1 with the transcript and waits for a package.
3. The channel uses `handoff_needed`, `suggested_team`, `urgency` and the case context to select a queue; the routing layer performs the actual transfer.
4. The **human agent** receives the case, the original conversation and the package.
5. P1 does not queue, transfer, store or cancel anything.

If the guest asks for a human before the automation has a package, the transfer should not wait indefinitely for P1. The channel can transfer with its own context and attach the P1 package when it arrives, provided the case record makes the package’s arrival state visible.

## When P1 is unavailable

The cancellation decision does not depend on P1 being available. The channel must preserve the honest state:

| P1 outcome | What the channel does |
|---|---|
| `200` with `HandoverPackage` | Attach the package and route using the case context |
| `503 provider_unavailable` | No package; transfer with channel context or defer; do not claim cancellation |
| `503 provider_rate_limited` | No package; the body’s `fallback` is not a `HandoverPackage`; route for human review; do not claim cancellation |
| `502 invalid_llm_response` | No package; transfer or defer; do not claim cancellation |
| `502 content_blocked` | No package; transfer or defer; do not claim cancellation |
| `500 proxy_error` / `llm_configuration_error` | No package; transfer or defer; do not claim cancellation |
| `422` input rejection | Fix the transcript or escalate the channel defect; do not treat the request as a cancellation |

The fallback body for `provider_rate_limited` carries `{"assessment": "unavailable", "error": "provider_rate_limited", "urgency": "high"}`. It is a triage object, not a model assessment; the channel must not present it as one.

## Setting the boundary for review

The connection is deliberately narrow:

- P6 decides **when** a conversation needs a human and **what** the human needs to continue.
- P1 produces the **structured assessment** of that conversation.
- The channel keeps the **original conversation** and the **case state**.
- The human makes the **policy or exception decision**.

This is the bridge to the P1 model: the journey supplies the context and the state; P1 supplies the structured handover package. Neither substitutes for the other.
