---
name: "Patient Education Handout"
category: customer-service
tools: [claude, chatgpt]
difficulty: intermediate
time_saved: "~20 min/handout"
version: 2.3
last_eval_score: 9.20
---

# 📄 Patient Education Handout

## Purpose

Turn a clinician's plan — a new diagnosis, a procedure, a medication, a self-care regimen, a lifestyle change — into a one- to two-page patient handout that is readable at a 6th–8th grade level, health-literacy-audited, culturally appropriate, teach-back-ready, and brand-consistent with the practice's after-visit-summary voice. The output is ready for the clinician to review, sign, and hand to the patient at end of visit — or attach to the patient portal / MyChart / after-visit summary.

## When to Use

Use this skill in any outpatient, inpatient, ED-discharge, ASC, dental, or behavioral-health setting whenever the patient needs a take-home explanation of:

- A **new diagnosis** (hypertension, T2DM, COPD, depression, atrial fibrillation, iron-deficiency anemia, H. pylori, UTI, preeclampsia risk, early pregnancy, etc.)
- A **new medication or medication change** (dose, frequency, how to take, side effects, interactions, refill logistics, storage, safe disposal, SGLT2i euglycemic DKA, GLP-1 GI side effects, anticoagulant bleeding precautions, antibiotic completion, high-risk black-box warnings)
- A **procedure or test** (endoscopy, colonoscopy, MRI with contrast, stress test, biopsy, vaccination, CT) — pre-op prep, what to expect, recovery
- A **self-care regimen** (wound care, diabetic foot care, BP monitoring at home, peak flow diary, asthma action plan, glucose log, warfarin diet, DASH diet basics)
- A **post-ED or post-hospital discharge** handout that complements but does not replace the discharge summary
- A **preventive or screening** topic (colon cancer screening options, HPV vaccination for adults, mammography, lung-cancer LDCT screening, AAA screening)
- A **pediatric** topic where the handout is for the caregiver (fever management, hydration in gastroenteritis, safe sleep, asthma spacer technique)
- A **behavioral health** topic (SSRI start, sleep hygiene, cognitive-behavioral self-care, 988 resource)
- A **multilingual** version of an existing English handout, produced in the patient's preferred language at the same reading level

This skill produces patient-facing content. It does **not** replace the clinician's explanation, is not a substitute for informed consent, and is not standalone discharge instructions.

## Required Input

Provide the following:

1. **Topic** — The specific diagnosis, medication, procedure, or behavior. Be specific: "metformin extended release 500 mg starting dose for newly-diagnosed T2DM in an adult" is more useful than "diabetes handout."
2. **Audience** — Adult patient, adolescent, caregiver of a pediatric patient, caregiver of a cognitively impaired adult, pregnancy-specific, post-op, etc. Default audience is the adult patient at 6th–8th grade reading level.
3. **Preferred language** — English, Spanish, Simplified Chinese, Vietnamese, Arabic, Russian, Haitian Creole, Tagalog, or other. If the language is not one the skill can produce accurately, flag it for a certified translator and produce English with translation-ready formatting.
4. **Reading-level target** — Default is 6th–8th grade (Flesch-Kincaid 6–8). CMS and AHRQ guidance recommend ≤ 8th grade for patient-facing health materials. Cite the practice's documented standard if it differs (e.g., 5th grade for low-literacy populations).
5. **Clinical context** — Relevant comorbidities, medications, allergies, age, sex at birth, pregnancy status. The handout should not contradict the patient's actual regimen (e.g., warfarin + NSAID warning).
6. **Practice-specific content** — Clinician name, callback number, portal instructions, after-hours line, local pharmacy if specified, referral partners if named. Pulled from `config.yml` where available.
7. **Format target** — One-page summary, two-page handout, AVS add-on, portal message attachment, or fridge-magnet-style short card. Default is one page.
8. **Any practice-specific safety or legal language** — e.g., "Do not stop this medication without calling us first," "Call 911 for [symptoms]," state-specific 911/poison-control numbers, 988 for behavioral health.

If the topic requires specialty expertise the input does not supply (e.g., specific chemotherapy protocol, rare genetic condition, complex post-op instructions), the skill returns a **"Clinician input needed"** list rather than inventing clinical detail.

## Instructions

