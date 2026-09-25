# Báo cáo cá nhân — Trần Quốc Bảo Long

## 1. Thông tin cá nhân

| Thông tin | Nội dung |
| --- | --- |
| Họ và tên | Trần Quốc Bảo Long |
| MSSV | 2A202602696 |
| Khóa/Lớp | K4-L3-DAY10 |
| Tên nhóm | TAL |
| Vai trò chính | Frontend/UI — xây dựng giao diện minh họa pipeline |
| Repository | https://github.com/Nituv05/K4-L3-DAY10-TAL-DataPipeline |
| Ngày hoàn thành | 2026-09-25 |

## 2. Vai trò và phạm vi công việc

Tôi phụ trách xây dựng giao diện **Pipeline Observatory** để minh họa luồng xử lý và trình bày kết quả data pipeline do hai thành viên còn lại thực hiện. Phần việc của tôi giúp người xem theo dõi dữ liệu từ raw snapshot đến baseline, corrupted và repaired, đồng thời đối chiếu chất lượng dữ liệu với hiệu quả RAG.

Tôi sử dụng các dataset, metrics và báo cáo chất lượng đã được nhóm tạo ra làm đầu vào cho UI. Tôi không nhận phần xây dựng ingestion, cleaning, embedding, ChromaDB, thuật toán đánh giá hoặc cơ chế corruption/repair là đóng góp cá nhân. Server Python trong phần bàn giao của tôi là lớp phục vụ UI, đọc artifacts và gọi lại các entrypoint có sẵn.

| Hạng mục phụ trách | File bàn giao | Kết quả |
| --- | --- | --- |
| Bố cục dashboard tiếng Việt | `ui/index.html`, `ui/style.css` | Tổng quan, sơ đồ pipeline, benchmark, quality, corruption và bảng dữ liệu; có bố cục thích ứng màn hình hẹp |
| Tương tác và trình bày dữ liệu | `ui/app.js` | Chuyển ba trạng thái, biểu đồ so sánh, tìm kiếm, phân trang và xem chi tiết bài báo |
| Kết nối UI với artifacts | `script/run_ui.py` | Đọc JSON trong `data/`, cung cấp API snapshot và phục vụ giao diện trên localhost |
| Điều khiển chạy từ UI | `script/run_ui.py`, `ui/app.js` | Gọi hai script có sẵn, hiển thị log và khóa yêu cầu chạy trùng trong phiên server |
| Kiểm tra lớp phục vụ UI | `script/test_ui.py` | Kiểm tra đọc artifacts, API, quyền truy cập và khóa job bằng mock |
| Hướng dẫn demo | `ui/README.md`, phần UI trong `README.md` | Cách khởi động, điều kiện môi trường và kịch bản trình bày |

Bằng chứng mã nguồn: commit **`90d4c4d` — `feat: Designed user interface for demonstration`** có các file UI, server và kiểm tra nêu trên. Các commit đưa artifacts data vào Git của thành viên khác không được dùng làm bằng chứng đóng góp cá nhân của tôi.

## 3. Kết quả data của nhóm được sử dụng làm đầu vào

Các số liệu dưới đây được đối chiếu từ artifacts hiện có trong repository. Đây là **kết quả chung của phần data pipeline**, được tôi sử dụng để trình bày và phân tích trên giao diện; không phải số liệu do UI tự tính hoặc kết quả một lần chạy pipeline do tôi xác nhận độc lập.

| Chỉ số/tín hiệu | Baseline | Corrupted | Repaired |
| --- | ---: | ---: | ---: |
| Số dòng dataset | 24 | 21 | 24 |
| Số câu hỏi benchmark | 10 | 10 | 10 |
| Retrieval Hit Rate | 1.0000 | 0.5000 | 1.0000 |
| Mean Token F1 | 1.0000 | 0.5788 | 1.0000 |
| Judge Accuracy — LLM | 1.0000 | 0.6000 | 1.0000 |
| Mean Judge Score — LLM | 5.0 | 3.4 | 5.0 |
| Great Expectations | PASS | FAIL | PASS |
| Số expectation thất bại / tổng số | 0/6 | 2/6 | 0/6 |
| Số bản ghi quá 180 ngày | 1/24 | 6/21 | 1/24 |
| Tỷ lệ quá hạn | 4.17% | 28.57% | 4.17% |
| Freshness SLA | PASS | FAIL | PASS |

