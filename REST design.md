# REST API Design — From First Principles (Java Edition)
 
The file you shared is about **REST API design**, but the example code is written in **Go**. This document teaches the same ideas in **Java**, from first principles, focusing heavily on **why** each thing exists.
 
REST is an architectural style built around resources, HTTP, and constraints that make client-server systems scalable.
 
---
 
## 1. First: what problem are we actually trying to solve?
 
Imagine we're building a **Todo application**.
 
The frontend has:
 
```text
User
 ├── sees tasks
 ├── creates a task
 ├── updates a task
 └── deletes a task
```
 
Now suppose the frontend and backend are separate programs.
 
The frontend needs to tell the backend:
 
> "Give me my tasks."
 
Then:
 
> "Create this task."
 
Then:
 
> "Update task 10."
 
Then:
 
> "Delete task 10."
 
So naturally, we need some way for two different programs to communicate.
 
### What could we do?
 
We could invent our own communication system:
 
```text
GET_TASKS
CREATE_TASK
UPDATE_TASK
DELETE_TASK
```
 
But now imagine:
 
* Company A creates APIs one way.
* Company B creates APIs another way.
* Company C creates APIs completely differently.
Every frontend developer has to learn a completely different system.
 
So we have a problem.
 
### The root problem
 
**How can different clients and servers communicate in a predictable, standardized way?**
 
That's where HTTP + REST conventions become useful.
 
REST is a set of agreements that allow clients and servers to communicate without needing to coordinate their internal implementations.
 
---
 
## 2. So what is an API?
 
Before REST, let's understand **API**.
 
Suppose our Java backend has:
 
```java
public Task getTask(int id) {
    // get task from database
}
```
 
This is a Java method.
 
But your React frontend cannot directly call:
 
```java
getTask(10);
```
 
Why?
 
Because React is running somewhere else.
 
Maybe:
 
```text
Browser
   |
   | Internet
   ↓
Java Backend
   |
   ↓
Database
```
 
The browser and Java program are separate processes.
 
So we need a communication boundary.
 
That boundary is the **API**.
 
Think of an API as a **restaurant waiter**.
 
```text
You
 |
 | "Give me biryani"
 ↓
Waiter/API
 |
 ↓
Kitchen/Backend
 |
 ↓
Food/Data
```
 
You don't walk into the kitchen.
 
Similarly:
 
```text
Frontend
   |
   | HTTP Request
   ↓
API
   |
   ↓
Backend
   |
   ↓
Database
```
 
---
 
## 3. But why REST?
 
Now comes the important question.
 
> **Okay, we have an API. But how should we design it?**
 
We need rules.
 
For example:
 
Should we write:
 
```text
/getUsers
```
 
or
 
```text
/users
```
 
Should updating a user use:
 
```text
POST
```
 
or
 
```text
PUT
```
 
What does:
 
```text
404
```
 
mean?
 
What about:
 
```text
401
403
500
```
 
REST gives us a common way of thinking about these things.
 
REST itself isn't a technology you install; it is an architectural style based on constraints.
 
---
 
## 4. REST — understand the name first
 
REST means:
 
**REpresentational State Transfer**
 
That sounds scary.
 
Let's break it down.
 
### Representational
 
Suppose the database contains:
 
```text
User
----------------
id = 10
name = Tamil
age = 21
```
 
The actual resource is the **user**.
 
But how do we send that user across the network?
 
We need a representation.
 
Usually:
 
```json
{
    "id": 10,
    "name": "Tamil",
    "age": 21
}
```
 
So:
 
```text
Actual resource
      ↓
Representation
      ↓
JSON
```
 
The same resource could theoretically have different representations such as JSON, HTML, or XML.
 
---
 
## 5. State
 
Now imagine a shopping cart.
 
At 10 AM:
 
```text
Cart
items = 1
total = ₹100
```
 
At 10:30:
 
```text
Cart
items = 3
total = ₹500
```
 
The **state** means the current condition of the resource.
 
So:
 
```text
Resource = Cart
 
State =
items = 3
total = ₹500
```
 
