# The Seltzer Review — Project Document

## Overview

**Site:** theseltzerreview.com  
**Purpose:** Matt's personal hub for ranking and reviewing 500+ spiked seltzers, tied to his YouTube channel (defnot_matty). Tagline: "The Defnotfinitive Spiked Seltzer Review."  
**Deployment:** GitHub Pages — pushing to `main` branch deploys live automatically.  
**Analytics:** Google Analytics G-49X33MQJR3  
**Repository:** `git remote` → GitHub (main branch = production)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Pages | Static HTML — no framework, no CMS |
| Styles | Tailwind CDN + inline `<style>` blocks |
| Fonts | Google Fonts: Fredoka + Nunito |
| Data source | Excel (.xlsx) spreadsheet |
| Build | Node.js scripts (`.mjs`) |
| Image processing | `sharp` (Node.js) |
| YouTube API | YouTube Data API v3 + OAuth 2.0 |
| Local dev server | `node serve.mjs` → http://localhost:3000 |
| Screenshots | `node screenshot.mjs http://localhost:3000` |

---

## Brand System

### Colors
```
--primary:    #FF4757   (red — CTAs, accents, eyebrows, active states)
--secondary:  #00D4B4   (teal)
--tertiary:   #FFD60A   (yellow)
--quaternary: #9B5DE5   (purple)

Background tints:
--bg-coral:   #FFF0F1
--bg-teal:    #E8FFFC
--bg-yellow:  #FFFBE0
--bg-purple:  #F3EEFF
--bg-neutral: #FFF8F5

Dark theme (reviews.html only):
--bg:        #0f0f0f
--surface:   #1a1a1a
--surface2:  #222
--border:    #2e2e2e
--text:      #f0f0f0
--muted:     #888
```

### Fonts
- **Fredoka** (400–700): headings, logo, section titles, drop caps
- **Nunito** (300/400/500/600/700, italic): body, nav, bylines, tags
- Never add other fonts — these two are the complete type system.

### Reference Design
The homepage is modeled after **Cookie & Kate** (cookieandkate.com) — magazine-style food blog with image-heavy card grids, warm white backgrounds, and editorial layout.

---

## Pages

