# Serialization: How Different Programming Languages Talk to Each Other

This blog is going to explain **Serialization** and **Deserialization**.

---

## The Problem Statement

Let's imagine that we have an application with a frontend built using **React**, which uses **JavaScript** as its programming language.

Now imagine that the backend is built using **Rust**.

If the frontend needs to collect some information from the backend, it needs to make a **GET request**.

This part is known to everyone.

But do we actually know how they communicate with each other beyond the **headers, URL, routing, HTTP methods**, etc.?

So, how can JavaScript communicate with a backend written in Rust or any other programming language?

Here, **Serialization enters as a hero**.

---

## What is Serialization?

Serialization can be thought of as **arranging data into a particular format**.

We have probably heard the word *serialized* before, where something is arranged in a particular order or format.

The same idea happens here.

**Serialization is the process of converting data from a programming language's native format into a common format that can be transmitted or stored.**

Before going further, we need to create a simple mental map.

The frontend will make a request using its own language's native data format.

Then, that data will be converted into some common format.

We will discuss what that format is later.

For now, this much is enough.

You may now have a question:

> **What is that format you are talking about?**

Yes — that format can be divided into **two types**.

---

## Format Types

Formats are the common standards into which we convert our own language's data or objects.

There are mainly two types:

1. **Text-Based Format**
2. **Binary Format**

---

## 1. Text-Based Format

Text-based formats are **human-readable**.

Some examples are:

- JSON
- XML
- YAML

For communication between applications, **JSON is one of the most commonly used formats**, especially with HTTP REST APIs.

The basic flow looks like this:

```text
Frontend
    ↓
Serialization
    ↓
JSON Format
```

For now, up to this point, this process is **Serialization**.

The frontend takes its native data and converts it into a common format such as JSON.

---

## 2. Binary Format

The other type is the **Binary Format**.

As we know, computers ultimately work with binary data — **0s and 1s**.

Binary serialization formats represent data in a compact binary form.

The main purpose is to make the data more **compact and efficient for transmission and storage**.

Some examples include:

- Protocol Buffers (Protobuf)
- Avro

Unlike text-based formats such as JSON, binary formats are generally **not human-readable**.

---

# Deserialization

Now we have reached the reverse process.

**Deserialization is the reverse of Serialization.**

Once the JSON format is sent through an HTTP request and received by the backend, it needs to be converted into the backend's native data format.

This process is called **Deserialization**.

The basic idea is:

```text
Common Format
      ↓
Native Data
```

For example:

```text
JSON → Rust Struct
```

or:

```text
JSON → Go Struct
```

---

# Serialization vs Deserialization

Let's clearly understand the difference between these two.

## Serialization

Serialization is the process of converting **native data into a common format**.

```text
Native Data → Common Format
```

For example:

```text
JavaScript Object → JSON
```

or:

```text
Go Struct → JSON
```

or:

```text
Python Object → JSON
```

This usually happens **before the data is sent across the network**.

---

## Deserialization

Deserialization is the process of converting the **common format back into native data**.

```text
Common Format → Native Data
```

For example:

```text
JSON → Rust Struct
```

or:

```text
JSON → Go Struct
```

or:

```text
JSON → Python Object
```

So remember this simple rule:

```text
Serialize   = Native → Common

Deserialize = Common → Native
```

Once you understand these two arrows, most of the concept becomes easy.

---

# What Does Language-Agnostic Mean?

You will sometimes hear the word **language-agnostic** in backend development.

It simply means:

> **The format does not care which programming language you are using.**

For example:

```text
JavaScript → JSON
Python     → JSON
Go         → JSON
Rust       → JSON
Java       → JSON
```

All these languages can communicate using the same common format.

The important thing is that each language knows how to convert its native data into the common format and how to convert it back.

---

# Let's Connect Everything Here

Now let's connect all the concepts together and understand what actually happens when a frontend communicates with a backend.

---

## Step 1 — Gather the Data

The frontend collects the user's input.

```text
User Input
    ↓
JavaScript Object
```

---

## Step 2 — Serialize

The JavaScript object is converted into JSON.

```text
JavaScript Object
       ↓
      JSON
```

---

## Step 3 — Send Through HTTP

The JSON is placed inside the HTTP request body.

```text
HTTP Request
     ↓
JSON Body
```

---

## Step 4 — Network Transmission

The network handles the lower-level communication.

Conceptually:

```text
JSON
 ↓
Bits
 ↓
Network
 ↓
Bits
 ↓
JSON
```

---

## Step 5 — Server Deserializes

The server receives the JSON and converts it into its own native structure.

For example:

```text
JSON
 ↓
Rust Struct
```

---

## Step 6 — Server Processes the Data

The backend can now perform its business logic.

For example:

```text
Validate User
      ↓
Save to Database
      ↓
Create Response
```

---

## Step 7 — Server Serializes the Response

The server converts its native data into JSON.

```text
Rust Struct
     ↓
   JSON
```

---

## Step 8 — Client Deserializes

The browser receives the JSON and converts it back into a JavaScript object.

```text
JSON
 ↓
JavaScript Object
 ↓
Update UI
```

---

# The Complete Flow

Now let's put everything together:

```text
              CLIENT
          JavaScript Object
                  ↓
             SERIALIZE
                  ↓
                 JSON
                  ↓
            HTTP / Network
                  ↓
                 JSON
                  ↓
            DESERIALIZE
                  ↓
              Rust Struct
                  ↓
               PROCESS
                  ↓
              Rust Struct
                  ↓
             SERIALIZE
                  ↓
                 JSON
                  ↓
            HTTP / Network
                  ↓
                 JSON
                  ↓
            DESERIALIZE
                  ↓
          JavaScript Object
                  ↓
              Update UI
```

This is the complete request-response flow involving **Serialization and Deserialization**.

---

# One Important Backend Concept: Don't Send Secrets

Serialization isn't only about converting data.

You also need to think about **what data should actually leave your application**.

Imagine your user object contains:

```text
name
email
password
```

You probably don't want the password appearing in your JSON response.

The important principle is:

> **Not every field in your internal object should necessarily become part of your API response.**

Your **internal model** and your **external API representation** don't always have to be identical.

For example, your backend might internally have:

```text
User
 ├── id
 ├── name
 ├── email
 └── password
```

But the API response might contain only:

```json
{
    "id": 1,
    "name": "Ada",
    "email": "ada@example.com"
}
```

The password stays inside the backend and is never exposed through the API response.

---

# Final Takeaway

Serialization is the process of converting:

```text
Native Data → Common Format
```

Deserialization is the reverse:

```text
Common Format → Native Data
```

For example:

```text
JavaScript Object
       ↓
   Serialization
       ↓
      JSON
       ↓
     Network
       ↓
      JSON
       ↓
  Deserialization
       ↓
    Rust Struct
```

So whenever you see:

```text
Frontend → API → Backend
```

remember what is happening underneath:

```text
Native Data
     ↓
Serialize
     ↓
JSON
     ↓
Network
     ↓
JSON
     ↓
Deserialize
     ↓
Native Data
```

**Two different machines.**

**Two different languages.**

**One common format.**
