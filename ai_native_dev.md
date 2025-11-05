# Practical combo pattern we recommend for AI Native Development

* Orchestrate with **OpenAI Agents SDK** (handoffs + tracing).
* Plug in **MCP servers** for your internal tools and data.
* For Gemini-specific tasks or GCP data ops, **expose ADK agents** behind HTTP/A2A and call them as tools from your OpenAI orchestrator.
* For developer-automation jobs, compose **Claude Agent SDK** sub-agents (computer use/code tools) and surface them via MCP or HTTP.

This keeps each framework doing what it’s best at, without lock-in. (All three support MCP or easy interop.)

Almost any agent/RAG/workflow framework can slot into that pattern. The two universal ways to plug them in are:

1. **Expose it as an HTTP microservice** and call it from the OpenAI Agents SDK as a **function tool** (like the ADK/Claude stubs).
2. **Wrap it as an MCP server** and attach it as an MCP tool.

Here’s where popular options fit:

### Orchestrator alternatives

* **LangGraph / LangChain** – Use *instead of* the OpenAI Agents SDK if you want graph/state-machine control. Either:

  * Run LangGraph inside the same gateway (library mode) and still call ADK/Claude via HTTP; or
  * Put LangGraph behind its own FastAPI service and call it as a tool from the OpenAI orchestrator.
* **CrewAI / AutoGen** – Treat a “crew” or agent swarm as a **subgraph** behind HTTP. Great for code-gen/dev-automation pods (parallel to your Claude subagent).
* **Semantic Kernel (SK)** – If you’ve got SK skills/planners already, expose a “plan/execute” endpoint and call it as a tool; or wrap your SK skill hub as an MCP server.

### Tooling / data layer swaps

* **LlamaIndex / LangChain RAG** – Run your RAG as a service (`/query`, `/ingest`) or serve it via MCP. The orchestrator decides *when* to use it; your RAG service handles *how*.
* **Haystack, Text-Generation-Inference, vLLM/Ollama** – Same: put behind HTTP, or register via MCP; route calls via LiteLLM if you prefer one endpoint.
* **Vector DBs** (pgvector, Pinecone, Weaviate, Qdrant) – Either expose CRUD/query endpoints that the orchestrator calls as tools, or pick an MCP server that fronts them.

### Routing, guardrails, observability

* **LiteLLM / OpenRouter** – Drop-in as your model router; set the gateway to call the router instead of OpenAI directly.
* **Guardrails / Guidance / DSPy** – Best embedded inside a **subservice** (e.g., “guardrailed-rag”) so the orchestrator just sees a clean tool.
* **Langfuse / LangSmith / Arize Phoenix / AgentOps** – Instrument the gateway and subservices for traces/metrics; doesn’t change the pattern.

### Workflow & long-running jobs

* **Temporal / Dagster / Prefect / Airflow** – For multi-hour tasks (ETL, evals), make a “job-runner” service that the orchestrator triggers, then polls/status-calls as a tool.

---

## How to add “any framework” in your current skeleton

1. Spin up a new microservice (e.g., `apps/langgraph_service`) with a `POST /run` that accepts `{prompt: string, context?: …}` and returns `{result: string}`.
2. In `apps/gateway/main.py`, add a function tool:

```python
@function_tool
def call_langgraph(prompt: str) -> str:
    import httpx
    r = httpx.post(f"{os.getenv('LANGGRAPH_URL')}/run", json={"prompt": prompt}, timeout=120)
    r.raise_for_status()
    return r.json().get("result", "")
```

3. Decide if it should be **MCP** instead—then run it as/behind an MCP server and register it with `HostedMCPTool(...)`.

That’s it—the orchestrator keeps doing handoffs/tracing; each extra framework is “just another tool” via HTTP or MCP. If you tell me which one you want first (LangGraph, CrewAI, AutoGen, SK, LlamaIndex), I’ll drop a ready-to-run microservice into the repo zip and wire it in.




## Research


compare Claude Agents SDK with OpenAI Agents SDK and Google ADK

https://chatgpt.com/share/690a2b80-0c60-8001-af53-b3970d924f63

