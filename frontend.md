# Frontend

RegMan frontend is a React 18 SPA built with Vite.

## Quickstart (Clone / Install / Run / Env Vars)

This repo does not duplicate setup instructions.

- Clone + install + run: [README.md](README.md)
- Required env vars: [README.md](README.md)

## Folder structure

Key directories under [RegMan.Frontend/src](https://github.com/RegManApp/RegMan.Frontend/tree/main/src):

- `api/` — API clients and Axios instance
- `components/` — reusable UI and shared components
- `contexts/` — app-wide state (auth, theme, unread counters)
- `hooks/` — shared hooks (including direction/RTL helpers)
- `i18n/` — localization setup + locales
- `pages/` — route-level pages
- `utils/` — constants, formatting, and helpers

## Pages vs components

- **Pages** own data fetching and route concerns.
- **Components** are presentation- and composition-focused.

This keeps UI reusable and makes data flow obvious during reviews.

## Auth flow

- Auth state is managed via Context: [src/contexts/AuthContext.jsx](https://github.com/RegManApp/RegMan.Frontend/blob/main/src/contexts/AuthContext.jsx)
- Tokens are stored in either `localStorage` (remember me) or `sessionStorage` (session-only).
- Token expiry is checked client-side (JWT `exp`), and the user is logged out on 401s.

## Role-based routing

- Route composition lives in: [src/App.jsx](https://github.com/RegManApp/RegMan.Frontend/blob/main/src/App.jsx)
- Route guards:
  - `ProtectedRoute` ensures authentication
  - `RoleGuard` enforces role access to routes

See: [src/components/auth](https://github.com/RegManApp/RegMan.Frontend/tree/main/src/components/auth)

## Axios instance & error handling

All HTTP calls use the centralized Axios instance:

- https://github.com/RegManApp/RegMan.Frontend/blob/main/src/api/axiosInstance.js

Key behaviors:

- Sets `baseURL` from `VITE_API_BASE_URL`.
- Automatically attaches `Authorization: Bearer <token>`.
- Unwraps the backend `ApiResponse<T>` envelope on success.
- Normalizes error UX via a response interceptor:
  - 401 clears tokens and redirects to `/login`
  - 400 shows validation errors from `errors` field

## i18n and RTL

- i18n setup: [src/i18n/index.js](https://github.com/RegManApp/RegMan.Frontend/blob/main/src/i18n/index.js)
- Language stored under `regman.language`.
- Applies RTL/LTR globally:
  - `document.documentElement.lang = 'en' | 'ar'`
  - `document.documentElement.dir = 'ltr' | 'rtl'`

UI components can additionally use direction helpers for layout nuances.

## SignalR integration

Realtime capabilities use SignalR:

- Shared hub client utilities under `src/api/` (SignalR client modules)
- Chat and notifications subscribe to server events and update local state.

Back-end hub endpoints:

- `/hubs/chat`
- `/hubs/notifications`

## Post-deploy Calendar Smoke Tests

Use this checklist to verify the newly added calendar features through the UI after deployment.

Prereqs

- Backend deployed with DB migrations applied (including `20260107220559_AddCalendarEnhancements`).
- Frontend deployed with `VITE_API_BASE_URL` pointing to the backend API base URL (must include `/api`).
- If testing Google Calendar: valid production `Google.ClientId/ClientSecret/RedirectUri` configured.

1. Calendar view (`/api/calendar/view`)

- Login and open Calendar.
- Verify the month loads without errors and displays events.
- Switch month back/forward and verify events reload.

2. Conflict detection (type + severity)

- Find or create a scenario that should produce a conflict.
- Verify the calendar day cell shows a conflict count badge.
- Click the day and verify the conflict list displays the conflict message/type and severity.

3. Calendar preferences persist & reload

- Go to Settings → Calendar Preferences.
- Change **Week starts on** and **Hide weekends**, then Save.
- Refresh the page (or log out/in).
- Verify Settings shows the saved values and Calendar reflects them (headers/grid).

4. Reminder rules (save → apply → trigger)

- Go to Settings → Reminder Rules.
- Adjust a rule (e.g., set “Minutes before” to `1`) and Save.
- Ensure you have an upcoming matching event within the next few minutes.
- Wait and confirm an in-app notification is raised at the expected time.

5. Google Calendar connect / disconnect / status

- Go to Settings → Google Calendar Integration.
- Connect and complete the Google consent flow; verify status shows connected + email.
- Disconnect; verify status becomes disconnected.
- Optionally reconnect to ensure the lifecycle works repeatedly.

After these manual steps pass, the features can be considered production-ready.
