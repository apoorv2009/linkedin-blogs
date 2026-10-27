# Roadmap

Writing types in this programme:

1. Curtain raiser: write-up 1 of each series. Sets the scene, shows what the demo hides.
2. Middle write-up: one design decision per write-up, defended like a design review.
3. Finale: the full system checklist for the series, plus honest lessons.
4. Course companion: the longer lesson behind each write-up. Not for LinkedIn.

## 1. AI System Design, from the whiteboard

Architecting resilient, scalable, and cost-aware enterprise AI systems without falling for demo-ware traps.

The storyline:

- **Act 1. Frame the problem.** Before building anything, set the constraints and pick the intelligence layer.
  - Series 1: "Build us an assistant that answers HR questions for 20,000 staff"
  - Series 2: "Pick an AI model for us, and don't lock us in"
- **Act 2. Build the core.** The parts that do the actual work: search at scale, routing, action, memory.
  - Series 3: "Make 2 million product descriptions searchable by meaning"
  - Series 4: "Answer the easy questions cheaply and the hard ones well"
  - Series 5: "Our support team needs an agent that can actually issue refunds"
  - Series 6: "Make the assistant remember customers between conversations"
- **Act 3. Make it safe and measurable.** Defend it, test it, and see inside it.
  - Series 7: "Stop the chatbot from leaking customer data"
  - Series 8: "How do we know the new model version is better, not just different?"
  - Series 9: "Our AI feature is slow for some customers and nobody can tell why"
- **Act 4. Run it at scale.** Keep it fast, affordable, and standing up when it grows.
  - Series 10: "The AI bill tripled last quarter. Fix it without breaking anything"
  - Series 11: "Run the models ourselves, inside our own network"
  - Series 12: "Get five teams' AI agents to work together without chaos"

