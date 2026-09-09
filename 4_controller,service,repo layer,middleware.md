# Backend Architecture, From Zero

Let's **not memorize "Controller → Service → Repository → Middleware."**

Instead, let's build the whole thing from **zero**, asking the natural questions that lead us to each concept.

The chapter is fundamentally answering one question:

> **"A request enters my backend. What should happen to it, and how should I organize the code that handles it?"**

---

## 1. First, forget Controllers, Services and Repositories

Imagine we are building a simple **book application**.

A user clicks:

> "Show me all books."

The browser sends:

```http
GET /books
```

Our server receives it.

Now ask:

### What does the server actually need to do?

Something roughly like:

```text
Request arrives
      ↓
Figure out what the request wants
      ↓
Check the input
      ↓
Do the actual business work
      ↓
Get data from database
      ↓
Send result back
```

That sounds simple.

But now let's ask:

> **Who should do each of these jobs?**

That's where the architecture comes from.

---

## 2. What happens if we put EVERYTHING in one function?

Suppose we don't know anything about architecture yet.

We could simply write:

```go
func GetBooks(w http.ResponseWriter, r *http.Request) {

    // read request

    // validate input

    // check authentication

    // business logic

    // write SQL query

    // execute database query

    // format response

}
```

And honestly...

### This works.

There is nothing technically stopping us from doing this.

The source explicitly makes this point: separating the layers is **not a hard requirement**; it is a design choice.

So why bother separating things?

---

## 3. Let's imagine the application becomes huge

Initially we have:

```text
GetBooks()
CreateBook()
DeleteBook()
UpdateBook()
```

Maybe putting everything inside each handler is manageable.

But then our application grows.

Now we have:

```text
100+ endpoints
authentication
authorization
emails
payments
notifications
database operations
external APIs
logging
rate limiting
etc.
```

And every handler starts looking like this:

```text
HTTP stuff
+
validation
+
business logic
+
database logic
+
email logic
+
authentication
+
...
```

Now something becomes obvious.

### The handler is doing too many unrelated jobs.

And that creates problems:

* difficult to understand
* difficult to modify
* difficult to test
* duplicated logic
* changes in one area can break another area

So we ask:

> **What's the root problem?**

The problem isn't that our code is long.

The deeper problem is:

> **Different responsibilities are mixed together.**

That's the root cause.

---

## 4. So how can we solve that?

Well...

If different things have different responsibilities...

### Why not separate them?

For example:

```text
HTTP-related work
        ↓
Business-related work
        ↓
Database-related work
```

Now each piece has one main responsibility.

And this gives us the three layers:

```text
Handler
   ↓
Service
   ↓
Repository
```

This is **separation of concerns**.

But we shouldn't memorize those names yet.

Let's discover what each one actually means.

---

## 5. First question: who talks HTTP?

Our request is:

```http
GET /books?sort=date
```

Someone needs to deal with things like:

```text
GET
/books
?sort=date
headers
body
status codes
JSON
```

Who should care about these?

### The HTTP-facing part of our application.

That's the **Handler / Controller**.

---

## 6. What should the Handler actually do?

Think of the handler as a **receptionist**.

Imagine you walk into a company.

The receptionist doesn't perform the actual engineering work.

They:

1. receive you
2. understand what you need
3. collect the necessary information
4. check whether the information is valid
5. send you to the correct department
6. give you the final answer

That's basically the handler.

The source describes its main job as controlling the flow of data between the client and server.

So:

```text
HTTP Request
     ↓
   Handler
```

The handler receives the request and extracts what it needs.

For example:

```http
GET /books?sort=date
```

The handler extracts:

```text
sort = "date"
```

Then it validates it.

For example:

```text
sort must be:
"name"
or
"date"
```

If it's invalid:

```http
400 Bad Request
```

If it's valid...

### Does the handler need to know how the business actually works?

Not really.

It can simply say:

> "Hey service, give me the books sorted by date."

And that's where the next question appears.

---

## 7. What should the Service do?

Let's say our application has a rule:

> Only books that are published should be shown.

Or perhaps:

> When creating a book, the owner must be the authenticated user.

Or:

> After creating a book, send an email to the owner.

These aren't really HTTP concerns.

They're **business rules**.

So we need somewhere to put them.

That becomes the:

### Service Layer

The service is basically the **brain of the business operation**.

The source calls this the actual processing/business logic and says the service should know nothing about HTTP.

So instead of:

```go
func GetBooks(w http.ResponseWriter, r *http.Request)
```

the service can look conceptually like:

```go
func ListBooks(ctx, sort) ([]Book, error)
```

Notice something important.

