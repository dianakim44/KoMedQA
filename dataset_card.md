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
pretty_name: MedQA-KR/EN
tags:
  - medical
  - medical-education
  - licensing-examination
  - KMLE
  - USMLE
  - bilingual
  - clinical-knowledge
---

# Dataset Card for MedQA-KR/EN

## Dataset Summary

MedQA-KR/EN is a **physician-validated bilingual (Korean–English) question-answering dataset** of **33,379 examination-style items** spanning **17 clinical specialties** aligned with the Korean Medical Licensing Examination (KMLE) subject taxonomy. The dataset combines **3,380 physician-authored items** with **29,999 synthetically generated items**, all validated by **30 licensed physicians** under a documented inter-rater-reliability protocol.

The principal intended use is medical-education research and language-model adaptation for medical-education applications, with particular relevance to Korean-language medical training where openly licensed bilingual resources are scarce.

## Supported Tasks

- **Multiple-choice question answering** (`q_type=1`, n=26,355): five-option items in the style of national licensing examinations.
- **Short-answer question answering** (`q_type=2`, n=3,444): brief constructed-response items.
- **Long-answer question answering** (`q_type=3`, n=3,580): extended-response items eliciting clinical reasoning.

The dataset is suitable for supervised fine-tuning, instruction tuning, retrieval-augmented question answering, and evaluation of medical knowledge in large language models.

## Languages

- Korean (`ko`): 33,379 items (original authoring language)
- English (`en`): 33,379 items (machine-translated by GPT-4o, physician-reviewed)

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

| Field         | Type    | Description                                                                   |
|---------------|---------|-------------------------------------------------------------------------------|
| `qa_id`       | integer | Unique identifier                                                             |
| `domain`      | integer | Specialty code (1–17, see Domain Codes below)                                 |
| `domain_name` | string  | Human-readable specialty name (JSONL only)                                    |
| `q_type`      | integer | 1 = multiple choice, 2 = short answer, 3 = long answer                        |
| `q_type_name` | string  | Human-readable question-format name (JSONL only)                              |
| `question`    | string  | Question text in the file's primary language                                  |
| `answer`      | string  | Answer text in the file's primary language                                    |

### Domain Codes and Counts

| Code | Specialty                                            | Count   |
|------|-------------------------------------------------------|---------|
| 1    | General Surgery                                      | 3,659   |
| 2    | Preventive Medicine                                  | 734     |
| 3    | Psychiatry                                           | 1,969   |
| 4    | Neurology / Neurosurgery                             | 2,134   |
| 5    | Dermatology                                          | 695     |
| 6    | Ophthalmology                                        | 740     |
| 7    | Otorhinolaryngology                                  | 532     |
| 8    | Urology                                              | 931     |
| 9    | Radiation Oncology                                   | 108     |
| 10   | Pathology                                            | 145     |
| 11   | Anesthesiology and Pain Medicine                     | 837     |
| 12   | Medical Law                                          | 30      |
| 13   | Miscellaneous (Pharmacology, Pathology, Anatomy)     | 2,239   |
| 14   | Obstetrics and Gynecology                            | 2,438   |
| 15   | Pediatrics                                           | 3,018   |
| 16   | Emergency Medicine                                   | 677     |
| 17   | Internal Medicine                                    | 12,493  |

### Data Splits

The dataset is released as a single split. Users designing fine-tuning or evaluation experiments are encouraged to construct their own splits stratified by specialty and format. For evaluation against real licensing-examination items, we recommend using the held-out external benchmarks **KorMedMCQA** (Korean) and **MedQA** (English) rather than withholding items from MedQA-KR/EN.

## Dataset Creation

### Source Data

The synthetic items were derived from a curated 222-million-token corpus of certified medical sources across seven categories: peer-reviewed literature, online medical information portals, government-issued clinical guidelines, professional-society guidelines, international-organization guidelines, medical textbooks, and other clinical materials (e.g., procedural consent forms).

### Annotation Process

- **Seed items**: 3,380 question-answer pairs were authored by 20 licensed Korean physicians (5 from each of 4 participating tertiary-care hospitals).
- **Synthetic generation**: GPT-4o (`gpt-4o-2024-08-06`) generated items grounded in source-corpus passages, using structured `[task, query, exemplar]` prompts.
- **Two-stage validation**: Automated LLM self-review with bounded self-revision (max depth 3), followed by review by 30 licensed physicians not involved in seed authoring. 1,500 items (5%) were independently triple-reviewed for inter-rater-reliability quantification (Fleiss' kappa).
- **Bilingual extension**: Korean items were translated by GPT-4o and re-reviewed by the same physician panel for clinical appropriateness.

### Personal and Sensitive Information

The dataset does not contain real patient data. All clinical vignettes are generic, fictional, or derived from textbook cases.

## Considerations for Using the Data

### Social Impact

The dataset is designed to support medical education and language-model adaptation for medical-education applications, contributing to United Nations Sustainable Development Goal 4 (Quality Education). The bilingual coverage particularly serves Korean medical education, which has had limited access to openly licensed examination-style resources.

### Biases

- The 17 specialties are weighted toward common KMLE topics; the distribution is heavy-tailed (Internal Medicine: 12,493; Medical Law: 30).
- The dataset reflects Korean clinical practice conventions and pharmaceutical availability as of 2024–2025. Items may not always reflect practice patterns in other jurisdictions.
- English items were produced through machine translation followed by physician review; subtle stylistic differences from natively-authored English may remain.

### Limitations

- The dataset does not include multimodal content (images, ECG traces, pathology slides). Pair with multimodal resources for vision-language work.
- Item difficulty has not been empirically calibrated against actual examinee performance. For IRT-based studies, supplement with KorMedMCQA.
- Three specialties (Medical Law: 30, Radiation Oncology: 108, Pathology: 145) have small sample sizes; per-specialty conclusions for these should be drawn with caution.

## Additional Information

### Dataset Curators

The dataset was constructed by a multi-institution team led by The Catholic University of Korea, with clinical collaborators from Samsung Medical Center, Seoul National University Hospital, Yonsei University Hospital, and four additional Korean tertiary-care hospitals.

### Licensing Information

Released under the [Creative Commons CC0 1.0 Universal Public Domain Dedication](https://creativecommons.org/publicdomain/zero/1.0/). Free for any use including commercial.

### Citation Information

```bibtex
@article{kim2025medqakren,
  title={MedQA-KR/EN: A Physician-Validated Bilingual Question-Answering Dataset
         for Medical Licensing Examination Education across 17 Clinical Specialties},
  author={Kim, Tong Min and others and Ko, Taehoon},
  journal={Scientific Data},
  year={2025},
  note={Submitted}
}
```

### Contributions

Contributions and corrections are welcome via GitHub Issues. Please verify clinical content against current guidelines before relying on individual items in patient-facing applications.
