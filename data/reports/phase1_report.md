# Phase 1 Report - Baseline Data Pipeline (Day 10)

> Sinh tu dong luc: `2026-09-25T08:15:11.963262+00:00`

## 1. Nguon du lieu & Lineage

| Hang muc | Gia tri |
| :--- | :--- |
| source_api | Crossref REST API |
| source_query | agentic retrieval augmented generation large language model |
| source_filter | from-pub-date:2026-03-29,has-abstract:true |
| raw_api_response | /Users/tinvu/Documents/LAB_VINUNI/K4-L3-DAY10-TAL-DataPipeline/data/raw/crossref_response.json |
| raw_records_json | /Users/tinvu/Documents/LAB_VINUNI/K4-L3-DAY10-TAL-DataPipeline/data/raw/crossref_records.json |
| raw_records | 24 |
| clean_rows | 24 |
| embedding_model | sentence-transformers/all-MiniLM-L6-v2 |
| collection_name | papers-baseline |
| top_k | 4 |
| llm_provider | gemini |
| llm_model | gemini-2.5-flash |
| run_date | 2026-09-25T08:13:00.803031+00:00 |

## 2. Chi so danh gia RAG (Baseline)

| Chi so | Gia tri |
| :--- | ---: |
| Hit Rate (Retrieval) | 1.0000 |
| Token F1 (trung binh) | 1.0000 |
| LLM Judge Accuracy | 1.0000 |
| LLM Judge Score (1-5) | 5 |
| So cau hoi danh gia | 10 |

**Ragas:** `skipped=Set RUN_RAGAS=1 to enable the slower Ragas pass.`

## 3. Data Quality Gate

### Great Expectations 1.x - Baseline

- Ket qua Quality Gate (GX 1.x): **PASS**
- So expectation danh gia: 6
- So expectation that bai: 0

| Expectation | Cot | Ket qua | Gia tri quan sat |
| :--- | :--- | :---: | :--- |
| expect_table_row_count_to_be_between | - | PASS | 24 |
| expect_column_values_to_not_be_null | paper_id | PASS | n/a |
| expect_column_values_to_be_unique | paper_id | PASS | n/a |
| expect_column_values_to_not_be_null | title | PASS | n/a |
| expect_column_values_to_not_be_null | text_for_embedding | PASS | n/a |
| expect_column_value_lengths_to_be_between | summary | PASS | n/a |

## 4. Freshness SLA

### Do tuoi du lieu - Baseline

- Ngay xuat ban moi nhat: `2026-07-22`
- Ngay xuat ban cu nhat: `2026-03-28`
- So ban ghi qua han (> 180 ngay): 1/24 (0.0417)
- Trang thai SLA: **PASS**
- Canh bao: OK - du lieu con tuoi.

## 5. Ket luan

- Quality Gate: **PASS** | Freshness SLA: **PASS**
- Hit Rate baseline dat **1.0000**, Token F1 dat **1.0000**.
- Day la moc tham chieu de doi chieu voi pha Corruption va Repair.
