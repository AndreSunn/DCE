> Public reference copy, exported 25 October 2026. Private case references have been replaced with explicit placeholders.
> Source status: **Accepted historical source**. This is a documentation derivative, not the byte-identical production artifact.
> Read the repository README and manifest for policy precedence, original/published hashes and the evaluation pause.

# Codex Sales Call Evaluation Rule Bank — Q01 to Q14

**Document purpose:** Authoritative question-level validation source for Sales call evaluations performed in the Call Evaluation Codex chat.

**Rule-bank version:** `sales-q01-q14-2026-10-15-mixed-language-review-v12`
**Source registry:** `question-rules-vNext-2026-09-03`  
**Permitted use:** Call evaluation, independent validation, comparison and adjudication in Codex chat.  
**Production status:** Unreleased CE Fix candidate. Runtime activation requires release checks, guarded writer configuration, Dataverse CE Fix readback, and live rulebank verification; this text alone does not activate the policy.

---

## 1. Mandatory Codex instruction

When this rule bank is supplied or referenced, Codex must:

1. Use the exact Sales Q01–Q14 question meanings and question-specific rules in this document.
2. Classify the call before answering any performance question.
3. Decide whether each question's dependency and opportunity were reached before selecting `Yes`, `No` or `N/A`.
4. Evaluate every question independently. Do not carry a decision or evidence from one question into another without checking that question's own rule.
5. Use the live employee/customer conversation as performance evidence.
6. Never use an IVR/whisper introduction, lead metadata, CRM data, caller ID, routing label, filename, previous score, human answer or AI answer as proof of a scored behaviour.
7. Use `SE Mapping DV` to resolve the canonical Sales Executive identity when the spoken name is unrecognisable, garbled or misspelled, provided the live conversation establishes that the speaker is the Sales Executive.
8. Keep identity resolution separate from Q02 scoring: the mapping may supply the correct SE name, but it may not invent a self-introduction that did not occur.
9. Use `Dealership Mapping DV` to resolve the canonical dealership identity when the dealership name is unrecognisable, garbled, misspelled or expressed through a known brand/location alias.
10. Keep dealership resolution separate from performance scoring: the mapping may populate the correct dealership record, but it may not prove a scored Sales behaviour.
11. Return one row for every question from Q01 through Q14, in that exact order.
12. Give a short evidence excerpt or a clear applicability explanation for every `Yes`, `No` and `N/A` answer. The explanation must agree with the stored result: a `Yes` explanation describes the completed behaviour, a `No` explanation describes the missed applicable behaviour, and an `N/A` explanation describes why the question was not applicable or not reached.
13. Mark a question `N/A` when the call ended before that question's stage or opportunity was reasonably reached.
14. Do not change earlier, observable decisions to `N/A` merely because the call ended later. Behaviours reached before the ending must still be evaluated.
15. Use controlled inference only when a strong conversational pattern resolves minor transcription noise. Never use inference to fill an ordinary sales-process gap.
16. Flag material ambiguity for human/audio review instead of forcing certainty.
17. Calculate the score deterministically using only applicable `Yes` and `No` answers.
18. Never invent a performance-rating threshold.
19. Create `cleanedTranscriptText` after classification and before Q01-Q14 scoring, preserving the actual spoken wording and order while correcting safe misspellings, punctuation, capitalization and spacing, removing non-conversation noise, and separating Sales Executive/customer turns where the speaker role is reliable.

If a later, explicit user-approved rule conflicts with this document, record the conflict and apply the newer rule only after the user confirms that it supersedes this rule-bank version.

---

## 2. Evidence hierarchy

Use evidence in this order:

1. User-provided audio, only when audio review is explicitly requested.
2. The cleaned, application-selected transcript.
3. Another usable transcript source, only to resolve ASR errors or unclear names, models, brands or speaker attribution.
4. Metadata, only for routing, identifiers and spelling—not for performance scoring.

Do not silently combine contradictory transcripts. If the evidence remains unclear, use `N/A with human review` and add a QA flag.

---

## 3. Sales Executive identity resolution — SE Mapping DV

`SE Mapping DV` is the approved Dataverse mapping source for resolving the canonical Sales Executive identity when transcription does not render the name recognisably.

### Resolution procedure

1. First inspect the live conversation for speaker role and self-introduction evidence.
2. Preserve the raw transcribed name exactly as heard or rendered in `rawSpokenSEName`.
3. If the name is recognisable, resolve it normally and use the mapping only to confirm canonical spelling or Dataverse identity.
4. If the name is unrecognisable, garbled, phonetically distorted or misspelled, query `SE Mapping DV` using the available dealership, department, destination, phone/extension, alias and active-employee context.
5. Select the mapped SE only when the result is unique and contextually consistent with the call.
6. Populate the canonical mapped identity in `salesExecutiveName` and, when available, preserve the mapped Dataverse identifier in `salesExecutiveId`.
7. Record `seNameResolutionSource=SE Mapping DV`, the matched mapping basis and a confidence level.
8. If several mapped candidates remain plausible or the mapping conflicts with the live conversation, do not guess. Set the SE identity to `Unresolved`, add an `SE_IDENTITY_REVIEW` QA flag and request human review.

### Separation from Q02 scoring

- `SE Mapping DV` is an identity-resolution and spelling source; it is not performance evidence.
- If the live employee says, for example, “Hi, this is [garbled name] from [example dealership],” Q02 may be `Yes` because a clear self-introduction occurred. The mapping may then recover the canonical name. Set `controlled_inference_used=true` and disclose that the mapping resolved the garbled name.
- If no self-introduction occurred, the mapping cannot make Q02 `Yes`. Apply Q02's normal `No` or `N/A` rule based on whether the introduction opportunity was reached.
- An IVR/whisper announcement, CRM owner, destination number or mapped employee record cannot independently prove that the mapped person spoke or introduced themselves.
- Do not overwrite a clearly spoken, contradictory identity merely because a different SE appears in the mapping. Flag the conflict for review.

### Required identity-resolution fields

| Field | Requirement |
|---|---|
| `rawSpokenSEName` | Exact recognised or garbled transcript rendering; blank only when no name was spoken |
| `salesExecutiveName` | Canonical resolved SE name or `Unresolved` |
| `salesExecutiveId` | Dataverse identifier when available and confidently mapped |
| `seNameResolutionSource` | `Live transcript`, `SE Mapping DV` or `Human review` |
| `seMappingBasis` | Dealership, department, phone/extension, alias or other mapping keys used |
| `seNameResolutionConfidence` | `high`, `medium` or `low` |
| `seIdentityQaFlag` | Empty when resolved; otherwise `SE_IDENTITY_REVIEW` |

---

## 4. Dealership identity resolution — Dealership Mapping DV

`Dealership Mapping DV` is the approved Dataverse mapping source for resolving the canonical dealership identity when transcription does not render the dealership name recognisably or consistently.

### Resolution procedure

1. First inspect the live conversation for a spoken dealership, branch, brand or location reference.
2. Preserve the raw transcribed dealership wording exactly as heard or rendered in `rawSpokenDealershipName`.
3. If the dealership is recognisable, resolve it normally and use the mapping to confirm the canonical Dataverse name and identifier.
4. If the dealership name is unrecognisable, garbled, phonetically distorted, abbreviated, misspelled or expressed as a known alias, query `Dealership Mapping DV` using the available brand, location, branch alias, destination number/queue, department and mapped-SE context.
5. Select the mapped dealership only when the result is unique and consistent with the available call context.
6. Populate the canonical mapped dealership in `dealershipName` and, when available, preserve its Dataverse identifier in `dealershipId`.
7. Record `dealershipResolutionSource=Dealership Mapping DV`, the mapping basis and the resolution confidence.
8. If several dealerships remain plausible or the mapping conflicts with the live conversation, do not guess. Set the dealership to `Unresolved`, add a `DEALERSHIP_IDENTITY_REVIEW` QA flag and request human review.

### Separation from performance scoring

- `Dealership Mapping DV` is an entity-resolution and canonical-naming source; it is not evidence that a scored Sales behaviour occurred.
- A garbled spoken branch name may be normalised through the mapping when brand/location or mapped-SE context uniquely identifies the dealership.
- If no dealership is spoken, routing or destination data may be used to populate the dealership record for call ownership only. It must be labelled as mapping/routing-derived and must not be quoted as live transcript evidence.
- Do not award Q01–Q14 because the dealership mapping contains a customer, SE, branch, lead, phone or enquiry record.
- Do not overwrite a clearly spoken, contradictory dealership merely because a different branch appears in the mapping. Flag the conflict for review.
- Do not infer a dealership from the vehicle brand alone when more than one mapped dealership represents that brand.

