---
name: content-review
description: Analyze content (YouTube videos, articles, tweet threads, podcasts) and deliver a watch/read/skip verdict with extracted insights. Trigger when user shares a URL with keywords like "review", "analyze", "worth my time", "should I watch", "should I read", or "review this". Also triggers on "content review" or just a URL followed by a question mark.
---

# Content Review

Analyze external content against the user's goals, frameworks, and time value. Extract the insights so they don't have to consume the full content unless it's truly worth their time.

## Trigger Words
Any URL + one of: `review`, `analyze`, `worth my time?`, `should I watch`, `should I read`, `evaluate`, `?`

## Workflow

### Step 1: Detect Content Type
From the URL, determine:
- **YouTube video** → needs transcription
- **Article/blog** → needs extraction
- **Tweet/thread** → needs thread unwinding
- **Podcast** → needs transcription (if audio available)

### Step 2: Extract Content (Sub-Agent)
Spawn a sub-agent with explicit execution instructions. See [references/sub-agent-prompt.md](references/sub-agent-prompt.md) for the spawn template.

**Critical — Definition of Done for sub-agent:**
- Output file MUST exist at the specified path
- File MUST contain substantive content (not a plan)
- Verify with file check after sub-agent completes

The sub-agent saves raw content + structured summary to `docs/reviews/[slug].md`.

### Step 3: Review Against Frameworks
After the sub-agent delivers, review the content against:

1. **Buyback Test:** Is consuming this content the highest-value use of the user's time, or can the insight be captured faster?
2. **Relevance Filter:** Does this relate to active goals and priorities? Check USER.md and any active project files.
3. **Novelty Check:** Do we already know this? Cross-reference against existing docs in `docs/citadel-ops/`, `docs/reviews/`, and memory files.
4. **Action Density:** How many actionable insights per minute of content? High action density = worth consuming. Low = extract and move on.

### Step 4: Deliver Verdict
Format the response as:

**Verdict emoji + label:**
- ⏭️ **Skip** — We already know this or it's not relevant. Here's the 1-2 things worth noting.
- 👀 **Skim** — Some useful bits but not worth full attention. Here are the highlights.
- 📖 **Read** — Worth reading the article/summary. Key frameworks extracted below.
- 🎬 **Watch** — Visual/demo content that loses value in text. Worth the time investment.

**Then provide:**
- 3-5 key insights extracted (so the user gets value regardless of verdict)
- What's new vs what we already have
- Any specific actions to take based on the content

### File Organization
- Raw transcripts/extractions → `docs/reviews/[descriptive-slug].md`
- Never put reviews in `docs/citadel-ops/` (that's for operational playbooks only)

## Content Extraction Methods

### YouTube Videos
1. Try `web_search` for existing transcripts: `"[video-id] transcript"`
2. Try transcript services via `web_fetch`
3. Fall back to `yt-dlp` + Whisper (check for existing venv first)
4. Supplement with web research about the video's content

### Articles
1. `web_fetch` with markdown extraction
2. For X/Twitter articles: use `api.fxtwitter.com` which returns full article content in JSON

### Tweet Threads
1. `api.fxtwitter.com/[user]/status/[id]` for individual tweets
2. Follow reply chains for threads

## Notes
- Keep verdicts concise — mobile-friendly formatting (no tables, use bullet lists)
- The user's time is the scarcest resource. Default to extracting value, not recommending consumption.
- When in doubt, extract the insights and skip. The bar for "watch the whole thing" should be high.