State is the current condition of a resource rather than its entire history.
 
---
 
## 6. Transfer
 
Now we have:
 
```text
Resource
   ↓
State
   ↓
Representation
```
 
We need to move that representation between client and server.
 
That's the **Transfer** part.
 
For example:
 
```text
Browser
   |
   | GET /users/10
   ↓
Java Server
   |
   | JSON
   ↓
Browser
```
 
That's REST in a very simple mental model:
 
> **Transfer a representation of a resource's current state between client and server using HTTP.**
 
---
 
## 7. Now the real question: how do we design our URLs?
 
Suppose we have users.
 
A beginner might write:
 
```text
/getUser
```
 
But there's a problem.
 
The URL is describing an **action**.
 
REST prefers the URL to describe the **thing/resource**.
 
So instead:
 
```text
/users
```
 
Then HTTP tells us what we want to do.
 
For example:
 
```text
GET /users
```
 
means:
 
> Give me users.
 
And:
 
```text
POST /users
```
 
means:
 
> Create a user.
 
This gives us a beautiful separation:
 
```text
URL      = WHAT
HTTP     = WHAT TO DO
```
 
REST explicitly recommends modeling resources as nouns rather than actions.
 
---
 
## 8. Why plural `/users`?
 
This is easier if we think in terms of a database.
 
Imagine:
 
```text
users
--------------------------------
1   Tamil
2   Arun
3   Ravi
```
 
`/users` represents the **collection**.
 
Now:
 
```text
/users/2
```
 
means:
 
> User number 2 inside the users collection.
 
So:
 
```text
/users
```
 
→ collection
 
```text
/users/2
```
 
→ one resource inside that collection.
 
The convention recommends plural resource names, including when accessing one item: `/books/123`, not `/book/123`.
 
---
 
## 9. Now ask: how do we perform different operations?
 
We have:
 
```text
/users
```
 
But we need:
 
```text
GET     → read
POST    → create
PUT     → replace
PATCH   → partial update
DELETE  → remove
```
 
Why do we need different methods?
 
Because the server needs to understand our **intent**.
 
Think about a library.
 
```text
GET
"Show me the book."
 
POST
"Add a new book."
 
PUT
"Replace this book with this new version."
 
PATCH
"Change only the title."
 
DELETE
"Remove this book."
```
 
These five HTTP methods are defined according to their intended purposes and idempotency properties.
 
---
 
## 10. Let's see this in Java
 
Suppose we have:
 
```java
class Task {
    private int id;
    private String title;
    private boolean done;
 
    public Task(int id, String title, boolean done) {
        this.id = id;
        this.title = title;
        this.done = done;
    }
}
```
 
Now imagine our Java server exposes:
 
```text
/v1/tasks
```
 
We can have:
 
```text
GET    /v1/tasks
POST   /v1/tasks
 
GET    /v1/tasks/10
PUT    /v1/tasks/10
PATCH  /v1/tasks/10
DELETE /v1/tasks/10
```
 
Notice something beautiful.
 
We don't need:
 
```text
/getTasks
/createTask
/updateTask
/deleteTask
```
 
The **HTTP method already tells us the action**.
 
---
 
## 11. But why `/v1`?
 
Now imagine our API is being used by 10,000 applications.
 
Today we return:
 
```json
{
    "id": 10,
    "name": "Tamil"
}
```
 
Tomorrow we decide:
 
```json
{
    "userId": 10,
    "fullName": "Tamil"
}
```
 
Oops.
 
Old frontend code expects:
 
```java
response.get("id");
response.get("name");
```
 
Now we changed:
 
```text
id   → userId
name → fullName
```
 
Existing clients break.
 
### So what do we need?
 
We need different versions.
 
```text
/v1/users
/v2/users
```
 
Now old applications can continue using:
 
```text
/v1/users
```
 
while new applications use:
 
```text
/v2/users
```
 
Versioning is recommended especially for breaking changes, with URL-path versioning such as `/v1/books` being a pragmatic common approach.
 
---
 
## 12. Now let's understand GET
 
Suppose:
 
