# 🔍 GraphQL Explained

## 🔹 What is GraphQL?

**GraphQL** = **Query Language for APIs**, created by Facebook (2015).\
- Clients request **exactly the data they need**.\
- All queries go through **a single endpoint** (`/graphql`).\
- Responses are in **JSON**.

👉 Think of GraphQL like a **custom sandwich shop 🥪**:\
You choose the exact ingredients (fields), and you get only what you
ordered.

------------------------------------------------------------------------

## 🔹 Key Components of GraphQL

### 1. **Schema**

-   Defines the blueprint of the API.\
-   Includes **types, queries, mutations, subscriptions**.

``` graphql
type User {
  id: ID!
  name: String!
  email: String!
}

type Query {
  getUser(id: ID!): User
}
```

------------------------------------------------------------------------

### 2. **Queries**

-   For **reading data** (like GET in REST).

``` graphql
query {
  getUser(id: 1) {
    name
    email
  }
}
```

👉 Response:

``` json
{
  "getUser": {
    "name": "Alice",
    "email": "alice@mail.com"
  }
}
```

------------------------------------------------------------------------

### 3. **Mutations**

-   For **writing/updating data** (like POST/PUT/DELETE in REST).

``` graphql
mutation {
  createUser(name: "Alice", email: "alice@mail.com") {
    id
    name
  }
}
```

------------------------------------------------------------------------

### 4. **Subscriptions**

-   For **real-time updates** (via WebSockets).

``` graphql
subscription {
  newMessage {
    id
    text
  }
}
```

------------------------------------------------------------------------

### 5. **Resolvers**

-   Functions that **fetch the data** for queries/mutations.

``` js
const resolvers = {
  Query: {
    getUser: (_, { id }) => db.users.find(user => user.id === id),
  },
};
```

------------------------------------------------------------------------

### 6. **Types**

-   Strongly typed system.\
-   Built-in: `String`, `Int`, `Boolean`, `ID`, `Float`.\
-   Supports custom types.

------------------------------------------------------------------------

## 🔹 Tools in GraphQL Ecosystem

-   **Apollo Server/Client** → Most popular implementation.\
-   **GraphQL Playground / GraphiQL** → Interactive IDEs for queries.\
-   **Relay (Facebook)** → Advanced client library.\
-   **Hasura** → Auto-generates GraphQL API from a database.\
-   **Prisma** → ORM & database toolkit.\
-   **Subscriptions-Transport-WS** → Enables real-time GraphQL
    subscriptions.

------------------------------------------------------------------------

## 🔹 Best Practices for GraphQL

1.  **Design a strong schema** → clear, well-structured types.\
2.  **Use pagination** for large lists → `limit`, `offset`, cursors.\
3.  **Limit query depth/complexity** → prevent abuse with expensive
    queries.\
4.  **Enable caching** → persisted queries, Apollo caching.\
5.  **Secure your API** → Auth, disable introspection in production.\
6.  **Batch resolvers** → Use DataLoader to avoid N+1 queries.\
7.  **Return useful errors** → via `errors` field.\
8.  **Monitor & log queries** → Apollo Studio, tracing tools.

------------------------------------------------------------------------

## 🔹 Limitations of GraphQL

⚠️ More complex to implement server-side.\
⚠️ Caching is harder compared to REST.\
⚠️ Risk of expensive queries if unrestricted.\
⚠️ Overkill for simple CRUD APIs.

------------------------------------------------------------------------

## ✅ Summary

-   **GraphQL** = query language where **clients ask exactly what they
    need**.\
-   Components: **Schema, Queries, Mutations, Subscriptions, Resolvers,
    Types**.\
-   Tools: Apollo, GraphiQL, Relay, Hasura, Prisma.\
-   Best Practices: schema design, pagination, query limits, caching,
    security, error handling.\
-   Use GraphQL when clients need **flexible data** and you want to
    avoid over/under-fetching.


# 🚀 Implementing GraphQL in .NET Core with Hot Chocolate

This guide shows how to build a **GraphQL API** in .NET Core using the
**Hot Chocolate** library by ChilliCream.

------------------------------------------------------------------------

## 🧱 What We'll Build

A simple API with **Authors** and **Books**: - **Queries**: list
authors/books, with filtering, sorting, pagination - **Mutations**: add
a book - In-memory EF Core database (easy to run)

------------------------------------------------------------------------

## 1) Create the Project & Add Packages

``` bash
dotnet new web -n GraphQLDemo
cd GraphQLDemo

# Add GraphQL + EF Core packages
dotnet add package HotChocolate.AspNetCore
dotnet add package HotChocolate.Data
dotnet add package HotChocolate.Data.EntityFramework
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

------------------------------------------------------------------------

## 2) Add Models & DbContext

**Models.cs**

``` csharp
using Microsoft.EntityFrameworkCore;

public class Author
{
    public int Id { get; set; }
    public string Name { get; set; } = default!;
    public ICollection<Book> Books { get; set; } = new List<Book>();
}

