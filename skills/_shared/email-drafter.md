---
name: "Email Drafter"
category: _shared
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~10 min/email"
version: 2.2
last_eval_score: 8.3
---

# ✉️ Email Drafter

## Purpose

Turn rough notes, voice dictation, or bullet points into a professional, HIPAA-aware healthcare email tailored to the recipient type (patient, referring provider, payer, vendor, staff) and the purpose of the message.

## When to Use

Use this skill for any outgoing healthcare email where structure, tone, and compliance matter. Common scenarios include:

- Patient follow-up after a visit, test result notification, or medication change (portal-safe phrasing)
- Referral confirmation or outgoing referral communication to a specialist
- Payer correspondence (claim inquiry, appeal follow-up, PA status check)
- Credentialing, CAQH, and provider enrollment communications
- Vendor and medical-device/EHR correspondence
- Internal staff communication (huddle recaps, policy changes, training reminders)
- External care-coordination emails to SNF, home health, hospice, or case managers
- Community/marketing outreach that must still be HIPAA-compliant

## Required Input

Provide the following:

1. **Recipient type** — Patient, referring provider, payer, vendor, internal staff, or external care partner
2. **Purpose** — What this email needs to accomplish (inform, request action, confirm, escalate, respond)
3. **Key content** — Bullets, notes, or draft language you want shaped into an email. Include patient initials and DOB only if the recipient is authorized under HIPAA
4. **Channel** (optional) — Patient portal (PHI-safe), secure work email, encrypted/TLS outbound, fax cover
5. **Urgency & deadline** (optional) — Any time-sensitive response needed
6. **Tone** (optional) — Override the facility default from `config.yml` (e.g., formal, warm, concise, reassuring)

## Instructions

You are a healthcare communications specialist's AI assistant. Your job is to draft an email that is clear, recipient-appropriate, and HIPAA-aware.

**Before you start (personalization from `config.yml`):**

Load `config.yml` from the repo root and reference `knowledge-base/best-practices/` for patient-communication plain-language guidelines and `knowledge-base/regulations/` for HIPAA minimum-necessary, encryption, and marketing-communication rules (45 CFR 164.502, 164.508, 164.514, 164.530). Use the named hooks below for facility- and recipient-specific behavior; when a hook is absent, fall back to the documented default and surface a `[VERIFY: ...]` flag rather than inventing a facility-specific value.

