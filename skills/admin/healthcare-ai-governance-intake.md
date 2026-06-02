---
name: "Healthcare AI Governance Intake & Vendor Review"
category: admin
tools: [claude, chatgpt]
difficulty: advanced
time_saved: "~45 min/intake"
version: 1.0
last_eval_score: null
---

# 🧭 Healthcare AI Governance Intake & Vendor Review

## Purpose

Produce a structured, auditable governance-intake record for any AI tool a health system, medical group, or community clinic is evaluating, piloting, expanding, renewing, or decommissioning — scored against an eight-pillar control framework aligned with the May 27, 2026 Coalition for Health AI (CHAI) governance playbook architecture, the forthcoming voluntary Joint Commission Responsible Use of AI certification track, the NIST AI Risk Management Framework, ISO 42001, the HHS AI Strategy (April 3, 2026 risk-management deadline), and the operative state and federal AI-in-healthcare regulatory perimeter. Output is designed for AI governance committees, AI Use Officers, CMIO / CNIO / CCIO offices, vendor-management offices, IT security and privacy teams running an AI-aware HIPAA risk analysis under 45 CFR §164.308(a)(1)(ii)(A), CDI and informatics leaders, medical-staff committees reviewing pilot-to-enterprise expansion, board AI subcommittees responding to the federal moratorium-dead environment, and community-health and safety-net clinics that need a resource-scaled intake pattern rather than the AMC-default deep-stack review.

This is a deployer's tool. It does not replace the vendor's own model card, the vendor's red-team report, the deployer's HIPAA risk analysis, the deployer's penetration test, the deployer's contract counsel, the deployer's IRB (when research applies), or any of the existing audit-class skills in this repo. It produces the upstream intake artifact that the audit-class skills consume — the structured pre-deployment record that captures what the tool is, who owns it, what evidence supports it, what risks were identified, what controls were attached, and what the renewal and decommission criteria are. The intake artifact is designed to feed into the AI Tool Registry export blocks that `ambient-scribe-note-audit.md`, `clinical-ai-copilot-response-audit.md`, `behavioral-health-ai-chatbot-compliance-review.md`, and `ai-chatbot-prompt-injection-audit.md` already produce, and into the source hierarchy of `policy-and-compliance-qanda.md`.

This skill is shaped by the CHAI playbook architecture released May 27, 2026 (eight pillars across organizational policy, structure, resources, lifecycle management, risk and impact assessments, responsible data management, third-party management, and education / training / feedback). It is intentionally engineered to be usable by community health centers and safety-net systems under shared-services models, not just AMCs — following the explicit democratization goal stated by CHAI's South Carolina Primary Health Care Association reviewer. It is intentionally separate from the audit-class skills because intake is *pre-decision* and *cross-tool*, while audit is *post-deployment* and *per-tool*.

## When to Use

Use this skill any time an AI tool enters or moves within the governance lifecycle. Common scenarios:

- **Pre-pilot intake.** A vendor proposal has arrived (ambient scribe, clinician copilot, patient-facing chatbot, agentic prior-auth platform, autonomous coding, no-show predictor, voice agent, ICU deterioration model, imaging assist, ED triage assist, scheduling optimizer, RCM autonomous-revenue-cycle stack) and the governance committee needs a one-document record before any pilot decision
- **Pilot-to-enterprise expansion.** A pilot has surfaced and the committee needs the upstream intake record to govern broader rollout — including a re-baselined risk-and-impact assessment per the CHI-Bench long-horizon agent reliability evidence captured in `ambient-scribe-note-audit.md` v1.4 and `prior-auth-letter-generator.md` v2.4
- **Multi-tool simultaneous deployment governance.** A Project-Pixel-style multi-tool Epic AI portfolio (twelve Epic AI capabilities deployed in one coordinated rollout) requires a consistent intake across every tool in the portfolio. Penn State Health's late-2026 full Epic AI suite intake is the same pattern
- **Annual renewal review.** A tool that has been in production for 12+ months is up for contract renewal and the committee needs a structured re-intake with post-deployment evidence layered onto the pre-deployment record
- **Vendor change / material model update.** The vendor has changed the base model, the system prompt, the retrieval index, the safety classifier, the hard-stop list, the agentic-extension scope, the data-flow architecture, or the BAA — and the committee needs a re-intake before the change reaches patients
- **State or federal compliance event.** Maryland HB 1563 effective June 1, 2026 (quarterly insurer reporting on AI-driven adverse decisions), CMS-0057-F AI-decision-reasoning disclosure already in force, the MACPAC June 2026 framework, the HTI-5 deregulation environment, Texas TRAIGA, California AB 489, Colorado HB26-1139, Connecticut SB 5, Vermont HB 814/816, Utah AI PA, Tennessee impersonation prohibition, the federal SAFE BOTs Act framework, or the 40+ state chatbot-bill wave requires a defensible intake record
- **Incident-triggered re-intake.** A patient-safety event, a PSO filing, an OCR or state AG contact, an FTC inquiry, a malpractice claim, a board inquiry, or a published vulnerability dossier (e.g., Mindgard / Doctronic) involving a deployed tool triggers a structured re-intake
- **Ambient-scribe consent posture review.** A two-party-consent-jurisdiction deployer (California, Florida, Illinois, Maryland, Massachusetts, Montana, New Hampshire, Pennsylvania, Washington) is responding to the Sutter / MemorialCare and parallel class actions and needs the intake to capture the per-encounter notification posture, the opt-out fallback, and the procurement-phase consent contract requirements
- **Joint Commission voluntary AI certification preparation.** The forthcoming Joint Commission Responsible Use of AI certification (built on the CHAI playbook architecture) requires the deployer to demonstrate the eight-pillar control set across the AI portfolio; this skill produces the per-tool intake record the certification surveyors will examine
- **Decommission record.** A tool is being retired (model end-of-life, vendor exit, performance failure, policy mismatch, contract non-renewal) and the committee needs the structured decommission decision, the data-disposition plan, the patient-notification plan if any, and the registry update
- **Community-health / safety-net resource-scaled intake.** A community health center, FQHC, RHC, safety-net hospital, or small medical group needs the intake at a resource-appropriate depth — the skill includes a "scaled control" tier for organizations operating under shared-services models that cannot reasonably stand up the full AMC control stack

