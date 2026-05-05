# What's New

Read this file when the user asks to "apply latest changes" or "update to the latest skill version" on an existing deck. Each version lists what changed and how to migrate.

---

## v2.2 — May 5, 2026

### API Fallback / Canned Mode

New feature — pre-captured fixtures replay automatically when live API calls fail, time out, or return empty content. `Shift+C` forces fixture replay manually (useful when WiFi is flaky on stage). A subtle 2px green dot next to the version number indicates canned mode is active. See `references/api-fallback.md`.

**To migrate an existing deck:**
1. Add the FIXTURES + CANNED MODE block (`fixtures` + `cannedMode` + `toggleCannedMode` + `updateCannedBadge` + `callWithFallback`) — see `references/api-fallback.md` → "State and Toggle"
2. Add the `<span id="cannedBadge">` indicator inside your footer
3. Add the `Shift+C` keyboard handler
4. Wrap each demo's live call with `callWithFallback(apiCall, fixture, timeoutMs)`
5. Add the `/api/fixtures` Flask endpoint (or equivalent on your backend)
6. Create `capture_fixtures.py` to pre-capture every demo response — see "Capture Script" section
7. Run the capture script and ship `fixtures.json` with the deck

### Presenter View Auto-Recovery + `Shift+R` Force-Resync

Presenter popup now auto-recovers when its script context gets lost (which can happen after browser tab-switches or sleep). The presenter script is extracted into a reusable `injectPresenterScript()` function that `goTo()` re-calls automatically on sync failure. `Shift+R` is a manual force-resync escape hatch with a green flash on the popup header to confirm.

**To migrate an existing deck:**
1. Refactor your inline script-injection in `openPresenter()` into a separate `injectPresenterScript()` function (see `references/presenter-view.md` → "The `openPresenter()` Function")
2. Persist the timer interval on `window._timerInterval` so re-injection doesn't start a duplicate timer
3. Wrap the `presenterWindow.updateSlide(current)` call in `goTo()` with try/catch and call `injectPresenterScript()` on failure
4. Add the `Shift+R` keyboard handler

### Speaker Badges (Multi-Presenter Decks)

New optional pattern — define a `speakerMap` once and both the practice panel and presenter popup show colored speaker labels. The presenter popup also shows a transition indicator on slides right before a handoff. Skip this if your deck has a single presenter.

**To migrate:**
1. Define `speakerMap` after `window.talkTrack = talkTrack;` and expose with `window.speakerMap = speakerMap;`
2. Add the speaker badge `<span>` and transition indicator `<div>` to the presenter popup HTML
3. Update the `injectPresenterScript()` body with the badge + transition rendering
4. (Optional) Add the `.speaker-label` CSS and update `updatePractice()` for the practice-panel label

See `references/presenter-view.md` → "Speaker Badges in Header" and `references/practice-mode.md` → "Speaker Label".

### Subtler Stage Notes in Presenter Popup

Stage-note styling in the presenter popup tightened up — smaller, dimmer, less visually competitive with the actual talk track. The practice-panel styling (which you read up close) stays bold; only the presenter view gets the subtler variant. Update the `.presenter-body .stage-note` CSS in your popup template — see `references/presenter-view.md`.

### Markdown Rendering in Demo Responses

Demo response display now renders markdown (bold, italic, code, lists, headers) instead of escaping it as plain text. Modern chat models return markdown by default, so this makes the response look like the user pasted it into a real client. See `references/live-demos.md` → "Markdown Rendering for Responses".

**To migrate:** add the `renderMarkdown()` function and replace `escapeHtml(data.content)` with `renderMarkdown(data.content)` in both `showResponse()` and the JSON-toggle's "back to text" branch.

---

## v2.1 — April 21, 2026

### Next Slide Preview

New feature — both the presenter popup (Shift+P) and practice panel (P) now show a visual "Up Next" thumbnail of the next slide below the talk track. Shows "End of deck" on the last slide.

**To migrate an existing deck:**
1. Add the `cloneSlidePreview()` function and `window.cloneSlidePreview = cloneSlidePreview;` (see `references/presenter-view.md` → "Next Slide Preview")
2. Add the preview CSS for both practice panel and presenter popup
3. Add the preview HTML containers to both the practice panel and presenter popup template
4. Add stylesheet transfer after `doc.close()` in `openPresenter()` — copies main `<style>` into popup so cloned slides render correctly
5. Update `updatePractice()` and the presenter's `updateSlide()` to call `cloneSlidePreview(n + 1)` and render the result
6. Use `document.adoptNode()` in the presenter popup to transfer cloned nodes across documents

See `references/presenter-view.md` → "Next Slide Preview" section for complete code.

---

## v2.0 — April 14, 2026

### Skill Restructure

SKILL.md trimmed from 1,219 lines to ~200 — core workflow only. Detailed patterns moved to references/ (loaded on demand):
- `practice-mode.md` — talk track panel, stage notes
- `slide-components.md` — stat cards, badges, quotes, timelines
- `live-demos.md` — API demo containers, copy buttons, JSON toggle
- `advanced-components.md` — comparison panels, cost bars, journey tracks
- `deployment.md` — Flask wrapper, Dockerfile, env vars

### Starter Template

Added `assets/starter.html` — a complete working deck template the agent copies and modifies instead of regenerating from scratch.

### Presenter View

New feature — Shift+P opens a popup with talk track, slide number, and timer for live presenting on a second screen. See `references/presenter-view.md`.

### Slide Persistence

Deck saves current slide to localStorage on every navigation and restores on page load.

### Keyboard-Only Navigation

Removed click-to-navigate — keyboard only (arrows, space, G, O, P).

---

## v1.0 — April 9, 2026

Initial release. Single-file HTML/CSS/JS slide deck with keyboard navigation, progress bar, animated transitions, practice mode with talk track, slide overview grid, and jump-to-slide overlay.
