---
name: blazor
description: Vertical-slice architecture rules for .NET 8+ Blazor Server apps. Use when creating or editing .razor/.razor.cs files, Features/ folders, MediatR handlers, or when the user asks about Blazor project structure, feature folders, or where a new use case should live.
---

# Blazor Server + Vertical Slice Architecture

Implement and evolve Blazor Server apps using vertical slices (feature-first), not layered architecture. Organize code by use case so each slice owns its UI, application logic, and data access. High cohesion inside a slice, low coupling between slices.

Target stack: modern .NET 8+ Blazor Server / Blazor Web App (Server). Prefer feature folders over Controllers/Services/Repositories layers. Avoid "god" services or generic repositories unless there is a proven cross-cutting need.

## Project structure

- Top-level `Features` folder (or equivalent).
- Under `Features`, one folder per feature or use case:
    - `Features/Users/GetUserList`
    - `Features/Users/UserDetails`
    - `Features/Events/ViewCreatedEvent`
- Each feature folder contains everything specific to that use case:
    - Blazor components / pages (`.razor`, partial `.razor.cs`).
    - Request / command / query types and response / view models.
    - Handler(s) — MediatR request handlers or an equivalent application-service class — implementing the business logic.
    - Slice-specific helpers (mappers, validators) that are not truly cross-cutting.

For a complete end-to-end example (component + query + handler + view model), see [example-slice.md](example-slice.md).

## UI rules

- Place the main page/component for a use case in its feature folder, not in a global `Pages`/`Components` folder. Co-locate `@page` directives and route attributes there.
- Treat UI as thin: binding, navigation, and dispatching requests — not business rules or complex data access.
- UI calls into a handler (via `IMediator` or a slice-local application service), passing well-defined request objects and rendering returned view models.
- Keep interaction state (current page, selected filters, etc.) in the component. Push business rules and data retrieval into the slice's handler.
- Use DI for `IMediator` or the slice's application service. Do not inject generic "service" types from a shared service layer unless they are truly cross-cutting.
- Prefer SSR pages/components with interactive render modes where applicable.

## Handlers and domain logic

- Each use case gets its own request type and handler (e.g. `GetUserListQuery` + `GetUserListHandler`).
- Handlers orchestrate the use case. They can inject `DbContext` or small domain services directly instead of going through a generic service layer.
- Request/response types and view models are per-slice. Do not create "one DTO to rule them all" unless it is truly shared and stable.

## Data access

- Straightforward EF Core in handlers (or equivalent), projecting directly to view models where appropriate.
- Avoid generic repositories or service layers by default. Introduce shared abstractions only when multiple slices clearly need them and the abstraction is well-understood.

## Shared code

- Narrowly scoped: `Infrastructure/Database/AppDbContext`, `Shared` for layout, primitive UI components, and simple utilities.
- Keep shared code small and generic. Do not move feature-specific logic into `Shared`.

## When implementing a new requirement

1. Restate the requirement in terms of one or more vertical slices.
2. Propose which feature folders and files to create or modify. Prefer a new slice over modifying unrelated slices.
3. If the codebase is currently layered, introduce vertical slices incrementally — wire up a new feature folder end-to-end rather than attempting a large refactor.
4. Ask targeted clarifying questions when ambiguous: where in the domain this feature belongs, whether it is a new slice or extends an existing one, and any domain rules that affect handler logic.

## Non-negotiables

- Do not introduce Clean, Onion, or Hexagonal architecture styles unless the user explicitly asks for them alongside vertical slices.
- Default to idiomatic C#, EF Core, and Blazor patterns for .NET 8+.

If any existing instructions in this repository conflict with this skill, follow the repository-specific instructions first and apply these vertical-slice rules as closely as possible.
