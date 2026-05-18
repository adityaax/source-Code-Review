## Vulnerable Code

```python
from flask import Flask, request, send_file
import os

app = Flask(__name__)

UPLOAD_FOLDER = "uploads"

@app.route("/upload", methods=["POST"])
def upload():
    file = request.files["file"]
    path = os.path.join(UPLOAD_FOLDER, file.filename)
    file.save(path)

    return "Uploaded"


@app.route("/download")
def download():
    filename = request.args.get("file")
    return send_file(os.path.join(UPLOAD_FOLDER, filename))


@app.route("/delete", methods=["POST"])
def delete():
    filename = request.form.get("file")
    os.remove(os.path.join(UPLOAD_FOLDER, filename))
    return "Deleted"

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Path Traversal in Upload**\
Attackers can overwrite arbitrary files.

**2. Unrestricted File Upload**\
No validation on file type.

**3. Arbitrary File Read**\
`/download` allows path traversal.

**4. Arbitrary File Deletion**\
Attackers can delete server files.

**5. Missing Authorization**\
No ownership checks exist.

</details>
