# RegMan

University Registration & Management System (students, instructors, admins) built as a real full‑stack product: **React/Vite SPA** + **ASP.NET Core Web API** + **SQL Server**.

- Website: https://regman.app
- Backend (ASP.NET Core `net8.0`): https://github.com/RegManApp/RegMan.Backend

# RegMan Documentation (Source of Truth)

This repository is the **main documentation hub** for RegMan.

RegMan is a university registration & academic management system built as a full-stack application:

- Frontend: React (Vite)
- Backend: ASP.NET Core Web API (.NET 8)
- Database: SQL Server (EF Core)

Quick links:

- Live website: https://regman.app
- Frontend repo: https://github.com/RegManApp/RegMan.Frontend
- Backend repo: https://github.com/RegManApp/RegMan.Backend

## Documentation Map

- Architecture overview: [architecture.md](architecture.md)
- API reference (grouped by domain): [api.md](api.md)
- Frontend deep-dive: [frontend.md](frontend.md)
- Backend deep-dive: [backend.md](backend.md)
- Repo presentation (GitHub/recruiter polish): [repo-presentation.md](repo-presentation.md)

## System Architecture (High-Level)

```text
React SPA
  ↓ HTTP (Axios) / WS (SignalR)
ASP.NET Core API (Controllers)
  ↓
Business Layer (Services)
  ↓
DAL (Repositories + Unit of Work)
  ↓
SQL Server
```

RegMan keeps critical rules server-side (registration timelines, seat capacity, approvals, grading → transcript/GPA sync) to keep data consistent across clients.

## Run The Full System Locally

This section is the canonical setup guide. Other repos link here (no duplication).

### 1) Clone repositories

```bash
git clone https://github.com/RegManApp/RegMan.Backend
git clone https://github.com/RegManApp/RegMan.Frontend
```

### 2) Prerequisites

- Git
- .NET SDK 8
- Node.js 18+ (recommended) + npm
- SQL Server (LocalDB or full SQL Server)

### 3) Backend configuration (environment variables)

The API requires these environment variables at startup:

- `ConnectionStrings__DefaultConnection` (SQL Server connection string)
- `Jwt__Key` (JWT signing key, **>= 32 characters**). The API will fail fast if missing/weak.

Optional (enables Google Calendar integration; otherwise it is disabled):

- `GOOGLE_CLIENT_ID`
- `GOOGLE_CLIENT_SECRET`
- `GOOGLE_REDIRECT_URI`

Example connection string for LocalDB (adjust as needed):

```text
Server=(localdb)\MSSQLLocalDB;Database=RegMan;Trusted_Connection=True;MultipleActiveResultSets=true;TrustServerCertificate=True
```

### 4) Run the backend

```bash
cd RegMan.Backend
dotnet restore

# Optional: apply migrations via EF CLI (the API also runs MigrateAsync() at startup)
dotnet tool install --global dotnet-ef
dotnet ef database update --project RegMan.Backend.DAL/RegMan.Backend.DAL.csproj --startup-project RegMan.Backend.API/RegMan.Backend.API.csproj

dotnet run --project RegMan.Backend.API/RegMan.Backend.API.csproj
```

Useful URLs (defaults from launch settings):

- Swagger UI: `https://localhost:7025/swagger` or `http://localhost:5236/swagger`
- API base: `http://localhost:5236/api`
- SignalR hubs: `/hubs/chat`, `/hubs/notifications`

### 5) Frontend configuration (environment variables)

The frontend uses Vite env vars:

- `VITE_API_BASE_URL` (**required**) — should include the `/api` suffix, e.g. `http://localhost:5236/api`
- `VITE_APP_NAME` (optional)

In the frontend repo:

```bash
cd RegMan.Frontend
copy .env.example .env.local
```

Then edit `.env.local` and set `VITE_API_BASE_URL` to match your backend URL.

### 6) Run the frontend

```bash
cd RegMan.Frontend
npm install
npm run dev
```

### Run order

1. Start the backend first
2. Start the frontend after the backend is running

## Need deeper details?

- Backend internals (auth/migrations/middleware): [backend.md](backend.md)
- Frontend internals (routing/auth/axios/i18n): [frontend.md](frontend.md)
- Endpoint reference (by domain): [api.md](api.md)