| Series | Topic | Level (L1 to L5) | What it covers | Status | Why a full series, not one article |
|---|---|---|---|---|---|
| *Act 1. Frame the problem* | | | | | |
| 1 | "Build us an assistant that answers HR questions for 20,000 staff" | L4 | Problem framing with the L1 to L5 scale; SLAs and non-functional requirements (time to first token vs total generation time, cost ceilings, throughput); deterministic vs model boundaries; parsing scanned PDFs and tables; chunking (fixed, semantic, recursive, parent-child); dense vs sparse embeddings and hybrid search; query rewriting and HyDE; reranking; context window management and lost in the middle; metadata filtering; document-level access control; grounding and handoff; freshness. Example: an HR policy and benefits assistant for a 20,000-person global company. Plan: [01-hr-assistant/](01-hr-assistant/README.md) | Planned (curtain raiser drafted) | A retrieval engine is a chain: parse, chunk, embed, search, rerank, fit into context. Each link fails in its own way and needs its own decision. |
| 2 | "Pick an AI model for us, and don't lock us in" | L3 | Model selection matrix: proprietary APIs (such as OpenAI, Anthropic, Gemini) vs open-weights models (such as Llama, Mistral, Qwen); small vs large vs reasoning models; fine-tuning (LoRA, QLoRA) vs in-context learning; gateway pattern and exit plan. Example: a shared model gateway for a bank's internal AI use cases. Plan: [02-model-choice/](02-model-choice/README.md) (not written yet) | Idea | "Which model?" is really five questions: quality, cost, speed, data terms, and exit plan. |
| *Act 2. Build the core* | | | | | |
| 3 | "Make 2 million product descriptions searchable by meaning" | L3 | Vector index types (HNSW, IVF, Flat) and their trade-offs; database selection (such as pgvector, Qdrant, Pinecone, Milvus); re-embedding cost; multilingual search; filtering at scale; speed under load. Example: product search for an online retailer with 2 million items in 6 languages. Plan: [03-product-search/](03-product-search/README.md) (not written yet) | Idea | Search that works on 3,000 documents behaves very differently on 2 million. Scale changes every choice. |
| 4 | "Answer the easy questions cheaply and the hard ones well" | L3 | Semantic routers by intent and complexity; cascading setups (cheapest model first, escalate on failure); escalation signals; routing evaluation. Example: an internal IT service desk assistant for a 50,000-seat company. Plan: [04-model-routing/](04-model-routing/README.md) (not written yet) | Idea | A router is a model making decisions about models. Getting it wrong is silent and expensive, in both directions. |
| 5 | "Our support team needs an agent that can actually issue refunds" | L4 | Agent control loops (ReAct, Plan-and-Execute, Reflexion); tool and function calling: schemas, validation, execution sandboxes; identity delegation (OAuth, On-Behalf-Of flow) for tool calls; approvals, safe retries, undo, spending limits. Example: a support agent for a telecom with refund, plan-change, and cancellation tools. Plan: [05-refund-agent/](05-refund-agent/README.md) (not written yet) | Idea | Reading is safe. Acting is not. Each action needs its own guard rails and its own identity. |
| 6 | "Make the assistant remember customers between conversations" | L3 | Short-term context memory vs long-term persistent memory (graph-based, key-value stores, vector history); entity extraction and user memory indexing; what to forget; consent. Example: a trip-planning assistant for a travel company. Plan: [06-assistant-memory/](06-assistant-memory/README.md) (not written yet) | Idea | Memory is a database the model writes to. Deciding what to keep, where, and for how long is several designs, not one. |
| *Act 3. Make it safe and measurable* | | | | | |
| 7 | "Stop the chatbot from leaking customer data" | L4 | Prompt injection defence (direct and indirect); PII redaction and masking (such as Presidio, regex and NER filters); hallucination detection; toxicity filtering; red-team testing. Example: a public insurance claims chatbot. Plan: [07-chatbot-safety/](07-chatbot-safety/README.md) (not written yet) | Idea | Attacks come in many shapes. No single filter stops them all, so the defence is layered. |
| 8 | "How do we know the new model version is better, not just different?" | L4 | RAG triad evals (context relevance, groundedness, answer relevance); LLM-as-a-judge vs heuristic and code-based evals; offline dataset evals vs online production monitoring; regression gates. Example: a contract-review assistant for a legal team. Plan: [08-model-upgrades/](08-model-upgrades/README.md) (not written yet) | Idea | Measuring language quality is hard and easy to fake. It needs a full method, not a tip. |
| 9 | "Our AI feature is slow for some customers and nobody can tell why" | L4 | Span tracing across prompts, retrieval, and tools (such as OpenTelemetry, LangSmith, Phoenix); token tracking, latency breakdown, cost per user and tenant; explicit and implicit feedback (thumbs, edits); human-in-the-loop review; preference data and fine-tuning datasets built from telemetry. Example: an AI writing assistant inside a B2B software product with 400 customer companies. Plan: [09-ai-observability/](09-ai-observability/README.md) (not written yet) | Idea | You cannot fix what you cannot see, and AI systems hide their work in many layers at once. |
| *Act 4. Run it at scale* | | | | | |
| 10 | "The AI bill tripled last quarter. Fix it without breaking anything" | L4 | Exact-match vs semantic caching (such as Redis, GPTCache); provider prompt caching; token rate limiting and quotas; fallback routing during provider outages and HTTP 429 rate-limit errors; cost per answer. Example: a customer service platform handling 1 million conversations a month. Plan: [10-ai-bill/](10-ai-bill/README.md) (not written yet) | Idea | Every saving trades against quality or speed. Each trade needs its own test. |
| 11 | "Run the models ourselves, inside our own network" | L4 | Model serving (such as vLLM, TGI, Ollama, Triton Inference Server); quantization (INT8, FP4, AWQ, GGUF); speculative decoding; KV-cache optimisation and reuse; batch processing vs real-time streaming (Server-Sent Events, WebSockets). Example: a clinical-notes summariser for a hospital group whose data may not leave its network. Plan: [11-self-hosted-models/](11-self-hosted-models/README.md) (not written yet) | Idea | Owning the GPUs means owning speed, memory, and uptime. Each one is a separate engineering problem. |
| 12 | "Get five teams' AI agents to work together without chaos" | L5 | Hierarchical supervisor agents vs peer-to-peer agent networks; state management across handoffs (LangGraph and AutoGen patterns as examples); contracts between agents; failure isolation; multi-region operation. Example: an order-to-cash workflow at a logistics company with quoting, booking, and invoicing agents. Plan: [12-agent-teams/](12-agent-teams/README.md) (not written yet) | Idea | One agent is a program. Five agents owned by five teams is an organisation problem as well as a technical one. |