**Before you start (personalization from `config.yml`):**

Read these named hooks once. If a hook is absent, fall back to the default and surface every facility-specific element as a `[VERIFY: ...]` flag — never invent a clinician name, NPI, callback number, portal URL, after-hours policy, pharmacy partner, or facility-specific safety language.

- `practice_specialty` — drives default topic library and tone (primary_care default; pediatrics; OB/GYN; cardiology; oncology; behavioral_health; endocrinology; orthopedics; dermatology; ED-discharge). Specialty seeds the default topic templates (e.g., asthma action plan + spacer technique for pediatrics; OB postpartum warning signs + lactation block for OB; chemo cycle + neutropenic-precaution block for oncology; SI-screening + 988 framing for behavioral health).
- `reading_level_default` — `grade_6_8` (default per CMS / AHRQ / Joint Commission), `grade_5` for documented low-literacy populations, `grade_3_4` (caregiver-of-pediatric simplified). Apply Flesch-Kincaid check before delivery; replace Latinate words with shorter Anglo-Saxon equivalents.
- `numeracy_rules` — health *numeracy* is a distinct competency from reading level, and is the dimension patient materials most often fail even when the prose scores well. Apply these rules to every number in the handout: (a) use **absolute risk with a consistent natural-frequency denominator** ("about 3 out of 100 people" — keep the same denominator throughout a handout), not relative risk ("cuts your risk in half") and not bare percentages for low-numeracy audiences; (b) **never present relative risk alone** — it systematically inflates perceived benefit; (c) use **whole numbers**, avoid decimals and avoid fractions like ½ (write "1 out of 4," not "25%" or "0.25"); (d) give a **plain anchor for clinical numbers** so a value is interpretable ("Your A1c is 8. We want it under 7." / "A good home blood pressure is under 130 over 80"); (e) keep **dose math concrete and pre-computed** ("take 2 pills," "give 5 mL using the marked syringe" — never make the patient compute mg/kg); (f) put time in everyday terms ("every morning," "for 5 days") rather than "BID" or "q12h." Default to these rules even when `numeracy_rules` is absent from config.
- `language_preferences_supported` — keyed list of languages the practice has certified-translation coverage for (e.g., `en`, `es`, `vi`, `zh-Hans`, `ar`, `ru`, `ht`, `tl`). For supported languages, produce the handout at the same reading level. For unsupported languages OR for high-stakes content (oncology, informed consent, complex surgical prep), produce English with translation-ready formatting and route through `certified_translation_workflow` rather than shipping a machine translation.
- `clinician_signature_block` — keyed per signing clinician role (attending physician, NP, PA, RN-led education visit, MA-prepared handout that requires clinician sign-off) with name, credentials, NPI where applicable, and the practice's exact "Reviewed by" line wording.
- `practice_contact_block` — main phone, after-hours line or policy, patient-portal name + URL, secure-message instructions, address. Used in §8 (Your Team and Next Steps). Never invent.
- `crisis_line_overlay` — 911 default + 988 default + Poison Control 1-800-222-1222 default; appended state-specific or community-specific lines (e.g., state mobile-crisis line, IPV hotline, child-abuse hotline) when the topic warrants (behavioral health, pregnancy + IPV screening, pediatric).
- `practice_voice` — `warm_plain` (default for adult patient), `caregiver_first` (pediatric / cognitively-impaired-adult caregiver), `clinical_warm` (post-op / oncology / cardiology where precision matters), `motivational_neutral` (substance-use / behavioral health, non-judgmental). Drives sentence rhythm and analogies.
- `pharmacy_and_partner_block` — preferred local pharmacy, mail-order pharmacy, durable-medical-equipment partner, lab partner, imaging partner. Inserted when the handout references a specific service (e.g., glucose meter source, peak-flow-meter source).
- `patient_facing_disclaimer_block` — facility-specific text ("This handout is for learning. It does not replace your clinician's advice…") plus any state-specific consent / advance-directive / minor-consent / reproductive-health language the practice appends.
- `safety_critical_topic_gate` — topics that require explicit clinician-in-the-loop review BEFORE the handout leaves the practice (suicidal ideation, IPV, severe mental illness, complex pediatric dosing, oncology regimens, pregnancy-risk medications, controlled-substance taper, opioid-use-disorder bridging). Skill flags `[CLINICIAN MUST PERSONALIZE]` and refuses to ship without the flag cleared.
- `21cca_information_blocking_default` — flag indicating that handouts attached to the chart will be visible to the patient via the portal in near-real time per the 21st Century Cures Act information-blocking rules; tunes language to be plain-direct rather than chart-shorthand.
- `output_destination` — `outputs/`, `chart_attachment`, `portal_message_attachment`, `print_friendly_avs_addon`, `multilingual_pair` (English + supported translation), or `caregiver_packet`.
- `config_missing_behavior` — `flag_and_proceed` (default — ship a complete handout with `[VERIFY: ...]` flags on every facility-specific element) vs. `block_and_ask`.