```text
GET /v1/tasks/10
```
 
We're asking:
 
> Give me task 10.
 
The server might return:
 
```json
{
    "id": 10,
    "title": "Learn Java",
    "done": false
}
```
 
In Java, conceptually:
 
```java
@GetMapping("/v1/tasks/{id}")
public Task getTask(@PathVariable int id) {
 
    return taskService.getTask(id);
}
```
 
The important idea isn't the Spring syntax.
 
The important idea is:
 
```text
GET
 ↓
READ
 ↓
Don't change server state
```
 
That's why GET is called **safe**.
 
Calling:
 
```text
GET /tasks/10
```
 
once or 1,000 times doesn't itself modify the task.
 
GET is safe and idempotent because it causes no server-side state change.
 
---
 
## 13. Now POST
 
Suppose we want to create:
 
```text
Learn Java
```
 
We send:
 
```text
POST /v1/tasks
```
 
Body:
 
```json
{
    "title": "Learn Java"
}
```
 
Java:
 
```java
@PostMapping("/v1/tasks")
public Task createTask(@RequestBody Task task) {
 
    return taskService.createTask(task);
}
```
 
The server creates:
 
```text
id = 1
title = Learn Java
```
 
Usually the response is:
 
```text
201 Created
```
 
Why 201?
 
Because:
 
> **Something new was created.**
 
`201 Created` is distinguished from `200 OK`: a POST that creates a new resource should return 201, generally with a Location pointing to the new resource.
 
---
 
## 14. Here's where idempotency becomes important
 
Imagine:
 
```text
POST /tasks
```
 
with:
 
```json
{
    "title": "Learn Java"
}
```
 
First request:
 
```text
Task 1 created
```
 
Send exactly the same request again:
 
```text
Task 2 created
```
 
Again:
 
```text
Task 3 created
```
 
So:
 
```text
POST
 ↓
create something
 ↓
repeating it can create another thing
```
 
Therefore POST is generally **not idempotent**.
 
Identical POST requests can create distinct resources each time.
 
---
 
## 15. What is idempotency?
 
This sounds complicated but it's actually simple.
 
Ask:
 
> **If I perform the same operation multiple times, does the final server state keep changing?**
 
Suppose:
 
```text
name = A
```
 
We send:
 
```text
PUT name = B
```
 
First:
 
```text
A → B
```
 
Again:
 
```text
B → B
```
 
Again:
 
```text
B → B
```
 
The final state remains:
 
```text
B
```
 
Therefore PUT is idempotent.
 
This same reasoning applies for PUT/PATCH updates.
 
---
 
## 16. PUT vs PATCH
 
This is one of the most important REST concepts.
 
Suppose our user is:
 
```json
{
    "name": "Tamil",
    "email": "tamil@gmail.com",
    "city": "Chennai"
}
```
 
I only want to change:
 
```text
city
```
 
### PATCH
 
Send:
 
```json
{
    "city": "Bangalore"
}
```
 
Meaning:
 
> Change this part.
 
### PUT
 
Send:
 
```json
{
    "name": "Tamil",
    "email": "tamil@gmail.com",
    "city": "Bangalore"
}
```
 
Meaning:
 
> Replace the whole representation with this.
 
So the mental model is:
 
```text
PATCH → change part
 
PUT → replace whole thing
```
 
**Warning:** using PUT with only `city` can wipe omitted fields such as `name` and `email`.
 
---
 
## 17. DELETE
 
Now:
 
```text
DELETE /v1/tasks/10
```
 
Java:
 
```java
@DeleteMapping("/v1/tasks/{id}")
public void deleteTask(@PathVariable int id) {
 
    taskService.deleteTask(id);
}
```
 
Usually:
 
```text
204 No Content
```
 
Why?
 
Because we've deleted the resource.
 
There's nothing meaningful left to return.
 
`204 No Content` is recommended for successful deletes where there is no response body.
 
---
 
## 18. Now a bigger problem: what if there are 1 million tasks?
 
Suppose:
 
```text
GET /v1/tasks
```
 
returns:
 
