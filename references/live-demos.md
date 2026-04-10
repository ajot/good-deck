# Live API Demos

Interactive demo slides with Run buttons, model dropdowns, API calls, response display, copy-to-clipboard, and JSON toggle. These make the deck a live application, not just a static presentation.

## Demo Container CSS

```css
.demo-container {
  max-width: 1000px;
  width: 100%;
  text-align: left;
}
.demo-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}
.demo-top-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

/* Model selector dropdown */
.model-select {
  padding: 8px 14px;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: rgba(0, 0, 0, 0.3);
  color: var(--text-primary);
  font-family: var(--font-mono);
  font-size: 13px;
  cursor: pointer;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%2394a3b8' d='M6 8L1 3h10z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 10px center;
  padding-right: 30px;
}
.model-select:hover { border-color: var(--accent); }
.model-select option { background: #111; color: var(--text-primary); }

/* Run button */
.run-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 24px;
  border-radius: 10px;
  border: none;
  background: var(--accent);
  color: white;
  font-family: var(--font-mono);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
}
.run-btn:hover { background: #0055cc; }
.run-btn:active { transform: scale(0.97); }
.run-btn:disabled {
  background: var(--bg-elevated);
  color: var(--text-muted);
  cursor: not-allowed;
}
.run-btn .spinner {
  display: inline-block;
  width: 14px; height: 14px;
  border: 2px solid rgba(255,255,255,0.3);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* Timer display */
.timer {
  font-family: var(--font-mono);
  font-size: 14px;
  color: var(--text-muted);
  min-width: 80px;
  text-align: right;
}
.timer.done { color: var(--success); font-weight: 600; }

/* Code block (request) */
.demo-code {
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid var(--border);
  border-radius: 12px 12px 0 0;
  position: relative;
  padding: 20px 24px;
  font-family: var(--font-mono);
  font-size: 13px;
  line-height: 1.7;
  color: var(--text-secondary);
  white-space: pre;
  overflow-x: auto;
}

/* Response area */
.demo-response {
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid var(--border);
  border-top: none;
  border-radius: 0;
  padding: 20px 24px;
  font-family: var(--font-mono);
  font-size: 13px;
  line-height: 1.6;
  color: var(--text-secondary);
  min-height: 80px;
  max-height: 200px;
  overflow-y: auto;
}
.demo-response .placeholder { color: var(--text-muted); font-style: italic; }
.demo-response .response-text { color: var(--text-primary); }

/* Meta bar below response */
.demo-meta {
  padding: 12px 24px;
  font-family: var(--font-mono);
  font-size: 13px;
  color: var(--text-muted);
  display: flex;
  gap: 24px;
  background: rgba(0, 0, 0, 0.2);
  border: 1px solid var(--border);
  border-top: none;
  border-radius: 0 0 12px 12px;
}
.demo-meta .meta-accent { color: var(--accent-light); font-weight: 600; }
```

## Copy Button CSS

```css
.copy-btn {
  position: absolute;
  top: 10px; right: 10px;
  padding: 4px 12px;
  border-radius: 6px;
  border: 1px solid var(--border);
  background: rgba(0, 0, 0, 0.3);
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 11px;
  cursor: pointer;
  transition: color 0.2s, border-color 0.2s;
  z-index: 5;
}
.demo-response .copy-btn { top: -6px; }
.copy-btn:hover { color: var(--text-primary); border-color: var(--accent); }
.copy-btn.copied { color: var(--success); border-color: var(--success); }
```

## JSON Toggle CSS

```css
.json-toggle {
  padding: 3px 10px;
  border-radius: 5px;
  border: 1px solid var(--border);
  background: none;
  color: var(--text-muted);
  font-family: var(--font-mono);
  font-size: 11px;
  cursor: pointer;
  transition: color 0.2s, border-color 0.2s;
  margin-left: auto;
}
.json-toggle:hover { color: var(--accent-light); border-color: var(--accent); }
.json-toggle.active { color: var(--accent-light); border-color: var(--accent); }
```

## Syntax Coloring Classes

```css
.code-key { color: var(--accent-light); }
.code-string { color: var(--success); }
```

## Demo HTML Pattern

```html
<div class="demo-container animate-in">
  <div class="demo-top">
    <div class="demo-top-left">
      <select class="model-select" id="demo1-model">
        <option value="model-name">model-name</option>
      </select>
      <button class="run-btn" onclick="runDemo(this, 'demo1')">Run</button>
    </div>
  </div>
  <div class="demo-code"><button class="copy-btn" onclick="copyCurl(this,'demo1')">Copy</button><span class="code-key">POST</span> <span class="code-string">https://api.example.com/v1/completions</span>

{
  <span class="code-key">"model"</span>: <span class="code-string">"model-name"</span>,
  <span class="code-key">"messages"</span>: [{ <span class="code-key">"role"</span>: <span class="code-string">"user"</span>, <span class="code-key">"content"</span>: <span class="code-string">"Your prompt here"</span> }]
}</div>
  <div class="demo-response" id="demo1-response">
    <span class="placeholder">Click Run to see the response...</span>
  </div>
</div>
```

**Important:** The `<div class="demo-code">` content uses `white-space: pre`, so formatting inside the tag matters. Don't indent the JSON content — it will show as indented in the slide.

## Demo JavaScript

### Utilities

```javascript
function escapeHtml(text) {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
}

function setBtnLoading(btn, loading) {
  if (loading) {
    btn.disabled = true;
    btn.innerHTML = '<span class="spinner"></span> Running...';
  } else {
    btn.disabled = false;
    btn.innerHTML = 'Run';
  }
}
```

