# Sub-Agent Spawn Template for Content Extraction

Use this template when spawning a sub-agent for content extraction. Customize the variables in brackets.

```
You must DO this work yourself — do NOT spawn sub-agents or just plan. Execute directly.

**Task:** Extract and document content from [CONTENT_TYPE]: [URL]
**Title:** [TITLE]
**Author:** [AUTHOR]

**Approach:**
[For YouTube:]
1. Use web_search for "[VIDEO_ID] transcript" to find existing transcripts
2. Try web_fetch on transcript services
3. If no transcript found, use yt-dlp + whisper (check for venv at /workspace first):
   - yt-dlp -x --audio-format wav -o "/tmp/audio.wav" "[URL]"
   - whisper /tmp/audio.wav --model base --output_format txt
4. Supplement with web_search for summaries and reviews of this specific content

[For Articles:]
1. Use web_fetch with markdown extraction on the URL
2. For X/Twitter: use api.fxtwitter.com/[user]/status/[id]

[For Tweet Threads:]
1. Use api.fxtwitter.com to get each tweet in the thread
2. Follow reply chains

**Output:** Write to /workspace/docs/reviews/[SLUG].md

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

**Definition of Done:** File exists at the specified path with substantive extracted content.
```

## Content Type Detection

| URL Pattern | Type | Method |
|---|---|---|
| youtube.com/watch, youtu.be/ | YouTube video | Transcription pipeline |
| x.com, twitter.com (with /status/) | Tweet or X Article | fxtwitter API |
| *.substack.com, medium.com | Article | web_fetch markdown |
| open.spotify.com, podcasts.apple.com | Podcast | Search for transcript |
| Everything else | Article | web_fetch markdown |
