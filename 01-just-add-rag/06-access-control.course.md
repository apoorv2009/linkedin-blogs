# Lesson 1.6: How Do You Stop A Salary Band Reaching The Wrong Screen?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 6](06-access-control.md).*

**The brief:** "Everyone can use it, but nobody sees what they shouldn't."

**Level:** L4. Organisation-wide, several data sources, real access rules.

**Time:** About 25 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Compare three places to enforce document access in RAG.
2. Design ACL fields on chunks and the filter that uses them.
3. Plan permission sync and its failure modes.
4. Decide which data should never enter a shared index.

---

## 1. The idea in plain words

A public library with badge-only rooms checks the badge at the door of the room. In RAG, "the door" is the search itself. If forbidden text is never retrieved, the model can never repeat it.

A language model is not a security boundary. Telling it "do not reveal salary data" in the prompt is a request, not a control.

---

## 2. The requirement that drives the decision

> **No chunk may be retrieved for a user unless that user is allowed to read its source document, at the moment of the question.**

"At the moment of the question" is the hard part. It means permission changes must reach the index quickly.

---

## 3. The options

### A. Separate indexes per audience

Analogy: separate buildings for the public library and the archive.

- Simple, auditable, physically separate.
- Breaks down with overlapping audiences: HR Germany, HR global, managers in Germany, managers globally. Each combination tends to need its own copy.

### B. Post-filtering

Analogy: checking the badge at the exit.

- Search returns top 50, then forbidden results are removed.
- If 45 of the 50 were forbidden, only 5 weak results remain. Ranking work was wasted.
- Any code path that forgets the filter (a debug endpoint, a new feature) leaks.

### C. Security trimming during search

Analogy: checking the badge at the door.

- Each chunk stores `access_groups` inherited from its document (lesson 1.2).
- Each query includes a filter: chunk is visible if it shares at least one group with the user.
- Forbidden chunks are never candidates. Ranking works only on allowed content.

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A | Simplicity, audit | Overlapping audiences | Two or three distinct audiences |
| B | Quick retrofit | Leaks, empty results | Only as a second check behind C |
| C | Precision, scale | Needs reliable permission sync | Default for enterprise RAG |

**Choice:** C, with B as an independent second check, and the most sensitive data kept out entirely.

---

## 4. A sketch of the filter

Chunk fields:

```json
{
  "chunk_id": "hr-handbook-de#leave-carryover-1",
  "access_groups": ["all-employees-de", "hr-global"],
  "country": "DE",
  "effective_from": "2026-03-01",
  "effective_to": null
}
```

Query, in pseudo-code:

```
user_groups = directory.groups_for(user_id)          // resolved at question time
filter = access_groups ANY_OF user_groups
     AND country IN [user_country, "GLOBAL"]
     AND effective_from <= today AND (effective_to IS NULL OR effective_to > today)
results = hybrid_search(question, filter, top=50)
```

Most search engines with filtering support this pattern natively. The exact syntax differs.

Rules worth writing down:

- **Default deny.** A chunk with an empty `access_groups` list matches nobody.
- **Resolve groups at question time**, or cache them for minutes, not days.
- **Nested groups** (a group inside a group) must be flattened consistently, either when indexing or when querying.

---

## 5. What never goes in the shared index

Some data is better fetched from its system of record, with that system checking the user:

- Salary bands and individual pay.
- Performance and disciplinary records.
- Draft restructuring or redundancy plans.

The assistant calls the source system on behalf of the user and gets only what that user is entitled to see. How identity is passed safely in that call (On-Behalf-Of delegation) is covered in #DefendingAI. How to bring a small amount of personal context into answers is lesson 1.9.

For a sponsor, keeping these out of the index removes the single largest data-leak risk in the design and usually shortens security and works council review.

---

## 6. Permission sync

| Question to answer | Why it matters |
|---|---|
| Where do document permissions come from? | Source system ACLs, a folder convention, or an HR-owned mapping |
| How often do they sync? | Stale permissions are the most common real-world leak |
| What happens on a role change or leaver? | Access must drop within the agreed window |
| What happens if sync fails? | Fail closed: stop serving changed documents, do not serve with old rules |

---

## 7. What would change my mind

- Two clear audiences that never overlap: Option A.
- The source platform offers permission-trimmed search through an API: query it instead of rebuilding trimming.

---

## 8. Failure modes

| Failure | Symptom | Guard |
|---|---|---|
| Stale group membership | Leaver still sees HR content | Short sync window, resolve groups at query time |
| Empty ACL treated as public | Unlabelled document visible to all | Default deny |
| Unfiltered side paths | Leak through suggestions, logs or caches | Apply the same filter to every path, review logs |
| Service-account search | Everyone effectively searches as admin | Search as the user (see #DefendingAI) |
| Over-broad groups | "All staff" used out of habit | Owner review of group mappings |

---

## 9. Rough numbers

| Item | Estimate |
|---|---|
| Groups per user | Tens to a few hundred [CHECK] |
| Group lookup time | Milliseconds if cached [CHECK] |
| Permission sync target | Within minutes [CHECK: agree with security] |
| Filter overhead | Low with native engine filtering [CHECK: measure with realistic group counts] |

---

## 10. Words to know

| Term | Plain meaning |
|---|---|
| ACL | List of who may read a resource |
| Security trimming | Removing forbidden items during search |
| Pre-filter / post-filter | Filtering before or after the ranking step |
| Default deny | No explicit permission means no access |
| System of record | The authoritative source for a type of data |

---

## 11. Exercise

1. List every audience for your documents and draw which ones overlap.
2. Choose Option A, B or C and justify it in two sentences.
3. Write the three most sensitive document types and decide: index with trimming, or fetch from source?

## Check your understanding

1. Why is a prompt instruction not an access control?
2. What goes wrong with post-filtering when most results are forbidden?
3. What should happen when permission sync fails?

---

**Next lesson (1.7):** *Which Copy Of The Leave Policy Is The True One?* Versions, effective dates and country overrides.
