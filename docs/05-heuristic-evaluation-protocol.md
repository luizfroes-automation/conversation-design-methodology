# Documentary Heuristic Evaluation Protocol

**Status:** Draft for methodological review. The full evaluation has not started.
**Baseline:** `conversation-design-methodology` at `afa3c601060185b2f8a758e042a1b8efa55c496d` (squash of PR #1).
**Reference contract:** `handover-intelligence-portfolio` at `cf738f258efd96b3a5c825310da4561dc1c79e39`.
**Scope:** define and pilot the method only. This document does not evaluate the whole journey and does not test users.

## 1. Purpose

This protocol defines a structured, traceable and reproducible way to review the **documented** cancellation journey for chat, voice and email. It is a heuristic evaluation of design documents, not a usability test and not a conformance audit.

The method must:

- identify the exact documentary baseline;
- connect each observation to an explicit criterion and a cited passage;
- separate what the document says from a hypothesis about a user consequence;
- record non-evaluable criteria without treating them as violations;
- make a pilot finding reproducible by another evaluator.

## 2. Baseline and evidence index

### 2.1 Object of evaluation

The baseline is the conversation-design Core at commit `afa3c60`. The documents are the only evidence; there is no running chat, voice or email implementation in this repository.

| Document | Role in the evaluation | SHA-256 at `afa3c60` |
|---|---|---|
| `README.md` | Scope, scenario, channel summary, principles | `9ccc754910e11731b544ebb16872e9b5d334f9f8a88a51ab1933ed8d1a965145` |
| `docs/01-problem-statement.md` | User problem, outcomes, assumptions, actors | `c2dcac9aa712492d3ce7729c9efa0bfc903b3a35fe068e7976f5016c9b9a6633` |
| `docs/02-cancellation-journey.md` | States, dialogues, exceptions, email walkthrough | `43278dad626ce2cb90b5e6085080667b09395052e497fb5de1b8cb16f38528ca` |
| `docs/03-handover-to-p1.md` | P1 contract, transfer ownership, conversation ownership | `c6c7674e52f41d7e67f92c0111a35f45290fa8a14f0005f77c126f53ac2e3ea7` |
| `docs/04-success-criteria.md` | Criteria, manual cases, metrics | `a1d2dd1e93f74f5cfc7e8690023f83c2ed9db68f278e53ead4cdf08e69918033` |

The hashes are of the Git blobs (`git show afa3c60:<path>`), not of a working-tree checkout, so line-ending configuration does not change the baseline.

### 2.2 Evidence index

| Evidence ID | Document and section | Used for |
|---|---|---|
| `EV-01` | README, “Scenario” and “Design principles” | Synthetic scenario, channel parity, truthfulness |
| `EV-02` | `01`, “Desired outcome” and “Assumptions” | Outcomes, state model premises |
| `EV-03` | `02`, “One request, one state model” | States and transitions |
| `EV-04` | `02`, channel flows and exception sections | Dialogue and repair behaviour |
| `EV-05` | `02`, “Email — documental walkthrough” | Email state progression |
| `EV-06` | `03`, “P1 contract”, “Transfer precedence”, “When P1 is unavailable” | Handover boundary and failure handling |
| `EV-07` | `04`, criteria and manual cases | Observable expectations |

### 2.3 Baseline rules

- Every finding names the baseline commit and a document section. Line numbers are not used as the only reference because they move.
- A later commit starts a new baseline; findings are not silently carried over.
- No code, interface, recording, transcript or production system is part of this baseline.
- The document is the evidence. An absence in the document is a **documentary gap**, not proof that the behaviour is absent in a future implementation.

## 3. Method

### 3.1 Evaluation question

> Does the documented journey make the cancellation state, the available choices, the error and recovery paths, and the human handover understandable and safe for the guest and the receiving agent?

### 3.2 Unit of analysis

One unit is a **channel × journey state × documentary passage**. A finding describes one issue in one unit. A cross-channel issue is recorded once as an `X` finding and linked to the channel instances.

### 3.3 Two passes

1. **Discovery pass:** read the baseline and record observations without assigning criteria or severity. Quote or paraphrase the passage.
2. **Classification pass:** map each observation to criteria, separate observation from hypothetical consequence, assign severity and both confidence judgements, and write a recommendation.

This separation reduces the risk of fitting the observation to the first criterion that comes to mind.

### 3.4 Evidence standard

- The observation must be traceable to a quoted or closely paraphrased passage.
- The hypothetical consequence must use language such as “a guest may…”, not “users will…”.
- A finding may cite more than one criterion; it must name the primary criterion.
- If the document does not state a behaviour, record `documentary gap`, not a violation.
- No frequency, probability, conversion or impact figure is inferred.

## 4. Frameworks and criteria

### 4.1 Nielsen’s ten usability heuristics

Source: Jakob Nielsen, *10 Usability Heuristics for User Interface Design*, Nielsen Norman Group, 24 April 1994, last reviewed 30 January 2024. The heuristics are broad rules of thumb, not specific test cases.

| Code | Heuristic | Application in this review | Status |
|---|---|---|---|
| `N1` | Visibility of system status | Whether the guest can tell request, verification, unknown, confirmed and notification states apart | Evaluable |
| `N2` | Match between system and real world | Whether cancellation and charge language matches the guest’s situation | Evaluable |
| `N3` | User control and freedom | Whether the guest can stop, go back, ask for a human or decline the charge | Evaluable |
| `N4` | Consistency and standards | Whether the same state and policy language is used across channels and against the P1 contract | Evaluable |
| `N5` | Error prevention | Whether the design prevents unverified consent, false confirmation and unsafe retry | Evaluable |
| `N6` | Recognition rather than recall | Whether the guest must remember a reference, code or earlier statement unnecessarily | Evaluable in dialogue; interface memory aids not evaluable |
| `N7` | Flexibility and efficiency of use | Whether a guest can use a shorter path or a human route without losing safety | Partially evaluable |
| `N8` | Aesthetic and minimalist design | Whether messages are focused; visual hierarchy not evaluable | Partially evaluable |
| `N9` | Help users recognize, diagnose and recover from errors | Whether unknown, failure and verification errors have a plain next step | Evaluable |
| `N10` | Help and documentation | Whether policy, charge and handover explanations are available in context | Evaluable in text; help UI not evaluable |

### 4.2 ISO 9241-110 interaction principles

Source: ISO 9241-110:2020, *Ergonomics of human-system interaction — Part 110: Interaction principles*, Edition 2, 2020-05 (confirmed 2025). The public ISO page confirms the title and scope; the full standard was not consulted. The principle names below are the commonly published list, and the mapping is the project’s application, not a clause-level reading or a conformance claim.

| Code | Principle | Application | Status |
|---|---|---|---|
| `I1` | Suitability for the user’s task | The journey supports cancellation rather than exposing internal system steps | Evaluable |
| `I2` | Self-descriptiveness | The state and the next step are explicit | Evaluable |
| `I3` | Conformity with user expectations | Cancellation and charge behaviour matches the stated policy | Evaluable |
| `I4` | Learnability | A guest can understand the flow without a tutorial | Partially evaluable |
| `I5` | Controllability | The guest can choose, decline, verify or transfer | Evaluable |
| `I6` | Use error robustness | Unknown and failure states have a safe recovery path | Evaluable |
| `I7` | User engagement | The tone supports continued interaction without pressure | Partially evaluable |

### 4.3 Voice scope reference and project-proposed criteria

Source for scope: ISO 9241-154:2013, *Ergonomics of human-system interaction — Part 154: Interactive voice response (IVR) applications*, Edition 1, 2013-02 (confirmed 2026). The public ISO page confirms that the standard addresses the user interface design of IVR applications; the full standard was not consulted.

`V1`–`V4` are **project-proposed criteria**, informed by general IVR concerns such as turn-taking, confirmation, error recovery and instructions that do not rely on a screen. They are not presented as criteria taken from ISO 9241-154, and no conformance is claimed.

| Code | Project-proposed voice criterion | Application | Status |
|---|---|---|---|
| `V1` | Turn-taking and barge-in | The caller can interrupt, correct or ask for a human | Partially evaluable from script |
| `V2` | Confirmation without visual memory | Key reference, charge and next step are read back or repeated | Evaluable from script |
| `V3` | Error recovery without a screen | Failure and unknown states have an audible next step | Evaluable from script |
| `V4` | No visual-only instruction | The voice flow does not depend on reading a page | Evaluable from script |

No claim is made that the voice journey conforms to ISO 9241-154, and `V1`–`V4` are not attributed to that standard.

### 4.4 Accessibility references

Source: W3C, *Web Content Accessibility Guidelines (WCAG) 2.2*, W3C Recommendation, 12 December 2024. WCAG applies to web content; it does not directly cover telephony or a voice user interface. No conformance claim is made.

Source for a future implemented chat UI: W3C, *WAI-ARIA 1.2*, W3C Recommendation, 6 June 2023. It is informative here and not evaluable without an interface.

| Code | Accessibility concern | Relevant WCAG 2.2 reference | Documentary | Runtime |
|---|---|---|---|---|
| `A1` | Names, labels and instructions | 1.1.1, 2.4.6, 3.3.2 | P — wording exists; semantic names do not | N — no interface |
| `A2` | Error identification and suggestion | 3.3.1, 3.3.3 | E — messages can be read | N — no rendered error state |
| `A3` | Error prevention and redundant entry | 3.3.4, 3.3.6, 3.3.7 | E — safeguards described | N — no interface |
| `A4` | Time limits and re-authentication | 2.2.1, 2.2.5, 2.2.6 | P — link timing described; session implementation absent | N — no runtime |
| `A5` | Status messages | 4.1.3 | P — state messages described | N — no ARIA/live-region behaviour |
| `A6` | Keyboard, focus, contrast, target size | 2.1.1, 2.4.7, 1.4.3, 2.5.8 | N — no visual interface | N — no interface; NA for voice-only surfaces |

Voice accessibility for deaf, hard-of-hearing, speech-impaired or cognitively impaired callers cannot be established from this baseline. That is a declared gap, not a pass. Visual-only criteria are `NA` for the voice channel, not evidence of compliance.

### 4.5 Project-specific criteria

These are **own adaptations**, not standards. They exist because the journey has states that generic heuristics do not name.

| Code | Criterion | Why it exists |
|---|---|---|
| `X1` | Truthful state language | A request, an unknown result and a confirmed cancellation must not share wording |
| `X2` | Consent bound to verification | A consent action must be tied to the identity session that authorises it |
| `X3` | Repair and clarification | The guest gets one focused question and an alternative before transfer |
| `X4` | Continuity across handover | The human receives the context without asking the guest to repeat it |
| `X5` | Channel parity | The same request reaches the same state and policy answer in all channels |
| `X6` | Unknown/reconciliation discipline | A timeout is not treated as success or failure; retry waits for reconciliation |

### 4.6 Source status

| Source | What was checked | Consequence for the protocol |
|---|---|---|
| Nielsen, ten heuristics; severity ratings | Public NN/g pages read | Criteria and severity scale use the published heuristics; adaptation is stated |
| ISO 9241-110:2020 | Public ISO page: title, edition and scope; full text not consulted | Principle names and mapping are a project application, not clause-level conformance |
| ISO 9241-154:2013 | Public ISO page: title, edition and scope; full text not consulted | `V1`–`V4` are project-proposed criteria; the standard is a scope reference only |
| WCAG 2.2 | W3C Recommendation page read; no implementation tested | Accessibility criteria are future-facing references; no conformance claim |
| WAI-ARIA 1.2 | W3C Recommendation page read; no interface exists | Informative for a future chat interface; not evaluable now |

The ISO public pages are not a substitute for the paid standards. Where the full text was not consulted, the protocol does not attribute individual criteria to the standard.

## 5. Coverage matrix

### 5.1 Status codes

Each status is applied twice: once to the **documentary description** and once to the **runtime/experience**. A documentary `E` never implies a runtime `E`.

- **E — evaluable:** the evidence needed for that dimension is present.
- **P — partially evaluable:** some evidence is present, but a material part is missing.
- **N — not evaluable:** the dimension requires an interface, runtime, audio, data or user observation that the baseline does not contain.
- **NA — not applicable:** the criterion does not apply to that stage or channel.

`N` and `NA` are coverage findings; they are not violations.

### 5.2 Documentary and runtime coverage by state

Cells use `Doc` for the document and `Run` for a running implementation. Criteria are the primary set from section 4.

| Journey state | Chat | Voice | Email |
|---|---|---|---|
| Identification | `Doc E (N1,N2,N5,I1,I2,X1)`; `Run N (identity provider)` | `Doc P (V1,V2,I2,X1)`; `Run N (ASR, identity provider)` | `Doc E (N1,N2,N5,I2,X1)`; `Run N (secure session, email identity)` |
| Clarification | `Doc E (N3,N9,I5,I6,X3)`; `Run N (dialogue engine)` | `Doc P (V1,V3,I5,I6,X3)`; `Run N (ASR repair)` | `Doc E (N3,N9,I5,I6,X3)`; `Run N (thread automation)` |
| Consent | `Doc E (N5,I5,X2)`; `Run N (session binding)` | `Doc P (V1,V2,I5,X2)`; `Run N (voice session, audio)` | `Doc E (N5,I5,X2)`; `Run N (secure session)` |
| Execution | `Doc E (N1,N4,X1,X6)`; `Run N (booking API)` | `Doc P (N1,N4,V3,X1,X6)`; `Run N (booking API, audio)` | `Doc E (N1,N4,X1,X6)`; `Run N (booking API)` |
| Unknown result | `Doc E (N1,N9,I6,X1,X6)`; `Run N (booking API, reconciliation)` | `Doc P (V3,I6,X1,X6)`; `Run N (booking API, audio)` | `Doc E (N1,N9,I6,X1,X6)`; `Run N (booking API, reconciliation)` |
| Notification | `Doc E (N1,N9,X1)`; `Run N (delivery provider)` | `Doc P (V3,X1)`; `Run N (delivery, audio)` | `Doc E (N1,N9,X1)`; `Run N (delivery provider)` |
| Human transfer | `Doc E (N3,N6,N10,I5,X4)`; `Run N (routing, agent desktop)` | `Doc P (V3,I5,X4)`; `Run N (transfer, audio)` | `Doc E (N3,N6,N10,I5,X4)`; `Run N (routing, agent desktop)` |

### 5.3 Cross-cutting runtime and interface gaps

| Area | Why runtime is not evaluable | Future method |
|---|---|---|
| Visual hierarchy, contrast, target size | No rendered interface | Inspect a build; WCAG 2.2 checks |
| Keyboard, focus order, screen reader | No DOM or assistive-technology behaviour | Manual accessibility testing |
| Latency and timeouts | No runtime | Instrumented prototype |
| Speech recognition accuracy, barge-in, audio quality | No voice system or audio | Voice prototype and listening tests |
| Actual identity-provider and booking-system behaviour | No integrations | Integration test environment |
| Notification delivery | No email/SMS provider | Delivery logs and test accounts |
| Transfer routing and agent desktop | No contact-centre system | Simulated or staging routing |
| User comprehension and emotional response | No participants | Usability test or interviews, outside this protocol |

The documentary layer can still be evaluated, but it describes intent. The runtime layer remains a gap until a build exists.

## 6. Findings model

### 6.1 Required fields

| Field | Rule |
|---|---|
| `id` | `HE-<channel>-NNN`; unique within the baseline; channel is `C`, `V`, `E` or `X` |
| `type` | `observation`, `documentary_gap` or `hypothesis` |
| `channel` | `chat`, `voice`, `email` or `cross-channel` |
| `state` | One state from section 5.2, or `cross-state` |
| `baseline` | Commit SHA and document section |
| `criterion` | Primary criterion code; secondary codes allowed |
| `observation` | What the document says; quote or close paraphrase |
| `hypothetical_consequence` | A possible guest or agent consequence, explicitly not measured |
| `severity` | 0–4, from section 7 |
| `confidence_observation` | Strength of the documentary observation: `high`, `medium` or `low`, from section 8.1 |
| `confidence_interpretation` | Strength of the interpretation or consequence: `high`, `medium` or `low`, from section 8.2 |
| `recommendation` | A concrete change to the document or a future test |
| `future_validation` | What would confirm or reject the hypothesis |
| `reproduction_status` | `pending`, `confirmed`, `not_reproduced` or `requires_review` |
| `reproduction_note` | Divergence and rationale when another evaluator reads the passage differently |

### 6.2 Writing rules

- One finding, one issue. Split a finding when the recommendation differs.
- Do not merge a safe-state observation with a separate notification issue.
- Do not use “users will”; use “a guest may”.
- A `documentary_gap` names what is missing and why it matters; it does not claim the implementation lacks it.
- When the missing element is in a **message** but the rule exists in the surrounding design, cite both passages and state the location of the absence. Record a communication gap in the message, not a missing rule in the design. Do not write “the design has no delivery estimate” when the estimate is present in the order rules but one status message omits it.
- A `hypothesis` is allowed when the consequence depends on an absent interface, but `confidence_interpretation` cannot exceed `low`.
- A finding that is not reproduced by a second evaluator is **kept** with `reproduction_status` set to `not_reproduced` or `requires_review`, the divergence and the rationale. Do not delete evidence because two evaluators read the passage differently.

## 7. Severity scale

Adapted from Jakob Nielsen, *Severity Ratings for Usability Problems*, Nielsen Norman Group, 1 November 1994. Nielsen combines frequency, impact and persistence; this documentary protocol deliberately does **not** estimate frequency or probability, because the baseline cannot measure them.

| Level | Label | Operational definition | Action rule |
|---|---|---|---|
| 0 | Not a problem | The passage is consistent with the criterion; no action | Close |
| 1 | Cosmetic | Wording or presentation friction with no effect on the state or task | Optional |
| 2 | Minor | Adds clarification, repetition or uncertainty, but a safe workaround is stated | Fix in the design pass |
| 3 | Major | Can produce a wrong state, block completion or require human rescue; the document does not contain a safe recovery | Resolve before implementation |
| 4 | Critical potential | Potential false confirmation, unauthorised cancellation, privacy breach or unrecoverable state; not observed in a running system | Resolve or explicitly accept before implementation |

### 7.1 Application rules

- Severity describes the **documentary finding and its potential**, not measured user impact.
- A direct contradiction with a stated safety rule starts at 3.
- A finding whose consequence depends on an absent interface is provisional and cannot be presented as observed.
- A `documentary_gap` with no evidence of implementation behaviour is at most 2 unless the missing text itself would permit a critical action.
- Severity and both confidence judgements are independent: a high-severity risk can have low interpretation confidence.

## 8. Confidence scales

Confidence describes the strength of the documentary support, not a statistical probability. It is recorded twice: once for what the document says and once for the interpretation placed on it.

### 8.1 Confidence in the observation

| Level | Definition | Example |
|---|---|---|
| High | The cited passage directly supports what is recorded as present or absent; no inference beyond reading | A checkout page says “Payment received” before the payment provider’s success response |
| Medium | The observation requires combining two passages or resolving a wording ambiguity | Two help pages describe the same account state with different labels |
| Low | The passage is incomplete, internally contradictory or depends on an absent implementation | A claim about what the guest sees before the page renders |

### 8.2 Confidence in the interpretation

| Level | Definition | Example |
|---|---|---|
| High | The inference is tightly constrained by the passage and alternatives are implausible | A checkout page says “Order placed” before the order service confirms success |
| Medium | One plausible alternative reading or user reaction remains | A help article says a setting is optional but does not say what happens when it is omitted |
| Low | The consequence depends on absent implementation, unknown user behaviour or an unstated system response | A claim about what a screen-reader user would hear |

Observation confidence and interpretation confidence are not averaged. A directly observed sentence can have high observation confidence while the claimed guest consequence remains low or medium.

## 9. Procedure

1. **Freeze the baseline.** Record repository, commit, document hashes and the P1 reference.
2. **Build the evidence index.** Confirm every file exists and every cited section is present.
3. **Discovery pass.** Read each channel and state; record observations without criteria or severity.
4. **Classification pass.** Apply the primary criterion, split findings, assign type, severity and both confidence judgements.
5. **Coverage check.** Mark every `N`/`NA` cell and add it to the limitations; do not convert it into a finding.
6. **Blind operational reproduction.** One or more blind evaluators apply the protocol to the pilot excerpt before reading the conclusion. Each evaluator is identified as AI or human, and the packet used is recorded. The result is recorded as `confirmed`, `not_reproduced` or `requires_review`, with any divergence and rationale. A human evaluator is not mandatory for this portfolio gate; the absence of one is a declared limitation.
7. **Consolidation.** Deduplicate, label hypotheses, and write the limitations; keep divergent readings as review information.
8. **Methodological review.** The protocol, the recorded reproductions and the divergences are reviewed before the full evaluation begins.

### 9.1 Review gate

The full evaluation may start only when:

- the baseline and hashes are confirmed;
- the frameworks and adaptations are approved;
- the coverage matrix is complete, including non-evaluable items;
- the findings model and severity rules are approved;
- a documented blind operational reproduction exists, with each evaluator identified as AI or human and the packet used recorded;
- the divergences are recorded and discussed;
- the limitations include the absence of human independent review, participant testing and interface observation;
- the new HEAD is approved methodologically.

## 10. Pilot — one email excerpt

### 10.1 Scope

The pilot evaluates the section **“Email — CONFIRM without a verified session”** in `docs/02-cancellation-journey.md` at `afa3c60`. It is one excerpt, not a judgement of the whole journey.

### 10.2 Evidence

The document says:

> We received your CONFIRM, but it is not linked to a verified session for this request. No change has been made to your booking.
>
> To continue, use the secure link below to verify your identity and review the charge on the secure page. A reply to this email is not enough on its own.

The section also states that the guest must verify and then confirm on the secure page, and that an unverified sender must not see booking details.

### 10.3 Finding

| Field | Value |
|---|---|
| `id` | `HE-E-001` |
| `type` | `observation` with a hypothetical consequence |
| `channel` | `email` |
| `state` | `verification_pending` |
| `baseline` | `afa3c60`, `docs/02-cancellation-journey.md`, “Email — CONFIRM without a verified session” |
| `criterion` | Primary `N1` (visibility of status and next step); secondary `N9`; `X2` supplies context; `X4` continuity |
| `observation` | The reply correctly refuses to treat the plain `CONFIRM` as consent, says no change has been made, and points to the secure link. It does not explicitly say whether the earlier `CONFIRM` will be carried forward or whether the guest must confirm again after verification. |
| `hypothetical_consequence` | A guest may believe the reply already completed the cancellation, or may believe the earlier `CONFIRM` will carry forward and not complete the secure-page step; this could add a clarification cycle or cause abandonment. This is not measured. |
| `severity` | 2 — minor: the safe state is explicit and the next step is present, but continuity of intent is not stated. |
| `confidence_observation` | High — the absence of a sentence linking the earlier `CONFIRM` to the secure-page confirmation is directly verifiable in the cited section. |
| `confidence_interpretation` | Low — the guest reaction depends on unknown user behaviour; the text does not measure comprehension. |
| `recommendation` | Add one sentence after “No change has been made”: “Your CONFIRM has been received, but it is not consent; after verification you will review the charge and confirm on the secure page.” Do not promise that booking details will be retained. |
| `future_validation` | A comprehension test with 5–8 participants, asking “What do you need to do next?” and “Has your booking changed?”; check whether the secure page retains booking details as a design question, without promising it to the guest. Independent reproduction is also required. Neither test is part of this protocol. |
| `reproduction_status` | `not_reproduced` — four AI operational readings did not reproduce this finding as framed: two found no problem and two found different gaps. No human independent review. |
| `reproduction_note` | The four readings are recorded in sections 10.7, 10.9, 10.10 and 10.11. Divergences are preserved; they are not user validation. |

`X2` is context, not the primary criterion: the document already requires consent inside the verified session. The finding is about the clarity of the message and the next step, so `N1` is primary and `N9` is secondary. The recommendation does not claim that the secure page retains booking details; that is a design question to verify. The observation confidence is high, but the interpretation confidence is low because the guest reaction is unknown; the severity remains a documentary priority, not a measured impact.

### 10.4 Traceability

| Link | Content |
|---|---|
| Evidence | `EV-04` / `EV-05`: email plain-CONFIRM section |
| Criterion | Primary `N1` visibility; secondary `N9` recovery; `X2` context (consent rule is documented); `X4` continuity |
| Observation | The refusal is safe; the continuity of the earlier intent is not stated |
| Hypothesis | Possible repeated request, confusion or abandonment — not measured |
| Recommendation | Clarify that final confirmation happens after verification, without promising data retention |
| Validation | Comprehension test and independent reproduction; neither is part of this protocol |

### 10.5 Pilot result

The protocol produces a traceable finding: a reader can move from the exact passage to a criterion, an observation, a hypothetical consequence, a severity, two confidence judgements and a recommendation. The observation confidence is high because the absence is directly readable; the interpretation confidence is low because the guest reaction is unknown and unmeasured. The pilot does **not** validate the guest experience or the frequency of the problem.

### 10.6 Blind operational reproduction packet — reviewer-only

> **Reviewer-only section. Do not include this section in the evaluator’s packet.** It states the expected result.

For a blind operational reproduction, the reviewer prepares a packet containing:

- sections 1–9 of this protocol, including the blank finding model;
- the baseline excerpt at `afa3c60`: only “Email — CONFIRM without a verified session”;
- no part of section 10 (`10.1`–`10.12`), including the pilot, comparison, calibration and expected result.

The evaluator fills the finding model without consulting the existing conclusion. Each evaluator is identified as AI or human, and the packet used is recorded with the result. The reviewer compares the observation, primary criterion, severity and recommendation and records `confirmed`, `not_reproduced` or `requires_review`, with any divergence and rationale.

Expected comparison: the same observation — the reply is safe but does not state whether the earlier `CONFIRM` carries forward — and the same primary criterion, `N1` visibility. A different severity is acceptable if justified. A different observation is recorded as `not_reproduced` or `requires_review`, not deleted. Four AI blind readings are recorded in sections 10.7, 10.9, 10.10 and 10.11; none reproduced the original finding as framed. A human evaluator is desirable but is not a mandatory gate in this portfolio context; the absence is declared in the limitations.

### 10.7 AI operational reading A — result

**Evaluator:** an AI session in OpenCode (`ses_e2e3714dfffeRcuNrundC2K67G`), with no access to any part of section 10, given only sections 1–9, the baseline excerpt and the blank finding model. Packet: `p6_blind_packet.md`.
**Status:** AI operational reading completed; this reading did not reproduce the original finding as framed; no human independent review was performed.
**Comparison status:** `requires_review`.

| Dimension | First pilot | Reading A | Reading |
|---|---|---|---|
| State | `verification_pending` | `Consent` (also touches `Execution`) | Divergence in state naming; both are adjacent to the same passage. |
| Observation | The reply is safe, but does not state whether the earlier `CONFIRM` carries forward or must be repeated after verification | The reply names verification and review but not the subsequent confirmation; verifying is not itself a cancellation | Partial convergence: both locate the missing final confirmation in the guest-facing text; the first frames it as continuity of intent, the second as an omitted step. |
| Primary criterion | `N1` visibility | `X1` truthful state language | Divergence; both are plausible and need calibration. |
| Severity | 2 | 2 | Convergence. |
| `confidence_observation` | High | High | Convergence. |
| `confidence_interpretation` | Low | Medium | Divergence: the AI’s medium conflicts with the section 8.2 rule for unknown user behaviour and needs review. |
| Recommendation | Clarify that final confirmation happens after verification; do not promise data retention | State that verification is not cancellation and that confirmation on the secure page is required; do not describe the page beyond the excerpt | Convergence in intent; divergence in wording and emphasis. |
| Type | `observation` | `documentary_gap` | Divergence; needs calibration. |

Reading A also identified three adjacent candidates — a transfer alternative with no stated invocation route, “the charge” as a definite reference, and the tension between the no-change statement and the no-disclosure rule — and kept them separate rather than folding them into the finding.

**Decision:** the operational reading shows that the packet can produce a finding without the conclusion, but it does **not** confirm the first reading. It is one of four AI operational readings; it is not a human review and it is not user validation. The calibration decision is recorded in section 10.8; it preserves this reading and defines how the protocol applies to this case.

### 10.8 Calibration decision

This decision applies the protocol to this case. It does not declare the AI reading wrong, and it does not rewrite either original evaluation.

| Dimension | Original pilot | Reading A | Calibrated decision | Reason |
|---|---|---|---|---|
| Type | `observation` | `documentary_gap` | `documentary_gap` | The guest-facing message omits the final confirmation step. The rule exists in the surrounding text, so the record must locate the gap in the message, not claim the rule is absent. |
| Primary criterion | `N1` | `X1` | `N1` | The issue is visibility of the next step. `X1` is secondary: the message does not make a false claim about the state. |
| State | `verification_pending` | `Consent` (touches `Execution`) | `verification_pending`, with `consent` related | The message arrives before verification; the later confirmation belongs to the consent stage. |
| `confidence_observation` | High | High | High | The absence of the instruction in the cited reply is directly verifiable. |
| `confidence_interpretation` | Low | Medium | Low | The guest reaction is unknown; section 8.2 assigns `low` to that kind of consequence. |
| Severity | 2 | 2 | 2, provisional | Both evaluations proposed it; it is a documentary priority, not measured impact. |
| Recommendation | Clarify that final confirmation happens after verification | State that verification is not cancellation and confirmation is required | After verifying identity and reviewing the charge, the guest must confirm on the secure page; the email `CONFIRM` alone remains insufficient | Clarifies the next step without turning the email reply into consent, and without promising data retention. |

The AI’s `X1`, `Consent`, `documentary_gap` and `medium` interpretation remain recorded as its original reading. The calibration is a case decision, not a retroactive correction of the evaluator.

This calibration compares the original pilot with **reading A only**. Readings B, C and D were recorded afterwards and are preserved in sections 10.9–10.11; they do not change this decision.

The three adjacent candidates — the transfer route, “the charge” reference and the no-change/no-disclosure tension — remain possibilities. They are not approved findings and do not expand the pilot into a full evaluation.

**Calibration status:** `accepted_for_this_case`. Methodological approval remains pending; no human independent review was performed. The AI readings did not validate the finding or the guest experience.

### 10.9 AI operational reading B — OpenCode `NO FINDING`

**Evaluator:** an AI session in OpenCode (`ses_e2e2221fcffe2VhIQd76Wfmcsd`), given the blind packet `p6_blind_packet_c866d61.md` and asked not to force a finding. It had no access to any part of section 10 and saw no prior conclusion.
**Result:** `NO FINDING`.
**Status for the original finding:** `not_reproduced`.

The evaluator found the reply consistent with `X1`, `X2`, `N5`, `N1`/`I2`, `N9`/`I6`, `N3`/`I5` and `X3` as applied to the excerpt. It considered the transfer alternative as a possible weakness, but rejected it as a finding because “A reply to this email is not enough on its own” is scoped to consent, the secure link is a stated next step, and the transfer offer is unconditional. It recorded the transfer point as a divergence, not an approved finding.

This is an **AI operational reading, not a human independent review**. It is the only reading that explicitly considered and rejected the transfer-route gap; it is not the same artefact as the external `NO FINDING` reading in section 10.10.

### 10.10 AI operational reading C — external `NO FINDING`

**Provenance:** a separate AI evaluator, not found in the local OpenCode session store. The response was supplied verbatim by the reviewer as “Resposta 1”; the full packet and session log are not available, only the baseline excerpt and the result.
**Result:** `NO FINDING`; the evaluator recorded `severity: 0 — Not a problem`.
**Status for the original finding:** `not_reproduced`.

The evaluator filled the model as an `observation` in state `Consent`, with `X2` as primary and `N5`, `N9`, `X3` as secondary. It stated that the `CONFIRM` is not bound to a verified session, that no change was made, that the secure link is the next step, and that transfer is available if the link cannot be used. It did **not** record considering or rejecting the transfer-route gap; it treated the transfer offer as declared.

| Field | Reading C (translated summary of the submitted model) |
|---|---|
| `type` | `observation` |
| `state` | `Consent` |
| `criterion` | `X2` primary; `N5`, `N9`, `X3` secondary |
| `observation` | The reply refuses the unverified `CONFIRM`, states that no change was made, names the secure link and the verification step, warns that a reply is not enough, and offers transfer if the link cannot be used. |
| `hypothetical_consequence` | None identified; the transfer alternative is declared for failure of the link. |
| `severity` | `0` — Not a problem |
| `confidence_observation` | `high` |
| `confidence_interpretation` | `high` |
| `reproduction_status` | `pending` in the original submission |

This is an **AI operational reading, not a human independent review**. Its `high` interpretation confidence is its own judgement; the protocol’s section 8.2 rule and the calibration in 10.8 retain `low` for the original finding. The calibration is not changed by this reading.

### 10.11 AI operational reading D — external `documentary_gap` (`N9`)

**Provenance:** a second external AI evaluator, also supplied verbatim by the reviewer as “Resposta 2”. It is distinct from reading C and from the OpenCode readings; the full packet and session log are not available.
**Result:** `documentary_gap`; `severity: 2`.
**Status for the original finding:** `not_reproduced`; this is a different gap.

| Field | Reading D (translated summary of the submitted model) |
|---|---|
| `type` | `documentary_gap` |
| `state` | `Consent` |
| `criterion` | `N9` primary |
| `observation` | The reply says a plain reply is not enough and offers human transfer, but does not state *how* the guest requests that transfer. |
| `hypothetical_consequence` | A guest who cannot use the secure link may feel stuck or abandon the process, because the preceding sentence invalidates email replies and the transfer has no stated trigger. |
| `severity` | 2 |
| `confidence_observation` | `high` |
| `confidence_interpretation` | `medium` |
| `reproduction_status` | `pending` in the original submission |

The evaluator also recorded that it found no problem with `X2` and `X1` before filling this gap. Its recommendation is to specify the fallback action — for example, a reply keyword such as `SUPPORT` — and to clarify that only the confirmation reply is blocked.

This reading is close to the transfer-route candidate that reading A listed as an adjacent possibility and reading B considered and rejected. Recording it as a separate reading preserves the divergence; it does not make the transfer route an approved finding. The full evaluation may examine it; the pilot does not expand to cover it.

This is an **AI operational reading, not a human independent review**.

### 10.12 Consolidated reproduction position

| Reading | Evaluator and provenance | Packet | Result | Status for the original finding |
|---|---|---|---|---|
| Original pilot | First evaluator, main session | not applicable | `N1` visibility; later calibrated to `documentary_gap` | Original, preserved |
| A | OpenCode subagent `ses_e2e3714…` | `p6_blind_packet.md` | `X1` primary, `documentary_gap`, severity 2 | `not_reproduced`; `requires_review` |
| B | OpenCode subagent `ses_e2e2221…` | `p6_blind_packet_c866d61.md` | `NO FINDING`; considered and rejected the transfer route | `not_reproduced` |
| C | External AI, “Resposta 1” | baseline excerpt; packet not recorded | `NO FINDING`, severity 0, `X2` | `not_reproduced` |
| D | External AI, “Resposta 2” | baseline excerpt; packet not recorded | `N9` primary, `documentary_gap`, severity 2 | `not_reproduced`; divergent finding |
| Calibration | Reviewer decision | not applicable | `documentary_gap`, `N1`, `verification_pending`, low interpretation confidence, severity 2 | `accepted_for_this_case` |

**Count reconciliation.** The reviewer’s account of three readings corresponds to A, C and D. Reading B is a fourth, distinct artefact: it is an OpenCode session using the revised packet, and its text differs from C. It is preserved separately rather than merged with C, because merging would delete the only reading that explicitly considered and rejected the transfer route.

**What the readings show.**

- None of the four AI readings reproduced the original finding as framed: that the reply does not state whether the earlier `CONFIRM` carries forward.
- B and C found **no problem** in the excerpt.
- A and D found **different gaps** (`X1` and `N9` respectively).
- “The original finding was not reproduced” is therefore broader than “no finding was found”: two readings found no problem, and two found a different problem.

These AI readings are operational reproductions of the protocol, not user validation and not human independent review. Partial agreement on severity (A, D and the original all propose 2) is not agreement on cause or on the strength of the evidence.

The transfer-route candidate remains separate, as a possibility to examine in the full evaluation. It is not an approved finding, it does not expand the pilot, and it is not counted as a reproduction failure.

No human independent review, participant testing or interface observation was performed.

## 11. Limitations

- The baseline is documentation, not a running product. It cannot show what a guest actually sees, hears or understands.
- No participants, telemetry, support tickets or production data are used.
- No WCAG conformance is claimed. WCAG does not cover telephony or voice interfaces directly.
- Voice-specific accessibility remains a declared gap; ISO 9241-154 is a scope reference only, and `V1`–`V4` are project-proposed criteria, not a clause-level conformance claim.
- Severity is a proposed heuristic priority, not measured impact or probability.
- Both confidence judgements describe documentary support, not statistical confidence.
- An absent statement is not proof of an absent behaviour.
- A `not evaluable` item is a coverage gap, not a violation.
- The pilot is a method test, not an evaluation of user experience.
- No human independent review, participant testing or interface observation was performed. The four AI readings are operational readings, not user validation; the full evaluation must repeat this limitation.
- The four AI readings diverged: two found no problem and two found different gaps; agreement on severity is not agreement on cause or on the strength of the evidence.
- Readings C and D were supplied verbatim by the reviewer; their packet and session logs are unavailable, so their provenance is weaker than the recorded OpenCode sessions A and B.
- The baseline may change after this protocol is approved; a new commit requires a new baseline record.

## 12. References

Accessed 12 November 2026.

1. Jakob Nielsen, *10 Usability Heuristics for User Interface Design*, Nielsen Norman Group, 24 April 1994, last reviewed 30 January 2024. https://www.nngroup.com/articles/ten-usability-heuristics/
2. Jakob Nielsen, *Severity Ratings for Usability Problems*, Nielsen Norman Group, 1 November 1994. https://www.nngroup.com/articles/how-to-rate-the-severity-of-usability-problems/
3. ISO 9241-110:2020, *Ergonomics of human-system interaction — Part 110: Interaction principles*, Edition 2, 2020-05. https://www.iso.org/standard/75258.html — public title/scope page only; full text not consulted.
4. ISO 9241-154:2013, *Ergonomics of human-system interaction — Part 154: Interactive voice response (IVR) applications*, Edition 1, 2013-02. https://www.iso.org/standard/52172.html — public title/scope page only; full text not consulted; `V1`–`V4` are project-proposed.
5. W3C, *Web Content Accessibility Guidelines (WCAG) 2.2*, W3C Recommendation, 12 December 2024. https://www.w3.org/TR/WCAG22/
6. W3C, *WAI-ARIA 1.2*, W3C Recommendation, 6 June 2023. https://www.w3.org/TR/wai-aria-1.2/
7. Conversation-design Core, `docs/02-cancellation-journey.md`, baseline `afa3c60`.
8. P1 public contract, `README.md` and `app/schema.py`, baseline `cf738f2`. https://github.com/luizfroes-automation/handover-intelligence-portfolio/tree/cf738f2

## 13. Evidence and approval checklist

### Evidence available

- [x] Baseline and the five document hashes verified.
- [x] Nielsen, ISO and WCAG references identified; the ISO full texts were not consulted and that limit is declared.
- [x] Coverage matrix drafted with documentary/runtime and `N`/`NA` distinctions.
- [x] Findings model, severity scale and both confidence scales defined.
- [x] Four AI operational readings recorded and identified as AI, with provenance: two OpenCode sessions (A, B) and two external responses (C, D).
- [x] Divergences and the calibration decision recorded without deleting any reading.
- [x] The distinction between “the original finding was not reproduced” and “no finding was found” stated explicitly.
- [x] Limitations declared, including the absence of human independent review, participant testing and interface observation.

### Approval gates

- [ ] Methodological approval of the new HEAD.
- [ ] Full evaluation authorised separately.

Until the approval gates are cleared, no full heuristic evaluation is started.
