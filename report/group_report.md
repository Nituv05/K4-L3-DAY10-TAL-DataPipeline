# Báo cáo nhóm — Day 10: Data Pipeline & Data Observability

## 1. Thông tin bài nộp

| Thông tin | Nội dung |
| --- | --- |
| Khóa/Lớp | K4-L3-DAY10 |
| Tên nhóm | TAL |
| Repository | https://github.com/Nituv05/K4-L3-DAY10-TAL-DataPipeline |
| Ngày cập nhật kết quả kỹ thuật | 2026-09-25 |

### Thành viên và phân công

| Thành viên | MSSV | Phần việc có bằng chứng | Trạng thái xác nhận |
| --- | --- | --- | --- |
| Lê Tuấn Anh | 2A202602952 | `1629c3e`: hoàn thiện ingestion, cleaning, GX, test set, corruption, orchestration và reporting | `report/2A202602952_LeTuanAnh.md` |
| Vũ Thường Tín | 2A202602955 | `e608ed8`: đưa 47 artifact vào Git; `2aee2f9`: xác minh Chroma/metrics và cập nhật báo cáo | `report/2A202602955_VuThuongTin.md` |
| Trần Quốc Bảo Long | 2A202602696 | `90d4c4d`: xây UI demo, server chạy cục bộ, kiểm thử và hướng dẫn | Chưa có báo cáo riêng; Long cần tự hoàn thành |

Phân công chi tiết nằm trong `docs/TEAM.md`. Báo cáo cá nhân của Long còn thiếu; `report/individual_report.md` là mẫu dùng chung. Quyền tác giả trong bảng dựa trên commit và file thực tế, không suy từ tên package.

## 2. Tóm tắt kết quả

Pipeline dùng 24 metadata bài báo từ raw snapshot Crossref, làm sạch văn bản và tạo `text_for_embedding` gồm năm phần. Mô hình `sentence-transformers/all-MiniLM-L6-v2` tạo vector cho ChromaDB. Bộ benchmark có 10 câu hỏi thuộc bốn loại: summary, authors, date và categories. Baseline đạt Retrieval Hit Rate 1.0000 và mean Token F1 1.0000. Sáu thao tác corruption được áp dụng cùng lúc, làm dữ liệu còn 21 dòng. Great Expectations báo fail ở tính duy nhất của `paper_id` và độ dài `summary`; freshness cũng fail vì 6/21 bản ghi quá 180 ngày. Hit Rate giảm xuống 0.5000, Token F1 xuống 0.5788. Repair đọc lại raw records, tạo 24 dòng, đưa quality và freshness về pass; hai chỉ số đánh giá phục hồi đúng mức baseline. Bảo Long xây UI cục bộ để trình bày các artifact và gọi hai entrypoint. Hai entrypoint đã được chạy lại với exit code 0; chi tiết ở `data/reports/run_verification.md`. Điểm judge hiện dùng heuristic dự phòng do Gemini evaluator không khả dụng; Ragas được bỏ qua. Agent demo không chạy thành công vì provider trả 404 cho `gemini-2.5-flash`. Những giới hạn này phải được nêu khi trình diễn kết quả.

## 3. Kiến trúc và luồng dữ liệu

```text
Crossref snapshot -> raw records -> cleaning -> GX và freshness
    -> MiniLM/Chroma baseline -> 10 câu hỏi -> baseline metrics
    -> 6 lỗi corruption -> GX/freshness + Chroma corrupted -> corrupted metrics
    -> rebuild từ raw records -> GX/freshness + Chroma repaired -> repaired metrics
    -> báo cáo đối chiếu
```

| Khối | Input | Xử lý | Output | Owner có bằng chứng |
| --- | --- | --- | --- | --- |
| Ingestion | `data/raw/crossref_response.json` hoặc Crossref API | Parse/fallback, lưu raw records | `data/raw/crossref_records.json` | Lê Tuấn Anh, `1629c3e` |
| Cleaning | Raw records | Khử markup, chuẩn hóa schema, deduplicate, tạo cột dẫn xuất | `data/clean/papers_clean.csv` và JSON | Lê Tuấn Anh, `1629c3e` |
| Observability | DataFrame từng trạng thái | GX 1.x và freshness SLA | `data/quality/*.json` | Lê Tuấn Anh viết module; Tín lưu artifact |
| Embedding/index | `text_for_embedding` | MiniLM, ba collection Chroma riêng | `data/chroma/`, `data/embeddings/` | Mã retrieval của starter/commit khác; Tín lưu index artifact |
| Evaluation | Cùng 10 câu hỏi, index từng trạng thái | Hit Rate, Token F1, heuristic judge khi LLM lỗi | `data/results/*_answers.json`, `*_metrics.json` | Lê Tuấn Anh viết test set; Tín lưu và đối chiếu kết quả |
| Repair/reporting | Raw records, metrics và quality | Dựng lại dữ liệu, lập bảng đối chiếu | `data/reports/*.md` | Lê Tuấn Anh viết pipeline; Tín xác minh báo cáo |
| UI demo | Artifact trong `data/` | Dashboard cục bộ, gọi hai entrypoint từ giao diện | `ui/`, `script/run_ui.py`, `script/test_ui.py` | Trần Quốc Bảo Long, `90d4c4d` |