This skill produces a governance work-product. It is **not** a clinical decision-support tool, **not** a substitute for counsel, **not** a substitute for the Privacy Officer or AI Use Officer, **not** a substitute for the vendor's own attestations, and **not** a substitute for the deployer's HIPAA risk analysis. It produces a structured intake record ready for the AI governance committee, the AI Tool Registry, the AI-aware HIPAA risk analysis, the Joint Commission voluntary AI certification file, the contract file, and the board AI subcommittee.

## Required Input

Provide items 1–8 at minimum. Items 9–14 materially improve the intake. If items 1–8 are not all present, return an **"Information still needed"** list before drafting and do not invent content.

1. **Tool identity and intended use case.** Vendor name, product name, version, base model (foundation LLM + any fine-tuning + provider) or non-LLM model class (e.g., gradient-boosted classifier, CNN, statistical), deployment surface (web app, mobile app, SMS, voice, EHR-embedded, hospital-branded, third-party telehealth, EMR sidecar, agentic-platform extension), the named "deployer of record," the intended use case (ambient scribe, clinician copilot, patient-facing chatbot, refill renewal, agentic PA, autonomous coding, no-show predictor, deterioration flag, imaging assist, ED triage, scheduling optimizer, RCM autonomy, voice agent, etc.), the audience (clinician-facing, patient-facing, both, administrative-only), and whether a Business Associate Agreement is in place.
2. **Risk tier classification.** Proposed risk tier — high-risk clinical, moderate-risk clinical, low-risk clinical, high-risk administrative, low-risk administrative — per the deployer's risk-tiering policy (typical NIST AI RMF and CHAI Pillar 5 framings). Include the reasoning for the proposed tier and any equity-sensitive workflow flag (behavioral health, OB, pediatrics, oncology, transplant, ED triage, refill / prescribing). If the deployer does not yet have a tiering policy, the intake should flag this as a Pillar 1 / Pillar 2 gap and propose a default tier from the rubric below.
3. **Lifecycle stage and decision under review.** Pre-pilot intake, pilot scoping, pilot-to-enterprise expansion, annual renewal, vendor change / material update, incident-triggered re-intake, compliance-event re-intake, certification preparation, or decommission. Identify the specific decision the committee will make at the end of this intake (approve / approve-with-conditions / pilot-with-fence / defer / decline / retire).
4. **Population, service line, and equity context.** Patient population the tool will touch (preferred-language distribution, age distribution, accommodation-need distribution, Medicaid / dual-eligible share, behavioral-health-comorbidity rate), service line(s) in scope (primary care, urgent care, ED, oncology, behavioral health, OB, pediatrics, transplant, inpatient, post-acute, home health, hospice), and any known equity-sensitive workflow that surfaced in vendor data or in prior audits.
5. **Vendor-supplied attestations.** Vendor's model card; vendor's published red-team report; vendor's OWASP 2025 LLM Top-10 attestation (for LLM-based tools); vendor's NIST AI RMF posture; vendor's ISO 42001 posture; vendor's HITRUST / SOC 2 / ISO 27001 status; vendor's third-party penetration-test summary; vendor's accuracy / fairness / drift evidence; vendor's clinical-evidence package (peer-reviewed publication, prospective RCT, retrospective validation, simulation bake-off, none); vendor's published intended-use statement and out-of-scope statement; vendor's hallucination / error-mode disclosures; vendor's published patient-notification language for any patient-facing surface; vendor's autonomous-action scope (read-only, draft-with-HITL, auto-accept, write-back, agentic).
6. **Deployer-side organizational posture.** The deployer's AI governance committee composition and meeting cadence; the named AI Use Officer / CMIO / CNIO / CCIO; the AI Tool Registry status (mature, in-progress, not-yet-established); the deployer's risk-tiering policy status; the deployer's monitoring and incident-reporting routes (PSO, AHRQ AI in Healthcare Safety Program, internal serious-incident workflow, OCR breach-determination pathway); the deployer's HIPAA risk-analysis cadence and AI-aware risk-analysis status; the deployer's clinician training program for the tool class; the deployer's patient-notification posture for patient-facing AI; the deployer's contract counsel involvement plan.
7. **Workflow integration plan.** How the tool integrates with the EHR, the patient portal, the clinical-decision-support stack, the billing / coding stack, the consent workflow, the chart-routing rules, the after-visit-summary, the patient-portal-message thread, and any downstream system (pharmacy, lab, imaging, scheduling, payer portal, UM vendor, peer-to-peer review process). Identify the named workflow owner, the named clinician champion, the named operational owner, and the named technical owner. Describe the HITL pattern (read-only, draft-with-HITL, auto-accept with sample QA, agentic with reviewer cycles, write-back to chart, write-back with dual-signature, autonomous-with-audit-log).
8. **Jurisdiction and regulatory posture.** Jurisdiction(s) in scope (state-by-state if multi-state); whether the deployment crosses a two-party-consent line for ambient capture; whether minors can reach the tool and the age-gate mechanism if any; the published AI-identity-disclosure cadence per California AB 489 / Connecticut SB 5 / Tennessee impersonation prohibition / federal SAFE BOTs framework / state chatbot-bill wave; the published licensure-impersonation prohibition posture; whether the deployment is subject to CMS-0057-F AI-decision-reasoning disclosure (impacted MA, Medicaid FFS and managed care, CHIP, FFE-QHP payers); whether the deployment will be reportable under Maryland HB 1563 quarterly insurer reporting (effective June 1, 2026) or any analogous state law; whether the deployment touches a Medicaid managed care PA workflow subject to the MACPAC June 2026 framework.

