# Domain 2: Data Preparation (14%)

This domain is the knowledge pipeline behind a RAG application: choosing the source documents, extracting their content with the right parser, filtering noise, chunking to fit both the embedding model and the LLM, landing the chunks in a Unity Catalog Delta table, and then measuring and improving what retrieval returns. Most questions describe a source collection or a retrieval symptom and ask which change fixes it in the data. A stated constraint (least privilege, least custom code, one index, no third-party libraries, incremental processing) usually decides between several workable options.

## 2.1 Document extraction, filtering and chunking into Delta tables

- **Choosing sources.** Ask four questions of every candidate:
  - **Authoritative?** Official manuals and published policies beat forum threads, chat logs and tickets.
  - **Current?** Archived editions compete with the current text at retrieval time.
  - **Holds the needed knowledge?** An assistant about your own rules needs your own documents. Public material repeats what the base model already knows.
  - **Fit for everyone who can query the index?** Salary sheets, case files and tickets with personal data answer no policy question and become retrievable by anyone.
- **Sensitive content:** keep it out at ingestion. Model instructions, query filters and partial masking all leave the record retrievable. Different audiences needing different documents is a design for per-user filtering or separate indexes.
- **Missing knowledge needs a new source.** Search the chunks table for the expected facts first:
  - Facts absent → a **coverage gap**. Ingest the source that holds them.
  - Facts present but not retrieved → a **retrieval problem** (chunking, embedding, query, ranking). See 2.2.
- **Extraction by format:**

| Source and need                      | Package or tool                                     | Note                                                                  |
| ------------------------------------ | --------------------------------------------------- | --------------------------------------------------------------------- |
| Text PDF, running text               | `pypdf` `extract_text()`, PyMuPDF `page.get_text()` | Tables come out as run-together text                                  |
| Text PDF with tables as rows         | `pdfplumber` `page.extract_tables()`                | Detects table structure from the text layer                           |
| Scanned PDF or image (no text layer) | OCR: Tesseract via `pytesseract`                    | Text-layer parsers return empty strings                               |
| Mixed formats, typed elements        | `unstructured` (`partition_pdf`)                    | `hi_res` + `infer_table_structure=True` keeps tables; `fast` does not |
| HTML pages                           | BeautifulSoup / lxml                                | Drop navigation and footers, keep body and headings                   |
| Word, PowerPoint                     | `python-docx`, `python-pptx`                        | Paragraphs, tables, slide text                                        |
| No installs allowed, or SQL-only     | `ai_parse_document`                                 | Built in                                                              |

- **`ai_parse_document`:**
  - Reads PDF, JPG/JPEG, PNG, TIFF, DOC/DOCX and PPT/PPTX, including scanned pages, from `read_files(..., format => 'binaryFile')`.
  - Returns a VARIANT whose `document.elements` lists typed elements (`title`, `section_header`, `table`, `figure`, `page_header`, `page_footer`, `page_number`, ...) with content and confidence.
  - Over the 500-page limit, parse in slices with `pageRange`. Limits belong to the function, not the compute.
- **Filter before you chunk:** running headers, footers and page numbers; web navigation and cookie banners; exact and near-duplicate pages (MinHash); quoted replies and signatures; superseded versions.
- **Do not strip what embeddings need:** lowercasing, punctuation removal and stop-word removal remove meaning. Never drop tables.
- **Versions:**
  - Only current rules matter → ingest only the current version, with an effective-date column.
  - Users ask about past dates → ingest all versions with `effective_from` / `effective_to` and filter each query, in one index.
- **Structure-aware splitters (LangChain):**

| Document                   | Splitter                                                  |
| -------------------------- | --------------------------------------------------------- |
| Markdown with `#` headings | `MarkdownHeaderTextSplitter` (heading text into metadata) |
| HTML with `<h2>`/`<h3>`    | `HTMLHeaderTextSplitter`                                  |
| Running prose              | `RecursiveCharacterTextSplitter`                          |
| Size only                  | `CharacterTextSplitter`, `TokenTextSplitter`              |

