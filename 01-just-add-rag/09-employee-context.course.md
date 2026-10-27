# Lesson 1.9: How Does The Assistant Know Your Country Without Knowing Too Much?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 9](09-employee-context.md).*

**The brief:** "It should know where I work and what my grade is."

**Level:** L4.

**Time:** About 20 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Separate policy questions from personal-data questions.
2. Design a minimal employee profile and use it as retrieval filters.
3. Decide when a tool call to the system of record is the right pattern.
4. Identify where personal data leaks outside the answer: logs, caches, analytics.

---

## 1. The idea in plain words

A library knows your membership type and home branch. It does not keep your diary on the shelves. The assistant should know a few facts about the person asking, fetched when they ask, and use them to choose the right documents.

---

## 2. The requirement that drives the decision

> **Answers must reflect the asker's own circumstances, using the fewest personal facts possible, fetched from the system of record at question time, and never exposing anyone else's data.**

---

## 3. Two kinds of personal question

| Kind | Example | Where the answer lives |
|---|---|---|
| Policy that depends on who you are | "What is my notice period?" | Policy documents, filtered by your country and contract |
| Your own live data | "How many leave days do I have left?" | The HR system, for you only |

Most questions are the first kind. They need a profile, not a record. The second kind needs a tool call, not search.

---

## 4. The options

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A. Personal records in the index | Quick on the demo | Copies personal data, cross-person matches, hard deletion | Never for HR records |
| B. Profile at question time | Filters, current data, small exposure | Needs HR API | Default |
| C. Tool calls to HR system | Live personal data | More parts, permission design | Own-data questions |

**Choice:** B always, C for a short, reviewed list of own-data tools.

---

## 5. Designing the profile

Start from the questions, not from the HR record. For each common question, ask which facts change the answer.

| Field | Used for | Used as |
|---|---|---|
| Country of employment | Which local rules apply | Filter |
| Work location (region or state) | Regional rules where they exist | Filter |
| Contract type (full-time, part-time, fixed-term) | Leave, notice, benefits eligibility | Filter and prompt fact |
| Grade band | Grade-dependent policies | Filter |
| Start date | Service-based entitlements | Prompt fact |
| Preferred language | Answer language | Prompt instruction |

Not needed for policy answers: name, salary, manager, performance, health, home address.

Flow:

```
profile  = hr_api.get_profile(user)                  // as the user, cached briefly
filters  = { country: profile.country, contract: profile.contract_type, ... }
passages = retrieve(question, filters + access_filter)   // lessons 1.4 to 1.7
prompt   = instructions + facts(profile, only_needed) + passages + question
```

Analogy: the librarian checks your membership card, then walks you to the right shelf. They do not read your card out loud to everyone in the queue.

---

## 6. Tools for own-data questions

For questions about live personal data, give the assistant a small number of narrow tools:

- `get_my_leave_balance()`
- `get_my_benefit_enrolments()`

Rules:

- Tools act **as the employee**, so the HR system enforces its own permissions. The identity delegation mechanics are in #DefendingAI.
- Tools take no "employee ID" argument the model can change. "My" means the signed-in user.
- Read-only first. Anything that changes data (booking leave) is a different risk class, covered in #AgenticAI.

---

## 7. Where personal data leaks besides the answer

| Place | Risk | Guard |
|---|---|---|
| Application logs | Prompts with personal facts stored long-term | Log references, not values; short retention |
| Tracing and debugging tools | Full prompts visible to many engineers | Redact before export (see #DefendingAI) |
| Answer caches | One person's answer served to another | Cache only non-personal answers, or key by user |
| Analytics | Questions linked to identities | Aggregate, pseudonymise |

For a sponsor, keeping personal data down to a few fields is what keeps the privacy and works council review short, and it limits the impact if any single component is breached.

---

## 8. What would change my mind

- The HR platform's own assistant already answers personal questions safely: integrate rather than rebuild.
- Real questions are almost all general policy: reduce the profile to country only.

---

## 9. Failure modes

| Failure | Symptom | Guard |
|---|---|---|
| Over-sharing | Full record in every prompt | Field allow-list |
| Stale profile | Old-country answers after a move | Fetch at question time, short cache |
| Cross-person data | Colleague's record in results | Never index HR records |
| Tool misuse | Model asks for someone else's data | No user ID parameter, act as the user |
| Log leakage | Personal data in long-lived logs | Redaction, retention limits |

---

## 10. Rough numbers

| Item | Estimate |
|---|---|
| Profile fields | 5 to 8 [CHECK] |
| Profile lookup | Tens of ms, cached for minutes [CHECK] |
| Profile tokens in prompt | Under 50 [CHECK] |
| Own-data tools at launch | 2 to 5 [CHECK: from real question samples] |

---

## 11. Words to know

| Term | Plain meaning |
|---|---|
| Minimum necessary context | Only the facts the answer needs |
| Data minimisation | Privacy principle of using only what is needed |
| System of record | The authoritative source for the data |
| Tool call | The model requesting data from a trusted function |
| Pseudonymisation | Replacing identities with codes in stored data |

---

## 12. Exercise

1. Take your 20 most common employee questions. Mark each as "policy depending on who you are" or "own live data".
2. For the first group, list the fields that change the answer.
3. For the second group, write the tool signature without any user ID parameter.

## Check your understanding

1. Why are profile facts used as filters before they are used as prompt text?
2. Why should a tool not accept an employee ID from the model?
3. Name two places personal data can leak that are not the answer itself.

---

**Next lesson (1.10):** *What Does Enterprise RAG Look Like On One Whiteboard?* The full architecture, the build order and the funding questions.
