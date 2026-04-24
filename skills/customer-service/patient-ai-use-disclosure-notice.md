---
name: "Patient AI-Use Disclosure Notice"
category: customer-service
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~25 min/notice"
version: 1.0
last_eval_score: null
---

# 🪪 Patient AI-Use Disclosure Notice

## Purpose

Draft a plain-language patient-facing disclosure that names *which* AI systems the practice uses in the patient's care, *when* and *how* the patient will encounter them, *what the patient can opt out of*, and *who to contact* with questions or to revoke consent. The output is calibrated to the wave of 2025–2026 state laws that now require this disclosure — Texas TRAIGA (HB 149, effective 1/1/2026), California AB 489 (effective 1/1/2026), Colorado HB26-1139, the Utah AI prior-authorization disclosure law (enacted 3/19/2026, effective 1/1/2027), and the broader 40+ state-AI-in-healthcare bills tracked across 25 states in 2026 — and to the 2026 federal direction set by the HHS AI Strategic Plan, the FDA January 2026 CDS guidance, and the CMS Interoperability and Prior Authorization Final Rule (CMS-0057-F) reasoning-disclosure expectations. The notice is patient-readable (6th–8th grade), bias-aware, and purpose-bound: it discloses what the law requires *and what a reasonable patient would want to know*, without legal hedging that obscures the answer.

## When to Use

Use this skill any time the practice needs a *patient-facing* disclosure of AI use — a written notice, a portal banner, a check-in iPad screen, a consent-form addendum, an after-visit-summary insert, a multilingual handout, or a script for the front desk to read. Common scenarios:

- A practice in **Texas, California, Colorado, Utah, or any state with an effective 2026 AI-in-healthcare disclosure law** is operationalizing patient notification for the first time
- A practice has rolled out an **ambient AI scribe** (Abridge, Suki, Nuance/Microsoft DAX, Ambience, Augmedix, Heidi, etc.) and needs to disclose to the patient that the encounter is being captured and drafted by AI before the visit begins
- A practice has rolled out an **AI patient-portal message triage or draft-reply tool** (Epic AI for Patients, Johns Hopkins OPTIC, MyChart AI replies, healow Genie, Lindy) and the patient is interacting with AI-drafted text without realizing it
- A practice has rolled out an **AI patient-facing chatbot** (PatientGPT, Emmie, K Health, Amazon One Medical AI assistant, hospital-branded AI front door) and needs a *first-screen* disclosure that the patient is talking to AI, not a human, plus a clear handoff path to a human
- A health plan or payer has rolled out **AI-assisted prior authorization or coverage review** and must, under Utah and forthcoming state laws, notify enrollees that AI is involved in the review of their authorization request (this notice is patient-facing, not provider-facing)
- A specialty practice is using **AI for clinical decision support** (radiology AI overreads, derm photo triage, EKG interpretation, sepsis scoring, ophthalmology DR screening) where the FDA's January 2026 CDS guidance allows non-device CDS *only when the clinician can independently review the AI's logic* — patients increasingly want to know the AI exists in the loop
- A practice is updating its **Notice of Privacy Practices** to add an AI section, or refreshing the **patient consent for treatment** to incorporate an AI use disclosure
- A practice is responding to a **patient question** ("Was my chart written by a robot?", "Is this email really from my doctor?", "Did AI deny my prior auth?") and wants a calibrated, written answer that the front desk can hand the patient
- The practice needs a **multilingual** or **low-literacy** version of an existing English AI disclosure
- The practice needs a **patient-facing summary of an internal AI-system change** (added a new tool, retired a tool, switched vendors, restricted use to a service line)

This skill produces patient-facing content. It is **not** legal advice, **not** a substitute for the practice's Privacy Officer or counsel, and **not** a substitute for the AI vendor's own Business Associate Agreement, model card, or HIPAA risk analysis. It produces a first-pass disclosure ready for compliance review.

## Required Input

Provide the following. Items 1–6 are required; 7–9 materially improve the calibration.

