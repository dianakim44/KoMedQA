# Validation Protocol

This document describes the construction and quality-control protocol applied to all items in **KoMedQA**. Full implementation details and quantitative validation results are reported in the accompanying Data Descriptor (Kim et al., 2025).

## Stage 1: Automated LLM self-review

Each LLM-generated item was first screened by GPT-4o on four dimensions:

| Dimension | Description |
|---|---|
| Semantic coherence | Question and answer text are internally consistent and well-formed |
| Internal consistency | The stated answer is supported by the question text |
| Clinical plausibility | Content does not contradict published clinical-practice guidelines |
| Grammatical completeness | Sentences are complete and grammatically correct |

**Clinical-plausibility checklist** (embedded in the self-review prompt):

1. Does not contradict published guidelines from the seven source categories
2. Uses terminology consistent with ICD-10 or its Korean adaptation (KCD-8)
3. Avoids dosages or procedures not supported by the source passage
4. Is answerable from the source passage rather than extra-source knowledge

Items scoring below the acceptance threshold on any dimension entered a **bounded self-revision** loop:

- GPT-4o was prompted to identify the deficiency, propose a revision, and re-evaluate
- **Maximum recursion depth: 2 rounds**
- Termination conditions: (i) all dimensions ≥ acceptance threshold, (ii) 2 rounds completed, or (iii) no improvement from the previous round
- Items still failing after self-revision were discarded

## Stage 2: Pre-filtering

To manage human-reviewer workload and target the human-review budget at items most likely to benefit from physician scrutiny, three pre-filters were applied:

1. **Rule-based length filter**: questions outside 30–1,000 characters or answers outside 5–2,000 characters were excluded.
2. **LLM topic-relevance filter**: items referring to historical or anachronistic content (e.g., pre-1990 clinical recommendations no longer in routine practice) were excluded.
3. **Reserve set**: a portion of remaining items was reserved for future dataset expansion and is not part of the current release.

After pre-filtering, **30,000 items** were routed to physician review.

## Stage 3: Physician review

### Reviewer panel

- **30 licensed physicians**, none of whom authored seed items
- Drawn from institutions other than the four tertiary-care hospitals that contributed seed items
- Mix of board-certified specialists and senior residents
- Specialty distribution selected so that each of the 17 KMLE specialty categories was reviewed primarily by physicians with corresponding clinical practice

### Workload

- Approximately 1,000 items per reviewer (with variation by specialty) over a multi-week review period
- Reviewers were instructed to allocate sufficient time per item to ensure clinical-quality judgments rather than to meet a strict daily quota
- The review window spanned three phases (August 2024, October 2024, late October 2024), with each phase ending in an export of all completed items to date

### Review interface

Reviewers used a custom desktop application (Text Annotator v1.0). For each item, reviewers performed seven actions:

1. **Edit the question text** (optional, free-form). The LLM-generated original is preserved via a "Show Original" toggle.
2. **Edit the answer text** (optional, free-form). The LLM-generated original is preserved in the same way.
3. **Rate question length appropriateness** on a 1–5 Likert scale (1 = very short, 3 = adequate, 5 = very long).
4. **Rate question quality** on a 1–5 Likert scale (1 = very inappropriate, 3 = adequate, 5 = very appropriate).
5. **Rate answer length appropriateness** on the same 1–5 Likert scale.
6. **Rate answer quality** on the same 1–5 Likert scale.
7. **Assign a specialty label** (one of the 17 KMLE categories).

The interface auto-saved each rating action and exported a per-batch JSON file on completion. Output records preserve both the original LLM-generated text and the physician-revised text in separate fields, enabling post-hoc analysis of edit magnitude.

### Single-rater design and quality safeguards

KoMedQA adopted a **single-rater** physician-review design — each item was reviewed by exactly one of the 30 physicians — as a deliberate trade-off to maximise coverage of the 33,379-item dataset within the available expert-review budget. This design choice does not include an overlap-based inter-rater-reliability statistic (such as Fleiss' or Cohen's kappa). Instead, review consistency was characterised through four complementary analyses on the full 30,000-item reviewed pool:

