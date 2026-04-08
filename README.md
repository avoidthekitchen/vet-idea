# Idea Validator

An agent skill that pressure-tests startup and product ideas against live market signals. Not idea generation — idea interrogation. You bring a thesis, the skill tells you whether the evidence supports it.

Given a product or business idea, the skill:

1. Extracts keywords (problem terms, audience, synonyms, candidate subreddits)
2. Searches Reddit for pain-point evidence
3. Checks Google Trends for demand trajectory
4. Scans the web for competitive landscape
5. Synthesizes everything into a quick-screen scorecard with a verdict

## Example Scorecard

```
IDEA: AI-powered invoice reconciliation for freelancers

PAIN POINT EVIDENCE:    Strong
TREND DIRECTION:        Accelerating
COMPETITOR DENSITY:     Moderate
QUICK VERDICT:          Investigate Further
```

See [`example-scorecards/`](example-scorecards/) for full examples with raw data.

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
├── SKILL.md                     # Instructions + pipeline for the agent
├── scripts/
│   ├── reddit_search.py         # Reddit pain-point search (no API key)
│   ├── trends_check.py          # Google Trends via trendspy (no API key)
│   └── web_scan.py              # Web search (Brave + Tavily + DuckDuckGo)
└── references/
    └── scoring-rubric.md        # Detailed scoring criteria with examples
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

| Source | What it provides | Cost | API key |
|--------|-----------------|------|---------|
| Reddit | Pain-point evidence, frustration signals | Free | None |
| Google Trends | Interest trajectory, trend direction | Free | None |
| Brave Search | High-quality web results | Free tier: $5/mo | Optional |
| Tavily | AI-optimized search results | Free tier: 1K/mo | Optional |
| DuckDuckGo | Web results (fallback) | Free | None |

## License

See [LICENSE](LICENSE).