public class Book
{
    public int Id { get; set; }
    public string Title { get; set; } = default!;
    public int AuthorId { get; set; }
    public Author Author { get; set; } = default!;
}

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> opts) : base(opts) { }
    public DbSet<Author> Authors => Set<Author>();
    public DbSet<Book> Books => Set<Book>();
}
```

------------------------------------------------------------------------

## 3) Define GraphQL Types (Query & Mutation)

**GraphQL.cs**

``` csharp
using HotChocolate;
using HotChocolate.Data;
using HotChocolate.Types;
using Microsoft.EntityFrameworkCore;

public class Query
{
    [UseDbContext(typeof(AppDbContext))]
    [UsePaging]
    [UseFiltering]
    [UseSorting]
    public IQueryable<Author> GetAuthors([ScopedService] AppDbContext db)
        => db.Authors;

    [UseDbContext(typeof(AppDbContext))]
    [UsePaging]
    [UseFiltering]
    [UseSorting]
    public IQueryable<Book> GetBooks([ScopedService] AppDbContext db)
        => db.Books;
}

public record AddBookInput(string Title, int AuthorId);
public record AddBookPayload(Book Book);

public class Mutation
{
    [UseDbContext(typeof(AppDbContext))]
    public async Task<AddBookPayload> AddBookAsync(
        AddBookInput input,
        [ScopedService] AppDbContext db,
        CancellationToken ct)
    {
        var book = new Book { Title = input.Title, AuthorId = input.AuthorId };
        db.Books.Add(book);
        await db.SaveChangesAsync(ct);
        await db.Entry(book).Reference(b => b.Author).LoadAsync(ct);
        return new AddBookPayload(book);
    }
}

public class AuthorType : ObjectType<Author>
{
    protected override void Configure(IObjectTypeDescriptor<Author> d)
    {
        d.Field(a => a.Id);
        d.Field(a => a.Name);
        d.Field(a => a.Books)
         .UsePaging()
         .UseFiltering()
         .UseSorting();
    }
}

public class BookType : ObjectType<Book>
{
    protected override void Configure(IObjectTypeDescriptor<Book> d)
    {
        d.Field(b => b.Id);
        d.Field(b => b.Title);
        d.Field(b => b.Author)
         .ResolveWith<Resolvers>(r => r.GetAuthor(default!, default!))
         .UseProjection();
    }

    private sealed class Resolvers
    {
        [UseDbContext(typeof(AppDbContext))]
        public Author GetAuthor([Parent] Book book, [ScopedService] AppDbContext db)
            => db.Authors.First(a => a.Id == book.AuthorId);
    }
}
```

------------------------------------------------------------------------

## 4) Wire Everything in Program.cs

**Program.cs**

``` csharp
using HotChocolate.Execution.Configuration;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddPooledDbContextFactory<AppDbContext>(opt =>
    opt.UseInMemoryDatabase("AppDb"));

builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>()
    .AddType<AuthorType>()
    .AddType<BookType>()
    .AddProjections()
    .AddFiltering()
    .AddSorting()
    .AddInMemorySubscriptions();

var app = builder.Build();

using (var scope = app.Services.CreateScope())
{
    var factory = scope.ServiceProvider.GetRequiredService<IDbContextFactory<AppDbContext>>();
    using var db = factory.CreateDbContext();

    if (!db.Authors.Any())
    {
        var a1 = new Author { Name = "Ursula K. Le Guin" };
        var a2 = new Author { Name = "Douglas Adams" };
        db.Authors.AddRange(a1, a2);
        db.Books.AddRange(
            new Book { Title = "The Left Hand of Darkness", Author = a1 },
            new Book { Title = "The Dispossessed", Author = a1 },
            new Book { Title = "The Hitchhiker's Guide to the Galaxy", Author = a2 }
        );
        db.SaveChanges();
    }
}

app.MapGraphQL("/graphql");

app.Run();
```

Run the app:

``` bash
dotnet run
```

👉 Open **http://localhost:5000/graphql** → Banana Cake Pop IDE.

------------------------------------------------------------------------

## 5) Example Queries

### List Books with Author

``` graphql
query {
  books(first: 10, where: { title: { contains: "the" } }, order: { title: ASC }) {
    nodes {
      id
      title
      author { id name }
    }
  }
}
```

### List Authors with Books

``` graphql
query {
  authors(first: 5) {
    nodes {
      id
      name
      books(first: 2, order: { title: ASC }) {
        nodes { id title }
      }
    }
  }
}
```

### Add a Book

``` graphql
mutation {
  addBook(input: { title: "A Wizard of Earthsea", authorId: 1 }) {
    book {
      id
      title
      author { id name }
    }
  }
}
```

------------------------------------------------------------------------

## ✅ Why Hot Chocolate?

-   Minimal boilerplate.\
-   Built-in **filtering, sorting, paging**.\
-   Automatic **projection** for EF Core.\
-   Easy to extend with **auth, subscriptions, validation**.

------------------------------------------------------------------------

## 🔧 Next Steps

-   Add **auth** (`[Authorize]`)\
-   Add **validations** with FluentValidation\
-   Add **subscriptions** for real-time updates\
-   Swap **InMemory DB** with SQL Server/Postgres