### Required dealership-resolution fields

| Field | Requirement |
|---|---|
| `rawSpokenDealershipName` | Exact recognised or garbled transcript rendering; blank only when no dealership was spoken |
| `dealershipName` | Canonical mapped dealership name or `Unresolved` |
| `dealershipId` | Dataverse identifier when available and confidently mapped |
| `dealershipResolutionSource` | `Live transcript`, `Dealership Mapping DV`, `Routing metadata` or `Human review` |
| `dealershipMappingBasis` | Brand, location, branch alias, destination/queue, department, mapped SE or other keys used |
| `dealershipResolutionConfidence` | `high`, `medium` or `low` |
| `dealershipIdentityQaFlag` | Empty when resolved; otherwise `DEALERSHIP_IDENTITY_REVIEW` |

---

## 5. Call classification before scoring

### `full_sales`

A meaningful vehicle-sales conversation with enough interaction to evaluate most sales behaviours. Evaluate all Q01–Q14 questions, but still apply the individual opportunity and dependency rule. Do not force every question to be applicable.

### `minimal_sales`

A real but limited Sales interaction, including an early deferral, denied enquiry, wrong person, short follow-up, browsing enquiry or price-only call. Score only the questions whose individual opportunity was reached. Deeper qualification questions are normally `N/A` when the call ends before their stage.

### Live follow-up routing versus actual conversation

- The only follow-up routing marker is the current source field `cra95_calltype = Lead Tracker`. Spoken words, dialer announcements, transcript preambles, `cra95_evaluationcalltype`, and `cra95_isfollowupcall` do not establish follow-up routing. `Connect` and `Call Manager` remain ordinary source types regardless of wording such as `Calling prospect` or `Prospect follow-up`.
- A source-routed follow-up can be `full_sales` or `minimal_sales` only when the live customer/employee turns establish eligible vehicle Sales. Score Q01–Q14 from those live turns, retaining the explicit follow-up authorization and human-review protections. The separate Lead Tracker hard-exclusion remains in force until independently changed through release control; this clarification grants no exception to it.
- Do not score IVR, dialer announcement, no-answer, voicemail-only, or service-only content. A transcript phrase alone proves neither follow-up routing nor any Q01–Q14 behaviour.
- Positive regression: a `Connect` call with a dialer phrase `Prospect follow-up call` and live vehicle Sales discussion is routed as ordinary Sales. A source row explicitly marked `cra95_calltype = Lead Tracker` activates the follow-up guard and requires its existing safety attestations.
- Negative regression: a `Connect` or `Call Manager` call with the same phrase but no live customer/employee interaction remains unscoreable; a Lead Tracker source with only voicemail also remains unscoreable.

### `not_scorable`

Use when there is no meaningful live employee/customer interaction, including voicemail, no answer, IVR/menu only, dead air, silence, an abandoned call, a transfer-only recording or a severe transcript failure.

- Set Q01–Q14 to `N/A`.
- Set `scorePercentage` to `null` or blank.
- Set rating to `Not Scored`.
- Do not treat the IVR/whisper message as a live sales interaction.

### `non_sales`

Use for Parts, Workshop, service, recruitment, suppliers, spam, administration or unrelated calls. Do not apply Sales Q01–Q14. Route to a separately approved rubric when available; never invent one.

---

## 6. Universal result mapping

| Decision state | Required result |
|---|---|
| The stage was reached and the behaviour was demonstrated | `Yes` |
| The stage was reached, the behaviour was applicable and the SE missed it | `No` |
| The call ended before the relevant stage or opportunity | `N/A` |
| A required dependency was absent | `N/A` |
| The question did not apply to the call context | `N/A` |
| Evidence is materially uncertain | `N/A` plus human/audio review |

`N/A` is an applicability decision, not a favourable score and not a substitute for missing evidence.

### Result/comment consistency guard

Before any DCE row is written, every Q01-Q14 result, comment, evidence and reason must be internally consistent.

- If `result=Yes`, the comment/evidence/reason must describe the behaviour that was completed.
- If `result=No`, the comment/evidence/reason must describe the applicable behaviour that was missed. It must not say that the behaviour occurred, was communicated, was completed or was established.
- If `result=N/A`, the comment/evidence/reason must explain why the question was not applicable, not reached, or blocked by a missing dependency. It must not say that the measured behaviour was completed.
- Never write a completed DCE row where the result and comment contradict each other. Repair the result or comment before saving. If the transcript does not support a clear repair, route the item to human/audio review instead of saving a contradictory completed evaluation.

---

## 7. Question-level rules

## Q01 — Correct person

**Question:** Did the Sales Executive confirm they were speaking to the correct person?

- **Dependency:** A live Sales opening where customer identity is relevant.
- **Opportunity reached:** Once a real employee/customer opening occurs.
- **Yes:** The intended person explicitly confirms their identity, or clearly confirms the relevant enquiry or identity without contradiction.
- **No:** The call reaches a fair identity-confirmation opportunity, but identity is omitted; or the live response establishes a wrong person, wrong number or denied intended-person context.
- **N/A:** No live interaction; a generic inbound call with no intended named prospect where identity is genuinely irrelevant; or the call ends before identity could reasonably be confirmed.
- **Evidence:** Use live customer identity or enquiry confirmation only. The IVR/whisper lead name, CRM name, caller ID or customer use of the employee's name is not proof.
- **Early-end rule:** If the live exchange ends before identity could reasonably be confirmed, use `N/A`.

## Q02 — Sales Executive name

**Question:** Did the Sales Executive provide their own name?

- **Dependency:** A live Sales opening.
- **Opportunity reached:** Once the employee engages the customer.
- **Yes:** The SE gives a live self-introduction by name. A clear self-introduction pattern with a garbled ASR rendering of the name may pass through disclosed controlled inference.
- **No:** A real opening occurs with a fair opportunity to introduce, but there is no self-introduction.
- **N/A:** No live interaction or the call ends before a reasonable introduction opportunity.
- **Evidence:** Use employee-owned self-identification only. Metadata, a colleague's name or the customer saying the employee's name does not count.
- **Early-end rule:** If the call ends before the employee could reasonably introduce themselves, use `N/A`.

## Q03 — Customer enquiry

**Question:** Did the Sales Executive confirm the customer's enquiry?

- **Dependency:** A relevant Sales enquiry or Sales conversation.
- **Opportunity reached:** Once the conversation moves beyond the greeting into the Sales purpose or enquiry context.
- **Yes:** The SE confirms or clarifies the actual vehicle, quote, finance request, purchase interest or reason for the Sales call.
- **No:** A meaningful Sales conversation proceeds, but the enquiry is not established.
- **N/A:** The call ends or is deferred before enquiry discussion; there is no relevant enquiry or live interaction; or the context never permits enquiry confirmation.
- **Evidence:** Use live confirmation of the customer's request. Do not use the automated lead description as proof.
- **Early-end rule:** If the customer defers before enquiry discussion, use `N/A` unless the enquiry was already confirmed.

## Q04 — Business or private use

**Question:** Did the Sales Executive ask whether the vehicle is for business or private use?

- **Dependency:** A Sales qualification opportunity.
- **Opportunity reached:** Once the call progresses beyond enquiry confirmation into qualification or deal discussion.
- **Yes:** The SE asks or explicitly confirms business, company, private or personal use. Customer-volunteered use counts only when the SE acknowledges or clearly works with it.
- **No:** Meaningful qualification reaches this stage, but intended use is omitted.
- **N/A:** The call ends or is deferred before use-purpose qualification; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** Use live wording about business, company, private or personal use.
- **Early-end rule:** If the customer requests a later callback before qualification, use `N/A`.

## Q05 — Spending capacity or budget

**Question:** Did the Sales Executive confirm the customer's spending capacity or budget?

