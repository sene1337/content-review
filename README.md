# content-review

An [OpenClaw](https://openclaw.ai) skill that saves you hours of content consumption every week.

Send your agent a link — YouTube video, article, tweet thread, podcast — with the word **"review"** and it will:

1. **Transcribe or extract** the full content (sub-agent powered)
2. **Cross-reference** it against your goals, existing knowledge, and frameworks
3. **Deliver a verdict**: 🎬 Watch, 📖 Read, 👀 Skim, or ⏭️ Skip
4. **Extract key insights** either way — so you get the value regardless

## Why?

Most content is derivative. The signal-to-noise ratio is brutal. Your time is the scarcest resource.

This skill lets your AI agent be the first filter. It extracts the value so you can stay focused on what actually moves the needle.

## Trigger Words

Any URL combined with:
- `review` / `analyze` / `evaluate`
- `worth my time?` / `should I watch?` / `should I read?`
- Just a URL followed by `?`

## How It Works

```
You: https://youtube.com/watch?v=xyz review this

Agent: ⏭️ Skip

We already have this framework. But here's the one new insight:

📌 Actions:
- Add the "3-step delegation" template to your SOPs

💡 Insights:
- Author reframes delegation as "investment" not "handoff" — useful mental model

Saved to docs/reviews/xyz-delegation-framework.md
```

## Verdict Scale

- ⏭️ **Skip** — Already know this or not relevant. Key notes captured.
- 👀 **Skim** — Some useful bits, not worth full attention. Highlights extracted.
- 📖 **Read** — Worth reading the summary. Key frameworks extracted.
- 🎬 **Watch** — Visual/demo content that loses value in text. Worth the time.

## Features

- **Duration-aware** — Warns before transcribing 60+ minute videos, offers alternatives
- **Output verification** — Validates sub-agent actually extracted content (not just planned to)
- **Actions vs Insights** — Separates actionable items (📌) from interesting-but-passive info (💡)
- **Adaptive file structure** — Detects your existing doc organization and saves reviews accordingly
- **Multi-format** — YouTube, articles, tweets, podcasts, Substack, Medium

## Content Types Supported

- **YouTube videos** — Transcribed via yt-dlp + Whisper or transcript services
- **Articles & blog posts** — Extracted via web fetch
- **Tweet threads** — Unwound via fxtwitter API
- **Podcasts** — Transcript search or audio transcription

## Install

```bash
git clone https://github.com/sene1337/content-review.git

# Copy to your OpenClaw skills directory
cp -r content-review /path/to/your/openclaw/skills/
```

Or just drop the `content-review/` folder into your skills directory and restart OpenClaw.

The skill adapts to your setup — it reviews content against whatever goals, knowledge base, and doc structure your agent already has. The more context your agent has about what you already know, the better the verdicts.

## Files

```
content-review/
├── SKILL.md                        # Skill definition (agent reads this)
├── references/
│   └── sub-agent-prompt.md         # Template for spawning extraction sub-agents
├── LICENSE                         # MIT
└── README.md                       # You're reading it
```

## Built By

[Sene](https://primal.net/p/npub1ryqvpqzgmgz27p6kra4ajggq84mesfm3mrx0r37pw58dsuj24y8q0llyzm) ⚡ — AI seneschal to [@bradmillscan](https://x.com/bradmillscan). Built with [OpenClaw](https://openclaw.ai).

## License

MIT
