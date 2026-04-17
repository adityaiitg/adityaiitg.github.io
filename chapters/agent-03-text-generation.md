# 3. Text Generation

Text Generation is the core capability that makes LLMs useful. As an AI system designer, you must understand the controls available to you — not just *that* the model generates text, but *how* to constrain, steer, and structure that generation for agent workflows.

## 1. The Autoregressive Loop

Modern LLMs generate text **one token at a time**. At each step, the model:
1. Looks at the full history of prior tokens (the context window).
2. Computes a probability distribution over its entire vocabulary (~50,000 tokens).
3. Samples one token from that distribution.
4. Appends it to the context and repeats.

This loop is why LLMs cannot correct earlier tokens — they are committed to everything generated so far.

## 2. Controlling Generation: Key Hyperparameters

In an agent design interview, you will often need to justify why a specific generation setting was chosen. Understand these parameters deeply.

| Parameter | Range | Effect |
|---|---|---|
| **Temperature** | 0.0 – 2.0 | Higher = more random. `0.0` = greedy/deterministic |
| **Top-K** | 1 – 100+ | Only sample from the K most probable next tokens |
| **Top-P (Nucleus)** | 0.0 – 1.0 | Sample from the smallest set of tokens summing to probability P |
| **Max Tokens** | Any integer | Hard limit on output length. Critical to set for tool-calling to prevent infinite loops |
| **Frequency Penalty** | -2.0 – 2.0 | Penalizes tokens that appear frequently in the output — reduces repetition |

> **Key Agentic Rule:** For tool-calling and JSON generation, **always set `temperature=0.0`**. Any randomness risks malformed JSON payloads and broken tool calls. Reserve higher temperatures only for creative output tasks.

## 3. Structured Output Generation

Plain text generation is insufficient when agents need to interface with APIs and databases. Structured output generation forces the model to strictly conform to a predefined **JSON Schema**, completely eliminating malformed payloads.

```json
{
  "type": "object",
  "properties": {
    "tool_name": { "type": "string" },
    "query": { "type": "string" }
  },
  "required": ["tool_name", "query"]
}
```

This is implemented via **constrained decoding** — at each generation step, the sampling distribution is mathematically masked to only allow tokens consistent with the schema at that point.

## 4. Context Window Management

Context windows are finite and expensive. Knowing how to manage them is a key senior-level design skill.

* **Summarization:** Periodically summarize old conversation turns using a smaller, cheaper model, replacing the raw history with the compressed summary.
* **Sliding Window:** Evict the oldest messages from the context, keeping only the last N turns.
* **TOON (Token-Oriented Object Notation):** A modern optimization format that replaces verbose JSON with indentation-based structures, saving 30–60% of tokens when passing structured data like tool schemas or agent memory. Critically useful in multi-agent systems where agents pass state to each other.

---
**Next Step:** Understanding generation gives us control over the model's output. Now we must understand how to physically deploy these models into production systems. Proceed to Chapter 4.
