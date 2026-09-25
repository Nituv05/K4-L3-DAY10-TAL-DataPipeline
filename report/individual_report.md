# Báo cáo cá nhân — Vũ Thường Tín

## 1. Thông tin cá nhân

| Thông tin | Nội dung |
| --- | --- |
| Họ và tên | Vũ Thường Tín |
| MSSV | 2A202602955 |
| Khóa/Lớp | K4-L3-DAY10 |
| Tên nhóm | TAL (theo tên repository; cần đối chiếu tên nhóm chính thức) |
| Vai trò chính | Đóng gói artifact và kiểm tra bằng chứng kết quả tích hợp |
| Repository | https://github.com/Nituv05/K4-L3-DAY10-TAL-DataPipeline |
| Ngày lập báo cáo | 2026-09-25 |
| Commit đóng góp | `e608ed8` — `Update repo`; đã nằm trong merge commit `1d17cdc` trên `origin/main` |

## 2. Vai trò và phạm vi công việc

### Phần việc có bằng chứng trong lịch sử commit

| Deliverable | File đưa vào Git | Input | Output bàn giao | Trạng thái |
| --- | --- | --- | --- | --- |
| Dữ liệu và benchmark | `data/clean/`, `data/eval/test_set.json` | Output pipeline, raw snapshot đã có trong repo | Dữ liệu ba trạng thái và 10 câu hỏi | Đã commit trên `tinvt` |
| Vector store | `data/chroma/`, `data/embeddings/` | Dữ liệu đã xử lý, MiniLM | Chroma DB và ba manifest | Đã commit trên `tinvt` |
| Kết quả và báo cáo | `data/quality/`, `data/results/`, `data/reports/` | Output GX, freshness, evaluation | Quality reports, metrics, answers, corruption log, Markdown reports | Đã commit trên `tinvt` |

Commit `e608ed8` thêm 47 artifact trong `data/`, không sửa `src/` hoặc `script/`. Tôi nhận trách nhiệm với phần đưa artifact vào Git và đối chiếu kết quả, không nhận quyền tác giả của các module do thành viên khác commit. Commit này đã nằm trong merge commit `1d17cdc` trên `origin/main`; trạng thái hiển thị trong GitHub Contributors vẫn cần kiểm tra.

### Hỗ trợ tích hợp

| Hoạt động | Thành phần được hỗ trợ | Kết quả |
| --- | --- | --- |
| Đưa output pipeline vào Git | Nhóm và người nghiệm thu | Có thể kiểm tra bộ artifact qua `git show --name-only e608ed8`. |
| Đối chiếu ba trạng thái | Báo cáo và demo | Cùng 10 ID câu hỏi; metrics khớp với answers đã lưu. |

## 3. Kết quả theo vai trò

| Nhiệm vụ | Artifact | Kết quả | Cách xác minh |
| --- | --- | --- | --- |
| Lưu dữ liệu xử lý | `data/clean/papers_clean*.json` | Baseline 24, corrupted 21, repaired 24 dòng | Đếm phần tử JSON. |
| Lưu benchmark | `data/eval/test_set.json` | 10 câu: summary 3, authors 3, date 2, categories 2 | Đếm `question_type`. |
| Lưu kết quả đánh giá | `data/results/*_metrics.json`, `data/results/*_answers.json` | Hit Rate 1.0 → 0.5 → 1.0; F1 1.0 → 0.5788 → 1.0 | Tính lại metrics từ answers. |
| Lưu tín hiệu quan sát | `data/quality/`, `data/results/corruption_log.json` | GX pass → fail → pass; freshness pass → fail → pass; sáu loại lỗi | Kiểm tra `success`, `is_fresh`, `events`. |

Output chính là bộ artifact trong commit `e608ed8`, nhất là `data/reports/corruption_report.md` và các JSON làm căn cứ cho bảng đối chiếu.

## 4. Giải thích phần kỹ thuật liên quan đến output bàn giao

### Vấn đề cần giải quyết

Người chấm cần đối chiếu kết quả pipeline mà không phụ thuộc vào một lần chạy trên máy cá nhân. Các artifact cho thấy dữ liệu, tín hiệu chất lượng và chỉ số đánh giá tại cùng một phiên bản Git.

### Cách các artifact nối với nhau

