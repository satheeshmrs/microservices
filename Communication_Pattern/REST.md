# 🌐 REST Explained in Layman's Terms

## 🔹 What is REST?

**REST** = **Representational State Transfer**.\
It's a style of designing **APIs** where clients (apps, browsers) talk
to servers using **resources**.

👉 Think of it like a **restaurant menu** 🍔:\
- **Menu items** = resources (users, products, orders).\
- **Waiter** = HTTP methods (GET, POST, PUT, DELETE).\
- **Kitchen** = server.

You order something, the kitchen prepares it, and the waiter brings it
back.

------------------------------------------------------------------------

## 🔹 REST vs RESTful

-   **REST** → the **concept** (the rules/architecture).\
-   **RESTful API** → an API that **follows REST rules properly**.

👉 Example:\
- `GET /getUsersList` → ❌ not RESTful (RPC-like).\
- `GET /users` → ✅ RESTful (resource-based).

So:\
- **REST** = theory.\
- **RESTful** = applying it correctly.

------------------------------------------------------------------------

## 🔹 REST Best Practices

1.  **Use nouns, not verbs, in URLs**\
    ✅ `GET /users/1`\
    ❌ `GET /getUserById`

2.  **Use proper HTTP methods**

    -   `GET` → read\
    -   `POST` → create\
    -   `PUT/PATCH` → update\
    -   `DELETE` → remove

3.  **Use plural nouns for collections**\
    ✅ `/users`\
    ❌ `/userList`

4.  **Use status codes properly**

    -   `200 OK` → success\
    -   `201 Created` → new resource added\
    -   `404 Not Found` → not found\
    -   `500 Internal Server Error` → server problem

5.  **Support filtering, sorting, pagination**

        GET /products?category=shoes&page=2&sort=price

6.  **Version your API**\
    Example: `/api/v1/users`

7.  **Secure it**\
    Always use **HTTPS**, authentication (JWT, OAuth2).

------------------------------------------------------------------------

## 🔹 Limitations of REST

⚠️ **Over-fetching** → Get more data than needed.\
Example: `GET /users/1` returns name, email, phone, address when you
only need the name.

⚠️ **Under-fetching** → Need multiple requests.\
Example: To fetch user + posts:\
- `GET /users/1`\
- `GET /users/1/posts`

⚠️ **No real-time updates** → Only request/response, no live push.

⚠️ **Statelessness = extra data** → Each request must include all info
(like tokens).

------------------------------------------------------------------------

## ✅ Summary

-   **REST** = API design concept.\
-   **RESTful API** = A proper implementation of REST rules.\
-   **Best Practices** = Use nouns, methods, status codes, versioning,
    security.\
-   **Limitations** = Over-fetching, under-fetching, no real-time,
    repeated data.

👉 REST is like a **restaurant menu**: clear and structured, but
sometimes you need 

# 🔄 PATCH vs PUT in REST APIs

## 🔹 PUT (Replace)

-   **PUT** updates or replaces an **entire resource**.\
-   If the resource exists → it is **replaced**.\
-   If it doesn't exist → sometimes it is **created**.

👉 Think of PUT like **replacing an old document with a new one**. 📄

### Example

``` http
PUT /users/1
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

-   Replaces the whole user object.\
-   If `"email"` is missing, it might get erased!

------------------------------------------------------------------------

## 🔹 PATCH (Partial Update)

-   **PATCH** updates **only part of a resource**.\
-   Leaves other fields unchanged.

👉 Think of PATCH like **editing a few lines in the document**. ✏️

### Example

``` http
PATCH /users/1
{
  "email": "alice_new@example.com"
}
```

-   Only the email field is updated.\
-   `id` and `name` stay the same.

------------------------------------------------------------------------

## 🔹 When to Use PUT

✅ Use **PUT** when:\
- You want to **replace the entire object**.\
- The client always sends the **full resource**.\
- Example: Updating a user profile form (all fields).

------------------------------------------------------------------------

## 🔹 When to Use PATCH

✅ Use **PATCH** when:\
- You only need to **update part of a resource**.\
- You want to avoid sending unnecessary data.\
- Example: Updating just the **email** or **status** of a user.

------------------------------------------------------------------------

## 🔹 Analogy

-   **PUT** → Replace your **resume** with a new one.\
-   **PATCH** → Fix a typo in your **resume**.

------------------------------------------------------------------------

## 🔹 Summary (Cheat Sheet)

  ------------------------------------------------------------------------
  Feature             PUT 🔄 (Replace)         PATCH ✏️ (Partial)
  ------------------- ------------------------ ---------------------------
  Purpose             Replace entire resource  Update part of a resource

  Requires all        ✅ Yes (usually)         ❌ No (only what changes)
  fields?                                      

  Risk                Missing fields may get   Safer for partial updates
                      erased                   

  Analogy             Replace whole document   Edit just one line
  ------------------------------------------------------------------------

------------------------------------------------------------------------

## ✅ Rule of Thumb

-   **PUT** → Full replacement.\
-   **PATCH** → Partial update.


