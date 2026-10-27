# AI System Design: From Real-World Problems to Production-Ready Solutions

Series plan, version 1 (27 Oct 2026). Covers Stages 1 to 3: reference analysis, roadmap, and a check of the sequence. Articles 1 to 3 are drafted in [articles/](articles/).

---

## Stage 1: What the reference material covers

### Primary reference: *System Design for the LLM Era* (Sampriti Mitra, 2026)

The book is about building systems that *use* LLMs, not about building models. It has three layers:

| Layer | What it covers | How useful it is for this series |
|---|---|---|
| Ch. 1, the building blocks | Tokens, embeddings, context window, temperature, prompts, RAG, the limits of naive RAG, knowledge graphs, GraphRAG, context engineering, agents (ReAct), modality, data residency, benchmarking (TTFT, TPS), failure types | Good vocabulary, but written as definitions. Every item has to be turned into a problem first. |
| Ch. 2, the patterns | LLM gateway, circuit breakers with tiered fallback, sync and async paths, streaming (SSE), exact, semantic and proactive caching, request coalescing, model router, prompt compression, ingestion pipeline, hybrid RAG, function calling, golden datasets, LLM-as-a-judge, new metrics, OWASP-style threats (prompt injection, output handling, excessive agency, data disclosure, model DoS, poisoning, supply chain), weighted evaluation for agents, privacy | The backbone of the series. Each pattern already follows a problem-then-solution shape, which makes it a good fit for the problem-first approach. |
| Ch. 3 to 6, case studies | AI IDE (AST chunking, code graphs, low-latency completion), adaptive learning (offline generation with human review, online personalisation), e-commerce search (query understanding, hybrid search, ranking), customer support agent (GraphRAG, guardrails, golden-set examples, monitoring thresholds, failure-mode analysis) | Good for later articles, and evidence that the patterns combine. The support case study's monitoring and evaluation sections are the richest. |

**Learning order in the book:** primitives, then patterns, then whole systems. That works for a book. On LinkedIn, where a reader may see only one post, every article has to start with a problem instead. This series keeps the book's order but rewrites each step as a problem.

**Copyright note:** the book is used as a map of concepts and cited as further reading. Its text, figures and examples are not reproduced or closely paraphrased.

### Supporting references

- **IITH Applied AI course (Days 1 to 4).** Follows a four-question arc: *LLM to software, software to intelligent, intelligent to autonomous, autonomous to production-ready*. It is stronger than the book on statelessness and per-turn cost, structured output, memory types (working, episodic, semantic, procedural), summarisation loss, LangGraph checkpointing, harness and loop engineering, nested tracing, prompt-cache hit rate as a health signal, and evaluation and guardrails.
- **Domain decks 1 to 5.** Agentic loop and `stop_reason`, coordinator and subagent, tool descriptions and structured tool errors (`isError`, retryable), MCP, explicit criteria and false positives, few-shot, context rot and lost-in-the-middle, case-facts blocks, escalation triggers.
- **Your ShopSense capstone evaluation (verified project experience).** This is the most valuable material because it is *yours* and it is specific. Examples: a gateway that fails over only on transient errors, a 54,990 refund that paused, survived a restart and paid exactly once, hybrid search plus reranking moving MRR@10 from 0.858 to 0.887, and a number guard that blocked an invented "14 days". It also has honest lessons: an unauthenticated approval endpoint, retrieved text not marked as untrusted, and traces without per-call cost. Several articles use these as first-person stories. **Please confirm the wording before each one is published.**

### Gaps in the reference material that the series must fill