9. **Clinical-evidence package and benchmark posture.** External published evidence (RCT, prospective cohort, retrospective validation, simulation, benchmark leaderboard, vendor white paper); internal pilot evidence (chart-level accuracy bake-off, time-savings study, denial-rate study, error-mode classification, equity-stratified performance); known relevant external benchmarks (CHI-Bench long-horizon healthcare workflow benchmark, HealthBench Professional, ARISE State of Clinical AI Report 2026, Harvard / Beth Israel ER o1 study, Ontario Auditor General multi-vendor bake-off, the head-to-head DAX-vs-Nabla pragmatic RCT); known relevant published vulnerability research (Mindgard Doctronic findings for patient-facing chatbots).
10. **Data flow and minimum-necessary posture.** PHI in scope (identifiers, dates, audio capture, free-text notes, structured fields, images, claims, eligibility); data flow diagram (capture → vendor server → processing → storage → retention → deletion); minimum-necessary justification per 45 CFR §164.502(b); BAA scope; data residency (U.S., specific state, cloud region); whether de-identification is in scope (45 CFR §164.514 Safe Harbor or Expert Determination); whether re-identification controls apply; whether free-text training-data use is contractually permitted or excluded; retention and deletion policy.
11. **Bias, fairness, and equity evidence.** Whether the vendor disclosed subgroup performance across age, language, race / ethnicity, sex, disability, payer mix; the deployer's planned paired-demographic re-audit cadence; any known equity-shaped error mode the audit-class skills have already documented for the tool class (e.g., the Elsevier 2026 Nurses Edition AI-adoption gap signal, the Ontario AG mental-health-context omission signal); whether the population mix the tool will serve materially differs from the population mix the vendor's evidence was generated on.
12. **Workforce education, training, and de-skilling posture.** Vendor's published training package; deployer's planned training cadence (initial, refresher, role-based, scope-of-license-aware); nursing-specific training plan (responsive to the Elsevier 2026 Nurses Edition signal that 68% of nurses report insufficient AI training and 60% are not confident in their organization's AI governance); planned competency-verification approach; planned feedback-collection cadence; planned de-skilling-monitoring approach (CHAI Pillar 8 names de-skilling as a named risk class).
13. **Decommission criteria and exit posture.** What measurable conditions trigger pause / hold / decommission (performance threshold, equity threshold, incident threshold, regulatory event, vendor exit, model end-of-life, contract non-renewal); the patient-notification plan if patient-facing; the data-disposition plan; the chart-historical-record posture (notes generated by an ambient scribe remain part of the legal medical record even after the tool is decommissioned); the registry-update plan.
14. **Resource-scaling posture.** Whether the deployer is operating under a shared-services model, an FQHC / RHC / safety-net designation, a community-clinic consortium, or a small-group resource constraint; whether the intake should apply the "scaled control" tier from CHAI's South Carolina Primary Health Care Association guidance; what the deployer's realistic monitoring cadence is given staffing; what the deployer's realistic third-party-audit cadence is given budget.

## Instructions

Produce a structured governance intake record in the nine sections below. Keep risk ratings calibrated — not every gap is a blocker. Use the rating rubric below exactly. Default to the eight-pillar control set described below; the deployer may add or remove controls based on scope.

**Risk rating rubric (use verbatim):**

- `🟢 LOW` — Control is present, evidence is acceptable, or the gap is cosmetic and addressable in the current cycle. Note for the registry; no action blocks the lifecycle decision under review.
- `🟡 MEDIUM` — Control is partially present (e.g., posture documented but evidence thin; pillar present but pillar owner unnamed; pilot data present but population mix mismatched; vendor attestation present but not deployer-verified). Document the gap in the intake record, add the gap to the renewal-trigger list, and proceed with conditions named in Section 7.
- `🟠 HIGH` — Control is materially absent (e.g., no AI Tool Registry entry, no risk-and-impact assessment, no minimum-necessary justification, no clinician training plan, no incident-reporting path, no patient-notification posture for a patient-facing tool, no decommission criteria) or evidence is materially mismatched (e.g., vendor evidence is on an inpatient population but deployment is ambulatory; vendor evidence is English-only but population mix is significantly non-English). Remediate before the lifecycle decision under review proceeds; the intake should propose specific remediation language. Pilot-with-fence may be appropriate.
- `🔴 CRITICAL` — Control absence creates an active risk pathway (e.g., agentic-extension scope without HITL on a high-risk workflow; ambient capture in a two-party-consent jurisdiction without a per-encounter notification posture; AI-driven adverse-determination workflow without a documented human reviewer of appropriate expertise under MACPAC June 2026 / CMS-0057-F; a patient-facing chatbot without a `ai-chatbot-prompt-injection-audit.md` pre-deployment red-team battery; a tool whose vendor has published material zero-day vulnerabilities that the deployer cannot verify are patched). Do not proceed; escalate to the AI governance committee, the AI Use Officer, counsel, and the board AI subcommittee.

### 1. Intake Header

Intake date, intake authoring committee or AI Use Officer, decision under review (Section 3), tool identity (vendor / product / version / base model / deployment surface), proposed risk tier with reasoning, jurisdictions in scope, audience, service line(s) in scope, input completeness (full / partial / limited), and any items on the "Information still needed" list.

### 2. Decision Under Review & Posture Summary

Three to six sentences. State the specific decision the committee will make (approve / approve-with-conditions / pilot-with-fence / defer / decline / retire) and the recommended posture based on the eight-pillar review that follows. Explicitly call out the highest-risk pillar and the single most consequential evidence gap, if any. End with a single-line verdict:

```
RECOMMENDED DECISION: [APPROVE / APPROVE-WITH-CONDITIONS / PILOT-WITH-FENCE / DEFER / DECLINE / RETIRE]
```

If `DEFER` or `DECLINE`, Section 7 should focus on the specific remediation conditions; if `APPROVE-WITH-CONDITIONS` or `PILOT-WITH-FENCE`, Section 7 must name the conditions in measurable terms.

### 3. Eight-Pillar Control Review (CHAI-aligned)

For each pillar, produce one finding block. Use this structure verbatim:

```
[Pillar N — Pillar Name]
  Control posture: [present and evidence verified / present and evidence partial / partially present / absent / not applicable to this tool class]
  Evidence reviewed: [list — model card / red-team report / pilot data / training plan / contract clause / etc.]
  Gap(s) identified: [specific — paraphrased; no PHI]
  Risk: [🟢 / 🟡 / 🟠 / 🔴]
  Owner named: [role and named person, or "unassigned"]
  Remediation: [response-quality log only / committee follow-up / contract amendment / vendor attestation request / pilot-with-fence condition / pre-go-live blocker / pull-from-surface]
```

**The eight pillars (CHAI May 27, 2026 architecture):**

1. **Organizational AI Policy.** Does the deployer have a published AI policy, is the tool in scope of that policy, and does the tool's intended use align with the policy's named in-scope and out-of-scope use cases? Confirm posture against the deployer's published patient-AI-use-disclosure scaffold (cross-reference `patient-ai-use-disclosure-notice.md`), the deployer's published prohibition on AI implying clinician licensure (California AB 489, Tennessee impersonation prohibition, AMA April 22 Congressional letter), and the deployer's named position on agentic-extension authority.
2. **Organizational Structure.** Is there a named AI governance committee with documented charter, meeting cadence, decision authority, and reporting line into the board AI subcommittee? Is there a named AI Use Officer, CMIO, CNIO, CCIO, Privacy Officer, Security Officer, and contract owner for this tool? Are the lines of accountability for this tool documented per the CHAI Pillar 2 architecture?
3. **Organizational Resources.** Is there documented budget, staffing, and technical capacity to monitor the tool through its lifecycle? For LLM-based tools, is there capacity to run periodic re-audits per the audit-class skills in this repo? For agentic-extension tools, is there capacity to apply the CHI-Bench-calibrated reviewer-cycle cadence captured in `ambient-scribe-note-audit.md` v1.4 and `prior-auth-letter-generator.md` v2.4? For community-health and safety-net deployers, apply the scaled-control tier — what is the realistic monitoring cadence given staffing?
4. **Responsible AI Lifecycle Management.** Is the tool tracked in the AI Tool Registry from intake through decommission? Are the lifecycle stages documented (pre-pilot → pilot → enterprise → renewal → decommission)? Are change-control events (vendor model update, system-prompt revision, retrieval-index update, safety-classifier retrain, hard-stop-list update, agentic-scope expansion) tracked and re-intake-triggered? Are decommission criteria named in measurable terms (Section 8)?
5. **Risk and Impact Assessments.** Is there a pre-deployment risk-and-impact assessment that names the named-population, named-equity-sensitive-workflow, and named-error-mode risks specific to this tool? Is there a planned post-deployment re-assessment cadence tied to incident triggers and renewal? For LLM-based tools, has the assessment incorporated the CHI-Bench reliability ceiling, the Ontario Auditor General multi-vendor error-mode signal, and the Mindgard / Doctronic patient-facing-chatbot vulnerability research where applicable? For agentic extensions, does the assessment treat the agentic class as a distinct higher-risk category per `ambient-scribe-note-audit.md` v1.4?
6. **Responsible Data Management and Use.** Is the BAA scope verified against the actual data flow? Is the minimum-necessary justification documented? Is the retention and deletion policy named and contractually enforced? For ambient capture, is the recording-boundary diagram documented and is the per-encounter notification posture defined for two-party-consent jurisdictions (cross-reference `ambient-scribe-note-audit.md` v1.4 patient-consent section)? Is the data-residency named and consistent with state and federal posture (CMIA, CIPA, Federal Wiretap Act, TX HB 300, NY SHIELD, IL BIPA, WA My Health My Data Act, CO HB24-1054, 42 CFR Part 2)? Is the training-data-use posture (whether free-text training-data use is contractually permitted or excluded) documented?
7. **Third-Party Management.** Is the vendor's red-team report verified by the deployer where verifiable? Is the model card current? Is the OWASP 2025 LLM Top-10 attestation present for LLM-based tools? Is the NIST AI RMF / ISO 42001 posture documented? Is the HITRUST / SOC 2 / ISO 27001 status current? Is the third-party penetration-test summary current? Is the patient-facing-chatbot pre-deployment red-team battery (per `ai-chatbot-prompt-injection-audit.md`) on file for any patient-facing surface? Is the agentic-extension contract scope explicit about HITL? Is the contract amenable to a pull-from-surface remediation path? For Maryland-effective and analogous-state deployments, does the vendor cooperation clause cover the quarterly insurer reporting obligation under HB 1563?
8. **Education, Training, and Feedback.** Is there a documented training plan for clinicians and operational owners, scoped to role and scope-of-license? Is there a nursing-specific track responsive to the Elsevier 2026 Nurses Edition signal (68% insufficient training, 60% not confident in governance, 41% AI use vs 57% physicians)? Is there a feedback-collection mechanism that closes the loop to the AI governance committee? Is there a de-skilling-monitoring approach? Is there a documented patient-education pathway for any patient-facing surface (cross-reference `patient-education-handout.md` and `patient-ai-use-disclosure-notice.md`)?

### 4. Cross-Skill Integration Map

Identify which existing skills in the repo are the operational complements to this intake and how the intake feeds them.

- **Ambient scribe.** Cross-reference `ambient-scribe-note-audit.md` v1.4 — feed the procurement-phase consent-posture checklist, the per-encounter audit-header notification field, the long-horizon agent reliability calibration, and the vendor selection audit checklist into the intake under Pillar 6 and Pillar 7.
- **Clinician-facing copilot.** Cross-reference `clinical-ai-copilot-response-audit.md` — feed the response-fidelity audit cadence, the HITL pattern documentation, and the scope-adherence audit into the intake under Pillar 4 and Pillar 7.
- **Patient-facing chatbot.** Cross-reference `ai-chatbot-prompt-injection-audit.md` — feed the pre-deployment red-team battery, the attack-pattern library, and the verdict (HARDENED / RESISTANT / EXPLOITABLE / SYSTEM-LEVEL FAILURE) into the intake under Pillar 5 and Pillar 7. For any behavioral-health-adjacent surface, also cross-reference `behavioral-health-ai-chatbot-compliance-review.md`.
- **Patient-AI-use disclosure.** Cross-reference `patient-ai-use-disclosure-notice.md` — feed the disclosure language posture, the AI-identity-disclosure cadence, and the population-tailored disclosure version into the intake under Pillar 1, Pillar 6, and Pillar 8.
- **Prior-authorization and denial.** Cross-reference `prior-auth-letter-generator.md` v2.4, `wiser-medicare-prior-auth-prep.md`, `denial-appeal-letter-writer.md`, and `payer-downcoding-rebuttal-letter.md` — feed the CMS-0057-F disclosure posture, the MACPAC June 2026 framework, the CHI-Bench prior-auth reliability calibration, and any state-specific obligation (Maryland HB 1563, Vermont HB 814) into the intake under Pillar 1 and Pillar 5.
- **Policy and compliance.** Cross-reference `policy-and-compliance-qanda.md` — feed the multi-state regulatory matrix and the source-hierarchy posture into the intake under Pillar 1.
- **Nursing handoff and nursing-specific surfaces.** Cross-reference `nurse-shift-handoff-isbarr.md` and the v1.1 Nursing-Specific Bias & HITL extension of `ambient-scribe-note-audit.md` — feed the nursing-side training, governance-confidence, and decision-representation evidence (Elsevier 2026 Nurses Edition) into the intake under Pillar 2 and Pillar 8.
- **Population-specific.** Cross-reference `sdoh-risk-assessment-summarizer.md`, `hedis-care-gap-chart-abstractor.md`, and the population-tailored handouts when the tool's population mix differs materially from the vendor's evidence base.

### 5. Multi-State Regulatory Mapping

Produce a one-row-per-jurisdiction table covering the deployment surface. Each row names the jurisdiction, the operative law(s), the operative obligation, the deployer's posture, and a `🟢 / 🟡 / 🟠 / 🔴` risk rating.

**Default jurisdictions and obligations to evaluate when relevant:**

- **Federal — CMS-0057-F.** AI-decision-reasoning disclosure on prior-auth denials for impacted MA, Medicaid FFS and managed care, CHIP, FFE-QHP payers (in force since January 1, 2026).
- **Federal — HHS AI Strategy.** Risk-management posture (April 3, 2026 deadline) and the December 2025 RFI cycle (closed February 23, 2026).
- **Federal — HTI-5 (proposed).** AI / predictive DSI model-card source-attribution disclosure retraction (proposed; not yet final). The intake should note that the federal certification track is contracting at the same time the state disclosure track is expanding.
- **Federal — FDA 2026 CDS and wearables guidance (January 6, 2026).** Enforcement discretion for non-device CDS with independently reviewable logic; QMSR / ISO 13485:2016 alignment.
- **Federal — SAFE BOTs framework.** AI-identity disclosure cadence, licensed-clinician impersonation prohibition, minor-specific protections, mandatory crisis-resource referrals.
- **Federal — MACPAC June 2026.** Recommended procedural-defect appeal framing for algorithmic-only adverse Medicaid determinations.
- **California.** AB 489 (prohibition on AI implying clinician licensure), CMIA, CIPA, Federal Wiretap Act exposure for ambient capture without consent (cross-reference the Sutter / MemorialCare and parallel class actions).
- **Texas.** TRAIGA AI disclosure posture; TX HB 300 medical privacy.
- **Colorado.** HB26-1139 AI disclosure; HB24-1054.
- **Connecticut.** SB 5 (chatbot AI-identity disclosure, crisis-referral, minor-specific provisions; effective January 1, 2027 for companion-chatbot provisions; awaiting signature as of the May 25, 2026 monitor pass).
- **Vermont.** HB 814 (neurological rights + AI in health services; restricts insurer AI in claim denial), HB 816 (AI in mental health) — Senate advancing.
- **Utah.** AI PA disclosure posture; the Office of AI Policy / Medical Licensing Board jurisdictional posture surfaced by the Doctronic showdown.
- **Tennessee.** 2026 impersonation prohibition.
- **Maryland.** HB 1563 (effective June 1, 2026 — quarterly insurer reporting on number of adverse decisions, type of service involved, and whether AI was utilized; expanded Insurance Commissioner investigation authority for significant increases in adverse determinations, particularly emergency-department denials) and Maryland's October 1, 2025 baseline AI utilization-management law (which requires AI-driven utilization decisions to consider the patient's entire clinical picture).
- **Georgia.** SB 544 (effective January 1, 2027 — permits insurer AI use in PA; surfaces the multi-state regulatory divergence relative to Maryland and Vermont).
- **Two-party-consent jurisdictions for ambient capture.** California, Florida, Illinois, Maryland, Massachusetts, Montana, New Hampshire, Pennsylvania, Washington.
- **42 CFR Part 2.** SUD-treatment record segregation in any tool that touches SUD-treatment records.
- **State HIE-specific provisions.** Where applicable.

