---
name: "Patient Portal Message Triage"
category: customer-service
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~4 min/message"
version: 1.0
last_eval_score: null
---

# 📬 Patient Portal Message Triage

## Purpose

Classify an inbound patient portal message (MyChart, Athena, eClinicalWorks, Epic MyChart, NextGen, or similar), route it to the correct queue, detect clinical urgency, surface the minimum context a human responder needs, and draft a reply the clinician or staff can verify and send in seconds. Roughly a third of portal messages are administrative and do not need a clinician; AI triage reclaims that time while escalating the ones that genuinely need clinical eyes.

## When to Use

Use this skill in outpatient and ambulatory specialty settings whenever inbound portal messages need to be sorted, de-escalated, or pre-drafted. Common scenarios include:

- Primary care inbox management where a single clinician routinely sees 80+ messages per day
- Nurse and MA triage pools filtering messages before they reach the provider
- Specialty clinics (cardiology, endocrinology, behavioral health, oncology, obstetrics) with high-volume refill and test-result questions
- Patient experience or contact center teams handling combined voice + portal intake
- FQHCs and community health centers where bilingual triage is needed
- Post-visit follow-up where the patient sends a question three days after the encounter
- "After-hours" portal messages that land in the next business day's queue
- Messages that mention billing, insurance, scheduling, or pharmacy on the same thread as a clinical question
- Bulk triage runs where 20–100 messages need to be pre-classified before the clinician opens the inbox

This skill assists routing and drafting. It does not autonomously send replies, does not close a thread, does not diagnose, and never generates a response to a message flagged as a safety event.

## Required Input

Provide the following per message:

1. **Message content** — The raw text of the portal message. If it is a reply in a thread, include the last 2–3 exchanges for context.
2. **Patient basics** — Age, sex at birth, preferred language, problem list (or top 3 diagnoses), active medications, allergies, most recent visit date and type, primary clinician, assigned care team. PHI should be handled per practice policy; de-identify for external-model testing.
3. **Sender identity** — Is this the patient? A proxy/legal representative with release on file? An unknown sender? (If identity is ambiguous, the skill returns a verification step instead of a draft.)
4. **Inbox type** — Provider personal pool, team pool, nurse triage pool, practice admin, behavioral health, billing, records, or shared.
5. **Practice routing map** — A short list of the practice's available queues (e.g., Scheduling, Refills-RN, Nurse-Triage, PA-Letters, Billing, Front-Desk, Clinical-Inbox-PCP). If not provided, the skill uses generic queue names and flags them for local mapping.
6. **Clinician preferences** — Response tone (warm/concise/formal), reading-level target, language preferences, and any templated language the practice already uses (e.g., refill policy, no-opioid policy, same-day vs. routine cutoffs).

If the sender is a proxy without release on file, a minor without parent/guardian linkage, or an unknown third party, the skill must not draft a clinical reply. It returns an identity-verification request instead.

## Instructions

For every message, produce a six-field structured output plus an optional draft reply.

### 1. Urgency Classification

Assign one of five levels and state why in a single sentence:

- **L0 — Emergent / 911**: Any language suggesting chest pain with features of ACS, stroke symptoms, active suicidal or homicidal ideation with plan/intent, severe bleeding, anaphylaxis, pregnancy with heavy bleeding or severe abdominal pain, pediatric respiratory distress, new focal weakness, overdose, acute severe depression with plan, or any statement of self-harm. **Action: do NOT draft a reply. Produce a "call-the-patient-now" alert with the exact phone number on file and a suggested script for a nurse/provider to call within minutes.** Tag `[SAFETY — IMMEDIATE]`.
- **L1 — Same-day clinical**: New symptom that may require same-day evaluation (e.g., fever + dysuria in a pregnant patient, new unilateral leg swelling, sudden vision change, post-op concern within 30 days, poorly controlled blood sugar with ketones, medication reaction). **Action: route to nurse triage with a proposed same-day-visit or telehealth offer.**
- **L2 — Clinical within 24–72 hours**: Persistent non-emergent symptoms, medication side effects, test-result follow-up the patient wants discussed, behavioral health concerns without immediate risk. **Action: route to clinician inbox with a pre-drafted reply.**
- **L3 — Administrative routine**: Refills on chronic meds with no red flags, scheduling, records, forms, billing questions, portal login, FMLA/disability paperwork, letters. **Action: route to the specific admin queue and pre-draft the standard response.**
- **L4 — Informational / no action**: Thank-you notes, acknowledgments, social check-ins. **Action: short acknowledgment draft or auto-close per practice policy.**

