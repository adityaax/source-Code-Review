## Vulnerable Code

```python
from flask import Flask, request
import requests

app = Flask(__name__)

@app.route("/fetch")
def fetch():
    url = request.args.get("url")

    response = requests.get(url)

    return response.text


@app.route("/metadata")
def metadata():
    return """
    internal-token=secret123
    db-password=rootpass
    """

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Server-Side Request Forgery (SSRF)**\
The `/fetch` endpoint allows arbitrary URL requests.

**2. Internal Resource Access**\
Attackers can access internal endpoints like `/metadata`.

**3. Sensitive Information Disclosure**\
Internal credentials are exposed.

**4. No URL Validation**\
No filtering of localhost or internal IP ranges.

**5. Potential Cloud Metadata Access**\
Could be abused to access instance metadata services.

</details>
