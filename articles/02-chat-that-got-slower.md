# The Support Chat That Got Slower With Every Message

*AI System Design: From Real-World Problems to Production-Ready Solutions · Part 2*

> Status: Draft for review · ~550 words · Contoso is a fictional company; the scenario is illustrative

---

A customer opens Contoso's support chat about an internet connection that keeps dropping.

At the start, the AI assistant replies in a second. It's helpful.

Forty messages later, every reply takes several seconds.

Then it says: *"Could you try restarting your router?"*

The customer did that twenty minutes ago. Twice. They also said they can't take phone calls at work. The assistant has just offered a phone call.

The customer gives up and asks for a human.

## The surprising reason: the AI remembers nothing

Most people assume a chat assistant remembers the conversation, the way a person does. It doesn't.

Every time you send a message, the system hands the AI **the whole conversation again**, from the very first message. The AI reads all of it, answers, and then forgets everything.

So as the chat gets longer:

- **It gets slower**, because there's more to read every time.
- **It gets more expensive**, because companies pay for every word the AI reads, every single time.
- **It gets less reliable.** Research shows AI models are better at using what's at the start and end of a long text than what's buried in the middle. "I already restarted the router" was buried in the middle.

## An everyday example: the stand-in doctor

Your usual doctor is away, so a stand-in doctor sees you. They have never met you. All they know is what's in your **patient file**.

After years of visits, that file is an inch thick. Reading all of it takes ages, and the note about your allergy is somewhere on page 37.

Good clinics solve this with a **one-page summary sheet** on the front of the file: allergies, current medicines, ongoing conditions. It's short, it's kept up to date, and it's always read first. The full history is still there if it's needed.

A well-designed AI assistant does the same:

- **A short "key facts" sheet** that travels with every message: what the customer has already tried, how they want to be contacted, what was promised.
- **The last few messages** in full, so the conversation flows.
- **The full history stored separately**, and looked up only when needed.

**Where the comparison breaks down:** a doctor decides what to skim. The AI reads everything it is given, and you pay for all of it. So deciding what it sees is up to the people who build the system.

## What this means for the business

Customers don't care about the technical cause. They feel one thing: *this company isn't listening.*

Getting this right means:

- **Customers don't repeat themselves**, so fewer of them give up and ask for a human.
- **Costs stay predictable**, because long chats stop getting more and more expensive.
- **Smoother handovers:** when a person takes over, they get the key facts sheet instead of 58 messages to scroll through.

## Key takeaway

The AI has no memory. Your system decides what it remembers, whether you planned it or not. Decide what must never be forgotten, and make sure it travels with every message.

**Over to you:** have you ever had to repeat yourself to a chatbot? What should it have remembered?

*For the engineers: model calls don't keep any state between requests. Keep a structured "case facts" block, place it near the end of the prompt, keep a short window of recent turns, and test that key facts survive long conversations.*

---

## Supporting visual (not part of the post)

**Concept:** Left, a doctor reading a one-page summary sheet, next to a huge overflowing patient file with a red flag buried in the middle. Right, one AI request drawn as a short stack: "Instructions", "Last few messages", "Key facts" (highlighted), "New question". The full history sits in a separate store, labelled "Look up when needed".

**Image-generation prompt:**

> Flat editorial illustration, landscape 1200x627, warm off-white background (#F6F1E7), navy line work (#1F2A44), teal (#2A9D8F) and amber (#E9A23B) accents with one small coral (#D1495B) flag, minimal shading, simple sans-serif labels. LEFT: a calm doctor at a clinic desk reading a single summary sheet; beside them a very thick overflowing patient file with a small coral flag sticking out of the middle labelled "Page 37". RIGHT: a short vertical stack of four rounded bars labelled "Instructions", "Last few messages", "Key facts" (amber, highlighted) and "New question"; to the side a filing-cabinet icon labelled "Full history" with a dotted arrow labelled "Look up when needed". Thin dashed lines link the summary sheet to "Key facts" and the thick file to "Full history". Small tag top-left "AI System Design · Part 2". No robots, no brains, no logos.

## References

1. Liu et al., *Lost in the Middle: How Language Models Use Long Contexts*, TACL 2024. https://arxiv.org/abs/2307.03172
2. Laban et al., *LLMs Get Lost In Multi-Turn Conversation*, 2025. https://arxiv.org/abs/2505.06120
