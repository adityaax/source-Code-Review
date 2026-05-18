## Vulnerable Code

```python
from flask import Flask, request, jsonify
import jwt

app = Flask(__name__)

SECRET = "supersecret"

users = {
    "1": {"username": "alice", "role": "user"},
    "2": {"username": "admin", "role": "admin"}
}

@app.route("/login", methods=["POST"])
def login():
    username = request.form.get("username")

    token = jwt.encode(
        {"user_id": "1", "role": username},
        SECRET,
        algorithm="HS256"
    )

    return jsonify({"token": token})


@app.route("/profile")
def profile():
    token = request.headers.get("Authorization")

    data = jwt.decode(token, options={"verify_signature": False})

    uid = request.args.get("id", data["user_id"])

    return jsonify(users.get(uid))


@app.route("/admin")
def admin():
    token = request.headers.get("Authorization")
    data = jwt.decode(token, SECRET, algorithms=["HS256", "none"])

    if data.get("role") == "admin":
        return "Welcome admin"

    return "Forbidden", 403

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Signature Verification Disabled**\
The `/profile` endpoint decodes JWT tokens without verifying signatures.

**2. Algorithm Confusion**\
The server accepts `alg=none`.

**3. Hardcoded Secret Key**\
The JWT secret is embedded in source code.

**4. Role Injection**\
Role is derived directly from user-controlled input.

**5. IDOR in Profile Access**\
Users can access arbitrary profiles using `?id=`.

</details>
