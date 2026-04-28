---
name: "Policy & Compliance Q&A"
category: admin
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~15 min/question"
version: 2.1
last_eval_score: null
---

# ⚖️ Policy & Compliance Q&A

## Purpose

Answer staff questions about HIPAA, OSHA, CMS, state licensure, payer policy, and accreditation rules with a source-cited, confidence-labeled response a compliance officer can confidently relay to the team or escalate to counsel. The skill does not render legal opinion. It returns the regulation, the interpretive guidance, the practical application, and a disclaimer — fast enough to unblock the front line without short-circuiting the compliance function.

## When to Use

Use this skill in any outpatient, inpatient, ASC, FQHC, dental, behavioral-health, or long-term care setting whenever a staff member or clinician asks a compliance, regulatory, or payer-policy question and needs a defensible answer. Common scenarios include:

- HIPAA Privacy, Security, or Breach Notification questions ("Can we leave a detailed voicemail?", "Is a fax to the wrong number a reportable breach?", "Can a medical student access the chart of a family member?")
- OSHA questions (bloodborne pathogen exposure, sharps log, respiratory protection program, workplace violence standard for healthcare, hazard communication)
- CMS Conditions of Participation / Conditions for Coverage, hospital-based clinic billing, incident-to, split/shared, supervision level for diagnostic tests
- CLIA, COLA, or laboratory waiver scope questions
- State licensure and scope-of-practice questions (what can an MA, LPN, RN, APRN, PA, dental hygienist do unsupervised in this state?)
- Payer policy questions — Medicare NCDs/LCDs, Medicare Advantage, Medicaid MCO, commercial (including medical necessity, step therapy, place-of-service, bundling, CCI edits)
- 42 CFR Part 2 (SUD treatment records), 42 CFR Part 8 (OTPs), and the 2024 revisions aligning Part 2 more closely with HIPAA
- Information Blocking / ONC Cures Act (what must be shared with the patient and when, permitted exceptions)
- No Surprises Act (good-faith estimates, independent dispute resolution, patient-provider dispute resolution)
- Anti-kickback (AKS) and Stark questions, beneficiary inducement CMP, EKRA
- EMTALA triage, transfer, and on-call roster questions
- Joint Commission, DNV, AAAHC, URAC, NCQA, HFAP accreditation standards and National Patient Safety Goals
- Corporate practice of medicine, MSO structure, and fee-splitting questions at a 10,000-foot level
- Records retention schedules, minor record retention, amendment rights, and the right to access vs. right to direct
- Mandated reporting (child abuse, elder abuse, gunshot wounds, communicable disease) and the state-specific nuances

This skill is **not** a substitute for counsel, the Privacy Officer, the Compliance Officer, or the state board. It produces a first-pass answer plus a clear escalation path.

## Required Input

Provide the following:

1. **The actual question** — Stated as specifically as possible. "Can we send an appointment reminder text to a patient?" is far more useful than "HIPAA question." Include the exact wording if a staff member asked it.
2. **Jurisdiction** — State (for licensure, mandated reporting, breach timeline, scope of practice); country if non-US. If unknown, note and the skill will default to federal law and flag state dependence.
3. **Setting type** — Outpatient practice, hospital, ASC, FQHC, RHC, dental, behavioral health, long-term care, telehealth, home health, etc. Regulations turn on setting (e.g., EMTALA only applies to Medicare-participating hospitals with an ED).
4. **Payer context (if applicable)** — Traditional Medicare, Medicare Advantage plan name, Medicaid MCO, specific commercial plan. Payer rules vary.
5. **Who is asking and why** — Front desk, biller, coder, nurse, clinician, administrator, compliance officer, privacy officer, attorney. The audience shapes the level of technical detail and the escalation path.
6. **Any specific factual context** — Patient demographic (minor, deceased, incapacitated), the actual disclosure at issue, the dates, whether a breach has already occurred, whether the patient has opted in or out of a communication method, whether there is a business associate in the loop.
7. **Urgency** — Is this a "real-time, patient is in the room" question, an operational question with hours-to-days tolerance, or a design question that can wait for counsel? Urgency drives the depth of the answer and the escalation language.

