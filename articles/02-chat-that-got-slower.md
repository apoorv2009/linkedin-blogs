# The Support Chat That Got Slower With Every Message

*AI System Design: From Real-World Problems to Production-Ready Solutions · Part 2*

> Status: Draft for review · ~1,400 words including tables and code · Scenario is illustrative (Contoso is a fictional company)

---

A Contoso broadband customer opens a support chat about a connection that keeps dropping.

**Turn 5:** the AI assistant replies in about a second. It's helpful and specific.

**Turn 40:** after line tests, a router restart, a firmware check and a long back-and-forth about an engineer visit, each reply takes several seconds.

**Turn 58:** the assistant says, *"Could you try restarting your router?"*

The customer did that at turn 12. Twice. They also said, early on, that they can't take phone calls at work and want updates by email. The assistant has just offered a callback.

Nothing crashed, and no alert fired. The customer simply concluded the AI wasn't listening, and asked for a human.

## The surprising root cause: the model remembers nothing

Most people assume a chat assistant "remembers" the conversation the way a person does. It doesn't.

Every call to a language model is independent. The model keeps no memory between requests. The feeling of a continuous conversation is created by **your application**, which sends the entire transcript again with every new message. Turn 40 resends turns 1 to 39 plus the new question.

Three consequences follow, and they explain the whole incident.

**1. Cost grows much faster than the conversation.** You pay for input tokens on every call, so you pay for the whole history again on every turn.

> *Hypothetical illustration.* Assume each turn adds about 300 tokens (question plus answer). Turn 40 sends about 12,000 tokens. Added up over all 40 turns, the conversation sends about **246,000 input tokens** to deliver 12,000 tokens of actual dialogue: roughly 20 times as much. Growth is close to quadratic.

**2. Latency grows with it.** The model has to process the whole input before it can produce the first word of its reply. A longer prompt means a longer wait.

**3. Quality degrades before you hit any limit.** This is the part most teams miss. Research has measured it in several ways:

- *Lost in the Middle* (Liu et al., TACL 2024) found that models use information at the beginning and end of a long input much better than information in the middle.
- *LLMs Get Lost in Multi-Turn Conversation* (Laban et al., 2025) found an average performance drop of 39% across six tasks when instructions arrived over several turns rather than all at once. Models made early assumptions and then failed to recover from them.
- Chroma's *Context Rot* study (2025) showed performance declining as input length grew, even on simple tasks.

"I already restarted the router" was at turn 12, deep in the middle of a long transcript. That's exactly where such facts get lost.

## Why the obvious fixes aren't enough

**"Use a model with a bigger context window."** A larger window postpones the hard failure. It does nothing for cost per turn, latency, or the facts that get lost in the middle. A bigger room doesn't make a cluttered desk easier to work at.

**"Summarise older turns."** This helps, but summaries lose information, and they lose it silently. A summary such as "Customer has connection drops; basic troubleshooting attempted" has just dropped *which* steps were done and the email-only preference. You won't notice until a customer does.

**"Keep only the last 10 turns."** Then turn 12 is gone completely.

## An analogy: the locum doctor and the patient file

You visit your clinic and your usual doctor is away. A locum (a stand-in doctor) sees you. They have never met you. Everything they know comes from the **file** handed to them as you walk in.

At your first visit the file is thin, so they read it in a minute. After years of visits it's an inch thick. Reading it all takes time you both don't have, and the note about your penicillin allergy is somewhere on page 37.

Good clinics solve this with a **summary sheet stapled to the front**: allergies, current medication, active conditions. It's short, maintained deliberately, and always read. The full history is still there if the doctor needs to look something up.

| At the clinic | In the AI system |
|---|---|
| Locum with no memory of you | A stateless model call |
| The file handed over each visit | The prompt your app assembles |
| Reading a thick file | Paying for and processing every token, every turn |
| The allergy on page 37 | A key fact lost in the middle |
| The summary sheet on the front | A pinned *case-facts* block |
| The archive on request | Older turns retrieved only when needed |

**Where the analogy breaks down:** a doctor uses judgment about which pages to skim. A model processes everything you send, and you pay for all of it. Also, a doctor knows the allergy note matters. The model gives no such guarantee, which is why critical facts must be *placed* deliberately, not just *included*.

## Designing the solution

**v0: resend everything.** Simple and correct for short chats. It degrades on long ones.

**v1: a sliding window of recent turns.** Cost is bounded, but early facts disappear.

**v2: window plus a rolling summary.** Better, but it loses information silently.

**v3: design the memory explicitly.** Each request is assembled from separate parts, each with a budget:

1. **Stable instructions** (identical on every call, so they can be served from the provider's prompt cache).
2. **A case-facts block**: a small, *structured* record of what must never be lost. Some of it comes from your systems (account, plan, open ticket). Some is extracted from the conversation into typed fields: steps already tried, contact preference, promised actions. Your code maintains it, not the model's goodwill.
3. **The last few turns**, word for word, so the conversation flows naturally.
4. **Older details retrieved on demand**: search the stored transcript when the current question needs them.
5. **Trimmed tool outputs**: keep the conclusion of a 5,000-token line test ("line noise above threshold on 2 of 3 tests"), not the raw dump.

In .NET, with `Microsoft.Extensions.AI`, the assembly step looks like this:

```csharp
var messages = new List<ChatMessage>
{
    new(ChatRole.System, SupportInstructions)          // stable prefix, cache-friendly
};

messages.AddRange(transcript.LastTurns(8));            // recent context, verbatim

messages.Add(new ChatMessage(ChatRole.User, $"""
    <case_facts>
    {caseFacts.ToPromptBlock()}
    </case_facts>

    {userMessage}
    """));                                             // critical facts placed at the end
```

The case facts sit **next to the new question**, at the end of the input, which is the position models use most reliably. The full transcript lives in your own store (for example Azure Cosmos DB), not in the prompt.

**Test it like any other feature.** Write "fact survival" tests: a scripted 60-turn conversation in which turn 12 states a constraint, and an assertion at turn 58 that the reply respects it. Run them whenever you change a prompt, a summariser or a model.

## Trade-offs

- **Extraction can be wrong.** If the step that fills case facts misreads the customer, the error persists. Show case facts to the human agent at handover so they can correct it.
- **More moving parts.** A transcript store, an extractor and a retrieval step mean more to build and monitor than "send the array".
- **Prompt caching helps cost and latency, not attention.** A cached prefix is cheaper to resend, but the model still has to work through a long context.
- **Privacy.** Stored transcripts and extracted facts are personal data. Retention rules apply from day one.

## Business impact

The customer at turn 58 didn't experience "context rot". They experienced a company that wasn't listening. Designing memory explicitly protects:

- **Customer experience:** fewer repeated questions, fewer unnecessary escalations.
- **Cost predictability:** spend per conversation stays roughly flat instead of growing with length.
- **Agent productivity:** a human who takes over gets a clean case-facts summary, not a 58-turn transcript.

Measure: escalation rate by conversation length, tokens per turn at the 50th and 95th percentiles, time to first token, and the pass rate of the fact-survival tests. Without these numbers, any improvement is only an expectation.

## Key takeaway

The model has no memory. Your application has a memory design, whether you chose one or not. Decide what must never be forgotten, store it outside the prompt, and place it where the model will use it.

**Over to you:** how does your team decide what a long-running AI conversation is allowed to forget?

---

## Supporting visual

1. **Image concept:** A locum doctor at a desk with a very thick patient file and a clean one-page summary sheet on top. Alongside it, the equivalent prompt assembly.
2. **Concept connection:** A stateless model gets only what the app hands it each turn. A curated, always-present facts block beats an ever-growing transcript.
3. **Visual composition:** *Left:* a doctor in a consultation room holding a single summary sheet; beside them an overflowing file with a small coral flag sticking out of the middle labelled "page 37". *Right:* a vertical stack representing one model request: "Instructions" (navy, with a small cache icon), "Recent turns" (teal), "Case facts" (amber, highlighted), "New question". Off to the side, a cylinder labelled "Full transcript" with a dotted arrow "retrieve on demand". *Centre:* dashed lines linking summary sheet to Case facts, thick file to Full transcript, and the buried flag to a small "lost in the middle" note.
4. **Labels:** "No memory between visits", "Summary sheet", "Lost in the middle", "Case facts", "Recent turns", "Retrieve on demand".
5. **Image-generation prompt:**

> Flat editorial illustration, landscape 1200x627, warm off-white background (#F6F1E7), navy line work (#1F2A44), accents teal (#2A9D8F), amber (#E9A23B) and a small coral (#D1495B) highlight, minimal shading, clean sans-serif labels. Split composition. LEFT HALF: a calm doctor at a clinic desk reading a single clean summary sheet; next to them a very thick overflowing paper patient file with a small coral sticky flag poking out of the middle labelled "Lost in the middle". RIGHT HALF: a vertical stack of four rounded bars representing one request to a model, top to bottom: "Instructions" (navy, small cache icon), "Recent turns" (teal), "Case facts" (amber, slightly glowing outline), "New question"; to the side a database cylinder labelled "Full transcript" with a dotted arrow labelled "Retrieve on demand" pointing into the stack. CENTRE: thin dashed connector lines linking the summary sheet to "Case facts" and the thick file to "Full transcript". Small tag top-left "AI System Design · Part 2". No robots, no brains, no futuristic effects, no logos.

---

## References

1. Nelson F. Liu et al., *Lost in the Middle: How Language Models Use Long Contexts*, TACL 2024. https://arxiv.org/abs/2307.03172
2. Philippe Laban, Hiroaki Hayashi, Yingbo Zhou, Jennifer Neville, *LLMs Get Lost In Multi-Turn Conversation*, 2025 (arXiv 2505.06120). https://arxiv.org/abs/2505.06120
3. Chroma Research, *Context Rot: How Increasing Input Tokens Impacts LLM Performance*, 2025. https://research.trychroma.com/context-rot
4. Microsoft Learn, *Prompt caching* (Azure OpenAI in Microsoft Foundry Models). Search "prompt caching" on learn.microsoft.com for the current page, because the URL has moved with the Foundry rebrand.
5. Further reading: Sampriti Mitra, *System Design for the LLM Era* (2026), Ch. 1 (tokens, context window) and Ch. 1.27 (context engineering).

**Verify before publishing:** the 39% figure and the "six tasks" wording against the arXiv abstract. The current Microsoft Learn URL for prompt caching. The token arithmetic is labelled as hypothetical and is correct for its stated assumption (300 × (1 + 2 + … + 40) = 246,000).
