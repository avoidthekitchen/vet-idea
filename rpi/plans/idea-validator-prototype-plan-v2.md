# Idea Validator — Prototype Plan v2

## Vision

A tool that pressure-tests startup and product ideas against live market signals. Not idea generation — idea interrogation. You bring a thesis, the tool tells you whether the evidence supports it.

The tool operates in two analysis tiers: a free quick screen that kills bad ideas fast, and a paid deep dive that builds a full evidence package for ideas worth pursuing.

**v2 change:** Development starts as a Claude Agent Skill — the fastest path to testing whether the analysis pipeline produces useful output. The skill is a low-cost proving ground. If the analysis works, it graduates to a standalone Python tool with persistence, a UI, and eventually idea generation.

---

## Core Design Principles

- **Validation-first, generation-later.** Start by testing ideas you already have. Idea generation becomes a natural later phase once you've calibrated which signals actually predict good opportunities.
- **Two-tier analysis funnel.** Most ideas die on first contact with evidence. Don't spend money on the ones that do.
- **Skill first, standalone second.** Prove the analysis logic works before investing in infrastructure. The conversation _is_ the UI until the analysis earns a real one.
- **Transparent assumptions.** Every score and estimate shows its reasoning. The tool is an analyst, not an oracle.

---

## Development Progression

```
Phase 0 — Agent Skill (prove the analysis works)
    │  Claude reads the skill, runs bundled Python scripts,
    │  returns a scorecard. Your interface is the chat.
    │  No UI, no database, no deployment.
    │
    │  [Validate on 15–20 ideas. Is the output useful?]
    │
    ▼
Phase A — Standalone Pipeline (graduate what works)
    │  Port proven scripts and prompts into a standalone
    │  Python CLI. Add SQLite persistence.
    │
    ▼
Phase B — Deep Dive Tier (add paid data sources)
    │  Keywords Everywhere, competitor profiling,
    │  Fermi market sizing.
    │
    ▼
Phase C — UI + Calibration (Streamlit, feedback loop)
    │
    ▼
Phase D — Refinement + Idea Generation Pivot
```

---

## Phase 0 — Agent Skill (Week 1)

### Purpose

Test the core analysis pipeline with zero infrastructure. Prove that the prompts, data sources, and scoring logic produce output you'd actually trust before writing a single line of application code.

### What the Skill Does

You say: _"Validate this idea: AI-powered invoice reconciliation for freelancers."_

Claude reads the skill's SKILL.md, runs the bundled scripts, and returns a scorecard — all within the conversation.

### Skill Structure

```
idea-validator/
├── SKILL.md                  # Instructions + scoring rubric
├── scripts/
│   ├── reddit_search.py      # Search Reddit for pain point signals
│   ├── trends_check.py       # Pull Google Trends via pytrends
│   └── web_scan.py           # Competitor surface scan via Brave Search
└── references/
    └── scoring-rubric.md     # Detailed scoring criteria + examples
```

### SKILL.md Responsibilities

The SKILL.md instructs Claude to:

1. **Extract keywords** from the user's idea thesis — core problem terms, audience descriptors, adjacent synonyms, and candidate subreddits. This is Claude's native capability; no script needed.

2. **Run `reddit_search.py`** with the extracted keywords. The script queries relevant subreddits, filters for posts with 20+ upvotes and frustration-indicating language ("wish there was," "looking for a tool," "does anyone know"), and returns structured JSON: post count, recency distribution, average upvotes, top thread URLs.

3. **Run `trends_check.py`** with the core keywords. The script pulls 12-month and 5-year interest-over-time data via pytrends, computes a simple linear slope, and returns: current interest level, 12-month trend direction, 5-year trend direction.

4. **Run `web_scan.py`** with the idea's value proposition as a search query. The script hits the Brave Search API, collects the top 10 results, and returns: page titles, URLs, snippets, and a rough categorization (direct competitor / adjacent tool / informational content / dead project).

5. **Synthesize all script outputs** into a scorecard using the scoring rubric from `references/scoring-rubric.md`. Claude does this natively — no script needed.

### Scoring Rubric (Summary)

The full rubric lives in `references/scoring-rubric.md` to keep SKILL.md concise. Key dimensions:

| Dimension | Strong | Moderate | Weak |
|---|---|---|---|
| Pain point evidence | 10+ recent Reddit posts with high engagement describing this frustration | 3–9 posts, or posts are older / low engagement | 0–2 posts, or only tangentially related |
| Trend direction | 12-month slope clearly positive; 5-year trend supports growth | Flat or ambiguous trajectory | Declining interest on both timeframes |
| Competitor density | Few or no direct competitors; adjacent tools exist (validates demand) | 3–5 direct competitors but clear gaps remain | 6+ well-funded direct competitors with full feature coverage |