- **Dependency:** Affordability or purchase-structure discussion is fairly reached.
- **Opportunity reached:** Once payment, finance, quotation, affordability or deal structure is discussed.
- **Yes:** The SE establishes any live-conversation spending-capacity signal from the customer, including an explicit budget, monthly affordability, deposit plus affordability, repayment expectation, price range, shortfall/top-up affordability, maximum spend, or another clear affordability/capacity discussion. If the SE asks for budget or monthly instalment and the customer gives an affordability-based answer, even if approximate or dependent on trade-in shortfall/top-up, Q05 must be `Yes`.
- **No:** The affordability stage is reached, but the customer's capacity or target is not established.
- **Unanswered budget question:** An SE question about budget or monthly payment is not itself a customer capacity answer. If the customer gives no budget, target, affordability, deposit or shortfall answer after the question, Q05 is `No` when the opportunity was reached. Attribute a capacity signal to the customer before requiring `Yes`.
- **N/A:** The call ends before the affordability stage; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** A vehicle price, advertised instalment, model price, or verbal quote alone is not the customer's budget. A customer asking for the vehicle price, the SE giving the vehicle price, a model price being mentioned, or a quote being discussed does not make Q05 `Yes` unless spending capacity, affordability, a repayment/monthly expectation, shortfall/top-up affordability, deposit plus affordability, or price range is also established.
- **Early-end rule:** If the customer defers before affordability discussion, use `N/A`.

## Q06 — Trade-in

**Question:** Did the Sales Executive ask whether the customer has a trade-in?

- **Dependency:** A deal-structure opportunity.
- **Opportunity reached:** Once finance, cash, quotation, budget or deal structure is discussed enough for a trade-in question to be fair.
- **Yes:** The SE asks whether there is a trade-in or explicitly confirms the customer's volunteered trade-in/no-trade-in status.
- **No:** Deal structure is reached, but trade-in status is omitted despite a fair opportunity.
- **N/A:** The call ends before trade-in qualification; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** Use live trade-in, trading-in, swap/swop or no-trade-in wording.
- **Early-end rule:** If the customer defers before deal structure, use `N/A`.

## Q07 — Trade-in settlement

**Question:** If there is a trade-in, did the Sales Executive ask whether it is settled or still financed?

- **Dependency:** A trade-in must exist.
- **Opportunity reached:** After the trade-in's existence is confirmed.
- **Yes:** The trade-in's settlement, paid-up, still-financed, owing, bank or settlement-requirement status is established.
- **No:** A trade-in exists, but its settlement status is omitted despite a fair opportunity.
- **N/A:** No trade-in exists; trade-in status was never reached because the call ended; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** Use live settled, paid-up, paid-off, still-financed, owing or bank-status wording.
- **Early-end rule:** If the call ends before a trade-in is established, use `N/A`. Never use `No` merely because there is no trade-in.

## Q08 — Cash or finance

**Question:** Did the Sales Executive confirm whether the customer is buying cash or using finance?

- **Dependency:** The purchase/payment stage is reached.
- **Opportunity reached:** Once quotation, finance, cash, affordability or purchase terms are discussed.
- **Yes:** The payment method is clearly established as cash, finance or an equivalent purchase method.
- **No:** The purchase discussion reaches the payment stage, but the payment method is not established.
- **N/A:** The call ends before payment discussion; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** Use live cash, finance, bank-finance or equivalent payment-method wording. Do not assume finance solely because an instalment is mentioned unless the live exchange clearly establishes financing.
- **Early-end rule:** If the customer defers before payment method can reasonably be established, use `N/A`.

## Q09 — In-house F&I department

**Question:** Did the Sales Executive market the in-house F&I department?

- **Dependency:** Finance is relevant, unless the SE voluntarily markets F&I.
- **Opportunity reached:** Once finance, an application, bank submission, monthly payment or dealership finance support is discussed.
- **Yes:** The SE communicates the value of dealership finance/F&I support, such as assistance with finance, bank negotiation, competitive or best bank rates, approval support, structuring the deal through finance, dealership finance/F&I capability, business-manager support, or relevant finance/VAPS support.
- **No:** Finance is relevant and a fair F&I opportunity is reached, but the dealership's in-house finance capability, benefit or support value is not marketed.
- **N/A:** A genuine cash-only path makes F&I irrelevant; the call ends before finance/F&I becomes relevant; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** A bare question of “cash or finance?” is Q08, not Q09. A bare mention of finance, a bank, MFC, an instalment, an application, or a payment plan is not enough for Q09. If Q09 is `N/A`, the comment must explain why F&I was not applicable or not reached and must not say that F&I value was communicated.
- **Early-end rule:** If the customer defers before finance relevance is reached, use `N/A`.

### Q05/Q09 regression examples

- **Q05 positive regression:** For call UUID `[PRIVATE_REFERENCE_REMOVED]`, Q05 must be `Yes` when the transcript shows the SE asked for budget/monthly instalment expectation and the customer discussed shortfall/top-up affordability. Reason: budget/monthly instalment qualification plus an affordability-based customer answer establishes spending capacity.
- **Q05 negative regression:** If the customer only asks “what is the price?” and the SE only gives the vehicle or model price, Q05 must not become `Yes` unless spending capacity, affordability, repayment expectation, deposit plus affordability, price range or shortfall/top-up affordability is also established.
- **Q05 unanswered-question regression:** If the SE asks for a budget or monthly target but the customer responds only that the purchase is still under discussion, Q05 is `No` when affordability was reached. The question wording and nearby vehicle prices cannot supply the absent customer answer.
- **Q09 consistency regression:** If the comment says F&I value was communicated, Q09 cannot be `N/A`. If Q09 is `N/A`, the comment must explain why F&I was not applicable or not reached.

## Q10 — Formal OTP or quotation

**Question:** Did the Sales Executive offer a formal OTP or quote?

- **Dependency:** A proposal opportunity exists or the customer requests pricing or a quotation.
- **Opportunity reached:** Once the customer requests a quotation/price or the purchase/finance discussion progresses far enough for a formal proposal.
- **Yes:** The SE offers or commits to send a formal quotation, OTP, written offer, written deal proposal, written pricing or an equivalent formal proposal.
- **No:** A substantive Sales interaction reaches a fair proposal opportunity, but no formal quotation, OTP or written proposal is offered.
- **N/A:** The call ends before proposal opportunity; the discussion is genuinely too early; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** Use live quotation, quote, OTP, written offer, formal pricing send-through or deal-proposal wording. General vehicle details, a brochure, specifications or a verbal price alone do not satisfy this rule.
- **Early-end rule:** If the customer defers before proposal opportunity, use `N/A` unless a quotation was already requested or offered.

## Q11 — Visit, viewing, appointment or test drive

**Question:** Did the Sales Executive make a genuine attempt to arrange a dealership visit, viewing, appointment or test drive?

- **Dependency:** A visit/appointment opportunity is fairly reached.
- **Opportunity reached:** Once the call progresses to a vehicle viewing, availability, appointment, test drive or sufficiently mature purchase discussion.
- **Yes:** The SE genuinely attempts to arrange a dealership visit, viewing, appointment or test drive.
- **No:** The call reaches a fair visit/appointment opportunity, but the SE makes no genuine attempt.
- **N/A:** The call ends before the visit stage; a dealership visit is genuinely irrelevant; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** Use live visit, viewing, dealership, branch, appointment or test-drive wording. A callback alone is not an appointment attempt.
- **Appointment-status rule:** Report `appointmentSecured=true` only when the customer accepts a date plus a specific time or bounded time window. A vague day, weekend, “later” or “come through sometime” is not a secured appointment, even if Q11 is `Yes` for the genuine attempt.
- **Early-end rule:** If the customer defers before appointment stage, use `N/A`.

## Q12 — Usable follow-up contact route

**Question:** Did the Sales Executive confirm the prospect's contact information?

