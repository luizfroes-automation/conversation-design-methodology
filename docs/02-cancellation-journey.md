# 02 — Cancellation journey

## One request, one state model

The three channels share the same states. The channel changes how the guest supplies information, not what counts as progress.

| State | What the automation knows | What it may say to the guest | What it must not say |
|---|---|---|---|
| `request_received` | A cancellation request exists | “I have received your request to cancel.” | “Your booking is cancelled.” |
| `booking_located` | A booking matches the reference or contact details | Nothing about the booking until identity is verified | Property, dates, price or policy details |
| `identity_verified` | The guest controls the contact on the booking | Booking summary and the applicable policy | That cancellation is already complete |
| `policy_resolved` | The applicable rule is unambiguous | The rule, the charge and what happens next | A promise of a refund outside the rule |
| `policy_ambiguous` | The rate, promotion or policy conflict and no single rule can be established | “I can’t decide which rule applies; I can transfer this for review.” | A final charge, refund or cancellation promise |
| `consent_bound` | The guest confirmed inside the verified session (email) or verified conversation (chat/voice) | “You have confirmed the cancellation on these terms.” | That the cancellation is already complete |
| `cancellation_unknown` | The cancellation call returned no definitive result | “I can’t confirm whether the cancellation went through; the booking must be reconciled first.” | “Cancelled”, “not cancelled” or “try again” before reconciliation |
| `cancellation_failed` | The booking system returned a definite failure | “The booking system did not cancel it.” | That the booking is cancelled |
| `cancellation_executed` | The booking system returned a successful cancellation result and a reference | “The booking system has cancelled the booking.” | That the guest has already been notified |
| `notification_sent` | The confirmation message was delivered | The cancellation reference and next steps | A different outcome than the system returned |
| `notification_failed` | The cancellation is confirmed, but delivery failed | “The cancellation is confirmed; the confirmation message did not go through.” | “Not cancelled” or a request to cancel again |

### The confirmation test

A cancellation is **confirmed by the booking system** only when all of the following are true:

1. the booking was located;
2. identity was verified;
3. the policy was resolved without ambiguity;
4. the booking system returned a successful cancellation response;
5. a cancellation reference exists.

Guest notification is a separate state. A cancellation remains confirmed if the confirmation message fails; the failure is handled as a notification incident, not as a reason to say the booking is still active or to ask the guest to cancel again. The guest-facing word “cancelled” requires the five system conditions above, not proof that the message was read.

“I have submitted the request”, “I have sent this to the team” and “your request is being processed” are not confirmation. They are truthful descriptions of an earlier state. A timeout or transport failure is `unknown` until reconciliation; it is neither success nor failure.

## Shared decision rules

| Decision | Conditions | Default action |
|---|---|---|
| **Resolve** | Booking located, identity verified, policy unambiguous, booking system available, guest has explicitly confirmed the charge and the cancellation | Execute cancellation and confirm with the system reference |
| **Clarify** | Missing or mistyped booking reference; code not received; guest has not explicitly confirmed; one channel-specific step can still be completed | Ask one focused question, explain why, and offer an alternative |
| **Transfer** | Booking not located after two attempts; identity not verified after a code resend or secure-link attempt; policy ambiguous; booking system unavailable; unknown result that cannot be reconciled; guest asks for a human; complaint, safety or vulnerability concern | Create a case, call P1 for a handover package when appropriate, route to the correct queue, keep the original conversation |
| **Unknown** | The cancellation call returned no definitive result | Hold the state as `unknown`; reconcile with the booking system before any retry or customer claim |
| **No change** | The automation cannot execute or the booking system returns a definite failure | Say explicitly that no cancellation has been confirmed and state what happens next |

A `no change` outcome is not a failure of honesty; it is the honest outcome when the system confirms that nothing changed. An `unknown` outcome is not a failure either; it is the honest state while reconciliation is pending.

The happy paths below use the **Flexible booking with no rate conflict**. The policy-ambiguity section uses the separate `SAVER-PROMO` variant; the automation does not execute that one.

## Chat flow

### Happy path — flexible booking, identity verified

```text
Guest: I need to cancel booking FF-4821-QM.
Bot:   I can help with that. Before I show the booking, I’ll send a six-digit
       code to the contact details on the booking. Would you like it by email
       or text?
Guest: Email, please.
Bot:   I’ve sent the code. Please enter the six digits when you have them.
Guest: [six-digit code]
Bot:   Thank you — identity verified. The booking is Seabrook Cabin,
       14–16 March 2027. The displayed rate is Flexible: cancellation is free
       until 28 February. Today is 2 March, so one night is chargeable and the
       breakfast add-on is non-refundable. Do you want to cancel on those terms?
Guest: Yes, cancel it.
Bot:   The booking system has cancelled the booking. Your cancellation
       reference is CAN-77821. A confirmation has been sent to the email on the
       booking. No further action is needed.
```