### Output: Quick Screen Scorecard

```
IDEA: [user's thesis]
DATE: [timestamp]

PAIN POINT EVIDENCE:    Strong / Moderate / Weak / None
  → [2-3 sentence summary with Reddit thread count and recency]

TREND DIRECTION:        Accelerating / Flat / Declining
  → [1 sentence on Google Trends trajectory]

COMPETITOR DENSITY:     Open / Moderate / Crowded
  → [1 sentence listing key competitors found]

QUICK VERDICT:          Investigate Further / Proceed with Caution / Kill
  → [2-3 sentence rationale]

DATA SOURCES:
  → Reddit: [X posts found across Y subreddits]
  → Google Trends: [12mo slope, 5yr slope]
  → Web results: [X competitors, Y informational]
```

### What Phase 0 Tests

This phase answers three questions before you invest further:

1. **Do the data sources return useful signals?** Reddit may not discuss every problem domain. Google Trends may be too noisy for niche ideas. Brave Search free tier may not surface the right competitors. You find out fast.

2. **Does the LLM synthesis add value beyond raw data?** If the scorecard just restates what the scripts returned without useful judgment, the prompts need work. If it's making overconfident claims, the rubric needs tightening.

3. **Is the two-tier funnel worth building?** If Phase 0 scorecards consistently give you a clear go/no-go signal, the full deep dive tier may be less critical than expected. If they're always ambiguous, the deep dive tier is essential.

### Phase 0 Tasks

- [ ] Write SKILL.md with keyword extraction instructions and synthesis prompt
- [ ] Write `reddit_search.py` — accepts keywords + subreddits, returns structured JSON
- [ ] Write `trends_check.py` — accepts keywords, returns slope + interest data
- [ ] Write `web_scan.py` — accepts query string, returns categorized search results
- [ ] Write `references/scoring-rubric.md` with detailed criteria and example scorecards
- [ ] Test on 5 ideas manually; tune prompts and rubric based on output quality
- [ ] Test on 10 more ideas; assess whether verdicts are consistent and useful
- [ ] Document which ideas the tool got right vs. wrong (informal log — this becomes calibration data later)

### Phase 0 Success Criteria

- Scorecards for at least 15 ideas completed
- At least 60% of verdicts feel directionally correct on manual review
- At least 2–3 ideas confidently killed that you might otherwise have wasted time exploring
- Clear notes on which data sources were most/least useful
- A decision: graduate to standalone tool, iterate further on the skill, or abandon

### Phase 0 Constraints & Limitations

- **No persistence.** Each conversation starts fresh. You lose history between sessions unless you manually save scorecards. Acceptable for 15–20 test runs; becomes painful at scale.
- **No deep dive tier.** Quick screening only. Keyword volume, competitor deep profiling, and market sizing wait for Phase B.
- **Network dependencies.** Scripts need outbound access to Reddit's API, Google Trends, and Brave Search. If any are blocked in the runtime environment, that module fails gracefully and the scorecard notes the missing data.
- **Session-bound.** Runs inside a single Claude conversation. No cron jobs, no scheduled re-runs, no monitoring.

### Cost

$0. All data sources are free tier. Claude handles synthesis natively.

---

## Phase A — Standalone Pipeline (Weekend 2)

### Purpose

Graduate the proven skill into a standalone Python tool. Add persistence so you can track ideas over time and start calibrating scoring accuracy.

### Prerequisite

Phase 0 completed. You have 15+ scorecards, notes on what worked, and refined prompts/scripts.

### What Changes From Phase 0

| Concern | Phase 0 (Skill) | Phase A (Standalone) |
|---|---|---|
| Interface | Claude conversation | Python CLI |
| LLM | Claude (native) | Kimi K2.5 via API (or any cheap cloud LLM) |
| Persistence | None | SQLite |
| Data scripts | Bundled in skill folder | Same scripts, imported as Python modules |
| Prompts | In SKILL.md | In prompt template files, version-controlled |
| Scheduling | Manual | Manual (cron-ready) |

### Tasks

- [ ] Create project structure; copy and refactor scripts from skill folder
- [ ] Replace Claude-native synthesis with LLM API calls (Kimi K2.5 or similar)
- [ ] Externalize prompts into template files so they're easy to version and tune
- [ ] Set up SQLite schema (see Data Persistence section below)
- [ ] Wire the CLI: `python validate.py "idea thesis"` → scorecard printed + stored
- [ ] Backfill any Phase 0 scorecards you saved into the database
- [ ] Test on 5 new ideas to confirm standalone pipeline matches skill quality

### Deliverable

