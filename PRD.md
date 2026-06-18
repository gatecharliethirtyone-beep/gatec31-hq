# Product Requirements Document
## GateCharlie31 Creator HQ (`gatec31-hq.html`)

**Version:** 1.0  
**Date:** 2026-06-18  
**Owner:** @gatec31  
**Status:** Shipped

---

## 1. Problem Statement

A flight sim YouTube creator at KDEN/Denver has no fast way to see which videos are working, which are dead, what titles/thumbnails perform in the niche, and when to publish next — without logging into YouTube Studio and manually combing through data. They need a single tool that reads the channel and tells them what to do next.

## 2. Goals

| Goal | Metric |
|------|--------|
| Surface real channel performance | Live data from YouTube Data API v3 |
| Rank every video fairly | Views per day (not raw views) |
| Identify patterns in winning vs losing content | Keyword frequency analysis across top/bottom quartile |
| Give actionable next steps | Title suggestions, thumbnail direction, publish schedule |
| Zero friction to open | One HTML file, no install, no login |

## 3. Non-Goals

- Private analytics (CTR, AVD, RPM) — requires OAuth, out of scope for v1
- Multi-channel support — scoped to @gatec31 only
- Video upload / scheduling — read-only tool
- Mobile-first layout — desktop/creator-at-desk use case

## 4. Users

Single user: the @gatec31 channel operator. Desktop browser, creator mindset, flight sim context (MSFS, KDEN, real-world×sim crossover content).

## 5. Features

### 5.1 Channel Connect
- Input: YouTube Data API v3 key
- On connect: fetch channel stats + all uploads (up to 200 videos, paginated)
- Persist key in `localStorage` so it survives page reload
- Error state: surface API error message clearly

### 5.2 Overview Tab
- Total subscribers, total views, video count
- Best day to publish (highest avg views/day by weekday)
- Channel health snapshot

### 5.3 Every Video Tab
- Full ranked list by views/day (descending)
- Per-video: title, thumbnail, publish date, views, likes, engagement rate, views/day, WIN/FLAT/DEAD flag
- WIN = top 25% by vpd, DEAD = bottom 25%, FLAT = middle 50%

### 5.4 Working / Not Tab
- Keyword frequency comparison: top performers vs bottom performers
- Niche word list scoped to flight sim / KDEN / MSFS context
- Best publish day with avg views/day by weekday

### 5.5 Titles & Thumbs Tab
- Content pillar breakdown: Destination Flights 40% / Tech & Setup 35% / Real World × Sim 25%
- Title formula templates per pillar
- Thumbnail direction per pillar (color, subject, text overlay guidance)
- CTA recommendations

### 5.6 Calendar Tab
- 4-week content plan
- Slots weighted by pillar percentages
- Publish day set to channel's best-performing weekday

## 6. Technical Constraints

- Single HTML file — no build tools, no npm, no server
- YouTube Data API v3 (public stats only, browser-side fetch)
- Chart.js via CDN (optional, for future charts)
- `localStorage` for API key persistence
- Must work by double-clicking the file in any modern browser

## 7. Design Constraints

- Cockpit-dark aesthetic matching @gatec31 brand strategy doc
- Fonts: Space Grotesk (headings), Space Mono (data/code), Syne (labels)
- Primary color: `#00A8E8` (radar blue)
- Background: `#0D1B2A` (cockpit dark)
- No emojis in data display — professional / HUD feel

## 8. Success Criteria

- Opens in browser with no errors
- Connects to @gatec31 channel and loads real data
- Every video appears ranked with correct vpd
- WIN/FLAT/DEAD flags match expected quartile logic
- Calendar shows 4 weeks of weighted content slots
