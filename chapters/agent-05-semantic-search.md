# 5. Semantic Search

Semantic search is how an AI agent retrieves relevant information from a knowledge base — its **long-term memory**. Unlike traditional keyword search, semantic search retrieves documents based on *meaning*, enabling the agent to find what the user intends rather than exactly what they typed.

## 1. Keyword Search vs. Semantic Search

Understanding this distinction is a classic interview entry point.

| Approach | Method | Strength | Weakness |
|---|---|---|---|
| **Keyword (Lexical)** | Exact term match (BM25, TF-IDF) | Fast, precise on exact terms | Fails on synonyms, paraphrases, intent |
| **Semantic (Dense)** | Cosine similarity on embeddings | Captures meaning and intent | Can miss exact term matches (e.g., product codes, IDs) |
| **Hybrid** | Weighted combination of both | Best of both worlds | Additional complexity; requires alpha-weight tuning |

> **Rule of Thumb:** Pure keyword search fails on natural language queries. Pure semantic search fails on ID lookups and highly niche acronyms. **Always propose Hybrid Search in a system design interview when retrieval quality matters.**

## 2. The Vector Database Architecture

To store and retrieve embeddings at scale, agents rely on **Vector Databases** (e.g., Pinecone, Qdrant, Weaviate, Milvus, pgvector).

### Ingestion Pipeline
1. **Chunking:** Break source documents into overlapping chunks (e.g., 512 tokens, 50-token overlap). Overlap preserves semantic context across chunk boundaries.
2. **Embedding:** Pass each chunk through the embedding model to produce a vector.
3. **Indexing:** Store the vector, source text, and metadata (document ID, timestamp, source URL) in the Vector DB.

### Query Pipeline
1. **Embed the query** using the same embedding model (critical — model must match).
2. **Approximate Nearest Neighbor (ANN) Search:** The Vector DB finds the top-K most similar vectors using **HNSW** (Hierarchical Navigable Small World) graphs — this runs in microseconds over millions of vectors.
3. **Return** the source chunks associated with those top-K vectors to the agent.

## 3. Chunking Strategies

Chunking is a surprisingly high-impact design decision. Wrong chunk sizes cause poor retrieval quality.

* **Fixed-size chunking:** Simple and fast; may cut sentences mid-way, losing context.
* **Sentence/paragraph-aware chunking:** Respect natural language boundaries. Higher quality, slightly more complex.
* **Hierarchical chunking:** Store both small chunks (for precision) and their parent document summaries (for context). Retrieve the small chunk but send the parent context to the LLM.

> **Key Interview Point:** The chunk size governs a precision/recall trade-off. Smaller chunks → higher precision but less context for the LLM. Larger chunks → richer context but lower precision and wasted tokens.

## 4. Re-Ranking

A Vector DB ANN search retrieves candidates quickly but imprecisely. A **Re-Ranker** model (Cross-Encoder architecture) then performs a slow, careful comparison of each candidate directly against the query.

**Two-stage retrieval:**
1. **Recall stage:** ANN retrieves the top-50 candidates rapidly.
2. **Precision stage:** Re-Ranker re-scores and re-orders those 50, returning only top-5.

This gives the accuracy of an exhaustive pairwise comparison with only 50 pairs instead of millions.

## 5. GraphRAG: Beyond Vector Search

Standard Vector DBs treat each document in isolation. **GraphRAG** fuses a **Knowledge Graph** with vector retrieval to enable multi-hop relational reasoning.

A knowledge graph extracts entities and relationships from documents:
```
Company A → acquired by → Company B → invested in by → Fund C
```

When an agent needs to answer "Which fund has exposure to companies acquired by Company B?", Vector search fails because no single document contains the full chain. GraphRAG traverses the entity graph to construct the answer.

> **When to propose GraphRAG:** Any time the user's queries require connecting dots *across* multiple documents or involve complex relational logic. Overkill for simple Q&A use cases.

---
**Next Step:** Retrieval gives the agent memory. But without knowing how to design the instructions that control it, the retrieved context is wasted. Proceed to Chapter 6: Prompt Engineering.
