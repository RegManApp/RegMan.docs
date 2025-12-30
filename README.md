# RegMan

University Registration & Management System (students, instructors, admins) built as a real full‑stack product: **React/Vite SPA** + **ASP.NET Core Web API** + **SQL Server**.

- Website: https://regman.app
- Backend (ASP.NET Core `net8.0`): https://github.com/RegManApp/RegMan.Backend
- Frontend (React/Vite): https://github.com/RegManApp/RegMan.Frontend
- API docs: [docs/api.md](docs/api.md)
- Repo presentation (GitHub + LinkedIn): [docs/repo-presentation.md](docs/repo-presentation.md)

## Project Overview

RegMan models the end-to-end lifecycle of university registration:

- **Students** search courses, add sections to a cart, validate checkout, enroll, drop/withdraw within allowed academic windows, view GPA and transcript, book office hours, chat, and sync events.
- **Instructors** manage office hours, view schedules, and communicate with students.
- **Admins** manage catalog data, academic calendar configuration, approvals/workflows, and operational oversight.

Core problems RegMan addresses:

- Centralizing registration rules (timeline gates, seat availability, approvals) **server-side**.
- Clear separation of concerns so business rules live outside controllers and are testable.
- “Product-grade” UX needs: consistent API envelope, validation responses, global error handling, realtime chat/notifications.

## Tech Stack

### Backend

- ASP.NET Core (`net8.0`)
- Entity Framework Core (SQL Server)
- ASP.NET Identity
- JWT Authentication + role-based authorization (Admin/Student/Instructor)
- SignalR (realtime chat + notifications)
- Google Calendar OAuth integration (connect + token storage)

### Frontend

- React 18
- Vite
- Axios (centralized instance + interceptors)
- Context API (auth/theme/realtime state)
- i18n with `i18next` / `react-i18next` (EN/AR)
- RTL support via `document.documentElement.dir` switching
- SignalR client (`@microsoft/signalr`)

## Architecture Overview

High-level request flow:

```text
React SPA
  ↓ HTTP (Axios) / WS (SignalR)
ASP.NET Core API (Controllers)
  ↓ calls
Business Layer (Services)
  ↓ via interfaces
DAL (Unit of Work + Repositories)
  ↓
SQL Server
```

Why this structure:

- **Controllers stay thin**: request/response mapping, authorization, and orchestration only.
- **Business Layer is the source of truth**: registration timelines, enrollment rules, seat checks, grade/GPA consistency.
- **DAL isolates persistence**: repositories expose queryable access patterns; Unit of Work groups writes.

Patterns used (as implemented in code):

- **Service Layer** (e.g., `CourseService`, `EnrollmentService`, `OfficeHoursService`)
- **Repository + Unit of Work** (`IUnitOfWork` + repository abstractions)
- **DTO pattern** for API contracts and view models
- **Dependency Injection** via `AddBusinessServices()` and `AddDataBaseLayer()`
- **Middleware pattern** for global exception handling + consistent error envelope

More detail:

- [docs/architecture.md](docs/architecture.md)
- [docs/backend.md](docs/backend.md)
- [docs/frontend.md](docs/frontend.md)
- [docs/repo-presentation.md](docs/repo-presentation.md)

## Features (by role)

### Student

- Course browsing + registration flow (**Cart → Checkout → Enroll**)
- Drop & Withdraw (enforced by academic timeline gates)
- GPA & Transcript
- Academic Calendar (timeline + user events)
- Chat (realtime)
- Office Hours booking
- Google Calendar connect (OAuth) and sync-ready event surface

### Instructor

- Office hours management (create/batch/update/delete + booking workflow)
- Student communication (chat)
- Schedule visibility (teaching events)

### Admin

- Course & category management
- Academic calendar setup (registration/withdraw windows)
- Enrollment approvals / declines
- Withdraw request review and actioning

## Repository Layout

This organization is split across multiple repositories:

- Backend: https://github.com/RegManApp/RegMan.Backend
- Frontend: https://github.com/RegManApp/RegMan.Frontend
- Documentation (hub): https://github.com/RegManApp/RegMan.docs
- Organization / meta: https://github.com/RegManApp/.github

## Local Development

### Prerequisites

- .NET SDK 8
- Node.js 18+ (recommended)
- SQL Server (LocalDB or full SQL Server)

### Backend

The API expects a SQL Server connection string named `DefaultConnection`.

Environment variables (Development):

- `ConnectionStrings__DefaultConnection` (SQL Server)
- `Jwt__Key` (>= 32 chars)
- Optional (Google Calendar integration):
  - `GOOGLE_CLIENT_ID`
  - `GOOGLE_CLIENT_SECRET`
  - `GOOGLE_REDIRECT_URI`

Run:

```bash
# from the RegMan.Backend repo root
cd RegMan.Backend.API

dotnet run
```

Notes:

- The API runs EF migrations at startup (`Database.MigrateAsync()`), which keeps schema in sync.
- Swagger UI is enabled: `GET /swagger`.

### Frontend

Set API base URL:

- `VITE_API_BASE_URL` (e.g. `http://localhost:5240`)

Run:

```bash
cd RegMan.Frontend
npm install
npm run dev
```

## API Documentation

- Human-readable: [docs/api.md](docs/api.md)
- Interactive: Swagger (`/swagger`) when running the API locally.

## Frontend Documentation

See [docs/frontend.md](docs/frontend.md) for:

- Folder structure (pages vs components)
- Auth/session persistence and role-based routing
- Axios instance + error handling conventions
- i18n language switching and RTL behavior
- SignalR integration points

## Backend Documentation

See [docs/backend.md](docs/backend.md) for:

- Startup pipeline (`Program.cs`)
- Authentication & authorization
- Global exception handling and response envelope
- Migrations + seeding strategy
- Why business rules are enforced server-side