## 4. Cách tái hiện kết quả

| Cấu hình | Giá trị dùng trong lần chạy |
| --- | --- |
| Python | 3.12.9 trong `.venv` |
| LLM provider/model | `gemini` / `gemini-2.5-flash` theo báo cáo pha 1; model trả 404 ở demo |
| Embedding | `sentence-transformers/all-MiniLM-L6-v2` |
| Raw records | 24 từ snapshot đã lưu; không bật `REFRESH_SOURCE` |
| Retrieval `top_k` | 4 |
| Freshness | `age_days > 180`; fail khi tỷ lệ quá hạn > 25% |
| Test set | 10 câu cố định; không bật `REFRESH_TEST_SET` |
| Ragas | Không bật `RUN_RAGAS` |

```bash
python -m pip install -e .
.venv/bin/python script/run_phase1.py
.venv/bin/python script/run_corruption_flow.py
python script/run_ui.py
```

| Lệnh | Kết quả đã quan sát | Bằng chứng |
| --- | --- | --- |
| `script/run_phase1.py` | Exit code 0, in `PHASE 1 HOAN TAT.` | `data/reports/run_verification.md`, baseline metrics/report |
| `script/run_corruption_flow.py` | Exit code 0, in `PHASE 2 HOAN TAT.` | `data/reports/run_verification.md`, ba metrics và comparison report |
| `.venv/bin/python -m unittest discover -s script -p test_ui.py` | 2/2 test pass trên Python 3.12.9 | Test HTTP assets, snapshot, quyền truy cập và khóa job bằng mock; không gọi LLM |
| `node --check ui/app.js` | Pass | Kiểm tra cú pháp JavaScript |

Lệnh UI mở dashboard cục bộ tại `http://127.0.0.1:8765`; UI đọc artifact đã lưu và chỉ gọi pipeline khi người dùng bấm nút chạy. Xem `ui/README.md` để trình diễn và dừng server bằng Ctrl+C. Kết quả hai entrypoint ở bảng được xác minh từ terminal, không suy ra từ việc UI hiển thị số liệu.

Lần thử pha 1 trong sandbox gặp lỗi DNS khi Hugging Face kiểm tra model, nên hai lệnh thành công ở bảng là lần chạy lại ngoài sandbox. Không có API key nào được ghi trong báo cáo.

## 5. Ingestion, cleaning và data contract

Raw snapshot có 24 records. `fetch_source_records()` hỗ trợ gọi Crossref khi bật `REFRESH_SOURCE`, retry cho lỗi 429/5xx rồi fallback sang snapshot; lần tái hiện ở đây đọc records đã lưu. Mỗi record có `paper_id`, `title`, `summary`, `authors`, `categories`, `published` và các metadata khác.

| Trường clean | Vai trò | Quy tắc |
| --- | --- | --- |
| `paper_id` | Khóa duy nhất/ground truth ID | Loại ID rỗng, deduplicate theo ID |
| `title` | Tìm kiếm, nhận diện bài | Chuẩn hóa whitespace, tối thiểu 8 ký tự |
| `summary` | Nội dung trả lời | Bỏ markup JATS/XML, tối thiểu 30 ký tự |
| `published` | Ngày và freshness | Chuẩn ISO, loại ngày rỗng |
| `age_days` | Độ tuổi dữ liệu | `run_date - published` theo ngày |
| `text_for_embedding` | Input MiniLM | Ghép Title, Authors, Published, Categories, Summary |

24 raw records tạo 24 dòng sạch trong lần chạy được ghi nhận. Trạng thái bẩn có 21 dòng vì bỏ 5 bản ghi mới rồi nhân bản 2 dòng. ID trong Chroma là `paper_id` kèm vị trí dòng; metadata giữ `paper_id` để tính Retrieval Hit Rate.

## 6. Evaluation setup

