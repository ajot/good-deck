# Practice Mode

A slide-in panel from the right that shows per-slide talk track notes. Toggled with the P key. The panel **pushes** the deck content rather than overlaying it.

## Practice Panel HTML

Include after the UI chrome elements:

```html
<div class="practice-panel" id="practicePanel">
  <div class="practice-header">
    <span class="practice-header-title">Talk Track</span>
    <button class="practice-close" onclick="togglePractice()">&times;</button>
  </div>
  <div class="practice-body" id="practiceBody"></div>
  <div class="practice-next-preview">
    <div class="practice-next-label">Up Next</div>
    <div class="practice-next-container" id="practiceNextPreview"></div>
  </div>
  <div class="practice-hint">Press P to toggle</div>
</div>
```

## Practice Panel CSS

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
  background: rgba(0, 105, 255, 0.2);
  color: var(--accent-light);
  font-style: normal;
  padding: 1px 4px;
  border-radius: 3px;
  display: inline;
  box-decoration-break: clone;
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

/* Talk-line bullets */
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

/* Next slide preview */
.practice-next-preview {
  padding: 16px 24px 8px;
  border-top: 1px solid rgba(255, 255, 255, 0.06);
  flex-shrink: 0;
}
.practice-next-label {
  font-family: var(--font-mono); font-size: 11px;
  text-transform: uppercase; letter-spacing: 1.5px;
  color: var(--text-muted); margin-bottom: 10px;
}
.practice-next-container {
  width: 100%; aspect-ratio: 16/9;
  overflow: hidden; position: relative;
  background:
    radial-gradient(ellipse 70% 55% at 80% 10%, rgba(0,105,255,0.08) 0%, transparent 60%),
    linear-gradient(180deg, var(--bg-deep, #1a1a2e) 0%, #16213e 100%);
  border-radius: 6px;
  border: 1px solid rgba(255, 255, 255, 0.06);
}
.practice-next-container .slide-mini {
  width: 1280px; height: 720px;
  transform: scale(0.27);
  transform-origin: top left;
  pointer-events: none;
  position: absolute; top: 0; left: 0;
  display: flex; flex-direction: column;
  padding: 48px 60px;
}
.practice-next-end {
  display: flex; align-items: center; justify-content: center;
  height: 100%;
  font-family: var(--font-mono); font-size: 12px;
  color: var(--text-muted); letter-spacing: 1px;
}
```

## Practice Panel JavaScript

```javascript
const talkTrack = {
  0: `<span class="stage-note">Navigation slide. Don't linger.</span>`,
  1: `<span class="stage-note">Let the audience settle.</span>
<div class="talk-line">"Welcome everyone..."</div>`,
  2: `<div class="talk-line">"Here's the key insight: <em>this changes everything</em>."</div>
<span class="stage-note">Pause here. Let the quote breathe.</span>`,
  // ... one entry per slide
};

const practicePanel = document.getElementById('practicePanel');
let practiceOpen = true; // on by default

// Initialize open
practicePanel.classList.add('open');
document.body.classList.add('practice-open');

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

  // Next slide preview
  const container = document.getElementById('practiceNextPreview');
  container.innerHTML = '';
  const nextClone = cloneSlidePreview(current + 1);
  if (nextClone) {
    container.appendChild(nextClone);
  } else {
    container.innerHTML = '<div class="practice-next-end">End of deck</div>';
  }
}
```

The `cloneSlidePreview()` function is defined in the presenter view reference — see `presenter-view.md` → "Next Slide Preview" → "Shared Clone Function". It must be defined before `updatePractice()` is called.

Add `updatePractice()` call inside the `goTo()` function, and add `P` key binding in the keyboard handler.

## Talk Track Formatting Guide

Each talk track entry should be broken into **scannable bullets** using `<div class="talk-line">` — one spoken beat per line. Stage notes go between the bullets as block callouts.

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

## Important CSS Notes

- Use `display: inline` on `.practice-body em` — without it, `<em>` inside flex/block containers can break text flow
- Add `box-decoration-break: clone` so highlights wrap correctly across lines
- Use blue accent (`rgba(0, 105, 255, 0.2)`) for emphasis highlights, NOT yellow — yellow conflicts visually with the amber stage notes
- Practice mode is **on by default** — initialize with `practiceOpen = true`
