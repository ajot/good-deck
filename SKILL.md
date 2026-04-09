---
name: good-deck
description: Use when the user asks to create a presentation, slide deck, or slides. Builds a single-file vanilla HTML/CSS/JS slide deck with keyboard navigation, progress bar, animated transitions, practice mode with talk track, and optional speaker script — no frameworks, no build step.
license: MIT
metadata:
  author: ajotwani
  version: "1.0"
---

# Good Deck

## Overview

Build presentation slide decks as a single `index.html` file using vanilla HTML, CSS, and JS. No frameworks, no build tools, no dependencies beyond Google Fonts. The result is a self-contained, deployable presentation with polished navigation, animations, and a built-in practice mode.

## When to Use

- User asks to "make a presentation", "create slides", "build a deck"
- User has content (outline, notes, stats) they want turned into a visual presentation
- User wants something lightweight — not PowerPoint, not Reveal.js, not a framework

## Workflow

1. **Gather content first** — Ask the user for: topic, audience, tone, key points/outline, any stats or images. If they have a markdown content doc, read it.
2. **Build the single HTML file** — All CSS, HTML, and JS in one `index.html`.
3. **Include a Slide 0** — Navigation/info slide (see Slide 0 section below).
4. **Include speaker avatars** on the title slide if speaker photos are available.
5. **Include practice mode** — Built-in talk track panel toggled with P key (see Practice Mode section).
6. **Optionally add a Flask wrapper** — For password-protected hosting and live API demos (see Deployment section).

## Architecture: Single-File Structure

```
project/
  index.html          # The entire deck — CSS + HTML + JS
  images/             # Any images referenced by slides (avatars, logos, screenshots)
  app.py              # Optional Flask wrapper for auth + API proxying
  requirements.txt    # Optional (flask, gunicorn, requests)
  Dockerfile          # Optional for deployment
```

Everything that matters is in `index.html`. The deck works by opening the file in a browser.

## CSS Foundation

### Variables & Theming

Define all colors, fonts, and spacing as CSS variables so the whole theme is tunable from one block. Ask the user about brand colors and tone, or pick a clean default.

```css
:root {
  --bg-deep: #0f0f0f;
  --bg-surface: #1a1a1a;
  --bg-elevated: #252525;
  --accent: #3b82f6;
  --accent-light: #60a5fa;
  --accent-glow: rgba(59, 130, 246, 0.25);
  --success: #22c55e;
  --warning: #eab308;
  --text-primary: #f1f5f9;
  --text-secondary: #94a3b8;
  --text-muted: #475569;
  --border: #334155;
  --font-display: 'Bricolage Grotesque', sans-serif;
  --font-body: 'Lexend', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}
```

**Font stack:** Use Google Fonts. Good defaults:
- Display/headlines: `Bricolage Grotesque` (bold, expressive)
- Body text: `Lexend` (clean, readable)
- Code/data: `JetBrains Mono`

Load via `<link>` in `<head>` — no local font files.

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

### Optional Background Grid

A subtle grid gives depth without distraction:

```css
body {
  background-image:
    linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px);
  background-size: 50px 50px;
}
```

## Slide System

### Slide Container & Transitions

```css
.deck {
  width: 100vw; height: 100vh; position: relative;
  transition: width 0.35s ease;
}
body.practice-open .deck {
  width: calc(100vw - 420px);
}

.slide {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-start;
  padding: 72px 80px 100px;
  opacity: 0;
  transform: translateX(80px);
  transition: opacity 0.5s ease, transform 0.5s ease;
  pointer-events: none;
  text-align: left;
}

/* Centered slides — for transitions, quotes, title, card grids */
.slide.slide-centered {
  justify-content: center;
  align-items: center;
  text-align: center;
}

/* Title slide — left-aligned, vertically centered */
.slide.slide-title {
  justify-content: center;
  align-items: flex-start;
}

.slide.active {
  opacity: 1;
  transform: translateX(0);
  pointer-events: auto;
}

.slide.exit-left {
  opacity: 0;
  transform: translateX(-80px);
}

.slide.exit-right {
  opacity: 0;
  transform: translateX(80px);
}
```

Each slide is a `<section class="slide">` inside a `<div class="deck">`. Only one has `.active` at a time.

**Layout classes:**
- Default (no class): left-aligned, top-start — good for content slides with text + code
- `.slide-centered`: centered both ways — for transitions, quotes, card grids, takeaways
- `.slide-title`: left-aligned, vertically centered — for title slides with speaker info

### Slide 0: Navigation / Info Slide

Always include a Slide 0 as the first slide. This is a speaker-only slide that shows navigation instructions. The slide counter should show `0` for this slide.

