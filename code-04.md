## Vulnerable Code

```python
from flask import Flask, request
import random

app = Flask(__name__)

tokens = {}

@app.route("/forgot-password", methods=["POST"])
def forgot():
    email = request.form.get("email")

    token = str(random.randint(1000, 9999))
    tokens[email] = token

    return f"Reset token: {token}"


@app.route("/reset-password", methods=["POST"])
def reset():
    email = request.form.get("email")
    token = request.form.get("token")
    new_pass = request.form.get("password")

    if tokens.get(email) == token:
        return "Password changed"

    return "Invalid token", 403

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Predictable Reset Tokens**\
Only 4-digit numeric tokens are used.

**2. Token Disclosure**\
Reset token is returned in response.

**3. No Expiration**\
Tokens remain valid indefinitely.

**4. Token Reuse**\
Tokens are not invalidated after use.

**5. User Enumeration**\
Response differences can reveal valid emails.


</details>
