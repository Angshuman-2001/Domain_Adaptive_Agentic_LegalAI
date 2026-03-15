------------------------------------
# DOMAIN-ADAPTIVE AGENTIC LEGAL AI
------------------------------------

### About the project

Standard Retrieval-Augmented Generation (RAG) systems often fail in strict domains like law due to hallucinations and irrelevant context retrieval. This project introduces a stateful, Agentic Legal AI designed to navigate complex regulatory frameworks (like the DPDP Act 2023 and IT Act 2000) with deterministic accuracy. Built using LangGraph, the architecture moves beyond linear pipelines by orchestrating a cyclical, self-correcting workflow of specialized agents (Router, Grader, Generator, and Auditor).

By integrating Hybrid Search (FAISS + BM25) via Reciprocal Rank Fusion (RRF) and a Cross-Encoder Reranker, the system tackles the 'lost-in-the-middle' problem by distilling 30 chunks down to the absolute top 5 most relevant legal facts. Furthermore, it employs a strict Hallucination Auditor loop that explicitly checks the generated response against grounded context, automatically triggering a re-generation if inconsistencies are found. Deployed on a resource-constrained T4 GPU using 4-bit NF4 Quantization (Qwen 2.5), this system bridges the gap between theoretical LLM capabilities and reliable, production-grade enterprise compliance.


```
                            ARCHITECTURE OF THE PROJECT
                         -----------------------------------

                                 [ USER QUERY ]
                                         │
                     "What is the penalty for a data breach?"
                                         │
                                         ▼
                             ┌──────────────────────────┐
                             │ AGENT 1: SEMANTIC ROUTER │
                             │   (Intent Classification │
                             │   via Pydantic Schema)   │
                             └────┬─────────┬────────┬──┘
                                  │         │        │
             ┌────────────────────┘         │        └────────────────────┐
             ▼                              ▼                             ▼
    (Intent: Out-of-Domain)        (Intent: DPDP Act)             (Intent: IT Act)
             │                              │                             │
             │                              ▼                             ▼
             │                 ┌────────────────────────┐    ┌────────────────────────┐
             │                 │   DPDP HYBRID STORE    │    │    IT HYBRID STORE     │
             │                 │  - FAISS (Dense)       │    │  - FAISS (Dense)       │
             │                 │  - BM25 (Sparse)       │    │  - BM25 (Sparse)       │
             │                 └────────────┬───────────┘    └────────────┬───────────┘
             │                              │                             │
             │                       (Top 30 Chunks)               (Top 30 Chunks)
             │                              │                             │
             │                              └──────────────┬──────────────┘
             │                                             │
             │                                             ▼
             │                              ┌──────────────────────────────┐
             │                              │   RECIPROCAL RANK FUSION     │
             │                              │    & CROSS-ENCODER RERANKER  │
             │                              │  Refines 30 Chunks -> Top 5  │
             │                              └──────────────┬───────────────┘
             │                                             │
             │                                             ▼
             │                              ┌──────────────────────────┐
             │                              │ AGENT 2: DOCUMENT GRADER │
             │                              │  (Evaluates Top 5 Chunks │
             │                              │   and drops irrelevant)  │
             │                              └──────────────┬───────────┘
             │                                             │
             │                                     ┌───────┴───────┐
             │                                     ▼               ▼
             │                           [If ALL 5 Irrelevant]  [If ≥1 Relevant]
             │                                     │               │
             │                                     │               ▼
             │                                     │    ┌──────────────────────────┐ <──────┐
             │                                     │    │ AGENT 3: GENERATOR AGENT │        │
             │                                     │    │   (Qwen 2.5 on T4 GPU)   │        │
             │                                     │    └────────────┬─────────────┘        │
             │                                     │                 │                      │
             │                                     │                 ▼                      │
             │                                     │    ┌──────────────────────────┐        │
             │                                     │    │  AGENT 4: AUDITOR AGENT  │        │
             │                                     │    │ (Checks if the answer is │        │
             │                                     │    │ grounded in source docs, │        │
             │                                     │    │  matches the user query, │        │
             │                                     │    │  and is not hallucinated)│        │
             │                                     │    │     (Max Retries: 2)     │        │
             │                                     │    └────────────┬─────────────┘        │
             │                                     │                 │                      │
             │                                     │          ┌──────┴──────┐               │
             │                                     │          ▼             ▼               │
             │                                     │    [Passes Audit] [Fails Audit]        │
             │                                     │          │             │               │
             │                                     │          │             └───────────────┘
             │                                     │          │         (Triggers Re-generation)
             ▼                                     ▼          ▼
    ┌──────────────────────────┐                   │    ┌──────────────────────────────────┐
    │   SMART FALLBACK AGENT   │<──────────────────┘    │      GENERATED LEGAL ANSWER      │
    │ (Gracefully deflects     │                        │----------------------------------│
    │  query & suggests        │                        │ "According to the DPDP Act..."   │
    │  external sources)       │                        └──────────────────────────────────┘
    └────────────┬─────────────┘
                 │
                 ▼
    ┌──────────────────────────────────┐
    │      FINAL FALLBACK MESSAGE      │
    │----------------------------------│
    │ "I am specialized strictly in... │
    │ so this is out of my domain."    │
    └──────────────────────────────────┘
```

