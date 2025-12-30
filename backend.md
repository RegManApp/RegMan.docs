# Backend

RegMan backend is an ASP.NET Core `net8.0` Web API using EF Core + SQL Server.

## Startup pipeline

The application is configured in [RegMan.Backend.API/Program.cs](https://github.com/RegManApp/RegMan.Backend/blob/main/RegMan.Backend.API/Program.cs):

- CORS policy `AllowRegman` for local + deployed origins
- SignalR hubs for realtime features
- Database layer + business services registered via extensions
- ASP.NET Identity with lockout and password rules
- JWT authentication (including SignalR token handoff via `access_token` query string for hub connections)
- Authorization policies and role checks
- Swagger with JWT bearer security definition
- Middleware: `GlobalExceptionMiddleware` wraps errors in a consistent JSON envelope
- Startup seeding and `Database.MigrateAsync()` for schema sync

## Authentication & authorization

- Identity is configured with DB-backed lockout policies.
- JWT validation enforces issuer/audience/signing key.
- Roles: `Admin`, `Student`, `Instructor`.
- Controllers use `[Authorize]` + `[Authorize(Roles = ...)]` for endpoint-level enforcement.

Important behaviors:

- Public registration is **forced to Student** to prevent privilege escalation.
- Auth tokens:
  - access token for API calls
  - refresh token stored server-side (hashed)

## Global exception handling

[RegMan.Backend.API/Middleware/GlobalExceptionMiddleware.cs](https://github.com/RegManApp/RegMan.Backend/blob/main/RegMan.Backend.API/Middleware/GlobalExceptionMiddleware.cs)

- Converts known application exceptions (`AppException`) into structured `ApiResponse` failures.
- Normalizes EF constraint conflicts (`DbUpdateException`) to HTTP 409.
- Avoids leaking sensitive exception details; includes a trace id in responses.

## Persistence and migrations

- EF Core migrations live under: [RegMan.Backend.DAL/Migrations](https://github.com/RegManApp/RegMan.Backend/tree/main/RegMan.Backend.DAL/Migrations)
- API applies migrations at startup (`Database.MigrateAsync()`), reducing runtime schema drift.

Common commands:

```bash
# from the RegMan.Backend repo root

dotnet ef migrations add <Name> \
  --project RegMan.Backend.DAL/RegMan.Backend.DAL.csproj \
  --startup-project RegMan.Backend.API/RegMan.Backend.API.csproj

dotnet ef database update \
  --project RegMan.Backend.DAL/RegMan.Backend.DAL.csproj \
  --startup-project RegMan.Backend.API/RegMan.Backend.API.csproj
```

## Seeding

The API seeds on startup:

- Roles (Admin/Student/Instructor)
- Admin user
- Default academic plan
- Default academic calendar settings row

This makes local setup reproducible.

## Business rules enforcement

RegMan deliberately enforces critical workflows server-side:

- Cart/enrollment actions are blocked when registration is outside the configured window.
- Drop/withdraw is only allowed in registration or withdraw windows.
- Grade edits can trigger transcript creation/removal and GPA recalculation.

This keeps the system consistent even if multiple clients or automation interact with the API.