Crossref snapshot có 24 raw records. Cleaning khử trùng lặp, chuẩn hóa văn bản, tính `age_days` và ghép title, authors, published, categories, summary thành `text_for_embedding`. MiniLM và Chroma tạo index. Bộ 10 câu hỏi cố định đánh giá baseline, corrupted và repaired. Corruption áp dụng sáu loại lỗi, ghi log và đo lại. Repair dựng lại dữ liệu từ raw records rồi index và đánh giá trên cùng bộ test.

Phạm vi commit của tôi là **các output** của luồng này. Lịch sử commit không chứng minh tôi viết thuật toán hoặc trực tiếp chạy từng script tạo output.

### Input, output và contract

| Thành phần | Mô tả |
| --- | --- |
| Input gốc | `data/raw/crossref_records.json`, 24 bản ghi; đã có trong repo trước commit của tôi. |
| Input đánh giá | `data/eval/test_set.json`, gồm câu hỏi, đáp án và `ground_truth_doc_ids`. |
| Output | Clean/corrupted/repaired data, Chroma, embeddings manifest, quality/freshness reports, answers, metrics, Markdown reports. |
| Module tạo output | `src/pipelines/phase1.py`, `src/pipelines/corruption_flow.py` và các module được chúng gọi. |
| Điều kiện cần kiểm soát | Artifact không khớp nhau; đường dẫn tuyệt đối trong manifest; heuristic judge bị hiểu nhầm thành LLM judge. |

### Cách xác minh khi lập báo cáo

```bash
git show --format= --name-only e608ed8
git status --short --branch
python3 -m json.tool data/results/baseline_metrics.json
python3 -m json.tool data/results/corrupted_metrics.json
python3 -m json.tool data/results/repaired_metrics.json
```

Tôi đã đối chiếu các JSON answers, quality, freshness và corruption log. Cùng 10 ID xuất hiện ở ba trạng thái; Hit Rate và F1 tính lại từ answers khớp metrics; log ghi sáu loại lỗi. Tôi chưa xác nhận hai entrypoint chạy lại với exit code 0 trên bản nộp cuối.

## 5. Một quyết định quan trọng

- **Bối cảnh:** Báo cáo cá nhân phải khớp Git history.
- **Phương án cân nhắc:** Nhận ownership mã nguồn pipeline; hoặc nhận ownership bộ artifact và việc kiểm tra kết quả.
- **Lựa chọn:** Ghi nhận phần artifact và đối chiếu kết quả có bằng chứng.
- **Lý do:** Commit `e608ed8` chứa 47 file dưới `data/`, không chứa thay đổi mã nguồn. Cách ghi này tránh gán phần viết module của thành viên khác cho mình.
- **Bằng chứng:** `git show --format= --name-only e608ed8`.

## 6. Vấn đề còn mở khi kiểm tra artifact

- **Triệu chứng:** Ba `data/embeddings/papers_embeddings*.json` lưu `persist_path` tuyệt đối của máy tạo artifact. Cả 30 verdict trong answers ghi `Fallback heuristic judge used because the LLM evaluator was unavailable.`; Ragas có trạng thái `skipped`.
- **Nguyên nhân:** Manifest ghi đường dẫn tuyệt đối; evaluator dùng heuristic khi LLM không khả dụng; Ragas cần bật `RUN_RAGAS`.
- **Ảnh hưởng:** Nạp manifest ở máy khác có thể không tìm thấy Chroma DB. Các điểm judge hiện tại không phải điểm do LLM chấm trực tiếp.
- **Trạng thái:** Chưa có commit của tôi sửa các vấn đề này. Cần mô tả đúng giới hạn khi báo cáo hoặc demo.
- **Bước tiếp theo:** Kiểm tra tái tạo index trên checkout sạch; nếu dùng LLM judge thì chạy lại và lưu bằng chứng thực tế.

## 7. Hiểu biết về luồng end-to-end

