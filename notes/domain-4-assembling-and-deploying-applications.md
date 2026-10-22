# Domain 4: Assembling and Deploying Applications (22%)

This domain turns a working prototype into a governed, deployed application: packaging the chain as an MLflow model and registering it to Unity Catalog, building and querying Mosaic AI Vector Search indexes, serving the model with the right permissions, credentials and Foundation Model API mode, running batch inference with `ai_query()`, storing memory, and running the whole thing through CI/CD, prompt lifecycle, MCP servers and a user-facing interface. Many items show short Python, SQL or a URI and ask which argument, privilege or step is right. Stated constraints such as least privilege, no redeploy, zero downtime or per-user data access decide between options that are all real features.

## 4.1 Coding chains with pyfunc and registering to Unity Catalog

- **Pick the flavor:**

| Flavor / interface             | Use when                                                          | Loads back as                     |
| ------------------------------ | ----------------------------------------------------------------- | --------------------------------- |
| `mlflow.langchain.log_model`   | Pure LCEL chain, no custom pre/post-processing; others compose it | A LangChain runnable (and pyfunc) |
| `mlflow.pyfunc.PythonModel`    | Your own Python around the LLM call: masking, validation, formats | pyfunc; you write the signature   |
| `mlflow.pyfunc.ResponsesAgent` | A new agent for Agent Framework (tools, multi-turn, streaming)    | pyfunc; signature inferred        |

- **Where code runs in a pyfunc model:**

| Method         | Runs                                                    | Belongs here                                        |
| -------------- | ------------------------------------------------------- | --------------------------------------------------- |
| `__init__`     | At instance creation; pickled if you log the instance   | Plain, picklable settings only                      |
| `load_context` | Once per load (each replica, batch job), before traffic | Clients, tokenizers, `context.artifacts` files      |
| `predict`      | Every request                                           | Masking, rewriting, retrieval, LLM call, formatting |

- **Pre- and post-processing in `predict`** apply to every caller: web app, Slack bot, REST and SQL `ai_query()` users.
  - Masking must happen **before** the endpoint call.
  - Post-processing: strip fences, validate JSON, reshape into fixed fields, remove internal chunk IDs, add a fallback.
  - A prompt instruction raises compliance; a gateway guardrail screens but cannot reshape; a signature describes shape but transforms nothing.
- **LCEL:** `prompt | llm | StrOutputParser()`. A RAG chain needs a branch per template variable: `{"context": retriever | format_docs, "question": RunnablePassthrough()}`. `ChatDatabricks` generates; `DatabricksEmbeddings` only produces vectors.
- **Models from code:** `chain.py` ends with `mlflow.models.set_model(chain)`; pass the path as `lc_model=` or `python_model=`.
  - The file is re-run in a fresh process, so it must build everything. A notebook-defined retriever gives `NameError`.
  - `ModelConfig(development_config=...)` is used while iterating; `log_model(model_config=...)` supplies values for the logged model. Log once per environment with its own config.
- **Packaging:**

| Missing                  | Argument                 | Behaviour                                       |
| ------------------------ | ------------------------ | ----------------------------------------------- |
| A package from an index  | `extra_pip_requirements` | Appends to the inferred list                    |
| A full exact environment | `pip_requirements`       | Replaces the inferred list                      |
| Conda spec               | `conda_env`              | Whole environment as YAML                       |
| Your own module          | `code_paths`             | Copied in and put on `sys.path`                 |
| A data file              | `artifacts`              | Copied in; path via `context.artifacts["name"]` |

- `conda_env`, `pip_requirements` and `extra_pip_requirements` are mutually exclusive. Inference can miss lazy imports. `/Workspace/...` paths are not readable from a serving container.
- **Signature:** Unity Catalog **requires** one. Logging with `input_example` infers it and stores the example. `ResponsesAgent` signatures are inferred.
- **RAG elements:** model flavor, embedding model, retriever, dependencies, input example, signature. Embed queries with the **same model** that built the index. Managed-embedding Delta Sync needs only the index name; self-managed or Direct Access also needs `text_column` and `embedding`.
- **Registering:** `mlflow.set_registry_uri("databricks-uc")` (default in MLflow 3); `"databricks"` is the legacy workspace registry. Name is `catalog.schema.model`. Each log with `registered_model_name` creates a new version.
- **Aliases, not stages.** UC has no Staging/Production stages. `set_registered_model_alias(name, "champion", version=6)` and load `models:/ml.support.support_rag@champion`. Registering a new version never moves an alias.
- **Model privileges** (all need `USE CATALOG` and `USE SCHEMA`):