### Response Display

```javascript
const demoRawData = {};

function showResponse(containerId, data) {
  const el = document.getElementById(containerId);
  demoRawData[containerId] = data;

  // Find or create meta bar
  let metaEl = document.getElementById(containerId + '-meta');
  if (!metaEl) {
    metaEl = document.createElement('div');
    metaEl.id = containerId + '-meta';
    metaEl.className = 'demo-meta';
    el.parentNode.insertBefore(metaEl, el.nextSibling);
  }
  if (data.error) {
    el.innerHTML = '<span style="color: #ef4444;">' + data.error + '</span>';
    metaEl.innerHTML = '';
    return;
  }
  el.dataset.showJson = 'false';
  el.innerHTML = '<div class="response-text">' + escapeHtml(data.content || 'No content') + '</div>';

  let meta = '<span>Model: <span class="meta-accent">' + (data.model || '?') + '</span></span>';
  if (data.usage && data.usage.total_tokens) {
    meta += '<span>Tokens: <span class="meta-accent">' + data.usage.total_tokens + '</span></span>';
  }
  meta += '<span>Time: <span class="meta-accent">' + data.ttft_ms + ' ms</span></span>';
  meta += '<button class="json-toggle" onclick="toggleJson(\'' + containerId + '\')">JSON</button>';
  metaEl.innerHTML = meta;
}
```

### JSON Toggle

```javascript
function toggleJson(containerId) {
  const el = document.getElementById(containerId);
  const data = demoRawData[containerId];
  if (!data) return;
  const btn = el.parentNode.querySelector('.json-toggle');

  if (el.dataset.showJson === 'true') {
    el.dataset.showJson = 'false';
    el.innerHTML = '<div class="response-text">' + escapeHtml(data.content || 'No content') + '</div>';
    if (btn) { btn.classList.remove('active'); btn.textContent = 'JSON'; }
  } else {
    el.dataset.showJson = 'true';
    el.innerHTML = '<div style="position: relative;"><button class="copy-btn" onclick="copyJson(\'' + containerId + '\')">Copy</button><pre style="font-size: 12px; white-space: pre-wrap; color: var(--text-secondary); margin: 0;">' + escapeHtml(JSON.stringify(data, null, 2)) + '</pre></div>';
    if (btn) { btn.classList.add('active'); btn.textContent = 'Response'; }
  }
}
```

### Copy Functions

```javascript
function copyJson(containerId) {
  const data = demoRawData[containerId];
  if (!data) return;
  navigator.clipboard.writeText(JSON.stringify(data, null, 2)).then(() => {
    const copyBtn = document.getElementById(containerId).querySelector('.copy-btn');
    if (copyBtn) {
      copyBtn.textContent = 'Copied!';
      copyBtn.classList.add('copied');
      setTimeout(() => { copyBtn.textContent = 'Copy'; copyBtn.classList.remove('copied'); }, 1500);
    }
  });
}

function copyCurl(btn, demoId) {
  // Build curl string based on the demo's model and prompt
  const modelEl = document.getElementById(demoId + '-model');
  const model = modelEl ? modelEl.value : 'default-model';
  const curl = `curl -X POST https://your-api.com/v1/completions \\
  -H "Authorization: Bearer $API_KEY" \\
  -H "Content-Type: application/json" \\
  -d '{"model": "${model}", "messages": [{"role": "user", "content": "..."}]}'`;

  navigator.clipboard.writeText(curl).then(() => {
    btn.textContent = 'Copied!';
    btn.classList.add('copied');
    setTimeout(() => { btn.textContent = 'Copy'; btn.classList.remove('copied'); }, 1500);
  });
}
```

### Running a Demo

```javascript
async function runDemo(btn, demoId) {
  setBtnLoading(btn, true);
  const responseEl = document.getElementById(demoId + '-response');
  responseEl.innerHTML = '<span class="placeholder">Calling API...</span>';

  try {
    const model = document.getElementById(demoId + '-model').value;
    const res = await fetch('/api/inference', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ model, prompt: 'Your prompt here' })
    });
    const data = await res.json();
    showResponse(demoId + '-response', data);
  } catch (e) {
    responseEl.innerHTML = '<span style="color: #ef4444;">Error: ' + e.message + '</span>';
  } finally {
    setBtnLoading(btn, false);
  }
}
```

## Click Navigation Exclusions

When using demo containers, exclude interactive elements from click navigation:

```javascript
document.addEventListener('click', (e) => {
  if (e.target.closest('.run-btn')) return;
  if (e.target.closest('.demo-container')) return;
  if (e.target.closest('button')) return;
  if (e.target.closest('.copy-btn')) return;
  // ... rest of click navigation
});
```

## Flask Backend for API Proxying

The Flask `app.py` proxies API calls so the frontend never exposes API keys:

```python
@app.route('/api/inference', methods=['POST'])
def inference():
    data = request.json
    headers = {
        'Authorization': f'Bearer {os.environ["MODEL_ACCESS_KEY"]}',
        'Content-Type': 'application/json'
    }
    payload = {
        'model': data.get('model', 'openai-gpt-4o'),
        'messages': [{'role': 'user', 'content': data.get('prompt', '')}],
        'max_tokens': 512
    }
    start = time.time()
    resp = requests.post('https://your-api.com/v1/chat/completions',
                         json=payload, headers=headers)
    elapsed = int((time.time() - start) * 1000)
    result = resp.json()
    return jsonify({
        'content': result['choices'][0]['message']['content'],
        'model': result.get('model'),
        'ttft_ms': elapsed,
        'usage': result.get('usage'),
    })
```
