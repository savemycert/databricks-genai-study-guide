# Domain 3: Application Development (30%)

The heaviest domain. It covers the application layer: which framework orchestrates the model calls, how prompts are templated, versioned, augmented and engineered, how you judge response quality and add enforced guardrails, how you pick the LLM and embedding model, and how agents are authored, logged, authenticated, traced, deployed and connected to Genie. Most questions name one deciding constraint (fewest packages, no redeploy, no code changes, per-user access, exact values, lowest cost once quality ties) and offer several options that would work in principle.

## 3.1 Frameworks, prompt augmentation and prompt engineering

- **Pick the framework whose abstraction matches the app's shape**, and use none for a single fixed call.

| Tool         | Built for                                        | Choose when                                                            |
| ------------ | ------------------------------------------------ | ---------------------------------------------------------------------- |
| LangChain    | Composing swappable components into a chain      | A fixed, linear sequence: retrieve → template → model → parse          |
| LangGraph    | Stateful graphs with loops and a checkpointer    | The agent loops, branches, or pauses for a human and resumes           |
| LlamaIndex   | Data connectors, indexes, query engines          | Ingesting many sources and querying them, no agent control flow        |
| DSPy         | Signatures and modules compiled against a metric | You have examples and a metric and want prompts searched automatically |
| Plain client | Calling an endpoint directly                     | One fixed call, fewest third-party packages                            |

- Foundation Model API endpoints accept the OpenAI client, and `mlflow.openai.autolog()` traces it without LangChain.
- MLflow Tracing is observability, not orchestration.
- **`databricks-langchain` classes:**

| Class                    | Wraps                                     |
| ------------------------ | ----------------------------------------- |
| `ChatDatabricks`         | A chat model on a serving endpoint        |
| `DatabricksEmbeddings`   | An embedding model on a serving endpoint  |
| `DatabricksVectorSearch` | A Vector Search index (`.as_retriever()`) |
| `UCFunctionToolkit`      | Unity Catalog functions as agent tools    |

- **`DatabricksVectorSearch` arguments:** a Delta Sync index with Databricks-managed embeddings needs only `index_name`. Self-managed embeddings or a Direct Access index also need `text_column` and `embedding` set to the **same model** that built the stored vectors.
- **Template variable syntax:**

| System                       | Variable       | Literal brace          |
| ---------------------------- | -------------- | ---------------------- |
| MLflow Prompt Registry       | `{{question}}` | Single braces are text |
| LangChain (f-string default) | `{question}`   | `{{` and `}}`          |

- A registry template written with single braces has no variables, so `format()` returns it unchanged.
- `prompt.to_single_brace_format()` gives LangChain a usable copy without changing the registered version.
- Literal JSON braces in a LangChain template are read as variables and fail; double them.
- **Prompt Registry (Beta):** prompts are UC entities `catalog.schema.name`.
  - `register_prompt()` creates a new immutable version. Fix a mistake by registering a new version.
  - `set_prompt_alias()` moves a mutable alias. That is both promotion and rollback.
  - `load_prompt("prompts:/main.hr.policy_qa/3")` pins version 3. `@production` follows the alias. `@latest` is the newest version, including unevaluated ones.
  - Release pattern: app loads the alias per request → register → evaluate → move the alias. No redeploy.
- **Automated optimization:**
  - DSPy optimizes only programs written in DSPy.
  - `mlflow.genai.optimize_prompts()` (Beta) works on registry prompts with `predict_fn`, `train_data` (inputs + expectations), `prompt_uris`, scorers and an optimizer, and registers the result as a **new version**.
  - `predict_fn` must load the template from the registered prompt. A hard-coded string ignores every candidate.
  - The new version is not live until you evaluate it and move the alias.