### `index.html` — Homepage
Light/white theme. Section order:
1. Sticky header/nav
2. Video hero (YouTube embed, currently `qwWOEjmDi2k`)
3. Hero section (site intro)
4. Featured Post (currently: Olé Chili Mango, rank #49 → `reviews.html?open=49`)
5. "Big 4" featured seltzers grid
6. About + Top 5 sidebar
7. New Videos (3 YouTube iframes: `TAD0DrhxKoc`, `-skYvjaz64U`, `wMZ_S19UURQ`)
8. Blog section
9. Socials section
10. Discord CTA / Contact
11. Footer

**Current Top 5:**
1. Montauk Pink Lemonade
2. White Claw Black Cherry
3. Kirkland Mango
4. Montauk Original Lemonade
5. Truly Cherry Pop

**⚠️ index.html is manually edited** — no build script. Update these items by hand.

---

### `reviews.html` — Full Rankings
**Dark theme** (#0f0f0f background). 500+ seltzers. **This file is generated — never edit directly.**

**Build command:** `node build-reviews.mjs`

#### Table Columns (9 total)
| Column | Contents |
|---|---|
| # | Rank number, or styled "N/A" grey badge for unranked |
| (image) | 58×58px can thumbnail with hover "+" lightbox button |
| Brand & Flavor | Brand (Fredoka), flavor (muted), alcohol type badge (color-coded) |
| Stats | ABV / Sugar / Calories |
| Review Notes | 3-line clamped with "Read more ↓" toggle |
| Beach | Beach count (teal color) |
| Terroir | Where consumed (📍 icon) |
| Date | Review date |
| Video Review | Red "▶ Watch" button or "—" |

**Mobile:** Hides Stats, Review Notes, Beach, Terroir, Date, Video columns. Shows 3-column list: Rank + Image + Brand/Flavor.

#### Interactive Features

**1. Search**
- Input filters rows client-side
- Searches `row.dataset.row` (embedded JSON — covers brand, flavor, notes, terroir, type)
- Live result count updates

**2. Filter Buttons**
- One button per alcohol type + "All"
- Each type has branded color (FMB=red, Vodka=teal, Tequila=yellow, Gin=purple, Whiskey/Scotch=orange, Kombucha=jade, Rum=hot pink, Bourbon=warm brown, Wine=grape purple, Mezcal=smoky gold)
- Filter + search use AND logic

**3. Row Click → Detail Modal**
Clicking a row (except links/buttons) opens `#rowModal`:
- Header: 120×120px can image + rank ("Unranked" for N/A) + brand + flavor + type badge
- Stats row: ABV · Sugar · Calories · Beach
- Full untruncated review notes
- Footer: 📍 terroir + 🗓 date (left) | "▶ Watch Review" link (right, if video exists)
- Action buttons: "+ More Pictures" and "+ More Videos" (UI only, no functionality yet)
- Close: ✕, click backdrop, or Escape key

**4. Can Image → Lightbox**
- Hover reveals "+" expand button
- Click opens full-size lightbox overlay
- Does NOT also trigger the row modal (stopPropagation)

**5. Notes Expand/Collapse**
- "Read more ↓" toggles full text / "Read less ↑"
- Does NOT trigger row modal (stopPropagation)

**6. URL Parameters**
- `?filter=Vodka` — activates filter on load
- `?open=105` — opens detail modal for rank 105 on load
- `?filter=FMB&open=42` — both together
- N/A items: `?open=N/A`

**7. Mobile Hamburger Nav**
- `#revHamburger` button toggles `.nav-open` on `.nav-links`
- Clicking outside nav closes it

---

### `blog.html` — Blog Template
Editorial long-form article format. Structure:
1. Same sticky header/nav as index.html
2. Dark hero — "The Pen is Mightier" with red accent + radial gradient
3. Article header: eyebrow + Playfair Display headline + italic dek + meta bar
4. Full-width hero image
5. Article body (max 720px centered):
   - Drop cap first paragraph
   - Pull quote (red left border)
   - Inline images, two-up image pairs, YouTube embeds, H2 headers, decorative dividers
6. Author byline (avatar, links: Twitch, Discord, Full Rankings)
7. Discord CTA
8. Footer

**Blog posts** live in `Blog Posts/[date]/index.html` — e.g. `Blog Posts/05-18-2026/index.html`

---

## Data Pipeline

### Source of Truth
`Defnotfinitive Hard Seltzer Rankings .xlsx` (project root)

**Excel column mapping:**
| Col | Field |
|---|---|
| A | Rank (number, or "N/A") |
| B | Brand |
| C | Flavor |
| D | Alcohol Type (FMB, Vodka, Tequila, Gin, etc.) |
| F | ABV (decimal — 0.05 = 5%) |
| G | Sugar |
| H | Calories |
| I | Price |
| J | Terroir |
| K | Date (Excel serial format) |
| L | Review notes |
| M | Beach count |

### Supporting JSON Files (in `seltzer-images/`)
- `rank-map.json` — rank → image file path
- `video-map.json` — rank → YouTube URL

---

## Update Workflows

### Workflow 1: Add New Seltzers / Update Review Data

1. Edit `Defnotfinitive Hard Seltzer Rankings .xlsx`
2. If new can photos: `node extract-images.mjs` (rebuilds `rank-map.json`)
3. If new videos: `node build-video-map.mjs` (rebuilds `video-map.json`)
4. `node build-reviews.mjs` (generates fresh `reviews.html`)
5. Commit and push to main

**Standard prompt:**
> "I've updated the Excel spreadsheet with new seltzer reviews. Please rebuild the reviews page — run `node build-reviews.mjs` and verify the output looks correct."

---

### ⚠️ Build Script Divergence Warning

`reviews.html` currently has 6 features NOT in `build-reviews.mjs`. Running the build script without updating it first will **wipe these features:**

1. Mobile hamburger nav + `.nav-open` toggle JS
2. Mobile responsive CSS (hidden columns, bottom-sheet modal, 2-col stats grid)
3. `rm-actions` section ("More Pictures" / "More Videos" buttons) with `.rm-action-btn` styles
4. N/A rank handling (styled badge in table, "Unranked" text in modal, string comparison in `?open=` param)
5. Search using `row.dataset.row` instead of `row.textContent`
6. Notes toggle on `.notes-toggle` element (with stopPropagation) instead of whole `.td-notes` cell

**Before running `node build-reviews.mjs`:** Update the build script to include all 6 of the above, then verify the output contains them.

---

### Workflow 2: Update YouTube Video Map

Run when new review videos have been posted:

```
node build-video-map.mjs
```

- Fetches all videos from playlist `PLry3VfdA7_wnQlWCwskdSb6_dd1Wr4MQp`
- Fuzzy-matches titles ("Seltzer Review: Brand - Flavor") to Excel rows
- Writes `seltzer-images/video-map.json`
- Then rebuild: `node build-reviews.mjs`

---

### Workflow 3: Update Latest Video Thumbnail

```
node update_thumbnail.mjs
```

Full pipeline:
1. Refreshes OAuth access token (reads from `oauth_tokens.json`)
2. Gets most recent video from Spiked Seltzer Reviews playlist
3. Checks for saved original at `temporary screenshots/thumbnail_original_{videoId}.jpg` — uses it if found, otherwise downloads from YouTube and saves it
4. Composites brand overlay (SPIKED top-left, SELTZER top-right, REVIEW large at bottom)
5. Uploads composited image as new custom thumbnail
6. Saves composite to `temporary screenshots/thumbnail_composited_{videoId}.jpg`

**Critical rule:** NEVER composite on a downloaded YouTube thumbnail after the first upload — YouTube serves the custom thumbnail from all CDN URLs. Always use the saved original as the base.

**If saved original is corrupted/missing** — extract clean frame from video stream:
```
C:\ytdl\yt-dlp --download-sections "*0:00-0:03" -f "bestvideo[height<=720][ext=mp4]" -o "clip.%(ext)s" [VIDEO_URL]
C:\ytdl\ffmpeg -i clip.mp4 -ss 00:00:01 -vframes 1 -q:v 2 -update 1 original.jpg
```

**OAuth reauth** (if token expires):
```
node reauth_youtube.mjs
```
Opens browser → authorize as `defnotmatty@gmail.com` → writes new tokens to `oauth_tokens.json`.

---

### Workflow 4: Update Homepage Manually

`index.html` is not generated — edit it by hand. Common updates:

- **Featured seltzer:** Update the featured post section (name, rank, `?open=` link)
- **New Videos:** Swap the 3 YouTube iframe `src` embed IDs
- **Top 5:** Update the 5 ranked seltzers in the About sidebar
- **Video hero:** Swap the hero iframe embed ID

**Standard prompt:**
> "Please update the homepage. The new featured seltzer is [Brand] [Flavor], rank #[N]. The three latest video IDs are [ID1], [ID2], [ID3]. The new Top 5 are: 1. [X], 2. [X], 3. [X], 4. [X], 5. [X]."

---

### Workflow 5: Add New Blog Post

1. Create `Blog Posts/[MM-DD-YYYY]/index.html`
2. Use `blog.html` as the template
3. Replace placeholder content (headline, dek, author meta, body paragraphs, images, videos)
4. Update the blog grid on `index.html` to link to the new post
5. Commit and push

---

### Workflow 6: Add Can Images

When new can photos are available:

```
node extract-images.mjs
```

- Reads `xl/media/` from the Excel file's internal zip structure
- Maps row positions to rank numbers
- Saves images to `seltzer-images/all/{rank}-{brand}-{flavor}.png`
- Rebuilds `seltzer-images/rank-map.json`

Then rebuild: `node build-reviews.mjs`

---

## Deployment

```
git add [files]
git commit -m "Description of changes"
git push origin main
```

GitHub Pages auto-deploys. Live at theseltzerreview.com within ~1 minute.

---

## YouTube / OAuth Credentials

- **Channel:** defnot_matty (`UCMzJOPeUbg9Qakr7nvvk2Kw`)
- **Account:** defnotmatty@gmail.com
- **API Key (read-only):** stored in `build-video-map.mjs`
- **OAuth Client:** registered in Google Cloud Console (defnotmatty@gmail.com)
- **Redirect URI:** `http://localhost:8765/callback`
- **Tokens:** `oauth_tokens.json` (gitignored)
- **Playlist ID:** `PLry3VfdA7_wnQlWCwskdSb6_dd1Wr4MQp`
- **Tools:** `yt-dlp` and `ffmpeg` installed at `C:\ytdl\`

---

## Key Files Reference

| File | Purpose |
|---|---|
| `index.html` | Homepage — edit manually |
| `reviews.html` | Rankings page — **generated, never edit directly** |
| `blog.html` | Blog article template |
| `Blog Posts/[date]/index.html` | Actual blog posts |
| `build-reviews.mjs` | Generates reviews.html from Excel + JSON maps |
| `build-video-map.mjs` | Rebuilds video-map.json from YouTube playlist |
| `extract-images.mjs` | Extracts can images from Excel, rebuilds rank-map.json |
| `update_thumbnail.mjs` | Updates YouTube thumbnail for latest video |
| `reauth_youtube.mjs` | Re-authenticates OAuth when token expires |
| `oauth_tokens.json` | OAuth tokens (gitignored) |
| `serve.mjs` | Local dev server on port 3000 |
| `screenshot.mjs` | Puppeteer screenshots to `temporary screenshots/` |
| `Defnotfinitive Hard Seltzer Rankings .xlsx` | Source of truth for all review data |
| `seltzer-images/rank-map.json` | Rank → can image path |
| `seltzer-images/video-map.json` | Rank → YouTube URL |
| `brand_assets/Spiked Seltzer Review Title Overlay.png` | 1536×1024 overlay PNG for thumbnails |
