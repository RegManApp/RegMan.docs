# Architecture

This document describes the architecture of RegMan at a level intended for technical reviewers.

## Layering

RegMan is split into three backend layers plus a separate frontend:

- **API** ([RegMan.Backend/RegMan.Backend.API](https://github.com/RegManApp/RegMan.Backend/tree/main/RegMan.Backend.API))
  - ASP.NET Core controllers, auth, middleware, Swagger, SignalR hubs
  - Responsibility: HTTP contract + authorization + orchestration
- **Business Layer** ([RegMan.Backend/RegMan.Backend.BusinessLayer](https://github.com/RegManApp/RegMan.Backend/tree/main/RegMan.Backend.BusinessLayer))
  - Services implementing business rules
  - Responsibility: domain rules and use-cases (registration, enrollment, GPA rules, office hours workflows)
- **DAL** ([RegMan.Backend/RegMan.Backend.DAL](https://github.com/RegManApp/RegMan.Backend/tree/main/RegMan.Backend.DAL))
  - EF Core entities, repositories, Unit of Work
  - Responsibility: persistence abstractions and queries
- **Frontend** ([RegMan.Frontend](https://github.com/RegManApp/RegMan.Frontend))
  - React SPA using Axios + Context API + i18n + SignalR client

## Communication

- **HTTP**: React uses a centralized Axios instance (`VITE_API_BASE_URL`) and attaches JWT tokens to requests.
- **Realtime**: SignalR hubs expose `/hubs/chat` and `/hubs/notifications`.
- **API response envelope**: the backend consistently returns `ApiResponse<T>` with `{ success, statusCode, message, data, errors }`.

## Backend “source of truth”

RegMan intentionally keeps critical rules on the server:

- Registration/withdraw windows are enforced before cart/enrollment actions.
- Seat counts and section capacity are enforced in server-side services.
- Grade edits trigger transcript/GPA synchronization in the backend.

This reduces the risk of clients bypassing constraints and keeps data consistent.

## Patterns in use

- **Service Layer**: controllers call service interfaces rather than embedding business logic.
- **Repository + Unit of Work**: writes are coordinated through `IUnitOfWork` and repositories.
- **DTO pattern**: the API surface is DTO-driven, keeping entity models decoupled from HTTP contracts.
- **Dependency Injection**: service registration is centralized (`AddBusinessServices`, `AddDataBaseLayer`).
- **Middleware**: cross-cutting concerns (exception handling) are centralized.

## Suggested next improvements (optional)

If you plan to keep evolving RegMan as a long-lived product:

- Add OpenAPI annotations and generate an API client (typed) for the frontend.
- Add CI checks (build backend + build frontend) and basic smoke tests.
- Add a small integration test project around critical flows (auth + enrollment timeline gates).
