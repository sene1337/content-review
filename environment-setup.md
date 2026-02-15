# Environment Setup & Pre-Flight

Before extraction, verify your environment:

## Tool Availability
- `yt-dlp --version` → if missing, skip to web-based extraction
- `whisper --help` → if missing, skip to web-based extraction
- Verify `web_search` works (not rate-limited)

## Environment Constraints
- Datacenter IPs get blocked by YouTube (yt-dlp will fail with 403) — if a prior yt-dlp attempt returned 403/HTTP error, don't retry
- If running in a sandbox without local tools, go straight to Tier 1 (web-based) extraction

## Strategy Selection
- Tools available + residential IP → full pipeline (yt-dlp + whisper)
- Tools missing OR datacenter → web-first extraction (search for transcripts/summaries)
- No results from web → ask user for help

**If pre-flight fails, skip directly to the appropriate fallback tier. Do not attempt methods that will fail.**
