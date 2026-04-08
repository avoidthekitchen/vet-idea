---
name: idea-validator
description: >
  Pressure-test startup and product ideas against live market signals. Given a product
  or business idea, this skill searches Reddit for pain-point evidence, checks Google
  Trends for demand trajectory, and scans the web for competitive landscape — then
  synthesizes everything into a quick-screen scorecard with a verdict. Use this skill
  whenever the user asks to validate, vet, evaluate, or stress-test an idea, startup
  concept, product hypothesis, or business thesis. Also trigger when the user mentions
  "should I build this," "is this a good idea," "is there demand for," "market
  validation," or presents a product concept and wants an evidence-based assessment.
---

# Idea Validator — Quick Screen

You are an analyst, not an oracle. Your job is to find evidence for and against
a product idea and present it honestly. Do not flatter the user's idea. Do not
dismiss it prematurely. Follow the evidence.

## Pipeline

When the user provides an idea, execute these steps in order:

### Step 1 — Extract Keywords

From the user's idea thesis, extract four categories of terms:

- **Core problem terms:** The specific pain or need (e.g., "invoice reconciliation," "meal prep planning")
- **Audience descriptors:** Who has this problem (e.g., "freelancers," "small business owners," "college students")
- **Adjacent synonyms:** Alternative phrasings people might use (e.g., "expense tracking," "budget meals")
- **Candidate subreddits:** 5-10 subreddits where people with this problem likely congregate

Present the extracted keywords to the user briefly before proceeding, in case
they want to adjust. Then move on.

### Step 2 — Gather Evidence (run all three scripts in parallel)

Run each of the bundled scripts and capture their JSON output. If a script fails
(e.g., API key missing, network blocked), note the failure and continue with
what you have. The scorecard should always note which data sources succeeded and
which failed.

**Script 1: Reddit pain-point search**

```bash
uv run python <skill-path>/scripts/reddit_search.py \
  --keywords "keyword1,keyword2,keyword3" \
  --subreddits "sub1,sub2,sub3"
```

This returns structured JSON with post counts, engagement metrics, and
frustration signals. The `--keywords` flag accepts a comma-separated list of
terms to search for. The `--subreddits` flag specifies which subreddits to
search. Use the audience descriptors and core problem terms as keywords, and
the candidate subreddits from Step 1.

**Script 2: Google Trends check**

```bash
uv run python <skill-path>/scripts/trends_check.py \
  --keywords "keyword1,keyword2,keyword3"
```

This returns interest-over-time data with slope calculations. Use the core
problem terms and adjacent synonyms as keywords.

**Script 3: Web competitor scan**

```bash
uv run python <skill-path>/scripts/web_scan.py \
  --query "descriptive search query about the product space"
```

