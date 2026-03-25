# KUru Presentation Improvement Notes

> **Goal:** Make listeners feel the problem is real and that solving it provides high value.
> Based on review of `KUru_Presentation.pptx.pdf` (17 slides) against the current SRS.

---

## Core Diagnosis

The slides are technically correct but emotionally flat. Listeners understand *what* KUru does but never feel *why it matters*. There is no human moment in the entire 17 slides. The deck tells — it does not show.

---

## Priority 1 — Critical Changes

### 1.1 Add a Student Persona Slide (Missing Entirely)

Insert a new slide **immediately after the title**, before any system description.

A single relatable scenario does more persuasion than any three bullet points. Suggested content:

> *Nong is a Mathayom 6 student. She wants to study something tech-related at KU.*
> *She finds 20+ engineering programs on the KU website.*
> *She searches for PLOs. She downloads a 60-page Thai PDF written for accreditation reviewers.*
> *She gives up and asks ChatGPT, which confidently gives her wrong quota numbers.*
> *TCAS Round 1 opens in 3 weeks. She applies to the wrong program.*
>
> **This is the experience of 50,000+ students every TCAS cycle.**

---

### 1.2 Elevate "4 Years — Cost of Wrong Choice" to a Headline

Currently this phrase is a small caption under a stat box at the bottom of Slide 3. It is the strongest line in the deck and it is invisible.

**Create a dedicated transition slide** before the three-problem cards:

> # A wrong faculty choice costs 4 years.
> *50,000+ students make this decision every year with no structured guidance.*

Then follow with the three specific problems (PLOs inaccessible, TCAS fragmented, no graduate visibility).

---

### 1.3 Fix the Narrative Order

