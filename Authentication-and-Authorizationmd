# Authentication & Authorization

## The History of How the Web Learned Who You Are and What You Can Do

Before moving deep into Authentication and Authorization, we need to step back and understand their history.

Most developers learn authentication like this:

```text
JWT
Cookies
Sessions
OAuth
RBAC
```

But this teaches us **what the technologies are**, not **why they were created**.

A better way to understand backend security is to follow the history:

```text
Problem
   ↓
First Solution
   ↓
New Limitation
   ↓
Next Solution
   ↓
New Limitation
   ↓
Evolution
```

Authentication and authorization evolved because every previous solution solved one problem and introduced another trade-off.

---

# 1. The First Problem — HTTP Does Not Remember You

Before authentication, we need to understand an important property of HTTP.

HTTP is fundamentally stateless.

Consider these two requests:

```http
GET /profile
```

and:

```http
GET /orders
```

We know that both requests may have been sent by the same person.

For example:

```text
Request 1 → user1
Request 2 → user1
```

But HTTP does not automatically remember that relationship.

From the protocol's point of view, they are separate requests.

```text
Request 1
    ↓
Response

Request 2
    ↓
Response
```

The second request does not automatically know anything about the first request.

HTTP provides communication between a client and a server.

It does not automatically maintain application-level identity between requests.

This behavior is commonly described as:

> HTTP is stateless.

Therefore, the first problem becomes:

> **How can we make several independent HTTP requests belong to the same authenticated user?**

That single question is the beginning of the history of web authentication.

---

# 2. The First Idea — Send the Credentials Every Time

The simplest solution is:

> Let the client introduce itself on every request.

For example:

```http
GET /profile

username=tamil
password=secret
```

Then:

```http
GET /orders

username=tamil
password=secret
```

The server can now identify the user for every request.

The problem is immediately visible.

We have solved:

```text
How does the server know who I am?
```

But we created another problem:

```text
Why should I keep sending my password?
```

A password is a long-term secret.

It should not become a credential that is repeatedly transmitted throughout the application.

So the next question becomes:

> **Can we authenticate once and then use something else for future requests?**

This question leads to sessions.

---

# 3. The Birth of Sessions

The next idea was:

> **Let the server remember that the user has already authenticated.**

The login flow becomes:

```text
Browser
   |
   | username + password
   v
Server
   |
   | validate credentials
   v
Create Session
```

Suppose the server creates:

```text
sessionId = abc123
```

The server stores:

```text
abc123 → user1
```

The browser receives the session identifier, normally through a cookie.

For example:

```http
Set-Cookie: sid=abc123
```

Then future requests contain:

```http
GET /profile
Cookie: sid=abc123
```

The server can now perform:

```text
abc123
   ↓
Find session
   ↓
user1
```

The browser no longer needs to repeatedly send:

```text
username
password
```

Instead, it sends:

```text
session identifier
```

---

# 4. What a Session Actually Means

A common beginner mistake is thinking:

```text
Cookie = Session
```

They are related, but they are not the same thing.

A cookie is a mechanism used by the browser to store and send data.

A session is authentication state maintained by the server.

For example:

```text
Cookie:

sid=abc123
```

The server may store:

```text
abc123 → {
    userId: 42,
    role: "admin"
}
```

So:

```text
Cookie
   ↓
Session Identifier
   ↓
Server-side Session
   ↓
User Identity
```

The important idea is that the cookie can simply be a pointer to state stored somewhere on the server.

---

# 5. Authentication Became Stateful

Now something important happened.

The server started remembering information about users.

For example:

```text
abc123 → user42
```

That means the server has state.

The server now needs to maintain things such as:

```text
session ID
user ID
session expiration
roles
session validity
logout state
```

Therefore session-based authentication is commonly called:

> **Stateful Authentication**

The architecture looks like:

```text
Browser
   |
   | session ID
   v
Application Server
   |
   | lookup
   v
Session Store
   |
   v
User
```

This solves the original problem very well.

But the web kept growing.

---

# 6. The Web Became Distributed

For a small application, one server may be enough.

But imagine the application becomes popular.

One server is no longer enough.

We may now have:

```text
                 Load Balancer
                      |
          ┌───────────┼───────────┐
          |           |           |
       Server A    Server B    Server C
```

The load balancer distributes incoming requests across multiple servers.

Now imagine the user logs in.

The login request reaches Server A:

```text
Browser
   |
Load Balancer
   |
Server A
```

Server A creates:

```text
abc123 → user42
```

Suppose Server A stores that session only in its own memory:

```text
Server A
└── abc123 → user42
```

Now the next request reaches Server C:

```text
Browser
   |
Load Balancer
   |
Server C
```

Server C receives:

```text
Cookie: sid=abc123
```

But Server C asks:

> What does `abc123` mean?

Server C does not know.

The session lives inside Server A.

Now we have a new problem:

> **How can multiple application servers share authentication state?**

---

# 7. Sticky Sessions

One possible solution is to keep the user attached to the same server.

For example:

```text
user42 → Server A
```

The load balancer remembers this relationship.

The flow becomes:

```text
User
  |
  v
Load Balancer
  |
  v
Server A
```

Future requests continue going to Server A.

This is usually called:

> **Sticky Sessions**

It can work.

But now imagine Server A crashes.

```text
Server A
    X
  crashed
```

The user's session state may disappear if it only existed in Server A's memory.

The load balancer may send the next request to Server B:

```text
User
  |
  v
Server B
```

But Server B does not know the session.

The user may suddenly appear logged out.

Therefore sticky sessions solve one problem but create a dependency on a specific application server.

Distributed systems usually try to avoid this.

So we need another solution.

---

# 8. Shared Session Storage

Instead of storing session state inside one application server, we can move the session state into a shared store.

For example:

```text
Server A ─┐
Server B ─┼──> Redis
Server C ─┘
```

Now the login flow becomes:

```text
Browser
   |
   v
Load Balancer
   |
   v
Server A
   |
   | create session
   v
Redis
```

Redis stores:

```text
abc123 → user42
```

Later, the request reaches Server C:

```text
Browser
   |
   v
Load Balancer
   |
   v
Server C
```

Server C receives:

```text
sid=abc123
```

and asks Redis:

```text
Who owns abc123?
```

Redis answers:

```text
user42
```

Now any application server can authenticate the request.

---

# 9. Why Redis Is Useful

Authentication can happen on almost every protected request.

That means session lookup can become extremely frequent.

A shared store such as Redis is useful because it is designed for fast data access.

A simplified session could look like:

```text
session:abc123
    ↓
{
    userId: 42,
    role: "admin"
}
```

We can also associate expiration information:

```text
session:abc123
TTL = 1800 seconds
```

When the session expires, it can automatically become invalid.

The architecture is now:

```text
                Load Balancer
                     |
          ┌──────────┼──────────┐
          |          |          |
       Server A   Server B   Server C
          |          |          |
          └──────────┼──────────┘
                     |
                   Redis
```

This solves the distributed session problem.

But we now have another architectural dependency.

---

# 10. The Shared-State Problem

Every authenticated request may now look like:

```text
Client
  |
  v
Application Server
  |
  v
Redis
  |
  v
Session
  |
  v
User
```

The application server must ask another system:

> Who is this session?

That is not necessarily bad.

In fact, this architecture is widely useful.

But it creates shared state.

Now imagine:

```text
100 application servers
10 Redis nodes
multiple regions
millions of users
```

The authentication system becomes dependent on a shared state layer.

So engineers started asking a different question:

> **Can the identity information travel with the request itself?**

That question takes us toward tokens.

---

# 11. Before Tokens — We Need to Understand Password Storage

There is another important part of authentication.

What should the database store after the user creates a password?

Definitely not:

```text
password = mySecret123
```

Passwords should not be stored in plaintext.

Instead, a password is transformed using a password hashing function.

```text
Password
   |
   v
Password Hashing Function
   |
   v
Stored Hash
```

During login:

```text
User Password
      |
      v
Password Verification
      |
      v
Compare with Stored Hash
```

---

# 12. Hashing vs Encryption

Hashing and encryption are different.

Encryption is designed to be reversible:

```text
Plaintext
   |
   v
Encryption
   |
   v
Ciphertext
   |
   v
Decryption
   |
   v
Plaintext
```

Hashing is intended to be one-way:

```text
Password
   |
   v
Hash Function
   |
   v
Hash
```

You normally do not decrypt a password hash.

Instead, you verify a candidate password against the stored hash.

---

# 13. Why Fast Hashing Is Bad for Passwords

General-purpose cryptographic hash functions such as SHA-256 are intentionally very fast.

That is useful for many cryptographic tasks.

But password hashing has a different requirement.

We want guessing passwords to be expensive.

Suppose an attacker steals a database containing password hashes.

They can attempt:

```text
password1
password2
password3
password4
...
```

If the hashing algorithm is extremely fast, the attacker can perform a huge number of guesses.

Password-specific algorithms are deliberately expensive.

Examples include:

```text
Argon2id
scrypt
bcrypt
```

These algorithms allow the cost to be tuned.

The goal is:

```text
Legitimate login
        ↓
Reasonable cost

Attacker guessing millions of passwords
        ↓
Very expensive
```

---

# 14. Salt

Passwords should normally be stored with a unique random salt.

Conceptually:

```text
password
   +
random salt
   |
   v
Password KDF
   |
   v
hash
```

For two users with the same password:

```text
User A
password = hello
salt = X
hash = H1
```

and:

```text
User B
password = hello
salt = Y
hash = H2
```

The resulting hashes can be different because the salts are different.

The salt does not need to be secret.

Its purpose is to make each password hash unique and defend against precomputed attacks such as rainbow tables.

---

# 15. Pepper

A pepper is different from a salt.

A pepper is a secret value that is kept separately from the password database.

Conceptually:

```text
password
   +
salt
   +
pepper
   |
   v
Password KDF
   |
   v
hash
```

The database may contain:

```text
salt
hash
parameters
```

while the pepper lives in a separate secret-management system.

The goal is to make a database-only compromise less useful to an attacker.

---

# 16. Work Factor

Modern password hashing functions allow the cost of hashing to be increased.

For example:

```text
Argon2
  ├── memory cost
  ├── time cost
  └── parallelism
```

As hardware becomes faster, the cost can be adjusted.

This creates an important security principle:

> **Password hashing should be intentionally expensive, but still practical for legitimate users.**

---

# 17. Returning to the Main Problem

We now have:

```text
HTTP
   ↓
Authentication Problem
   ↓
Password
   ↓
Session
   ↓
Stateful Authentication
   ↓
Distributed Systems
   ↓
Shared Session Store
```

But now another question appears:

> **Can we make the client carry enough information to identify itself without requiring a session lookup every time?**

At first glance, the idea seems simple.

We could send:

```json
{
    "userId": 42,
    "role": "admin"
}
```

But there is a huge problem.

The client controls the request.

---

# 18. Never Trust the Client

Suppose the server receives:

```json
{
    "userId": 42,
    "role": "user"
}
```

The attacker can change it:

```json
{
    "userId": 42,
    "role": "admin"
}
```

If the server trusts this data, the attacker has elevated their privileges.

So we need:

```text
Identity Information
       +
Proof that it has not been modified
```

That is where cryptographic signatures become important.

---

# 19. The Idea of Signing Data

Suppose the server wants to send:

```text
userId = 42
role = admin
```

The server has a signing key.

It computes a cryptographic signature over the data.

Conceptually:

```text
Data
 +
Signing Key
      |
      v
Signature
```

The client receives:

```text
Data + Signature
```

Later, the client sends both back.

The server can verify:

```text
Does this signature actually belong to this data?
```

If the attacker changes:

```text
role = admin
```

to:

```text
role = superadmin
```

the original signature no longer matches.

The attacker can modify the data.

But without the signing key, they cannot create a valid signature for the modified data.

This gives us a powerful principle:

> **The client may carry the information, but the client must not be able to forge the proof.**

---

# 20. From Session IDs to Signed Information

Now the evolution looks like this:

```text
Password
   ↓
Password on every request
   ↓
Sessions
   ↓
Server-side state
   ↓
Distributed Systems
   ↓
Shared Session Store
   ↓
Question:
"Can identity travel with the request?"
   ↓
Cryptographic Signatures
   ↓
Signed Tokens
```

This leads us toward JWT.

---

# 21. JWT — When Identity Started Travelling

JWT stands for:

> JSON Web Token

A JWT is a compact token containing claims that can be cryptographically signed.

The important idea is:

```text
Identity / Claims
       +
Cryptographic Signature
       ↓
Token
```

Instead of sending only:

```text
abc123
```

the client can carry a signed collection of claims.

The server verifies the token.

This can remove the need for a session lookup just to discover the identity contained in the token.

---

# 22. Anatomy of a JWT

A JWT has three parts:

```text
HEADER.PAYLOAD.SIGNATURE
```

For example:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiI0MiIsInJvbGUiOiJhZG1pbiJ9
.
signature
```

The three parts are:

```text
1. Header
2. Payload
3. Signature
```

Each part has a different purpose.

---

# 23. The Header

Example:

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

The header contains metadata about the token.

Common fields include:

```text
alg → algorithm
typ → token type
```

The algorithm identifies the cryptographic algorithm associated with the signature.

However, a secure verifier should not blindly accept whatever algorithm the token claims to use.

The application should explicitly configure which algorithms are allowed.

For example:

```text
Allowed Algorithms:
RS256
```

instead of:

```text
Use whatever the token says
```

---

# 24. The Payload

The payload contains claims.

Example:

```json
{
    "sub": "42",
    "role": "admin",
    "iat": 1757000000,
    "exp": 1757003600
}
```

Common claims include:

```text
sub → subject
iat → issued at
exp → expiration
```

Application-specific claims might include:

```text
role
permissions
tenant
organization
```

A claim is simply a statement about the token or the subject represented by it.

---

# 25. The `sub` Claim

`sub` means:

> Subject

In authentication systems, it is commonly used to represent the identity associated with the token.

For example:

```json
{
    "sub": "42"
}
```

can mean:

```text
This token represents user 42.
```

The exact meaning of `sub` depends on the application, but it should identify the subject consistently.

---

# 26. The `iat` Claim

`iat` means:

> Issued At

Example:

```json
{
    "iat": 1757000000
}
```

It tells us when the token was issued.

This can be useful when implementing policies such as:

```text
Reject tokens created before a security reset.
```

---

# 27. The `exp` Claim

`exp` means:

> Expiration Time

Example:

```json
{
    "exp": 1757003600
}
```

Once the current time is past the expiration time:

```text
Token
   ↓
Expired
   ↓
Reject
```

Short token lifetimes limit the useful lifetime of stolen tokens.

---

# 28. JWT Payloads Are Not Secret

This is one of the most important JWT concepts.

A normal JWT is signed.

It is not automatically encrypted.

Therefore:

```text
JWT
 ≠
Encrypted JSON
```

The payload is typically encoded using Base64URL.

Anyone holding the token can decode the payload.

Therefore do not put secrets inside it.

Bad:

```json
{
    "password": "secret123",
    "creditCard": "..."
}
```

Better:

```json
{
    "sub": "42",
    "role": "admin",
    "exp": 1757003600
}
```

The goal is to keep the token focused on identity and authorization-related claims.

---

# 29. Base64URL Is Encoding, Not Encryption

The JWT parts are represented using Base64URL encoding.

Conceptually:

```text
JSON
   ↓
Base64URL Encoding
   ↓
Encoded Data
```

Base64URL is not encryption.

So:

```text
Encoding
    ≠
Encryption
```

The actual security property comes from the cryptographic signature.

---

# 30. The Signature

The signature is the cryptographic proof attached to the token.

Conceptually:

```text
Base64URL(Header)
        +
"."
        +
Base64URL(Payload)
        |
        v
Signing Algorithm
        +
Signing Key
        |
        v
Signature
```

The final token becomes:

```text
HEADER.PAYLOAD.SIGNATURE
```

When the server receives the token later, it verifies that the signature matches the header and payload.

---

# 31. The Tampering Example

Original token claims:

```json
{
    "sub": "42",
    "role": "user"
}
```

The attacker wants:

```json
{
    "sub": "42",
    "role": "admin"
}
```

They can modify the payload.

But the original signature corresponds to:

```text
sub = 42
role = user
```

It does not correspond to:

```text
sub = 42
role = admin
```

So verification fails.

```text
Modified Payload
       +
Original Signature
       ↓
Verification
       ↓
FAIL
```

This is the heart of signed JWT authentication.

---

# 32. The JWT Login Flow

The complete process looks like this:

```text
Client
   |
   | username + password
   v
Authentication Server
   |
   | verify password
   v
User Identity
   |
   | create claims
   v
Sign JWT
   |
   v
Return JWT
```

The client now stores or otherwise obtains the token.

---

# 33. JWT on a Future Request

The client sends:

```http
GET /profile
Authorization: Bearer <JWT>
```

The server receives the request.

The authentication layer can:

```text
1. Extract the token
2. Parse its structure
3. Verify the signature
4. Check the allowed algorithm
5. Validate expiration
6. Validate required claims
7. Establish the user's identity
```

If everything is valid:

```text
JWT
 ↓
user42
```

Then the request continues.

---

# 34. JWT Middleware

Authentication is usually implemented as middleware because many routes need the same logic.

For example:

```text
Request
   |
   v
Authentication Middleware
   |
   v
