# When Your AI Provider Has a Bad Afternoon

*AI System Design: From Real-World Problems to Production-Ready Solutions · Part 3*

> Status: Draft for review · ~640 words · Contoso is a fictional company; the scenario is illustrative. The "From my own project" paragraph is the author's own work: **confirm the wording before publishing.**

---

On a busy Monday afternoon, the company that provides Contoso's AI service starts having problems. It isn't fully down. It's just slow, and it turns some requests away.

Three Contoso teams find out in three different ways:

- **Customer support:** the chat assistant spins for a full minute, then fails. Customers wait, then leave.
- **Finance:** the overnight invoice job keeps retrying, again and again, making the jam worse for everyone.
- **HR:** the policy assistant just freezes.

Later someone asks a simple question: *"How much is each team spending on AI?"* Nobody knows. Each team set up its own connection, its own account and its own rules.

## Why this is harder than it looks

When most systems fail, you switch to a backup copy and carry on. AI is different in two ways:

- **A backup AI isn't a copy.** Switching to a different AI model is like asking a different expert. You'll get an answer, but it may not be the same answer.
- **Everyone shares the same limit.** One team's huge batch job can use up the capacity another team's live chat needs.

## An everyday example: the company travel desk

Large companies don't let every employee book flights directly with airlines. They have a **travel desk**.

The travel desk:

- holds the airline contracts in one place,
- applies company travel rules and budgets,
- knows how much each department spends,
- and when a flight is cancelled, **rebooks you on another airline**, so you keep moving.

For AI, this is called a **gateway**: one central point that every team goes through. It keeps the accounts in one place, sets fair limits for each team, tracks spending, and switches to a backup when the main provider has trouble.

**Where the comparison breaks down:** another airline still lands in the same city. A backup AI might give a different, possibly weaker, answer. So the backup has to be tested *before* the bad day, not during it.

## Different jobs need different rules

The biggest lesson from that Monday: **not every request should fail the same way.**

- **A customer waiting in a chat** can't wait a minute. Try once more quickly, then use the tested backup, or say honestly: *"We're busy. Please try again shortly."*
- **An overnight batch job** has no one waiting. It should pause, wait its turn, and try again later, giving way to customers first.

## From my own project

In a recent capstone project, I built a gateway that switches to a backup AI provider only when the main one is temporarily unavailable. It never switches just because an answer looked wrong. If an answer fails our checks, the system tries to fix it once, then stops safely instead of guessing. The reviewer's most useful feedback: I wasn't recording when the backup was used, so I couldn't prove how often it happened. Now I always record it.

## What this means for the business

- **Fewer outages felt by customers:** a bad afternoon at the provider becomes a small slowdown, not a breakdown.
- **Customers come first:** live conversations keep priority over background work.
- **No surprise bills:** spending per team is visible, and limits prevent runaway costs.

## Key takeaway

Don't let a problem at your AI provider become a problem for your customers. Put one front desk between your teams and your providers, give each kind of job its own rules, and test your backup before you need it.

**Over to you:** the last time a key supplier or service let you down, who noticed first: your team or your customers?

*For the engineers: use a central LLM gateway (for example the AI gateway features in Azure API Management), with per-team token limits, a circuit breaker, retries with jitter that honour Retry-After, and fallback models evaluated in advance. Tag every response with the model that served it.*

---

## Supporting visual (not part of the post)

**Concept:** Left, an airport travel desk rebooking a traveller after a cancelled flight. Right, three team boxes feed one "Gateway". From it, a red dashed line goes to "Main AI (having trouble)" and a green line to "Tested backup".

**Image-generation prompt:**

> Flat editorial illustration, landscape 1200x627, warm off-white background (#F6F1E7), navy line work (#1F2A44), teal (#2A9D8F), amber (#E9A23B) and coral (#D1495B) accents, minimal shading, simple sans-serif labels. LEFT: an airport company travel desk; a departures board behind shows one flight marked "Cancelled" in coral; the desk agent hands a traveller a new ticket labelled "Rebooked". RIGHT: three small boxes labelled "Support", "Finance", "HR" all flowing into one larger box labelled "Gateway"; from it a coral dashed line to a box "Main AI (having trouble)" and a teal solid line to a box "Tested backup". Thin dashed lines link the travel desk to the gateway and the cancelled flight to the main AI. Small tag top-left "AI System Design · Part 3". No robots, no brains, no logos, no real airline or company names.

## References

1. Microsoft Learn, *AI gateway capabilities in Azure API Management*. https://learn.microsoft.com/azure/api-management/genai-gateway-capabilities
2. Azure Architecture Center, *Circuit Breaker pattern*. https://learn.microsoft.com/azure/architecture/patterns/circuit-breaker