For each jurisdiction in scope, name the posture and rating. Where the deployer has not yet established a posture (e.g., new state, post-deployment regulatory event), flag as a `🟠 HIGH` until posture is documented.

### 6. Evidence Verification & Gap List

For each of the eight pillars, list the specific evidence the deployer reviewed, the specific evidence the deployer requested but did not receive, and the specific evidence the deployer plans to generate (internal pilot, paired-demographic re-audit, post-deployment audit per the audit-class skills). Surface the gap list as a numbered remediation queue with owner, target date, and gating-vs-non-gating designation. For LLM-based tools, the evidence list should explicitly include the deployer's stated position on the CHI-Bench reliability ceiling (does the deployer accept the ~28% strict-pass ceiling as the operating baseline, or does the deployer claim a higher reliability ceiling — and on what evidence?). For patient-facing surfaces, the evidence list should explicitly include the pre-deployment prompt-injection battery per `ai-chatbot-prompt-injection-audit.md`.

### 7. Conditions, Fences, and Renewal Triggers

If the recommended decision is `APPROVE-WITH-CONDITIONS` or `PILOT-WITH-FENCE`, name the conditions in measurable terms. Each condition should name (a) the measurable threshold or event, (b) the named owner, (c) the audit cadence or detection mechanism, (d) the consequence if the threshold is breached (re-intake / pause / decommission). Typical fences for 2026 deployments:

- **Population fence.** Pilot is limited to a named patient population (e.g., adult ambulatory; English-only first wave; primary-care first wave; non-behavioral-health first wave) until paired-demographic re-audit is complete.
- **Workflow fence.** Tool is limited to read-only / draft-with-HITL / non-agentic scope until the CHI-Bench-calibrated reviewer-cycle cadence is established and the agentic-extension audit class is built out.
- **Volume fence.** Tool is limited to a named caseload-per-shift threshold to avoid the late-session endurance collapse documented by CHI-Bench (<4% pass at 25 cases per session in the worst-performing configurations).
- **Vendor-change fence.** Any vendor change of base model, system prompt, retrieval index, safety classifier, hard-stop list, or agentic-extension scope triggers re-intake before the change reaches patients.
- **Equity fence.** Tool is paused if subgroup performance drift exceeds a named threshold, or if a paired-demographic re-audit surfaces a population-blind error mode.
- **Consent fence.** For ambient capture in a two-party-consent jurisdiction, tool is paused if the per-encounter notification posture cannot be evidenced at the audit-header level.
- **Regulatory fence.** Tool is re-intaked on any of: a new state law taking effect (Maryland HB 1563, Connecticut SB 5, Vermont HB 814/816 if signed), a CMS subregulatory action on the MACPAC June 2026 framework, a finalized HTI-5 rule, an FDA enforcement action on the tool class, a Joint Commission Responsible Use of AI certification rule update, a CHAI playbook update.

