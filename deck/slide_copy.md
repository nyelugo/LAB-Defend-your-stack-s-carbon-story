# Defend your stack's carbon story — AI Pre-Scan (Path B)

**Deck:** carbon-story-ai-pre-scan · **Audience:** skeptical CTO · **Path:** B — retrofit audit

## 00 - Cover
- **Title:** Defend your stack's carbon story
- **Subtitle:** A green retrofit audit of AI Pre-Scan — Path B · Ugo Ahukannah
- **Layout:** cover

## 01 - Hook
- **Title:** The cheapest compute is the call you don't repeat
- **Subtitle:** Audience: CTO · Path B retrofit audit of AI Pre-Scan
- **Layout:** bullets-4
- **Body:**
AI Pre-Scan takes a company name and returns an evidence-backed draft inventory of the AI systems that company appears to use.
Its own LESSONS.md already records the finding this audit is built on: the research loop re-runs identical queries, at 3x the spend for the same candidates.
That is a cost line, a latency line and a carbon line at once — which is why this is a CTO conversation, not a sustainability one.
Every number here comes from the repository or the evaluation. Where I am guessing, the slide says so.

## 02 - One unit of value
- **Title:** R = one completed pre-scan of one company
- **Layout:** statement
- **Body:**
One unit of value is one completed scan: a company name in, an evidence-backed inventory and a discussion list out.
Everything that follows is measured per R — calls per R, tokens per R, cost per R, seconds per R.
Boundary: from the operator pressing scan to the report rendering. Excludes the adviser's reading time and the downstream compliance checker.

## 03 - Where the compute actually is
- **Title:** Hotspots: three passes, twelve pages, one large model
- **Subtitle:** Defense of the assessment — measured from the code, not estimated
- **Layout:** bullets-6
- **Body:**
MAX_RESEARCH_PASSES = 3 and MAX_PAGES_PER_SCAN = 12, so one R can reach 36 page fetches.
Every fetched page gets one gpt-4o extraction call — the largest model in the stack, applied uniformly with no triage.
Each pass issues the same 10 web results plus 20 news results; across three passes that is up to 90 result sets for one company.
Validated passages are embedded into Pinecone with text-embedding-3-small — and store.query has no caller, so nothing reads them back.
The vendor corpus namespace has no writer at all: storage allocated, never filled, never read.
Repeated work is the dominant hotspot. It is also the cheapest to fix.

## 04 - The repeated work, precisely
- **Title:** Passes 2 and 3 redo pass 1
- **Subtitle:** The caching candidate, and the reason the eval costs what it does
- **Layout:** bullets-4
- **Body:**
A retry re-issues identical queries rather than refining them, so it cannot find a better source — it can only pay again for the same one.
Findings are replaced per pass rather than accumulated, so a finding evidenced on pass 1 can be lost and re-derived on pass 3.
Nothing is cached between passes: no query cache, no fetched-page cache, no extraction cache keyed by content hash.
Evidence: a full 12-company evaluation run costs roughly $2 and 25 minutes, and five runs were needed because single runs are not reproducible.

## 05 - Honest assumptions
- **Title:** What I am guessing, and what I do not know
- **Subtitle:** "Why should we believe this?" — the parts I cannot evidence yet
- **Layout:** bullets-6
- **Body:**
ASSUMED: OpenAI and Pinecone run in US regions on a grid I cannot see; neither exposes per-call energy, so I cannot compute a real SCI figure.
ASSUMED: extraction dominates energy per R because it is the largest model applied most often — plausible from call counts, not measured.
ASSUMED: page fetches are small relative to inference; unverified, and wrong if a scan hits large PDFs.
KNOWN: call counts, pass limits, page caps and model names — these are read from the code and are not estimates.
KNOWN: the duplicate-query defect and the eval cost, both recorded in the repository before this audit.
UNKNOWN: how often a real scan actually reaches pass 3 in production, because there is no production telemetry.

## 06 - Pillars map
- **Title:** Where each hotspot lands
- **Subtitle:** Carbon, energy, hardware, measurement
- **Layout:** bullets-4
- **Body:**
ENERGY + CARBON: duplicate passes and uniform gpt-4o extraction — avoidable joules per R, and the largest share of it.
HARDWARE: embeddings written but never read, plus an empty vendor namespace — storage and index capacity provisioned for nothing.
MEASUREMENT: no per-R telemetry exists; cost and wall-clock are the only proxies available today.
CARBON: region is unchosen rather than chosen — the grid the work runs on is currently an accident of vendor defaults.

