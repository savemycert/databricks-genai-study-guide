# Domain 1: Design Applications (14%)

This domain is the design step before any code: which model task produces the output the business needs, how a prompt or response schema forces that output into a parseable shape, which chain components and tools close the gap from input to output, and when an Agent Bricks agent does the job without code. Questions open with a Generative AI Engineer and a concrete requirement, and the wrong options are usually workable approaches that miss one stated constraint: exactness, volume, latency or access.

## 1.1 Prompt design and model task selection

- **Pick the task from the output, not the input.** Read the requirement for what must come back, and that names the task.

| Requirement asks for                                   | Model task                     | Output                          |
| ------------------------------------------------------ | ------------------------------ | ------------------------------- |
| One category from a fixed list (route a ticket)        | Single-label classification    | Exactly one label               |
| Every category that applies from a fixed list          | Multi-label classification     | Zero, one or several labels     |
| How the writer feels                                   | Sentiment analysis             | One attitude label or score     |
| Specific values that appear in the text (invoice no.)  | Entity or field extraction     | Named fields, copied verbatim   |
| A shorter version keeping the main points              | Summarization                  | Free text, same language        |
| The same content in another language                   | Translation                    | Free text, new language         |
| New text (a reply, a description)                      | Text generation                | Free text                       |
| Closeness in meaning, near-duplicates, semantic search | Embeddings + vector similarity | Vectors and similarity scores   |
| The same text with sensitive values hidden             | Entity masking                 | Original text with placeholders |

- **Add only tasks someone asked for.** A sentiment score or follow-up draft that no consumer needs is cost and risk with no benefit.
- **Single vs multi-label is decided per field.** A maintenance report has one machine (single-label) but can list several fault types (multi-label).
- **Open-ended values are extraction.** Classification can only return labels you supplied. Every medication named in a note, including ones not on any list, must be extracted.
- **Exact values need extraction fields.** A serial number or meter reading must not travel inside a summary, which restates content in new words.
- **Summarization changes length, translation changes language.** A short Spanish bulletin from long English notes needs both. Map every hop of a conversation to a language.
- **Embeddings for meaning at scale.** Pairwise chat prompts grow with the number of pairs (quadratic). Embedding each item once grows with the number of items.
- **Thousands of labels:** embed inputs and label descriptions, retrieve the top candidate labels by similarity, then let an LLM choose from that short list.
- **Decompose:** one task per required output, ordered by dependency. Fixed business rules (per-diem limit, eligibility) go in SQL or Python, because code is reproducible and auditable.
- **Classify first** when each document type has its own schema. One schema requiring every type's fields pushes the model to fill fields the document does not have.
- **AI Functions vs `ai_query`:**
  - Task-specific functions (`ai_classify`, `ai_extract`, `ai_mask`, `ai_similarity`, `ai_summarize`, `ai_translate`, `ai_analyze_sentiment`) need no prompt and no endpoint choice. They run on Databricks-managed models.
  - `ai_summarize` `max_words` defaults to 50; 0 means no limit.
  - `ai_query(endpoint, request, ...)` fits when you must name the endpoint (a fine-tuned or provisioned throughput endpoint), write your own prompt, or enforce a JSON shape. Arguments: `responseFormat` (schema), `modelParameters` (temperature, max tokens), `failOnError => false` (per-row error instead of failing the query).
- **Format prompts:**
  - Make the format explicit and measurable: structure, count, length, and what must not appear.
  - Tell a chat model to return only the object, with no lead-in, sign-off or Markdown fences.
  - Standing rules for a multi-turn chat go in the **system prompt**, which is sent with every request.
  - Reasoning plus a parseable answer: reasoning inside `<reasoning>` delimiters first, then the answer inside `<answer>` delimiters that code reads.
- **Few-shot examples** fix failing cases without training. Target the inputs that fail. Balance labels and input styles, because the model copies every pattern, including label distribution.
- **Structured outputs:**

