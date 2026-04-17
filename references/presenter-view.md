# Presenter View

A popup window that shows speaker notes on your laptop while the audience sees the clean deck on the projector. Opened with Shift+P or a button on slide 0. Stays synced with the main deck via direct JS calls.

## How It Works

1. **Shift+P** opens a `window.open()` popup positioned on the right side of the screen
2. The main deck's `goTo()` function calls `presenterWindow.updateSlide(n)` on every slide change
3. The popup reads `talkTrack` and `totalSlides` from the parent window via `window.opener`
4. Opening the presenter view auto-disables practice mode (P key) since you don't want notes on both screens
5. Closing the popup re-enables practice mode

## Integration

### Variables

Add these after your existing slide variables (`current`, `totalSlides`, etc.):

```javascript
// Presenter view state
let presenterWindow = null;
let presenterActive = false;

// Expose for presenter popup (const/let are NOT window properties)
window.totalSlides = totalSlides;
```

And after the `talkTrack` object definition:

```javascript
window.talkTrack = talkTrack;
```

### The `openPresenter()` Function

Add before the keyboard handler:

```javascript
function openPresenter() {
  if (presenterWindow && !presenterWindow.closed) {
    presenterWindow.focus();
    return;
  }

  const w = 520;
  const h = window.screen.height;
  const left = window.screen.width - w;
  presenterWindow = window.open('', 'presenterView',
    `width=${w},height=${h},left=${left},top=0,menubar=no,toolbar=no,location=no,status=no`
  );

  const doc = presenterWindow.document;
  doc.open();
  doc.write(`<!DOCTYPE html>