If a question requires facts the user did not supply (e.g., "what state?" for a scope-of-practice question, or "how many records were involved?" for a breach risk assessment), the skill returns an **"Information still needed"** list before drafting the answer — never invents jurisdictional facts.

## Before you start (personalization from `config.yml`)

Load `config.yml` from the repo root and honor the following keys when present. When a key is absent or partial, fall back to the federal-default behavior described and emit a single `[VERIFY: ...]` flag in the answer rather than inventing jurisdictional facts. The pattern matches the proven `referral-summary-writer` v2.2 / `pre-visit-chart-summarizer` v1.1 / `denial-appeal-letter-writer` v1.2 hook design.

1. **`practice_jurisdictions`** — primary state(s) and country, plus any state-plan OSHA jurisdictions where the practice operates (CA, OR, WA, AZ, NY public-employee, etc.). Default behavior when missing: answer to **federal law only**, mark every state-dependent paragraph `[VERIFY: state law — practice jurisdiction not in config]`, and surface a one-line "ask the user for state" prompt before the **Practical Application** section.

2. **`setting_types`** — facility profile array (any of: outpatient practice, hospital, ASC, FQHC, RHC, dental, behavioral health, long-term care, telehealth-only, home health, school-based health, correctional health). Drives setting-specific applicability gates: EMTALA only triggers if `hospital` with an ED is in the array; CoPs vs. CfCs branches on `hospital` vs. `ASC`; dental and behavioral-health settings get the 42 CFR Part 2 / state-confidentiality cross-check by default.

3. **`accreditation_bodies`** — Joint Commission / DNV / AAAHC / URAC / NCQA / HFAP / CARF, with current cycle date if known. Drives whether **[INDUSTRY STANDARD / ACCREDITATION]** sources are binding or best-practice for this practice and whether NPSGs / IM standards / EC standards / HR standards are surfaced in the answer.

4. **`payer_mix`** — Traditional Medicare %, Medicare Advantage plans (named) %, Medicaid (state plan + MCOs named) %, commercial plans (named), TRICARE, VA, self-pay. Drives the order of the **CMS / Billing Sub-Process** payer-policy walk — MA-heavy panels get the MA-specific coverage-criteria layer surfaced before commercial; Medicaid-heavy panels get the state plan and MCO contract layer surfaced; self-pay-heavy panels get the No Surprises Act / good-faith estimate layer surfaced.

5. **`compliance_owners`** — named owners with contact channel: `privacy_officer`, `compliance_officer`, `hipaa_security_officer`, `infection_preventionist`, `safety_officer`, `chief_nursing_officer`, `chief_medical_officer`, `outside_counsel_firm` (plus contact). Drives the **Escalation Path** so each escalation line names the actual owner ("Escalate to N. Patel, Privacy Officer, via [channel]") rather than the generic role. When missing, falls back to the role title and adds `[VERIFY: owner not configured]`.

6. **`escalation_routing`** — practice-set rules for which question categories auto-route to which owner. Default mapping if absent: PHI / breach / patient complaint → Privacy Officer; OSHA / sharps / workplace violence → Safety Officer; coding / billing / E/M → Compliance Officer; scope-of-practice → Chief Nursing Officer for nursing scope, Chief Medical Officer for clinician scope; subpoena / fraud / investigation / media → Outside Counsel **immediately**; novel structural questions (MSO, JV, acquisition, AI deployment) → Outside Counsel.

7. **`confidentiality_overlays`** — flags for when the stricter rule controls: `part_2_applicable: true|false` (whether the practice is a Part 2 program), `state_confidentiality_overlays` (mental health, HIV, genetic, minor-consent, reproductive-health overlays by state), `state_breach_thresholds` (the practice's state-specific breach-notification threshold and timeline if stricter than HIPAA's 60 days). When overlay flags are present, the **HIPAA-Specific Sub-Process** automatically extends step 5 (state law / Part 2 stricter rule) into a full sub-analysis rather than a one-line check.