Controller
```

The middleware can:

```text
Extract token
      ↓
Verify token
      ↓
Identify user
      ↓
Attach identity to request
      ↓
Continue
```

If authentication fails:

```text
Request
   |
   v
Authentication Middleware
   |
   X
401 Unauthorized
```

The controller is never called.

---

# 35. Request Context

After authentication, the server may attach:

```text
request.user = {
    id: 42,
    role: "admin"
}
```

Now downstream code does not need to decode the JWT again.

It can simply use:

```text
request.user.id
request.user.role
```

The controller does not need to know whether the identity came from:

```text
Session
JWT
OAuth
API Key
```

It only needs the authenticated identity.

---

# 36. Stateful vs Stateless Authentication

Now we can clearly compare the two.

## Stateful Authentication

```text
Client
  |
  | Session ID
  v
Server
  |
  | Lookup
  v
Session Store
  |
  v
User
```

The server remembers authentication state.

---

## Stateless JWT Authentication

```text
Client
  |
  | JWT
  v
Server
  |
  | Verify
  v
Claims / Identity
```

The token carries the claims required for identity verification.

The server does not need a session record for every token simply to establish the identity represented by that token.

---

# 37. Why Stateless Authentication Was Attractive

Imagine:

```text
             Load Balancer
                  |
       ┌──────────┼──────────┐
       |          |          |
    Server A   Server B   Server C
```

With sessions:

```text
Server A ─┐
Server B ─┼──> Session Store
Server C ─┘
```

Each server may need access to shared state.

With a JWT:

```text
Server A → verify JWT
Server B → verify JWT
Server C → verify JWT
```

This allows each application server to independently validate the token.

This can make horizontal scaling and distributed architectures easier.

But we have not eliminated trade-offs.

---

# 38. The JWT Logout Problem

Suppose the token is valid for:

```text
1 hour
```

The user logs in.

They receive:

```text
JWT123
```

Then they click:

```text
Logout
```

What does the server delete?

With a session:

```text
sessionId = abc123
```

the server can delete:

```text
abc123
```

Done.

But with a purely stateless JWT:

```text
JWT123
```

there may be no server-side record to delete.

The token may remain valid until:

```text
exp
```

Therefore:

> **JWT makes revocation more difficult than stateful sessions.**

This is one of the most important JWT trade-offs.

---

# 39. Token Theft

Suppose an attacker steals a valid JWT.

```text
User
  |
  | JWT
  v
Attacker
```

The attacker sends:

```http
Authorization: Bearer <stolen-token>
```

The server verifies:

```text
Signature ✓
Expiration ✓
Claims ✓
```

The server may not know that the token is now being used by the attacker.

This means:

> **A stolen bearer token can often be used until it expires or is otherwise invalidated.**

This is why token lifetime and token storage matter.

---

# 40. Short-Lived Access Tokens

A common response is to reduce the lifetime of access tokens.

For example:

```text
Access Token
   ↓
15 minutes
```

Now if the token is stolen, the attacker has less time to use it.

But we immediately create another usability problem.

If the access token expires every 15 minutes, users would constantly need to log in again.

We need another mechanism.

That leads to refresh tokens.

---

# 41. Refresh Tokens

A common architecture becomes:

```text
Login
  |
  ├── Access Token
  |      ↓
  |   Short-lived
  |
  └── Refresh Token
         ↓
      Longer-lived
```

The access token is used to access APIs.

When it expires:

```text
Client
   |
   | Refresh Token
   v
Authentication Server
   |
   | new Access Token
   v
Client
```

The user does not need to type their password again.

---

# 42. Why Use Two Tokens?

Because the security requirements are different.

The access token is used frequently:

```text
Client → API
Client → API
Client → API
```

So making it short-lived reduces the risk window.

The refresh token is used less often:

```text
Client
   |
   | refresh
   v
Auth Server
```

This creates a balance:

```text
Short-lived API credential
        +
Longer-lived renewal credential
```

---

# 43. Refresh Token Rotation

Refresh tokens should also be managed carefully.

A common approach is rotation.

For example:

```text
Refresh Token A
       |
       v
used successfully
       |
       v
Refresh Token B
```

The old refresh token becomes invalid.

Now imagine an attacker tries to reuse Token A.

The server can detect:

```text
Refresh Token A
       ↓
already rotated
       ↓
reuse detected
```

This can be used as a signal that the refresh token may have been stolen.

---

# 44. HS256 — Symmetric Signing

JWTs can use symmetric signing algorithms.

For example:

```text
HS256
```

The same secret is used to:

```text
Sign
Verify
```

Conceptually:

```text
             Secret
              /  \
             /    \
          Sign    Verify
```

This is simple.

But it introduces a trust issue.

Any service that possesses the shared secret can potentially create tokens.

Imagine:

```text
Service A ─┐
Service B ─┤
Service C ─┼──> SAME SECRET
Service D ─┤
Service E ─┘
```

A compromise of one service may therefore affect the token trust boundary.

---

# 45. RS256 — Asymmetric Signing

Now consider asymmetric cryptography.

The system has:

```text
Private Key
Public Key
```

The private key signs:

```text
Private Key
     |
     v
   SIGN
```

The public key verifies:

```text
Public Key
     |
     v
  VERIFY
```

The architecture becomes:

```text
                 Private Key
                     |
                     v
             Authentication Server
                     |
                     | Signed JWT
                     v
        ┌────────────┼────────────┐
        |            |            |
      User         Order        Payment
     Service       Service       Service
        |            |            |
        └────────────┼────────────┘
                     |
                 Public Key
```

Other services can verify tokens without receiving the private key.

This creates a useful separation:

```text
Issuer
   ↓
Can sign

Verifier
   ↓
Can verify
```

That is especially useful in distributed systems.

---

# 46. Why Algorithm Pinning Matters

A token contains an `alg` field.

It might say:

```json
{
    "alg": "RS256"
}
```

But a secure verifier should not blindly trust the token.

The application should explicitly configure:

```text
Allowed algorithm:
RS256
```

The verifier should then reject tokens using unexpected algorithms.

Therefore JWT verification is not:

```text
decode(token)
```

It is:

```text
Parse
 +
