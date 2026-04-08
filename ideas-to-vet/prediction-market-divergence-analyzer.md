# Prediction Market Divergence Analyzer

**Project Research & Prototype Plan**

*A cross-platform insight tool for understanding why prediction markets disagree*

April 2026 — DRAFT

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Core Thesis & Value Proposition](#2-core-thesis--value-proposition)
3. [Competitive Landscape](#3-competitive-landscape)
4. [Technical Architecture](#4-technical-architecture)
5. [Phased Prototype & MVP Plan](#5-phased-prototype--mvp-plan)
6. [Risks, Limitations & Mitigations](#6-risks-limitations--mitigations)
7. [Infrastructure & Estimated Cost](#7-infrastructure--estimated-cost)
8. [Success Metrics](#8-success-metrics)
9. [Immediate Next Steps](#9-immediate-next-steps)

---

## 1. Executive Summary

This document captures the research, competitive analysis, and phased build plan for a prediction market divergence analysis tool. The core idea: when Kalshi and Polymarket disagree on the same event, something interesting is happening. The goal is not to trade on those gaps, but to explain them — surfacing insights for data analysts, journalists, researchers, and civically engaged citizens.

The tool would ingest real-time pricing data from both platforms, detect meaningful divergences, classify them by type, correlate them with external data sources, and produce narrative explanations. After events resolve, it would score each platform's accuracy over time, building a rolling calibration record.

The competitive landscape is active but narrowly focused on arbitrage execution. No existing tool seriously attempts to explain why divergence exists or who gets events right more often. This represents a genuine whitespace opportunity, particularly for journalistic and civic applications.

---

## 2. Core Thesis & Value Proposition

Prediction markets are genuinely different information ecosystems. Kalshi's users are US-based, KYC'd, CFTC-regulated, with dollar limits on positions. Polymarket's users are largely crypto-native, international, pseudonymous, and can take substantially larger positions. When these two populations disagree significantly on the same event, the disagreement itself is informative.

The strongest evidence comes from high-volume markets. During the 2024 US election, Polymarket and Kalshi showed persistent spreads on identical outcomes, and post-mortems suggested Polymarket's whale traders had better-calibrated probabilities. For Fed decisions, major elections, and geopolitical events, divergence windows of 2–8% emerge hours to days before resolution — well above structural noise.

### 2.1 Target Use Cases

- **Data analysis and personal curiosity:** Understanding how information flows differently through regulated vs. unregulated markets.
- **Journalism:** Surfacing patterns like "A single wallet that was correct on 8 of 10 previous political markets placed a large trade 6 hours before this divergence began."
- **Civic research:** Building a public record of which platforms are better calibrated on which categories of events over time.
- **Academic:** Providing structured datasets for prediction market efficiency research.

---

## 3. Competitive Landscape

The prediction market tooling ecosystem is active and growing, but almost exclusively focused on trading execution — finding and exploiting price gaps. The explanation and insight layer is essentially unoccupied.

### 3.1 Existing Tools

| Tool | Focus | Pricing | Relevance |
|------|-------|---------|-----------|
| Oddpool | Cross-venue arbitrage scanning, whale tracking, dominance index, WebSocket API | Free tier + paid Pro/API | Closest overlap on data aggregation; no explanation layer |
| Prediction Hunt | Free aggregator across Kalshi, Polymarket, PredictIt, ProphetX with arbitrage alerts | Free | Good market matching reference; no analysis beyond price gaps |
| OddsPipe | Developer API for cross-platform prices, historical candlesticks, spread detection (88K+ markets) | Free API key | Useful as data source or sanity check; pure data, no insight |
| Prediedge | Whale tracking and insider activity signals for Polymarket and Kalshi | Paid | Overlaps on whale tracking; no external data correlation |
| Open-source arb bots (GitHub) | Python bots using text similarity to match 10K+ markets across platforms | Free / MIT | Valuable for market-matching logic; no insight layer |

### 3.2 The Gap: Nobody Explains Why

Every existing tool answers "where is the divergence?" None seriously attempts "why does the divergence exist?" Specifically, no current tool:

- Correlates divergence events with external data sources (congressional disclosures, SEC filings, breaking news)
- Classifies divergence by temporal pattern (gradual drift vs. sudden spike vs. persistent disagreement)
- Tracks which platform was correct after resolution, building calibration scores over time
- Produces narrative explanations suitable for non-trader audiences
- Maps on-chain wallet behavior to divergence timing for pattern detection

### 3.3 Build vs. Buy Assessment

**Build from raw APIs:** Both Kalshi and Polymarket offer free, documented APIs. At 15-minute polling, rate limits are not a concern. No paid aggregator subscription is needed for the MVP.

**Leverage open source:** Existing MIT-licensed arb bots provide tested market-matching logic via text similarity. This is the hardest plumbing problem and it's already solved.

**Use free tools as sanity checks:** Prediction Hunt's free tier and Oddpool's free search can validate matching accuracy without creating a dependency.

---

## 4. Technical Architecture

### 4.1 Layer 1: Data Ingestion

Primary data sources for the MVP and subsequent phases:

| Source | Data Provided | Access Method | Phase |
|--------|--------------|---------------|-------|
| Kalshi REST API | Market prices, order books, trade history | Free API, polling every 15 min | MVP |
| Polymarket CLOB API | Market prices, order book depth, trade history | Free API, polling every 15 min | MVP |
| Polygon blockchain (Polymarket) | On-chain wallet data, trade attribution, position sizes | Public RPC / Dune Analytics | Phase 2 |
| Congressional trading disclosures | Legislator stock trades via Capitol Trades / Quiver Quantitative | Free API / scraping | Phase 2 |
| SEC EDGAR | 8-K filings, Form 4 insider transactions | Free API | Phase 3 |
| News APIs (NewsAPI, Benzinga, or similar) | Breaking news, sentiment, event catalysts | Free tier or paid | Phase 2 |
| Federal Register | Proposed rules, comment periods, regulatory calendar | Free API | Phase 3 |
| Social media (Bluesky API, X) | Narrative tracking, community sentiment | Free (Bluesky) / paid (X) | Phase 3 |

### 4.2 Layer 2: Market Matching & Divergence Detection

The contract matching problem is the core engineering challenge. Kalshi and Polymarket may phrase the same event differently and use different resolution criteria. The approach:

- Adopt text similarity matching from existing open-source arb bots as a starting point.
- Manually curate and verify matches for the initial 5–10 high-volume markets in the MVP.
- Build automated matching with NLP in Phase 2, using the manual curation as a training/validation set.
- Subtract structural baseline spread (1–3% from fees, position limits, regulatory differences) before flagging informational divergence.
- Classify divergence events by temporal pattern: gradual drift, sudden spike, convergence after divergence, persistent disagreement.

### 4.3 Layer 3: Divergence Explanation Engine

For each divergence event exceeding a configurable threshold, the engine correlates with external data:

- Recent news events (within a configurable time window around the divergence)
- SEC filings and congressional trading disclosures temporally proximate to the divergence
- On-chain wallet analysis: concentration, whale activity, historical accuracy of specific wallets
- Volume and order book depth differences between platforms (thin markets produce mechanical divergence)
- LLM-powered narrative generation (Claude API) to produce plain-English summaries of each divergence event

### 4.4 Layer 4: Resolution Tracking & Calibration Scoring

After events resolve, the system scores each platform's performance: which was closer to the correct outcome, which moved toward the correct price first, and how often. Over time, this builds an independently valuable dataset — a rolling calibration leaderboard for prediction markets that journalists and researchers would cite.

### 4.5 Layer 5: Narrative Mapping (Post-MVP)

For each significant divergence event, use an LLM to pull and cluster relevant news articles, social posts, and official statements into a timeline alongside price movements. The output: an automated investigative groundwork package showing what each market priced, what the public information environment looked like, and when they converged.

### 4.6 Layer 6: Historical Analogues (Post-MVP)

When a new divergence event occurs, search the historical database for structurally similar past events (same category, similar pattern, comparable volume). Surface how those resolved. Example output: "The last three times these platforms diverged by 8%+ on a Fed market, Polymarket was closer to the outcome."

---

## 5. Phased Prototype & MVP Plan

### 5.1 Phase 0 — Manual Validation (Week 1)

**Goal:** Confirm the divergence signal exists and is interesting before writing code.

| Task | Output | Time |
|------|--------|------|
| Pick 5–10 high-volume markets active on both Kalshi and Polymarket (Fed, elections, major geopolitical) | Curated market list with verified matching resolution criteria | 2 hrs |
| Manually track prices on both platforms daily for one week, logging in a spreadsheet | Spreadsheet with divergence observations, timestamps, and notes | 15 min/day |
| For each observed divergence, search for correlated news or wallet activity | Annotated log of divergence events with candidate explanations | 3 hrs |
| Assess: Are the patterns interesting enough to automate? | Go/no-go decision for Phase 1 | 1 hr |

### 5.2 Phase 1 — Data Pipeline MVP (Weeks 2–4)

**Goal:** Automated data collection and divergence detection for curated markets.

| Task | Technical Notes | Priority |
|------|----------------|----------|
| Set up local database (DuckDB or SQLite for simplicity) | Lightweight, zero-config; upgrade to Postgres later if needed | P0 |
| Write polling scripts for Kalshi and Polymarket APIs (15-minute intervals) | Python with requests library; cron or simple scheduler | P0 |
| Implement manual market matching for 5–10 curated contracts | JSON config mapping Kalshi ticker to Polymarket CLOB ID | P0 |
| Build divergence detection: flag when spread exceeds configurable threshold (e.g., 3%+) | Simple comparison after subtracting estimated structural baseline | P0 |
| Store historical price snapshots for both platforms | Enables backtesting and resolution scoring later | P0 |
| Basic CLI or notebook output showing divergence events with timestamps | No frontend yet; validate data quality first | P1 |

**Polling frequency:** Every 15 minutes. This captures divergence events that develop over hours to days, filters out sub-minute noise, stays well within API rate limits, and is more than sufficient for a next-day-hold or insight-only use case.

### 5.3 Phase 2 — Explanation Layer (Weeks 5–8)

**Goal:** For each divergence event, surface candidate explanations from external data.

| Task | Technical Notes | Priority |
|------|----------------|----------|
| Integrate news API (NewsAPI or similar) to pull articles timestamped around divergence events | Keyword search based on market topic; rank by temporal proximity | P0 |
| Add Polygon on-chain data for Polymarket whale tracking | Track wallet concentration, large trades, historical accuracy of specific wallets | P0 |
| Implement divergence classification (gradual drift, sudden spike, convergence, persistent) | Time-series pattern matching on the stored price history | P1 |
| Build LLM-powered narrative summaries (Claude API) | Input: divergence data + correlated news + wallet activity. Output: plain-English explanation. | P1 |
| Add congressional trading disclosures (Capitol Trades) | Temporal correlation: did a legislator trade before a divergence event? | P2 |

### 5.4 Phase 3 — Resolution Scoring & Dashboard (Weeks 9–12)

**Goal:** Track who gets events right, and provide a visual interface.

| Task | Technical Notes | Priority |
|------|----------------|----------|
| Implement resolution tracking: after event resolves, score Kalshi vs. Polymarket accuracy | Which was closer? Which moved toward correct price first? By how much? | P0 |
| Build rolling calibration leaderboard by market category | Fed, elections, crypto, sports — each scored independently | P0 |
| Build simple web dashboard (React or plain HTML) showing active divergences | Display: market, both prices, spread, classification, correlated signals | P0 |
| Add historical divergence timeline view | D3.js or Recharts; price overlay with news/event annotations | P1 |
| Add divergence detail pages with narrative explanation | LLM-generated summary + underlying data + links to primary sources | P1 |

### 5.5 Phase 4 — Advanced Features (Weeks 13–20)

**Goal:** Expand data sources, automate content, and increase analytical depth.

- Automated NLP-based market matching to expand beyond manually curated contracts.
- Narrative mapping layer: LLM-generated timelines of news + prices + wallet activity per divergence event.
- Historical analogues: when a new divergence occurs, surface structurally similar past events and how they resolved.
- SEC EDGAR integration for 8-K filings and Form 4 insider transactions.
- Social media narrative tracking (Bluesky API) to correlate community sentiment with platform-specific price moves.
- Automated content generation: weekly digest of most interesting divergence findings for a newsletter or blog.
- RSS feed and embeddable charts for journalists.

---

## 6. Risks, Limitations & Mitigations

| Risk | Severity | Mitigation |
|------|----------|------------|
| Platform risk: Polymarket or Kalshi restricts API access, changes terms, or shuts down | High | Modular architecture to swap platforms; track Metaculus, Manifold as fallbacks |
| Small N problem: only 20–50 meaningfully comparable contracts at any given time | Medium | Frame as case study and insight tool, not statistical claims; value compounds over time |
| Attribution is always speculative: cannot prove a first mover "knew" something | Medium | Frame as "suggestive patterns," not conclusions; always state correlation vs. causation |
| Structural divergence mistaken for informational signal | Medium | Estimate and subtract baseline spread from fees, position limits, and regulatory differences |
| Resolution criteria mismatch between platforms | High | Manual curation for MVP; automated flagging when criteria text changes mid-market |
| Audience sustainability: journalists/researchers are hard to retain beyond election cycles | Medium | Build compounding assets (calibration data, historical database) that increase in value over time |
| Regulatory gray area if insights are used for trading | Low (for insight tool) | Position explicitly as analysis/journalism tool; no trading recommendations |

### 6.1 Hard Limitations That Cannot Be Fully Mitigated

- **Platform risk is existential and outside your control.** Polymarket operates in regulatory uncertainty; the CFTC could act against it at any time.
- **The small N problem is a permanent constraint** of the prediction market space's current size. This limits statistical power but not case-study value.
- **Attribution will always be speculative.** You can show timing patterns but never prove knowledge. This is acceptable for journalism if framed carefully.

---

## 7. Infrastructure & Estimated Cost

| Component | Tool / Service | Est. Monthly Cost |
|-----------|---------------|-------------------|
| Database | DuckDB or SQLite (local) → Supabase or Railway (if scaling) | $0–$20 |
| Hosting | Vercel, Fly.io, or Railway | $0–$20 |
| Claude API | Narrative generation, divergence explanations | $10–$30 |
| News API | NewsAPI free tier or Benzinga | $0–$25 |
| On-chain data | Public Polygon RPC or Dune Analytics free tier | $0 |
| Scheduling | Cron / GitHub Actions / Inngest free tier | $0 |
| **Total** | | **$10–$95/month** |

---

## 8. Success Metrics

### 8.1 Phase 0–1 (Validation)

- At least 3 divergence events observed manually that have plausible, interesting explanations.
- Automated pipeline captures divergence events that match manual observations.
- Personal conviction that the data is interesting enough to continue.

### 8.2 Phase 2–3 (MVP)

- Explanation engine produces narratives that a non-expert would find informative for at least 50% of flagged divergence events.
- Resolution scoring shows measurable calibration differences between platforms in at least one market category.
- At least one divergence finding interesting enough to write about publicly.

### 8.3 12-Month Horizon

| Metric | Target |
|--------|--------|
| Divergence events analyzed | 200+ |
| Resolved events with calibration scoring | 100+ |
| Findings cited by a journalist or researcher | At least 1 |
| Public dashboard monthly visitors | 500+ |
| Newsletter / blog subscribers (if launched) | 250+ |

---

## 9. Immediate Next Steps

**This week:** Identify 5–10 high-volume markets active on both platforms. Manually verify that resolution criteria match. Begin daily price logging in a spreadsheet.

**Next week:** For each observed divergence, search for correlated news, wallet activity, or regulatory events. Assess whether the patterns are interesting.

**Week 3:** If validation is positive, begin Phase 1: set up database, write polling scripts, implement basic divergence detection.

**Week 4:** Review open-source arb bot code for market-matching logic. Adapt for your pipeline.

---

*The data already exists. The gap is the explanation layer.*