```text
1,000,000 tasks
```
 
That's terrible.
 
Why?
 
```text
Database
   ↓
1,000,000 rows
   ↓
Java objects
   ↓
JSON serialization
   ↓
Huge network response
   ↓
Browser
```
 
The user might only need the first 20.
 
### So how do we solve it?
 
**Pagination.**
 
Instead of:
 
```text
GET /tasks
```
 
we can use:
 
```text
GET /tasks?page=1&limit=20
```
 
Meaning:
 
> Give me page 1, 20 tasks.
 
Large collections should be returned in chunks rather than dumping the entire dataset.
 
---
 
## 19. Java pagination idea
 
Suppose:
 
```java
@GetMapping("/v1/tasks")
public List<Task> getTasks(
        @RequestParam(defaultValue = "1") int page,
        @RequestParam(defaultValue = "10") int limit) {
 
    return taskService.getTasks(page, limit);
}
```
 
If:
 
```text
page = 2
limit = 10
```
 
then conceptually:
 
```text
Tasks:
 
1  2  3  4  5  6  7  8  9  10
11 12 13 14 15 16 17 18 19 20
21 22 ...
 
       ↑
     page 2
```
 
Response could be:
 
```json
{
    "data": [
        ...
    ],
    "pagination": {
        "page": 2,
        "limit": 10,
        "totalItems": 100,
        "totalPages": 10
    }
}
```
 
This uses a general `{data, pagination}` structure and includes page, limit, total items, and total pages.
 
---
 
## 20. Now filtering
 
Suppose we only want completed tasks.
 
Instead of downloading everything:
 
```text
GET /tasks
```
 
we can say:
 
```text
GET /tasks?done=true
```
 
Meaning:
 
> Give me only tasks where done = true.
 
Java:
 
```java
@GetMapping("/v1/tasks")
public List<Task> getTasks(
        @RequestParam(required = false) Boolean done) {
 
    return taskService.findTasks(done);
}
```
 
We can combine:
 
```text
/tasks?done=true&title=java
```
 
The task API supports filtering using query parameters such as `done` and `title`.
 
---
 
## 21. Now sorting
 
Suppose we have:
 
```text
Task C
Task A
Task B
```
 
We might want:
 
```text
A
B
C
```
 
So:
 
```text
GET /tasks?sortBy=title&sortOrder=ascending
```
 
Now the server sorts.
 
But there's an important problem.
 
What if the client sends:
 
```text
sortBy=someRandomDatabaseColumn
```
 
That's dangerous.
 
So we create an **allow-list**:
 
```java
Set<String> allowedFields =
        Set.of("id", "title", "createdAt");
```
 
Then:
 
```java
if (!allowedFields.contains(sortBy)) {
    throw new IllegalArgumentException("Invalid sort field");
}
```
 
Use an allow-list for sortable fields rather than allowing arbitrary field names.
 
---
 
## 22. Now the most important debugging concept: status codes
 
Suppose frontend calls:
 
```text
GET /users/999
```
 
but user 999 doesn't exist.
 
How should Java tell the frontend?
 
We could return:
 
```text
200 OK
```
 
with:
 
```json
{
    "error": "user not found"
}
```
 
But this is confusing.
 
Why?
 
Because:
 
```text
200
```
 
means:
 
> The request succeeded.
 
So we have a better solution.
 
```text
404 Not Found
```
 
Now the HTTP status itself communicates the problem.
 
Use meaningful status codes rather than returning `200` for everything.
 
---
 
## 23. The status-code mental model
 
Don't memorize 50 codes.
 
First ask:
 
> **Who is responsible for the problem?**
 
### 2xx
 
```text
2xx = SUCCESS
```
 
Examples:
 
```text
200 → worked
201 → created
204 → worked, nothing to return
```
 
### 4xx
 
```text
4xx = CLIENT PROBLEM
```
 
Something is wrong with the request.
 
```text
400 → bad request
401 → not authenticated
403 → authenticated but not allowed
404 → resource doesn't exist
409 → conflict
422 → validation failed
429 → too many requests
```
 
