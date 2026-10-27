# Lesson 1.7: Which Copy Of The Leave Policy Is The True One?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 7](07-versions-and-country-rules.md).*

**The brief:** "The policy changed last week. Does the bot know?"

**Level:** L4.

**Time:** About 25 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Model document versions with effective dates.
2. Represent global and local rule precedence as data.
3. Handle deletions and drafts safely.
4. Test the boundaries around a policy change.

---

## 1. The idea in plain words

A library removes old editions from the open shelves and keeps local supplements next to national guides. Retrieval needs the same two habits: show only the version that applies on the relevant date, and know which rule wins for which person.

---

## 2. The requirement that drives the decision

> **For any question, the assistant must use the policy version in effect on the relevant date, and must apply local rules over global ones where the local rule says so.**

---

## 3. The options

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A. Replace in place | Simplicity | No history or future, fragile deletes | Single country, no future-dated policies |
| B. Effective dates | Correct for any date | Needs reliable dates | Policies announced ahead |
| C. Dates plus override layer | Correct per country, explainable | Needs precedence owners | Global HR |

**Choice:** C.

---

## 4. Modelling versions

Each document version is its own record. Chunks inherit:

```json
{
  "doc_id": "leave-policy",
  "version": "2026.2",
  "status": "published",
  "effective_from": "2026-03-01",
  "effective_to": "2027-01-01",
  "scope": "DE",
  "overrides_topic": "annual-leave",
  "precedence": "local"
}
```

Retrieval filter for "today":

```
status = "published"
AND effective_from <= as_of_date
AND (effective_to IS NULL OR effective_to > as_of_date)
AND scope IN [user_country, "GLOBAL"]
```

`as_of_date` is normally today in the employee's time zone. It can be a future date if the question is about the future ("What changes in January?"), which the question understanding step must detect.

Analogy: the library's catalogue card says "on loan from 1 March, withdrawn from 1 January". The shelf follows the card.

---

## 5. Representing precedence

Do not ask the model to work out which rule wins. Give it the answer.

1. Retrieve chunks for both `GLOBAL` and the user's country.
2. Group by `overrides_topic`.
3. Where a local chunk overrides a global chunk for the same topic, keep both, and label them in the context:

```
[GLOBAL POLICY: Annual leave] ...
[GERMANY: overrides the global rule for employees based in Germany] ...
```

4. The instruction to the model is then simple: when a local rule is present, it applies to this employee; mention the global rule only if helpful.

Who owns precedence? A named HR owner per country. Without one, the data will drift.

For a sponsor, country HR owners confirming dates and precedence is usually the longest-lead item in this lesson, so it should be requested before engineering starts.

---

## 6. Deletions and drafts

- **Published location only.** Ingest from the place HR publishes to, never from working folders.
- **Tombstones.** When a source document disappears, record its deletion and remove its chunks from every index and cache.
- **Reconciliation.** Regularly compare the list of documents in the index with the list at the source. Anything in the index and not at the source is a zombie.

---

## 7. What would change my mind

- One country, no future-dated changes: Option A with careful deletion.
- HR platform already exposes policies with scope and dates: consume them directly.

---

## 8. Failure modes

| Failure | Symptom | Guard |
|---|---|---|
| Zombie policy | Deleted rule still quoted | Tombstones, reconciliation |
| Early draft | Next year's draft answers today | Published location only, status field |
| Silent override | Global rule wins over local law | Precedence as data, grouped context |
| Wrong date | Answers flip at the wrong moment | Use employee time zone, test boundary days |
| File date used as effective date | Policy "changes" when someone fixes a typo | Dates from HR metadata |

---

## 9. Rough numbers

| Item | Estimate |
|---|---|
| Policy versions per year | Hundreds across all countries [CHECK] |
| Publish-to-assistant target | Within a day [CHECK] |
| Countries needing an override owner | About 30 [CHECK] |

---

## 10. Words to know

| Term | Plain meaning |
|---|---|
| Effective date | When a version starts or stops applying |
| Scope | Which countries or groups a document applies to |
| Precedence | Which rule wins when two apply |
| Tombstone | A record of deletion used to clean up everywhere |
| Reconciliation | Comparing index contents with the source to find drift |

---

## 11. Exercise

1. Pick one policy topic (annual leave, notice period) and list every document that covers it across countries.
2. Mark which are global, which are local, and which local ones override which global ones.
3. Write the test questions for the day before and the day after its next change.

## Check your understanding

1. Why is "last modified" a poor effective date?
2. Why should precedence be data rather than a prompt instruction?
3. How would the assistant answer "What will my leave be next year?"

---

**Next lesson (1.8):** *Why Does Giving The Model More Text Make The Answer Worse?* Context budgets and noise filtering.
