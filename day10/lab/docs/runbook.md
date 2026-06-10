# Runbook — Lab Day 10 (incident tối giản)

---

## Symptom

> User / agent thấy gì? (VD: trả lời “14 ngày” thay vì 7 ngày)
- Agent cung cấp thông tin sai lệch so với chính sách mới nhất (ví dụ: hoàn tiền 14 ngày, nghỉ phép 10 ngày).
- Agent trả lời các đoạn văn bản bị lỗi font hoặc chứa tiền tố lạ như "Nội dung không rõ ràng: ".

---

## Detection

> Metric nào báo? (freshness, expectation fail, eval `hits_forbidden`)
- Cảnh báo từ `freshness_check` (FAIL/WARN) báo hiệu dữ liệu vector đã quá hạn SLA 24h.
- Pipeline ETL bị HALT do Expectation Suite bắt được lỗi (ví dụ: chứa chuỗi 14 ngày).
- Hệ thống CI/CD chạy file `eval_retrieval.py` và báo `hits_forbidden` = yes, điểm correctness sụt giảm.

---

## Diagnosis

| Bước | Việc làm | Kết quả mong đợi |
|------|----------|------------------|
| 1 | Kiểm tra `artifacts/manifests/*.json` | Xác định lô dữ liệu ETL gần nhất, các chỉ số raw/cleaned/quarantine. |
| 2 | Mở `artifacts/quarantine/*.csv` | Kiểm tra xem các record đúng có bị loại nhầm không, hay nguồn xuất dữ liệu có vấn đề. |
| 3 | Chạy `python eval_retrieval.py` | Kiểm tra phạm vi ảnh hưởng của việc hallucinate (bao nhiêu câu test bị sai). |

---

## Mitigation

> Rerun pipeline, rollback embed, tạm banner “data stale”, …
- Rerun pipeline với cờ validate bật đầy đủ.
- Nếu nguồn gốc lỗi xuất phát từ database (CSV xuất bị sai), tạm thời revert Vector store (ChromaDB) về phiên bản snapshot của ngày hôm trước.
- Kích hoạt thông báo "Dữ liệu đang được đồng bộ, thông tin có thể cũ" trên giao diện Helpdesk.

---

## Prevention

> Thêm expectation, alert, owner — nối sang Day 11 nếu có guardrail.
- Phối hợp với Data Engineering team (owner) cấu hình chu kỳ xuất CSV tự động chính xác hơn.
- Cập nhật thêm cleaning rules vào file `transform/cleaning_rules.py` ngay khi phát hiện format lạ.
- Xây dựng hệ thống Alert bắn thẳng vào `#data-alerts` trên Slack khi pipeline HALT.
