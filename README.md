# KoMedQA

**A Physician-Validated Bilingual (Korean–English) Question-Answering Dataset for Medical Licensing Examination Education across 17 Clinical Specialties**

[![License: CC0-1.0](https://img.shields.io/badge/License-CC0_1.0-lightgrey.svg)](https://creativecommons.org/publicdomain/zero/1.0/) [![Data: 33,379 pairs](https://img.shields.io/badge/Data-33%2C379_QA_pairs-blue.svg)](#dataset-composition) [![Languages: KO, EN](https://img.shields.io/badge/Languages-Korean%20%7C%20English-green.svg)](#languages)

**KoMedQA** is a bilingual (Korean–English) question-answering dataset of **33,379 examination-style items** spanning **17 clinical specialties** aligned with the Korean Medical Licensing Examination (KMLE) subject taxonomy. It combines **3,380 physician-authored seed items** with **29,999 LLM-augmented items** that were subsequently reviewed by **30 licensed physicians** under a documented quality-control protocol.

The dataset is released under **Creative Commons CC0 1.0 Universal** to enable unrestricted reuse in medical education, examination preparation, language-model training, and cross-cultural medical-assessment research.

---

## Quick start

```bash
git clone https://github.com/dianakim44/KoMedQA.git
cd KoMedQA
```

```python
import pandas as pd
import json

# CSV (single file with both languages)
df = pd.read_csv("data/medqa_kr_en_full.csv")
print(df.shape)  # (33379, 7)

# JSONL (separated by language; recommended for ML pipelines)
with open("data/korean/medqa_kr.jsonl", encoding="utf-8") as f:
    kr = [json.loads(line) for line in f]
with open("data/english/medqa_en.jsonl", encoding="utf-8") as f:
    en = [json.loads(line) for line in f]
print(len(kr), len(en))  # 33379 33379
```

Use cases include curriculum design, item-response-theory studies, examination-preparation platforms, language-model fine-tuning, and cross-cultural assessment-validity research.

---

## Repository structure

```
KoMedQA/
├── data/
│   ├── medqa_kr_en_full.csv          # Master file: both languages, 7 columns
│   ├── dataset_statistics.json       # Counts by specialty and format
│   ├── korean/
│   │   └── medqa_kr.jsonl            # Korean items only (33,379 lines)
│   └── english/
│       └── medqa_en.jsonl            # English items only (33,379 lines)
├── README.md
├── LICENSE                           # CC0 1.0
├── CITATION.cff                      # Citation metadata
├── dataset_card.md                   # HuggingFace-style dataset card
└── docs/
    ├── data_schema.md                # Field definitions and value mappings
    ├── validation_protocol.md        # Physician review and quality-control procedure
    └── changelog.md
```

---

## Dataset composition

### Question formats

| Format | `q_type` code | Count | Share |
|---|---|---|---|
| Multiple choice (5 options) | `1` | 26,355 | 78.9 % |
| Short answer | `2` | 3,444 | 10.3 % |
| Long answer | `3` | 3,580 | 10.7 % |
| **Total** |   | **33,379** | 100 % |

### Clinical specialties (KMLE taxonomy)

| Code | Specialty | Count | Share |
|---|---|---|---|
| 17 | Internal Medicine | 12,493 | 37.43 % |
| 1  | General Surgery | 3,659 | 10.96 % |
| 15 | Pediatrics | 3,018 | 9.04 % |
| 14 | Obstetrics and Gynecology | 2,438 | 7.30 % |
| 13 | Miscellaneous (Pharmacology, Pathology, Anatomy) | 2,239 | 6.71 % |
| 4  | Neurology / Neurosurgery | 2,134 | 6.39 % |
| 3  | Psychiatry | 1,969 | 5.90 % |
| 8  | Urology | 931 | 2.79 % |
| 11 | Anesthesiology and Pain Medicine | 837 | 2.51 % |
| 6  | Ophthalmology | 740 | 2.22 % |
| 2  | Preventive Medicine | 734 | 2.20 % |
| 5  | Dermatology | 695 | 2.08 % |
| 16 | Emergency Medicine | 677 | 2.03 % |
| 7  | Otorhinolaryngology | 532 | 1.59 % |
| 10 | Pathology | 145 | 0.43 % |
| 9  | Radiation Oncology (low-resource) | 108 | 0.32 % |
| 12 | Medical Law (low-resource) | 30 | 0.09 % |
| **Total** |   | **33,379** | 100 % |

> **Note on low-resource specialties.** Medical Law (n = 30), Radiation Oncology (n = 108), and Pathology (n = 145) have notably smaller sample sizes than other specialties. When publishing per-specialty results for these domains, please flag them or exclude them unless your analysis is specifically designed to handle small samples.

---

## Data schema

Each row / JSON object has the following fields:

| Field | Type | Description |
|---|---|---|
| `qa_id` | integer | Unique identifier for each item |
| `domain` | integer | Specialty code (see table above) |
| `q_type` | integer | 1 = MCQ (5-option), 2 = short answer, 3 = long answer |
| `question` | string | Question text in Korean (in `medqa_kr.jsonl`) or English (in `medqa_en.jsonl`) |
| `answer` | string | Answer text in the same language as `question` |
| `question_translated`\* | string | English translation of the question (CSV master file only) |
| `answer_translated`\* | string | English translation of the answer (CSV master file only) |

\* Only in the master CSV. The JSONL variants flatten Korean and English into separate files for ML pipelines.

For multiple-choice items (`q_type = 1`), the five options are concatenated inside `question` with leading `1)` `2)` `3)` `4)` `5)` markers, and the `answer` field contains the correct option label and explanation.

See [`docs/data_schema.md`](docs/data_schema.md) for full details.

---

## Construction and validation

The dataset was constructed through a six-stage expert-in-the-loop curation pipeline:

1. **Source-data preparation.** A 752-million-token Korean medical corpus was assembled from seven categories of certified sources (peer-reviewed journals, online medical information portals, Korean government and medical-society clinical guidelines, international guidelines from WHO/CDC and similar bodies, medical textbooks, and consent forms). A human-readable, segment-level version of the corpus (222 M tokens) is also available. **3,380 seed items** were authored by **20 licensed Korean physicians**.

2. **Prompt construction.** Structured prompts of the form `[T, Q, X]` — task description grounded in a corpus passage, generation parameters, and a physician-authored exemplar — were built per generation step.

3. **LLM-based augmentation.** GPT-4o (`gpt-4o-2024-08-06`) generated 69,678 synthetic items grounded in the source corpus.

4. **Automated screening and self-review.** Items first passed an LLM self-review with a four-dimension rubric (semantic coherence, internal consistency, clinical plausibility, format compliance) and bounded self-revision with a maximum recursion depth of two rounds. A rule-based filter then excluded items by topic relevance, length, and format validity, leaving **30,000 items routed to physician review**.

5. **Physician review.** Each of the 30,000 augmented items was reviewed by one of **30 licensed physicians** recruited from institutions not participating in seed authorship. Reviewers optionally edited the question and answer text, rated each item on a four-dimension 5-point Likert scale (question length, question quality, answer length, answer quality), and assigned a specialty label. The single-rater design was a deliberate trade-off for coverage breadth. Review consistency was characterised post-hoc through four complementary analyses on the full 30,000-item reviewed pool, including a strong score–edit consistency relationship (Spearman ρ = −0.32 for answer quality versus answer edit ratio, p < 1 × 10⁻³⁰⁰), indicating that reviewers applied a consistent rubric rather than scoring at random. The complete protocol is documented in [`docs/validation_protocol.md`](docs/validation_protocol.md).

6. **Bilingual extension.** Validated Korean items were translated to English by GPT-4o and re-reviewed by the same physician panel; 98 questions and 19 answers (0.35 % of items) were flagged inappropriate and revised in place.

**Ethics.** The study was reviewed by the Catholic University of Korea, Songeui Medical Campus IRB (CMC IRB) and granted exemption under project reference **MC24EISI0042** on 26 April 2024. No patient data, clinical records, or identifiable health information were collected, used, or distributed.

---

## Demonstrated utility

The dataset has been used to fine-tune two language model families and evaluated on KorMedMCQA (Korean) and MedQA (English):

| Model | Benchmark | Baseline | Best fine-tuned | Δ |
|---|---|---|---|---|
| HCX-003 | KorMedMCQA (KMLE) | 45.0 % | 59.3 % (MCQ-only) | **+14.3 pp** |
| GPT-4o  | KorMedMCQA (KMLE) | 84.6 % | 90.5 % (MCQ-only) | +5.9 pp |
| HCX-003 | MedQA (USMLE)     | 49.6 % | 53.8 % (long-only) | +4.2 pp |
| GPT-4o  | MedQA (USMLE)     | 86.9 % | 86.9 % | ≈ 0 (saturation) |

The substantial gain on HCX-003 (a Korean-optimised mid-size model) reflects the dataset's principal intended use case: **adapting Korean-centric or mid-size models for Korean medical education**. The dataset is *not* expected to advance state-of-the-art performance on already-saturated frontier models, particularly on English benchmarks. Full per-condition results are reported in the Data Descriptor.

---

## Languages

- **Korean (`ko`)**: 33,379 items. Original authoring language.
- **English (`en`)**: 33,379 items. Machine-translated by GPT-4o and physician-reviewed (99.71 % of questions and 99.94 % of answers passed without revision).

---

## License

This dataset is dedicated to the public domain under the **[Creative Commons CC0 1.0 Universal Public Domain Dedication](https://creativecommons.org/publicdomain/zero/1.0/)**.

You are free to copy, modify, distribute, and use the dataset, including for commercial purposes, without asking permission and without attribution. Attribution is appreciated but not required.

> ⚠️ **Source materials note.** The certified Korean medical corpus used to ground the LLM-based augmentation step (Stage 1) included some materials distributed under restrictive licenses. These materials were used *only as upstream context for LLM generation and are not redistributed*. The released KoMedQA items are newly LLM-generated text reviewed by physicians and contain no verbatim reproduction of any source material (no item reproduces a 15-or-more-consecutive-word span from any single source); they are therefore released under CC0.

---

## Citation

If you use KoMedQA in your research, please cite our Data Descriptor:

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

The structured citation file [`CITATION.cff`](CITATION.cff) is also provided.

---

## Related resources

- AI Hub Korean release: <https://www.aihub.or.kr/aihubdata/data/view.do?dataSetSn=71874>
- AI Hub English release: <https://www.aihub.or.kr/aihubdata/data/view.do?dataSetSn=71875>
- KorMedMCQA benchmark: [Kweon et al., 2024](https://arxiv.org/abs/2403.01469)
- MedQA (USMLE) benchmark: [Jin et al., 2021](https://github.com/jind11/MedQA)

---

## Acknowledgments

This work was supported by the Korea Health Technology R&D Project through the Korea Health Industry Development Institute (KHIDI) (RS-2023-KH134423, RS-2023-KH134396), the National Research Foundation of Korea (NRF) (2022R1C1C2002987), and the Basic Medical Science Facilitation Program of the Catholic Medical Center.

We thank the 30 physician reviewers and 20 seed-item authors whose work makes this dataset possible.

---

## Contact

- **Corresponding author**: Taehoon Ko, Ph.D. — <thko@catholic.ac.kr>
- **Joint corresponding author**: Youngrong Lee — <lyr6830@yuhs.ac>
- **Issues / corrections**: Please open an issue on GitHub.
