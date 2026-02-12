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

**Duration check (YouTube/podcasts):** Before spawning extraction, check video length via web_search or yt-dlp metadata (`yt-dlp --print duration [URL]`). If >60 minutes, warn the user:
> "This is a [X]-minute video. Full transcription will take a while. Want me to: (a) do full extraction, (b) extract intro + key chapters only, or (c) search for existing summaries/highlights?"

For videos <60 minutes, proceed automatically.

### Step 2: Extract Content (Sub-Agent)
Spawn a sub-agent with explicit execution instructions. See [references/sub-agent-prompt.md](references/sub-agent-prompt.md) for the spawn template.

**Critical — Output Verification:**
After the sub-agent completes, verify the output file:
1. Check that the file exists at the expected path
2. Check that it has substantive content (not empty, not just a plan/outline)
3. If verification fails, retry extraction with a direct approach (no sub-agent) or report the failure clearly

```
# Verification pattern (do this after sub-agent reports done):
read the output file
if file doesn't exist or is empty or is just an outline:
  → retry with web_search + web_fetch yourself (no sub-agent)
  → or clearly tell the user extraction failed and why
```

### Step 3: Review Against Frameworks
After verified extraction, review the content against:

1. **Relevance Filter:** Does this relate to the user's active goals and priorities? Check USER.md if it exists.
2. **Novelty Check:** Does the user (or their agent) already have this knowledge? Check existing docs for overlap.
3. **Action Density:** How many actionable insights per minute of content? High action density = worth consuming. Low = extract and move on.
4. **Time Value Test:** Is consuming this content the highest-value use of the user's time, or can the insight be captured faster from the extraction?

### Step 4: Deliver Verdict
Format the response as:

**Verdict emoji + label:**
- ⏭️ **Skip** — Already known or not relevant. Here's the 1-2 things worth noting.
- 👀 **Skim** — Some useful bits but not worth full attention. Here are the highlights.
- 📖 **Read** — Worth reading the summary. Key frameworks extracted below.
- 🎬 **Watch** — Visual/demo content that loses value in text. Worth the time investment.

**Then provide:**

📌 **Actions** (things to do based on this content):
- [Immediate actions, research tasks, things to add to systems]

💡 **Insights** (worth knowing, no action needed):
- [Key frameworks, mental models, interesting data points]

Also note:
- What's new vs what's already known
- Source quality assessment (credible? experienced? selling something?)

### Step 5: Save Review
Save the review to a sensible location:

1. **If `docs/reviews/` exists** → save there as `[descriptive-slug].md`
2. **If `docs/` exists but no `reviews/`** → create `docs/reviews/` and save there
3. **If no `docs/` directory** → create `docs/reviews/` and save there
4. **Never overwrite existing files** — append a number if slug already exists

The review file should contain:
- Source info (title, author, URL, date, length)
- Verdict and reasoning
- Full extracted content or detailed summary
- Actions and insights (separated)

## Content Extraction Methods

### YouTube Videos
1. Try `web_search` for existing transcripts: `"[video-id] transcript"`
2. Try transcript services via `web_fetch`
3. Fall back to `yt-dlp` + Whisper (use `--model base` — it's fast and reliable)
4. Supplement with web research about the video's content

### Articles
1. `web_fetch` with markdown extraction
2. For X/Twitter articles: use `api.fxtwitter.com` which returns full content in JSON

### Tweet Threads
1. `api.fxtwitter.com/[user]/status/[id]` for individual tweets
2. Follow reply chains for threads

### Podcasts
1. Search for existing transcripts first (many podcasts publish them)
2. Fall back to audio download + Whisper if available

## Notes
- Keep verdicts concise — mobile-friendly formatting (no tables, use bullet lists)
- The user's time is the scarcest resource. Default to extracting value, not recommending consumption.
- When in doubt, extract the insights and skip. The bar for "watch the whole thing" should be high.
- This skill works for any agent — it adapts to whatever knowledge base and doc structure already exists.