- **Dependency:** Q12 measures the Sales Executive's collection or confirmation of the prospect's own contact information when relevant for customer follow-up. Giving the salesperson's own contact details is a different behaviour and does not satisfy Q12. Apply the N/A exception when the prospect supplies their own usable contact details unsolicited and the SE neither asks for nor confirms them.
- **Opportunity reached:** Once callback, quotation sending, document sending or follow-up is discussed and there is a fair opportunity to request or confirm the prospect's contact information. If the prospect supplies usable details without being asked and the SE does not confirm them, apply the unsolicited-details N/A exception rather than a missed-opportunity No.
- **Yes:** The SE asks for the prospect's own phone number, WhatsApp number, email address or another relevant usable contact detail and the prospect supplies or confirms it; or the SE confirms contact details the prospect volunteered. For example, the SE asks whether the current number is the prospect's suitable contact or WhatsApp number and the prospect agrees. One relevant usable route is sufficient; reading every digit aloud is not required. The prospect volunteering details alone, without an SE request or confirmation, is N/A rather than Yes.
- **No:** Contact collection or confirmation is relevant, a fair opportunity is reached, and neither a qualifying SE-led exchange nor an N/A condition applies. Giving only the salesperson's own number or inviting the prospect to contact the salesperson does not count as confirming the prospect's details and remains No when applicable. Do not use No for the specific unsolicited-details N/A exception.
- **N/A:** The prospect supplies their own usable contact details without being asked and the SE neither solicits nor confirms them: the prospect has removed the need to ask, so this is not credited or penalised as an SE action. Separately, use N/A if the call ends before a fair contact-confirmation opportunity, there is no relevant continuing contact context, wrong-person/no-enquiry context, or no live interaction. A number supplied in response to an SE request is not unsolicited. A qualifying SE request-and-response or confirmation makes Q12 Yes even if details were initially volunteered. Material ambiguity about speaker roles, contact ownership or confirmation requires human/audio review, not an automatic N/A.
- **Evidence:** Identify who initiated the contact-details exchange and quote the prospect's response and any SE confirmation. Yes requires an SE request answered by the prospect or SE confirmation of the prospect's volunteered details. N/A for unsolicited provision requires evidence that a usable prospect-owned route was supplied without an SE request or confirmation. Do not assume that an ambiguous 'okay' or 'thanks' confirms the details. Giving the salesperson's own number, caller ID, CRM details, metadata or the fact that a call connected cannot alone prove Q12. Keep Q12 separate from Q13 acceptance. Incomplete details, unclear ownership or mixed exchanges that do not clearly fit a branch require human/audio review; do not automatically apply the unsolicited-details exception.
- **Early-end rule:** If the call ends before the prospect's contact information could reasonably be requested or confirmed, use N/A. Preserve a qualifying SE request-and-response or confirmation established before a later ending. Unsolicited prospect details without an SE request or confirmation remain N/A under the specific exception, even if the call reaches its normal end.

## Q13 — Clear next steps

**Question:** Did the Sales Executive agree on a next step or obtain the customer's acceptance of an offer of future Sales assistance?

- **Dependency:** A meaningful live Sales interaction reaches closing, follow-up or an offer of future Sales assistance, or a callback has already been agreed. Browsing or not being ready to buy does not by itself exclude an accepted offer of future Sales help.
- **Opportunity reached:** Once the parties discuss follow-up, sending information or documents, an appointment, a callback, or the Sales Executive's availability to help the customer with a future Sales enquiry.
- **Yes:** Either (1) a specific future action is identified, its owner is clear and the customer accepts or commits to it; or (2) the Sales Executive offers future Sales assistance and the customer accepts or positively acknowledges that offer in the context of the conversation. Under the accepted-assistance option, no specific next task, appointment, deadline or callback time is required. A clearly accepted invitation to ask for help later can qualify, including on a browsing enquiry.
- **No:** The interaction reaches a fair closing, follow-up or future-assistance opportunity, but neither a specific next action nor an offer of future Sales assistance is accepted by the customer. A one-sided offer, vague promise, ignored invitation or internal intention alone does not qualify.
- **N/A:** There is no meaningful live Sales interaction, or the call ends before any next-step or future-assistance opportunity is reasonably reached. Use N/A when neither next steps nor future Sales assistance is relevant. A browsing enquiry or a completed discussion is not automatically N/A when future Sales help is offered and accepted. Material uncertainty about the offer, customer acceptance or speaker roles requires human/audio review rather than a forced Yes or No.
- **Evidence:** Quote both the agreed action or offer of future Sales assistance and the customer's response supporting acceptance. Acceptance can be explicit or contextually clear, such as 'Thanks, I will' in direct response to an offer; it need not be a formal promise. Silence or a routine farewell alone is not proof. A courteous response counts only when the context clearly links it to acceptance of the assistance offered. This draft broadens the baseline specific-action requirement for customer-accepted future-assistance offers only; it does not make every vague or one-sided promise count. Evaluate Q12 separately: Q13 acceptance does not prove confirmation of the prospect's contact information.
- **Early-end rule:** A brief call with an agreed callback or a customer-accepted offer of future Sales assistance can be Yes. An unexpected ending before either opportunity is reached is N/A. Preserve a qualifying agreement or acceptance established before a later interruption.

## Q14 — Respect and professionalism

**Question:** Did the Sales Executive treat the customer respectfully throughout the call?

- **Dependency:** Enough live interaction exists to assess conduct.
- **Opportunity reached:** Once there is assessable live employee/customer interaction.
- **Yes:** The SE is professional, patient and appropriately helpful, with no supported disrespect.
- **No:** There is clear disrespect, hostility, mockery, dismissiveness, argumentativeness, inappropriate language or inattentiveness to expressed needs.
- **N/A:** There is no meaningful live interaction or the fragment is too short to assess fairly.
- **Evidence:** Use live tone and conduct evidence. Do not infer professionalism from metadata.
- **Early-end rule:** If the call is too fragmentary to assess, use `N/A`. If meaningful interaction exists, Q14 must be `Yes` or `No`.

---

## 8. Controlled inference

Controlled inference is permitted only when the transcript strongly supports a behaviour despite minor ASR noise. Examples include a clear self-introduction pattern with a garbled name or an unmistakable enquiry reference with a mis-transcribed model.

Controlled inference must never:

- invent an unasked qualification question;
- convert an assumed prior action into `Yes`;
- use CRM or IVR metadata as performance evidence;
- turn a vague future possibility into an agreed next step or secured appointment; or
- resolve material ambiguity that requires audio/human review.

Whenever it is used, set `controlled_inference_used=true` and include the words `Controlled inference used` in the reason.

---

## 9. Scoring

For a scoreable Sales call:

```text
scorePercentage = Yes / (Yes + No) × 100
```

- Exclude every `N/A` from the numerator and denominator.
- Display the calculation, for example: `10 Yes / 12 applicable = 83.3%`.
- A blank answer is invalid; use an evidence-supported `Yes`, `No` or `N/A`.
- If `Yes + No = 0`, the percentage is `null` or blank and the call is `Not Scored`.
- Do not assign `Excellent`, `Good Efficient`, `Needs Coaching` or `Requires Attention` unless a current approved threshold table is supplied.

---

## 9A. Transcript Cleaning Requirement — Spelling and Speaker Separation Without Rewriting

The evaluator must create a cleaned version of the transcript for every call classified as `full_sales` or `minimal_sales`.

This requirement is additional to the existing DCE Sales evaluation rules. It must not change, override, reinterpret or weaken any Q01–Q14 rule.

### New output field

Add the following field to the evaluation output:

`cleanedTranscriptText`

For:

- `full_sales` → populate `cleanedTranscriptText`
- `minimal_sales` → populate `cleanedTranscriptText`
- `not_scorable` → `cleanedTranscriptText: null`
- `non_sales` → `cleanedTranscriptText: null`

### Required cleanup

`cleanedTranscriptText` must remain a faithful transcript cleanup, not a rewritten conversation.

The evaluator must:

1. Correct obvious misspellings and speech-to-text spelling errors when the intended word is strongly supported by the surrounding transcript.
2. Correct punctuation, sentence boundaries, capitalization and spacing where doing so does not change the speaker's wording.
3. Remove obvious non-conversation transcript noise such as IVR instructions, automated menu messages, whisper messages, call-routing messages, hold music descriptions, transcription system text, duplicated ASR fragments and meaningless audio artefacts.
4. Separate the Sales Executive and customer into turns where the speaker role is reliable from the conversation. Preferred labels are `Sales Executive:` and `Customer:`.
5. Use neutral speaker labels only when a speaking turn is real but the role cannot reliably be determined. Do not guess a role for a turn.
6. Preserve fillers, repetitions, false starts, awkward phrasing, tone, objections, sentiment, poor wording and unprofessional wording when they are part of what was actually said.

### Not allowed

The evaluator must not:

- rewrite the transcript into cleaner sales language;
- paraphrase, polish, summarize, reorder or translate the conversation;
- convert fragments into complete sentences when the speaker did not speak complete sentences;
- replace a speaker's words with synonyms or more professional phrases;
- change grammar beyond punctuation, capitalization and spacing;
- normalize amounts, dates, vehicle names, dealership names, product names or finance terms into different wording unless it is strictly an obvious spelling correction supported by the transcript;
- convert spoken numbers into a different numeric representation when that changes wording or certainty;
- infer missing words;
- add questions the Sales Executive did not ask;
- add answers the customer did not give;
- manufacture missing sales behaviour;
- convert an unclear statement into a definite statement;
- infer a budget, trade-in, cash/finance answer, appointment, contact detail or next step that was not said;
- improve the Sales Executive's sales performance;
- remove poor, rude or unprofessional wording if it was actually spoken.

