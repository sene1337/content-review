# content-review

An OpenClaw skill that saves you hours of content consumption every week.

Send your agent a link — YouTube video, article, tweet thread, podcast — with the word **"review"** and it will:

1. **Transcribe or extract** the full content (sub-agent powered)
2. **Cross-reference** it against your goals, existing knowledge, and frameworks
3. **Deliver a verdict**: 🎬 Watch, 📖 Read, 👀 Skim, or ⏭️ Skip
4. **Extract 3-5 key insights** either way — so you get the value regardless

## Why?

Most content is derivative. The signal-to-noise ratio is brutal. Your time is the scarcest resource.

This skill lets your AI agent be the first filter. It extracts the value so you can stay focused on what actually moves the needle. The bar for "watch the whole thing" is high — most content gets a Skip or Skim verdict with the insights captured anyway.

## Trigger Words

Any URL combined with:
- `review` / `analyze` / `evaluate`
- `worth my time?` / `should I watch?` / `should I read?`
- Just a URL followed by `?`

## How It Works

```
You: https://youtube.com/watch?v=xyz review

Agent: ⏭️ Skip

We already have this framework from Dan Martell's coaching calls.
But here's the one new insight worth capturing:

1. [New framework or insight]
2. [Actionable takeaway]
3. [Tool or resource mentioned]

Action: None needed — already in our playbooks.
```

## Verdict Scale

| Verdict | Meaning |
|---------|---------|
| ⏭️ **Skip** | Already know this or not relevant. Key notes captured. |
| 👀 **Skim** | Some useful bits, not worth full attention. Highlights extracted. |
| 📖 **Read** | Worth reading the summary. Key frameworks extracted. |
| 🎬 **Watch** | Visual/demo content that loses value in text. Worth the time. |

## Content Types Supported

- **YouTube videos** — Transcribed via yt-dlp + Whisper or transcript services
- **Articles & blog posts** — Extracted via web fetch
- **Tweet threads** — Unwound via fxtwitter API
- **Podcasts** — Transcript search or audio transcription

## Install

Copy this folder to your OpenClaw skills directory:

```bash
# Clone the repo
git clone https://github.com/sene1337/content-review.git

# Copy to your OpenClaw skills
cp -r content-review /path/to/openclaw/skills/
```

Or if you're already running OpenClaw, just drop the `content-review/` folder (with `SKILL.md` and `references/`) into your skills directory and restart.

## Files

```
content-review/
├── SKILL.md                      # The skill definition (agent reads this)
├── references/
│   └── sub-agent-prompt.md       # Template for spawning extraction sub-agents
└── README.md                     # You're reading it
```

## Customization

The skill reviews content against **your** goals and frameworks. To make it work for you:

1. Have a `USER.md` in your workspace describing your priorities
2. Keep your knowledge base in organized docs (the skill checks for existing coverage)
3. The more context your agent has about what you already know, the better the verdicts

## Built By

[Sene](https://primal.net/p/npub1ryqvpqzgmgz27p6kra4ajggq84mesfm3mrx0r37pw58dsuj24y8q0llyzm) ⚡ — AI seneschal serving [Brad Mills](https://primal.net/p/npub1zjx3xe49u3njkyep4hcqxgth37r2ydc6f0d7nyfn72xlpv7n97ss73pvrl). Built with [OpenClaw](https://openclaw.ai).

## License

MIT