1. **Crossref đến vector index:** Raw response được parse thành records; cleaning chuẩn hóa và sinh `text_for_embedding`; MiniLM tạo vector để Chroma lưu cùng document và metadata.
2. **Evaluation:** Mỗi câu có ground truth và document ID đúng. Retrieval hit xảy ra khi ít nhất một ID đúng được truy xuất; Token F1 so khớp token của câu trả lời với tham chiếu.
3. **Quality khác freshness:** GX kiểm tra số dòng, null, uniqueness và độ dài summary. Freshness tính tỷ lệ bài quá 180 ngày; trên 25% thì fail.
4. **Cùng test set:** Giữ cố định câu hỏi và đáp án để so sánh ba trạng thái dữ liệu công bằng.
5. **Repair thành công:** Dữ liệu trở về 24 dòng, GX và freshness pass, Hit Rate và F1 trở lại baseline trên cùng 10 câu.

## 8. Phân tích kết quả

| Metric/signal | Baseline | Corrupted | Repaired | Nhận xét |
| --- | ---: | ---: | ---: | --- |
| `retrieval_hit_rate` | 1.0000 | 0.5000 | 1.0000 | Mất 5/10 hit rồi phục hồi đủ 10/10. |
| `mean_token_f1` | 1.0000 | 0.5788 | 1.0000 | Chất lượng câu trả lời giảm rồi phục hồi. |
| `judge_accuracy` | 1.0000 | 0.6000 | 1.0000 | Đây là heuristic judge dự phòng theo answers JSON. |
| `mean_judge_score` | 5.0 | 3.2 | 5.0 | Cùng giới hạn heuristic như trên. |
| GX quality | Pass | Fail: 2 expectation | Pass | Trạng thái bẩn vi phạm uniqueness của `paper_id` và độ dài `summary`. |
| Freshness | Pass: 1/24 cũ | Fail: 6/21 cũ | Pass: 1/24 cũ | Tỷ lệ quá hạn 4.17% → 28.57% → 4.17%. |

**Hai chuỗi bằng chứng:**

1. Sáu thao tác corruption làm thay đổi số dòng, nội dung và ngày xuất bản → GX và freshness fail → Hit Rate giảm 1.0 xuống 0.5, F1 giảm 1.0 xuống 0.5788.
2. Repair dựng lại 24 dòng từ raw records → GX và freshness pass → Hit Rate và F1 trở về 1.0.

Không thể quy mức giảm metrics cho **một** dạng corruption riêng lẻ, vì suite áp dụng đủ sáu loại trước khi đánh giá. Mất năm bản ghi mới có thể góp phần vào retrieval miss, nhưng cần thí nghiệm riêng từng lỗi để xác định tác động. Một kết quả cần đọc kỹ: vài câu ở trạng thái corrupted có `retrieval_hit=False` nhưng `token_f1=1.0`; đáp án ngắn có thể trùng nhau dù lấy sai tài liệu. QA cũng ưu tiên khớp chính xác tiêu đề trong câu hỏi, nên baseline 1.0 chưa đo riêng chất lượng semantic retrieval.

## 9. Điều học được và hướng cải thiện

1. Raw snapshot, test set cố định và artifact theo từng trạng thái giúp truy vết thay đổi.
2. GX và freshness phát hiện các vấn đề khác nhau trong dữ liệu.
3. Metrics tổng hợp cần được đọc cùng answers và cơ chế evaluator để tránh kết luận quá mức.

Nếu có thêm thời gian, tôi sẽ đánh giá tách riêng sáu loại corruption và kiểm tra khả năng tái tạo artifact trên checkout sạch. Mỗi lần thử cần dùng cùng `test_set.json` và lưu metrics riêng để so sánh.

## 10. Tự kiểm tra trước khi nộp

- [x] Phạm vi đóng góp được giới hạn theo commit `e608ed8`.
- [x] Kết luận định lượng có JSON metrics, answers, quality và freshness để đối chiếu.
- [x] Không ghi hai script đã chạy lại thành công trên bản nộp cuối khi chưa xác minh.
- [x] Báo cáo không chứa API key hoặc token.
- [ ] Xác nhận tên nhóm chính thức và nội dung cá nhân với nhóm.
- [x] Xác nhận commit `e608ed8` đã được đưa vào `origin/main` qua merge commit `1d17cdc`.
- [ ] Kiểm tra tên tác giả xuất hiện trong GitHub Contributors của nhánh mặc định.
- [ ] Chạy lại hai entrypoint trên phiên bản cuối và ghi kết quả.

**Họ và tên:** Vũ Thường Tín
**Ngày:** 2026-09-25
