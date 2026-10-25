> Public reference copy, exported 25 October 2026. Private case references have been replaced with explicit placeholders.
> Source status: **Accepted historical source**. This is a documentation derivative, not the byte-identical production artifact.
> Read the repository README and manifest for policy precedence, original/published hashes and the evaluation pause.

# Codex Sales Call Evaluation Rule Bank — Q01 to Q14

**Document purpose:** Authoritative question-level validation source for Sales call evaluations performed in the Call Evaluation Codex chat.

**Rule-bank version:** `sales-q01-q14-2026-09-12-spelling-speaker-separation-no-rewrite`
**Source registry:** `question-rules-vNext-2026-09-03`  
**Permitted use:** Call evaluation, independent validation, comparison and adjudication in Codex chat.  
**Production status:** Active DCE runtime rule-bank version. This version preserves the Q01-Q14 scoring meanings and requires `cleanedTranscriptText` to correct safe misspellings and separate Sales Executive/customer turns where reliable, while never rewriting wording.

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
12. Give a short evidence excerpt or a clear applicability explanation for every `Yes`, `No` and `N/A` answer.
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
- **Yes:** The customer's budget, affordability, monthly target, deposit, maximum spend, price range, repayment expectation or equivalent capacity is established.
- **No:** The affordability stage is reached, but the customer's capacity or target is not established.
- **N/A:** The call ends before the affordability stage; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** A vehicle price or advertised instalment alone is not automatically the customer's budget. Evidence must connect an amount or range to the customer's capacity or target.
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
- **Yes:** The SE explicitly promotes the dealership's F&I department, business manager, finance department, dealership finance capability or a customer benefit of that support. Examples include assistance with approval, shopping for competitive/best bank rates, structuring the finance application or relevant VAPS/protection products.
- **No:** Finance is relevant and a fair F&I opportunity is reached, but the dealership's in-house finance capability or benefit is not marketed.
- **N/A:** A genuine cash-only path makes F&I irrelevant; the call ends before finance/F&I becomes relevant; wrong-person/no-enquiry context; or no live interaction.
- **Evidence:** The wording must communicate dealership-owned F&I or finance support. A generic mention of finance, a bank, MFC, an instalment, an application or a payment plan alone is insufficient.
- **Early-end rule:** If the customer defers before finance relevance is reached, use `N/A`.

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