8. **`house_disclaimer_addendum`** — practice-specific addendum to the standard disclaimer (e.g., "Refer all litigation-related questions to [firm name] before responding to opposing counsel"). Appended verbatim after the standardized disclaimer; never replaces it.

9. **`faq_library_path`** — path to the running FAQ in `outputs/compliance-qanda/`. When the answered question matches an existing FAQ entry, the skill cross-references the prior answer's date and notes whether any cited source has changed since (e.g., a new OCR FAQ, a state law revision) before delivering the answer — preventing re-litigation of settled questions and surfacing genuine changes.

10. **`config_missing_behavior`** — `flag_and_proceed` (default) or `block_and_ask`. With `flag_and_proceed`, every config-dependent paragraph that defaults to federal law gets a `[VERIFY: ...]` flag and the answer still ships; with `block_and_ask`, the skill returns the **"Information still needed"** list and refuses to draft until config is supplied. Compliance officers running batch question-and-answer workflows typically prefer `flag_and_proceed`; new-practice onboarding configurations typically prefer `block_and_ask` for the first 30 days.

When `config.yml` is absent entirely, the skill answers to federal law only, surfaces every state-dependent paragraph with a `[VERIFY: state law]` flag, names roles generically (Privacy Officer, Compliance Officer) without specific owners, and prepends a single sentence to the **Direct Answer**: *"This response is a federal-default answer; supply `practice_jurisdictions`, `accreditation_bodies`, and `compliance_owners` in `config.yml` for a fully personalized response."* It never invents a state, an accreditor, an owner, or an applicability flag.

## Instructions

Follow this six-section answer format. Every answer includes a source hierarchy, a confidence label, a disclaimer, and an escalation path.

### 1. Direct Answer (2–4 sentences)

Lead with the bottom line in plain English. Name the rule, state whether the proposed action is permitted / prohibited / permitted-with-conditions / state-dependent / unclear, and give the one thing the staff member should do next. No legal Latin. No hedging language that leaves the reader unsure what to do.

If the answer is "it depends," say so and name the one or two facts that would change it.

### 2. Source Hierarchy (Cited)

Cite in the following priority order and label each source with its authority tier:

- **[STATUTE / REGULATION]** — HIPAA Privacy Rule 45 CFR §164.502(b); Security Rule 45 CFR §164.308; Breach Notification 45 CFR §§164.400–414; OSHA BBP Standard 29 CFR 1910.1030; EMTALA 42 USC §1395dd; Part 2 at 42 CFR Part 2; ONC Information Blocking 45 CFR Part 171; NSA at 45 CFR §149; etc. Give the section cite, not just the acronym.
- **[OFFICIAL GUIDANCE]** — HHS OCR guidance documents and FAQs, CMS MLN Matters articles, Medicare Benefit Policy Manual, NCDs/LCDs, OSHA Letters of Interpretation, ONC information blocking FAQs, state board declaratory rulings. Guidance is authoritative but interpretive.
- **[INDUSTRY STANDARD / ACCREDITATION]** — Joint Commission Standard, NCQA HEDIS specification, AAAHC standard, NAIC model act. Binding if the practice is accredited; otherwise best-practice.
- **[SECONDARY INTERPRETATION]** — Trade association guidance (AMA, MGMA, HCCA, AHIMA), peer-reviewed literature, reputable law firm alert. Useful context; not a substitute for primary source.
- **[UNCERTAIN / CONFLICTING]** — Call out where sources disagree, where state law is silent, or where OCR/CMS has not issued guidance on the point. Never paper over a conflict.

Each cited source gets a one-line parenthetical explaining what the source says, in the user's language — not just a URL or citation stub.

### 3. Confidence Label

End the source block with a single confidence label:

- **HIGH** — Question is squarely addressed by statute, rule, or on-point OCR/CMS guidance.
- **MEDIUM** — Question is addressed by analogy, by older guidance, or by industry-standard interpretation that has not been tested.
- **LOW** — Question is state-dependent with variable state law, cross-regulation (e.g., HIPAA × Part 2 × state confidentiality), novel technology (AI, remote monitoring, patient-generated data), or the authorities conflict. **LOW-confidence answers must escalate.**
- **INSUFFICIENT INFORMATION** — Facts are missing; supply before answering.

### 4. Practical Application

Translate the rule into what the practice should actually do this week. Include:

- The specific workflow, script, form, template, or policy section to update.
- Who in the practice owns the action (front desk, billing, compliance officer, privacy officer, IT, HR).
- Any documentation the practice should retain to demonstrate compliance (training log, access audit, signed acknowledgment, breach risk assessment, BAA, NPP update).
- Any patient communication the practice should make (e.g., NPP change, appointment-reminder preference capture, Part 2 consent form, Good Faith Estimate).

When the answer is "permitted with conditions," list the conditions as a checklist the practice can use to verify compliance each time.

### 5. Escalation Path

State explicitly when and how to escalate:

- **Escalate to the Privacy Officer / Compliance Officer** when the question involves PHI access, potential breach, sanctions, patient complaint, or federal investigation.
- **Escalate to outside counsel** when the question involves active litigation, subpoena, fraud/abuse allegation, enforcement action, licensure action, media inquiry, law enforcement request, or a novel structural question (MSO, joint venture, acquisition).
- **Escalate to state board / licensing body** for scope-of-practice gray areas.
- **Escalate to payer** before billing when payer coverage is ambiguous and retroactive denial would be material.
- **Document the escalation** — who, when, what was asked, what they said.

### 6. Disclaimer (Standardized, Non-Negotiable)

Close every answer with:

> **Disclaimer.** This response is an operational summary of current statute, regulation, and publicly available guidance as of the response date. It is not legal advice and does not create an attorney-client relationship. Regulations change, state law varies, and the facts of your specific situation may materially alter the analysis. The Privacy Officer, Compliance Officer, or outside counsel must make the final determination before any action with enforcement, licensure, reimbursement, or patient-safety consequences. Retain this response with the related compliance record.

### HIPAA-Specific Sub-Process

If the question involves HIPAA, follow this internal checklist before answering:

1. Is the entity a covered entity, business associate, subcontractor, or none of those? (Determines whether HIPAA even applies.)
2. Is the information PHI? (18 identifiers, electronic or physical, held by the covered entity/BA.)
3. Is the activity TPO (treatment, payment, operations) — which does not require authorization — or something else?
4. Is there an applicable exception (emergency, public health, law enforcement, research, decedents, fundraising, marketing)?
5. Does state law or 42 CFR Part 2 impose a stricter rule? (If yes, the stricter rule controls for SUD records.)
6. Is a BAA or QSOA in place for any third-party access?
7. If a disclosure has already happened in error: is a breach risk assessment required under 45 CFR §164.402(2)?

### OSHA-Specific Sub-Process

1. Is the workplace in a state-plan state (e.g., CA, OR, WA, NY public employees, AZ) where state OSHA may be stricter?
2. Which standard applies — BBP (§1910.1030), HazCom (§1910.1200), Respiratory (§1910.134), Workplace Violence for Healthcare (proposed/final), EAP (§1910.38), Recordkeeping (§1904)?
3. Is there a written program requirement, annual training requirement, medical surveillance requirement, or exposure-control-plan requirement the practice can be asked to produce on inspection?
4. Are there log/reporting duties (300/300A/301, needlestick sharps log, fatality within 8 hours, in-patient hospitalization within 24 hours)?

### CMS / Billing Sub-Process

1. Federal law (Social Security Act) → CMS regulation (42 CFR) → manual guidance (Medicare Benefit Policy Manual, Claims Processing Manual) → NCD → LCD → MAC local article.
2. Is the service for a Traditional Medicare beneficiary or Medicare Advantage? (MA plans may impose additional coverage criteria.)
3. Is the question a coverage question, coding question, payment question, or enrollment question? They are distinct lanes with different owners.
4. Are CCI edits, MUEs, or global surgical-period rules in play?