What the automation confirmed: identity, booking, policy and cancellation result. It recorded notification delivery separately; if the message had failed, the cancellation would remain confirmed and the failure would be handled as a notification incident. The guest did not repeat the booking reference, dates or contact details.

### Booking not located

```text
Guest: Cancel my booking.
Bot:   I can help. What is the booking reference, or the email address used for
       the booking?
Guest: FF-9999-XX
Bot:   I can’t locate a booking with that reference. Please check the reference
       in your confirmation email; it usually starts with FF-.
Guest: It’s the only one I have.
Bot:   I still can’t locate it with those details. I won’t guess at a booking
       or reveal any booking information. I can transfer you to bookings
       support with the reference you provided, or you can try the email
       address used for the booking.
Guest: Transfer me.
Bot:   I’m transferring you now with your reference and this conversation.
       Nothing has been cancelled.
```

Rules: two attempts at location, no invention, no disclosure, transfer with the details supplied and the failed attempts.

### Identity not confirmed

```text
Bot:   I’ve sent a new code to the email on the booking.
Guest: I don’t have access to that email any more.
Bot:   I can’t verify identity from this chat alone. I can send a secure link
       to the phone number on the booking instead, or transfer you to bookings
       support. I can’t show the booking or cancel it before identity is
       verified.
Guest: I can’t receive either.
Bot:   I’m transferring you to bookings support. Please have the booking
       reference and a way to confirm the contact details ready. Nothing has
       been cancelled.
```

Rules: no booking details before verification; no cancellation; one code resend and one alternative method, then transfer.

### Policy ambiguous

```text
Bot:   Identity verified. I can see Seabrook Cabin, 14–16 March 2027.
       The booking screen says “Flexible”, but the rate code is SAVER-PROMO
       and the partner promotion in the booking notes says “free cancellation”.
       Those rules disagree. I can’t decide which one applies, and I won’t
       promise a refund or a charge I can’t substantiate.
Guest: So can I cancel?
Bot:   I can transfer you to the cancellations policy team, who can review the
       rate code and the promotion. Nothing has been cancelled yet.
Guest: Please transfer.
Bot:   Transferring now with the booking, the rate code, the promotion note and
       this conversation.
```

Rules: state the conflict, do not choose the friendlier rule, transfer to policy review, no cancellation until the policy is resolved.

### Booking system unavailable

```text
Guest: Yes, cancel it.
Bot:   I tried to submit the cancellation, but the booking system did not
       respond. The result is unknown: the booking may or may not have changed.
       I can’t say it is cancelled, and I can’t offer a retry until the booking
       has been reconciled.
Guest: What now?
Bot:   I can transfer you to a human who can reconcile the booking, or hold the
       case for that check. Your case reference is CASE-3317. I won’t offer a
       retry until we know whether the cancellation went through.
```

Rules: keep the state `unknown`; reconcile with the booking system before any retry or customer claim. A retry is offered only after reconciliation shows that no cancellation was applied. If reconciliation is not possible, transfer.

### Cancellation confirmed, notification failed

```text
Bot:   The booking system has cancelled the booking. Your cancellation
       reference is CAN-77821. The confirmation email did not go through.
       The cancellation is confirmed; the message delivery failed.
Guest: Should I cancel again?
Bot:   No. The booking is already cancelled. I can resend the confirmation,
       send it to another contact, or transfer you to a human. You do not need
       to cancel again.
```

Rules: the cancellation is confirmed by the system and remains confirmed; the notification failure is a separate incident. Do not describe the booking as active and do not ask the guest to cancel again.

## Voice flow

### Happy path — caller cannot use the app

```text
Agent: Fable & Fern Stays, how can I help?
Guest: I need to cancel a booking.
Agent: I can help. What is the booking reference, or the phone number used for
       the booking?
Guest: FF-4821-QM.
Agent: Thank you. I can send a code to the phone number on the booking, or a
       secure link. Which is easier for you?
Guest: Text me the code.
Agent: I’ve sent a six-digit code. Please read it back when you have it.
Guest: [reads the six digits].
Agent: Thank you. I’ve verified your identity. I have Seabrook Cabin,
       14 to 16 March 2027. The displayed rate is Flexible. Cancellation is
       free until 28 February; today is 2 March, so one night is chargeable and
       the breakfast add-on is non-refundable. Would you like me to cancel on
       those terms?
Guest: Yes.
Agent: The booking system has cancelled it. Your cancellation reference is
       CAN-77821. I’ve sent the confirmation to the contact on the booking.
```