`python validate.py "AI invoice reconciliation for freelancers"` prints a scorecard and stores all raw data + results in SQLite.

---

## Phase B — Deep Dive Tier (Weekend 3)

### Purpose

Add paid data sources and the full analysis report for ideas that survive the quick screen.

### Additional Data Sources

| Source | What It Provides | Cost |
|---|---|---|
| Keywords Everywhere API | Actual monthly search volume, CPC, competition score | ~$0.01 per keyword ($1 / 100 credits) |
| Brave Search API (additional calls) | Deep competitor profiling | Within free tier if used sparingly |
| Product Hunt (scraping) | Recent launches in the space, traction signals | Free |
| Capterra / G2 (scraping) | Competitor pricing, feature lists, review sentiment | Free |

### Deep Dive Pipeline Steps

1. **Search volume validation.** Run 15–25 keywords through Keywords Everywhere. Get actual monthly search volume, CPC (proxies commercial intent), and competition score.

2. **Competitor deep profile.** For each competitor from the quick screen, scrape landing page and pricing page. Extract: pricing tiers, core features, positioning language, approximate maturity.

3. **Product Hunt / community traction scan.** Search for recent launches in the same space. Check upvote counts, comment sentiment, traction signals.

4. **Fermi market sizing.** Dedicated LLM call that produces a back-of-envelope market size estimate. The prompt forces the LLM to:
   - State every assumption explicitly
   - Show the math step by step
   - Assign a confidence level (low / medium / high)
   - Flag which assumptions are weakest

5. **Full synthesis.** Long-context LLM call ingesting all quick screen and deep dive data to produce the full report.

### Output: Deep Dive Report

```
IDEA VALIDATION REPORT
======================
Idea:    [thesis]
Date:    [timestamp]
Quick Screen: [scorecard summary]

1. PAIN POINT ANALYSIS
   - Evidence summary
   - Representative quotes/threads (with URLs)
   - Community size and activity metrics

2. TREND TRAJECTORY
   - Google Trends data (12mo + 5yr)
   - Acceleration/deceleration assessment
   - Seasonal patterns if detected

3. SEARCH VOLUME & COMMERCIAL INTENT
   - Top keywords with monthly volume and CPC
   - Volume trajectory (growing/shrinking keyword set)
   - Commercial intent assessment based on CPC distribution

4. COMPETITIVE LANDSCAPE
   - Competitor matrix: name, pricing, core features, maturity
   - Identified gaps (features/audiences no one serves well)
   - Dead competitors (and why they may have failed)
   - Saturation assessment: growing niche vs. mature market

5. MARKET SIZE ESTIMATE (Fermi)
   - Stated assumptions
   - TAM/SAM/SOM estimates with math shown
   - Confidence level and weakest assumptions flagged

6. OVERALL ASSESSMENT
   - Demand strength:         [1-5 with justification]
   - Timing:                  [1-5 with justification]
   - Competition intensity:   [1-5 with justification]
   - Distribution difficulty: [1-5 with justification]
   - Verdict:                 [Build / Explore further / Pass]
   - Key risks and open questions
```

### Cost Per Deep Dive

- **API cost:** ~$0.15–0.30 (mostly Keywords Everywhere)
- **LLM cost:** ~$0.05–0.15 (one long-context call)
- **Total:** ~$0.20–0.45 per idea
- **Expected usage:** 3–4 ideas/week
- **Estimated monthly cost:** $5–10

### Tasks

- [ ] Integrate Keywords Everywhere API
- [ ] Build competitor deep profiling module (scrape landing + pricing pages)
- [ ] Build Product Hunt search module
- [ ] Build Fermi market sizing prompt (with explicit assumption forcing)
- [ ] Build full synthesis prompt
- [ ] Extend SQLite schema for deep dive results
- [ ] Wire CLI: `python validate.py "idea thesis" --deep` produces full report
- [ ] Test on 5 ideas that passed the quick screen

---

## Phase C — UI + Calibration (Weekend 4)

### Purpose

Streamlit interface. Begin systematically calibrating scoring accuracy.

### Tasks

- [ ] Streamlit app with text input, quick screen button, deep dive button
- [ ] Display scorecard and full report in-app
- [ ] History view — browse past ideas and their results
- [ ] Add user feedback field: after manual review, rate the tool's accuracy
- [ ] Begin tracking quick screen → deep dive correlation for heuristic tuning

### Deliverable

A local web app at `localhost:8501` you'd actually use daily.

---

## Phase D — Refinement + Idea Generation Pivot (Weeks 5–7)

### Purpose

Improve signal quality. Optionally begin building toward inbound idea detection.

### Tasks

