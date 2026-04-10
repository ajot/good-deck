# Advanced Components

Patterns for comparison panels, cost visualizations, journey tracks, and other specialized slide layouts.

## Step Badge

Small label above a headline to indicate a stage or section:

```css
.step-badge {
  display: inline-block;
  padding: 6px 16px;
  border-radius: 100px;
  font-family: var(--font-mono);
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1.5px;
  background: var(--accent-subtle);
  border: 1px solid rgba(0, 105, 255, 0.2);
  color: var(--accent-light);
  margin-bottom: 12px;
}
```

```html
<div class="step-badge animate-in">Section Name</div>
<h2 class="headline-sm animate-in">Your headline here.</h2>
```

## Background Image Overlays

Use a gradient overlay on background images to maintain text readability:

```html
<section class="slide slide-centered" style="background: linear-gradient(rgba(8,14,36,0.8), rgba(8,14,36,0.88)), url('images/photo.png') center/cover no-repeat;">
  <h2 class="headline animate-in">Text over image</h2>
</section>
```

Adjust opacity values (0.7–0.9) based on image brightness. Darker images need less overlay.

## Side-by-Side Comparison Panels

Two panels side by side for comparing options, approaches, or results:

```css
.compare-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  max-width: 1000px;
  width: 100%;
}
.compare-panel {
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
}
.compare-panel .panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  border-bottom: 1px solid var(--border);
}
.compare-panel .panel-name {
  font-family: var(--font-mono);
  font-size: 14px;
  font-weight: 600;
}
.compare-panel .panel-rule {
  font-size: 12px;
  color: var(--text-muted);
}
.compare-panel .panel-response {
  padding: 14px 16px;
  font-size: 13px;
  line-height: 1.6;
  color: var(--text-secondary);
  min-height: 80px;
  max-height: 120px;
  overflow-y: auto;
}
.compare-panel .panel-meta {
  padding: 10px 16px;
  font-family: var(--font-mono);
  font-size: 12px;
  color: var(--text-muted);
  display: flex;
  align-items: center;
  gap: 16px;
  border-top: 1px solid var(--border);
  flex-wrap: wrap;
}
.panel-btn {
  padding: 3px 10px;
  border-radius: 5px;
  border: 1px solid var(--border);
  background: none;
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 11px;
  cursor: pointer;
}
.panel-btn:hover { color: var(--accent-light); border-color: var(--accent); }
```

```html
<div class="compare-grid animate-in">
  <div class="compare-panel">
    <div class="panel-header">
      <div>
        <div class="panel-name">Option A</div>
        <div class="panel-rule">Description</div>
      </div>
      <button class="panel-btn" onclick="copyOptionA()">Copy</button>
    </div>
    <div class="panel-response" id="panel-a-response">
      <span class="placeholder">Waiting...</span>
    </div>
    <div class="panel-meta">
      <span>Time: <span class="meta-val" id="panel-a-time">—</span></span>
      <span>Tokens: <span class="meta-val" id="panel-a-tokens">—</span></span>
    </div>
  </div>
  <div class="compare-panel">
    <!-- same structure for Option B -->
  </div>
</div>
```

## Cost Comparison Bars

Visual bars showing relative costs between options:

```css
.cost-bar-wrap {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
}
.cost-bar-track {
  flex: 1;
  height: 6px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 3px;
  overflow: hidden;
}
.cost-bar-fill {
  height: 100%;
  border-radius: 3px;
  transition: width 1s ease;
}
.cost-bar-fill.cheap { background: var(--success); }
.cost-bar-fill.expensive { background: #ef4444; }
.meta-val { font-weight: 600; }
.meta-val.cheap { color: var(--success); }
.meta-val.expensive { color: #ef4444; }
```

```html
<div class="cost-bar-wrap">
  <span>Cost:</span>
  <div class="cost-bar-track">
    <div class="cost-bar-fill cheap" id="cost-bar" style="width: 0%"></div>
  </div>
  <span class="meta-val cheap" id="cost-amount">—</span>
</div>
```

Set bar width dynamically based on relative cost:
```javascript
const maxCost = Math.max(costA, costB);
document.getElementById('cost-bar-a').style.width = (costA / maxCost * 100) + '%';
document.getElementById('cost-bar-b').style.width = (costB / maxCost * 100) + '%';
```

## Journey / Progression Track

A horizontal track showing stages with connected dots:

```css
.journey-track {
  display: flex;
  align-items: center;
  gap: 0;
  max-width: 800px;
  width: 100%;
  margin: 24px auto;
}
.journey-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  flex-shrink: 0;
}
.journey-dot {
  width: 16px; height: 16px;
  border-radius: 50%;
  background: var(--border);
  border: 2px solid var(--border);
  transition: background 0.4s, border-color 0.4s;
}
.journey-dot.active {
  background: var(--accent);
  border-color: var(--accent);
  box-shadow: 0 0 12px var(--accent-glow);
}
.journey-connector {
  flex: 1;
  height: 2px;
  background: var(--border);
}
.journey-connector.filled {
  background: var(--accent);
}
.journey-label {
  font-family: var(--font-mono);
  font-size: 12px;
  color: var(--text-muted);
}
.journey-label.active { color: var(--accent-light); }
```

```html
<div class="journey-track animate-in">
  <div class="journey-node">
    <div class="journey-dot active"></div>
    <div class="journey-label active">Serverless</div>
  </div>
  <div class="journey-connector filled"></div>
  <div class="journey-node">
    <div class="journey-dot"></div>
    <div class="journey-label">Dedicated</div>
  </div>
  <div class="journey-connector"></div>
  <div class="journey-node">
    <div class="journey-dot"></div>
    <div class="journey-label">GPU Droplets</div>
  </div>
</div>
```

## Construction / WIP Banner

Add `.slide-wip` to any slide to show a yellow/black striped construction bar at the top:

```css
.slide-wip::before {
  content: '';
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 5px;
  background: repeating-linear-gradient(
    -45deg,
    #eab308 0, #eab308 10px,
    transparent 10px, transparent 20px
  );
  z-index: 100;
  pointer-events: none;
}
```

```html
<section class="slide slide-centered slide-wip">
  <!-- slide content -->
</section>
```

## Version Footer

Small version link in the bottom-left corner, linking to a changelog:

```html
<a href="/changelog" target="_blank"
   style="font-family: var(--font-mono); font-size: 10px; color: rgba(255,255,255,0.2);
          text-decoration: none; padding-left: 15px;"
   onmouseover="this.style.textDecoration='underline';this.style.color='#fff'"
   onmouseout="this.style.textDecoration='none';this.style.color='rgba(255,255,255,0.2)'">v1.0</a>
```

Place this below the footer logo element.