```html
<!-- ========== SLIDE 0: Navigation ========== -->
<section class="slide slide-centered">
  <div style="font-family: var(--font-mono); font-size: 14px; color: var(--text-muted); line-height: 2.2;" class="animate-in">
    <span style="color: var(--text-secondary);">&larr; &rarr;</span> &nbsp;Navigate &nbsp;&nbsp;&middot;&nbsp;&nbsp;
    <span style="color: var(--text-secondary);">Space</span> &nbsp;Next &nbsp;&nbsp;&middot;&nbsp;&nbsp;
    <span style="color: var(--text-secondary);">G</span> &nbsp;Jump to slide &nbsp;&nbsp;&middot;&nbsp;&nbsp;
    <span style="color: var(--text-secondary);">P</span> &nbsp;Practice mode &nbsp;&nbsp;&middot;&nbsp;&nbsp;
    <span style="color: var(--text-secondary);">Click</span> &nbsp;Left = back, rest = forward
  </div>
</section>
```

### Speaker Avatars on Title Slide

If speaker photos are available, include circular avatars with name and title on the title slide:

```css
.speaker-row {
  display: flex;
  gap: 48px;
  margin-top: 16px;
}
.speaker {
  display: flex;
  align-items: center;
  gap: 16px;
}
.speaker-avatar {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid rgba(255, 255, 255, 0.2);
}
.speaker-name {
  font-family: var(--font-body);
  font-size: 16px;
  font-weight: 600;
  color: var(--text-primary);
}
.speaker-title {
  font-family: var(--font-body);
  font-size: 13px;
  color: var(--text-muted);
  margin-top: 2px;
}
```

```html
<div class="speaker-row animate-in">
  <div class="speaker">
    <img src="images/speaker1.jpg" class="speaker-avatar" alt="Name">
    <div>
      <div class="speaker-name">Speaker Name</div>
      <div class="speaker-title">Title, Company</div>
    </div>
  </div>
  <!-- more speakers -->
</div>
```

### HTML Pattern

```html
<div class="deck">

  <!-- ========== SLIDE 0: Navigation ========== -->
  <section class="slide slide-centered">
    <!-- navigation instructions (see above) -->
  </section>

  <!-- ========== SLIDE 1: Title ========== -->
  <section class="slide slide-centered">
    <div class="title-main animate-in">Talk Title<br><span class="text-accent">Subtitle</span></div>
    <div class="title-sub animate-in">One-line description</div>
    <div class="speaker-row animate-in"><!-- avatars --></div>
  </section>

  <!-- ========== SLIDE 2: Section Transition ========== -->
  <section class="slide slide-centered">
    <div class="label animate-in">Section Name</div>
    <h2 class="headline animate-in">Big headline<br><span class="text-accent">with accent.</span></h2>
  </section>

  <!-- ========== SLIDE 3: Content ========== -->
  <section class="slide slide-centered">
    <div class="label animate-in">Topic Label</div>
    <h2 class="headline-sm animate-in">Headline</h2>
    <div class="card-grid card-grid-3 animate-in"><!-- cards --></div>
  </section>

  <!-- ... more slides ... -->
</div>
```

## Staggered Entry Animations

Elements with `.animate-in` fade up when their slide becomes active:

```css
.animate-in {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}

.slide.active .animate-in {
  opacity: 1;
  transform: translateY(0);
}

/* Stagger each child */
.slide.active .animate-in:nth-child(1) { transition-delay: 0.1s; }
.slide.active .animate-in:nth-child(2) { transition-delay: 0.25s; }
.slide.active .animate-in:nth-child(3) { transition-delay: 0.4s; }
.slide.active .animate-in:nth-child(4) { transition-delay: 0.55s; }
.slide.active .animate-in:nth-child(5) { transition-delay: 0.7s; }
.slide.active .animate-in:nth-child(6) { transition-delay: 0.85s; }
.slide.active .animate-in:nth-child(7) { transition-delay: 1.0s; }
.slide.active .animate-in:nth-child(8) { transition-delay: 1.15s; }
```

## Presentation Engine (JavaScript)

This is the core navigation logic. Include it in a `<script>` tag at the end of the body.

### Navigation Core

```javascript
const slides = document.querySelectorAll('.slide');
const totalSlides = slides.length;
let current = 0;

function goTo(n) {
  if (n < 0 || n >= totalSlides) return;
  const direction = n > current ? 'left' : 'right';

  slides.forEach(s => s.classList.remove('active', 'exit-left', 'exit-right'));
  slides[current].classList.add(direction === 'left' ? 'exit-left' : 'exit-right');

  current = n;
  slides[current].classList.add('active');

  // Update progress bar
  document.getElementById('progressFill').style.width =
    ((current / (totalSlides - 1)) * 100) + '%';

  // Update slide counter
  document.getElementById('slideCounter').textContent =
    (current + 1) + ' / ' + totalSlides;

  // Hide nav hint after first navigation
  document.getElementById('navHint').classList.add('hidden');

  // Trigger per-slide animations
  triggerAnimations(current);
}
```