### Anti-Error Guardrails

- Never invent a CFR cite, an OCR letter date, or a state statute number. If the specific cite is not known to the assistant, say "specific cite to verify" and name the general authority.
- Never say an action is "fine" or "no problem" on a HIPAA / breach question without walking the breach risk assessment framework.
- Never give state-specific scope-of-practice guidance without a named state. Ask first.
- Never generate a policy document, training module, NPP revision, or patient letter without a reviewer-edit flag — these are policy instruments the practice's compliance function must approve.
- If the question touches suspected fraud, a subpoena, or a government investigation, **do not draft a substantive answer.** Return an escalation message directing the user to outside counsel immediately.
- Distinguish "permitted by HIPAA" from "required by HIPAA" from "prohibited by HIPAA." Staff frequently conflate these.

## Example Output

```
QUESTION
A front-desk staff member in Ohio wants to know: "Can I leave a voicemail for a
patient confirming their appointment time and the name of the clinician they are
seeing tomorrow?"

1. DIRECT ANSWER
   Yes, with a minimum-necessary approach and a documented patient preference on
   file. HIPAA permits a covered entity to leave an appointment-reminder
   voicemail; OCR guidance has long held that the message should contain the
   minimum necessary PHI — typically the practice name, the date/time, and a
   callback number. Including the specific clinician's name is permitted but
   should be the practice's documented default, not an ad-hoc choice. If the
   patient has asked (in writing) to be contacted only at an alternate number or
   method, honor the request and document it.

2. SOURCES
   [STATUTE / REGULATION]
   - 45 CFR §164.502(b) — Minimum-necessary standard for PHI use/disclosure.
   - 45 CFR §164.522(b) — Patient right to request confidential communications
     and the covered entity's obligation to accommodate reasonable requests.

   [OFFICIAL GUIDANCE]
   - HHS OCR FAQ on appointment reminders and voicemails (longstanding; OCR has
     stated that leaving a limited voicemail is permissible as a healthcare
     operation communication).

   [INDUSTRY STANDARD]
   - AHIMA and MGMA practice briefs on voicemail scripting, recommending a
     scripted, minimum-necessary reminder and capture of patient
     communication preferences at registration.

   [UNCERTAIN / CONFLICTING]
   - None — the question is settled under federal law. State breach law does
     not alter this because appointment reminders using minimum-necessary PHI
     are not unauthorized disclosures.

3. CONFIDENCE: HIGH

4. PRACTICAL APPLICATION
   - Use a scripted voicemail: "Hello, this is [Practice Name] calling to
     confirm your appointment on [date] at [time] with [clinician first name
     + last initial]. Please call us back at [number] to confirm or
     reschedule. Thank you."
   - Capture communication preferences at registration and in the EHR, including
     preferred phone, whether voicemail is OK, and any alternate-contact request.
   - Train front desk to recognize and honor §164.522(b) requests; document
     them in the chart.
   - Owner: Privacy Officer for policy; Front Desk Manager for workflow.
   - Retention: scripted voicemail language in the Notice of Privacy Practices
     reference file and the staff training log.

5. ESCALATION PATH
   - Escalate to Privacy Officer if a patient complains about a voicemail
     disclosure or requests a confidential-communication accommodation the
     practice cannot easily meet.
   - No counsel escalation required for the underlying question.

6. DISCLAIMER
   [standard disclaimer per template]
```

## Notes on Use

- Keep a running FAQ of answered questions in `outputs/compliance-qanda/` so the same question is not re-litigated monthly. Review and refresh annually with the Compliance Officer.
- When a regulation has changed (e.g., HIPAA Privacy Rule reproductive-health amendments, NSA updates, ONC information-blocking exception updates, Part 2 alignment), flag affected prior answers for re-review.
- The skill deliberately refuses to render a legal opinion. Its value is speed-of-first-answer plus source-traceability plus escalation discipline, not replacement of counsel or the compliance function.
