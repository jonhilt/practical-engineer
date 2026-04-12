# Example: a complete `GetUserList` slice

A concrete end-to-end vertical slice for a "list all users" use case. Demonstrates how a single feature folder owns its component, request/response types, handler, and data access — all colocated.

## Folder layout

```
Features/Users/GetUserList/
├── UserList.razor
├── GetUserListQuery.cs
└── UserListItem.cs
```

Three files: the Blazor page, the query + handler (colocated in one file — common VSA idiom), and the per-slice view model. No separate `Services/`, `Repositories/`, or `Models/` folders. Everything this use case needs is here.

## `UserListItem.cs` — per-slice view model

```csharp
namespace MyApp.Features.Users.GetUserList;

public record UserListItem(Guid Id, string DisplayName, string Email, DateOnly JoinedOn);
```

View models are per-slice. Don't try to share a `UserDto` across `GetUserList`, `UserDetails`, and `EditUser` — each use case projects exactly the fields it needs, and they evolve independently.

## `GetUserListQuery.cs` — request + handler

```csharp
using MediatR;
using Microsoft.EntityFrameworkCore;
using MyApp.Infrastructure.Database;

namespace MyApp.Features.Users.GetUserList;

public record GetUserListQuery(string? SearchTerm = null) : IRequest<IReadOnlyList<UserListItem>>;

public class GetUserListHandler(AppDbContext db)
    : IRequestHandler<GetUserListQuery, IReadOnlyList<UserListItem>>
{
    public async Task<IReadOnlyList<UserListItem>> Handle(GetUserListQuery request, CancellationToken ct)
    {
        var query = db.Users.AsNoTracking();

        if (!string.IsNullOrWhiteSpace(request.SearchTerm))
        {
            var term = request.SearchTerm.Trim();
            query = query.Where(u => u.DisplayName.Contains(term) || u.Email.Contains(term));
        }

        return await query
            .OrderBy(u => u.DisplayName)
            .Select(u => new UserListItem(u.Id, u.DisplayName, u.Email, u.JoinedOn))
            .ToListAsync(ct);
    }
}
```

Key points:

- Query and handler live in one file. They change together; splitting them just adds navigation cost.
- Handler injects `AppDbContext` directly. No `IUserRepository` in between — the repository *is* `DbContext`.
- Projects straight to `UserListItem` with `.Select(...)`. EF Core turns this into a `SELECT` that fetches only the columns the view model needs — no over-fetching, no manual mapping.
- `AsNoTracking()` because this is a read-only query. Don't pay for change tracking you won't use.
- C# 12 primary-constructor DI keeps the class tight.

## `UserList.razor` — the Blazor page

```csharp
@page "/users"
@using MediatR
@using MyApp.Features.Users.GetUserList
@inject IMediator Mediator

<PageTitle>Users</PageTitle>

<h1>Users</h1>

<input @bind="_searchTerm" @bind:event="oninput" @bind:after="Search" placeholder="Search..." />

@if (_users is null)
{
    <p>Loading…</p>
}
else if (_users.Count == 0)
{
    <p>No users found.</p>
}
else
{
    <table class="table">
        <thead>
            <tr><th>Name</th><th>Email</th><th>Joined</th></tr>
        </thead>
        <tbody>
            @foreach (var user in _users)
            {
                <tr>
                    <td>@user.DisplayName</td>
                    <td>@user.Email</td>
                    <td>@user.JoinedOn.ToString("yyyy-MM-dd")</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {
    private string? _searchTerm;
    private IReadOnlyList<UserListItem>? _users;

    protected override Task OnInitializedAsync() => Search();

    private async Task Search()
    {
        _users = await Mediator.Send(new GetUserListQuery(_searchTerm));
    }
}
```

Key points:

- `@page "/users"` lives *in the slice folder*, not in a central `Pages/` directory. The route and the handler ship together.
- The component is thin: `@bind` for interaction state, `IMediator.Send` to dispatch, a table to render. No LINQ, no EF, no business rules.
- `_searchTerm` and `_users` are interaction state — they belong in the component. The filter *logic* lives in the handler.
- The component only references `GetUserListQuery` and `UserListItem`, both defined in its own slice. It has no reference to the `User` entity, `AppDbContext`, or any other slice.

## What's NOT in this slice

Things that would pull logic out of the slice and back into layered architecture — don't add them:

- `IUserService` / `UserService` — the handler *is* the service.
- `IUserRepository` / `UserRepository` — `DbContext` is the repository.
- A shared `UserDto` in `Shared/Models/` — projections stay per-slice.
- An AutoMapper profile — `.Select(u => new UserListItem(...))` is clearer and hides nothing.
- A `BaseHandler<TRequest, TResponse>` — handlers are small; inheritance hierarchies cost more than they save.

Each of these is a step back toward layered architecture. Add them only when two or more slices clearly, provably need the same thing.

## Adding a related use case

If a new requirement comes in — say "view user details" — do *not* extend `GetUserList`. Create a sibling folder:

```
Features/Users/
├── GetUserList/
│   ├── UserList.razor
│   ├── GetUserListQuery.cs
│   └── UserListItem.cs
└── UserDetails/
    ├── UserDetails.razor
    ├── GetUserDetailsQuery.cs
    └── UserDetailsViewModel.cs
```

Both slices query the same `Users` table via `AppDbContext`, but they have independent queries, handlers, and view models. `UserDetailsViewModel` can include fields `UserListItem` doesn't, and vice versa, without either slice affecting the other.
