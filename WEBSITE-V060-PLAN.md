# Leapfrog v0.6.0 Website Update Plan

**Date:** 2026-04-04
**Status:** Ready for implementation
**Site:** leapfrogmcp.com (Astro static, single-page)
**Deploy:** Cloudflare Pages via `wrangler.toml`

---

## Current State Summary

The site is a single `index.astro` page with inline `<style>` and a shared `global.css`. Key assets: `hero-video.mp4`, `og-card.png`, `session-grid.png`, `hero-sentinel.png`, and 5 video clips in `public/images/clips/`. The CSS variables use a navy palette (`--lf-void: #222733`) rather than the design system's pure black (`#000000`). This navy variant is intentional for the website and should be preserved -- it is warmer on screen than pure void.

The current page structure (top to bottom):
1. Nav
2. Hero (multi-session headline + terminal animation)
3. Stats strip (15 sessions, 31 tools, 537 tests, 10x tokens, 0 cloud)
4. "The Problem" section (agents fighting over one browser)
5. Session grid (15 tiled windows)
6. Features (8 cards, prose-heavy)
7. Comparison table (vs Playwright MCP, agent-browser, Stagehand)
8. Get Started (install block)
9. Origin story (blockquote)
10. Footer

---

## 1. Updated Hero Section

### Current
```
Overline: "Multi-session browser MCP server"
Headline: "Your agents share a browser. / Ours don't have to."
Sub: "15 parallel sessions. Full isolation. One server."
Terminal: npx leapfrog → v0.5.2 banner
```

### New

**Overline:** `The command center for AI browser agents`

**Headline (two lines, same fade-in stagger pattern):**
```
Line 1 (fade-in, 0.1s):  Your browser gets smarter
Line 2 (phosphor glow, 0.3s):  every time you use it.
```

The shift: v0.5.x led with isolation ("yours don't have to"). v0.6.0 leads with intelligence ("gets smarter"). Isolation is still prominent but moves to the stats strip and problem section.

**Subheadline (fade-in, 0.5s):**
```
Self-improving intelligence. Auto-tiling windows. Smart alerts.
15 parallel sessions that learn from every visit.
```

**CTAs:** Keep "Get Started" primary + "View on GitHub" outline. No change.

**Terminal animation update:**
```
$  npx leapfrog

   @..@     Leapfrog v0.6.0
  (----)    36 tools | 15 max sessions | stealth on
 ( >__< )   learning. (12 domains, 847 visits)
```

Changes from current terminal:
- Version bump to v0.6.0
- Tool count update to 36 (verify exact number before implementation)
- Third line changes from `watching.` to `learning. (12 domains, 847 visits)` -- this seeds the self-improving concept immediately
- Keep the same `--line-delay` stagger timing

**Stats strip update:**

| Stat | Old | New | Label |
|------|-----|-----|-------|
| 1 | 15 | 15 | parallel sessions |
| 2 | 31 | 36 | tools |
| 3 | 537 | 600+ | tests (update to actual count) |
| 4 | 10x | 31x | fewer tokens vs Playwright |
| 5 | 0 | 0 | cloud deps |

Add a 6th stat:
| 6 | (new) | 40% | fewer tokens by visit 50 |

Implementation: Add a 6th `.stat` div. The `data-count="40"` with suffix `%`. Update the counter JS to handle the `%` suffix the same way it handles `x`.

**Hero visual:** Keep the existing `hero-video.mp4` for now. The video clips (command-center-wide, etc.) are better suited for the new tiling section below the fold. If a new hero video is produced showing the tiling + HUD in action, swap it then. No blocker.

---

## 2. "Self-Improving Intelligence" Section

This is the single biggest addition to the page. It goes **immediately after the stats strip**, before "The Problem" section. It is the first content section a visitor scrolls into.

### Section structure