| Gap | Why it matters | Where it is covered |
|---|---|---|
| Identity and authorisation in retrieval and tools | The book filters by `tenant_id`. Enterprises need document-level permissions, the user's identity passed through to tools (on-behalf-of), and approvers who are authenticated. | Parts 8, 16 |
| Structured output and validation as an architectural layer | The book mentions JSON validation in passing. In practice it is the first boundary between probability and business logic. | Part 1 |
| Memory engineering | The book offers only a sliding window and summarisation. It does not cover memory types, write policies, or testing what survives a summary. | Parts 2, 11 |
| Tool design quality and MCP | The book covers function calling in one paragraph. Tool descriptions, error contracts and MCP decide whether agents work at all. | Part 14 |
| Durable execution, idempotency and state | Agents that act must survive restarts and must not repeat side effects. | Part 15 |
| Depth of evaluation | Missing from the book: retrieval metrics (recall@k, MRR), keeping retrieval separate from generation, bias and calibration of the LLM judge, and maintaining the evaluation dataset. | Parts 7, 18 |
| Indirect prompt injection | The book focuses on injection by the user. Documents, emails and tool results that carry instructions are the bigger enterprise risk. | Part 19 |
| Changing prompts and models safely (LLMOps) | Versioning, gated releases, canaries and rollback for prompts and models. | Part 24 |
| Freshness, versioning and deletion in knowledge bases | "Updated yesterday" and "deleted under GDPR" are both ingestion problems. | Parts 4, 5 |
| Governance, audit and regulation | Audit trails, decision records, and the EU AI Act risk categories for HR and credit use cases. | Parts 16, 23 |
| Adoption and trust UX | Citations, visible uncertainty and graceful refusal decide whether people use the system. | Parts 9, 21 |
| .NET and Azure implementation | The book's examples are Python and AWS (OpenSearch, Neptune, Spark, Glue). Your audience works in .NET and Azure. | Throughout, where it fits |

### Claims in the reference that need careful handling (your credibility matters)

1. **"Hallucination is solved by grounding with RAG."** RAG *reduces* unsupported answers. It does not remove them, because models can misread or ignore the retrieved context. The series always says "reduces".
2. **"Temperature 0 gives consistency."** It lowers variation but does not guarantee identical output. Thinking Machines (Horace He, 10 Sept 2025) reports 80 unique completions from 1,000 runs at temperature 0 on Qwen3-235B-A22B-Instruct-2507 with standard inference settings. Some reasoning models don't expose temperature at all. Also, consistent is not the same as correct.
3. **"Embeddings are irreversible, so storing only vectors protects the source."** Research on embedding inversion (Morris et al., *Text Embeddings Reveal (Almost) As Much As Text*, EMNLP 2023) shows that a lot of the original text can be recovered. Vectors have to be treated as sensitive data.
4. **"Fine-tune on 2024 to 2025 documents to update knowledge."** Fine-tuning is unreliable for adding facts and hard to keep current. It is better for behaviour, format and style. Retrieval is the usual answer for knowledge that changes.
5. **Model names will date** (GPT-5, Claude 3 Haiku, Llama 3). The articles describe model *tiers* and name products only where needed, as of the publication date.
6. **"Tokens = words".** The book's example splits text by word. Real tokenisers split into sub-word pieces, and this affects cost estimates.

---

## Writing style (updated 27 Oct 2026 after feedback)

Write for a non-technical reader first. An engineer should still find it accurate.

- **Length:** about 450 to 550 words per post. One idea per post.
- **Words:** short sentences and everyday words. Explain any technical term in plain words the first time it appears, or leave it out.
- **No code** in the post body. Diagrams only in the image.
- **Shape:** a short story, why it happens, an everyday comparison (and where it breaks down), the simple fix, what it means for the business, the key takeaway, a question for readers.
- **For the engineers:** one italic line at the end naming the technical terms and tools, so technical readers know the depth is there.
- The image brief, the generation prompt and two or three references sit below the post. They are not part of what gets published.
- The roadmap table below still lists technical concepts. They are the *ideas behind* each post, not the vocabulary of the post.

## The direction of the series

**One fictional company, many real problems.** Every scenario takes place at **Contoso**, Microsoft's long-running fictional company. .NET and Azure readers recognise it at once as an illustrative example, so no hypothetical incident can be mistaken for a documented one. Contoso has the departments the scenarios need: accounts payable, customer support, HR, finance analysts, engineering, and operations.

