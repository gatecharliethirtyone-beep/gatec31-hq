# Implementation Guide
## GateCharlie31 Creator HQ (`gatec31-hq.html`)

**Version:** 1.0  
**Date:** 2026-06-18

---

## 1. Architecture

### Single-File Approach

Everything lives in `gatec31-hq.html`:

```
gatec31-hq.html
├── <style>          CSS with design tokens
├── <body>           HTML shell (nav, hero, tab panels)
└── <script>         App IIFE module
    ├── Config       API base URL, niche word list, content pillars
    ├── State        key, channelId, videos[], raw channel stats
    ├── API layer    api(path) → fetch + JSON
    ├── Load flow    resolveId() → load() → render*()
    └── Renderers    renderOverview, renderBreakdown, renderWorking,
                     renderSuggest, renderCalendar
```

No build step. No dependencies except Google Fonts (CDN) and Chart.js (CDN, optional).

---

## 2. YouTube Data API v3 — Key Concepts

### Getting a Free API Key

1. Go to [console.cloud.google.com](https://console.cloud.google.com/)
2. Create a new project (name it anything)
3. Navigate to **APIs & Services → Library**
4. Search "YouTube Data API v3" → Enable
5. Go to **APIs & Services → Credentials**
6. Click **Create Credentials → API Key**
7. Copy the key — paste into the app

**Quota:** 10,000 units/day free. Each `videos.list` call costs 1 unit per video. A full 200-video load costs ~6 units. Well within free tier.

**Restriction (optional but recommended):** In the API key settings, restrict to HTTP referrers `localhost` and `file://` to prevent misuse if the key leaks.

### API Calls Made by the App

```
GET channels?part=statistics,contentDetails&forHandle=@gatec31
    → subscribers, total views, uploads playlist ID

GET playlistItems?part=contentDetails&playlistId={uploadsId}&maxResults=50
    → video IDs (paginated, up to 200)

GET videos?part=snippet,statistics&id={id1,id2,...id50}
    → per-video: title, publishedAt, thumbnail, views, likes, comments
```

All calls are read-only public endpoints. No OAuth required.

---

## 3. Core Data Structures

### Video Object
```javascript
{
  id:        "dQw4w9WgXcQ",
  title:     "Flying into KDEN — Real World ILS Approach | MSFS 2024",
  published: "2024-11-15T18:00:00Z",
  day:       "Fri",
  thumb:     "https://i.ytimg.com/vi/.../mqdefault.jpg",
  views:     14200,
  likes:     312,
  comments:  44,
  age:       215,          // days since publish
  vpd:       66.05,        // views per day
  eng:       2.51          // (likes+comments)/views * 100
}
```

### Performance Flags
```javascript
function flagOf(v, q1, q3) {
  if (v.vpd >= q3) return 'win';
  if (v.vpd <= q1) return 'dead';
  return 'flat';
}
// q1 = 25th percentile vpd, q3 = 75th percentile vpd
```

---

## 4. Load Flow

```
User pastes API key + clicks CONNECT
  │
  ▼
resolveId(handle)
  → GET channels?forHandle=@gatec31
  → returns { channelId, uploadsPlaylistId, stats }
  │
  ▼
load()
  → paginated GET playlistItems (50 at a time, up to 200 videos)
  → batch GET videos?id=... (50 at a time)
  → build videos[] array with computed vpd, eng, age
  → compute quartiles (q1, q3)
  → flag each video win/flat/dead
  │
  ▼
renderOverview()   ← auto-shown on connect
```

---

## 5. Niche Word List

Used by `renderWorking()` to compare keyword frequency in top vs bottom quartile titles:

```javascript
const NICHE_WORDS = [
  'kden','denver','approach','landing','takeoff','ils','vfr','ifr',
  'msfs','msfs2024','2024','real','world','sim','flight','tutorial',
  'cockpit','atc','runway','gate','charlie','737','a320','cessna',
  'vatsim','ivao','review','setup','hardware','joystick','throttle'
];
```

To add words: edit the `NICHE_WORDS` array in the `<script>` section.

---

## 6. Content Pillars

```javascript
const PILLARS = [
  { name: 'Destination Flights', weight: 0.40,
    titles: [
      'Flying into {AIRPORT} — Real World {APPROACH} | MSFS 2024',
      '{AIRPORT} to {AIRPORT}: The Route You\'ve Never Seen in a Sim',
      'What ATC Actually Says at {AIRPORT} | Full Flight'
    ],
    thumb: 'Cockpit shot at golden hour. Airport identifier in large white text. Blue radar accent stripe.',
    cta:   'Ask viewers: "What airport should I fly next?"'
  },
  { name: 'Tech & Setup', weight: 0.35,
    titles: [
      'My KDEN Home Cockpit Setup — Every Piece of Hardware Explained',
      'Why I Switched to {HARDWARE} — Honest Review After 6 Months',
      'The One MSFS Setting That Changed Everything'
    ],
    thumb: 'Flat lay or close-up of hardware. Clean background. Brand colors visible.',
    cta:   'Link hardware in description. Ask: "What\'s in your setup?"'
  },
  { name: 'Real World × Sim', weight: 0.25,
    titles: [
      'I Flew the Same Route as a Real Pilot — Here\'s What the Sim Gets Wrong',
      'Real KDEN ATC Recording vs MSFS — How Close Is It?',
      'What Real Pilots Think of MSFS 2024'
    ],
    thumb: 'Split screen: real vs sim. Or real pilot photo side by side with cockpit screenshot.',
    cta:   'Tag real pilots or airlines. Cross-post to aviation communities.'
  }
];
```

---

## 7. Calendar Logic

```javascript
// 4 weeks × 2 posts/week = 8 slots
// Weighted by pillar: 40/35/25 → ~3 Destination, 3 Tech, 2 RealWorld
// Publish day: set to channel's best-performing weekday (computed from data)

const weeks = 4;
const postsPerWeek = 2;
// Assign pillar by weight using a shuffle-weighted array
```

---

## 8. Extending the App

### Add a new tab
1. Add a `<button class="tab-btn" data-tab="newtab">Label</button>` in the nav
2. Add a `<div id="tab-newtab" class="tab-panel">` in the content area
3. Add a `renderNewTab()` function in the script
4. Add `case 'newtab': renderNewTab(); break;` in the `switchTab()` function

### Add private analytics (future)
Requires OAuth 2.0 flow. Cannot be done in a pure static file without a backend redirect URI. Options:
- Add a lightweight Netlify/Vercel serverless function as OAuth callback
- Use Google OAuth Playground to generate a short-lived token manually

### Add Shorts detection
YouTube Shorts have duration ≤ 60 seconds. The current API calls don't fetch `contentDetails.duration`. To add Shorts detection:
```javascript
// Add contentDetails to the videos?part= call
const vj = await api(`videos?part=snippet,statistics,contentDetails&id=${chunk}`);
// Then check:
const isShort = parseDuration(v.contentDetails.duration) <= 60;
```

---

## 9. Troubleshooting

| Issue | Fix |
|-------|-----|
| "API key invalid" | Double-check key in Google Cloud Console. Make sure YouTube Data API v3 is enabled for that project. |
| No videos load | Check browser console for CORS or quota errors |
| Quota exceeded | Free quota resets at midnight Pacific. 10k units/day. Full load uses ~6 units. |
| Channel not found | App uses `forHandle=@gatec31`. If handle changes, update `resolveId()` call |
| Stale data | Click CONNECT again to re-fetch. No auto-refresh — YouTube API has quota limits. |

---

## 10. File Delivery

The only file end users (you) need is `gatec31-hq.html`. Everything else in this repo is documentation.

```
gatec31-hq.html     ← the app. download this.
README.md           ← quick start
PRD.md              ← product requirements
DESIGN-SPEC.md      ← visual design reference
IMPLEMENTATION.md   ← this file
```
