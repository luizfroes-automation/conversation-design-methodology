# 01 — Problem statement

## The user problem

A guest who wants to cancel a booking often has to work out how the request will be handled, whether it has been accepted, and what will happen next. The request can start in chat, voice or email, but the customer experience is often fragmented:

- the guest repeats the booking reference, dates, email and reason in more than one place;
- the automation confirms that it “received the request”, which the guest can read as “the booking is cancelled”;
- the cancellation policy is unclear at the point of request, or the rate code and the promotion disagree;
- the guest cannot verify identity quickly, especially on voice or email;
- the booking system is unavailable, but the customer is left unsure whether anything changed;
- a handover to a human happens without the context already collected.

The design problem is therefore not only “how do we cancel a booking?”. It is also “how do we make the state of the request honest and continuous across channels?”.

## Desired outcome

For the synthetic scenario, a guest should be able to make one cancellation request in any of the three channels and receive one of four honest outcomes:

1. **Cancelled and confirmed** — identity was verified, the policy allowed cancellation, the booking system executed it, and the guest received a confirmation with a cancellation reference.
2. **Clarification needed** — the automation needs one more piece of information, such as a booking reference, a secure-link confirmation or the guest’s explicit confirmation of the cancellation.
3. **Transferred with context** — the case goes to a human because the booking cannot be located, identity cannot be verified, the policy is ambiguous or the booking system is unavailable. The human receives the original conversation and the actions already taken.
4. **No change made** — the request could not be completed, and the guest is told clearly that no cancellation has been confirmed.

The guest should never have to infer which of these outcomes happened.

## Fictional booking and policy premises

These premises are part of the design, not measured facts about a real business.

| Premise | Value used in this Core |
|---|---|
| Company | Fable & Fern Stays (fictional) |
| Guest | Alex Morgan (synthetic) |
| Booking reference | `FF-4821-QM` |
| Property | Seabrook Cabin |
| Stay | 14–16 March 2027 |
| Rate | Flexible, with a breakfast add-on |
| Payment | £320, paid in full |
| Contact on booking | `alex.morgan@example.test`, `+44 7700 900123` |
| Flexible policy | Free cancellation until 14 days before check-in; after that, one night is charged; breakfast is non-refundable |
| Saver policy | Non-refundable |
| Ambiguity | The booking displays “Flexible”, but the rate code is `SAVER-PROMO` and a partner promotion says “free cancellation” |

The guest requests cancellation on **2 March 2027**, which is 12 days before check-in. Under the displayed Flexible rule, one night would be charged; under the rate code, nothing would be refunded. That conflict is the policy-ambiguity case.

## Actors and systems

| Actor or system | Responsibility in this design |
|---|---|
| Guest | Makes the request, verifies identity, confirms the cancellation when required, receives the outcome |
| Channel automation | Runs the conversation in chat, voice or email; collects the reference; initiates verification; explains the policy; calls the booking system; decides whether to resolve, clarify or transfer |
| Booking system | Source of truth for the booking, the rate code, the policy and the cancellation result |
| Identity service | Sends and checks the one-time code or secure link; confirms that the guest controls the contact on the booking |
| Channel orchestration | Applies the shared rules, calls P1 when a handover package is needed, and hands the case to the contact-centre routing layer |
| P1 handover service | Produces a `HandoverPackage` from a transcript; it does not cancel, queue, transfer or store the conversation |
| Human agent | Receives the case, the original conversation and the handover package; makes any policy exception decision |

## Assumptions

- The booking system exposes a read operation for the booking and a cancellation operation that returns an explicit result.
- The identity service can send a one-time code to the email address or phone number already on the booking.
- A successful cancellation returns a cancellation reference and changes the booking status; a failed or timed-out call does not.
- The channel keeps the original conversation. P1 may receive a transcript snapshot, but it does not become the system of record.
- A human agent can correct the suggested team; P1’s `suggested_team` is advice, not an irreversible routing decision.
- The guest may ask for a human at any point; that request is honoured without requiring the automation to finish its script.

## Non-goals

- Designing a new cancellation policy or deciding refund eligibility outside the stated rules.
- Replacing the booking system, payment provider, identity provider or contact-centre platform.
- Handling complaints, chargebacks, fraud investigations or vulnerable-customer escalation beyond a safe transfer.
- Measuring real conversion, cost or satisfaction.
- Producing a formal heuristic evaluation; that belongs to the Enhancement.

## Open questions for review

1. Is one-time code to the contact on the booking sufficient for chat and voice, or should high-value bookings require a second factor?
2. For email, should a reply from the registered address plus a secure link be the minimum, or is the link alone sufficient?
3. When the policy is ambiguous, should the automation be allowed to quote the displayed rule as “the rule I can see” without implying it is final?
4. How many failed location or verification attempts should trigger a transfer rather than another clarification loop?
5. Which team should receive a policy ambiguity in P1’s current `suggested_team` enum: `billing` or `escalations`?

These questions are deliberately left open for review; the Core states a recommendation where the journey needs a default.
