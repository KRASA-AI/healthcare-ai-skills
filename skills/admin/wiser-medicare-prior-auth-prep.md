---
name: "WISeR Medicare Prior Auth Prep"
category: admin
tools: [claude, chatgpt]
difficulty: advanced
time_saved: "~30 min/submission"
version: 1.1
last_eval_score: 9.20
---

# 🧾 WISeR Medicare Prior Auth Prep

## Purpose

Assemble a WISeR-ready prior authorization or pre-payment review packet for a Traditional (Original) Medicare beneficiary who lives in a WISeR state and is scheduled for a service on the WISeR list. The goal is a clean, evidence-complete submission that clears AI screening and human clinical review on the first pass, avoids pre-payment denial, and keeps the provider on a fast track toward gold-carding.

## When to Use

Use this skill whenever a practice, ASC, hospital, or DME supplier is preparing a service that falls under the CMS WISeR (Wasteful and Inappropriate Service Reduction) Model. Common scenarios include:

- Cervical or lumbar spinal fusion and related posterior/lateral instrumented procedures
- Epidural steroid injections, facet joint injections, and select interventional pain procedures
- Skin substitute applications for chronic non-healing wounds (diabetic foot ulcers, venous stasis ulcers)
- Arthroscopic knee surgery for osteoarthritis, incontinence procedures, and other selected WISeR service categories
- Services rendered on or after January 15, 2026 in the six WISeR states (New Jersey, Ohio, Oklahoma, Texas, Arizona, Washington)
- Providers choosing the PA pathway vs. the pre-payment review pathway for a WISeR-covered service
- Resubmissions after a WISeR denial or request for additional documentation
- Expedited (48-hour) requests when a routine 72-hour determination would jeopardize life, health, or ability to regain maximum function
- Practices working toward gold-card status who need every submission to reflect tight adherence to existing Medicare coverage policy

This skill does NOT grant Medicare approval, replace clinician judgment, or change underlying coverage policy. It produces a structured draft a credentialed reviewer validates before submission.

## Required Input

Provide the following before running the skill:

1. **Beneficiary & service basics** — Beneficiary name/MBI (or redacted placeholder), date of birth, state of service, ordering/rendering provider NPI, facility, planned date of service, and the specific CPT/HCPCS codes plus ICD-10 diagnoses being requested.
2. **WISeR service category** — The specific service bucket (e.g., cervical fusion, epidural steroid injection, skin substitute, incontinence control device). If you are unsure whether a code is on the WISeR list for your state, flag it and the skill will note verification steps.
3. **Requesting pathway** — Prior authorization (before service) OR pre-payment review (claim submitted, awaiting review) OR resubmission after a non-affirmation. Note whether the request is routine or expedited.
4. **Clinical backbone** — H&P or problem-focused note, imaging report(s), prior conservative therapy details (duration, response, reason for failure), relevant specialist consult notes, labs, and any failed alternatives.
5. **Coverage policy reference** — The applicable Medicare National Coverage Determination (NCD), Local Coverage Determination (LCD) / Local Coverage Article (LCA), NCCI edits, and any Medicare Benefit Policy Manual chapter that governs the service. If unknown, the skill will list what to look up.
6. **Documentation gaps the submitter already suspects** — e.g., missing PT notes, no documented failure of conservative therapy, incomplete imaging narrative, out-of-window diagnostic tests.
7. **Provider history (optional)** — Prior WISeR approval rate for this service, any open CAP or gold-carding progress notes.

If required context is missing, return an **"Information still needed"** list rather than fabricating clinical detail. WISeR reviewers cite missing documentation as the top reason for non-affirmation.

## Configuration (personalization from `config.yml`) — v1.1

Items 1, 5, and 7 of the Required Input above are **standing facts about the practice**, not facts about the patient — the rendering provider's NPI/PTAN/TIN does not change between submissions, the MAC jurisdiction does not change between submissions, and the governing LCD for a given service line changes only when the MAC revises it. Re-keying them per packet is the single largest source of both wasted time and transcription error (a wrong PTAN is an administrative non-affirmation before a reviewer ever reads the clinical case).

Load `config.yml` from the repo root and honor the following keys when present. When a key is absent or partial, fall back to the documented default and emit a `[VERIFY: ...]` flag rather than inventing an identifier, a jurisdiction, or a policy number. This is the same hook pattern used by `prior-auth-letter-generator` and `payer-downcoding-rebuttal-letter` v1.1.

1. **`provider_identifiers`** — keyed roster per rendering/ordering provider: display name, credentials, specialty, **NPI, PTAN, TIN**, and the contact line for clarifying questions. Auto-fills **Section 1 (Submission Header)** and the **Section 8 attestation** signature block. When the provider is not in the roster, print the name as given and flag `[VERIFY: provider not in roster — confirm NPI/PTAN before submission]`. Never invent an NPI, PTAN, or TIN.

