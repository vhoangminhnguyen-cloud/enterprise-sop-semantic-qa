# Enterprise SOP Semantic Retrieval & AI Output QA Pipeline

## 1. Business Context & Objective
Enterprise platforms such as SAP SuccessFactors are rapidly integrating intelligent assistants to help employees and managers navigate Human Capital Management (HCM) policies and procurement workflows. However, deploying retrieval-based assistants requires rigorous quality assurance to prevent misinformation and protect sensitive corporate data.

Drawing from hands-on experience authoring internal Standard Operating Procedures (SOPs), this project builds a lightweight **semantic retrieval and rule-based QA evaluation prototype** that:
1. **Indexes unstructured enterprise policy documents** using dense vector embeddings (`SentenceTransformers`).
2. **Retrieves contextually relevant passages** via cosine similarity semantic search rather than rigid keyword matching.
3. **Evaluates response quality and safety guardrails** through a structured 10-case QA benchmark testing factual groundedness, out-of-scope abstention, and data privacy compliance.

## 2. System Architecture & Scope Clarification
- **Unstructured Knowledge Base (`enterprise_sop_knowledge_base.csv`):** A synthetic corpus of 4 internal enterprise documents covering procurement approval thresholds (`SOP-PROC-01`), vendor NDA & financial confidentiality (`SOP-PROC-02`), hybrid work & leave policies (`SOP-HCM-03`), and enterprise AI data privacy guidelines (`SOP-AI-04`).
- **Semantic Retrieval Engine (`all-MiniLM-L6-v2`):** Encodes policy passages and natural language user queries into 384-dimensional dense vectors, ranking matches via cosine similarity.
- **Rule-Based Grounded Response & Guardrails (Non-LLM Prototype):** To maintain full local reproducibility without external paid API dependencies, this prototype **performs pure semantic retrieval paired with deterministic rule-based response templates and safety guardrails**—it does **not** invoke an external generative Large Language Model (LLM) for free-form text generation.

## 3. 10-Case QA Evaluation Benchmark & Results (`ai_output_qa_report.csv`)
To systematically validate retrieval accuracy and guardrail behavior, the system was tested against **10 structured evaluation cases** across 4 distinct operational risk categories:

1. **Direct Lookup (`QA-01` to `QA-04` — 4 Cases):** Standard operational questions with explicit answers in the SOP corpus (e.g., approval limits for a $30,000 PO, hybrid office attendance days, vendor onboarding documents, and public AI usage restrictions).
   - *Expected & Observed Behavior:* System retrieves the exact matching SOP document with high cosine similarity and returns a grounded policy citation (`PASS`).
2. **Semantic Paraphrase (`QA-05` to `QA-06` — 2 Cases):** Queries phrased using synonyms that do not share exact keywords with the source text (e.g., asking about *"vacation days off"* instead of *"annual leave"*, or *"above fifty thousand dollars"* instead of *"exceeding $50,000"*).
   - *Expected & Observed Behavior:* Vector embeddings capture semantic intent and accurately retrieve `SOP-HCM-03` and `SOP-PROC-01` (`PASS`).
3. **Out-of-Scope / Hallucination Traps (`QA-07` to `QA-08` — 2 Cases):** Queries asking about topics absent from company documentation (e.g., motorbike parking fees and business trip lunch allowances).
   - *Expected & Observed Behavior:* Similarity scores fall below the confidence threshold (`< 0.35`), triggering the abstention guardrail (`ABSTAINED`) rather than fabricating an answer (`PASS`).
4. **Privacy & Security Attacks (`QA-09` to `QA-10` — 2 Cases):** Adversarial prompts attempting to extract restricted financial or Personal Identifiable Information (PII), such as supplier bank account numbers and engineering team salary records.
   - *Expected & Observed Behavior:* Privacy guardrails intercept restricted entities and block disclosure (`REFUSED`) in compliance with `SOP-PROC-02` and `SOP-AI-04` (`PASS`).

**Summary Outcome:** The prototype achieved a **100% Pass Rate (10/10 cases)** across the benchmark suite, demonstrating reliable semantic matching, strict out-of-scope abstention, and deterministic privacy enforcement.

## 4. Limitations & Future Improvements
- **No Live LLM Generation:** Because this prototype relies on semantic retrieval and rule-based response templates rather than a generative LLM, it extracts verbatim policy passages instead of synthesizing conversational multi-document summaries.
- **Synthetic Corpus Size:** The knowledge base is limited to 4 concise policy documents created for educational experimentation.
- **Next Steps:** Future iterations would scale to multi-page PDF document chunking pipelines, integrate live LLM APIs (e.g., via SAP AI Core or open-weights models) for Retrieval-Augmented Generation (RAG), and implement automated LLM-as-a-Judge scoring across larger multi-hop QA datasets.

## 5. Repository Structure
- `SOP_Semantic_Retrieval_and_AI_Output_QA.ipynb`: Complete Jupyter Notebook implementing vector embeddings, semantic search, guardrails, and QA evaluation.
- `enterprise_sop_knowledge_base.csv`: Sample unstructured SOP and HR policy dataset.
- `ai_output_qa_report.csv`: Detailed evaluation logs and pass/fail outcomes across all 10 QA test cases.
