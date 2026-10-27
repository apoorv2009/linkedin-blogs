# Roadmap

Writing types in this programme:

1. Curtain raiser: write-up 1 of each series. Sets the scene, shows what the demo hides.
2. Middle write-up: one design decision per write-up, defended like a design review.
3. Finale: the full system checklist for the series, plus honest lessons.
4. Course companion: the longer lesson behind each write-up. Not for LinkedIn.

## 1. AI System Design, from the whiteboard

Architecting resilient, scalable, and cost-aware enterprise AI systems without falling for demo-ware traps.

The storyline:

- **Act 1. Frame the problem.** Before building anything, learn to read the brief.
  - Series 1: "Build us an assistant that answers HR questions for 20,000 staff"
  - Series 2: "Let the sales team ask our data warehouse questions in plain English"
  - Series 3: "Pick an AI model for us, and don't lock us in"
- **Act 2. Build the core.** The parts that do the actual work.
  - Series 4: "Make 2 million product descriptions searchable by meaning"
  - Series 5: "Automate invoice processing across 300 suppliers' formats"
  - Series 6: "Our support team needs an agent that can actually issue refunds"
- **Act 3. Make it safe and measurable.** Prove it works, and prove it can't be talked into things.
  - Series 7: "Stop the chatbot from leaking customer data"
  - Series 8: "How do we know the new model version is better, not just different?"
  - Series 9: "Prove to the auditors how the AI reached that answer"
- **Act 4. Run it at scale.** Keep it fast, affordable, and legal when it grows.
  - Series 10: "The AI bill tripled last quarter. Fix it without breaking anything"
  - Series 11: "Roll it out to 14 countries, each with its own data rules"
  - Series 12: "Get five teams' AI agents to work together without chaos"

