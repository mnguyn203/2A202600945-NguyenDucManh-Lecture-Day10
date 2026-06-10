# AI in Action — Lecture Slides: Day 08 · 09 · 10

> **Course:** AI in Action – Phase 1 (Nền tảng)
> **Topic block:** Từ RAG đến Multi-Agent đến Data Pipeline — xây dựng hệ AI vận hành thực tế

---

## Overview

Ba ngày học này tạo thành một mạch kiến thức liên tục: cùng một artifact (trợ lý nội bộ cho khối CS + IT Helpdesk) được nâng cấp qua từng ngày — từ RAG có kiểm soát, sang điều phối đa agent, đến tầng data pipeline và observability đảm bảo hệ chạy bền.

```
Day 08 ─ RAG grounded          →   Day 09 ─ Supervisor-Workers      →   Day 10 ─ Data + Observe
   Retrieve đúng đoạn               Route + trace + MCP                  Freshness · quality · alert
```

---

## Day 08 — RAG Pipeline

**File:** [`day08/lecture-08.html`](day08/lecture-08.html)

### Vấn đề mở bài
Vector store đã có, nhưng agent vẫn trả lời sai — tại sao?

### Nội dung chính

| Chủ đề | Chi tiết |
|--------|----------|
| **Indexing pipeline** | Chunking đúng, metadata rõ, freshness có kiểm soát |
| **Retrieval strategy** | Dense vs Sparse vs Hybrid; query transformation; top-k & rerank funnel |
| **Grounded prompting** | Inject context đúng cách để model bớt hallucinate |
| **RAG evaluation** | Đo bằng scorecard (faithfulness, relevance, correctness), không bằng cảm giác |

### Hoạt động trong lớp
1. **Error Tree** — phân tích nguyên nhân gốc rễ của RAG failure
2. **Chunking Clinic** — so sánh chiến lược chunk
3. **Retrieval decision map** — chọn chiến lược phù hợp với use case
4. **Prompt Surgery** — sửa grounded prompt thực tế

### Deliverables
- `rag_pipeline.py` — indexing + retrieval end-to-end
- `eval_scorecard.csv` — kết quả đo retrieval và answer quality
- `rag_architecture.md` — tài liệu thiết kế

---

## Day 09 — Multi-Agent & Kết Nối Hệ Thống

**File:** [`day09/lecture-09.html`](day09/lecture-09.html)

### Vấn đề mở bài
Một agent giỏi vẫn quá tải khi bài toán phức tạp — khi nào nên tách hệ?

### Nội dung chính

| Chủ đề | Chi tiết |
|--------|----------|
| **Multi-agent patterns** | Supervisor-Workers, Pipeline, Peer-to-peer, Hierarchical |
| **Supervisor-Worker** | Route theo tín hiệu quan sát được (task type, confidence, risk) |
| **Worker contract** | Rõ input · rõ output · rõ lỗi — chuẩn để test và thay thế |
| **MCP architecture** | Agent cắm vào năng lực bên ngoài (tool, API) theo chuẩn chung |
| **A2A vs MCP** | Phân biệt giao việc cho agent khác vs lấy capability từ bên ngoài |
| **LangGraph** | Node, edge, state, route function, HITL checkpoint |
| **Trace & Observability** | Ghi task · route reason · worker IO · answer cuối để debug và học |

### Hoạt động trong lớp
1. **Agent overload map** — xác định điểm single-agent quá tải
2. **Chọn pattern** — so sánh 4 pattern cho 3 use case thực tế
3. **Tách artifact Day 08** — chia RAG agent thành supervisor + workers
4. **Phân biệt MCP với A2A** — hands-on classification

### Deliverables
- `supervisor_agent.py` — router với route logic rõ ràng
- `worker_*.py` — retrieval, policy, synthesis workers
- `trace_log.jsonl` — trace ghi đủ bước cho mỗi run
- `mcp_config.json` — khai báo tool/MCP connection

---

## Day 10 — Data Pipeline & Data Observability

**File:** [`day10/lecture-10.html`](day10/lecture-10.html)

### Vấn đề mở bài
Data từ database công ty đột nhiên sai — agent hallucinate. Hệ của bạn có biết không?

> *Garbage in → garbage out. Đừng debug model trước khi debug pipeline.*

### Nội dung chính

| Chủ đề | Chi tiết |
|--------|----------|
| **ETL vs ELT** | Transform trước hay sau load — quyết định dựa trên governance, latency, cost |
| **Batch vs Streaming** | Trade-off latency vs complexity; khi nào cần streaming thực sự |
| **Ingestion layer** | CDC, rate limit, backpressure, retry + backoff, DLQ |
| **Data transformation** | Làm sạch PII, chuẩn hoá schema, dedupe, encoding |
| **Data quality as code** | Expectation suite chạy như unit test trong CI/CD |
| **5 pillars of observability** | Freshness · Volume · Distribution · Schema · Lineage |
| **Orchestration** | DAG, dependency, retry, idempotency, SLA alert |
| **Incident triage** | Runbook — detect → isolate → fix → verify → post-mortem |