1. **Edit prevalence and magnitude.** 84 % of questions and 93.4 % of answers were retained without modification; the median character-level edit ratio for items that were edited was small, indicating that reviewers used the edit channel sparingly and surgically rather than rewriting wholesale.
2. **Likert distributions.** Mean question-quality and answer-quality ratings of 3.44 and 3.43 respectively (on the 1–5 scale) with appropriately concentrated distributions, indicating that reviewers used the full scale rather than defaulting to a midpoint.
3. **Score–edit consistency.** A strong Spearman rank correlation between answer quality and answer edit ratio (ρ = −0.32, p < 1 × 10⁻³⁰⁰), demonstrating that reviewers applied a consistent rubric — items they scored as lower-quality were systematically edited more — rather than scoring at random.
4. **Cross-phase stability.** Score distributions remained stable across the three review phases (Cramér's V = 0.13), indicating no drift in reviewer behaviour over time.

Full quantitative results for each analysis are reported in the Data Descriptor (Section 3.2). The four-analysis framework was selected because the single-rater design precludes overlap-based reliability statistics; the analyses jointly establish that physician review was systematic and consistent across reviewers and across time.

## Stage 4: Bilingual translation review

The 33,379 validated Korean items were translated to English by GPT-4o using a structured prompt requiring:

- Preservation of medical terminology in standardised English forms (ICD-10 aligned)
- Preservation of all five options for MCQ items, with the same correctness mapping
- Cultural-context preservation where Korea-specific information is material (e.g., national-vaccination-program schedules)
- Conversion of Korean idiomatic clinical phrasing to clinically standard English equivalents

Translated items were re-reviewed by the same 30-physician panel using an *appropriate* / *inappropriate* decision plus in-line revision. Of the 33,379 translated items, **99.71 % of questions and 99.94 % of answers** were judged appropriate; the remaining 98 questions and 19 answers were revised in place. All 33,379 items in the released dataset have been reviewed.

Users requiring publication-quality English (e.g., for English-language patient-facing chatbots) should consider additional editing; the English version is best characterised as physician-spot-checked machine translation rather than as native English authorship.

## Quality summary

| Stage | Items in | Items out | Notes |
|---|---|---|---|
| LLM augmentation | seed prompts + corpus | ~70 k | grounded in 752 M token Korean medical corpus |
| LLM self-review + pre-filter | ~70 k | ~30 k | 4-dimension rubric, max-depth-2 self-revision |
| Physician review (Korean) | 30,000 | 30,000 | single-rater, 4 Likert + free-form edits |
| Merge with seed items | + 3,380 | 33,379* | seed items not subject to LLM self-review |
| English translation review | 33,379 | 33,379 | 99.71 % Q / 99.94 % A appropriate as-is |

*The released dataset comprises 33,379 items (1-item difference from 30,000 + 3,380 reflects deduplication at release time).

## Reproducibility

This protocol document, together with the prompt templates and schemas in `dataset_card.md` and `docs/data_schema.md`, is intended to enable independent reproduction of the construction-and-validation procedure at the methodological level. The data-generation pipeline relies on commercial APIs (OpenAI GPT-4o and NAVER HyperCLOVA X) that are not freely redistributable; the wrapper scripts that orchestrate these API calls are available from the corresponding authors of the Data Descriptor on reasonable request for purposes of academic replication or methodological extension.

## Ethics

The study was reviewed by the Catholic University of Korea, Songeui Medical Campus IRB (CMC IRB) and granted exemption under project reference **MC24EISI0042** on 26 April 2024. No patient data, clinical records, or identifiable health information were collected, used, or distributed. Physician participation was voluntary, with informed consent for inclusion of contributed content and remuneration provided through a formal contractor agreement.
