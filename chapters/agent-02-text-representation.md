# 2. Text Representation (Embeddings)

Before an AI agent can search its memory, find relevant documents, or compare concepts, text must be converted into a format a machine can compute with. **Embeddings** are the numerical representation of language and are the fundamental enabler of AI memory.

## 1. What is an Embedding?

An embedding model maps a piece of text — a word, a sentence, or an entire document — to a **fixed-length vector of floating-point numbers** in a high-dimensional space (e.g., 768 or 1536 dimensions).

The key property: **semantic similarity = geometric proximity**. Text with similar meaning lands close together in this vector space, regardless of the exact words used.

* *"The automobile crashed"* and *"The car had an accident"* will have vectors with very high cosine similarity.
* *"Revenue grew 15%"* and *"Pizza is delicious"* will land far apart.

## 2. The Embedding Pipeline

When designing an agent's memory system, the ingestion process follows three steps:

1. **Tokenization:** Break input text into sub-word pieces (tokens). Most tokenizers use Byte-Pair Encoding (BPE). A rule of thumb: ~1 token ≈ 0.75 words in English.
2. **Encoding:** Pass tokens through the embedding model. The model outputs a single vector representation for the full input (for sentence/doc-level tasks, the `[CLS]` token or a mean-pool aggregation is used).
3. **Storage:** Write the resulting vector + its metadata and source text to a Vector Database for retrieval.

## 3. Similarity Metrics

When an agent does a memory lookup, it computes the distance between the query vector and stored document vectors. Two main metrics:

| Metric | Formula (Intuition) | Best For |
|---|---|---|
| **Cosine Similarity** | Angle between vectors | Sentence comparison — direction matters, not magnitude |
| **Dot Product** | Magnitude × alignment | When both similarity and embedding magnitude are meaningful |
| **Euclidean Distance** | Physical distance in space | Rarely preferred in high-dimensional NLP tasks |

> **Key Interview Point:** Cosine similarity is the standard for semantic search because it normalizes for vector magnitude, making it robust to differences in text length.

## 4. Choosing an Embedding Model

* **General-purpose:** `text-embedding-3-large` (OpenAI) — 3072 dimensions, excellent for most tasks. Can be truncated to 256 dimensions with minimal quality loss for cost savings.
* **Multimodal:** Models that embed both text and images into the same vector space (e.g., CLIP, Amazon Titan Multimodal), enabling cross-modal search.
* **Domain-specific fine-tuning:** For specialized corpora (legal, medical), fine-tune an embedding model on in-domain sentence pairs using contrastive loss to significantly improve retrieval precision.

> **Rule of Thumb:** Never blindly use a generic embedding model for a specialized domain. Always evaluate retrieval quality on a domain-specific benchmark (Recall@10) before going to production.

---
**Next Step:** With the mechanism of representation understood, we need to know how LLMs generate text, and the parameters that control this process. Proceed to Chapter 3.
