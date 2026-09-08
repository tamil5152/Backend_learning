# Authentication and Authorization for Backend Engineers

> Source: https://youtu.be/A95rliroC8Q

## 1. Authentication and Authorization

Every application eventually needs to answer two fundamental questions:

- Who are you?
- What are you allowed to do?

These lead to two different concepts:

- **Authentication** answers: "Who are you?"
- **Authorization** answers: "What can you do?"

Authentication establishes the identity of a user or system.

Authorization takes that established identity and determines which actions, resources, or operations that identity is permitted to access.

A typical flow looks like this:

```text
Username + Password
        ↓
    Authentication
        ↓
     "This is Alice"
        ↓
     Authorization
        ↓
"What is Alice allowed to do?"
```

A normal user might be allowed to read their own profile, while an administrator might be allowed to read, modify, or delete other users.

## 2. How Authentication Evolved

Authentication did not begin with passwords, JWTs, or OAuth.

The underlying problem is much older:

> How can I prove that I am who I claim to be?

**Recognition**

In small communities, identity could be established simply through recognition. People knew each other.

As societies grew larger, people started interacting with strangers. Recognition was no longer enough, so systems for explicitly proving identity became necessary.

**Physical Proof**

One early solution was to use physical objects or marks as proof of identity. Wax seals are a simple example.

The underlying principle was:

```text
Identity
   ↓
Something difficult to reproduce
   ↓
Proof of identity
```

Modern authentication systems are still based on this fundamental idea.

## 3. Something You Know

Authentication systems eventually began using secrets. A secret could be shared between two parties.

```text
Person A: "What is the secret phrase?"
Person B: "The secret phrase is X."
```

If the person knew the correct secret, they were considered authentic. This eventually evolved into passwords.

This is the authentication factor:

> Something you know.

Examples include:

- Passwords
- PINs
- Passphrases
- Security questions

## 4. The Mainframe Era

When computers became capable of supporting multiple users, authentication became a much more important problem.

One computer could have many different users:

```text
User A
User B
User C
```

The system needed a way to distinguish between them. Each user needed:

```text
Identity + Proof of Identity
```

This led to widespread password-based authentication.

But storing passwords created another problem. Suppose a database contains:

```text
username | password
---------|---------
alice    | hello123
bob      | password
```

If an attacker compromises the database, every password is exposed.

This leads to the question:

> How can we verify a password without storing the actual password?

## 5. Password Hashing

The solution is password hashing.

Instead of storing:

```text
password = "hello123"
```

the system stores a hash:

```text
hash("hello123") = <hash>
```

When the user logs in:

```text
User enters password
        ↓
Hash the password
        ↓
Compare with stored hash
        ↓
Match?
   /        \
 Yes         No
 ↓            ↓
Authenticate  Reject
```

The original password does not need to be stored.

Modern password systems also use techniques such as salting and password-specific hashing algorithms to make attacks harder.

The important principle is:

> Never store plaintext passwords.

## 6. Cryptography Changes Authentication

As computer networks became larger, authentication required stronger mechanisms.

Cryptography introduced powerful primitives for proving identity and authenticity.

Important developments include:

- Symmetric cryptography
- Asymmetric cryptography
- Public/private key pairs
- Diffie-Hellman key exchange
- Digital signatures
- Public Key Infrastructure

These became building blocks for modern authentication systems.

## 7. Kerberos and Ticket-Based Authentication

Another important development was Kerberos.

Instead of repeatedly sending credentials to every service, a trusted authentication system could issue tickets.

Conceptually:

```text
User
 ↓
Authentication Server
 ↓
Ticket
 ↓
Service
```

The ticket acts as proof that the user has already authenticated.

This introduces an important pattern:

> Authenticate once, receive a credential, and use that credential for subsequent requests.

This idea appears repeatedly in modern authentication systems.

## 8. Authentication on the Web

Now we reach the problem backend engineers deal with constantly.

HTTP is stateless. That means the server does not automatically remember previous requests.

Imagine:

```text
Request 1:
"Here are my credentials."

Response:
"Okay, I know who you are."

Request 2:
"Give me my profile."
```

HTTP itself does not automatically remember that Request 2 came from the same person who made Request 1. Each request is independent.

So we need a mechanism that allows the server to recognize the client across multiple requests. This is where:

- Sessions
- Cookies
- Tokens

become important.

## 9. Sessions

A session allows the server to remember a user's authentication state.

Suppose Alice logs in:

```text
Alice
  ↓
Username + Password
  ↓
Server validates credentials
  ↓
Create session
  ↓
Generate session ID
```

The server might store:

```text
session_id → user information
```

For example:

```text
abc123 → Alice
```

The client receives:

```text
abc123
```

Usually this session ID is stored inside a cookie.

On the next request:

```text
Client
  ↓
Cookie: session_id=abc123
  ↓
Server
  ↓
Look up abc123
  ↓
Alice
```

The server now knows who made the request.

## 10. Stateful Authentication

Sessions are called stateful authentication because the server maintains authentication state.

That state might be stored in:

- Application memory
- A database
- Redis
- Another session store

Conceptually:

```text
Client
   |
   | session ID
   ↓
Server
   |
   | lookup
   ↓
Session Store
   |
   ↓
User Identity
```

**Why Stateful Authentication Is Useful**

The biggest advantage is control.

Because the server owns the session state, it can immediately invalidate it.

```text
Session exists
     ↓
User authenticated

Session deleted
     ↓
Session no longer valid
```

This makes stateful authentication useful when immediate session control is important.

## 11. The Scaling Problem

Now imagine the application becomes large.

Instead of one backend server, we have:

```text
              ┌── Server A
Client ──────┼── Server B
              ├── Server C
              └── Server D
```

Where does the session live?

If the session is stored only in Server A's memory, Server B cannot find it.

One option is sticky sessions, where a user is repeatedly sent to the same server. But that introduces operational complexity.

A common approach is to move session state into a shared store:

```text
              ┌── Server A ──┐
Client ──────┼── Server B ──┼── Redis
              ├── Server C ──┤
              └── Server D ──┘
```

Now all backend servers can access the same session state.

But this introduces another dependency and another source of operational overhead.

This motivates the idea of stateless authentication.

## 12. Stateless Authentication

Instead of storing authentication state on the server, we can give the client a token that carries information about the authenticated identity.

The server can then verify that token without performing a session lookup.

This is the basic idea behind JWT-based authentication.

Conceptually:

```text
Client
  ↓
Token
  ↓
Server
  ↓
Verify Token
  ↓
Identify User
```

## 13. JSON Web Tokens

A JWT is commonly represented as:

```text
HEADER.PAYLOAD.SIGNATURE
```

It has three major components.

**Header**

The header describes information about the token, including the signing algorithm.

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload**

The payload contains claims.

Example:

```json
{
  "sub": "123",
  "role": "admin",
  "iat": 1690000000
}
```

Claims can include information such as:

- User ID
- Role
- Issued-at time
- Expiration time
- Other application claims

An important rule is:

> A JWT payload should not be treated as a secret.

JWT encoding is not the same as encryption.

## 14. The JWT Signature

The signature allows the server to determine whether the token has been modified.

Conceptually:

```text
Header
   +
Payload
   ↓
Signing Algorithm + Key
   ↓
Signature
```

When the server receives the JWT:

```text
JWT
 ↓
Read Header + Payload
 ↓
Verify Signature
 ↓
Valid?
```

Suppose the token originally contains:

```text
role=user
```

An attacker changes it to:

```text
role=admin
```

The signature should no longer match. Therefore the server can detect the modification.

The important idea is:

> The signature provides integrity, not secrecy.

## 15. Why JWTs Scale Well

Consider a distributed backend:

```text
              ┌── Server A
              ├── Server B
Client ──────┼── Server C
              └── Server D
```

Each server can independently verify a JWT.

There does not necessarily need to be a centralized session lookup for every request.

This makes JWTs attractive for:

- Distributed systems
- Microservices
- APIs
- Mobile applications
- Horizontally scaled systems

## 16. The Major JWT Trade-Off

JWTs introduce an important trade-off.

Suppose a user receives a JWT that expires in one hour.

The server may not have a central session that can simply be deleted.

Therefore:

```text
JWT issued
   ↓
JWT remains valid
   ↓
Expiration time reached
   ↓
JWT becomes invalid
```

If the token is stolen, it may remain usable until it expires.

This creates a revocation problem.

Stateless authentication provides scalability and portability, but controlling individual tokens becomes harder.

