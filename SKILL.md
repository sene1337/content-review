---
name: content-claw
description: Analyze content (YouTube videos, articles, tweet threads, podcasts) and deliver a watch/read/skip verdict with extracted insights. Trigger when user shares a URL with keywords like "review", "analyze", "worth my time", "should I watch", "should I read", or "review this". Also triggers on "content review", "content claw", or just a URL followed by a question mark. Do NOT use for simple link sharing without review intent, bookmarking URLs for later, or when user just wants to open/visit a link.
---

# Content Claw 🦞

Analyze external content against the user's goals, frameworks, and time value. Extract the insights so they don't have to consume the full content unless it's truly worth their time.

**Folder structure:**
- `docs/content-claw/caught/` — 📖 Read and 🎬 Watch verdicts. The keepers.
- `docs/content-claw/released/skim/` — 👀 Skim verdicts. Monthly batch review, then purged.
- `docs/content-claw/released/skip/` — ⏭️ Skip verdicts. Auto-purge after 14 days.

**File naming scheme:** `{category}--{source}--{title-slug}.md`
- **Category:** `ai-agents`, `bitcoin`, `health`, `media`, `business`, `tech`, `finance`, `culture`, `ops` (extend as needed)
- **Source type:** `article`, `video`, `podcast`, `thread`, `paper`
- **Title slug:** Short descriptive slug using hyphens
- **Separator:** Double-dash `--` (doesn't conflict with hyphens in slugs)
- Example: `ai-agents--article--clawvault-memory-architecture.md`

At a glance you can see category, format, and topic without opening the file.

## Workflow

### Step -1: Duplicate Check ⭐⭐
**Before ANY extraction work**, check if this URL has already been reviewed:
1. `grep -rl "<cleaned-URL>" docs/content-claw/` (search all caught + released folders)
2. Also check partial URL matches (e.g., tweet ID, YouTube video ID) in case the URL format differs
3. **If found:** Tell the user it's already been reviewed, show the file path and verdict. Ask if they want a re-review or update.
4. **If not found:** Proceed to Step 0.

This prevents duplicate files and wasted extraction work. No exceptions.

### Step 0: Pre-Flight Checks ⭐
Verify tools and set extraction strategy before starting. See [references/environment-setup.md](references/environment-setup.md) for full checklist.

### Step 0.5: Sanitize URLs 🧹
Before fetching or searching ANY shared URL, strip tracking parameters:
- **Twitter/X:** Remove `?s=`, `&s=`, `?t=`, `&t=`, `?ref_src=`, `&ref_src=`
- **YouTube:** Remove `&si=`, `?si=`, `&feature=`, `?feature=`, `&pp=`
- **Universal:** Remove `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content`, `utm_id`, `fbclid`, `gclid`, `igshid`, `ref`, `mc_cid`, `mc_eid`
- **Rule:** Strip everything after `?` or `&` that matches these patterns. Keep essential params (e.g., YouTube `v=`, `t=` for timestamps, `list=` for playlists).

Example: `https://youtube.com/watch?v=abc123&si=tracking_garbage&t=120` → `https://youtube.com/watch?v=abc123&t=120`

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

**⚠️ Context Window Protection Rule:**
The purpose of this skill is to keep the main agent's context clean. Heavy extraction work MUST happen in a sub-agent, not in the main session.

- **Tier 1 (web-based, <3k tokens):** OK to run in main agent — lightweight metadata and search results
- **Tier 2 (transcript/content extraction, 5k+ tokens):** MUST spawn a sub-agent. No exceptions. The main agent's job is to orchestrate and review, not to do heavy extraction.
- **Model routing:** Sub-agents should use a cheaper model (e.g., Sonnet) for extraction work. The main agent handles judgment (verdict, relevance, actions) on its primary model.

**If Tier 1 produces enough content for a solid review → proceed to Step 3.**
**If Tier 1 is insufficient → spawn a sub-agent for Tier 2. Do NOT do Tier 2 work yourself.**

---

**Use the fallback hierarchy — stop at the first tier that works:**

**Tier 1: Lightweight web extraction (~500-2.5k tokens)** ⭐ Try first, main agent
- YouTube: `web_search "[video title] transcript"` or `web_search "[video-id] transcript"`
- YouTube: `web_fetch` on known transcript services (e.g., `kome.ai/api/transcript?url=[URL]`)
- YouTube: oEmbed for metadata (`https://www.youtube.com/oembed?url=[URL]&format=json`)
- Articles: `web_fetch` with markdown extraction
- Tweets: `api.fxtwitter.com/[user]/status/[id]`
- If sufficient content extracted → skip to Step 3

**Tier 2: Sub-agent extraction (~5k tokens)** — ALWAYS a sub-agent, NEVER main agent
- Spawn sub-agent (use cheaper model like Sonnet) with explicit instructions (see [references/sub-agent-prompt.md](references/sub-agent-prompt.md))
- **Critical: After spawning, WAIT for the sub-agent to complete. Do NOT attempt parallel extraction.**
- Verify output file exists and has substantive content before proceeding
- Sub-agent session is disposable — its context gets thrown away after extraction, keeping your main context clean

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

**Source block (required at top of every review file):**

```markdown
## Source
- **Title:** [Content title]
- **Author:** [Creator name] (@handle if applicable)
- **URL:** [Clean URL — tracking params stripped]
- **Date:** [Publication date]
- **Length:** [Duration or word count]
- **Shared by:** [Who shared it] via [channel], [date]
- **Context:** [1-2 sentences: what was happening when the link was shared — what project, conversation, or train of thought prompted it. This is for future memory recall.]
```

**Filing rules by verdict:**

| Verdict | Destination | Retention |
|---|---|---|
| 📖 **Read** | `docs/content-claw/caught/{cat}--{src}--{slug}.md` | Permanent |
| 🎬 **Watch** | `docs/content-claw/caught/{cat}--{src}--{slug}.md` | Permanent |
| 👀 **Skim** | `docs/content-claw/released/skim/{cat}--{src}--{slug}.md` | Monthly batch review |
| ⏭️ **Skip** | `docs/content-claw/released/skip/{cat}--{src}--{slug}.md` | Auto-purge after 14 days |

- Naming: `{category}--{source-type}--{title-slug}.md` (see top of file for categories/sources)
- Create directories as needed
- Never overwrite existing files — append a number if slug already exists

**Review file contents:**
- Source block (as above)
- Verdict and reasoning
- Full extracted content or detailed summary
- Actions and insights (separated)

### Step 6: Released Folder Maintenance 🧹

**Auto-purge (agent-driven, no human input needed):**
- Files in `released/skip/` older than 14 days → delete automatically during heartbeats or periodic maintenance
- No confirmation needed. Skips were definitively not worth it.

**Monthly batch review (low-friction human decision):**
- Once per month, scan `released/skim/` for accumulated reviews
- Surface a numbered summary to the user:

> 🦞 **Content Claw — Monthly Purge**
> [N] skims from [month]. Promote or release?
>
> 1. "Article title" — one-line context reminder
> 2. "Article title" — one-line context reminder
> ...
>
> Reply with numbers to promote to `caught/`, or "clear all"

- Promoted files move to `docs/content-claw/caught/`
- Remaining files get deleted
- Track last purge date to avoid double-prompting

## Known Limitations

- **YouTube bot detection:** YouTube blocks yt-dlp from datacenter/cloud IPs. If you're running on a VPS or in a sandbox, yt-dlp will likely fail with 403 errors. Use web-based transcript extraction (Tier 1) instead.
- **Rate limiting:** web_search providers may rate-limit after repeated queries. Space out searches or reduce query count.
- **Long videos (>2hr):** Whisper transcription is CPU-intensive. For very long content, prefer searching for existing transcripts.

## Notes
- Keep verdicts concise — mobile-friendly formatting (no tables, use bullet lists)
- The user's time is the scarcest resource. Default to extracting value, not recommending consumption.
- When in doubt, extract the insights and skip. The bar for "watch the whole thing" should be high.
- This skill works for any agent — it adapts to whatever knowledge base and doc structure already exists.