When `config.yml` is absent entirely, ship a one-page primary-care handout at 6th–8th grade reading level in warm-plain voice with the AHRQ universal-precautions teach-back block, generic "Reviewed by: ___" sign-off, generic 911 / 988 / Poison Control crisis lines, and `[VERIFY: ...]` flags on every facility-specific element (clinician name, callback, after-hours policy, portal, pharmacy partner). Never invent a clinician name, NPI, callback number, portal URL, or pharmacy partner.

Follow this eight-section handout template. Every handout is written at the specified reading level, uses plain language, and is formatted for readability (short sentences, headings, white space, bullet points, bolded key actions). Avoid clinical jargon unless it is named and defined.

### 1. Title and One-Line Purpose

- Plain-language title: **"High Blood Pressure (Hypertension): What It Means and What You Can Do"** — not "Essential HTN Patient Education."
- One-line purpose under the title: **"A quick guide to help you understand your diagnosis and the plan we made today."**

### 2. What Is It? (2–4 short sentences)

Define the condition / medication / procedure in plain language, the way a clinician would explain it to a neighbor. Use analogies where they help (e.g., "Your blood pressure is how hard your blood pushes on your artery walls — like water pressure in a garden hose"). Name the condition and its common name in the same sentence.

Include one sentence on **why it matters for this patient** — not a generic population statement.

### 3. What to Expect (3–6 bullets)

Concrete, short bullets. Each bullet is something the patient will actually experience or do. Examples:

- "Your pharmacy will fill the prescription today. The medicine comes as a small white tablet."
- "Most people feel no difference for the first 1–2 weeks."
- "Some people have mild stomach upset at first. Taking it with food usually helps."

For procedures: step through pre-op, day-of, and recovery in the patient's voice.

For diagnoses: name what they will feel, what they will monitor, and when they will see the team again.

### 4. How to Care for It (Self-Care Steps)

The practical core. Numbered steps, each starting with an action verb. Example for a new metformin start:

1. Take 1 pill with your biggest meal each day for the first week.
2. Starting week 2, take 1 pill with breakfast and 1 pill with dinner.
3. Use a pill box or a reminder on your phone so you do not miss doses.
4. Bring your pill bottle to every visit.
5. Check your blood sugar as the clinic showed you; write the number down.

Include **dose, timing, route, duration, and refill plan** for medications. Include **diet, activity, monitoring, and equipment** for self-care plans. Include **wound/incision care, activity limits, and follow-up** for procedures.

### 5. Medication Safety (if a medication is involved)

A standalone block. Cover:

- **Side effects to expect** (minor, usually resolve) — short list.
- **Side effects to call us about** (moderate) — specific and actionable.
- **Side effects to call 911 about** (severe — anaphylaxis, severe bleeding, chest pain, stroke symptoms, severe allergic reaction, suicidal thoughts for applicable meds).
- **Interactions** — the ones actually relevant to this patient's medication list and this med class (e.g., metformin + contrast; SGLT2i + insulin + euglycemic DKA risk; anticoagulant + NSAIDs; SSRIs + triptans; opioids + benzodiazepines).
- **Food, alcohol, or grapefruit-juice interactions** if applicable.
- **Storage** (refrigeration, light, room temperature) — critical for insulin, GLP-1, biologics.
- **Safe disposal** — DEA take-back, drug deactivation pouch, or flush-list for high-risk controlled substances.
- **Pregnancy / breastfeeding** if applicable.
- **Do not stop suddenly** warning for taper-requiring meds (steroids, SSRIs, beta blockers, benzodiazepines).