Here’s a crisp, side-by-side on the three you asked about—what they are, what they’re great at, and when to pick which.

# TL;DR

* **OpenAI Agents SDK** → Minimal, production-ready agent framework with built-in tracing, strong multi-agent “handoffs,” realtime voice agents, hosted tools, and MCP support. Good default if you want fast iteration and provider-agnostic wiring (incl. via LiteLLM). ([github.com][1])
* **Claude Agent SDK (Anthropic)** → Code-centric, security-forward SDK with rich built-in tools (computer use, code/bash, web fetch/search, memory), fine-grained permissions, streaming, MCP, and new “Agent Skills.” Great when building coding/automation agents around Claude. ([Claude Docs][2])
* **Google ADK (Agent Development Kit)** → Open-source (Apache-2.0), multi-language (Python/Java), model-flexible (Gemini + Anthropic/OpenAI/Open/local via LiteLLM), deep GCP integrations (BigQuery/Spanner/Vertex AI Search), multi-agent/A2A protocol, managed deployment to Vertex AI Agent Engine/Cloud Run/GKE, and bidi streaming. Best for Gemini-first or GCP-native stacks. ([github.com][3])

---

## Feature comparison (at a glance)

| Area                          | OpenAI Agents SDK                                                                                   | Claude Agent SDK                                                                                            | Google ADK                                                                                                            |
| ----------------------------- | --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Languages**                 | Python, JS/TS                                                                                       | Python, TS                                                                                                  | Python, Java                                                                                                          |
| **Core model focus**          | OpenAI first; provider-agnostic (via LiteLLM)                                                       | Claude first                                                                                                | Gemini first; also Anthropic/OpenAI/open/local via LiteLLM                                                            |
| **Multi-agent orchestration** | Native *handoffs* between agents                                                                    | Skills + tools; (multi-agent patterns are possible, but handoff is not a first-class primitive like OpenAI) | Built-in “Agent teams” + **A2A protocol** for inter-agent comms                                                       |
| **Realtime / voice**          | Realtime (voice) agents in the SDK                                                                  | Streaming; (voice handled via API/workbench patterns)                                                       | Bidi (live) streaming with Gemini Live                                                                                |
| **Built-in / hosted tools**   | Hosted tools: web/file search, computer use, code interpreter, image gen; plus function tools & MCP | Bash, code execution, **computer use**, text editor, web fetch/search, **memory**; MCP                      | Google Search grounding, BigQuery/Spanner/Bigtable, Vertex AI Search/RAG, code execution toolsets; MCP, OpenAPI tools |
| **State & sessions**          | Sessions built-in                                                                                   | Session mgmt & context compaction                                                                           | Sessions, state, memory; Vertex AI Express Mode                                                                       |
| **Observability**             | Built-in tracing (UI)                                                                               | Monitoring hooks in SDK                                                                                     | Logging/Cloud Trace; supports AgentOps/Arize/Phoenix/Weave                                                            |
| **Deployment**                | Run anywhere (lib); pairs with OpenAI Platform (Agent Builder)                                      | Run anywhere (lib)                                                                                          | Vertex AI **Agent Engine** (managed), **Cloud Run**, **GKE**, or any container                                        |
| **License**                   | MIT                                                                                                 | MIT                                                                                                         | Apache-2.0                                                                                                            |

**Sources:** OpenAI primitives/tracing/JS+Realtime/hosted tools/LiteLLM & MCP ([github.com][1]); Claude tools/MCP/Skills ([Claude Docs][2]); ADK built-ins/models/deploy/A2A/licensing/streaming ([docs.cloud.google.com][4]).

---

## What stands out about each

### OpenAI Agents SDK

* **Minimal, powerful primitives**: Agents, Guardrails, *Handoffs*, Sessions—easy to compose multi-agent workflows. ([github.com][1])
* **First-class observability**: Built-in tracing UI for runs. ([github.com][1])
* **Hosted capabilities**: one-line access to web/file search, computer use, code interpreter, images; plus function tools & MCP servers. ([openai.github.io][5])
* **Realtime voice agents** (browser + server flows). ([openai.github.io][6])
* **Model flexibility**: works with OpenAI APIs and “any model via LiteLLM.” ([openai.github.io][7])

