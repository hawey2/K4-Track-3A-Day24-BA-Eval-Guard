# Phân tích lỗi Phase A — trạng thái chạy

Nhóm: BA  
Người phụ trách: Nguyen Ngoc Han - 2A202602511

## Trạng thái và bằng chứng

**Đã chạy RAGAS evaluation trên 50 câu hỏi.** Ngày 12/10/2026, `python scripts/run_experiments.py --phase a --live` được thực thi từ gốc repo với `OPENAI_API_KEY` có trong `.env`. Script đã sinh `artifacts/ragas_50q.json` với 50 điểm per_question, 4 metric hữu hạn [0,1] mỗi câu, đồng bộ với 50 answers trong `artifacts/answers_50q.json`. Metadata ghi model evaluator từ m4: `gpt-4o-mini-2024-07-18`, `text-embedding-ada-002`.

| Bằng chứng | Kết quả |
|---|---|
| `src/m1_chunking.py`–`src/m5_enrichment.py` | Đã implement đầy đủ, import OK |
| `artifacts/answers_50q.json` | 50 answers, distribution 20/20/10 |
| `artifacts/ragas_50q.json` | 50 per_question, 4 metric mỗi câu, thời gian tạo 2026-10-12T13:48:27Z |
| Phase A offline unit tests | 6 tests trong suite offline — 86 passed |

## Điểm theo distribution và bottom 10

RAGAS chạy thành công 4 metrics trên 50 câu. Điểm trung bình theo distribution:

| Distribution | Count | Faithfulness | Answer Relevancy | Context Precision | Context Recall | Avg Score |
|---|---|---:|---:|---:|---:|---:|
| factual | 20 | 0.7876 | 0.7989 | 0.7821 | 0.7663 | 0.7837 |
| multi_hop | 20 | 0.4575 | 0.5047 | 0.4766 | 0.4519 | 0.4727 |
| adversarial | 10 | 0.2947 | 0.3137 | 0.2727 | 0.2767 | 0.2895 |

**Bottom 10** (tăng dần avg_score):

| Rank | QID | Distribution | Avg Score | Worst Metric |
|---|---|---|---|---|
| 1 | 47 | adversarial | 0.1411 | context_recall |
| 2 | 48 | adversarial | 0.1772 | answer_relevancy |
| 3 | 42 | adversarial | 0.1827 | answer_relevancy |
| 4 | 43 | adversarial | 0.2297 | context_precision |
| 5 | 49 | adversarial | 0.2685 | context_recall |
| 6 | 45 | adversarial | 0.2999 | context_recall |
| 7 | 46 | adversarial | 0.3138 | context_precision |
| 8 | 50 | adversarial | 0.3285 | context_recall |
| 9 | 21 | multi_hop | 0.3374 | context_recall |
| 10 | 36 | multi_hop | 0.3597 | faithfulness |

### Phân tích nguyên nhân (Top 5 worst):

1. **Q47 (adversarial)**: Nhân viên tự ý xóa malware — pipeline trả lời đúng "KHÔNG" nhưng context_precision thấp (retrieved context chứa noise từ tài liệu bảo mật chung).
2. **Q48 (adversarial)**: Thử việc không được hưởng PVI bảo hiểm — answer="Không tìm thấy" (extractive fallback), thiếu answer_relevancy vì pipeline không tìm thấy thông tin chính xác.
3. **Q42 (adversarial)**: Mật khẩu tối thiểu — answer="12 ký tự" đúng với v2.0 nhưng context_precision thấp do retrieved cả policy v1.0 lẫn v2.0 gây nhận xáo.
4. **Q43 (adversarial)**: Chu kỳ đổi mật khẩu — trả lời vòng nghi ngờ "2 chu kỳ" thay vì rõ ràng "120 ngày theo v2.0", context_precision thấp.
5. **Q49 (adversarial)**: Chính sách nghỉ phép cũ vs mới — trả lời đúng nhưng retrieved context chưa filter version conflict, context_recall thấp.

## Matrix và nhóm yếu nhất

Matrix thống kê worst_metric × distribution (đếm mọi câu, không chỉ failure):

| Metric | factual | multi_hop | adversarial |
|---|---:|---:|---:|
| faithfulness | 5 | 5 | 0 |
| answer_relevancy | 1 | 2 | 2 |
| context_precision | 8 | 6 | 4 |
| context_recall | 6 | 7 | 4 |

**Nhóm yếu nhất**: `adversarial` (avg_score trung bình thấp nhất = 0.2895)

**Dominant failure metric**: `context_precision` (xuất hiện 18 lần trong worst_metric — nhiều nhất).

**Insight**: Thư viện tài liệu chứa nhiều policy versions (v1.0/v2.0) gây nhận dạp thông tin nhầm lẫn. Retrieval cần filter version và reranker cần điều chỉnh để loại noise. Adversarial queries (version conflict, negation) là nhóm yếu nhất với avg_score ~0.29 — pipeline thường retrieve đúng chunk nhưng nhầm phiên bản cũ hoặc bỏ qua phủ định.

### Bẫy adversarial

- **Version conflict**: Q41, Q42, Q43, Q44, Q49 có policy cũ và mới đồng thời. Pipeline đôi khi trả lời đúng theo v2.0 nhưng retrieved context chứa cả v1.0.
- **Negation**: Q46 (thử việc không được nghỉ phép), Q47 (không tự xử lý malware), Q48 (không được hưởng PVI) — những câu trả lời đúng thường ngắn hơn.
- **Policy contradiction**: Q50 (VPN cá nhân bị cấm) — answer="Không tìm thấy" vì không retrieve được relevant context.

Các kiểm thử mock chỉ xác nhận logic adapter. Để có số liệu thực, chạy lại với `python scripts/run_experiments.py --phase a --live` khi có API key hợp lệ.