### Keyboard Navigation

```javascript
document.addEventListener('keydown', (e) => {
  // If jump overlay is open, handle input there (see below)
  if (jumpOverlay.classList.contains('visible')) {
    e.preventDefault();
    if (e.key === 'Escape') { hideJump(); return; }
    if (e.key === 'Enter') { commitJump(); return; }
    if (e.key === 'Backspace') {
      jumpBuffer = jumpBuffer.slice(0, -1);
      jumpInput.textContent = jumpBuffer || '_';
      return;
    }
    if (e.key >= '0' && e.key <= '9') {
      jumpBuffer += e.key;
      jumpInput.textContent = jumpBuffer;
      if (jumpTimeout) clearTimeout(jumpTimeout);
      jumpTimeout = setTimeout(commitJump, 1200);
    }
    return;
  }

  if (e.key === 'ArrowRight' || e.key === ' ' || e.key === 'Enter') {
    e.preventDefault(); goTo(current + 1);
  }
  if (e.key === 'ArrowLeft' || e.key === 'Backspace') {
    e.preventDefault(); goTo(current - 1);
  }
  if (e.key === 'Home') { e.preventDefault(); goTo(0); }
  if (e.key === 'End') { e.preventDefault(); goTo(totalSlides - 1); }
  if (e.key === 'g' || e.key === 'G') { e.preventDefault(); showJump(); }
});
```

**Supported keys:**
| Key | Action |
|-----|--------|
| Arrow Right / Space / Enter | Next slide |
| Arrow Left / Backspace | Previous slide |
| Home | First slide |
| End | Last slide |
| G | Jump to slide overlay |
| P | Toggle practice mode panel |
| O | Slide overview grid (thumbnail view of all slides) |

### Click Navigation

```javascript
document.addEventListener('click', (e) => {
  if (e.target.closest('a')) return;           // Don't hijack links
  if (e.target.closest('.jump-overlay')) return;
  if (e.target.closest('.slide-counter')) return;
  if (e.target.closest('.run-btn')) return;     // Don't hijack demo buttons
  if (e.target.closest('.demo-container')) return;
  if (e.target.closest('.compare-grid')) return;
  if (e.target.closest('button')) return;
  if (e.target.closest('.practice-panel')) return;
  const x = e.clientX;
  if (x < window.innerWidth / 3) goTo(current - 1);
  else goTo(current + 1);
});
```

Left third of screen = back, rest = forward. Interactive elements (buttons, demo areas, practice panel) are excluded from click navigation.

### Jump-to-Slide Overlay

A modal that appears on `G` key, lets user type a slide number:

```javascript
let jumpBuffer = '';
let jumpTimeout = null;
const jumpOverlay = document.getElementById('jumpOverlay');
const jumpInput = document.getElementById('jumpInput');

function showJump() {
  jumpBuffer = '';
  jumpInput.textContent = '_';
  jumpOverlay.classList.add('visible');
}
function hideJump() {
  jumpOverlay.classList.remove('visible');
  jumpBuffer = '';
  if (jumpTimeout) clearTimeout(jumpTimeout);
}
function commitJump() {
  const num = parseInt(jumpBuffer);
  if (num >= 1 && num <= totalSlides) goTo(num - 1);
  hideJump();
}
```

### Counter Animations

For stat slides — numbers animate from 0 to target:

```javascript
function animateCounter(el) {
  const target = parseFloat(el.dataset.target);
  const prefix = el.dataset.prefix || '';
  const suffix = el.dataset.suffix || '';
  const decimals = parseInt(el.dataset.decimals || '0');
  const duration = 2000;
  const start = performance.now();

  function update(now) {
    const elapsed = now - start;
    const progress = Math.min(elapsed / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 4);  // ease-out quart
    const value = eased * target;
    el.textContent = prefix + (decimals > 0
      ? value.toFixed(decimals)
      : Math.round(value).toLocaleString()) + suffix;
    if (progress < 1) requestAnimationFrame(update);
  }
  requestAnimationFrame(update);
}
```

Usage in HTML:
```html
<div class="counter" data-target="1.8" data-prefix="$" data-suffix="M ARR" data-decimals="1">$0</div>
```

### Per-Slide Animation Triggers

