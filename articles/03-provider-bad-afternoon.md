# When Your AI Provider Has a Bad Afternoon

*AI System Design: From Real-World Problems to Production-Ready Solutions · Part 3*

> Status: Draft for review · ~1,400 words including tables and code · Scenario is illustrative (Contoso is a fictional company). The ShopSense paragraph is the author's own project. **Confirm the wording before publishing.**

---

At 2:10 pm on a busy Monday, Contoso's AI model provider starts returning errors. It isn't a full outage. Some requests get `429 Too Many Requests`, and the ones that succeed are slow.

Three teams find out in three different ways.

**Customer support:** the chat assistant shows a spinner for 60 seconds before failing. Its code retries with exponential backoff, which is sensible for a background job and painful for a person waiting.

**Accounts payable:** the invoice batch job (from Part 1) retries immediately and repeatedly. Thousands of requests pile onto a provider that is already asking everyone to slow down. Its retries now compete with the support chat for the same quota.

**HR:** the policy assistant has no timeout at all. Requests hang until the web server gives up.

At 4 pm someone asks a simple question: *how much did each team spend on AI this month?* Nobody can answer without logging into three consoles, because each team has its own API key, its own SDK version and its own retry logic.

## Why this is harder than a normal API dependency

Engineers already know how to build around unreliable dependencies. But a model API differs from a typical service in ways that break familiar habits:

- **It's slow even when healthy.** A response that takes seconds is normal, so "slow" and "failing" are harder to tell apart.
- **Rate limits are shared and token-based.** One team's batch job can use up the capacity another team's live chat depends on.
- **A failover target isn't a replica.** If a database fails over, the replica returns the same data. If you fall back to a different model, you get *different answers*: different wording, different accuracy, possibly different adherence to your output schema.
- **Cost depends on the request.** A retry isn't free, and a runaway loop is a budget incident, not just a performance one.

## An analogy: the corporate travel desk

Large companies don't let every employee book flights directly with airlines. They use a **travel desk**.

The travel desk holds the airline contracts. It applies company policy (economy below six hours). It tracks spend by department. And when an airline cancels a flight, it **rebooks** travellers on another carrier, so the traveller keeps moving without having to know anything about the airline's problems.

| At the travel desk | In the AI system |
|---|---|
| Employees and departments | Applications and teams |
| Airline contracts and credentials | Provider API keys, held in one place |
| Travel policy and budgets | Quotas, token limits and model rules per team |
| Rebooking after a cancellation | Fallback to another deployment or model |
| Spend report per department | Token and cost telemetry per team |

**Where the analogy breaks down:** a seat on another airline gets you to the same city. A fallback model may not get you to the same answer. And if the travel desk itself closes, *nobody* travels: a central gateway is a new critical dependency and must be built to be highly available.

## The concept: an LLM gateway

An **LLM gateway** is a single internal entry point for every model call. Applications talk to the gateway. The gateway talks to providers and handles:

- **Authentication:** provider keys live in one place, and applications use their own identities.
- **Quotas and priorities:** token-per-minute limits per team, so batch work can't starve interactive traffic.
- **Timeouts and retries:** with jitter, honouring the provider's `Retry-After` header, and limited per request.
- **Circuit breaking:** stop sending traffic to a backend that is failing, then probe it before restoring traffic.
- **Fallback:** to another region, another deployment, or a pre-approved alternative model.
- **Telemetry:** tokens, latency, errors and cost per team, per model, per request.

## Designing the solution

**v0: every team calls the provider SDK directly.** This is fast to start, and it led to the Monday incident.

**v1: a shared client library.** In .NET, standardise on `IChatClient` from `Microsoft.Extensions.AI` and on `Microsoft.Extensions.Http.Resilience` for timeouts, retries with jitter and circuit breaking. Every team gets the same behaviour. But each process still enforces its own limits in isolation, and there is no shared view of quota or spend.

**v2: a central gateway.** On Azure, **Azure API Management** has AI gateway capabilities built for this: token-limit policies per consumer (`llm-token-limit`), backend pools with priority and weighting across model deployments, circuit-breaker rules that can take the trip duration from the backend's `Retry-After` header, semantic caching, and token-usage metrics. You can also build the gateway as your own ASP.NET Core service. You get more control but take on more to operate.

**The most important design decision:** different callers need different failure behaviour. One policy for everyone is how the Monday incident happened.

| | Interactive (chat) | Background (invoice batch) |
|---|---|---|
| Timeout | Short, sized to the user experience | Long |
| Retries | At most one quick retry, then fall back or fail clearly | Exponential backoff with jitter, honouring `Retry-After` |
| When the primary is down | Pre-approved fallback model, or an honest "try again shortly" | Pause and queue; nobody is waiting |
| Priority | High | Low, and it yields capacity first |

A 60-second retry on an interactive path isn't resilience. It's downtime that takes longer to show.

