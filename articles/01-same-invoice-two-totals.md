# Same Invoice, Two Different Totals: Why AI Output Isn't Data Until You Make It Data

*AI System Design: From Real-World Problems to Production-Ready Solutions · Part 1*

> Status: Draft for review · ~1,400 words including tables and code · Scenario is illustrative (Contoso is a fictional company)

---

The demo went perfectly.

Contoso's accounts payable team dropped twenty supplier invoices into a prototype. For each one, an AI model returned the supplier, the invoice number, the date, the line items and the total. The finance director asked how soon it could go live.

In the second week of the pilot, three things happened.

One invoice came back as a sentence: *"The total appears to be approximately 4,512.40, including tax."* The posting service expected a number and threw an exception.

Then the same invoice, processed twice, produced two different totals. In one run the model took the subtotal; in the other, the total including tax.

The third problem was the one that worried me most. An invoice with no purchase order number came back *with* one. It was correctly formatted, plausible, and invented.

None of this means the model was bad. It means we had wired a probabilistic component straight into a deterministic process.

## Why this happens

A language model doesn't look up an answer. It generates text one token at a time, each token chosen from a probability distribution shaped by everything before it. What comes out is *fluent language*, not *validated data*.

That one fact explains all three failures:

- **The sentence:** prose is a perfectly plausible continuation. Nothing forced a number.
- **The two totals:** when a document is ambiguous (several amounts, unclear labels), small differences in generation can tip the choice.
- **The invented PO number:** we asked for a field the invoice didn't contain. A fluent continuation is more likely than an admission that the field is missing.

## Why the obvious fixes aren't enough

**"Set temperature to zero."** This reduces variation, but it doesn't guarantee identical output. Researchers at Thinking Machines ran the same prompt 1,000 times at temperature 0 on an open-weight Qwen3 model, using standard inference settings, and got 80 distinct completions. The cause was how requests are batched on the server. More importantly, *consistent* isn't *correct*: a model can return the wrong total the same way every time.

**"Tell it to return JSON."** This usually works, and "usually" is the problem. At 10,000 invoices a month, even a rare formatting failure becomes a daily incident.

**"Use a better model."** Better models fail less often, but they still fail. Finance can't run its controls on "less often".

## An analogy: the bank deposit slip

Think about depositing cash at a bank branch.

You could walk up and say: *"I'd like to put in about four and a half thousand, into my savings, I think it's the one ending 72."* A bank doesn't run on sentences. So it gives you a **deposit slip** with fixed boxes: account number, amount in figures, amount in words, date.

Then a **teller** checks the slip. Do the figures match the words? Does the account exist? Does the cash counted match the amount written? Anything unusual goes to a supervisor.

That's the architecture we need:

| At the bank | In the AI system |
|---|---|
| A customer speaking freely | The model generating text |
| The deposit slip | A schema the output must match |
| The teller's checks | Business-rule validators in code |
| Counting the cash | Checking each value against the source document |
| The supervisor | A human review queue |

**Where the analogy breaks down:** a teller can count the cash. A validator can't fully "count" the truth. It can check shape, arithmetic and consistency with your systems, but a well-formed, internally consistent value can still be wrong. That is why the last two layers matter.

## Designing the solution, one layer at a time

**v0: prompt in, text out.** This is what the demo did. It works until it meets real documents.

**v1: constrain the shape.** Most major model APIs now support *structured outputs*: you supply a JSON schema and the response is constrained to match it. This removes the sentence-instead-of-number failure, as long as the provider supports schema enforcement for the model you use. It does nothing for wrong values.

In .NET, `Microsoft.Extensions.AI` exposes this as a typed call:

```csharp
public record InvoiceExtraction(
    string SupplierName,
    string InvoiceNumber,
    DateOnly InvoiceDate,
    string Currency,
    decimal Subtotal,
    decimal TaxAmount,
    decimal Total,
    string? PurchaseOrderNumber,      // null when it is not printed on the invoice
    List<InvoiceLine> Lines);

public record InvoiceLine(string Description, decimal Quantity, decimal UnitPrice, decimal Amount);

ChatResponse<InvoiceExtraction> response =
    await chatClient.GetResponseAsync<InvoiceExtraction>(messages);

if (!response.TryGetResult(out InvoiceExtraction? invoice))
    return Outcome.ToReview("Output did not match the schema");
```

Notice `string?` on the PO number. **If every field is required, you're asking the model to make something up when a value is missing.** Give it a legal way to say "not present". Our invented PO number was a schema design bug, not just a model bug.

**v2: validate the meaning.** Now deterministic code checks what the schema can't:

- line amounts add up to the subtotal (within a rounding tolerance)
- subtotal plus tax equals the total
- the invoice date isn't in the future
- the supplier and PO exist in the ERP, and the PO is open
- the invoice number hasn't already been posted (duplicate check)

These are the controls finance already trusts. Code runs them, not the model.

**v3: evidence and routing.** For high-value fields, ask the model to return where it found each value (a page number and the quoted text), and check that the quoted text actually appears in the document. Then route:

- all checks pass and the amount is below a threshold: post automatically
- any check fails, or the amount is above the threshold: send to a review queue, with the failed checks shown to the reviewer

