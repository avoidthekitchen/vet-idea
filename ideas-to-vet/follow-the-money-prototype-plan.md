# Follow the Money — Prototype Plan

## Vision

A civic tech tool that makes corporate influence on U.S. politics viscerally understandable to a general audience — not just researchers and journalists. Built on public government data, powered by agentic AI synthesis, distributed via automated social content and a newsletter.

---

## Core Thesis

The data already exists. The gap is the **synthesis and storytelling layer** — turning dry FEC filings into striking, shareable facts that make people feel the scale of the problem.

---

## Phase 1 — Data Foundation (Weeks 1–3)

**Goal:** Get the raw data flowing and queryable locally.

### Data Sources
- **FEC Bulk Data** — `fec.gov/data/browse-data` — candidate filings, PAC contributions, expenditures (updated nightly)
- **OpenSecrets API** — industry-level aggregations, lobbying data, revolving door records
- **Senate/House Lobbying Disclosures** — `lda.senate.gov` — which firms lobbied which agencies on what issues
- **Congress.gov API** — bill sponsorships, voting records (needed to correlate money → votes)

### Tasks
- [ ] Set up local Postgres or DuckDB instance for FEC bulk data ingestion
- [ ] Write ingestion scripts for FEC contribution files (they're pipe-delimited CSVs)
- [ ] Pull OpenSecrets API for industry aggregations
- [ ] Map Congress member IDs across datasets (FEC uses different IDs than Congress.gov)
- [ ] Build basic correlation query: contributions to member X → member X's votes on bills tagged with industry Y

### Key Technical Challenge
FEC data is messy — donor names are inconsistent, employer fields are user-entered. You'll need fuzzy matching or OpenSecrets' pre-cleaned industry classifications as a shortcut.

---

## Phase 2 — Agentic Analysis Layer (Weeks 3–6)

**Goal:** Natural language queries that surface non-obvious correlations automatically.

### Core Queries to Support
- "Which senators received the most from pharmaceutical companies before voting on drug pricing bills?"
- "Show me all contributions to [member] from energy industry in the 6 months before [vote]"
- "Which industries increased donations to [member] after they changed position on [issue]?"
- "Who are the top 10 donors to PACs supporting [member]?"

### Implementation Approach
- Use Claude API (claude-sonnet-4) with tool use for structured FEC queries
- Build a SQL query generation tool the agent calls against your local DB
- Add a "correlation scoring" function: temporal proximity of donation → vote, weighted by size
- Confidence thresholds — only surface correlations above a defined score to avoid noise

### Prototype Milestone
A CLI or simple web form where you type a question and get back a narrative answer + the underlying data. Not pretty yet — just correct and trustworthy.

---

## Phase 3 — Data Visualization (Weeks 6–9)

**Goal:** Make the data feel real, not abstract.

### Visualization Types (Priority Order)
1. **Money flow Sankey diagrams** — industry → PACs → candidates
2. **Timeline charts** — donation spikes overlaid on key votes
3. **Comparison bars** — "Senator X received 4x more from pharma than the Senate average"
4. **Geographic maps** — donor concentration by state/district

### Stack Suggestion
- **D3.js or Observable Plot** for custom viz
- **Recharts** if using React
- Keep charts embeddable as standalone images — critical for social sharing

### Key Design Principle
Every chart needs a **one-sentence plain-English headline** auto-generated above it. The data is for credibility; the headline is for impact.

---

## Phase 4 — Automated Content Distribution (Weeks 9–12)

**Goal:** The tool surfaces stories autonomously; you lightly curate before publishing.

### Automated "Story Finder" Agent
- Runs nightly against fresh FEC data
- Scores correlations for newsworthiness (size of donation, timing, vote margin, public salience of the issue)
- Drafts 3–5 candidate social posts per day
- Flags top candidate for your review before publishing

### Output Channels
| Channel | Format | Frequency |
|---|---|---|
| Bluesky | Single striking stat + chart image | 1x daily |
| Threads | Same, slight reformat | 1x daily |
| Substack newsletter | Weekly digest of top 5 findings, narrative format | 1x weekly |
| RSS feed | All findings, machine-readable | Continuous |

### Content Voice Guidelines
- **Lead with the number.** "$2.3M from pharmaceutical companies" not "significant pharmaceutical donations"
- **No editorializing.** State what the data shows. The data is damning enough.
- **Always link to primary source.** Every post links to the underlying FEC filing.
- **Flag uncertainty.** If correlation ≠ causation, say so briefly. Credibility is the product.

### Editorial Review Flow
1. Agent drafts posts nightly
2. You review a digest each morning (target: 5 minutes)
3. Approve, edit, or discard
4. Scheduler publishes approved posts

---

## Phase 5 — Public Web Interface (Weeks 12–16)

**Goal:** Give journalists, researchers, and engaged citizens a self-serve tool.

### Core Features (MVP)
- Natural language search bar ("What has Senator X received from oil companies?")
- Member profile pages with contribution breakdown + voting record overlay
- Industry pages showing which members they fund most heavily
- "Trending correlations" feed — most striking findings surfaced this week
- Embeddable charts for journalists

### Nice-to-Have (Post-MVP)
- Email alerts ("notify me when [member] receives donations from [industry]")
- API access for journalists and researchers
- "Explain this to me" mode — plain English explainer for any finding

---

## Credibility & Trust Architecture

This is the most important non-technical consideration.

- **Every claim links to primary source** (specific FEC filing ID)
- **Methodology page** explaining how correlations are scored and what they do/don't prove
- **Correction policy** — public log of any errors found and fixed
- **No anonymous posting** — your name or a named organization behind it
- **Distinction between correlation and causation** — always stated explicitly

---

## Infrastructure & Cost (Solo Dev Target)

| Component | Tool | Est. Monthly Cost |
|---|---|---|
| Database | Supabase free tier or Railway | $0–$20 |
| Hosting | Vercel or Fly.io | $0–$20 |
| Claude API | Agentic queries + content drafting | $20–$50 |
| Scheduling | GitHub Actions or Inngest | $0 |
| Social publishing | Buffer or custom Bluesky/Threads API | $0–$15 |
| **Total** | | **~$40–$105/month** |

---

## Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Accuracy error goes viral | High confidence threshold before publishing; always link to raw source |
| Perceived partisan bias | Cover all parties equally; let data speak; methodology transparency |
| FEC data quality issues | Use OpenSecrets pre-cleaned data as primary; FEC bulk as secondary |
| Burnout maintaining it | Design for <5 hrs/week after Phase 4; automate aggressively |
| Platform bans automated accounts | Multi-platform from day one; own your audience via newsletter/RSS |

---

## Success Metrics (12 Months)

- At least one finding cited by a journalist or major publication
- 1,000+ Substack subscribers
- Tool used by at least one advocacy org or newsroom regularly
- Personal conviction that it's surfacing things that wouldn't otherwise be found

---

## Immediate Next Steps

1. **This week:** Sign up for OpenSecrets API key; download one FEC bulk data file and explore the schema
2. **Next week:** Get a single end-to-end query working locally — one member, one industry, one vote correlation
3. **Week 3:** Build the first shareable chart and draft the methodology page

---

*"The truth is in the data. The impact is in the storytelling."*