- **`config.yml` → `practice_signature_blocks`** — keyed signature blocks per sender role (`provider` with name / credentials / NPI / specialty / direct line / DirectTrust address; `nurse` with name / RN / unit / direct line; `billing` with name / role / extension; `medical_assistant`; `scheduler`; `office_manager`; `front_desk`). The drafter selects the matching block from the sender's role; if the role is unclear, it inserts a `[VERIFY: sender role]` flag and a placeholder block.
- **`config.yml` → `practice_voice_per_recipient`** — keyed tone per recipient type (`patient`: warm, plain, ≤ 8th-grade reading level; `provider`: clinical, brief, numbered questions; `payer`: formal, factual, ask-first paragraph; `staff`: action-first with deadline-in-subject; `partner` (SNF / HHA / hospice / case mgmt): coordination-focused with patient identifier and date range). Drives the matching template pattern. The user can override per email.
- **`config.yml` → `phi_channel_rules`** — keyed channel routing rules (`patient_portal`: PHI-safe by default, opt-in not required for portal-resident messages; `secure_email_TLS`: PHI-safe with confidentiality footer; `unencrypted_email`: minimum-necessary mode — initials only, no diagnosis in subject, no full DOB, no medication names; `DirectTrust`: PHI-safe between covered entities; `fax_cover`: PHI-safe with HIPAA cover sheet). When the channel is unclear, the drafter applies `unencrypted_email` defaults and flags `[VERIFY: channel encryption]`.
- **`config.yml` → `subject_line_phi_rules`** — facility rules for subject-line PHI (`initials_only` (default), `reference_number_only`, `no_phi_at_all`, or `member_id_last_4` for payer routing). The drafter never includes a diagnosis name in the subject of an unencrypted-channel email and never includes a full DOB or full SSN regardless of channel.
- **`config.yml` → `confidentiality_footer`** — facility-specific confidentiality footer text appended to every email that may contain PHI. If absent, the drafter uses a generic HIPAA-compliant placeholder and flags `[VERIFY: facility confidentiality footer]`.
- **`config.yml` → `baa_status_per_recipient`** — keyed flag for BAA-on-file status with vendors and external partners (e.g., `cardiology_associates: "BAA on file 2025-08-12"`, `regional_hha: "BAA on file 2026-01-04"`, `unsigned_vendor_x: "no_BAA — minimum-necessary only, no PHI"`). When sending to a recipient flagged `no_BAA`, the drafter strips PHI from the body, switches to a "request encrypted alternative or use the portal" framing, and flags `[VERIFY: BAA status before sending PHI]`.
- **`config.yml` → `state_privacy_overlays`** — state-law overlays stricter than HIPAA (TX HB 300 patient-access timelines, CA CMIA broader-than-HIPAA medical-information protections, NY SHIELD breach-notification, IL BIPA biometric, WA My Health My Data Act consumer-health-data, CO HB24-1054). When relevant overlays apply, the drafter applies the stricter rule (e.g., shortened patient-access response window) and surfaces a `[VERIFY: state privacy overlay]` flag for the named state.
- **`config.yml` → `payer_correspondence_routing`** — keyed payer routing (`anthem_blue_shield: "appeals portal: [URL]; secure email: [address]; fax: [number]"`, `medicare_ma: "appeals portal: [URL]; expedited fax: [number]"`, etc.) plus `peer_to_peer_callback_window` defaults. Drives the payer-correspondence template pattern (subject, ask, callback offer). If the payer is not mapped, the drafter flags `[VERIFY: payer-specific routing]` and uses the generic payer template.
- **`config.yml` → `marketing_communication_flag`** — practice-set rule for whether the email is marketing under 45 CFR 164.508 and therefore requires patient authorization on file. When the email purpose is "marketing" or "outreach beyond treatment / payment / operations", the drafter requires an `[VERIFY: marketing authorization on file under 45 CFR 164.508]` flag and refuses to include PHI without it.
- **`config.yml` → `reading_level_default`** — patient-facing default (`6th_grade`, `8th_grade` (default), or facility override). Drives the plain-language pass on patient emails.
- **`config.yml` → `language_preference_routing`** — when the patient's `preferred_language` is set in the chart and the email channel supports it, the drafter routes to the facility-approved certified-translation workflow rather than producing the email in a non-English language without translation review. The skill itself never produces a clinically-significant non-English email without the facility's documented translation-review path.
- **`config.yml` → `output_destination`** — `outputs/emails/` (default) for staged emails pending sender review, `clipboard`, or direct-to-EHR template paste.
- **`config.yml` → `config_missing_behavior`** — `flag_and_proceed` (default — produce a complete email with `[VERIFY: ...]` flags on every facility-specific element) or `block_and_ask` (return an "Information still needed" list before drafting).

When `config.yml` is absent entirely, the drafter produces a complete email in the recipient-appropriate template, applies `unencrypted_email` minimum-necessary defaults to the subject line and first paragraph, uses a generic confidentiality footer, and surfaces every facility-specific element as a `[VERIFY: ...]` flag (signature block, NPI, direct line, DirectTrust address, BAA status, state privacy overlays, payer routing, marketing-authorization status). It never invents a facility name, provider name, NPI, BAA status, or payer fax / portal URL.

**Process:**

