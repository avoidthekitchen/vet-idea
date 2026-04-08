# Personalized AI Radio Station

*Concept Overview & Open Questions*

*Draft — April 2026*

---

## The Problem

Podcast discovery is broken for power listeners. You eventually exhaust the shows that match your interests, and searching for new ones is a time sink with diminishing returns. Existing podcasts are generic by nature — they're made for broad audiences, not for one person's evolving curiosity.

Meanwhile, AI research tools are proving that personalized, deep exploration of topics works well in text. The missing piece is extending that same personalization to audio — a format that fits naturally into commutes, walks, and chores.

## The Idea

An AI-powered, always-on personal radio station that generates audio content tailored to your interests. It acts like a research-savvy radio host who knows what you care about, finds relevant new information, and presents it to you as listenable content — on demand, endlessly.

The core loop is:

- Understand your interests from signals you already generate (saved articles, upvotes, bookmarks).
- Continuously discover and curate new sources — articles, papers, blog posts, reports — on those topics.
- Generate audio content (essays, briefings, conversations) from those curated sources.
- Adapt over time as your interests shift and as you steer it directly.

The key insight is that this is not AI synthesizing from scratch. It curates real sources first, then generates audio about those sources. The AI layer sits between discovery and presentation, not between nothing and content.

## Proof of Concept: NotebookLM

Google's NotebookLM already demonstrates the core value proposition in miniature. You feed it specific documents, and it generates a surprisingly engaging podcast-style conversation about those documents. The output is genuinely useful and more tailored than any existing podcast on the same topics.

The gap NotebookLM leaves open is automation and continuity. You have to manually select sources each time. There's no persistent interest profile, no ongoing discovery, and no continuous stream. This project would close that gap — making the NotebookLM experience automatic, personalized, and infinite.

## How It Learns Your Interests

Rather than asking users to manually configure topics, the system ingests signals from platforms where you already express preferences:

| Signal Source | What It Reveals |
|---|---|
| Instapaper / Pocket | Articles you've saved — strong intent signal for topics you want to go deeper on. |
| Hacker News favorites | Technical interests, emerging tools, industry trends you're tracking. |
| Reddit upvotes / saves | Broader interest graph across communities and subject areas. |
| Direct steering | Explicit requests: "Follow developments in agent infrastructure" or "More on energy policy." |
| Listening behavior | Skipping, replaying, or finishing episodes signals what's landing and what isn't. |

These signals feed an evolving interest profile. The system doesn't just match keywords — it should understand topic clusters and adjacent interests over time.

## Content Generation Pipeline

The generation pipeline has three stages:

### Stage 1: Source Discovery & Curation

Based on your interest profile, the system continuously scans for new, relevant content across the web — articles, research papers, blog posts, reports, policy documents. It ranks candidates by relevance, recency, and source quality, then selects the best material for audio generation.

### Stage 2: Script Generation

An LLM reads the curated sources and produces an audio script. Formats could include briefings ("Here's what happened this week in agent infrastructure"), deep dives (a 15-minute essay synthesizing three papers on a topic), or conversational discussions (two-voice NotebookLM-style dialogue about a set of articles).

### Stage 3: Audio Synthesis

Text-to-speech converts the script to audio. Voice quality is critical — the output needs to be pleasant enough for extended passive listening, not robotic or fatiguing.

## User Experience (V1)

The V1 experience is passive. You press play and a personalized stream begins — a mix of topic briefings, deep dives, and updates on areas you follow. It behaves like a radio station that happens to only cover things you care about.

You can steer it by saving articles, favoriting content on linked platforms, or explicitly telling it to follow or drop a topic. But the default mode is lean-back: it handles discovery and presentation, you just listen.

A future V2 or V3 could introduce active engagement — the ability to pause, ask follow-up questions, request more depth on something just mentioned, or branch into a related topic in real time. But that's a fundamentally different interaction model and should be treated as a separate phase.

## Open Questions

The following questions were raised during initial brainstorming and remain unresolved. They represent the areas most likely to shape architectural and product decisions.

### Source Discovery & Curation

- How does the system decide what new content to surface? Keyword matching against saved topics is brittle. Semantic search is better but expensive at scale. What's the right balance?
- How do you prevent algorithmic drift — the system gradually feeding you tangentially related rabbit holes instead of genuinely useful updates?
- How broad should the crawl be? Full web? Curated source lists? RSS feeds from high-quality publications? Each has different cost and quality tradeoffs.
- How does source quality get assessed? Not all articles on a topic are worth turning into audio. What signals indicate a source is substantive enough?

### Content Quality & Depth

- When the AI generates scripts from sources, how do you ensure it's doing genuinely useful intellectual work rather than producing plausible-sounding but shallow summaries?
- What's the right mix of content types? All briefings gets monotonous. All deep dives is exhausting. How does the system calibrate?
- How do you handle topics where the AI doesn't have enough new information to say anything interesting? Silence is better than filler, but how does the system know the difference?

### Two Different Products in One

- Generating audio from content you've already saved is one product (known sources, on-demand generation).
- Discovering new content based on your interest profile and generating audio from that is a different product (unknown sources, continuous curation).
- These have different technical requirements, different quality bars, and different user expectations. Which comes first? Can they coexist in V1, or does one need to be solved before the other?

### Voice & Listening Experience

- Current TTS is good but not great for extended listening. Is the technology ready for 30+ minute sessions without listener fatigue?
- Single voice or multi-voice? Conversational formats (like NotebookLM) are more engaging but harder to produce consistently.
- How do you handle pacing, transitions between topics, and "station" feel so it doesn't feel like a series of disconnected segments?

### Cost & Feasibility

- LLM-based content ranking at scale is expensive. What's the per-user cost of continuous source discovery?
- Audio generation (TTS) adds another cost layer. What's the unit economics of generating, say, 2 hours of personalized audio per user per day?
- Can the system pre-generate content during off-peak hours to reduce real-time compute pressure, or does freshness matter too much?

### Competitive Landscape

- NotebookLM could add continuous discovery and become this product overnight. What's the defensibility?
- Spotify, Apple, and other podcast platforms have the distribution and listener data. If they build AI-generated personalized shows, how does an indie project compete?
- Is there a niche (power users, specific verticals, privacy-conscious listeners) where a smaller player has a structural advantage?

## Possible Next Steps

1. Build a minimal proof of concept: take 3–5 saved Instapaper articles, use an LLM to generate a script, convert to audio with a TTS API. Evaluate whether the output is worth listening to.
2. Test the source discovery loop: given a set of interest signals, can an LLM reliably find and rank new content that a human would consider relevant and high-quality?
3. Estimate unit economics: cost per minute of generated audio, cost per discovery cycle, and what that implies for a sustainable product.
4. Determine V1 scope: just saved-article-to-audio, or full discovery + generation from day one?
