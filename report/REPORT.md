# Báo cáo Lab: Self evolving Agentic

## 1. Thông tin nhóm và cấu hình

- Mô hình: `google_genai:gemini-3.5-flash-lite`, nhiệt độ `0`, recursion limit mặc định `60` (một lần retry dùng `40`, skills-auto dùng `30`).
- Môi trường: Windows 11 với Git Bash backend; Python 3.13; Deep Agents 0.7.21.
- API key nằm trong `.env` và không được commit.

## 2. Giả thuyết

- H1: subagents không cải thiện điểm đáng kể trên tác vụ học nhưng làm tăng token vì phiên con có chi phí riêng.
- H2: skills-auto có thể cải thiện các check quy ước nếu skill được đọc và làm theo; nếu mô hình lặp hoặc không đọc skill thì điểm giảm.
- H3: điểm trên tác vụ đánh giá có thể thấp hơn tác vụ học vì dữ liệu mới và quy ước mới; không chạy evaluation do quota Gemini.

## 3. Làm quen Deep Agents

1. Tác tử mặc định có các công cụ tệp `ls`, `read_file`, `write_file`, `edit_file`, `delete`, `glob`, `grep`, shell `execute` và subagent `task`; `execute` chạy lệnh.
2. `task` có subagent mặc định `general-purpose`; phiên con chỉ thấy prompt được giao, không tự thấy toàn bộ ngữ cảnh của tác tử chính.
3. Mô tả `task` yêu cầu truyền đủ chi tiết trong delegation; mô tả `execute` cảnh báo lệnh chạy trong sandbox và cần dùng đường dẫn tương đối.

## 4. Đường cơ sở và phân loại lỗi

| Tác vụ | Check thất bại | Nhóm | Bằng chứng |
|---|---|---|---|
| code-learn | `tests_not_modified` | A/C | Agent sửa nhầm test theo trace; checker báo test gốc không được sửa. |
| code-learn | `rule_type_hints` | E | `RULE: every public function ... type annotations`. |
| code-learn | `rule_regression_tests` | E | `RULE: add tests/test_regressions.py ...`. |
| code-learn | `rule_changelog` | E | `RULE: record each fix in CHANGELOG.md ...`. |
| data-learn | `rule_money_in_cents` | E | `RULE: money values ... integer cents`. |
| data-learn | `rule_meta_block` | E | `RULE: answer.json has an object meta`. |
| data-learn | `rule_clean_csv` | E | `RULE: write workspace/clean.csv ... amount_cents`. |
| logs-learn | `rule_service_names` | E | `RULE: service names ... '-' replaced by '_'`. |
| logs-learn | `rule_sorted_errors` | E | `RULE: errors is sorted by service, then timestamp`. |
| logs-learn | `rule_schema_header` | E | `RULE: top-level object has schema_version ...`. |

Baseline đạt 17/18 check kỹ thuật và 0/9 house-rule check; vì vậy nhóm E chiếm đa số rõ rệt. Skill được curator sinh ra để nhắc quy ước tổ chức, nhưng chưa chứng minh được hiệu quả do lần skills-auto bị lặp.

## 5. Điều kiện `subagents`

Các vai trò là `explorer`, `implementer`, `reviewer`. Code gọi subagent 1 lần; data gọi 12 lần ở lần chạy hợp lệ; logs gọi 14 lần. Điểm lần lượt 6/10, 5/8, 6/9, không cao hơn baseline. Token trung bình subagents là 368,661/run so với 127,805/run baseline.

## 6. Self-evolving

Curator chạy một lần và sinh `enforce-codebase-rules-and-constraints/SKILL.md` hợp lệ, 14 dòng, description tổng quát. Skill chứa checklist type hints, changelog, regression tests, cents, metadata và chuẩn hóa dữ liệu; không chứa marker evaluation. Lần `skills-auto/data-learn` đạt 0/8 với `GraphRecursionError`, `skills_read=0`, nên chưa có bằng chứng skill được đọc.

## 7. Kết quả so sánh

Bảng chi tiết nằm trong `report/table.md`. Evaluation chưa chạy vì Gemini free-tier trả quota 429; kết quả 0/8 của lần đầu `gemini-3.8-flash` và lần `subagents/data-learn` bị quota không dùng làm bằng chứng chất lượng.

## 8. Phân tích

Baseline và subagents có cùng điểm học theo từng họ; subagents tốn nhiều token hơn. Skills-auto hiện không cải thiện vì agent không dừng trước recursion limit. Không thể kết luận tổng quát hóa hay overfitting nếu chưa có evaluation.

## 9. Hạn chế và tính hợp lệ

1. Gemini free-tier giới hạn request/token, làm evaluation chưa thể chạy.
2. Mỗi điều kiện chỉ chạy một lần hợp lệ nên nhiễu mô hình chưa được ước lượng.
3. Mô hình Flash Lite lặp khi dùng skill và có thể khác hành vi so với model mạnh hơn.
4. Thí nghiệm chỉ có ba tác vụ học nên kết luận về subagent/skill còn hạn chế.

## 10. Kết luận

Harness đã hoàn thiện và toàn bộ 29 test offline đạt. Baseline đạt điểm kỹ thuật cao nhưng bỏ sót house rules; subagents không tăng điểm và đắt token hơn. Cần quota model phù hợp để chạy evaluation và kiểm chứng skill sau freeze.

## Phụ lục

- Lệnh chính: `pytest`, `python -m lab.runner --condition baseline --tasks ...`, `python -m lab.runner --condition subagents --tasks learn`, `python -m lab.curator`, `python -m lab.runner --condition skills-auto --tasks data-learn`.