<html><head><title>Presenter View</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: var(--bg-deep, #1a1a2e);
    color: #e0e0e0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    height: 100vh;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }
  .presenter-header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 16px 24px;
    border-bottom: 1px solid rgba(255,255,255,0.08);
    flex-shrink: 0;
  }
  .presenter-label {
    font-family: monospace; font-size: 11px;
    text-transform: uppercase; letter-spacing: 2px;
    color: rgba(255,255,255,0.4);
  }
  .presenter-timer {
    font-family: monospace; font-size: 18px;
    color: var(--accent, #0069FF); letter-spacing: 1px;
  }
  .presenter-slide-info {
    padding: 20px 24px 16px;
    border-bottom: 1px solid rgba(255,255,255,0.06);
    flex-shrink: 0;
  }
  .presenter-slide-number {
    font-size: 36px; font-weight: 800; color: #fff;
  }
  .presenter-slide-total {
    font-family: monospace; font-size: 14px;
    color: rgba(255,255,255,0.35); margin-left: 4px;
  }
  .presenter-progress {
    height: 3px; background: rgba(255,255,255,0.06);
    border-radius: 2px; margin-top: 12px; overflow: hidden;
  }
  .presenter-progress-fill {
    height: 100%; background: var(--accent, #0069FF);
    border-radius: 2px; transition: width 0.3s ease; width: 0%;
  }
  .presenter-body {
    padding: 24px; overflow-y: auto; flex: 1;
    font-size: 15px; line-height: 1.7;
    color: rgba(255,255,255,0.7);
  }
  .presenter-body .talk-line {
    margin-bottom: 12px; padding-left: 20px; position: relative;
  }
  .presenter-body .talk-line::before {
    content: '›'; color: var(--accent-light, #4d9aff);
    font-weight: 700; font-size: 18px;
    position: absolute; left: 0; top: 0; line-height: 1.5;
  }
  .presenter-body strong { color: #fff; font-weight: 600; }
  .presenter-body em {
    background: rgba(0,105,255,0.2); color: var(--accent-light, #4d9aff);
    font-style: normal; padding: 1px 4px; border-radius: 3px;
  }
  .presenter-body .stage-note {
    display: block; margin: 12px 0; padding: 8px 12px;
    border-left: 3px solid #f5a623;
    background: rgba(245,166,35,0.08);
    border-radius: 0 6px 6px 0;
    font-family: monospace; font-size: 12px;
    text-transform: uppercase; letter-spacing: 1.5px; color: #f5a623;
  }
  .presenter-footer {
    padding: 12px 24px;
    border-top: 1px solid rgba(255,255,255,0.08);
    font-family: monospace; font-size: 11px;
    color: rgba(255,255,255,0.25); text-align: center;
    flex-shrink: 0;
  }
</style>
</head><body>
  <div class="presenter-header">
    <span class="presenter-label">Presenter View</span>
    <span class="presenter-timer" id="timer">00:00</span>
  </div>
  <div class="presenter-slide-info">
    <span class="presenter-slide-number" id="slideNum">0</span>
    <span class="presenter-slide-total" id="slideTotal">/ 0</span>
    <div class="presenter-progress"><div class="presenter-progress-fill" id="progressFill"></div></div>
  </div>
  <div class="presenter-body" id="talkBody">
    <span style="color: rgba(255,255,255,0.3);">Waiting for slide navigation...</span>
  </div>
  <div class="presenter-footer">Navigate from main window · Shift+P to close</div>
</body></html>`);
  doc.close();

  // Inject script via DOM to avoid nested script tag issues
  const presenterScript = presenterWindow.document.createElement('script');
  presenterScript.textContent = `
    var opener = window.opener;
    var timerInterval = null;

    function updateSlide(n) {
      var total = opener ? opener.totalSlides : 0;
      document.getElementById('slideNum').textContent = n;
      document.getElementById('slideTotal').textContent = '/ ' + (total - 1);
      document.getElementById('progressFill').style.width =
        total > 1 ? ((n / (total - 1)) * 100) + '%' : '0%';

      var track = opener.talkTrack[n];
      document.getElementById('talkBody').innerHTML = track ||
        '<span style="color: rgba(255,255,255,0.3);">No talk track for this slide.</span>';

      if (n > 0 && !timerInterval) {
        var start = Date.now();
        timerInterval = setInterval(function() {
          var elapsed = Math.floor((Date.now() - start) / 1000);
          var m = String(Math.floor(elapsed / 60)).padStart(2, '0');
          var s = String(elapsed % 60).padStart(2, '0');
          document.getElementById('timer').textContent = m + ':' + s;
        }, 1000);
      }
    }
  `;
  presenterWindow.document.body.appendChild(presenterScript);

  // Initial sync
  try { presenterWindow.updateSlide(current); } catch(e) {}

  // Activate presenter mode, close practice panel
  presenterActive = true;
  if (practiceOpen) {
    practiceOpen = false;
    practicePanel.classList.remove('open');
    document.body.classList.remove('practice-open');
  }
}
```

### Sync in `goTo()`

Add after the `localStorage.setItem` line in `goTo()`:

```javascript
// Sync presenter view
if (presenterWindow && !presenterWindow.closed) {
  try { presenterWindow.updateSlide(current); } catch(e) {}
} else if (presenterActive) {
  presenterActive = false;
}
```

### Keyboard Shortcut

Replace the P key handler:

```javascript
// Before (practice only):
if (e.key === 'p' || e.key === 'P') { e.preventDefault(); togglePractice(); }

// After (Shift+P for presenter, P for practice):
if (e.key === 'P' && e.shiftKey) { e.preventDefault(); openPresenter(); return; }
if ((e.key === 'p' || e.key === 'P') && !e.shiftKey) { e.preventDefault(); if (!presenterActive) togglePractice(); }
```

### Slide 0 Button

Add inside the navigation hints div on slide 0. This avoids popup blocker issues since button clicks always allow popups:

```html
<br>
<button onclick="openPresenter()" style="
  margin-top: 20px;
  background: rgba(var(--accent-rgb, 0,105,255), 0.15);
  border: 1px solid rgba(var(--accent-rgb, 0,105,255), 0.3);
  color: var(--accent, #0069FF);
  font-family: var(--font-mono);
  font-size: 13px;
  padding: 10px 24px;
  border-radius: 6px;
  cursor: pointer;
  letter-spacing: 1px;
">OPEN PRESENTER VIEW</button>
```

## Critical Implementation Notes

### 1. Never write `</script>` inside a script block

The HTML parser scans for `</script>` to close the current `<script>` element — even inside JS strings, template literals, or comments. This will break silently:

```javascript
// BAD — this comment contains the closing tag and breaks the page:
// Inject script to avoid </script> issues

// BAD — template literal with script tags:
doc.write(`<script>code here</script>`);  // outer script block ends early

// GOOD — inject script via DOM instead:
const s = document.createElement('script');
s.textContent = `your code here`;
document.body.appendChild(s);
```

### 2. Expose variables for cross-window access

`const` and `let` declarations are NOT properties of `window`. A popup accessing `window.opener.myVar` will get `undefined` if `myVar` was declared with `const` or `let`.

```javascript
// BAD — popup can't access these:
const totalSlides = slides.length;
const talkTrack = { ... };

// GOOD — explicitly expose on window:
const totalSlides = slides.length;
window.totalSlides = totalSlides;

const talkTrack = { ... };
window.talkTrack = talkTrack;
```

### 3. Inject popup script after `doc.close()`

Scripts written inside `doc.write()` run before the document is fully built. Instead, create a `<script>` element via DOM and append it after `doc.close()`. This ensures the popup's DOM is ready and avoids the nested `</script>` problem entirely.

## Timer

- Starts on first slide advance past slide 0 (not on page load)
- Displays `MM:SS` format
- No pause/stop/reset controls — reloading the deck resets it
- Sufficient for talks up to 99 minutes

## Presenter Setup Workflow

1. Open the deck in your browser
2. Press **Shift+P** or click the button on slide 0
3. Drag the popup to your laptop screen, keep the deck on the projector
4. Navigate with arrow keys in the main deck — the popup follows
5. If you close the popup accidentally, press Shift+P to reopen (syncs instantly)
