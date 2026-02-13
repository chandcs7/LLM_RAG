**Justification of LLM Model and RAG Pipeline**

1) Document ingestion choice (PyMuPDF)
Choice: PyMuPDF (fitz) for extracting text page-wise.
Justification:
10-K PDFs are large and multi-column.PyMuPDF is faster and more reliable than many PDF readers.Page-wise extraction preserves page number metadata, enabling citations in final answers.
It supports block-based extraction, which improves reading order and reduces column mixing.

2) Cleaning strategy (remove headers patterns + formatting fixes + table extract with mark down strings)
Choice: Remove repeated headers/footers + normalize whitespace + fix hyphenated word breaks.
Justification:
Repeated page headers and pollute embeddings and reduce retrieval quality.
Cleaning and extracting tables separately improves semantic search because embeddings represent meaning, and noise reduces similarity accuracy.
Fixing hyphenated words prevents vocabulary fragmentation (e.g., “manufac- turing”).

3) Metadata preservation (document, page, section)
Choice: Attach metadata per chunk:
document,page,section (Item 1A, Item 7, etc.)
Justification:
Required for traceability, auditing and user trust.Enables accurate citations.Allows filtering retrieval by document or section (Apple vs Tesla)

4) Chunking design (1000 chars + overlap 100)

Choice: chunk_size=1000, chunk_overlap=100.
Justification:10-K content is dense; small chunks lose context and increase retrieval noise.
Large chunks increase token cost and can dilute relevance.Overlap preserves continuity across paragraph boundaries

5) Vector store choice (Chroma)

Choice: Chroma for vector storage.
Justification:
Open-source and very easy to integrate with metadata.Supports persistent storage. Good for rapid prototyping and local deployment.

6) Embedding model choice
Choice: all-MiniLM-L6-v2 (or similar) for embeddings.
Justification:Produces strong semantic embeddings for retrieval tasks.
Lightweight and runs without paid APIs.Efficient for large PDF chunking workloads.

7) Retrieval strategy (top-k + reranking)
Choice: Two-stage retrieval:
Bi-encoder similarity search (Chroma)
Cross-encoder reranking
Justification:
Bi-encoders are fast but sometimes return “topic-similar” chunks instead of “answer-bearing” chunks.Cross-encoders score query+chunk jointly, improving precision.
This is especially important for 10-Ks because many sections share similar terms (risk, revenue, cash flow).This is one of the constraint faced during building RAG pipeline

8) Generation model choice (Microsoft Phi2)
Justification:
Small efficient LLM model. lower compute and memory footprint. feasible to run on Google colab CPU

9) Memory optimization choices (why your pipeline didn’t crash)
Choice:chunking,batching embeddings,limiting retrieved context,model loading based on 4 bit-quantization 
Justification:
Prevents CPU OOM during embedding and generation.Ensures stable performance for large 10-K documents.
Maintains consistent latency for interactive chatbot use.

10) Also, incorporated conditions to prevent model hallucinations by post processing the output
