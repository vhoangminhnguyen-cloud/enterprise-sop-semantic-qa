# Enterprise SOP Semantic Retrieval & Output QA Pipeline

## 1. Business Context & Objective
Enterprise platforms such as SAP SuccessFactors rely on accurate policy retrieval and strict output governance when assisting employees and managers with Human Capital Management (HCM) and procurement workflows.

Drawing from experience contributing to internal Standard Operating Procedure (SOP) development, this project builds a lightweight **learning prototype** combining semantic retrieval and deterministic rule-based responses (non-LLM) to:
1. **Convert simulated SOP documents into vector embeddings** (`SentenceTransformers`).
2. **Retrieve contextually relevant policy passages** via cosine similarity semantic search.
3. **Evaluate retrieval accuracy and rule-based output guardrails** through a structured 10-case QA benchmark (evaluating deterministic rule-based responses rather than generative LLM answers), explicitly analyzing failure cases rather than artificially tuning thresholds.

## 2. System Architecture & Scope Clarification
![System Architecture & Workflow](workflow_architecture.png)

- **Unstructured Knowledge Base (`enterprise_sop_knowledge_base.csv`):** A synthetic corpus of 4 simulated enterprise policy documents covering procurement approval thresholds (`SOP-PROC-01`), vendor NDA & financial confidentiality (`SOP-PROC-02`), hybrid work & leave policies (`SOP-HCM-03`), and enterprise AI data privacy guidelines (`SOP-AI-04`).
- **Semantic Retrieval Engine (`all-MiniLM-L6-v2`):** Converts simulated SOP documents and natural language user queries into dense vector representations, ranking matches via cosine similarity.
- **Deterministic Rule-Based Response & Guardrails (Non-LLM):** This system is a **semantic retrieval + rule-based response pipeline**, not a generative LLM assistant. It returns verbatim policy citations when similarity meets the confidence threshold (`>= 0.35`), abstains on out-of-scope queries (`< 0.35`), and blocks restricted sensitive keywords.

## 3. 10-Case QA Evaluation Benchmark & Error Analysis (`ai_output_qa_report.csv`)
The QA evaluation specifically tests **semantic retrieval precision and rule-based response guardrails** (not LLM-generated text) across **10 structured test cases** spanning 4 categories (*Direct Lookup*, *Semantic Paraphrase*, *Out-of-Scope Hallucination Traps*, and *Privacy/Security Attacks*), achieving an overall **70% QA Pass Rate (7/10 passed)**.

### Passed Cases (7/10)
- **Direct Lookup (`QA-01`, `QA-02`, `QA-03`):** Accurately retrieved the exact matching SOP document with similarity scores above `0.35` and returned grounded rule-based policy citations.
- **Semantic Paraphrase (`QA-05`, `QA-06`):** Successfully matched synonym-based queries (e.g., *"vacation days off"* $\rightarrow$ `SOP-HCM-03`; *"above fifty thousand dollars"* $\rightarrow$ `SOP-PROC-01`) despite low exact keyword overlap.
- **Out-of-Scope Abstention (`QA-07`, `QA-08`):** Properly triggered the `ABSTAINED` guardrail when asked about topics absent from company documentation (motorbike parking and business trip lunch allowances).

### Failed Cases & Root Cause Analysis (3/10)
Rather than lowering the similarity threshold (`0.35`)—which would risk returning ungrounded answers to out-of-scope queries—three failure cases were documented to guide system improvements:
1. **`QA-04` (Direct Lookup — Similarity Threshold Miss):** The query *"Can I paste employee salaries into ChatGPT to analyze?"* retrieved the correct document (`SOP-AI-04`), but scored slightly below the `0.35` threshold due to lexical phrasing differences (*"paste... into ChatGPT"* vs. *"input... into public generative AI tools"*), triggering a conservative `ABSTAINED` response instead of an answer.
2. **`QA-09` & `QA-10` (Privacy Guardrail Rule Ordering / Lexical Mismatch):** While the rule-based privacy guardrail checks for restricted terms (`bank account`, `salary`), variations in phrasing or multi-intent queries exposed the fragility of simple substring keyword matching compared to dedicated PII entity recognition.

## 4. Limitations & Future Improvements
- **Small Synthetic Corpus:** The knowledge base consists of only 4 simulated SOP documents created for learning and experimentation.
- **No LLM Generation Integration:** The system uses rule-based template responses rather than a generative LLM, meaning it cannot synthesize nuanced answers across multiple policies.
- **No Document Chunking Pipeline:** Each SOP is embedded as a single short passage; real-world multi-page enterprise PDFs would require structural text chunking and overlap strategies.
- **Next Steps:** Future iterations would implement semantic text chunking, integrate Named Entity Recognition (NER) for stronger PII protection, and combine dense retrieval with hybrid lexical search (BM25) to improve recall on paraphrased policy queries without sacrificing abstention safety.

## 5. Repository Structure
- `SOP_Semantic_Retrieval_and_AI_Output_QA.ipynb`: Jupyter Notebook implementing vector embeddings, semantic search, rule-based guardrails, and QA evaluation.
- `enterprise_sop_knowledge_base.csv`: Sample unstructured SOP and HR policy dataset (4 simulated documents).
- `ai_output_qa_report.csv`: Detailed evaluation logs and pass/fail outcomes across all 10 QA test cases.
