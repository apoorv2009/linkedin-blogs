# Lesson 1.3: How Small Can You Cut A Policy Before It Stops Making Sense?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 3](03-chunking.md).*

**The brief:** "Just split the documents into pages."

**Level:** L3.

**Time:** About 20 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Explain why chunk boundaries decide answer completeness.
2. Compare fixed-size, structure-aware and parent and child chunking.
3. Apply heading paths and table rules to a chunker.
4. Test a chunking strategy without building the whole system.

---

## 1. The idea in plain words

Search returns pieces, not documents. Whatever ends up in one piece is all the model sees of that part of the document. If a rule and its exception land in different pieces, only luck brings them back together.

Analogy: handing visitors photocopies instead of books. The skill is choosing where the photocopy starts and stops.

---

## 2. The requirement that drives the decision

> **Each chunk must make sense on its own, and a rule must never be separated from the conditions that change it.**

---

## 3. The options in more detail

### A. Fixed-size chunks

Cut every N tokens with an overlap of M tokens.

- Simple to build and reason about.
- Ignores meaning. The cut lands wherever the counter says.
- Overlap helps with sentences on the boundary, but not with a condition two paragraphs away.

### B. Structure-aware chunks

Use the parsed structure from lesson 1.2: headings, lists, tables.

- One section becomes one chunk where it fits a size limit.
- Oversized sections are split at paragraph boundaries, never mid-sentence.
- Undersized neighbours under the same heading can be merged.

### C. Parent and child chunks

- Index small "child" chunks (a paragraph, a table row group) for precise matching.
- Store a pointer to the "parent" (the whole section).
- At answer time, return the parent of each matching child, removing duplicates.

Analogy: the index card points to the shelf, and you take the whole folder off the shelf.

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A | Any text, simple | Splits meaning | No usable structure |
| B | Keeps rules whole | Uneven sizes, relies on parsing | Structured policies |
| C | Precision plus completeness | Storage and complexity | Long sections with buried details |

**Choice:** B as the base, C for sections above the size limit.

---

## 4. Two rules that matter more than chunk size

**Heading path.** Prefix each chunk with its position in the document:

```
[Employee Handbook, Germany > Leave > Carry-over of annual leave]
Unused annual leave of up to 5 days may be carried over...
```

This helps both keyword and meaning search, and helps the model know where the text sits. Some teams go further and add a one-sentence summary of the document to each chunk. Test whether it helps before paying for it.

**Tables.** Keep a table in one chunk if it fits. If it does not, split by rows and repeat the header row in every piece, so "Gold | 1,500" never appears without its column names.

---

## 5. Testing chunking cheaply

You do not need the full system to test chunking.

1. Write 30 real questions with the passage that answers each one.
2. Chunk the documents.
3. For each question, check: is the complete answer inside a single chunk, or one parent?
4. Read 20 random chunks out loud. Any chunk that needs the previous one to make sense is a defect.

Formal measurement of retrieval quality is covered in #AIEvalEngine. This quick test catches most chunking problems early.

For a sponsor, the useful fact is that a chunking defect found by this one-day test costs a day, while the same defect found by employees costs trust in the whole assistant.

---

## 6. What would change my mind

- FAQ-style content: each question and answer is already a chunk.
- Very short documents (a one-page notice): index whole.
- Poor parsing: no structure to follow, so fix lesson 1.2 first.

---

## 7. Failure modes

| Failure | Symptom | Guard |
|---|---|---|
| Orphaned conditions | Answers missing "unless" and "only if" | Structure-aware boundaries, parent return |
| Context-free chunks | Pronouns and "the above" with nothing above | Heading path, merge small neighbours |
| Split tables | Numbers without labels | Repeat header rows |
| Oversized chunks | Slow, expensive, diluted answers | Size limit, child chunks |
| Duplicated parents | Same section sent three times | Remove duplicates after retrieval |

---

## 8. Rough numbers

| Item | Estimate |
|---|---|
| Chunk size range | 200 to 800 tokens [CHECK: tune on your questions] |
| Overlap (fixed size only) | 10 to 20 percent [CHECK] |
| Chunks for the HR library | 200,000 to 400,000 [CHECK] |
| Storage for embeddings at 1,536 dimensions, float32 | About 6 KB per chunk, so 1.2 to 2.4 GB raw [CHECK: model dimension] |

---

## 9. Words to know

| Term | Plain meaning |
|---|---|
| Chunk | One searchable piece of a document |
| Chunking | The rules for cutting documents into chunks |
| Overlap | Repeated text across neighbouring chunks |
| Heading path | Breadcrumb of headings added to a chunk |
| Parent and child chunking | Match small chunks, return their whole section |

---

## 10. Exercise

1. Take one long policy you know well. Chunk it three ways: fixed 300 tokens, by heading, and parent and child.
2. Write 10 questions about it. For each method, mark whether the full answer arrives in one chunk.
3. Note which method wins and why.

## Check your understanding

1. Why does overlap not solve the "conditions on the next page" problem?
2. When is parent and child chunking worth its extra storage?
3. What should happen to a table that is too big for one chunk?

---

**Next lesson (1.4):** *Why Does The Assistant Find "Parental Leave" But Miss "HR-PL-204"?* Keyword search, meaning search, and how to combine them.