If a passage is unclear and cannot safely be corrected as a misspelling or assigned to a speaker, keep the uncertainty rather than guessing, for example: `Customer: I was looking for the [unclear] model.`

### Dealer and Sales Executive relevance

The cleaned transcript must still make sense within a motor dealership sales environment, but dealership context may be used only to correct obvious misspellings already present in the transcript.

Mappings may assist with spelling only when the transcript itself already provides sufficient spoken evidence for the identity. A mapping, CRM record, caller ID, lead record or other external metadata must never be inserted into `cleanedTranscriptText` as though it was spoken during the call.

### Sales evaluation relationship

Call processing order must be:

**Raw Transcript → Call Classification → Transcript Cleaning → Q01–Q14 Evaluation**

The original classification rules remain unchanged. Only `full_sales` and `minimal_sales` proceed to Q01–Q14 scoring.

Existing completed DCE Sales evaluations may be explicitly reopened only by a human instruction for re-evaluation. During that controlled re-evaluation, regenerate `cleanedTranscriptText` under this rule and re-score Q01–Q14 from the cleaned transcript and the selected primitive evidence. Do not alter the Logimeter source row.

`cleanedTranscriptText` is a transcript cleanup of the raw transcript and must not create new evidence. Every `Yes`, `No` or `N/A` decision must still be supportable by the actual conversation contained in the raw transcript. Cleaning may make existing wording easier to read and speaker roles easier to follow, but it may never manufacture or improve evidence.

### Runtime storage contract

The evaluation packet may carry `cleanedTranscriptText`. The internal alias `clean_transcript` remains accepted for backward compatibility during active worker catch-up. If both are supplied, they must represent the same normalized text.

For DCE writes, the guarded writer stores the final cleaned transcript only in the approved DCE clean-transcript fields and keeps the full transcript out of `cra95_evaluationjson` and logs.

### Core principle

The cleaned transcript should read like the same transcript after spelling, punctuation, capitalization, spacing, noise cleanup and reliable SE/customer turn separation. It must preserve actual wording and never turn the call into a better-written version of itself.

---
## 10. Required evaluation output

Every evaluation must include:

- Call classification: `full_sales`, `minimal_sales`, `not_scorable` or `non_sales`.
- Meaningful live interaction: `true` or `false`.
- Transcript quality: `good`, `usable`, `poor` or `severe_failure`.
- `cleanedTranscriptText`: populated for `full_sales` and `minimal_sales`; `null` for `not_scorable` and `non_sales`.
- Raw spoken SE name, canonical mapped SE name, Dataverse SE identifier when available, mapping source, mapping basis and identity-resolution confidence.
- Raw spoken dealership name, canonical mapped dealership name, Dataverse dealership identifier when available, mapping source, mapping basis and identity-resolution confidence.
- A short call summary.
- Exactly one result for each Q01–Q14, in order, for Sales evaluations.
- Score percentage and applicable-question denominator.
- Strengths, coaching opportunities and recommended next action.
- QA flags for low confidence, poor transcript quality, controlled inference, uncertain Q09–Q13, question-set ambiguity or evidence requiring audio review.

Use these fields for each question:

| Field | Requirement |
|---|---|
| `questionNumber` | Q01 through Q14 |
| `questionText` | Exact question meaning from this rule bank |
| `result` | Exactly `Yes`, `No` or `N/A` |
| `applicable` | `true` only for `Yes` or `No`; `false` for `N/A` |
| `evidence` | Short live-transcript excerpt, or explicit reason why no stage/opportunity was reached |
| `reason` | The specific question rule applied to the evidence |
| `confidence` | `high`, `medium` or `low` |
| `controlled_inference_used` | `true` or `false` |

---

## 11. Independent validation/source-check mode

When an existing human or AI evaluation is supplied, Codex must not treat either answer set as ground truth. Validate each question independently against this rule bank and the live evidence.

For each question, report:

| Field | Meaning |
|---|---|
| Original answer | Existing answer being checked |
| Validated answer | Independent `Yes`, `No` or `N/A` decision |
| Rule applied | Exact Q-specific opportunity, dependency, evidence or early-end rule |
| Evidence | Live transcript evidence or applicability explanation |
| Outcome | `Match`, `Confirmed original error` or `Unresolved` |
| Confidence | `high`, `medium` or `low` |

Recalculate the score from the validated answers. Report disagreements separately from confirmed errors. An unresolved disagreement is not automatically an AI error or a human error.

---

## 12. Final validation checklist

Before returning an evaluation, Codex must confirm:

- [ ] The call was classified before scoring.
- [ ] `cleanedTranscriptText` was created after classification and before Q01-Q14 scoring, or set to `null` for `not_scorable`/`non_sales`.
- [ ] The cleaned transcript removes only transcript noise/formatting problems and does not invent facts, speech, questions, answers or sales behaviours.
- [ ] Only the correct Sales question set was used.
- [ ] An unrecognisable SE name was checked against `SE Mapping DV`.
- [ ] The raw spoken name and canonical mapped identity were kept separate.
- [ ] The SE mapping was not used to award Q02 when no live self-introduction occurred.
- [ ] Ambiguous or conflicting SE matches were marked `Unresolved` with `SE_IDENTITY_REVIEW`.
- [ ] An unrecognisable dealership name was checked against `Dealership Mapping DV`.
- [ ] The raw spoken dealership wording and canonical mapped dealership were kept separate.
- [ ] Dealership mapping or routing data was not used as proof of any Q01–Q14 behaviour.
- [ ] Ambiguous or conflicting dealership matches were marked `Unresolved` with `DEALERSHIP_IDENTITY_REVIEW`.
- [ ] Q01–Q14 appear exactly once and in order.
- [ ] Each question was checked against its own rule.
- [ ] Each `No` represents an applicable, reached opportunity that was missed.
- [ ] Each `N/A` has a dependency, context or early-ending reason.
- [ ] Questions not reached because the call ended early were not marked `No`.
- [ ] Questions reached before a later call ending were still evaluated.
- [ ] No IVR, lead, CRM, caller-ID or prior-evaluation metadata was used as performance evidence.
- [ ] Q07 is `N/A` when no trade-in exists.
- [ ] Q09 is not awarded for a generic bank/payment reference alone.
- [ ] Q10 is not awarded for general vehicle details without a formal quote/OTP/proposal commitment.
- [ ] Q11 attempt and `appointmentSecured` status are reported separately.
- [ ] Q12 is supported by a qualifying SE request-and-response, SE confirmation of prospect-owned details, or a documented N/A exception.
- [ ] Q13 is supported by either a specific accepted next action or a customer-accepted offer of future Sales assistance.
- [ ] Q14 is `Yes` or `No` whenever meaningful live interaction exists.
- [ ] Controlled inference is disclosed.
- [ ] The score excludes `N/A` and shows its denominator.
- [ ] No rating threshold was invented.

---

## 13. Minimum response format for the Codex chat

```markdown
Classification: full_sales | minimal_sales | not_scorable | non_sales
cleanedTranscriptText: [same wording as the call after spelling/punctuation/spacing/noise cleanup and reliable Sales Executive/customer turn separation for full_sales/minimal_sales, otherwise null]
Meaningful live interaction: true | false
Raw spoken SE name: [transcript rendering or blank]
Resolved SE: [canonical SE name or Unresolved]
SE Mapping DV ID: [Dataverse ID when available]
SE resolution source: Live transcript | SE Mapping DV | Human review
SE resolution confidence: high | medium | low
Raw spoken dealership name: [transcript rendering or blank]
Resolved dealership: [canonical dealership name or Unresolved]
Dealership Mapping DV ID: [Dataverse ID when available]
Dealership resolution source: Live transcript | Dealership Mapping DV | Routing metadata | Human review
Dealership resolution confidence: high | medium | low
Score: [Yes count] / [applicable count] = [percentage]%
Rating: [approved rating or Unresolved—thresholds not supplied]

| Q | Result | Applicable | Evidence | Rule-based reason | Confidence |
|---|---|---:|---|---|---|
| Q01 | Yes/No/N/A | true/false | “...” | ... | high/medium/low |
...
| Q14 | Yes/No/N/A | true/false | “...” | ... | high/medium/low |

QA flags: []
Strengths: ...
Coaching actions: ...
```