Algorithm Validation
 +
Signature Verification
 +
Claim Validation
```

---

# 47. JWT Is Not the Same as a Cookie

Another common misunderstanding:

```text
JWT = Cookie
```

This is incorrect.

A cookie is a browser mechanism.

JWT is a token format.

They can be used together.

For example:

```text
Cookie
   ↓
Contains JWT
```

Or:

```text
Authorization Header
   ↓
Contains JWT
```

So:

```text
Cookie = transport/storage mechanism
JWT    = credential/token format
```

---

# 48. Cookie Security

Authentication cookies may use:

```text
HttpOnly
Secure
SameSite
```

These attributes serve different purposes.

### HttpOnly

`HttpOnly` prevents JavaScript from directly reading the cookie through browser APIs such as `document.cookie`.

This can reduce the ability of injected scripts to directly extract authentication cookies.

### Secure

`Secure` tells the browser to send the cookie only over secure HTTPS connections.

### SameSite

`SameSite` controls how cookies are sent in cross-site contexts.

This is useful for reducing certain cross-site request risks.

The correct setting depends on the application's architecture.

---

# 49. CSRF

CSRF means:

> Cross-Site Request Forgery

Suppose the browser automatically sends an authentication cookie.

A malicious website may attempt to trick the user's browser into making a request to another website where the user is authenticated.

Conceptually:

```text
User Browser
   |
   | automatically includes cookie
   v
Target Application
```

The attacker wants the browser to perform an authenticated action without the user's intentional interaction with the target application.

This is why cookie-based authentication often needs CSRF protections.

Possible defenses include:

```text
SameSite cookies
CSRF tokens
Origin / Referer validation
```

---

# 50. Session Fixation

Another session-related attack is session fixation.

The general idea:

```text
Attacker knows session ID
        ↓
Victim authenticates
        ↓
Server keeps the same session ID
        ↓
Attacker may know authenticated session
```

The defense is to regenerate the session identifier after successful authentication.

For example:

```text
Before login:
session = abc123

After login:
session = xyz789
```

The old session identifier should no longer be usable.

---

# 51. API Keys

Not every client is a human.

Sometimes another application needs to call your API.

For example:

```text
Application A
      |
      | API request
      v
Application B
```

There may be no browser.

There may be no login form.

A simple API key can identify the calling application.

For example:

```http
X-API-Key: <secret>
```

The server associates that credential with:

```text
Application
Permissions
Quota
Status
```

API keys are useful for many machine-to-machine integrations.

---

# 52. API Keys Are Credentials

An API key is not just an identifier.

It is a credential.

Therefore:

```text
Do not log it casually
Rotate it
Protect it
Restrict its permissions
Expire it when appropriate
```

Treat API keys like secrets.

---

# 53. The Delegation Problem

Now imagine another scenario.

You are building an application.

You want to access a user's Google account.

The terrible approach would be:

```text
User
   |
   | Google password
   v
Your Application
```

Now your application knows the user's password.

That is extremely dangerous.

The application may potentially gain far more access than it actually needs.

We need a better idea:

> **Can the user give the application limited permission without giving the application the password?**

This is the delegation problem.

And it leads to OAuth.

---

# 54. OAuth

OAuth is primarily an authorization framework.

Its core idea is:

> **Delegate limited access without sharing the user's password with the client application.**

Conceptually:

```text
User
  |
  | grants permission
  v
Authorization Server
  |
  | access token
  v
Application
  |
  | access token
  v
Resource Server
```

The application receives a credential that represents delegated access.

---

# 55. OAuth Roles

OAuth defines important actors.

### Resource Owner

The person who owns the data.

```text
User
```

### Client

The application requesting access.

```text
Your Application
```

### Authorization Server

The system responsible for authenticating the user and issuing authorization credentials.

```text
Identity Provider / Authorization Server
```

### Resource Server

The API hosting the protected resource.

```text
Protected API
```

---

# 56. OAuth Flow

A simplified flow:

```text
User
  |
  v
Client Application
  |
  | redirect
  v
Authorization Server
  |
  | authenticate user
  |
  | obtain consent
  v
Authorization Code
  |
  v
Client
  |
  | exchange code
  v
Authorization Server
  |
  v
Access Token
  |
  v
Resource Server
```

Notice that the application does not need the user's password.

---

# 57. Scopes

OAuth allows permissions to be limited using scopes.

For example:

```text
read:profile
read:email
read:calendar
write:calendar
```

Instead of:

```text
Full account access
```

the application might receive:

```text
read:email
```

This follows the principle of:

> **Least Privilege**

Give the application only the permissions it needs.

---

# 58. Authorization Code Flow

A modern browser-based authorization flow commonly looks like:

```text
Client
   |
   | authorize
   v
Authorization Server
   |
   | authenticate + consent
   v
Authorization Code
   |
   v
Client
   |
   | token exchange
   v
Authorization Server
   |
   v
Access Token
```

The authorization code is short-lived and intended to be exchanged for tokens.

---

# 59. PKCE

PKCE stands for:

> Proof Key for Code Exchange

It addresses authorization-code interception attacks.

The client first creates:

```text
code_verifier
```

and derives a:

```text
code_challenge
```

The client sends the challenge during authorization.

Later, during the token exchange, it sends the original verifier.

Conceptually:

```text
Client
   |
   | code_challenge
   v
Authorization Server
   |
   | authorization code
   v
Client
   |
   | code + code_verifier
   v
