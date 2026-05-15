---
language:
  - ko
  - en
license: cc0-1.0
size_categories:
  - 10K<n<100K
task_categories:
  - question-answering
  - text-generation
task_ids:
  - multiple-choice-qa
  - open-domain-qa
pretty_name: KoMedQA
tags:
  - medical
  - medical-education
  - licensing-examination
  - KMLE
  - USMLE
  - bilingual
  - clinical-knowledge
---

# Dataset Card for KoMedQA

## Dataset Summary

**KoMedQA** is a physician-validated bilingual (Korean–English) question-answering dataset of **33,379 examination-style items** spanning **17 clinical specialties** aligned with the Korean Medical Licensing Examination (KMLE) subject taxonomy. The dataset combines **3,380 physician-authored seed items** with **29,999 LLM-augmented items** that were subsequently reviewed by **30 licensed physicians** under a documented expert-in-the-loop curation protocol.

The principal intended use is medical-education research and language-model adaptation for medical-education applications, with particular relevance to Korean-language medical training where openly licensed bilingual resources have been scarce.

## Supported Tasks

- **Multiple-choice question answering** (`q_type = 1`, n = 26,355): five-option items in the style of national licensing examinations.
- **Short-answer question answering** (`q_type = 2`, n = 3,444): brief constructed-response items.
- **Long-answer question answering** (`q_type = 3`, n = 3,580): extended-response items eliciting clinical reasoning.

The dataset is suitable for supervised fine-tuning, instruction tuning, retrieval-augmented question answering, and evaluation of medical knowledge in large language models.

## Languages

- **Korean (`ko`)**: 33,379 items (original authoring language).
- **English (`en`)**: 33,379 items (machine-translated by GPT-4o and physician-reviewed; 99.71 % of questions and 99.94 % of answers passed without revision).

## Dataset Structure

### Data Instances

```json
{
  "qa_id": 61,
  "domain": 4,
  "domain_name": "Neurology/Neurosurgery",
  "q_type": 1,
  "q_type_name": "multiple_choice",
  "question": "75세 여성이 혈관성 치매(VaD)로 진단받고...",
  "answer": "3) HBOT와 donepezil 병용 시..."
}
```

### Data Fields

| Field | Type | Description |
|---|---|---|
| `qa_id` | integer | Unique identifier |
| `domain` | integer | Specialty code (1–17, see Domain Codes below) |
| `domain_name` | string | Human-readable specialty name (JSONL only) |
| `q_type` | integer | 1 = multiple choice, 2 = short answer, 3 = long answer |
| `q_type_name` | string | Human-readable question-format name (JSONL only) |
| `question` | string | Question text in the file's primary language |
| `answer` | string | Answer text in the file's primary language |

### Domain Codes and Counts

| Code | Specialty | Count |
|---|---|---|
| 1  | General Surgery | 3,659 |
| 2  | Preventive Medicine | 734 |
| 3  | Psychiatry | 1,969 |
| 4  | Neurology / Neurosurgery | 2,134 |
| 5  | Dermatology | 695 |
| 6  | Ophthalmology | 740 |
| 7  | Otorhinolaryngology | 532 |
| 8  | Urology | 931 |
| 9  | Radiation Oncology | 108 |
| 10 | Pathology | 145 |
| 11 | Anesthesiology and Pain Medicine | 837 |
| 12 | Medical Law | 30 |
| 13 | Miscellaneous (Pharmacology, Pathology, Anatomy) | 2,239 |
| 14 | Obstetrics and Gynecology | 2,438 |
| 15 | Pediatrics | 3,018 |
| 16 | Emergency Medicine | 677 |
| 17 | Internal Medicine | 12,493 |

### Data Splits

The dataset is released as a single split. Users designing fine-tuning or evaluation experiments are encouraged to construct their own splits stratified by specialty and format. For evaluation against real licensing-examination items, we recommend using the held-out external benchmarks **KorMedMCQA** (Korean) and **MedQA** (English) rather than withholding items from KoMedQA.

## Dataset Creation

### Source Data

The augmented items were derived from a curated **752-million-token** Korean medical corpus (markdown-only version; a human-readable, segment-level variant comprising 222 million tokens is also maintained) assembled from seven source categories: peer-reviewed journal articles; online medical information portals; Korean government clinical guidelines; Korean medical-society guidelines; international-organisation guidelines (WHO, CDC, and similar bodies); medical textbooks; and other materials (chiefly surgical and procedural consent forms). Token-level proportions are reported in the Data Descriptor. The corpus is not redistributed because several constituent source materials carry licenses incompatible with public redistribution; aggregate composition is reported and the corpus-indexing logs are available from the corresponding authors on reasonable request.

### Annotation Process

