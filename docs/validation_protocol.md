# Validation Protocol

This document describes the validation protocol applied to all items in MedQA-KR/EN, including inter-rater-reliability quantification and conflict resolution. Full implementation details and quantitative results are reported in the accompanying Data Descriptor (Kim et al., 2025).

## Stage 1: Automated LLM self-review

Each generated item was first screened by GPT-4o on four dimensions:

| Dimension                   | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| Semantic coherence          | Question and answer text are internally consistent and well-formed          |
| Internal consistency        | The stated answer is supported by the question text                         |
| Clinical plausibility       | Content does not contradict published clinical-practice guidelines          |
| Grammatical completeness    | Sentences are complete and grammatically correct                            |

**Clinical-plausibility checklist** (included in the self-review prompt):

1. Does not contradict published guidelines from the seven source categories
2. Uses terminology consistent with ICD-10 or its Korean adaptation (KCD-8)
3. Avoids dosages or procedures not supported by the source passage
4. Is answerable from the source passage rather than extra-source knowledge

Items scoring <4 on any dimension entered a **bounded self-revision** loop:

- GPT-4o was prompted to identify the deficiency, propose a revision, and re-evaluate
- Maximum recursion depth: **3 rounds**
- Termination conditions: (i) all dimensions ≥4, (ii) 3 rounds completed, or (iii) no improvement from previous round
- Items still failing after self-revision were discarded

## Stage 2: Pre-filtering

To manage human-reviewer workload, three pre-filters were applied:

1. **Rule-based length filter**: questions outside 30–1000 characters or answers outside 5–2000 characters were excluded
2. **LLM topic-relevance filter**: items referring to historical or anachronistic content (e.g., pre-1990 clinical recommendations no longer applicable) were excluded
3. **Reserve set**: 80% of remaining items were reserved for future dataset expansion

After pre-filtering, **29,999 items** entered physician review.

## Stage 3: Physician review

### Reviewer panel

- **30 licensed physicians**, none of whom authored seed items
- Drawn from hospitals other than the 4 institutions that contributed seed items
- 14 board-certified specialists + 16 senior residents
- Specialty distribution selected so that each of the 17 specialties was reviewed primarily by physicians with corresponding clinical practice

### Workload

- ~1,000–1,200 items per reviewer over a 6-week period
- ~30 items per working day target

### Review interface

For each item, reviewers performed five actions:

1. Assign a specialty label (1 of 17)
2. Score the **question** on a 1–5 Likert scale across three dimensions: content accuracy, clinical appropriateness, format clarity
3. Score the **answer** on the same three dimensions
4. Decide: **accept** / **accept with edits** (inline revision supported) / **reject**
5. Optionally provide free-text comments

### Inter-rater reliability

**Overlap set design**: 1,500 items (5% of the validation set) were independently reviewed by **3 reviewers each**, with assignment stratified by specialty and format. Reviewers were blinded to each other's assignments. The remaining items were reviewed by a single physician each.

**Fleiss' kappa** was computed on the accept / accept-with-edits / reject decision treated as a three-category nominal variable. The overall and per-specialty kappa values are reported in the Data Descriptor (Table 7), interpreted using the Landis–Koch framework.

### Conflict resolution

A tiered procedure was applied:

1. **Majority decision** (2 of 3 reviewers agreed): adopt the majority decision
2. **All three disagree** (1 accept / 1 edit / 1 reject): escalate to a board-certified specialist in the relevant field for final adjudication
3. **Edit conflicts** (multiple reviewers proposed incompatible edits): senior reviewer adjudicates the final wording

All conflict-resolution decisions are logged in `data/conflict_resolution_log.jsonl` (released with the dataset) to support transparency analyses.

## Stage 4: Bilingual translation review

The validated Korean items were translated to English by GPT-4o using a structured prompt requiring:

- Preservation of medical terminology in standardized English forms (ICD-10 aligned)
- Preservation of all five options for MCQ items with the same correctness mapping
- Cultural-context preservation where Korea-specific information is material (e.g., national-vaccination-program schedules)
- Conversion of Korean idiomatic clinical phrasing to clinically standard English equivalents

Translated items were re-reviewed by the same 30-physician panel using a simpler appropriate / inappropriate decision plus in-line revision. Of 33,379 translated items, **99.7% of questions and 99.9% of answers** were judged appropriate; the remaining items were revised in-place.

## Quality summary

| Stage                          | Items in | Items out | Acceptance rate |
|--------------------------------|----------|-----------|-----------------|
| Generation                     | —        | 69,678    | —               |
| LLM self-review + pre-filter   | 69,678   | 29,999    | 43.1%           |
| Physician review (Korean)      | 29,999   | 29,999    | 99.7% as-is or with edits |
| Merge with seed items          | +3,380   | 33,379    | —               |
| English translation review     | 33,379   | 33,379    | 99.7% Q / 99.9% A |

## Reproducibility

Implementation code for each stage is in the `/code/` directory (when included in the release):

- `generate.py`: prompt construction and GPT-4o calls
- `self_review.py`: bounded self-revision loop
- `pre_filter.py`: rule-based and LLM-based filters
- `irr.py`: Fleiss' kappa computation
- `translate.py`: bilingual extension

A `manifest.json` documents the exact API model version (`gpt-4o-2024-08-06`), seed values, and timestamps for each generation and validation run, supporting reproducibility audits.