### 6. When to Call Us / When to Go to the ER

A clearly separated **"Call us"** vs **"Go to the ER or call 911"** block. Use a simple two-column layout or two short boxes. Rules:

- "Call us" triggers are moderate-severity, practice-callable (new rash without shortness of breath, persistent nausea, missed doses, questions).
- "Go to the ER / call 911" triggers are red flags: chest pain, trouble breathing, stroke symptoms (facial droop, arm weakness, speech change, time to call — FAST), severe bleeding, fainting, severe allergic reaction, suicidal thoughts (include **988**).
- Never bury a life-threatening symptom in a bullet list. Give it its own line in bold.

### 7. Teach-Back Prompts

Three to five short questions the patient can use to check their own understanding, and the clinician can use to confirm teach-back at the visit. Examples:

- "Can you tell me in your own words what [condition] is?"
- "When will you take the medicine? What if you forget a dose?"
- "What would make you call us? What would make you call 911?"
- "What is the plan if the medicine upsets your stomach?"
- "When is your next visit?"

Teach-back is AHRQ-recommended universal-precautions practice; every handout includes prompts regardless of apparent literacy.

### 8. Your Team and Next Steps

- **Your clinician:** [Clinician Name], [Practice Name]
- **Office phone:** [Number]
- **After-hours:** [Number or policy]
- **Patient portal:** [Portal name and link or "MyChart"]
- **Next visit:** [Date / "we will call to schedule"]
- **Crisis/help:** 911 for emergencies; 988 for mental health crisis; poison control 1-800-222-1222.

### Format, Reading Level, and Accessibility Rules

- **Reading level:** 6th–8th grade by default. Verify with a Flesch-Kincaid check before delivery. Replace long or Latinate words with shorter Anglo-Saxon equivalents ("doctor" not "physician," "use" not "utilize," "help" not "facilitate," "long-lasting" not "chronic" with the clinical term parenthetically the first time).
- **Sentence length:** Target ≤ 15 words. Break up anything longer.
- **Numeracy (distinct from reading level):** Numbers are audited separately from prose. Use absolute risk with one consistent natural-frequency denominator ("3 out of 100"), never relative risk alone; use whole numbers, not decimals or percentages, for low-numeracy audiences; give every clinical number a plain interpretive anchor ("Your A1c is 8 — we want it under 7"); pre-compute all dose math so the patient never multiplies or divides; express timing in everyday words, not Latin abbreviations. See the `numeracy_rules` config hook above. A handout that passes Flesch-Kincaid but presents risk as "lowers your risk by 40%" or dosing as "1 mg/kg" has failed the numeracy check.
- **Structure:** Clear headings, short paragraphs (≤ 4 lines), bullets over prose for instructions, bold for action items, numbered lists for sequences.
- **Visuals (optional):** Suggest where a simple icon or diagram would help (e.g., inhaler-with-spacer technique, AAA screening anatomy). Do not generate images; name the visual the handout should include.
- **Font guidance for print:** ≥ 12 pt body, ≥ 14 pt headings, sans-serif, adequate white space, high contrast. This is an output-formatting hint, not a body-copy element.
- **Accessibility:** Provide large-print and audio alternatives on request. Section 504 and ADA considerations apply.
- **Tone:** Warm, specific, non-judgmental. Avoid shaming language around weight, adherence, substance use, or mental health.

### Multilingual and Cultural Rules

- **Translation:** If the patient's preferred language is supported, produce the handout at the same reading level as the English version. If not supported or the content is high-stakes (oncology, informed consent, complex surgical prep), flag for a certified medical translator rather than shipping an uncertified machine translation.
- **Cultural tailoring:** Use examples and analogies that fit the patient's likely context (e.g., dietary guidance that respects common cultural food patterns). Do not stereotype.
- **Family / caregiver framing:** For communities where family decision-making is common, explicitly invite the caregiver into the plan where clinically appropriate and the patient consents.
- **Plain language across languages:** Literal translation often reads above 8th grade. The non-English version must be authored for reading level, not word-for-word.

### Anti-Error Guardrails