Never downgrade urgency because the patient is polite or apologetic. Common phrases like "sorry to bother you, but I've had chest pain since this morning" must be treated as L0/L1 based on the clinical content, not the tone.

### 2. Intent & Topic Tags

Select one primary intent and zero-to-three secondary topics. Standard intent set:

- Symptom question
- Medication question (new, refill, side effect, interaction, cost)
- Test-result question
- Prior auth / referral question
- Scheduling / rescheduling / cancellation
- Billing / insurance / statement
- Records request / forms / letters / FMLA
- Behavioral health / mood / sleep
- Preventive / screening question
- Device or monitoring question (CGM, BP cuff, wearable)
- Care coordination (SNF, home health, specialty handoff)
- Dissatisfaction / complaint
- Social / thank-you

### 3. Key Context the Responder Needs (Pre-Chart Review)

Three to six bullet points, each sourced from the patient chart input, including:

- Most recent relevant diagnosis, labs, or imaging date
- Active medication that affects this message (e.g., "on apixaban" for a bruising question)
- Last relevant visit date and what was addressed
- Any open care gap or missing data (e.g., "no BMP in 90 days")
- Care manager or specialist involvement
- Known SDOH or access barrier if flagged in the chart

Never fabricate chart data. If context is missing, surface it as a "chart pull needed" bullet rather than inventing values.

### 4. Suggested Routing Decision

State the target queue using the practice's routing map. If the message crosses domains (e.g., clinical + billing), recommend a **primary** queue and a **secondary** handoff, not a split message. Include SLA guidance based on urgency level.

### 5. Draft Reply (L2, L3, L4 only)

Produce a verify-and-send draft with the following rules:

- Reading level: 6th–8th grade by default, or the language-level specified by the practice.
- Tone: warm, specific, and brief. Default to 60–120 words unless the question genuinely requires more.
- Open with an acknowledgment of the patient's message in their own words.
- Answer the direct question first; add only the context the patient needs.
- Close with a clear next step and a realistic timeframe.
- Never make a new diagnosis, never change a dose, never promise a clinical outcome.
- For medication refills: check against the practice's refill policy if provided; default to "we will have your clinician review this and respond within [timeframe]" if policy is not supplied.
- For test results: reflect only what the patient is allowed to see per the practice's release policy; offer to schedule a visit when clinical judgment is required.
- For behavioral health: never imply diagnosis or prognosis; offer next step + crisis-resource mention when appropriate.
- Always include a clear re-contact instruction: how to reach the team if symptoms change, and the appropriate after-hours or 911 guidance.
- Close with a human name placeholder (`[Clinician/Team Name]`), never sign the message as "your AI assistant."

For L0 and L1 messages, do **not** produce a drafted reply. Produce the escalation script for the human responder instead.

### 6. Metadata Block for the EHR

Output a compact block that staff can drop into the EHR:

```
Triage: L[level] — [urgency reason]
Intent: [primary] ([secondary tags])
Route to: [queue] (secondary: [queue])
SLA: [same-day / 24h / 72h / routine]
Context flags: [SDOH / polypharmacy / post-op / pregnancy / behavioral health / peds proxy / etc.]
Safety flags: [NONE | SAFETY — IMMEDIATE | SENSITIVE CHANNEL | IDENTITY — VERIFY]
Reviewer action: [approve-and-send / nurse-call-out / provider-review / admin-close]
```

### Safety & Privacy Guardrails

- **Safety-first stop** on any explicit or implicit mention of suicidal ideation, homicidal ideation, self-harm, abuse, overdose, stroke symptoms, chest pain with ACS features, anaphylaxis, pediatric respiratory distress, pregnancy + heavy bleeding, or loss of consciousness. No draft reply — only the escalation alert.
- **Identity** — If the sender's identity cannot be confirmed as the patient or a proxy with release on file, refuse to draft a clinical reply and output a verification request.
- **Pediatrics + adolescent confidentiality** — Respect state-specific adolescent confidentiality laws. For messages touching sexual health, substance use, or mental health for minors, flag for clinician review and do not draft content that would be visible to a linked proxy without confirmation of the consent configuration.
- **Behavioral health** — Never suggest a specific diagnosis. Include a crisis-line reminder (e.g., 988 Suicide & Crisis Lifeline in the US) when mental health content appears, but defer final wording to the clinician.
- **Sensitive content** — IPV disclosures, HIV/STI content, substance use, and immigration concerns route to the appropriate private queue; do not auto-include in a shared thread.
- **PHI minimization** — Do not include chart data in the draft reply beyond what the patient already knows or is clearly entitled to see.
- **Regulatory context** — The FDA's 2026 guidance on non-device clinical decision support (single clinically appropriate recommendation, clinician can independently review the logic) shapes how this skill's output is framed: it is a pre-draft for clinician review, not an autonomous diagnostic or treatment decision.
- **Never auto-send** — The output is always a draft plus routing. A human must approve before the patient sees a reply.

