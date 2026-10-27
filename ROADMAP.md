# AI System Design, from the Whiteboard

**Programme theme:** Architecting resilient, scalable, and cost-aware enterprise AI systems without falling for demo-ware traps.

**Profile:** System design. *Design it like you would defend it in a design review.*

**Author:** .NET/AI Architect, 17+ years building enterprise microservices, vector search pipelines and multi-agent workflows.

**Who it is for:** Engineers learning AI system design, and business leaders who sign off on it. If a 16-year-old student and a CEO can both follow a write-up, it is ready to publish.

**Roadmap last updated:** 27 Oct 2026

---

## How every write-up is built

Each write-up starts with a business brief, said out loud by a stakeholder, and walks the same spine:

1. The requirement that drives the decision.
2. Two or three architectural options.
3. A plain-words trade-off table: Option | Good at | Bad at | Pick it when.
4. The choice, and why.
5. What would change my mind.
6. Failure modes.
7. Rough numbers for latency, cost and scale, marked [CHECK] until verified.

Every write-up has a matching course lesson. One running example per series, start to finish.

## Level scale

| Level | What it looks like |
|---|---|
| L1 | Single user, single API call, an isolated prompt. |
| L2 | A small pilot: one team, one data source, friendly users. |
| L3 | Team-level deployment, custom data sources, real active users. |
| L4 | Organisation-wide: several data sources, access rules, an on-call rota. |
| L5 | Multi-region enterprise system, highly regulated, dozens of interconnected backend APIs. |

L1, L3 and L5 are the programme's anchor levels. L2 and L4 are the steps in between.

---

## Acts and series

| Act | Purpose | Series |
|---|---|---|
| Act 1 | Frame the problem | 1, 2 |
| Act 2 | Build the core | 3, 4 |
| Act 3 | Make it safe and measurable | 5, 6 |
| Act 4 | Run it at scale | 7 to 12 |