End of rule bank.



## Version approval

Reason: Rollout candidate for the user-approved sales-q01-q14-2026-09-06-15-36-05 structured Q12 and Q13 changes. This candidate preserves the old immutable workbench approval record, fixes the final checklist wording for Q12 and Q13, and remains inactive until DCE runtime migration and release checks pass.

Evaluator activation is tracked separately. Approval of this bank does not rewrite historical evaluations.

## Evaluator-provenance release guidance (2026-09-28)

This version records the user-approved move of new DCE evaluation and human-review generation to gpt-6-sol with medium reasoning. It does not change any Q01–Q14 scoring criterion, historical decision, or approved transcript primitive.

Before a new score or review item is written, verify the producing model and reasoning effort from the active task/runtime, then require the matching `gpt-6-sol` provenance in the guarded writer input and persisted DCE evaluation. Never label work produced by a different or unverified model as `gpt-6-sol` or `gpt-5.6-sol`. The review queue does not have a model field; retain a truthful run-level provenance attestation and do not imply a queue row stores the model.

Preserve all previously completed DCE rows, human Pending/In Review decisions, and historical provenance. Source Logimeter remains GET-only. Continue to require both approved source transcript primitives and hashes, current source and queue ETags, the active bank, global lease, alternate key, five-evaluation/five-review per-run caps, and post-write readback/audit.

Release control for the earlier model-provenance migration required positive and negative provenance regressions, shadow checks, a Dataverse CE Fix update, and the then-current release gate. The DCE-only source-routing release below has a newer, explicit user-approved approval policy; it does not remove technical provenance validation or Dataverse CE Fix logging.

## Source-only follow-up routing clarification (2026-09-28)

This version removes transcript-phrase and dialer-instruction follow-up routing. Only source `cra95_calltype = Lead Tracker` establishes that route. It preserves Q01–Q14 criteria, both approved Logimeter transcript primitives, gpt-6-sol medium provenance, and all historical DCE and human-review decisions. For this DCE-only routing release, the user is the sole rollout approver and has explicitly removed [legacy approvers] signoff, shadow, canary, and other administrative release-approval gates. No [legacy approvers] approval is claimed. Technical positive/negative regression, source GET-only, CE Fix logging, consistent bank/writer activation, and post-release verification remain mandatory.

## User-adjudicated September worklist review tagging (2026-09-28)

The user supplied the exact private list of call identifiers in `[private workbook filename removed]` (SHA-256 `[private workbook fingerprint removed]`) and directed that substantive Sales calls on this list have no review tag. This instruction does not alter Q01–Q14 answers or the live-evidence requirements. An Excel identifier is routing/scope data, not performance evidence.

For a listed UUID with one current source row, both approved source transcript primitives, a meaningful live Sales conversation, no completed DCE row, and no protected Pending/In Review human queue item, `minimal_sales` classification alone must not add `MINIMAL_SALES_EDGE` or set Review Required to Yes. Retain any separate, evidence-supported QA flag: poor or contradictory transcript, controlled inference, speaker/identity ambiguity, uncertain Q09–Q13, low confidence, or another approved review reason still requires human review. Generic `Calling prospect`, `press 1 to call the prospect`, or `Prospect follow-up` dialer wording does not establish follow-up routing. Source `cra95_calltype = Lead Tracker` remains subject to its existing hard-exclusion and follow-up protections.

Do not score voicemail, no-live, non-Sales, or unusable evidence from the spreadsheet assertion. Do not overwrite an existing DCE score, review queue row, or human decision. The writer must verify exact allowlist membership against the frozen workbook hash or a versioned derived allowlist, current Logimeter source/queue ETags, both approved source primitive hashes, the global lease, active bank, alternate key, write caps, and post-write readback. Existing rows bearing an automatic `MINIMAL_SALES_EDGE` review indicator require separately logged, controlled correction; this rule does not silently rewrite them.

Positive regression: a listed `Connect` UUID with a short, live, unambiguous vehicle enquiry that ends with an accepted callback may be scored under Q01–Q14 without Review Required solely because it is `minimal_sales`, even when the dialer says `Prospect follow-up`. Negative regressions: the same call outside the exact list keeps the existing automatic minimal-sales review; a listed voicemail/no-live call remains excluded; a listed call with material QA ambiguity retains review; any existing Completed DCE or active human review is preserved; a source row marked `Lead Tracker` remains excluded. Activation requires CE Fix logging, positive/negative tests, source-only static checks, the applicable release gate, and live readback; historical rows are not silently rewritten.

## Protected local-decision prewrite gate (2026-09-29 candidate)

Before a new DCE score is written, the guarded writer must check authoritative local reconciliation decisions for the exact source row and, for the frozen September workbook allowlist, the exact workbook call UUID. A human/audio deferral, policy exception, terminal exclusion, or malformed/missing required workbook decision is a safety stop even when the source transcript is present and the evaluation packet asks for Review Required=No. Only an explicitly retryable pending-transcription, pending-secondary, deferred-scoring, or ready-for-scoring decision may proceed to the other independent writer gates. This gate does not authorize a score: source and queue ETags, both approved transcript primitive hashes, model provenance, active rule bank, global lease, alternate key, write caps, and post-write readback remain mandatory. It does not change Q01–Q14 scoring criteria, correct existing rows, or close human reviews. Existing-row re-evaluation remains subject to separate exact-row authorization and protections.

Positive regression: a workbook call with a current, exact-source retryable decision can proceed to the remaining independent writer gates. Negative regressions: both documented workbook calls with protected human/audio decisions, any policy-exception decision, absent workbook decision, source identity drift, and malformed decision state stop before slot reservation or write. This guidance is inactive until CE Fix logging, positive and negative regressions, shadow checks, guarded writer integration, and coordinated rulebank/config release are verified.

## Scored minimal-Sales Review Queue reconciliation (2026-09-29 candidate)

When a non-workbook call already has exactly one independently verified Completed DCE row classified `minimal_sales`, marked `Review Required=Yes` with an open DCE review status, and the separate DCE Review Queue has no row, the review writer may create one item labelled `scored_minimal_sales`. It must not describe the scored classification as ambiguous or imply that the DCE score is absent. The review reason must include `MINIMAL_SALES_EDGE` and the detail must explain that human review of an already scored edge call is pending.

Creation requires the exact current source and DCE identities and ETags, both current approved source primitive hashes checked against the prepared evidence packet, the DCE-stored Soniox hash and AssemblyAI hash when that provider is proven, the DCE clean-transcript hash, a zero-row queue read under the active alternate key, active gpt-6-sol medium host provenance, active rulebank, global lease, and five-new-review cap. A secondary primitive marked Secondary/Unverified is not stored as an AssemblyAI hash in DCE; compare it to the prepared packet and source ETag instead. Post-write readback must verify the new queue row and leave the DCE score and human-owned fields unchanged. Any workbook allowlist UUID, existing queue item, non-Completed or non-minimal DCE, Review Required=No, closed review state, or evidence drift is a safety stop. Only `cra95_calltype = Lead Tracker` triggers follow-up routing; this path never removes its separate hard exclusion. No historical DCE row is rewritten.

Positive regression: one current non-workbook Completed minimal-Sales DCE with Review Required=Yes, matching approved source hashes and no queue row may create exactly one truthful `scored_minimal_sales` review item. Negative regressions: workbook call, duplicate or absent DCE, existing human queue, DCE ETag drift, source primitive or clean hash drift, missing approved primitive, non-minimal or non-Completed DCE, Review Required=No, closed review state, source identity drift, and unverified model provenance.

## Audio-linked workbook coverage and provisional review (2026-09-29 candidate)

The user confirmed the exact 687 distinct IDs in the audio-linked September workbook as Sales evaluation targets. This later instruction supersedes the earlier no-review-only disposition for those 687 IDs, including their 677 overlaps with the original 759-ID workbook. It does not rewrite an existing Completed DCE score, close an open human Review Queue item, or change the other original workbook IDs. The workbook identifier establishes evaluation scope, not what any seller said; only live transcript evidence supports Q01–Q14 answers. Dialer instructions, voicemail and IVR are not customer speech. Only source `cra95_calltype = Lead Tracker` marks follow-up.