Nguồn đối chiếu:

- Dataset: `data/clean/papers_clean.json`, `papers_clean_corrupted.json`, `papers_clean_repaired.json`.
- Metrics: [baseline_metrics.json](../data/results/baseline_metrics.json), [corrupted_metrics.json](../data/results/corrupted_metrics.json), [repaired_metrics.json](../data/results/repaired_metrics.json).
- Kiểm định: `data/quality/{baseline,corrupted,repaired}_quality_report.json` và ba file `freshness_report*.json`.
- Diễn biến tiêm lỗi: [corruption_log.json](../data/results/corruption_log.json).
- Bằng chứng chạy của nhóm: [run_verification.md](../data/reports/run_verification.md), [corruption_report.md](../data/reports/corruption_report.md).

Log ghi nhận sáu thao tác: bỏ 5 bản ghi mới nhất, xóa summary của 2 dòng, chèn nhiễu vào 2 dòng, cắt tiêu đề của 2 dòng, lùi ngày của 3 dòng và nhân bản 2 dòng. Vì một bản ghi có thể chịu nhiều thao tác, không cộng các số này để suy ra số bản ghi lỗi duy nhất. Dataset sau corruption có `24 - 5 + 2 = 21` dòng; repair tái tạo về 24 dòng từ raw snapshot.

## 4. Thiết kế và cách kết nối giao diện

### Mục tiêu trình bày

Từ các kết quả đã có, tôi tổ chức UI để người xem trả lời được ba câu hỏi: dữ liệu đang ở bước nào, lỗi ảnh hưởng đến chất lượng ra sao, và sau phục hồi các chỉ số có trở lại baseline hay không.

Giao diện gồm sơ đồ tương tác bảy bước, bốn thẻ tổng quan, biểu đồ benchmark, khu vực Quality Gate/Freshness, sáu kịch bản corruption và bảng khám phá corpus. Khi chọn Baseline, Corrupted hoặc Repaired, các thẻ chất lượng và dữ liệu tương ứng được cập nhật. Biểu đồ benchmark giữ cả ba trạng thái để tiện so sánh. Bảng dữ liệu cho phép chọn raw snapshot hoặc trạng thái đang xem, tìm theo tiêu đề/DOI/nội dung, phân trang và mở chi tiết JSON.

### Luồng kết nối

```text
Artifacts do pipeline của nhóm sinh ra trong data/
    → script/run_ui.py đọc JSON
    → GET /api/snapshot
    → ui/app.js ánh xạ dữ liệu
    → dashboard, biểu đồ, quality và bảng bài báo
```

| Dữ liệu đầu vào | Trường sử dụng | Vị trí trình bày |
| --- | --- | --- |
| Raw/clean JSON | `paper_id`, `title`, `summary`, `primary_category`, `published` | Bảng bài báo và hộp chi tiết |
| Metrics JSON | `samples`, `retrieval_hit_rate`, `mean_token_f1`, `mean_judge_score` | Thẻ Hit Rate và biểu đồ benchmark |
| Quality JSON | `success`, `evaluated_expectations`, `unsuccessful_expectations`, `expectations` | Thẻ Data Quality và danh sách kiểm định |
| Freshness JSON | `is_fresh`, `stale_rows`, `stale_ratio`, `threshold_days`, `latest_published` | Thẻ Freshness và thông tin độ tươi |
| Corruption log | `events`, `step`, `affected_rows` | Sáu kịch bản và số dòng bị tác động |

