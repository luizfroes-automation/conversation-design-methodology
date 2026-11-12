# 04 — Success criteria

## What “success” means for this Core

Success is not “the bot completes the cancellation quickly”. It is that the guest gets an **honest, consistent state** in every channel, and that a human can continue without making the guest start again.

The criteria below are observable in a manual walkthrough. They are not measured production results.

## Criteria

| ID | Criterion | Observable evidence | Pass condition |
|---|---|---|---|
| SC-01 | Correct completion | Booking system shows the booking cancelled; guest receives a cancellation reference | Cancellation exists in the system and the reference matches the confirmation sent |
| SC-02 | No false confirmation | Guest is never told “cancelled” before the booking system returns success | No confirmation phrase in any transcript, email or voice script before a successful system response |
| SC-03 | No silent failure | When the system is unavailable or the result is unknown, the guest is told that no cancellation is confirmed | The words “not confirmed” or equivalent appear, with a next step |
| SC-04 | Identity before disclosure | Booking details are hidden until verification | No property, date, price or policy detail before the verified state |
| SC-05 | Policy integrity | Ambiguous policy is transferred, not interpreted | No automation promise of refund or charge when the rate code and promotion conflict |
| SC-06 | Request is not confirmation | The acknowledgement distinguishes request from confirmed cancellation | The acknowledgement contains a clear “not a cancellation confirmation” statement |
| SC-07 | Continuity after transfer | Human receives the original conversation and the collected context | Human does not ask for booking reference, dates, contact details or reason again |
| SC-08 | Channel parity | The same scenario reaches the same state and policy answer in chat, voice and email | State and policy outcome match across the three channels |
| SC-09 | Explicit confirmation | The automation asks the guest to confirm the charge and the cancellation before executing | A confirmation turn is present in every channel before the cancellation call |
| SC-10 | Safe retry boundary | A failed or unknown cancellation is not retried blindly | No second cancellation call after an unknown result without a human decision |
| SC-11 | P1 boundary | The handover package is not treated as a cancellation result | P1 `handoff_needed: true` never appears as “cancelled” in the customer record |
| SC-12 | Data minimisation | Only synthetic data appears in examples and tests | No real name, booking, contact, employer or production detail |

## Manual test cases

Each case is run in the channel named. “Pass” means every expected observation is true.

### Chat

| ID | Setup | Expected observable result |
|---|---|---|
| CHAT-01 | `FF-4821-QM`, Flexible, identity verified, 12 days before check-in, guest confirms | Booking cancelled; cancellation reference shown; one-night charge stated; breakfast non-refundable stated |
| CHAT-02 | Reference not found after two attempts | No booking details disclosed; transfer to bookings support; “nothing has been cancelled” |
| CHAT-03 | Identity code fails twice; guest cannot use the alternative | No booking details; no cancellation; transfer to bookings support |
| CHAT-04 | Displayed Flexible, rate code `SAVER-PROMO`, promotion says free cancellation | Conflict stated; no cancellation; no refund promise; transfer to policy review |
| CHAT-05 | Booking system times out after the guest confirms | “Not confirmed”; no “cancelled”; case reference; transfer or retry option |
| CHAT-06 | Guest says “I want a human” before verification | Transfer without further identity interrogation; no booking disclosure; case context preserved |
| CHAT-07 | Guest asks to cancel but does not confirm the charge | Clarification or explicit confirmation requested; no cancellation executed |

### Voice

| ID | Setup | Expected observable result |
|---|---|---|
| VOICE-01 | `FF-4821-QM`, Flexible, code by SMS, guest confirms | Booking cancelled; reference read back; charge and breakfast rule stated |
| VOICE-02 | Caller cannot receive a code or secure link | No booking details; no cancellation; transfer to bookings support |
| VOICE-03 | Caller gives a reference that cannot be found twice | No guessing; transfer with the reference and failed attempts |
| VOICE-04 | Ambiguous rate and promotion | Conflict read accurately; no cancellation; transfer to policy review |
| VOICE-05 | Booking system unavailable | “Not confirmed”; no cancellation; case reference read back; transfer or later retry |
| VOICE-06 | Caller interrupts and asks for a human | Barge-in honoured; transfer; no forced script completion |

