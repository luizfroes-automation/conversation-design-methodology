# Conversation Design & Multi-Channel Methodology (P6)

**Status:** Core draft for review.
**Scope of this delivery:** conversation design artefacts only. No implementation, integration, production data or formal heuristic evaluation.

## Purpose

This project defines how one customer request can be handled coherently across **chat, voice and email**, without the customer having to repeat information and without the automation claiming more than the systems can confirm.

The Core uses a single synthetic scenario: a guest wants to cancel a booking. The same request is designed three times, once per channel, against shared rules for identity, policy, confirmation and handover.

## Scenario

The fictional company is **Fable & Fern Stays**, a small holiday-accommodation business. The guest is **Alex Morgan**. The booking is:

| Field | Synthetic value |
|---|---|
| Booking reference | `FF-4821-QM` |
| Property | Seabrook Cabin |
| Dates | 14–16 March 2027 |
| Rate | Flexible, with a breakfast add-on |
| Payment | £320 total, paid in full |
| Contact on booking | `alex.morgan@example.test`, `+44 7700 900123` |

The cancellation policy is fictional:

- **Flexible rate:** free cancellation until 14 days before check-in. After that, one night is charged. The breakfast add-on is non-refundable.
- **Saver rate:** non-refundable.
- **Ambiguous case:** the booking screen shows “Flexible”, but the rate code is `SAVER-PROMO` and a partner promotion says “free cancellation”. The automation cannot decide which rule applies.

All names, references, dates, prices, policies and dialogues are synthetic. No employer, client or production system is represented.

## The same request in three channels

| Channel | What the customer can do | What the automation can confirm | Main constraint |
|---|---|---|---|
| Chat | Send messages in the app or web chat | Booking found, identity verified, cancellation confirmed; notification tracked separately | The customer can paste a reference and complete a one-time code without leaving the conversation |
| Voice | Speak to an automated voice agent, with barge-in and transfer | Booking found, identity verified, cancellation confirmed; notification tracked separately | Identity often needs a secure link or a code read back; the caller may not have the booking reference |
| Email | Reply to an automated email thread | Booking found, identity verified, cancellation confirmed; notification tracked separately | Asynchronous; email alone is not identity; the thread is the record until a case is transferred |

The channel changes the mechanics, not the policy. A request is handled in the same order everywhere: locate the booking, verify identity, resolve the policy, execute only if allowed, then confirm from the booking system.

## Core artefacts

- [01 — Problem statement](docs/01-problem-statement.md)
- [02 — Cancellation journey](docs/02-cancellation-journey.md)
- [03 — Handover to P1](docs/03-handover-to-p1.md)
- [04 — Success criteria](docs/04-success-criteria.md)

## Reading order

1. Read the problem statement for the user problem and the fictional assumptions.
2. Read the cancellation journey for the shared state model, channel flows and decision rules.
3. Read the P1 handover to see what the human receives and which parts belong to the channel, not to P1.
4. Read the success criteria for observable tests and proposed metrics.

## Design principles

1. **A request is not a confirmation.** “I want to cancel” starts a request. A cancellation exists when the booking system returns a cancellation confirmation and reference; notifying the guest is a separate step.
2. **Confirm only what the system confirms.** The automation may say “I have found your booking” only after a match, and “your booking is cancelled” only after the cancellation response.
3. **Verify before disclosing.** No property, date, price or policy detail is revealed before identity is verified.
4. **Do not interpret an ambiguous policy.** If the rate, promotion and policy disagree, the automation states what it can see and transfers; it does not choose the friendlier rule.
5. **Transfer with context.** The human receives the original conversation, the booking reference, the verification state, the policy evidence and the actions already taken. The customer is not asked to repeat them.
6. **Channel parity.** Chat, voice and email use the same identity, policy and confirmation rules; only the interaction mechanics differ.
7. **Unknown is a state.** A timeout or transport failure is neither success nor failure; reconcile with the booking system before any retry or customer claim.
8. **Notification is not the cancellation.** If the system confirms but the message fails, the booking remains cancelled and the delivery failure is handled separately.
9. **Redact verification material.** One-time codes, secure links and tokens never enter the P1 transcript; only the verification result does.
10. **Consent is bound to verification.** In email, the confirmation is an action inside the secure session, not a plain reply. A `CONFIRM` without that session must re-verify before anything changes.

## Out of scope

- Formal heuristic evaluation; this is an Enhancement after the journey is reviewed.
- Chatbot, IVR, email or booking-system implementation.
- Real customer data, real policies, real integrations or employer systems.
- Pricing, refunds, complaints or chargeback handling beyond the cancellation journey.
- Measuring the proposed metrics; the Core defines them, it does not claim results.

## Synthetic data and safety

All dialogues use fictional people, references and systems. The design deliberately avoids real booking engines, payment providers and employer processes. Where a value would normally identify a person, the examples use reserved or example-only values.
