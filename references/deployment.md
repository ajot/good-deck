# Deployment

Optional Flask wrapper for password-protected hosting and live API demos.

## Flask App (`app.py`)

Serves a login page, then `index.html` once authenticated. Serves `images/` directory behind the same auth check. Includes API proxy endpoints for live demos.

```python
import os
import time
import requests
from flask import Flask, request, jsonify, send_from_directory, session, redirect

app = Flask(__name__)
app.secret_key = os.environ.get('SECRET_KEY', 'change-me')
PASSWORD = os.environ.get('APP_PASSWORD', 'password')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        if request.form.get('password') == PASSWORD:
            session['auth'] = True
            return redirect('/')
        return 'Wrong password', 401
    return '''
    <form method="post" style="padding:40px;font-family:sans-serif">
      <input type="password" name="password" placeholder="Password" autofocus>
      <button type="submit">Enter</button>
    </form>
    '''

@app.route('/')
def index():
    if not session.get('auth'):
        return redirect('/login')
    return send_from_directory('.', 'index.html')

@app.route('/images/<path:filename>')
def images(filename):
    if not session.get('auth'):
        return redirect('/login')
    return send_from_directory('images', filename)

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
        'max_tokens': data.get('max_tokens', 512)
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

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080, debug=True)
```

## Requirements (`requirements.txt`)

```
flask
gunicorn
requests
```

## Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["gunicorn", "-b", "0.0.0.0:8080", "app:app"]
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `APP_PASSWORD` | Slide deck login password |
| `SECRET_KEY` | Flask session secret |
| `MODEL_ACCESS_KEY` | API key for inference provider |

Set these as environment variables in your hosting platform (e.g., DigitalOcean App Platform console).

## Running Locally

```bash
pip install -r requirements.txt
cp .env.example .env  # Fill in API keys
python app.py          # Serves at http://localhost:8080
```

Access via `http://localhost:8080` (NOT `file://` — the live demos need the Flask backend).