- Header splitters are often followed by a size-based splitter so a long section still fits the embedding limit.
- **Tables:** split large tables into row groups that **repeat the header row**. For single-value lookups, write each row as self-describing text (`Model: X200; Weight: 4 kg`).
- **Embedding limit:** input past the limit is **silently truncated**, not split. `databricks-bge-large-en` has a 512-token window (1,024-dimensional output); `databricks-gte-large-en` accepts 8,192 tokens.
- **Measure in the embedding model's tokens.** LangChain's default length function counts characters. `from_tiktoken_encoder` counts OpenAI tokens, which differ. Use `from_huggingface_tokenizer` with the embedding model's tokenizer. Overlap comes out of `chunk_size`, not on top of it.
- **Context budget:** prompt and history + (chunks retrieved × chunk size) + answer reserve ≤ context window. Max chunk size = (window − prompt − answer reserve) ÷ chunks retrieved.
- **Size trade-off:** tiny chunks lose the conditions in the next paragraph; chapter chunks blur many subjects into one vector. A paragraph or section of a few hundred tokens is the usual middle.
- **Overlap:** about 10-20 percent of chunk size. Zero splits boundary facts; half the chunk creates near-duplicates and roughly doubles chunk count.
- **Writing chunks, in order:**
  1. Read raw files from a UC volume as `binaryFile`.
  2. Parse and filter.
  3. Chunk, explode to one row per chunk, add a unique id and metadata columns.
  4. `saveAsTable("catalog.schema.table")`.
  5. Enable change data feed (`delta.enableChangeDataFeed = true`).
  6. Create the Delta Sync index with the id as primary key and the text column as the embedding source.
- **Delta Sync source needs:** a UC Delta table, a unique primary key, and change data feed (standard endpoint). `_id` is reserved. Temp views and files saved to a path cannot be indexed.
- **Metadata columns** (source path, page, product line, effective dates) come back with results for citations and work as filters. Prepending the source to the chunk text gives nothing to filter on.
- **Incremental ingestion:** Auto Loader (`cloudFiles`, `binaryFile`) with a fixed `checkpointLocation` and `trigger(availableNow=True)`. A new checkpoint makes every file look new again.
- **Changed documents:** delete the document's old rows, then append the new chunks (or a scoped `replaceWhere` overwrite). Deletes flow through change data feed to the index.
- **Least-privilege grants** (plus `USE CATALOG` and `USE SCHEMA`):

| Job must                          | Grant                        | Not                         |
| --------------------------------- | ---------------------------- | --------------------------- |
| Read files in a volume            | `READ VOLUME`                | `SELECT`, `WRITE VOLUME`    |
| Create a new chunks table         | `CREATE TABLE` on the schema | `MODIFY` on a catalog       |
| Append to a table it does not own | `MODIFY` and `SELECT`        | `CREATE TABLE`, `APPLY TAG` |

- Run scheduled pipelines as a **service principal** with exactly those grants.
- **Traps:**
  - No retrieval setting returns text that was never ingested. Smaller chunks, more results, hybrid search and re-rankers only change matching and ordering.
  - Answering from the base model's own knowledge is not a fix; it invents your rules.
  - A re-ranker scores relevance, not recency, so it does not fix stale versions.
  - A metadata filter does not remove boilerplate that is in every document.
  - Re-parsing everything with `mode("overwrite")` forces the index to re-embed every chunk.
  - Table maintenance commands work on files, not rows, and never remove an outdated document's chunks.

