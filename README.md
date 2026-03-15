# DOMAIN ADAPTIVE AGENTIC LEGAL AI

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
