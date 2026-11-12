# 04 — Success criteria

## What “success” means for this Core

Success is not “the bot completes the cancellation quickly”. It is that the guest gets an **honest, consistent state** in every channel, and that a human can continue without making the guest start again.

The criteria below are observable in a manual walkthrough. They are not measured production results.

## Criteria

| ID | Criterion | Observable evidence | Pass condition |
|---|---|---|---|
| SC-01 | Correct completion | Booking system shows the booking cancelled; the reference matches the system record | Cancellation exists in the system and the reference matches; notification is recorded separately |
| SC-02 | No false confirmation | Guest is never told “cancelled” before the booking system returns success | No confirmation phrase in any transcript, email or voice script before a successful system response |
| SC-03 | No silent failure | When the system is unavailable or the result is unknown, the guest is told the state is `unknown` | The word “unknown” or equivalent appears, with a reconciliation or transfer next step |
| SC-04 | Identity before disclosure | Booking details are hidden until verification | No property, date, price or policy detail before the verified state |
| SC-05 | Policy integrity | Ambiguous policy is transferred, not interpreted | No automation promise of refund or charge when the rate code and promotion conflict |
| SC-06 | Request is not confirmation | The acknowledgement distinguishes request from confirmed cancellation | The acknowledgement contains a clear “not a cancellation confirmation” statement |
| SC-07 | Continuity after transfer | Human receives the original conversation and the collected context | Human does not ask for booking reference, dates, contact details or reason again |
| SC-08 | Channel parity | The same scenario reaches the same state and policy answer in chat, voice and email | State and policy outcome match across the three channels |
| SC-09 | Explicit confirmation | The automation asks the guest to confirm the charge and the cancellation before executing | Chat/voice: a confirmation turn in the verified conversation; email: confirmation inside the verified secure session |
| SC-10 | Safe retry boundary | An unknown result is not retried until reconciliation | No second cancellation call after an unknown result before reconciliation; a retry follows only when reconciliation shows no cancellation |
| SC-11 | P1 boundary | The handover package is not treated as a cancellation result | P1 `handoff_needed: true` never appears as “cancelled” in the customer record |
| SC-12 | Data minimisation | Only synthetic data appears in examples and tests | No real name, booking, contact, employer or production detail |
| SC-13 | Verification material redacted | The P1 transcript contains no code, secure link or token | The payload shows `[verification code redacted]` or equivalent and carries only the verification result |
| SC-14 | Unknown, executed and notified are distinct | The case records separate cancellation and notification states | `unknown` is not used as success or failure; a notification failure does not undo a confirmed cancellation |
| SC-15 | Transfer precedence | A journey-required transfer or guest request proceeds independently of P1 | Transfer happens even when P1 is unavailable or returns `handoff_needed: false`; the package is advice |
| SC-16 | Email consent binding | Email consent is tied to the session that completed identity verification | A plain `CONFIRM` reply without a verified session does not cancel; the system asks for verification |

## Manual test cases

Each case is run in the channel named. “Pass” means every expected observation is true.

### Chat

| ID | Setup | Expected observable result |
|---|---|---|
| CHAT-01 | `FF-4821-QM`, Flexible, identity verified, 12 days before check-in, guest confirms | Consent states one-night charge and breakfast rule; booking cancelled; reference shown; notification status recorded separately |
| CHAT-02 | Reference not found after two attempts | No booking details disclosed; transfer to bookings support; “nothing has been cancelled” |
| CHAT-03 | Identity code fails twice; guest cannot use the alternative | No booking details; no cancellation; transfer to bookings support |
| CHAT-04 | Displayed Flexible, rate code `SAVER-PROMO`, promotion says free cancellation | Conflict stated; no cancellation; no refund promise; transfer to policy review |
| CHAT-05 | Booking system times out after the guest confirms | State is `unknown`; no “cancelled”; case reference; no retry until reconciliation; transfer or hold |
| CHAT-06 | Guest says “I want a human” before verification | Transfer without further identity interrogation; no booking disclosure; case context preserved |
| CHAT-07 | Guest asks to cancel but does not confirm the charge | Clarification or explicit confirmation requested; no cancellation executed |
| CHAT-08 | Cancellation confirmed but confirmation message fails | Booking remains cancelled; reference retained; notification failure handled separately; resend/alternative offered; guest is not asked to cancel again |

### Voice

| ID | Setup | Expected observable result |
|---|---|---|
| VOICE-01 | `FF-4821-QM`, Flexible, code by SMS, guest confirms | Consent states one-night charge and breakfast rule; booking cancelled; reference read back; notification status recorded |
| VOICE-02 | Caller cannot receive a code or secure link | No booking details; no cancellation; transfer to bookings support |
| VOICE-03 | Caller gives a reference that cannot be found twice | No guessing; transfer with the reference and failed attempts |
| VOICE-04 | Ambiguous rate and promotion | Conflict read accurately; no cancellation; transfer to policy review |
| VOICE-05 | Booking system unavailable | State is `unknown`; no cancellation claim; no retry before reconciliation; case reference read back; transfer or hold |
| VOICE-06 | Caller interrupts and asks for a human | Barge-in honoured; transfer; no forced script completion |
| VOICE-07 | Unknown result; reconciliation shows no cancellation applied | No retry before reconciliation; after reconciliation, a retry may be offered under policy; the state is updated explicitly |
| VOICE-08 | Unknown result; reconciliation shows cancellation applied | No retry; cancellation confirmed with reference; notification status recorded separately |