```javascript
function triggerAnimations(slideIndex) {
  const slide = slides[slideIndex];

  // Counter animations
  slide.querySelectorAll('.counter').forEach((el, i) => {
    setTimeout(() => animateCounter(el), i * 300);
  });

  // Team member stagger (if present)
  slide.querySelectorAll('.team-member').forEach((member, i) => {
    member.style.transitionDelay = (0.3 + i * 0.06) + 's';
  });

  // Timeline animation (if present)
  const timeline = slide.querySelector('[data-timeline]');
  if (timeline) {
    const fill = timeline.querySelector('.h-timeline-fill');
    const items = timeline.querySelectorAll('.h-timeline-item');
    if (fill) fill.style.width = '0';
    items.forEach(item => item.classList.remove('visible'));
    requestAnimationFrame(() => {
      if (fill) fill.style.width = '100%';
      items.forEach((item, i) => {
        setTimeout(() => item.classList.add('visible'), 300 + i * 350);
      });
    });
  }
}
```

### Initialize

```javascript
slides[0].classList.add('active');
document.getElementById('slideCounter').textContent = 0;
```

Note: Slide counter starts at 0 (for the navigation/info slide). The `goTo()` function uses `current` directly as the display number.

## UI Chrome (Progress Bar, Counter, Footer, Nav Hint)

Always include these elements after the `.deck` div:

```html
<!-- Progress Bar -->
<div class="progress-bar"><div class="progress-fill" id="progressFill"></div></div>

<!-- Footer (logo or event branding, bottom-left) -->
<div class="slide-footer">
  <img src="images/logo.png" alt="Event Logo" class="slide-footer-logo">
</div>

<!-- Slide Counter (blue circle, bottom-right) -->
<div class="slide-counter" id="slideCounter" onclick="showJump()"></div>

<!-- Navigation Hint (fades after first nav) -->
<div class="nav-hint" id="navHint">Arrow keys to navigate · G to jump · P to practice</div>
```

```css
.progress-bar {
  position: fixed; bottom: 0; left: 0; right: 0;
  height: 2px; background: transparent; z-index: 100;
}
.progress-fill {
  height: 100%; width: 0;
  background: var(--accent);
  opacity: 0.4;
  transition: width 0.4s ease;
}

/* Blue circle slide counter */
.slide-counter {
  position: fixed; bottom: 24px; right: 32px;
  width: 40px; height: 40px;
  border-radius: 50%;
  background: var(--accent);
  color: white;
  font-family: var(--font-body); font-size: 14px; font-weight: 600;
  display: flex; align-items: center; justify-content: center;
  z-index: 100;
  cursor: pointer;
  transition: right 0.35s ease;
}
body.practice-open .slide-counter {
  right: 452px;
}

/* Persistent footer */
.slide-footer {
  position: fixed; bottom: 24px; left: 32px;
  z-index: 100;
}
.slide-footer-logo {
  height: 56px;
  width: auto;
}

.nav-hint {
  position: fixed; bottom: 70px; left: 50%;
  transform: translateX(-50%);
  font-family: var(--font-mono); font-size: 13px;
  color: var(--text-muted); z-index: 100;
  transition: opacity 0.5s;
}
.nav-hint.hidden { opacity: 0; pointer-events: none; }
```

The slide counter is clickable — opens the jump overlay.

## Practice Mode

A slide-in panel from the right that shows per-slide talk track notes. Toggled with the P key. The panel **pushes** the deck content rather than overlaying it.

### Practice Panel HTML

Include after the UI chrome elements:

```html
<!-- Practice Panel -->
<div class="practice-panel" id="practicePanel">
  <div class="practice-header">
    <span class="practice-header-title">Talk Track</span>
    <button class="practice-close" onclick="togglePractice()">&times;</button>
  </div>
  <div class="practice-body" id="practiceBody"></div>
  <div class="practice-hint">Press P to toggle</div>
</div>
```

### Practice Panel CSS

```css
.practice-panel {
  position: fixed;
  top: 0; right: 0; bottom: 0;
  width: 420px;
  background: rgba(0, 0, 0, 0.85);
  backdrop-filter: blur(16px);
  border-left: 1px solid rgba(255, 255, 255, 0.1);
  z-index: 150;
  transform: translateX(100%);
  transition: transform 0.35s ease;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
.practice-panel.open { transform: translateX(0); }

.practice-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 20px 24px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  flex-shrink: 0;
}
.practice-header-title {
  font-family: var(--font-mono); font-size: 12px;
  text-transform: uppercase; letter-spacing: 2px; color: var(--text-muted);
}
.practice-close {
  background: none; border: none; color: var(--text-muted);
  font-size: 20px; cursor: pointer; padding: 4px 8px; border-radius: 4px;
}
.practice-close:hover { color: var(--text-primary); background: rgba(255,255,255,0.08); }

.practice-body {
  padding: 24px; overflow-y: auto; flex: 1;
  font-family: var(--font-body); font-size: 15px;
  line-height: 1.8; color: var(--text-secondary);
}
.practice-body strong { color: var(--text-primary); font-weight: 600; }

/* Yellow highlighter for key phrases to emphasize */
.practice-body em {
  background: rgba(245, 166, 35, 0.25);
  color: #ffd580;
  font-style: normal;
  padding: 1px 4px;
  border-radius: 3px;
}

/* Stage notes — self-reminders, not talk track */
.practice-body .stage-note {
  display: block;
  margin: 12px 0;
  padding: 8px 12px;
  border-left: 3px solid #f5a623;
  background: rgba(245, 166, 35, 0.08);
  border-radius: 0 6px 6px 0;
  font-size: 13px;
  color: #f5a623;
  font-style: italic;
}

.practice-slide-label {
  font-family: var(--font-mono); font-size: 11px;
  text-transform: uppercase; letter-spacing: 1.5px;
  color: var(--accent-light); margin-bottom: 16px;
}
.practice-hint {
  padding: 16px 24px;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  font-family: var(--font-mono); font-size: 11px;
  color: var(--text-muted); flex-shrink: 0; text-align: center;
}
```

