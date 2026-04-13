---
name: good-deck
description: Use when the user asks to create a presentation, slide deck, or slides. Builds a single-file vanilla HTML/CSS/JS slide deck with keyboard navigation, progress bar, animated transitions, practice mode with talk track, and optional speaker script — no frameworks, no build step.
license: MIT
metadata:
  author: ajotwani
  version: "2.0"
---

# Good Deck

Build presentation slide decks as a single `index.html` file using vanilla HTML, CSS, and JS. No frameworks, no build tools, no dependencies beyond Google Fonts. The result is a self-contained, deployable presentation with polished navigation, animations, and a built-in practice mode.

## When to Use

- User asks to "make a presentation", "create slides", "build a deck"
- User has content (outline, notes, stats) they want turned into a visual presentation
- User wants something lightweight — not PowerPoint, not Reveal.js, not a framework

## Workflow

1. **Gather content first** — Ask the user for: topic, audience, tone, key points/outline, any stats or images. If they have a markdown content doc, read it.
2. **Copy the starter template** — Copy `assets/starter.html` to the project as `index.html`. This has the full slide system, navigation, practice mode, and example slides ready to go.
3. **Build out slides** — Replace the example slides with the user's content. Use the component patterns below and in `references/` for layouts.
4. **Add talk track** — Populate the `talkTrack` JS object with per-slide speaker notes.
5. **Optionally add live demos** — Read `references/live-demos.md` for interactive API demo patterns.
6. **Optionally add Flask wrapper** — Read `references/deployment.md` for password-protected hosting.

## Architecture

```
project/
  index.html          # The entire deck — CSS + HTML + JS
  images/             # Any images referenced by slides
  app.py              # Optional Flask wrapper for auth + API proxying
  requirements.txt    # Optional (flask, gunicorn, requests)
  Dockerfile          # Optional for deployment
```

Everything that matters is in `index.html`. The deck works by opening the file in a browser.

## Progressive Disclosure — Reference Files

The core patterns are below. For specialized features, load these references on demand:

- **Live API demos** (run buttons, model dropdowns, copy-to-clipboard, JSON toggle, spinners) → Read `references/live-demos.md`
- **Practice mode** (talk track panel, stage notes, formatting guide) → Read `references/practice-mode.md`
- **Slide components** (stat cards, badges, quote cards, team grids, timelines, code highlighters) → Read `references/slide-components.md`
- **Advanced components** (comparison panels, cost bars, journey tracks, step badges, background overlays, WIP banner) → Read `references/advanced-components.md`
- **Deployment** (Flask wrapper, Dockerfile, env vars, password auth) → Read `references/deployment.md`

## CSS Foundation

### Variables & Theming

Define all colors, fonts, and spacing as CSS variables. Ask the user about brand colors, or use this default:

```css
:root {
  --bg-deep: #0f0f0f;
  --bg-surface: rgba(255, 255, 255, 0.05);
  --bg-elevated: rgba(255, 255, 255, 0.08);
  --accent: #3b82f6;
  --accent-light: #60a5fa;
  --accent-glow: rgba(59, 130, 246, 0.25);
  --accent-subtle: rgba(59, 130, 246, 0.08);
  --success: #22c55e;
  --warning: #eab308;
  --text-primary: #f1f5f9;
  --text-secondary: rgba(255, 255, 255, 0.7);
  --text-muted: rgba(255, 255, 255, 0.4);
  --border: rgba(255, 255, 255, 0.1);
  --card-bg: rgba(255, 255, 255, 0.95);
  --card-text: #1a1a2e;
  --card-text-muted: #5a5a7a;
  --font-display: 'Bricolage Grotesque', sans-serif;
  --font-body: 'Lexend', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}
```

Load fonts via Google Fonts `<link>` in `<head>`:
- Display: `Bricolage Grotesque` (bold, expressive)
- Body: `Lexend` (clean, readable)
- Code: `JetBrains Mono`