- **Seed items.** 3,380 question–answer pairs were authored by 20 licensed Korean physicians (5 from each of 4 participating tertiary-care hospitals).
- **LLM augmentation.** GPT-4o (`gpt-4o-2024-08-06`) generated items grounded in source-corpus passages, using structured `[task, query, exemplar]` prompts that combine a task description, a generation directive, and a physician-authored exemplar.
- **LLM self-review.** An automated four-dimension rubric (semantic coherence, internal consistency, clinical plausibility, grammatical completeness) was applied, with bounded self-revision at **maximum recursion depth 2 rounds**. Items still failing after self-revision were discarded.
- **Physician review.** 30,000 items were reviewed by 30 licensed physicians not involved in seed authoring, using a **single-rater design**: each item was reviewed by exactly one physician, who optionally edited the question and answer text and rated the item on four Likert dimensions (question length, question quality, answer length, answer quality) and assigned a specialty label. The single-rater design was a deliberate trade-off for coverage breadth.
- **Review-consistency validation.** Review consistency was characterised post-hoc through four complementary analyses on the full reviewed pool, including a strong score–edit consistency relationship (Spearman ρ = −0.32 for answer quality versus answer edit ratio, *p* < 1 × 10⁻³⁰⁰), demonstrating that reviewers applied a consistent rubric across the dataset.
- **Bilingual extension.** Validated Korean items were translated to English by GPT-4o and re-reviewed by the same physician panel; 98 questions and 19 answers were flagged inappropriate and revised in place.

Full implementation details are reported in the Data Descriptor (Kim et al., 2025) and in [`docs/validation_protocol.md`](docs/validation_protocol.md).

### Personal and Sensitive Information

The dataset does not contain real patient data. All clinical vignettes are generic, fictional, or derived from textbook cases. The study was reviewed by the Catholic University of Korea, Songeui Medical Campus IRB (CMC IRB) and granted exemption under project reference **MC24EISI0042** on 26 April 2024.

## Considerations for Using the Data

### Social Impact

The dataset is designed to support medical education and language-model adaptation for medical-education applications, contributing to United Nations Sustainable Development Goal 4 (Quality Education). The bilingual coverage particularly serves Korean medical education, which has had limited access to openly licensed examination-style resources.

### Biases

- The 17 specialties are weighted toward common KMLE topics; the distribution is heavy-tailed (Internal Medicine: 12,493; Medical Law: 30).
- The dataset reflects Korean clinical practice conventions and pharmaceutical availability as of 2024–2025. Items may not always reflect practice patterns in other jurisdictions.
- English items were produced through machine translation followed by physician spot-check review; subtle stylistic differences from natively-authored English may remain.
- Augmented items inherit the LLM source's training data biases. The physician review process mitigates but does not eliminate this risk.

### Limitations

- The dataset does not include multimodal content (images, ECG traces, pathology slides). Pair with multimodal resources for vision-language work.
- Item difficulty has not been empirically calibrated against actual examinee performance. For IRT-based studies, supplement with KorMedMCQA.
- Three specialties (Medical Law: 30, Radiation Oncology: 108, Pathology: 145) have small sample sizes; per-specialty conclusions for these should be drawn with caution.
- The single-rater physician-review design (without an overlap-based inter-rater reliability statistic) was a deliberate trade-off for coverage breadth. Consistency of review behaviour was characterised post-hoc through complementary analyses (see Data Descriptor).

## Additional Information

### Dataset Curators

The dataset was constructed by a multi-institution team led by The Catholic University of Korea, with clinical collaborators from Samsung Medical Center, Seoul National University Hospital, Yonsei University Hospital, and additional Korean tertiary-care hospitals.

### Licensing Information

Released under the [Creative Commons CC0 1.0 Universal Public Domain Dedication](https://creativecommons.org/publicdomain/zero/1.0/). Free for any use including commercial.

### Citation Information

```bibtex
@article{kim2025komedqa,
  title   = {KoMedQA: A bilingual Korean--English medical question-answering dataset
             across 17 clinical specialties},
  author  = {Kim, Tong Min and Kim, Chansik and Lee, Youngrong and Yang, Kwangmo and
             Lee, Yohan and Lee, Hyung-Chul and Kim, Byung-Hoon and Kim, Taehun and
             Kim, Huichung and Hwang, Injin and Park, Jungchan and Kim, Taeyoung and
             Lee, Hyeonhoon and Lee, Yushin and Park, Jinseok and Kim, Sung Hyun and
             Ko, Taehoon},
  journal = {Scientific Data},
  year    = {2025},
  note    = {Submitted}
}
```

### Contributions

Contributions and corrections are welcome via GitHub Issues. Please verify clinical content against current guidelines before relying on individual items in patient-facing applications.