Each named condition becomes a row in the AI Tool Registry and a trigger in the renewal cadence.

### 8. Decommission Criteria and Exit Posture

Name the specific measurable conditions that trigger pause / hold / decommission. Typical criteria for 2026 deployments:

- **Performance failure.** Named threshold for accuracy, fairness, drift, hallucination rate, error mode, or downstream-impact metric.
- **Equity failure.** Subgroup-performance threshold breach or paired-demographic re-audit failure.
- **Incident threshold.** Named threshold of patient-safety events, near-misses, PSO filings, or incident-triggered re-audits.
- **Regulatory event.** Loss of vendor accreditation, vendor BAA termination, vendor breach, OCR enforcement action, state AG action, FTC inquiry, malpractice claim involving the tool, board-mandated decommission.
- **Vendor exit / model end-of-life.** Vendor sunsets the product, vendor changes the base model in a way the contract does not permit, vendor cannot evidence ongoing red-team and safety-classifier maintenance.
- **Contract non-renewal.** Operative for any reason including budget, organizational direction, M&A.

For each criterion, name (a) the named owner for triggering the decommission decision, (b) the patient-notification plan if patient-facing, (c) the data-disposition plan (return / destroy / retain per the legal-medical-record obligation; notes generated by an ambient scribe remain part of the chart even after decommission), (d) the registry-update plan, (e) the clinician-notification plan, (f) the regulatory-reporting plan if any.

### 9. Audit Trail & Export Block

Produce a copy-paste-ready block for the AI Tool Registry, the contract file, the board AI subcommittee minutes, the Joint Commission Responsible Use of AI certification file, the AI-aware HIPAA risk-analysis matrix, and the vendor-management file. The block should include: tool identity; lifecycle stage; recommended decision; conditions and fences; renewal triggers; decommission criteria; named owners by pillar; cross-skill linkage list; multi-state regulatory mapping summary; evidence-gap remediation queue with target dates; the date of the next scheduled re-intake.

Close the intake with three explicit lines:

```
INTAKE RECORD ID: [registry ID — to be assigned by AI Tool Registry on commit]
NEXT RE-INTAKE DUE: [date — calculated from renewal cadence and any triggered re-intake]
INTAKE OWNER OF RECORD: [named AI Use Officer / CMIO / CNIO / CCIO]
```

## Worked Example (synthetic — fictional vendor, fictional product, fictional deployer)

**Tool:** "Vendor Alpha" (fictional) — Vista Ambient Scribe v3.2, foundation model "Atlas-2-Health-Tuned" (vendor-asserted; no third-party verification of fine-tune corpus), deployment surface: EHR-embedded sidecar within Epic, deployer of record: a multi-state community-health-network operating in California, Washington, and Texas, serving primary care and behavioral health, including a substantial Spanish-preferred population and a moderate dual-eligible share. BAA in place. Lifecycle stage: pilot-to-enterprise expansion (12-month pilot at three primary-care sites is complete; expansion would add 11 more sites including the behavioral-health service line). Audience: clinician-facing for documentation; patients are in scope for active per-encounter notification.

**Intake header.** Intake date 2026-06-01, AI Use Officer (named), decision under review: pilot-to-enterprise expansion. Proposed risk tier: high-risk clinical (behavioral-health expansion + Spanish-preferred population mix). Jurisdictions: CA, WA, TX. Service lines: primary care, behavioral health. Input completeness: partial — vendor model card and red-team report on file; OWASP 2025 LLM Top-10 attestation absent; Spanish-language performance evidence absent; agentic-extension scope undefined.

**Decision under review & posture summary.** The committee will decide whether to expand the pilot from three primary-care sites to fourteen sites including behavioral health. The pilot evidence supports primary-care expansion under conditions but does not support behavioral-health expansion at this time. The single most consequential evidence gap is Spanish-language performance evidence; the highest-risk pillar is Pillar 6 (responsible data management and use) due to the two-party-consent posture for California and Washington capture. Recommended: **PILOT-WITH-FENCE** — expand primary care to all fourteen sites under fences; defer behavioral-health expansion pending paired-demographic re-audit and a `behavioral-health-ai-chatbot-compliance-review.md`-aligned policy review of the audio-capture posture for behavioral-health encounters.

```
RECOMMENDED DECISION: PILOT-WITH-FENCE
```

**Eight-pillar review (abbreviated for the worked example):**

