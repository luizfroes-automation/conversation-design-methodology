# P6-ENH-01 — Documentary Heuristic Evaluation: Methodology Protocol

**Status:** Draft for methodological review. The full evaluation has not started.
**Ticket:** P6-ENH-01 — Methodology Discovery.
**Baseline:** `conversation-design-methodology` at `afa3c601060185b2f8a758e042a1b8efa55c496d` (squash of PR #1).
**P1 reference:** `handover-intelligence-portfolio` at `cf738f258efd96b3a5c825310da4561dc1c79e39`.
**Scope:** define and pilot the method only. This document does not evaluate the whole journey and does not test users.

## 1. Purpose

This protocol defines a structured, traceable and reproducible way to review the **documented** P6 cancellation journey for chat, voice and email. It is a heuristic evaluation of design documents, not a usability test and not a conformance audit.

The method must:

- identify the exact documentary baseline;
- connect each observation to an explicit criterion and a cited passage;
- separate what the document says from a hypothesis about a user consequence;
- record non-evaluable criteria without treating them as violations;
- make a pilot finding reproducible by another evaluator.

## 2. Baseline and evidence index

### 2.1 Object of evaluation

The baseline is the P6 Core at commit `afa3c60`. The documents are the only evidence; there is no running chat, voice or email implementation in this repository.

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
2. **Classification pass:** map each observation to criteria, separate observation from hypothetical consequence, assign severity and confidence, and write a recommendation.

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

Source: ISO 9241-110:2020, *Ergonomics of human-system interaction — Part 110: Interaction principles*, Edition 2, 2020-05 (confirmed 2025). The standard is paywalled; this protocol uses the principles at a high level and does not reproduce or claim clause-level conformance.

| Code | Principle | Application | Status |
|---|---|---|---|
| `I1` | Suitability for the user’s task | The journey supports cancellation rather than exposing internal system steps | Evaluable |
| `I2` | Self-descriptiveness | The state and the next step are explicit | Evaluable |
| `I3` | Conformity with user expectations | Cancellation and charge behaviour matches the stated policy | Evaluable |
| `I4` | Learnability | A guest can understand the flow without a tutorial | Partially evaluable |
| `I5` | Controllability | The guest can choose, decline, verify or transfer | Evaluable |
| `I6` | Use error robustness | Unknown and failure states have a safe recovery path | Evaluable |
| `I7` | User engagement | The tone supports continued interaction without pressure | Partially evaluable |

### 4.3 Voice-specific reference

Source: ISO 9241-154:2013, *Ergonomics of human-system interaction — Part 154: Interactive voice response (IVR) applications*, Edition 1, 2013-02 (confirmed 2026). The standard is paywalled; the protocol uses its subject matter at principle level only.

| Code | Voice criterion | Application | Status |
|---|---|---|---|
| `V1` | Turn-taking and barge-in | The caller can interrupt, correct or ask for a human | Partially evaluable from script |
| `V2` | Confirmation without visual memory | Key reference, charge and next step are read back or repeated | Evaluable from script |
| `V3` | Error recovery without a screen | Failure and unknown states have an audible next step | Evaluable from script |
| `V4` | No visual-only instruction | The voice flow does not depend on reading a page | Evaluable from script |

No claim is made that the voice journey conforms to ISO 9241-154.

### 4.4 Accessibility references

Source: W3C, *Web Content Accessibility Guidelines (WCAG) 2.2*, W3C Recommendation, 12 December 2024. WCAG applies to web content; it does not directly cover telephony or a voice user interface. No conformance claim is made.

Source for a future implemented chat UI: W3C, *WAI-ARIA 1.2*, W3C Recommendation, 6 June 2023. It is informative here and not evaluable without an interface.

| Code | Accessibility concern | Relevant WCAG 2.2 reference | Status |
|---|---|---|---|
| `A1` | Names, labels and instructions | 1.1.1, 2.4.6, 3.3.2 | Partially evaluable in text |
| `A2` | Error identification and suggestion | 3.3.1, 3.3.3 | Evaluable in text |
| `A3` | Error prevention and redundant entry | 3.3.4, 3.3.6, 3.3.7 | Evaluable in text |
| `A4` | Time limits and re-authentication | 2.2.1, 2.2.5, 2.2.6 | Partially evaluable; secure-link timing is described, implementation is not |
| `A5` | Status messages | 4.1.3 | Partially evaluable; visual/ARIA behaviour is not |
| `A6` | Keyboard, focus, contrast, target size | 2.1.1, 2.4.7, 1.4.3, 2.5.8 | Not evaluable without an interface |

Voice accessibility for deaf, hard-of-hearing, speech-impaired or cognitively impaired callers cannot be established from this baseline. That is a declared gap, not a pass.

### 4.5 Project-specific criteria

These are **own adaptations**, not standards. They exist because the P6 journey has states that generic heuristics do not name.

| Code | Criterion | Why it exists |
|---|---|---|
| `X1` | Truthful state language | A request, an unknown result and a confirmed cancellation must not share wording |
| `X2` | Consent bound to verification | A consent action must be tied to the identity session that authorises it |
| `X3` | Repair and clarification | The guest gets one focused question and an alternative before transfer |
| `X4` | Continuity across handover | The human receives the context without asking the guest to repeat it |
| `X5` | Channel parity | The same request reaches the same state and policy answer in all channels |
| `X6` | Unknown/reconciliation discipline | A timeout is not treated as success or failure; retry waits for reconciliation |

## 5. Coverage matrix

### 5.1 Status codes

- **E — evaluable:** the document contains enough evidence to assess the criterion.
- **P — partially evaluable:** the document states the intended behaviour, but the actual implementation, timing or sensory experience is absent.
- **N — not evaluable:** the criterion requires an interface, runtime, audio, data or user observation that the baseline does not contain.
- **NA — not applicable:** the criterion does not apply to that stage or channel.

`N` and `NA` are coverage findings; they are not violations.

### 5.2 Channel × state coverage

Cells list the primary criteria and the strongest status. The full list of criteria is in section 4.

| Journey state | Chat | Voice | Email |
|---|---|---|---|
| Identification | E (`N1,N2,N5,I1,I2,X1`) | P (`V1,V2,I2,X1`); identity provider not runnable | E (`N1,N2,N5,I2,X1`); session described |
| Clarification | E (`N3,N9,I5,I6,X3`) | P (`V1,V3,I5,I6,X3`); ASR repair not runnable | E (`N3,N9,I5,I6,X3`) |
| Consent | E (`N5,I5,X2`) | P (`V1,V2,I5,X2`); voice confirmation scripted only | E (`N5,I5,X2`); secure session described |
| Execution | E (`N1,N4,X1,X6`); booking API not runnable | P (`N1,N4,V3,X1,X6`); booking API and audio not runnable | E (`N1,N4,X1,X6`); booking API not runnable |
| Unknown result | E (`N1,N9,I6,X1,X6`) | P (`V3,I6,X1,X6`); audible recovery not runnable | E (`N1,N9,I6,X1,X6`) |
| Notification | E (`N1,N9,X1`); delivery not runnable | P (`V3,X1`); delivery and audio not runnable | E (`N1,N9,X1`); delivery not runnable |
| Human transfer | E (`N3,N6,N10,I5,X4`); routing UI not present | P (`V3,I5,X4`); transfer experience not runnable | E (`N3,N6,N10,I5,X4`); routing UI not present |

### 5.3 Not evaluable without an implementation

| Area | Why not evaluable | Future method |
|---|---|---|
| Visual hierarchy, contrast, target size | No rendered interface | Inspect a build; WCAG 2.2 checks |
| Keyboard, focus order, screen reader | No DOM or assistive-technology behaviour | Manual accessibility testing |
| Latency and timeouts | No runtime | Instrumented prototype |
| Speech recognition accuracy, barge-in, audio quality | No voice system or audio | Voice prototype and listening tests |
| Actual identity-provider and booking-system behaviour | No integrations | Integration test environment |
| Notification delivery | No email/SMS provider | Delivery logs and test accounts |
| Transfer routing and agent desktop | No contact-centre system | Simulated or staging routing |
| User comprehension and emotional response | No participants | Usability test or interviews, outside this protocol |

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
| `confidence` | `high`, `medium` or `low`, from section 8 |
| `recommendation` | A concrete change to the document or a future test |
| `future_validation` | What would confirm or reject the hypothesis |

### 6.2 Writing rules

- One finding, one issue. Split a finding when the recommendation differs.
- Do not merge a safe-state observation with a separate notification issue.
- Do not use “users will”; use “a guest may”.
- A `documentary_gap` names what is missing and why it matters; it does not claim the implementation lacks it.
- A `hypothesis` is allowed when the consequence depends on an absent interface, but confidence cannot exceed `low`.
- Delete a finding if a second evaluator cannot reproduce the observation from the cited passage.

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
- Severity and confidence are independent: a high-severity risk can have low confidence.

## 8. Confidence scale

Confidence describes the strength of the documentary support, not a statistical probability.

| Level | Definition | Example |
|---|---|---|
| High | Direct, explicit passage supports the observation; no inference beyond reading | The dialogue says “No change has been made” while the state remains `verification_pending` |
| Medium | The observation requires combining two explicit passages or one interpretive step | A message gives a next step but does not acknowledge the guest’s earlier intent |
| Low | The observation depends on missing implementation, ambiguous wording or an unstated behaviour | A claim about what a screen-reader user would hear |

## 9. Procedure

1. **Freeze the baseline.** Record repository, commit, document hashes and the P1 reference.
2. **Build the evidence index.** Confirm every file exists and every cited section is present.
3. **Discovery pass.** Read each channel and state; record observations without criteria or severity.
4. **Classification pass.** Apply the primary criterion, split findings, assign type, severity and confidence.
5. **Coverage check.** Mark every `N`/`NA` cell and add it to the limitations; do not convert it into a finding.
6. **Pilot reproduction.** A second evaluator applies the protocol to the pilot excerpt and compares the observation, criterion and recommendation.
7. **Consolidation.** Deduplicate, label hypotheses, and write the limitations.
8. **Review gate.** The protocol and pilot are approved before the full evaluation begins.

### 9.1 Review gate

The full evaluation may start only when:

- the baseline and hashes are confirmed;
- the frameworks and adaptations are approved;
- the coverage matrix is complete, including non-evaluable items;
- the findings model and severity rules are approved;
- the pilot is reproducible;
- the limitations are accepted.

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
| `criterion` | Primary `X2` (consent bound to verification); secondary `N5`, `N9`, `I5`, `I6`, `X4` |
| `observation` | The reply correctly refuses to treat the plain `CONFIRM` as consent, says no change has been made, and points to the secure link. It does not explicitly say whether the earlier `CONFIRM` will be carried forward or whether the guest must confirm again after verification. |
| `hypothetical_consequence` | A guest may believe the reply already completed the cancellation, or may believe the earlier `CONFIRM` will carry forward and not complete the secure-page step; this could add a clarification cycle or cause abandonment. This is not measured. |
| `severity` | 2 — minor: the safe state is explicit and the next step is present, but continuity of intent is not stated. |
| `confidence` | Medium — the observation is explicit; the consequence requires one inference about comprehension. |
| `recommendation` | Add one sentence after “No change has been made”: “Your CONFIRM has been received, but it is not consent; after verification you will review the charge and confirm on the secure page. You will not need to re-enter the booking details.” |
| `future_validation` | A comprehension test with 5–8 participants, asking “What do you need to do next?” and “Has your booking changed?”; a second evaluator should reproduce the finding from the same passage. This test is outside the current ticket. |

### 10.4 Traceability

| Link | Content |
|---|---|
| Evidence | `EV-04` / `EV-05`: email plain-CONFIRM section |
| Criterion | `X2` consent binding; `N5` error prevention; `N9` recovery; `I5` controllability; `I6` error robustness; `X4` continuity |
| Observation | The refusal is safe; the continuity of the earlier intent is not stated |
| Hypothesis | Possible repeated request, confusion or abandonment — not measured |
| Recommendation | Add the intent-continuity sentence without accepting the plain reply as consent |
| Validation | Comprehension test and independent reproduction; not part of this ticket |

### 10.5 Pilot result

The protocol produces a traceable finding: a reader can move from the exact passage to a criterion, an observation, a hypothetical consequence, a severity, a confidence and a recommendation. A second evaluator can reproduce the observation without seeing the first evaluator’s conclusion. The pilot does **not** validate the guest experience or the frequency of the problem.

### 10.6 Reproduction check

A second evaluator should:

1. check out `afa3c60` and open `docs/02-cancellation-journey.md`;
2. read only “Email — CONFIRM without a verified session”;
3. fill the finding model without reading section 10.3;
4. compare the observation, primary criterion and recommendation;
5. record any difference in severity or confidence with a reason.

Expected result: the same observation — the reply is safe but does not state whether the earlier `CONFIRM` carries forward — and the same primary criterion, `X2` consent binding. A different severity is acceptable if justified; a different observation means the finding is not yet reproducible. Independent reproduction is a review gate; it has not been claimed as completed in this draft.

## 11. Limitations

- The baseline is documentation, not a running product. It cannot show what a guest actually sees, hears or understands.
- No participants, telemetry, support tickets or production data are used.
- No WCAG conformance is claimed. WCAG does not cover telephony or voice interfaces directly.
- Voice-specific accessibility remains a declared gap; ISO 9241-154 informs the review but is not a clause-level conformance claim.
- Severity is a proposed heuristic priority, not measured impact or probability.
- Confidence is documentary support, not statistical confidence.
- An absent statement is not proof of an absent behaviour.
- A `not evaluable` item is a coverage gap, not a violation.
- The pilot is a method test, not an evaluation of user experience.
- A single evaluator can miss findings; the full evaluation requires an independent review sample and a documented calibration.
- The baseline may change after this protocol is approved; a new commit requires a new baseline record.

## 12. References

Accessed 12 November 2026.

1. Jakob Nielsen, *10 Usability Heuristics for User Interface Design*, Nielsen Norman Group, 24 April 1994, last reviewed 30 January 2024. https://www.nngroup.com/articles/ten-usability-heuristics/
2. Jakob Nielsen, *Severity Ratings for Usability Problems*, Nielsen Norman Group, 1 November 1994. https://www.nngroup.com/articles/how-to-rate-the-severity-of-usability-problems/
3. ISO 9241-110:2020, *Ergonomics of human-system interaction — Part 110: Interaction principles*, Edition 2, 2020-05. https://www.iso.org/standard/75258.html
4. ISO 9241-154:2013, *Ergonomics of human-system interaction — Part 154: Interactive voice response (IVR) applications*, Edition 1, 2013-02. https://www.iso.org/standard/52172.html
5. W3C, *Web Content Accessibility Guidelines (WCAG) 2.2*, W3C Recommendation, 12 December 2024. https://www.w3.org/TR/WCAG22/
6. W3C, *WAI-ARIA 1.2*, W3C Recommendation, 6 June 2023. https://www.w3.org/TR/wai-aria-1.2/
7. P6 Core, `docs/02-cancellation-journey.md`, baseline `afa3c60`.
8. P1 public contract, `README.md` and `app/schema.py`, baseline `cf738f2`. https://github.com/luizfroes-automation/handover-intelligence-portfolio/tree/cf738f2

## 13. Approval checklist

- [ ] Baseline and hashes verified.
- [ ] Nielsen, ISO and WCAG references accepted, with adaptations declared.
- [ ] Coverage matrix accepted, including `N`/`NA`.
- [ ] Findings model and severity/confidence rules accepted.
- [ ] Pilot reproduced by a second evaluator.
- [ ] Limitations accepted.
- [ ] Full evaluation authorised separately.

Until this checklist is approved, no full heuristic evaluation is started.