UI kiểm tra snapshot mỗi 3 giây và chỉ dựng lại phần dữ liệu khi nội dung thay đổi, giúp hạn chế mất trạng thái tương tác do cập nhật không cần thiết. Văn bản từ artifacts được escape hoặc đưa qua `textContent` trước khi hiển thị. File chưa tồn tại được biểu diễn bằng “—”/“Chưa có dữ liệu”; JSON hỏng có thông báo lỗi đọc file.

Hai nút chạy gọi lại `script/run_phase1.py` và `script/run_corruption_flow.py` bằng cùng Python chạy server. UI không triển khai lại logic pipeline. Server chỉ lắng nghe localhost, giới hạn các file được phục vụ, kiểm tra token phiên cho yêu cầu chạy và giữ tối đa 600 dòng log.

## 5. Quyết định thiết kế quan trọng

Tôi chọn đọc trực tiếp artifacts có sẵn thay vì nhập thủ công số liệu vào giao diện. Cách này giữ UI thống nhất với kết quả của hai thành viên phụ trách data, giúp thay đổi metrics sau mỗi lần chạy được phản ánh mà không phải sửa mã frontend.

Tôi sử dụng HTML/CSS/JavaScript thuần và server Python standard library để giảm bước cài đặt cho phần minh họa. Chỉ xem artifacts không cần API key hay tải mô hình. Khi người dùng chủ động chạy pipeline từ UI, môi trường vẫn phải có đầy đủ dependencies và cấu hình của pipeline.

Một nguyên tắc khác là tách **trạng thái thực thi** khỏi **trạng thái chất lượng**. Script kết thúc với exit code 0 không đồng nghĩa dữ liệu đạt Quality Gate. Giao diện có ghi chú rằng luồng lab tiếp tục index dữ liệu lỗi để đo suy giảm.

## 6. Kiểm tra và bằng chứng bàn giao

Trong quá trình xây dựng UI, đã kiểm tra mở dashboard trên trình duyệt, hiển thị 24 raw records, chọn bước Quality Gate, tìm kiếm bài báo và mở hộp chi tiết nội dung/JSON. Bố cục màn hình hẹp cũng đã được quan sát trực tiếp.

| Kiểm tra đã thực hiện | Kết quả và phạm vi |
| --- | --- |
| `node --check ui/app.js` | Qua kiểm tra cú pháp JavaScript |
| `python -m unittest discover -s script -p test_ui.py` | Hai bài kiểm tra qua; bao gồm file thiếu/hỏng, đọc metrics, phục vụ assets/API, từ chối truy cập file ngoài danh sách, token và khóa chạy trùng |
| Tìm kiếm `Ghost` trên raw snapshot | Trả về 2/24 bản ghi phù hợp |
| Mở chi tiết bài báo | Hiển thị tiêu đề, DOI, summary và JSON của bản ghi |
| Chọn bước Quality Gate | Hiển thị mô tả bước và đường dẫn artifacts tương ứng |

Kiểm tra lệnh chạy trong `test_ui.py` dùng mock, không gọi LLM và không chứng minh pipeline đã thực thi end-to-end qua UI. Ở lần kiểm tra UI ban đầu, `.venv` trên máy không khởi động được; dashboard được kiểm tra bằng Python có sẵn trong môi trường Codex. Bằng chứng hai entrypoint chạy thành công là tài liệu `data/reports/run_verification.md` do nhóm cung cấp, không phải kết quả chạy lại do tôi thực hiện trong phần UI.

Cách mở sản phẩm bàn giao từ thư mục gốc bằng Python đang hoạt động:

```bash
python script/run_ui.py
```

Truy cập `http://127.0.0.1:8765`; hướng dẫn chi tiết và kịch bản demo nằm trong [ui/README.md](../ui/README.md).

## 7. Hiểu biết về pipeline end-to-end

