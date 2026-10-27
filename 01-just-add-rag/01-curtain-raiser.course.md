# Lesson 1.1: Why Does "Just Add RAG" Sound Like 2 Weeks But Take 6 Months?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 1](01-curtain-raiser.md).*

**The brief:** "Just add RAG over our HR docs. Two weeks?"

**Level:** L1 to L5. This lesson looks at the whole journey. Later lessons zoom in on one box at a time.

**Time:** About 25 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Explain RAG to a non-technical colleague in two sentences.
2. Name the five hidden parts that separate a RAG demo from a production system.
3. Compare three starting architectures for the HR assistant and say when each one is the right pick.
4. List the failure modes a design review should ask about before anything is built.

---

## 1. The idea in plain words

Imagine an open-book exam. The student does not memorise the textbook. When a question arrives, they find the right pages, read them, and write an answer in their own words, quoting the page.

That is all RAG is. The language model is the student. Your documents are the textbook. A search step finds the right pages before the model writes anything.

The name comes from the three steps: **retrieve** the relevant text, **augment** the question with that text, then **generate** the answer.

The demo version is a student with one thin textbook. The enterprise version is a student with 5,000 textbooks, some of them out of date, some of them locked, and some of them in German.

---

## 2. The running example

An HR policy and benefits assistant for a 20,000-person global enterprise. It reads:

- Employee handbooks (PDF).
- Country-specific labour law summaries.
- Benefits guides, full of tables.
- Salary bands (confidential spreadsheets).
- Individual employee context: country, grade, start date, from the HR system of record.