There is no:

```text
HTTP request
HTTP response
HTTP status code
```

Why?

Because the service shouldn't care whether the request came from:

```text
HTTP API
CLI
background job
another service
```

Its job is simply:

> **"Given this input, perform the business operation."**

---

## 8. But wait...

The service needs books.

Where do books actually live?

Usually:

```text
Database
```

So the service could directly write SQL:

```go
func ListBooks(...) {

    db.Query("SELECT ...")

    // business logic
}
```

Technically...

### That also works.

But now we've created another problem.

The service is doing:

```text
business logic
+
database logic
```

Again, responsibilities are mixed.

So we ask:

> **Can we separate database work too?**

Yes.

And that gives us the:

### Repository Layer

---

## 9. What is the Repository?

Think of the repository as the **database specialist**.

The service says:

> "I need all books sorted by date."

The repository knows:

> "Okay, I know how to talk to PostgreSQL."

So:

```text
Service
   ↓
"Give me books sorted by date"
   ↓
Repository
   ↓
SQL
   ↓
Database
```

The repository's concern is specifically talking to the database: constructing queries, executing them, and returning the result.

---

## 10. Now the entire thing makes sense

We didn't memorize:

```text
Controller
Service
Repository
```

We **arrived at them**.

Because we had a problem:

```text
Everything mixed together
```

So we separated responsibilities:

```text
             HTTP responsibility
                    ↓
               HANDLER
                    ↓
          Business responsibility
                    ↓
                SERVICE
                    ↓
          Database responsibility
                    ↓
              REPOSITORY
                    ↓
               DATABASE
```

That's the fundamental idea.

---

## 11. Let's follow one real request

Suppose the browser sends:

```http
GET /books?sort=date
```

What happens?

Let's walk through it like we're debugging the server.

### Step 1 — Request enters the server

The operating system sends the request to the port our server is listening on.

For example:

```text
localhost:3000
```

That's our **entry point**.

---

## 12. But which function should handle `/books`?

The server might have:

```text
GET /books
POST /books
GET /books/:id
DELETE /books/:id
```

So we need something that asks:

> "Which function should handle this particular request?"

That's **routing**.

Conceptually:

```text
GET /books
      ↓
   Router
      ↓
ListBooksHandler
```

Routing maps the request's method and path to a particular handler.

---

## 13. But wait — should EVERY request immediately reach the handler?

Think about this.

Suppose someone sends:

```http
GET /books
Authorization: invalid-token
```

Do we want the request to reach the handler?

Probably not.

We should first ask:

> "Is this person authenticated?"

And maybe:

> "Is this request allowed?"

And maybe:

> "Are they sending too many requests?"

And maybe:

> "Should we log this request?"

Notice something interesting.

These operations are **not specific to `/books`**.

They might be needed for:

```text
/users
/books
/orders
/payments
/products
```

So should we copy the same authentication code into every handler?

### That would be terrible.

We'd repeat:

```text
check authentication
check rate limit
log request
...
```

inside hundreds of handlers.

So what's the root problem?

> **We have common work that needs to happen across many requests.**

---

## 14. So how can we solve that?

Instead of putting common logic inside every handler...

### Put it in the request's path.

Like this:

```text
Request
   ↓
Logging
   ↓
Authentication
   ↓
Rate limiting
   ↓
Routing
   ↓
Handler
```

These are called:

### Middleware

The source defines middleware as functions that execute in the "gaps" between execution contexts in the request lifecycle.

Think of middleware like **security checkpoints at an airport**.

You don't write:

```text
Passenger → security → gate
```

inside every passenger's personal code.

The airport creates a common process:

```text
Entrance
   ↓
Security
   ↓
Passport check
   ↓
Gate
```

Same idea.

---

## 15. But how does middleware know where to send the request next?

This is where:

```go
next()
```

comes in.

Imagine:

```text
Middleware 1
     ↓
Middleware 2
     ↓
Middleware 3
     ↓
Handler
```

Middleware 1 finishes its job and says:

> "Okay, I'm done. Continue."

That's:

```go
next()
```

The source describes `next()` as the function that passes execution to the next context in the chain.

---

## 16. But middleware doesn't HAVE to call `next()`

This is actually powerful.

Imagine:

```text
Request
   ↓
Authentication
```

Authentication checks the token.

### Valid?

Then:

```text
next()
 ↓
Handler
```

### Invalid?

We don't want to continue.

So:

```text
Authentication
      ↓
   401 Unauthorized
      ↓
     STOP
```

That's called **short-circuiting**.

The source specifically describes these two possibilities: call `next()` to continue, or stop and send a response immediately.