| Approach               | Guarantees                 | Use when                                 |
| ---------------------- | -------------------------- | ---------------------------------------- |
| Prompt-only "use JSON" | Nothing                    | Prototypes, tolerant loaders             |
| `json_object`          | Valid JSON, any keys       | Keys vary per input                      |
| `json_schema`          | Your keys, types and enums | A fixed object that code parses or joins |

- **Schema design:** repeating data is an array of objects. Several values from a closed set is an array of strings with an `enum`. Fields that can be absent are `["string", "null"]`, with an instruction to return null. Size keywords such as `minItems` and `maxLength` are not guaranteed, so validate in code.
- **Label sets:** an `enum` of exact strings, a fallback label (such as `other`) routed to review, and definitions where two labels overlap.
- **Pipeline I/O from a business goal:**
  - Inputs are what is provided plus what is retrieved, scoped by a key (store ID) and by identity taken from the authenticated session, enforced in the retrieval filter.
  - Outputs are the consumer's shape plus what the business must verify: passage-level citations, a model-judged "not covered" field, or a structured handoff record for a human queue.
  - Nobody waiting, high volume: batch over a Delta table with `ai_query` or an AI Function. A user waiting: Model Serving endpoint, streamed for long answers. One app can use both with a shared output schema.
- **Traps:**
  - `temperature`, `max_tokens` and stop sequences are not format controls. Low temperature makes output repeatable, not correctly shaped, and `max_tokens` can cut JSON mid-object.
  - `ai_mask` hides entities inside the text; `ai_extract` pulls them out as values.
  - `ai_similarity` returns a number to sort and threshold; `ai_classify` returns a label.
  - A required string field makes the model invent a value when the document has none.
  - A similarity search always returns its nearest passages, so "no results" rarely happens. Ask the model whether the passages answer the question.
  - Streaming does not make generation faster; it shows the first words sooner.

