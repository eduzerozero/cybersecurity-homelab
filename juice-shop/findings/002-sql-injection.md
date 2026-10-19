# Finding 002 – SQL Injection Authentication Bypass

## Overview

**Application:** OWASP Juice Shop

**Environment:** Local Docker Homelab

**Severity:** Critical (preliminary assessment)

**Vulnerability:** SQL Injection / Authentication Bypass

**Affected Endpoint:** `POST /rest/user/login`

## Description

During my security testing of OWASP Juice Shop,
I discovered a SQL Injection vulnerability in
the login functionality.

By manipulating the email parameter in a login
request, I was able to bypass authentication
and log in as an administrator without knowing
the administrator's password.

## Steps to Reproduce

1. Open OWASP Juice Shop in the browser.
2. Navigate to the login page.
3. Intercept the login request using Burp Suite.
4. Send the request to Burp Repeater.
5. Modify the email parameter using the following payload:

   ```text
   ' OR 1=1--
   ```

6. Send the modified request to the server.
7. Inspect the HTTP response and authentication token.

## HTTP Request

The modified request was sent to:

```http
POST /rest/user/login HTTP/1.1
Content-Type: application/json
```

Example request body:

```json
{
  "email": "' OR 1=1--",
  "password": "test"
}
```

## Results

The server responded with:

```http
HTTP/1.1 200 OK
```

The response contained an authentication token
for the administrator account:

```text
admin@juice-sh.op
```

OWASP Juice Shop also displayed a successful
challenge notification.

This confirmed that the login authentication
could be bypassed using SQL Injection.

## Security Impact

An attacker may be able to bypass the normal
authentication process without knowing a valid
password.

In this test, the vulnerability allowed access
to an administrator account.

This could lead to unauthorized access to
sensitive information and administrative
functionality.

## Root Cause

The observed behavior is consistent with unsafe
SQL query construction using user-controlled input.

The exact vulnerable source code has not yet
been reviewed in this investigation.

## Recommendation

Use parameterized SQL queries or prepared
statements instead of concatenating user input
into SQL queries.

Validate user input and handle database errors
without exposing internal information.

Apply least-privilege permissions to database
accounts.

## Conclusion

This test demonstrated how SQL Injection can
be used to bypass authentication in a vulnerable
web application.

I successfully reproduced the vulnerability
using Burp Suite in my local OWASP Juice Shop
environment.

The investigation helped me understand the
importance of secure database queries and
server-side authentication.