Voice-specific rules: read back the reference and dates; never ask for a full card number; allow barge-in; do not disclose booking details while identity is unverified; offer a secure link when the caller cannot receive a code.

### Voice — booking not located

Ask for the reference or the email address used. Read the reference back character by character. After two attempts, transfer to bookings support. Do not guess a property or date from the caller’s description.

### Voice — identity not confirmed

Offer a code resend once and a secure link once. If neither works, transfer. Do not read the booking back to “help the caller recognise it”; that would disclose it to an unverified caller.

### Voice — policy ambiguous

Read the two conflicting facts exactly as they appear, state that the automation cannot decide, and transfer to policy review. Do not paraphrase the promotion as a promise.

### Voice — system unavailable

State that the booking system did not respond and that the cancellation result is `unknown`; the booking may or may not have changed. Do not offer a retry until reconciliation confirms that no cancellation was applied. Offer a transfer or hold the case for reconciliation, and read back the case reference slowly.

## Email flow

Email is asynchronous and the thread is the visible record. The automation must label the state in the reply so that “request received” cannot be mistaken for “cancelled”.

### Happy path — Flexible booking without a rate conflict

This is the no-conflict variant: the rate code is also Flexible, and no partner promotion conflicts with it. The `SAVER-PROMO` ambiguity is handled separately and is not executed.

```text
Subject: Your cancellation request — FF-4821-QM

We have received your request to cancel booking FF-4821-QM.
This is not a cancellation confirmation.

To protect your booking, verify your identity using the secure link below.
The link expires in 30 minutes:
[Secure verification link]

Once verified, you will review the applicable charge and confirm the
cancellation on the secure page. A reply to this email is not a confirmation.
No change has been made to your booking yet.
```

After verification, the automation sends the consent message:

```text
Subject: Review and confirm your cancellation — FF-4821-QM

Your identity is verified for this request. Use the secure page below to review
the charge and confirm the cancellation. This email is not the consent step.

Booking: Seabrook Cabin, 14–16 March 2027.
Displayed rate: Flexible.
Cancellation charge: one night, because the request is after 28 February.
The breakfast add-on is non-refundable.

[Secure confirmation page]

No change has been made to your booking yet. If you reply CONFIRM without using
the secure page, we will ask you to verify again: a reply alone is not linked to
the verified session.
```

The guest reviews the charge and confirms in the verified session:

```text
[Secure page — verified session for FF-4821-QM]

Booking: Seabrook Cabin, 14–16 March 2027.
Displayed rate: Flexible.
Cancellation charge: one night; breakfast non-refundable.

[Confirm cancellation]   [Keep booking]

Guest: (selects Confirm cancellation)
```

Only then does the automation execute the cancellation and send:

```text
Subject: Cancellation confirmed — FF-4821-QM

Your booking at Seabrook Cabin, 14–16 March 2027, has been cancelled.
Cancellation reference: CAN-77821.
Charge: one night, as the Flexible rate applies after 28 February.
The breakfast add-on is non-refundable.
A confirmation is attached to this thread. No further action is needed.
```

Rules: the first email is a request acknowledgement, not a confirmation; the consent email states the charge and sends the guest to the secure page; consent is the action taken inside the verified session, not a plain `CONFIRM` reply; if the session is missing or expired, the guest must verify again; identity requires the secure link or a code, not the From address alone; the subject line changes only after the system confirms; the thread keeps the full record.

### Email — CONFIRM without a verified session

A plain `CONFIRM` email is not consent. It is only a request to proceed; it must be bound to the session that completed identity verification.

```text
Guest email: CONFIRM

Automatic reply:
We received your CONFIRM, but it is not linked to a verified session for this
request. No change has been made to your booking.

To continue, use the secure link below to verify your identity and review the
charge on the secure page. A reply to this email is not enough on its own.
[Secure verification link]

If you cannot use the link, we can transfer you to bookings support.
```

Rules: do not cancel; do not disclose booking details to an unverified sender; ask for verification; after the verification step, the guest must confirm on the secure page. If verification cannot be completed, transfer.

### Email — booking not located

Reply asking for the booking reference or the email address used. Do not quote any booking details. After two attempts, reply that the booking cannot be located with the details provided and transfer the thread to bookings support.