| Thành phần | Cấu hình |
| --- | --- |
| Test set | `data/eval/test_set.json`, 10 câu: summary 3, authors 3, date 2, categories 2 |
| Ground truth | Một `ground_truth_doc_ids` chứa `paper_id` tương ứng mỗi câu |
| Vector store | `papers-baseline`, `papers-corrupted`, `papers-repaired` |
| Embedding và top_k | MiniLM L6 v2, top 4 |
| Câu trả lời benchmark | `retrieval/qa.py` lấy metadata/summary từ tài liệu đứng đầu; ưu tiên exact title lookup khi câu hỏi chứa tiêu đề trong dấu nháy |
| Chấm điểm | Hit Rate và Token F1; judge có fallback heuristic khi LLM evaluator lỗi |

Cả ba trạng thái dùng cùng `test_set.json` và cùng 10 ID câu hỏi, nên chênh lệch metrics không do thay đề. Do exact title lookup, Hit Rate 1.0 ở baseline không chứng minh riêng hiệu năng semantic search.

## 7. Kết quả baseline

Raw, clean CSV/JSON, embedding manifest, Chroma collection, test set, baseline metrics, quality/freshness report và `data/reports/phase1_report.md` đều có trên đĩa. Chroma baseline có 24 documents theo manifest.

| Metric | Giá trị | Giới hạn diễn giải |
| --- | ---: | --- |
| `retrieval_hit_rate` | 1.0000 | Có exact title lookup trong QA benchmark. |
| `mean_token_f1` | 1.0000 | So khớp token của câu trả lời với ground truth. |
| `judge_accuracy` | 1.0000 | Heuristic dự phòng, không phải LLM judge trực tiếp. |
| `mean_judge_score` | 5.0000 | Cùng giới hạn heuristic. |
| Ragas | Skipped | `RUN_RAGAS` không được bật. |

## 8. Data quality và freshness

GX 1.x chạy trên DataFrame qua ephemeral context và pandas data source. Suite có sáu expectation thuộc bốn loại: row count 5–5000, not null ở `paper_id`/`title`/`text_for_embedding`, unique `paper_id`, và summary dài 30–20000 ký tự. Baseline pass cả sáu. Corrupted fail hai expectation: uniqueness của `paper_id` và độ dài `summary`; repaired pass cả sáu.

Freshness đo số dòng có `age_days > 180`. Baseline là 1/24 = 4.17% (pass), corrupted 6/21 = 28.57% (fail), repaired 1/24 = 4.17% (pass), với ngưỡng cảnh báo > 25%. Quality gate trong luồng thí nghiệm **phát cảnh báo nhưng không chặn index dữ liệu bẩn**, nhằm cho phép đo mức suy giảm; chưa có cơ chế chặn production.

## 9. Corruption scenarios và repair

| Loại lỗi | Số dòng bị tác động | Hiệu ứng quan sát được |
| --- | ---: | --- |
| `drop_latest_records` | 5 | Mất 20% bản ghi mới nhất. |
| `blank_summary` | 2 | Summary rỗng; góp phần làm fail độ dài summary. |
| `inject_noise` | 2 | Ký tự rác trong summary. |
| `truncate_title` | 2 | Title dưới 8 ký tự. |
| `stale_date` | 3 | Ngày xuất bản lùi 365 ngày. |
| `duplicate_rows` | 2 | `paper_id` không còn duy nhất. |

`data/results/corruption_log.json` ghi sự kiện và ID liên quan. Suite áp dụng cả sáu lỗi cùng lúc; hiện chưa có số liệu riêng để kết luận lỗi nào làm giảm Hit Rate nhiều nhất. Repair dùng `data/raw/crossref_records.json` làm nguồn đáng tin cậy, chạy lại cleaning và dựng collection riêng. Với cùng raw records và run date, logic cleaning cho cùng output; kết quả repair đã quay về 24 dòng và các metrics baseline.

## 10. So sánh ba trạng thái

| Metric/signal | Baseline | Corrupted | Repaired | Thay đổi khi bẩn | Mức phục hồi |
| --- | ---: | ---: | ---: | ---: | ---: |
| Retrieval Hit Rate | 1.0000 | 0.5000 | 1.0000 | -0.5000 | Đủ mức baseline |
| Mean Token F1 | 1.0000 | 0.5788 | 1.0000 | -0.4212 | Đủ mức baseline |
| Judge Accuracy (heuristic) | 1.0000 | 0.6000 | 1.0000 | -0.4000 | Đủ mức baseline |
| Judge Score (heuristic) | 5.0 | 3.2 | 5.0 | -1.8 | Đủ mức baseline |
| GX | Pass | Fail | Pass | 2 expectation fail | Pass |
| Freshness | Pass | Fail | Pass | 4.17% → 28.57% quá hạn | 4.17% quá hạn |

