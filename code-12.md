## Vulnerable Code

```python
from flask import Flask, request, send_file
import os

app = Flask(__name__)

UPLOAD_DIR = "uploads"


@app.route("/upload", methods=["POST"])
def upload():
    file = request.files["file"]

    path = os.path.join(UPLOAD_DIR, file.filename)
    file.save(path)

    return "Uploaded"


@app.route("/files")
def files():
    name = request.args.get("name")

    return send_file(os.path.join(UPLOAD_DIR, name))
```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Path Traversal in File Download**  
User-controlled filename is directly joined into filesystem path.

Example payload:

```bash
../../etc/passwd
```

Can disclose arbitrary server files.

**2. Arbitrary File Write**  
Uploaded filename is not sanitized.

An attacker may upload files using traversal payloads such as:

```bash
../../app.py
```

Potentially overwriting server files.

**3. Unrestricted File Upload**  
Application allows uploading arbitrary file types. Could enable malicious script upload or storage abuse.

**4. Missing File Size Validation**  
No upload size restriction. Could lead to disk exhaustion / denial of service.

</details>