## 07 - Improvement 1: cache across passes
- **Title:** Make a retry cheaper than the first attempt
- **Subtitle:** Pillar: energy, carbon
- **Layout:** bullets-4
- **Body:**
PROBLEM: passes 2 and 3 re-issue identical queries and re-fetch identical URLs, paying full price for a known answer.
CHANGE: cache search results and fetched page text for the life of one scan, keyed by query string and URL hash.
WHY IT IS SAFE: this is in-scan only. It does not reintroduce the cross-scan page retention removed for data-protection reasons — see the tradeoff slide.
Pattern: this is the "Cache static data" idea from the Green Software Patterns catalog, applied within a request rather than at the edge.

## 08 - Improvement 2: triage before the large model
- **Title:** Stop sending every page to gpt-4o
- **Subtitle:** Pillar: energy, carbon
- **Layout:** bullets-4
- **Body:**
PROBLEM: extraction runs gpt-4o on all 12 pages per pass regardless of whether a page plausibly mentions an AI system.
CHANGE: cheap deterministic prefilter, then gpt-4o-mini triage, and escalate to gpt-4o only for pages that survive both.
GUARDRAIL: pair the downgrade with a sample evaluation on the existing 12-company ground truth — no blind model downgrade.
WHY A CTO CARES: this is the single largest cost line per R, and the eval harness to prove it is safe already exists.

## 09 - Improvement 3: stop storing what nothing reads
- **Title:** Delete the write path with no reader
- **Subtitle:** Pillar: hardware, energy
- **Layout:** bullets-4
- **Body:**
PROBLEM: every validated passage is embedded and upserted to Pinecone, and store.query has no caller — the read side does not exist.
PROBLEM: the vendor namespace has a schema and no writer, so index capacity is provisioned for data that never arrives.
CHANGE: either build the reader that justifies the write, or stop writing until the feature that needs it is real.
This is embodied carbon, not just runtime: an index costs hardware whether or not anything queries it.

## 10 - Improvement 4: accumulate, then exit early
- **Title:** Keep what pass 1 already proved
- **Subtitle:** Pillar: energy, measurement
- **Layout:** bullets-4
- **Body:**
PROBLEM: findings are replaced per pass rather than accumulated, so proven work is discarded and re-derived.
CHANGE: accumulate findings across passes and exit as soon as the gate has enough evidence, instead of always spending the pass budget.
SECOND ORDER: this also fixes a correctness defect already recorded in LESSONS.md — a finding lost because a later pass did not re-find it.
The greenest version of this system is the one that stops early, and stopping early is also the more accurate one.

## 11 - Measurement plan
- **Title:** What I would track for two weeks
- **Subtitle:** Pillar: measurement — proxies named honestly as proxies
- **Layout:** bullets-6
- **Body:**
Duplicate query rate per R — identical queries issued more than once in one scan. Target: zero.
gpt-4o calls per R and tokens per R, split by pass, so the triage change can be proved rather than asserted.
Cost per R and seconds per R as energy proxies — they are correlated with energy, they are not energy.
Passes used per R, to learn how often production actually reaches the pass-3 budget.
Pinecone vector count and read count — a read count that stays at zero is the evidence for improvement 3.
SUCCESS: duplicate query rate at zero, gpt-4o calls per R down by half with recall unchanged on the 12-company set.

## 12 - Caveats
- **Title:** What I am not claiming
- **Subtitle:** Honesty before optimisation
- **Layout:** bullets-4
- **Body:**
No carbon-neutral claim, and no offsets. I have no energy data and no grid intensity for the regions involved, so no SCI figure is computed here.
Cost and latency are proxies. A cheaper scan is very probably a lower-energy scan, but I cannot prove the conversion without vendor telemetry.
Offsets would not change any number on these slides; reducing calls per R would. That is the only lever this audit actually pulls.
TRADEOFF: the in-scan cache in improvement 1 must not become cross-scan page retention — that was removed deliberately for data-protection reasons, and carbon does not outrank it.

## 13 - Before and after hypothesis
- **Title:** If we cache and triage, what should move
- **Subtitle:** Stretch — a falsifiable prediction, measured on the existing eval harness
- **Layout:** bullets-4
- **Body:**
HYPOTHESIS: in-scan caching removes the duplicate search and fetch cost of passes 2 and 3, cutting search and fetch work per R by up to two thirds on any scan that retries.
HYPOTHESIS: triage cuts gpt-4o calls per R by roughly half, since most fetched pages never yield a finding.
FALSIFIABLE: recall and over-claim rate on the 12-company ground truth must not move outside their existing run-to-run variance.
If recall drops outside that band, the triage change is wrong and gets reverted — that is the test, decided before the change.