Corruption → GX/freshness chuyển sang fail → retrieval và Token F1 giảm trên cùng test set. Repair từ raw → GX/freshness trở lại pass → hai metrics trở lại baseline. Không thể tách quan hệ nhân quả của từng lỗi riêng vì sáu lỗi được tiêm đồng thời. Một vài câu corrupted có `retrieval_hit=False` nhưng `token_f1=1.0`, nên trùng đáp án ngắn không bảo đảm đã lấy đúng document.

## 11. Vấn đề tích hợp quan trọng

- **Triệu chứng:** Pha 1 chạy xong nhưng agent demo in thông báo bị bỏ qua; judge trong answers dùng fallback.
- **Nguyên nhân quan sát được:** Gemini provider trả `404 NOT_FOUND` cho model `gemini-2.5-flash` trong lần chạy demo. Lệnh đầu trong sandbox cũng gặp lỗi DNS tới Hugging Face.
- **Cách xử lý trong lần nghiệm thu:** Chạy lại hai entrypoint ngoài sandbox để model embedding tải được; giữ nguyên cấu hình khi đối chiếu dữ liệu, ghi rõ Gemini demo và LLM judge chưa thành công. Chưa thay model hoặc bịa số liệu LLM.
- **Xác minh:** Hai exit code 0, `data/reports/run_verification.md`, và `reasoning` trong 30 answers JSON.

## 12. Giới hạn và hướng cải thiện

| Giới hạn | Ảnh hưởng | Cách cải thiện có thể kiểm chứng |
| --- | --- | --- |
| Gemini model cấu hình trả 404; judge fallback, demo không có output | Không có bằng chứng LLM judge/agent demo thực | Chọn model được tài khoản hỗ trợ, chạy lại, kiểm tra `reasoning` và file demo thực tế. |
| Ragas skipped | Chưa có faithfulness/context metrics | Bật `RUN_RAGAS=1` khi evaluator sẵn sàng, lưu kết quả hoặc lỗi nguyên văn. |
| Exact title lookup trong benchmark | Hit Rate baseline không đo riêng semantic retrieval | Thêm câu hỏi paraphrase không chứa exact title; so sánh metrics. |
| GX fail nhưng vẫn index | Luồng này chưa phải quality gate chặn serving | Tách chế độ demo corruption và production; ở production dừng trước index khi `gate_passed=False`. |
| Manifest lưu đường dẫn Chroma tuyệt đối | Nạp trực tiếp trên máy khác có thể lỗi | Dùng đường dẫn theo project root; kiểm tra trên checkout sạch. |
| Chưa tách từng corruption | Chưa quy được mức suy giảm cho từng loại lỗi | Ablation từng lỗi trên cùng test set, lưu metrics riêng. |
| UI đọc artifact theo trạng thái đã lưu | Màn hình không tự chứng minh lần chạy pipeline hoặc tính nhất quán giữa nhiều file | Kiểm tra timestamp/exit code và đối chiếu `data/reports/run_verification.md`; không chạy hai job đồng thời. |

## 13. Checklist trước khi nộp

- [x] Hai entrypoint đã chạy lại với exit code 0; số liệu khớp artifacts trong lần kiểm tra.
- [x] Baseline, corrupted, repaired dùng cùng 10 câu hỏi; metrics khớp answers.
- [x] Quality/freshness conclusions khớp JSON tương ứng.
- [x] Giới hạn heuristic judge, Ragas và agent demo được nêu rõ.
- [x] UI test 2/2 pass và JavaScript qua kiểm tra cú pháp.
- [x] Tên, MSSV và phần việc có bằng chứng của ba thành viên đã được ghi trong `docs/TEAM.md`.
- [ ] Long tự hoàn thành báo cáo riêng `report/2A202602696_TranQuocBaoLong.md`; Tín và Lê đã có file theo MSSV.
- [x] Commit `2aee2f9` chứa SQLite cùng ba segment hiện hành, metrics và báo cáo; đã push lên `origin/tinvt`.
- [x] Commit artifact `2aee2f9` đã vào `main` qua PR #4; phần cập nhật báo cáo hiện ở nhánh `tinvt`.
- [ ] Merge phần cập nhật báo cáo từ `tinvt` vào nhánh nộp `main` sau khi nhóm duyệt.
- [ ] Kiểm tra Contributors của nhánh mặc định và từng người nộp link lên LMS.