**Four questions per article.** What is the problem? Why is it harder than it looks? Which concept helps? Which architectural decisions make it reliable?

**An architecture that is built up step by step.** Each article grows a baseline (v0) into a robust design (v1, v2, v3) and names the limitation that forces each step. The same Contoso platform grows across the series, so Part 25 can draw the whole system from decisions the reader has already seen being made.

**Evidence over claims.** Every number is either cited, clearly marked as a hypothetical with its assumptions stated, or taken from your ShopSense build. Nothing else.

**Six arcs:**

1. **The model is a component, not the product** (Parts 1 to 3). How LLMs actually behave as a dependency.
2. **Grounding answers in what the organisation knows** (Parts 4 to 10).
3. **Memory and context** (Parts 11 to 12).
4. **From answering to acting** (Parts 13 to 17).
5. **Earning trust: evaluation and security** (Parts 18 to 20).
6. **Running it for real** (Parts 21 to 25).

---

## Stage 2: The roadmap

| Part | Article title | Real-world business problem (Contoso, illustrative) | Main concept | Analogy | Practical solution | Learning outcome | Suggested visual |
|---|---|---|---|---|---|---|---|
| **Arc 1: The model is a component** |||||||||
| 1 | Same Invoice, Two Different Totals | Accounts payable pilots AI extraction of invoice fields. A great demo, then prose answers, totals that drift between runs, and a PO number that was never on the invoice. | Probabilistic generation; structured output; validation layers | **Bank deposit slip and teller**: free speech becomes fixed fields, and someone checks them | Schema-constrained output, then business-rule validators, then evidence per field, then routing to a review queue | Model output becomes data only after code has checked it | Left: customer, slip, teller. Right: LLM, schema, validators, auto-post or review queue |
| 2 | The Support Chat That Got Slower With Every Message | The support assistant is fast at turn 5, slow at turn 40, and by turn 58 has forgotten the customer's key constraint | Stateless model calls; context window; per-turn cost; context rot | **Locum doctor with a patient file** that gets thicker at every visit | Pinned case-facts block, recent-turn window, retrieval of older turns, tool output trimmed, tests that facts survive a summary | Memory is an application design decision, made deliberately or by accident | Left: doctor with a thick file and a summary sheet on top. Right: prompt assembly with a facts block and a window |
| 3 | When Your AI Provider Has a Bad Afternoon | Three teams call model APIs directly. A rate-limit spike hangs the chat for 60 s, batch retries make it worse, and nobody knows what each team spends. | LLM gateway; retries with jitter; circuit breaker; tiered fallback; quotas | **Corporate travel desk** that rebooks you when the airline cancels | Shared client, then a central gateway (Azure API Management AI gateway), with separate interactive and batch policies, and fallbacks that have been evaluated | Separate the application's health from the provider's | Left: travel desk rerouting travellers. Right: services, gateway, primary and fallback models, breaker |
| **Arc 2: Grounding answers** |||||||||
| 4 | The Policy Changed Yesterday. The AI Didn't Notice. | An employee asks about expense limits that changed yesterday and gets a confident, outdated answer | Knowledge cutoff; fine-tuning vs long context vs retrieval; RAG | **Open-book exam**: memorising beforehand vs looking up the right page | Retrieve, then augment, then generate, with freshness metadata and a check of document version | When RAG is the right tool and when it isn't | Left: student with an open textbook and sticky tabs. Right: question, retriever, context, model |
| 5 | 10,000 Documents, and Half of Them Are Messy | Contracts, scanned forms and wiki pages with tables, headers and duplicates make retrieval useless | Ingestion pipeline: parsing, cleaning, chunking, metadata, PII scrubbing, deletion | **Professional movers labelling boxes by room**: a box marked "misc" is never found again | Pipeline driven by events (Azure Functions plus queue), structure-aware chunking, metadata schema, propagation of deletes | Retrieval quality is decided at ingestion | Left: labelled boxes on a conveyor. Right: parse, clean, chunk, enrich, index |
| 6 | Search by Meaning Found "Pump Failure". It Missed "KX-4410". | Field engineers search manuals by part number. Semantic search returns pages that are similar but wrong. | Embeddings; vector similarity; keyword search (BM25); hybrid search with RRF | **City map of neighbourhoods plus a street directory**: meaning gets you to the right area, an exact name finds the house | Azure AI Search hybrid query (vector + keyword) with reciprocal rank fusion | Similar is not the same as relevant, and exact identifiers need exact matching | Left: map with clusters and a directory index. Right: two retrievers merging into one ranked list |
| 7 | The Right Answer Was Retrieved. It Was Ranked Ninth. | The correct clause is in the top 20 but not the top 3, so the answer is wrong | Retrieve-then-rerank; cross-encoders; recall@k vs precision; MRR | **Hiring funnel**: a broad CV screen, then a careful interview panel | Retrieve wide, then rerank (Azure AI Search semantic ranker or a cross-encoder), measured with recall@k and MRR. ShopSense numbers. | Measure retrieval separately from generation | Left: CV stack, then shortlist, then panel. Right: top-50, reranker, top-5 |
| 8 | Everyone Can Ask. Not Everyone Can See. | An intern asks the assistant about a salary review and retrieval returns a restricted board paper | Permission-aware retrieval; security trimming; identity passed through (Entra ID) | **Hotel key card**: the lift only goes to floors your card allows | Permission metadata (ACLs) at ingestion, filters applied at query time from the user's token, never left to the prompt | Authorisation belongs to retrieval, not to the model | Left: lift panel with locked floors. Right: user token, filtered index, allowed chunks only |
| 9 | "According to Which Document?" | Analysts compare quarterly reports. An answer with a wrong figure and no source destroys trust. | Citations; grounding checks; refusing when unsure; numeric verification | **Newsroom fact-check desk**: every claim needs a source, or it doesn't run | Answers with citation IDs, a groundedness check, a number guard (ShopSense), and "not found in the sources" as a valid answer | Trust comes from answers that can be verified, not from fluency | Left: editor checking claims against clippings. Right: answer sentences linked to source chunks |
| 10 | When Similar Isn't Related | "Which suppliers are affected by the recall of component X?" needs relationships, not similar text | Knowledge graphs; GraphRAG; structured data vs unstructured; safe text-to-SQL | **Family tree vs photo album**: photos look alike, the tree shows who is actually related | Route to vector search, graph traversal or parameterised SQL depending on the question | Choose retrieval by the shape of the question | Left: album vs tree. Right: router, then vector, graph or SQL |
| **Arc 3: Memory and context** |||||||||
| 11 | The Assistant Asked for My Order Number Five Times | A returning customer has to re-explain history at every session | Working, episodic, semantic and procedural memory; write policies | **Nursing shift handover**: what must be passed on, and what can be dropped | Store memories per customer with a write policy, typed preferences, and retrieval at session start | Remembering is a product decision with privacy consequences | Left: handover board. Right: four memory stores feeding one context |
| 12 | Context Engineering: Deciding What the Model Sees | Answers get worse as more instructions, documents and history are added | Context assembly; token budgets; ordering; prompt caching | **Pilot's pre-flight briefing pack**: curated pages, not the whole manual | A context builder with priorities, a budget per source, and a stable prefix that can be cached | Less, better-ordered context beats more context | Left: briefing pack vs bookshelf. Right: context window as a budgeted layout |
| **Arc 4: From answering to acting** |||||||||
| 13 | Should This Be an Agent or a Workflow? | Expense claims: rules, receipts, approvals. Is an agent the right tool? | Deterministic workflow vs AI at certain steps vs agentic loop | **Recipe, line cook, head chef**: increasing judgment, increasing risk | Compare three designs on one process, then pick the least autonomous one that works | Autonomy has a cost and has to earn its place | Three lanes: rails, rails with judgment points, open loop with guard rails |
| 14 | Giving AI Hands: Tools an Agent Can Actually Use | The agent keeps calling the wrong tool, or gives up and makes something up | Tool calling; tool descriptions; structured errors; MCP | **Restaurant order tickets**: the waiter writes the ticket, the kitchen cooks | Precise tool contracts, `isError` with a retryable flag, MCP server in .NET, least privilege | The model asks; your code acts | Left: waiter, ticket rail, kitchen. Right: model, tool call JSON, executor |
| 15 | The Refund That Must Happen Exactly Once | The process crashes mid-refund. Restarting must not pay twice. | Durable state; checkpointing; idempotency keys; outbox | **Numbered cheques**: a cheque can be presented twice but cashed only once | Checkpointed workflow, idempotent writes, outbox pattern. ShopSense story (54,990, paid once). | Agents that act need transactional discipline | Left: cheque book with serials. Right: checkpoint, crash, resume, idempotent write |
| 16 | Maker and Checker: Where Humans Must Stay in Charge | A high-value refund needs approval. Who approves, and how do we know it was them? | Human-in-the-loop; plan-approve-execute; excessive agency; audit | **Banking maker-checker rule** | Risk gate in code, interrupt for approval, authenticated reviewer role, decision record. ShopSense lesson: an unauthenticated approval endpoint. | Human oversight only works if the approval path is secured | Left: two-key vault. Right: agent proposes, gate, authenticated approver, execute |
| 17 | When One Agent Isn't Enough (and When It Is) | One giant prompt tries to triage, research, draft and check | Coordinator and subagents; critic loops; cost of coordination | **Newsroom**: editor, reporters, fact-checker | Coordinator with scoped subagents and a capped critic loop that escalates when the cap is hit | Multi-agent design is an org chart. Keep it small. | Left: newsroom desk layout. Right: coordinator, specialists, reviewer |
| **Arc 5: Earning trust** |||||||||
| 18 | It Passed the Demo. Will It Pass Monday? | Leadership asks, "How do we know it's good?" | Golden datasets; metrics per stage; LLM-as-judge and its biases; regression gates | **Driving test**: one impressive lap is not a licence | Golden set from real tickets, scorers for retrieval and answer, an eval gate in CI/CD, results committed before and after each change | If you can't show the before and after, you can't claim it's better | Left: driving test course. Right: test set, scorers, gate, deploy |
| 19 | The Document That Gave the AI Instructions | A supplier PDF contains hidden text telling the assistant to approve payment | Direct and indirect prompt injection; treating data as untrusted; output handling | **Fake "urgent payment" memo in the accounts-payable tray** | Mark retrieved and tool content as data, screen it, least-privilege tools, no autonomous payment. ShopSense lesson. | Every input channel is an attack surface | Left: forged memo in an inbox tray. Right: content, injection screen, sandboxed model, gated tools |
| 20 | Guardrails Are Layers, Not a Wall | "Can we just add a filter?" | Input and output checks; PII masking; policy checks; rate and cost limits | **Airport security**: several checkpoints, none of them perfect | Layered checks along the request path, with false-positive rates measured | No single control is enough; measure what each layer catches | Left: airport checkpoints. Right: request path with numbered control points |
| **Arc 6: Running it for real** |||||||||
| 21 | Fast in the Demo, 18 Seconds in Production | Users abandon the assistant during peak hours | TTFT, TPOT; streaming; sync vs async paths; latency budgets | **Restaurant service**: bread arrives at once, catering is prepared ahead | Latency budget per stage, SSE streaming from ASP.NET Core, async queue for long jobs | Perceived latency and real latency are designed separately | Left: courses arriving over time. Right: latency waterfall |
| 22 | You Can't Fix What You Can't See | One complaint, a million traces | Nested tracing; OpenTelemetry GenAI conventions; cost per span; cache hit rate as a signal | **Air traffic control**: each flight tracked, and the anomaly found | OpenTelemetry, Application Insights, nested spans per agent and tool, PII redaction | Observability for AI means tracing decisions, not just requests | Left: radar room. Right: trace tree with a highlighted slow span |
| 23 | The AI Bill Nobody Forecast | Finance asks why spend tripled this month | Token economics; model routing; exact, semantic and prompt caching; budgets and chargeback | **Hospital triage**: not every patient needs the specialist | Router by task type, caching layers, token quotas per team in the gateway | Cost is an architectural property | Left: triage desk. Right: router sending traffic to model tiers, cache in front |
| 24 | Changing a Prompt Is a Production Deployment | A "small wording tweak" on Friday breaks extraction on Monday | Prompt and model versioning; canaries; eval gates; rollback | **Recipe change at a restaurant chain**: test kitchen before every branch | Prompts as versioned artifacts, staged rollout, golden-set gate, quick rollback | Treat prompts and model versions like code | Left: test kitchen to branches. Right: version, eval, canary, full rollout |
| 25 | The Whole Contoso Platform, One Decision at a Time | "Show me the full architecture before we fund phase 2" | Reference architecture; deployment choices (public API, private cloud, self-hosted, data residency) | **City planning**: shared roads and utilities, not a well in every garden | End-to-end diagram built from Parts 1 to 24, with a decision log | Architecture is the record of trade-offs, not the boxes | Fold-out blueprint where each component is tagged with its Part number |