### 5xx
 
```text
5xx = SERVER PROBLEM
```
 
The server failed.
 
The five status-code families follow this first-digit model.
 
---
 
## 24. 401 vs 403
 
This confuses almost everyone initially.
 
Think about a security guard.
 
### 401
 
Guard says:
 
> "Who are you?"
 
You have no valid identity.
 
```text
401 Unauthorized
```
 
### 403
 
Guard says:
 
> "I know who you are, but you're not allowed inside."
 
```text
403 Forbidden
```
 
So:
 
```text
401 → Who are you?
 
403 → I know you, but NO.
```
 
This is the same authentication-vs-authorization distinction.
 
---
 
## 25. 404 has a subtle rule
 
Consider:
 
```text
GET /users/999
```
 
User doesn't exist.
 
```text
404
```
 
Makes sense.
 
But now:
 
```text
GET /users?name=Zack
```
 
Suppose nobody is named Zack.
 
Should that be:
 
```text
404
```
 
No.
 
It's a **successful search that found zero results**.
 
So:
 
```json
{
    "data": []
}
```
 
with:
 
```text
200 OK
```
 
Think:
 
```text
GET /users/999
       ↓
"I asked for ONE specific thing"
       ↓
Doesn't exist
       ↓
404
```
 
versus:
 
```text
GET /users?name=Zack
       ↓
"I asked you to SEARCH"
       ↓
Search succeeded
       ↓
Found nothing
       ↓
200 + []
```
 
---
 
## 26. Now statelessness — very important
 
Imagine you have:
 
```text
Server 1
Server 2
Server 3
```
 
And a load balancer:
 
```text
                 ┌── Server 1
Client → Load ───┼── Server 2
        Balancer └── Server 3
```
 
Request 1:
 
```text
Client → Server 1
```
 
Request 2:
 
```text
Client → Server 2
```
 
Request 3:
 
```text
Client → Server 3
```
 
No problem **if every request contains everything needed to process it**.
 
That's the idea of statelessness.
 
For example:
 
```http
GET /users/10
 
Authorization: Bearer xyz
```
 
The server doesn't need to say:
 
> "Oh, I remember this client from request #1."
 
The request carries the information needed.
 
Statelessness is one of REST's major scalability benefits because any server can handle any request.
 
---
 
## 27. Why is this useful?
 
Without statelessness:
 
```text
User
 ↓
Server 1
 ↓
Session stored only here
```
 
Next request:
 
```text
User
 ↓
Server 2
```
 
Server 2 says:
 
> "Who are you? I don't know you."
 
Now we have a problem.
 
With stateless requests:
 
```text
Request
 ├── authentication
 ├── parameters
 ├── resource
 └── operation
```
 
Any server can process it.
 
Therefore:
 
```text
More users
   ↓
Add more servers
   ↓
Load balancer distributes requests
   ↓
Any server can handle any request
```
 
That's one of the core reasons statelessness matters.
 
---
 
## 28. Now custom actions
 
Sometimes an operation doesn't naturally fit CRUD.
 
For example:
 
```text
Archive organization
Clone project
Send email
```
 
Is:
 
```text
archive
```
 
really just an update?
 
Maybe not.
 
Imagine archiving an organization triggers:
 
```text
Change status
     ↓
Delete/cancel projects
     ↓
Cancel tasks
     ↓
Send emails
     ↓
Revoke access
     ↓
Start background cleanup
```
 
That's much more than:
 
```text
status = archived
```
 
So we can model it as:
 
```text
POST /organizations/5/archive
```
 
Use POST for custom actions that don't map cleanly to ordinary CRUD semantics.
 
---
 
## 29. How does this look in Java?
 
With Spring Boot:
 
```java
@PostMapping("/v1/organizations/{id}/archive")
public Organization archiveOrganization(
        @PathVariable int id) {
 
    return organizationService.archive(id);
}
```
 
Notice:
 
```text
POST
 ↓
custom operation
 ↓
/organizations/{id}/archive
```
 
---
 
## 30. Error responses
 
Here's another real-world problem.
 
