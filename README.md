# LAB | Defend your stack's carbon story

**Path B — green retrofit audit** · **Author:** Ugo Ahukannah · Ironhack AI Consulting Bootcamp, Week 7

**Artifact audited:** [AI Pre-Scan](https://github.com/nyelugo/PROJECT-AI-Pre-Scan) — my own bootcamp
project. A company name in; an evidence-backed draft inventory of that company's AI systems out.

**Audience:** a skeptical CTO. **One unit of value (R):** one completed pre-scan of one company.

## Files

| File | Contents |
|---|---|
| [`carbon-story-ai-pre-scan-path-b-ugo-ahukannah.pptx`](./carbon-story-ai-pre-scan-path-b-ugo-ahukannah.pptx) | **The deliverable.** 13 content slides plus a title slide, with speaker notes on every slide |
| [`retrofit-audit-template.md`](./retrofit-audit-template.md) | The Path B retrofit audit template, completed — hotspots with their code evidence, prioritised opportunities, measurement plan, risks and tradeoffs |
| [`deck/slide_copy.md`](./deck/slide_copy.md) | Slide source, in the layout-archetype format the deck was generated from |
| [`deck/output/deck.pptx`](./deck/output/deck.pptx) | Raw generator output, before speaker notes were attached — kept so the build is reproducible |

## Deck structure

Hook and audience → **R** → hotspots → the repeated-work defect → honest assumptions → pillars map →
**four improvements**, each labelled with its GSF pillar → measurement plan → caveats → before/after
hypothesis.

## Why the numbers are defensible

Every hotspot is read from the repository rather than estimated: `MAX_RESEARCH_PASSES = 3`,
`MAX_PAGES_PER_SCAN = 12`, `extract.MODEL = gpt-4o` on every fetched page, `web_search(limit=10)` and
`news(limit=20)` per pass, and `store.query` with no caller. The headline defect — *"the research
loop re-runs identical queries rather than refining them — 3× the spend for the same candidates"* —
was recorded in the project's own `LESSONS.md` **before** this audit, so it is not a finding written
to fit the assignment.

## What this deck does not claim

No carbon-neutral claim, no offsets, and no SCI figure — neither OpenAI nor Pinecone exposes
per-call energy, and the deployment region is unknown. Cost and latency are used as proxies and are
labelled as proxies. The deck also names the tension it cannot dissolve: the proposed in-scan cache
must not become the cross-scan page retention that was deliberately removed from the project for
data-protection reasons.