### Practice Panel JavaScript

```javascript
// Talk track data — one entry per slide index
const talkTrack = {
  0: `<span class="stage-note">Navigation slide. Don't linger.</span>`,
  1: `<span class="stage-note">Let the audience settle.</span>"Welcome everyone..."`,
  2: `"Here's the key insight: <em>this changes everything</em>."<span class="stage-note">Pause here. Let the quote breathe.</span>`,
  // ... one entry per slide
};

const practicePanel = document.getElementById('practicePanel');
let practiceOpen = false;

function togglePractice() {
  practiceOpen = !practiceOpen;
  practicePanel.classList.toggle('open', practiceOpen);
  document.body.classList.toggle('practice-open', practiceOpen);
  if (practiceOpen) updatePractice();
}

function updatePractice() {
  if (!practiceOpen) return;
  const body = document.getElementById('practiceBody');
  const track = talkTrack[current];
  body.innerHTML = '<div class="practice-slide-label">Slide ' + current + '</div>'
    + (track || '<span style="color: var(--text-muted);">No talk track for this slide.</span>');
}
```

Add `updatePractice()` call inside the `goTo()` function, and add `P` key binding in the keyboard handler.

### Talk Track Formatting Guide

Each talk track entry should be broken into **scannable bullets** using `<div class="talk-line">` — one spoken beat per line. Stage notes go between the bullets as block callouts. This makes it easy to glance at during practice without reading walls of text.

**Elements:**

| Element | Usage | Renders as |
|---------|-------|------------|
| `<div class="talk-line">text</div>` | One spoken beat/sentence | Bulleted line with › marker |
| `<em>text</em>` | Key phrases to emphasize on stage | Blue highlighted inline text |
| `<strong>text</strong>` | Section labels | Bold white |
| `<span class="stage-note">text</span>` | Self-reminders (don't read aloud) | Amber callout block with left border |

**Example:**
```javascript
9: `<div class="talk-line">Then people start using your thing. Traffic is real now.</div>
<div class="talk-line">So you move to Dedicated.</div>
<span class="stage-note">Point at the curl differences.</span>
<div class="talk-line">And here's the key — it's literally <em>two lines</em>. New endpoint URL, new access token.</div>
<div class="talk-line">Everything else — completely identical.</div>`
```

**CSS for talk-line bullets:**
```css
.practice-body .talk-line {
  margin-bottom: 12px;
  padding-left: 20px;
  position: relative;
}
.practice-body .talk-line::before {
  content: '›';
  color: var(--accent-light);
  font-weight: 700;
  font-size: 18px;
  position: absolute;
  left: 0;
  top: 0;
  line-height: 1.5;
}
```

**Important CSS notes:**
- Use `display: inline` on `.practice-body em` — without it, `<em>` inside flex/block containers can break text flow
- Add `box-decoration-break: clone` so highlights wrap correctly across lines
- Use blue accent (`rgba(0, 105, 255, 0.2)`) for emphasis highlights, NOT yellow — yellow conflicts visually with the amber stage notes
- Practice mode should be **on by default** — initialize with `practiceOpen = true` and add `.open` and `.practice-open` classes on load

**Enabling practice mode by default:**
```javascript
const practicePanel = document.getElementById('practicePanel');
let practiceOpen = true;
practicePanel.classList.add('open');
document.body.classList.add('practice-open');
// ... later in init:
updatePractice();
```

## Jump-to-Slide Overlay HTML

```html
<div class="jump-overlay" id="jumpOverlay">
  <div class="jump-box">
    <div class="jump-label">Go to slide</div>
    <div class="jump-number" id="jumpInput">_</div>
    <div class="jump-hint">Type a number, then Enter · Esc to cancel</div>
  </div>
