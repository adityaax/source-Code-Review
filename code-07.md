## Vulnerable Code

```python
from flask import Flask, request, jsonify
import time

app = Flask(__name__)

wallets = {
    "alice": 100
}

used_coupons = []

@app.route("/apply-coupon", methods=["POST"])
def apply_coupon():
    user = request.form.get("user")
    coupon = request.form.get("coupon")

    if coupon not in used_coupons:
        time.sleep(2)
        wallets[user] += 50
        used_coupons.append(coupon)

        return "Coupon applied"

    return "Already used"


@app.route("/balance")
def balance():
    user = request.args.get("user")
    return jsonify({"balance": wallets.get(user, 0)})

```

<details>
<summary><b>Reveal Solution</b></summary>

## Findings

**1. Race Condition in Coupon Redemption**\
Concurrent requests can redeem the same coupon multiple times.

**2. Missing Atomic Operations**\
Coupon validation and redemption are not transactional.

**3. User-Controlled Account Selection**\
Attackers can apply coupons to arbitrary wallets.

**4. No Authentication**\
Coupon application requires no session validation.

**5. Business Logic Abuse**\
Unlimited wallet inflation is possible.

</details>
