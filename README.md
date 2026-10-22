# Databricks GenAI Study Guide: Databricks Certified Generative AI Engineer Associate

A free, open study guide for the **Databricks Certified Generative AI Engineer Associate (Databricks GenAI)** exam: revision notes for every domain, side-by-side comparisons of commonly confused services, a glossary, 20 worked sample questions, and the official syllabus as a checklist with a full free lesson for every topic.

Maintained by [SaveMyCert](https://www.savemycert.com/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide), where you can read every lesson free, [practice with explained questions](https://www.savemycert.com/practice/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide) and [take timed mock exams](https://www.savemycert.com/mocks/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide).

## Contents

- [Exam at a glance](#exam-at-a-glance)
- [Exam domains](#exam-domains)
- [What is in this repo](#what-is-in-this-repo)
- [Syllabus checklist](#syllabus-checklist)
  - [Domain 1: Design Applications](#domain-1-design-applications)
  - [Domain 2: Data Preparation](#domain-2-data-preparation)
  - [Domain 3: Application Development](#domain-3-application-development)
  - [Domain 4: Assembling and Deploying Applications](#domain-4-assembling-and-deploying-applications)
  - [Domain 5: Governance](#domain-5-governance)
  - [Domain 6: Evaluation and Monitoring](#domain-6-evaluation-and-monitoring)
- [How to study for Databricks GenAI](#how-to-study-for-databricks-genai)
- [Sample questions](sample-questions.md)
- [Free resources](#free-resources)

## Exam at a glance

| | |
|---|---|
| Exam code | Databricks GenAI |
| Level | Associate |
| Questions | 45 |
| Time limit | 90 min |
| Passing score | Pass / fail |
| Format | Multiple choice & multiple selection |
| Exam fee | $200 |
| Valid for | 2 years |

Exam details change. Always confirm them in the official [Databricks Certified Generative AI Engineer Associate exam page and guide](https://www.databricks.com/learn/certification/genai-engineer-associate) from Databricks.

## Exam domains

| # | Domain | Weight | Topics |
|---|---|---|---|
| 1 | [Design Applications](#domain-1-design-applications) | 14% | 2 |
| 2 | [Data Preparation](#domain-2-data-preparation) | 14% | 2 |
| 3 | [Application Development](#domain-3-application-development) | 30% | 4 |
| 4 | [Assembling and Deploying Applications](#domain-4-assembling-and-deploying-applications) | 22% | 4 |
| 5 | [Governance](#domain-5-governance) | 8% | 1 |
| 6 | [Evaluation and Monitoring](#domain-6-evaluation-and-monitoring) | 12% | 2 |

That is 6 domains and 15 topics. Spend your time in proportion to the weights: the heaviest domain decides more of your score than the lightest.

## What is in this repo

| File | What it gives you |
|---|---|
| [Domain 1 notes](notes/domain-1-design-applications.md) | Design Applications: condensed revision notes per topic |
| [Domain 2 notes](notes/domain-2-data-preparation.md) | Data Preparation: condensed revision notes per topic |
| [Domain 3 notes](notes/domain-3-application-development.md) | Application Development: condensed revision notes per topic |
| [Domain 4 notes](notes/domain-4-assembling-and-deploying-applications.md) | Assembling and Deploying Applications: condensed revision notes per topic |
| [Domain 5 notes](notes/domain-5-governance.md) | Governance: condensed revision notes per topic |
| [Domain 6 notes](notes/domain-6-evaluation-and-monitoring.md) | Evaluation and Monitoring: condensed revision notes per topic |
| [Commonly confused services](comparisons.md) | Side-by-side tables of the services questions set against each other |
| [Glossary](glossary.md) | Every in-scope term and service in one sentence |
| [Sample questions](sample-questions.md) | 20 worked questions with answers and reasoning |
| [Exam-day guide](exam-day-guide.md) | Booking, testing options, scoring, results and retakes |

## Syllabus checklist

Tick each topic off once you can explain it without notes. The "Must know" facts are the ones questions turn on. Each lesson link goes to the complete, free lesson.

### Domain 1: Design Applications

**Weight: 14%.** Designing LLM applications: prompts that produce a required format, model tasks for a business need, chain components, multi-stage tool use and when to use Agent Bricks. Official weighting 14%.

📝 Revision notes: [Domain 1: Design Applications](notes/domain-1-design-applications.md)

- [ ] **Prompt design and model task selection**
  <br>Design a prompt that elicits a specifically formatted response; Select model tasks to accomplish a given business requirement; Translate business use case goals into a description of the desired inputs and outputs for the AI pipeline.
  - 📖 Lesson: [Databricks GenAI: Prompt Design, Output Formats and Model Task Selection](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-prompt-design-model-task-selection/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Pick the task from the output the business needs: a known list means classification, values in the text mean extraction, shorter text means summarization, another language means translation, closeness in meaning means embeddings.
  - Must know: Decide single- or multi-label per output field from how many values can truly apply; open-ended names (including new ones) must be extracted, not classified.
- [ ] **Chain components, tools and Agent Bricks**
  <br>Select chain components for a desired model input and output; Define and order tools that gather knowledge or take actions for multi-stage reasoning; Determine how and when to use Agent Bricks (Knowledge Assistant, Multiagent Supervisor, Information Extraction) to solve problems.
  - 📖 Lesson: [Databricks GenAI: Chain Components, Agent Tools and Agent Bricks](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-chain-components-tools-agent-bricks/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Trace the data to pick chain components: retriever → prompt template → chat model → output parser, with the parser chosen from the required output (string, list, typed object).
  - Must know: Skip the retriever when the whole input fits in the context window, and use retrieval, not fine-tuning, to keep answers current with changing documents.

### Domain 2: Data Preparation

**Weight: 14%.** Preparing knowledge for RAG: choosing source documents, extracting and filtering content, chunking strategies, writing chunks to Delta tables in Unity Catalog, evaluating retrieval and re-ranking. Official weighting 14%.

📝 Revision notes: [Domain 2: Data Preparation](notes/domain-2-data-preparation.md)

- [ ] **Document extraction, filtering and chunking into Delta tables**
  <br>Apply a chunking strategy for a given document structure and model constraints; Filter extraneous content in source documents that degrades quality of a RAG application; Choose the appropriate Python package to extract document content from provided source data and format; Define operations and sequence to write given chunked text into Delta Lake tables in Unity Catalog; Identify needed source documents that provide necessary knowledge and quality for a given RAG application.
  - 📖 Lesson: [Databricks GenAI Engineer: Document Extraction, Filtering and Chunking into Delta Tables](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-document-chunking-delta-tables/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Index only authoritative, current sources meant for everyone who can query the index; keep sensitive records out at ingestion.
  - Must know: If the answer is not in the chunks table, add the source that holds it; retrieval tuning cannot create missing knowledge.
- [ ] **Retrieval evaluation, advanced chunking and re-ranking**
  <br>Use tools and metrics to evaluate retrieval performance; Design retrieval systems using advanced chunking strategies; Explain the role of re-ranking in the information retrieval process.
  - 📖 Lesson: [Databricks GenAI Engineer: Retrieval Evaluation, Advanced Chunking and Re-ranking](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-retrieval-evaluation-reranking/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Precision@k divides relevant-in-top-k by k; recall@k divides it by all labeled relevant chunks. Measure both at the k the chain actually passes to the LLM.
  - Must know: As k grows, recall can only rise and precision usually falls; that is expected, not a regression.

### Domain 3: Application Development

**Weight: 30%.** Building the application: frameworks such as LangChain, prompt augmentation and engineering, quality and safety assessment, guardrails, choosing LLMs and embedding models, and agentic systems with MLflow, Agent Framework and Genie. Official weighting 30%.

📝 Revision notes: [Domain 3: Application Development](notes/domain-3-application-development.md)

- [ ] **Frameworks, prompt augmentation and prompt engineering**
  <br>Select Langchain/similar tools for use in a Generative AI application; Augment a prompt with additional context from a user's input based on key fields, terms, and intents; Create a prompt that adjusts an LLM's response from a baseline to a desired output.
  - 📖 Lesson: [Databricks GenAI Engineer: LangChain, Prompt Augmentation and Prompt Engineering](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-langchain-prompt-engineering/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: LangChain chains fixed, linear steps; LangGraph handles loops, branching and pause-and-resume state; LlamaIndex centres on ingestion and query engines; DSPy compiles prompts against a metric; a single fixed call needs no framework.
  - Must know: ChatDatabricks is for chat endpoints, DatabricksEmbeddings for embedding endpoints, DatabricksVectorSearch for indexes and UCFunctionToolkit for Unity Catalog functions as tools.
- [ ] **Response quality, safety and guardrails**
  <br>Qualitatively assess responses to identify common issues such as quality and safety; Select chunking strategy based on model & retrieval evaluation; Implement LLM guardrails to prevent negative outcomes.
  - 📖 Lesson: [Databricks GenAI Engineer: Response Quality, Safety and LLM Guardrails](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-response-quality-safety-guardrails/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Assess responses by reading them beside their retrieved context; latency, length and similarity scores cannot reveal hallucinations or unsafe content.
  - Must know: If the right fact is in the retriever span but the answer is wrong, it is a generation failure; if the fact never reached the prompt, fix retrieval.
- [ ] **Choosing LLMs and embedding models**
  <br>Select the best LLM based on the attributes of the application to be developed; Select an embedding model context length based on source documents, expected queries, and optimization strategy; Select a model from a model hub or marketplace for a task based on model metadata/model cards; Select the best model for a given task based on common metrics generated in experiments.
  - 📖 Lesson: [Databricks GenAI Engineer: Choosing LLMs and Embedding Models](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-choosing-llms-embedding-models/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Filter on hard requirements first, then choose the smallest, cheapest model that meets the measured quality bar.
  - Must know: The LLM's context window must hold the full prompt plus the maximum reply; buy no more window than that.
- [ ] **Agentic development with MLflow, Agent Framework and Genie**
  <br>Utilize MLflow and Agent Framework for developing agentic systems; Compare the evaluation and monitoring phases of the Gen AI application life cycle; Enable multi-agent systems to leverage Genie Spaces or conversational API to retrieve data.
  - 📖 Lesson: [Databricks GenAI Engineer: Building Agents with MLflow, Agent Framework and Genie](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-agent-framework-mlflow-genie/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: ResponsesAgent wraps any framework's agent with an inferred signature and automatic tracing; streaming needs predict_stream yielding delta events and a final output_item.done event.
  - Must know: Per-request extras go in custom_inputs; model_config is fixed when the logged version loads.

### Domain 4: Assembling and Deploying Applications

**Weight: 22%.** Assembling and deploying: pyfunc chains, RAG application elements, registering to Unity Catalog, Mosaic AI Vector Search, Model Serving and Foundation Model APIs, ai_query() batch inference, memory stores, CI/CD, prompt lifecycle, MCP servers and user interfaces. Official weighting 22%.

📝 Revision notes: [Domain 4: Assembling and Deploying Applications](notes/domain-4-assembling-and-deploying-applications.md)

- [ ] **Coding chains with pyfunc and registering to Unity Catalog**
  <br>Code a chain using a pyfunc model with pre- and post-processing; Code a simple chain according to requirements; Choose the basic elements needed to create a RAG application: model flavor, embedding model, retriever, dependencies, input examples, model signature; Register the model to Unity Catalog using MLflow.
  - 📖 Lesson: [Databricks GenAI Engineer: Coding pyfunc Chains and Registering Them to Unity Catalog](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-pyfunc-chains-unity-catalog-registration/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Pure LangChain chain that others will compose: LangChain flavor. Custom pre/post-processing: pyfunc PythonModel. New agent for Agent Framework: ResponsesAgent.
  - Must know: load_context runs once per load (clients, tokenizers, artifact files); predict runs per request; __init__ state is pickled when you log an instance.
- [ ] **Mosaic AI Vector Search**
  <br>Create and query a Vector Search index; Explain the key concepts and components of Mosaic AI Vector Search; Configure vector search for a particular solution based on number of embeddings, update frequency, latency, and cost requirements.
  - 📖 Lesson: [Databricks GenAI Engineer: Mosaic AI Vector Search Indexes, Endpoints, Querying and Sizing](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-mosaic-ai-vector-search/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: An endpoint is serving compute that hosts up to 50 indexes; each index is a Unity Catalog object governed with grants.
  - Must know: Delta Sync follows a Delta table (Databricks-computed or self-managed embeddings); Direct Vector Access has no table and changes only through upsert and delete calls.
- [ ] **Model serving, access control, batch inference and memory**
  <br>Control access to resources from model serving endpoints; Identify how to serve an LLM application that leverages Foundation Model APIs; Identify batch inference workloads and apply ai_query() appropriately; Configure a persistent datastore to store and retrieve intermediate memory or structured information.
  - 📖 Lesson: [Databricks GenAI Engineer: Model Serving Access, Foundation Model APIs, ai_query() and Agent Memory](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-model-serving-ai-query-agent-memory/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Endpoint permissions are CAN VIEW < CAN QUERY < CAN MANAGE; callers need CAN QUERY, version rollbacks and traffic changes need CAN MANAGE, and UC privileges on the model do not open the endpoint.
  - Must know: Secrets reach a served model as environment variables written exactly {{secrets/scope/key}}; the endpoint creator needs READ on the scope, callers do not.
- [ ] **CI/CD, prompt lifecycle, MCP servers and user interfaces**
  <br>Apply CI/CD best practices such as updating a Vector Search index, promoting prompts across environments, and testing individual components of an agent; Integrate managed, external, and custom MCP servers based on a given application requirements; Apply prompt version control and manage prompt lifecycle; Develop an appropriate interactive user facing interface for an agent usage scenario (Apps, Slack, Teams, etc.).
  - 📖 Lesson: [Databricks GenAI Engineer: CI/CD for Agents, Prompt Lifecycle, MCP Servers and Agent Interfaces](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-cicd-prompt-lifecycle-mcp-servers/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Deploy code, not models, when production data may only be read in production: a bundle with dev, staging and prod targets builds each environment's index and endpoints there.
  - Must know: CI should run as a service principal using a federation policy that trusts the CI system's OIDC token, so no long-lived secret is stored.

### Domain 5: Governance

**Weight: 8%.** Governance for GenAI: masking and guardrails against malicious input, licensing of data sources, and mitigating problematic text in source data. Official weighting 8%.

📝 Revision notes: [Domain 5: Governance](notes/domain-5-governance.md)

- [ ] **Guardrails, masking and data licensing**
  <br>Use masking techniques as guard rails to meet a performance objective; Select guardrail techniques to protect against malicious user inputs to a Gen AI application; Use legal/licensing requirements for data sources to avoid legal risk; Recommend an alternative for problematic text mitigation in a data source feeding a GenAI application.
  - 📖 Lesson: [Databricks GenAI Engineer: Guardrails, PII Masking and Data Licensing](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-guardrails-pii-masking-data-licensing/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: PII detection Mask replaces personal data and lets the request continue; Block rejects it. Use Mask when questions must still be answered.
  - Must know: Only input-side masking keeps personal data from reaching a model or external provider; the Safety guardrail has no Mask behaviour.

### Domain 6: Evaluation and Monitoring

**Weight: 12%.** Evaluating and monitoring: quantitative model selection, agent evaluation with MLflow scoring, tracing and custom scorers, judges and ground truth, SME feedback, inference tables, Agent Monitoring, AI Gateway and cost control. Official weighting 12%.

📝 Revision notes: [Domain 6: Evaluation and Monitoring](notes/domain-6-evaluation-and-monitoring.md)

- [ ] **Evaluating LLMs and agents**
  <br>Select an LLM choice (size and architecture) based on a set of quantitative evaluation metrics; Evaluate agent performance using MLflow scoring and tracing; Identify evaluation judges that require ground truth; Use Databricks custom Scorers for evaluating agents and LLMs; Incorporate SME feedback to improve agent performance.
  - 📖 Lesson: [Databricks GenAI Engineer: Evaluating LLMs and Agents with MLflow Scorers and Judges](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-evaluating-llms-agents-mlflow-scorers/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Compare candidate LLMs on one shared, representative evaluation set with identical scorers, plus latency and cost; eliminate on every hard threshold first, then optimise the stated objective.
  - Must know: MoE generation speed tracks active parameters, but GPU memory must hold the total parameters.
- [ ] **Monitoring, inference logging and cost control**
  <br>Select key metrics to monitor for a specific LLM deployment scenario; Use inference logging to assess deployed RAG application performance; Use Databricks features to control LLM costs; Use inference tables and Agent Monitoring to track a live LLM endpoint; Use AI Gateway (Inference Tables, Usage Tables, and rate limiting) to track an LLM or agent deployed via Agent Framework.
  - 📖 Lesson: [Databricks GenAI Engineer: Monitoring, Inference Tables and LLM Cost Control](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-monitoring-inference-tables-cost-control/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
  - Must know: Match the metric to the risk: time to first token for streaming chat, throughput for deadline-bound batch work, tokens per request for cost, a judge score for quality.
  - Must know: Inference tables hold request and response payloads; rows arrive in the background within about an hour, oversized payloads are null with logging_error_codes, and error responses such as 429 may not be logged.

## How to study for Databricks GenAI

1. **Read the lesson for each topic** in the checklist above, starting with the heaviest domain. Every lesson is free on the [Databricks GenAI revision notes](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide).
2. **Practice straight after reading.** Answer [Databricks GenAI practice questions](https://www.savemycert.com/practice/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide) on the topic you just read. Each option comes with an explanation of why it is right or wrong.
3. **Review what you got wrong**, re-read that lesson section, and tick the topic off only when you get its questions right.
4. **Take a full-length [Databricks GenAI mock exam](https://www.savemycert.com/mocks/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)** under the real time limit. Aim to pass mocks comfortably before you book.
5. **On the last day**, skim the [Databricks GenAI cheat sheet](https://www.savemycert.com/cheat-sheet/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide) instead of starting anything new.

## Sample questions

[sample-questions.md](sample-questions.md) has 20 worked Databricks GenAI questions with the answer, why each option is right or wrong, and the reasoning steps.

## Free resources

- [Databricks Certified Generative AI Engineer Associate exam page and guide](https://www.databricks.com/learn/certification/genai-engineer-associate): the official source (Databricks)
- [Databricks GenAI certification overview](https://www.savemycert.com/certifications/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)
- [Databricks GenAI revision notes](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide): every lesson, free to read
- [Databricks GenAI practice questions](https://www.savemycert.com/practice/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide): with an explanation on every option
- [Databricks GenAI mock exams](https://www.savemycert.com/mocks/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide): full-length and timed
- [Databricks GenAI cheat sheet](https://www.savemycert.com/cheat-sheet/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide): the key facts on one page
- [All certification study guides](https://github.com/savemycert/certification-study-guides)

## Contributing

Spotted an error or an out-of-date fact? [Open an issue](../../issues) with the topic and a link to the official source. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License and disclaimer

This guide is licensed under [CC BY 4.0](LICENSE). You can reuse and adapt it, including commercially, as long as you credit **SaveMyCert** with a link to https://www.savemycert.com/.

This is an independent study resource. It is not affiliated with or endorsed by Databricks. Databricks Certified Generative AI Engineer Associate and Databricks GenAI are trademarks of their respective owner. Exam domains and weights are taken from the official exam guide linked above.