| Action                             | Privilege                                           |
| ---------------------------------- | --------------------------------------------------- |
| Create a registered model          | `CREATE MODEL` (or `CREATE FUNCTION`) on the schema |
| Add a version to an existing model | Ownership or `CREATE MODEL VERSION` on it           |
| Load the model                     | `EXECUTE` on the registered model                   |

- **Validate before serving:** `mlflow.models.predict(model_uri, input_data=..., env_manager="uv")` (virtualenv is default). `pip_requirements_override` tests a fix for one run; `update_model_requirements` persists it.
- **Traps:**
  - `env_manager="local"` and `load_model(...).predict()` in the notebook hide missing dependencies.
  - Table privileges (`SELECT`, `MODIFY`) do not apply to models.
  - A pyfunc wrapper around a LangChain chain loads back as pyfunc, not as a runnable.
  - Tags cannot be used to load a model.

📖 Full lesson: [Databricks GenAI Engineer: Coding pyfunc Chains and Registering Them to Unity Catalog](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-pyfunc-chains-unity-catalog-registration/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 4.2 Mosaic AI Vector Search

- **Two resources:** an **endpoint** (serving compute; standard or storage-optimized; hosts up to **50 indexes**; bills while hosting indexes, even idle) and an **index** (a UC object `catalog.schema.index` on exactly one endpoint, governed by grants). Recent docs also call the product Databricks AI Search.
- **Index types:**

| Index type                      | Source rows                      | Who embeds | Changes via                 | Query with                 |
| ------------------------------- | -------------------------------- | ---------- | --------------------------- | -------------------------- |
| Delta Sync, Databricks-computed | UC Delta table, text column      | Databricks | Sync (triggered/continuous) | `query_text`               |
| Delta Sync, self-managed        | Delta table with a vector column | You        | Sync                        | `query_vector`, same model |
| Direct Vector Access            | None; a schema you define        | You        | `upsert` and `delete` calls | `query_vector`             |

- **`create_delta_sync_index` argument pairs:** Databricks-computed = `embedding_source_column` + `embedding_model_endpoint_name`. Self-managed = `embedding_vector_column` + `embedding_dimension`. They never mix.
  - `pipeline_type`: `"TRIGGERED"` or `"CONTINUOUS"`.
  - `columns_to_sync` is fixed at creation and not supported on storage-optimized endpoints.
  - "Save computed embeddings" writes vectors to `<index>_writeback_table` for reuse.
  - Storage-optimized needs an embedding dimension divisible by 16.
- **Sync modes:**

| Mode                 | Updates when                    | Cost                                       |
| -------------------- | ------------------------------- | ------------------------------------------ |
| Triggered            | Only on `sync()` or Sync now    | Cheapest                                   |
| Continuous           | Within seconds of table changes | Always-on streaming cluster; standard only |
| Direct Vector Access | On your `upsert`/`delete`       | No pipeline                                |

- Triggered indexes do not watch the table. Direct Access has no table link, so app-side deletes need an index `delete` call.
- Schema or embedding-column change → build a new index, switch the app once synced, then delete the old one.
- **`similarity_search`:** `query_text` or `query_vector`, `columns` (what each row carries), `num_results`, `filters`, `query_type`.
  - `"ANN"` (default), `"HYBRID"` (keyword + vector; roughly twice the resources; max 200 results; a self-managed index needs both `query_text` and `query_vector`), `"FULL_TEXT"` (Beta, keyword only).
  - SDK paginates ANN up to 10,000 rows.
- **Filter syntax by endpoint:** standard takes a dictionary (`{"revision >=": 3}`); storage-optimized takes a SQL-like string (`"revision >= 3"`). Storage-optimized overfetches then filters, so very selective filters can return nothing.
- **SQL `vector_search()`** (Public Preview): serverless or pro SQL warehouse, named arguments, `ANN` or `HYBRID`, max 100 results, no paging, no Direct Access indexes. Not the same as `ai_query()`.
- **Scoring:** HNSW with L2 distance. Score = 1 / (1 + d²). Normalize vectors for cosine-equivalent ranking. For unit vectors d² = 2 − 2·cos, so cosine 0.7 → score 0.625.
- **Endpoint type** (Databricks guidance at the time of writing; figures change):

|                 | Standard                                            | Storage-optimized          |
| --------------- | --------------------------------------------------- | -------------------------- |
| Capacity        | About 320M vectors at 768 dims (about 160M at 1536) | More than a billion        |
| Latency         | Tens of milliseconds                                | A few hundred milliseconds |
| Throughput      | Higher; scale with `target_qps`                     | Modest                     |
| Cost per vector | Higher                                              | Much lower                 |
| Sync modes      | Triggered and continuous                            | Triggered only             |

- HTTP 429 at peak on a standard endpoint → raise `target_qps`. Higher dimensions shrink standard capacity.
- **Cost:** consolidate idle indexes onto fewer endpoints (they do not scale to zero); triggered sync; storage-optimized for large latency-tolerant indexes.
- **Latency:** service principal OAuth instead of PATs; `num_results` of about 10-100; ANN unless keyword matching is needed; keep the embedding endpoint from scaling to zero; a separate query embedding endpoint (`model_endpoint_name_for_query`) serving the **same model**.
- **Query privileges:** `USE CATALOG`, `USE SCHEMA` and `SELECT` **on the index**.
- **Traps:**
  - `SELECT` on the source table does not grant the index.
  - Endpoint `CAN USE` lets you create indexes; it does not authorize reading index data.
  - Schema-level `SELECT` works but fails least privilege.
  - Deleting an index first to reuse its name causes an outage.

📖 Full lesson: [Databricks GenAI Engineer: Mosaic AI Vector Search Indexes, Endpoints, Querying and Sizing](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-mosaic-ai-vector-search/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 4.3 Model serving, access control, batch inference and memory

- **Endpoint permissions** (each includes the one below):

| Level        | Allows                                                                       | Give to                                   |
| ------------ | ---------------------------------------------------------------------------- | ----------------------------------------- |
| `CAN VIEW`   | See and list the endpoint                                                    | People checking it exists                 |
| `CAN QUERY`  | Plus sending requests                                                        | Callers, app SPs, `ai_query()` identities |
| `CAN MANAGE` | Plus config (version, traffic split, scaling, env vars), delete, permissions | Owners and on-call operators              |

- UC privileges on the registered model do not open the endpoint. Rollbacks and traffic changes need `CAN MANAGE`.
- **Secrets:** endpoint environment variables written exactly `{{secrets/<scope>/<key>}}`. The identity that creates or updates the endpoint needs `READ` on the scope; callers need only `CAN QUERY`. Read the value with `os.environ[...]`, usually in `load_context`.
- **Agent authentication methods:** automatic passthrough (declared resources), on-behalf-of-user (caller's identity), **manual** (credentials you supply).
  - Manual fits unsupported resources, such as loading Prompt Registry prompts at runtime.
  - Use service principal OAuth (`DATABRICKS_HOST`, `DATABRICKS_CLIENT_ID`, `DATABRICKS_CLIENT_SECRET` as secret references). For the Prompt Registry, the SP needs `CREATE FUNCTION`, `EXECUTE` and `MANAGE` on the prompts' schema.
  - **Trap:** overriding these variables turns off passthrough for **every** resource, so the SP needs access to the index and LLM endpoint too.
- **AI Gateway rate limits** (QPM or TPM): endpoint limit, default per-user limit, custom limit per user or service principal, group limits. An app SP calling for hundreds of people is one caller and needs its own limit. TPM is for foundation or external model endpoints; custom model and agent endpoints take QPM only.
- **Serving modes:**

|         | Pay-per-token                         | Provisioned throughput                            | External model endpoint                |
| ------- | ------------------------------------- | ------------------------------------------------- | -------------------------------------- |
| Setup   | None; ready-made                      | You create and size it                            | Provider key in a Databricks secret    |
| Billing | Per token (Databricks)                | Per capacity unit (Databricks)                    | By the provider under your contract    |
| Models  | A fixed set                           | Base models plus fine-tuned UC models             | Third-party (OpenAI, Anthropic, ...)   |
| Fits    | Prototyping, low or irregular traffic | Production throughput, predictable latency, HIPAA | One governed entry point to a provider |

- A fine-tuned UC model cannot use the base model's pay-per-token endpoint. A priority pay-per-token tier exists for latency-sensitive production traffic.
- **OpenAI client:** `base_url=f"{host}/serving-endpoints"`, a Databricks token, and `model=` the serving endpoint name. `/api/2.0/serving-endpoints` is the management API, not the query path.
- **Serving the application:** register the chain in UC and deploy that version. `agents.deploy("main.support.agent", 3)` also provisions resource credentials, the Review App and tracing.
- **`ai_query()` for batch:** inputs in a table, large volume, nobody waiting. Real-time chat or checkout checks need an endpoint.

| Argument          | Purpose                                                          |
| ----------------- | ---------------------------------------------------------------- |
| `responseFormat`  | Structure a chat model's output (DDL type string or JSON schema) |
| `returnType`      | SQL type of the result, mainly for custom model endpoints        |
| `modelParameters` | `max_tokens`, `temperature`                                      |
| `failOnError`     | `false` returns a struct with `errorMessage` per row             |

- Submit the full dataset in one query. Not on Classic SQL warehouses: use serverless or pro SQL, or DBR 15.4 LTS+. The runner needs `CAN QUERY` plus `SELECT` on the input.
- **Memory:** serving replicas are stateless across requests.

| Requirement                                              | Where state lives                                 |
| -------------------------------------------------------- | ------------------------------------------------- |
| Short chats, never resumed                               | Client resends the `messages` list                |
| Resumable sessions, many users, millisecond per-turn I/O | **Lakebase** (managed Postgres), keyed by session |
| Fast lookups of data whose source is a Delta table       | Lakebase synced table                             |
| Analytics over conversations                             | Delta tables (inference tables)                   |

- **Long-term memory** is scoped by the authenticated user ID from the request context, never a name the LLM extracts. Grant the deployed agent's service principal read and write on the store.
- **Traps:**
  - A secret path without the double braces is a plain-text string.
  - Libraries installed on a cluster do not enable `ai_query()`; the runtime version does.
  - State in a Python attribute in `predict` lives in one replica only.

📖 Full lesson: [Databricks GenAI Engineer: Model Serving Access, Foundation Model APIs, ai_query() and Agent Memory](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-model-serving-ai-query-agent-memory/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 4.4 CI/CD, prompt lifecycle, MCP servers and user interfaces

- **Bundles** (Declarative Automation Bundles, formerly Databricks Asset Bundles) describe code, jobs and endpoints with a target per environment (dev, staging, prod).
- **Deploy code vs deploy models:**

| Pattern       | What moves                                                   | Fits                                                    |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| Deploy code   | Source; each target builds its own index, model and endpoint | Usual choice; required when prod data stays in prod     |
| Deploy models | A dev-built model version promoted to prod                   | Only when dev may build from production-equivalent data |

- **CI auth:** run as a service principal with **workload identity federation**: a federation policy trusts the CI system's OIDC issuer, and the short-lived token is exchanged for a Databricks token. OAuth client secrets in GitHub and PATs are long-lived.
- **Test layers:**

| Layer                  | Checks                                                   | When                        |
| ---------------------- | -------------------------------------------------------- | --------------------------- |
| Component (unit)       | One tool, parser or retriever; LLM stubbed; fixed inputs | Every pull request          |
| Evaluation gate        | Whole agent or prompt version vs dataset and thresholds  | Before promotion            |
| Post-deploy smoke test | The deployed endpoint answers representative requests    | After staging reports Ready |
| Monitoring             | Scorers on live traces                                   | Continuously                |

- Unit tests are automated, deterministic, cheap and isolated. A stubbed LLM can return malformed JSON on purpose. Retriever tests assert expected chunk IDs.
- **Evaluation gates:** evaluate a pinned version number, then point production at that same number. Do not resolve `@latest` twice, move the alias first, or re-register on promotion.
- **Ready means the container started.** Undeclared resources only fail on a real request, so smoke-test the endpoint.
- **Index freshness in pipelines:** triggered Delta Sync → call `index.sync()` as the last step after the write. Change data feed is a prerequisite, not a trigger. Storage-optimized supports only triggered sync. Schema or embedding-model change → create new index, wait for initial sync, switch, delete old.
- **Prompt lifecycle:**

| URI                                       | Resolves to                       | Use for                      |
| ----------------------------------------- | --------------------------------- | ---------------------------- |
| `prompts:/main.support.answer/6`          | Version 6 forever                 | Regression tests, evaluation |
| `prompts:/main.support.answer@production` | The alias's current version       | Deployed apps                |
| `prompts:/main.support.answer@latest`     | Newest registration, draft or not | Experiments only             |

- Commit messages and tags are metadata; they never change what a URI resolves to.
- **Promotion:** one prompt with an alias per environment. Load the source alias, read `.version`, then `set_prompt_alias(name, alias="staging", version=cand.version)`. Promotion never registers anything.
- **Rollback:** point production back at the previous version. Also move any alias a scheduled job copies from, or it will undo the rollback.
- **`load_prompt` caching:** alias loads cached with a 60-second TTL by default (`MLFLOW_ALIAS_PROMPT_CACHE_TTL_SECONDS`); version loads cached with no TTL. `cache_ttl_seconds=5` speeds propagation; `0` hits the registry every call.
- **MCP server types:**

| Type               | What it is                                                                                          | Choose when                                  |
| ------------------ | --------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| Databricks-managed | Provided servers for Genie, Vector Search, UC functions, Databricks SQL, and some third-party tools | Databricks already provides one; no code     |
| External           | A vendor server that already speaks MCP over Streamable HTTP, registered via a UC HTTP connection   | A vendor publishes an MCP server             |
| Custom             | Your own server hosted on Databricks Apps                                                           | Nothing exists, such as an internal REST API |

- A plain REST API cannot be registered as an external MCP server. For an OpenAPI spec, use the OpenAPI MCP server template on Databricks Apps.
- **HTTP connection auth decides whose identity the vendor sees.** Only **per-user OAuth (user-to-machine)** gives per-person access and attribution. Bearer token, M2M and shared U2M are one identity.
- **Registered MCP privileges:** callers need `EXECUTE` on the MCP plus `USE CATALOG` and `USE SCHEMA`; `USE CONNECTION` is for authors and admins and bypasses the MCP's policies if given to callers. (Legacy proxy-URL pattern: callers needed `USE CONNECTION`.)
- **Custom MCP on Databricks Apps:** Streamable HTTP (not stdio); bind `0.0.0.0` on `DATABRICKS_APP_PORT`; app name starts with `mcp-` for AI Playground; clients use the app URL plus `/mcp`; callers need `CAN USE` on the app.
- **Interfaces:**

| Interface               | Best for                                            |
| ----------------------- | --------------------------------------------------- |
| Databricks Apps chat UI | Production internal web chat with SSO and streaming |
| MLflow Review App       | Pre-launch expert feedback into evaluation          |
| AI Playground           | Prototyping and comparing models, prompts, tools    |
| Slack app / Teams bot   | Users who stay in their messaging tool              |

- A tool integration lets the agent call **out** to a platform; a chat interface needs a bot that carries messages **in**.
- **Apps authorization:** app authorization runs as the app's service principal (declare the serving endpoint as a resource with **Can query**). Per-user rows need **user authorization** (model-serving scope, forwarding the user's token) **and** OBO in the agent.
- **Teams (Public Preview):** Azure Bot Service bot + Entra ID app registration and OAuth connection + a Databricks OAuth federation policy that trusts the tenant issuer and bot audience, so the user's Entra token is exchanged for a Databricks token. Add OBO in the agent.
- **Traps:**
  - Folder copies or manual notebook exports are not CI/CD.
  - LLM-judge evaluation does not belong in unit tests.
  - A sync timer every few minutes wastes rebuilds on a storage-optimized index.
  - If the token reaching Databricks belongs to one shared identity, every user gets that identity's rows.

📖 Full lesson: [Databricks GenAI Engineer: CI/CD for Agents, Prompt Lifecycle, MCP Servers and Agent Interfaces](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-cicd-prompt-lifecycle-mcp-servers/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

[← Back to the study guide](../README.md)