Level scale: L1 = a single prompt or API call, such as a simple summariser. L3 = a team-level assistant over custom enterprise data with RAG. L5 = a multi-region, regulated, autonomous agent system connected to legacy systems. L2 and L4 sit between their neighbours.

### Topic coverage map

Every topic has exactly one home. Other series link to it instead of repeating it.

| Topic area | Sub-topic | Home |
|---|---|---|
| 1. Problem framing and requirements | L1 to L5 complexity scale | Series 1, write-up 2 |
| | SLAs and NFRs: latency budgets, cost ceilings, throughput | Series 1, write-up 3 |
| | Deterministic vs non-deterministic boundaries | Series 1, write-up 4 |
| 2. Model selection, routing, optimisation | Proprietary vs open weights; SLM vs LLM vs reasoning models | Series 2 |
| | Fine-tuning (LoRA, QLoRA) vs in-context learning | Series 2 |
| | Semantic routers and cascades | Series 4 |
| | Quantization, speculative decoding, KV-cache | Series 11 |
| 3. Retrieval and data pipelines | Multimodal parsing (scans, tables, charts) | Series 1, write-up 5 |
| | Chunking strategies | Series 1, write-up 6 |
| | Dense vs sparse embeddings, hybrid search | Series 1, write-up 7 |
| | Query expansion, rewriting, HyDE | Series 1, write-up 8 |
| | Reranking and Precision@K | Series 1, write-up 9 |
| | Context window, lost in the middle, compression | Series 1, write-up 10 |
| | Vector index types and database selection | Series 3 |
| 4. Agents and multi-agent systems | ReAct, Plan-and-Execute, Reflexion; tool calling | Series 5 |
| | Memory architectures, entity extraction | Series 6 |
| | Hierarchical vs peer-to-peer; state across handoffs | Series 12 |
| 5. Security and guardrails | Prompt injection, PII redaction, hallucination and toxicity filters | Series 7 |
| | Document-level access control in vector search | Series 1, write-up 12 |
| | Identity delegation (OAuth, On-Behalf-Of) for tool calls | Series 5 |
| 6. Evaluation and observability | RAG triad, LLM-as-a-judge vs code evals, offline vs online | Series 8 |
| | Span tracing, token and cost per tenant | Series 9 |
| | Human-in-the-loop, feedback, telemetry-to-fine-tuning pipelines | Series 9 |
| 7. Scaling, infrastructure, cost | Exact vs semantic caching; prompt caching | Series 10 |
| | Static KV-cache reuse | Series 11 (Series 10 links to it) |
| | Batch vs streaming; model serving | Series 11 |
| | Rate limits, quotas, fallback on outages and 429s | Series 10 |

### Notes

- **Basics covered elsewhere:** none yet. This is the first programme. Planned links inside it:
  - Series 3 links to Series 1 write-up 7 for dense, sparse, and hybrid search basics, and covers only index types and scale.
  - Series 8 links to Series 1 write-up 13 for grounding, and covers measurement.
  - Series 10 links to Series 1 write-up 3 for cost ceilings, to Series 4 for routing, and to Series 11 for KV-cache reuse.
  - Series 7 links to Series 1 write-up 12 for document-level access control, and covers attacks and filters.
- **External inspiration:** the "requirement, options, trade-offs, decision" spine borrows from the Architecture Decision Record idea, first described by Michael Nygard in "Documenting Architecture Decisions" (2011).
- **Retired** (27 Oct 2026, when the programme was re-planned around the seven topic areas above):
  - "Let the sales team ask our data warehouse questions in plain English" (text-to-SQL): outside the seven topic areas. May return as a bonus series.
  - "Automate invoice processing across 300 suppliers' formats": document parsing now lives in Series 1 write-up 5.
  - "Prove to the auditors how the AI reached that answer": tracing now lives in Series 9.
  - "Roll it out to 14 countries, each with its own data rules": data locality now sits in Series 11, multi-region in Series 12.