📖 Full lesson: [Databricks GenAI Engineer: Document Extraction, Filtering and Chunking into Delta Tables](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-document-chunking-delta-tables/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

## 2.2 Retrieval evaluation, advanced chunking and re-ranking

- **Recall@k vs precision@k:** same numerator (relevant chunks in the top k).
  - Recall@k = relevant in top k ÷ all labeled relevant. Did the needed material reach the model?
  - Precision@k = relevant in top k ÷ k. How much of what reached the model is noise?
  - Worked: 5 labeled relevant, top 8 returned, 3 relevant → precision@8 = 0.375, recall@8 = 0.6.
- **Measure at the k the chain passes.** Recall@40 can improve while nothing new reaches the top 4 the LLM reads.
- **As k grows:** recall can only stay or rise; precision usually falls. Expected, not a regression.
- **Rank-aware metrics:**
  - **MRR** averages 1 ÷ rank of the first relevant result. One correct item per question.
  - **NDCG@k** discounts each relevant result by position and accepts graded labels. Several useful items, order matters.
  - Raw similarity scores are not a quality metric.
- **MLflow 3 retrieval judges** (`mlflow.genai.scorers`, run by `mlflow.genai.evaluate`):

| Judge                   | Compares                           | Needs expectations? | Failure means                      |
| ----------------------- | ---------------------------------- | ------------------- | ---------------------------------- |
| `RetrievalRelevance`    | Each chunk vs the request          | No                  | Off-topic or wrong-source chunks   |
| `RetrievalGroundedness` | Response vs retrieved chunks       | No                  | Unsupported claims (hallucination) |
| `RetrievalSufficiency`  | Retrieved chunks vs expected facts | Yes                 | Context lacked needed information  |

- `RelevanceToQuery` (response vs question) and `Correctness` (response vs expected facts, needs expectations) do not read retrieved documents.
- On unlabeled production traces, only `RetrievalRelevance` and `RetrievalGroundedness` can run.
- **Tracing contract:** judges read spans of type `RETRIEVER`. In custom code use `@mlflow.trace(span_type=SpanType.RETRIEVER)` and return a list of `Document`s with `page_content` and metadata (`doc_uri`, `chunk_id`), not one joined string. Autologged framework retrievers are marked automatically.
- **Diagnosis from judge patterns:**

| Pattern                                | Stage at fault | Fix                                                         |
| -------------------------------------- | -------------- | ----------------------------------------------------------- |
| Sufficiency passes, groundedness fails | Generation     | Answer-from-context instructions, citations, stronger model |
| Groundedness passes, sufficiency fails | Retrieval      | Chunking, hybrid search, enrichment, more candidates        |
| Relevance fails, groundedness passes   | Retrieval      | Exact-identifier matching, chunks that name their source    |

- **Evaluation set:** one fixed set of representative questions, used to compare every configuration and rerun after changes.
  - Golden (SME-written): best quality, most expert time.
  - Synthetic (LLM-generated from the corpus, SME-reviewed on a sample): fits when labels are missing and experts are scarce.
  - Logged production questions: real but unlabeled.
  - Label with **expected facts and source document IDs**, not chunk IDs, so the set survives re-chunking.
  - Do not generate a separate set from each candidate's own chunks; it biases each comparison.
- **Advanced chunking:**

| Symptom                                                      | Technique             |
| ------------------------------------------------------------ | --------------------- |
| Small chunks match well, answers miss surrounding conditions | Parent-child          |
| No headings; fixed chunks mix subjects (transcripts)         | Semantic chunking     |
| Chunks don't say which document or product they are from     | Contextual enrichment |

- **Parent-child:** index only children with a `parent_id`; store parents once in a Delta table. Retrieve extra children, keep the best per `parent_id`, then fetch N distinct parents.
- **Semantic chunking:** boundaries where adjacent-sentence similarity drops. Costs extra embedding calls; use structure where it exists. Sub-split oversized chunks to the token limit.
- **Contextual enrichment:** prepend title, product or section to the chunk text **before embedding**. A metadata column alone does not change the vector.
- **Hybrid search:** `query_type="HYBRID"` runs keyword and vector retrieval and merges by **Reciprocal Rank Fusion**. Default is `ANN`; `FULL_TEXT` is keyword-only. Fixes exact identifiers (part numbers, error codes) without breaking natural-language queries.
- **Re-ranking:** a second stage, typically a **cross-encoder** reading query and candidate together. Accurate but costly per candidate, so it only scores tens of first-stage candidates.
  - It can only reorder. Recall over the candidate list is unchanged; judge it by precision@n or NDCG@n on what reaches the LLM.
  - Needed chunk sits deeper than the candidates handed over → widen the candidate set.
- **Built-in reranker:** `reranker=DatabricksReranker(columns_to_rerank=[...])` on `similarity_search`. No new endpoint, index or training.
  - Reads listed columns in order, about the first 2,000 characters. Put the chunk text first, then short fields like the title.
  - Fits RAG agents at low to moderate QPS with a budget of a second or more. Not for millisecond-budget autocomplete; use hybrid search or a lightweight cross-encoder on Model Serving.
- **Traps:**
  - Swapping precision and recall denominators.
  - Missing chunk from the candidates → first-stage fix (hybrid, enrichment, more candidates). Present but ranked too low → reranker.
  - `RetrievalRelevance` can pass on on-topic chunks that still miss the fact. Confirm with `RetrievalSufficiency`.
  - Fine-tuning the embedding model is a last resort after chunking, enrichment, hybrid search and reranking.

📖 Full lesson: [Databricks GenAI Engineer: Retrieval Evaluation, Advanced Chunking and Re-ranking](https://www.savemycert.com/revision/databricks-generative-ai-engineer-associate/databricks-genai-retrieval-evaluation-reranking/?utm_source=github&utm_medium=readme&utm_campaign=databricks-genai-study-guide)

[← Back to the study guide](../README.md)