Suppose the frontend sends:
 
```json
{
    "email": "hello"
}
```
 
But the email is invalid.
 
We could return:
 
```text
422
```
 
with:
 
```json
{
    "error": "invalid email"
}
```
 
But as our application grows, we might have:
 
```text
Endpoint A
→ {error: "..."}
 
Endpoint B
→ {message: "..."}
 
Endpoint C
→ "something went wrong"
 
Endpoint D
→ HTML error page
```
 
Now frontend developers have a nightmare.
 
### So what's the root problem?
 
**Inconsistent error communication.**
 
### Solution?
 
Create one standard error structure.
 
For example:
 
```json
{
    "error": {
        "code": "validation_failed",
        "message": "Some fields are invalid.",
        "details": [
            {
                "field": "email",
                "issue": "must be a valid email"
            }
        ],
        "requestId": "req_123"
    }
}
```
 
Use a consistent, machine-readable error envelope with `code`, `message`, `details`, and `requestId`.
 
---
 
## 31. Putting everything together in Java
 
Now imagine we're building:
 
## Task Management API
 
Our resource is:
 
```text
tasks
```
 
So our API becomes:
 
```text
GET    /v1/tasks
POST   /v1/tasks
 
GET    /v1/tasks/10
PUT    /v1/tasks/10
PATCH  /v1/tasks/10
DELETE /v1/tasks/10
```
 
And:
 
```text
GET /v1/tasks?page=1&limit=10
```
 
for pagination.
 
```text
GET /v1/tasks?done=true
```
 
for filtering.
 
```text
GET /v1/tasks?sortBy=title&sortOrder=ascending
```
 
for sorting.
 
This mirrors the task API structure using `/v1/tasks`, separating list/create from get/update/delete by HTTP method.
 
---
 
## 32. The Java/Spring Boot structure
 
A clean Java backend might look like:
 
```text
src/main/java/com/example/taskapi
 
        ├── controller
        │      └── TaskController.java
        │
        ├── service
        │      └── TaskService.java
        │
        ├── repository
        │      └── TaskRepository.java
        │
        ├── model
        │      └── Task.java
        │
        └── exception
               └── GlobalExceptionHandler.java
```
 
Why separate these?
 
Because each layer has a job.
 
```text
HTTP request
     ↓
Controller
     ↓
Service
     ↓
Repository
     ↓
Database
```
 
Think of it like a restaurant:
 
```text
Customer
   ↓
Waiter       → Controller
   ↓
Manager      → Service
   ↓
Storage      → Repository
   ↓
Kitchen/DB   → Database
```
 
---
 
## 33. Controller
 
The controller deals with HTTP.
 
```java
@RestController
@RequestMapping("/v1/tasks")
public class TaskController {
 
    @GetMapping
    public List<Task> getTasks() {
        return taskService.getTasks();
    }
 
    @PostMapping
    public Task createTask(@RequestBody Task task) {
        return taskService.createTask(task);
    }
 
    @GetMapping("/{id}")
    public Task getTask(@PathVariable int id) {
        return taskService.getTask(id);
    }
 
    @PutMapping("/{id}")
    public Task updateTask(
            @PathVariable int id,
            @RequestBody Task task) {
 
        return taskService.updateTask(id, task);
    }
 
    @DeleteMapping("/{id}")
    public void deleteTask(@PathVariable int id) {
        taskService.deleteTask(id);
    }
}
```
 
The important part isn't memorizing annotations.
 
Understand the mapping:
 
```text
HTTP request
      ↓
Controller method
```
 
For example:
 
```text
GET /v1/tasks/10
          ↓
getTask(10)
```
 
---
 
## 34. Service
 
Why not put everything in the controller?
 
Because then the controller becomes:
 
```text
HTTP parsing
+ validation
+ business logic
+ database logic
+ error handling
+ sorting
+ pagination
+ ...
```
 
Eventually it becomes a monster.
 
So we separate business logic.
 
```java
@Service
public class TaskService {
 
    public Task getTask(int id) {
 
        return taskRepository.findById(id)
                .orElseThrow(
                    () -> new TaskNotFoundException(id)
                );
    }
}
```
 