Authorization Server
```

The server checks that the verifier corresponds to the earlier challenge.

If an attacker steals only the authorization code, the attacker cannot redeem it without the verifier.

---

# 60. OAuth and Authentication Are Not the Same

This distinction is extremely important.

OAuth primarily answers:

> **What can this application access?**

Authentication answers:

> **Who is the user?**

OAuth by itself is not designed primarily as a user identity protocol.

That is where OpenID Connect enters the picture.

---

# 61. OpenID Connect

OpenID Connect, or OIDC, adds an identity layer on top of OAuth 2.0.

Conceptually:

```text
OAuth
   +
Identity
   ↓
OpenID Connect
```

OIDC introduces an:

> ID Token

The ID token is commonly a JWT.

It can contain claims about the authenticated user.

For example:

```json
{
    "sub": "12345",
    "iss": "https://issuer.example.com",
    "aud": "client123",
    "iat": 1757000000,
    "exp": 1757003600
}
```

---

# 62. OAuth vs OIDC

Think about the questions:

```text
OAuth:
"What can this application access?"

OIDC:
"Who authenticated?"
```

Therefore:

```text
OAuth  → Authorization
OIDC   → Authentication / Identity
```

---

# 63. Access Token vs ID Token

These should not be confused.

## Access Token

Used to access protected APIs.

```text
Application
    |
    | Access Token
    v
Resource API
```

---

## ID Token

Used to communicate identity information to the client.

```text
Identity Provider
       |
       | ID Token
       v
Client
```

An ID token is not simply a replacement for an API access token.

They serve different purposes.

---

# 64. "Sign In With Google"

When a user presses:

```text
Sign in with Google
```

the high-level flow is:

```text
Your Application
      |
      | Redirect
      v
Google
      |
      | User authenticates
      |
      | User grants consent
      v
Authorization Code
      |
      v
Your Application
      |
      | Exchange code
      v
Google
      |
      v
Tokens
```

OIDC gives the application identity information about the authenticated user.

---

# 65. Authentication vs Authorization

Now return to the two original questions.

Suppose the server knows:

```text
userId = 42
```

Authentication is complete.

The server knows:

```text
Who are you?
```

But it still needs to ask:

```text
What are you allowed to do?
```

That is authorization.

For example:

```text
user42
   |
   ├── View profile      ✓
   ├── Edit own profile  ✓
   ├── Delete users      ✗
   └── Change system settings ✗
```

---

# 66. Authentication Begins Before Authorization

A simplified request pipeline:

```text
Request
   |
   v
Authentication
   |
   v
Who are you?
   |
   v
Identity
   |
   v
Authorization
   |
   v
What can you do?
   |
   v
Permission
```

Therefore:

```text
Authentication = Identity
Authorization  = Permission
```

---

# 67. The Bad Authorization Model

Imagine an application has:

```text
ADMIN_SECRET=SUPER_SECRET
```

and every administrator uses the same secret.

The server checks:

```text
if secret == SUPER_SECRET
    allow
```

This creates a huge problem.

If the secret leaks:

```text
Attacker
   |
   v
SUPER_SECRET
   |
   v
Admin Access
```

There is no individual identity.

There is no fine-grained permission model.

The better idea is to associate permissions with authenticated identities.

That leads to RBAC.

---

# 68. RBAC

RBAC means:

> Role-Based Access Control

Instead of directly assigning every permission to every user, users can be assigned roles.

For example:

```text
User
Moderator
Admin
```

Then each role has permissions.

```text
User
 ├── read profile
 ├── create post
 └── edit own post

Moderator
 ├── everything User has
 └── moderate content

Admin
 ├── everything Moderator has
 └── manage users
```

---

# 69. RBAC Request Flow

Suppose the request is:

```http
DELETE /users/42
```

Authentication first establishes:

```text
userId = 100
role = moderator
```

Authorization then checks:

```text
Can moderator delete users?
```

Result:

```text
NO
```

Therefore:

```http
403 Forbidden
```

---

# 70. 401 vs 403

These two status codes are frequently confused.

## 401 Unauthorized

Usually means:

> The request is not successfully authenticated.

Examples:

```text
No credentials
Invalid token
Expired token
Invalid session
```

Conceptually:

```text
Who are you?
↓
I cannot establish your identity.
```

---

## 403 Forbidden

Means:

> The server knows who you are, but you are not allowed to perform this action.

Conceptually:

```text
Who are you?
↓
user42

Can you delete users?
↓
No.
```

So:

```text
401 → Authentication problem
403 → Authorization problem
```

---

# 71. Role Hierarchy

RBAC can be extended using role inheritance.

For example:

```text
User
  ↑
Moderator
  ↑
Admin
```

An administrator can inherit moderator permissions.

Conceptually:

```text
Admin
  |
  +── Moderator permissions
  |
  +── Admin permissions
```

This can reduce duplicated configuration.

---

# 72. Role Explosion

But eventually roles can become too specific.

Imagine:

```text
editor
editor-finance
editor-hr
editor-legal
editor-finance-business-hours
editor-hr-business-hours
...
```

Roles start representing every combination of conditions.

This is called:

> **Role Explosion**

At this point, role-based authorization alone becomes difficult to manage.

---

# 73. ABAC

ABAC means:

> Attribute-Based Access Control

Instead of checking only a role, the authorization decision can consider several attributes.

For example:

```text
Subject
Resource
Action
Environment
```

---

## Subject

Information about the user:

```text
userId
role
department
clearance
```

---

## Resource

Information about the requested resource:

```text
owner
department
classification
status
```

---

## Action

For example:

```text
read
write
delete
```

---

## Environment

For example:

```text
time
IP
device
location
```

---

# 74. Example ABAC Policy

Suppose a user wants to edit a document.

The policy could be:

```text
Allow if:

user owns the document

OR

user is an editor
AND
user belongs to the document's department

AND

document is not archived

AND

request happens during allowed hours
```

Now authorization depends on more than:

```text
role = editor
```

It depends on context.

This is more powerful but also more complex.

---

# 75. Policy-Based Authorization

At larger organizations, authorization often becomes a policy problem.

Instead of scattering authorization rules throughout application code:

```java
if (user.role.equals("admin")) {
    ...
}
```

we can define policies separately.

Conceptually:

```text
Request
   |
   v