### Email

| ID | Setup | Expected observable result |
|---|---|---|
| EMAIL-01 | Guest emails `FF-4821-QM`; secure link; consent page states charge; guest confirms on the secure page | First reply says request received, not cancelled; consent page states one-night charge; no execution before the secure-page confirmation; separate confirmation email with reference |
| EMAIL-02 | Reference cannot be found after two replies | No booking details; thread transferred; “nothing has been cancelled” |
| EMAIL-03 | Secure link expires; no alternative verification | No booking details; transfer or one documented replacement link; no cancellation |
| EMAIL-04 | Ambiguous rate and promotion | Conflict stated; no final policy promise; thread transferred to policy review |
| EMAIL-05 | Booking system unavailable | State is `unknown`; no cancellation confirmed; no retry until reconciliation; case reference; no confirmation template |
| EMAIL-06 | Handover after verification | Human sees the full thread and the P1 package; customer is not asked to repeat the booking reference or reason |
| EMAIL-07 | Cancellation confirmed but confirmation email fails | Booking remains cancelled; notification failure handled separately; resend or alternative contact; no re-cancellation request |
| EMAIL-08 | A plain `CONFIRM` email arrives without a verified session | No change; no booking details; reply asks the guest to verify and use the secure page, or transfer; no cancellation |

### Handover

| ID | Setup | Expected observable result |
|---|---|---|
| HAND-01 | Ambiguous policy transferred with P1 available | Human sees package and channel context; `handoff_needed: true`; no cancellation record; customer not asked to repeat |
| HAND-02 | P1 unavailable during a transfer | Channel transfers with its own context; no package is fabricated; no cancellation claimed |
| HAND-03 | P1 returns `provider_rate_limited` fallback | Channel treats the fallback as triage context, not a `HandoverPackage`; human reviews; no cancellation claimed |
| HAND-04 | Human resolves the policy exception | Human records the decision; customer receives one clear outcome; the case shows who decided and why |
| HAND-05 | Transcript contains a verification code or secure link before sending to P1 | Payload contains `[verification code redacted]` or equivalent; no code, link or token; summary carries only the verification result |
| HAND-06 | Guest asks for a human; P1 is unavailable or returns `handoff_needed: false` | Transfer proceeds; package, if present, is attached as advice; it does not veto the transfer |
| HAND-07 | P1 returns `502 content_blocked` with fallback | Fallback is treated as a triage signal, not a `HandoverPackage`; human review; no cancellation claimed |
| HAND-08 | Cancellation confirmed; notification fails | Case records cancellation and notification separately; booking stays cancelled; delivery failure is followed up; no re-cancellation request |

## Metrics proposed for later measurement

These are **proposed** metrics, not measured results. Targets are hypotheses until a pilot defines a baseline.

| Metric | Definition | Proposed target | Status |
|---|---|---|---|
| False-confirmation rate | Confirmed cancellations that were not confirmed by the booking system, divided by all cancellation requests | 0 in the manual suite; production target to be set after baseline | Proposed |
| Correct-completion rate | Requests that end with a system-confirmed cancellation and reference, divided by eligible requests | Baseline first; no target yet | Proposed |
| Notification-delivery rate | Confirmed cancellations whose reference reached the guest through an agreed route, divided by confirmed cancellations | Baseline first; no target yet | Proposed |
| Repeat-information rate | Handovers where the human asks for information already in the case, divided by handovers | 0 in all handover cases | Proposed |
| Handover-context completeness | Required handover items present in the case, divided by handovers | 100% in the manual suite | Proposed |
| Policy-exception precision | Ambiguous-policy cases transferred rather than decided by automation, divided by ambiguous cases | 100% in the manual suite | Proposed |
| Unknown-state reconciliation time | Time from an unknown cancellation result to a reconciled booking state | Baseline first; no target yet | Proposed |
| Time to first truthful state | Time from request to the first message that correctly states the state | Baseline first; no target yet | Proposed |
| Abandonment after failure | Guests who stop after a system failure without a next step, divided by failure cases | Baseline first; no target yet | Proposed |

The metrics are deliberately conservative. A high completion rate is not a success if it includes a false confirmation; a fast transfer is not a success if the human has to start again.

## Core definition of done

- The same synthetic request is designed for chat, voice and email.
- All four exception families are covered: booking not located, identity not confirmed, policy ambiguous, system unavailable.
- Each channel distinguishes request received, cancellation unknown, cancellation confirmed and notification failed.
- Email consent is bound to the verified secure session; a plain `CONFIRM` reply does not authorise a change.
- The P1 handover uses only fields the current public schema supports and names the channel-side fields that P1 does not have.
- Verification codes, secure links and tokens are removed before the transcript reaches P1.
- A journey-required transfer proceeds even when P1 is unavailable or returns `handoff_needed: false`.
- The original conversation is retained by the channel; the human receives it with the case.
- Manual cases cover confirmation, privacy, policy, failure, unknown reconciliation, notification failure and continuity.
- The email flows have a documentary walkthrough with the expected state at each step; it is not an implementation claim.
- The documents are reviewed before any heuristic evaluation.

## Out of scope for this Core

- Formal heuristic evaluation.
- Usability testing with participants.
- Production telemetry or measured baselines.
- Implementation of chat, voice, email or booking-system integrations.
- Refund, complaint, fraud or vulnerable-customer procedures beyond a safe transfer.