- Never invent a dose, frequency, duration, indication, or side effect.
- Never recommend a non-FDA-approved indication as if it were standard care.
- Never contradict the clinician's stated plan; if the input is ambiguous, return "Clinician input needed" rather than guessing.
- Never generate a handout for a topic involving suicidal ideation, severe mental illness, complex pediatric dosing, oncology regimens, or pregnancy-risk medications without an explicit "clinician must personalize" flag.
- Never copy manufacturer-trademarked patient materials verbatim. Paraphrase and cite the brand name only where clinically necessary.
- Every handout ends with a clinician-review-and-sign line: **"Reviewed by: [Clinician Name]   Date: ______"**
- Every handout includes a disclaimer: **"This handout is for learning. It does not replace your clinician's advice. If your situation changes, call us or seek care."**

## Example Output

```
High Blood Pressure: What It Means and What You Can Do

A short guide from [Practice Name] to help you live well with high blood pressure.

What Is High Blood Pressure?
Blood pressure is how hard your blood pushes on the walls of your arteries.
When it stays too high for too long, it can hurt your heart, brain, and kidneys.
Most people with high blood pressure feel just fine — that is why we measure it.

What to Expect This Month
- We started you on lisinopril 10 mg, once each morning.
- You may feel no change. That is normal and good.
- A small number of people get a dry cough. Call us if you do.
- We will check your blood pressure again in 4 weeks.

How to Care for It
1. Take 1 lisinopril pill every morning, with or without food.
2. Check your blood pressure at home 2 times a week. Write the numbers down.
3. Bring the numbers and the pill bottle to every visit.
4. Try to walk 20–30 minutes most days.
5. Cut back on salty foods like chips, deli meat, and canned soup.

Medicine Safety
- Side effects we want to know about: dry cough, dizziness when you stand up,
  swelling of the lips or tongue.
- **Call 911** if your lips, tongue, or throat swell, or you have trouble
  breathing. That is a rare but serious reaction.
- Do not start new over-the-counter medicines without asking us — especially
  ibuprofen (Advil, Motrin) or naproxen (Aleve).
- Keep the pills at room temperature, out of reach of children.
- If you miss a dose, take it when you remember unless it is almost time for
  the next dose. Do not take two pills at once.

When to Call Us
- Dry cough that does not go away
- Home blood pressure above 160/100 two days in a row
- Feeling lightheaded when you stand up
- Any question about the medicine

Call 911 or Go to the ER
- Chest pain or pressure
- Trouble breathing
- Weakness on one side of your body or trouble speaking (stroke signs)
- Swelling of lips, tongue, or throat

Teach-Back — Please Answer in Your Own Words
- Why do we treat high blood pressure even when you feel fine?
- When do you take the pill? What do you do if you forget?
- What would make you call the office? What would make you call 911?

Your Team and Next Steps
- Your clinician: Dr. A. Romero, [Practice Name]
- Office phone: [Number]
- After-hours: [Number]
- Patient portal: [Name/Link]
- Next visit: 4 weeks — we will call to schedule

Crisis help: 911 for emergencies. 988 for mental health. Poison Control 1-800-222-1222.

Reviewed by: _________________   Date: __________

This handout is for learning. It does not replace your clinician's advice.
If your situation changes, call us or seek care.
```

### Second worked example — pediatric asthma action plan (caregiver-first voice, v2.2 addition)

The first example covers the highest-volume adult primary-care use case (new antihypertensive start, warm-plain voice, English-only). The second worked example below covers the highest-volume pediatric use case where the handout is for the **caregiver, not the patient** and where the safety stakes are concrete and immediate: an asthma action plan after a clinic visit for moderate persistent asthma in a 7-year-old. The example exercises a different combination of `config.yml` hooks (`practice_specialty=pediatrics`, `practice_voice=caregiver_first`, `reading_level_default=grade_5`, `language_preferences_supported` Spanish-paired output, `safety_critical_topic_gate` for the red-zone ED block, named visual suggestion for spacer technique, AAP/NHLBI-aligned color-coded zones).

**Input from the clinician:**