## 17. Stateful vs Stateless

The fundamental difference is:

**Stateful**

```text
Client
  ↓
Session ID
  ↓
Server
  ↓
Session Store
  ↓
User
```

**Stateless**

```text
Client
  ↓
JWT
  ↓
Server
  ↓
Verify JWT
  ↓
User
```

**Stateful Advantages**
- Easy revocation
- Centralized control
- Easy session management
- Server can inspect active sessions

**Stateful Disadvantages**
- Requires shared session storage when scaled
- Adds infrastructure
- Requires session-state management

**Stateless Advantages**
- Easy horizontal scaling
- No session lookup is inherently required
- Tokens are portable
- Useful for distributed systems

**Stateless Disadvantages**
- Revocation is harder
- Token theft can be dangerous
- Key management becomes critical
- Token lifetime must be designed carefully

## 18. Hybrid Authentication

There is no requirement to choose a completely stateful or completely stateless architecture.

A hybrid approach can combine both.

For example:

```text
JWT
 ↓
Verify Signature
 ↓
Check Revocation State
 ↓
Allow / Reject
```

A blacklist or allowlist can be maintained in something like Redis.

This provides some benefits of stateless tokens while restoring some server-side control.

The trade-off is that the system is no longer completely stateless.

## 19. Cookies

A cookie is a browser mechanism for storing and sending data with HTTP requests.

Cookies are often used to transport authentication credentials.

For example:

```text
Set-Cookie: session_id=abc123
```

The browser can then send:

```text
Cookie: session_id=abc123
```

on later requests.

Cookies themselves are not authentication.

They are a storage and transport mechanism that can carry authentication information.

A cookie can contain:

- Session IDs
- JWTs
- Other application state

## 20. HttpOnly Cookies

Authentication cookies are commonly marked:

```text
HttpOnly
```

This prevents ordinary JavaScript running in the browser from directly reading the cookie.

Other important cookie attributes include:

- Secure
- HttpOnly
- SameSite

These should be configured deliberately.

## 21. API Key Authentication

An API key is a secret credential primarily used for programmatic access.

Conceptually:

```text
Client
  ↓
API Key
  ↓
Backend API
```

API keys are commonly useful for:

- Server-to-server communication
- Developer APIs
- Programmatic integrations
- Identifying applications

They are generally not the primary mechanism for interactive user login.

## 22. OAuth

OAuth solves a different problem.

Imagine an application wants access to a resource belonging to a user.

The application should not need to ask the user for another service's password.

Instead:

```text
User
 ↓
Application
 ↓
Authorization Server
 ↓
User grants permission
 ↓
Access Token
 ↓
Application
```

The application can then use the access token to access the resources that were permitted.

The key idea is delegated authorization.

OAuth allows an application to access resources on behalf of a user without requiring the application to obtain the user's password.

## 23. OAuth 1.0

OAuth 1.0 introduced delegated authorization but relied heavily on cryptographic request signing.

This made implementation relatively complicated.

OAuth 2.0 simplified many aspects of the model and commonly uses bearer access tokens.

## 24. OAuth 2.0 Flows

Different application types need different OAuth flows.

**Authorization Code**

Commonly used when a backend can securely participate in the authorization exchange.

```text
User
 ↓
Client
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Backend
 ↓
Access Token
```

**Implicit Flow**

Historically used for browser-based applications.

It is now generally discouraged in modern systems because safer approaches exist.

**Client Credentials**

Used for machine-to-machine communication.

```text
Service A
   ↓
Client Credentials
   ↓
Authorization Server
   ↓
Access Token
   ↓
Service B
```

There is no human user involved in this flow.

**Device Authorization**

Useful for devices where entering credentials or handling browser redirects is difficult.

Examples include:

- Smart TVs
- Consoles
- Limited-input devices

## 25. OAuth Is Not Authentication

This distinction is extremely important.

OAuth primarily answers:

> What is this application allowed to access?

It does not fundamentally answer:

> Who is this user?

That is where OpenID Connect comes in.

## 26. OpenID Connect

OpenID Connect adds an authentication and identity layer on top of OAuth 2.0.

Conceptually:

```text
OAuth
  ↓
Authorization

OpenID Connect
  ↓
Authentication + Identity
```

OpenID Connect introduces an ID token representing the authenticated user.

