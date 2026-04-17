# 1. Large Language Models (LLMs)

Large Language Models are the **foundational engine** of modern Agentic AI systems. Before designing any agentic system in an interview, an interviewer expects you to deeply understand what an LLM is, its constraints, and why those limitations necessitate an entire agent architecture around it.

## 1. The Transformer Architecture

All modern LLMs are built on the **Transformer architecture** (Vaswani et al., 2017). Its primary innovation is the **Self-Attention Mechanism**, which allows the model to weigh the importance of every word relative to every other word in a sequence — simultaneously and in parallel.

* **Key Interview Clarifier:** The attention mechanism is $O(n^2)$ in both memory and compute with respect to sequence length, which is *why* context windows are limited and *why* long-context efficiency is a hard engineering problem.
* **Context Window:** The maximum number of tokens the model can "see" at once. Modern models range from 8K (compact) to 1M+ tokens (Gemini 1.5). Exceeding this limit is a critical architectural failure point in agent design.
* **Parameters:** The learned numerical weights of the model. Larger parameter counts generally improve reasoning quality, at the cost of inference latency and GPU memory.

## 2. Types of Language Models

Understanding the spectrum of model types is essential for the model selection question in any agent design interview.

1. **Base Models:** Pre-trained purely on next-token prediction on internet-scale text. Powerful but unpredictable—they will complete your instruction as if they're continuing a training document, not answering you.
2. **Instruct-Tuned (Chat) Models:** Base models fine-tuned with Supervised Fine-Tuning (SFT) on curated Q&A pairs, then aligned via RLHF (Reinforcement Learning from Human Feedback). These models reliably follow instructions.
3. **Agentic / Tool-Calling Models:** Further fine-tuned specifically to output structured JSON payloads for function calling and to pause generation while awaiting a tool response. Examples: GPT-5.4, Claude Opus 4.7, Gemini 2.5 Pro.

## 2.5 Reasoning Models vs. Standard Models

A new paradigm in LLMs is the distinction between **Reasoning models** (designed for test-time compute) and **Standard models** (optimized for speed and chat).

| Feature | Reasoning Models (e.g., o1, R1) | Standard Models (e.g., GPT-5.4, Claude Opus) |
|---|---|---|
| **Mechanism** | Uses "internal" Chain-of-Thought tokens before responding | Generates final output tokens immediately |
| **Primary Strength** | Complex logic, math, difficult coding, multi-step planning | Speed, creative writing, simple instructions, conversational flow |
| **Latency** | High (seconds to minutes of "thinking" time) | Low (milliseconds to seconds) |
| **Hallucination** | Significantly lower in logical/mathematical tasks | Higher when "forced" to answer complex logic too quickly |

### When to Use What?

* **Use Reasoning Models when:**
    * The task requires high accuracy in logic (e.g., verifying a security exploit).
    * You need an agent to generate a complex plan with 10+ dependencies.
    * The problem is a "needle in a haystack" logic puzzle.
* **Use Standard Models when:**
    * Low latency is critical for user experience.
    * The task is summarization, creative drafting, or simple data extraction.
    * You are building a high-volume, cost-sensitive application.

## 3. Why Agents Need More Than Just an LLM

A standalone LLM, however powerful, has three fundamental limitations that drive the entire agent architecture:

| Limitation | Impact | Agent Solution |
|---|---|---|
| **Knowledge Cutoff** | Cannot access real-time or proprietary data | Retrieval-Augmented Generation (RAG) |
| **No Persistent Memory** | Forgets everything after the context window | Vector DB + Memory Management |
| **Hallucination** | Confidently generates false information | Tool Use + Grounding |

> **Rule of Thumb:** Always articulate this triangle in an interview. The three core pillars of agent design exist precisely to compensate for these three LLM weaknesses.

## 4. Model Selection Trade-offs

When asked "what model would you use?" in an interview, frame your answer around concrete trade-offs:

* **Frontier models** (GPT-5.4, Claude Opus 4.7, Claude Mythos): Maximum reasoning quality; highest latency (~1-3s/request) and highest token cost.
* **Reasoning models** (o1, o3, R1): Extreme logical precision for planning/coding; can take 10s-30s+ to "think".
* **Mid-tier models** (Llama 4 70B, Qwen-2.5 72B): 80-90% of frontier quality at 10x lower cost. Ideal for production at scale.
* **Small models** (Llama 4 Scout, Qwen-2.5 7B): Sub-100ms latency; excellent for routing, classification, and extraction sub-tasks within a multi-agent pipeline.

> **Rule of Thumb:** Use a **Reasoning Model** for the "Controller/Planner" agent that decides the strategy, and **Standard Frontier/Small Models** for executing individual tool steps.

---
**Next Step:** With the engine understood, we need to understand how the LLM represents language as vectors — the basis for memory and semantic retrieval. Proceed to Chapter 2.