### Email

| ID | Setup | Expected observable result |
|---|---|---|
| EMAIL-01 | Guest emails `FF-4821-QM`; secure link; guest confirms | First reply says request received, not cancelled; after verification and confirmation, a separate confirmation email with reference |
| EMAIL-02 | Reference cannot be found after two replies | No booking details; thread transferred; “nothing has been cancelled” |
| EMAIL-03 | Secure link expires; no alternative verification | No booking details; transfer or one documented replacement link; no cancellation |
| EMAIL-04 | Ambiguous rate and promotion | Conflict stated; no final policy promise; thread transferred to policy review |
| EMAIL-05 | Booking system unavailable | Reply says no cancellation confirmed; case reference; no confirmation template |
| EMAIL-06 | Handover after verification | Human sees the full thread and the P1 package; customer is not asked to repeat the booking reference or reason |

### Handover

| ID | Setup | Expected observable result |
|---|---|---|
| HAND-01 | Ambiguous policy transferred with P1 available | Human sees package and channel context; `handoff_needed: true`; no cancellation record; customer not asked to repeat |
| HAND-02 | P1 unavailable during a transfer | Channel transfers with its own context; no package is fabricated; no cancellation claimed |
| HAND-03 | P1 returns `provider_rate_limited` fallback | Channel treats the fallback as triage context, not a `HandoverPackage`; human reviews; no cancellation claimed |
| HAND-04 | Human resolves the policy exception | Human records the decision; customer receives one clear outcome; the case shows who decided and why |

## Metrics proposed for later measurement

These are **proposed** metrics, not measured results. Targets are hypotheses until a pilot defines a baseline.

| Metric | Definition | Proposed target | Status |
|---|---|---|---|
| False-confirmation rate | Confirmed cancellations that were not confirmed by the booking system, divided by all cancellation requests | 0 in the manual suite; production target to be set after baseline | Proposed |
| Correct-completion rate | Requests that end with a system-confirmed cancellation and a delivered reference, divided by eligible requests | Baseline first; no target yet | Proposed |
| Repeat-information rate | Handovers where the human asks for information already in the case, divided by handovers | 0 in HAND-01 to HAND-04 | Proposed |
| Handover-context completeness | Required handover items present in the case, divided by handovers | 100% in the manual suite | Proposed |
| Policy-exception precision | Ambiguous-policy cases transferred rather than decided by automation, divided by ambiguous cases | 100% in the manual suite | Proposed |
| Time to first truthful state | Time from request to the first message that correctly states the state | Baseline first; no target yet | Proposed |
| Abandonment after failure | Guests who stop after a system failure without a next step, divided by failure cases | Baseline first; no target yet | Proposed |

The metrics are deliberately conservative. A high completion rate is not a success if it includes a false confirmation; a fast transfer is not a success if the human has to start again.

## Core definition of done

- The same synthetic request is designed for chat, voice and email.
- All four exception families are covered: booking not located, identity not confirmed, policy ambiguous, system unavailable.
- Each channel distinguishes request received from cancellation confirmed.
- The P1 handover uses only fields the current public schema supports and names the channel-side fields that P1 does not have.
- The original conversation is retained by the channel; the human receives it with the case.
- Manual cases cover confirmation, privacy, policy, failure and continuity.
- The documents are reviewed before any heuristic evaluation.

## Out of scope for this Core

- Formal heuristic evaluation.
- Usability testing with participants.
- Production telemetry or measured baselines.
- Implementation of chat, voice, email or booking-system integrations.
- Refund, complaint, fraud or vulnerable-customer procedures beyond a safe transfer.