### Anti-Error Guardrails

- Never invent lab values, medication doses, or visit dates.
- If the patient asks a question the chart cannot answer, say so and recommend a phone call or a scheduled visit.
- Do not translate clinical advice into a language you cannot produce accurately; flag for human translator or bilingual staff.
- For refill requests, never assert quantity, day supply, or pharmacy unless they appear in the input. Default to "we will confirm with your usual pharmacy on file."
- If a message mentions both a clinical and a billing question, do not produce a billing answer that implies clinical guidance, and vice versa.

## Example Output

```
INBOUND PORTAL MESSAGE — Triage Summary

Patient: 58F, English, active Rx: metformin 1000 mg BID, lisinopril 20 mg daily,
atorvastatin 40 mg; last PCP visit 2026-02-10; HbA1c 2026-02-08: 8.1% [VERIFY].
Message: "Hi Dr. ___, I got my blood sugar meter reading and it said 312 this morning
fasting. I didn't eat anything unusual. I feel fine. Do I need to do something or just
keep watching it? Sorry for bothering you on a Sunday."

1. Urgency: L2 — Elevated fasting glucose without symptoms; not emergent, but poor
   control warrants clinician follow-up within 24–72 h. Confirm no DKA features
   (nausea, vomiting, abdominal pain, fruity breath) in next touch.

2. Intent: symptom question (glycemic control); secondary: medication question,
   preventive.

3. Context the responder needs:
   - HbA1c 8.1% at last draw (2026-02-08) — trending up from 7.4% in Nov 2025 [VERIFY]
   - On metformin 1000 mg BID; no SGLT2i or GLP-1 on board
   - No recent UA or BMP on file (>60 days)
   - Lives alone; prior note of food-cost concern — SDOH flag
   - No prior DKA history; not on insulin

4. Routing: Primary queue "Nurse-Triage-PCP" with SLA 24 h; secondary: coordinate
   same-week PCP telehealth.

5. Draft reply (verify and send):

   Hi [Patient first name], thanks for reaching out — it is not a bother.

   A fasting reading of 312 is higher than we want, even with no symptoms. Please
   watch for nausea, vomiting, belly pain, fruity breath, or feeling very tired;
   if any of those show up, call the office right away, and if you feel very sick
   or short of breath, call 911.

   In the meantime: keep taking your usual medicines, drink water, and check your
   sugar again in a few hours. I will have [Clinician/Team Name] review your
   numbers today and reach out with next steps, likely a short visit this week to
   adjust your plan.

   If your reading is above 350 or you do not feel well, please call the office now.

   — [Clinician/Team Name]

6. Metadata
   Triage: L2 — elevated fasting glucose, asymptomatic
   Intent: symptom-question (diabetes_control); medication
   Route to: Nurse-Triage-PCP (secondary: PCP-Telehealth-Scheduling)
   SLA: 24h
   Context flags: SDOH (food cost), trending A1c, labs-stale
   Safety flags: NONE
   Reviewer action: nurse-approve-and-send, schedule telehealth follow-up
```

## Notes on Deployment

- This skill assumes human-in-the-loop review before any reply is sent. 2026 FDA CDS guidance, practice professional-liability exposure, and patient trust all require a verified human signer.
- The skill's most durable value is classification + context surfacing; the draft reply is a "save me keystrokes" feature, not a substitute for clinical reasoning.
- Volume handling: the skill is designed to be run per-message, but can be batched by paging through a queue of 20–50 messages; keep the same chart-context hygiene per message.
- Equity: monitor whether drafts for non-English-preferred patients, older patients, or historically marginalized groups systematically receive longer wait suggestions or less specific next steps. Re-train practice templates and language-level defaults if drift appears.
- 2025–2026 research (JHM BIDS, Dartmouth, Epic AI for Patients) consistently shows inbox AI improves clinician experience most when classification is tight and drafts are conservative. Err toward "route to human" when in doubt.
