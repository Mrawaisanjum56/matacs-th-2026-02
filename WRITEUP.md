# Write-up: AI proposal triage design

**Principle:** AI reads messy language. Code does anything that must be identical on a re-run and defensible in an audit. A human decides.

## 1. AI vs. rules

**AI (one step):** read each proposal and extract facts into a fixed JSON shape (licence holder, insurance, past installs with their period, price, weeks, warranties, team items), each with an exact quote and page number. Free-form PDFs are the one thing rules cannot read well.

**Plain code or rules (everything else):** file checks, OCR, injection scan, schema and quote checks, mandatory pass/fail, budget and 16-week limits, scoring, sorting, one-line reasons, audit log.

**Why here:** an AI-given score cannot be shown as arithmetic, can drift between runs, and can be talked into a different answer by the proposal text. If the AI only reports facts, the worst a mistaken or manipulated reading can do is a wrong fact, and each fact is shown to a person with its quote. Criteria live in versioned admin configuration, never in prompts or code.

## 2. Consistency and explainability

- **Same proposals ranked twice?** After extraction, yes: scoring is pure code with exact decimals, scores rounded to 2 places before comparing, and a fixed tie-break (experience, timeline, lower price, letter). Extraction at temperature 0 is *not* guaranteed identical, so its output is stored by file hash plus prompt, model and schema version. A re-rank reuses the stored, reviewer-confirmed facts and makes no new AI call.
- **No dependence on other bids:** price is scored against a fixed scale, not against the cheapest bid, so adding or removing a bid never changes another bid's score. I also checked that A > E > C holds across a wide range of price scales.
- **Seeing why:** for each proposal the reviewer sees every criterion with its evidence quote and page, 0-5 score, weight and points, plus a reason line generated from those numbers.

## 3. Human in the loop and audit

Review points: (1) the reviewer picks the category and starts the run; (2) confirms uncertain, conflicting or mandatory evidence before eligibility is decided; (3) resolves flagged proposals by asking the bidder; (4) reviews the ranked list and may reinstate an ineligible proposal or reorder only by entering a reason; (5) the award itself is a human decision outside the tool.

**Logged (append-only):** who and when; file hashes; scan hits; prompt, model, schema and criteria versions; evaluation date; raw extraction; each reviewer correction (old value, new value, reason); score breakdown; status and reasons; overrides; final decision.

## 4. Changing criteria

Edits create a **new version**; the old one is never changed, so past results stay explainable. A run pins the version at start. Weight or band changes need only a re-score from stored facts (no AI call). A new or changed criterion needs re-extraction of that field, then reviewer confirmation. A mandatory-rule change can flip eligibility, so the tool shows a before/after diff and keeps both results labelled by version, never mixing versions in one ranking. Weights must total 100 or the save is rejected.

## 5. Risks and safeguards

- **Wrong extraction or poor OCR:** page numbers and quotes; code rejects any quote not found in the source text; unreadable files go to the reviewer.
- **Made-up facts:** the model may answer "not stated" and is told not to infer. Unsupported answers are rejected.
- **Manipulation (prompt injection):** proposal text is untrusted data in delimiters (closing tags stripped). The AI never scores, so an embedded order cannot change a score. A keyword scan and the model's own flag raise an **integrity flag** for the reviewer, and the attempt is logged rather than silently dropped.
- **Bias or unfair scoring:** one approved rubric applied identically; no company size, location or personal data is scored; the log lets overrides be audited.
- **Price not comparable:** bids may include different items; the reviewer checks scope before award.
- **Over-trusting the tool:** the reviewer sees evidence, not only a rank, and overrides need a reason.
- **Cost and privacy:** one cached call per proposal; size limits; bounded retries; PDFs go only to an approved provider.

## 6. How the design ranks the sample set

| Proposal | Outcome | Why |
|---|---|---|
| A | Rank 1 (75.96) | Best experience, schedule and price among ranked bids |
| E | Rank 2 (63.73) | Full marks on warranty and team, but less experience and a higher price than A |
| C | Rank 3 (52.65, provisional) | Weakest team evidence; also carries an integrity flag (see below) |
| D | Flagged | No schedule, so the 16-week limit cannot be verified; indicative score 63.65 to 72.65 |
| B | Ineligible | The electrical licence belongs to a subcontractor, not the bidder (mandatory) |

- **C's embedded instruction** asking for first place and full marks is ignored for scoring, kept in the log, and shown to the reviewer as an integrity matter to decide.
- **C's experience:** its stated period starts before the 5-year window, so its count of 9 is an upper bound. Even if all 9 counted, its best score (52.65) stays below E, so the rank cannot change (range 34.65 to 52.65).
- **D's range** overlaps E, so its schedule could change the order. It is flagged, not guessed.

**Assumptions:** the brief gives weights but no scoring bands, so the bands (in [PROMPTS.md](PROMPTS.md)) are my proposal for admin approval. The evaluation date is 2026-10-28; "100 kW or more" is inclusive; year-only dates get benefit of the doubt in the boundary year (A and E, "since 2021") and are noted for the reviewer; a mandatory criterion that is not stated fails, while a missing budget or schedule is flagged, not failed.