### Base Reset

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html, body {
  width: 100%; height: 100%;
  overflow: hidden;
  background: var(--bg-deep);
  color: var(--text-primary);
  font-family: var(--font-body);
  -webkit-font-smoothing: antialiased;
}
```

## Slide System

Slides are `<section class="slide">` inside `<div class="deck">`. Only one has `.active` at a time.

**Layout classes:**
- Default: left-aligned, top-start — content slides with text + code
- `.slide-centered`: centered both ways — transitions, quotes, card grids
- `.slide-title`: left-aligned, vertically centered — title slides

### Slide 0: Navigation Info

Always include as the first slide. Shows keyboard shortcuts for the presenter.

### HTML Pattern

```html
<div class="deck">
  <section class="slide slide-centered"><!-- Slide 0: nav info --></section>
  <section class="slide slide-centered"><!-- Slide 1: title --></section>
  <section class="slide slide-centered"><!-- Slide 2+ --></section>
</div>
```

### Speaker Avatars on Title Slide

If speaker photos are available, include circular avatars with name and title. See the starter template for the full CSS pattern.

## Staggered Entry Animations

Elements with `.animate-in` fade up when their slide becomes active:

```css
.animate-in {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}
.slide.active .animate-in { opacity: 1; transform: translateY(0); }
.slide.active .animate-in:nth-child(1) { transition-delay: 0.1s; }
.slide.active .animate-in:nth-child(2) { transition-delay: 0.25s; }
.slide.active .animate-in:nth-child(3) { transition-delay: 0.4s; }
/* ... up to nth-child(8) at 1.15s */
```

## Typography

```css
.headline { font-family: var(--font-display); font-size: 72px; font-weight: 800; line-height: 1.1; }
.headline-md { font-family: var(--font-display); font-size: 56px; font-weight: 800; line-height: 1.15; }
.headline-sm { font-family: var(--font-display); font-size: 40px; font-weight: 700; line-height: 1.2; }
.subtitle { font-size: 22px; color: var(--text-secondary); line-height: 1.5; max-width: 700px; }
.label { font-family: var(--font-mono); font-size: 14px; text-transform: uppercase; letter-spacing: 2px; color: var(--accent-light); margin-bottom: 16px; }
.text-accent { color: var(--accent-light); }
.text-muted { color: var(--text-muted); }
```

## Card Grid

```css
.card-grid { display: grid; gap: 20px; max-width: 900px; width: 100%; }
.card-grid-2 { grid-template-columns: 1fr 1fr; }
.card-grid-3 { grid-template-columns: 1fr 1fr 1fr; }
.card {
  background: var(--card-bg); color: var(--card-text);
  border-radius: 16px; padding: 28px 24px; text-align: left;
}
.card h3 { font-family: var(--font-display); font-size: 20px; font-weight: 700; margin-bottom: 8px; }
.card p { font-size: 15px; line-height: 1.5; color: var(--card-text-muted); }
.card-label { font-family: var(--font-mono); font-size: 11px; text-transform: uppercase; letter-spacing: 1.5px; color: var(--accent); margin-bottom: 8px; }
.card-highlight { border: 2px solid var(--accent); }
```

## Keyboard Navigation

| Key | Action |
|-----|--------|
| Arrow Right / Space / Enter | Next slide |
| Arrow Left / Backspace | Previous slide |
| Home | First slide |
| End | Last slide |
| G | Jump to slide overlay |
| P | Toggle practice mode panel |
| O | Slide overview grid |

All navigation JS is in the starter template. Click-to-navigate is disabled — use keyboard only.

## UI Chrome

Always include after the `.deck` div:
- **Progress bar** — 2px bar at bottom, fills as you advance
- **Slide counter** — Blue circle at bottom-right, clickable to open jump overlay
- **Footer** — Logo or event branding at bottom-left
- **Nav hint** — Fades after first navigation

All included in the starter template.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `overflow: hidden` on `html, body` | Causes scroll bleed between slides |
| Missing `pointer-events: none` on inactive slides | Click events leak to wrong slide |
| No `e.preventDefault()` on Space/Enter keys | Page scrolls or form submits |
| Animations not resetting on revisit | Use `triggerAnimations()` to reset state |
| Images not loading when deployed | Make sure Flask wrapper serves all asset directories |
| Accidental slide navigation on click | Click-to-navigate is disabled by default — keyboard only |
| Practice panel overlays content | Use `body.practice-open .deck { width: calc(100vw - 420px); }` |
| `demo-code` indentation is wrong | Content uses `white-space: pre` — formatting inside the tag matters |

## Sizing for Screen Sharing

When presentations will be screen-shared (Zoom, Meet), scale everything up ~20%. Headlines should be 64px+ minimum, body text 20px+, labels 18px+. Test at the resolution you'll actually present at.
