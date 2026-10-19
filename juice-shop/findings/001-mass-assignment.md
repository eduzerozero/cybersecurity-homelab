# Finding 001 – Mass Assignment and Privilege Escalation

## Overview

**Application:** OWASP Juice Shop

**Environment:** Local Docker Homelab

**Severity:** High (preliminary assessment)

**Vulnerability:** Mass Assignment / Privilege Escalation

**Affected Endpoint:** `POST /api/Users/`

## Description

During security testing, I discovered that the user
registration endpoint accepts an additional `role`
property.

By adding `"role": "admin"` to the registration
request, I was able to create a new account with
administrator privileges.

## Steps to Reproduce

1. Open OWASP Juice Shop and register a new test account.
2. Intercept the registration request using Burp Suite.
3. Send the request to Burp Repeater.
4. Add the following property to the JSON body:

   ```json
   "role": "admin"
   ```

5. Send the modified registration request.
6. Log in with the newly created account.
7. Inspect the JWT payload and verify the assigned role.
8. Navigate to `/#/administration`.

## Results

The registration request returned:

```http
HTTP/1.1 201 Created
```

The response contained:

```json
{
  "id": 27,
  "role": "admin",
  "isActive": true
}
```

After logging in, I confirmed that the JWT
contained the administrator role.

I was also able to access the administration
panel and view other users' email addresses.

## Security Impact

An unauthenticated user can create an account
with administrator privileges.

This may allow unauthorized access to
administrative functionality and sensitive
user information.

## Root Cause

The registration endpoint accepts a security-sensitive
property supplied by the client.

The server does not sufficiently restrict which
properties can be assigned during registration.

## Recommendation

Use an allowlist of accepted registration fields.

Assign the default user role on the server.

Do not allow clients to control privileged
properties such as user roles.

Enforce authorization checks on all
administrative API endpoints.

## Conclusion

This test demonstrated how Mass Assignment can
lead to privilege escalation.

The vulnerability was reproduced in my local
OWASP Juice Shop environment for educational
security testing.