Policy Engine
   |
   | evaluate
   v
Allow / Deny
```

This makes authorization more centralized and auditable.

---

# 76. Authentication Middleware

A common application structure is:

```text
Request
   |
   v
Logging
   |
   v
CORS
   |
   v
Rate Limiting
   |
   v
Authentication
   |
   v
Authorization
   |
   v
Controller
```

Authentication middleware establishes identity.

Authorization middleware checks whether that identity has access.

---

# 77. Middleware Order Matters

Authorization needs identity.

Therefore:

```text
Authentication
       ↓
Authorization
```

not the reverse.

The authorization layer cannot determine permissions if the application has not established who the requester is.

---

# 78. The Full Request Lifecycle

A production API can conceptually look like:

```text
Client
   |
   v
Load Balancer
   |
   v
Security / Edge Layer
   |
   v
Logging
   |
   v
Rate Limiting
   |
   v
Authentication
   |
   v
Authorization
   |
   v
Controller
   |
   v
Service
   |
   v
Repository
   |
   v
Database
```

Each layer has a responsibility.

---

# 79. User Enumeration

Authentication can leak information accidentally.

Suppose the login endpoint responds:

```text
User does not exist
```

for an unknown account.

But responds:

```text
Incorrect password
```

for an existing account.

An attacker can now determine which accounts exist.

For example:

```text
email1 → User does not exist
email2 → Incorrect password
```

The attacker learns:

```text
email2 exists
```

This is called:

> User Enumeration

A safer design often uses generic authentication failure messages such as:

```text
Authentication failed
```

instead of exposing which part of the credentials was incorrect.

---

# 80. Timing Attacks

Information can also leak through response timing.

For example:

```text
Unknown user
    ↓
return immediately
```

while:

```text
Known user
    ↓
perform password verification
    ↓
return
```

The second operation may take noticeably longer.

An attacker can repeatedly measure response times.

This can reveal whether an account exists.

Authentication systems can reduce such timing differences by doing comparable work in failure cases.

---

# 81. Constant-Time Comparison

Some secret comparisons should use constant-time comparison routines.

Instead of:

```text
regular string comparison
```

security-sensitive applications can use cryptographic comparison functions designed to avoid revealing information through timing.

Examples in programming environments include:

```text
crypto.subtle
hmac.compare_digest
```

The exact API depends on the language.

---

# 82. The Complete Evolution of Authentication

Now we can finally see the entire story.

```text
HTTP is stateless
        ↓
Need to associate requests with a user
        ↓
Password on every request
        ↓
Repeated credential transmission is undesirable
        ↓
Sessions
        ↓
Server stores identity state
        ↓
Applications become distributed
        ↓
Session state must be shared
        ↓
Redis / Shared Session Store
        ↓
Question:
"Can identity travel with the request?"
        ↓
Client cannot be trusted
        ↓
Cryptographic Signatures
        ↓
Signed Tokens
        ↓
JWT
        ↓
JWT revocation becomes difficult
        ↓
Short-lived Access Tokens
        +
Refresh Tokens
        ↓
Applications need delegated access
        ↓
OAuth
        ↓
OAuth needs standardized identity
        ↓
OIDC
```

---

# 83. The Evolution of Authorization

Authorization followed a similar path.

```text
Everyone can do everything
        ↓
Shared administrator secret
        ↓
Role-Based Access Control
        ↓
Role Hierarchy
        ↓
Role Explosion
        ↓
Attribute-Based Access Control
        ↓
Policy-Based Authorization
```

---

# 84. Sessions vs JWT

The easiest way to remember the difference:

## Session

The server remembers you.

```text
Client
   |
   | session ID
   v
Server
   |
   | lookup
   v
Session Store
```

---

## JWT

The token carries signed claims.

```text
Client
   |
   | JWT
   v
Server
   |
   | verify
   v
Identity
```

---

# 85. Sessions vs JWT — Trade-offs

| Feature | Session | JWT |
|---|---|---|
| Server-side state | Yes | Not required for token verification |
| Revocation | Easy | More difficult |
| Logout | Simple | Requires additional design |
| Distributed verification | Requires shared state | Can be independently verified |
| Central control | Strong | Lower |
| Horizontal scaling | Needs shared session infrastructure | Often simpler |
| Token contents | Server-side | Carried by token |
| Token theft | Can revoke centrally | Valid token may remain useful |
| Stateless | No | Yes, when no server-side token state is used |

The important lesson is:

> **JWT is not automatically better than sessions.**

They represent different architectural trade-offs.

---

# 86. Sessions Are Still Useful

A common misconception is:

```text
Sessions = old
JWT = modern
```

That is not the right way to think.

Sessions are extremely useful when you want:

```text
Centralized control
Easy revocation
Immediate logout
Server-side state
```

JWTs are useful when you want:

```text
Portable signed claims
Distributed verification
Reduced dependence on centralized session lookups
```

The correct question is:

> **Which architecture fits the problem?**

---

# 87. The Difference Between Cookie and Token

Remember:

```text
Cookie ≠ JWT
```

A cookie is a browser mechanism for storing and sending data.

A JWT is a token representation.

Possible combinations include:

```text
Session ID in Cookie
```

or:

```text
JWT in Cookie
```

or:

```text
JWT in Authorization Header
```

These are architecture choices.

---

# 88. The Difference Between Authentication and Authorization

The entire chapter can be reduced to two questions.

### Authentication

```text
Who are you?
```

Possible technologies:

```text
Session
JWT
OIDC
API Key
```

---

### Authorization

```text
What are you allowed to do?
```

Possible models:

```text
RBAC
ABAC
Policy-based access control
```

Therefore:

```text
Authentication
       ↓
Identity
       ↓
Authorization
       ↓
