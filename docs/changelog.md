# Changelog

All notable changes to MedQA-KR/EN will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0.0] — 2025-09-03

### Added
- Initial public release of MedQA-KR/EN
- 33,379 Korean items spanning 17 KMLE-aligned clinical specialties
- 33,379 English translations (machine-translated by GPT-4o, physician-reviewed)
- Three question formats: multiple-choice (5 options), short answer, long answer
- Master CSV and per-language JSONL files
- Released under Creative Commons CC0 1.0 Universal

### Notes on data preparation
- One internal record without an English translation (a single short-answer item in Pediatrics) was excluded from the public release so that the Korean and English subsets are exactly aligned at 33,379 items each.

## Planned

- [ ] v1.1 — Add empirical item-difficulty calibration where examinee data is available
- [ ] v1.2 — Add specialty-level cross-walks to USMLE Step 1 / Step 2 content outlines
- [ ] v1.3 — Add an evaluation-split file with held-out items for benchmark use