---

## Stage 3: Checking the sequence

**Prerequisites run forward only.** Part 1 (output is probabilistic, so validate it) underpins Parts 9, 14 and 18. Part 2 (statelessness, tokens) underpins Parts 11, 12 and 23. Part 3 (gateway) is the place where Parts 20, 22 and 23 later add controls. Parts 4 to 7 build retrieval before Part 8 secures it and Part 9 makes it verifiable. Agents (13 to 17) come after tools and state are understood. Evaluation (18) comes after there is enough to evaluate, although Parts 1, 7 and 9 already introduce measuring locally, so readers never meet a design without a way to test it.

**No topic is taught twice:**

| Concept | Taught in | Referenced in |
|---|---|---|
| Tokens, context window, statelessness | 2 | 11, 12, 23 |
| Structured output, validation | 1 | 9, 14, 18 |
| Gateway, retries, circuit breaker | 3 | 20, 22, 23 |
| Caching (exact, semantic, prompt) | 23 (prompt cache first mentioned in 12) | 21 |
| Retrieval metrics | 7 | 18 |
| Human approval | 16 | 15, 19 |
| Prompt injection | 19 | 8, 14, 20 |

**Gaps closed by the plan:** identity (8, 16), memory (2, 11), tools and MCP (14), durable execution (15), deep evaluation (7, 18), indirect injection (19), LLMOps (24), deployment and residency (25).

