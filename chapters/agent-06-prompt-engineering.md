# 6. Prompt Engineering

Prompt Engineering is the **software architecture of the LLM era**. For agentic systems, the prompt is not a simple question — it is a complete behavioral contract that defines the agent's persona, capabilities, memory, and safety constraints. Getting this wrong renders even the most sophisticated retrieval and tool infrastructure useless.

## 1. The Anatomy of an Agent Prompt

A production agent system prompt typically contains these components **in order**:

1. **Persona & Role Definition:** What the agent *is*.
2. **Capability Boundaries:** What the agent *can* and — critically — *cannot* do.
3. **Output Format Instructions:** Exactly how the agent must structure its response (e.g., JSON schema, markdown format).
4. **Tool Descriptions:** A compressed, precise schema for every available tool.
5. **Few-Shot Examples:** 2–5 worked examples of ideal tool-call trajectories.
6. **Current Memory/Context:** Retrieved RAG documents, conversation history.
7. **User Query:** The actual user input (always last).

> **Key Interview Point:** Order matters. Placing constraints early in the system prompt makes them harder for the model to "forget" when the context gets long.

## 2. Core Prompting Techniques

### System Prompts & Personas
The system prompt is the highest-authority instruction layer. For agents, focus on **negative constraints** — what the agent must *not* do — as models are more compliant with explicit prohibitions than vague guidance.

*Example:* `"You are a senior SQL engineer. You ONLY output valid, executable SQL. You are STRICTLY FORBIDDEN from generating DROP, TRUNCATE, or DELETE statements without an explicit WHERE clause."`

### Few-Shot Prompting
The single highest-ROI technique for improving agent reliability. By injecting 2–5 examples of ideal tool-call trajectories directly into the prompt, you implicitly teach the model the expected output format, parameter naming convention, and reasoning style — without any fine-tuning.

### Chain-of-Thought (CoT)
Instruct the model to reason step-by-step before committing to an answer or tool call. This drastically reduces hallucination on multi-step reasoning because the model "shows its work" in intermediate tokens before arriving at a conclusion.

*Add to system prompt:* `"Before calling any tool, articulate your reasoning step by step in a <think> tag."`

## 3. The ReAct Framework (Reason + Act)

**ReAct** is the foundational prompt design pattern for autonomous agents. The prompt constrains the model into a strict cyclical cadence:

```
Thought: I need to find Q3 revenue for Company A. I'll search the filing database.
Action: search_documents(query="Company A Q3 2024 revenue", top_k=5)
Observation: [Document: "Company A reported $2.3B in Q3 2024 revenue..."]
Thought: I now have the answer. No further tool calls needed.
Final Answer: Company A's Q3 2024 revenue was $2.3 billion.
```

This pattern makes the agent's reasoning fully **auditable** — every step is logged and inspectable by your observability platform.

## 4. Prompt Design Trade-offs

| Design Choice | Trade-off |
|---|---|
| Detailed system prompt | Higher reliability vs. more tokens per request (higher cost) |
| Many few-shot examples | Better format adherence vs. fewer tokens left for reasoning |
| Strict output constraints | Fewer hallucinations vs. less flexibility for edge cases |
| Chain-of-Thought enabled | Better accuracy vs. 2-3x more output tokens generated |

> **Rule of Thumb:** In early development, make the prompt as detailed as possible. In production optimization, profile token usage per call and trim every non-essential sentence.

---
**Next Step:** With well-engineered prompts, the agent can reason correctly. The final layer is connecting it to external knowledge and real-time data — the RAG pipeline. Proceed to Chapter 7.
