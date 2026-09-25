# Xác minh lần chạy cuối — 2026-09-25

## Lệnh và kết quả quan sát

Chạy tại thư mục gốc repository với Python 3.12.9 trong `.venv`:

| Lệnh | Exit code | Dấu hiệu hoàn tất |
| --- | ---: | --- |
| `.venv/bin/python script/run_phase1.py` | 0 | `PHASE 1 HOAN TAT.`; 24 raw/clean records, collection `papers-baseline`, 10 câu hỏi, Hit Rate 1.0000, Token F1 1.0000. |
| `.venv/bin/python script/run_corruption_flow.py` | 0 | `PHASE 2 HOAN TAT.`; 6 loại corruption, 21 dòng bẩn, repair về 24 dòng; bảng so sánh ba trạng thái được in ra. |

Lần thử đầu của pha 1 trong sandbox bị gián đoạn sau các lần retry DNS tới Hugging Face. Hai kết quả exit code 0 ở trên là lần chạy lại ngoài sandbox, sau khi mô hình tải thành công. Các lệnh không bật `REFRESH_SOURCE`, `REFRESH_TEST_SET` hoặc `RUN_RAGAS`, nên dùng raw records và test set sẵn có.

## Đối chiếu artifact sau khi chạy

| Trạng thái | Dòng dữ liệu | Hit Rate | Mean Token F1 | GX | Freshness |
| --- | ---: | ---: | ---: | --- | --- |
| Baseline | 24 | 1.0000 | 1.0000 | Pass | Pass, 1/24 quá 180 ngày |
| Corrupted | 21 | 0.5000 | 0.5788 | Fail, 2 expectation | Fail, 6/21 quá 180 ngày |
| Repaired | 24 | 1.0000 | 1.0000 | Pass | Pass, 1/24 quá 180 ngày |

- Cùng 10 ID câu hỏi ở `baseline_answers.json`, `corrupted_answers.json` và `repaired_answers.json`. Hit Rate và Token F1 tính lại từ answers khớp các metrics JSON.
- `corruption_log.json` ghi đủ `drop_latest_records`, `blank_summary`, `inject_noise`, `truncate_title`, `stale_date`, `duplicate_rows`.
- SQLite Chroma chứa ba collection `papers-baseline`, `papers-corrupted`, `papers-repaired`. Mỗi collection trỏ đến một vector segment hiện có trên đĩa. Khi commit SQLite mới, cần đưa cả ba thư mục segment mà SQLite hiện tham chiếu vào cùng commit.

## Giới hạn quan sát trong lần chạy

- Cả 30 verdict trong ba file answers dùng `Fallback heuristic judge used because the LLM evaluator was unavailable.`. Các trường `judge_accuracy` và `mean_judge_score` vì vậy là **điểm heuristic dự phòng**, không phải đánh giá trực tiếp bằng LLM. Ragas có trạng thái `skipped`.
- Agent demo ở pha 1 bị bỏ qua do provider trả `404 NOT_FOUND` cho `gemini-2.5-flash`. Exit code 0 chỉ chứng minh pipeline dữ liệu và evaluation dự phòng hoàn tất; không chứng minh agent demo với Gemini chạy thành công.
- QA benchmark ưu tiên tra cứu đúng tiêu đề trích từ câu hỏi trước khi dùng thứ tự semantic search. Do đó baseline Hit Rate 1.0 không đo riêng chất lượng semantic retrieval.
- Luồng corruption vẫn index dữ liệu sau khi GX báo fail để đo tác động. Quality gate hiện phát cảnh báo, chưa thực sự chặn index trong luồng thí nghiệm.

Đối chiếu trực tiếp bằng các file `data/results/*_metrics.json`, `data/results/*_answers.json`, `data/quality/*.json`, `data/results/corruption_log.json` và hai báo cáo trong `data/reports/`.