For an in-scope target with one usable approved transcript primitive, assess the live conversation even if the other primitive is absent. Hash and compare each available primitive; represent an absent primitive as null rather than fabricating a second transcript. Infer seller and buyer roles from conversational ownership when generic speaker labels are wrong or reused, but use a review flag when residual role uncertainty affects an answer. For an unsupported individual Q answer, apply its Yes/No/N/A and uncertainty rules without inventing dialogue. A substantive Sales target with materially uncertain Q evidence receives a grounded provisional Q01–Q14 DCE evaluation with Review Required=Yes and an open human Review Queue preserved. A review flag is not a substitute for a DCE row.

Before a new score, verify exact workbook membership, current source identity and ETag, current open Review Queue identity and ETag when present, hashes of all available approved transcript primitives, local decision history and the exact authorized re-adjudication, active rulebank, actual gpt-6-sol host provenance, global lease, alternate key, five-score cap and post-write Dataverse readback. Preserve completed DCE rows and human-owned decisions; never convert unsupported terminal or deferral labels into completion. If no live conversation or no usable approved transcript exists, record a precise pending or terminal reason instead of a fabricated score.

Positive regressions: an unscored clear Sales call newly listed in the audio workbook receives a grounded Review Required=No evaluation; a listed call with material uncertainty receives a provisional Review Required=Yes score without closing its human queue; a single usable primary transcript supports evaluation with a null secondary hash; and wrong or reused speaker labels do not alone exclude a call when conversational roles are clear. Negative regressions: non-listed calls retain baseline review behavior; an active human queue is not overwritten; a completed DCE row is not duplicated; missing all approved transcript evidence, voicemail/IVR-only evidence, source or queue ETag drift, available-primitive hash drift, model mismatch, lease failure and alternate-key mismatch stop a write.

## Conservative role separation for a merged live speaker block (2026-09-29 candidate)

An approved ASR primitive may place both sides of a live Sales conversation in one generic speaker block. That label is not proof that only one person spoke. For an audio-workbook target, an evaluator may separate a cleaned transcript into `Sales Executive:` and `Customer:` turns only at utterance boundaries supported by the dialogue itself. The spoken-word sequence and order must remain a conservative subsequence of an approved primitive; speaker labels add no dialogue. A reliable anchor can be a live seller self-introduction, the seller's ownership of a vehicle enquiry or quotation, or a clear question-and-answer exchange that identifies the buyer. A dialer prompt, IVR, source metadata, guessed identity, or generic ASR speaker number is not a role anchor. Preserve ambiguous fragments as unassigned; do not attribute them merely to satisfy the writer's explicit-label gate.

If both roles cannot be separated reliably, do not force a DCE write. Retain the precise evidence blocker and an open human review where applicable. If reliable turns support a score but some question-level attribution remains uncertain, use the question's N/A/uncertainty rule, retain an appropriate QA flag, and write only a grounded provisional evaluation with Review Required=Yes. This clarification does not relax source/queue ETags, hashes of every available primitive, alternate key, global lease, actual model provenance, run caps, or post-write readback.

Positive regression: a merged generic speaker block with an explicit live seller self-introduction and clear buyer answers can be split at those utterance boundaries without changing words or their order. Negative regressions: an IVR-only block, an unanchored generic speaker block, an invented/reordered answer, a changed primitive hash, or a guessed identity cannot be made scoreable by adding role labels.

## Audio-workbook writer version gate correction (2026-09-29 candidate)

The guarded writer must recognize this exact versioned audio-workbook rulebank when it is the configured active bank and its SHA-256 is verified. A hardcoded check for the prior source-type-only-hotfix version must not reject a later approved audio-workbook bank before the existing call-level safeguards execute. This is a version compatibility correction only: it does not relax exact workbook membership, one current source row, source and open-queue ETags, every available primitive hash, the exact-call authorization, global lease, alternate key, actual model provenance, five-score/five-review caps, conservative cleaned-transcript validation, or post-write readback. A different or unverified bank still fails closed.

Positive regression: this exact v2 active bank reaches the unchanged exact-call guard for a user-designated Sales target. Negative regressions: the prior-version-only writer rejects v2, a different or hash-drifted bank is rejected, a missing authorization or changed source/queue evidence stops the write, and an existing Completed DCE row remains protected.

## Source call-type-only Lead Tracker writer gate (2026-09-29)

This version resolves a writer mismatch with the already approved source-only follow-up rule. Only the Logimeter source field `cra95_calltype` equal to `Lead Tracker` establishes the follow-up route or its hard exclusion from ordinary Sales scoring. Auxiliary source routing text, a lead reference, a lead name or state, and automated dialer phrases are not substitutes for that call type. This change affects routing only; it does not modify Q01–Q14 answers, transcript selection, existing DCE rows, or human decisions. Preserve all ETag, transcript-hash, global-lease, alternate-key, model-provenance, run-cap, and readback protections.

Positive regression: a source row whose call type is exactly `Lead Tracker` remains excluded from ordinary Sales scoring. Negative regression: a `Connect` or `Call Manager` Sales call remains eligible when any auxiliary source field contains the words `Lead Tracker`, and a dialer phrase about calling a prospect does not mark follow-up.

## Protected open-review workbook reconciliation (2026-09-29)

For an exact audio-workbook Sales target whose earlier local decision was `review_policy_conflict` solely because the older workbook instruction required Review Required=No, the later user adjudication permits a newly grounded provisional Q01–Q14 evaluation with Review Required=Yes. This narrow precedent applies only when there is no existing DCE row and exactly one current, open, unresolved human Review Queue item. The queue is preserved, not replaced or resolved. The earlier decision remains in the audit history; an independently staged exact-call authorization must explicitly name it and bind the current source and queue identities and ETags, hashes of every available approved transcript primitive, and the evaluated packet hash.

Positive regression: an audio-workbook target with the stated earlier conflict, one current open queue and no DCE can pass to the remaining guarded writer checks after grounded evaluation and exact-call authorization. Negative regressions: an out-of-scope target, a completed DCE, a closed or duplicate queue, a different protected deferral or policy exception, source or queue ETag drift, primitive or evaluation hash drift, missing model attestation, lease, alternate-key or run-cap failure must still stop the write. This precedent changes neither Q01–Q14 scoring criteria nor a human decision.

## Historical clean-hash compatibility for protected review (2026-09-29 candidate)

An open Review Queue item may carry a SHA-256 of an earlier cleaned transcript derived from the same approved source primitives. A newly prepared, conservatively cleaned and role-separated Sales evaluation need not have identical cleaned-text bytes. For an exact protected workbook call, do not equate that historical derived-text hash with a source-primitive hash, and do not copy it into the new evaluation. Require the current source and queue identities and ETags, the queue fingerprint and unresolved status, matching hashes for every currently available approved source primitive, the new evaluation's own cleaned-text hash and immutable evaluation-packet hash, no pre-existing DCE row, host-model provenance, global lease, alternate key, per-run caps and post-write readback. Preserve the open queue and its historical hash unchanged. A missing or changed source primitive, missing queue fingerprint, changed queue ETag, unsafe transcript cleaning, or mismatch in the new packet hash still stops the write.

Positive regression: a protected open review with matching current primitive hashes and queue ETag may receive a grounded provisional Q01–Q14 score even when its older queue clean-text hash differs from the newly role-separated clean text. Negative regressions: source/queue ETag drift, either primitive-hash drift, absent fingerprint, resolved queue, duplicate DCE, or new evaluation clean-text/packet hash drift fails closed. This changes no question criteria or human outcome.

## Missing historical derived clean hash (2026-09-29 candidate)

Some legacy open Review Queue items have no historical cleaned-transcript SHA-256. That nullable, derived field is not an approved source primitive and must never be invented or copied into the new evaluation. For an exact protected workbook target, its absence alone does not block a grounded provisional Q01–Q14 score when the current source and queue identities and ETags, valid queue fingerprint and unresolved status, matching hashes of every available approved source primitive, new conservatively cleaned transcript and its own SHA-256, immutable evaluation-packet hash, no existing DCE row, host-model provenance, lease, alternate key, caps and readback all pass. Leave the queue and its null historical field unchanged. If a historical clean hash is present, preserve it without requiring equality to newly cleaned text; reject a malformed non-null value. This changes no Q01–Q14 criteria or human outcome.

Positive regressions: valid protected review with a null historical derived clean hash or a different valid historical hash passes this narrow compatibility check. Negative regressions: missing fingerprint, malformed non-null historical hash, source/queue ETag drift, either current primitive hash drift, new cleaned-text or packet hash drift, closed review or existing DCE fail closed.

## Exact human reopening of a conflicting no-live terminal (2026-09-29 candidate)

