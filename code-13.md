## Vulnerable Code

```python
from flask import Flask, request, redirect, session
import requests

app = Flask(__name__)
app.secret_key = "dev-secret-key"

CLIENT_ID = "client_123"
CLIENT_SECRET = "supersecret"
REDIRECT_URI = "https://app.example.com/oauth/callback"

@app.route("/login")
def login():
    oauth_url = (
        "https://auth.example.com/oauth/authorize"
        f"?client_id={CLIENT_ID}"
        "&response_type=code"
        f"&redirect_uri={REDIRECT_URI}"
        "&scope=profile email"
    )

    return redirect(oauth_url)


@app.route("/oauth/callback")
def callback():
    code = request.args.get("code")

    token_resp = requests.post(
        "https://auth.example.com/oauth/token",
        data={
            "grant_type": "authorization_code",
            "code": code,
            "redirect_uri": REDIRECT_URI,
            "client_id": CLIENT_ID,
            "client_secret": CLIENT_SECRET,
        },
    )

    access_token = token_resp.json()["access_token"]

    profile = requests.get(
        "https://auth.example.com/userinfo",
        headers={"Authorization": f"Bearer {access_token}"},
    ).json()

    session["email"] = profile["email"]

    next_url = request.args.get("next", "/dashboard")
    return redirect(next_url)


@app.route("/dashboard")
def dashboard():
    return f"Hello {session.get('email')}"
```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Missing OAuth `state` Parameter**  
Authorization request does not include `state`, and callback does not validate it. Can lead to OAuth CSRF / forced login with attacker-controlled account.

**2. Open Redirect in OAuth Callback**  
User-controlled `next` parameter is redirected without validation. Can redirect victims to attacker-controlled domains for phishing.

**3. Hardcoded Secrets**  
OAuth client secret and Flask session secret are hardcoded in source. If exposed, attacker may impersonate app or forge sessions.

**4. Missing Token Validation**  
Application trusts token and `/userinfo` response without validating issuer, audience, or expiration. May allow authentication with invalid tokens.

**5. Authorization Code Replay (Provider Dependent)**  
No replay protection is implemented after receiving authorization code. If provider mishandles code invalidation, same code may be reused.

</details>