| Series | Topic | Level (L1 to L5) | What it covers | Status | Why a full series, not one article |
|---|---|---|---|---|---|
| *Act 1. Frame the problem* | | | | | |
| 1 | "Build us an assistant that answers HR questions for 20,000 staff" | L4 | Scope and question types, where knowledge lives (prompt, search, or fine-tuning), document ingestion, country and role context, permission-aware search, keyword vs meaning-based search, saying "I'm not sure", answer quality testing, keeping policies fresh, cost and speed. Example: an HR policy and benefits assistant for a 20,000-person global company. Plan: [01-hr-assistant/](01-hr-assistant/README.md) | Planned (curtain raiser drafted) | Each hidden part (permissions, country rules, freshness, testing) is its own design decision with real trade-offs. One article could only list them. |
| 2 | "Let the sales team ask our data warehouse questions in plain English" | L3 | Turning questions into database queries vs a curated business layer, schema grounding, query safety, row-level security, confident wrong numbers. Example: a sales analytics assistant over a retail data warehouse with hundreds of tables. Plan: [02-sales-data-questions/](02-sales-data-questions/README.md) (not written yet) | Idea | A wrong number looks exactly like a right one. Catching that needs several layers, each worth its own write-up. |
| 3 | "Pick an AI model for us, and don't lock us in" | L4 | Model selection criteria, the gateway pattern, prompt portability, fallbacks, data and contract terms. Example: a shared model gateway for a bank's internal AI use cases. Plan: [03-model-choice/](03-model-choice/README.md) (not written yet) | Idea | "Which model?" is really five questions: quality, cost, speed, data terms, and exit plan. |
| *Act 2. Build the core* | | | | | |
| 4 | "Make 2 million product descriptions searchable by meaning" | L3 | Vector index choices at scale, re-embedding cost, multilingual search, filtering, speed under load. Example: product search for an online retailer with 2 million items in 6 languages. Plan: [04-product-search/](04-product-search/README.md) (not written yet) | Idea | Search that works on 3,000 documents behaves very differently on 2 million. Scale changes every choice. |
| 5 | "Automate invoice processing across 300 suppliers' formats" | L3 | Document extraction, structured output, confidence scores, human review queues, validation against the finance system. Example: accounts payable intake for a manufacturer. Plan: [05-invoice-intake/](05-invoice-intake/README.md) (not written yet) | Idea | The happy path is easy. The 300 weird formats and the review queue are the project. |
| 6 | "Our support team needs an agent that can actually issue refunds" | L4 | Tool design, permissions per action, approval steps, safe retries, undo, spending limits. Example: a support agent for a telecom with refund, plan-change, and cancellation tools. Plan: [06-refund-agent/](06-refund-agent/README.md) (not written yet) | Idea | Reading is safe. Acting is not. Each action needs its own guard rails. |
| *Act 3. Make it safe and measurable* | | | | | |
| 7 | "Stop the chatbot from leaking customer data" | L4 | Prompt injection, data leakage, input and output filters, red-team testing, personal data handling. Example: a public insurance claims chatbot. Plan: [07-chatbot-safety/](07-chatbot-safety/README.md) (not written yet) | Idea | Attacks come in many shapes. No single filter stops them all, so the defence is layered. |
| 8 | "How do we know the new model version is better, not just different?" | L4 | Test sets, automated grading and its limits, regression gates, side-by-side trials, human review. Example: an internal contract-review assistant for a legal team. Plan: [08-model-upgrades/](08-model-upgrades/README.md) (not written yet) | Idea | Measuring language quality is hard and easy to fake. It needs a full method, not a tip. |
| 9 | "Prove to the auditors how the AI reached that answer" | L5 | Tracing, logging, data lineage, retention, explanations, regulatory duties. Example: a credit-decision support assistant at a lender. Plan: [09-audit-trail/](09-audit-trail/README.md) (not written yet) | Idea | Audit needs evidence from every layer, collected before anyone asks for it. |
| *Act 4. Run it at scale* | | | | | |
| 10 | "The AI bill tripled last quarter. Fix it without breaking anything" | L4 | Caching, model routing, prompt trimming, batching, budgets, cost per answer. Example: a customer service platform handling 1 million conversations a month. Plan: [10-ai-bill/](10-ai-bill/README.md) (not written yet) | Idea | Every saving trades against quality or speed. Each trade needs its own test. |
| 11 | "Roll it out to 14 countries, each with its own data rules" | L5 | Data residency, multi-region deployment, regional model availability, failover, local language and law. Example: a medical-information assistant for a pharmaceutical company. Plan: [11-fourteen-countries/](11-fourteen-countries/README.md) (not written yet) | Idea | Every country adds a rule, and the rules collide. Untangling them takes many steps. |
| 12 | "Get five teams' AI agents to work together without chaos" | L5 | Orchestration patterns, contracts between agents, shared protocols, failure isolation, end-to-end tracing. Example: an order-to-cash workflow at a logistics company with quoting, booking, and invoicing agents. Plan: [12-agent-teams/](12-agent-teams/README.md) (not written yet) | Idea | One agent is a program. Five agents owned by five teams is an organisation problem as well as a technical one. |

Level scale: L1 = one user, a single prompt or API call. L3 = one team, real documents, an active user base. L5 = multi-region, highly regulated, dozens of interconnected systems. L2 and L4 sit between their neighbours.

### Notes

- **Basics covered elsewhere:** none yet. This is the first programme. When a later series needs an idea already explained here, it links back instead of repeating it. Planned links:
  - Keyword vs meaning-based search basics live in Series 1 write-up 7. Series 4 links there and covers only scale.
  - Answer quality testing basics live in Series 1 write-up 9. Series 8 links there and covers model upgrades.
  - Cost and speed for a single assistant live in Series 1 write-up 11. Series 10 links there and covers cost at platform scale.
- **External inspiration:** the "requirement, options, trade-offs, decision" spine borrows from the Architecture Decision Record idea, first described by Michael Nygard in "Documenting Architecture Decisions" (2011).
- **Retired:** none yet.
