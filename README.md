# Healthcare AI Skills

**Free, open-source AI prompts and workflows built for healthcare professionals.** Clone this repo, point your AI assistant at it, and start saving hours every week.

> Built and maintained by [KRASA AI](https://krasa.ai) — free AI tutorials and skills for every industry.
> See all industries at [krasa.ai/industries](https://krasa.ai/industries).

---

## What's Inside

This repo is a complete AI toolkit for healthcare. Every skill is a standalone prompt file that works with **Claude, ChatGPT, or any major AI assistant** — no coding required.

| Skill | What it does | Time saved |
|-------|-------------|------------|
| Clinical Note Drafter | Transform dictation, bullet points, or free-text encounter details into a structured clinical note (SOAP, H&P, progress note, or procedure note) ready for provider review and chart signature. | ~15 min/note |
| Discharge Summary Generator | Transform clinical encounter data, hospital course notes, and treatment records into a structured, comprehensive discharge summary ready for the medical record and care-transition handoff. | ~20 min/summary |
| Medication Reconciliation Assistant | Compare a patient's medication lists across care-transition points (admission, transfer, discharge, primary care follow-up, specialty visit) and produce a reconciled, structured medication list with changes, discrepancies, therapeutic duplications, interaction concerns, and high-priority clarifying questions — reducing the chance that an adverse drug event slips through a care transition. | ~15 min/reconciliation |
| Nurse Shift Handoff (I-SBARR) Generator | Turn a nurse's working patient data — recent vitals, assessments, drips, pending orders, labs, and open safety concerns — into a complete, standardized Introduction–Situation–Background–Assessment–Recommendation–Readback (I-SBARR) handoff that the outgoing RN can review, edit, and verbally deliver to the incoming RN in under two minutes per patient. | ~6 min/patient handoff |
| Pre-Visit Chart Summarizer | Synthesize a patient's medical record into a concise, actionable summary that a clinician can review in under two minutes before walking into the exam room. | ~10 min/patient |
| Referral Summary Writer | Compile a patient's relevant history, clinical findings, workup results, and specific consultation questions into a concise, well-organized referral letter that gives the receiving specialist everything they need to prepare for the consultation. | ~10 min/referral |
| SDOH Risk Assessment Summarizer | Turn a completed Social Determinants of Health (SDOH) screening — the G0136 standardized risk assessment or equivalent — plus relevant chart context into a concise, actionable SDOH summary that a clinician, care manager, or community health worker can use at the point of care. | ~10 min/encounter |
| Patient AI-Use Disclosure Notice | Draft a plain-language patient-facing disclosure that names *which* AI systems the practice uses in the patient's care, *when* and *how* the patient will encounter them, *what the patient can opt out of*, and *who to contact* with questions or to revoke consent. | ~25 min/notice |
| Patient Education Handout | Turn a clinician's plan — a new diagnosis, a procedure, a medication, a self-care regimen, a lifestyle change — into a one- to two-page patient handout that is readable at a 6th–8th grade level, health-literacy-audited, culturally appropriate, teach-back-ready, and brand-consistent with the practice's after-visit-summary voice. | ~20 min/handout |
| Patient Portal Message Triage | Classify an inbound patient portal message (MyChart, Athena, eClinicalWorks, Epic MyChart, NextGen, or similar), route it to the correct queue, detect clinical urgency, surface the minimum context a human responder needs, and draft a reply the clinician or staff can verify and send in seconds. | ~4 min/message |
| AI Health Chatbot Prompt-Injection Pre-Deployment Audit | Run a structured red-team / prompt-injection test battery against a patient-facing AI health chatbot — a symptom-triage assistant, a portal-message responder, a refill-renewal bot, a behavioral-health intake bot, a hospital-branded patient assistant, a voice-first front-door agent, or any other conversational AI surface that talks directly to patients — and produce a risk-ranked finding report against the deployer's stated scope, the vendor's published guardrails, and the 2026 patient-facing chatbot threat surface. | ~25 min/audit |
| Ambient Scribe Note & Coding Audit | Audit an ambient-AI-drafted clinician note against an auditable source (structured encounter data, clinician-corrected final note, or — where policy allows — the transcript/audio log) to flag note inflation, phantom diagnoses, over-stated HPI or ROS elements, E/M-level drift, HCC upcoding, and assessment/plan content that was not discussed in the encounter. | ~12 min/audit |
| Behavioral-Health AI Chatbot Compliance Review | Review the conversational flows, system prompts, knowledge-base snippets, escalation rules, and patient-facing copy of any AI chatbot that a behavioral-health practice, payer, EAP, school-based program, digital-therapeutic, wellness vendor, or hospital intends to deploy for patients, members, students, or the general public — and produce a structured compliance finding report that maps each element against the 2026 regulatory perimeter the AMA, state legislatures, FDA, and federal bill drafters have drawn around mental-health AI chatbots. | ~35 min/review |
| Clinical AI Copilot Response Audit | Audit one or more responses from a clinician-facing AI Q&A copilot — the conversational tool that lives inside the EHR or alongside it and answers a clinician's plain-language question about the chart by reading progress notes, hospital policy, orders, recent labs, and other chart-resident sources — and produce a structured finding report against the underlying chart and the deployer's stated scope. | ~18 min/audit |
| Coding Review Assistant | Review clinical documentation against ICD-10 (and emerging ICD-11) and CPT/HCPCS codes to identify under-coding, over-coding, mismatches, and missed opportunities — helping maximize appropriate reimbursement while maintaining compliance. | ~10 min/encounter |
| Denial Appeal Letter Writer | Draft a persuasive, evidence-based appeal letter in response to a payer claim denial, referencing clinical guidelines, medical necessity criteria, and patient-specific documentation to support overturning the denial. | ~30 min/letter |
| HEDIS Care Gap & Chart Abstraction Assistant | Review a patient chart against a list of HEDIS (or other quality program) measures to identify open care gaps, extract supporting documentation evidence for hybrid/ECDS submission, and flag potential numerator/denominator/exclusion hits so the quality team can close gaps or submit compliant chart evidence. | ~20 min/chart |
| Payer Downcoding Rebuttal Letter | Draft a code-specific, evidence-grounded rebuttal letter when a payer has *silently downcoded* a claim — most commonly an E/M level, an inpatient DRG, an observation-to-inpatient reclassification, or a procedure bundled without a modifier — rather than issuing a formal denial. | ~35 min/letter |
| Policy & Compliance Q&A | Answer staff questions about HIPAA, OSHA, CMS, state licensure, payer policy, and accreditation rules with a source-cited, confidence-labeled response a compliance officer can confidently relay to the team or escalate to counsel. | ~15 min/question |
| Prior Auth Letter Generator | Draft a comprehensive prior authorization request letter with clinical justification, supporting evidence, and payer-specific formatting to maximize the probability of first-pass approval. | ~25 min/letter |
| WISeR Medicare Prior Auth Prep | Assemble a WISeR-ready prior authorization or pre-payment review packet for a Traditional (Original) Medicare beneficiary who lives in a WISeR state and is scheduled for a service on the WISeR list. | ~30 min/submission |
| Email Drafter | Turn rough notes, voice dictation, or bullet points into a professional, HIPAA-aware healthcare email tailored to the recipient type (patient, referring provider, payer, vendor, staff) and the purpose of the message. | ~10 min/email |
| Meeting Summarizer | Turn raw healthcare meeting notes, transcripts, or voice dictation into a structured summary separating decisions, action items, open questions, and care-coordination follow-ups — with HIPAA-aware handling of any patient details discussed. | ~15 min/meeting |
| Review Responder | Craft a HIPAA-compliant, platform-appropriate response to an online healthcare review (positive, negative-clinical, negative-operational, or false/defamatory) that builds trust with prospective patients without acknowledging a provider–patient relationship or discussing any PHI. | ~10 min/review |

**Total time saved per use: ~425+ minutes across all skills.**

## Quick Start

### 1. Clone this repo

```bash
git clone https://github.com/KRASA-AI/healthcare-ai-skills.git
cd healthcare-ai-skills
```

### 2. Open a skill with your AI assistant

Open any file in `skills/` with Claude, ChatGPT, or any major AI assistant. Each skill is a self-contained prompt with clear instructions — no coding required.

The first time you use a skill, your AI assistant will ask for your business details (company name, service area, rates, tools you use, etc.) so it can personalize the output. Save those details to a `config.yml` at the repo root and every future skill will use them automatically.

## Repo Structure

```
healthcare-ai-skills/
├── knowledge-base/          # Industry context and references
│   ├── industry-overview.md # Market trends and pain points
│   ├── terminology/         # Industry jargon and acronyms
│   ├── regulations/         # Compliance requirements
│   ├── best-practices/      # Industry standards
│   └── tools-ecosystem/     # Common software and tools
├── skills/                  # The prompt library
│   ├── operations/          # Day-to-day operational skills
│   ├── sales/               # Sales and lead management
│   ├── admin/               # Administrative and compliance
│   └── customer-service/    # Client-facing communication
└── outputs/                 # Your generated content (gitignored)
```

## How Skills Work

Each skill file is a Markdown document with YAML frontmatter:

```markdown
---
name: Skill Name
category: operations
tools: [claude, chatgpt]
time_saved: "~20 min/use"
version: 1.0
---

# Skill Name

## Purpose
What this skill does and when to use it.

## Instructions
Step-by-step prompt for the AI assistant.
```

You open the file in your AI assistant, provide any required input (measurements, notes, client info), and get polished output. Skills reference your `config.yml` automatically for company name, rates, preferred formats, and other business details.

## For AI Assistants

If you are an AI assistant reading this repo, see `.claude/CLAUDE.md` for full instructions. The short version:

1. **Check for `config.yml`** at the repo root. If it exists, load it — it holds the user's business context (company name, rates, service area, tools, team size, etc.) and every skill should use it for personalization.
2. **If `config.yml` is missing**, before running a skill that benefits from personalization, ask the user for the relevant business details and offer to save them to `config.yml` so future runs are automatic.
3. **Load the relevant `knowledge-base/` files** for industry terminology, regulations, and best practices before generating output.
4. **Run the requested skill** from `skills/` using the user's input.
5. **Save any deliverables** to `outputs/` (gitignored) if the user wants to keep them.

## Learn More

- **Healthcare AI guide**: [krasa.ai/industries/healthcare](https://krasa.ai/industries/healthcare)
- **All industry AI skills**: [krasa.ai/industries](https://krasa.ai/industries)
- **About KRASA AI**: [krasa.ai](https://krasa.ai)

## License

MIT — use these skills however you want.