</div>
```

```css
.jump-overlay {
  position: fixed; inset: 0;
  background: rgba(0,0,0,0.7);
  backdrop-filter: blur(8px);
  display: flex; align-items: center; justify-content: center;
  opacity: 0; pointer-events: none;
  transition: opacity 0.3s; z-index: 200;
}
.jump-overlay.visible { opacity: 1; pointer-events: auto; }

.jump-box {
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 48px 64px;
  text-align: center;
}
.jump-label {
  font-family: var(--font-mono); font-size: 14px;
  text-transform: uppercase; letter-spacing: 2px;
  color: var(--text-muted); margin-bottom: 16px;
}
.jump-number {
  font-family: var(--font-mono); font-size: 72px;
  font-weight: 700; color: var(--accent-light);
}
.jump-hint {
  font-size: 13px; color: var(--text-muted);
  margin-top: 16px; font-family: var(--font-mono);
}
```

## Slide Overview Grid

A fullscreen overlay showing thumbnail previews of all slides, accessible via the `O` key. Each thumbnail is a scaled-down clone of the actual slide DOM, so it always reflects current content.

### Overview HTML

```html
<div class="overview-overlay" id="overviewOverlay">
  <div class="overview-header">
    <div class="overview-title">All Slides</div>
    <button class="overview-close" onclick="hideOverview()">&times;</button>
  </div>
  <div class="overview-grid" id="overviewGrid"></div>
</div>
```

### Overview CSS

```css
.overview-overlay {
  position: fixed; inset: 0;
  background: rgba(0, 0, 0, 0.92);
  backdrop-filter: blur(12px);
  z-index: 250;
  opacity: 0; pointer-events: none;
  transition: opacity 0.3s ease;
  overflow-y: auto; padding: 40px;
}
.overview-overlay.visible { opacity: 1; pointer-events: auto; }
.overview-header {
  display: flex; align-items: center; justify-content: space-between;
  margin-bottom: 24px;
}
.overview-title { font-family: var(--font-display); font-size: 24px; font-weight: 700; }
.overview-close {
  background: none; border: none; color: var(--text-muted);
  font-size: 28px; cursor: pointer; padding: 4px 12px; border-radius: 6px;
}
.overview-close:hover { color: var(--text-primary); background: rgba(255,255,255,0.08); }
.overview-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 16px;
}
.overview-thumb {
  background: rgba(255, 255, 255, 0.04);
  border: 2px solid transparent;
  border-radius: 10px; cursor: pointer;
  transition: border-color 0.2s, background 0.2s;
  overflow: hidden;
}
.overview-thumb:hover { border-color: var(--accent); background: rgba(0, 105, 255, 0.08); }
.overview-thumb.current { border-color: var(--accent); background: rgba(0, 105, 255, 0.12); }
.overview-thumb-preview {
  width: 100%; aspect-ratio: 16/9;
  overflow: hidden; position: relative;
  background: var(--bg-deep);
  border-radius: 8px 8px 0 0;
}
.overview-thumb-preview .slide-mini {
  width: 1280px; height: 720px;
  transform: scale(0.172); transform-origin: top left;
  pointer-events: none; position: absolute; top: 0; left: 0;
}
.overview-thumb-info { padding: 12px 14px; }
.overview-thumb-num {
  font-family: var(--font-mono); font-size: 12px; color: var(--accent-light); margin-bottom: 4px;
}
.overview-thumb-title {
  font-family: var(--font-body); font-size: 14px; font-weight: 500;
  color: var(--text-secondary); line-height: 1.4;
}
```

### Overview JavaScript

```javascript
const overviewOverlay = document.getElementById('overviewOverlay');

function buildOverview() {
  const grid = document.getElementById('overviewGrid');
  grid.innerHTML = '';
  slides.forEach((slide, i) => {
    const thumb = document.createElement('div');
    thumb.className = 'overview-thumb' + (i === current ? ' current' : '');

    // Clone slide as mini preview
    const preview = document.createElement('div');
    preview.className = 'overview-thumb-preview';
    const miniSlide = slide.cloneNode(true);
    miniSlide.className = 'slide-mini';
    miniSlide.style.cssText = slide.style.cssText;
    miniSlide.querySelectorAll('.animate-in').forEach(el => {
      el.style.opacity = '1'; el.style.transform = 'none';
    });
    miniSlide.querySelectorAll('button, select, a, input').forEach(el => el.remove());
    preview.style.pointerEvents = 'none';
    preview.appendChild(miniSlide);

    // Title from slide content
    const headline = slide.querySelector('.headline, .headline-md, .headline-sm, .big-quote, .title-main');
    const label = slide.querySelector('.label, .step-badge');
    let title = headline ? headline.textContent.trim().substring(0, 50)
              : label ? label.textContent.trim() : 'Slide ' + i;

    const info = document.createElement('div');
    info.className = 'overview-thumb-info';
    info.innerHTML = '<div class="overview-thumb-num">Slide ' + i + '</div>'
      + '<div class="overview-thumb-title">' + title + '</div>';

    thumb.appendChild(preview);
    thumb.appendChild(info);
    const slideIndex = i;
    thumb.addEventListener('click', (e) => { e.stopPropagation(); goTo(slideIndex); hideOverview(); });
    grid.appendChild(thumb);
  });
}

