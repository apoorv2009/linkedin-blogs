# Series 1: #JustAddRAG

*AI System Design, from the Whiteboard. Act 1: Frame the problem.*

**The brief:** "Just add RAG over our docs."

**Core promise:** Why a 2-week RAG demo takes 6 months to reach enterprise production.

**Core analogy:** Building a city library. A bookshelf at home works because you know every book. A city library needs a catalogue, an index, librarians who rank what is useful, rooms that need a badge, and a habit of removing old editions. RAG in an enterprise is the library, not the bookshelf.

**Course module:** Module 1: Enterprise RAG Architecture. Each write-up maps to one lesson, 1.1 to 1.10.

[Back to the programme roadmap](../ROADMAP.md)

---

## The running example

An HR policy and benefits assistant for a 20,000-person global enterprise. Every write-up in this series uses this example and only this example.

Employees ask it things like "How many days of annual leave do I get?", "Does my health plan cover physiotherapy?" and "What is the notice period in my country?"

What it has to read:

| Source | What it is like | Why it is hard |
|---|---|---|
| Employee handbooks (PDF) | Long, well written, updated once or twice a year | Several versions live in several folders |
| Country-specific labour law summaries | One per country, often written by local HR or counsel | They override the global handbook, and the assistant must know which applies |
| Benefits guides | Heavy on tables: plans, limits, eligibility | Tables break when turned into plain text |
| Salary bands | Spreadsheets, highly confidential | Must never reach someone who is not allowed to see them |
| Individual employee context | Country, grade, manager, start date, from the HR system of record | Personal data: needed for a correct answer, dangerous in the wrong place |

Working assumptions for the numbers in this series. Each one is a planning estimate, not a measurement:

| Assumption | Value |
|---|---|
| Employees | 20,000 (given in the brief) |
| Countries with their own rules | About 30 [CHECK] |
| Documents in scope | 3,000 to 5,000 [CHECK] |
| Questions on a normal day | About 1,500 [CHECK] |
| Questions on a peak day (open enrolment, year-end) | 5 to 10 times normal [CHECK] |

---

## Scope

**In this series:** ingestion pipelines, chunking, hybrid search (BM25 plus vectors), cross-encoder reranking, document-level access control, document versions and country overrides, context window bloat, using individual employee context safely in retrieval.

**Not in this series** (each has its own home, and write-ups here link to it):

- Prompt injection hidden inside PDFs, PII redaction, and On-Behalf-Of identity in tool calls: #DefendingAI.
- Measuring answer quality with the RAG Triad and regression tests: #AIEvalEngine.
- Fine-tuning vs in-context learning, prompt caching, and picking cheaper models: #ModelRouting.
- Hosting the vector database, rate limits and multi-region failover: #EnterpriseAIScale.

---

## Write-ups and publication tracker

Status is one of: **Planned** (outline ready), **Draft for review**, or **Published D Mon YYYY**.

