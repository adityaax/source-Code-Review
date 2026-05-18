## Vulnerable Code

```python
from flask import Flask, request
import pickle
import base64

app = Flask(__name__)

@app.route("/import-session", methods=["POST"])
def import_session():
    data = request.form.get("session")

    decoded = base64.b64decode(data)
    session = pickle.loads(decoded)

    return f"Welcome {session.get('username')}"


@app.route("/debug")
def debug():
    return str(request.environ)

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Insecure Deserialization**\
The application deserializes user-controlled input using `pickle.loads()`, which can lead to arbitrary code execution.

**2. Arbitrary Code Execution Risk**\
A crafted pickle payload can execute attacker-controlled commands on the server.

**3. Missing Integrity Validation**\
Session data is accepted without any cryptographic validation.

**4. Sensitive Environment Disclosure**\
The `/debug` endpoint exposes server environment variables.

**5. Lack of Input Validation**\
Base64-decoded input is trusted without sanitization.

</details>