```
[Pillar 1 — Organizational AI Policy]
  Control posture: present and evidence partial
  Evidence reviewed: published AI policy v3.1, AB 489-aligned licensure-impersonation prohibition, patient-AI-use-disclosure scaffold
  Gap(s) identified: policy is silent on agentic-extension authority; policy is silent on behavioral-health audio-capture posture
  Risk: 🟡 MEDIUM
  Owner named: AI Use Officer (named)
  Remediation: committee follow-up — policy v3.2 to add agentic-extension authority and behavioral-health audio-capture posture before behavioral-health expansion is reconsidered

[Pillar 2 — Organizational Structure]
  Control posture: present and evidence verified
  Evidence reviewed: AI governance committee charter, monthly cadence, named AI Use Officer / CMIO / CNIO, board AI subcommittee reporting line
  Gap(s) identified: nursing voice on committee is single-seat; behavioral-health voice on committee is absent
  Risk: 🟡 MEDIUM
  Owner named: AI Use Officer (named)
  Remediation: committee follow-up — add nursing-leadership second seat and behavioral-health seat before behavioral-health expansion

[Pillar 3 — Organizational Resources]
  Control posture: partially present
  Evidence reviewed: pilot QA budget; monitoring staffing at 0.4 FTE
  Gap(s) identified: 0.4 FTE cannot sustain the post-deployment audit cadence the `ambient-scribe-note-audit.md` v1.4 procurement-phase + post-deployment combined cadence implies for fourteen sites
  Risk: 🟠 HIGH
  Owner named: CFO (named) + AI Use Officer (named)
  Remediation: pilot-with-fence condition — budget to 1.0 FTE monitoring before expansion goes live at all fourteen sites

[Pillar 4 — Responsible AI Lifecycle Management]
  Control posture: present and evidence verified
  Evidence reviewed: AI Tool Registry entry, change-control log shows three vendor system-prompt revisions during the pilot
  Gap(s) identified: decommission criteria documented but not measurable in performance threshold terms
  Risk: 🟡 MEDIUM
  Owner named: AI Use Officer (named)
  Remediation: committee follow-up — Section 8 decommission criteria expanded to measurable terms before expansion go-live

[Pillar 5 — Risk and Impact Assessments]
  Control posture: present and evidence partial
  Evidence reviewed: pre-deployment RIA, post-pilot RIA refresh
  Gap(s) identified: RIA does not incorporate the CHI-Bench reliability ceiling for any agentic-extension; RIA does not incorporate the Ontario AG mental-health-context omission signal for the behavioral-health expansion; paired-demographic re-audit for Spanish-preferred population has not been run
  Risk: 🟠 HIGH
  Owner named: CMIO (named)
  Remediation: pilot-with-fence condition — Spanish-language paired-demographic re-audit before expansion includes Spanish-preferred patients; behavioral-health expansion deferred until Ontario-AG-aligned mental-health-context audit complete

[Pillar 6 — Responsible Data Management and Use]
  Control posture: partially present
  Evidence reviewed: BAA on file; minimum-necessary justification documented; retention policy at 30 days post-summary-verification
  Gap(s) identified: recording-boundary diagram does not address the after-visit waiting-room ambient capture risk; per-encounter notification posture for California and Washington capture is verbal-only with no audit-header field; training-data-use clause permits free-text training use by default (opt-out not asserted)
  Risk: 🟠 HIGH
  Owner named: Privacy Officer (named) + contract owner (named)
  Remediation: pilot-with-fence condition — contract amendment to assert training-data opt-out; deployment of audit-header notification field per `ambient-scribe-note-audit.md` v1.4 before expansion; recording-boundary diagram update before expansion

[Pillar 7 — Third-Party Management]
  Control posture: partially present
  Evidence reviewed: vendor model card; vendor red-team report; vendor SOC 2; pen-test summary
  Gap(s) identified: OWASP 2025 LLM Top-10 attestation absent; vendor's published Spanish-language performance evidence absent; vendor's NIST AI RMF posture not documented; vendor's ISO 42001 posture not documented
  Risk: 🟠 HIGH
  Owner named: contract owner (named)
  Remediation: pilot-with-fence condition — OWASP attestation, Spanish-language performance evidence, NIST AI RMF posture, and ISO 42001 posture obtained before expansion go-live

[Pillar 8 — Education, Training, and Feedback]
  Control posture: present and evidence partial
  Evidence reviewed: clinician training package; competency-verification check at end of pilot
  Gap(s) identified: nursing-specific training track absent (Elsevier 2026 Nurses Edition signal); de-skilling-monitoring approach not documented; feedback-collection cadence informal
  Risk: 🟡 MEDIUM
  Owner named: CNIO (named)
  Remediation: committee follow-up — nursing-specific track and de-skilling-monitoring approach in place by 90-day post-go-live mark
```

**Cross-skill integration map.** The intake feeds: `ambient-scribe-note-audit.md` v1.4 (procurement-phase consent posture, long-horizon agent reliability calibration, vendor selection audit checklist, per-encounter notification audit-header field); `patient-ai-use-disclosure-notice.md` (Spanish-language disclosure version); `policy-and-compliance-qanda.md` (multi-state regulatory matrix); `behavioral-health-ai-chatbot-compliance-review.md` (deferred until behavioral-health expansion is reconsidered); `clinical-note-drafter.md`, `referral-summary-writer.md`, `discharge-summary-generator.md` (downstream-consumer skills).

**Multi-state regulatory mapping (summary).** California (AB 489 + CMIA + CIPA + Wiretap Act — `🟠 HIGH` until audit-header notification field deployed and training-data opt-out asserted); Washington (My Health My Data Act + two-party-consent — `🟠 HIGH` same conditions); Texas (TRAIGA + HB 300 — `🟡 MEDIUM` pending TRAIGA posture finalization). Federal CMS-0057-F is not in scope for this tool (ambient scribe, not payer-side PA). HTI-5 is in scope at the AI Tool Registry export level — `🟡 MEDIUM`.

**Evidence-gap remediation queue (top items).**

