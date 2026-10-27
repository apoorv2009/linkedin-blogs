# Lesson 1.5: Why Is The First Search Result So Often Not The Best One?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 5](05-reranking.md).*

**The brief:** "Just send the top 20 results to the model."

**Level:** L3.

**Time:** About 20 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Explain the difference between a bi-encoder and a cross-encoder.
2. Design a two-stage retrieval: wide recall, then precise reranking.
3. Set shortlist sizes using a latency budget.
4. Place filters correctly relative to reranking.

---

## 1. The idea in plain words

A quick catalogue search finds books with promising titles. A careful librarian then reads a page of each next to your question and hands you the best three. Search is built for speed across everything. Reranking is built for judgement across a few.

---

## 2. The requirement that drives the decision

> **The passages sent to the model must be the few that best answer this question, not the many that loosely match it.**

---

## 3. Why search ranks imperfectly

Vector search uses a **bi-encoder**: the question and each chunk are turned into embeddings *separately*, then compared. That separation is what makes it fast enough to search 300,000 chunks. It is also why it misses nuance: it never looks at the question and the chunk side by side.

A **cross-encoder** feeds the question and one chunk into the model *together* and outputs a relevance score. It sees how each word in the question relates to each word in the passage. Far more accurate, far too slow to run on every chunk.

Analogy: comparing two people's CVs by their one-line summaries (bi-encoder) versus interviewing them for the specific job (cross-encoder).

---

## 4. The two-stage pattern

```
Stage 1: filters + hybrid search   -> top 50 candidates     (tens of ms)
Stage 2: cross-encoder reranking   -> top 5 passages        (100 to 300 ms)
Stage 3: context assembly          -> prompt for the model  (lesson 1.8)
```

Stage 1 is about **recall**: make sure the right passage is somewhere in the 50.
Stage 2 is about **precision**: make sure the right passage is in the 5.

---

## 5. The options

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A. No reranking | Speed, simplicity | Buried answers, more tokens sent | Search already ranks the answer first almost always |
| B. Cross-encoder | Accuracy for a small delay | Extra model, input length limits | Default for enterprise RAG |
| C. LLM reranking | Nuanced relevance rules | Cost, latency, run-to-run variation | Low volume, high value |

Examples for Option B include open-source models such as the BGE reranker family and hosted services such as Cohere Rerank. Treat them as examples to test, not as recommendations.

**Choice:** B.

---

## 6. Setting the shortlist sizes

Work backwards from the latency budget.

- Total budget for retrieval before the model starts: say 500 milliseconds [CHECK: your target].
- Hybrid search: about 50 milliseconds [CHECK].
- Reranker cost grows roughly linearly with the number of candidates. Measure the time for 50 on your hardware, then pick the largest shortlist that fits.
- Keep only as many passages after reranking as the context budget in lesson 1.8 allows, often 3 to 8.

Check recall at stage 1 separately: for your test questions, how often is the right passage inside the top 50? If it is often missing, the problem is in search or chunking, not reranking.

---

## 7. Where filters go

```
filters (country, access, effective date)  ->  search  ->  rerank  ->  context
```

Never rerank first and filter later. The reranker only knows relevance, not rules, and filtering late can empty the shortlist or, worse, let a forbidden passage through if a step is skipped. Access control is covered properly in lesson 1.6.

For a sponsor, the takeaway is that reranking usually lowers the cost of each answer, because the expensive model reads 5 passages instead of 20, so it is rarely the line item to cut.

---

## 8. What would change my mind

- Test results show the right passage is already first in search almost every time.
- The reranker cannot meet the latency budget on any available hardware.
- The use case needs relevance rules too complex for a cross-encoder, and volume is low enough for Option C.

---

## 9. Failure modes

| Failure | Symptom | Guard |
|---|---|---|
| Recall miss at stage 1 | Reranker cannot find what is not there | Measure recall@50 on test questions |
| Truncation | Deciding sentence cut off | Chunk sizes within the reranker's input limit |
| Late filtering | Wrong-country or forbidden passages | Filters before search |
| Fixed score threshold | Good passages dropped after documents change | Keep top N, review thresholds |
| Hosting gap | Reranker down, whole assistant down | Fallback to search order with a warning metric |

---

## 10. Rough numbers

| Item | Estimate |
|---|---|
| Candidates reranked | 50 [CHECK] |
| Reranking time | 100 to 300 ms on suitable hardware [CHECK] |
| Passages kept | 3 to 8 [CHECK] |
| Typical reranker input limit | Often around 512 tokens per question and passage pair [CHECK: model card] |

---

## 11. Words to know

| Term | Plain meaning |
|---|---|
| Recall | Did the right passage make it into the candidate list at all? |
| Precision | Are the passages we kept actually the right ones? |
| Bi-encoder | Encodes question and passage separately; fast |
| Cross-encoder | Reads question and passage together; accurate |
| recall@k | Share of questions where the right passage is in the top k |

---

## 12. Exercise

1. For 30 real questions, record the position of the correct passage in your search results.
2. Calculate recall@5, recall@20 and recall@50.
3. If recall@50 is high but recall@5 is low, you need reranking. If recall@50 is low, go back to lessons 1.3 and 1.4.

## Check your understanding

1. Why is a cross-encoder too slow to use as the main search?
2. Why must filters be applied before reranking?
3. What does it mean if recall@50 is poor?

---

**Next lesson (1.6):** *How Do You Stop A Salary Band Reaching The Wrong Screen?* Document-level access control in retrieval.
