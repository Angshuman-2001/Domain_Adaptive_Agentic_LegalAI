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

**Demo 1: Successful cegal context retrieval & generation**


**Demo 2: Smart Fallback Mechanism (Out-of-Domain Query)**
*(Yahan wo screenshot paste karein jisme model ne query reject karke fallback message diya tha)*
