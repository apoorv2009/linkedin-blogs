# Lesson 1.10: What Does Enterprise RAG Look Like On One Whiteboard?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 10](10-finale.md). Module summary.*

**The brief:** "Show me the whole thing on one page before we fund it."

**Level:** L5. The full design, ready for a design review.

**Time:** About 30 minutes, plus the capstone exercise.

---

## What you will be able to do after this lesson

1. Draw the full enterprise RAG architecture as two flows: ingestion and query.
2. Justify each component with the requirement it serves.
3. Present a build order and explain why it is in that order.
4. Answer the funding questions a sponsor will ask.

---

## 1. The idea in plain words

A city library has a catalogue desk, where new books are checked, labelled and shelved, and a reading room, where visitors are helped to find what they need. Enterprise RAG has the same two halves. Most demo trouble comes from building a reading room with no catalogue desk.

---

## 2. The requirement, restated

From lesson 1.1:

> **Every answer must be correct for the person asking (their country, their grade, the current policy), must show where it came from, and must never use a document that person is not allowed to see.**

Every component below exists to serve a part of that sentence.

---

## 3. The architecture

```
INGESTION (on document change)
  Sources ──> Discover + fingerprint ──> Parse (layout-aware) ──> Clean
          ──> Label (country, version, dates, precedence, access)
          ──> Chunk (by heading, parent/child) ──> Embed ──> Hybrid index
          ──> Tombstones + reconciliation for deletions

QUERY (on every question)
  Employee ──> Identity + groups + small profile (HR system, as the user)
           ──> Filters: access, country, effective date
           ──> Hybrid search (keywords + vectors, rank fusion) ── top 50
           ──> Cross-encoder rerank ── top ~5
           ──> Context assembly (budget, best first, overrides grouped, labels)
           ──> Language model ──> Answer with sources
           ──> Own-data tools (leave balance, enrolments) as the user
```

| Part of the requirement | Components that serve it | Lessons |
|---|---|---|
| Correct for the person | Profile, country filters, precedence | 1.7, 1.9 |
| Current policy | Fingerprints, effective dates, tombstones | 1.2, 1.7 |
| Shows the source | Labels carried into context | 1.2, 1.8 |
| Never a forbidden document | ACL labels, security trimming, sensitive data kept out | 1.6 |
| Finds the right passage | Parsing, chunking, hybrid search, reranking | 1.2 to 1.5 |
| Fast and affordable | Rerank shortlist, context budget | 1.5, 1.8 |

---

## 4. Default choices and when to change them

| Component | Default | Change when | Lesson |
|---|---|---|---|
| Parsing | Layout-aware, vision fallback | Plain prose, or structured source available | 1.2 |
| Metadata | Fixed record from sprint one | Never postpone | 1.2 |
| Chunking | Structure-aware, parent and child for long sections | Short FAQ content | 1.3 |
| Search | Hybrid with RRF | No codes or names in real queries | 1.4 |
| Reranking | Cross-encoder, 50 to about 5 | Search already precise | 1.5 |
| Access | Security trimming, default deny | Two non-overlapping audiences | 1.6 |
| Versions | Effective dates, precedence as data | Single country, no future dates | 1.7 |
| Context | Token budget, best first | Whole-document question types | 1.8 |
| Personal data | Profile at question time, narrow tools | HR platform assistant available | 1.9 |

---

## 5. The build order

| Phase | Builds | Why in this order |
|---|---|---|
| 1 | Ingestion, metadata, deletion handling | Only phase whose mistakes force full re-processing |
| 2 | Hybrid search with access, country and date filters | Safety and correctness before polish |
| 3 | Reranking and context budget | Quality and cost, safe to tune later |
| 4 | Profile and own-data tools | Depends on HR system access approvals |
| 5 | Answer quality measurement and launch gates | See #AIEvalEngine |

Things that are not engineering but sit on the critical path:

- A named HR owner per country for dates and precedence.
- Security sign-off on permission sync and default deny.
- Privacy and works council review of the profile fields and logs.
- Access to the HR system's API with on-behalf-of identity (see #DefendingAI).

For a sponsor, these four non-engineering items usually set the launch date more than any technical choice, so they should be started in the first week alongside phase 1.

---

## 6. Rough numbers for the whole system

| Item | Estimate |
|---|---|
| Documents | 3,000 to 5,000 [CHECK] |
| Pages | About 100,000 [CHECK] |
| Chunks | 200,000 to 400,000 [CHECK] |
| Raw vector storage at 1,536 dimensions | 1.2 to 2.4 GB [CHECK: model dimension] |
| Full parse cost | Roughly 1,000 to 1,500 US dollars per run [CHECK] |
| Retrieval latency (filters, search, rerank) | Roughly 150 to 400 ms [CHECK] |
| Input tokens per question | About 3,000 to 5,000 including instructions [CHECK] |
| Questions per day | About 1,500 normal, 5 to 10 times at peak [CHECK] |

Scaling this across regions, handling provider rate limits and choosing where the vector database runs are covered in #EnterpriseAIScale. Using cheaper models for simple questions is covered in #ModelRouting.

---

## 7. The funding questions, with short answers

| Question | Short answer |
|---|---|
| Why not just the demo? | It cannot tell countries, versions or permissions apart. |
| What is the riskiest part? | Access control and personal data. Designed first, not last. |
| What takes longest? | Labels, permissions and country owners, not code. |
| What does each answer cost? | Set by the context budget; calculate from tokens per question and your provider's price. |
| Could we buy instead? | Yes, if a platform meets the same requirement. Use this design as the checklist. |
| How will we know it works? | Measure before launch. See #AIEvalEngine. |

---

## 8. What would change my mind

- An existing HR platform assistant meets the requirement in section 2: buy and integrate.
- The real question mix is mostly personal live data: lead with tools (lesson 1.9), not document search.
- The library is small and single-country: a simpler subset (lessons 1.2, 1.4, 1.6) may be enough.

---

## 9. Top failure modes across the module

| Failure | Lesson |
|---|---|
| Flattened tables and missing scans | 1.2 |
| Rules separated from their conditions | 1.3 |
| Codes and names not found | 1.4 |
| Right document, wrong passage | 1.5 |
| Forbidden document retrieved | 1.6 |
| Old, draft or wrong-country policy | 1.7 |
| Right passage ignored in a long prompt | 1.8 |
| Personal data over-shared or logged | 1.9 |

---

## 10. Words to know (module recap)

| Term | Plain meaning | Lesson |
|---|---|---|
| RAG | Find the right text, then let the model answer with it | 1.1 |
| Ingestion pipeline | From source document to searchable chunks | 1.2 |
| Metadata | Labels like country, version, access | 1.2 |
| Chunk | One searchable piece | 1.3 |
| Hybrid search | Keywords and meaning together | 1.4 |
| Cross-encoder | Accurate reranking model | 1.5 |
| Security trimming | Filtering forbidden items during search | 1.6 |
| Effective date, precedence | Which version and which rule applies | 1.7 |
| Context budget | How much text the model gets | 1.8 |
| Minimum necessary context | Only the personal facts needed | 1.9 |

---

## 11. Capstone exercise

Design a RAG system for a different domain you know (IT runbooks, product manuals, legal clauses). Produce one page with:

1. The one requirement that drives the design.
2. Both flows, ingestion and query, as box-and-arrow sketches.
3. The defaults table from section 4, with your choice and a one-line reason per row.
4. A build order with the non-engineering items on the critical path.
5. Rough numbers, every one marked [CHECK] until you have measured it.

Present it to a colleague as if it were a design review. Note the first three questions they ask.

---

**Next module (2):** *#ModelRouting.* Stop using the most expensive model for every question: routers, cascades and caching.
