# Repo Presentation (GitHub + Showcase)

This page is for “production credibility” polish: how to present RegMan on GitHub, in releases, and on LinkedIn without duplicating content.

## Quickstart (Clone / Install / Run / Env Vars)

Canonical setup instructions live in the docs entry point:

- Clone + dependencies + run order: [README.md](README.md)
- Required environment variables: [README.md](README.md)

## GitHub repository sidebar (copy/paste)

### About

> University Registration & Management System (React/Vite + ASP.NET Core + SQL Server). Role-based workflows for students/instructors/admins: enrollment, cart/checkout, GPA/transcripts, office hours, chat, academic calendar, and Google Calendar integration.

### Website

- `https://regman.app`

### Topics

Suggested topics (pick ~8–15):

- `aspnetcore`
- `dotnet`
- `dotnet8`
- `ef-core`
- `sqlserver`
- `jwt`
- `identity`
- `signalr`
- `react`
- `vite`
- `axios`
- `i18next`
- `rtl`
- `oauth2`
- `google-calendar`

## Releases and versioning

RegMan can look more “real” if you publish tagged releases, even for small iterations.

### Recommended strategy

- Use **SemVer**: `MAJOR.MINOR.PATCH`
  - `MAJOR`: breaking API/UI changes
  - `MINOR`: new features (new pages/endpoints) without breaking existing behavior
  - `PATCH`: bugfixes and internal refactors

If you don’t want to commit to long-term SemVer promises, keep it simple:

- Start at `0.1.0` and increment minor/patch as features stabilize.

### What to include in a release

- Summary: 1–2 lines
- Highlights: 3–7 bullets (features users notice)
- Technical notes: migrations, config changes, new env vars
- Verification: what you ran (e.g., `dotnet build`, `npm run build`)

### Where to mention migrations

- Release notes: “DB migration required” + name
- Docs: keep the detailed EF guidance in [backend.md](backend.md)

## Packages (why it may be empty)

It’s normal for GitHub “Packages” to be empty.

Use Packages only if you publish:

- A reusable NuGet package (shared library) or
- A Docker image (API container)

If RegMan is a single application repo, packages being empty is not a negative signal.

## Where to showcase what

Avoid duplicating long text in multiple places. Link instead.

### GitHub README (top-level)

Keep it “decision-maker friendly”:

- What it is (one paragraph)
- Tech stack
- Architecture diagram
- Role-based feature list
- Local run instructions (short)
- Links to deeper docs (API, backend, frontend)

### `/docs/*`

Put reviewer-grade detail here:

- API endpoint reference: [api.md](api.md)
- Architecture rationale: [architecture.md](architecture.md)
- Backend pipeline/auth/migrations: [backend.md](backend.md)
- Frontend routing/auth/i18n/realtime: [frontend.md](frontend.md)

### Website

Keep it product-facing:

- Screenshots / demo flows
- Minimal technical detail
- A short “Built with …” stack footer is fine

### LinkedIn (post or project)

Write for impact and outcomes, then link to GitHub.

#### LinkedIn description (short)

> RegMan is a full-stack university registration system built with React (Vite) and ASP.NET Core (.NET 8). It supports role-based workflows (student/instructor/admin), enrollment rules enforced server-side, GPA/transcript management, realtime chat/notifications with SignalR, and Google Calendar OAuth integration.

#### LinkedIn bullet highlights

- Built a layered backend (Controllers → Services → EF Core DAL) with consistent API envelopes and centralized exception handling
- Implemented role-based authorization (Admin/Student/Instructor) with JWT + Identity
- Delivered end-to-end registration workflows: cart/checkout/enroll, drop/withdraw with academic timeline gating
- Added student support features: GPA + transcript, office hours booking, and realtime chat/notifications (SignalR)
- Internationalized the UI with EN/AR switching and RTL support

#### What _not_ to paste into LinkedIn

- Full endpoint lists, DTO schemas, or config blocks → link to [api.md](api.md)

## Quick links

- API reference: [api.md](api.md)
- Architecture: [architecture.md](architecture.md)
- Backend details: [backend.md](backend.md)
- Frontend details: [frontend.md](frontend.md)
