# Corruption & Repair Report - Doi Chieu 3 Trang Thai (Day 10)

> Sinh tu dong luc: `2026-09-25T09:00:41.333302+00:00`

## 1. Bang so sanh hieu nang RAG

| Chi so | Baseline (sach) | Corrupted (ban) | Repaired (phuc hoi) | Delta ban vs sach | Delta phuc hoi vs sach |
| :--- | ---: | ---: | ---: | ---: | ---: |
| Hit Rate (Retrieval) | 1.0000 | 0.5000 | 1.0000 | -0.5000 v | +0.0000 = |
| Token F1 (trung binh) | 1.0000 | 0.5788 | 1.0000 | -0.4212 v | +0.0000 = |
| LLM Judge Accuracy | 1.0000 | 0.6000 | 1.0000 | -0.4000 v | +0.0000 = |
| LLM Judge Score (1-5) | 5 | 3.2000 | 5 | -1.8000 v | +0.0000 = |
| So cau hoi danh gia | 10 | 10 | 10 | - | - |

## 2. Data Quality Gate (Great Expectations 1.x)

### Tren du lieu bi tiem loi (Corrupted)

- Ket qua Quality Gate (GX 1.x): **FAIL**
- So expectation danh gia: 6
- So expectation that bai: 2

| Expectation | Cot | Ket qua | Gia tri quan sat |
| :--- | :--- | :---: | :--- |
| expect_table_row_count_to_be_between | - | PASS | 21 |
| expect_column_values_to_not_be_null | paper_id | PASS | n/a |
| expect_column_values_to_be_unique | paper_id | FAIL | n/a |
| expect_column_values_to_not_be_null | title | PASS | n/a |
| expect_column_values_to_not_be_null | text_for_embedding | PASS | n/a |
| expect_column_value_lengths_to_be_between | summary | FAIL | n/a |

### Sau khi phuc hoi (Repaired)

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

## 3. Freshness SLA

### Corrupted

- Ngay xuat ban moi nhat: `2026-06-09`
- Ngay xuat ban cu nhat: `2025-06-10`
- So ban ghi qua han (> 180 ngay): 6/21 (0.2857)
- Trang thai SLA: **FAIL**
- Canh bao: CANH BAO - 6/21 ban ghi qua han 180 ngay.

### Repaired

- Ngay xuat ban moi nhat: `2026-07-22`
- Ngay xuat ban cu nhat: `2026-03-28`
- So ban ghi qua han (> 180 ngay): 1/24 (0.0417)
- Trang thai SLA: **PASS**
- Canh bao: OK - du lieu con tuoi.

## 4. Phan tich & Ket luan

- **Silent Failure:** tren du lieu bi tiem loi, Agent van tra loi troi chay nhung Hit Rate tut tu 1.0000 xuong 0.5000 - khong he co exception nao duoc nem ra.
- **Quality Gate bat duoc loi:** GX tren du lieu ban cho ket qua **FAIL** voi 2 expectation that bai.
- **Freshness canh bao:** CANH BAO - 6/21 ban ghi qua han 180 ngay.
- **Idempotent Repair:** tai tao tu `data/raw/crossref_records.json` da dua Quality Gate ve **PASS** va Hit Rate ve **1.0000** - phuc hoi 100% phong do baseline.
- Chay lai `script/run_corruption_flow.py` bao nhieu lan cung cho ket qua phuc hoi giong nhau vi buoc repair doc lai tu ban sao luu thô, khong phu thuoc trang thai hien tai.