```html
<section class="section scroll-reveal" id="intelligence">
  <div class="container">
    <p class="overline">Self-improving intelligence</p>
    <h2>Visit #1: default everything.<br/>Visit #50: it already knows.</h2>
    <p class="text-secondary" style="max-width: 640px; margin-bottom: 3rem;">
      Leapfrog remembers what works for each website. Wait strategies,
      stealth tiers, cookie dismissals, API endpoints. Every visit
      makes the next one faster, cheaper, and more reliable.
    </p>

    <!-- Visit counter visual -->
    <div class="intel-comparison">
      <div class="intel-card intel-before">
        <div class="intel-visit-badge">Visit #1</div>
        <div class="intel-terminal">
          <!-- Terminal showing verbose first-visit output -->
        </div>
        <div class="intel-stat">~1,550 tokens</div>
      </div>

      <div class="intel-arrow">
        <!-- Animated arrow or "learns" connector -->
      </div>

      <div class="intel-card intel-after">
        <div class="intel-visit-badge phosphor">Visit #50</div>
        <div class="intel-terminal">
          <!-- Terminal showing optimized output -->
        </div>
        <div class="intel-stat phosphor">~487 tokens</div>
      </div>
    </div>

    <!-- Flywheel diagram -->
    <div class="flywheel">
      <!-- 4 nodes in a circle, connected by arrows -->
    </div>

    <!-- Technical depth toggle -->
    <details class="intel-details">
      <summary>How it works under the hood</summary>
      <div class="intel-details-body">
        <!-- ~/.leapfrog/domains/ file tree -->
      </div>
    </details>
  </div>
</section>
```

### Visit Counter Visual (the hero of this section)

Two terminal cards side by side (stacked on mobile). Left = Visit #1, Right = Visit #50.

**Visit #1 terminal content:**
```
github.com — first visit
  stealth:    full (14 patches)
  wait:       networkidle (default)
  cookies:    detect + dismiss
  snapshot:   full page scan
  tokens:     ~1,550
```

**Visit #50 terminal content:**
```
github.com — visit #50
  stealth:    tier-2 (4 patches)     ← learned
  wait:       domcontentloaded       ← learned
  cookies:    skip (none present)    ← learned
  snapshot:   scoped (#main)         ← learned
  tokens:     ~487                   40% fewer
```

The "learned" annotations on the right card are in phosphor green. This makes the improvement tangible and specific.

**CSS approach:**
- `.intel-comparison` — `display: grid; grid-template-columns: 1fr auto 1fr; gap: 2rem; align-items: center;`
- On mobile (`max-width: 768px`): `grid-template-columns: 1fr;` with the arrow rotated 90deg
- Each `.intel-card` uses the existing `.terminal` component style (dark deep background, border, rounded corners)
- `.intel-visit-badge` — pill badge, same style as `.session-badge` but larger (font-size 0.875rem)
- The arrow between cards: a simple `>` or `-->` in JetBrains Mono at 2rem, with a CSS animation that pulses opacity. Or use an SVG arrow with a phosphor glow.

**Animation:** The two cards use the existing `scroll-reveal` pattern. The left card reveals first (0ms), arrow next (200ms), right card last (400ms). The "learned" labels on the right card fade in with a stagger after the card appears, drawing the eye to what changed. Pure CSS animation, no JS required.

### Flywheel Diagram

Four nodes arranged in a horizontal row (not a circle -- circles are hard to read on mobile):

```
[ Visit site ] → [ Leapfrog observes ] → [ Stores in domain profile ] → [ Next visit is optimized ] → (loops back)
```

Implementation: Four small cards (120px wide) connected by thin lines with small arrow indicators. Each card has an icon and a 2-3 word label. The connecting lines use `background: var(--lf-border)` with the arrow in phosphor green.

| Node | Icon | Label |
|------|------|-------|
| 1 | Globe/compass | Visit site |
| 2 | Eye/binoculars | Observe |
| 3 | Database/folder | Store |
| 4 | Zap/lightning | Optimize |

On mobile, this becomes a vertical stack with downward arrows.

CSS only. No JS. The nodes use the existing `.badge` style scaled up.

### Technical Depth Toggle

A `<details>` element (native HTML, no JS) that reveals a file tree view:

```
~/.leapfrog/domains/
  github.com.json
  npmjs.com.json
  stripe.com.json
  ...

// github.com.json (excerpt)
{
  "visits": 47,
  "wait_strategy": "domcontentloaded",
  "stealth_tier": 2,
  "cookie_consent": "none",
  "known_endpoints": ["/api/graphql", "/api/v3/..."],
  "avg_tokens": 512
}
```

Style the `<details>` with the terminal component look. The `<summary>` gets a small chevron indicator (CSS triangle via `::before`). When open, it shows the code block inside the terminal body.

```css
.intel-details {
  max-width: 640px;
  margin: 2.5rem auto 0;
}
.intel-details summary {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.8125rem;
  color: var(--lf-text-muted);
  cursor: pointer;
  padding: 0.75rem 0;
  border-bottom: 1px solid var(--lf-border);
  list-style: none;
}
.intel-details summary::before {
  content: '>';
  display: inline-block;
  margin-right: 0.5rem;
  color: var(--lf-phosphor);
  transition: transform 0.2s;
}
.intel-details[open] summary::before {
  transform: rotate(90deg);
}
.intel-details-body {
  /* uses .terminal styling */
  margin-top: 1rem;
}
```

---

## 3. Updated Feature Grid

### Section changes

**Overline:** `What's in the box` (keep)
**Headline change:** `31 tools. Zero filler.` --> `36 tools. 11 new in v0.6.0.`

The feature grid moves from 8 prose-heavy cards to 11 compact cards with icons. Each card gets:
- An icon (Tabler icon name or emoji fallback)
- A feature name (bold, Space Grotesk)
- A one-line description (IBM Plex Sans, secondary color)

### Card layout change

Current cards are paragraph-heavy -- each has a bolded sentence as h3 plus a full paragraph. The new cards are compact: icon + name + one-liner. This fits 11 features without the page becoming a novel.

```css
.feature-card-v2 {
  background: var(--lf-surface);
  border: 1px solid var(--lf-border);
  border-radius: 12px;
  padding: 1.5rem;
  transition: border-color 0.2s, box-shadow 0.2s;
}
.feature-card-v2:hover {
  border-color: var(--lf-border-bright);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}
.feature-icon {
  font-size: 1.5rem;
  margin-bottom: 0.75rem;
  /* If using Tabler SVGs, set width: 28px; height: 28px; stroke: var(--lf-phosphor); */
}
.feature-card-v2 h3 {
  font-size: 1.0625rem;
  font-weight: 600;
  margin-bottom: 0.375rem;
}
.feature-card-v2 p {
  font-size: 0.875rem;
  color: var(--lf-text-secondary);
  line-height: 1.6;
}
```

Grid: `grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));` stays. With 11 cards, this naturally creates 3 rows of ~4 on desktop, stacking to 1-column on mobile.

### The 11 Features

Priority tiers: **Tier 1** = above the fold / first row. **Tier 2** = second row. **Tier 3** = third row.

| # | Icon (Tabler) | Feature Name | One-liner | Tier |
|---|---|---|---|---|
| 1 | `icon-brain` | Self-Improving Intelligence | Gets faster and smarter with every visit. 40% fewer tokens by visit 50. | 1 |
| 2 | `icon-layout-grid` | Auto Window Tiling | Headed browsers tile automatically. Add, remove, reflow. | 1 |
| 3 | `icon-alert-circle` | Smart Intervention Alerts | @..@ tells you exactly when and where you're needed. Fullscreen + chime. | 1 |
| 4 | `icon-palette` | Live HUD & Color Borders | Green = working. Blue = loading. Red = error. Status at a glance. | 1 |
| 5 | `icon-tag` | Smart Session Names | Sessions name themselves by domain. No more random IDs. | 2 |
| 6 | `icon-pointer` | Click Ripple & Agent Cursor | See the green cursor glide and click. Agent actions become visible. | 2 |
| 7 | `icon-remote` | Sidecar Remote Control | Focus, zoom, screenshot, emergency-stop via URL. Works with Alfred/Raycast. | 2 |
| 8 | `icon-cookie` | Cookie Consent Crusher | Auto-dismisses OneTrust, CookieBot, TrustArc. Silently. | 2 |
| 9 | `icon-pin` | Pinned Sessions | Pin a session and it stays alive. No idle timeout. | 3 |
| 10 | `icon-video` | Session Recording & Tracing | Full Playwright traces + video screencasts. Time-travel debugging. | 3 |
| 11 | `icon-bell` | Sound & Notifications | One warm chime when you're needed. macOS notification. That's it. | 3 |