- [ ] Tune prompts based on accuracy feedback from first 30+ ideas
- [ ] Add Hacker News as a secondary pain point source
- [ ] Improve competitor analysis with G2/Capterra review sentiment
- [ ] Explore flipping the pipeline: scan Reddit/HN daily for pain points that score well on quick screen signals automatically
- [ ] If the generation pivot works, build a daily digest of top 3 auto-detected opportunities

---

## Data Persistence (Phase A onward)

### SQLite Schema

```sql
CREATE TABLE ideas (
    id INTEGER PRIMARY KEY,
    thesis TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE quick_screen_results (
    id INTEGER PRIMARY KEY,
    idea_id INTEGER REFERENCES ideas(id),
    pain_score TEXT,          -- strong/moderate/weak/none
    trend_direction TEXT,     -- accelerating/flat/declining
    competitor_density TEXT,  -- open/moderate/crowded
    verdict TEXT,             -- investigate/caution/kill
    raw_data JSON,           -- all fetched data for replay/review
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE deep_dive_results (
    id INTEGER PRIMARY KEY,
    idea_id INTEGER REFERENCES ideas(id),
    keywords JSON,            -- volume, CPC, competition per keyword
    competitors JSON,         -- structured competitor matrix
    market_size_estimate JSON,-- Fermi estimates with assumptions
    overall_scores JSON,      -- demand, timing, competition, distribution
    verdict TEXT,             -- build/explore/pass
    full_report TEXT,         -- rendered markdown report
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_feedback (
    id INTEGER PRIMARY KEY,
    idea_id INTEGER REFERENCES ideas(id),
    phase TEXT,               -- quick_screen / deep_dive
    accuracy_rating INTEGER,  -- 1-5: how accurate was the verdict?
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Tech Stack

| Component | Phase 0 (Skill) | Phase A+ (Standalone) |
|---|---|---|
| LLM | Claude (native in skill) | Kimi K2.5 via API (or any cheap cloud LLM) |
| Reddit data | `reddit_search.py` script | Same script, imported as module |
| Trend data | `trends_check.py` (pytrends) | Same script, imported as module |
| Web search | `web_scan.py` (Brave Search API) | Same script, imported as module |
| Keyword volume | — | Keywords Everywhere API (Phase B) |
| Database | None | SQLite |
| Interface | Claude conversation | CLI → Streamlit |
| Hosting | Local | Local |

---

## Timeline Summary

| Phase | Effort | Cumulative Ideas Tested | Key Decision |
|---|---|---|---|
| Phase 0 — Agent Skill | ~1 week (evenings) | 15–20 | Is the analysis useful? Go/no-go on standalone build. |
| Phase A — Standalone CLI | 1 weekend | 25–30 | Are the prompts good enough to add paid data? |
| Phase B — Deep Dive | 1 weekend | 35–40 | Does deep dive materially improve on quick screen? |
| Phase C — UI + Calibration | 1 weekend | 50+ | Is scoring accuracy improving with feedback? |
| Phase D — Generation Pivot | 2–3 weeks | 70+ | Can the tool find ideas, not just vet them? |

---

## Known Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| Phase 0 has no persistence | Can't compare ideas across sessions | Manually save scorecards; accept this until Phase A |
| No absolute search volume until Phase B | Can't distinguish 500 vs. 50K searches/month | Google Trends slope + Reddit volume as directional proxy |
| Market sizing is LLM estimation, not data | Fermi estimates can be wildly off | Force visible assumptions; flag confidence; treat as hypothesis |
| Reddit bias toward technical audiences | Misses problems in non-technical markets | Add HN, niche forums, Quora in Phase D |
| Competitor scraping is brittle | Sites change structure; some block scrapers | Accept partial data; prioritize Product Hunt and Brave results |
| LLM synthesis quality varies | Scoring consistency depends on prompt quality | Store raw data so you can re-run with improved prompts |
| Phase 0 network access is environment-dependent | Some APIs may be blocked in Claude's sandbox | Scripts fail gracefully; scorecard notes missing data |

---

## Future Extensions (Post-Prototype)

- **Idea generation mode:** Flip the pipeline to scan Reddit/HN/Trends daily and surface pain points that match strong scoring signals automatically.
- **Comparison mode:** Validate 5 ideas in batch and rank them against each other.
- **Trend monitoring:** For ideas you're actively building, run weekly re-checks to track whether signals are strengthening or fading.
- **Export to pitch format:** Generate a one-page brief from a deep dive report suitable for sharing with co-founders or advisors.
- **Skill as ongoing companion:** Even after building the standalone tool, keep the Agent Skill maintained as a lightweight way to do quick checks inside any Claude conversation without switching tools.

---

*"Don't fall in love with the idea. Fall in love with the evidence."*
