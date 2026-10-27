# Lesson 1.2: What Actually Happens To A PDF Before The Assistant Can Read It?

*Module 1: Enterprise RAG Architecture. Course companion to [#JustAddRAG 2](02-pdf-ingestion.md).*

**The brief:** "We'll just upload the PDFs."

**Level:** L3. Real users, real document sets, one team running it.

**Time:** About 20 minutes, plus the exercise.

---

## What you will be able to do after this lesson

1. Describe the stages of an ingestion pipeline in order.
2. Choose between plain extraction, layout-aware parsing and vision models for a given document set.
3. Design the metadata record every chunk inherits.
4. Make re-ingestion cheap by processing only what changed.

---

## 1. The idea in plain words

A library does not shelve donated books straight from the box. It catalogues them first. Ingestion is the catalogue desk of a RAG system: everything the assistant will ever know passes through it.

If a table is flattened here, no model downstream can rebuild it. If a country label is missing here, no prompt downstream can guess it reliably.

---

## 2. The requirement that drives the decision

> **Every piece of text the assistant can retrieve must be faithful to the source (including tables) and must carry the labels needed to filter it: country, version, effective date and access groups.**

---

## 3. The pipeline, stage by stage

| Stage | What happens | HR example |
|---|---|---|
| 1. Discover | Find documents in the source systems and record where each came from | Handbooks in a document library, country summaries in a legal folder |
| 2. Fingerprint | Compute a hash of each file to detect changes and duplicates | Three copies of the same handbook become one |
| 3. Parse | Extract text, headings and tables | The benefits table stays a table |
| 4. Clean | Remove headers, footers, page numbers, boilerplate | "Company Confidential, Page 4 of 62" disappears |
| 5. Label | Attach the metadata record | country = DE, version = 2026.2, effective from 1 Mar 2026 |
| 6. Split | Cut into chunks (lesson 1.3) | |
| 7. Embed and index | Make chunks searchable (lessons 1.4 and 1.6) | |

Stages 1, 2 and 5 are the ones demos skip.

---

## 4. The options for parsing

**A. Plain text extraction.** Reads the PDF's text layer. Analogy: photocopying a page and then reading only the words, ignoring where they sat on the page.

**B. Layout-aware parsing.** Detects headings, reading order, columns and tables, and runs OCR on scanned pages. Output is usually Markdown or JSON with structure intact. Examples: Docling, Unstructured, Azure AI Document Intelligence, Amazon Textract.

**C. Vision model per page.** Renders the page as an image and asks a multimodal model to transcribe it into structured text. Analogy: asking a very capable but occasionally imaginative person to retype every page.

| Option | Good at | Bad at | Pick it when |
|---|---|---|---|
| A | Speed, zero cost, simplicity | Tables, scans, multi-column layouts | Born-digital prose only |
| B | Tables, headings, most scans | Per-page cost, odd layouts need tuning | Mixed enterprise documents |
| C | Hard layouts, forms, handwriting | Cost, speed, risk of invented text | A small fallback set |

**Choice:** B by default, C as a fallback for pages B flags as low confidence, and A never for anything with tables.

---

## 5. The metadata record

Design this before loading the first document. A sketch:

```json
{
  "doc_id": "hr-handbook-de",
  "source_uri": "documents/hr/handbooks/de/handbook.pdf",
  "content_hash": "sha256:...",
  "title": "Employee Handbook, Germany",
  "doc_type": "handbook",
  "country": "DE",
  "language": "de",
  "version": "2026.2",
  "effective_from": "2026-03-01",
  "effective_to": null,
  "owner": "HR Germany",
  "access_groups": ["all-employees-de"],
  "policy_codes": ["HR-PL-204", "HR-PL-310"]
}
```

Every chunk inherits this record. Lesson 1.6 uses `access_groups`. Lesson 1.7 uses `version`, `effective_from` and `effective_to`. Lesson 1.4 uses `policy_codes`.

Where do the values come from? In order of preference: the source system's own fields, then a lookup table maintained by HR, then extraction from the document itself, checked by a person. A label guessed by a model and never checked is a liability.

---

## 6. Making re-ingestion cheap

Analogy: a library does not re-catalogue every book when one new edition arrives.

- Store the fingerprint (`content_hash`) of each source file.
- On each run, process only new or changed files.
- When a file disappears from the source, remove its chunks from the index. Deletion is a feature, not an afterthought.
- Keep the parsed output, so a change in the splitting step does not require paying to parse again.

For a sponsor, the decision that protects the timeline is fixing the metadata record in sprint one, because changing it later means re-processing and re-checking the entire library.

---

## 7. What would change my mind

- The policies already exist as structured intranet pages or HR platform articles: ingest those and skip PDF parsing for them.
- The document set is tiny and stable: a person can clean it once by hand.

---

## 8. Failure modes

| Failure | Symptom | Guard |
|---|---|---|
| Flattened tables | Wrong plan limits in answers | Layout-aware parsing, table spot checks |
| Empty scanned pages | "No information found" for an existing policy | Flag pages with no text layer, route to OCR |
| Duplicates | Same passage returned three times | Fingerprints, one canonical copy |
| Orphaned chunks | Deleted policy still quoted | Deletion handling on every run |
| Wrong labels | Correct text, wrong country | Labels from source systems, reviewed lookups |
| Hidden instructions in a document | Model behaves oddly on certain questions | Covered in #DefendingAI |

---

## 9. Rough numbers

| Item | Estimate |
|---|---|
| Pages in the library | About 100,000 [CHECK] |
| Cloud layout parsing price | Roughly 10 to 15 US dollars per 1,000 pages [CHECK: current price list] |
| Full parse | Roughly 1,000 to 1,500 US dollars [CHECK] |
| Parsing time per page | About 1 to 5 seconds, run in parallel [CHECK] |
| Weekly changed documents | A few dozen [CHECK: ask HR] |

The money is small. The time to check output quality across 30 countries is not.

---

## 10. Words to know

| Term | Plain meaning |
|---|---|
| Ingestion pipeline | The steps from source document to searchable chunks |
| OCR | Turning an image of text into text |
| Layout-aware parsing | Extraction that keeps headings, columns and tables |
| Content hash | A fingerprint of a file, used to spot changes and duplicates |
| Metadata | Labels like country, version and access groups |

---

## 11. Exercise

1. Take 20 real documents from one domain you know. Run them through a plain text extractor and read the output.
2. Count: how many tables broke, how many pages were empty, how many headers repeated?
3. Write the metadata record for your domain. For each field, name where its value will come from.

## Check your understanding

1. Why is parsing cost usually not the expensive part of ingestion?
2. A policy was deleted at the source last month and is still being quoted. Which stage is missing?
3. Why should labels come from source systems rather than from a model reading the document?

---

**Next lesson (1.3):** *How Small Can You Cut A Policy Before It Stops Making Sense?* Chunking strategies and why a table must stay with its heading.
