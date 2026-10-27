# Series 1: #HRAssistantDesign

This series takes one short brief, "Build us an assistant that answers HR questions for 20,000 staff", and designs the system behind it the way you would defend it in a design review. Each write-up picks one hidden part of the architecture (scope, country rules, permissions, freshness, testing, cost) and walks through the options, the trade-offs, and the choice. The running example never changes: an HR policy and benefits assistant for a 20,000-person global company, with PDF handbooks, country-specific laws, salary ranges, and private employee data.

A write-up is marked *Published* only once it is live on LinkedIn.

| # | Title | Topic | Status | LinkedIn |
|---|---|---|---|---|
| 1 | [Why Does An HR Bot Look Finished After Two Weeks?](01-curtain-raiser.md) | Framing The Brief | Draft | not yet |
| 2 | What Is The HR Assistant Actually Allowed To Answer? | Scope And Question Types | Planned | not yet |
| 3 | Should HR Knowledge Live In The Prompt, In A Search Index, Or In The Model? | Where Knowledge Lives | Planned | not yet |
| 4 | How Do You Turn 3,000 Messy PDFs Into Something A Bot Can Search? | Document Ingestion | Planned | not yet |
| 5 | How Does The Bot Give An Employee In Pune And One In Porto Different, Correct Answers? | Country And Role Context | Planned | not yet |
| 6 | Who Is Allowed To See Which Salary Range? | Permission-Aware Search | Planned | not yet |
| 7 | Keyword Search, Meaning Search, Or Both? | Hybrid Retrieval | Planned | not yet |
| 8 | What Should The Bot Say When It Isn't Sure? | Grounding And Human Handoff | Planned | not yet |
| 9 | How Do You Know The Answers Are Right Before 20,000 People Find Out? | Answer Quality Testing | Planned | not yet |
| 10 | What Happens When The Leave Policy Changes On A Tuesday? | Freshness And Versioning | Planned | not yet |
| 11 | What Does It Cost To Answer 20,000 People, And How Fast? | Cost, Speed, And Capacity | Planned | not yet |
| 12 | The HR Assistant Design Review Checklist | Finale | Planned | not yet |

Write-ups are numbered from 1 in titles and posts, for easy reference. There is no fixed schedule.

Core analogy for the whole series: **the hotel front desk.** The friendly desk is the demo. The key cabinet, guest register, house rules per branch, noticeboard, duty manager, and logbook are the back office that this series designs.

Course mapping: Module 1, Enterprise RAG Architecture. Write-up N maps to lesson 1.N.

## After each publish

- [ ] Paste the LinkedIn URL into the table above and change Status to *Published*.
- [ ] Post the series index as the first comment, then pin it.
- [ ] Update the series index comment on every earlier write-up in this series, so the "(next)" link now points to the new post.
- [ ] Update the Status column for Series 1 in [ROADMAP.md](../ROADMAP.md).
- [ ] Render the cover PNG from the HTML in `images/` and check it at phone size.
- [ ] Search the article and post for em or en dashes one last time.
- [ ] After a week, copy useful reader questions into the "Reader questions" section of the matching `.course.md` file.
- [ ] Resolve any `[CHECK]` and `[MY STORY]` markers that are still open before the next write-up goes out.