**Icon implementation:** Use inline SVGs from Tabler Icons (https://tabler.io/icons), sized at 28x28, stroke `var(--lf-phosphor)`, stroke-width 1.5. If SVG integration is too heavy for the first pass, use emoji fallback:

| Feature | Emoji Fallback |
|---|---|
| Self-Improving Intelligence | brain emoji or circled "AI" text in phosphor |
| Auto Window Tiling | grid/window emoji |
| Smart Alerts | frog `@..@` in phosphor (brand moment) |
| Live HUD | palette/circle emoji |
| Smart Session Names | tag emoji |
| Click Ripple | pointer emoji |
| Sidecar | remote/gamepad emoji |
| Cookie Crusher | cookie emoji |
| Pinned Sessions | pin emoji |
| Session Recording | film/video emoji |
| Sound | bell emoji |

**Recommendation:** Use Tabler SVG icons inlined. They match the design system's duotone spec (stroke in phosphor, fill in `rgba(0,255,65,0.1)`). 11 small SVGs add negligible weight.

### Preserving the old feature cards

The 8 existing feature cards described v0.5.x capabilities (multi-session, stealth, network intercept, crash recovery, persistence, token efficiency, popups, etc.). These do NOT disappear. They move to a secondary "Core Capabilities" section below the v0.6.0 feature grid, or they merge into the feature grid as the bottom rows. Recommended approach: merge the most important ones (multi-session, stealth, token efficiency, network intercept) into a "Core" row at the bottom of the same grid:

| # | Feature Name | One-liner | Tier |
|---|---|---|---|
| 12 | 15 Parallel Sessions | Each one isolated. Own cookies, storage, fingerprint. Zero collisions. | 3 |
| 13 | 14-Patch Stealth | Mouse curves, typing delays, navigator patches. Sites think you're real. | 3 |
| 14 | Network Intelligence | See every HTTP request. Block, mock, or capture any of them. | 3 |
| 15 | Crash Recovery | Browser dies, sessions clean, fresh Chromium on the next call. | 3 |

This gives a 15-card grid (11 new + 4 core). On desktop with `minmax(280px, 1fr)`, this lays out as 4 columns x 4 rows (last row has 3 cards). Clean.

---

## 4. Competitive Positioning Update

### Current comparison table

The table compares Leapfrog vs Playwright MCP vs agent-browser vs Stagehand across 8 rows. Keep this structure but update the data and add rows.

### Changes to existing rows

| Row | Current Leapfrog Value | New Value |
|------|---|---|
| Tokens / page | ~1,400 | ~1,400 (visit 1) / ~487 (visit 50) |
| Tools | (not a row) | Add row: 36 tools |

### New rows to add

| Row | Leapfrog | Playwright MCP | agent-browser | Stagehand |
|-----|----------|----------------|---------------|-----------|
| Self-improving | Yes (per-domain) | No | No | No |
| Auto tiling | Yes (grid + master-stack) | No | No | Cloud only |
| Human intervention | @..@ fullscreen + chime | No | No | No |
| Session recording | Playwright traces + video | Traces only | No | Cloud |
| Window HUD | Color-coded borders | No | No | No |

### Positioning copy update

Replace the current intro paragraph:

**Before:**
> agent-browser wins on raw token count. browser-use has 81K stars. Stagehand has the best natural language DX. Leapfrog ships the combination nobody else does.

**After:**
> Other tools do one thing well. Leapfrog gets better at everything -- automatically. Visit #50 uses 31x fewer tokens than Playwright MCP on the same page. And it learned that on its own.

### Callout block below the table

Add a standalone callout/highlight box below the table:

```html
<div class="comparison-callout">
  <p class="mono phosphor" style="font-size: 1.125rem; margin-bottom: 0.5rem;">
    Visit #50: ~487 tokens.
  </p>
  <p class="text-secondary" style="font-size: 0.9375rem;">
    Playwright MCP: ~14,000 tokens. Same page. Same data.
    That's 31x fewer tokens -- and Leapfrog figured it out by itself.
  </p>
</div>
```

Style:
```css
.comparison-callout {
  background: var(--lf-deep);
  border: 1px solid var(--lf-border);
  border-left: 3px solid var(--lf-phosphor);
  border-radius: 10px;
  padding: 1.5rem 2rem;
  margin-top: 2rem;
  max-width: 640px;
}
```

---

## 5. New Screenshots Needed

### Screenshot 1: Auto-Tiling Grid (HERO CANDIDATE)

**What:** 6 headed browser windows perfectly tiled in a 3x2 grid, each showing a different website. Each window has a colored HUD border (mix of green/blue/purple to show different states). Session names visible in the HUD bar (`[github]`, `[stripe]`, `[gmail]`, etc.).

**Browser config:**
- macOS, Leapfrog running headed with `LEAP_HEADLESS=false`
- 6 sessions active, visiting: github.com, stripe.com/docs, gmail.com, npmjs.com, vercel.com, linear.app
- Screenshot the entire screen (all 6 windows visible)
- Resolution: Retina 2x, crop to 1920x1080 logical pixels

**Annotations:** None on the screenshot itself. Let the visual speak. Add a caption below: `6 sessions. Auto-tiled. Each one isolated.`

**Usage:** Self-improving section visual or replacement hero background. Also good for og-card.

### Screenshot 2: Visit Counter Side-by-Side

**What:** Two terminal windows showing Leapfrog's domain profile output. Left = first visit (verbose), right = 50th visit (optimized). This may be better as a designed graphic rather than a real screenshot, since the domain profile data needs to be curated for readability.

**Browser config:** N/A -- this is a terminal capture or designed mockup.

**Approach:** Create as a styled HTML component (the `.intel-comparison` section described above). Screenshot the rendered component for use in social/OG.

### Screenshot 3: @..@ Intervention Alert

**What:** A single browser window in fullscreen showing the `@..@ intervention alert -- frog eyes centered on screen, arrow pointing to a CAPTCHA element, with the HUD border flashing amber/red.

**Browser config:**
- Trigger a human intervention on a site with a visible CAPTCHA (e.g., any site with reCAPTCHA)
- Leapfrog headed mode, single session
- Screenshot at the moment the @..@ overlay appears

**Annotations:** Arrow overlay pointing from @..@ to the CAPTCHA with text "Handle this. Click Done." (this should be the actual UI, not a post-overlay).

**Usage:** Feature card for Smart Alerts. Also good for X/Twitter posts.

### Screenshot 4: Click Ripple in Action

**What:** A browser window mid-action, showing the green agent cursor approaching a button, with a green ripple animation expanding from a previous click point.

**Browser config:**
- Headed mode, single session
- Navigate to a form-heavy page
- Capture mid-animation (screen recording > frame extract, or slow-motion the animation)

**Annotations:** None needed. The green cursor and ripple are self-explanatory.

**Usage:** Feature card for Click Ripple. Video clip variant for the scrolling reel.

### Screenshot 5: HUD Color States

**What:** A 2x2 grid of browser windows, each in a different HUD state: green (active), blue (loading), red (error), purple (complete). Each window's colored border is clearly visible.

**Browser config:**
- 4 sessions in headed mode
- Manually trigger different states or capture during a real multi-session run
- Crop to show all 4 windows with their colored borders prominently visible

**Annotations:** Small labels below each window: "Active", "Loading", "Error", "Complete" -- or add them as overlays in post.

**Usage:** Feature card for Live HUD. Could also be a horizontal strip image.

### Screenshot 6: Domain Profile File Tree

**What:** Terminal showing `ls ~/.leapfrog/domains/` with 8-10 domain JSON files, then `cat ~/.leapfrog/domains/github.com.json` showing the learned profile.

**Browser config:** N/A -- terminal capture.

**Approach:** Run the actual commands and capture iTerm2 output. Use the Leapfrog color scheme if iTerm profile is available.

**Usage:** Technical depth toggle content in the self-improving section.

---

## 6. OG Card / Social Preview Update

### Current og-card.png

8.5MB PNG. Likely the v0.5.x card. Needs full replacement.

### New OG Card Concept

**Dimensions:** 1200x630px (standard OG)

**Layout:**
```
+--------------------------------------------------+
|                                                    |
|   @..@                                             |
|  (----)    LEAPFROG v0.6.0                        |
| ( >__< )                                          |
|                                                    |
|   The command center for                           |
|   AI browser agents.                               |
|                                                    |
|   Self-improving intelligence.                     |
|   15 parallel sessions.                            |
|   Zero cloud.                                      |
|                                                    |
|   Visit #1: 1,550 tokens                           |
|   Visit #50: 487 tokens  <-- phosphor green        |
|                                                    |
+--------------------------------------------------+
```

**Background:** `--lf-void` (#222733) with the dot grid pattern at 3% opacity
**Frog:** ASCII art in phosphor green, left-aligned, large
**Text:** Space Grotesk for headlines, JetBrains Mono for the visit counter
**Accents:** The "Visit #50" line is in phosphor green. Everything else is `--lf-text-primary`
**Border:** 1px solid `--lf-border` with a subtle phosphor glow on the left edge

**File requirements:**
- Export as PNG, optimize to under 500KB (current 8.5MB is way too heavy for OG)
- Also export a 2x version (2400x1260) for high-DPI social previews
- Name: `og-card-v060.png`, then swap the reference in `index.astro`

**Implementation:** Create as an HTML page, screenshot with Leapfrog or Playwright, optimize with `img_optimize` MCP tool.

### Twitter Card

Same image works for `twitter:card` since we use `summary_large_image`. Update the meta tags:

```html
<meta property="og:title" content="Leapfrog v0.6.0 — Self-Improving Browser Intelligence for AI Agents" />
<meta property="og:description" content="15 parallel sessions that get smarter with every visit. Auto-tiling, smart alerts, session recording. Zero cloud." />
```

---

## 7. Implementation Notes

### Files to modify

| File | Changes |
|------|---------|
| `src/pages/index.astro` | All HTML content, inline styles, inline script |
| `src/styles/global.css` | New component styles (intel section, flywheel, feature-card-v2, comparison-callout, details toggle) |

### New files (none required but recommended)

No new Astro components are needed -- the site is a single page with inline everything. All changes go into the existing two files. However, if the implementation gets unwieldy, consider extracting sections into Astro components:

| Potential component | Contents |
|---|---|
| `src/components/IntelSection.astro` | The self-improving intelligence section |
| `src/components/FeatureGrid.astro` | The 15-card feature grid |
| `src/components/ComparisonTable.astro` | The updated comparison table |

This is optional. The current pattern of everything in `index.astro` works fine for a single-page site.

### New assets needed

| Asset | Location | Source |
|------|----------|--------|
| `og-card-v060.png` | `public/images/` | Designed + screenshotted |
| `screenshot-tiling.png` | `public/images/` | Captured from headed Leapfrog |
| `screenshot-intervention.png` | `public/images/` | Captured from headed Leapfrog |
| `screenshot-hud-states.png` | `public/images/` | Captured from headed Leapfrog |
| 11 Tabler SVG icons (inline) | Embedded in `index.astro` | https://tabler.io/icons |

### CSS additions to `global.css`

```css
/* === Self-Improving Intelligence Section === */

.intel-comparison {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 2rem;
  align-items: center;
  margin-top: 2.5rem;
}

.intel-card {
  background: var(--lf-deep);
  border: 1px solid var(--lf-border);
  border-radius: 12px;
  overflow: hidden;
}

.intel-visit-badge {
  display: inline-block;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.875rem;
  font-weight: 500;
  padding: 0.25rem 0.75rem;
  border-radius: 999px;
  border: 1px solid var(--lf-border-bright);
  color: var(--lf-text-muted);
  margin-bottom: 1rem;
}
.intel-visit-badge.phosphor {
  color: var(--lf-phosphor);
  border-color: rgba(0, 255, 65, 0.3);
  background: rgba(0, 255, 65, 0.06);
}

.intel-stat {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 1.5rem;
  font-weight: 700;
  text-align: center;
  padding: 1rem;
  border-top: 1px solid var(--lf-border);
  color: var(--lf-text-primary);
}
.intel-stat.phosphor {
  color: var(--lf-phosphor);
  text-shadow: 0 0 20px rgba(0, 255, 65, 0.3);
}

.intel-arrow {
  font-family: 'JetBrains Mono', monospace;
  font-size: 1.5rem;
  color: var(--lf-text-muted);
  animation: arrowPulse 2s ease-in-out infinite;
}
@keyframes arrowPulse {
  0%, 100% { opacity: 0.4; }
  50% { opacity: 1; color: var(--lf-phosphor); }
}

.intel-terminal {
  padding: 1.25rem 1.5rem;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.8125rem;
  line-height: 1.8;
  color: var(--lf-text-secondary);
}
.intel-terminal .learned {
  color: var(--lf-phosphor);
  font-size: 0.75rem;
}

/* Flywheel */
.flywheel {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0;
  margin: 3rem auto;
  max-width: 720px;
}
.flywheel-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  min-width: 100px;
}
.flywheel-icon {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: var(--lf-surface);
  border: 1px solid var(--lf-border);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.25rem;
}
.flywheel-label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.6875rem;
  color: var(--lf-text-muted);
  text-align: center;
}
.flywheel-connector {
  flex: 1;
  height: 1px;
  background: var(--lf-border);
  position: relative;
  min-width: 40px;
}
.flywheel-connector::after {
  content: '>';
  position: absolute;
  right: -4px;
  top: -8px;
  color: var(--lf-phosphor);
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.75rem;
}

