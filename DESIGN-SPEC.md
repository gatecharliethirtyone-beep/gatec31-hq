# Design Specification
## GateCharlie31 Creator HQ

**Version:** 1.0  
**Date:** 2026-06-18

---

## 1. Design Language

### Philosophy
HUD / cockpit instrument panel. Data is the UI. Every element should feel like it belongs in an aircraft or ATC system — not a marketing dashboard. No gradients for decoration. No rounded corners for softness. Precision over polish.

### Reference
`gatec31youtubestrategy_4.html` — the existing @gatec31 strategy doc. All new work must match or extend this aesthetic.

---

## 2. Color Tokens

```css
--cockpit:    #0D1B2A   /* page background — deep navy */
--panel:      #1A2D42   /* card/panel background */
--surface:    #243B55   /* elevated surface, table rows */
--radar:      #00A8E8   /* primary accent — electric blue */
--radar-dim:  #0077A8   /* hover/active state */
--win:        #00C896   /* WIN flag — green */
--dead:       #E84855   /* DEAD flag — red */
--flat:       #888EA8   /* FLAT flag — muted */
--text:       #E8EDF2   /* primary text */
--text-dim:   #8899AA   /* secondary/label text */
--border:     #2A3F55   /* panel borders */
```

---

## 3. Typography

| Use | Font | Weight | Size |
|-----|------|--------|------|
| Page title / tab labels | Space Grotesk | 600 | 14px |
| Section headings | Space Grotesk | 700 | 18–24px |
| Body / descriptions | Space Grotesk | 400 | 14px |
| Data values / numbers | Space Mono | 400 | 13–16px |
| Badges / flags / tags | Syne | 700 | 11px |

All fonts loaded from Google Fonts CDN. No fallback web fonts — if CDN fails, system monospace is acceptable.

---

## 4. Layout

### Shell
```
┌─────────────────────────────────────────┐
│  NAV: Logo + 5 tabs                     │
├─────────────────────────────────────────┤
│  HERO: API key input + CONNECT button   │
│        (hides after successful connect) │
├─────────────────────────────────────────┤
│  CONTENT AREA: active tab panel         │
│                                         │
│                                         │
└─────────────────────────────────────────┘
```

- Max width: 1200px, centered
- Padding: 24px horizontal
- Nav: sticky top, `--cockpit` background, bottom border `--border`

### Cards / Panels
- Background: `--panel`
- Border: 1px solid `--border`
- Border-radius: 4px (minimal — not soft)
- Padding: 20px
- No box-shadow — flat depth via color only

### Stat Cards (Overview tab)
- 3-column grid on desktop
- Large number: Space Mono 32px `--radar`
- Label: Space Grotesk 12px `--text-dim` uppercase tracked

---

## 5. Components

### WIN / FLAT / DEAD Badge
```
WIN  → background: rgba(0,200,150,0.15), color: #00C896, border: 1px solid #00C896
FLAT → background: rgba(136,142,168,0.10), color: #888EA8, border: 1px solid #888EA8
DEAD → background: rgba(232,72,85,0.15), color: #E84855, border: 1px solid #E84855
```
Font: Syne 700 11px, uppercase, letter-spacing 1px, padding 2px 6px, border-radius 2px.

### Video Row (Every Video tab)
```
[ RANK ]  [ THUMB ]  [ TITLE + DATE ]  [ VIEWS ]  [ VPD ]  [ ENG% ]  [ BADGE ]
```
- Alternating row background: `--panel` / `--surface`
- Hover: `--surface` + left border 2px `--radar`
- Thumbnail: 80×45px, object-fit cover

### Tab Navigation
- Inactive: `--text-dim`, no underline
- Active: `--radar`, 2px bottom border `--radar`
- Hover: `--text`
- Font: Space Grotesk 600 13px uppercase

### Connect Button
- Background: `--radar`
- Color: `--cockpit`
- Font: Space Grotesk 700 14px uppercase
- Padding: 10px 28px
- Border-radius: 2px
- Hover: `--radar-dim`
- No border

### API Key Input
- Background: `--surface`
- Border: 1px solid `--border`
- Focus border: `--radar`
- Color: `--text`
- Font: Space Mono 13px
- Placeholder color: `--text-dim`

---

## 6. States

| State | Treatment |
|-------|-----------|
| Loading | "LOADING..." text in `--radar`, Space Mono |
| Error | Red panel with error message, `--dead` color |
| Empty (no API key) | Hero section visible, tabs disabled |
| Connected | Hero collapses, Overview tab auto-selected |
| No data | "No data available" placeholder in `--text-dim` |

---

## 7. Responsive

- Desktop (>900px): full layout as designed
- Tablet (600–900px): stat cards 2-column, video table horizontally scrollable
- Mobile (<600px): not a design target for v1, but must not break catastrophically

---

## 8. Motion

Minimal. No animations for decoration.
- Tab switch: instant
- Data load: no spinner, text state change only
- Hover transitions: `transition: 150ms ease` on color/border only
