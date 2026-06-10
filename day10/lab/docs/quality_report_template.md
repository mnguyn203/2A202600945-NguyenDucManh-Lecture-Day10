# Quality report — Lab Day 10 (nhóm)

**run_id:** _______________  
**Ngày:** _______________

---

## 1. Tóm tắt số liệu

| Chỉ số | Trước | Sau | Ghi chú |
|--------|-------|-----|---------|
| raw_records | 247 | 247 | Tổng số dòng trong file CSV gốc |
| cleaned_records | 44 | 44 | Số dòng được đưa vào Vector Database |
| quarantine_records | 203 | 203 | Số dòng bị chặn lại (không hợp lệ) |
| Expectation halt? | Có (Fail ở rules refund_no_stale_14d_window) | Không (Pass tất cả) | Khi inject bad data, hệ thống báo lỗi nhưng cho pass qua do cờ skip. |

---

## 2. Before / after retrieval (bắt buộc)

> Đính kèm hoặc dẫn link tới `artifacts/eval/before_after_eval.csv` (hoặc 2 file before/after).

**Câu hỏi then chốt:** refund window (`q_refund_window`)  
**Trước (Dữ liệu bẩn):** `Yêu cầu hoàn tiền được chấp nhận trong vòng 14 ngày làm việc kể từ xác nhận đơn.` (Retrieval sai: hits_forbidden = yes)  
**Sau (Dữ liệu sạch):** `Yêu cầu được gửi trong vòng 7 ngày làm việc làm việc kể từ thời điểm xác nhận đơn hàng.` (Retrieval đúng: contains_expected = yes, hits_forbidden = no)

**Merit (khuyến nghị):** versioning HR — `q_leave_version` (`contains_expected`, `hits_forbidden`, cột `top1_doc_expected`)

**Trước:** Dữ liệu bẩn có chứa bản 2025 (`10 ngày phép năm`)
**Sau:** Bản 2025 đã bị loại bỏ hoàn toàn khỏi Vector Store, chỉ còn bản 2026 (`12 ngày phép năm`).

---

## 3. Freshness & monitor

> Kết quả `freshness_check` trả về **WARN**. Lý do: `no_timestamp_in_manifest` hoặc latest_exported_at là `2026/04/07` trong khi chạy hệ thống ngày hiện tại `2026-06-10`, vi phạm SLA 24 giờ.
> Pipeline Ingest đã bỏ lỡ dữ liệu cập nhật hoặc hệ thống xuất báo cáo bị lỗi. Cần trigger lại bản cập nhật hoặc gửi alert cho team Data Engineering qua kênh #data-alerts.

---

## 4. Corruption inject (Sprint 3)

> Chúng tôi đã chạy lệnh với cờ `--no-refund-fix --skip-validate` để bỏ qua quá trình tự động làm sạch (không replace 14 -> 7) và bắt ép hệ thống nuốt lỗi expectation.
> Phát hiện: Expectation `refund_no_stale_14d_window` đã bắt được lỗi (3 violations) nhưng chúng ta bypass nó. Kết quả là Retrieval đã bốc nhầm chunk 14 ngày, khiến AI trả lời sai quy định công ty. Điều này chứng minh "Garbage in, Garbage out".

---

## 5. Hạn chế & việc chưa làm

- …