2. **`mac_jurisdiction_map`** — the practice's WISeR-state → MAC mapping and the MAC's submission channel (portal URL, fax, contact), e.g. `AZ: { mac: "Noridian JF", portal: "[url]" }`, `WA: { mac: "Noridian JF" }`, `NJ: { mac: "Novitas JL" }`, `TX: { mac: "Novitas JH" }`, `OH: { mac: "CGS J15" }`, `OK: { mac: "Novitas JH" }`. Auto-fills the Section 1 MAC line and the submission-routing note. When absent, the skill states the MAC as `[VERIFY: MAC jurisdiction for state of service]` and does not guess. Note that MAC jurisdiction assignments are a CMS contracting fact that can change — a configured map is authoritative for the practice only until the MAC changes, so pair this key with `policy_refresh_date` below.

3. **`wiser_policy_map`** — the practice's LCD/NCD/LCA citation library, keyed by **WISeR service category × MAC**, each entry carrying the policy number, title, effective date, the criterion list the policy imposes (duration thresholds, imaging look-back windows, conservative-therapy minimums, per-region injection caps), and the local coverage article for billing/coding detail. This is the key that makes **Section 4 (Policy Alignment Crosswalk)** render pre-populated with the *practice's actual governing criteria* instead of asking the submitter to look them up per packet. When absent, the crosswalk still renders, but every criterion row is emitted as `[VERIFY: retrieve governing LCD/NCD criterion]` and the skill lists which policy documents to pull.

4. **`policy_refresh_date`** — the date the `wiser_policy_map` was last verified against the MAC's published policy set. If the map is older than 90 days at run time, the skill prepends a single line to the output: `[VERIFY: policy map last refreshed {date} — confirm LCD effective date and revision history before submission]`. A stale configured citation is more dangerous than a missing one, because it looks authoritative. This flag is not suppressible.

5. **`service_line_attachment_packs`** — standing attachment checklists per WISeR service category, so **Section 7** ships pre-enumerated rather than reconstructed. Defaults if absent: **spinal fusion** → H&P, imaging report + PDF, ≥6–12 weeks conservative-care documentation, PT attendance log, medication trial log, surgical plan, attestation. **Epidural steroid / facet injection** → H&P, imaging with radicular correlation, prior-injection history with dates and levels (to evidence the per-region 12-month cap), conservative-care log, attestation. **Skin substitute** → wound etiology documentation, serial wound measurements, off-loading/compression compliance record, standard-wound-care failure across the policy-required duration, vascular assessment, product/units justification, attestation. **Arthroscopy for OA** → imaging, mechanical-symptom documentation, conservative-care log, attestation.

6. **`wiser_determination_log`** — the practice's running record of prior WISeR determinations for this service line: affirmation rate, the reviewer reasons cited on any non-affirmation, and gold-carding progress. Drives two things: the Section 2 framing when the practice has a strong track record on this service, and — more usefully — a **pre-submission check against the practice's own prior non-affirmation reasons**, so the packet does not repeat a defect this practice has already been dinged for. When absent, the skill skips the check and notes that no determination history was supplied.

7. **`expedited_criteria`** — the practice's documented criteria for requesting the 48-hour expedited determination rather than the 72-hour routine one. Drives the Section 1 expedited justification. When absent, the skill applies the statutory standard (life, health, or ability to regain maximum function) and refuses to draft an expedited justification that the supplied record does not support — see Anti-Error Guardrails.

8. **`config_missing_behavior`** — `flag_and_proceed` (default) or `block_and_ask`. `flag_and_proceed` ships the packet with `[VERIFY: ...]` flags on missing keys; `block_and_ask` returns the missing-key list first. High-volume ASC and pain-management workflows typically prefer `flag_and_proceed`; new-practice onboarding typically prefers `block_and_ask`.

When `config.yml` is absent entirely, the skill still produces the full eight-section packet with every practice-specific field replaced by a `[VERIFY: ...]` placeholder. It never invents an NPI, PTAN, TIN, MAC assignment, LCD number, or policy criterion.

## Instructions

Follow this eight-section packet structure. Use plain text, structured headers, and embedded `[VERIFY]` flags on any statement that should be confirmed by the ordering provider or coder before submission.

### 1. Submission Header

