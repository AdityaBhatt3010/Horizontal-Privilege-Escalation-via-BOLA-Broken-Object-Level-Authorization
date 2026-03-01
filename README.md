# Horizontal Privilege Escalation via Broken Object Level Authorization (BOLA)

**Broken Object Level Authorization (BOLA) enables cross-user document viewing, modification, and unauthorized deletion via direct object reference.**

## GHSA References

* **GHSA-gj3h-wcpc-5pw7**
  [https://github.com/karnop/realtime-collaboration-platform/security/advisories/GHSA-gj3h-wcpc-5pw7](https://github.com/karnop/realtime-collaboration-platform/security/advisories/GHSA-gj3h-wcpc-5pw7)

* **GHSA-m56q-pj22-83xq**
  [https://github.com/karnop/realtime-collaboration-platform/security/advisories/GHSA-m56q-pj22-83xq](https://github.com/karnop/realtime-collaboration-platform/security/advisories/GHSA-m56q-pj22-83xq)

![Cover](https://github.com/user-attachments/assets/61e7acab-e62b-44b3-831e-adf241a8cdd9) <br/>

---

## Summary

The application fails to enforce proper object-level authorization when resolving documents using direct object references:

```
/documents/{documentId}
```

An authenticated user can:

* View documents belonging to other users
* Modify document content in real time
* Perform collaborative edits
* Permanently delete documents via the REST API

This results in horizontal privilege escalation across users.

Affected versions: **All (main branch)**
Severity: **High**

---

# Vulnerability 1 — Unauthorized Viewing & Modification

(GHSA-gj3h-wcpc-5pw7)

Documents are assigned explicit owner-bound permissions:

```json
"$permissions": [
  "read(\"user:OWNER_ID\")",
  "update(\"user:OWNER_ID\")",
  "delete(\"user:OWNER_ID\")"
]
```

Despite these restrictions:

* Authenticated User B navigates to:

  ```
  /documents/{DocumentID}
  ```
* Document loads successfully
* Live editing is enabled
* Changes persist and sync in real time

Delete operations are restricted, but update operations are not properly validated.

This confirms update-level authorization bypass.

---

# 🔥 PoC — Unauthorized View & Modify

## Steps to Reproduce

1. Register Account A
2. Create a document
3. Copy the Document ID
4. Register Account B
5. Login as Account B
6. Navigate to:

```
https://realtime-collaboration-platform-steel.vercel.app/documents/{DocumentID}
```

<img width="1918" height="1032" alt="Share_Escape" src="https://github.com/user-attachments/assets/46af0de7-627d-421f-afcc-b4dc6e29c0d6" /> <br/>

## Result

* Document loads
* Editing is permitted
* Changes are reflected in real time
* Owner sees modified content

Impact: **Confidentiality + Integrity compromise**

---

# Vulnerability 2 — Unauthorized Deletion

(GHSA-m56q-pj22-83xq)

Deletion permissions are defined at metadata level but not enforced at the API layer.

Endpoint:

```
DELETE /v1/databases/{databaseId}/collections/documents/documents/{documentId}
```

---

# 🔥 PoC — Unauthorized Deletion

```javascript
fetch("https://cloud.appwrite.io/v1/databases/6981d87b002e1c8dbc0d/collections/documents/documents/69981f2500182f323cd2", {
  method: "DELETE",
  credentials: "include",
  headers: {
    "X-Appwrite-Project": "6981d34b0036b9515a07"
  }
});
```

<img width="1916" height="1028" alt="Unauth_Del" src="https://github.com/user-attachments/assets/acdfc817-ca8a-42e1-b58c-b2b5b1ec6f33" /> <br/>

## Result

* HTTP 204 No Content
* Document permanently deleted
* Owner cannot recover it

Impact: **Confidentiality + Integrity + Availability compromise**

---

# Why This Is Critical

This vulnerability:

* Enables horizontal privilege escalation
* Breaks object-level authorization
* Allows unauthorized state-changing operations
* Aligns with OWASP API Top 10 — API1: Broken Object Level Authorization
* Maps to MITRE CWE-639

---

# Recommended Remediation

* Enforce strict server-side ownership validation
* Validate requesting user against permission set
* Implement backend-level authorization middleware
* Audit all state-changing endpoints
* Never rely on client-side routing for access control

---

# ⭐ Follow Me & Connect

🔗 GitHub: [https://github.com/AdityaBhatt3010](https://github.com/AdityaBhatt3010) <br/>
💼 LinkedIn: [https://www.linkedin.com/in/adityabhatt3010/](https://www.linkedin.com/in/adityabhatt3010/) <br/>
✍️ Medium: [https://medium.com/@adityabhatt3010](https://medium.com/@adityabhatt3010) <br/>

---