```
Patient: J.R., 7yo male. New dx moderate persistent asthma. Spirometry FEV1
72% pred pre-bronchodilator, +14% post-albuterol. Daytime sx 4-5x/wk,
nighttime cough 2-3x/mo, 2 ED visits in past year (no admission, no
intubation). Allergic rhinitis comorbid. No NSAID exposure issues.
Caregivers: mother (primary, prefers Spanish), grandmother (secondary,
English).

Plan today:
- Start fluticasone 88 mcg HFA, 2 puffs BID via valved holding chamber
  (spacer) with mask
- Continue albuterol 90 mcg HFA, 2 puffs q4h PRN with spacer
- Oral prednisolone 1 mg/kg/day x 5 days for current exacerbation
- Spacer technique demonstrated; rinse mouth after fluticasone
- F/u 4 weeks, asthma control test + spirometry
- Action plan: green/yellow/red zones with peak-flow personal best TBD
  next visit (too young for reliable PEF baseline today)
- School action plan copy to school nurse
```

**Generated handout (English version — Spanish-paired version produced in parallel at same reading level):**

```
J.R.'s Asthma Action Plan

A plan from [Practice Name] to help J.R. breathe well and to help you
know what to do when his asthma changes.

What Is Asthma?
Asthma makes the small tubes in J.R.'s lungs swell and get tight.
When that happens, he coughs, wheezes, or has a hard time breathing.
We can keep his asthma quiet with one medicine every day, and we can
treat a bad day with a different medicine.

J.R.'s Two Medicines
- Controller (every day, even when he feels fine):
  Fluticasone 88 inhaler — 2 puffs through the spacer with mask in the
  morning, 2 puffs at night. After each use, wipe his mouth and have him
  rinse and spit if he can.
- Rescue (only when he is coughing or having a hard time breathing):
  Albuterol 90 inhaler — 2 puffs through the spacer with mask. He can
  use it every 4 hours if he needs it. Tell us if he is using it more
  than 2 days a week — that means his asthma is not under control yet.

How to Use the Spacer (this part matters most)
1. Shake the inhaler. Put it in the back of the spacer.
2. Put the mask gently over J.R.'s nose and mouth — make a good seal.
3. Press the inhaler once.
4. Let him breathe in and out 6 normal breaths through the mask.
5. Wait 30–60 seconds before the next puff.
[Visual to include in print: spacer-with-mask diagram showing seal and
6-breath count — do not generate; pull from the practice's pediatric
asthma toolkit.]

Today's Short Steroid Course
- Prednisolone liquid — give the dose we marked on the bottle, once a
  day, with food, for 5 days. Do not stop early. It is normal for him to
  feel hungrier or a little more energetic.

Asthma Zones — What to Do Each Day

GREEN ZONE — He feels good. No cough. Playing normally. Sleeping well.
- Give the controller (fluticasone) 2 puffs morning and night, every day.
- He does not need albuterol.

YELLOW ZONE — He has a cough, mild wheeze, or wakes up at night, OR he
is starting a cold.
- Give albuterol 2 puffs through the spacer. Wait 20 minutes.
- If he is better, you can repeat albuterol every 4 hours for 24 hours.
- Keep giving the daily fluticasone.
- Call us if he stays in the yellow zone for more than 24 hours, or if
  he needs albuterol more than every 4 hours.

RED ZONE — Call 911 or Go to the ER
- He is breathing fast and you can see his ribs sucking in with each
  breath, OR
- His lips or fingertips look blue or gray, OR
- He cannot talk in full sentences because he is too short of breath, OR
- The albuterol did not help after 2 doses 20 minutes apart, OR
- He is too sleepy or limp to wake up normally.

These are emergencies. Give 2 puffs of albuterol on the way and call 911.

When to Call Us (Not 911)
- He needed albuterol more than 2 days in a row
- He has missed school because of asthma
- He is waking up at night coughing
- He has any new medicine question

Teach-Back — Please Tell Us in Your Own Words
- When does J.R. take the fluticasone? When does he take the albuterol?
- What does the yellow zone look like? What do you do first?
- What would make you call 911 instead of us?
- Why do we keep giving the fluticasone even when he feels good?

Your Team and Next Steps
- J.R.'s clinician: [Pediatrician Name], [Practice Name]
- Office phone: [Number] (open 8–5)
- After-hours nurse line: [Number]
- Patient portal: [Name/Link]
- Next visit: 4 weeks for asthma control check and breathing test
- School: we are sending a copy of this plan to J.R.'s school nurse
  with your permission

Crisis help: 911 for emergencies. Poison Control 1-800-222-1222.

Reviewed by: _________________   Date: __________
[CLINICIAN MUST PERSONALIZE: peak-flow personal best deferred to next
visit; update zones with PEF numbers once established.]

This handout is for learning. It does not replace your clinician's
advice. If J.R. changes, call us or seek care.
```