📖 Full lesson: [Databricks GenAI: Prompt Design, Output Formats and Model Task Selection](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-prompt-design-model-task-selection/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 1.2 Chain components, tools and Agent Bricks

- **Trace the data** from what arrives to what the consumer needs. Each component closes one gap.

| Component       | Does                                                      | Needed when                                        |
| --------------- | --------------------------------------------------------- | -------------------------------------------------- |
| Retriever       | Searches a Vector Search index, returns chunks + metadata | Knowledge the model lacks, too large to send whole |
| Prompt template | Combines input, passages, history and instructions        | Almost always                                      |
| Chat model      | Generates the response                                    | Always                                             |
| Output parser   | Turns the reply into a string, list or typed object       | Always; type chosen from the consumer              |
| Memory          | Carries earlier turns                                     | Multi-turn conversations only                      |

- **RAG order:** retriever → prompt template → chat model → output parser. A retriever after the model means the model answers before seeing the passages.
- **Leave out what the input does not need.** If one transcript or contract always fits the context window, put it in the prompt directly. A retriever over its chunks can drop relevant parts.
- **Freshness is retrieval's job.** Fine-tuning bakes in a snapshot and changes style or task behaviour; retrieval supplies current facts.
- **Chat prompt template:** fixed instructions in the system message, variable input in the user message.
- **Shape, not truth:** a JSON schema makes the reply parseable but not correct. Build a "sources" list in code from the retrieved chunks' metadata; a model asked to write URLs can invent plausible ones.
- **Follow-ups:** a history-aware rewrite step turns chat history plus the latest message into a standalone query. Skip it on the first turn (no history), since it costs an extra LLM call.
- **Structured fields that decide access** (`tenant_id`, plan tier) go to the Vector Search query as a **metadata filter**, so disallowed chunks never reach the prompt. The filter column must be in the index's synced columns. `columns_to_sync` cannot change on an existing Delta Sync index, so rebuild it.
- **Chain, agent or router:**

| Design             | Fits when                                   | Trade-off                                                |
| ------------------ | ------------------------------------------- | -------------------------------------------------------- |
| Sequential chain   | Same steps every time                       | Predictable cost and latency; cannot adapt               |
| Tool-calling agent | Requests need different sources or actions  | Calls only what's needed; pays a planning loop each time |
| Router + agent     | Most traffic simple, a minority needs tools | A cheap classifier keeps simple requests off the agent   |

- **Tool definitions are what the model reads:** name, description and parameter schema.
  - Unity Catalog function tools: the function name, function `COMMENT` and parameter `COMMENT`s. Documentation on the underlying table does not reach the model.
  - Vector Search retriever tools: a distinct `tool_name` and `tool_description` each, or two retrievers look identical.
  - State parameter formats (ISO `YYYY-MM-DD`). Put today's date in the system prompt so "last Tuesday" can be resolved.
- **Tool type by data shape:**

| Need                                        | Tool type                                            |
| ------------------------------------------- | ---------------------------------------------------- |
| Exact current record by ID                  | Unity Catalog function querying the live table       |
| Guidance in unstructured text               | Vector Search retriever tool                         |
| Open-ended analytical questions over tables | Genie space as a tool                                |
| A state-changing action                     | Purpose-built action tool calling the owning service |

- **Ordering:** dependent calls wait for their inputs (email → `customer_id` → `order_id`). Gather knowledge (the cancellation policy) before any irreversible action. Independent calls run in parallel, so latency is roughly the slowest call.
- **Safe action tools:**
  - Enforce preconditions inside the tool. Prompt rules, descriptions and tool order influence the model but guarantee nothing.
  - Least privilege: narrow tools, not "run any SQL" or "call any API". UC SQL/Python functions suit reads and computation; state changes go through the owning service's API (for example via a UC HTTP connection).
  - Idempotency: a timeout does not mean the action failed. Key the tool on a request-unique ID and return the recorded result on retry.
- **Agent Bricks:**

| Agent                  | Use for                                                       | Output                           |
| ---------------------- | ------------------------------------------------------------- | -------------------------------- |
| Knowledge Assistant    | Conversational Q&A over documents with sources                | Answers with citations           |
| Information Extraction | The same fields from every document, loaded into a table      | JSON records per document        |
| Multiagent Supervisor  | One chat entry point routing across existing agents and tools | Answers assembled from subagents |

- **Knowledge Assistant:** sources are UC volume files, a UC table with a content column, or a Vector Search index; one assistant can hold several sources. File sources need a manual **Sync** (incremental); index sources follow the index. Wrong shape for exact live lookups.
- **Information Extraction:** field names, types and especially **descriptions** drive what is extracted. Fix a wrong date by describing the field precisely and saying what it is not. Runs ad hoc or as a scheduled pipeline; PDFs are parsed first (`ai_parse_document`).
- **Supervisor:** routes by each subagent's **description**. Subagents can be Genie spaces, Knowledge Assistant endpoints, serving endpoints and UC functions. It uses only subagents the end user can access, and nothing runs as the creator.
- **Product status:** Databricks docs now label Knowledge Assistant and Supervisor Agent as legacy, and Information Extraction has a newer version. The March 2026 guide still names all three, and use-case fit is what is tested.
- **Agent Bricks vs custom:** standard retrieve-and-answer, extract or route → Agent Bricks. Mandatory custom code per request, filters computed by internal logic, or a fixed output contract → a custom Agent Framework agent logged with MLflow.
- **Experts improve Agent Bricks** with example questions and guidelines (Knowledge Assistant) or field descriptions (Information Extraction). They need **CAN MANAGE** on the agent; query or data access is not enough.
- **Traps:**
  - Fixes applied after the tool choice (reranking, more results, bigger context, retrying) do not fix a wrong tool choice. Improve the descriptions.
  - Wrapping a fixed two-step workflow in an agent or supervisor adds latency for flexibility nobody uses.
  - Putting the tenant in the prompt, reranking or scrubbing answers reduces leaks but does not prevent them. Filter in the search.
  - Missing company conventions with good retrieval → examples and guidelines, not fine-tuning or a bigger embedding model.

📖 Full lesson: [Databricks GenAI: Chain Components, Agent Tools and Agent Bricks](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-chain-components-tools-agent-bricks/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

[← Back to the study guide](../README.md)
