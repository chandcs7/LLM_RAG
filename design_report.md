**Concept Design Report for RAG Pipeline Process***
1. Chunking Strategy: Recursive Character Splitting
Methodology: We utilized the RecursiveCharacterTextSplitter with a chunk_size of 1000 and an overlap of 100.
Justification: Financial reports (10-K filings) are highly structured with nested sections and tables. 
Recursive splitting prioritizes break points at double newlines and paragraph ends, which preserves the semantic context of financial disclosures. 
The 10% overlap ensures that data spanning across page or paragraph boundaries (common in balance sheets) is not lost, allowing the retrieval engine to capture complete data points.

2. LLM Choice: Microsoft Phi-2 (Quantized)
Selection: microsoft/phi-2 
Optimization: Loaded with BitsAndBytesConfig 4-bit quantization.
Justification: Given the constraint to run on CPU, a large model (like Llama-3 or GPT-4) would be too slow or crash the session.
Phi-2 provides "textbook quality" reasoning at a small scale. Quantization reduces the memory footprint from ~10GB to ~2.5GB,
significantly improving execution speed on CPU while maintaining the precision required to extract exact dollar amounts from the context.

3. Out-of-Scope Handling
Logic: Strict citation-based prompt engineering and post processing the output answer to avoid model hallucination
Mechanism: The prompt explicitly instructs the model to return "Not Answerable" if the context does not contain the answer.
Citation Enforcement: By mandating a specific citation format [Filename, Section, Page X], the model is forced to ground its response in the provided text.
If it cannot find a specific source for the data, it fails the internal validation and triggers the "Not Answerable" response,
effectively preventing hallucinations when asked questions about future years or companies not in the database.
