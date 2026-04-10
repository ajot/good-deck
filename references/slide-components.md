# Slide Components

Reusable component patterns for common slide layouts. Include the CSS for any components you use.

## Stat Card

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

Counter animation JS (include in your script):
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
    const eased = 1 - Math.pow(1 - progress, 4);
    const value = eased * target;
    el.textContent = prefix + (decimals > 0
      ? value.toFixed(decimals)
      : Math.round(value).toLocaleString()) + suffix;
    if (progress < 1) requestAnimationFrame(update);
  }
  requestAnimationFrame(update);
}
```

Trigger counters in `triggerAnimations()`:
```javascript
slide.querySelectorAll('.counter').forEach((el, i) => {
  setTimeout(() => animateCounter(el), i * 300);
});
```

## Badge / Pill

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

## Tweet / Quote Card

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

## Takeaway / Bullet List

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

## Team / People Grid

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
    <img src="images/name.jpg" class="team-photo" alt="Name">
    <div class="team-name">First Last</div>
  </div>
</div>
```

Stagger team members in `triggerAnimations()`:
```javascript
slide.querySelectorAll('.team-member').forEach((member, i) => {
  member.style.transitionDelay = (0.3 + i * 0.06) + 's';
});
```

## Horizontal Timeline

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
</div>
```

Animate in `triggerAnimations()`:
```javascript
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
```

## Image Card

```html
<img src="images/screenshot.png" alt="Description"
     style="max-width: 560px; border-radius: 12px;
            border: 1px solid var(--border);
            box-shadow: 0 8px 32px rgba(0,0,0,0.4);">
```

## Code Highlighter Block

Yellow highlighter effect to call attention to important lines in code blocks:

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
