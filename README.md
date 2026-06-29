# youtube-transcript-plugin

A Claude Desktop / Cowork **plugin marketplace** that ships a single plugin (`youtube-transcript`) which teaches Claude how to **extract transcripts from YouTube videos** — from a full URL or a bare video ID, with or without timestamps.

This repo **does not install an MCP server.** The plugin is a Cowork skill plus a small Python script. When you ask Claude for a YouTube transcript, the skill runs the script, which pulls the video's captions via the [`youtube-transcript-api`](https://pypi.org/project/youtube-transcript-api/) library.

## Why this plugin

Out of the box, Claude can't reliably pull a YouTube transcript — it can't watch the video, and scraping the page rarely returns clean captions. This plugin gives Claude a deterministic path:

- **Recognizes** when you want a transcript (you paste a YouTube URL or ID and ask for captions/subtitles/transcript).
- **Extracts the video ID** from any common YouTube URL format, or accepts a raw 11-character ID.
- **Fetches official captions** — manually added if present, otherwise auto-generated.
- **Formats two ways** — plain text, or `[MM:SS] text` timestamped output (`[HH:MM:SS]` for longer videos).
- **Saves the result** to the filename you ask for, or falls back to `<video-id>-transcript.txt`.

## Prerequisites

The script uses [`uv`](https://docs.astral.sh/uv/) to manage its single dependency. `uv` must be available on the machine running Cowork. If it isn't installed:

```bash
# macOS
brew install uv
```

```bash
# Linux / macOS (standalone installer)
curl -LsSf https://astral.sh/uv/install.sh | sh
```

The script declares its dependency inline (PEP 723), so `uv run` installs `youtube-transcript-api` automatically on first use — no manual `pip install` needed.

## Repo layout

```
youtube-transcript-plugin/                  # repo root = a marketplace
├── .claude-plugin/
│   └── marketplace.json                    # marketplace manifest (lists 1 plugin)
├── README.md                               # you are here
├── LICENSE                                 # MIT
└── plugins/
    └── youtube-transcript/                 # the plugin itself
        ├── .claude-plugin/
        │   └── plugin.json                 # plugin manifest
        └── skills/
            └── youtube-transcript/
                ├── SKILL.md                # main skill
                └── scripts/
                    └── get_transcript.py   # uv-run caption fetcher (PEP 723)
```

The marketplace pattern means future contributors can add more plugins under `plugins/<name>/` and register them in `marketplace.json` — the install URL stays the same.

## Install

### Option 1 — Cowork "Add marketplace" (recommended)

1. Open Claude Desktop → **Customize** → **Marketplace** (or whatever your version labels the marketplace settings panel).
2. Click **+ Add marketplace** (sometimes labeled **Sync from URL**).
3. URL: `kugamon/youtube-transcript-plugin` (the GitHub `owner/repo` shorthand) — or paste the full URL `https://github.com/kugamon/youtube-transcript-plugin`.
4. Click **Sync**. The marketplace gets registered and you'll see one plugin: `youtube-transcript`.
5. Click **Install** on the `youtube-transcript` plugin.
6. Restart Claude (Cmd+Q + reopen on macOS) so the skill is loaded into the system prompt.

### Option 2 — Local folder

1. Clone the repo or download the source.
2. In Claude Desktop → **Customize** → **Personal plugins** → **+** → **Local folder**.
3. Pick `plugins/youtube-transcript/` (the directory that contains `.claude-plugin/plugin.json`).
4. Toggle on. Restart Claude.

### Option 3 — Manually pin to settings.json

For power users who want the marketplace registered globally:

```json
{
  "extraKnownMarketplaces": [
    { "url": "https://github.com/kugamon/youtube-transcript-plugin" }
  ],
  "enabledPlugins": ["youtube-transcript"]
}
```

## Verify

After installing and restarting, test the skill:

> "Get me the transcript of this YouTube video: https://youtu.be/dQw4w9WgXcQ"

Claude should load the `youtube-transcript` skill, run the script, and return the captions. For timestamps:

> "Give me the timestamped transcript of https://www.youtube.com/watch?v=dQw4w9WgXcQ"

## How to trigger it

Ask Claude things like:

- "Get me the transcript of this YouTube video: `<url>`"
- "Pull the captions from `youtu.be/dQw4w9WgXcQ`"
- "Give me the timestamped transcript of `<url>`"
- "Transcribe this and save it to `notes.txt`: `<url>`"

## Running the script directly

You don't need Claude to use the fetcher — it's a standalone script:

```bash
# Plain text
uv run plugins/youtube-transcript/skills/youtube-transcript/scripts/get_transcript.py "VIDEO_URL_OR_ID"

# With timestamps
uv run plugins/youtube-transcript/skills/youtube-transcript/scripts/get_transcript.py "VIDEO_URL_OR_ID" --timestamps
```

### Supported URL formats

- `https://www.youtube.com/watch?v=VIDEO_ID`
- `https://youtu.be/VIDEO_ID`
- `https://youtube.com/embed/VIDEO_ID`
- `https://youtube.com/v/VIDEO_ID`
- Raw 11-character video ID

### Output

- **Without `--timestamps`** (default): plain text, one line per caption segment.
- **With `--timestamps`**: `[MM:SS] text` (or `[HH:MM:SS]` for videos an hour or longer).

When Claude saves a plain-text transcript, it cleans the line breaks into readable paragraphs without altering the wording. The raw caption text itself is never modified.

## Troubleshooting

**`uv: command not found`.** Install `uv` (see [Prerequisites](#prerequisites)) and restart your terminal / Claude.

**"Could not extract video ID".** The URL didn't match a known format. Pass the bare 11-character ID, or a standard `watch?v=` / `youtu.be/` URL.

**"No transcripts were found" / errors fetching captions.** The video has captions disabled, or none are available in any language. Captions must be enabled (manual or auto-generated) for the API to return anything.

**Plugin loads but the skill doesn't trigger.** Restart Claude Desktop fully (Cmd+Q on macOS). Skills are loaded at session start. Make sure your message includes a YouTube URL or ID and asks for a transcript/captions/subtitles.

**"This repository isn't a marketplace — no manifest found."** Make sure you're on `main` and that `.claude-plugin/marketplace.json` exists at the repo root.

## Contributing

PRs welcome. Useful contributions:

- Language selection (let the user request a specific caption track).
- Output formats (SRT/VTT export, JSON with start/duration).
- More robust ID extraction (Shorts, live URLs, playlist-wrapped links).
- New plugins (drop a folder under `plugins/<name>/` and register it in `marketplace.json`).

## License

MIT — see [LICENSE](./LICENSE).

This project is **not affiliated with YouTube, Google LLC, or Anthropic.** It relies on the third-party `youtube-transcript-api` library; YouTube's terms and caption availability may change at any time.