---

## 17. Now another problem appears

Suppose authentication successfully figures out:

```text
user_id = 123
role = admin
```

The handler needs this information.

But how do we pass it?

We could do something ugly like:

```text
middleware
   ↓
next(userID, role)
   ↓
next(...)
   ↓
next(...)
   ↓
handler
```

Now every function needs to explicitly receive and pass those values.

That's annoying.

So ask:

> **Can all the functions participating in this one request share some request-specific information?**

Yes.

That leads us to:

### Request Context

---

## 18. What is Request Context?

Imagine every request gets a small backpack.

```text
Request
  🎒
```

During the request lifecycle, different parts of the server can put useful information into that backpack.

Authentication puts:

```text
userID = 123
role = admin
```

Tracing might put:

```text
requestID = abc-123
```

Timeout handling might carry:

```text
deadline = ...
```

Then the handler can access the information.

The key idea is:

> **The context is state/storage associated with one specific request.**

The source emphasizes that every request gets its own context, so state doesn't leak between requests.

---

## 19. Why is that useful for authentication?

Here's an important security problem.

Suppose the client sends:

```json
{
    "title": "My Book",
    "user_id": 999
}
```

Should the server trust:

```text
user_id = 999
```

?

### Absolutely not.

The client controls that JSON.

A malicious user could simply change it:

```json
{
    "title": "My Book",
    "user_id": 1
}
```

Now they are trying to pretend they're user 1.

So what's the correct approach?

Authentication middleware verifies the user's credentials.

Then:

```text
verified user
      ↓
userID = 123
      ↓
request context
      ↓
handler
```

The handler uses the **server-verified identity**, not the client's claimed identity.

This is a beautiful example of why the concepts connect.

---

## 20. Now let's put EVERYTHING together

A request might travel like this:

```text
                 HTTP REQUEST
                      │
                      ↓
                 Entry Point
                      │
                      ↓
                  Middleware
                      │
             ┌────────┴────────┐
             ↓                 ↓
          Logging        Authentication
                               │
                         userID / role
                               │
                               ↓
                         Request Context
                               │
                               ↓
                            Router
                               │
                               ↓
                           Handler
                               │
                    validate / transform
                               │
                               ↓
                           Service
                               │
                         business logic
                               │
                               ↓
                         Repository
                               │
                           SQL query
                               │
                               ↓
                           Database
```

Then the result travels back:

```text
Database
   ↓
Repository
   ↓
Service
   ↓
Handler
   ↓
HTTP Response
   ↓
Client
```

The source summarizes this full lifecycle in essentially this order.

---

## 21. The most important mental model

Don't memorize definitions like:

> "A repository is a data-access abstraction."

Instead, remember **why it exists**.

### We started with a problem:

```text
One giant handler
```

### Why was that bad?

Because:

```text
HTTP
+
business logic
+
database
+
common request logic
```

were all mixed together.

### So we separated responsibilities:

```text
                    WHY?
                     │
                     ↓
          Too many responsibilities
                     │
                     ↓
           Separate responsibilities
                     │
        ┌────────────┼─────────────┐
        ↓            ↓             ↓
      HTTP         Business      Database
        ↓            ↓             ↓
    Handler       Service      Repository
```

Then we noticed another problem:

```text
Authentication
Logging
Rate limiting
CORS
```

are common to many requests.

So:

```text
Common request logic
        ↓
   Middleware
```

Then we noticed middleware needs to pass information through the request.

So:

```text
Per-request shared state
        ↓
   Request Context
```

And finally:

```text
next()
```

exists because middleware needs a way to say:

> **"I'm finished. Let the request continue."**

---

## 22. One final analogy

Imagine a restaurant.

A customer places an order.

```text
Customer
   ↓
Receptionist
   ↓
Kitchen manager
   ↓
Cook
   ↓
Storage
```

Now map that to our backend:

| Restaurant            | Backend              |
| ---------------------- | --------------------- |
| Customer               | Client                |
| Receptionist           | Handler               |
| Kitchen manager        | Service               |
| Storage                | Repository/Database   |
| Security at entrance   | Middleware            |
| Order ticket           | Request Context       |
| "Next customer"        | `next()`              |

The receptionist shouldn't go into the storage room and start managing inventory.

The cook shouldn't deal with HTTP status codes.

The storage person shouldn't decide business rules.

**Each person has a responsibility.**

That's the entire philosophy behind this chapter.

> **The architecture isn't about creating three fancy folders called `controllers`, `services`, and `repositories`.**
>
> **It's about taking one complicated problem and separating it into responsibilities so each piece has a clear reason to exist.**
