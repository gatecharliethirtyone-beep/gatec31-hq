# GateCharlie31 Creator HQ

Single-file YouTube channel intelligence tool for **@gatec31** — flight sim / KDEN / MSFS.

## How to use

1. Download `fable5.html`
2. Open it in any browser (double-click, or drag into Chrome/Firefox)
3. Get a free YouTube Data API v3 key from [Google Cloud Console](https://console.cloud.google.com/)
   - Create a project → Enable "YouTube Data API v3" → Credentials → Create API Key
4. Paste the key into the app and click **CONNECT**

The app pulls live public data from @gatec31 and shows:

| Tab | What it does |
|-----|-------------|
| **Overview** | Subscribers, total views, upload count, best publish day |
| **Every Video** | All uploads ranked by views/day with WIN / FLAT / DEAD flags |
| **Working / Not** | Keyword patterns from top vs bottom performers |
| **Titles & Thumbs** | Niche-specific suggestions by content pillar |
| **Calendar** | 4-week weighted content plan |

## No backend, no login, no build tools

Pure HTML + CSS + JS. Everything runs in the browser. Your API key is stored only in `localStorage` on your own machine — nothing is sent anywhere except YouTube's public API.

## What's public vs private

YouTube's public API gives you views, likes, comments, and publish dates for any channel. **CTR, Average View Duration, and revenue are private** — those require OAuth and YouTube Studio. This tool works entirely on public stats.