| # | Series | Topic (the brief) | Core promise | Core analogy | Running example | Act | Status |
|---|---|---|---|---|---|---|---|
| 1 | [#JustAddRAG](01-just-add-rag/README.md) | "Just add RAG over our docs." | Why a 2-week RAG demo takes 6 months to reach enterprise production. | Building a city library | HR policy and benefits assistant for 20,000 employees | 1 | In progress |
| 2 | #ModelRouting | "Just use the best model for everything." | Stop using expensive top-tier LLMs for simple classification and extraction tasks. | A restaurant kitchen: the head chef does not chop onions | Multi-tier customer support ticket processing | 1 | Not started |
| 3 | #AgenticAI | "Can the bot just fix the incident itself?" | Move from passive chatbots to autonomous systems that do not loop endlessly or break production. | Hiring a new employee | Automated IT infrastructure incident resolution bot | 2 | Not started |
| 4 | #DefendingAI | "Make sure it can't leak customer data." | Secure non-deterministic AI pipelines against prompt injection, data leaks and unauthorised access. | Multi-stage municipal water filtration | Financial advisory assistant with access to customer bank accounts | 2 | Not started |
| 5 | #AIEvalEngine | "Prove it's accurate before we ship it." | If you cannot measure your LLM system's accuracy deterministically, you cannot ship it. | Assembly line quality control | Medical record summarisation engine | 3 | Not started |
| 6 | #EnterpriseAIScale | "Now roll it out to every country." | Scale from single-prompt prototypes to multi-region, fault-tolerant AI platforms. | From one food truck to a global franchise | Global multi-lingual e-commerce recommendation and support | 3 | Not started |
| 7 | #AIGovernance (proposed) | "Legal needs to sign this off." | Make an AI system auditable before a regulator or a court asks. | Building permits and inspections | CV screening assistant for a recruitment agency | 4 | Not started |
| 8 | #LLMOps (proposed) | "Can we change the prompt on Friday afternoon?" | Ship prompt and model changes without breaking what already works. | A pharmacy changing a medicine formula: trials, batch numbers, recalls | Contract clause extraction service for a legal team | 4 | Not started |
| 9 | #LegacyMeetsAI (proposed) | "Plug the AI into our 15-year-old ERP." | Add AI to old, important systems without rewriting them. | Fitting a modern kitchen into a listed heritage building | AI order assistant in front of a legacy .NET ERP at a manufacturer | 4 | Not started |
| 10 | #MultimodalAI (proposed) | "It needs to read invoices and take phone calls too." | Design for photos, scans and voice, each with its own cost and latency. | A hospital triage desk that takes walk-ins, phone calls and letters | Insurance claims intake | 4 | Not started |
| 11 | #SovereignAI (proposed) | "Our data cannot leave the country." | Run models where the law says the data must live. | Running your own power generator instead of using the grid | Public-sector citizen services assistant | 4 | Not started |
| 12 | #AIPlatform (proposed) | "Every team is building its own chatbot. Make it stop." | Turn many one-off AI projects into one shared platform. | City utilities: shared roads, water and power, not a well in every garden | Internal AI platform for a 50-team enterprise | 4 | Not started |

Series 7 to 12 are proposals. Each one is confirmed and planned in detail before its curtain raiser is written.

### Topic boundaries (so nothing is taught twice)

| Concept | Taught in | Other series link to it |
|---|---|---|
| Ingestion, chunking, hybrid search, reranking, document-level access control, context bloat | #JustAddRAG | All |
| Prompt caching, KV-cache reuse, fine-tuning vs in-context learning, cascading models | #ModelRouting | #JustAddRAG, #EnterpriseAIScale |
| Agent loops, tool permissions, persistent memory, human-in-the-loop | #AgenticAI | #DefendingAI, #LegacyMeetsAI |
| Prompt injection (including via PDFs), PII redaction, On-Behalf-Of identity in tool calls | #DefendingAI | #JustAddRAG, #AgenticAI |
| RAG Triad, LLM-as-a-Judge, regression tests, tracing | #AIEvalEngine | #JustAddRAG, #LLMOps |
| Rate limits and 429 fallbacks, streaming, vector DB hosting, FinOps chargebacks | #EnterpriseAIScale | #AIPlatform, #SovereignAI |

---

## Master publication tracker

Status is one of: **Planned** (outline ready), **Draft for review**, or **Published D Mon YYYY**.

### Series 1: #JustAddRAG

| # | Write-up | Topic (the brief) | Level | Course lesson | Status | LinkedIn |
|---|---|---|---|---|---|---|
| 1 | [Why Does "Just Add RAG" Sound Like 2 Weeks But Take 6 Months?](01-just-add-rag/01-curtain-raiser.md) | "Just add RAG over our HR docs. Two weeks?" | L1 to L5 | [1.1](01-just-add-rag/01-curtain-raiser.course.md) | Draft for review | [Pending Publish] |
| 2 | [What Actually Happens To A PDF Before The Assistant Can Read It?](01-just-add-rag/02-pdf-ingestion.md) | "We'll just upload the PDFs." | L3 | [1.2](01-just-add-rag/02-pdf-ingestion.course.md) | Draft for review | [Pending Publish] |
| 3 | [How Small Can You Cut A Policy Before It Stops Making Sense?](01-just-add-rag/03-chunking.md) | "Just split the documents into pages." | L3 | [1.3](01-just-add-rag/03-chunking.course.md) | Draft for review | [Pending Publish] |
| 4 | [Why Does The Assistant Find "Parental Leave" But Miss "HR-PL-204"?](01-just-add-rag/04-hybrid-search.md) | "It's AI search. It understands what we mean." | L3 | [1.4](01-just-add-rag/04-hybrid-search.course.md) | Draft for review | [Pending Publish] |
| 5 | [Why Is The First Search Result So Often Not The Best One?](01-just-add-rag/05-reranking.md) | "Just send the top 20 results to the model." | L3 | [1.5](01-just-add-rag/05-reranking.course.md) | Draft for review | [Pending Publish] |
| 6 | [How Do You Stop A Salary Band Reaching The Wrong Screen?](01-just-add-rag/06-access-control.md) | "Everyone can use it, but nobody sees what they shouldn't." | L4 | [1.6](01-just-add-rag/06-access-control.course.md) | Draft for review | [Pending Publish] |
| 7 | [Which Copy Of The Leave Policy Is The True One?](01-just-add-rag/07-versions-and-country-rules.md) | "The policy changed last week. Does the bot know?" | L4 | [1.7](01-just-add-rag/07-versions-and-country-rules.course.md) | Draft for review | [Pending Publish] |
| 8 | [Why Does Giving The Model More Text Make The Answer Worse?](01-just-add-rag/08-context-bloat.md) | "Give it everything. The context window is huge." | L3 | [1.8](01-just-add-rag/08-context-bloat.course.md) | Draft for review | [Pending Publish] |
| 9 | [How Does The Assistant Know Your Country Without Knowing Too Much?](01-just-add-rag/09-employee-context.md) | "It should know where I work and what my grade is." | L4 | [1.9](01-just-add-rag/09-employee-context.course.md) | Draft for review | [Pending Publish] |
| 10 | [What Does Enterprise RAG Look Like On One Whiteboard?](01-just-add-rag/10-finale.md) | "Show me the whole thing on one page before we fund it." | L5 | [1.10](01-just-add-rag/10-finale.course.md) | Draft for review | [Pending Publish] |

### Series 2 to 12

Write-ups are planned in detail when the series before them reaches its finale. Rows are added here at that point.

---

## Post-publish update loop

When a write-up goes live on LinkedIn and the link is shared:

1. Set its status to `Published D Mon YYYY` in this file and in the series README.
2. Replace `[Pending Publish]` with `[Read on LinkedIn](<URL>)` in both files.
3. Update the pinned series index block inside the article file, and in every later article of that series.
4. When write-up 1 of a series is published, also replace `[Pending Publish]` in the "*New to the series?*" line of every later write-up in that series.
