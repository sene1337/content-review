# Sub-Agent Spawn Template for Content Extraction

Use this template when spawning a sub-agent for content extraction. Customize the variables in brackets.

## Important Rules
1. **Use Absolute Paths** — Always use the full absolute path to your workspace (e.g., `/Users/yourname/.openclaw/workspace/`), NOT relative paths like `/workspace/`.
2. **After spawning: WAIT** — Do NOT attempt parallel extraction while the sub-agent runs. Wait for completion, then verify output.
3. **Only spawn if Tier 1 failed** — Web-based extraction should be attempted first. Only spawn a sub-agent if lightweight methods didn't produce enough content.

## Template

```
You must DO this work yourself — do NOT spawn sub-agents or just plan. Execute directly.

**Task:** Extract and document content from [CONTENT_TYPE]: [URL]
**Title:** [TITLE]
**Author:** [AUTHOR]

**Step 0 — Pre-Flight (do this FIRST, before any extraction):**
1. Check: `which yt-dlp` — if not found, SKIP to web-only extraction
2. Check: `which whisper` — if not found, SKIP to web-only extraction
3. If both exist, proceed with full pipeline

**Approach:**
[For YouTube:]
1. Use web_search for "[VIDEO_ID] transcript" to find existing transcripts
2. Try web_fetch on transcript services (e.g., kome.ai)
3. ONLY if no transcript found AND yt-dlp exists: yt-dlp -x --audio-format wav -o "/tmp/audio.wav" "[URL]"
   - If yt-dlp fails (403, blocked, any error): STOP trying yt-dlp. Move to step 4.
   - If yt-dlp succeeds: whisper /tmp/audio.wav --model base --output_format txt
4. Supplement with web_search for summaries and reviews of this specific content

[For Articles:]
1. Use web_fetch with markdown extraction on the URL
2. For X/Twitter: use api.fxtwitter.com/[user]/status/[id]

[For Tweet Threads:]
1. Use api.fxtwitter.com to get each tweet in the thread
2. Follow reply chains

**Retry Limits (HARD — do not exceed):**
- web_search: max 2 attempts
- web_fetch: max 1 per URL
- yt-dlp: max 1 attempt total (if it fails, it fails)
- If total failed attempts > 5: STOP and write what you have

**Output:** Write to [ABSOLUTE_WORKSPACE_PATH]/docs/reviews/[SLUG].md

Create the docs/reviews/ directory if it doesn't exist.

**Document Structure:**
- Source info (title, author, URL, date, length)
- Full transcript or detailed content extraction
- Key frameworks/insights identified (numbered list)
- Notable quotes
- Tools, resources, or links mentioned

**Rules:**
- Write to the file as you go — don't wait until the end
- Do NOT report complete without creating the output file
- Do NOT just describe what you would do — actually do it
- Include actual content, not summaries of what the content "probably" covers
- Use absolute paths for ALL file operations
- If a method fails once, do NOT retry the same method — move to the next approach

**Definition of Done:** File exists at the specified path with substantive extracted content (not an outline or plan).
```

## Content Type Detection

| URL Pattern | Type | Method |
|---|---|---|
| youtube.com/watch, youtu.be/ | YouTube video | Transcription pipeline |
| x.com, twitter.com (with /status/) | Tweet or X Article | fxtwitter API |
| *.substack.com, medium.com | Article | web_fetch markdown |
| open.spotify.com, podcasts.apple.com | Podcast | Search for transcript |
| Everything else | Article | web_fetch markdown |