Now:
 
```text
Controller
"Someone requested task 10."
 
Service
"Let's figure out whether task 10 exists and what business rules apply."
 
Repository
"Let's get task 10 from the database."
```
 
---
 
## 35. Repository
 
The repository deals with persistence.
 
With Spring Data JPA:
 
```java
public interface TaskRepository
        extends JpaRepository<Task, Integer> {
}
```
 
Now:
 
```java
taskRepository.findById(10);
```
 
can retrieve the task.
 
So our complete mental flow becomes:
 
```text
GET /v1/tasks/10
          ↓
TaskController
          ↓
TaskService
          ↓
TaskRepository
          ↓
Database
          ↓
Task
          ↓
JSON
          ↓
HTTP Response
```
 
---
 
## 36. The entire REST idea in one picture
 
Remember this:
 
```text
                    CLIENT
                      |
                      | HTTP Request
                      ↓
              ┌─────────────────┐
              │  REST API       │
              │                 │
              │ URL = RESOURCE  │
              │ METHOD = INTENT │
              └────────┬────────┘
                       |
              ┌────────┴────────┐
              ↓                 ↓
         Controller          Status Code
              |
              ↓
           Service
              |
              ↓
         Repository
              |
              ↓
           Database
```
 
And the client gets:
 
```text
HTTP Response
 ├── Status Code
 ├── Headers
 └── Representation
       ↓
      JSON
```
 
---
 
## 37. The most important mental model
 
Don't memorize REST as a list of rules.
 
Instead, think:
 
### We have a resource.
 
```text
Task
```
 
### We give that resource a URL.
 
```text
/tasks
/tasks/10
```
 
### We use HTTP methods to express intent.
 
```text
GET     → read
POST    → create/action
PUT     → replace
PATCH   → partial update
DELETE  → remove
```
 
### We use status codes to communicate the result.
 
```text
200 → worked
201 → created
204 → worked, nothing to return
400 → request is malformed
401 → not authenticated
403 → not allowed
404 → resource doesn't exist
409 → conflict
422 → validation problem
500 → server problem
```
 
### We use query parameters for collection operations.
 
```text
?page=2
&limit=20
&sortBy=title
&sortOrder=ascending
&done=true
```
 
### We keep the API predictable.
 
```text
/v1/tasks
/v1/users
/v1/projects
```
 
Same patterns everywhere.
 
This consistency is one of the major practical principles of good API design.
 
---
 
## 38. Finally: REST vs Java
 
This is **very important**.
 
REST is **not Java**.
 
REST is an architectural style.
 
Java is a programming language.
 
Spring Boot is a Java framework.
 
So:
 
```text
REST
 ↑
architectural style
 
Spring Boot
 ↑
Java framework
 
Java
 ↑
programming language
```
 
You can build a REST API using:
 
```text
Java + Spring Boot
Java + Jakarta REST
Go + net/http
Python + FastAPI
Node.js + Express
C# + ASP.NET
```
 
The original worked example happened to use Go's standard HTTP library, but the REST principles themselves are independent of Go.
 
---
 
## 🧠 If you remember only this
 
When someone asks you **"What is REST API?"**, don't start with a definition.
 
Think through the problem:
 
```text
We need frontend ↔ backend communication.
              ↓
We need a common communication style.
              ↓
HTTP gives us standardized communication.
              ↓
REST gives us a way to organize APIs around resources.
              ↓
URL identifies the resource.
              ↓
HTTP method tells the intent.
              ↓
Status code tells the result.
              ↓
JSON carries the representation.
              ↓
Stateless requests allow easy scaling.
              ↓
Pagination/filtering/sorting make large collections manageable.
              ↓
Versioning prevents breaking existing clients.
              ↓
Consistent errors make the API predictable.
```
 
That is the **first-principles picture of REST**.
 
And once this picture is clear, things like `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `404`, `401`, pagination, versioning, and Spring Boot annotations stop looking like random rules — **each one exists because it solves a specific problem.**