- **Augmentation from the request:**
  - **Key fields** (invoice number, product version, region) drive an exact lookup whose facts go into the prompt as labeled context. Similarity search returns lookalikes and fine-tuning freezes changing data.
  - Supply facts the model cannot know, such as today's date.
  - Fields that scope access come from the authenticated session, never from the message text.
  - **Terms:** expand internal acronyms and codenames with a glossary, or map a codename to the official name before retrieval. Resolve an ambiguous term with a key field (the user's team).
  - **Intents:** classify with a small, cheap model over a fixed label set with an "other" label, then add only that intent's instruction block. A message can carry several intents.
- **Structure:** delimit and label each chunk (`<doc id="HR-12" title="...">`) and ask for cited IDs. Wrap user-supplied content in tags and say it is content, not instructions. Pass fewer chunks, best first ("lost in the middle"). Put the latest question last.
- **Baseline to target:** state audience, tone, length and format explicitly, and show one or two target-style examples. Prefer positive instructions over lists of things to avoid. Distill long style guides into a few rules plus an example.
- **Step-by-step reasoning:** reasoning **before** the answer, the answer in its own delimited field, and only the answer shown to users. Add it only to intents that need it, since it adds tokens and latency.
- **Grounding:** answer only from the context, with a named fallback. Use an exact sentinel (such as `INSUFFICIENT_CONTEXT`) when code branches on it. Allow answers supported by passages taken together, with citations.
- **Traps:**
  - `max_tokens` limits output length and truncates; it does not shape structure or fix "lost in the middle".
  - Temperature changes randomness, not the target and not invention.
  - A bigger model still needs the target stated.
  - "Answer only if a single passage states it" over-refuses answers that need two passages.
  - "Give the category, then explain" produces a justification, not better accuracy.

📖 Full lesson: [Databricks GenAI Engineer: LangChain, Prompt Augmentation and Prompt Engineering](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-langchain-prompt-engineering/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 3.2 Response quality, safety and guardrails

- **Assess qualitatively by reading the response beside its retrieved context** in an MLflow trace, and tag each issue separately.

| Issue        | Looks like                                                     | Usual fix direction                                |
| ------------ | -------------------------------------------------------------- | -------------------------------------------------- |
| Relevance    | Grounded, but answers a different question                     | Retrieval (wrong document)                         |
| Completeness | Covers part of what the context held                           | Prompt and generation; chunking if never retrieved |
| Groundedness | Claims in no retrieved chunk                                   | Grounding instructions                             |
| Format       | Right content, wrong presentation for the channel              | Format instruction or post-processing              |
| Safety       | Harmful advice, toxic or biased content, exposed personal data | Enforced guardrail; often change the index         |

- Latency, token counts, length and similarity scores are operational signals. They cannot reveal hallucination or unsafe content.
- **Retrieval vs generation:** read the retriever span.
  - Fact in the retrieved chunks, answer wrong → **generation** failure.
  - Fact never reached the prompt → **retrieval** failure. "Cannot find it" under a grounding instruction is correct behaviour.
  - Nothing about the subject retrieved → check the content is indexed. A **triggered** Delta Sync index picks up new rows only when a sync runs; **continuous** keeps syncing.
- **Hallucination** is usually the model filling a gap from general knowledge. Cheapest fix: a grounding instruction plus a fallback. Fine-tuning is slow and rarely first.
- **Safety vs quality:** safety issues block release and need enforced controls. **Grounded is not safe:** salary sheets in an HR index can be quoted with every word supported. Remove such material from that application's corpus.
- **Over-refusal** (safe, in-scope requests blocked) is a quality failure. Track false positives and false negatives on a labeled set. Make the rule more precise; never drop required protection.
  - Over-broad prompt rule → name the specific harms and give allowed examples.
  - Built-in Safety filter misfiring on a specialist domain → it has a fixed taxonomy, so use a classifier you control (Llama Guard).
- **Choosing chunking from evaluation:** same labeled set, embedding model, retriever settings, prompt and LLM for every candidate. Apply hard constraints (token budget, latency) as a filter, then rank on recall@k or hit rate and answer correctness. Similarity scores are not comparable across chunk sizes.
- **Context window budget:** window − prompt and history − answer reserve, divided by max chunk size, rounded down. Example: 32,768 − 4,000 − 2,000 = 26,768 → 13 chunks of 2,000 fit, 14 do not.
- **Chunking symptoms:**

| Symptom                                                  | Change to evaluate                                         |
| -------------------------------------------------------- | ---------------------------------------------------------- |
| Passages end mid-thought                                 | Add overlap                                                |
| Multi-part answers partial, rest in a neighbouring chunk | Larger, structure-aligned chunks with some overlap         |
| Values retrieved without their labels                    | Structure-aware chunking                                   |
| Right document, diffuse answers, expensive prompts       | Smaller, section-level chunks                              |
| Retrieval depends on where in the chunk the answer sits  | Chunks within the embedding limit, or a longer-input model |

- Embedding limits: `databricks-bge-large-en` 512 tokens, `databricks-gte-large-en` 8,192 tokens. Past the limit is silently truncated.
- **AI Gateway guardrails** on a Model Serving endpoint, no app code change:
  - **Safety:** built-in Llama Guard-based classifier with a fixed taxonomy. No custom categories.
  - **PII detection:** Block, Mask or None. No custom term lists, no harmful-content detection.
  - **Per endpoint:** a second endpoint the app starts using is unprotected until configured.
  - **Per direction:** an input check never sees retrieved documents or tool results; only an output check does.
  - **Output guardrails do not run on streaming requests or embeddings models.** If nothing unscreened may ever display, request without streaming.
  - **Topic moderation and keyword filtering are deprecated and no longer enforced.** Implement them as your own pre/post-processing checks.
  - Rate limits, usage tracking and inference tables do not block content.
  - Databricks labels these endpoint settings legacy and offers Unity Gateway as the newer layer; the exam guide describes the endpoint guardrails.
- **Llama Guard** as a chain step: custom categories written into its prompt, no training. Output is `safe`, or `unsafe` plus category codes. Check the turn the rule concerns; a rule about what the assistant says checks the drafted reply.
- **Layered guardrails:** prompt rules shape behaviour; independent checks enforce it. Mechanical rules get deterministic code checks: cited IDs vs retrieved IDs, schema validation with a failure path, required text by string comparison.
- **Traps:**
  - Reranking cannot bring back a chunk that was never retrieved or never embedded.
  - Asking the main model to police itself is still prompting; it lowers the rate, it guarantees nothing.
  - The Safety guardrail knows nothing about fabricated IDs, malformed JSON or missing disclaimers.

📖 Full lesson: [Databricks GenAI Engineer: Response Quality, Safety and LLM Guardrails](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-response-quality-safety-guardrails/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 3.3 Choosing LLMs and embedding models

- **Order of decision:** hard requirements (pass/fail) → quality bar on your own evaluation data → minimise the objective (cost or latency).
- **Hard requirements:** languages, tool calling, structured output, context window, where prompts may be processed, licence, fine-tuning plans, release status (GA-only policies).

| Attribute                                   | Implication                                           |
| ------------------------------------------- | ----------------------------------------------------- |
| High volume, simple task                    | Small instruct model                                  |
| User waiting on screen                      | No long thinking phase; fast-streaming standard model |
| Multi-step reasoning, latency-tolerant      | A reasoning model, if it measurably raises accuracy   |
| Long inputs that cannot be chunked          | Window that holds the input plus the reply            |
| Agent calling APIs; output to a typed table | Function calling; structured outputs                  |
| Data stays in workspace; future fine-tuning | Open weights on Databricks Model Serving              |

- **Smallest model that meets the bar.** Per-token price usually dominates cost. Compare on a representative evaluation set, not hand-picked prompts.
- **One-off vs per-request costs:** fine-tuning is paid once and suits narrow, repetitive tasks with plenty of labeled examples. At high volume, per-request costs dominate.
- **Context window = everything in the request + max output.** Example: 40,000-token transcript + 1,500 instructions + 3,000 output = 44,500, so 32K fails and 64K fits. Buy no more than you need.
- **Structured outputs:** `response_format` with a JSON schema constrains decoding. Prompt instructions only encourage a format.
- **Reasoning vs standard instruct:** reasoning models bill thinking tokens and delay the first visible word. Use one only when a test shows a gain on dependent-step work and latency allows.
- **Hosting:**

| Option                                 | Prompts processed in              | Fine-tune weights? | Billing                                                     |
| -------------------------------------- | --------------------------------- | ------------------ | ----------------------------------------------------------- |
| Open weights on provisioned throughput | Databricks Model Serving          | Yes                | Reserved capacity; steady or sensitive load                 |
| Foundation Model APIs pay-per-token    | Databricks-hosted shared endpoint | No                 | Per token; variable or modest traffic                       |
| External model endpoint                | Forwarded to the provider         | No                 | Provider key in Databricks secrets; governed via AI Gateway |

- **Base vs Instruct:** deploy Instruct; start fine-tuning and continued pretraining from base. A deployed base model keeps writing in the style of its input instead of answering.
- **Embedding model input length** must cover the longest chunk **and** the longest query.

| Endpoint                          | Max input    | Notes                                                   |
| --------------------------------- | ------------ | ------------------------------------------------------- |
| `databricks-gte-large-en`         | 8,192 tokens | English, 1,024 dims; long chunks, short whole documents |
| `databricks-bge-large-en`         | 512 tokens   | English, 1,024 dims; paragraph-sized passages           |
| `databricks-qwen3-embedding-0-6b` | about 32K    | Multilingual; Public Preview at the time of writing     |

- **Match the unit to the queries:** precise lookups favour short passages. Whole-document questions with one row per document need a window that holds the longest document, or embed a summary. Long pasted queries push the query side over the limit too.
- **Consistency:** queries and documents use the same model. Switching means re-embedding the corpus and changing the query path together. Equal dimensions do not make vectors comparable.
- **Cross-language retrieval** needs a multilingual **embedding** model, not only a multilingual LLM.
- **Finding models:**
  - `system.ai`: Databricks-provided, permissive licences, already in UC, servable from Catalog Explorer.
  - Marketplace: shared into a read-only UC catalog when you obtain the listing.
  - External hubs (Hugging Face): download, log with MLflow and register in UC yourself.
- **Model cards screen; experiments decide.** Check task type, languages, context length and licence.
- **Licences:** Apache 2.0 and MIT permit commercial use. CC BY-NC and research-only do not. "Other" means read the licence. A fine-tune is bound by its base model's licence.
- **Experiment metrics:**

| Task                          | Metric                                        |
| ----------------------------- | --------------------------------------------- |
| Summarisation with references | ROUGE (recall-oriented n-gram overlap)        |
| Translation with references   | BLEU (precision-oriented n-gram overlap)      |
| Short factual answers         | Exact match                                   |
| Classification                | Accuracy, per-class precision, recall, F1     |
| Fit to domain text            | Perplexity on held-out text (lower is better) |

- **Thresholds filter, the objective ranks.** A run that leads on one metric is out if it breaks another requirement.
- **Is the gap real?** Only on the same evaluation set, and small sets make small gaps noise.
- **Traps:**
  - Overall accuracy hides a rare class. Use recall when missing a case is costly, precision when false alarms are.
  - Extra context window adds no value once the requirement is met.
  - A preview model is out when policy allows only GA features.

📖 Full lesson: [Databricks GenAI Engineer: Choosing LLMs and Embedding Models](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-choosing-llms-embedding-models/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 3.4 Agentic development with MLflow, Agent Framework and Genie

- **`ResponsesAgent`:** subclass `mlflow.pyfunc.ResponsesAgent` and implement `predict` (and `predict_stream`). Any framework's agent stays as it is. You get an inferred signature, automatic tracing and compatibility with AI Playground, the Review App, inference tables and monitoring. `ChatAgent` is the older interface.
- **Streaming:** `predict_stream` yields `response.output_text.delta` events sharing one `item_id`, then a `response.output_item.done` event with the full item, which MLflow uses to aggregate the trace.
- **Per-request values:**

| Mechanism              | Scope                                                    |
| ---------------------- | -------------------------------------------------------- |
| `custom_inputs`        | Per request (session ID, tier); typed, read in `predict` |
| `custom_outputs`       | Per request, returned to the caller                      |
| `model_config`         | Fixed when the logged version loads                      |
| Endpoint env variables | Shared by all requests                                   |

- **Models from Code:** `agent.py` ends with `mlflow.models.set_model(agent)`, and `log_model(python_model="agent.py")` (or `lc_model=`) gets the file path. Avoids pickling `WorkspaceClient` or compiled graphs.
  - `code_paths` copies local modules the agent imports (`ModuleNotFoundError` otherwise).
  - `pip_requirements` lists published packages, not local folders.
  - Validate with `mlflow.models.predict(model_uri, input_data=...)`, which loads the logged artifact as serving will.
- **Authentication:**

|                   | Automatic passthrough            | On-behalf-of-user (OBO)                                       |
| ----------------- | -------------------------------- | ------------------------------------------------------------- |
| Identity          | The endpoint's service principal | The requesting user                                           |
| Declared as       | Resources (`SystemAuthPolicy`)   | `UserAuthPolicy(api_scopes=[...])`                            |
| Every caller gets | The same access                  | Their own UC permissions, row filters, audit                  |
| Client built      | Anywhere                         | Inside `predict`, per request (`ModelServingUserCredentials`) |

- Passthrough covers only resources declared at log time (`DatabricksServingEndpoint`, `DatabricksVectorSearchIndex`, `DatabricksFunction`, `DatabricksGenieSpace`, `DatabricksTable`, `DatabricksSQLWarehouse`). Undeclared means unreachable, and adding a tool means **logging a new version**.
- A Genie space also needs its tables, SQL warehouse and UC functions declared.
- OBO needs an admin to enable it. If some users fail through the same version, those users lack access to the resource. Granting the service principal changes nothing for user-authorized calls.
- One agent can mix both with `AuthPolicy(system_auth_policy=..., user_auth_policy=...)`.
- **Tracing:** `mlflow.<flavor>.autolog()` for supported libraries; `@mlflow.trace` (with `span_type`) for custom functions; `mlflow.start_span()` for a code block. `update_current_trace()` adds tags, and `start_run`/`log_metric` are experiment tracking; neither creates spans.
- **`databricks.agents.deploy(uc_model_name, version)`** sets up the serving endpoint, short-lived credentials, a **Review App**, real-time tracing to the experiment and AI Gateway inference tables. It does not create indexes or Genie spaces.
  - New version: call `deploy()` again with the same UC model name. The endpoint updates in place, same URL, no gap.
  - A UC alias such as `champion` does not change what the endpoint serves until you deploy again.
  - The Review App collects stakeholder feedback as assessments on traces and hosts expert labeling sessions.
- **Evaluation vs monitoring:**

|         | Evaluation (offline)                               | Monitoring (production)                                  |
| ------- | -------------------------------------------------- | -------------------------------------------------------- |
| When    | Before release; a release gate                     | Continuously after deployment                            |
| API     | `mlflow.genai.evaluate(data, predict_fn, scorers)` | Scorers `.register()` then `.start(sampling_config=...)` |
| Inputs  | Curated dataset, often with expectations           | Live traffic, no expectations                            |
| Results | An evaluation run                                  | Assessments on scored traces                             |

- Reuse the same scorers in both phases so numbers are comparable.
- `ScorerSamplingConfig`: `sample_rate` 1.0 for required checks such as safety, roughly 0.05-0.2 for expensive judges; `filter_string` excludes test traffic.
- To compare versions fairly, evaluate both on the same labeled dataset. Live scores reflect whatever traffic each got.
- **Judges and ground truth:** `Correctness`, `RetrievalSufficiency` and `ExpectationsGuidelines` need expectations and stay offline. `RelevanceToQuery`, `Safety`, `RetrievalGroundedness`, `RetrievalRelevance` and `Guidelines` run in monitoring.
- **Traces to evaluation data:** `mlflow.search_traces` → `dataset.merge_records(traces)` (into a UC evaluation dataset from `create_dataset`) → experts add expected facts in a labeling session → `mlflow.genai.evaluate` on every future version.
- **Genie vs retriever:** Genie turns questions into SQL run on a SQL warehouse under UC permissions, so totals and filters are exact. A retriever returns a few similar chunks and suits documents.
- **`GenieAgent`** (`databricks_langchain`) wraps one space as a worker in a coded supervisor, with the space ID and a **description** that drives routing. One `GenieAgent` per space; keep spaces narrowly focused.
- **Improve Genie inside the space:** example SQL for hard multi-part questions, SQL expressions for metrics, column descriptions and synonyms, then text instructions last. Supervisor prompts cannot fix wrong SQL.
- **Conversation API is asynchronous:** `start-conversation` → poll the message until `COMPLETED`, `FAILED` or `CANCELLED` → read `query-result` from the attachment. Follow-ups post to the same conversation.
  - Poll every 1-5 seconds, back off up to a minute, stop after about 10 minutes.
  - One conversation per user session.
- **Genie runs SQL as the caller:** `CAN RUN` on the space plus the Databricks SQL entitlement, `CAN USE` on the warehouse, `SELECT` on tables plus `USE CATALOG` and `USE SCHEMA`. For per-person row filters, call with each user's OAuth token (or OBO), not one service principal.
- **Traps:**
  - A client-side stream flag cannot stream an agent that never yields partial output.
  - Calling the in-memory agent in a notebook does not validate the logged artifact.
  - `expected response = actual response` makes every answer pass.
  - Misrouting to the wrong worker is a description problem, not a space problem.

📖 Full lesson: [Databricks GenAI Engineer: Building Agents with MLflow, Agent Framework and Genie](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-agent-framework-mlflow-genie/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

[← Back to the study guide](../README.md)