The full assumptions, with [CHECK] marks, are in the [series README](README.md#the-running-example).

---

## 3. The requirement that drives the decision

A design review is easier when one requirement is clearly in charge. For this system it is:

> **Every answer must be correct for the person asking (their country, their grade, the current policy), must show where it came from, and must never use a document that person is not allowed to see.**

Notice what is *not* in charge: which model, which vector database, which cloud. Those choices come later and are easier to change.

Three other requirements sit just below it:

| Requirement | Plain-words version |
|---|---|
| Freshness | When a policy changes, the old answer stops within a day [CHECK: confirm the target with HR]. |
| Response time | An answer starts appearing within a few seconds. |
| Peak load | Open enrolment week does not take it down. |

---

## 4. Three ways to start

### Option A: Paste everything into the prompt

Skip search. Put the documents straight into the model's prompt and ask the question.

*Analogy:* answering a question by reading the whole library aloud to someone first.

This works when the "library" is a few documents. Modern models accept very long prompts, so for a single team handbook it is a fair choice.

### Option B: The classic demo

Split documents into pieces, turn each piece into an embedding (a list of numbers that captures meaning), store them in a vector database, and send the closest few pieces to the model with the question.

*Analogy:* a bookshelf at home. Quick to set up, works well while you know every book on it.

This is the three-box whiteboard sketch from the write-up.

### Option C: A layered retrieval pipeline

The same idea as Option B, with the hidden parts added on purpose:

1. **Cataloguing:** parse documents properly, keep tables intact, and label every piece with country, version, effective date and who may read it.
2. **Indexing:** search by meaning *and* by exact words, so "HR-PL-204" is found as easily as "parental leave".
3. **Ranking:** take the top 50 candidates and re-score them with a more careful model, keeping the best few.
4. **Access control:** remove anything the person asking may not read, as part of the search, not after it.
5. **Noise filtering:** send the model a small, focused set of passages within a fixed budget.

*Analogy:* a city library, with a catalogue, a call-number system, a librarian who checks the books before handing them over, and a badge-only room.

A fourth path people raise is training (fine-tuning) a model on the HR documents. It does not solve freshness or access control, and it is covered properly in #ModelRouting, so it is not compared here.

---

## 5. The trade-off table

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A. Paste everything into the prompt | No search to build. Very fast to try. Nothing to keep in sync. | Cannot fit thousands of documents. Pays for every page on every question. Cannot hide one paragraph from one person. Long prompts bury the useful part. | Tens of documents, one audience, low volume, nothing confidential. |
| B. Classic demo (vector search, top few pieces) | Quick to build. Understands meaning, not just keywords. Cheap per question. | Misses exact codes and names. Mangles tables. Keeps old versions. Leaks anything that sits in the index. | An internal pilot for one country with non-confidential documents, to learn what users actually ask. |
| C. Layered retrieval pipeline | Right answer for the right person. Finds codes and meaning. Handles versions and country rules. Can be audited. | More moving parts. Longer to build. Needs someone to own the catalogue and the permissions sync. | Many users, confidential content, more than one country or legal regime. In other words: this brief. |

---

## 6. The choice, and why

**Choice: Option C, built in stages, starting from the skeleton of Option B.**

The reason is not that Option C is fancier. It is about what is expensive to add later.

Ranking and noise filtering can be bolted on at any time. They change the query path, not the stored data.

Access labels, country, version and effective date are different. They live on every stored piece of every document. Adding them later means re-processing and re-indexing the whole library. So the rule for the first sprint is:

> Build the small pipeline, but design the catalogue (the metadata on every piece) for the big one.

A practical build order:

1. Ingestion with metadata and access labels from day one.
2. Hybrid search with access filtering.
3. Ranking and a context budget.
4. Personal context (country, grade) at question time.

For a sponsor, this ordering is the main lever on the timeline: getting the catalogue right in month one is what stops a re-index from costing a month in month five.

---

## 7. What would change my mind

- **The corpus is small.** If HR content is under a few hundred pages and everyone may read all of it, Option A is simpler and good enough.
- **Nothing is confidential and there is one country.** Option B, carefully done, may be all that is needed.
- **The HR platform already offers an assistant** that respects its own permissions and covers the same documents. Then the design question becomes "buy and integrate", not "build".
- **Users mostly ask about themselves**, not about policy. Then the core of the system is a safe query into the HR system, not document search, and lesson 1.9 becomes the main lesson.

---

## 8. Failure modes to raise in the design review

| Failure | What the employee sees | Which hidden part prevents it | Lesson |
|---|---|---|---|
| Wrong country's rule | A confident leave answer that is wrong for them | Cataloguing (country labels), country override layer | 1.2, 1.7 |
| Confidential leak | A salary band they should never see | Access control during search | 1.6 |
| Stale policy | Last year's rule, quoted perfectly | Versions and effective dates | 1.7 |
| Broken tables | Benefit limits that belong to the wrong plan | Layout-aware parsing | 1.2 |
| Missed exact terms | "I could not find that" for a policy code | Hybrid search | 1.4 |
| Right document, wrong passage | An answer that is close but not quite | Ranking | 1.5 |
| Bloated prompt | Slow, expensive answers that miss the key line | Noise filtering | 1.8 |
| Too much personal data in the prompt | Nothing, until an audit | Minimum necessary context | 1.9 |

Two related risks live in other series. Instructions hidden inside a PDF that try to steer the model are covered in #DefendingAI. Measuring how often answers are actually right is covered in #AIEvalEngine.

---

## 9. Rough numbers

These are back-of-the-envelope figures to size the problem. Every input is an assumption, so every output is marked [CHECK].

**Why Option A cannot work at this size**

| Input | Value |
|---|---|
| Documents | 5,000 [CHECK] |
| Pages per document, average | 20 [CHECK] |
| Tokens per page, average | About 500 [CHECK] |
| Total | About 50 million tokens |

The largest context windows on commercial models today are in the range of 1 to 2 million tokens [CHECK]. The whole library is roughly 25 to 50 times bigger than that, before anyone asks a question.

**How big the index is for Option C**

| Input | Value |
|---|---|
| Pieces (chunks) per page | About 3 [CHECK] |
| Total pieces | 5,000 x 20 x 3 = 300,000 |
| Embedding size | 1,536 numbers, a common size [CHECK: depends on the embedding model] |
| Storage per embedding | 1,536 x 4 bytes = about 6 KB |
| Raw vector storage | 300,000 x 6 KB = about 1.8 GB, before indexes and replicas |

This is a small index by vector database standards. Size is not the hard part here. Correctness is.

**Tokens per question**

| Design | Input tokens per question | Notes |
|---|---|---|
| Option B, top 20 pieces of about 500 tokens | About 10,000 [CHECK] | Plus instructions and the question |
| Option C, best 5 pieces after ranking | About 2,500 to 3,000 [CHECK] | Roughly 3 to 4 times fewer input tokens |

To turn tokens into money, multiply by your provider's current price per million input and output tokens, then by daily questions (about 1,500 normal, 5 to 10 times that at peak [CHECK]).

**Latency budget for one answer (Option C)**

| Step | Rough time |
|---|---|
| Hybrid search | Tens of milliseconds [CHECK] |
| Ranking 50 candidates with a cross-encoder | 100 to 300 milliseconds on suitable hardware [CHECK] |
| Access filtering | Part of search when done properly, so close to zero extra |
| Model starts writing | Under 1 to 2 seconds for the first words [CHECK] |
| Full answer | A few seconds, depending on length and model [CHECK] |

---

## 10. Words to know

| Term | Plain meaning |
|---|---|
| **RAG** (retrieval-augmented generation) | Find the right text first, then let the model answer using it. An open-book exam. |
| **Token** | A small piece of text, often part of a word. Models read, write and charge by the token. |
| **Context window** | The maximum amount of text a model can read in one go. |
| **Embedding** | A list of numbers that represents the meaning of a piece of text, so similar meanings sit close together. |
| **Vector database** | A store that finds embeddings close to a question's embedding. Examples include pgvector, Qdrant and Pinecone. |
| **Chunk** | One piece of a document, cut small enough to search and send to the model. |
| **BM25** | A long-established keyword ranking method. Good at exact words and codes. |
| **Hybrid search** | Running keyword search and meaning search together and merging the results. |
| **Reranker / cross-encoder** | A slower, more careful model that re-scores a short list of search results against the question. |
| **ACL** (access control list) | The list of who may read a document. |
| **Security trimming** | Removing results a user may not see as part of the search itself. |
| **Metadata** | Labels on each chunk: country, version, effective date, owner, access. |

---

## 11. Exercise

Pick a real document collection from your own organisation (policies, runbooks, product manuals).

1. Write the one requirement that should drive the design, in the same shape as section 3.
2. Find five "ugly" documents: a scan, a big table, two versions of the same thing, a country or team exception, and something confidential.
3. For each, say which of the five hidden parts would have to handle it.
4. Fill in the trade-off table for your collection. Which option would you pick, and what would change your mind?

## Check your understanding

1. Why is metadata the one thing you should not postpone, even in a first sprint?
2. Option B returns a perfectly written answer from last year's policy. Which hidden part failed?
3. A colleague says, "Context windows are huge now, we don't need search." What numbers would you show them?

---

**Next lesson (1.2):** *What Actually Happens To A PDF Before The Assistant Can Read It?* Cataloguing the library: parsing, tables, scans and the labels every chunk needs.