**Current order:**
1. Title
2. **Scope (what we don't build)** ← kills momentum
3. Problem
4. Objectives
5. Why AI
6–9. Competitors (×4 slides)
10. Comparison table
11–16. Technical slides
17. Closing

**Recommended order:**
1. Title
2. Student persona / human story *(new)*
3. "4 years" headline *(new)*
4. Three problems with evidence
5. Why no existing solution works *(condensed)*
6. Comparison table
7. Our solution + features
8. **Why we designed it this way** *(new — design rationale: RIASEC grounding, O\*NET pipeline, α blending)*
9. How the AI works *(recommendation pipeline)*
10. Data sources
11. Model choices + baseline
12. Architecture
13. Success metrics
14. Timeline
15. **Scope** *(moved here — now context, not disclaimer)*
16. Closing

The scope slide should come near the end. Listing what you won't build before the audience understands the pain reads as a defensive disclaimer, not confident scoping.

---

## Priority 2 — High Impact Changes

### 2.1 Consolidate Four Competitor Slides Into Two

Slides 6–9 each dedicate a full slide to one competitor (ChulaGENIE, Knowva, TCAS Official, ChatGPT). Four identical-structure slides in a row cause the audience to stop reading by slide 7.

**Option A — Two combined slides:**
- Slide A: ChulaGENIE + Knowva side by side (closest AI competitors)
- Slide B: mytcas.com + ChatGPT side by side (most commonly used tools)

**Option B — Cut all four entirely.** The comparison table (Slide 10) already tells the whole story. The individual competitor slides add length without adding persuasion. Keep only the table.

---

### 2.2 Show the O\*NET Step in the Recommendation Pipeline (Slide 12)

The current recommendation pipeline column reads:

```
Interest profile (8-topic grid + follow-up)   ← vague: no RIASEC framing
    ↓
Skill cluster mapping (Weighted skill vector)  ← O*NET step is invisible here
    ↓
pgvector similarity (Candidate program retrieval)
    ↓
Neo4j graph traversal (Faculty → PLO → Skill → Career)
    ↓
Fit score ranking
    ↓
LLM explanation
```

This has two problems. First, "Interest profile (8-topic grid + follow-up)" uses the same vague pre-RIASEC language that has been updated everywhere else in the SRS — the slide should say "RIASEC interest profile" to be consistent. Second, the O\*NET matching step is entirely absent: the slide jumps from interest profile directly to skill cluster mapping, hiding the key credibility insight where student interests are matched to 900+ real occupations using profiles **empirically derived from surveys of employed workers**. This is what separates KUru from a generic interest quiz.

**Corrected pipeline to show:**

```
5-step RIASEC elicitation (2–4 min)
    ↓  tile grid → pairwise comparisons → scenario image → dealbreaker filter → confidence check
RIASEC vector (L2-normalised)
    ↓
O*NET career matching — precomputed profiles from real worker surveys
    ↓
Target skill derivation (O*NET taxonomy)
    ↓
SkillCluster crosswalk (O*NET → Neo4j taxonomy)
    ↓
Neo4j PLO query → Fit score ranking
    ↓
Gemini 2.5 Flash explanation
    ↓
Behavioural re-ranking — α blending (registered users only)
    ↓  new user: α = 1.0 (pure RIASEC) → decays toward α ≈ 0.2 as interactions accumulate
```

The phrase "empirically derived from surveys of employed workers" is in the SRS but never appears in the slides. It belongs on this slide.

**Two specific errors in the previously-suggested corrected pipeline (now fixed above):**

- The first step was still labelled "8-topic RIASEC grid + follow-up" — the design is now a 5-step adaptive elicitation, not a grid plus one follow-up.
- Stage 6 (behavioural re-ranking with α blending) was missing entirely from the pipeline diagram.

---

### 2.3 Rewrite Slide 4 Objectives as Student Outcomes

Current tiles are product feature names. Rewrite as outcomes the student experiences:

| Current | Suggested rewrite |
|---------|------------------|
| Surface hidden curriculum | *Students can see what they'll actually learn before they apply — in plain Thai or English* |
| Personalized program match | *Your RIASEC interest profile, matched to real careers, mapped to the right KU program* |
| Demystify TCAS admission | *Know exactly what score you need, for which round, before the deadline* |
| Show graduate outcomes | *See what KU graduates actually earn in their first job — by program* |
| Grounded curriculum chatbot | *Ask anything about any KU program. Every answer cites the exact มคอ.2 section* |
| KU-specific, not generic | *Built exclusively for KU — not a general chatbot guessing from old training data* |

---

## Priority 3 — Medium Impact

### 3.1 Graduate Outcome Data Needs Its Own Problem Statement

Slide 4 has "Show graduate outcomes" as one of six equal tiles. But this data point matters most to **Thai parents**, who are often the real decision-makers for a student's TCAS application.

Add a line to the problem slide or student persona slide:

> *Parents are asked to support a 4-year commitment but cannot find out what KU graduates actually earn. No public source shows first job title and salary range by KU program. KUru shows this — sourced directly from faculty.*

This reframes the target audience from just students to also parents, which significantly expands the perceived value of the system.

---

### 3.2 Move the Closing Tagline to the Opening

The closing slide (Slide 17) contains the best sentence in the entire deck:

> **"We turn the documents students never read into a personal advisor."**

This line is on the last slide, which most evaluators barely read. It should appear in some form on Slide 1 or the problem slide — ideally as the subtitle on the title card, replacing the current "An AI-powered academic pathway advisor for students exploring programs at Kasetsart University."

---

## Priority 4 — Minor Fixes

### 4.1 Outdated Model Name in Slide 11 Code Snippet

The data pipeline snippet on Slide 11 reads:
```
PyMuPDF → chunk by structure → embed (text-embedding-004) → pgvector + Neo4j
```

The project now uses `text-embedding-001`. Update the snippet to match.

---

### 4.2 Vague "Interests" Language on Slide 4 Description Text

The description under the "Personalized program match" tile reads:
> *"Semantic similarity + knowledge graph maps interests to the right KU program."*

"Interests" should be "RIASEC interest profile" — every other section of the SRS now uses this framing and the slide should match.

Also check Slide 2 (Scope): "Interest discovery — find the right KU program" should read "RIASEC interest discovery — find the right KU program."

---

### 4.3 "Who's" Grammar on Slide 14

The system architecture slide (Slide 14) reads:
> *"Students or people who's interested in KU curriculum"*

Should be: **"Students or anyone interested in KU programs"**

---

### 2.4 Add a "Design Rationale" Slide — Why We Made These Choices

**This slide does not currently exist and should be added.** It is the single most important slide for an advisor audience. It shows that the design decisions were made for principled reasons, not guesswork.

**Where to place it:** Between "Our solution + features" and "How the AI works" — as a bridge that explains the *why* before the *how*.

**The core argument the slide should communicate:**

The original design (8-tile grid + one follow-up) was intentionally simple but too shallow to produce a defensible RIASEC vector. The redesign addressed this at three levels:

**Level 1 — Better signal.** Replaced the two-step elicitation with a five-step adaptive flow grounded in Holland's RIASEC model. The tile grid provides fast orientation; adaptive pairwise comparisons target only the dimensions that are ambiguous for *this specific student*; a visual scenario gives non-verbal confirmation; a dealbreaker filter removes definite exclusions; a confidence check downweights uncertain answers. Output: a 6-dimensional L2-normalised RIASEC vector. Total time: 2–4 minutes.

**Level 2 — Grounded in real career data.** Instead of mapping the RIASEC vector directly to KU programs (a speculative leap), the system now routes through O\*NET — 900+ occupations with RIASEC interest profiles derived from surveys of *employed workers*. The student's vector matches careers first, then the skills those careers require are traced back to KU programs via Neo4j. Every recommendation is now auditable through a full chain: interest → career → skill → PLO → program.

**Level 3 — Gets better over time.** The initial RIASEC vector is an approximation. A blending weight α starts at 1.0 (pure RIASEC) and decays toward 0.2 as the student accumulates interaction data — program saves, chatbot queries, TCAS guide views. The more the student uses KUru, the more their actual exploration pattern replaces the initial questionnaire approximation. A student who never corrects the system still gets progressively better recommendations.

**Suggested slide title:** *"Why we designed it this way"* or *"Three layers of reliability"*

**One-sentence version for the spoken narrative:**
> *"KUru starts with a validated theoretical framework to get you into the right neighbourhood on your first visit, uses 900+ real careers as an intermediate step so nothing is guesswork, and then gets progressively more accurate as your actual behaviour replaces the initial approximation — all while remaining completely transparent about how each recommendation was reached."*

**What NOT to include on this slide:** α values, interaction weight table, the word "cosine similarity." Those go on the technical slides. This slide is for the advisor who wants to know whether the system is academically defensible.

---

### 2.5 Clarify Individual Behavioural Blending vs. Collaborative Filtering

If any slide describes the α blending mechanism as "collaborative filtering," correct it. The SRS now explicitly distinguishes two things:

- **MVP (built):** Single-user behavioural blending — the α mechanism weights each student's *own* interaction history (saves, card views, chatbot queries) against their RIASEC score. No cross-student comparison.
- **Phase 2 (deferred):** True collaborative filtering — using interaction patterns of students with *similar* RIASEC profiles to inform each other's recommendations.

Presenting α blending as collaborative filtering overstates the MVP and will be caught by an advisor who reads the SRS. If the slide mentions personalisation, the safe framing is: *"Recommendations improve as you use it — your saves and browsing history progressively shift the ranking toward what you actually care about."*

---

## Summary Table

| Priority | Slide(s) | Issue | Action |
|----------|----------|-------|--------|
| Critical | After Slide 1 | No human story | Add student persona slide |
| Critical | Before Slide 3 | "4 years" hook is buried | Add dedicated headline slide |
| Critical | Slide 2 | Scope before problem kills momentum | Move scope to near the end |
| High | Slides 6–9 | Four identical competitor slides | Consolidate to 2 slides or cut to just the table |
| High | Missing | No slide explaining *why* the design was made this way | Add "Design Rationale" slide between features and technical slides (see §2.4) |
| High | Slide 12 | Pipeline missing 5-step elicitation label, Stage 6, and O\*NET | Show full 9-step pipeline per §2.2 corrected diagram |
| High | Slide 4 | Objectives are feature names, not outcomes | Rewrite as student-facing outcomes |
| High | Any slide | α blending described as "collaborative filtering" | Correct to "single-user behavioural blending"; defer CF to Phase 2 |
| Medium | Slides 3, 4 | Graduate outcome data underweighted | Add parent-facing framing to problem section |
| Medium | Slide 17 | Best tagline is on the last slide | Move to Slide 1 subtitle |
| Minor | Slide 11 | `text-embedding-004` in code snippet | Update to `text-embedding-001` |
| Minor | Slides 2, 4, 12 | Vague "interests" / "Interest profile" language predating RIASEC integration | Replace with "RIASEC interest profile" throughout |
| Minor | Slide 14 | "who's" grammar error | Fix to "anyone interested in" |