This is why modern "Sign in with ..." systems can use OAuth/OIDC-based identity providers.

## 27. Authentication vs Authorization

After authentication, the system knows:

```text
Who is this?
```

But that is not enough. The system must also determine:

```text
What can this person do?
```

That is authorization.

For example:

```text
Alice
 ↓
Authenticated
 ↓
Role = User
 ↓
Can read profile
Can edit own profile
Cannot delete other users
```

While:

```text
Bob
 ↓
Authenticated
 ↓
Role = Admin
 ↓
Can read users
Can modify users
Can delete users
```

The key relationship is:

```text
Authentication → Identity
Authorization  → Permissions
```

## 28. Role-Based Access Control

One common authorization model is RBAC: Role-Based Access Control.

Instead of assigning permissions independently to every user, users are assigned roles.

For example:

```text
User
 ├── read
 └── write-own-data

Moderator
 ├── read
 ├── write
 └── moderate

Admin
 ├── read
 ├── write
 ├── delete
 └── manage-users
```

The relationship becomes:

```text
User → Role → Permissions
```

This makes authorization easier to manage.

## 29. Authorization During a Request

A backend request can be thought of as a pipeline:

```text
HTTP Request
     ↓
Authentication
     ↓
Identify User
     ↓
Determine Role / Attributes
     ↓
Authorization
     ↓
Check Permission
     ↓
Allowed?
   /      \
 Yes       No
 ↓          ↓
Continue    403 Forbidden
```

Authentication should establish identity before authorization evaluates what that identity is permitted to do.

## 30. Authentication Does Not Mean Permission

A user can successfully authenticate and still be denied access.

The key distinction is:

```text
Authenticated ≠ Authorized
```

For example:

```text
Authentication
      ↓
"This is Alice."
      ↓
Authorization
      ↓
"Alice is not allowed to delete this resource."
      ↓
403 Forbidden
```

## 31. Password Security

Passwords should never be stored in plaintext.

Bad:

```text
password = "mypassword123"
```

Better:

```text
password
   ↓
Password Hashing
   ↓
Stored Hash
```

Password storage should use appropriate password hashing algorithms, salting, and secure credential-handling practices.

## 32. Avoid Information Leakage

Authentication systems should avoid revealing unnecessary information.

For example, an application should be careful about responses that allow attackers to determine whether an account exists.

Rather than revealing detailed authentication state, systems can use generic responses such as:

```text
Authentication failed
```

The goal is to avoid giving attackers useful reconnaissance information.

## 33. Timing Attacks

Authentication can also leak information through timing differences.

For example:

```text
Invalid username
        ↓
Fast response

Valid username + wrong password
        ↓
Slower response
```

An attacker may use timing differences to discover whether an account exists.

Security-sensitive comparisons should therefore be implemented carefully, using constant-time comparison where appropriate and avoiding unnecessary timing differences.

## 34. Token Security

Authentication tokens are credentials.

If an attacker steals a valid token, they may be able to impersonate the user.

Therefore:

- Keep token lifetimes reasonable.
- Protect signing keys.
- Rotate secrets or keys appropriately.
- Avoid putting sensitive information into JWT payloads.
- Use refresh mechanisms when appropriate.
- Have a revocation strategy when required.

## 35. JWT Revocation

Because JWTs are stateless, revocation can be difficult.

**Short-Lived Tokens**

```text
Access Token
 ↓
Short Expiration
 ↓
Smaller Attack Window
```

Short-lived access tokens reduce the amount of time a stolen token remains useful.

**Revocation List**

```text
JWT
 ↓
Check Revocation List
 ↓
Revoked?
 /     \
Yes     No
 ↓       ↓
Reject   Continue
```

**Key Rotation**

Changing signing keys can invalidate tokens depending on the architecture.

However, careless key rotation can also invalidate many legitimate tokens at once.

The correct choice depends on the system.

## 36. Authentication Providers

Building authentication correctly is difficult.

There are many concerns:

- Password hashing
- Credential storage
- Session management
- Token generation
- Token expiration
- Key management
- Key rotation
- MFA
- Account recovery
- Security monitoring
- Revocation
- Attack protection

For production systems, using a mature authentication provider can reduce security and operational risks.

Examples include specialized identity platforms such as:

- Auth0
- Clerk
- Other identity providers

The important lesson is:

> Understand authentication deeply, but do not automatically reinvent security-critical infrastructure in production.

## 37. Learn by Building

There is an important difference between learning and production engineering.

For learning, building authentication yourself can be extremely useful.

You can implement:

```text
Authentication
      ↓
Password Hashing
      ↓
Sessions
      ↓
Cookies
      ↓
JWT
      ↓
OAuth
      ↓
Authorization
```

This teaches you how the system works internally.

For production, mature libraries and identity providers can provide a safer foundation.

## 38. Emerging Authentication Technologies

Authentication continues to evolve.

**Passwordless Authentication**

Instead of passwords, users can authenticate using cryptographic credentials or hardware-backed credentials.

WebAuthn is an important technology in this area.

**Zero Trust**

Traditional systems often assumed that something inside the network could be trusted.

Zero Trust changes that assumption.

The core idea is:

> Never automatically trust a request simply because of where it came from.

Each request should be evaluated based on identity, permissions, context, and policy.

**Decentralized Identity**

Another direction is decentralized identity.

The goal is to give users greater control over their identity credentials, potentially using decentralized identifiers and verifiable credentials.

**Behavioral Biometrics**

Authentication can incorporate behavioral signals such as:

- Typing patterns
- Mouse movement
- Interaction patterns
- Device behavior

The goal can be continuous evaluation instead of checking identity only once.

**Post-Quantum Cryptography**

Future quantum computers could threaten some existing cryptographic algorithms.

This creates a need for cryptographic algorithms designed to resist quantum attacks.

Post-quantum cryptography is therefore another important area for the future.

## 39. The Complete Mental Model

The whole system can be understood as a pipeline:

```text
                     REQUEST
                       |
                       v
               ┌─────────────────┐
               │ Authentication  │
               └────────┬────────┘
                       |
                       v
                 Who is the user?
                       |
                       v
               ┌─────────────────┐
               │ Authorization   │
               └────────┬────────┘
                       |
                       v
              What can they do?
                       |
                       v
                 Permission Check
                       |
               ┌────────┴────────┐
               |                 |
            Allowed            Denied
               |                 |
               v                 v
         Execute Request    403 Forbidden
```

Authentication establishes identity.

Authorization establishes permission.

Sessions, cookies, JWTs, API keys, OAuth, OIDC, and RBAC are mechanisms used to implement these ideas.

## 40. The Four Approaches at a Glance

| Approach   | Main Idea                         | Typical Use                    |
|------------|------------------------------------|----------------------------------|
| Stateful   | Server stores session state        | Web applications                 |
| Stateless  | Client carries a signed token      | APIs and distributed systems     |
| API Key    | Secret identifies a client/app     | Programmatic APIs                |
| OAuth/OIDC | Delegated access and identity      | Third-party integrations         |

## 41. The Most Important Mental Models

- **Authentication** — WHO ARE YOU?
- **Authorization** — WHAT CAN YOU DO?
- **Session** — The server remembers your authentication state.
- **JWT** — The client carries a signed token that the server can verify.
- **Cookie** — A browser mechanism for storing and sending data.
- **OAuth** — Allow another application to access resources on your behalf.
- **OpenID Connect** — Authentication and identity built on top of OAuth 2.0.
- **RBAC** — Role → Permissions

## 42. Final Takeaway

Authentication and authorization are not simply login forms.

They are systems for answering two fundamental questions:

```text
Who are you?
        ↓
Authentication

What are you allowed to do?
        ↓
Authorization
```

The technologies surrounding authentication evolved because the underlying problems evolved.

The progression can be understood as:

```text
Recognition
   ↓
Physical Proof
   ↓
Shared Secrets
   ↓
Passwords
   ↓
Cryptography
   ↓
Sessions
   ↓
Tokens
   ↓
OAuth / OIDC
   ↓
Passwordless and Modern Identity Systems
```

For a backend engineer, the important thing is not memorizing a particular library or copying an authentication tutorial.

The important thing is understanding the underlying trade-offs:

```text
Stateful
    ↕
Stateless

Centralized Control
    ↕
Scalability

Easy Revocation
    ↕
Portable Tokens
```

Once these trade-offs are understood, authentication libraries and frameworks become implementation details rather than mysterious pieces of code.

## Source

https://youtu.be/A95rliroC8Q