A confirmed no-live terminal remains final by default. Where an approved transcript for one exact audio-workbook call contains a discernible live buyer response and a seller vehicle-enquiry opening, the human may explicitly reopen that exact call. The prior terminal must remain in the historical ledger, visibly superseded rather than deleted or overwritten. Reopening requires a readback-verified CE Fix row identifying the exact workbook row and approved change, plus an independent exact-call reopening record bound to the source ID and ETag, both currently available approved transcript primitive hashes, the old terminal evidence hash and reason codes, and the open Review Queue identity and ETag. It also requires a newly grounded provisional Q01–Q14 evaluation under this version, Review Required=Yes, no existing DCE row, immutable evaluation-packet hash, model provenance, global lease, alternate key, per-run caps and post-write readback. The human Review Queue stays open.

This precedent authorizes only the exact user-reopened [private date and workbook row removed] after its CE Fix release passes. It does not make a dialer greeting into a buyer response, reopen another excluded call, or permit any rescore of an existing Completed DCE row. A row-level terminal without two-way live Sales evidence or exact approval remains excluded or blocked. Positive regression: the exact approved, live-response case with matching original terminal and all gates may write one new provisional DCE. Negative regressions: another UUID; no buyer response; missing/mismatched terminal, source or queue ETag, primitive or packet hash; absent CE Fix approval/release; existing DCE; resolved review; lease, alternate-key, cap or model failure all stop the write.

## Host-attested low-effort provenance for the sole evaluator (2026-09-29 candidate)

For this exact approved Codex evaluator chat only, the host may report `gpt-6-sol` at `low` reasoning effort. The evaluator must read the actual host thread/session identity, SQLite model and effort, and fresh rollout turn context; the two host effort values must agree. `medium` remains accepted as before. This is not permission to assert a different effort, accept a different model or chat, or bypass transcript, lease, alternate-key, run-cap, review, or readback gates. Positive regressions: actual same-thread host `low` and prior `medium`. Negative regressions: another thread at `low`, mismatched host effort, other models or efforts, missing identity and stale turn context.

## Absent secondary primitive in protected review (2026-09-29 candidate)

A live Sales call may have one usable approved primary transcript while the approved secondary transcript field is genuinely empty. For an unresolved protected Review Queue item, a historical null secondary hash may match only the SHA-256 of that currently empty normalized secondary field; do not fabricate a second transcript or require its text. The present primary transcript hash must match the queue's primary hash exactly. The current source and queue identities and ETags, review fingerprint and open status, hash of every available primitive, independently hashed cleaned transcript and evaluation packet, no existing DCE, model provenance, global lease, alternate key, five-write caps and post-write readback remain mandatory. Preserve the queue and its historical null value unchanged.

Positive regression: a queue with matching non-empty primary hash, null historical secondary hash, and currently empty secondary primitive can receive a grounded provisional Q01–Q14 score from the primary transcript. Negative regressions: a non-empty secondary that appeared or changed, mismatched primary hash, malformed queue hash, closed queue, changed source/queue ETag or fingerprint, missing new evaluation hash, existing DCE, or failed lease, model, cap or alternate-key gate must stop the write. This clarification changes no question criterion or human outcome.

## Windows host session-path attestation (2026-09-30 candidate)

The host-model provenance check may receive an extended-length Windows drive-letter path (`\\?\C:\...`) for the active Codex session from host SQLite. Normalize only that local drive-letter prefix before path validation. The resolved path and its real filesystem target must both be strictly inside the approved host `.codex\sessions` directory. Reject device and UNC paths, relative paths, root/sibling paths, traversal, and symlink escapes. Continue requiring matching active thread/session identity, host model and effort, and fresh turn context before any write. This is an operational path-compatibility repair, not a Q01–Q14 scoring change and not permission to bypass the global lease, source or review ETags, primitive hashes, alternate key, write caps, or post-write readback.

Positive regressions: an ordinary local session path and the same path with the local extended-length drive prefix both attest. Negative regressions: UNC or device prefix, relative or outside path, traversal or symlink escape, wrong model/effort/thread, and stale turn context all fail closed.

## Host-attested `gpt-6.1-sol` producer provenance (2026-09-30 candidate)

Permit a new DCE evaluation, associated review item, or evidence-backed terminal disposition from `gpt-6.1-sol` only when the active host thread row and a fresh recorded turn context both attest to exactly `gpt-6.1-sol` at `medium` effort, and the producer label in the proposed write matches that attestation. The prior `gpt-6-sol` label remains readable for historical records; this release does not silently relabel them. The sole-chat `low` exception remains limited to `gpt-6-sol`. A mismatch, absent or stale context, or unapproved effort stops before any write. Q01–Q14 evidence and applicability criteria are unchanged. Source and review ETags, transcript primitive hashes, global lease, alternate key, five-new-evaluation/five-new-review caps, and post-write readback remain mandatory.

Positive regressions: a same-thread, fresh host row and turn context both reporting `gpt-6.1-sol` at `medium`, with an equal producer label, pass provenance; historical `gpt-6-sol` completed records remain readable. Negative regressions: mixed `gpt-6-sol`/`gpt-6.1-sol` host evidence, a forged or differing producer label, `low` effort on `gpt-6.1-sol`, another model, stale or missing turn context, or any failed ordinary writer gate stops the write.

## Sole-chat Light effort compatibility (2026-09-30)

The user explicitly approved continuing this sole evaluator chat on Light. For this exact approved chat only, permit `gpt-6.1-sol` at `low` reasoning effort when the host thread row and fresh recorded turn context both attest to that same model and effort. This narrow clause supersedes only the v9 prohibition on low effort for that model in this chat. Medium remains permitted; the existing gpt-6-sol sole-chat low exception remains historical. Producer provenance must equal the actual attested model. Never relabel previously prepared evaluations: review them again under the actual current model and effort before writing. Other chats at low effort, mismatched host/turn values, unapproved models or efforts, absent or stale context remain fail-closed.

Positive regressions: the approved sole chat with matching gpt-6.1-sol/low host and fresh turn; matching gpt-6.1-sol/medium; retained historical gpt-6-sol/low sole-chat attestation. Negative regressions: low effort in another chat; low/medium mismatch in either direction; differing models; high effort; stale context; missing turn identities; wrong session paths. Q01-Q14, evidence and N/A criteria, source/review ETags, all primitive hashes, global lease, alternate key, Completed protection, five-new-score/five-new-review caps, and post-write readback are unchanged. The 30-write-cap candidate remains unreleased.


## Approved October sole-agent invocation scope (2026-10-04)

The user approved October scoring. A separately released October invocation must restrict source-created dates to October 1–31, 2026 Africa/Johannesburg (UTC September 30 22:00 inclusive to October 31 22:00 exclusive). Preserve September checkpoints and invocation safeguards. Connect first, then other source call types in arrival order. No workbook imports, existing Completed rescores, authoritative terminal overwrites or human-review closures. This operational precedent changes no Q01–Q14 criteria or model/effort requirements. Every approved primitive hash, source/queue ETag, fresh live prewrite check, global lease, alternate key, cap of five new evaluations/five review items per run and independent Completed postwrite readback remain mandatory. The 30-write-cap candidate remains unreleased. Positive regression: October boundary source passes only the October date gate; negative regressions: September, November, invalid date, model/effort mismatch, protected decision, evidence drift, duplicate DCE and cap/lease failures remain blocked. Candidate must pass the existing release checks before promotion; any failed release requires renewed explicit approval after repair and a passing gate.

## Mixed-language and vernacular review-only routing (2026-10-15)

When a readable transcript contains material code-switching or South African vernacular speech, including Afrikaans, isiZulu, isiXhosa, Sesotho, Setswana or another vernacular, the automatic evaluator must not create a Q01–Q14 score or a completed DCE evaluation. It must preserve the source record and create or retain a guarded human-review item with `MIXED_LANGUAGE_TRANSCRIPT_UNCERTAINTY`. This is review-only routing, not an exclusion: language use alone must never cause a call to be discarded, penalized, translated, reconstructed, or scored by inference. A human reviewer may later resolve the call through the controlled review workflow using approved transcript evidence.

Positive regression: a readable, otherwise scoreable mixed-language call creates or retains exactly one guarded review item and creates no completed DCE score. Negative regressions: a monolingual scoreable call remains eligible for ordinary guarded scoring; a mixed-language call is not excluded or source-mutated; and completed DCE rows or existing human decisions are not overwritten.
