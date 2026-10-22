# Domain 5: Governance (8%)

The smallest domain, with a single topic. It covers keeping personal data, malicious input, unlicensed content and problematic text from harming users or the business. Questions pair a governance requirement with a business constraint (an answer-rate target, a p95 latency budget, no extra model call, streaming that must stay on, one shared index, a paid product, a control that must hold even if the model obeys) and offer near-twin options. The deciding question is usually where the control sits: before or after the model, in the data or at the output, in the prompt or in permissions.

## 5.1 Guardrails, masking and data licensing

- **AI Gateway PII detection** on a Model Serving endpoint, configured per direction (requests, responses):
  - **Block** rejects the whole request or response. The user gets no answer.
  - **Mask** replaces the personal data and lets the call continue. The model never sees it and the user still gets an answer.
  - **None** turns detection off for that direction.
- **Mask meets a performance objective.** If Block rejects every question that mentions a colleague's name, the answer rate collapses; Mask keeps the privacy rule and the answers. Use Block only when policy demands outright rejection.
- **Direction matters:**

| Setting                   | Effect on a request with personal data                   | Fits when                                                       |
| ------------------------- | -------------------------------------------------------- | --------------------------------------------------------------- |
| Input PII: Mask           | Replaced, request continues                              | Data must not reach the model or provider; answers still needed |
| Input PII: Block          | Rejected                                                 | Policy demands rejection                                        |
| Output PII: Mask or Block | Model or provider already saw it; only the reply changes | The concern is what the reply shows                             |
| Safety guardrail          | Blocks harmful content; no Mask; ignores personal data   | Unsafe content, not privacy                                     |

- Only **input-side** masking keeps personal data from an external model provider.
- Databricks now documents these settings on its legacy AI Gateway governance page (Public Preview) while a newer gateway rolls out.
- **Guardrail limits:** output guardrails do not support streaming responses or embeddings models; input guardrails still run before generation. In a RAG chain or agent, the endpoint guardrail runs at generation, **after** retrieval and tool calls, and never sees citations shown straight from Vector Search.
- **`ai_mask(content, labels)`** replaces each entity of the listed types with `[MASKED]`, for example `ai_mask(body, array('person', 'email', 'phone'))`. No prompt or endpoint needed. DBR 15.4 LTS+, tuned for English.
  - Only listed labels are masked. Leaving one out leaves it visible; adding one the app needs (product names, error codes) destroys retrieval content.
  - Changing labels later means rewriting the masked table and re-syncing; changing only new rows leaves old rows unmasked.
- **Near-twin functions:** `ai_extract` returns entity values as fields; `ai_classify` returns one label for the text; `ai_query` runs a custom prompt. None of them mask.
- **`ai_mask` vs Unity Catalog column masks and row filters:** column masks are query-time and user-scoped, admins still see raw values, and they cannot find names inside free text. Row filters hide whole rows. When everyone must retrieve the text but nobody may see the entities inside it, redact the stored text with `ai_mask`.
- **Mask once at ingestion** for data in the knowledge base: write source text through `ai_mask` into the chunk table and let the Delta Sync index sync from it.
  - Paid once per document, not per request; no model call added to the request path.
  - Covers citations shown from Vector Search and streamed replies, which endpoint guardrails cannot clean.
  - Keep endpoint input masking for what users type.
- **Masking beats excluding records.** When most records contain some personal data, dropping them loses most of the knowledge and skews what remains. Synthetic data drifts; access restrictions or Block still leave the data stored and retrievable.
- **Direct prompt injection / jailbreak:** user input that tries to change the model's role or rules ("ignore your instructions", "role-play an assistant with no rules", "reveal your configuration").
  - Screen each user message with a **model-based injection classifier** before the model call. Phrase lists miss rewordings and reject ordinary text.
  - Keep roles separate: user text as its own user message, never concatenated into the system message.
  - Keep secrets out of prompts.
- **Where to screen:**

| Screening point                                        | Already spent when it blocks | Role                       |
| ------------------------------------------------------ | ---------------------------- | -------------------------- |
| User message at the start of the chain                 | Nothing beyond the check     | First line                 |
| Input guardrail on the generation endpoint in an agent | Retrieval and tool calls     | Useful but late for cost   |
| Output guardrail on the reply                          | The whole request            | Second layer               |
| Inference table review                                 | Everything                   | Monitoring, not prevention |

- On a plain chat endpoint, an endpoint input guardrail stops harmful requests before any generation tokens and works with streaming on.
- **Indirect prompt injection:** instructions planted in content the app reads (crawled pages, uploads, emails, fetched pages). Screen content at ingestion and at fetch time; prefer trusted sources.
- **Blast-radius controls** hold even when the model obeys an injection:
  - Least-privilege tool credentials (a read-only SQL tool's SP gets `SELECT` on just its tables).
  - On-behalf-of-user auth so UC row filters apply per user.
  - Derive sensitive parameters (an email recipient) in tool code, not from model arguments.
  - Restrict and confirm actions: domain allowlists, human approval for high-impact calls.
  - API keys in Databricks secrets read by tool code, never in a system prompt.
- **Data licensing:**

| Source                                     | Commercial GenAI use?                                  |
| ------------------------------------------ | ------------------------------------------------------ |
| No license                                 | No; all rights stay with the author                    |
| CC BY-NC                                   | No, without separate commercial permission             |
| CC BY                                      | Yes, with appropriate credit (title, author, license)  |
| Site terms forbidding scraping or AI reuse | Legal risk; use a licensed feed if the owner sells one |
| License allows retrieval but not training  | Use it only for retrieval and quoting                  |

- "Publicly visible" is not a license. Crediting, masking, or using content only for retrieval does not lift an NC restriction. A collection someone else assembled in breach of terms carries the same risk.
- **Provenance per chunk:** store source ID, title, author or partner and license as columns. They travel into Vector Search results for attribution, audit and removal.
- **One index, different rights:** add a license column, include it in the index, and pass a metadata filter that excludes restricted licenses on every query from the paid app. Filtering after retrieval still retrieves the chunks; a UC tag on the source table is not filterable.
- **The index schema is fixed at creation.** A new column needs a new index.
- **Removing content when a license ends:**

| Index type             | How to remove                                   |
| ---------------------- | ----------------------------------------------- |
| Delta Sync, triggered  | Delete source rows, then run a sync             |
| Delta Sync, continuous | Delete source rows; change data feed carries it |
| Direct Vector Access   | Delete vectors by primary key through the API   |

- A Delta `DELETE` is logical: earlier versions keep data files until `VACUUM` after the retention period.
- **Problematic source text** (abuse, spam, superseded rules, misinformation, biased wording): fix it in the data.
  - A minority of bad records → filter at ingestion, for example with an `ai_classify` step, and load only current documents by status metadata.
  - A pervasive problem → recommend an authoritative alternative with a named owner and review process (tax authority guidance over a community tips board). Replace problematic prompt examples too.
- **Traps:**
  - Output-side PII detection cannot stop data reaching an external provider.
  - Rewording the system prompt, few-shot refusals, lower temperature, PII masking, rate limits and logging do not prevent injection.
  - Any check that depends on the model's cooperation is not a control.
  - A query filter or prompt rule that excludes a licensor's content is not removal.
  - An output Safety guardrail alone keeps blocking replies because the abusive text is still retrieved; it also does not judge accuracy, bias or tone.

📖 Full lesson: [Databricks GenAI Engineer: Guardrails, PII Masking and Data Licensing](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-guardrails-pii-masking-data-licensing/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

[← Back to the study guide](../README.md)
