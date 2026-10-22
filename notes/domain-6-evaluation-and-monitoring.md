# Domain 6: Evaluation and Monitoring (12%)

This domain covers measuring quality before release and watching it after: choosing a model size and architecture from evaluation numbers, running `mlflow.genai.evaluate`, knowing which judges need ground truth, writing custom scorers and judges, bringing in subject-matter experts, and then monitoring the live endpoint with the right metrics, logs, Agent Monitoring, rate limits and cost controls. Questions often show a small table of results, a record dictionary or a short scorer, and several options are real features. Requirements such as no judge-model cost, no stored text, no new identities or no self-built job decide between them.

## 6.1 Evaluating LLMs and agents

- **Choosing a model from evaluation results:** same representative set, same scorers and judge configuration for every candidate, plus latency and cost.
  1. List hard thresholds (minimum Correctness mean, minimum groundedness, p95 latency ceiling, memory budget).
  2. Eliminate every candidate that misses any threshold, even narrowly.
  3. Rank survivors on the stated objective.
- Leaderboards, general benchmarks and parameter counts describe general capability, not your app.
- **Dense vs mixture-of-experts (MoE):**

|                        | Dense             | MoE                               |
| ---------------------- | ----------------- | --------------------------------- |
| Parameters per token   | All               | Only the active experts           |
| Generation speed       | Scales with total | Scales with **active** parameters |
| GPU memory for weights | Total             | **Total** (every expert resident) |

- With a memory budget, compare it with MoE **total** parameters first. For throughput with memory to spare, compare dense size with MoE active parameters.
- **`mlflow.genai.evaluate(data, predict_fn, scorers)`** creates an evaluation run; every record gets a trace with assessments.
  - Records: `inputs` (dict whose keys are `predict_fn` parameter names, since MLflow calls `predict_fn(**inputs)`), optional `outputs`, optional `expectations` (`expected_facts`, `expected_response`, `guidelines`).
  - Data: list of dicts, pandas or Spark DataFrame, or a UC evaluation dataset.
  - `mlflow.genai.to_predict_fn("endpoints:/<name>")` evaluates a deployed endpoint (`apps:/<name>` for a Databricks App).
- **Scoring modes:**

| Data passed                           | `predict_fn`      | Scored                               |
| ------------------------------------- | ----------------- | ------------------------------------ |
| `inputs` (+ expectations)             | Required          | Fresh outputs, each with a new trace |
| `inputs` + `outputs` (+ expectations) | Must not be given | The stored outputs as they are       |
| A trace column (`search_traces`)      | Must not be given | The finished executions              |

- To test a new version on captured production requests: rebuild records from each trace's request as `inputs`, drop the trace column, keep `predict_fn`.
- Stored question-and-answer tables have no trace, so tool or retrieval scorers have nothing to read.
- **Diagnose with traces:** fact missing from the retriever span → retrieval defect. Fact present, answer ignores it → generation defect. Wrong tool or tool error → tool selection or the tool.
- **Comparing runs:** select two runs in the Evaluation runs view and Compare, or pair traces from `search_traces(run_id=...)` by `inputs`. Run means can hide per-request regressions.
- **Ground truth by judge:**

| Judge                                         | Needs expectations?                                   |
| --------------------------------------------- | ----------------------------------------------------- |
| `Correctness`                                 | Yes (`expected_facts` or `expected_response`)         |
| `RetrievalSufficiency`                        | Yes                                                   |
| `ToolCallCorrectness`                         | Yes in Databricks' judges table (expected tool calls) |
| `ExpectationsGuidelines`                      | No reference answer, but per-record guidelines        |
| `RelevanceToQuery`, `Safety`, `Guidelines`    | No                                                    |
| `RetrievalGroundedness`, `RetrievalRelevance` | No (read the retriever span)                          |
| `ToolCallEfficiency`                          | No                                                    |

- Spend labeling budget on records scored by `Correctness` or `RetrievalSufficiency`; start reference-free judges immediately.
- **`Guidelines` vs `ExpectationsGuidelines`:** global rules applied to every response (brand, legal, tone) vs rules stored per record (a leave answer must name the approval form). Both are pass/fail.
- **Code scorers (`@scorer`):** for exact, deterministic, zero-model-cost checks (valid JSON, required field, length, regex, tool order).
  - Parameters filled by name: `inputs`, `outputs`, `expectations`, `trace`.
  - Return a primitive or one `Feedback` (one metric), or a list of distinctly named `Feedback` (several metrics).
  - An uncaught exception becomes `Feedback(value=None, error=...)` for that record; evaluation continues.