**Fallbacks have to be earned in advance.** Before a model goes into the fallback chain, run it through the same test set and schema validation as the primary (Part 1's checks still apply). Tag every response with the model that actually served it, so quality dashboards don't silently mix the two.

**From my own build.** In a recent capstone project (ShopSense, an e-commerce support system), I built a provider-agnostic gateway that fails over to another vendor *only* on transient errors (timeouts, 429s, 5xx), never on a validation failure. When a response failed schema validation, the system allowed one repair attempt and then *failed closed*, rather than trying another model and hoping. The evaluator's most useful criticism was that one fallback path wasn't flagged in telemetry, so I couldn't prove how often it ran. That lesson is in the paragraph above.

## Trade-offs

- **A new critical component.** The gateway needs redundancy, its own monitoring and capacity planning. Calls that used to fail separately can now fail together.
- **Extra latency.** Usually small compared with model inference, but measure it.
- **Lowest-common-denominator risk.** A single unified API can hide useful provider-specific features. Allow controlled pass-through.
- **When to skip it.** One application, one team, low volume? A shared client library (v1) is probably enough. Add the central gateway when several teams, budgets or providers are involved.
- **Fallback isn't always right.** For a regulated decision, an honest "unavailable" can be safer than an answer from a model you haven't validated for that task.

## Business impact

The Monday incident cost Contoso three things: customer patience, an invoice backlog, and visibility of spend. A gateway addresses each:

- **Business continuity:** a degraded provider becomes reduced quality on some requests, not an outage on all of them.
- **Fair capacity:** critical customer-facing work keeps its priority during contention.
- **Financial control:** spend per team and per use case is visible daily, and quotas prevent surprises.
- **Easier change:** switching or adding a model becomes a configuration and evaluation exercise, not a rewrite in every team.

Measure: error rate and p95 latency as users see them, the share of requests served by fallback, the rate of 429s per team, and cost per request per team, all captured *before* and after the change.

## Key takeaway

Separate your application's health from your provider's health. Put one gateway between them, give each type of caller its own failure policy, and never fall back to a model you haven't already tested.

**Over to you:** when your AI provider slowed down last time, which of your systems noticed first, your monitoring or your users?

---

## Supporting visual

1. **Image concept:** A corporate travel desk rebooking a traveller after a cancelled flight. Beside it, an LLM gateway routing around a failing provider.
2. **Concept connection:** A single point of policy, credentials and rerouting separates callers from supplier failures. The fallback is a different "carrier", not a clone.
3. **Visual composition:** *Left:* an airport departures board with one flight marked "Cancelled" in coral; a travel-desk agent hands a traveller a new boarding pass for another airline; a small ledger on the desk labelled "Spend by team". *Right:* three app boxes (Support chat, Invoice batch, HR assistant) feeding one gateway box containing small icons: key, gauge, breaker switch. From the gateway, a coral dashed line to "Primary model" with an open breaker symbol, and a teal solid line to "Fallback (tested)". A small queue icon beside Invoice batch labelled "Wait & retry". *Centre:* dashed connectors linking travel desk to Gateway, cancelled flight to Primary model, new boarding pass to Fallback (tested).
4. **Labels:** "Cancelled", "Rebooked", "Spend by team", "LLM gateway", "Circuit open", "Fallback (tested)", "Wait & retry".
5. **Image-generation prompt:**

> Flat editorial illustration, landscape 1200x627, warm off-white background (#F6F1E7), navy line work (#1F2A44), accents teal (#2A9D8F), amber (#E9A23B) and coral (#D1495B), minimal shading, clean sans-serif labels. Split composition. LEFT HALF: an airport corporate travel desk; behind it a departures board with one row marked "Cancelled" in coral; the desk agent hands a traveller a new boarding pass labelled "Rebooked"; a small ledger on the desk labelled "Spend by team". RIGHT HALF: a clean architecture diagram: three small rounded boxes on the left labelled "Support chat", "Invoice batch", "HR assistant" all flowing into one larger box labelled "LLM gateway" containing tiny icons of a key, a gauge and a breaker switch; from the gateway a coral dashed line to a box "Primary model" marked with an open circuit-breaker symbol, and a teal solid line to a box "Fallback (tested)"; a small queue icon next to "Invoice batch" labelled "Wait & retry". CENTRE: thin dashed connector lines linking the travel desk to the gateway and the cancelled flight to the primary model. Small tag top-left "AI System Design · Part 3". No robots, no brains, no futuristic effects, no logos, no real airline or company names.

---

## References

1. Microsoft Learn, *AI gateway capabilities in Azure API Management*. https://learn.microsoft.com/azure/api-management/genai-gateway-capabilities
2. Microsoft Learn, *Build resilient HTTP apps* (`Microsoft.Extensions.Http.Resilience`). https://learn.microsoft.com/dotnet/core/resilience/http-resilience
3. Azure Architecture Center, *Circuit Breaker pattern*. https://learn.microsoft.com/azure/architecture/patterns/circuit-breaker
4. Marc Brooker, *Timeouts, retries, and backoff with jitter*, Amazon Builders' Library. https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/
5. Further reading: Sampriti Mitra, *System Design for the LLM Era* (2026), Ch. 2.1 (LLM gateway, circuit breakers and tiered fallbacks).

**Verify before publishing:** the exact policy names and tier availability in reference 1 (`llm-token-limit` is not available in every API Management tier). Confirm that the ShopSense description matches your repository: failover on transient errors only, one repair turn then fail closed, and the unflagged fallback (evaluation report, M1 and M7).
