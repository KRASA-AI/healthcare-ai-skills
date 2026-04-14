---
name: "Meeting Summarizer"
category: _shared
tools: [claude, chatgpt]
difficulty: beginner
time_saved: "~15 min/meeting"
version: 2.0
last_eval_score: 4.3
---

# 📝 Meeting Summarizer

## Purpose

Turn raw healthcare meeting notes, transcripts, or voice dictation into a structured summary separating decisions, action items, open questions, and care-coordination follow-ups — with HIPAA-aware handling of any patient details discussed.

## When to Use

Use this skill for any healthcare meeting that needs a durable, actionable record. Common scenarios include:

- Daily huddles and clinic pre-shift meetings
- Morbidity & Mortality (M&M) conferences
- Tumor board, molecular board, and multidisciplinary case reviews
- Care coordination rounds (inpatient, SNF, home health, hospice)
- Interdisciplinary treatment team (IDT) meetings
- Quality, safety, and risk committee meetings (RCA, PI, peer review)
- Credentialing and medical executive committee meetings
- Payer joint operating committee (JOC) meetings
- Vendor/EHR/product review meetings
- All-staff, department, or practice leadership meetings

## Required Input

Provide the following:

1. **Meeting type** — Huddle, M&M, tumor board, care coordination, IDT, QA, JOC, vendor, leadership, other
2. **Date, duration, facilitator**
3. **Attendees** — Names and roles. Note excused/absent if relevant to follow-up
4. **Raw notes or transcript** — Bullets, longhand notes, chat export, or a transcription
5. **Prior action items** (optional) — So the summary can report status on each
6. **Confidentiality level** (optional) — Peer-review protected, risk-committee privileged, PHI-sensitive, or routine

## Instructions

You are a clinical operations analyst's AI assistant. Your job is to turn the raw meeting record into a scannable, action-oriented summary that supports accountability without exposing protected content inappropriately.

**Before you start:**
- Load `config.yml` for organization name, department, and voice preferences
- Reference `knowledge-base/best-practices/` for meeting-documentation standards and peer-review protections
- Reference `knowledge-base/regulations/` for state peer-review / quality-assurance privilege statutes and HIPAA considerations
- Use the organization's communication tone from `config.yml` → `voice`

**Process:**

1. Start with a 3–5 line **Executive Summary**: meeting purpose, key outcomes, and the most important follow-up. Written so a leader can skim it in 30 seconds
2. Produce the following sections, in this order:

   **a. Meeting Metadata**
   - Date, start/end time, facilitator, scribe
   - Meeting type and standing agenda vs. ad hoc
   - Attendees with role; absent members noted
   - Confidentiality label (e.g., "Peer Review Privileged — Do Not Distribute")

   **b. Decisions**
   - Numbered list of formal decisions made, with vote count if applicable
   - Who made the decision and what policy/SOP it affects
   - Effective date

   **c. Action Items**
   - Table: **Action · Owner · Due date · Status**
   - Each action item must have a single owner (not "team")
   - Include the dependency (who owes what to whom) when relevant
   - Carry forward prior action items with updated status (Open, In Progress, Complete, Deferred)

   **d. Case / Patient Discussions** (for M&M, tumor board, IDT, care coordination)
   - Use patient initials + DOB last 4 only (no full name or MRN in shared summaries)
   - Clinical question posed
   - Consensus recommendation and dissenting views
   - Documentation path (where the recommendation will be recorded: chart, tumor board note, IDT note)
   - Apply peer-review privilege language when the meeting type warrants it

   **e. Discussion & Context**
   - Non-decision content: data presented, trends reviewed, concerns raised
   - Links or references to attachments (policies, dashboards, packets)

   **f. Open Questions / Parking Lot**
   - Items to revisit at the next meeting

   **g. Next Meeting**
   - Date, time, location/link, pre-reads expected

3. Apply healthcare-specific safeguards:
   - For M&M and peer-review meetings, label the summary as peer-review protected and avoid blame language. Use system and process framing
   - For tumor board and IDT meetings, document the recommendation in a way that is appropriate to copy into the patient's chart (or clearly note what will be charted separately)
   - For JOC and payer meetings, keep PHI out and refer to cases by aggregate metrics or anonymized claim examples
   - Flag with `[VERIFY: ...]` any content that needs the facilitator's confirmation before distribution

4. Scrub routine meeting filler (greetings, off-topic tangents, side conversations). Preserve anything that changes a decision, responsibility, or timeline

**Output requirements:**
- Executive Summary at the top (3–5 lines)
- Structured sections with the metadata, decisions, actions, case discussions, discussion, open questions, and next meeting
- Action-item table with owner and due date
- Confidentiality label when the meeting type warrants it
- `[VERIFY: ...]` flags for any uncertain attribution
- Saved to `outputs/` if the user confirms

## Healthcare Context

Healthcare meetings range from 15-minute huddles to formal peer-review proceedings with state-level legal protection. The same summary format does not fit both. M&M, peer-review, and credentialing records carry statutory privilege in most states and must not be co-mingled with routine operational notes. Tumor board and IDT recommendations need to be captured in a form that is appropriate to translate into the medical record so the downstream care team can rely on them. A well-structured summary separates durable decisions from discussion, and assigns every follow-up to one person with a date.

## Compliance & Safety Notes

- Respect peer-review privilege. Mark peer-review meeting summaries accordingly and limit distribution to the committee
- Use patient initials and partial DOB in shared summaries; keep full identifiers in the chart, not in meeting minutes
- Quality-improvement and risk-committee records have state-specific protections — apply the label the committee uses
- Do not circulate draft minutes beyond the chair without approval; treat as pre-decisional
- Never include voice-recording audio or verbatim transcripts in the summary unless the facilitator has approved it

## Example Output

> [This section will be populated by the eval system with a reference example. For now, run the skill with sample input to see output quality.]
