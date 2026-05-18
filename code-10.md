## Vulnerable Code

```python
from flask import Flask, request
from lxml import etree

app = Flask(__name__)

@app.route("/import-xml", methods=["POST"])
def import_xml():
    xml_data = request.data

    parser = etree.XMLParser(resolve_entities=True)
    root = etree.fromstring(xml_data, parser)

    return root.text


@app.route("/secret")
def secret():
    if request.remote_addr != "127.0.0.1":
        return "Forbidden", 403

    return "db_password=supersecret"

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. XML External Entity (XXE) Injection**\
The application parses attacker-controlled XML with external entity resolution enabled, allowing malicious XML payloads to define and resolve external entities.

**2. Arbitrary Local File Disclosure**\
An attacker can abuse XXE to read sensitive files from the server filesystem such as `/etc/passwd`, application configuration files, or stored secrets.

**3. Server-Side Request Forgery (SSRF)**\
External entities can force the server to make requests to internal or external resources, enabling access to services not directly reachable by attackers.

**4. Internal Secret Disclosure via XXE Pivoting**\
Although the `/secret` endpoint is restricted to localhost access, XXE-based SSRF allows attackers to bypass this restriction and retrieve sensitive credentials.

**5. Lack of XML Input Validation**\
The application processes user-supplied XML without schema validation or sanitization, increasing parser abuse risk.

</details>
