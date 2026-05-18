## Vulnerable Code

```python
from flask import Flask, request, session, jsonify

app = Flask(__name__)
app.secret_key = "devkey"

projects = {
    "1": {"owner": "alice", "status": "draft"}
}

@app.route("/login", methods=["POST"])
def login():
    session["user"] = request.form.get("username")
    return "Logged in"


@app.route("/create-project", methods=["POST"])
def create():
    pid = request.form.get("id")
    projects[pid] = {
        "owner": session["user"],
        "status": "draft"
    }
    return "Created"


@app.route("/approve", methods=["POST"])
def approve():
    pid = request.form.get("id")
    projects[pid]["status"] = "approved"
    return "Approved"


@app.route("/archive", methods=["POST"])
def archive():
    pid = request.form.get("id")

    if request.headers.get("X-Manager") == "true":
        projects[pid]["status"] = "archived"
        return "Archived"

    return "Forbidden", 403


@app.route("/project")
def project():
    pid = request.args.get("id")
    return jsonify(projects.get(pid))

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Authentication Bypass**\
The login endpoint only requires a username and does not verify a password. An attacker can authenticate as any user by simply submitting their username.

**2. Missing Authorization in Approval Workflow**\
Any authenticated user can approve arbitrary projects.

**3. Broken Access Control in Archive Function**\
Manager access is granted via user-controlled `X-Manager` header.

**4. IDOR in Project Access**\
Users can access arbitrary project data using `?id=`.

**5. State Transition Bypass**\
Projects can be approved or archived without workflow validation.

**6. Hardcoded Secret Key**\
The Flask secret key is predictable and exposed.

</details>