function showOverview() { buildOverview(); overviewOverlay.classList.add('visible'); }
function hideOverview() { overviewOverlay.classList.remove('visible'); }
```

Add `O` key binding in the keyboard handler, and `Escape` to close. Thumbnails are regenerated on every open so they always reflect current slide content.

---

## Reusable Slide Component Patterns

Each component below includes both HTML and CSS. Include the CSS for any components you use.

### Stat Card

```css
.stat-card {
  background: var(--bg-surface);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 40px 48px;
  text-align: center;
}
.stat-label {
  font-family: var(--font-body);
  font-size: 20px;
  color: var(--text-secondary);
  margin-top: 8px;
}
.counter {
  font-family: var(--font-display);
  font-size: 96px;
  font-weight: 800;
  color: var(--accent-light);
  text-shadow: 0 0 40px var(--accent-glow);
  line-height: 1;
}
```

```html
<div class="stat-card animate-in">
  <div class="counter" data-target="56" data-suffix="%">0</div>
  <div class="stat-label">Brand new customers</div>
</div>
```

### Badge / Pill

```css
.badge {
  display: inline-block;
  padding: 8px 20px;
  border-radius: 100px;
  font-family: var(--font-mono);
  font-size: 14px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1px;
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  color: var(--text-secondary);
}
.badge-accent {
  background: rgba(59, 130, 246, 0.1);
  border-color: rgba(59, 130, 246, 0.3);
  color: var(--accent-light);
}
.badge-green {
  background: rgba(34, 197, 94, 0.1);
  border-color: rgba(34, 197, 94, 0.3);
  color: var(--success);
}
```

```html
<span class="badge">Default</span>
<span class="badge badge-accent">Highlighted</span>
<span class="badge badge-green">Success</span>
```

### Image Card (screenshot, blog post, etc.)
```html
<img src="images/screenshot.png" alt="Description"
     style="max-width: 560px; border-radius: 12px;
            border: 1px solid var(--border);
            box-shadow: 0 8px 32px rgba(0,0,0,0.4);">
```

### Tweet / Quote Card

```css
.tweet-card {
  background: var(--bg-surface);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 32px 36px;
  max-width: 700px;
  text-align: left;
}
.tweet-header { display: flex; align-items: center; gap: 12px; margin-bottom: 12px; }
.tweet-avatar {
  width: 48px; height: 48px; border-radius: 50%;
  background: var(--accent); display: flex; align-items: center; justify-content: center;
  font-family: var(--font-display); font-weight: 700; font-size: 16px; color: white;
}
.tweet-name { font-weight: 600; font-size: 20px; }
.tweet-handle { font-size: 17px; color: var(--text-secondary); }
.tweet-body { font-size: 24px; line-height: 1.5; margin-bottom: 16px; }
.tweet-meta { font-size: 16px; color: var(--text-muted); font-family: var(--font-mono); }
.tweet-views { color: var(--text-secondary); margin-left: 12px; }
```

```html
<div class="tweet-card">
  <div class="tweet-header">
    <div class="tweet-avatar">ND</div>
    <div>
      <div class="tweet-name">Nader Dabit</div>
      <div class="tweet-handle">@dabit3</div>
    </div>
  </div>
  <div class="tweet-body">Quote text here...</div>
  <div class="tweet-meta">Jan 25, 2026 <span class="tweet-views">487K views</span></div>
</div>
```

### Takeaway / Bullet List

```css
.takeaway-list {
  display: flex; flex-direction: column; gap: 28px;
  max-width: 700px; text-align: left;
}
.takeaway-item {
  display: flex; align-items: center; gap: 20px;
  font-size: 28px; line-height: 1.4;
}
.takeaway-icon { font-size: 32px; flex-shrink: 0; }
```

```html
<div class="takeaway-list">
  <div class="takeaway-item animate-in">
    <span class="takeaway-icon">icon</span>
    <span><strong class="text-accent">Key point</strong> — detail</span>
  </div>
