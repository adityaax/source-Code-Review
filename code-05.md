## Vulnerable Code

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

API_KEY = "internal-secret-key"

users = {
    "1": {"username": "alice", "password": "alice123"},
    "2": {"username": "bob", "password": "bob123"}
}

@app.route("/internal/stats")
def stats():
    if request.headers.get("X-Internal") == "true":
        return jsonify({
            "users": len(users),
            "api_key": API_KEY
        })

    return "Forbidden", 403


@app.route("/admin/export")
def export():
    key = request.args.get("key")

    if key == API_KEY:
        return jsonify(users)

    return "Unauthorized", 401


@app.route("/debug")
def debug():
    return jsonify({
        "headers": dict(request.headers),
        "config": {
            "api_key": API_KEY
        }
    })

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Header-Based Authentication Bypass**\
Admin functionality trusts user-controlled `X-Internal` header.

**2. Hardcoded API Key Disclosure**\
Sensitive API key is embedded directly in source code.

**3. Sensitive Data Exposure**\
`/admin/export` exposes usernames and passwords.

**4. Debug Information Leak**\
The `/debug` endpoint leaks internal config values.

**5. API Key Disclosure via Stats Endpoint**\
The stats response exposes the internal API key.

</details>
