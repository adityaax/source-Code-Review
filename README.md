# source-Code-Review

Practice source code review through realistic vulnerable code snippets.

source-Code-Review is built for security researchers, bug bounty hunters, and learners who want to improve their ability to identify vulnerabilities by **reading and understanding source code.**

Instead of exploitation-focused labs, this repository helps build a core skill many researchers overlook:

**manual source code review.**

---

## Why This Repository Exists

Many learners focus heavily on payloads and automated tooling.

But strong security researchers often find impactful vulnerabilities by simply reviewing code and spotting:

- Broken trust assumptions
- Authorization flaws
- Logic mistakes
- Unsafe input handling

This repository helps develop that review mindset.

---

## What You'll Learn

Through these challenges, you'll practice identifying:

- Broken Access Control
- Authentication Bypass
- IDOR
- SQL Injection
- Session Management Issues
- Privilege Escalation
- Business Logic Flaws
- Token Validation Mistakes
- Trust Boundary Failures

---

## How It Works

Each challenge follows a simple review flow.

### 1. Read the Code

Review the provided vulnerable source code snippet.

---

### 2. Analyze It Yourself

Before revealing the answer, ask:

- What input is attacker-controlled?
- Where is trust granted?
- Is authorization enforced correctly?
- Can application state be manipulated?
- What assumptions does this code make?

---

### 3. Reveal the Solution

Each file contains a hidden solution section explaining:

- The vulnerability
- Why it exists
- Security impact
- Reviewer mindset

---

## Challenge Format

Each challenge is uploaded as an individual markdown file.

```text
code-1.md
code-2.md
code-3.md
...
```

Every file includes:

- Vulnerable source code
- Solution walkthrough

---

**Note: Found an error or inconsistency in any solution? Feel free to open an issue.**

## Example

```python
from flask import Flask, request, session, jsonify
import sqlite3

app = Flask(__name__)
app.secret_key = "secret"

def get_db():
    conn = sqlite3.connect("users.db")
    conn.row_factory = sqlite3.Row
    return conn


@app.route("/profile")
def profile():
    if "user_id" not in session:
        return "Unauthorized", 403

    uid = request.args.get("id", session["user_id"])

    db = get_db()
    user = db.execute(
        "SELECT id, username, email FROM users WHERE id=?",
        (uid,)
    ).fetchone()

    return jsonify(dict(user))
```

<details>
<summary><b>Reveal Solution</b></summary>

## Finding

**Insecure Direct Object Reference (IDOR)**\
The application checks whether the user is authenticated:

```python
if "user_id" not in session:
```

However, it does not verify whether the requested profile belongs to the authenticated user. An authenticated attacker can access another user's profile by changing the ID: /profile?id=2
