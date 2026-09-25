# Xác minh lần chạy Gemini — 2026-09-25

## Môi trường và lệnh

Hai entrypoint được chạy trong bản sao tạm của repository bằng `.venv` Python 3.12.9, cùng mã nguồn và raw snapshot của commit chuẩn bị nộp. Kết quả tạo ra được đối chiếu và đưa về `data/results/` cùng `data/reports/` trong repository. Cấu hình dùng `LLM_PROVIDER=gemini`, `LLM_MODEL=gemini-3.5-flash-lite`; API key chỉ đọc từ `.env` cục bộ và không nằm trong Git. Không bật `REFRESH_SOURCE`, `REFRESH_TEST_SET` hoặc `RUN_RAGAS`.

| Lệnh | Exit code | Dấu hiệu hoàn tất |
| --- | ---: | --- |
| `.venv/bin/python script/run_phase1.py` | 0 | `PHASE 1 HOAN TAT.`; 24 bản ghi, 10 câu hỏi, Agent demo ghi hai câu trả lời. |
| `.venv/bin/python script/run_corruption_flow.py` | 0 | `PHASE 2 HOAN TAT.`; 6 loại corruption, 21 dòng bẩn, repair về 24 dòng. |

## Kết quả đối chiếu

| Trạng thái | Dòng dữ liệu | Hit Rate | Mean Token F1 | Judge Accuracy | Mean Judge Score | Judge source | GX | Freshness |
| --- | ---: | ---: | ---: | ---: | ---: | --- | --- | --- |
| Baseline | 24 | 1.0000 | 1.0000 | 1.0000 | 5.0 | `llm` | Pass | Pass, 1/24 quá 180 ngày |
| Corrupted | 21 | 0.5000 | 0.5788 | 0.6000 | 3.4 | `llm` | Fail, 2 expectation | Fail, 6/21 quá 180 ngày |
| Repaired | 24 | 1.0000 | 1.0000 | 1.0000 | 5.0 | `llm` | Pass | Pass, 1/24 quá 180 ngày |

- Cùng 10 ID câu hỏi ở mỗi trạng thái; Hit Rate và Token F1 tính lại từ answers khớp metrics.
- Cả 30 verdict có reasoning do LLM trả về; không có câu nào dùng chuỗi `Fallback heuristic judge used`.
- `data/results/agent_demo_answers.json` có hai câu trả lời Gemini dạng văn bản, độ dài lần lượt 664 và 2136 ký tự trong lần chạy này.
- Ba collection `papers-baseline`, `papers-corrupted`, `papers-repaired` tồn tại trong Chroma đã commit. Manifest có đường dẫn tương đối `data/chroma`.

## Giới hạn vẫn còn

Ragas vẫn `skipped` vì không bật `RUN_RAGAS`. QA benchmark ưu tiên tìm đúng tiêu đề từ câu hỏi nên Hit Rate baseline 1.0 không đo riêng semantic retrieval. Luồng corruption vẫn index dữ liệu sau khi GX fail để đo tác động; đây là chế độ thí nghiệm, không phải quality gate chặn phục vụ production. Điểm Judge có thể biến động khi chạy lại vì là đánh giá bằng LLM.