1. OWASP 2025 LLM Top-10 attestation — contract owner — by 2026-07-15 — gating
2. Spanish-language performance evidence — CMIO — by 2026-07-31 — gating for Spanish-preferred expansion
3. Audit-header notification field deployment — Privacy Officer — by 2026-07-15 — gating
4. Training-data opt-out contract amendment — contract owner — by 2026-07-15 — gating
5. Monitoring FTE expansion (0.4 → 1.0) — CFO — by 2026-08-01 — gating for fourteen-site go-live
6. Nursing-leadership second committee seat + behavioral-health seat — AI Use Officer — by 2026-07-31 — gating for behavioral-health expansion reconsideration

**Conditions, fences, and renewal triggers.** Population fence: behavioral-health expansion deferred pending Ontario-AG-aligned mental-health-context audit. Workflow fence: agentic-extension scope is read-only (no auto-accept; no write-back to chart without HITL). Volume fence: 35-visits-per-shift cap until late-session sampling weight evidence is published. Vendor-change fence: any base-model / system-prompt / retrieval-index / safety-classifier change triggers re-intake. Equity fence: paused if Spanish-preferred subgroup error rate exceeds the English-preferred subgroup error rate by more than a named threshold. Consent fence: paused if per-encounter notification audit-header field cannot be evidenced at the case level. Regulatory fence: re-intake on (a) Maryland HB 1563 quarterly reporting precedent reaching California or Washington, (b) Connecticut SB 5 signature triggering reciprocal state action, (c) HTI-5 finalization, (d) Joint Commission Responsible Use of AI certification publication, (e) CHAI playbook update.

**Decommission criteria.** Performance failure if monthly hallucination rate exceeds a named threshold for any service line; equity failure if paired-demographic re-audit surfaces a `🟠 HIGH` or `🔴 CRITICAL` finding; incident threshold of three patient-safety events in 90 days; regulatory event if vendor loses SOC 2 or BAA terminates; vendor exit if Atlas-2-Health-Tuned reaches model end-of-life. Owner: AI Use Officer. Patient-notification plan: portal message and AVS overlay to active patients within 30 days of decommission. Data-disposition: vendor returns or destroys raw audio per BAA; verified summaries remain part of the chart per legal-medical-record obligation. Registry-update plan: AI Tool Registry status changed to "decommissioned" with effective date and reason.

**Audit trail & export block.** [Copy-paste-ready block as described in Section 9, populated with the above content.]

```
INTAKE RECORD ID: [to be assigned by AI Tool Registry on commit]
NEXT RE-INTAKE DUE: 2026-12-01 (six-month re-intake; or sooner if any fence is triggered)
INTAKE OWNER OF RECORD: AI Use Officer (named)
```

## Disclosures, Limits, and Cross-References

This skill produces a structured intake record. It does not constitute legal advice, does not substitute for the deployer's Privacy Officer or counsel, does not substitute for the deployer's HIPAA risk analysis, does not substitute for the vendor's own attestations or red-team report, and does not substitute for the existing audit-class skills in this repo. The intake reflects the May 27, 2026 CHAI eight-pillar architecture, the forthcoming voluntary Joint Commission Responsible Use of AI certification track, the NIST AI Risk Management Framework, ISO 42001, the HHS AI Strategy (April 3, 2026 risk-management deadline), and the operative state and federal AI-in-healthcare regulatory perimeter as of June 1, 2026. The deployer is responsible for refreshing the regulatory mapping at each re-intake.

For the operational audits the intake feeds into, see `ambient-scribe-note-audit.md` (procurement-phase + post-deployment audit), `clinical-ai-copilot-response-audit.md` (clinician-facing copilot fidelity), `behavioral-health-ai-chatbot-compliance-review.md` (behavioral-health policy + clinical scope), `ai-chatbot-prompt-injection-audit.md` (patient-facing chatbot attack surface), `patient-ai-use-disclosure-notice.md` (patient notification scaffold), and `policy-and-compliance-qanda.md` (multi-state regulatory Q&A). For the downstream artifacts the intake protects, see the clinical-documentation skills (`clinical-note-drafter.md`, `discharge-summary-generator.md`, `referral-summary-writer.md`, `pre-visit-chart-summarizer.md`) and the patient-communication skills (`patient-portal-message-triage.md`, `patient-education-handout.md`).

External references: Coalition for Health AI (CHAI) governance playbooks (May 27, 2026); Joint Commission + CHAI Responsible Use of AI guidance (initial September 2025; in-depth playbooks May 27, 2026; voluntary AI certification forthcoming); NIST AI Risk Management Framework; ISO 42001; HHS AI Strategy and April 3, 2026 risk-management deadline; HHS RFI on AI clinical care (December 2025 — closed February 23, 2026); CMS-0057-F (in force January 1, 2026); CMS-0062-P (comment window open through June 15, 2026); FDA 2026 CDS and wearables guidance (January 6, 2026); FDA QMSR / ISO 13485:2016 alignment; HTI-5 proposed rule (December 22, 2025 — comment closed February 27, 2026); MACPAC June 2026 Report Medicaid PA automation framework; CHI-Bench long-horizon healthcare workflow benchmark; ARISE State of Clinical AI Report 2026; Ontario Auditor General multi-vendor ambient-scribe procurement audit (May 13, 2026); Elsevier Clinician of the Future 2026 Nurses Edition; NHS England ambient scribing guidance (March 31, 2026); California AB 489; Texas TRAIGA; Colorado HB26-1139 / HB24-1054; Connecticut SB 5; Vermont HB 814/816; Maryland HB 1563 (effective June 1, 2026); Maryland October 1, 2025 AI utilization-management law; Georgia SB 544 (effective January 1, 2027); Tennessee 2026 impersonation prohibition; Utah AI PA and Office of AI Policy posture; federal SAFE BOTs framework; OWASP 2025 LLM Top-10; 45 CFR §164.308(a)(1)(ii)(A) (HIPAA security risk analysis); 45 CFR §164.502(b) (minimum necessary); 42 CFR Part 2 (SUD-treatment records); AAN 2026 AI position statement; AMA April 22, 2026 Congressional letter; AHRQ AI in Healthcare Safety Program / PSO.

Attack-pattern, vulnerability, and incident references in this skill paraphrase and operationalize published material; they do not reproduce attack payloads verbatim, and they do not substitute for the vendor's own red-team report.
