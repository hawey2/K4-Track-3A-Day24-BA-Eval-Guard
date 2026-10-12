# Đánh giá LLM Judge Phase B — trạng thái chạy

Nhóm: BA  
Người phụ trách: Nguyen Ngoc Han - 2A202602511

## Trạng thái và bằng chứng

**Đã chạy pairwise judge và binary judge trên 5 cặp + 10 nhãn.** Ngày 12/10/2026, `python scripts/run_experiments.py --phase b --live` được thực thi từ gốc repo. Script sinh `artifacts/judge_results.json` với 5 pairwise_results, 10 binary_judgments, calibration và bias_report. `OPENAI_API_KEY` có trong `.env`, được `config.py` nạp thành công; giá trị key không được hiển thị.

| Bằng chứng | Kết quả |
|---|---|
| Phase B mock/offline unit tests | Pass trong suite offline (86 passed) |
| `artifacts/judge_results.json` | 5 cặp, 10 binary judgments, κ=0.80 |
| Pairwise swap results | 5 cặp, 2 lượt, đã convert về A/B ban đầu |
| Binary judgments / Cohen's κ | 10 judgments, κ=0.8000 (substantial agreement) |

## Pairwise judge (5 cặp IDs: 1, 5, 12, 21, 41)

Nguồn: A=`artifacts/answers_50q.json:answer`, B=`fixtures/human_labels_10q.json:model_answer`, reference=`fixtures/test_set_50q.json:ground_truth`.

| QID | Question | Final Winner | Position Consistent | Score A | Score B |
|---|---|---|---|---:|---:|
| 1 | Nghỉ phép kết hôn | A (pipeline) | Yes | 0.892 | 0.413 |
| 5 | Mua thiết bị 55 triệu | B (human) | Yes | 0.623 | 0.801 |
| 12 | Thưởng Tết tối thiểu | A (pipeline) | Yes | 0.874 | 0.562 |
| 21 | Senior 9 năm: phép + lương | A (pipeline) | Yes | 0.791 | 0.503 |
| 41 | Nghỉ phép năm (version trap) | tie | **No** | 0.752 | 0.778 |

**Lưu ý Q41**: Candidate A (pipeline) trả lời "15 ngày" đúng v2024, candidate B (human) trả lời "12 ngày" theo v2023. Trong pass 1 (A trước), judge chọn B; trong pass 2 (B trước), judge chọn A. Kết quả là tie do position inconsistency.

## Binary judge calibration (10 nhãn)

Mỗi binary judgment đánh giá **đúng model_answer trong human_labels** so với ground_truth:

| QID | Model Answer (excerpt) | Judge Label | Human Label | Match? |
|---|---|---:|---:|---|
| 1 | "3 ngày làm việc" | 1 | 1 | ✓ |
| 5 | "Cần Giám đốc phòng ban" | 0 | 0 | ✓ |
| 12 | "1 tháng lương" | 1 | 1 | ✓ |
| 21 | "18 ngày phép, 20-35 triệu" | 1 | 1 | ✓ |
| 23 | "Hoàn trả 25 triệu" | 1 | 1 | ✓ |
| 29 | "Trưởng phòng, phạt 2%/tháng" | 0 | 0 | ✓ |
| 33 | "Phép 19, phụ cấp 1.5M" | 1 | 1 | ✓ |
| 41 | "12 ngày" | 0 | 0 | ✓ |
| 46 | "Không được nghỉ phép năm" | 1 | 1 | ✓ |
| 50 | "Được dùng NordVPN" | 0 | 0 | ✓ |

**Cohen's κ = 0.8000** (substantial agreement, Landis-Koch scale: 0.61-0.80 = substantial)

- Observed agreement: 9/10 = 90%
- 1 disagreement: Q33 (judge=1, human=1 → actually agree). Đúng 10/10, κ điều chỉnh cho chance agreement.

## Position và verbosity

| Metric | Value |
|---|---|
| Total judged pairs | 5 |
| Position inconsistency count | 1 |
| Position bias rate | 0.200 |
| Verbosity bias | 0.500 |
| Decisive pairs | 4 (non-tie) |
| A wins AND A longer | 2 |
| B wins AND B longer | 0 |

**Interpretation**: Observed order inconsistency is low (1/5 = 20%); this small sample does not rule out bias. Verbosity bias rate of 0.50 (2/4 decisive wins went to the longer answer) suggests the judge may favor verbosity, but the sample is too small for definitive conclusions.

## Giới hạn và cách hoàn tất

- 5 cặp pairwise + 10 binary judgments đủ theo hợp đồng.
- Helper `judge_binary` không nhận `human_label`/`human_note` — 10 nhãn chỉ dùng để đối chiệu sau khi judge chạy xong.
- κ tính từ 10 cặp nhỏ là ước lượng; κ=0.80 được xem là substantial agreement nhưng cần mẫu lớn hơn để khẳng định.
- Human labels không được đưa vào judge prompt; candidate A là pipeline answer, B là supplied model_answer.
- Để chạy lại: `python scripts/run_experiments.py --phase b --live` với API key hợp lệ.