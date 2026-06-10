# Data contract — Lab Day 10

> Bắt đầu từ `contracts/data_contract.yaml` — mở rộng và đồng bộ file này.

---

## 1. Nguồn dữ liệu (source map)

| Nguồn | Phương thức ingest | Failure mode chính | Metric / alert |
|-------|-------------------|-------------------|----------------|
| `policy_refund_v4` | Export CSV định kỳ | Có chứa dữ liệu cũ (14 ngày) | Số lượng bản ghi bị dính stale data |
| `access_control_sop` | Export CSV định kỳ | Bị thiếu doc_id trong quá trình ingest | Số lượng bản ghi bị loại (quarantine) do unknown doc_id |
| `hr_leave_policy` | Export CSV định kỳ | Có bản HR cũ năm 2025 (xung đột version) | Số bản ghi có hiệu lực trước năm 2026 |

---

## 2. Schema cleaned

| Cột | Kiểu | Bắt buộc | Ghi chú |
|-----|------|----------|---------|
| chunk_id | string | Có | … |
| doc_id | string | Có | … |
| chunk_text | string | Có | … |
| effective_date | date | Có | … |
| exported_at | datetime | Có | … |

---

## 3. Quy tắc quarantine vs drop

> Record bị flag đi đâu? Ai approve merge lại?

---

## 4. Phiên bản & canonical

> Source of truth cho policy refund: file nào / version nào?