- **Trace-based scorers:** `trace.search_spans(span_type=SpanType.TOOL)` for tool order, `SpanType.RETRIEVER` outputs for empty retrieval. `all()` over zero spans is `True`, so handle missing spans explicitly.
- **`make_judge`:** custom LLM judge from instructions with `{{ inputs }}`, `{{ outputs }}`, `{{ expectations }}` or `{{ trace }}` (trace-based judges need a model specified).
  - `feedback_value_type`: `bool` pass/fail, `int`/`float` scale, `Literal[...]` fixed categories enforced by structured output, `str` free text. Set it explicitly.

| Need                                             | Best fit                         |
| ------------------------------------------------ | -------------------------------- |
| Exact, deterministic, no model cost              | `@scorer`                        |
| Global plain-language rules, pass/fail           | `Guidelines`                     |
| Custom scale or categories over request/response | `make_judge` with inputs/outputs |
| Semantic judgement about intermediate steps      | `make_judge` with `{{ trace }}`  |

- **SME feedback:**
  - **Expectation** = what the app should produce (ground truth). **Feedback** = an assessment of what it did produce.
  - **Labeling sessions** hold traces, label schemas (such as expected facts) and reviewers; experts work in the Review App. `session.sync(to_dataset=...)` merges by `inputs`: matching records' same-named expectations are overwritten, unmatched traces become new records.
  - **Chat testing:** `get_review_app()` and `add_agent(agent_name=..., model_serving_endpoint=...)`, share the URL ending `/chat`. Reviewers need an account identity, the workspace access entitlement and `CAN_QUERY` on the endpoint.
- **Judge alignment:** `judge.align()` returns a new judge. It needs human **feedback** whose name exactly matches the judge's name, at least 10 labeled traces (50-100 better), around 30% negative labels, and real expert rationales.
- **Traps:**
  - A gate is not the objective; the most grounded model is wrong when groundedness was only a floor.
  - Adding a `predict_fn` to stored outputs replaces them.
  - Per-question rules in a global `Guidelines` judge cause false failures.
  - Feedback logged under a different name is ignored by alignment.

