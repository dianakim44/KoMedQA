# MedQA-KR/EN

**A Physician-Validated Bilingual Question-Answering Dataset for Medical Licensing Examination Education across 17 Clinical Specialties**

[![License: CC0-1.0](https://img.shields.io/badge/License-CC0_1.0-lightgrey.svg)](https://creativecommons.org/publicdomain/zero/1.0/)
[![Data: 33,379 pairs](https://img.shields.io/badge/Data-33%2C379_QA_pairs-blue.svg)](#dataset-composition)
[![Languages: KO, EN](https://img.shields.io/badge/Languages-Korean%20%7C%20English-green.svg)](#languages)

MedQA-KR/EN is a **bilingual (Korean–English) question-answering dataset of 33,379 examination-style items** spanning **17 clinical specialties** aligned with the Korean Medical Licensing Examination (KMLE) subject taxonomy. It combines **3,380 physician-authored items** with **29,999 synthetically generated items**, all validated by **30 licensed physicians** under a documented inter-rater-reliability protocol.

The dataset is released under **Creative Commons CC0 1.0 Universal** to enable unrestricted reuse in medical education, examination preparation, language-model training, and cross-cultural medical-assessment research.

---

## Quick start

```bash
git clone https://github.com/<your-org>/MedQA-KR-EN.git
cd MedQA-KR-EN
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
MedQA-KR-EN/
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
    ├── validation_protocol.md        # Physician review and IRR procedure
    └── changelog.md
```

---

## Dataset composition

### Question formats

| Format                       | q_type code | Count   | Share  |
|------------------------------|-------------|---------|--------|
| Multiple choice (5 options)  | `1`         | 26,355  | 78.9%  |
| Short answer                 | `2`         | 3,444   | 10.3%  |
| Long answer                  | `3`         | 3,580   | 10.7%  |
| **Total**                    |             | **33,379** | 100%   |

### Clinical specialties (KMLE taxonomy)

| Code | Specialty                                            | Count   | Share  |
|------|-------------------------------------------------------|---------|--------|
| 17   | Internal Medicine                                    | 12,493  | 37.43% |
| 1    | General Surgery                                      | 3,659   | 10.96% |
| 15   | Pediatrics                                           | 3,018   | 9.04%  |
| 14   | Obstetrics and Gynecology                            | 2,438   | 7.30%  |
| 13   | Miscellaneous (Pharmacology, Pathology, Anatomy)     | 2,239   | 6.71%  |
| 4    | Neurology / Neurosurgery                             | 2,134   | 6.39%  |
| 3    | Psychiatry                                           | 1,969   | 5.90%  |
| 8    | Urology                                              | 931     | 2.79%  |
| 11   | Anesthesiology and Pain Medicine                     | 837     | 2.51%  |
| 6    | Ophthalmology                                        | 740     | 2.22%  |
| 2    | Preventive Medicine                                  | 734     | 2.20%  |
| 5    | Dermatology                                          | 695     | 2.08%  |
| 16   | Emergency Medicine                                   | 677     | 2.03%  |
| 7    | Otorhinolaryngology                                  | 532     | 1.59%  |
| 10   | Pathology                                            | 145     | 0.43%  |
| 9    | Radiation Oncology (low-resource)                    | 108     | 0.32%  |
| 12   | Medical Law (low-resource)                           | 30      | 0.09%  |
| **Total** |                                                 | **33,379** | 100% |

> **Note on low-resource specialties.** Medical Law (n=30), Radiation Oncology (n=108), and Pathology (n=145) have notably smaller sample sizes than other specialties. When publishing per-specialty results for these domains, please flag them or exclude them unless your analysis is specifically designed to handle small samples.

---

## Data schema

Each row / JSON object has the following fields:

| Field                    | Type    | Description                                                                 |
|--------------------------|---------|-----------------------------------------------------------------------------|
| `qa_id`                  | integer | Unique identifier for each item                                             |
| `domain`                 | integer | Specialty code (see table above)                                            |
| `q_type`                 | integer | 1 = MCQ (5-option), 2 = short answer, 3 = long answer                       |
| `question`               | string  | Question text in Korean (in `medqa_kr.jsonl`) or English (in `medqa_en.jsonl`) |
| `answer`                 | string  | Answer text in the same language as `question`                              |
| `question_translated`*   | string  | English translation of the question (CSV master file only)                  |
| `answer_translated`*     | string  | English translation of the answer (CSV master file only)                    |

\* Only in the master CSV. The JSONL variants flatten Korean and English into separate files for ML pipelines.

For multiple-choice items (`q_type=1`), the five options are concatenated inside `question` with leading `1)` `2)` `3)` `4)` `5)` markers, and the `answer` field contains the correct option label and explanation.

See [`docs/data_schema.md`](docs/data_schema.md) for full details.

---

## Construction and validation

The dataset was constructed using a modular five-stage pipeline:

1. **Source-data preparation.** A 222-million-token corpus was curated from seven categories of certified sources (clinical guidelines, society publications, government documents, textbooks, peer-reviewed literature, online medical portals, and other clinical materials). 3,380 seed items were authored by 20 licensed Korean physicians.

2. **Prompt construction.** Structured `[task, query, exemplar]` prompts were built per generation step.

3. **Item generation.** GPT-4o (`gpt-4o-2024-08-06`) generated synthetic items grounded in the source corpus, with novelty enforced via sentence-embedding cosine-similarity threshold (< 0.85).

4. **Two-stage validation.** Items first passed automated LLM self-review (four-dimension scoring with bounded self-revision, max depth 3). Items then went through physician review by 30 licensed reviewers, of which 1,500 items were independently reviewed by 3 reviewers for **inter-rater reliability (Fleiss' kappa)**. Conflicts were resolved by majority vote and senior-reviewer adjudication.

5. **Bilingual extension.** Validated Korean items were translated to English by GPT-4o and re-reviewed by the same physician panel.

See [`docs/validation_protocol.md`](docs/validation_protocol.md) for the complete protocol.

---

## Demonstrated utility

The dataset has been used to fine-tune two model families:

| Model     | Benchmark           | Baseline | Best fine-tuned    | Δ              |
|-----------|---------------------|----------|---------------------|----------------|
| HCX-003   | KorMedMCQA (KMLE)   | 45.0%    | 59.3% (MCQ-only)    | **+14.3 pp**   |
| GPT-4o    | KorMedMCQA (KMLE)   | 84.6%    | 90.5% (MCQ-only)    | +5.9 pp        |
| HCX-003   | MedQA (USMLE)       | 49.6%    | 53.8% (LA-only)     | +4.2 pp        |
| GPT-4o    | MedQA (USMLE)       | 86.9%    | 86.9%               | ≈ 0 (ceiling)  |

The substantial gain on HCX-003 (a Korean-optimized mid-size model) reflects the dataset's principal intended use case: **adapting mid-size or regional models for Korean medical education**.

---

## Languages

- **Korean (`ko`)**: 33,379 items. Original authoring language.
- **English (`en`)**: 33,379 items. Machine-translated by GPT-4o and physician-reviewed.

---

## License

This dataset is dedicated to the public domain under the **[Creative Commons CC0 1.0 Universal Public Domain Dedication](https://creativecommons.org/publicdomain/zero/1.0/)**.

You are free to copy, modify, distribute, and use the dataset, including for commercial purposes, without asking permission and without attribution. Attribution is appreciated but not required.

> ⚠️ **Source materials note.** Some materials used during corpus construction were originally distributed under restrictive licenses (e.g., CC-BY-NC, all-rights-reserved). Written re-licensing permissions were obtained from the rights holders prior to incorporation; records are retained by the corresponding author. The resulting items in MedQA-KR/EN are sufficiently transformed (no item reproduces ≥15 consecutive words from any single source) and are released under CC0.

---

## Citation

If you use MedQA-KR/EN in your research, please cite our Data Descriptor:

```bibtex
@article{kim2025medqakren,
  title={MedQA-KR/EN: A Physician-Validated Bilingual Question-Answering Dataset
         for Medical Licensing Examination Education across 17 Clinical Specialties},
  author={Kim, Tong Min and Kim, Chansik and Lee, Youngrong and others
          and Ko, Taehoon},
  journal={Scientific Data},
  year={2025},
  note={Submitted}
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

- **Corresponding author**: Taehoon Ko, Ph.D. — thko@catholic.ac.kr
- **Joint corresponding author**: Youngrong Lee — lyr6830@yuhs.ac
- **Issues / corrections**: Please open an issue on GitHub.
