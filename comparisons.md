# Databricks GenAI Commonly Confused Services and Concepts

Side-by-side comparisons of the Databricks GenAI services and ideas that exam questions most often set against each other. Each table ends with the one thing to remember.

- [Domain 1: Design Applications](#domain-1-design-applications)
- [Domain 2: Data Preparation](#domain-2-data-preparation)
- [Domain 3: Application Development](#domain-3-application-development)
- [Domain 4: Assembling and Deploying Applications](#domain-4-assembling-and-deploying-applications)
- [Domain 5: Governance](#domain-5-governance)
- [Domain 6: Evaluation and Monitoring](#domain-6-evaluation-and-monitoring)

## Domain 1: Design Applications

### Classification vs Extraction vs Summarization

|            | Classification                    | Extraction                            | Summarization                     |
| ---------- | --------------------------------- | ------------------------------------- | --------------------------------- |
| Returns    | Labels from a fixed set           | Values found in the text, verbatim    | New, shorter prose                |
| Good for   | Routing, tagging, counting        | Columns, reconciliation, joins on IDs | Human reading: recaps and digests |
| Limitation | Only labels that were on the list | Values, not a judgement about them    | Paraphrases and may drop details  |
| Exam cue   | "one of these queues", "tag all"  | "exactly as written", "every name"    | "a short overview for managers"   |

**Remember:** if the answer might not be on a list you maintain, it must be extracted, and exact values never travel inside a summary.

### Task-specific AI Function vs ai_query

|                | Task-specific AI Function (`ai_classify`, `ai_extract`, ...) | `ai_query`                                              |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| Prompt         | None needed                                                  | You write it                                            |
| Model          | Databricks-managed                                           | The serving endpoint you name                           |
| Output control | Fixed per function                                           | `responseFormat` schema, `modelParameters`              |
| Exam cue       | "least effort for SQL analysts"                              | "our evaluated fine-tuned endpoint", "fixed JSON shape" |

**Remember:** use the function when it already returns exactly what you need; reach for `ai_query` when the endpoint, prompt or schema must be yours.

### json_object vs json_schema vs prompt-only JSON

|            | Prompt-only "respond in JSON" | `json_object`                | `json_schema`                                      |
| ---------- | ----------------------------- | ---------------------------- | -------------------------------------------------- |
| Guarantees | Nothing                       | Valid JSON, any keys         | Your keys, types and enums                         |
| Fits       | Prototypes                    | Keys that vary per input     | A fixed object that code parses                    |
| Watch for  | Occasional broken JSON        | Not supported by every model | Size keywords not enforced; nullable fields needed |

**Remember:** only a schema in the request makes the endpoint enforce the shape, and a schema still guarantees shape, not truth.

### ai_mask vs ai_extract

|              | `ai_mask`                              | `ai_extract`                             |
| ------------ | -------------------------------------- | ---------------------------------------- |
| Output       | The original text with entities masked | The entities as separate values          |
| Text changed | Yes, in place                          | No                                       |
| Exam cue     | "de-identify before sharing the notes" | "load the names and emails into columns" |

**Remember:** mask hides, extract pulls out.

### Sequential chain vs Tool-calling agent vs Router

|          | Sequential chain             | Tool-calling agent                         | Router in front of an agent           |
| -------- | ---------------------------- | ------------------------------------------ | ------------------------------------- |
| Fits     | The same steps every request | Requests need different sources or actions | Most traffic simple, some needs tools |
| Cost     | Predictable                  | Planning loop on every request             | Cheap classifier on every request     |
| Weakness | Cannot adapt                 | Less predictable, slower                   | One more component to maintain        |
| Exam cue | "summarize, then translate"  | "some need orders, some need policies"     | "90% are simple FAQs"                 |

**Remember:** only a step in front of the agent keeps simple requests out of its planning loop.

### Knowledge Assistant vs Information Extraction vs Multiagent Supervisor

|            | Knowledge Assistant                | Information Extraction              | Multiagent Supervisor                      |
| ---------- | ---------------------------------- | ----------------------------------- | ------------------------------------------ |
| Problem    | Open questions over documents      | The same fields from every document | One entry point over several agents        |
| Output     | Answers with citations             | Rows in a table                     | Answers assembled from subagents           |
| Steered by | Sources, examples, guidelines      | Field names, types and descriptions | Subagent descriptions                      |
| Exam cue   | "chat with our manuals, cite them" | "load contract dates into a table"  | "one chat over Genie and the HR assistant" |

**Remember:** custom code on every request, computed filters or a fixed output contract means a custom Agent Framework agent instead.

### Prompt rule vs Enforcement in the tool

|                       | Prompt rule or tool description       | Check inside the tool                |
| --------------------- | ------------------------------------- | ------------------------------------ |
| Effect                | Makes the right behaviour more likely | Makes the wrong behaviour impossible |
| Survives a pushy user | No                                    | Yes                                  |
| Exam cue              | "tell the agent never to..."          | "must never", "guarantee"            |

**Remember:** when the requirement says "never" or "guarantee", the answer enforces it in code: preconditions, least privilege and idempotency inside the tool.

## Domain 2: Data Preparation

### pypdf vs pdfplumber vs pytesseract vs ai_parse_document

|              | pypdf / PyMuPDF     | pdfplumber                  | pytesseract (OCR)               | `ai_parse_document`                  |
| ------------ | ------------------- | --------------------------- | ------------------------------- | ------------------------------------ |
| Reads        | The PDF text layer  | The text layer, with tables | Page images                     | PDF, images, Word, PowerPoint, scans |
| Tables       | Run-together text   | Rows and cells              | Plain text                      | Typed `table` elements               |
| Scanned PDFs | Empty strings       | Empty strings               | Yes                             | Yes                                  |
| Install      | Third-party package | Third-party package         | Third-party package + Tesseract | Built in                             |

**Remember:** first ask whether the PDF has a text layer. No layer means OCR or `ai_parse_document`; "no libraries" or "SQL-only" means `ai_parse_document`.

### Coverage gap vs Retrieval problem

|           | Coverage gap                           | Retrieval problem                                   |
| --------- | -------------------------------------- | --------------------------------------------------- |
| Test      | Expected fact absent from chunks table | Fact present in the table but not retrieved         |
| Fix       | Ingest the source that holds it        | Chunking, embeddings, query, hybrid search, ranking |
| Wrong fix | Any retrieval tuning                   | Adding more sources                                 |

**Remember:** no retrieval setting can return text that was never ingested.

### Precision@k vs Recall@k

|            | Precision@k                           | Recall@k                                   |
| ---------- | ------------------------------------- | ------------------------------------------ |
| Formula    | Relevant in top k ÷ k                 | Relevant in top k ÷ all labeled relevant   |
| Question   | How much of what was passed is noise? | Did the needed material get passed at all? |
| As k grows | Usually falls                         | Stays the same or rises                    |
| Use when   | Few chunks fit; noise is costly       | Missing information is costly              |

**Remember:** precision divides by what was returned, recall by what should have been found, and both are measured at the k the chain passes.

### MRR vs NDCG

|        | MRR                                    | NDCG@k                                        |
| ------ | -------------------------------------- | --------------------------------------------- |
| Scores | Rank of the first relevant result only | Every relevant result, discounted by position |
| Labels | Binary                                 | Graded (highly, partly, not relevant)         |
| Fits   | One correct item per question          | Several useful items whose order matters      |

**Remember:** recall and precision ignore order inside the top k; use MRR or NDCG when order is the point, such as judging a reranker.

### RetrievalRelevance vs RetrievalGroundedness vs RetrievalSufficiency

|                    | RetrievalRelevance        | RetrievalGroundedness      | RetrievalSufficiency               |
| ------------------ | ------------------------- | -------------------------- | ---------------------------------- |
| Compares           | Each chunk vs the request | The response vs the chunks | The chunks vs the expected facts   |
| Needs expectations | No                        | No                         | Yes                                |
| A failure means    | Wrong or off-topic chunks | Hallucinated claims        | Context missing needed information |

**Remember:** only the expectation-free judges can score unlabeled production traces, and all three read `RETRIEVER` spans.

### Parent-child vs Semantic chunking vs Contextual enrichment

|         | Parent-child                        | Semantic chunking                  | Contextual enrichment                        |
| ------- | ----------------------------------- | ---------------------------------- | -------------------------------------------- |
| Changes | What is sent to the LLM             | Where chunk boundaries fall        | The text that gets embedded                  |
| Fixes   | Precise matches that lack context   | Chunks that mix subjects           | Near-identical chunks from different sources |
| Cost    | A parent lookup and dedup by parent | Extra embedding calls at ingestion | An ingestion change only                     |

**Remember:** the three combine; enriched small children can return their parents.

### Hybrid search vs Re-ranking

|                        | Hybrid search                       | Re-ranking                                    |
| ---------------------- | ----------------------------------- | --------------------------------------------- |
| Stage                  | First stage (finding candidates)    | Second stage (ordering candidates)            |
| Can add missed chunks? | Yes                                 | No                                            |
| Fixes                  | Exact identifiers that vectors blur | The right chunk ranked too low for the prompt |
| Databricks             | `query_type="HYBRID"` (RRF merge)   | `DatabricksReranker(columns_to_rerank=...)`   |

**Remember:** a chunk missing from the candidates is a first-stage problem; a chunk present but ranked too low is a reranker problem.

## Domain 3: Application Development

### LangChain vs LangGraph vs LlamaIndex vs DSPy

|           | LangChain                            | LangGraph                                         | LlamaIndex                        | DSPy                                         |
| --------- | ------------------------------------ | ------------------------------------------------- | --------------------------------- | -------------------------------------------- |
| Built for | Linear chains of standard components | Stateful graphs that loop and branch              | Ingestion, indexes, query engines | Programs compiled against a metric           |
| Fits      | Retrieve → template → model → parse  | Retry another tool, pause for approval and resume | Querying many ingested sources    | Re-optimizing prompts when the model changes |
| Exam cue  | "fixed sequence of steps"            | "loop", "human approval", "resume later"          | "connect many data sources"       | "labeled examples and a metric"              |

**Remember:** a single fixed model call needs no framework at all; the plain OpenAI client works with Foundation Model APIs and can be traced with `mlflow.openai.autolog()`.

### Prompt Registry version URI vs alias URI

|              | `prompts:/name/3`  | `prompts:/name@production`    | `prompts:/name@latest`           |
| ------------ | ------------------ | ----------------------------- | -------------------------------- |
| Resolves to  | Always version 3   | Whatever the alias points to  | Newest version, evaluated or not |
| Changes when | Never              | `set_prompt_alias()` moves it | Any new registration             |
| Use for      | Reproducible tests | Serving apps                  | Development only                 |

**Remember:** serve from a named alias, register fixes as new versions, evaluate, then move the alias to promote or roll back.

### DSPy vs mlflow.genai.optimize_prompts

|                | DSPy                             | `mlflow.genai.optimize_prompts()`           |
| -------------- | -------------------------------- | ------------------------------------------- |
| Works on       | Programs written in DSPy         | Prompts already in the Prompt Registry      |
| Output         | A compiled program               | A new version of the same registered prompt |
| Rewrite needed | Yes, into signatures and modules | No                                          |

**Remember:** `optimize_prompts` needs a `predict_fn` that loads the registered template, and its result is not live until you move the alias.

### Retrieval failure vs Generation failure

|                | Retrieval failure                         | Generation failure                    |
| -------------- | ----------------------------------------- | ------------------------------------- |
| Retriever span | The needed fact is missing                | The needed fact is there              |
| Answer         | "Cannot find it", or wrong-source content | Wrong, contradicted or embellished    |
| Fix            | Index, chunking, queries, filters, sync   | Grounding instructions, prompt, model |

**Remember:** read the retriever span first; a fix aimed at the stage that worked changes nothing.

### Safety guardrail vs PII detection vs Llama Guard vs Deterministic check

|             | AI Gateway Safety               | AI Gateway PII detection | Llama Guard step                | Deterministic code check                |
| ----------- | ------------------------------- | ------------------------ | ------------------------------- | --------------------------------------- |
| Catches     | Harmful content, fixed taxonomy | Personal data            | Your own categories             | Exact rules: IDs, schema, required text |
| Custom?     | No                              | No custom term lists     | Yes, in its prompt, no training | Yes                                     |
| Code change | None                            | None                     | A chain step                    | Pre/post-processing                     |

**Remember:** endpoint guardrails are per endpoint and per direction, output guardrails skip streaming and embeddings, and topic and keyword filters are deprecated.

### Reasoning model vs Standard instruct model

|                    | Standard instruct                | Reasoning model               |
| ------------------ | -------------------------------- | ----------------------------- |
| Best for           | Classification, extraction, chat | Dependent multi-step problems |
| First visible word | Fast                             | Delayed by the thinking phase |
| Tokens billed      | Input + output                   | Input + output + thinking     |

**Remember:** a reasoning model must earn its cost with a measured accuracy gain, and an interactive app usually cannot wait for it.

### Base vs Instruct model

|                    | Base                               | Instruct                           |
| ------------------ | ---------------------------------- | ---------------------------------- |
| Trained to         | Continue text                      | Follow instructions and converse   |
| Use for            | Fine-tuning, continued pretraining | Deployment as an assistant         |
| Symptom if misused | Keeps writing in the input's style | Tuning eroded by raw-text training |

**Remember:** deploy Instruct, fine-tune from base.

### databricks-gte-large-en vs databricks-bge-large-en

|           | `databricks-gte-large-en`                        | `databricks-bge-large-en` |
| --------- | ------------------------------------------------ | ------------------------- |
| Max input | 8,192 tokens                                     | 512 tokens                |
| Vectors   | 1,024 dimensions                                 | 1,024 dimensions          |
| Fits      | Long chunks, short whole documents, long queries | Paragraph-sized passages  |

**Remember:** both are English; switching between them means re-embedding everything, because equal dimensions do not make vectors comparable.

### ROUGE vs BLEU vs Perplexity

|           | ROUGE                       | BLEU                         | Perplexity                                |
| --------- | --------------------------- | ---------------------------- | ----------------------------------------- |
| Task      | Summarisation               | Translation                  | Fit to domain language                    |
| Measures  | Recall of reference n-grams | Precision against references | How well the model predicts held-out text |
| Better is | Higher                      | Higher                       | Lower                                     |

**Remember:** for an imbalanced classifier, look at per-class recall or precision, not overall accuracy.

### Automatic passthrough vs On-behalf-of-user authorization

|             | Automatic passthrough                           | On-behalf-of-user                                      |
| ----------- | ----------------------------------------------- | ------------------------------------------------------ |
| Acts as     | The endpoint's service principal                | The person who sent the request                        |
| Declared    | Resources at log time                           | API scopes in a `UserAuthPolicy`                       |
| Row filters | Same for every caller                           | Per user                                               |
| Exam cue    | "shared index", "users may not access directly" | "each employee sees only their rows", "audit per user" |

**Remember:** an undeclared resource fails under passthrough, and a new resource means logging a new version.

### Evaluation vs Monitoring

|          | Evaluation                               | Monitoring                           |
| -------- | ---------------------------------------- | ------------------------------------ |
| Question | Is this version good enough to ship?     | Is the live app still good?          |
| Data     | Curated dataset, often with expectations | Sampled live traces, no expectations |
| Judges   | All, including Correctness               | Reference-free only                  |

**Remember:** use the same scorers in both, and compare versions offline on one labeled set, not on live scores.

### Genie space vs Vector Search retriever

|            | Genie space                     | Vector Search retriever     |
| ---------- | ------------------------------- | --------------------------- |
| Data       | Governed, structured tables     | Unstructured documents      |
| Returns    | Exact SQL results over all rows | The top few similar chunks  |
| Aggregates | Correct                         | Wrong: an LLM sums a sample |

**Remember:** fix wrong Genie SQL inside the space (example SQL, expressions, synonyms), and fix misrouting in the worker description.

## Domain 4: Assembling and Deploying Applications

### LangChain flavor vs pyfunc PythonModel vs ResponsesAgent

|               | `mlflow.langchain` flavor           | `PythonModel` subclass                    | `ResponsesAgent` subclass        |
| ------------- | ----------------------------------- | ----------------------------------------- | -------------------------------- |
| Fits          | Pure LCEL chain others will compose | Custom pre/post-processing around the LLM | New agents for Agent Framework   |
| Signature     | Inferred from an input example      | You supply it or an input example         | Inferred automatically           |
| Loads back as | A LangChain runnable                | pyfunc                                    | pyfunc speaking the agent format |

**Remember:** if other teams need the runnable itself, log with the LangChain flavor; any pyfunc wrapper hides it.

### `__init__` vs load_context vs predict

|      | `__init__`                   | `load_context`                       | `predict`                        |
| ---- | ---------------------------- | ------------------------------------ | -------------------------------- |
| Runs | When the instance is created | Once per model load                  | Every request                    |
| Fits | Plain picklable settings     | Clients, tokenizers, artifact files  | Request-specific work            |
| Risk | Live clients fail to pickle  | None if setup is request-independent | Setup here is paid on every call |

**Remember:** work that does not depend on the request goes in `load_context`.

### pip_requirements vs extra_pip_requirements vs code_paths vs artifacts

|               | `pip_requirements`  | `extra_pip_requirements` | `code_paths`       | `artifacts`                 |
| ------------- | ------------------- | ------------------------ | ------------------ | --------------------------- |
| Adds          | A full package list | Extra packages           | Your local modules | Data files                  |
| Inferred list | Replaced            | Kept                     | Not affected       | Not affected                |
| At load       | Installed           | Installed                | On `sys.path`      | Path in `context.artifacts` |

**Remember:** only one of `pip_requirements`, `extra_pip_requirements` and `conda_env` per call; `code_paths` and `artifacts` combine with any of them.

### Delta Sync (managed) vs Delta Sync (self-managed) vs Direct Vector Access

|             | Delta Sync, managed embeddings                              | Delta Sync, self-managed                          | Direct Vector Access       |
| ----------- | ----------------------------------------------------------- | ------------------------------------------------- | -------------------------- |
| Source      | Delta table with a text column                              | Delta table with a vector column                  | None                       |
| Create args | `embedding_source_column` + `embedding_model_endpoint_name` | `embedding_vector_column` + `embedding_dimension` | Schema, key, vector column |
| Updates     | Sync                                                        | Sync                                              | `upsert` / `delete`        |
| Query       | `query_text`                                                | `query_vector`                                    | `query_vector`             |

**Remember:** only an index that knows its embedding model accepts `query_text`.

### Triggered vs Continuous sync

|           | Triggered                               | Continuous                                   |
| --------- | --------------------------------------- | -------------------------------------------- |
| Updates   | Only when `sync()` or Sync now runs     | Within seconds of each table change          |
| Cost      | Cheapest mode                           | Streaming cluster always running             |
| Endpoints | Standard and storage-optimized          | Standard only                                |
| Fits      | Batch-loaded tables; call `sync()` last | Tables edited all day that must show at once |

**Remember:** a triggered index never watches its table, and change data feed is a prerequisite, not a trigger.

### Standard vs Storage-optimized Vector Search endpoint

|                   | Standard                        | Storage-optimized          |
| ----------------- | ------------------------------- | -------------------------- |
| Scale             | Hundreds of millions of vectors | Over a billion             |
| Latency           | Tens of milliseconds            | A few hundred milliseconds |
| Throughput        | Scales with `target_qps`        | Modest                     |
| Filters           | Dictionary                      | SQL-like string            |
| `columns_to_sync` | Supported                       | Not supported              |

**Remember:** tight latency and high QPS mean standard; huge, cost-sensitive, latency-tolerant indexes mean storage-optimized.

### CAN VIEW vs CAN QUERY vs CAN MANAGE (serving endpoint)

|         | `CAN VIEW`   | `CAN QUERY`                       | `CAN MANAGE`                                      |
| ------- | ------------ | --------------------------------- | ------------------------------------------------- |
| Allows  | See and list | Send requests                     | Change config, roll back, delete, set permissions |
| Give to | Observers    | Apps, analysts, `ai_query()` jobs | Owners and operators                              |

**Remember:** UC `EXECUTE` on the model does not let anyone call the endpoint.

### Pay-per-token vs Provisioned throughput vs External model endpoint

|         | Pay-per-token              | Provisioned throughput                   | External model endpoint       |
| ------- | -------------------------- | ---------------------------------------- | ----------------------------- |
| Models  | A fixed Databricks set     | Base plus fine-tuned UC models           | Third-party provider models   |
| Billing | Per token                  | Per provisioned capacity                 | By the provider               |
| Fits    | Prototypes, irregular load | Production guarantees, fine-tunes, HIPAA | Governed access to a provider |

**Remember:** a fine-tuned UC model needs a provisioned throughput endpoint.

### Real-time endpoint vs ai_query() batch

|                 | Real-time serving endpoint | `ai_query()`                                     |
| --------------- | -------------------------- | ------------------------------------------------ |
| Someone waiting | Yes                        | No                                               |
| Input           | One request                | Rows already in a table                          |
| Error handling  | Per request                | `failOnError => false` isolates bad rows         |
| Compute         | The endpoint               | Serverless/pro SQL or DBR 15.4 LTS+, not Classic |

**Remember:** "nightly", "backfill" and "rows in a table" mean `ai_query()`; "at checkout" or "in a chat" means an endpoint.

### Automatic passthrough vs Manual authentication

|             | Automatic passthrough               | Manual authentication                                  |
| ----------- | ----------------------------------- | ------------------------------------------------------ |
| Credentials | Short-lived, for declared resources | Your SP OAuth values in secret-backed env vars         |
| Covers      | Supported resource types only       | Anything the SP can reach, such as the Prompt Registry |
| Side effect | None                                | Disables passthrough for every resource                |

**Remember:** after switching to manual auth, grant the service principal access to the index and LLM endpoint too.

### Deploy code vs Deploy models

|             | Deploy code                                 | Deploy models                          |
| ----------- | ------------------------------------------- | -------------------------------------- |
| Moves       | Source code through bundle targets          | A model version built in dev           |
| Data assets | Built in each environment from its own data | Carry whatever dev embedded            |
| Fits        | Production data must stay in production     | Dev may use production-equivalent data |

**Remember:** the data-access constraint decides; deploy code is the usual recommendation.

### Component test vs Evaluation gate vs Smoke test

|         | Component test                | Evaluation gate                   | Smoke test                            |
| ------- | ----------------------------- | --------------------------------- | ------------------------------------- |
| Scope   | One tool, parser or retriever | The whole agent or prompt version | The deployed endpoint                 |
| LLM     | Stubbed                       | Real, with scorers                | Real, through the endpoint's identity |
| Catches | Logic bugs                    | Quality regressions               | Undeclared resources, serving faults  |

**Remember:** Ready only means the container started.

### Managed vs External vs Custom MCP server

|              | Databricks-managed      | External                                  | Custom                      |
| ------------ | ----------------------- | ----------------------------------------- | --------------------------- |
| Who hosts    | Databricks              | The vendor                                | You, on Databricks Apps     |
| Prerequisite | Databricks provides one | The vendor already speaks MCP             | None exists                 |
| Governance   | Built in                | UC HTTP connection; callers get `EXECUTE` | App permissions (`CAN USE`) |

**Remember:** a plain REST API is not an MCP server; wrap it in a custom server (the OpenAPI template saves most of the code).

### App authorization vs User authorization (Databricks Apps)

|             | App authorization                          | User authorization + OBO in the agent |
| ----------- | ------------------------------------------ | ------------------------------------- |
| Identity    | The app's service principal                | Each signed-in user                   |
| Row filters | Same rows for everyone                     | Each user's own rows                  |
| Setup       | Endpoint as an app resource with Can query | Model-serving scope plus agent OBO    |

**Remember:** forwarding the user's token does nothing if the agent's tools still run as the endpoint's service principal.

## Domain 5: Governance

### PII detection Mask vs Block

|              | Mask                                       | Block                                     |
| ------------ | ------------------------------------------ | ----------------------------------------- |
| What happens | Personal data replaced; the call continues | The whole request or response is rejected |
| User gets    | An answer                                  | No answer                                 |
| Fits         | Privacy rule plus an answer-rate objective | Policy demands outright rejection         |

**Remember:** Mask keeps the data from the model and keeps the answers; Block keeps the data from the model by giving up the answer.

### Input-side vs Output-side guardrails

|                                      | Input (requests)                                  | Output (responses)                               |
| ------------------------------------ | ------------------------------------------------- | ------------------------------------------------ |
| Runs                                 | Before the model or provider receives the request | After generation                                 |
| Keeps data from a provider           | Yes                                               | No; the provider already saw it                  |
| Streaming                            | Still runs                                        | Not supported                                    |
| Sees retrieved docs and tool results | No                                                | Only what reaches the reply through the endpoint |

**Remember:** "must never be sent to the provider" means input-side; "must never be shown" needs output-side, and streaming rules output guardrails out.

### ai_mask vs ai_extract vs ai_classify

|         | `ai_mask(text, labels)`                       | `ai_extract(text, labels)`    | `ai_classify(text, labels)`   |
| ------- | --------------------------------------------- | ----------------------------- | ----------------------------- |
| Returns | The text with labelled entities as `[MASKED]` | The entity values as fields   | One label for the whole text  |
| Use for | Redacting before indexing                     | Pulling entities into columns | Filtering records by category |

**Remember:** only `ai_mask` redacts, and it only masks the labels you list.

### ai_mask at ingestion vs Unity Catalog column masks

|                         | `ai_mask` at ingestion               | UC column mask / row filter                                          |
| ----------------------- | ------------------------------------ | -------------------------------------------------------------------- |
| When it applies         | Once, when text is written           | At query time, per user                                              |
| Who still sees raw data | Nobody (the stored text is redacted) | Admins and excluded groups                                           |
| Free text               | Finds entities inside it             | Cannot reliably find names in free text; row filters drop whole rows |

**Remember:** when everyone must retrieve the text but nobody may see the personal data inside it, redact the stored text.

### Masking records vs Excluding records

|              | Mask the entities, keep the records | Exclude records with personal data           |
| ------------ | ----------------------------------- | -------------------------------------------- |
| Privacy      | Met                                 | Met                                          |
| Content      | All useful content kept             | Most content lost when most records have PII |
| Distribution | Unchanged                           | Skewed                                       |

**Remember:** exclusion, synthetic data and access restrictions either lose content or leave the data retrievable.

### Direct vs Indirect prompt injection

|                 | Direct                                       | Indirect                                      |
| --------------- | -------------------------------------------- | --------------------------------------------- |
| Arrives in      | What the user types                          | Content the app reads: pages, uploads, emails |
| User's question | The attack itself                            | Can be completely ordinary                    |
| Defend at       | The user message, before retrieval and tools | Ingestion and fetch time                      |

**Remember:** screening is probabilistic, so pair it with blast-radius controls in code and permissions.

### Prompt-level defences vs Blast-radius controls

|                                    | Prompt-level (system prompt rules, refusal examples) | Blast-radius (permissions, code)                                              |
| ---------------------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------- |
| Depends on the model               | Yes                                                  | No                                                                            |
| Holds after a successful injection | No                                                   | Yes                                                                           |
| Examples                           | "Never reveal...", lower temperature                 | Least-privilege grants, OBO auth, tool-derived parameters, approvals, secrets |

**Remember:** a manipulated model cannot be trusted to enforce its own rules.

### CC BY vs CC BY-NC vs No license

|                | CC BY              | CC BY-NC                        | No license                          |
| -------------- | ------------------ | ------------------------------- | ----------------------------------- |
| Commercial use | Allowed            | Not without separate permission | Not without the author's permission |
| Condition      | Appropriate credit | Non-commercial only             | All rights reserved                 |

**Remember:** public is not licensed, and retrieval-only use does not lift an NC restriction.

### Filter at ingestion vs Switch the source

|      | Filter at ingestion                           | Recommend an alternative source              |
| ---- | --------------------------------------------- | -------------------------------------------- |
| Fits | A minority of records are problematic         | The problem pervades the source              |
| How  | `ai_classify` labels, current-status metadata | An authoritative, curated or licensed source |

**Remember:** fix problematic text in the data; output guardrails are a backstop, not the fix.

## Domain 6: Evaluation and Monitoring

### Dense model vs Mixture-of-experts model

|                         | Dense      | Mixture-of-experts      |
| ----------------------- | ---------- | ----------------------- |
| Parameters per token    | All        | Only the active experts |
| Generation speed tracks | Total size | Active parameters       |
| GPU memory tracks       | Total size | Total size              |

**Remember:** an MoE is fast like its active count but must fit in memory like its total count.

### Threshold vs Objective

|          | Threshold (gate)                                        | Objective                       |
| -------- | ------------------------------------------------------- | ------------------------------- |
| Role     | Eliminates candidates                                   | Ranks the survivors             |
| Examples | Minimum correctness, p95 latency ceiling, memory budget | Lowest cost, highest throughput |
| Mistake  | Ignoring it because a model wins the objective          | Treating a floor as the goal    |

**Remember:** apply every threshold first; if only one model survives, the objective never gets a say.

### Running predict_fn vs Scoring stored outputs or traces

|        | With `predict_fn`                   | Stored `outputs` or a trace column            |
| ------ | ----------------------------------- | --------------------------------------------- |
| Scores | Fresh answers from the app you pass | Answers that already exist                    |
| Traces | New, one per record                 | Only if a trace column was passed             |
| Fits   | Testing a version                   | Judging past answers, or a model that is gone |

**Remember:** passing a `predict_fn` with stored outputs replaces them; to test a new version on old requests, keep only the `inputs`.

### Guidelines vs ExpectationsGuidelines

|            | `Guidelines`             | `ExpectationsGuidelines`       |
| ---------- | ------------------------ | ------------------------------ |
| Rules live | Passed once to the judge | In each record's expectations  |
| Applied to | Every response           | Only that record's response    |
| Fits       | Brand, tone, legal rules | Question-specific requirements |

**Remember:** per-question rules in a global judge produce false failures wherever a rule does not apply.

### @scorer vs make_judge

|               | `@scorer` code scorer        | `make_judge` LLM judge                           |
| ------------- | ---------------------------- | ------------------------------------------------ |
| Checks        | Exact, objective rules       | Subjective judgement: rubric, tone, evidence use |
| Deterministic | Yes                          | No                                               |
| Model cost    | None                         | An LLM call per record                           |
| Output shape  | Primitive or `Feedback` list | Set by `feedback_value_type`                     |

**Remember:** tool order and presence checks belong in a code scorer reading `trace.search_spans`.

### Expectation vs Feedback

|             | Expectation                                               | Feedback                                |
| ----------- | --------------------------------------------------------- | --------------------------------------- |
| Meaning     | What the app should produce                               | An assessment of what it did produce    |
| Used by     | Correctness, RetrievalSufficiency, ExpectationsGuidelines | Quality tracking, judge alignment       |
| Logged with | `mlflow.log_expectation`, expected-facts schema           | `mlflow.log_feedback`, feedback schemas |

**Remember:** judge alignment reads feedback named exactly like the judge, not expected facts.

### Inference table vs Usage table vs Billing table vs Traces

|               | Inference table                                   | `system.serving.endpoint_usage`     | `system.billing.usage`                  | MLflow traces                            |
| ------------- | ------------------------------------------------- | ----------------------------------- | --------------------------------------- | ---------------------------------------- |
| Holds         | Request and response payloads, latency            | Exact token counts, `usage_context` | DBUs by SKU, endpoint name, custom tags | Every step of a request, scorer feedback |
| Does not hold | Exact tokens, oversized payloads, some error rows | Any text                            | Per-request detail, tokens, text        | Billing figures                          |
| Use for       | Audit, eval data                                  | Chargeback per caller or tenant     | DBU spend, budgets                      | Live debugging, monitoring               |

**Remember:** choose the record by the unit asked for: text, tokens, DBUs or steps. Inference table rows arrive in the background, typically within about an hour, so use traces for anything needed now.

### Time to first token vs End-to-end latency vs Throughput

|             | TTFT                                | End-to-end latency               | Throughput                 |
| ----------- | ----------------------------------- | -------------------------------- | -------------------------- |
| Measures    | Wait before the first token appears | Time to the full reply           | Work completed per minute  |
| Matters for | Streaming chat                      | Non-streamed interactive replies | Batch jobs with a deadline |

**Remember:** when nobody waits on a single request, per-request latency is irrelevant.

### Rate limit vs Budget

|        | AI Gateway rate limit                 | Account budget                         |
| ------ | ------------------------------------- | -------------------------------------- |
| Effect | Rejects requests over the limit (429) | Sends an alert                         |
| Unit   | QPM or TPM per scope                  | Spend, filtered by tag                 |
| Fits   | Stop a runaway loop or heavy caller   | Notify finance without blocking anyone |

**Remember:** only a rate limit enforces; inference tables, usage tables and budgets observe.

### Scale to zero vs Pay-per-token vs Provisioned throughput

|           | Scale to zero                           | Pay-per-token                     | Provisioned throughput                  |
| --------- | --------------------------------------- | --------------------------------- | --------------------------------------- |
| Removes   | Idle compute on a hosted endpoint       | Reserved capacity altogether      | Nothing idle; you pay while provisioned |
| Trade-off | Slow first response after quiet periods | Shared capacity, no guarantee     | Paid whether used or not                |
| Fits      | Dev, test, occasional use               | Sporadic use of a supported model | Steady volume, guarantees, fine-tunes   |

**Remember:** mostly idle reserved capacity for a model also offered pay-per-token is a billing-model mismatch.

[← Back to the study guide](README.md)