**Variety of analogies:** 25 analogies from banking, healthcare, travel, logistics, hospitality, journalism, hiring, education and aviation. None repeats, and none reuses the discarded series' "city library".

**Pacing for a daily series:** a weekday schedule gives 25 working days. Arcs end at Parts 3, 10, 12, 17, 20 and 25, which are natural points to post a short recap.

---

## Visual identity (applies to every image)

- **Style:** flat editorial illustration with clean lines and limited shading, in the spirit of business-newspaper explainers. No robots, glowing brains, circuit-board backgrounds or abstract "AI" swirls.
- **Layout:** landscape 1200 × 627 (LinkedIn article cover) or square 1080 × 1080 (feed post). Two panels: **left** the real-world analogy, **right** the AI architecture, and **centre** thin dashed connector lines pairing each analogy element with its technical counterpart.
- **Palette:** ink navy `#1F2A44` (lines, text), warm paper `#F6F1E7` (background), teal `#2A9D8F` (the "correct path"), amber `#E9A23B` (checks and decisions), coral `#D1495B` (failure and risk). Nothing else.
- **Typography:** labels in a geometric sans (Inter or IBM Plex Sans), sentence case, 2 to 4 words each, at most 8 labels per image.
- **Series mark:** small top-left tag "AI System Design · Part N". Bottom-right tag "Illustrative" whenever the right panel simplifies the real architecture.
- **People:** diverse, professional, neutral expressions, with no recognisable real people or brands.
