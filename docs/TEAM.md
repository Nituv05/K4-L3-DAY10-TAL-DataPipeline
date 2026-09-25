# Nhóm TAL — Day 10 Data Pipeline & Data Observability

- **Lớp:** K4-L3-DAY10
- **Repository:** https://github.com/Nituv05/K4-L3-DAY10-TAL-DataPipeline
- **Hình thức:** Ba thành viên dùng chung repository; phần việc dưới đây đối chiếu theo commit và artifact hiện có.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò và checkpoint | Deliverable chính | Báo cáo cá nhân |
| ---: | --- | --- | --- | --- | --- |
| 1 | Lê Tuấn Anh | 2A202602952 | Pipeline dữ liệu và tích hợp; CP0–CP5 | `src/ingestion/`, `src/observability/`, `src/evaluation/testset.py`, `src/pipelines/` | [`report/2A202602952_LeTuanAnh.md`](../report/2A202602952_LeTuanAnh.md) |
| 2 | Vũ Thường Tín | 2A202602955 | Artifact, đối chiếu kết quả và báo cáo; CP2–CP5 | `data/clean/`, `data/chroma/`, `data/eval/`, `data/quality/`, `data/results/`, `data/reports/`, `report/group_report.md` | [`report/2A202602955_VuThuongTin.md`](../report/2A202602955_VuThuongTin.md) |
| 3 | Trần Quốc Bảo Long | 2A202602696 | Giao diện demo và hỗ trợ trình bày; CP6 | `ui/`, `script/run_ui.py`, `script/test_ui.py`, hướng dẫn UI trong `README.md` | [`report/2A202602696_TranQuocBaoLong.md`](../report/2A202602696_TranQuocBaoLong.md) |

## Bằng chứng Git và ranh giới sở hữu

| Thành viên | Commit tiêu biểu | Nội dung có thể đối chiếu |
| --- | --- | --- |
| Lê Tuấn Anh | `1629c3e`, `ad4e3d7`, `ea8202e` | Hoàn thiện 8 file của tầng dữ liệu/pipeline; viết báo cáo cá nhân và đưa về tên file theo MSSV. |
| Vũ Thường Tín | `e608ed8`, `2aee2f9`, `ae93326`, `7951b12` | Đưa artifact vào Git, xác minh ba trạng thái, sửa đường dẫn manifest và ghi rõ nguồn Judge trong báo cáo. |
| Trần Quốc Bảo Long | `90d4c4d`, `ba94b77` | Thêm UI demo, server Python, bộ kiểm thử UI và báo cáo cá nhân. |

`src/core/`, `src/retrieval/` và `src/evaluation/metrics.py` đã có trong starter repo hoặc được sửa bởi commit khác; bảng trên không gán quyền tác giả các module đó cho ba thành viên nếu lịch sử commit không chứng minh. Báo cáo riêng mô tả phần việc mỗi người tự thực hiện; không dùng chung `report/individual_report.md` làm bài nộp cá nhân.

## Cá nhân

### Lê Tuấn Anh — 2A202602952

- **Đóng góp:** Parse/fallback Crossref, cleaning và cột `text_for_embedding`, GX 1.x/freshness, benchmark 10 câu, suite 6 lỗi, orchestration baseline/corruption/repair và hai báo cáo pipeline. Xem commit `1629c3e` cùng [`báo cáo riêng`](../report/2A202602952_LeTuanAnh.md).
- **Bàn giao:** Mã nguồn CP0–CP5 để các thành viên khác chạy và kiểm tra; artifact thực tế trong `data/` được commit bởi Tín.
- **Điều học được được ghi trong báo cáo riêng:** Giữ raw snapshot và test set cố định để kiểm tra tính tái lập, dùng GX 1.x để phát hiện dữ liệu bẩn.

### Vũ Thường Tín — 2A202602955

- **Đóng góp:** Đưa 47 artifact sạch/bẩn/phục hồi vào Git tại `e608ed8`; chạy lại hai entrypoint, kiểm tra Chroma 24/21/24 document, đối chiếu metrics với answers và lưu `data/reports/run_verification.md` tại `2aee2f9`; sửa đường dẫn manifest và ghi rõ nguồn Judge tại `7951b12`; cập nhật báo cáo nhóm và cá nhân.
- **Bàn giao:** Bộ artifact nhất quán, bảng metrics và [`báo cáo riêng`](../report/2A202602955_VuThuongTin.md). Không nhận quyền tác giả mã nguồn pipeline/retrieval.
- **Điều học được:** Exit code 0, quality pass và LLM judge là ba khẳng định khác nhau; cần đọc answers và log thực tế trước khi kết luận.

### Trần Quốc Bảo Long — 2A202602696

- **Đóng góp:** Thiết kế UI demo bằng `ui/index.html`, `ui/style.css`, `ui/app.js`; tạo server `script/run_ui.py`, kiểm thử `script/test_ui.py` và tài liệu `ui/README.md` trong commit `90d4c4d`.
- **Bàn giao:** Dashboard đọc artifact, hiển thị pipeline/quality/metrics và cho phép chạy hai pha từ giao diện. Đây là phần giao diện/demo, không phải quyền tác giả của `src/retrieval/`.
- **Báo cáo riêng:** [`report/2A202602696_TranQuocBaoLong.md`](../report/2A202602696_TranQuocBaoLong.md), do Long commit tại `ba94b77`.

## Việc nhóm cần xác nhận trước khi nộp

- [x] Ba thành viên có báo cáo riêng theo MSSV trên nhánh `main`.
- [x] Nhóm xác nhận cả ba thành viên đã hoàn tất phần Git/Contributors trên nhánh `main`.
- [x] Nhóm xác nhận mỗi thành viên đã tự nộp link repository trên VLearn LMS.
