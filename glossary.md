# Databricks GenAI Glossary

225 Databricks GenAI terms and services, each in one plain-English sentence.

[#](#) · [A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [J](#j) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [Q](#q) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w)

## #

- **@scorer**: A decorator that turns a Python function into a code-based scorer.

## A

- **Account budget**: A billing alert on spend that notifies without blocking requests.
- **Active parameters**: The parameters an MoE model uses for each token, which drive its generation speed.
- **Agent Bricks**: Databricks' low-code way to build common agents from configuration rather than code.
- **Agent Monitoring**: Running registered scorers automatically on sampled live traces of a deployed agent.
- **agents.deploy()**: The Agent Framework call that deploys a UC model version with an endpoint, Review App, tracing and inference tables.
- **AI Functions**: Databricks SQL functions such as `ai_classify`, `ai_extract` and `ai_summarize` that run a task on Databricks-managed models without a prompt.
- **AI Gateway**: Model Serving endpoint features for guardrails, rate limits, usage tracking and inference tables.
- **AI Gateway fallback**: Routing a request to the next served entity when one returns 429 or a 5XX error.
- **ai_mask**: A Databricks SQL AI function that replaces entities of the listed types with `[MASKED]`.
- **ai_parse_document**: A built-in Databricks AI function that turns a document's binary content into typed layout elements.
- **ai_query**: A Databricks SQL function that calls a named serving endpoint with your own prompt and optional `responseFormat` and `modelParameters`.
- **artifacts**: A `log_model` argument that packages data files, read back through `context.artifacts`.
- **Authoritative source**: A replacement data source with a named owner, a review process and authority on the subject.
- **Auto Loader**: A Databricks streaming source (`cloudFiles`) that processes only new files, tracked in a checkpoint.
- **Automatic authentication passthrough**: Agent access to declared resources using short-lived credentials for the endpoint's service principal.

## B

- **backfill_scorers**: A function that runs registered scorers over traces logged before monitoring started.
- **Base model**: A model version trained only to continue text, used as the starting point for fine-tuning.
- **Blast radius**: How much harm a manipulated model can do, limited by permissions and code rather than prompts.
- **BLEU**: A precision-oriented n-gram overlap metric for translation against reference translations.
- **Block (PII behaviour)**: Rejecting a request or response in which personal data is detected.
- **Boilerplate**: Repeated text such as headers, footers and navigation that dominates embeddings and crowds out real answers.

## C

- **cache_ttl_seconds**: A `load_prompt` argument that sets how long an alias-based prompt load is cached.
- **CAN MANAGE (endpoint)**: The serving endpoint permission that allows configuration changes, deletion and permission management.
- **CAN QUERY**: The serving endpoint permission that allows sending requests.
- **CC BY**: A Creative Commons license that allows commercial use with appropriate credit.
- **CC BY-NC**: A Creative Commons license that forbids commercial use.
- **Chain**: An ordered set of components, such as retriever, prompt template, model and parser, that turns an input into an output.
- **Change data feed**: A Delta table feature that publishes row-level inserts, updates and deletes, required for Delta Sync on a standard endpoint.
- **ChatDatabricks**: The `databricks-langchain` class for a chat model on a Model Serving endpoint.
- **Checkpoint location**: The storage path where Auto Loader records which files it has already processed.
- **Chunk overlap**: Text copied from the end of one chunk into the start of the next so boundary facts stay whole.
- **Chunking**: Splitting extracted text into pieces sized for the embedding model and the LLM's context budget.
- **code_paths**: A `log_model` argument that packages local modules the agent imports.
- **Column mask**: A Unity Catalog function that changes what a given user sees in a column at query time.
- **Context budget**: The share of the generator's context window left for retrieved chunks after the prompt and answer reserve.
- **Contextual enrichment**: Prepending identifying context such as title or product to a chunk's text before embedding it.
- **Correctness**: A built-in judge that checks a response against expected facts or an expected response.
- **Coverage gap**: A failure where the needed fact is not in any indexed chunk, fixed only by ingesting a new source.
- **CREATE MODEL VERSION**: The Unity Catalog privilege to add versions to an existing registered model.
- **Cross-encoder**: A model that reads the query and a candidate together and outputs a relevance score.
- **Custom tags (endpoint)**: Key-value tags on a serving endpoint that appear in billing rows for cost allocation.
- **custom_inputs**: A request field that carries typed per-request values, such as a session ID, to an agent.

## D

- **Databricks Apps**: A Databricks service for hosting web apps, such as agent chat UIs and custom MCP servers.
- **DatabricksEmbeddings**: The `databricks-langchain` class for an embedding model on a serving endpoint.
- **DatabricksReranker**: The Vector Search built-in reranker, enabled on `similarity_search` with `columns_to_rerank`.
- **DatabricksVectorSearch**: The `databricks-langchain` class that exposes a Vector Search index as a LangChain vector store.
- **Declarative Automation Bundles**: The current name for Databricks Asset Bundles, which deploy code, jobs and endpoints per target.
- **Delta Sync index**: A Vector Search index that stays in sync with a Unity Catalog Delta source table.
- **Deterministic output check**: A code check, such as schema validation or citation matching, that enforces an exact rule.
- **Direct prompt injection**: User input written to override an application's instructions or role.
- **Direct Vector Access index**: A vector index with no source table, written only through upsert and delete calls.
- **DSPy**: A framework that declares steps as signatures and modules and compiles their prompts against a metric.

## E

- **Embedding**: A vector representation of text in which texts with similar meaning sit close together.
- **Embedding token limit**: The maximum input an embedding model reads; anything longer is silently truncated.
- **ENABLE_MLFLOW_TRACING**: An environment variable that turns on production tracing for agents on endpoints you create yourself.
- **Entity extraction**: Pulling specific values that appear in a text, such as names or IDs, out as separate fields.
- **Enum**: A schema constraint that limits a field to a closed set of exact strings.
- **env_manager**: An `mlflow.models.predict` option that chooses an isolated (virtualenv, uv) or local environment.
- **Evaluation dataset**: A Unity Catalog table of evaluation records, grown from traces with `merge_records`.
- **Evaluation gate**: A pipeline step that promotes a pinned candidate version only if its evaluation scores pass.
- **Evaluation run**: An MLflow run created by `mlflow.genai.evaluate` in which each record has a trace with assessments.
- **EXECUTE (model)**: The Unity Catalog privilege needed to load a registered model.
- **Expectations**: Ground-truth fields, such as expected facts, stored with evaluation rows for judges that need them.
- **ExpectationsGuidelines**: A built-in pass/fail judge that applies rules stored in each record's expectations.
- **External model endpoint**: A Model Serving endpoint that forwards requests to a third-party provider using a key stored in secrets.
- **extra_pip_requirements**: A `log_model` argument that appends packages to the inferred requirements.

## F

- **failOnError**: An `ai_query` option that, when false, returns a per-row error instead of failing the whole query.
- **Fallback label**: A catch-all label such as `other` for inputs that fit none of the defined labels.
- **Feedback**: An MLflow assessment object holding a value, a name and a rationale or error.
- **feedback_value_type**: The `make_judge` setting that fixes the type of value a judge returns.
- **Few-shot examples**: Input and output pairs placed in the prompt to show the model exactly what to produce.

## G

- **Genie Conversation API**: The asynchronous REST API for starting Genie conversations, polling messages and reading query results.
- **Genie space**: A Databricks natural-language interface that answers analytical questions by generating SQL over tables.
- **GenieAgent**: The `databricks_langchain` class that wraps a Genie space as a worker in a coded multi-agent system.
- **Golden evaluation set**: Questions and expected answers hand-written by subject-matter experts.
- **Grounding instruction**: A prompt rule to answer only from the supplied context, paired with a defined fallback.
- **Guidelines judge**: A built-in pass/fail judge that applies one set of natural-language rules to every response.

## H

- **Hallucination (ungrounded response)**: A claim in a response that the retrieved context does not support.
- **Hard threshold**: A requirement that eliminates any candidate model that misses it.
- **HNSW**: The approximate nearest-neighbor graph algorithm Vector Search uses.
- **HTMLHeaderTextSplitter**: A LangChain splitter that splits HTML at header elements and stores their text as metadata.
- **Human approval (human-in-the-loop)**: Requiring a person to confirm a high-impact tool call before it runs.
- **Hybrid search**: Running keyword and vector retrieval together and merging the results.

## I

- **Idempotent tool**: An action tool that returns the recorded result instead of acting again when retried with the same request ID.
- **Indirect prompt injection**: An instruction planted in content the application reads, such as a web page or document.
- **Inference table**: An AI Gateway Delta table that logs request and response payloads for a serving endpoint.
- **Information Extraction**: An Agent Bricks agent that extracts the same schema of fields from every document into a table.
- **Ingestion-time masking**: Redacting sensitive entities once in the pipeline before chunks are written and indexed.
- **Injection classifier**: A model that screens text for attempts to override instructions, catching paraphrases and variants.
- **Input example**: A sample request saved with a model, from which MLflow can infer the signature.
- **Input guardrail**: A check that runs on a request before the model or external provider receives it.
- **Instruct model**: A model version tuned to follow instructions and hold a conversation.

## J

- **Jailbreak**: A prompt injection aimed at bypassing a model's safety rules.
- **json_object**: A response format that guarantees valid JSON but not which keys it contains.
- **json_schema**: A response format that enforces the keys, types and enums you declare.
- **Judge alignment**: Calling `judge.align()` to make an LLM judge follow expert feedback on labeled traces.

## K

- **Knowledge Assistant**: An Agent Bricks agent that answers questions over documents and cites its sources.

## L

- **L2 distance**: Euclidean distance between vectors, from which the Vector Search score is derived.
- **Label schema**: A definition of the question a reviewer answers, such as expected facts or a quality rating.
- **Labeling session**: A run that holds traces, label schemas and assigned reviewers for expert labeling in the Review App.
- **Lakebase**: Databricks' managed Postgres, used for low-latency agent session and memory storage.
- **LangChain**: A framework for composing prebuilt components such as retrievers, templates, models and parsers into linear chains.
- **LangGraph**: A framework for stateful agent graphs whose steps can loop and branch, with a checkpointer that persists state.
- **LCEL (LangChain Expression Language)**: LangChain's syntax for composing runnables with the `|` operator.
- **Least privilege**: Granting a tool or identity only the access its task needs.
- **License**: The terms under which a data source may be stored, retrieved, quoted or used for training.
- **License filter**: A Vector Search metadata filter that excludes chunks under restricted licenses.
- **Llama Guard**: An LLM fine-tuned to classify a prompt or response as safe or unsafe against categories in its prompt.
- **LlamaIndex**: A framework centred on data connectors, indexes and query engines for answering over ingested sources.
- **load_context**: The pyfunc method that runs once per model load to create clients and read packaged files.
- **logging_error_codes**: An inference table column explaining why a payload was not logged, such as size limits.
- **Long-term memory**: User facts and preferences kept across conversations, keyed by the authenticated user.
- **Lost in the middle**: The tendency of models to ignore facts placed in the middle of a long context.

## M

- **make_judge**: An MLflow function that creates a custom LLM judge from instructions and template variables.
- **Manual authentication**: Supplying an agent's own credentials in environment variables, which disables automatic passthrough.
- **MarkdownHeaderTextSplitter**: A LangChain splitter that splits Markdown at headings and stores the heading text as metadata.
- **Mask (PII behaviour)**: Replacing detected personal data and letting the request or response continue.
- **Metadata filter**: A condition on an index column applied inside a vector search so disallowed chunks are never returned.
- **MinHash**: A technique for finding near-duplicate documents so they can be removed before chunking.
- **Mixture-of-experts (MoE)**: A model architecture that routes each token through a few expert sub-networks.
- **MLflow Prompt Registry**: A Unity Catalog store for prompts with immutable versions and mutable aliases.
- **mlflow.genai.evaluate**: The MLflow 3 harness that runs or reads an app's outputs over evaluation records and applies scorers.
- **Model alias**: A movable pointer, such as `champion`, to one Unity Catalog model version.
- **Model card**: Published metadata about a model, such as task, languages, context length and licence, used to screen candidates.
- **Model Context Protocol (MCP)**: A standard way for agents to discover and call tools exposed by a server.
- **Model signature**: The schema of a model's inputs and outputs, required for Unity Catalog registration.
- **Model task**: The kind of work a model does, such as classification, extraction, summarization or translation, chosen from the output the business needs.
- **ModelConfig**: An MLflow class that supplies configuration values to a model logged from code.
- **Models from Code**: Logging an agent as a Python file ending in `set_model`, which MLflow re-executes at load time.
- **MRR (mean reciprocal rank)**: The average of 1 ÷ the rank of the first relevant result across questions.
- **Multi-label classification**: Assigning every applicable label from a fixed list, which can be zero, one or several.
- **Multiagent Supervisor (Supervisor Agent)**: An Agent Bricks agent that routes requests across subagents using their descriptions.

## N

- **NDCG@k**: A rank-aware metric that discounts each relevant result by position and accepts graded relevance labels.

## O

- **OCR (optical character recognition)**: Recognising text from page images, for example with Tesseract through `pytesseract`.
- **On-behalf-of-user authorization**: Agent access to resources using the requesting user's identity and permissions.
- **optimize_prompts**: An MLflow function that searches for better prompt text against scorers and registers the result as a new version.
- **Output guardrail**: A check that runs on a response after generation, not supported for streaming or embeddings.
- **Output parser**: The component that converts the model's reply into a string, list or typed object.
- **Over-refusal**: A guardrail or prompt rule blocking a safe, in-scope request.

## P

- **Parent-child (small-to-big) chunking**: Embedding small child chunks for matching and passing their larger parent sections to the LLM.
- **Pay-per-token**: Foundation Model APIs billing on shared Databricks-hosted endpoints.
- **Perplexity**: A measure of how well a model predicts held-out text, where lower is better.
- **PII (personally identifiable information)**: Data such as names, addresses or card numbers that identifies a person.
- **PII detection guardrail**: The AI Gateway filter that blocks or masks personal data in requests or responses.
- **Precision@k**: The share of the top k results that are relevant.
- **predict**: The pyfunc method that runs on every request.
- **predict_fn**: The function `mlflow.genai.evaluate` calls with each record's `inputs` as keyword arguments.
- **Problematic text**: Abuse, spam, superseded rules, misinformation or biased wording in a data source.
- **Prompt alias**: A movable name, such as `production`, that points at one prompt version and is used to promote or roll back.
- **Prompt augmentation**: Adding request-specific context, chosen from the input's key fields, terms and intent, to a prompt.
- **Prompt template**: The component that combines variable input, retrieved passages and instructions into the prompt.
- **Provenance**: The recorded origin of a chunk, such as source ID, author and license.
- **Provisioned throughput**: Reserved Model Serving capacity used for open-weights and fine-tuned models.
- **Provisioned throughput endpoint**: A Model Serving endpoint with dedicated capacity for base or fine-tuned models.
- **pyfunc**: MLflow's generic Python model interface, which any flavor can be loaded and served through.
- **PythonModel**: An MLflow base class for custom models with `load_context` and `predict` methods.

## Q

- **QPM / TPM**: Queries per minute and tokens per minute, the units of AI Gateway rate limits.
- **Query rewrite (condense) step**: An LLM call that turns chat history plus a follow-up into a standalone retrieval query.

## R

- **Rate limit**: An AI Gateway cap on queries or tokens per minute for an endpoint, user, service principal or group.
- **Re-ranking**: A second retrieval stage that rescores first-stage candidates against the query and reorders them.
- **READ VOLUME**: The Unity Catalog privilege to read files in a volume.
- **Reasoning model**: A model that generates a billed thinking phase before its visible answer.
- **Recall@k**: The share of all relevant chunks for a question that appear in the top k results.
- **Reciprocal Rank Fusion (RRF)**: A method that merges ranked lists by their rank positions rather than raw scores.
- **RecursiveCharacterTextSplitter**: A LangChain splitter that tries paragraph, line, sentence and word breaks to stay under a size.
- **Registry URI**: The MLflow setting that decides where models are registered; `databricks-uc` means Unity Catalog.
- **responseFormat**: The `ai_query` argument that constrains each row's output to a JSON schema.
- **ResponsesAgent**: The MLflow interface for wrapping agent code with `predict` and `predict_stream` methods.
- **Retrieval-augmented generation (RAG)**: Answering with an LLM that is given relevant passages retrieved from a knowledge base at query time.
- **Retriever**: The chain component that searches an index and returns the most relevant chunks with their metadata.
- **RETRIEVER span**: An MLflow trace span of type RETRIEVER whose output, a list of documents, retrieval judges read.
- **Review App**: A web chat interface where stakeholders give feedback and experts label expected facts.
- **Role separation**: Sending user text as its own user message rather than inside the system message.
- **ROUGE**: A recall-oriented n-gram overlap metric for summarisation against reference summaries.
- **Router**: A cheap classification step that sends simple requests to a chain and complex ones to an agent.
- **Row filter**: A Unity Catalog function that hides whole rows from users at query time.
- **RunnablePassthrough**: An LCEL step that forwards its input unchanged, for example the question in a RAG chain.

## S

- **Safety guardrail**: The AI Gateway filter that screens requests and responses for harmful content with a fixed taxonomy.
- **Scale to zero**: A serving setting that stops billing compute while an endpoint is idle, at the cost of a slow first response.
- **Scorer**: A judge or custom function that assesses traces in evaluation or monitoring.
- **ScorerSamplingConfig**: Per-scorer monitoring settings: `sample_rate` and `filter_string`.
- **Secret reference**: An environment variable value written `{{secrets/scope/key}}` that Model Serving resolves to a secret.
- **Semantic chunking**: Placing chunk boundaries where similarity between adjacent sentences drops sharply.
- **Sentinel string**: An exact output such as `INSUFFICIENT_CONTEXT` that code can detect reliably.
- **served_entity_id**: The column identifying which served model answered a request on a multi-entity endpoint.
- **Service principal**: A non-human identity used to run jobs with exactly the grants they need.
- **Short-term memory**: Conversation state keyed by session ID.
- **Single-label classification**: Assigning exactly one label from a fixed list to an input.
- **Smoke test**: A few representative requests sent to a newly deployed endpoint before production.
- **Storage-optimized endpoint**: A Vector Search endpoint type for very large, cost-sensitive indexes with higher latency.
- **Streamable HTTP**: The MCP transport used by servers hosted as web apps, including on Databricks Apps.
- **StrOutputParser**: An LCEL parser that extracts plain text from a chat model's message.
- **Structured outputs**: A request setting (`response_format`) that makes a chat endpoint return JSON constrained by a schema.
- **Synced table**: A Lakebase table that replicates a Unity Catalog Delta table for fast lookups.
- **Synthetic evaluation set**: Questions and expected facts generated from the corpus by an LLM, then reviewed.
- **System prompt**: The message sent with every request that carries standing instructions with the highest priority.
- **system.ai**: The Unity Catalog schema holding Databricks-provided foundation models with permissive licences.
- **system.billing.usage**: The billing system table that records DBUs by SKU, endpoint and custom tag.

## T

- **target_qps**: A standard endpoint setting that provisions more query throughput at extra cost.
- **Terms of use**: A website's rules, which may forbid scraping or reuse in AI systems.
- **Text layer**: The machine-readable text inside a digitally generated PDF, which scanned PDFs usually lack.
- **Throughput**: The amount of work, such as requests or records, completed per unit of time.
- **Time to first token (TTFT)**: The wait before the first token of a streamed reply appears.
- **to_predict_fn**: An MLflow helper that turns a serving endpoint or Databricks App URI into a predict function.
- **to_single_brace_format()**: A Prompt Registry method that returns a LangChain-ready copy of a double-brace template.
- **Tool-calling agent**: A model that decides at run time which tools to call, and in what order, for each request.
- **ToolCallCorrectness**: A built-in judge that checks the agent called the expected tools with the expected arguments.
- **ToolCallEfficiency**: A built-in judge that looks for redundant or wasteful tool calls without ground truth.
- **Total parameters**: All of a model's parameters, which must fit in GPU memory.
- **Triggered sync mode**: A Delta Sync index mode that picks up source changes only when a sync is run.

## U

- **UCFunctionToolkit**: The `databricks-langchain` class that exposes Unity Catalog functions as agent tools.
- **Unity Catalog function tool**: A UC SQL or Python function exposed to an agent, described to the model by its name and comments.
- **Unity Catalog HTTP connection**: A governed object that stores an external service's endpoint and credentials.
- **Unity Gateway**: The newer Databricks governance layer offered alongside the legacy endpoint-level AI Gateway settings.
- **Usage tracking**: An AI Gateway feature that logs token counts and requester per request to `system.serving.endpoint_usage`.
- **usage_context**: A map sent in the request body and copied into the usage table to attribute usage to users or tenants.
- **USE CONNECTION**: The Unity Catalog privilege to use an HTTP connection directly, meant for MCP authors and admins.
- **User authorization (Apps)**: A Databricks Apps mode that forwards each signed-in user's token to downstream resources.

## V

- **VACUUM**: A Delta command that removes data files of earlier table versions after the retention period.
- **Vector index**: A Unity Catalog object holding vectors and synced columns, searchable by similarity.
- **Vector Search endpoint**: Serving compute that hosts up to 50 Vector Search indexes.
- **vector_search()**: A SQL table-valued function that queries a Vector Search index from a SQL warehouse.

## W

- **Workload identity federation**: Exchanging a CI system's short-lived OIDC token for a Databricks token through a federation policy.
- **Writeback table**: A Unity Catalog table that stores the embeddings Databricks computed for a managed index.

[← Back to the study guide](README.md)