/* Details toggle */
.intel-details {
  max-width: 640px;
  margin: 2.5rem auto 0;
}
.intel-details summary {
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.8125rem;
  color: var(--lf-text-muted);
  cursor: pointer;
  padding: 0.75rem 0;
  border-bottom: 1px solid var(--lf-border);
  list-style: none;
  user-select: none;
}
.intel-details summary::-webkit-details-marker { display: none; }
.intel-details summary::before {
  content: '>';
  display: inline-block;
  margin-right: 0.5rem;
  color: var(--lf-phosphor);
  transition: transform 0.2s;
  font-weight: 700;
}
.intel-details[open] summary::before {
  transform: rotate(90deg);
}
.intel-details-body {
  margin-top: 1rem;
}

/* Feature card v2 (compact) */
.feature-card-v2 {
  background: var(--lf-surface);
  border: 1px solid var(--lf-border);
  border-radius: 12px;
  padding: 1.5rem;
  transition: border-color 0.2s, box-shadow 0.2s;
}
.feature-card-v2:hover {
  border-color: var(--lf-border-bright);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}
.feature-icon {
  width: 28px;
  height: 28px;
  margin-bottom: 0.75rem;
  color: var(--lf-phosphor);
}
.feature-icon svg {
  width: 100%;
  height: 100%;
  stroke: currentColor;
  stroke-width: 1.5;
  fill: none;
}
.feature-card-v2 h3 {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 1.0625rem;
  font-weight: 600;
  margin-bottom: 0.375rem;
  color: var(--lf-text-primary);
}
.feature-card-v2 p {
  font-size: 0.875rem;
  color: var(--lf-text-secondary);
  line-height: 1.6;
}