This returns categorized search results. Construct a natural-language query
that describes the product space (e.g., "AI invoice reconciliation tool for
freelancers").

### Step 3 — Synthesize the Scorecard

Read the scoring rubric at `<skill-path>/references/scoring-rubric.md` for
detailed criteria. Then produce a scorecard using this exact format:

```
IDEA: [user's thesis]
DATE: [current date]

PAIN POINT EVIDENCE:    Strong / Moderate / Weak / None
  → [2-3 sentence summary referencing Reddit thread count, engagement, recency]

TREND DIRECTION:        Accelerating / Flat / Declining
  → [1 sentence summarizing Google Trends trajectory]

COMPETITOR DENSITY:     Open / Moderate / Crowded
  → [1 sentence naming key competitors or noting absence of competition]

QUICK VERDICT:          Investigate Further / Proceed with Caution / Kill
  → [2-3 sentence rationale tied to the evidence above]

COMPETITOR PROFILE:
  | Competitor | Target Audience | Pricing | Maturity | Key Differentiator |
  |------------|----------------|---------|----------|-------------------|
  | [name]     | [who they serve] | [free/freemium/paid] | [emerging/established/mature] | [what makes them distinct] |
  | ...        | ...            | ...     | ...      | ...               |

PERCEPTUAL MAP:
  Axis X: [meaningful dimension 1, e.g., "General audience → Niche specialist"]
  Axis Y: [meaningful dimension 2, e.g., "Basic tool → Full platform"]

  Quadrant map showing where each competitor sits and where the whitespace is.
  Use a text-based 2x2 or markdown table. Label the quadrants and mark
  competitors with dots or initials. Highlight the gap.

  Example:
                    Full Platform
                         |
             CompetitorA  |  CompetitorB
                         |
    General -----------+----------- Niche
                         |
                         |
                   CompetitorC  |
                         |
                    Basic Tool

  [1-2 sentences explaining the whitespace and where the user's idea would sit]

DATA SOURCES:
  → Reddit: [status — X posts found across Y subreddits, or "failed: reason"]
  → Google Trends: [status — 12mo slope, 5yr slope, or "failed: reason"]
  → Web results: [status — X competitors, Y informational, or "failed: reason"]
```

**Scoring principles:**

- Each dimension (pain, trend, competition) is scored independently.
- The verdict follows from the weight of evidence, not a formula. Two strong
  signals and one weak one does not automatically mean "Investigate Further" —
  use judgment.
- If a data source failed, say so explicitly and note how that affects
  confidence in the verdict.
- Do not hedge every statement. If the evidence clearly points one way, say so.
  If it's genuinely ambiguous, say that too.

**Perceptual map guidance:**

The perceptual map is the most insight-dense part of the scorecard. Its job is
to make the competitive whitespace visible at a glance. To build a useful one:

- Choose axes that matter for this specific idea. "Price vs. quality" is too
  generic. Better: axes derived from the idea's value proposition and the
  competitors actually found. Examples: "breadth of coverage vs. depth of
  analysis," "DIY tool vs. managed service," "consumer-focused vs.
  enterprise-focused."
- If fewer than 2 direct competitors were found, the map may not be useful.
  In that case, note "Insufficient competitors to map" and explain why
  (e.g., "the space is too early for meaningful positioning analysis").
- The agent should place competitors based on what can be inferred from their
  titles, descriptions, and URLs. Be transparent about uncertainty.
- Always highlight the whitespace — the empty quadrant or underpopulated region
  where the user's idea would sit.

**Competitor profile guidance:**

For each direct competitor found, extract what you can from the web scan
results. This is a structured summary, not deep research — the agent should
work from titles, descriptions, and URLs, not visit each site. Include:

- **Target audience:** Who they serve (infer from description). "General" if unclear.
- **Pricing:** "Free," "Freemium," "Paid," or "Unknown" — only claim what the
  description explicitly states.
- **Maturity:** "Emerging" (new/unestablished), "Established" (active with
  users), or "Mature" (well-known, full-featured).
- **Key differentiator:** What sets them apart from the rest (one line).

If zero direct competitors were found, replace the table with: "No direct
competitors found — the competitive whitespace is open."

### Step 4 — Offer Follow-Up

After presenting the scorecard, ask the user if they want to:
- Run a **Feature Comparison Matrix** — deeper extraction of competitor features,
  pricing tiers, and capabilities by visiting competitor landing pages. This
  takes longer but produces a detailed side-by-side comparison. Offer this
  especially when COMPETITOR DENSITY is Moderate or Crowded, since the
  matrix helps identify differentiation opportunities.
- Explore a specific dimension in more detail
- Validate a different idea
- Discuss what the results mean for next steps

## Dependencies

The project uses `uv` for dependency management. Run `uv sync` to install.
The `uv run` prefix ensures scripts use the project's virtual environment.

**Web search** (`web_scan.py`) tries three providers in parallel:
- **DuckDuckGo** — always available, no API key needed (fallback)
- **Tavily** — set `TAVILY_API_KEY` for AI-optimized results (free: 1K/month)
- **Brave Search** — set `BRAVE_API_KEY` for high-quality results (free: $5/month credit)

At least DuckDuckGo will work without any configuration. More providers = better
coverage. The script reports per-provider status so the scorecard can note gaps.

**Trends** (`trends_check.py`) uses `trendspy` (installed via uv). No API key needed.

**Reddit search** uses public Reddit JSON endpoints. No API key needed.