### Modle UI & output demonstrations

Below are the actual outputs generated by the Domain-Adaptive Agentic Legal AI. The system features an interactive UI designed for legal professionals. It successfully processes complex, plain-text queries, triggers the LangGraph state-machine, and synthesizes accurate, plain-English legal summaries grounded strictly in the DPDP and IT Acts.

**Demo 1: Successful legal context retrieval & generation**

a)
![image alt](https://github.com/Angshuman-2001/Domain_Adaptive_Agentic_LegalAI/blob/25c3814d5b0bc7bff0c03b10c3ab077dba7f430e/correct.png)
![image alt](https://github.com/Angshuman-2001/Domain_Adaptive_Agentic_LegalAI/blob/788894a2dcbabf874141109609c15fd8c85c321f/correct2.png)
**Demo 2: Smart fallback mechanism (out-of-domain query)**

a)
![image alt](https://github.com/Angshuman-2001/Domain_Adaptive_Agentic_LegalAI/blob/5d67b0e457c0559e7d9c47b9932ea33823ca65d4/fallback.png)

b)
![image alt](https://github.com/Angshuman-2001/Domain_Adaptive_Agentic_LegalAI/blob/29ff39f663327e9f7560fc2d75c63ff6def5324d/fallback2.png)

c)
![image alt](https://github.com/Angshuman-2001/Domain_Adaptive_Agentic_LegalAI/blob/bbcd771e9a50e25385543a0d28e91f5ed6cf47bd/fallback3.png)
![image alt](https://github.com/Angshuman-2001/Domain_Adaptive_Agentic_LegalAI/blob/bbcd771e9a50e25385543a0d28e91f5ed6cf47bd/fallback4.png)


## Execution Environment & Generated Artifacts

**Note on Execution & Storage:**
This project was developed, tested, and executed in **Google Colab** utilizing the NVIDIA T4 GPU. To ensure data persistence and prevent the need to re-process massive legal documents on every runtime initialization, the environment was explicitly mounted to a dedicated **Google Drive** folder. 

During the execution of step 1 (Data Ingestion and Preprocessing), the system automatically creates and saves the following artifacts permanently to the drive:

* **`DPDP_FAISS_Index/` & `IT_FAISS_Index/`**: The persistently saved Dense Vector embeddings generated by the BAAI/bge-large model.
* **`docstore.pkl`**: The persistent document store containing the actual textual chunks and metadata. This is critical for the Parent-Child chunking architecture to map dense vectors back to their original human-readable text.
* **`bm25_DPDP_Index.pkl` & `bm25_IT_Index.pkl`**: Serialized sparse keyword indices required for the Hybrid Search engine.

By storing these artifacts locally in Google Drive, the system achieves instant cold-starts in subsequent runs, completely bypassing the heavy document chunking and embedding phases and saving critical GPU compute time.