/* Comparison callout */
.comparison-callout {
  background: var(--lf-deep);
  border: 1px solid var(--lf-border);
  border-left: 3px solid var(--lf-phosphor);
  border-radius: 10px;
  padding: 1.5rem 2rem;
  margin-top: 2rem;
  max-width: 640px;
}

/* Mobile overrides for new sections */
@media (max-width: 768px) {
  .intel-comparison {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
  .intel-arrow {
    transform: rotate(90deg);
    text-align: center;
  }
  .flywheel {
    flex-direction: column;
    gap: 0.5rem;
  }
  .flywheel-connector {
    width: 1px;
    height: 32px;
    min-width: unset;
  }
  .flywheel-connector::after {
    right: -8px;
    top: unset;
    bottom: -4px;
    transform: rotate(90deg);
  }
}
```

### Script changes

The inline `<script>` in `index.astro` needs one update:

1. **Stats counter:** Update to handle the `%` suffix on the new 6th stat. Currently it only handles `x` for the token stat. Change the suffix logic:

```javascript
// Current:
const suffix = target === 10 ? 'x' : '';

// New:
const label = stat.querySelector('.stat-label').textContent;
const suffix = label.includes('fewer tokens') && target === 31 ? 'x'
             : label.includes('visit 50') ? '%'
             : '';
```

Better approach -- use a `data-suffix` attribute on each `.stat` div:

```html
<div class="stat" data-count="31" data-suffix="x">...</div>
<div class="stat" data-count="40" data-suffix="%">...</div>
```

Then in JS:
```javascript
const suffix = stat.dataset.suffix || '';
```

2. **Scroll reveal:** The new `#intelligence` section needs the `scroll-reveal` class (already handled by the existing IntersectionObserver).

No other JS changes needed. The `<details>` toggle, CSS animations, and flywheel are all pure CSS/HTML.

### Meta tag updates

```html
<title>Leapfrog v0.6.0 — Self-Improving Browser Intelligence for AI Agents</title>
<meta name="description" content="Self-improving browser MCP server. 15 isolated sessions that learn from every visit. Auto-tiling, smart alerts, session recording. One install. Zero cloud." />
<meta property="og:title" content="Leapfrog v0.6.0 — Self-Improving Browser Intelligence for AI Agents" />
<meta property="og:description" content="15 parallel sessions that get smarter with every visit. Auto-tiling, smart alerts, session recording. Zero cloud." />
<meta property="og:image" content="/images/og-card-v060.png" />
```

### Section ordering (final page structure)

1. **Nav** (unchanged)
2. **Hero** (updated copy, terminal shows v0.6.0 + learning status)
3. **Stats strip** (updated numbers, add 6th stat for 40% improvement)
4. **Self-Improving Intelligence** (NEW -- the centerpiece section)
5. **The Problem** (keep, still relevant -- "your agents are waiting in line")
6. **Session Grid** (keep, the 15-window visual is still the signature move)
7. **Feature Grid** (rewritten -- 15 compact cards with icons)
8. **Comparison Table** (updated rows + callout box)
9. **Get Started** (update version in terminal, otherwise keep)
10. **Origin Story** (keep)
11. **Footer** (keep)

### Version references to update

- Hero terminal: `v0.5.2` --> `v0.6.0`
- Hero terminal: `31 tools` --> `36 tools` (verify exact)
- Hero terminal: `watching.` --> `learning. (12 domains, 847 visits)`
- Stats strip: `31` tools --> `36` tools
- Stats strip: `10x` fewer tokens --> `31x` fewer tokens
- Stats strip: `537` tests --> actual test count
- Get Started install block: no change needed (npx leapfrog auto-resolves)
- OG image reference: `og-card.png` --> `og-card-v060.png`

### Build & deploy

```bash
cd /Users/ted/Projects/leapfrogmcp.com
npm run build   # generates dist/
npx wrangler pages deploy dist/
```

No Astro config changes needed. Output remains `static`.

---

## Appendix: What NOT to Change

- **Navy color palette** -- The website uses `#222733` void instead of `#000000`. This is intentional and reads better on screens. Do not "fix" it to match the design system's pure black.
- **Session grid section** -- The 5x3 grid of 15 session windows is the visual signature. Keep it.
- **Origin story** -- The blockquote about two agents crashing is the founding narrative. Keep it.
- **Hero video** -- The current `hero-video.mp4` works. Replace only when a v0.6.0 demo video showing tiling+HUD is available.
- **Frog blink animation** -- The 8-second blink cycle on `@..@` is a charming ambient detail. Keep it.
- **Footer tagline** -- "Every agent needs a scout." is the primary tagline. Keep it.