1. **Ingestion:** Lưu raw snapshot/records từ Crossref để có dữ liệu gốc phục vụ truy vết và tái tạo.
2. **Cleaning:** Chuẩn hóa nội dung, loại trùng và tạo các trường dẫn xuất như `age_days`, `summary_chars`, `text_for_embedding`.
3. **Observability:** Great Expectations kiểm tra quy tắc dữ liệu; freshness đánh giá tỷ lệ bản ghi quá 180 ngày, fail khi tỷ lệ vượt 25%.
4. **Index và evaluation:** MiniLM tạo embeddings, Chroma lưu các collection riêng; dùng cùng bộ 10 câu hỏi để đối chiếu ba trạng thái.
5. **Corruption:** Áp dụng sáu dạng lỗi, ghi log, kiểm định và đánh giá lại để quan sát suy giảm.
6. **Repair:** Đọc lại raw records, tái tạo dataset/index rồi đánh giá trên cùng bộ câu hỏi để kiểm chứng phục hồi.

UI nằm ở lớp trình bày và điều khiển demo của luồng này. Giá trị đóng góp của tôi là làm cho các bằng chứng dữ liệu dễ theo dõi và giải thích hơn.

## 8. Phân tích kết quả từ góc nhìn UI

Trên bộ kết quả của nhóm, corruption làm Hit Rate giảm từ 1.0 xuống 0.5 và Token F1 từ 1.0 xuống khoảng 0.5788. Đồng thời, GX phát hiện hai expectation thất bại và freshness vượt ngưỡng 25%. Trình bày các tín hiệu cạnh nhau giúp giải thích hiện tượng silent failure: hệ thống vẫn có thể hoàn tất xử lý nhưng chất lượng dữ liệu và truy xuất đã suy giảm.

Sau repair, dataset trở về 24 dòng, GX/Freshness về PASS và Hit Rate/Token F1 trở lại 1.0. Đây là bằng chứng phục hồi trên bộ thử nghiệm hiện có. Vì sáu loại lỗi được áp dụng cùng lúc, bảng so sánh không đủ để kết luận riêng loại lỗi nào gây ra bao nhiêu phần suy giảm.

## 9. Điều học được và hướng cải thiện

Tôi đã học được cách tích hợp qua contract artifacts để phần frontend có thể phát triển độc lập với pipeline, đồng thời phải xử lý rõ trường hợp chưa có dữ liệu, file đọc lỗi và tác vụ đang chạy.

Nếu tiếp tục phát triển, tôi sẽ ưu tiên bổ sung nhãn phân biệt LLM/heuristic judge, hiển thị run ID và thời điểm sinh metrics, cảnh báo artifacts khác lần chạy, và thêm chế độ trình chiếu từng bước. Phần kiểm thử tiếp theo là xác minh đầy đủ thao tác chạy baseline rồi corruption/repair từ UI trong môi trường pipeline đã cấu hình hoàn chỉnh.

## 10. Tự kiểm tra trước khi nộp

- [x] Thông tin cá nhân thống nhất với tên Trần Quốc Bảo Long và MSSV 2A202602696.
- [x] Phạm vi đóng góp được giới hạn ở UI, lớp kết nối phục vụ UI, kiểm tra và hướng dẫn demo.
- [x] Có commit `90d4c4d` làm bằng chứng mã nguồn UI.
- [x] Số liệu data được dẫn từ artifacts của nhóm, không nhận là kết quả cá nhân tự tạo.
- [x] Phân biệt kiểm tra UI với bằng chứng chạy pipeline end-to-end của nhóm.
- [x] Đối chiếu kết quả Judge dùng LLM trong artifact mới; Ragas vẫn được bỏ qua.
- [x] Báo cáo không chứa API key hoặc secret.
- [x] Đồng bộ phân công trong `docs/TEAM.md` với vai trò UI thực tế; bảng hiện còn ghi vai trò RAG & Vector Index cho tôi.
- [x] Kiểm tra hiển thị đóng góp trên GitHub Contributors và tự nộp link repository lên VLearn LMS.

**Họ và tên:** Trần Quốc Bảo Long
**Ngày:** 2026-09-25