### Claude Agent SDK

* **Strong “work agent” toolset** out of the box: bash/code exec, **computer use**, web fetch/search, **memory**, fine-grained permissions; streaming. ([Claude Docs][2])
* **Agent Skills**: modular, composable capability packs you can attach across Claude Code/API/SDK. ([Claude Docs][2])
* **MCP**: native connector + remote servers. ([Claude Docs][2])

### Google ADK

* **GCP-native toolkit**: **BigQuery/Spanner/Bigtable** tools, Vertex AI Search / RAG Engine, Google Search grounding; plus third-party tools. ([docs.cloud.google.com][4])
* **Model-agnostic by design**: Gemini (AI Studio/Vertex), Anthropic, and **OpenAI/open/local via LiteLLM/Ollama/vLLM**. ([google.github.io][8])
* **Multi-agent & interop**: Agent teams + **A2A protocol** for cross-agent communication. ([google.github.io][9])
* **Enterprise deployment**: One-click to **Vertex AI Agent Engine**, or ship to **Cloud Run/GKE/containers**; observability via Cloud Trace & friends; bidi streaming with Gemini Live. ([google.github.io][10])
* **Open-source (Apache-2.0)** across repos/packages. ([github.com][3])

---

## Which should you pick?

Given your stack (Kubernetes + Docker, FastAPI, PostgreSQL/Kafka; interest in OpenAI & Gemini; goal of agentic systems):

* **If you want the fastest path to production multi-agent flows with great tracing and optional realtime voice** → **OpenAI Agents SDK**. Start here for a clean orchestration layer, and still call Gemini/others via LiteLLM when needed. ([github.com][1])
* **If you’re building deep “computer use” and code-editing automations around Claude** (e.g., autonomous dev assistants) → **Claude Agent SDK** for its built-in tools and Skills. ([Claude Docs][2])
* **If you’re standardizing on Gemini and GCP services, want managed runtime + native data tools** → **Google ADK** with Vertex AI Agent Engine (plus Cloud Run/GKE for portability). ([google.github.io][10])

---

## Practical combo pattern I recommend

* Orchestrate with **OpenAI Agents SDK** (handoffs + tracing).
* Plug in **MCP servers** for your internal tools and data.
* For Gemini-specific tasks or GCP data ops, **expose ADK agents** behind HTTP/A2A and call them as tools from your OpenAI orchestrator.
* For developer-automation jobs, compose **Claude Agent SDK** sub-agents (computer use/code tools) and surface them via MCP or HTTP.

This keeps each framework doing what it’s best at, without lock-in. (All three support MCP or easy interop.) ([Claude Docs][2])

If you want, I can sketch a small reference architecture (repo layout + Docker/Helm) that wires these together with handoffs/MCP and a FastAPI edge.

[1]: https://github.com/openai/openai-agents-python "GitHub - openai/openai-agents-python: A lightweight, powerful framework for multi-agent workflows"
[2]: https://docs.claude.com/en/api/agent-sdk/overview "Agent SDK overview - Claude Docs"
[3]: https://github.com/google/adk-python?utm_source=chatgpt.com "google/adk-python"
[4]: https://docs.cloud.google.com/agent-builder/agent-engine/develop/adk "Develop an Agent Development Kit agent  |  Vertex AI Agent Builder  |  Google Cloud Documentation"
[5]: https://openai.github.io/openai-agents-js/guides/tools/?utm_source=chatgpt.com "Tools | OpenAI Agents SDK - GitHub Pages"
[6]: https://openai.github.io/openai-agents-js/?utm_source=chatgpt.com "OpenAI Agents SDK TypeScript - GitHub Pages"
[7]: https://openai.github.io/openai-agents-python/ "OpenAI Agents SDK"
[8]: https://google.github.io/adk-docs/agents/models/ "Models & Authentication - Agent Development Kit"
[9]: https://google.github.io/adk-docs/a2a/ "ADK with Agent2Agent (A2A) Protocol - Agent Development Kit"
[10]: https://google.github.io/adk-docs/deploy/ "Deploying Your Agent - Agent Development Kit"
