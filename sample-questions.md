# Databricks GenAI Sample Questions with Answers

20 worked practice questions for the **Databricks Certified Generative AI Engineer Associate (Databricks GenAI)** exam. Try each one before you open the answer.

For many more, with the same explanation on every option, use the [Databricks GenAI practice questions](https://www.savemycert.com/practice/databricks-generative-ai-engineer-associate/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide) on SaveMyCert.

## Question 1

*Design Applications*

**A Generative AI Engineer at a telecom company must place each incoming support ticket into one of five existing queues: billing, outage, device, porting or cancellation. Each ticket goes to exactly one queue, and the queue name is written back to the ticketing system. Which model task should the engineer select for this requirement?** Choose one.

- **a.** Text generation that drafts a reply to each ticket for the queue
- **b.** Text classification of each ticket against the five queue names
- **c.** Named entity recognition over each ticket to find the queue terms
- **d.** Text summarization of each ticket into a short queue description

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Drafting a reply is a later step for an agent; it does not produce the routing decision the requirement asks for.
- **b** ✅ Routing a ticket to exactly one of a fixed set of known queues is single-label text classification; the output is one of the five labels.
- **c** ❌ Entity recognition pulls spans such as phone numbers or plan names out of the text; it does not decide which queue the ticket belongs to.
- **d** ❌ A summary condenses the ticket but returns free text, not one of the five queue names the ticketing system expects.

**The concept.** Model task selection starts from the output the business needs: a choice among known categories is classification.

**Why this is correct.** The required output is one value from a closed list of five queues, and each ticket gets exactly one. That is single-label text classification. Summarization and generation return free text that would still need to be mapped to a queue, and entity recognition extracts spans from the ticket rather than deciding a category.

**How to reason it out**

1. Look at the output the system needs: one of five fixed queue names.
2. A fixed, known set of categories means classification.
3. Exactly one queue per ticket means single-label, not multi-label.

> **Exam tip:** Closed list of categories as the output: choose text classification.

</details>

📖 Learn this topic: [Databricks GenAI: Prompt Design, Output Formats and Model Task Selection](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-prompt-design-model-task-selection/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 2

*Design Applications*

**A Generative AI Engineer is designing a pipeline that tags product reviews for a quality team. The team maintains six issue categories, such as packaging damage, late delivery and missing parts. Analysis of past reviews shows that many reviews describe two or three problems at once, and the team wants every applicable category recorded. How should the engineer describe the model task?** Choose one.

- **a.** Single-label classification that returns the most likely category for each review
- **b.** Multi-label classification that returns every matching category for each review
- **c.** Sentiment analysis that returns a positive, negative or mixed score for each review
- **d.** Extractive summarization that returns the key sentence about issues in each review

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ Single-label classification keeps only one issue, so a review that mentions damage and late delivery would lose one of them.
- **b** ✅ Reviews can describe several issues and the team wants all of them, so the task must be allowed to return more than one label per review.
- **c** ❌ Sentiment tells the team how the customer feels, not which of the six issue categories the review describes.
- **d** ❌ A key sentence is free text that someone would still have to map to categories, and it may cover only one issue.

**The concept.** Classification can be single-label (one category per input) or multi-label (any number of categories per input).

**Why this is correct.** The deciding facts are that one review can describe several problems and that the team wants every applicable category. Single-label classification would force a choice and drop issues. Sentiment addresses a different question, and a summary sentence is unstructured text. Multi-label classification returns each matching category from the fixed set; Databricks ai_classify supports this with its multilabel option.

**How to reason it out**

1. Identify the output: categories from a fixed list of six.
2. Check whether one input can belong to several categories: here it can.
3. Choose multi-label classification so no applicable category is dropped.

> **Exam tip:** If an input can fall into several known categories and you need all of them, specify multi-label classification.

</details>

📖 Learn this topic: [Databricks GenAI: Prompt Design, Output Formats and Model Task Selection](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-prompt-design-model-task-selection/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 3

*Design Applications*

**A Generative AI Engineer is building a chain that receives an employee's question as a plain string and must return a plain-text answer grounded in the company's travel policy documents, which are already chunked into a Vector Search index. The policies change every few weeks, and answers must reflect the current text without retraining any model. Which sequence of chain components meets these requirements?** Choose one.

- **a.** Retriever over the policy index, prompt template that inserts the passages, chat model, string output parser
- **b.** Prompt template with the question, chat model, retriever over the policy index, string output parser
- **c.** Retriever over the policy index, chat model, prompt template that inserts the passages, JSON output parser
- **d.** Prompt template with the question, chat model fine-tuned on the policies, string output parser

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Correct. Retrieval runs first so the current policy passages can be placed into the prompt; the chat model answers from them and a string parser returns plain text. Updating the index keeps answers current with no retraining.
- **b** ❌ The retriever runs after the model has already answered, so the passages never reach the prompt and the answer is not grounded in the policy text.
- **c** ❌ The prompt template must be filled before the model is called, and a JSON parser does not fit the required plain-text output.
- **d** ❌ A fine-tuned model would have to be retrained every time the policies change, which the requirement rules out, and it cannot cite the current text.

**The concept.** A retrieval-augmented chain is a pipeline of components whose order follows the data: query in, context retrieved, prompt built, model called, output parsed.

**Why this is correct.** The input is a raw question and the output is plain text grounded in documents that change often. That calls for retrieval at query time (so changes in the index show up immediately), a prompt template that combines the question with the retrieved passages, a chat model to generate, and a string output parser. Putting the retriever after the model, or the template after the model, breaks the data flow; fine-tuning bakes in a snapshot of the policies and must be repeated on every change.

**How to reason it out**

1. Identify the input (a question string) and the output (plain text).
2. Note the grounding requirement on frequently changing documents: retrieve at query time instead of training.
3. Order components by data flow: retriever, then prompt template, then model, then parser.
4. Pick the parser that produces the required output format.

> **Exam tip:** Order chain components by data flow: retrieve, build the prompt, generate, parse into the required output.

</details>

📖 Learn this topic: [Databricks GenAI: Chain Components, Agent Tools and Agent Bricks](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-chain-components-tools-agent-bricks/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 4

*Data Preparation*

**A Generative AI Engineer is building a RAG assistant over 4,000 signed supplier contracts that were scanned to PDF years ago. A test extraction with pypdf returns empty strings for almost every page, because the files contain only page images with no text layer. Which approach should the engineer use to get the contract text?** Choose one.

- **a.** Render pages to images and run Tesseract OCR on them through pytesseract
- **b.** Open each file with PyMuPDF and call get_text() on each of its page objects
- **c.** Call pypdf extract_text() with extraction_mode set to "layout"
- **d.** Use pdfplumber to collect the words and characters found on each page

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The pages are images, so the text has to be recognised optically. pytesseract wraps the Tesseract OCR engine and returns the text that appears in each rendered page image.
- **b** ❌ PyMuPDF's get_text() reads the PDF's text layer, just as pypdf does. With image-only pages there is no text layer, so it also returns almost nothing.
- **c** ❌ Layout mode only changes how pypdf arranges text it finds in the text layer. It adds no OCR, so image-only pages still come back empty.
- **d** ❌ pdfplumber is excellent for text-based PDFs and tables, but it works from the embedded character objects. Scanned pages have none, so it cannot recover the text.

**The concept.** Scanned PDFs are images: extracting their text needs OCR, not a text-layer PDF parser.

**Why this is correct.** The symptom (empty strings from pypdf) tells you the PDFs have no text layer. pypdf, PyMuPDF get_text() and pdfplumber all read that embedded layer, so switching between them changes nothing. Only an OCR engine such as Tesseract (via pytesseract) can turn page images into text.

**How to reason it out**

1. Read the symptom: empty extraction means no text layer, not a parser bug.
2. Separate text-layer parsers (pypdf, PyMuPDF, pdfplumber) from OCR engines.
3. Choose OCR (pytesseract over rendered page images) for scanned documents.

> **Exam tip:** No text layer means OCR: text-layer PDF parsers cannot read scanned pages.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Document Extraction, Filtering and Chunking into Delta Tables](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-document-chunking-delta-tables/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 5

*Data Preparation*

**A Generative AI Engineer is indexing a help center exported as HTML. Every article uses <h2> and <h3> elements for its sections. The team wants each chunk to stay inside one section and to carry the section titles as metadata so answers can cite them. Which approach should the engineer use to split the pages?** Choose one.

- **a.** RecursiveCharacterTextSplitter.from_language() with the Language.HTML setting
- **b.** LangChain MarkdownHeaderTextSplitter configured for the ## and ### headers
- **c.** BeautifulSoup get_text() followed by a RecursiveCharacterTextSplitter pass
- **d.** LangChain HTMLHeaderTextSplitter configured to split on the h2 and h3 tags

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ The HTML language preset prefers to break at tags, but it does not record which section a chunk belongs to, so the section titles are not attached as metadata.
- **b** ❌ MarkdownHeaderTextSplitter looks for Markdown # markers. Raw HTML uses <h2>/<h3> elements, so it would find no headers and return the page unsplit.
- **c** ❌ get_text() flattens the page into plain text, so the header structure is lost before splitting. Chunks follow character counts and carry no section-title metadata.
- **d** ✅ HTMLHeaderTextSplitter splits at the header elements you list and attaches each header's text to the resulting chunks as metadata, which is exactly the section-aligned output required.

**The concept.** Structure-aware splitters split on a document's own headings and keep those headings as chunk metadata.

**Why this is correct.** The requirement has two parts: chunks bounded by sections, and section titles kept as metadata. HTMLHeaderTextSplitter does both for HTML. Flattening with get_text() loses the structure, the HTML language preset ignores which section a chunk sits in, and the Markdown splitter expects # markers rather than heading elements.

**How to reason it out**

1. Identify the source format (HTML with heading elements).
2. Note both requirements: section-bounded chunks and header metadata.
3. Pick the splitter that understands that format's headers and records them.

> **Exam tip:** Match the splitter to the format: HTML headings use HTMLHeaderTextSplitter, Markdown headings use MarkdownHeaderTextSplitter.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Document Extraction, Filtering and Chunking into Delta Tables](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-document-chunking-delta-tables/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 6

*Data Preparation*

**A Generative AI Engineer is comparing retrieval configurations for a compliance assistant. The chain passes the top 20 chunks to an LLM with a large context window, and reviewers say the costly failure is a relevant regulation clause that never reaches the model at all. Labeled relevant chunks exist for 200 test questions. Which metric should the engineer use to compare the configurations?** Choose one.

- **a.** Precision@3 against the labeled relevant chunks
- **b.** Average similarity score of the top 20 results
- **c.** Recall@20 against the labeled relevant chunks
- **d.** Mean reciprocal rank of the first relevant chunk

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ Precision@3 measures how clean the first three results are; the chain passes 20, and a missing clause at rank 15 or 40 is not what it measures.
- **b** ❌ Similarity scores show how close vectors are, not whether the labeled relevant clauses were returned, and they are not comparable across embedding models.
- **c** ✅ Recall@20 is the share of all relevant chunks that appear in the 20 passed to the model, which is exactly the failure reviewers care about.
- **d** ❌ MRR rewards ranking one relevant chunk early; it says nothing about whether the other relevant clauses were retrieved.

**The concept.** Recall@k is the fraction of all relevant items that appear in the top k results; precision@k is the fraction of the top k results that are relevant.

**Why this is correct.** When missing information is the expensive failure and the model can read many chunks, recall at the number of chunks actually passed (k = 20) is the metric to optimise. Precision@3 suits cases where only a few chunks are used, MRR only cares about the first hit, and raw similarity scores are not a quality metric against ground truth.

**How to reason it out**

1. Identify the costly failure: a relevant chunk never retrieved.
2. Note how many chunks reach the model (20).
3. Pick the metric that counts retrieved relevant items over all relevant items at that k.

> **Exam tip:** Missed facts are costly and k is large: optimise recall@k at the k you actually pass to the LLM.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Retrieval Evaluation, Advanced Chunking and Re-ranking](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-retrieval-evaluation-reranking/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 7

*Application Development*

**A Generative AI Engineer is building a document Q&A app on Databricks. The flow is fixed and linear: retrieve chunks from a Vector Search index, fill a prompt template, call a chat model and parse the reply into a field. The engineer wants ready-made, swappable components for each of those steps instead of writing glue code. Which tool fits best?** Choose one.

- **a.** DSPy, compiling the prompt template steps against a metric and labeled examples
- **b.** MLflow Tracing, recording the retriever, prompt and chat model calls as spans
- **c.** Hugging Face Transformers, running the chat model in a text-generation pipeline
- **d.** LangChain, chaining a retriever, prompt template, chat model and output parser

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ DSPy optimizes prompts and modules against a metric. The scenario has no metric or training set and asks only for ready-made components to wire together.
- **b** ❌ Tracing observes an application that already exists; it does not supply the components that make up the chain.
- **c** ❌ Transformers loads and runs models. It offers no retriever, prompt template or output-parser components to assemble a RAG flow.
- **d** ✅ LangChain provides exactly these building blocks and composes them into a chain, with Databricks integrations for the retriever and the chat model.

**The concept.** LangChain is the general-purpose framework for composing LLM application components into chains.

**Why this is correct.** The requirement is a fixed sequence of standard steps with prebuilt, interchangeable parts. That is LangChain's core purpose: retrievers, prompt templates, chat models and output parsers composed into one chain. DSPy targets metric-driven optimization, Transformers is a model runtime, and MLflow Tracing only observes what runs.

**How to reason it out**

1. Note the shape: a fixed, linear flow of standard steps.
2. Ask which tool supplies components for every step.
3. Rule out tools that optimize, run models or observe instead of compose.

> **Exam tip:** Fixed pipeline of standard steps: compose it with LangChain components.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: LangChain, Prompt Augmentation and Prompt Engineering](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-langchain-prompt-engineering/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 8

*Application Development*

**A Generative AI Engineer is designing a support agent that calls tools, inspects each result and loops back to try another tool when one fails. Refund requests must pause for a human approver and resume hours later from the same point, with the conversation state intact. Which framework is the best fit for this control flow?** Choose one.

- **a.** LlamaIndex, indexing the support documents behind a single query engine
- **b.** LangChain, piping a fixed sequence of runnables into one linear chain
- **c.** LangGraph, modelling the steps as a graph with cycles and a checkpointer
- **d.** DSPy, defining signatures for each step and compiling against a metric

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ LlamaIndex centres on indexing and querying data. A query engine answers questions; it does not model looping tool calls with human approval.
- **b** ❌ A linear chain runs its steps once in order. It has no built-in loops or persisted state to pause for a human and resume later.
- **c** ✅ LangGraph is built for stateful workflows: nodes and edges allow loops and branches, and a checkpointer persists state so a run can pause for approval and resume.
- **d** ❌ DSPy improves prompts against a metric. It does not provide checkpointed, resumable control flow with approval pauses.

**The concept.** LangGraph adds stateful, cyclical and resumable control flow on top of LangChain components.

**Why this is correct.** Two requirements decide it: loops (retry another tool after a failure) and durable state across a pause (human approval hours later). LangGraph models the agent as a graph whose edges can cycle and whose checkpointer saves state for resumption. A plain LangChain chain is linear and stateless, LlamaIndex is data-centric, and DSPy is an optimizer.

**How to reason it out**

1. Spot loops and branches in the requirement.
2. Spot the need to pause and resume with saved state.
3. Choose the framework that offers cycles plus checkpointing.

> **Exam tip:** Loops, branches and pause-resume state point to LangGraph, not a linear chain.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: LangChain, Prompt Augmentation and Prompt Engineering](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-langchain-prompt-engineering/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 9

*Application Development*

**A Generative AI Engineer reviews MLflow traces for a RAG assistant that told a customer "Your device warranty lasts 12 months." The policy says 24 months. In the trace, the retriever span's top-ranked chunk reads "All devices carry a 24-month warranty from the date of purchase." Which stage of the application should the engineer investigate first?** Choose one.

- **a.** The generation stage, because the right fact reached the prompt but went unused
- **b.** The embedding stage, because the query and chunk vectors were too far apart
- **c.** The retrieval stage, because the retriever returned the wrong policy document for this question
- **d.** The chunking stage, because the warranty sentence was split across two chunks

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ The retriever span proves the right sentence was in the context. The failure happened after retrieval, when the LLM produced its answer, so the prompt and model are where to look.
- **b** ❌ A poor embedding match would keep the chunk out of the results. Here it was ranked first.
- **c** ❌ The top chunk is the right document and contains the right figure, so retrieval did its job for this request.
- **d** ❌ The full sentence, including the 24-month figure, appears inside one retrieved chunk, so nothing was split.

**The concept.** A trace separates retrieval from generation: check what reached the prompt before blaming either stage.

**Why this is correct.** The retriever span shows the correct 24-month sentence ranked first, so retrieval, chunking and embeddings all worked for this question. The model still answered 12 months, which is a generation failure: it ignored or contradicted its context. Fixes belong in the prompt (stronger grounding instructions) or the model choice, not the index.

**How to reason it out**

1. Open the retriever span and read what context the model received.
2. If the answer is in the context, the failure is in generation.
3. If the answer is missing from the context, the failure is in retrieval or chunking.

> **Exam tip:** Right fact in the context but wrong answer means a generation problem, not a retrieval one.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Response Quality, Safety and LLM Guardrails](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-response-quality-safety-guardrails/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 10

*Application Development*

**A Generative AI Engineer is choosing the LLM for a pipeline that labels about 500,000 short support tickets per day with one of six fixed categories. On a labelled sample, every candidate instruct model scores within one point of the others on accuracy. The business wants the lowest cost per ticket and fast turnaround. Which model should the engineer choose?** Choose one.

- **a.** A small 8-billion-parameter base model without instruction tuning
- **b.** A large reasoning model with extended thinking turned on
- **c.** A small instruction-tuned model of about 8 billion parameters
- **d.** A 70-billion-parameter instruction-tuned model with a 128K context

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ A base model is not tuned to follow instructions, so it is a poor fit for prompting it to return one label. The evaluated candidates were instruct models.
- **b** ❌ Extended thinking spends extra tokens and time on each ticket, raising cost and latency for no measured gain in accuracy on this task.
- **c** ✅ Every candidate already meets the accuracy bar, so the cheapest and fastest model that does wins. A small instruct model costs far less per token and returns sooner.
- **d** ❌ It classifies no better than the small model here, yet costs several times more per token. A long context adds nothing for short tickets.

**The concept.** Pick the smallest model that meets the quality requirement when volume, cost or latency dominate.

**Why this is correct.** Model size buys quality only where the task needs it. Short-ticket classification into fixed labels is a simple task, and the sample shows the candidates perform the same. At 500,000 requests a day, the per-token price and latency of a large or reasoning model multiply into real cost with no accuracy return. A base model is the wrong variant for instruction-following use.

**How to reason it out**

1. Check whether the quality requirement separates the candidates.
2. If several meet it, compare cost and latency at the expected volume.
3. Choose the smallest instruct model that meets the bar.

> **Exam tip:** When candidates tie on quality, the smaller instruct model wins on cost and latency.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Choosing LLMs and Embedding Models](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-choosing-llms-embedding-models/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 11

*Application Development*

**A Generative AI Engineer has a working LangGraph agent that calls two tools. The team wants to log it with MLflow and serve it on Databricks with streaming responses, automatic model signature inference and automatic tracing, without rewriting the graph's logic. Which approach meets these requirements?** Choose one.

- **a.** Wrap the compiled graph in a class that subclasses mlflow.pyfunc.PythonModel with a hand-written signature
- **b.** Wrap the compiled graph in a class that subclasses mlflow.pyfunc.ResponsesAgent and implements predict
- **c.** Rewrite each graph node as a Unity Catalog Python function and expose the set through UCFunctionToolkit
- **d.** Recreate the agent as an Agent Bricks Knowledge Assistant that points at the same two tools

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ A plain PythonModel can serve the graph, but the signature must be written by hand and it lacks the agent-specific request/response schema, streaming events and automatic tracing that ResponsesAgent provides.
- **b** ✅ Correct. ResponsesAgent is the interface Databricks recommends for authoring agents. A thin wrapper converts requests and outputs, and MLflow infers the signature and traces predict and predict_stream automatically.
- **c** ❌ UCFunctionToolkit turns UC functions into tools an agent can call; it does not package the agent itself, and it means rewriting the graph's logic, which the team wants to avoid.
- **d** ❌ Knowledge Assistant is a no-code builder for question answering over documents; it does not import an existing LangGraph graph or its custom tool logic.

**The concept.** Mosaic AI Agent Framework agents are authored in code and packaged with an MLflow interface; Databricks recommends mlflow.pyfunc.ResponsesAgent.

**Why this is correct.** ResponsesAgent is a typed, framework-agnostic wrapper: the existing LangGraph, OpenAI SDK or custom agent stays as it is, and the wrapper's predict (and optionally predict_stream) translates between the Responses-style request and the agent's own format. In return MLflow infers the signature, traces predict and predict_stream automatically, and Databricks tools (AI Playground, Review App, inference tables) understand the output. A plain PythonModel works but leaves signature and streaming to the author; rewriting nodes as UC functions or rebuilding in Agent Bricks both discard the existing graph.

**How to reason it out**

1. Note the requirement to keep the existing graph logic.
2. Note the features wanted: streaming, automatic signature, automatic tracing.
3. Pick the MLflow interface designed for agents that wraps any framework.
4. Reject options that rebuild the agent elsewhere.

> **Exam tip:** Keep the agent's framework; wrap it in mlflow.pyfunc.ResponsesAgent to get signature inference, streaming and tracing.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Building Agents with MLflow, Agent Framework and Genie](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-agent-framework-mlflow-genie/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 12

*Application Development*

**A Generative AI Engineer has an app that makes one call to a Foundation Model API chat endpoint with a fixed prompt. A security review requires the fewest added third-party packages, and the team must capture an MLflow trace of every call during development. Which approach meets both requirements?** Choose one.

- **a.** Call the endpoint with the OpenAI client and record each prompt with mlflow.log_param()
- **b.** Wrap the call in a LangChain chain with ChatDatabricks and enable mlflow.langchain.autolog()
- **c.** Call the endpoint with the OpenAI client and enable mlflow.openai.autolog()
- **d.** Model the call as a one-node LangGraph graph and attach a checkpointer to record it

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ This keeps the dependencies small, but logging a parameter to a run is not a trace and captures neither the response nor timing spans.
- **b** ❌ This produces traces, but it adds LangChain packages for a single call, breaking the fewest-packages requirement.
- **c** ✅ Foundation Model APIs are OpenAI-compatible, so the existing client works with no framework. MLflow's OpenAI autologging records each call as a trace.
- **d** ❌ A checkpointer saves graph state for resumption; it is not tracing. It also adds a framework the single call does not need.

**The concept.** A framework is optional: a single call can use the OpenAI-compatible client directly, and MLflow autologging traces it.

**Why this is correct.** Frameworks earn their place when they remove glue code. One fixed call needs none, and Foundation Model API endpoints accept the OpenAI client. MLflow tracing autologging covers that client, so traces come without adding LangChain or LangGraph. The LangChain option traces but adds packages; the log_param option keeps packages low but produces no traces.

**How to reason it out**

1. Check whether the app's flow actually needs an orchestration framework.
2. Recall that Foundation Model APIs are OpenAI-compatible.
3. Pick the tracing method that matches the client you keep.
4. Reject options that meet one requirement and break the other.

> **Exam tip:** One fixed call needs no framework: use the OpenAI-compatible client and trace it with MLflow autologging.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: LangChain, Prompt Augmentation and Prompt Engineering](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-langchain-prompt-engineering/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 13

*Assembling and Deploying Applications*

**A Generative AI Engineer works in a workspace whose default catalog is hive_metastore, on a cluster with MLflow 2.x. A pyfunc RAG chain has been logged to a run, and it must now be registered as a governed model under the catalog ml and schema support. Calls to register it keep creating models in the legacy workspace registry. Which line should the engineer run before registering?** Choose one.

- **a.** mlflow.set_registry_uri("databricks-uc")
- **b.** mlflow.set_tracking_uri("databricks-uc")
- **c.** mlflow.set_registry_uri("databricks")
- **d.** mlflow.set_tracking_uri("databricks")

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ This points the MLflow client's model registry at Unity Catalog, so the next registration creates a Unity Catalog model.
- **b** ❌ The tracking URI controls where runs and experiments are logged, not which model registry receives registrations.
- **c** ❌ The databricks registry URI is the legacy workspace model registry, which is exactly what the engineer is trying to leave.
- **d** ❌ The tracking URI only decides where runs and experiments are logged, and databricks is already the workspace default; registrations still go to the legacy registry.

**The concept.** The MLflow registry URI decides where registered models are created; databricks-uc means Unity Catalog.

**Why this is correct.** With MLflow 2.x and a non-Unity Catalog default catalog, the client still registers to the workspace registry. Setting the registry URI to databricks-uc switches registration to Unity Catalog. Setting the tracking URI, to either value, is a near-twin that only moves runs and experiments, and the plain databricks registry URI is the legacy registry. MLflow 3 uses databricks-uc by default, which is why the stem fixes the version.

**How to reason it out**

1. Separate the tracking target (runs) from the registry target (models).
2. Unity Catalog registration needs the registry URI databricks-uc.
3. Run it before calling register_model or log_model with a registered model name.

> **Exam tip:** Registry URI databricks-uc = Unity Catalog models; the tracking URI is a different setting.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Coding pyfunc Chains and Registering Them to Unity Catalog](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-pyfunc-chains-unity-catalog-registration/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 14

*Assembling and Deploying Applications*

**A Generative AI Engineer is logging a pyfunc support chain with mlflow.pyfunc.log_model and wants the same call to create a new model version in Unity Catalog. The model must live in the catalog ml, under the schema support, with the model name support_rag. Which argument should the engineer pass to the logging call?** Choose one.

- **a.** registered_model_name="models:/ml.support.support_rag"
- **b.** registered_model_name="support_rag@ml.support"
- **c.** registered_model_name="ml/support/support_rag"
- **d.** registered_model_name="ml.support.support_rag"

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ models:/ is a URI scheme for loading a model, not part of the name used when registering one.
- **b** ❌ The @ suffix is how an alias is referenced when loading; it cannot carry the catalog and schema.
- **c** ❌ Unity Catalog names use dots between catalog, schema and model, not slashes.
- **d** ✅ Unity Catalog models are addressed by the full three-level name catalog.schema.model, which this value supplies.

**The concept.** Unity Catalog models are registered under a three-level name: catalog.schema.model.

**Why this is correct.** Passing registered_model_name to log_model registers the logged model as a new version, and for Unity Catalog the name must be the full catalog.schema.model form. The slash form, the models:/ loading URI and the @alias syntax are all real MLflow notations, but each belongs to a different job.

**How to reason it out**

1. Recognise that registration needs a name, not a URI.
2. Unity Catalog names follow catalog.schema.model with dots.
3. Keep models:/ and @alias for loading, not registering.

> **Exam tip:** Register with catalog.schema.model; load with models:/catalog.schema.model@alias.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Coding pyfunc Chains and Registering Them to Unity Catalog](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-pyfunc-chains-unity-catalog-registration/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 15

*Assembling and Deploying Applications*

**A Generative AI Engineer is indexing 600 million chunks of archived engineering reports, embedded at 768 dimensions, for an internal research assistant. Researchers send a few hundred queries a day and accept about half a second for retrieval. The source Delta table is rebuilt once a week, and the budget is tight. Which configuration should the engineer choose?** Choose one.

- **a.** A standard endpoint with a Delta Sync index in continuous sync mode
- **b.** A storage-optimized endpoint with a Delta Sync index in triggered sync mode
- **c.** A standard endpoint with a Delta Sync index in triggered sync mode
- **d.** A storage-optimized endpoint with a Delta Sync index in continuous sync mode

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ This exceeds standard endpoint capacity and adds a continuously running sync cluster to refresh data that changes only weekly.
- **b** ✅ Storage-optimized endpoints hold over a billion 768-dimension vectors at a lower cost per vector, their added latency fits the half-second budget, and they support triggered sync, which suits a weekly rebuild.
- **c** ❌ A standard endpoint holds about 320 million 768-dimension vectors, so 600 million chunks do not fit, and it costs more per vector.
- **d** ❌ Storage-optimized endpoints do not support continuous sync mode; only triggered sync is available on them.

**The concept.** Endpoint type follows vector count, latency and cost; sync mode follows how often the data changes.

**Why this is correct.** Standard endpoints top out around 320 million vectors at 768 dimensions, so 600 million chunks need a storage-optimized endpoint, which is also cheaper per vector and adds only a few hundred milliseconds of latency. Weekly data does not need continuous sync, and storage-optimized endpoints only offer triggered sync anyway, which rules out the continuous storage-optimized option.

**How to reason it out**

1. Compare 600 million vectors with standard endpoint capacity (about 320 million at 768 dimensions).
2. Check that the half-second latency budget tolerates a storage-optimized endpoint.
3. Match sync mode to a weekly refresh: triggered.
4. Remember storage-optimized endpoints support triggered sync only.

> **Exam tip:** Hundreds of millions of vectors, relaxed latency, tight budget: storage-optimized endpoint with triggered sync.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Mosaic AI Vector Search Indexes, Endpoints, Querying and Sizing](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-mosaic-ai-vector-search/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 16

*Assembling and Deploying Applications*

**A Generative AI Engineer has deployed a support chain to a Model Serving endpoint. A group of data analysts must be able to send requests to it from their notebooks. They must not be able to change the served model version, edit the endpoint configuration or decide who else has access. Which permission should the engineer grant the analysts' group?** Choose one.

- **a.** CAN MANAGE on the serving endpoint
- **b.** EXECUTE on the registered UC model
- **c.** CAN QUERY on the serving endpoint
- **d.** CAN VIEW on the serving endpoint

<details>
<summary>Show answer and explanation</summary>

**Answer: c**

- **a** ❌ CAN MANAGE also allows updating the configuration, deleting the endpoint and changing permissions, which the analysts must not do.
- **b** ❌ EXECUTE governs loading the Unity Catalog model, not sending requests to a serving endpoint, which has its own permissions.
- **c** ✅ CAN QUERY adds sending requests to what CAN VIEW allows, and stops short of changing configuration or permissions.
- **d** ❌ CAN VIEW only lets the group get and list the endpoint; requests to it would be refused.

**The concept.** Serving endpoints have their own permission levels: CAN VIEW, CAN QUERY and CAN MANAGE.

**Why this is correct.** Each level includes the one below it. CAN VIEW allows get and list, CAN QUERY adds querying, and CAN MANAGE adds updating the config, deleting the endpoint and modifying permissions. Callers who only send requests need CAN QUERY. Unity Catalog privileges on the underlying model do not control who can call the endpoint.

**How to reason it out**

1. Note what the group must do: send requests only.
2. Map that to the lowest endpoint level that allows querying.
3. Reject levels that add config or permission changes.
4. Remember the endpoint is secured separately from the registered model.

> **Exam tip:** Callers get CAN QUERY; only owners and operators get CAN MANAGE.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Model Serving Access, Foundation Model APIs, ai_query() and Agent Memory](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-model-serving-ai-query-agent-memory/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 17

*Governance*

**A Generative AI Engineer runs a customer-support chat model behind a Databricks Model Serving endpoint. Customers often paste their phone numbers and home addresses into questions. Compliance says those personal details must not reach the model, and the product owner requires such questions to still be answered rather than rejected. Which endpoint setting meets both requirements?** Choose one.

- **a.** Input PII detection set to Mask under the endpoint's AI Gateway
- **b.** The Safety guardrail enabled for the endpoint's request inputs
- **c.** Input PII detection set to Block in the endpoint's AI Gateway
- **d.** Per-user rate limits configured in the endpoint's AI Gateway

<details>
<summary>Show answer and explanation</summary>

**Answer: a**

- **a** ✅ Mask replaces the detected personal details and lets the request continue, so the model never sees them and the customer still gets an answer.
- **b** ❌ The Safety guardrail screens harmful content; phone numbers and addresses are not what it looks for.
- **c** ❌ Block keeps the details from the model, but it rejects the whole request, which breaks the requirement to answer it.
- **d** ❌ Rate limits cap request volume per user; they do nothing about personal details inside a request.

**The concept.** AI Gateway PII detection on a serving endpoint has three behaviours: Block, Mask and None.

**Why this is correct.** Both Block and Mask stop detected personal data from reaching the model, but Block rejects the request while Mask removes the details and lets the request proceed. When the objective is to keep answering, Mask is the setting. Safety screening and rate limits are other gateway features that do not detect personal data.

**How to reason it out**

1. Requirement 1: personal details must not reach the model, so PII detection must act on inputs.
2. Requirement 2: the request must still be answered, which rules out rejecting it.
3. Mask removes the details and forwards the rest; Block rejects the request.

> **Exam tip:** PII must not reach the model and the request must still succeed = PII detection set to Mask.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Guardrails, PII Masking and Data Licensing](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-guardrails-pii-masking-data-licensing/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 18

*Governance*

**A Generative AI Engineer enabled PII detection with Block on the inputs of an HR assistant's serving endpoint. Since then, about 18% of requests fail, mostly because employees mention a colleague's name while asking about leave rules. HR's objective is that at least 98% of questions get an answer, and personal data must still never reach the model. What should the engineer change?** Choose one.

- **a.** Replace input PII detection with the Safety guardrail
- **b.** Add a system prompt rule telling the model to disregard names
- **c.** Switch input PII detection off and keep it on outputs
- **d.** Switch the input PII detection behaviour from Block to Mask

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ The Safety guardrail targets harmful content, so names would pass through to the model unscreened.
- **b** ❌ The names would still be sent to the model; a prompt rule cannot stop data that has already arrived.
- **c** ❌ The answer rate recovers, but names would then reach the model, which breaks the second requirement.
- **d** ✅ Mask strips the names and forwards the rest of the question, so the leave-rule question is answered and no personal data reaches the model.

**The concept.** Block versus Mask is a trade-off between rejecting a request and sanitising it.

**Why this is correct.** The failures come from the Block behaviour rejecting any request that contains a name. Mask keeps the protection (the model sees a placeholder instead of the name) while letting the request complete, which is how masking serves an answer-rate objective. Turning input detection off or relying on a prompt rule lets personal data reach the model, and the Safety guardrail does not look for personal data at all.

**How to reason it out**

1. Find the cause of the failures: Block rejects every request containing a name.
2. Keep the protection that personal data never reaches the model.
3. Pick the behaviour that sanitises instead of rejecting: Mask.

> **Exam tip:** When blocking hurts a success-rate objective, mask the sensitive part instead of rejecting the request.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Guardrails, PII Masking and Data Licensing](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-guardrails-pii-masking-data-licensing/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 19

*Evaluation and Monitoring*

**A Generative AI Engineer has built an evaluation dataset of 400 customer questions for a support agent. The records hold only the questions; nobody has written reference answers or expected facts yet. The engineer plans to run several built-in MLflow judges this week. Which judge will not produce meaningful results until the records gain expectations?** Choose one.

- **a.** Safety
- **b.** Guidelines
- **c.** RelevanceToQuery
- **d.** Correctness

<details>
<summary>Show answer and explanation</summary>

**Answer: d**

- **a** ❌ Safety checks the response for harmful content and needs no reference answer.
- **b** ❌ Guidelines applies criteria the engineer passes to the judge, so it runs without per-record labels.
- **c** ❌ RelevanceToQuery judges whether the response addresses the request using only inputs and outputs.
- **d** ✅ Correctness compares the response with expected facts or an expected response, so it needs expectations on each record.

**The concept.** Some built-in judges compare the response with ground truth supplied as expectations; others judge the response on its own.

**Why this is correct.** Correctness asks whether the response agrees with the expected facts or expected response stored in the record's expectations. Without them it has nothing to compare against. RelevanceToQuery and Safety read only the request and response, and Guidelines reads the criteria given to the judge itself, so all three can run on a questions-only dataset today.

**How to reason it out**

1. Note what the records contain: inputs only.
2. Ask which judge compares the answer with a reference.
3. Correctness needs expected facts or an expected response, so it waits for labels.

> **Exam tip:** Correctness needs ground truth; RelevanceToQuery, Safety and Guidelines do not.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Evaluating LLMs and Agents with MLflow Scorers and Judges](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-evaluating-llms-agents-mlflow-scorers/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

## Question 20

*Evaluation and Monitoring*

**A Generative AI Engineer runs a customer-facing chat model on a Databricks Model Serving endpoint. The audit team asks for a governed Unity Catalog table, queryable with SQL, that holds the full text of every prompt and every model reply the endpoint handles. The chain code must not change. What should the engineer enable on the endpoint?** Choose one.

- **a.** The Safety guardrail for the endpoint's traffic
- **b.** AI Gateway inference tables for the endpoint
- **c.** Custom tags on the endpoint for billing reports
- **d.** AI Gateway usage tracking for the endpoint

<details>
<summary>Show answer and explanation</summary>

**Answer: b**

- **a** ❌ The Safety guardrail screens prompts and replies for harmful content; it does not keep a record of them.
- **b** ✅ Inference tables log each request and response payload as JSON to a Delta table in the catalog and schema you choose, with no change to the chain.
- **c** ❌ Custom tags label the endpoint's spend in system.billing.usage; they capture no request content.
- **d** ❌ Usage tracking writes token and character counts per request to system tables; it does not store the prompt or reply text.

**The concept.** AI Gateway inference tables log raw request and response payloads of a serving endpoint to a Unity Catalog Delta table.

**Why this is correct.** The requirement is the full text of prompts and replies in a governed, SQL-queryable table. Inference tables are the AI Gateway feature that captures payloads, along with status code, latency and requester, without touching the application. Usage tracking records counts, not text; custom tags only label billing rows; guardrails filter traffic but keep no log.

**How to reason it out**

1. Identify what must be stored: the content of requests and responses.
2. Match it to the AI Gateway feature that logs payloads: inference tables.
3. Rule out features that record counts, costs or safety verdicts only.

> **Exam tip:** Need the actual prompts and replies? Inference tables. Need counts? Usage tracking.

</details>

📖 Learn this topic: [Databricks GenAI Engineer: Monitoring, Inference Tables and LLM Cost Control](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-monitoring-inference-tables-cost-control/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

---

[← Back to the Databricks GenAI study guide](README.md)
