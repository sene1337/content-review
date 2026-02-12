---
name: content-review
description: Analyze content (YouTube videos, articles, tweet threads, podcasts) and deliver a watch/read/skip verdict with extracted insights. Trigger when user shares a URL with keywords like "review", "analyze", "worth my time", "should I watch", "should I read", or "review this". Also triggers on "content review" or just a URL followed by a question mark.
---

# Content Review

Analyze external content against the user's goals, frameworks, and time value. Extract the insights so they don't have to consume the full content unless it's truly worth their time.

## Trigger Words
Any URL + one of: `review`, `analyze`, `worth my time?`, `should I watch`, `should I read`, `evaluate`, `?`

## Workflow

### Step 0: Pre-Flight Checks ⭐
Before any extraction, verify your environment:

1. **Check tool availability:**
   - `yt-dlp --version` → if missing, skip to web-based extraction
   - `whisper --help` → if missing, skip to web-based extraction
   - Verify `web_search` works (not rate-limited)
2. **Detect environment constraints:**
   - Datacenter IPs get blocked by YouTube (yt-dlp will fail) — if a prior yt-dlp attempt returned 403/HTTP error, don't retry
   - If running in a sandbox without local tools, go straight to Tier 1 (web-based) extraction
3. **Set extraction strategy BEFORE starting:**
   - Tools available + residential IP → full pipeline (yt-dlp + whisper)
   - Tools missing OR datacenter → web-first extraction (search for transcripts/summaries)
   - No results from web → ask user for help

**If pre-flight fails, skip directly to the appropriate fallback tier. Do not attempt methods that will fail.**

### Step 1: Detect Content Type
From the URL, determine:
- **YouTube video** → needs transcription
- **Article/blog** → needs extraction
- **Tweet/thread** → needs thread unwinding
- **Podcast** → needs transcription (if audio available)

**Duration check (YouTube/podcasts):** Before spawning extraction, check video length via web_search or yt-dlp metadata (`yt-dlp --print duration [URL]`). If >60 minutes, warn the user:
> "This is a [X]-minute video. Full transcription will take a while. Want me to: (a) do full extraction, (b) extract intro + key chapters only, or (c) search for existing summaries/highlights?"

For videos <60 minutes, proceed automatically.

### Step 2: Extract Content

**Use the fallback hierarchy — stop at the first tier that works:**

**Tier 1: Lightweight web extraction (~500 tokens)** ⭐ Try first
- YouTube: `web_search "[video title] transcript"` or `web_search "[video-id] transcript"`
- YouTube: `web_fetch` on known transcript services (e.g., `kome.ai/api/transcript?url=[URL]`)
- YouTube: oEmbed for metadata (`https://www.youtube.com/oembed?url=[URL]&format=json`)
- Articles: `web_fetch` with markdown extraction
- Tweets: `api.fxtwitter.com/[user]/status/[id]`
- If sufficient content extracted → skip to Step 3

**Tier 2: Sub-agent extraction (~5k tokens)** — Only if Tier 1 fails AND tools are available
- Spawn sub-agent with explicit instructions (see [references/sub-agent-prompt.md](references/sub-agent-prompt.md))
- **Critical: After spawning, WAIT for the sub-agent to complete. Do NOT attempt parallel extraction.**
- Verify output file exists and has substantive content before proceeding

**Tier 3: Ask user for help (~1k tokens)** — Last resort
- "I couldn't extract this content automatically. Could you paste the transcript/key points, or should I work from the title and description only?"

**Retry Limits (hard caps):**
- `web_search`: max 2 attempts per URL
- `web_fetch`: max 1 attempt per specific URL
- `yt-dlp`: max 1 attempt (if blocked, it's blocked — don't retry)
- Total retries across all methods: abort if >5
- If a method returns an error, log it and move to next tier — don't repeat

**Output Verification (after any extraction):**
1. Check that the output file exists at the expected path
2. Check that it has substantive content (not empty, not just a plan/outline)
3. If verification fails after Tier 2, fall through to Tier 3 — don't retry the same approach

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

## Known Limitations

- **YouTube bot detection:** YouTube blocks yt-dlp from datacenter/cloud IPs. If you're running on a VPS or in a sandbox, yt-dlp will likely fail with 403 errors. Use web-based transcript extraction (Tier 1) instead.
- **Rate limiting:** web_search providers may rate-limit after repeated queries. Space out searches or reduce query count.
- **Long videos (>2hr):** Whisper transcription is CPU-intensive. For very long content, prefer searching for existing transcripts.

## Notes
- Keep verdicts concise — mobile-friendly formatting (no tables, use bullet lists)
- The user's time is the scarcest resource. Default to extracting value, not recommending consumption.
- When in doubt, extract the insights and skip. The bar for "watch the whole thing" should be high.
- This skill works for any agent — it adapts to whatever knowledge base and doc structure already exists.
