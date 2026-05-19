## Vulnerable Code

```python
from flask import Flask, request, session

app = Flask(__name__)
app.secret_key = "secret123"


@app.route("/login", methods=["POST"])
def login():
    username = request.form.get("username")

    session["user"] = username

    return "Logged in"


@app.route("/admin")
def admin():
    if session.get("user") == "admin":
        return "FLAG{admin-access}"

    return "Forbidden", 403


@app.route("/change-role", methods=["POST"])
def change_role():
    if not session.get("user"):
        return "Login required", 401

    session["user"] = request.form.get("role")

    return "Role updated"
```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Authentication Bypass**\
Login accepts any username without password verification.

**2. Privilege Escalation**\
An attacker can set: role=admin via /change-role. This grants admin access.

**3. Broken Access Control**\
Admin authorization relies only on session username value. No server-side role validation.

**4. Hardcoded Secret Key**\
Hardcoded/Predictable secret may allow session forgery.

</details>