Permission
```

---

# 89. A Real Example

Imagine a project-management application.

A user logs in:

```text
POST /login
```

The server verifies the password.

Then the authentication server creates:

```json
{
    "sub": "user_42",
    "role": "project_lead",
    "exp": 1757003600
}
```

The token is signed.

The client later requests:

```http
POST /projects/100/members

Authorization: Bearer <JWT>
```

The server performs:

```text
1. Extract JWT
2. Verify signature
3. Validate expiration
4. Read subject
5. Identify user42
6. Read role
7. Check authorization
```

Suppose:

```text
role = project_lead
```

and project leads are allowed to add members.

The request is allowed.

If:

```text
role = viewer
```

the user is authenticated but not authorized.

The response becomes:

```http
403 Forbidden
```

---

# 90. One Complete Backend Security Pipeline

```text
                         REQUEST
                            |
                            v
                    ┌──────────────┐
                    │Authentication│
                    └──────┬───────┘
                           |
                           v
                      WHO ARE YOU?
                           |
                           v
                        Identity
                           |
                           v
                    ┌──────────────┐
                    │Authorization │
                    └──────┬───────┘
                           |
                           v
                    WHAT CAN YOU DO?
                           |
                           v
                       Permission
                           |
                           v
                      Controller
                           |
                           v
                       Service
                           |
                           v
                       Database
```

---

# 91. One Final Historical Diagram

The entire history can be remembered as:

```text
HTTP
 |
 | Stateless
 v
Password on Every Request
 |
 | Repeated credential transmission
 v
Sessions
 |
 | Server remembers identity
 v
Stateful Authentication
 |
 | Application grows
 v
Distributed Servers
 |
 | Shared session state needed
 v
Redis / Shared Session Store
 |
 | Centralized authentication state
 v
Question:
"Can identity travel with the request?"
 |
 | Client cannot be trusted
 v
Cryptographic Signatures
 |
 v
Signed Tokens
 |
 v
JWT
 |
 | Token theft / revocation problem
 v
Short-lived Access Tokens
 +
Refresh Tokens
 |
 v
OAuth
 |
 | Delegated access
 v
OIDC
 |
 | Identity layer
 v
Modern Authentication


Authorization:

Everyone
   ↓
Shared Admin Secret
   ↓
RBAC
   ↓
Role Hierarchy
   ↓
Role Explosion
   ↓
ABAC
   ↓
Policy-Based Authorization
```

---

# 92. The Most Important Mental Model

Do not memorize JWT as:

```text
Header + Payload + Signature
```

That only tells you its structure.

Understand the reason it exists:

```text
HTTP does not remember you
        ↓
We need persistent identity
        ↓
Passwords were repeatedly transmitted
        ↓
Sessions let the server remember
        ↓
Distributed systems required shared session state
        ↓
Engineers wanted identity to travel with requests
        ↓
But clients cannot be trusted
        ↓
Cryptographic signatures provide integrity
        ↓
Signed claims become useful
        ↓
JWT becomes one standardized representation
```

That is the real story.

---

# 93. The Most Important Security Principles

Keep these principles in your mind:

```text
Never trust the client
Never store plaintext passwords
Use password-specific hashing
Use unique salts
Protect secrets
Use HTTPS
Use secure cookie settings
Keep access tokens short-lived
Protect refresh tokens
Validate JWT signatures
Pin allowed algorithms
Validate expiration
Do not put secrets inside JWT payloads
Use least privilege
Separate authentication from authorization
Regenerate session IDs after login
Protect cookie-based authentication against CSRF
Do not leak account existence unnecessarily
Use secure secret comparison where appropriate
```

---

# 94. Final Cheat Sheet

```text
Authentication
= Who are you?

Authorization
= What can you do?

HTTP Statelessness
= Each request is independent at the protocol level.

Session
= Server-side authentication state.

Cookie
= Browser mechanism for storing/sending data.

JWT
= Signed token containing claims.

Header
= Token metadata.

Payload
= Claims.

Signature
= Cryptographic integrity/authenticity proof.

Base64URL
= Encoding, not encryption.

sub
= Subject / identity.

iat
= Issued-at timestamp.

exp
= Expiration timestamp.

Access Token
= Short-lived credential used for API access.

Refresh Token
= Credential used to obtain new access tokens.

HS256
= Symmetric signing.

RS256
= Asymmetric signing.

API Key
= Application or machine credential.

OAuth
= Delegated authorization.

OIDC
= Identity layer built on OAuth 2.0.

RBAC
= Role-Based Access Control.

ABAC
= Attribute-Based Access Control.

401
= Authentication failed / credentials missing or invalid.

403
= Identity known, but action forbidden.

CSRF
= Cross-Site Request Forgery.

Session Fixation
= Attacker attempts to make a victim authenticate using a known session identifier.
```

---

# 95. Final Lesson

Authentication did not start with JWT.

Authorization did not start with RBAC.

OAuth did not appear simply because the industry wanted another protocol.

Every one of these technologies appeared because an earlier solution became insufficient for a new problem.

The evolution is the important part:

```text
Problem
   ↓
Solution
   ↓
New Problem
   ↓
New Solution
```

That is how backend engineering evolves.

The real lesson is therefore not:

> "How do I create a JWT?"

The deeper question is:

> **"What problem is this authentication mechanism solving, and what trade-off does it introduce?"**

Once that becomes clear, the syntax, libraries, frameworks and middleware become implementation details.

---

# 96. Final Mental Model

```text
                     USER
                       |
                       v
                Authentication
                       |
                       v
                  "WHO ARE YOU?"
                       |
                       v
                    Identity
                       |
                       v
                Authorization
                       |
                       v
                "WHAT CAN YOU DO?"
                       |
                       v
                   Permission
                       |
                       v
                Business Logic
                       |
                       v
                    Database
```

And underneath that entire system is one simple idea:

```text
Authentication
      ↓
Identity
      ↓
Authorization
      ↓
Permission
```

That is the foundation of backend security.
