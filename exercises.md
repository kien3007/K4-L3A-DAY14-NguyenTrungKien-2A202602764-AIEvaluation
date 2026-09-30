# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Câu hỏi mang tính xã giao, chào hỏi hoặc ngoài phạm vi kiến thức mà trợ lý từ chối hợp lệ ("Tôi không tìm thấy thông tin trong tài liệu"). | Câu trả lời nghiệp vụ (chính sách đổi trả, bảo hành, giá) chứa thông tin bịa đặt/hallucination, sai lệch so với context tài liệu. | Siết chặt prompt (grounding instruction), hạ temperature = 0.0, thêm guardrail kiểm tra hallucination trước khi phản hồi. |
| Answer Relevance | Câu hỏi mơ hồ hoặc quá ngắn, hệ thống trả lời chi tiết kèm hướng dẫn phân loại các trường hợp khiến tỷ lệ trùng lặp từ khóa loãng nhưng vẫn hữu ích. | Câu trả lời lạc đề (off-topic), trả lời lan man hoặc chuyển hướng sang vấn đề không liên quan đến thắc mắc của khách hàng. | Tối ưu hóa prompt để câu trả lời đi thẳng vào trọng tâm (direct answer first), kiểm tra prompt injection hoặc query rewriting. |
| Context Recall | Câu hỏi đơn giản chỉ cần 1 ý nhỏ trong gold context, expected answer ngắn gọn hơn tài liệu nguồn và không yêu cầu trích xuất toàn bộ context. | Retriever bỏ sót hoàn toàn văn bản chính sách/evidence quan trọng, khiến Generator không có thông tin để trả lời đúng. | Nâng cấp retrieval: tăng Top-K, điều chỉnh chunk size/overlap, áp dụng Hybrid Search (Dense + BM25) hoặc Query Expansion. |
| Context Precision | Hệ thống ưu tiên recall cao (Top-K lớn), chunk liên quan nằm ở rank 2 hoặc rank 3 thay vì rank 1 nhưng Generator vẫn tổng hợp chính xác. | Tài liệu liên quan quan trọng bị xếp ở rank rất thấp hoặc bị vùi lấp bởi các chunk rác (noise) ở đầu, làm giảm độ chính xác của câu trả lời. | Tích hợp module Re-ranking (Cross-Encoder / Cohere Rerank), cải thiện embedding model và tối ưu hóa semantic chunking. |
| Completeness | Câu hỏi mở/đa khía cạnh, expected answer liệt kê toàn diện lý thuyết nhưng câu trả lời thực tế chỉ tập trung giải quyết tình huống phổ biến nhất. | Bỏ sót các điều kiện tiên quyết hoặc bước bắt buộc trong quy trình (ví dụ: thiếu thời hạn đổi trả 7 ngày, thiếu hóa đơn gốc). | Bổ sung checklist hướng dẫn cấu trúc câu trả lời vào prompt, áp dụng Chain-of-Thought hoặc self-verification trước khi trả lời. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:*
> - **Condition 1 (Thứ tự gốc A-B):** Cung cấp cho LLM Judge cặp câu trả lời theo thứ tự [Answer A, Answer B] cùng prompt đánh giá so sánh. Ghi nhận phán quyết của Judge.
> - **Condition 2 (Đảo thứ tự B-A):** Đảo ngược vị trí trình bày thành [Answer B, Answer A] với cùng prompt, tiêu chí đánh giá và context. Ghi nhận phán quyết của Judge.
> - **Đo lường & Kết luận:** Tính tỷ lệ Judge chọn phương án ở vị trí số 1 trong cả 2 điều kiện. Nếu tỷ lệ chọn vị trí số 1 lệch đáng kể so với 50% (ví dụ > 65%) hoặc Judge đổi lựa chọn sang bất kỳ answer nào đứng trước, kết luận có Position Bias. Khắc phục bằng cách chạy cả 2 chiều (Swap Evaluation) rồi lấy điểm trung bình.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:*
> - **Bổ sung tiêu chí Conciseness (Súc tích & Trọng tâm):** Quy định rõ trong rubric trừ điểm các câu trả lời dài dòng, lặp từ, chèn câu đệm sáo rỗng hoặc lặp lại nguyên văn câu hỏi.
> - **Chấm điểm theo Checklist ý chính (Key Information Elements):** Định nghĩa rubric dựa trên danh sách các thông tin cốt lõi bắt buộc phải có, mỗi ý tương ứng điểm số cụ thể thay vì đánh giá chất lượng tổng thể dựa trên cảm quan độ dài.
> - **Ràng buộc giới hạn độ dài:** Đưa vào rubric quy chuẩn độ dài phù hợp (ví dụ: câu trả lời tối ưu trong khoảng 50–150 từ; nếu vượt quá 200 từ mà không có thông tin mới sẽ bị trừ 1 mức điểm).

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**
> *Câu trả lời:*
> LLM Judge không có nhận thức thực tế và dễ mắc các thiên kiến cố hữu (positional, verbosity, self-preference, leniency/severity bias). Việc calibrate với tập nhãn do con người/chuyên gia thẩm định giúp:
> 1. Đo lường mức độ đồng thuận và độ tương quan (Cohen's Kappa, Spearman/Pearson correlation) giữa LLM Judge và đánh giá của con người.
> 2. Phát hiện các trường hợp Judge cho điểm bất hợp lý để tinh chỉnh system prompt, rubric tiêu chí và bổ sung few-shot calibration examples.
> 3. Thiết lập ngưỡng tin cậy (threshold) chuẩn xác trước khi đưa LLM Judge vào làm quality gate tự động trong pipeline CI/CD.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | >= 0.85 | Trợ lý tư vấn khách hàng tuyệt đối không được hallucinate/bịa đặt thông tin hoặc chính sách của cửa hàng, tránh rủi ro pháp lý và giữ uy tín. |
| Answer Relevance | >= 0.80 | Đảm bảo câu trả lời tập trung trực tiếp giải quyết câu hỏi của khách hàng, không trả lời lan man hoặc lạc đề. |
| Completeness | >= 0.75 | Đảm bảo khách hàng nhận được đầy đủ các điều kiện, quy định và các bước thao tác cần thiết để tự giải quyết vấn đề. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:*
> - **Offline Evaluation:** Thực hiện trong môi trường phát triển (Dev) và quy trình CI/CD trước khi release (pre-deployment). Sử dụng Golden Dataset cố định để đo lường tự động, kiểm thử hồi quy (regression testing) và làm quality gate ngăn chặn bản build lỗi triển khai lên production.
> - **Online Evaluation:** Thực hiện liên tục trên môi trường Production với lưu lượng người dùng thật. Giám sát các chỉ số ngầm và tín hiệu trực tiếp (thumbs up/down, tỷ lệ chuyển tiếp nhân viên hỗ trợ, CSAT, bounce rate) để phát hiện data drift hoặc suy giảm chất lượng vận hành.
> - **Human Review:** Thực hiện định kỳ để audit chất lượng hệ thống, đánh giá chuyên sâu các ca điểm thấp (low confidence/outliers), phân tích root cause (5 Whys), calibrate LLM Judge và biên tập/làm giàu Golden Dataset.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