| # | Write-up | Topic (the brief) | Level | Course lesson | Status | LinkedIn |
|---|---|---|---|---|---|---|
| 1 | [Why Does "Just Add RAG" Sound Like 2 Weeks But Take 6 Months?](01-curtain-raiser.md) | "Just add RAG over our HR docs. Two weeks?" | L1 to L5 | [1.1](01-curtain-raiser.course.md) | Draft for review | [Pending Publish] |
| 2 | [What Actually Happens To A PDF Before The Assistant Can Read It?](02-pdf-ingestion.md) | "We'll just upload the PDFs." | L3 | [1.2](02-pdf-ingestion.course.md) | Draft for review | [Pending Publish] |
| 3 | [How Small Can You Cut A Policy Before It Stops Making Sense?](03-chunking.md) | "Just split the documents into pages." | L3 | [1.3](03-chunking.course.md) | Draft for review | [Pending Publish] |
| 4 | [Why Does The Assistant Find "Parental Leave" But Miss "HR-PL-204"?](04-hybrid-search.md) | "It's AI search. It understands what we mean." | L3 | [1.4](04-hybrid-search.course.md) | Draft for review | [Pending Publish] |
| 5 | [Why Is The First Search Result So Often Not The Best One?](05-reranking.md) | "Just send the top 20 results to the model." | L3 | [1.5](05-reranking.course.md) | Draft for review | [Pending Publish] |
| 6 | [How Do You Stop A Salary Band Reaching The Wrong Screen?](06-access-control.md) | "Everyone can use it, but nobody sees what they shouldn't." | L4 | [1.6](06-access-control.course.md) | Draft for review | [Pending Publish] |
| 7 | [Which Copy Of The Leave Policy Is The True One?](07-versions-and-country-rules.md) | "The policy changed last week. Does the bot know?" | L4 | [1.7](07-versions-and-country-rules.course.md) | Draft for review | [Pending Publish] |
| 8 | [Why Does Giving The Model More Text Make The Answer Worse?](08-context-bloat.md) | "Give it everything. The context window is huge." | L3 | [1.8](08-context-bloat.course.md) | Draft for review | [Pending Publish] |
| 9 | [How Does The Assistant Know Your Country Without Knowing Too Much?](09-employee-context.md) | "It should know where I work and what my grade is." | L4 | [1.9](09-employee-context.course.md) | Draft for review | [Pending Publish] |
| 10 | [What Does Enterprise RAG Look Like On One Whiteboard?](10-finale.md) | "Show me the whole thing on one page before we fund it." | L5 | [1.10](10-finale.course.md) | Draft for review | [Pending Publish] |

---

## Outlines

**1. Why Does "Just Add RAG" Sound Like 2 Weeks But Take 6 Months?** (curtain raiser)
The demo works on the happy path. The hidden part of the library decides whether it survives. Lesson 1.1 compares three starting architectures: paste everything into the prompt, the classic demo, and a layered retrieval pipeline.

**2. What Actually Happens To A PDF Before The Assistant Can Read It?**
Library analogy: cataloguing a donated box of books. Options: plain text extraction, layout-aware parsing, OCR for scans. Covers tables, headers and footers, metadata (country, version, owner), and why re-ingesting 5,000 documents is the cost you pay for skipping metadata on day one.

**3. How Small Can You Cut A Policy Before It Stops Making Sense?**
Library analogy: tearing pages out of a book versus photocopying whole chapters. Options: fixed-size chunks, structure-aware chunks (by heading), parent and child chunks. Covers overlap, keeping a table with its caption, and chunk size versus answer quality.

**4. Why Does The Assistant Find "Parental Leave" But Miss "HR-PL-204"?**
Library analogy: asking a librarian by topic versus looking up a call number. Options: vector search only, keyword (BM25) only, hybrid with rank fusion. Covers policy codes, acronyms, country names and multi-lingual queries.

**5. Why Is The First Search Result So Often Not The Best One?**
Library analogy: the librarian who reads the first page of each book before handing it over. Options: no reranking, cross-encoder reranking, LLM-based reranking. Covers the latency and cost budget of reranking 50 candidates.

**6. How Do You Stop A Salary Band Reaching The Wrong Screen?**
Library analogy: the rare books room that needs a badge. Options: separate indexes per audience, filter after search, filter during search (security trimming). Covers syncing permissions from the source system and what happens when someone changes role.

**7. Which Copy Of The Leave Policy Is The True One?**
Library analogy: removing old editions and shelving local editions next to the national one. Options: overwrite in place, versioned documents with effective dates, a country override layer. Covers deletion, effective dates, and "the global rule unless your country says otherwise".

**8. Why Does Giving The Model More Text Make The Answer Worse?**
Library analogy: the librarian who hands you 40 books when you asked one question. Options: fixed top-k, a token budget, compression or summarisation of retrieved text. Covers "lost in the middle", cost per question and latency. (Prompt caching lives in #ModelRouting.)

**9. How Does The Assistant Know Your Country Without Knowing Too Much?**
Library analogy: a librarian who knows your membership type but not your diary. Options: index personal records, fetch a small profile at question time, let the model call the HR system. Covers minimum necessary context and audit. (Identity delegation in tool calls lives in #DefendingAI.)

**10. What Does Enterprise RAG Look Like On One Whiteboard?** (finale)
The full architecture on one page, the build order, and the questions to ask before funding it. Links forward to #ModelRouting.