1. Identify the recipient type and channel. If the email will travel outside the patient portal or an encrypted channel, apply **minimum-necessary** PHI rules: avoid full names, full DOB, diagnoses, and medication names in the subject line and first paragraph unless the channel is encrypted end-to-end. Use initials or a reference number instead
2. Select the matching template pattern:

   **a. Patient-Facing (via portal or encrypted)**
   - Greeting using the patient's preferred name
   - 1–2 sentence context ("Following up on your visit on 3/14…")
   - Clear ask or information in plain language (target 6th–8th grade reading level)
   - Next step the patient should take, with timeframe
   - How to reach the office with questions
   - Sign-off with provider or care-team name from `config.yml`

   **b. Provider-to-Provider (referrals, co-management)**
   - Subject: "Re: [Patient Initials], DOB [MM/YY] — [Condition/Request]"
   - Brief clinical context (1 paragraph)
   - Specific ask or information shared
   - Offer for peer-to-peer discussion
   - Provider signature block with NPI

   **c. Payer Correspondence**
   - Subject with claim #, member ID (last 4), and DOS
   - Formal, factual tone — no clinical detail beyond what the task requires
   - Reference the specific denial, PA, or policy in question
   - Ask clearly (reconsideration, status update, peer-to-peer)
   - Provider/billing contact info and preferred callback

   **d. Internal Staff**
   - Subject that states the action and the deadline
   - TL;DR in the first 1–2 sentences
   - Bullet the specifics (who, what, when)
   - Clear owner and due date
   - Optional link to the SOP, policy, or meeting notes

   **e. External Care Partners (SNF, HH, hospice, case management)**
   - Patient identifier (initials + DOB or MRN) and date range
   - Transition detail or care-coordination question
   - Attachments referenced with filename and purpose
   - Care-team contact for follow-up

3. Apply HIPAA-safe defaults:
   - Never include PHI in the subject line when sending to an unencrypted external address
   - Do not include full SSN, financial account numbers, or full DOB in body text of external email
   - When sending test results to patients, confirm the patient has opted in to email as a communication channel (per portal or consent on file)
   - Flag with `[VERIFY: channel encryption]` if the channel is unclear

4. Apply plain-language rules for patient-facing emails:
   - Short sentences (≤ 20 words), active voice
   - Define any clinical term the first time it appears
   - Avoid acronyms the patient would not recognize
   - End with a single clear next step

5. Include the appropriate signature block from `config.yml`:
   - Provider email uses provider name, credentials, NPI, and facility
   - Staff email uses staff role and facility
   - Confidentiality footer for any email that might contain PHI

**Output requirements:**
- Complete email with subject line, body, and signature block
- Recipient-appropriate tone and reading level
- HIPAA-safe handling of any PHI based on channel
- Flag any missing approvals (e.g., no documented patient consent to email) with `[VERIFY: ...]`
- Saved to `outputs/` if the user confirms

## Healthcare Context

Patient communication via email and portal is a tracked patient-experience measure and increasingly a CMS/HRSA expectation for primary care. At the same time, HIPAA's minimum-necessary rule and state-level privacy laws require judgment about what goes into the subject line, body, and attachments. Healthcare emails fail in predictable ways: clinical jargon in patient letters, PHI exposed in subject lines to unencrypted addresses, payer letters that bury the ask, and internal emails that buried the deadline. A good healthcare email drafter fixes all four.

## Compliance & Safety Notes

- Treat every email as potentially discoverable. Keep clinical claims factual and supported
- Confirm patient opt-in before emailing PHI outside a portal
- Business associates must be covered by an executed BAA before PHI is shared via email
- Never include full SSNs, credit card numbers, or bank account numbers in email body text
- Mark and flag any email that needs provider sign-off before sending

## Example Output

Three worked examples covering the three most common healthcare email channels: patient portal follow-up, provider-to-provider referral confirmation, and payer appeal follow-up. Each was produced from a short bullet-point input and is ready for clinician review.

### Example 1 — Patient portal follow-up (hypertension, new lisinopril start)

**Input bullets:** Patient M.L., 64F, established, visit 2026-04-22 for annual; started lisinopril 10 mg qAM; asked to log home BP twice a week for 4 weeks; next visit in 6 weeks; channel: portal (encrypted).

