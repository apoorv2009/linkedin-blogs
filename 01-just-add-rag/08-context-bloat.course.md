# Lesson 1.8: Why Does Giving The Model More Text Make The Answer Worse?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 8](08-context-bloat.md).*

**The brief:** "Give it everything. The context window is huge."

**Level:** L3.

**Time:** About 20 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Explain why more retrieved text can lower answer quality.
2. Design a context assembly step with a token budget.
3. Estimate input tokens and their effect on cost and latency.
4. Decide when compression is worth an extra step.

---

## 1. The idea in plain words

A librarian who hands you 40 books has not helped more than one who hands you 2. You will skim, and you will miss things. Models do something similar: they read everything you send, pay more attention to some parts than others, and charge you for all of it.

---

## 2. The requirement that drives the decision

> **The model must receive the smallest set of text that fully answers the question, in an order that puts the most useful text where it is most likely to be used.**

---

## 3. What goes into a prompt

| Part | Typical size | Who controls it |
|---|---|---|
| Fixed instructions (system prompt) | Hundreds to thousands of tokens | The team, and it grows quietly |
| Employee facts | Tens of tokens | Lesson 1.9 |
| Retrieved passages | Hundreds to tens of thousands of tokens | This lesson |
| Conversation history | Grows with each turn | Trim or summarise older turns |
| The question | Tens of tokens | The employee |

Context assembly is the step that decides what goes into each part and in what order.

---

## 4. The options

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A. Fixed top N | Simplicity | Unpredictable size, duplicates | Uniform chunk sizes |
| B. Token budget | Predictable cost and latency | Relies on good ranking | Default |
| C. Compression or extraction | Smallest, sharpest input | Extra call, can drop a condition | Long passages, strict limits |

**Choice:** B.

---

## 5. A context assembly recipe

```
passages = rerank(question, candidates)            // lesson 1.5, best first
passages = remove_duplicates(passages)              // same text from copies
passages = group_overrides(passages)                // lesson 1.7, local with global
context  = []
for p in passages:
    if tokens(context) + tokens(p) > PASSAGE_BUDGET: break
    context.append(label(p) + p.text)               // title, country, version, link
prompt = instructions + employee_facts + context + question
```

Notes:

- **Label each passage** with its title, country, version and source link, so the answer can cite it.
- **Order matters.** Best passage first. Some teams also repeat the single best passage near the end; test before adopting.
- **Do not cut a passage in half** to fit the budget. Skip it or stop.

Analogy: packing a small suitcase. Decide the size first, then pack the most important things first, and leave the spare shoes behind.

---

## 6. Why long inputs hurt

1. **Attention spreads.** The 2023 "Lost in the Middle" study (Liu and colleagues) showed weaker use of information placed in the middle of long inputs. Newer models vary [CHECK: test your model with the answer placed at different positions].
2. **Distractors.** Passages that are related but wrong (annual leave when the question is about sick leave) pull answers off course.
3. **Latency.** Longer inputs generally increase the time before the first word appears.
4. **Cost.** Input tokens are billed on every question.

For a sponsor, the context budget is the main dial that sets cost per question, so fixing it in design gives finance a number they can plan around instead of a monthly surprise.

Provider-side prompt caching can reduce the cost and latency of repeated fixed instructions. It is covered in #ModelRouting.

---

## 7. What would change my mind

- Questions that genuinely need whole documents, such as comparisons between two handbooks: route those to a larger budget.
- Measured results show your model is accurate with long inputs at your scale: raise the budget gradually and re-measure.

---

## 8. Failure modes

| Failure | Symptom | Guard |
|---|---|---|
| Lost in the middle | Right passage sent, wrong answer | Budget, best first |
| Distractor passages | Answer drifts to a related topic | Reranking, smaller budget |
| Duplicates | Same text three times, others crowded out | Remove duplicates |
| Prompt creep | Slow rise in cost and latency | Track tokens per question as a metric |
| Long conversations | Old turns crowd out passages | Trim or summarise history |

---

## 9. Rough numbers

| Item | Estimate |
|---|---|
| Fixed instructions | 500 to 1,500 tokens [CHECK: measure yours] |
| Passage budget | 2,000 to 4,000 tokens [CHECK] |
| Naive top 20 | About 10,000 tokens [CHECK] |
| Questions per day | About 1,500 normal, 5 to 10 times at peak [CHECK] |

Monthly input tokens = questions per day x tokens per question x about 30. Multiply by your provider's current price per million input tokens, then repeat for output tokens.

---

## 10. Words to know

| Term | Plain meaning |
|---|---|
| Token | The unit models read and bill by |
| Context window | The maximum input a model accepts |
| Context budget | The amount you choose to use |
| Context assembly | Choosing and ordering what goes into the prompt |
| Distractor | Related but wrong text that misleads the answer |

---

## 11. Exercise

1. Take 10 real questions. For each, build a prompt with 20 passages and with a 3,000-token budget.
2. Compare answers, input tokens and time to first word.
3. Move the correct passage to the start, middle and end of the long prompt. Does the answer change?

## Check your understanding

1. Why is a fixed number of passages unpredictable in cost?
2. Where should the best passage go, and why?
3. Which part of the prompt tends to grow without anyone noticing?

---

**Next lesson (1.9):** *How Does The Assistant Know Your Country Without Knowing Too Much?* Personal context with minimum necessary data.
