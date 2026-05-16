```
from flask import Flask, request, session, jsonify
import sqlite3

app = Flask(__name__)
app.secret_key = "dev-secret-key"

def get_db():
    conn = sqlite3.connect("users.db")
    conn.row_factory = sqlite3.Row
    return conn


@app.route("/login", methods=["POST"])
def login():
    username = request.form.get("username")
    password = request.form.get("password")

    db = get_db()
    user = db.execute(
        f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
    ).fetchone()

    if user:
        session["user_id"] = user["id"]
        session["role"] = user["role"]
        return "Logged in"

    return "Invalid credentials", 401


@app.route("/profile")
def profile():
    if "user_id" not in session:
        return "Unauthorized", 403

    uid = request.args.get("id", session["user_id"])

    db = get_db()
    user = db.execute(
        "SELECT id, username, email, role FROM users WHERE id=?",
        (uid,)
    ).fetchone()

    return jsonify(dict(user))


@app.route("/update-email", methods=["POST"])
def update_email():
    if "user_id" not in session:
        return "Unauthorized", 403

    new_email = request.form.get("email")
    user_id = request.form.get("user_id")

    db = get_db()
    db.execute(
        "UPDATE users SET email=? WHERE id=?",
        (new_email, user_id)
    )
    db.commit()

    return "Email updated"


@app.route("/admin/delete-user", methods=["POST"])
def delete_user():
    if session.get("role") == "admin" or request.headers.get("X-Admin") == "true":
        uid = request.form.get("id")

        db = get_db()
        db.execute("DELETE FROM users WHERE id=?", (uid,))
        db.commit()

        return "Deleted"

    return "Forbidden", 403


if __name__ == "__main__":
    app.run(debug=True)
```

<details>
<summary><b>Reveal Solution</b></summary>

1. SQL Injection in /login
The login query directly inserts user input into the SQL statement using string interpolation. An attacker can inject SQL payloads (such as ' OR '1'='1) to bypass authentication and log in without valid credentials.

2. Insecure Secret Key Disclosure
The Flask secret key is hardcoded as dev-secret-key. If an attacker knows this value, they may forge signed session cookies and manipulate session data, potentially assigning themselves elevated privileges like admin access.

3. IDOR in /profile
The profile endpoint accepts a user-controlled id parameter without validating ownership. Any authenticated user can access another user’s profile by changing the ID value. Example: /profile?id=2

4. Unauthorized Email Modification
The /update-email endpoint trusts the user_id supplied in the request instead of using the authenticated session user. An attacker can modify another user's email by submitting a different user_id, resulting in broken access control.

5. Privilege Escalation in /admin/delete-user
Admin access is granted if the request contains: X-Admin: true

6. Debug Mode Enabled
The application runs with: app.run(debug=True)

</details>
