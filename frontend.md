# Frontend

RegMan frontend is a React 18 SPA built with Vite.

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

- [frontend/RegMan.Frontend/src/api/axiosInstance.js](../frontend/RegMan.Frontend/src/api/axiosInstance.js)
- [src/api/axiosInstance.js](https://github.com/RegManApp/RegMan.Frontend/blob/main/src/api/axiosInstance.js)

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
