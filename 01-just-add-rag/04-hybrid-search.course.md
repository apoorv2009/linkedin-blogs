# Lesson 1.4: Why Does The Assistant Find "Parental Leave" But Miss "HR-PL-204"?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 4](04-hybrid-search.md).*

**The brief:** "It's AI search. It understands what we mean."

**Level:** L3.

**Time:** About 25 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Explain the strengths of keyword search and meaning search in one sentence each.
2. Merge two ranked lists with Reciprocal Rank Fusion.
3. Use metadata filters to narrow search before ranking.
4. Plan for multiple languages in both kinds of index.

---

## 1. The idea in plain words

A librarian understands descriptions. A call number points to one exact shelf. A good library offers both, and a good HR assistant does too.

---

## 2. The requirement that drives the decision

> **The assistant must find the right passage whether the employee describes the topic in everyday words or quotes an exact code, name or term.**

---

## 3. How each kind of search works

### Meaning search (dense vectors)

1. Each chunk is turned into an embedding by an embedding model.
2. The question is turned into an embedding by the same model.
3. The index returns chunks whose embeddings are closest, usually by cosine similarity.
4. Large indexes use approximate nearest neighbour structures such as HNSW to stay fast.

Strong on: paraphrase, synonyms, cross-language matches with a multilingual model.
Weak on: codes, IDs, rare names, exact numbers.

### Keyword search (BM25)

1. Text is split into terms using language rules (lowercasing, stemming, compound splitting).
2. Each chunk is scored on how many question terms it contains, with rare terms worth more and very long chunks penalised a little.

Strong on: exact codes, names, acronyms.
Weak on: different words for the same idea.

---

## 4. The options

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A. Meaning only | Conversational questions | Codes and rare terms | No codes or names in real queries |
| B. Keywords only | Exact matches | Paraphrase | Expert users who search with exact terms |
| C. Hybrid with fusion | Both | Two indexes, tuning | General employee population |

**Choice:** C.

---

## 5. Merging with Reciprocal Rank Fusion

Raw scores from the two searches are on different scales, so adding them is misleading. Reciprocal Rank Fusion uses positions instead:

```
score(chunk) = sum over each list of  1 / (k + rank_in_that_list)
```

`k` is a constant, commonly 60, which stops the very top positions from dominating. A chunk ranked 1st by keywords and 10th by meaning beats a chunk ranked 3rd by only one of them.

Analogy: two judges each rank the contestants. You do not add their scores, which might be out of 10 and out of 100. You reward contestants both judges placed high.

Some engines also offer weighted score blending or learned fusion. Start with RRF; it needs no tuning to be reasonable.

---

## 6. Filters before ranking

Labels from lesson 1.2 become filters:

- `country IN (employee_country, "GLOBAL")`
- `doc_type IN ("handbook", "policy", "benefits_guide")`
- `language` where relevant

Filtering first means both searches only consider eligible chunks. Access control is also a filter, covered properly in lesson 1.6. Effective dates are covered in lesson 1.7.

For a sponsor, the point to remember is that filters are cheap to add when labels exist and expensive when they do not, which is why lesson 1.2 comes first in the budget.

---

## 7. Languages

- **Keyword index:** choose the language analyser per document or per field. German needs compound handling. Japanese and Chinese need proper word segmentation.
- **Meaning index:** use a multilingual embedding model if employees may ask in one language about documents written in another. Test it on your languages; quality varies.
- **Codes and numbers** behave the same in every language, which is one more reason to keep keyword search.

---

## 8. What would change my mind

- Real query logs show almost no codes, names or acronyms: meaning search alone may be enough.
- Users are HR specialists who search like lawyers: keyword search with strong filters may be enough.

---

## 9. Failure modes

| Failure | Symptom | Guard |
|---|---|---|
| Codes missed | "Not found" for valid policy codes | Keyword index |
| Synonyms missed | "Maternity" does not find "parental" | Meaning index |
| Score scale mismatch | One search always wins | Rank-based fusion |
| Wrong analyser | Poor results in one language | Per-language analysers, tests per language |
| Filters skipped | Other countries' rules in results | Labels as filters, not query words |

---

## 10. Rough numbers

| Item | Estimate |
|---|---|
| Index size | 200,000 to 400,000 chunks [CHECK] |
| Hybrid query latency | Tens of milliseconds [CHECK: engine and hardware] |
| Candidates per list before fusion | 25 to 50 [CHECK] |
| RRF constant k | 60 is a common default |

---

## 11. Words to know

| Term | Plain meaning |
|---|---|
| Embedding | Numbers that capture meaning |
| Dense retrieval / vector search | Finding chunks by closeness of embeddings |
| BM25 | Keyword ranking that rewards rare matching terms |
| HNSW | A common structure for fast approximate vector search |
| Hybrid search | Keyword and vector search together |
| Reciprocal Rank Fusion | Merging ranked lists by position |
| Analyser | Language rules that split text into searchable terms |

---

## 12. Exercise

1. Write 20 questions about a document set you know: 10 conversational, 10 with codes or names.
2. Run them through a vector search and a keyword search (many engines let you try both locally).
3. Merge with RRF by hand for five questions. Did the right chunk rise?

## Check your understanding

1. Why can't you simply add a BM25 score and a cosine similarity?
2. Why should "Germany" in a question become a filter?
3. What changes for keyword search when documents are in German?

---

**Next lesson (1.5):** *Why Is The First Search Result So Often Not The Best One?* Reranking and its latency budget.