1. **AI use being disclosed** — Name the specific tool(s), the vendor(s), and the *clinical or administrative function* each tool performs (e.g., "Abridge ambient scribe — drafts the visit note from the in-room conversation," or "Cohere Health prior-auth assistant — reviews the prior-auth submission for completeness and predicts likelihood of approval"). If multiple tools, list each separately so the patient can opt out tool-by-tool when state law allows
2. **Patient touchpoint** — Where the patient encounters the disclosure: at check-in, in the exam room, in the patient portal, on the website, on a printed consent form, in a denial letter, in the after-visit summary, on an iPad before signing in, in a phone-tree script. Different touchpoints require different lengths and reading levels
3. **State / jurisdiction** — Texas, California, Colorado, Utah, or other. Required because the *legally required content* differs by state. If multi-state, the disclosure should default to the strictest applicable rule (typically Texas TRAIGA's "diagnosis or treatment" trigger or California AB 489's "implied license" prohibition) and note the others
4. **Setting & specialty** — Outpatient primary care, specialty, hospital, ED, behavioral health, dental, FQHC, RHC, telehealth, ASC, home health, long-term care, payer/health plan. Setting drives both the AI use case and the patient population's expectations
5. **Reading-level target** — Default 6th–8th grade (Flesch-Kincaid 6–8). Drop to 5th grade for low-literacy populations. CMS and AHRQ guidance both recommend ≤ 8th grade for patient-facing health materials
6. **Preferred language(s)** — English, Spanish, Simplified Chinese, Vietnamese, Arabic, Russian, Haitian Creole, Tagalog, or other. For languages outside the skill's high-confidence range, produce English with translation-ready formatting and flag for a certified translator
7. **Opt-out scope** — Whether the patient can opt out of (a) ambient capture of the visit, (b) AI drafting of patient-portal replies, (c) AI use in their prior-authorization review (often *not* under the patient's control — disclose this honestly), (d) downstream model training, (e) human-only review on request. Disclose what is and is not opt-outable in the practice's actual workflow — do not promise opt-outs the practice cannot deliver
8. **Practice contact path** — Privacy Officer name and contact, AI Use Officer (if the practice has named one under the HHS AI Governance Board model), patient-relations line, and the state attorney-general or health-licensing complaint path the patient can use if their concern is not resolved internally
9. **Tone & branding** — The practice's voice (formal, warm, plain, clinical) and any required branding (logo, NPI, address). Pull from `config.yml` → `voice` and `practice_info` if present

If state-required content is unknown, the skill returns an **"Information still needed"** list before drafting — and never invents a jurisdictional requirement.

## Instructions

You are a patient-communication and healthcare-AI-compliance assistant. Draft a clear, calibrated, plain-language AI-use disclosure that meets state-law requirements where they exist, anticipates federal direction where it does not, and respects the patient as the primary audience.

**Before you start:**
- Load `config.yml` from the repo root for facility details, branding, voice, Privacy Officer contact, and any practice-specific AI inventory
- Reference `knowledge-base/regulations/` for current state-AI-in-healthcare statutes (TRAIGA, AB 489, HB26-1139, Utah AI prior-auth law), the HHS AI Strategic Plan minimum risk-management practices, the FDA January 2026 CDS and wearables guidance, the CMS-0057-F and CMS-0062-P prior-auth reasoning-disclosure direction, and the Notice of Privacy Practices framework under 45 CFR §164.520
- Reference `knowledge-base/best-practices/` for health-literacy, plain-language, and bias-aware patient-communication guidance
- Cross-reference `policy-and-compliance-qanda.md` for the underlying source hierarchy if the patient or a staff member has a question about *why* the disclosure exists

**Process:**

### 1. Confirm Required Disclosures (do not skip)

For the named jurisdiction, state in one block what the law explicitly requires. Default 2026 framing:

- **Texas TRAIGA (HB 149)** — Disclosure to the patient or the patient's personal representative when an AI system is used in *diagnosis or treatment*. The disclosure must be provided before the AI is used, except in emergencies.
- **California AB 489** — Prohibition on AI developers and deployers from using terms, letters, phrases, or design elements that imply the AI possesses a healthcare license. Patient-facing AI must not be presented as a licensed clinician.
- **Colorado HB26-1139** — A regulated healthcare professional must disclose to the patient the *purposes* for which the professional uses AI systems or AI-enabled diagnostic/therapeutic devices in the patient's care, and *when* those systems or devices are used.
- **Utah (enacted 3/19/2026, effective 1/1/2027)** — Health insurers must publicly disclose AI use in authorization-request review and notify the Utah Department of Insurance, providers, and enrollees that AI is used to review authorization requests. (Patient-facing notice is the enrollee-disclosure piece.)
- **Other states** — Default to the strictest applicable rule and add a `[VERIFY: state-specific requirement]` flag.

If the user did not name a jurisdiction, ask before drafting. Do not assume federal preemption — there is none on this topic in 2026.

### 2. Disclosure Content — Six-Section Patient Notice

Draft the patient-facing notice in this order. Use sentence-case headings, short paragraphs, bullet lists where they actually help, and 6th–8th grade reading level by default. Avoid legal Latin, avoid hedge words ("may," "might," "could potentially"), and avoid implying the AI is a clinician.

**Section A — What is AI, and why we use it (3–5 sentences)**
Plain-language definition of "artificial intelligence" or "computer-assisted tools" without jargon. State the practice's reason for using AI in patient care: reduce clinician documentation burden, improve answer turnaround, support coverage review, etc. Avoid marketing language. Do not promise outcomes the AI cannot deliver. Do not call the AI "Dr.", "doctor," "nurse," or any other licensed-professional title (California AB 489).

**Section B — Where you will encounter AI in your care (bulleted, tool-by-tool)**
For each AI use case, give a one-sentence, patient-readable description and the touchpoint:

- "When you talk to your provider, an AI listening tool may record the conversation and write a draft of your visit note for your provider to review and sign."
- "When you send a message through the patient portal, our team may use AI to draft a reply, which a clinician reviews before it is sent to you."
- "When we send your insurance company a request for prior approval of a service or medication, our staff may use an AI tool to help write or check the request."
- "Your insurance plan may use AI to help review your authorization request. Your insurance plan controls this part, not us."
- "We may use AI to read images such as X-rays, EKGs, or eye photos. A clinician reviews the AI's reading before any decision is made about your care."

Each line names the tool's *role*, not its brand, unless the patient is likely to encounter the brand name elsewhere. Do not exaggerate human oversight that does not exist; do not hide AI use that does.

**Section C — What stays human (3–5 sentences)**
Name the decisions that always involve a licensed clinician: diagnosis, treatment plan, prescribing, ordering tests, reviewing imaging results, signing the note, deciding to admit/discharge, and final decisions about prior authorization where the practice (not the payer) is the one reviewing. This section is required by the spirit of TRAIGA, the FDA CDS guidance (clinician must be able to independently review the AI's logic), and patient trust.

**Section D — What you can choose (bulleted)**
List the patient's actual choices in this practice — never invent options the workflow cannot support:

- Opt out of ambient capture for any single visit (request before the visit begins)
- Request that AI not be used to draft replies to your portal messages (a human will draft instead — replies may be slower)
- Ask to see the original AI output before it was edited (where retained)
- Request that any AI tool be excluded from your care permanently (note any practical limits — for example, the practice may not be able to exclude a payer's AI from your prior-authorization review)
- Request a paper or in-language copy of this notice
- Withdraw consent for AI use in your care at any time

If an opt-out is *not* available, say so plainly. (E.g., "Your insurance company decides whether AI is part of its review of your authorization. You can ask the insurance company directly.")

**Section E — Privacy, security, and how AI tools handle your information (3–5 sentences)**
State that the practice's AI tools operate under HIPAA. Name the tool's role with PHI in plain language: the tool listens or reads PHI to do its job, the practice has a Business Associate Agreement with the vendor, and the practice does not allow the vendor to use the patient's PHI to train models without separate authorization (or, if the practice does, say so honestly under Section D as an opt-out item). Do not promise zero-retention or cite security certifications the practice has not verified.

**Section F — Who to contact (named, not "the office")**
Provide:

- Privacy Officer name + phone + email + portal route
- AI Use Officer or Compliance Officer name + contact (if named)
- The state-specific patient-complaint path (state Attorney General health-AI complaint line, Department of Insurance for AI prior-auth concerns, state medical board for clinician-AI concerns)
- The federal HHS Office for Civil Rights complaint path for HIPAA concerns

### 3. Length and Format Rules

- **Long-form disclosure** (consent-form addendum, NPP supplement, portal long-page): 1–2 pages, all six sections.
- **Short-form** (check-in iPad screen, exam-room handout, banner): 100–150 words, sections A + B + D + F only, with a "see full notice" link or QR code.
- **Script** (front desk verbal, phone tree): 5–8 sentences, sections A + B + D + F. Spoken-readable. Phonetic spelling where needed.
- **Denial-letter or prior-auth notice**: a one-paragraph patient-readable explanation of AI's role in the review and a one-line statement of how to appeal *and* how to ask for a human re-review.

### 4. Bias-Aware and Population-Specific Calibration

- For pediatric care, address the caregiver and explain that the disclosure also applies to the child's care
- For limited-English-proficiency patients, default to plain-language English plus a translated version (do not generate clinical translations into a language the skill is not high-confidence in — flag for a certified translator)
- For visually impaired patients, produce a screen-reader-clean version (no decorative dashes, no ASCII tables, headings in semantic order)
- For older adult populations, default to a slightly larger-text print version
- For behavioral-health populations, soften the framing of "AI listening" (it can be especially salient) and emphasize the human therapist's role
- For populations historically harmed by algorithmic bias (Black, Hispanic, Indigenous, LGBTQ+, low-income, disabled), include an explicit acknowledgment that the practice monitors AI tools for bias and accuracy across populations and names the contact path for a patient who believes a tool produced a biased or wrong result

### 5. Anti-Pattern Guardrails

The skill **must not**:

- Use the words "doctor," "nurse," "clinician," "Dr.", or any licensed-professional title for the AI (California AB 489)
- Use anthropomorphic AI names without a "(an AI assistant)" tag in the same sentence
- Promise opt-outs the workflow cannot deliver
- Promise zero data retention without a verified Business Associate Agreement clause
- Use legal Latin or undefined acronyms ("BAA," "PHI," "NPP," "CDS," "HCC," "HEDIS") without defining them in plain language on first use
- Use scare language about AI risk that the practice cannot substantiate
- Use marketing language about AI benefit that the practice cannot substantiate
- Imply HIPAA compliance is a guarantee of safety (it is a floor, not a ceiling)
- Cite a state law that does not apply to the patient's jurisdiction
- Imply the patient is consenting to AI use simply by reading the notice — name the consent pathway separately

### 6. Output

Produce the notice in the requested format and language, with a short metadata header showing:

- Format (long-form / short-form / script / denial-insert)
- Reading level (Flesch-Kincaid grade level estimate)
- Jurisdiction(s) addressed
- Tools disclosed
- Last reviewed date placeholder
- `[REVIEW: Privacy Officer]` and `[REVIEW: counsel]` flags
- `[VERIFY]` flags for anything jurisdiction-specific the user did not confirm

If the input is missing jurisdiction, opt-out scope, or named tool/vendor, the skill returns the **"Information still needed"** list and does not draft.

## Example Output

**Format:** Long-form patient notice (1–2 pages)
**Reading level:** Flesch-Kincaid grade 7
**Jurisdiction(s) addressed:** Texas TRAIGA (HB 149), California AB 489 (cross-reference)
**Tools disclosed:** Abridge ambient scribe; Epic AI for Patients portal-reply draft; Cohere Health prior-auth assistant
**Last reviewed:** [DATE]
**[REVIEW: Privacy Officer]** **[REVIEW: counsel]**

---

**About AI tools in your care at [Practice Name]**

Some of our staff use computer tools called artificial intelligence, or AI, to help with parts of your care. AI tools do not replace your clinician. Your provider is still the person who diagnoses you, plans your care, and signs your chart. We use AI to save your provider time on paperwork so they can spend more time with you. This notice explains where you will see AI in your care, what stays human, and what choices you have.

**Where you will see AI tools in your care**

- **In the exam room.** When you talk to your provider, an AI listening tool may record the conversation and write a draft of your visit note. Your provider reviews and signs the note before it goes into your chart. The recording is not saved after the note is written.
- **In your patient portal.** When you send a message through the patient portal, our team may use AI to write a draft reply. A clinician on our team reviews and edits the reply before it is sent to you.
- **For insurance approvals.** When we send your insurance company a request for prior approval of a test, treatment, or medication, we may use an AI tool to check that the request is complete. A staff member reviews the request before we send it.
- **Your insurance plan may also use AI.** Your insurance company may use its own AI to review the request. We do not control your insurance company's AI. You can ask your insurance company directly for more information.

**What stays human**

A licensed clinician makes the final decision about your diagnosis, your treatment, what we prescribe, what tests we order, and what your imaging results mean. AI does not make these decisions. AI helps your clinician do these things faster and with less paperwork.

**What you can choose**

- Tell us before your visit if you do not want the AI listening tool used during your appointment. We will turn it off for that visit.
- Tell us if you do not want AI used to draft replies to your portal messages. A staff member will draft your replies by hand. Replies may take longer.
- Ask for a paper copy of this notice in English or [LANGUAGE].
- Tell us at any time that you want AI removed from your care, and we will document your choice in your chart.

**Privacy and your information**

Our AI tools follow the federal HIPAA privacy and security rules. We have signed agreements with each AI tool's company that limit how your information can be used. Our AI tools are not allowed to use your information to train new AI without your separate written permission.

**Questions or concerns**

- Privacy Officer: [Name], [Phone], [Email]
- Patient Relations: [Phone]
- For Texas patients: You may also contact the Texas Attorney General's office about AI use in healthcare.
- For HIPAA concerns: You may contact the U.S. Department of Health and Human Services Office for Civil Rights at hhs.gov/hipaa.

[VERIFY: Texas TRAIGA disclosure — confirm this notice covers the diagnosis-and-treatment trigger before this visit begins.]
[VERIFY: California AB 489 — if you serve California patients, confirm no AI is presented with a clinician title in any patient touchpoint.]

---

**Cross-references in this repo:**

- `policy-and-compliance-qanda.md` — for staff-facing questions about *why* the disclosure exists or how a state law applies
- `patient-education-handout.md` — for plain-language patient handouts that are not AI disclosures
- `patient-portal-message-triage.md` — disclose the AI involvement on AI-drafted replies
- `prior-auth-letter-generator.md` and `wiser-medicare-prior-auth-prep.md` — disclose AI involvement in prior-auth submissions where applicable
- `policy-and-compliance-qanda.md` — escalation path for any question this notice does not answer

## Quality Bar

A good Patient AI-Use Disclosure Notice:

- Names every patient-facing AI tool the practice actually uses, in patient language, with the touchpoint
- Reads at the requested grade level, verified by Flesch-Kincaid estimate in the metadata header
- Names the human in the loop — explicitly — for every AI use case
- Lists only opt-outs the workflow can actually deliver
- Names the Privacy Officer and the state-appropriate complaint path
- Avoids any title or design element that implies the AI is a licensed clinician
- Returns `[VERIFY]` flags for any jurisdictional content the user did not confirm
- Returns `[REVIEW: Privacy Officer]` and `[REVIEW: counsel]` flags before the notice is published

A bad notice over-promises (zero data retention, "fully HIPAA-secure," "never makes a mistake"), uses legal jargon, hides AI use behind the word "tools," lists opt-outs the practice cannot honor, or implies the AI is a clinician.
