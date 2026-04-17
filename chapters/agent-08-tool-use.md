# 8. Tool Use & Execution Engines

Tool Use (Function Calling) is the mechanism that transforms an LLM from a conversational model into a **true autonomous agent**. It allows the model to take actions: searching the web, querying databases, running code, calling APIs, or controlling a browser. This is the most technically complex and interview-critical chapter.

## 1. The Tool Use Loop

The agentic execution cycle has four distinct steps, run repeatedly until the agent determines the task is complete:

1. **Tool Definition:** Describe each available tool to the model using a precise schema (name, description, parameters with types and descriptions). The quality of this description directly determines how reliably the model will use the tool.
2. **Tool Calling:** The model pauses text generation and outputs a structured `tool_call` payload — e.g., `{"name": "search_web", "arguments": {"query": "Q3 2024 Apple revenue"}}`.
3. **Execution:** Your server-side code parses the payload, executes the actual Python function or API call, and collects the result.
4. **Observation Injection:** The result is injected back into the conversation as a `tool_result` message. The model reads it and decides whether to call another tool or synthesize the final answer.

![Agentic Tool Use Loop](assets/agent-tool-loop.png)


> **Common Gotcha:** The model does not run code. Your application does. The model only *declares intent* in a structured format. You are responsible for safe execution.

## 2. Execution Patterns

### Sequential (Single-Step)
The agent calls one tool at a time, reads the result, then decides the next action. Simplest to implement. Latency = sum of all tool call latencies.

### Parallel Tool Calling
When multiple tools can be called independently, modern models can emit **multiple tool calls in a single API response**. Your application launches them concurrently:
```python
# Model emits two tool calls simultaneously:
# search_web(query="Apple Q3 revenue")
# search_web(query="Google Q3 revenue")
# Your code runs them in parallel with asyncio/threading.
```
**Latency = max(tool latencies)** instead of sum — critical for time-sensitive agent tasks.

### Multi-Step Looping
Encapsulate the tool-call/execution cycle in a `while loop`:
```python
while response.tool_calls:
    # Execute all tool calls
    # Append results to message history
    response = llm.chat(messages=history, tools=tools)
# Loop exits when model returns text with no tool calls
```
The agent self-terminates when it determines no more tools are needed.

## 3. Deterministic Control: Forcing Tool Use

In production, you often cannot afford to let the LLM decide whether to use a tool or answer from memory. Use `tool_choice` to override model autonomy:

| Setting | Behavior | Use Case |
|---|---|---|
| `auto` (default) | Model decides whether to use tools | General purpose |
| `required` | Model MUST use a tool — cannot answer directly | When fresh data is always mandatory |
| `none` | Model MUST answer directly — no tools | Response generation phase of a pipeline |

## 4. Designing Tool Schemas

A poorly described tool is worse than no tool — the model will misuse it. Follow these rules:

* **Descriptions are prompt engineering:** Write the `description` field as a clear, specific instruction. Specify *when* to use the tool, *what* its output is, and any edge cases.
* **Use precise parameter types:** `string`, `integer`, `enum` (preferred over open-ended strings when possible — forces the model into valid choices).
* **Use TOON for large tool sets:** When defining 10+ tools, encoding schemas in TOON (Token-Oriented Object Notation) instead of verbose JSON saves 30–60% of context tokens, leaving more room for reasoning.

## 5. Model Context Protocol (MCP)

Historically, every agent required custom code to connect to every data source — the **M×N integration problem** (M models × N data sources). Anthropic's **Model Context Protocol (MCP)** solves this with a universal open standard.

**Architecture:**
* **MCP Server:** A lightweight adapter you write once for a data source (e.g., your SQL database, local filesystem, or Slack workspace). It exposes `Tools`, `Resources`, and `Prompts` via a standardized JSON-RPC interface.
* **MCP Client:** Built into the agent host (Claude Desktop, custom app). Automatically discovers and registers all tools the MCP Server exposes.
* **Result:** Any MCP-compatible agent can connect to any MCP-compatible server without custom glue code.

**Security model:** All tool calls route through explicit user-consent gates. The host environment surfaces approval prompts instead of allowing the agent to silently execute.

> **Why this matters in interviews:** MCP is rapidly becoming the industry standard for enterprise agent tool connectivity. Mentioning it demonstrates awareness of the real-world integration complexity beyond simple API wrapping.

## 6. Safety & Guardrails

Tool execution introduces serious risk. Design guardrails at every layer:

* **Input Validation:** Strictly validate tool arguments against the schema before execution. Never pass raw model output to `exec()` or `shell()`.
* **Code Sandboxing:** Execute LLM-generated code in an isolated environment (Docker container, AWS Lambda, E2B sandbox) with no access to the host filesystem or network.
* **Destructive Action Gates:** Build a `require_approval` tool that pauses the agent and prompts the human before any high-stakes action (send email, delete record, make payment).
* **Prompt Injection Defense:** Sanitize web-scraped content before it enters the agent's context. A malicious webpage could contain instructions designed to hijack the agent's next action.

---
**You have completed the Agentic AI System Design track.** You now have the full stack: LLMs → Embeddings → Generation Control → Deployment → Semantic Search → Prompt Engineering → RAG → Tool Use.
