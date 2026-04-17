# 4. Deployment

Deploying an Agentic AI system to production is vastly more complex than hosting a traditional REST API. The system is **stateful** (it maintains conversation history), **non-deterministic** (output varies per call), and **expensive** (each reasoning step costs real money). Each of these factors requires dedicated engineering decisions.

## 1. Clarifying Questions (Interview)

When asked to design an AI agent deployment, always clarify scope first:
* **Latency requirements?** Real-time user-facing (<2s total) vs. background async task (minutes are acceptable)?
* **Data residency?** Can traffic leave the corporate network? (Determines Cloud API vs. Private deployment)
* **Expected QPS?** Low-traffic internal tool vs. high-scale consumer product?
* **Cost budget per task?** Defines which model tier is viable.

## 2. Deployment Topologies

The central decision is where the model runs. Each option has a different trade-off in cost, control, and latency.

| Topology | Examples | Pros | Cons |
|---|---|---|---|
| **Cloud API (SaaS)** | OpenAI, Anthropic | Zero infra, instant scale, frontier models | Data leaves boundary, variable latency, metered cost |
| **Managed Hosting (Private VPC)** | AWS Bedrock, Azure AI Foundry | Data stays in your cloud boundary; HIPAA/SOC2 compliant | Still managed by provider; limited model customization |
| **Self-Hosted Bare Metal** | Llama 4 / Qwen-2.5 on A100/H100 clusters with vLLM | Full control over weights; absolute privacy | Massive DevOps overhead: CUDA, Kubernetes, autoscaling |
| **Edge / On-Device** | Llama 4 Scout on laptops/phones | Zero network latency; works offline | Severely limited model size; restricted context windows |

> **Rule of Thumb:** Start with a Cloud API for your MVP to validate the product. Migrate to managed or self-hosted only when you have data residency requirements or when token costs become prohibitive at scale.

## 3. Streaming: The Agent UX Contract

All production-grade Agentic UIs rely on streaming to minimize perceived latency. The architecture uses **Server-Sent Events (SSE)** or WebSockets.

For multi-step agents, the implementation has a critical layer of complexity: the UI layer must silently process the intermediate `tool_call` and `tool_result` stream events in the background and only begin displaying tokens to the user when the **final synthesis step** begins generating. Exposing raw tool JSON to end-users is a UX failure.

## 4. Optimization Strategies

Agents are expensive by default. Production systems must embed cost controls from day one.

### Semantic Caching
Instead of making a fresh LLM or search API call for every request, use **Semantic Caching** (e.g., GPTCache). When a new query arrives:
1. Embed the query into a vector.
2. Check a fast cache (Redis + Vector DB) for a semantically similar past query (cosine similarity > 0.97 threshold).
3. If found, return the cached response instantly — bypassing all LLM and API costs.

This is extremely effective for customer support and FAQ agents where 30-40% of queries are semantically near-duplicate.

### Semantic Routing
Use a tiny, cheap classification model to route requests based on intent *before* hitting the expensive frontier model:
* Simple queries → fast, cheap model (e.g., Qwen-2.5 7B or Llama 4 Scout)
* Complex multi-step reasoning → frontier model (GPT-5.4, Claude Opus 4.7)
* Cached frequent intents → return cached answer directly

## 5. Evaluation: LLM-as-a-Judge

Traditional unit tests cannot evaluate the quality of non-deterministic agent outputs. The modern standard is **LLM-as-a-Judge**: using a stronger model (e.g., GPT-5.4 or Claude Opus 4.7) to automatically score your agent's trajectories against rubrics.

| Eval Dimension | What it Measures |
|---|---|
| **Trajectory Accuracy** | Did the agent select the right sequence of tools? |
| **Tool Hallucination** | Did the agent invent tool parameters not in the schema? |
| **Answer Faithfulness** | Is the final answer supported by the retrieved evidence? |
| **Safety** | Did the agent attempt any prohibited or dangerous actions? |

Log every `ToolCall`, reasoning `Thought`, and final answer to an observability platform (**LangSmith** or **Phoenix**). Integrate the Judge into your CI/CD pipeline so that a regression in agent trajectory quality automatically fails a deployment.

---
**Next Step:** With the deployment infrastructure decided, the next critical capability is giving the agent long-term memory through semantic search. Proceed to Chapter 5.