### Hoạt động trong lớp
1. **ETL hay ELT? Batch hay Streaming?** — phân loại 4 tình huống thực tế
2. **Source map & failure points** — ingestion plan cho DB + API + PDF
3. **Dirty data repair** — sửa dataset có missing, duplicate, wrong format
4. **Incident triage** — xử lý freshness breach theo runbook

### Deliverables
- `etl_pipeline.py` — ingest → clean → validate → embed end-to-end
- `quality/expectations.py` — expectation suite kiểm tra data quality
- `monitoring/freshness_check.py` — freshness + volume monitor
- `before_after_eval.csv` — bằng chứng data quality ảnh hưởng answer quality
- `pipeline_architecture.md` + `data_contract.md` + `runbook.md`

---

## Mạch xuyên suốt 3 ngày

```
                  ┌──────────────────────────────────────────────────────┐
                  │              Trợ lý nội bộ CS + IT Helpdesk          │
                  └──────────────────────────────────────────────────────┘
                            ↑               ↑               ↑
                         Day 08          Day 09          Day 10
                      RAG grounded   Supervisor +    Data pipeline
                      retrieve đúng  workers route   + observability
                      đoạn, đo được  trace rõ,MCP    detect issue
                                     A2A                 sớm
```

Mỗi ngày xây trên artifact của ngày trước:
- **Day 08** cung cấp `rag_pipeline` làm nền retrieval
- **Day 09** bọc nó vào `retrieval_worker`, thêm `policy_worker`, `synthesis_worker` và supervisor route
- **Day 10** đảm bảo dữ liệu feeding vào toàn bộ hệ không bị stale, dirty hay missing

---

## Tech Stack

| Layer | Công cụ |
|-------|---------|
| LLM | Claude / GPT-4o |
| Embedding | text-embedding-3-small / BGE |
| Vector store | ChromaDB / Qdrant / pgvector |
| Orchestration | LangGraph / CrewAI |
| MCP | MCP SDK |
| Pipeline | Python ETL / Prefect / Airflow |
| Quality | Great Expectations |
| Monitoring | Custom + Grafana |

---

## How to Use

Mỗi slide deck là một file HTML độc lập, chạy thẳng trên browser — không cần cài thêm server:

```bash
# Mở trực tiếp trong browser
open day08/lecture-08.html
open day09/lecture-09.html
open day10/lecture-10.html
```

**Điều hướng:**
- `←` / `→` hoặc `Space` — chuyển slide
- `N` — toggle speaker notes
- `Home` / `End` — về đầu / cuối

---

## Tổng Kết Quá Trình Thực Hiện Lab Day 10 (Data Pipeline & Observability)

Phần này tóm tắt những công việc đã được giải quyết xuyên suốt bài thực hành Lab Day 10 nhằm xây dựng một Data Pipeline chuẩn chỉnh, sẵn sàng cấp dữ liệu sạch cho Agent.

### 1. Vấn đề cần giải quyết
- Dữ liệu thô (`policy_export_dirty.csv`) chứa export từ 5 hệ thống nhưng Pipeline ban đầu chỉ xử lý một phần, khiến Vector Database thiếu hụt kiến thức.
- Dữ liệu bị bẩn, chứa các tiền tố lạ (`Nội dung không rõ ràng: `), ký tự rác (`!!!`).
- Tồn tại dữ liệu lỗi thời:
  - Bản HR ghi 10 ngày phép năm (năm 2025) xung đột với bản 12 ngày phép năm (2026).
  - Quy định hoàn tiền cũ (14 ngày) xung đột với quy định mới (7 ngày).
- Nếu đưa thẳng vào Agent, Agent sẽ bị ảo giác (hallucinate) và đưa ra thông tin sai lệch cho nhân viên.

### 2. Các công việc đã thực hiện
**Sprint 1: Phân tích & Bổ sung Scope**
- Phát hiện tài liệu `access_control_sop` bị bỏ sót khỏi Pipeline.
- Cập nhật biến `ALLOWED_DOC_IDS` trong file `transform/cleaning_rules.py` để bổ sung nguồn này.
- Cập nhật `contracts/data_contract.yaml` để khai báo quyền sở hữu (owner) và cấu hình alert SLA.

**Sprint 2: Làm sạch dữ liệu (Data Cleaning) & Đo lường chất lượng (Expectations)**
- Code thêm 3 rule làm sạch (Cleaning Rules) vào `cleaning_rules.py`:
  1. Loại bỏ các chunk chứa nội dung *"bản sync cũ"*.
  2. Xóa sạch các tiền tố nhiễu *"Nội dung không rõ ràng: "* và rác *"!!!"*.
  3. Loại bỏ triệt để các văn bản HR cũ chứa *"10 ngày phép năm"*.
