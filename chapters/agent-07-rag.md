# 7. Retrieval-Augmented Generation (RAG)

RAG is the **dominant architecture** for eliminating LLM hallucinations and granting agents access to proprietary, real-time knowledge. It is almost certainly the most important topic to understand deeply for any Agentic AI system design interview. The core idea: instead of relying on the model's frozen training data, dynamically inject relevant, sourced context into the prompt at inference time.

## 1. Why RAG Exists

The gap between a standalone LLM and a production-grade knowledge agent:

| Problem | Standalone LLM | With RAG |
|---|---|---|
| **Knowledge Cutoff** | Cannot answer about events after training date | Retrieves real-time documents |
| **Proprietary Data** | Has no access to internal company docs | Queries your private Vector DB |
| **Hallucination** | Generates plausible-sounding falsehoods | Grounds answers in cited source documents |
| **Auditability** | No way to trace where an answer came from | Every claim is traceable to a source chunk |

## 2. The Standard RAG Pipeline

![Standard RAG Pipeline](assets/agent-rag-pipeline.png)


**Step 1 — Retrieval:** The user's query is embedded. The Vector DB runs ANN search and returns the top-K most semantically relevant document chunks (typically K=5–20).

**Step 2 — Augmentation:** The retrieved chunks are injected into the LLM's context window, typically formatted as:
```
Context Document 1: [text from chunk]
Context Document 2: [text from chunk]
...
Question: [user query]
Answer only using the context above. Cite sources by document number.
```

**Step 3 — Generation:** The LLM reads the injected context and generates an answer grounded strictly in the provided documents, including citations.

## 3. Advanced RAG Techniques

### Chunking Strategies
The quality of retrieval is highly sensitive to how documents are split. See Chapter 5 for the full breakdown. The key principle: **chunk for retrieval precision, but make each chunk self-contained enough to be understood in isolation**.

### Re-Ranking (Critical for Precision)
ANN search optimizes for speed, not precision. Add a **Cross-Encoder Re-Ranker** as a second stage:
1. Vector DB retrieves candidate top-50 chunks (fast, approximate).
2. Re-ranker scores each of the 50 against the query one-by-one (slower, precise).
3. Only top-5 are passed to the LLM.

This two-stage approach achieves **high recall** from the Vector DB and **high precision** from the Re-Ranker, dramatically improving answer quality.

### Query Rewriting / HyDE
Users often ask vague queries. Before embedding and retrieval:
* **Query Rewriting:** Use the LLM to rewrite the user's query into a longer, more specific form that is more likely to match document vocabulary.
* **HyDE (Hypothetical Document Embeddings):** Ask the LLM to generate a *hypothetical document* that would answer the query, then embed *that* hypothetical document for retrieval. This closes the vocabulary gap between query style and document style.

### GraphRAG: Multi-Hop Retrieval
When questions require traversing *relationships* across multiple documents (e.g., "What are the risks for funds exposed to companies that Company A acquired?"), standard RAG fails because no single chunk holds the full answer.

**GraphRAG** maintains a **Knowledge Graph** of entities and their relationships, extracted from the document corpus. The retrieval stage traverses the graph to collect the relevant entity chains, then passes the assembled evidence to the LLM.

> **When to Propose GraphRAG:** Only when your use case involves complex relational reasoning across many documents. The infrastructure overhead is significant — knowledge graph extraction, storage (Neo4j), and traversal logic.

## 4. Agentic RAG

In a multi-step agent, RAG is not a single call but a **repeated tool within the agent loop**. The agent decides *when* to search, *what* to search for, and *how many* times to search based on its current state:

```
Agent Thought: "I need Q3 revenue for Company A, then compare to Q2."
Tool Call 1: search_docs(query="Company A Q3 2024 revenue")
Tool Call 2: search_docs(query="Company A Q2 2024 revenue")
Agent Thought: "I now have both figures. I can compute the comparison."
```

This multi-step search pattern allows the agent to progressively refine its context — critical for complex research or analysis tasks.

---
**Next Step:** With memory and context established, the final layer of agent capability is taking *actions* in the real world through Tool Use. Proceed to Chapter 8.