- Beneficiary identifiers (redact MBI to last four if shared outside PHI channels)
- WISeR state + MAC jurisdiction (e.g., Noridian JF for AZ/WA, Novitas for NJ/TX, CGS for OH)
- Pathway: Prior Authorization vs. Pre-Payment Review; routine vs. expedited with justification
- Requesting provider name, NPI, PTAN, TIN, contact for clarifying questions
- Planned date of service and site of service (ASC, HOPD, office)
- CPT/HCPCS code(s), ICD-10 diagnosis code(s), units, and modifiers
- Submission date and tracking number when available

### 2. Medical Necessity Summary (1–2 paragraphs)

Lead with the clinical story in plain language: who the beneficiary is, the functional deficit or disease burden, what has been tried, why the requested service is the next clinically appropriate step under the governing NCD/LCD, and the anticipated outcome. Tie back to at least one patient-specific data point (imaging finding, functional score, duration of symptoms) and at least one policy citation (NCD/LCD section or Medicare Benefit Policy Manual chapter).

### 3. Clinical Facts Table

Organize into a structured block:

```
[Relevant history]
  Symptom onset: ...
  Duration: ... (must meet NCD/LCD duration threshold — [VERIFY])
  Functional impact: ... (ADLs, work, caregiving)

[Diagnostics]
  Imaging: modality, date, interpretation (must fall inside policy look-back window — [VERIFY])
  Labs / EMG / nerve conduction / urodynamics / etc. as applicable

[Conservative therapy tried]
  Modality | Dates | Duration | Response | Reason discontinued
  ... rows as needed ...

[Contraindications / red flags addressed]
  ...

[Failed alternatives or step therapy if applicable]
  ...
```

The table must match the specific requirements for the WISeR service category (e.g., spinal fusion typically requires documented failure of 6–12 weeks of conservative care, recent imaging, radicular correlation; epidural steroid injections require documented radicular pain with imaging correlation and typically no more than the policy-allowed injections per region per 12 months; skin substitutes require documented wound etiology, appropriate off-loading/compression, standard wound care failure across the policy-required duration).

### 4. Policy Alignment Crosswalk

Create a two-column crosswalk showing NCD/LCD criterion ↔ patient-specific evidence that satisfies it. Example row format:

```
Policy Criterion                                | Evidence in Record
------------------------------------------------|-----------------------------------
≥ 6 weeks documented conservative therapy       | PT notes 2025-11-03 through 2026-01-14; see Section 3
Radicular pain with imaging correlation         | MRI 2026-02-12 — L5-S1 disc herniation compressing S1 root; clinical exam matches dermatome
Failure of second-line non-opioid pharmacology  | Gabapentin 300 mg TID x 6 weeks; see med list
```

Flag any row where evidence is thin or missing as `[GAP — ADD BEFORE SUBMISSION]`.

### 5. Safety and Alternatives Statement

Briefly document: why non-operative or lower-intensity alternatives are inadequate or have failed; any comorbidity mitigations (anticoagulation plan, cardiac clearance, diabetes control for wound patients); shared decision-making discussion and patient preference where relevant. Note ASA class for procedural cases.

### 6. Requested Action & Code Set

State precisely what is being requested:

- CPT/HCPCS with units
- Expected modifiers
- Any bundled or secondary codes (laminectomy + fusion, imaging guidance, multiple levels)
- Site of service justification (ASC vs. HOPD vs. inpatient if applicable)
- Expected length of stay or number of sessions if episodic

### 7. Attachments Checklist

Enumerate every attached document with date and page count. Typical set:

- H&P and most recent progress note
- Relevant imaging report(s) and the actual report PDF (not just snippet)
- Operative or treatment plan from the rendering provider
- Conservative care documentation (PT/OT notes, wound care flow sheets, injection history)
- Medication list showing failed pharmacologic trials
- Specialist consult letters
- Any prior WISeR/PA correspondence or non-affirmation letters (if resubmission)
- Executed ABN if applicable
- Signed attestation from rendering provider

### 8. Rendering Provider Attestation

Close with a single-paragraph attestation for the rendering provider's signature: "I certify the service described above is medically reasonable and necessary for this beneficiary, consistent with [cite NCD/LCD], and that the documentation submitted accurately reflects the beneficiary's clinical status as of [date]."

### Resubmission & Expedited Add-Ons

- **If this is a resubmission**, add a Section 1A titled "Response to Non-Affirmation Findings" that lists each reviewer reason verbatim and pairs it with the specific new/clarified evidence that resolves it. Do not re-argue the merits without addressing the reviewer's stated grounds.
- **If expedited**, add a clinical justification for expedited review (life, health, function, pain uncontrolled, risk of deterioration) in Section 1.

### Safety & Compliance Flags

Automatically surface `[SAFETY]` notes if the record shows:

- Service code that is outside the WISeR list for the beneficiary's state (skill should flag to verify rather than submitting as WISeR)
- Documentation that is outside the policy look-back window
- Missing signatures, dates, or credentials on critical source documents
- Any mismatch between ordered code and clinical indication (e.g., lumbar MRI cited for cervical fusion request)
- Pattern that looks like upcoding (e.g., multi-level fusion with only single-level imaging correlation)

### Anti-Error Guardrails

- Never invent imaging findings, PT note content, lab values, or dates.
- If the user supplies a code that does not match the service described, flag the mismatch and stop — do not re-label.
- If the NCD/LCD citation is unknown, say so and list the specific Medicare policy documents to retrieve (by number if known).
- If the submitter asks for a letter that overstates urgency or severity to push through expedited review, refuse the edit and explain.

## Example Output

```
WISeR PRIOR AUTHORIZATION REQUEST — Draft v1

1. Submission Header
   Beneficiary: Jane D., MBI ending 4417 [VERIFY]
   DOB: 1952-07-14
   State: Arizona (Noridian JF)
   Pathway: Prior Authorization, routine
   Requesting provider: A. Romero, MD — NPI 1234567890, PTAN AZ-XXXX [VERIFY]
   Facility: Mesa Pain & Spine ASC
   Planned date of service: 2026-05-02
   CPT: 64483 (TF epidural injection, lumbar/sacral, single level) + 77003
   ICD-10: M54.16 (radiculopathy, lumbar), M51.16 (intervertebral disc disorder with radiculopathy, lumbar)

2. Medical Necessity Summary
   73-year-old Traditional Medicare beneficiary in Arizona with 14 weeks of right-sided L5
   radiculopathy confirmed on 2026-02-12 MRI, refractory to documented conservative care including
   6 weeks of physical therapy, gabapentin titration, and activity modification. Transforaminal
   epidural steroid injection is requested as the next clinically appropriate step under Noridian
   LCD [VERIFY LCD#] for epidural steroid injections, with the goal of functional improvement and
   deferral of surgical escalation.

3. Clinical Facts
   [table per template]

4. Policy Alignment Crosswalk
   [crosswalk per template; each LCD criterion paired with record evidence]

5. Safety and Alternatives
   ...

6. Requested Action & Code Set
   CPT 64483 x 1 unit; 77003 imaging guidance bundled; HOPD not required — ASC appropriate for
   this uncomplicated, non-sedated single-level TF ESI.

7. Attachments
   - H&P 2026-04-02 (3 pp)
   - Lumbar MRI report 2026-02-12 (2 pp)
   - PT discharge summary 2026-01-14 (2 pp)
   - Medication reconciliation 2026-04-02 (1 p)
   - Signed attestation (1 p)

8. Attestation
   [single paragraph with signature block]

[GAPS FLAGGED FOR FIX BEFORE SUBMISSION]
   - Confirm Noridian LCD number for ESI, current effective date
   - Pull PT attendance log (number of completed sessions) — currently only discharge summary attached
   - Verify patient has not had ≥ 3 TF ESIs at same level in last 12 months
```

## Notes on Policy Boundaries

- WISeR only affects **Traditional (Original) Medicare Parts A/B** in the six listed states. Medicare Advantage prior auth is a separate process.
- WISeR does **not** change underlying Medicare coverage rules — it changes how compliance with those rules is reviewed. The skill's output must therefore always tie to existing NCD/LCD/Medicare Benefit Policy Manual language, not to invented criteria.
- Gold-carding is scheduled to pilot mid-2026. Practices working toward it should keep a structured log of every WISeR determination; this skill's output is designed to be archived alongside each submission.
- Nothing in this skill constitutes legal or billing advice. Final review by a credentialed coder and the rendering provider is required before submission.

## Version History

- **v1.1 (2026-07-13, skill evaluator)** — Added the **Configuration** section: `provider_identifiers` (NPI/PTAN/TIN roster), `mac_jurisdiction_map`, `wiser_policy_map` (LCD/NCD citation library keyed by service category × MAC, which pre-populates the Section 4 crosswalk), `policy_refresh_date` (non-suppressible staleness flag at 90 days), `service_line_attachment_packs`, `wiser_determination_log` (pre-submission check against the practice's own prior non-affirmation reasons), `expedited_criteria`, and `config_missing_behavior`. Closes the standing-fact re-entry gap in Required Input items 1, 5, and 7. Strictly additive — no guardrail, `[VERIFY]` convention, `[SAFETY]` flag, template section, or worked example was removed or weakened; every hook degrades to a `[VERIFY]` flag when config is absent.
- **v1.0** — Initial release: eight-section WISeR packet, policy-alignment crosswalk, safety/compliance flags, anti-error guardrails, worked example.