- Code thêm 2 expectation mới vào `quality/expectations.py` để làm chốt chặn:
  1. `no_unclear_prefix`: Báo lỗi nếu vẫn còn tiền tố lạ lọt qua luồng clean.
  2. `no_exclamation_spam`: Báo lỗi nếu vẫn còn ký tự rác lọt qua luồng clean.
- Chạy toàn bộ tiến trình `python etl_pipeline.py run`, kết quả exit 0, Pipeline OK. 44 chunk sạch sẽ được tạo embedding.

**Sprint 3: Inject Corruption (Bơm lỗi để thử nghiệm)**
- Cố ý chạy lệnh pipeline với cờ bypass (`--no-refund-fix --skip-validate`) để ép dữ liệu bẩn (14 ngày) vào ChromaDB.
- Khởi chạy đánh giá `eval_retrieval.py`. Kết quả: Model truy xuất sai thông tin, dính bẫy `hits_forbidden = yes`.
- Chạy lại Pipeline tốt để khôi phục trạng thái Vector Database chuẩn. AI lấy lại điểm 100%.

**Sprint 4: Monitoring, Docs & Grading**
- Chạy `python etl_pipeline.py freshness` để kiểm tra độ tươi của dữ liệu, phát hiện ngày cập nhật hợp đồng quá cũ, kích hoạt cảnh báo SLA WARN.
- Hoàn thiện toàn bộ tài liệu vận hành:
  - `docs/runbook.md`: Hướng dẫn xử lý sự cố.
  - `docs/quality_report_template.md`: So sánh Before/After.
  - `reports/group_report.md`: Báo cáo nhóm chi tiết.
- Chạy module tự động chấm điểm `python grading_run.py` và xuất sắc Pass **10/10 câu hỏi**.

### 3. Công nghệ áp dụng
- **Python**: Ngôn ngữ lập trình chính cho ETL Pipeline.
- **Sentence Transformers** (model `all-MiniLM-L6-v2`): Khởi tạo Embedding vector cho dữ liệu văn bản.
- **ChromaDB**: Cơ sở dữ liệu Vector lưu trữ tài liệu sau khi làm sạch.
- **HuggingFace Hub**: Để tải model nhúng dữ liệu.
- **Đóng gói Expectation Suite**: Tư duy Data Quality Testing (giống mô hình Great Expectations).

*Bằng cách xử lý triệt để Garbage In từ tầng dữ liệu, mô hình ở các Day sau sẽ hoạt động một cách tin cậy tuyệt đối.*

### 4. Hướng dẫn 

**Danh sách các file code đã sửa:**
1. `day10/lab/transform/cleaning_rules.py`: 
   - Thêm `access_control_sop` vào danh sách allowlist.
   - Bổ sung 3 rule quarantine: bắt chuỗi *"bản sync cũ"*, lọc tiền tố rác *"Nội dung không rõ ràng: "* & *"!!!"*, và khóa cứng *"10 ngày phép năm"*.
2. `day10/lab/quality/expectations.py`:
   - Thêm 2 hàm expectation mới `no_unclear_prefix` và `no_exclamation_spam` để đảm bảo dữ liệu qua clean là hoàn toàn sạch.
3. `day10/lab/contracts/data_contract.yaml` & `day10/lab/docs/data_contract.md`:
   - Bổ sung thông tin Owner, SLA, và thêm nguồn tài liệu `access_control_sop`.
4. `day10/lab/docs/runbook.md`, `day10/lab/docs/quality_report_template.md`, `day10/lab/reports/group_report.md`:
   - Điền đầy đủ số liệu Before/After, nguyên nhân lỗi và phương án xử lý (Incident Triage).

**Hướng dẫn chạy lại (Reproduce):**
Yêu cầu đã kích hoạt môi trường ảo (venv) và di chuyển vào thư mục `day10/lab`.

1. **Chạy Pipeline chuẩn (làm sạch & tạo vector):**
   ```bash
   python etl_pipeline.py run
   ```
   *(Kết quả mong đợi: `exit 0`, Pipeline OK)*

2. **Chạy đánh giá (Grading 10/10):**
   ```bash
   python grading_run.py --out artifacts/eval/grading_run.jsonl
   ```
   *(Kết quả mong đợi: Console hiển thị thanh tiến trình tải model và tạo ra file jsonl lưu kết quả 10 câu Pass 100%).*

3. **Thử nghiệm bơm dữ liệu hỏng (Inject Corruption):**
   ```bash
   python etl_pipeline.py run --run-id inject-bad --no-refund-fix --skip-validate
   python eval_retrieval.py --out artifacts/eval/after_inject_bad.csv
   ```
   *(Kết quả mong đợi: Cố tình bỏ qua bước Validate khiến dữ liệu bẩn lọt vào ChromaDB, model sẽ sinh ra câu trả lời sai (ví dụ: hoàn tiền 14 ngày).*

---

*AI in Action · VinUniversity · 2026*
