# Finding 003 – Broken Access Control (IDOR)

## Overview

**Application:** OWASP Juice Shop

**Environment:** Local Docker Homelab

**Severity:** High (preliminary assessment)

**Vulnerability:** Broken Access Control / Insecure Direct Object Reference (IDOR)

**Affected Endpoint:** `GET /rest/basket/:id`

## Description

During my security testing of OWASP Juice Shop,
I discovered a Broken Access Control vulnerability
in the shopping basket functionality.

By modifying the basket ID in an HTTP request,
I was able to access another user's basket
while using my own authentication token.

This vulnerability is also known as an
Insecure Direct Object Reference (IDOR).

## Steps to Reproduce

1. Open OWASP Juice Shop and log in with a test account.
2. Open Burp Suite and navigate to HTTP History.
3. Find a request to the shopping basket endpoint.
4. Send the request to Burp Repeater.
5. Keep the original JWT authentication token.
6. Send the following request to access my own basket:

   ```http
   GET /rest/basket/6 HTTP/1.1
   Authorization: Bearer <JWT_TOKEN>
   ```

7. Change the basket ID from `6` to `5`:

   ```http
   GET /rest/basket/5 HTTP/1.1
   Authorization: Bearer <JWT_TOKEN>
   ```

8. Send the modified request and inspect the response.

## Results

The request to my own basket returned:

```http
HTTP/1.1 200 OK
```

The response contained basket information
associated with my test account.

After changing the basket ID to `5`,
the server also responded with:

```http
HTTP/1.1 200 OK
```

I was able to view basket information
belonging to another user without changing
my authentication token.

I also tested the endpoint without a valid
JWT token.

The server returned:

```http
HTTP/1.1 401 Unauthorized
```

This showed that authentication was required,
but the application did not properly verify
whether the authenticated user was authorized
to access the requested basket.

## Security Impact

An authenticated attacker may be able to access
shopping basket information belonging to
other users.

Depending on the data stored in the basket,
this could expose private information
and violate user confidentiality.

The test confirmed unauthorized read access.
Modification of another user's basket was
not tested.

## Root Cause

The observed behavior indicates that the
application does not correctly enforce
object-level authorization for the requested
basket ID.

The server accepts a user-controlled basket
identifier without sufficiently checking
whether the basket belongs to the authenticated
user.

## Recommendation

Implement server-side authorization checks
for every request to a shopping basket.

Verify that the authenticated user owns
the requested basket before returning data.

Do not rely on client-side checks or
the secrecy of object identifiers.

Use automated tests to verify that users
cannot access objects belonging to
other accounts.

## Conclusion

This test demonstrated how Broken Access
Control can expose information belonging
to other users.

I successfully reproduced an IDOR vulnerability
using Burp Suite in my local OWASP Juice Shop
environment.

I learned that authentication alone is not
enough to protect user data.

Applications must also verify that each
authenticated user is authorized to access
the requested resource.
