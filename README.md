# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Awais Anjum
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** 2 hour over 2 days

## What this is

A design for an AI-assisted proposal triage tool, from upload to ranked output. The principle: **AI reads messy text, code decides, humans confirm.** Only one step uses AI (fact extraction). Eligibility, scoring, ranking and reasons are deterministic code driven by admin-defined, versioned criteria.

Result for the sample set: **A is first, E second, C third (provisional, with an integrity flag), D is flagged (no schedule), B is ineligible (licence held by a subcontractor).**

## Files

| File | Contents |
|---|---|
| [WRITEUP.md](WRITEUP.md) | Answers to the five questions, how the sample set is ranked, assumptions |
| [diagram/process-flow.png](diagram/process-flow.png) | Process flow showing AI, rule and human steps |
| [PROMPTS.md](PROMPTS.md) | The extraction prompt as sent, plus the rules the code applies |
| [ranking.json](ranking.json) | Expected ranking of A to E, computed with the rules and worked scores in PROMPTS.md |

This is a design exercise, so there is no code or deployed app in this repository.

## How I used AI tools

 I used ChatGPT to brainstorm a first draft, and review it and to help write this final version; I recalculated the scores to check the numbers. **Where I corrected the AI-assisted draft:** (1) it scored price relative to the cheapest eligible bid, so adding or removing one bid changed every other bid's score; I replaced that with a fixed price scale. (2) It scored C using its full stated count of installs even though C's stated period starts before the 5-year window; I changed this so such a count is only an upper bound, shown as a score range, and the proposal is ranked only if the range cannot change its position.