The reviewer's corrections become labelled examples for your test set. That set is the foundation of Part 18 in this series.

```
Invoice PDF → Model (schema-constrained) → Schema check → Business rules → Evidence check
                                                                               ↓
                                                     Auto-post  ←  pass + low value
                                                     Review queue ← any failure / high value
```

## Trade-offs

- **Schema enforcement isn't universal.** Support varies by provider, model and schema feature. Some keywords may be ignored or rejected. Validate in code even when the API promises compliance.
- **Very strict schemas can hurt quality.** Forcing a value into a box the model is unsure about can make it guess. Nullable fields and an explicit "uncertain" flag often give better results than a rigid schema.
- **Validation adds latency and engineering effort.** For invoices that's cheap compared with a wrong payment. For a low-stakes summary, plain text may be fine.
- **Review queues need people.** If the thresholds are too strict, you've rebuilt manual processing with extra steps. Tune them against measured error rates, not instinct.

## Business impact

The original goal wasn't "use AI". It was faster, cheaper invoice processing *without weakening financial controls*. This design keeps the controls in code, where auditors can inspect them, and uses the model for the hard part: reading messy documents.

Measure it before anyone claims a benefit:

- **Field-level accuracy** against a labelled sample (not just a document-level pass or fail)
- **Straight-through rate:** the share posted with no human touch
- **Exception rate** and the reasons, by supplier
- **Errors caught before posting** compared with errors found after
- **Cost and time per invoice**, including review time

Before the pilot, the expected benefit is only an estimate. After a few weeks of these numbers, it's evidence.

## Key takeaway

A model's answer becomes business data only after your code has checked it. Constrain the shape, validate the meaning, demand evidence, and send the doubtful cases to a human.

**Over to you:** where in your systems does AI output flow into a process that expects exact values, and what checks it on the way?

---

## Supporting visual

1. **Image concept:** A bank teller's counter on the left. The same flow as an AI extraction pipeline on the right.
2. **Concept connection:** Free-form model output is turned into fixed fields (the slip), checked by deterministic rules (the teller), and escalated when it doesn't add up (the supervisor).
3. **Visual composition:** *Left foreground:* a customer handing a filled deposit slip across a counter. The teller holds a calculator and the slip, with a small "supervisor" desk behind. *Right:* an invoice document flows into a box labelled "Model", then a form-shaped "Schema" card, then a row of three check-mark gates, splitting into two arrows: "Auto-post" (teal) and "Review queue" (amber). *Centre:* dashed lines connect slip to Schema, teller to the gates, supervisor to Review queue.
4. **Labels:** "Speech → Slip", "Model → Schema", "Teller checks", "Business rules", "Supervisor", "Review queue", "Auto-post".
5. **Image-generation prompt:**

> Flat editorial illustration, landscape 1200x627, warm off-white paper background (#F6F1E7), navy line work (#1F2A44), accents in teal (#2A9D8F) and amber (#E9A23B), minimal shading, clean modern sans-serif labels. Split composition. LEFT HALF: a bank branch counter; a customer hands a filled paper deposit slip with visible boxed fields to a teller who checks it with a calculator; a small supervisor desk behind the teller. RIGHT HALF: a simple left-to-right flow diagram: a paper invoice icon, a rounded box labelled "Model", a form-shaped card labelled "Schema", three small gate icons with check marks labelled "Business rules", then the flow splits into a teal arrow labelled "Auto-post" and an amber arrow labelled "Review queue". CENTRE: thin dashed connector lines linking deposit slip to Schema, teller to Business rules, supervisor desk to Review queue. Small tag top-left "AI System Design · Part 1". No robots, no glowing brains, no futuristic effects, no logos, no real brand names.

---

## References

1. Microsoft Learn, *Request a response with structured output* (Microsoft.Extensions.AI quickstart). https://learn.microsoft.com/dotnet/ai/quickstarts/structured-output
2. Microsoft Learn, `ChatClientStructuredOutputExtensions.GetResponseAsync` API reference. https://learn.microsoft.com/dotnet/api/microsoft.extensions.ai.chatclientstructuredoutputextensions.getresponseasync
3. Horace He and Thinking Machines Lab, *Defeating Nondeterminism in LLM Inference*, September 2025 (source of the 1,000 runs / 80 completions result). https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/
4. OWASP, *Top 10 for LLM Applications 2025*: LLM05 Improper Output Handling. https://genai.owasp.org/llm-top-10/
5. Further reading: Sampriti Mitra, *System Design for the LLM Era* (2026), Ch. 1 (managing non-determinism) and Ch. 2 (secure output handling).

**Verified 27 Oct 2026:** reference 3 (Qwen3-235B-A22B-Instruct-2507, temperature 0, 1,000 completions, 80 unique). `ChatResponse<T>.TryGetResult` returns false when the output can't be parsed (Microsoft Learn `ChatResponse<T>` reference). **Still to check:** the `GetResponseAsync<T>` overloads for the package version you name, because the JSON-schema switch was renamed between versions (`useNativeJsonSchema` → `useJsonSchemaResponseFormat`).
