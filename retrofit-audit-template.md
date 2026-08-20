# Path B — retrofit audit template

The lab's Path B template, completed. The deck presents this; this file is the working note behind it.

**Artifact:** [AI Pre-Scan](https://github.com/nyelugo/PROJECT-AI-Pre-Scan) — my Week 5/6 bootcamp
project. A company name goes in; an evidence-backed draft inventory of that company's AI systems
comes out, each finding tied to a quoted passage, with a discussion list of what public evidence
cannot settle.

**Primary user journey:** an external compliance adviser types a client's company name → the system
searches public sources, fetches pages, extracts candidate systems with a language model, and gates
each finding on quoted support and source currentness → she reads the report, settles the open
questions with the client, and carries the verified facts into a separate deterministic checker.

**One unit of value (R):** one completed pre-scan of one company. Boundary: from the operator
pressing scan to the report rendering; excludes reading time and the downstream checker.

## Hotspots — compute and data movement

| Hotspot | Evidence | Why it concentrates |
|---|---|---|
| Repeated research passes | `MAX_RESEARCH_PASSES = 3`, and `LESSONS.md`: *"the research loop re-runs identical queries rather than refining them — 3× the spend for the same candidates"* | A retry re-issues the same queries, so it cannot find a better source — only pay again for the same one |
| Uniform large-model extraction | `extract.MODEL = gpt-4o`, one call per fetched page, `MAX_PAGES_PER_SCAN = 12` | The largest model in the stack runs on every page regardless of whether the page plausibly mentions AI |
| Search and news fan-out | `web_search(limit=10)`, `news(limit=20)` per pass | Up to 90 result sets per company across three passes |
| Write-only vector storage | `store.upsert` is called; **`store.query` has no caller** | Embedding compute plus index capacity, with a read side that does not exist |
| Empty vendor namespace | `VENDOR_NAMESPACE` has no writer | Index provisioned for data that never arrives — embodied carbon with no runtime use |
| Discarded work between passes | `LESSONS.md`: findings are replaced per pass, not accumulated | Proven findings are dropped and re-derived |

## Green opportunities, prioritised

1. **In-scan cache for queries and fetched pages** (energy, carbon) — quick win. Keyed by query
   string and URL hash, living only for the duration of one scan. Makes a retry cheaper than the
   first attempt instead of identical to it.
2. **Triage before the large model** (energy, carbon) — medium. Deterministic prefilter, then
   `gpt-4o-mini`, escalating to `gpt-4o` only for pages that survive. Paired with a sample
   evaluation on the existing 12-company ground truth, never a blind downgrade.
3. **Stop writing what nothing reads** (hardware, energy) — quick win. Either build the reader that
   justifies the embedding write, or stop writing until the feature that needs it is real. Same for
   the empty vendor namespace.
4. **Accumulate findings and exit early** (energy, measurement) — medium. Keep what a pass proved and
   stop as soon as the gate is satisfied, rather than always spending the pass budget. Also fixes a
   recorded correctness defect.

## What I would measure to prove improvement

Over two weeks of real use: duplicate query rate per R (target zero) · `gpt-4o` calls and tokens per
R, split by pass · cost per R and seconds per R as **proxies**, named as proxies · passes actually
used per R · Pinecone vector count and read count. **Success:** duplicate query rate at zero and
`gpt-4o` calls per R halved, with recall and over-claim rate unchanged on the 12-company set.

## Risks and tradeoffs

- **The cache must not become retention.** Whole-page storage was removed from this project on
  19 August for data-protection reasons. An in-scan cache is compatible with that only if it dies
  with the scan; a cross-scan page cache would re-create the problem, and carbon does not outrank
  privacy here.
- **Triage can cost recall.** Recall is already below target at 0.444, so a change that trades
  accuracy for energy would be the wrong trade in this system. The revert condition is stated in
  advance: recall outside its existing run-to-run variance kills the change.
- **Run-to-run variance exceeds most improvements.** The evaluation moves by more between identical
  runs than between code changes, so a single run cannot demonstrate a saving — any claim needs
  repeated runs, which themselves cost roughly $2 and 25 minutes each.
- **No SCI figure is computed.** Neither OpenAI nor Pinecone exposes per-call energy, and the region
  and grid intensity are unknown. Cost and latency are the honest proxies available.
