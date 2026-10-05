#OWASP Juice Shop Lab


## Overview

This project is a part of my personal Cybersecurity Home Lab.

With this project, i want to practice my skills in web application security,
Docker and  my penetration testing.

## Lab Environment

-Raspberry Pi
-Docker
-OWASP Juice Shop
-Kali Linux
-Local home network
 
## Architecture

Kali Linux
    |
    | Security Testing
    |
    v
Raspberry Pi
    |
    v
Docker
    |
    v
OWASP Juice Shop
Port 3000

## Planned Security Tests

I plan to use this lab to learn:

-Network reconnaissance with Nmap
-HTTP request analysis
-Burp Suite
-Authentication vulnerabilities
-Cross-Site Scipting (XSS)
-Injection vulnerabilities
-Broken Access Control


## Initial Reconnaissance

The first step was to verify that the Raspberry Pi was reachable from my Kali Linux machine.

### Connectivity Test

I first used `ping` to check whether the Raspberry Pi was reachable on my local network.

    ping <LAB-IP>

This confirmed that communication between my Kali Linux machine and the Raspberry Pi was possible.

### Port and Service Discovery

Next, I used Nmap to check whether TCP port 3000, which is used by the OWASP Juice Shop container, was reachable.

    nmap -sV -p 3000 <LAB-IP>

The scan showed that port 3000/tcp was open.

After that, I experimented with additional Nmap options to become more familiar with the tool and understand how different scan options affect the results.

One of the commands I tested was:

    nmap -sV --version-all -p 3000 <LAB-IP>

The `-sV` option enables service/version detection, while `--version-all` tells Nmap to try all available version detection probes.

Interestingly, while experimenting with Nmap and service detection, I also completed one of the challenges in OWASP Juice Shop.


## HTTP Analysis with Burp Suite

After testing the application with Nmap, I started using Burp Suite to get more information about the login process.

I used Burp Suite to inspect the HTTP communication between my browser and OWASP Juice Shop.

First, I navigated to the login page and looked at the requests that appeared in the Burp Suite HTTP history.

After that, I created a test account to understand what happens when a user registers on the website.

While creating the account, I used Burp Suite to inspect:

- the HTTP request
- the request method (GET, POST, etc.)
- the API endpoint
- the data sent in the request body
- the HTTP status code returned by the server

My goal was not only to complete a challenge, but to understand how the browser communicates with the Juice Shop backend and how user data is sent to the API.


## Login and Authentication Analysis

After creating a test account, I wanted to take a closer look at the login process of OWASP Juice Shop.

I used Burp Suite to compare a failed login attempt with a successful login attempt.

### Failed Login

First, I intentionally used wrong login credentials to see how the application responds to an unsuccessful login.

With Burp Suite, I inspected the request and response and looked at:

- the HTTP method
- the API endpoint
- the request body
- the response
- the HTTP status code

### Successful Login

After that, I logged in with the correct credentials and compared the successful request and response with the failed login attempt.

One interesting difference I noticed was that after a successful login, the application returns an authentication token.

I observed that authentication-related information is stored by the application in the browser, including in cookies and local storage.

This was interesting because it showed me how a web application can keep track of an authenticated user after the login request is completed.


## Authentication Token Analysis

After comparing successful and failed login requests, I noticed that OWASP Juice Shop returns an authentication token after a successful login.

I wanted to understand what this token contains and how the application uses it.

### Testing Authentication with Burp Repeater

First, I used Burp Suite Repeater to manually send requests to the Juice Shop API.

I tested a request to the basket endpoint:

    GET /rest/basket/6

With a valid authentication token, the request was accepted by the application.

Then I removed the token from the Authorization header:

    Authorization: Bearer

The server responded with:

    HTTP/1.1 401 Unauthorized

After that, I tried using an invalid token:

    Authorization: Bearer abc123

The server again responded with:

    HTTP/1.1 401 Unauthorized

This helped me understand that the application checks the authentication token before allowing access to protected resources.

## JWT Analysis

While investigating the authentication token, I noticed that the token consists of three parts separated by dots.

I decoded the token and found out that it is a JSON Web Token (JWT).

A JWT consists of:

    HEADER.PAYLOAD.SIGNATURE

### Header

After decoding the header, I received:

    {
        "typ": "JWT",
        "alg": "RS256"
    }

The `typ` field identifies the token as a JWT.

The `alg` field shows that RS256 is used for the signature.

### Payload

Next, I decoded the payload.

The payload contained information about my test account, including:

    {
        "data": {
            "id": <USER-ID>,
            "email": "<TEST-EMAIL>",
            "password": "<REDACTED>",
            "role": "customer",
            "isActive": true
        },
        "bid": <BASKET-ID>,
        "iat": <TIMESTAMP>
    }

One important thing I learned is that the JWT payload is not encrypted. The information inside the payload can be decoded and read.

I also noticed that my user role was stored inside the token:

    "role": "customer"

This was interesting because user roles can be important when an application decides which resources a user is allowed to access.

### Signature

When I decoded the signature, I did not receive readable JSON like with the header and payload.

Instead, the result contained binary data.

This helped me understand that the signature has a different purpose than the header and payload. It is used as part of the cryptographic verification of the token.
