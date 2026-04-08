# Idea Validator

An agent skill that pressure-tests startup and product ideas against live market signals. Not idea generation — idea interrogation. You bring a thesis, the skill tells you whether the evidence supports it.

## Two Modes

### Quick Screen (1-2 min)

Fast analysis using three bundled Python scripts:

1. Extracts keywords (problem terms, audience, synonyms, candidate subreddits)
2. Searches Reddit for pain-point evidence
3. Checks Google Trends for demand trajectory
4. Scans the web for competitive landscape
5. Synthesizes everything into a scorecard with verdict, competitor profile, and perceptual map

### Deep Dive (5-10 min)

Runs the Quick Screen first, then layers on analysis using agent-native web tools:

1. **Competitor deep profiling** — visits competitor landing pages and pricing pages via web_fetch, extracting pricing tiers, features, positioning, and maturity signals
2. **Product Hunt traction scan** — finds recent launches in the space, extracts upvote counts and community engagement
3. **G2/Capterra review sentiment** — identifies top complaints and praises across competitors
4. **Search volume estimation** — directional search volume from publicly available sources
5. **Fermi market sizing** — structured TAM/SAM/SOM estimate with stated assumptions and confidence level
6. **Full validation report** — 8-section report with 1-5 scoring across demand, timing, competition, and distribution difficulty

## Example Output

**Quick Screen:**
```
PAIN POINT EVIDENCE:    Strong
TREND DIRECTION:        Accelerating
COMPETITOR DENSITY:     Moderate
QUICK VERDICT:          Investigate Further
```

**Deep Dive:**
```
Demand strength:         3/5
Timing:                  4/5
Competition intensity:   4/5
Distribution difficulty:  4/5
VERDICT:                 Pass
```

See [`example-scorecards/`](example-scorecards/) for full scorecards and reports with raw data.

## Install

Requires [uv](https://docs.astral.sh/uv/).

```bash
git clone https://github.com/avoidthekitchen/vet-idea.git
cd vet-idea
uv sync
```

## API Keys (optional)

The skill works with zero configuration — DuckDuckGo and Reddit require no API keys. For better web search coverage, add one or both:

```bash
export BRAVE_API_KEY="your_key"    # Free: $5/month credit — https://brave.com/search/api/
export TAVILY_API_KEY="your_key"   # Free: 1,000 searches/month — https://tavily.com/
```

Add these to your shell profile (e.g. `~/.zshrc` or `~/.zprofile`) to persist across sessions.

## Use as an Agent Skill

This skill is designed to be loaded by AI coding agents (Claude Code, OpenCode, Codex, etc.). To install:

1. Copy or symlink the skill directory to your agent's skills folder
2. The agent will trigger it when you ask to validate, vet, evaluate, or stress-test an idea

For Claude Code:
```bash
ln -s $(pwd)/skills/idea-validator ~/.claude/skills/idea-validator
```

Then in a new session:
```
> Validate this idea: AI-powered invoice reconciliation for freelancers

# Or go straight to a Deep Dive:
> Should I build an AI-powered radio station?
```

## Run Scripts Manually

```bash
# Reddit pain-point search
uv run python skills/idea-validator/scripts/reddit_search.py \
  --keywords "freelancer invoicing,expense tracking" \
  --subreddits "freelance,smallbusiness,bookkeeping"

# Google Trends
uv run python skills/idea-validator/scripts/trends_check.py \
  --keywords "freelance invoicing,invoice automation"

# Web competitor scan
uv run python skills/idea-validator/scripts/web_scan.py \
  --query "AI invoice reconciliation tool for freelancers"
```

## Skill Structure

```
skills/idea-validator/
├── SKILL.md                     # Instructions + pipelines for the agent
├── scripts/
│   ├── reddit_search.py         # Reddit pain-point search (no API key)
│   ├── trends_check.py          # Google Trends via trendspy (no API key)
│   └── web_scan.py              # Web search (Brave + Tavily + DuckDuckGo)
└── references/
    ├── scoring-rubric.md        # Quick Screen scoring criteria with examples
    └── deep-dive-rubric.md      # Deep Dive methodology for all 8 report sections
```

## Project Structure

```
vet-idea/
├── skills/                      # Agent skills
│   └── idea-validator/          # The idea validator skill
├── ideas-to-vet/                # Ideas to validate (source material)
├── example-scorecards/          # Completed scorecards from test runs
├── rpi/plans/                   # Original prototype plan (Phase 0 spec)
└── pyproject.toml               # Python dependencies (managed by uv)
```

## Data Sources

### Quick Screen (scripts)

| Source | What it provides | Cost | API key |
|--------|-----------------|------|---------|
| Reddit | Pain-point evidence, frustration signals | Free | None |
| Google Trends | Interest trajectory, trend direction | Free | None |
| Brave Search | High-quality web results | Free tier: $5/mo | Optional |
| Tavily | AI-optimized search results | Free tier: 1K/mo | Optional |
| DuckDuckGo | Web results (fallback) | Free | None |

### Deep Dive (agent-native)

| Source | What it provides | Cost |
|--------|-----------------|------|
| Competitor sites | Pricing, features, positioning, maturity | Free |
| Product Hunt | Launch traction, upvotes, community engagement | Free |
| G2 / Capterra / Trustpilot | Review sentiment, top complaints and praises | Free |
| Public search data | Directional search volume estimates | Free |

Deep Dive uses the agent's built-in `web_search` and `web_fetch` tools — no additional setup required.

## License

See [LICENSE](LICENSE).