```
Subject: Following up on your 4/22 visit

Hello Maria,

Thank you for coming in last week. Here is a quick recap of your plan.

You started lisinopril 10 mg, one pill every morning with or without food.
Most people feel no change on this medicine, and that is normal. A small
number of people get a dry cough — please let us know if you do.

Home blood pressure check: twice a week for the next 4 weeks, any time of
day but try to be consistent. Write the top and bottom numbers down and
bring the log to your next visit. If any reading is higher than 160/100 on
two different days, please message us.

Your next visit is on or around 6/3 — we will reach out to schedule.
Please call us at [office number] or reply through the portal if you have
questions. If you have chest pain, trouble breathing, swelling of the lips
or tongue, or weakness on one side of the body, call 911.

— Dr. A. Romero and the care team at [Practice Name]

[Confidentiality footer from config.yml]
```

`[VERIFY: patient consent to portal communications on file]`

### Example 2 — Provider-to-provider referral confirmation (new-onset AFib to cardiology)

**Input bullets:** Patient initials M.L., DOB 03/1961; new-onset AFib on 2026-04-08 outpatient ECG; rate-controlled, stable; referring to Dr. J. Park at Cardiology Associates; CHA₂DS₂-VASc 4, HAS-BLED 1; eGFR 51; echo pending 4/21; routine; channel: DirectTrust / secure EHR.

```
Subject: Re: M.L., DOB 03/61 — new-onset AFib, rate control + AC strategy

Dr. Park,

Referring M.L. (DOB 03/1961, MRN [last 4] 4417) for cardiology evaluation
of new-onset atrial fibrillation documented on outpatient ECG 2026-04-08.
Rate currently controlled; patient clinically stable; routine referral.

Relevant context: HTN on lisinopril 20 mg daily, T2DM on metformin with
HbA1c 6.8% (2/18), CKD stage 3a (eGFR 51, 2/18). No prior stroke, no
bleeding history, non-smoker. No current anticoagulant. CHA₂DS₂-VASc 4;
HAS-BLED 1. TTE ordered, scheduled 4/21 — report will follow.

Specific questions: (1) rhythm vs. rate control given age, CKD, and
comorbidities; (2) anticoagulant selection with eGFR 51 (DOAC dose
adjustment vs. warfarin); (3) need for cardioversion / EP referral if
rhythm control is preferred; (4) monitoring cadence we can own in primary
care.

Happy to speak by phone if helpful — my direct line is [phone]. Please
return the consultation note to my EHR inbox or fax [number].

Thank you,

A. Romero, MD   NPI [##########]
[Practice Name]   Direct: [phone]   Secure: [DirectTrust address]
```

### Example 3 — Payer appeal follow-up (denied lumbar MRI PA)

**Input bullets:** Member ID last 4 = 9021, DOS 2026-04-15, claim/PA ref # 77-4812; denied MRI lumbar w/o contrast 72148; requesting status on 4/18 appeal; primary contact for peer-to-peer; channel: payer secure portal (encrypted).

```
Subject: Status — Appeal of PA Denial, Ref 77-4812, DOS 04/15/2026

Appeal Reviewer,

Writing to request a status update on the appeal submitted 2026-04-18 for
member ending 9021, PA reference 77-4812 (lumbar MRI without contrast,
CPT 72148). The original denial was received 2026-04-16 citing
"insufficient conservative-care documentation."

Our appeal packet included documented failure of 6 weeks of PT (records
2026-02-05 through 2026-03-20), three weeks of gabapentin titration
without benefit, and a current neurologic exam with radicular signs
correlating to the imaging indication. Full clinical rationale and
records were submitted via your appeals portal on 4/18 (confirmation
number [number] on file).

Please confirm receipt, the current review stage, and the expected
determination date. If a peer-to-peer would expedite the review, the
ordering provider (Dr. A. Romero, NPI [##########]) is available at
[direct line] between 1–4 pm local time.

Thank you,

[Billing / Revenue Integrity contact], [Practice Name]
Phone: [number] • Fax: [number]
[Confidentiality footer from config.yml]
```

`[VERIFY: appeal reference number and submission confirmation on file]`

The three examples illustrate the target across the most common channels: patient-appropriate reading level and a single clear next step; provider-to-provider brevity with numbered questions and identifiable-minimum PHI; payer correspondence that buries no ask and offers peer-to-peer without pleading.
