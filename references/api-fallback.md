# API Fallback / Canned Mode

A safety net for live API demos. Pre-captures real API responses to a fixtures file, then silently falls back to those fixtures when live calls **time out, error out, or return empty content**. Also provides a `Shift+C` manual toggle to force fixture replay (useful when WiFi is flaky on stage).

## Why this exists

Live demos are great until they fail in front of 200 people. With this pattern:

- A flaky network, expired token, or upstream provider hiccup is invisible to the audience — the deck just shows the captured response
- You can pre-flight check by toggling `Shift+C` and confirming every demo still produces a sensible result without the network
- The captured `ttft_ms` is replayed (with jitter) so even the perceived latency feels real

## Three pieces

1. **`capture_fixtures.py`** — a script you run before the talk. Calls every live endpoint your deck uses, captures the JSON response, writes `fixtures.json`. Idempotent — failed captures preserve previous good data.
2. **`/api/fixtures`** — a Flask endpoint that serves the JSON behind your existing auth.
3. **`callWithFallback()`** — a JS wrapper that races the live call against a timeout, and falls back to the fixture on any failure. Wrap every demo's live call with it.

## Frontend Integration

### State and Toggle

Add near the top of your deck's main script, before any demo functions:

```javascript
// ===== FIXTURES + CANNED MODE =====
let fixtures = {};
fetch('/api/fixtures').then(r => r.json()).then(data => { fixtures = data || {}; }).catch(() => {});

let cannedMode = localStorage.getItem('cannedMode') === 'true';

function toggleCannedMode() {
  cannedMode = !cannedMode;
  try { localStorage.setItem('cannedMode', cannedMode); } catch(e) {}
  updateCannedBadge();
}

function updateCannedBadge() {
  const badge = document.getElementById('cannedBadge');
  if (badge) badge.style.display = cannedMode ? 'inline-block' : 'none';
}

async function callWithFallback(apiCall, fixture, timeoutMs = 15000) {
  if (cannedMode && fixture) {
    // Replay captured latency with ±15% jitter, capped at 12s
    const baseDelay = Math.min(fixture.ttft_ms || 2000, 12000);
    const jitter = baseDelay * (0.85 + Math.random() * 0.3);
    await new Promise(r => setTimeout(r, jitter));
    return JSON.parse(JSON.stringify(fixture));
  }
  try {
    const result = await Promise.race([
      apiCall(),
      new Promise((_, rej) => setTimeout(() => rej(new Error('timeout')), timeoutMs)),
    ]);
    if (result && result.error) throw new Error(result.error);
    if (result && (!result.content || !result.content.trim())) throw new Error('empty');
    return result;
  } catch (e) {
    if (fixture) {
      console.warn('Live call failed, using fixture:', e.message);
      return JSON.parse(JSON.stringify(fixture));
    }
    throw e;
  }
}

// Initialize badge on load
setTimeout(updateCannedBadge, 0);
```

The wrapper treats three things as failure: timeout, an `error` field on the response, or empty/whitespace `content`. All three silently fall back to the fixture if one is provided.

### Indicator Dot

A subtle 2px green dot next to the version number in the footer — barely visible from the projector but unmissable up close. Add inside your footer container:

```html
<span id="cannedBadge"
      style="display: none; width: 2px; height: 2px; border-radius: 50%;
             background: rgba(34,197,94,0.45); margin-left: 4px;"
      title="Canned mode (Shift+C to toggle)"></span>
```

### Keyboard Shortcut

Add to your existing `keydown` handler, before the existing `P`/`Shift+P` handlers:

```javascript
if (e.key === 'C' && e.shiftKey) { e.preventDefault(); toggleCannedMode(); return; }
```

### Wrapping a Demo

Convert each `runDemo()` from a direct `fetch` to a `callWithFallback()`:

```javascript
// Before:
async function runDemo1(btn) {
  setBtnLoading(btn, true);
  try {
    const resp = await fetch('/api/inference', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ model, prompt }),
    });
    const data = await resp.json();
    showResponse('demo1-response', data);
  } finally { setBtnLoading(btn, false); }
}

// After:
async function runDemo1(btn) {
  setBtnLoading(btn, true);
  try {
    const data = await callWithFallback(
      async () => {
        const resp = await fetch('/api/inference', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ model, prompt }),
        });
        return await resp.json();
      },
      fixtures.demo1_basic,   // the captured fixture for this demo
      15000                    // timeout in ms (use 30000 for slow demos like web search or MCP)
    );
    showResponse('demo1-response', data);
  } finally { setBtnLoading(btn, false); }
}
```

The fixture key (`demo1_basic`) must match what your capture script writes into `fixtures.json`.

## Backend: `/api/fixtures` Flask Endpoint

Add to `app.py`:

```python
@app.route("/api/fixtures", methods=["GET"])
@require_auth
def api_fixtures():
    """Return cached API responses for fallback when live calls fail.
    Run capture_fixtures.py to refresh."""
    import os, json
    fixtures_path = os.path.join(os.path.dirname(os.path.abspath(__file__)), "fixtures.json")
    if not os.path.exists(fixtures_path):
        return jsonify({}), 200
    try:
        with open(fixtures_path) as f:
            return jsonify(json.load(f))
    except Exception as e:
        return jsonify({"error": str(e)}), 500
```

Sits behind your existing auth — fixtures are demo content, but no reason to leak them.

## Capture Script

`capture_fixtures.py` is project-specific because it knows your endpoint shapes and demo prompts. Generic skeleton:

```python
#!/usr/bin/env python3
"""Capture API responses for fallback fixtures. Run before the talk to refresh.

Idempotent: existing fixtures.json is read first; only successful captures
are updated. Failed captures leave previous data intact.
"""
import json
import os
import sys
import time
from datetime import datetime
from pathlib import Path

import requests
from dotenv import load_dotenv

load_dotenv()

API_URL = os.environ["YOUR_API_URL"]
API_KEY = os.environ["YOUR_API_KEY"]
FIXTURES_PATH = Path(__file__).parent / "fixtures.json"


def call_api(payload: dict) -> dict:
    """Match your Flask backend's response shape exactly."""
    headers = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}
    start = time.time()
    try:
        resp = requests.post(API_URL, headers=headers, json=payload, timeout=60)
        elapsed_ms = round((time.time() - start) * 1000)
        resp.raise_for_status()
        data = resp.json()
        content = data["choices"][0]["message"]["content"]
        return {
            "content": content,
            "model": data.get("model"),
            "ttft_ms": elapsed_ms,
            "usage": data.get("usage", {}),
        }
    except Exception as e:
        return {"error": str(e), "ttft_ms": round((time.time() - start) * 1000)}


def is_good(result: dict) -> bool:
    """A capture is good if it has non-empty content and no error."""
    return bool(result and not result.get("error") and result.get("content", "").strip())


def load_existing() -> dict:
    if FIXTURES_PATH.exists():
        try:
            return json.loads(FIXTURES_PATH.read_text())
        except Exception:
            return {}
    return {}


def save(fixtures: dict):
    fixtures["captured_at"] = datetime.utcnow().isoformat() + "Z"
    FIXTURES_PATH.write_text(json.dumps(fixtures, indent=2))


def capture_demos(fixtures: dict):
    """One entry per demo, keyed by the same name used in fixtures.{key} on the frontend."""
    demos = {
        "demo1_basic":      {"model": "your-default-model", "messages": [{"role": "user", "content": "What is 2+2?"}]},
        "demo2_websearch":  {"model": "your-default-model", "messages": [{"role": "user", "content": "Latest..."}], "tools": [{"type": "web_search"}]},
        # ... one entry per demo
    }
    for key, payload in demos.items():
        print(f"Capturing {key}...", end=" ", flush=True)
        result = call_api(payload)
        if is_good(result):
            fixtures[key] = result
            print(f"ok ({result['ttft_ms']}ms)")
        else:
            print(f"FAILED ({result.get('error', 'empty')}) — keeping previous")


if __name__ == "__main__":
    fixtures = load_existing()
    capture_demos(fixtures)
    save(fixtures)
    print(f"Wrote {FIXTURES_PATH}")
```

Run with `./venv/bin/python capture_fixtures.py` before the talk. Add `fixtures.json` to git so it ships with the deck.

## Pre-Talk Verification

1. Run `capture_fixtures.py` — confirm every demo captures successfully
2. Restart the Flask app
3. Hit `Shift+C` — green dot appears next to version
4. Click through every demo — each should show the fixture content with realistic latency
5. Hit `Shift+C` again — green dot disappears
6. Reload the page — `cannedMode` persists from `localStorage`
7. (Optional) Disable WiFi mid-demo to verify silent fallback works without the manual toggle

## Implementation Notes

- **Empty content as failure**: many model APIs (especially reasoning models) sometimes return a `200 OK` with empty `content`. The wrapper treats that as failure so the fixture takes over. Without this check, you'd see a blank response and look broken on stage.
- **Deep clone the fixture**: `JSON.parse(JSON.stringify(fixture))` prevents mutation between calls (e.g. if `showResponse` modifies the object).
- **Timeout per demo**: 15s default is right for fast chat completions. Bump to 30s for demos involving tool calls (web search, MCP) since they have a multi-hop pattern.
- **Idempotent capture**: failed captures must not overwrite previous good data. The script reads existing `fixtures.json` first and only updates keys whose new capture passes `is_good()`.
- **Subtle indicator**: a 2px dim dot is intentionally easy to miss — it's an operator status, not a UI feature. You don't want the audience wondering what the green dot means.
