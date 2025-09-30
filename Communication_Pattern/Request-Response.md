# 🌐 REST vs GraphQL vs gRPC

## 🔹 What Are They?

Three major **API communication styles**:\
- **REST** → Resource-based, uses HTTP verbs.\
- **GraphQL** → Query language, client specifies exactly what data they
want.\
- **gRPC** → Remote Procedure Call, binary protocol for high
performance.

------------------------------------------------------------------------

## 🔹 Flowchart: How They Work

``` mermaid
flowchart TD
    A[Client] -->|HTTP GET/POST/PUT/DELETE| B[REST Server]
    A -->|GraphQL Query | C[GraphQL Server]
    A -->|RPC Method Call binary| D[gRPC Server]

    B -->|JSON/XML Response| A
    C -->|JSON exact fields| A
    D -->|Protobuf binary| A
```

------------------------------------------------------------------------

## 🔹 REST (Representational State Transfer)

### Example

    GET /users/1 → { "id": 1, "name": "Alice" }

### ✅ Pros

-   Simple, widely used.\
-   Works with HTTP tools (Postman, browser).\
-   Easy caching.

### ❌ Cons

-   Over-fetching / under-fetching.\
-   Multiple requests for complex data.

------------------------------------------------------------------------

## 🔹 GraphQL

### Example Query

``` graphql
query {
  user(id: 1) {
    name
    email
    posts {
      title
    }
  }
}
```

👉 Response:

``` json
{
  "user": {
    "name": "Alice",
    "email": "alice@mail.com",
    "posts": [{ "title": "GraphQL vs REST" }]
  }
}
```

### ✅ Pros

-   No over/under-fetching.\
-   Single endpoint.\
-   Strongly typed schema.

### ❌ Cons

-   Complex to implement.\
-   Caching is harder.

------------------------------------------------------------------------

## 🔹 gRPC (Google Remote Procedure Call)

### Example `.proto`

``` proto
service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}

message UserRequest {
  int32 id = 1;
}

message UserResponse {
  string name = 1;
  string email = 2;
}
```

### ✅ Pros

-   Extremely fast (binary over HTTP/2).\
-   Supports streaming.\
-   Strongly typed, auto-generated code.

### ❌ Cons

-   Harder debugging (binary not human-readable).\
-   Browser support limited (needs gRPC-web).

------------------------------------------------------------------------

## 🔹 Comparison Table

  ----------------------------------------------------------------------------------
  Feature                      REST 🌐          GraphQL 🔍        gRPC ⚡
  ---------------------------- ---------------- ----------------- ------------------
  **Style**                    Resource-based   Query language    RPC (functions)

  **Transport**                HTTP/1.1         HTTP/1.1 or       HTTP/2
                                                HTTP/2            

  **Data Format**              JSON, XML        JSON              Protocol Buffers
                                                                  (binary)

  **Schema**                   None             Strongly typed    Strongly typed

  **Over/Under Fetching**      Yes              No                No

  **Performance**              Medium           Good              Very High

  **Streaming**                ❌ No            ❌ Limited        ✅ Yes

  **Ease of Use**              Easy             Medium            Harder

  **Best For**                 Public APIs,     Flexible data     High-performance
                               CRUD apps        needs             microservices
  ----------------------------------------------------------------------------------

------------------------------------------------------------------------

## 🔹 Real-Life Analogy

-   **REST** → Restaurant menu 📖 (choose from fixed dishes).\
-   **GraphQL** → Custom sandwich shop 🥪 (choose exact ingredients).\
-   **gRPC** → Personal chef 👨‍🍳 (call direct methods).

------------------------------------------------------------------------

## ✅ Summary

-   **REST** → Simple, universal, but can cause over/under-fetching.\
-   **GraphQL** → Flexible, efficient, but more complex.\
-   **gRPC** → Super fast, streaming-ready, but complex and less
    human-friendly.