📖 Full lesson: [Databricks GenAI Engineer: Evaluating LLMs and Agents with MLflow Scorers and Judges](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-evaluating-llms-agents-mlflow-scorers/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 6.2 Monitoring, inference logging and cost control

- **Pick the metric that isolates the stated risk:**

| Concern                                       | Metric                                           |
| --------------------------------------------- | ------------------------------------------------ |
| Streaming chat feels slow before text appears | Time to first token (TTFT)                       |
| Full replies take too long                    | End-to-end latency (median and tail)             |
| Offline job must finish by a deadline         | Throughput vs work remaining                     |
| Spend creeping up                             | Input and output tokens per request, per caller  |
| Failures and throttling                       | Error rate by status code (4XX, 429, 5XX)        |
| Unsupported answers                           | Groundedness-style judge score on sampled traces |

- **Inference tables** (AI Gateway): request and response payloads, status, latency, requester, `served_entity_id` and request ID in a UC Delta table. No app code change.
  - Rows are delivered in the background, typically **within about an hour**. Use MLflow traces for anything needed now.
  - Oversized payloads are null with `logging_error_codes` (such as `MAX_REQUEST_SIZE_EXCEEDED`), even when the call returned 200.
  - Error responses (401, 403, 429, 500) may not be logged, so do not count throttling from this table.
  - Streamed responses are aggregated into one logged response.
- **Traffic splits and fallbacks:** group by `served_entity_id` (join `system.serving.served_entities` for names) to compare a canary within the same window. With AI Gateway fallbacks (external models), the tables log the first successful attempt, so an outage covered by a fallback shows as rows served by the fallback entity.
- **Usage tracking:** `system.serving.endpoint_usage` holds exact input/output tokens, requester, status, served entity and request ID, but **no text**. Join the inference table on request ID to read the costliest prompts.
  - `requester` behind an app backend is that one service principal.
  - **`usage_context`**: a map sent in the JSON request body (inside `extra_body` with the OpenAI client) and copied into the usage table, to attribute tokens per end user or tenant.
- **Which record:**

| Record                          | Holds                                                      | Use for                           |
| ------------------------------- | ---------------------------------------------------------- | --------------------------------- |
| Inference table                 | Payloads, status, latency, requester                       | Audit, debugging, evaluation data |
| `system.serving.endpoint_usage` | Token counts, requester, `usage_context`                   | Token reporting, chargeback       |
| `system.billing.usage`          | DBUs by SKU, `usage_metadata.endpoint_name`, `custom_tags` | Spend, cost allocation, budgets   |
| MLflow traces                   | Every step in near real time, scorer feedback              | Live debugging, monitoring        |

- **Assessing a deployed RAG app from logs:** parse inference-table payloads into `inputs` and `outputs` and run `mlflow.genai.evaluate` without a `predict_fn`. Payloads alone support answer-level judges (`RelevanceToQuery`, `Safety`, `Guidelines`); retrieval judges need traces with retriever spans; `Correctness` needs labeled expectations.
- **Regression on real traffic:** pull last month's real questions into an evaluation dataset and evaluate live and candidate versions with the same scorers.
- **`agents.deploy()`** sets up real-time tracing to the experiment, inference tables and the Review App. It does **not** start monitoring scorers, set rate limits or enable scale to zero. Endpoints you create yourself need `ENABLE_MLFLOW_TRACING=true` on the served entity.
- **Agent Monitoring:** registered scorers run on sampled live traces and attach feedback; allow roughly 15-20 minutes for the first scores.
  - Register, then start with `ScorerSamplingConfig` (`sample_rate`, `filter_string`); `update()` changes sampling; `stop()` sets the rate to zero.
  - `backfill_scorers` (from `databricks.agents.scorers`) scores past traces; default window is the last seven days, or set `start_time` and `end_time`.
  - Custom `@scorer` functions for monitoring must be defined in a Databricks notebook with imports **inside** the function; class-based scorers are not supported.
- **Rate limits enforce; tables and budgets only observe.** Exceeding a limit returns HTTP 429.

| Scope                            | Caps                                    |
| -------------------------------- | --------------------------------------- |
| Endpoint                         | All traffic combined                    |
| User (Default)                   | Each user without a more specific limit |
| Custom user or service principal | One named identity                      |
| Custom group                     | Group members                           |

- **Precedence:** custom limits override User (Default); a user-specific limit beats a group limit; a user in several groups is limited only after exceeding all of them (effectively the most generous); the endpoint limit still caps everyone.
- **Agent and custom model endpoints take QPM only.** To cap an agent's tokens, set TPM on the foundation or external model endpoint it calls.
- **Limits key on identity, never on `usage_context`.** To cap per tenant, give each tenant its own service principal.
- **Cost levers:**

| Lever                            | Fixes                                                        |
| -------------------------------- | ------------------------------------------------------------ |
| Scale to zero                    | Idle compute on hosted endpoints (slow first response after) |
| Pay-per-token                    | Sporadic, low-volume use of a supported model                |
| Provisioned throughput           | Steady high volume, guarantees, fine-tuned models            |
| `ai_query` batch inference       | Table-shaped offline work sent row by row                    |
| `max_tokens` and concise prompts | Unread output tokens                                         |
| Rate limits                      | Runaway loops and heavy callers                              |

- **Spend tracking:** `system.billing.usage` serving SKUs; group by `usage_metadata.endpoint_name` (covers every endpoint) or `custom_tags` set on the endpoint. Tags on UC models or experiments do not reach billing. Scale-to-zero restarts appear under a SKU containing `SERVERLESS_REAL_TIME_INFERENCE_LAUNCH`. An account budget filtered on a tag alerts but never blocks.
- **Traps:**
  - TTFT, not total latency, explains "nothing appears for a while" in streaming chat.
  - An empty inference table minutes after a load test usually means rows have not landed yet.
  - Mostly idle provisioned throughput for a model also offered pay-per-token is a billing-model mismatch; limits and chargeback do not change it.
  - A data profiling monitor watches statistical drift, not judged answer quality.

📖 Full lesson: [Databricks GenAI Engineer: Monitoring, Inference Tables and LLM Cost Control](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-monitoring-inference-tables-cost-control/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

[← Back to the study guide](../README.md)