### Email — identity not confirmed

Send one secure link. If it expires or is not used, send one replacement only if the request is still active; otherwise transfer. Never send the booking summary to an unverified address.

### Email — policy ambiguous

Reply with the two conflicting facts, state that the policy team must review them, and transfer the thread. Do not say “you can cancel free of charge” or “you will be charged one night” as a final answer.

### Email — system unavailable

Reply that the booking system did not respond and that the cancellation result is `unknown`; the booking may or may not have changed. Give a case reference and state whether the thread has been transferred or is held for reconciliation. Do not offer a retry until reconciliation confirms that no cancellation was applied. Do not send a confirmation template.

### Email — cancellation confirmed, notification failed

If the booking system confirms the cancellation but the confirmation email cannot be delivered, keep the case open and retry delivery or use another contact route. State that the cancellation is confirmed and the message delivery failed. Do not send a “request received” or “not cancelled” message, and do not ask the guest to cancel again.

### Email — documental walkthrough

This is a documentary simulation. The repository contains no implementation that executes these cases; the table records the expected state at each step.

| Scenario | Step | Expected state | Expected observable |
|---|---|---|---|
| Happy path | Request arrives | `request_received` | Acknowledgement says it is not a cancellation confirmation |
| Happy path | Secure link completed | `identity_verified` | Session is bound to the booking; no details shown before this point |
| Happy path | Consent page reviewed | `policy_resolved` | Charge is displayed; no change made |
| Happy path | Guest confirms on the secure page | `consent_bound` | Consent is tied to the verified session; execution is authorised |
| Happy path | Booking system returns success | `cancellation_executed` | Cancellation reference exists |
| Happy path | Confirmation email delivered | `notification_sent` | Reference delivered; thread closed |
| Plain `CONFIRM` | Email arrives without a verified session | `verification_pending` | No change; reply asks the guest to verify |
| Plain `CONFIRM` | Guest completes verification and confirms on the secure page | `identity_verified` → `consent_bound` | Consent is now bound; execution may proceed |
| Plain `CONFIRM` | Guest cannot verify | `verification_pending` → transfer | No change; transfer with the thread |
| Ambiguous policy | Identity verified | `identity_verified` | Booking visible; rate conflict identified |
| Ambiguous policy | Policy checked | `policy_ambiguous` → transfer | No execution; no final charge promise |
| System unavailable | Guest confirms; call returns no result | `cancellation_unknown` | No retry; no success or failure claim |
| System unavailable | Reconciliation shows cancellation applied | `cancellation_executed` | Reference exists; no retry |
| System unavailable | Reconciliation shows no cancellation | `cancellation_failed` | Retry may be offered under policy |
| Notification failure | System confirms; email delivery fails | `notification_failed` | Booking remains cancelled; delivery is followed up separately |

## Channel parity checks

Before a journey is considered coherent, the same synthetic request should produce the same state and the same policy answer in all three channels:

| Check | Chat | Voice | Email |
|---|---|---|---|
| Booking details hidden before identity | Yes | Yes | Yes |
| One-time code or secure link | Yes | Yes | Yes |
| Explicit confirmation before execution | Yes | Yes | Yes |
| Consent bound to verification | Yes, in conversation | Yes, in conversation | Yes, secure session |
| Confirmation only after system response | Yes | Yes | Yes |
| Ambiguous policy transferred, not guessed | Yes | Yes | Yes |
| System failure never described as success | Yes | Yes | Yes |
| Unknown result reconciled before retry | Yes | Yes | Yes |
| Notification failure separated from cancellation | Yes | Yes | Yes |
| Verification code or link never sent to P1 | Yes | Yes | Yes |
| Original conversation retained by the channel | Yes | Yes | Yes |
| Human receives context, not a request to repeat | Yes | Yes | Yes |

## What the automation can confirm at each step

- **Request received:** only that the request exists and has been logged.
- **Booking located:** that a match exists in the booking system; not that the caller owns it.
- **Identity verified:** that the guest completed the code or secure-link check.
- **Policy resolved:** that one rule applies and the automation can quote it.
- **Cancellation unknown:** that the cancellation call returned no definitive result; the booking must be reconciled before a retry or a customer claim.
- **Cancellation executed:** that the booking system returned a successful cancellation result and a reference.
- **Notification sent:** that the channel delivered the confirmation message to the contact on the booking.
- **Notification failed:** that the cancellation is confirmed but delivery failed; this is a separate incident and does not change the booking state.

If any step is missing, the automation must not skip ahead to a later sentence.