</div>
```

### Team / People Grid

```css
.team-grid {
  display: flex; flex-wrap: wrap; gap: 24px;
  justify-content: center; max-width: 900px;
}
.team-member {
  display: flex; flex-direction: column; align-items: center; gap: 8px;
  opacity: 0; transform: translateY(16px);
  transition: opacity 0.4s ease, transform 0.4s ease;
}
.slide.active .team-member { opacity: 1; transform: translateY(0); }
.team-photo {
  width: 64px; height: 64px; border-radius: 50%;
  object-fit: cover; border: 2px solid var(--border);
}
.team-name { font-size: 13px; color: var(--text-secondary); }
```

```html
<div class="team-grid">
  <div class="team-member animate-in">
    <img src="people/name.jpg" class="team-photo" alt="Name">
    <div class="team-name">First Last</div>
  </div>
</div>
```

### Horizontal Timeline

```css
.h-timeline { position: relative; padding: 40px 0; width: 100%; max-width: 900px; }
.h-timeline-track {
  position: absolute; top: 50%; left: 0; right: 0;
  height: 3px; background: var(--border);
  transform: translateY(-50%);
}
.h-timeline-fill {
  height: 100%; width: 0; background: var(--accent);
  transition: width 2s ease;
}
.h-timeline-item {
  position: relative; text-align: center;
  opacity: 0; transform: translateY(10px);
  transition: opacity 0.4s ease, transform 0.4s ease;
}
.h-timeline-item.visible { opacity: 1; transform: translateY(0); }
.h-timeline-date {
  font-family: var(--font-mono); font-size: 14px;
  font-weight: 700; color: var(--accent-light);
}
.h-timeline-label {
  font-size: 13px; color: var(--text-secondary);
  margin-top: 4px; max-width: 120px;
}
```

```html
<div class="h-timeline" data-timeline>
  <div class="h-timeline-track">
    <div class="h-timeline-fill"></div>
  </div>
  <div class="h-timeline-item">
    <div class="h-timeline-date">Jan 25</div>
    <div class="h-timeline-label">Event description</div>
  </div>
  <!-- more items -->
</div>
```

### Code Highlighter Block

Use a yellow highlighter effect to call attention to new/important lines in code blocks:

```css
.code-highlight-block {
  background: rgba(245, 166, 35, 0.2);
  border-left: 3px solid #f5a623;
  padding: 4px 8px;
  margin: 2px -8px;
  display: inline-block;
  border-radius: 0 6px 6px 0;
}
```

```html
<div class="demo-code">
  <span class="code-key">"model"</span>: <span class="code-string">"gpt-4o"</span>,
  <span class="code-highlight-block"><span class="code-key">"tools"</span>: [{ <span class="code-key">"type"</span>: <span class="code-string">"web_search"</span> }]</span>
</div>
```

## Speaker Script Format

With practice mode built in, a separate `script.md` is optional. The talk track lives in the `talkTrack` JS object. But if the user wants a standalone script file, use this format:

```markdown
**SLIDE 1 — Title: Slide Title Here**

> Speaker narration goes here. Written in first person, conversational tone.
> Matches the audience and presentation style.

---

**SLIDE 2 — Next Slide Title**

> More narration...
```

## Deployment (Optional Flask Wrapper)

For password-protected hosting on DigitalOcean App Platform:

**`app.py`** — Flask app that serves a login page, then `index.html` once authenticated. Serves `images/` directory behind the same auth check. Can also include API proxy endpoints for live demos.

**`requirements.txt`:**
```
flask
gunicorn
requests
```

**`Dockerfile`:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["gunicorn", "-b", "0.0.0.0:8080", "app:app"]
```

Set `APP_PASSWORD` and `SECRET_KEY` as environment variables in the hosting platform.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `overflow: hidden` on `html, body` | Causes scroll bleed between slides |
| Missing `pointer-events: none` on inactive slides | Click events leak to wrong slide |
| No `e.preventDefault()` on Space/Enter keys | Page scrolls or form submits |
| Animations not resetting on revisit | Use `triggerAnimations()` to reset state each time |
| Counter shows NaN | Ensure `data-target` is set and is a valid number |
| Images not loading when deployed | Make sure Flask wrapper serves all asset directories |
| Buttons/interactive elements trigger slide navigation | Exclude `.run-btn`, `.demo-container`, `button`, `.practice-panel` from click handler |
| Practice panel overlays content | Use `body.practice-open .deck { width: calc(100vw - 420px); }` to push content |
| Slide counter shows "1 / N" format in circle badge | Use just the number: `slideCounter.textContent = current;` |
| Slide 0 counter shows 1 instead of 0 | Initialize with `current` not `current + 1` |

## Sizing for Screen Sharing

When presentations will be screen-shared (Zoom, Meet), scale everything up ~20%. Headlines should be 64px+ minimum, body text 20px+, labels 18px+. Test at the resolution you'll actually present at.
