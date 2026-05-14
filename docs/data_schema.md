# Data Schema

## File overview

| File                                  | Format | Items   | Size    | Description                                         |
|---------------------------------------|--------|---------|---------|-----------------------------------------------------|
| `data/medqa_kr_en_full.csv`           | CSV    | 33,379  | ~33 MB  | Master file with both Korean and English in one row |
| `data/korean/medqa_kr.jsonl`          | JSONL  | 33,379  | ~21 MB  | Korean items only, one JSON object per line         |
| `data/english/medqa_en.jsonl`         | JSONL  | 33,379  | ~21 MB  | English items only, one JSON object per line        |
| `data/dataset_statistics.json`        | JSON   | —       | ~2 KB   | Aggregate counts                                    |

The CSV uses UTF-8 encoding (no BOM) with `,` delimiter and `"` quote character; newlines inside fields are preserved with `\n`.

## Fields in the master CSV

| Column                | Type    | Required | Notes                                                     |
|-----------------------|---------|----------|-----------------------------------------------------------|
| `qa_id`               | integer | yes      | Unique within the dataset                                 |
| `domain`              | integer | yes      | 1–17, see Domain Codes                                    |
| `q_type`              | integer | yes      | 1 = MCQ; 2 = short answer; 3 = long answer                |
| `question`            | string  | yes      | Question text in Korean                                   |
| `answer`              | string  | yes      | Answer text in Korean                                     |
| `question_translated` | string  | yes      | English translation of the question                       |
| `answer_translated`   | string  | yes      | English translation of the answer                         |

## Fields in the JSONL variants

Each line is a self-contained JSON object:

```json
{
  "qa_id": 61,
  "domain": 4,
  "domain_name": "Neurology/Neurosurgery",
  "q_type": 1,
  "q_type_name": "multiple_choice",
  "question": "<text in this file's primary language>",
  "answer": "<text in this file's primary language>"
}
```

The JSONL variants flatten Korean and English into separate files; the `qa_id` field allows joining across the two files.

## Domain codes

| Code | Korean                    | English short form                                |
|------|---------------------------|----------------------------------------------------|
| 1    | 일반외과                  | General Surgery                                    |
| 2    | 예방의학                  | Preventive Medicine                                |
| 3    | 정신의학                  | Psychiatry                                         |
| 4    | 신경과/신경외과           | Neurology/Neurosurgery                             |
| 5    | 피부과                    | Dermatology                                        |
| 6    | 안과                      | Ophthalmology                                      |
| 7    | 이비인후과                | Otorhinolaryngology                                |
| 8    | 비뇨기과                  | Urology                                            |
| 9    | 방사선종양학과            | Radiation Oncology                                 |
| 10   | 병리과                    | Pathology                                          |
| 11   | 마취통증의학과            | Anesthesiology and Pain Medicine                   |
| 12   | 의료법규                  | Medical Law                                        |
| 13   | 기타 (약리학, 병리, 해부) | Miscellaneous                                      |
| 14   | 산부인과                  | Obstetrics and Gynecology                          |
| 15   | 소아과                    | Pediatrics                                         |
| 16   | 응급의학과                | Emergency Medicine                                 |
| 17   | 내과                      | Internal Medicine                                  |

## Question-type codes

| Code | Format          | Description                                                  |
|------|-----------------|--------------------------------------------------------------|
| 1    | multiple_choice | Five-option single-answer multiple-choice in KMLE/USMLE style |
| 2    | short_answer    | Brief constructed-response (typically 1–3 sentences)         |
| 3    | long_answer     | Extended constructed-response with clinical reasoning        |

## Multiple-choice item conventions

For `q_type=1` items, the five options are concatenated inside the `question` field with leading `1)` `2)` `3)` `4)` `5)` markers separated by newlines. The `answer` field begins with the correct option label (e.g., `3)`) followed by the option text and, for some items, a brief explanation.

Example structure:

```
question: "75세 여성이 혈관성 치매로 진단받고...
1) 옵션 1 텍스트
2) 옵션 2 텍스트
3) 옵션 3 텍스트
4) 옵션 4 텍스트
5) 옵션 5 텍스트"

answer: "3) HBOT와 donepezil 병용 시..."
```

If you need the options programmatically split out, the `1)` `2)` ... markers reliably appear at start-of-line.

## Loading examples

### Python (pandas)

```python
import pandas as pd
df = pd.read_csv("data/medqa_kr_en_full.csv")
# Filter to multiple-choice items in Internal Medicine
internal_mcq = df[(df.domain == 17) & (df.q_type == 1)]
```

### Python (JSONL)

```python
import json
with open("data/korean/medqa_kr.jsonl", encoding="utf-8") as f:
    items = [json.loads(line) for line in f]
```

### HuggingFace Datasets

```python
from datasets import load_dataset
# After uploading to HuggingFace Hub:
ds = load_dataset("<your-org>/MedQA-KR-EN")
```

### R

```r
library(readr)
df <- read_csv("data/medqa_kr_en_full.csv", locale = locale(encoding = "UTF-8"))
```