**What this second example demonstrates:**

- **Caregiver-first voice** — every action verb is directed to the caregiver ("give him," "let him breathe," "tell us"), not the 7-year-old. Demonstrates the `practice_voice=caregiver_first` hook from the §1 config block.
- **Pediatric specialty defaults** — spacer technique block, color-coded zone framework (NHLBI EPR-4 / AAP-aligned), school-nurse routing, and weight-based steroid dosing line surface automatically when `practice_specialty=pediatrics` is set. The §1 hook list called out asthma action plan + spacer technique as the pediatric-specialty default and this example shows it firing.
- **Reading level lowered** — drafted at grade 5 (per `reading_level_default=grade_5` for caregiver-of-pediatric simplified) rather than the adult default of grade 6–8. Sentence length stays ≤ 12 words; Latinate vocabulary replaced ("medicine" not "medication," "fast" not "rapid," "tight" not "constricted").
- **Multilingual pair output** — handout shipped as English + Spanish at the same reading level per the mother's preferred language, rather than a one-language default. This exercises the `language_preferences_supported` hook and the `output_destination=multilingual_pair` value rather than `outputs/`.
- **Safety-critical red zone** — the Red Zone block is formatted as a standalone bolded line rather than a buried bullet, per the §6 anti-error rule ("Never bury a life-threatening symptom in a bullet list"). Five specific red-flag presentations are named (work of breathing, cyanosis, inability to speak in sentences, albuterol failure after 2 doses, altered mentation) rather than the generic "trouble breathing."
- **Clinician-must-personalize gate fires** — peak-flow personal best is deferred to the next visit because reliable PEF in a 7-year-old often requires technique coaching across multiple sessions. The handout ships with an explicit `[CLINICIAN MUST PERSONALIZE]` flag rather than fabricating a PEF number, exercising the `safety_critical_topic_gate` config hook for the pediatric-dosing category.
- **Two-tier triage discipline** — separate "When to Call Us (Not 911)" and "Red Zone — Call 911 or Go to the ER" blocks, mirroring the §6 two-column rule. Yellow Zone is a self-care path with explicit time limits before escalation, not a "watch and wait" without a deadline.
- **Visual placeholder, not generated image** — spacer-with-mask diagram is named and the handout points to the practice's pediatric asthma toolkit rather than generating an image, per the §"Format, Reading Level, and Accessibility Rules" rule ("Do not generate images; name the visual the handout should include").

Together the two worked examples now demonstrate the skill's range across the two highest-volume use cases in outpatient education: an adult chronic-condition medication-start in warm-plain voice (Example 1) and a pediatric chronic-condition action plan in caregiver-first voice with a parallel Spanish version, color-coded zones, and a fired safety-critical gate (Example 2). Together they exercise eleven of the thirteen named config hooks (`practice_specialty`, `reading_level_default`, `language_preferences_supported`, `clinician_signature_block`, `practice_contact_block`, `crisis_line_overlay`, `practice_voice`, `safety_critical_topic_gate`, `21cca_information_blocking_default`, `output_destination`, `config_missing_behavior`), leaving only `pharmacy_and_partner_block` and `patient_facing_disclaimer_block` as worked-example debt for a future targeted run.

## Notes on Policy and Evidence

- CMS, AHRQ, and Joint Commission all recommend patient education at or below the 8th-grade reading level; the federal Plain Writing Act supports plain-language public-health content.
- AHRQ's Health Literacy Universal Precautions Toolkit is the operational reference for teach-back, clear communication, and the "assume low literacy for everyone" design principle.
- The 21st Century Cures Act information-blocking rules mean patient-facing content attached to the chart is typically visible to the patient via the portal in near-real time. Draft accordingly.
- For medication handouts, reference the current FDA-approved label and MedlinePlus for drug-specific safety content; the skill paraphrases those sources rather than reproducing them.
- Any handout covering suicide, IPV, substance use, eating disorders, or reproductive health requires a clinician-in-the-loop review before it leaves the practice.
