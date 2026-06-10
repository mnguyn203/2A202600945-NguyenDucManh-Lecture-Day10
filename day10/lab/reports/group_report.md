# Báo Cáo Nhóm — Lab Day 10: Data Pipeline & Data Observability

**Tên nhóm:** ___________  
**Thành viên:**
| Tên | Vai trò (Day 10) | Email |
|-----|------------------|-------|
| AI | Ingestion / Raw Owner | ai@example.com |
| AI | Cleaning & Quality Owner | ai@example.com |
| AI | Embed & Idempotency Owner | ai@example.com |
| AI | Monitoring / Docs Owner | ai@example.com |

**Ngày nộp:** ___________  
**Repo:** ___________  
**Độ dài khuyến nghị:** 600–1000 từ

---

> **Nộp tại:** `reports/group_report.md`  
> **Deadline commit:** xem `SCORING.md` (code/trace sớm; report có thể muộn hơn nếu được phép).  
> Phải có **run_id**, **đường dẫn artifact**, và **bằng chứng before/after** (CSV eval hoặc screenshot).

---

## 1. Pipeline tổng quan (150–200 từ)

> Nguồn raw là gì (CSV mẫu / export thật)? Chuỗi lệnh chạy end-to-end? `run_id` lấy ở đâu trong log?

**Tóm tắt luồng:**
Dữ liệu raw (Export CSV chứa 247 dòng từ 5 nguồn) được đọc. Sau đó chạy qua các luật Clean, lọc bỏ các dòng quarantine (203 dòng). Dữ liệu sau Clean (44 dòng) tiếp tục qua bộ Expectation kiểm tra chất lượng. Nếu Pass hết sẽ được nhúng qua model BGE/MiniLM và đẩy lên ChromaDB, đồng thời ghi lại Manifest và check Freshness.

**Lệnh chạy một dòng (copy từ README thực tế của nhóm):**
`python etl_pipeline.py run`

---

## 2. Cleaning & expectation (150–200 từ)

> Baseline đã có nhiều rule (allowlist, ngày ISO, HR stale, refund, dedupe…). Nhóm thêm **≥3 rule mới** + **≥2 expectation mới**. Khai báo expectation nào **halt**.

### 2a. Bảng metric_impact (bắt buộc — chống trivial)

| Rule / Expectation mới (tên ngắn) | Trước (số liệu) | Sau / khi inject (số liệu) | Chứng cứ (log / CSV / commit) |
|-----------------------------------|------------------|-----------------------------|-------------------------------|
| Xóa chuỗi "Nội dung không rõ ràng: " | Lọt rác vào Embed | Chuỗi được xóa hoàn toàn | Trong `after_fix_eval.csv` không còn tiền tố này |
| Xóa ký tự rác "!!!" | Lọt rác vào Embed | Ký tự bị dọn sạch | Code trong hàm clean_rows |
| Expectation `no_unclear_prefix` | Khi chưa có rule clean, expectation báo lỗi | Khi chạy rule clean, expectation pass | Log pipeline hiển thị OK |
| Expectation `no_exclamation_spam` | Tương tự | Pass thành công | Log pipeline hiển thị OK |

**Rule chính (baseline + mở rộng):**
- Bổ sung `access_control_sop` vào `ALLOWED_DOC_IDS`.
- Xóa tiền tố "Nội dung không rõ ràng: " và "!!!".
- Cập nhật rule cấm "10 ngày phép năm" bên cạnh cờ check date.

**Ví dụ 1 lần expectation fail (nếu có) và cách xử lý:**
Khi mới thêm data, system báo `expectation[hr_leave_no_stale_10d_annual] FAIL (halt) :: violations=2`. Lỗi do các văn bản không bị lỗi ngày ISO nhưng vẫn chứa chuỗi "10 ngày phép năm". Chúng tôi xử lý bằng cách thêm rule vào cleaning: quarantine toàn bộ nếu văn bản chứa "10 ngày phép năm". Lần chạy sau đó exit 0.

---

## 3. Before / after ảnh hưởng retrieval hoặc agent (200–250 từ)

> Bắt buộc: inject corruption (Sprint 3) — mô tả + dẫn `artifacts/eval/…` hoặc log.

**Kịch bản inject:**
Chạy pipeline với option `--no-refund-fix --skip-validate`. Điều này bỏ qua bước sửa lỗi refund window (14 thành 7) và lờ đi kết quả của expectation báo FAIL, buộc insert dữ liệu rác (14 ngày) vào vector store.

**Kết quả định lượng (từ CSV / bảng):**
- Trạng thái bẩn: Câu hỏi `q_refund_window` trả về kết quả 14 ngày (sai). `hits_forbidden = yes`.
- Trạng thái sạch: Câu hỏi trả về 7 ngày (đúng). `contains_expected = yes`, `hits_forbidden = no`. Tương tự với xung đột version HR (10 ngày vs 12 ngày). Dữ liệu sạch cho kết quả 12 ngày chuẩn xác.

---

## 4. Freshness & monitoring (100–150 từ)

> SLA bạn chọn, ý nghĩa PASS/WARN/FAIL trên manifest mẫu.
Chúng tôi cấu hình SLA 24 giờ. `freshness_check` báo `WARN` do `latest_exported_at` của dữ liệu là 2026/04/07 (thời điểm quá xa so với hiện tại). Nếu dữ liệu vượt SLA một mốc ngắn nó có thể WARN, nhưng nếu quá cũ nó có thể FAIL và trigger alert tới slack `#data-alerts`.

---

## 5. Liên hệ Day 09 (50–100 từ)

> Dữ liệu sau embed có phục vụ lại multi-agent Day 09 không? Nếu có, mô tả tích hợp; nếu không, giải thích vì sao tách collection.
Dữ liệu nhúng của Day 10 hoàn toàn có thể được dùng lại như một base knowledge mới cho Day 09. Agent ở Day 09 khi tìm kiếm qua Vector Database sẽ chỉ truy cập những nội dung đã trải qua đường ống ETL khắt khe này. Việc này tăng độ chính xác của agent, tránh agent tìm kiếm thấy tài liệu sai lệch.

---

## 6. Rủi ro còn lại & việc chưa làm

- …
