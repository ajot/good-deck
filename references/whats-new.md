# What's New

Read this file when the user asks to "apply latest changes" or "update to the latest skill version" on an existing deck. Each version lists what changed and how to migrate.

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

Initial release. Single-file HTML/CSS/JS slide deck with keyboard navigation, progress bar, animated transitions, practice mode with talk track, presenter view, slide overview grid, and jump-to-slide overlay.
