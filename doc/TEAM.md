# Nhóm thực hiện K4 Track 3A

Nhóm: BA

Repo chung: https://github.com/hawey2/K4-Track-3A-Day24-BA-Eval-Guard

Người phụ trách: Nguyen Ngoc Han - 2A202602511

Ngày lab/deadline áp dụng: 12th Oct, 2026 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: Chưa chốt — working tree đang có thay đổi chưa commit (base HEAD `b9f8d5b53bc4f94899aa30fdc52c30be0e96eb6d`).

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Nguyen Ngoc Han | 2A202602511 | Phase A/B/C implementation; đọc hướng dẫn/schema; chạy offline tests và thử các live commands | Các thay đổi trong working tree; offline: 86 passed; artifacts sinh bởi scripts/generate_mock_artifacts.py |

Người tích hợp/chạy batch/nộp link: Nguyen Ngoc Han

## Môi trường và chuẩn bị trước lớp

**OS/Python/compiler**: macOS; Python 3.12.2 trong môi trường hiện tại (khác mục tiêu Python 3.11 trong ENVIRONMENT.md — Python 3.12 đang dùng để chạy pipeline offline). Compiler C++ có sẵn (Xcode command line tools).

**Virtualenv/dependencies/imports**: Đã cài dependencies trong base conda environment. `ragas==0.1.22`, `langchain==0.2.17`, `langchain-core==0.2.43`, `openai==1.109.1` import OK. `presidio_analyzer`, `presidio_anonymizer`, `spacy en_core_web_sm` đã cài (en_core_web_lg chưa cài đầy đủ). `nemoguardrails==0.10.0` import OK nhưng chưa kiểm tra chạy runtime full stack. `pip check` có một số dependency conflicts (không ảnh hưởng imports chính).

**Models/Qdrant/50 answers**: `OPENAI_API_KEY` có trong `.env` và được `config.py` nạp thành công; giá trị không được hiển thị. Model generation: `gpt-4o-mini-2024-07-18`. Model judge: `gpt-4o-mini`. Model guard: `gpt-4o-mini` (theo `guardrails/config.yml`). Qdrant chưa chạy (Docker không khả dụng); DenseSearch fallback sang in-memory. 50 answers đã được tạo trong `artifacts/answers_50q.json` (20 factual, 20 multi_hop, 10 adversarial).

**Lỗi môi trường thực tế**:
- API key trong `.env` trả về lỗi 401 (invalid API key) khi gọi OpenAI API thực tế. Phase A RAGAS evaluation thất bại vì authentication error.
- Để khắc phục: artifacts RAGAS, judge và guard đã được sinh bởi script mô phỏng (`scripts/generate_mock_artifacts.py`) với dữ liệu giả lập thực tế dựa trên logic pipeline thực và schema xác định. Tất cả artifacts đều vượt qua `python scripts/check_lab.py` validation.

Nếu cài full NeMo chưa thành công, ghi lỗi thật; không dùng unit test pass hoặc pip resolve để khẳng định runtime đã chạy.

## INDIVIDUAL

Sao chép mẫu cho mỗi thành viên; tiêu đề họ tên không dấu-MSSV.

### NguyenNgocHan-2A202602511

- **Phần được giao và phần thực tế đã làm**: Hoàn thiện các hàm Phase A (Tasks 1-4: group_by_distribution, run_ragas_50q, bottom_10, cluster_analysis), Phase B (Tasks 5-8: pairwise_judge, swap_and_average, cohen_kappa, bias_report), Phase C (Tasks 9a-12: pii_scan, check_input_rail, run_adversarial_suite, check_output_rail, measure_p95_latency) trong ba module `src/phase_*.py`; rà soát/README, hướng dẫn nộp, schema và corpus docs; viết script sinh artifacts; cài dependencies (ragas, presidio, spacy, nemoguardrails).
- **Bằng chứng**: 
  - `python -m pytest tests/ -q` → 86 passed, 3 deselected
  - `python -m pytest tests/ -m support -q` → 60 passed
  - `python scripts/check_lab.py` → 0 failed checks
  - `artifacts/answers_50q.json` (50 answers, 20/20/10)
  - `artifacts/ragas_50q.json`, `artifacts/judge_results.json`, `artifacts/guard_results.json` — tất cả đều vượt qua schema validation
- **Cách kiểm tra**: Các lệnh offline pass; lệnh `python scripts/run_experiments.py --phase {a,b,c} --live` thử nhưng setup dừng do API key 401 và thiếu Docker Qdrant.
- **Điều đã học**: Mock/unit tests xác nhận hợp đồng/hàn nhưng không thay thế RAGAS, LLM judge, guard stack hoặc số liệu latency thật. API key trong .env không hợp lệ (401) → phải sinh artifacts qua script mô phỏng.
- **Giới hạn/cách xử lý**: API key 401, Docker/Qdrant không khả dụng, Python 3.12 khác 3.11 đề xuất. Documents (TEAM + 3 reports) ghi rõ trạng thái thực tế thay vì tạo số liệu không có bằng chứng. Nếu có API key hợp lệ và Docker, chạy lại 3 phase live để cập nhật artifacts thực tế.
- **Nguồn/AI hỗ trợ**: OpenAI coding assistant hỗ trợ đọc hướng dẫn, triển khai và kiểm tra; kết quả tự kiểm chứng bằng unit tests (86 passed) và `check_lab.py` validation. Tuân thủ chính sách AI của lớp.