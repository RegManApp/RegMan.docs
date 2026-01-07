# API Documentation

This document describes the RegMan HTTP API surface as implemented in the ASP.NET Core controllers.

## Quickstart (Clone / Install / Run / Env Vars)

Canonical local setup instructions live in the docs entry point:

- [README.md](README.md)

## Conventions

### Base URL

Configured by environment.

- Local: e.g. `http://localhost:5236`

### Authentication

Most endpoints require JWT.

- Header: `Authorization: Bearer <accessToken>`
- Roles are enforced via `[Authorize(Roles = ...)]`.

### Response envelope

Most endpoints return an `ApiResponse<T>` envelope. The following JSON is the response example used throughout this document (unless an endpoint explicitly returns something else):

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Success",
  "data": {},
  "errors": null
}
```

Error example:

```json
{
  "success": false,
  "statusCode": 400,
  "message": "Validation failed",
  "data": null,
  "errors": {
    "field": ["error message"]
  }
}
```

### Swagger

When running locally, Swagger UI is available at:

- `GET /swagger`

---

## Auth

### `POST /api/auth/register`

- Auth: No
- Role: Public (server forces `Student`)
- Description: Create a student account.
- Body:

```json
{
  "fullName": "Jane Doe",
  "email": "student@demo.local",
  "address": "Cairo",
  "password": "StrongPassword1!"
}
```

- Response (example):

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Success",
  "data": "User registered successfully",
  "errors": null
}
```

### `POST /api/auth/login`

- Auth: No
- Role: Public
- Description: Login and receive access + refresh tokens.
- Body:

```json
{
  "email": "student@demo.local",
  "password": "StrongPassword1!"
}
```

- Response (example):

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Success",
  "data": {
    "accessToken": "<jwt>",
    "refreshToken": "<refresh>",
    "email": "student@demo.local",
    "fullName": "Jane Doe",
    "role": "Student",
    "userId": "<identity-user-id>",
    "instructorTitle": null
  },
  "errors": null
}
```

### `POST /api/auth/change-password`

- Auth: Yes
- Role: Any authenticated user
- Description: Change password using current password.
- Body:

```json
{
  "currentPassword": "OldPassword1!",
  "newPassword": "NewPassword1!",
  "confirmNewPassword": "NewPassword1!"
}
```

- Response: `ApiResponse<string>`

### `GET /api/auth/me`

- Auth: Yes
- Role: Any authenticated user
- Description: Returns the current user’s profile info. For students and instructors, includes role-specific `profile` payload.
- Response: `ApiResponse<object>`

### `POST /api/auth/refresh`

- Auth: No
- Role: Public
- Description: Exchange a refresh token for a new access token + refresh token.
- Body:

```json
{ "refreshToken": "<refresh>" }
```

- Response: `ApiResponse<LoginResponseDTO>`

### `POST /api/auth/logout`

- Auth: Yes
- Role: Any authenticated user
- Description: Revokes the provided refresh token.
- Body:

```json
{ "refreshToken": "<refresh>" }
```

- Response: `ApiResponse<string>`

---

## Courses

### `GET /api/course`

- Auth: Yes
- Role: Admin, Instructor, Student
- Description: Paginated course summaries.
- Query:
  - `page` (default 1)
  - `pageSize` (default 12)
  - optional filters: `search`, `courseName`, `creditHours`, `courseCode`, `courseCategoryId`
- Response: `ApiResponse<PaginatedResponse<ViewCourseSummaryDTO>>`

### `GET /api/course/{id}`

- Auth: Yes
- Role: Admin, Instructor, Student
- Description: Course details.
- Response: `ApiResponse<ViewCourseDetailsDTO>`

### `POST /api/course`

- Auth: Yes
- Role: Admin
- Description: Create a course.
- Body:

```json
{
  "courseName": "Data Structures",
  "creditHours": 3,
  "courseCode": "CS201",
  "courseCategoryId": 1,
  "description": "..."
}
```

- Response: `ApiResponse<ViewCourseDetailsDTO>`

### `PUT /api/course`

- Auth: Yes
- Role: Admin
- Description: Update a course.
- Body:

```json
{
  "courseId": 12,
  "courseName": "Data Structures",
  "creditHours": 3,
  "courseCode": "CS201",
  "courseCategoryId": 1,
  "description": "..."
}
```

- Response: `ApiResponse<ViewCourseDetailsDTO>`

### `DELETE /api/course/{id}`

- Auth: Yes
- Role: Admin
- Description: Delete a course.
- Response: `ApiResponse<string>`

### `GET /api/coursecategories`

- Auth: Yes
- Role: Any authenticated user
- Description: List course categories.
- Response: `ApiResponse<object>`
- Data shape (example):

```json
[{ "id": 1, "name": "ComputerScience", "value": "ComputerScience" }]
```

### `GET /api/coursecategories/{id}`

- Auth: Yes
- Role: Any authenticated user
- Description: Get category by id.
- Response: `ApiResponse<object>`
- Data shape (example):

```json
{ "id": 1, "name": "ComputerScience", "value": "ComputerScience" }
```

---

## Sections

All endpoints require:

- Auth: Yes

### `POST /api/section`

- Role: Admin
- Description: Create a section.
- Body: `CreateSectionDTO`
- Response: `ApiResponse<ViewSectionDTO>`

### `GET /api/section/{id}`

- Role: Admin, Instructor, Student
- Description: Get section by id.
- Body: None
- Response: `ApiResponse<ViewSectionDTO>`

### `GET /api/section?semester={semester}&year={year}&instructorId={id}&courseId={id}&seats={seats}`

- Role: Admin, Instructor, Student
- Description: Filter sections.
- Query (all optional):
  - `semester`
  - `year` (sent as a date value; only the year component is typically used)
  - `instructorId`
  - `courseId`
  - `seats`
- Response: `ApiResponse<IEnumerable<ViewSectionDTO>>`

### `PUT /api/section`

- Role: Admin
- Description: Update a section.
- Body: `UpdateSectionDTO`
- Response: `ApiResponse<ViewSectionDTO>`

### `DELETE /api/section/{id}`

- Role: Admin
- Description: Delete a section.
- Body: None
- Response: `ApiResponse<bool>`

---

## Cart & Enrollment

### `POST /api/cart?scheduleSlotId={scheduleSlotId}`

- Auth: Yes
- Role: Student
- Description: Add a schedule slot to the current student cart.
- Request body: none
- Response: `ApiResponse<string>`

### `POST /api/cart/by-course/{courseId}`

- Auth: Yes
- Role: Student
- Description: Adds the first available section/schedule slot for the course.
- Request body: none
- Response: `ApiResponse<string>`

### `DELETE /api/cart/{cartItemId}`

- Auth: Yes
- Role: Student
- Description: Remove cart item.
- Response (example):

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Success",
  "data": {
    "cartId": 1,
    "cartItems": []
  },
  "errors": null
}
```

### `GET /api/cart`

- Auth: Yes
- Role: Student
- Description: View current cart.
- Response: `ApiResponse<ViewCartDTO>`

### `POST /api/cart/checkout`

- Auth: Yes
- Role: Student
- Description: Validation-only checkout (idempotent).
- Response: `ApiResponse<object>` (includes validation details)

### `POST /api/cart/enroll`

- Auth: Yes
- Role: Student
- Description: Enroll all cart items.
- Response: `ApiResponse<string>`

### `GET /api/cart/my-enrollments`

- Auth: Yes
- Role: Student
- Description: Returns the student’s current enrollments. On failures, API returns an empty array (UX requirement).
- Response: `ApiResponse<IEnumerable<ViewEnrollmentDTO>>`

### `GET /api/enrollment/{id}`

- Auth: Yes
- Role: Any authenticated user (server enforces ownership unless Admin)
- Description: Get enrollment by id.
- Response: `ApiResponse<ViewEnrollmentDTO>`

### `PUT /api/enrollment/{id}`

- Auth: Yes
- Role: Admin, Instructor (instructors limited to their own sections)
- Description: Update grade and (admin-only) status.
- Body (example):

```json
{
  "grade": "A",
  "status": 3,
  "declineReason": null
}
```

- Response: `ApiResponse<string>`

### `DELETE /api/enrollment/{id}`

- Auth: Yes
- Role: Admin
- Description: Delete enrollment and return seat.
- Response: `ApiResponse<string>`

### `POST /api/enrollment/{id}/drop`

- Auth: Yes
- Role: Student (own enrollment) or Admin
- Description: Drop/withdraw an enrollment if within registration or withdraw windows.
- Response: `ApiResponse<string>`

### `POST /api/enrollment/{id}/approve`

- Auth: Yes
- Role: Admin
- Description: Approve pending enrollment.
- Response: `ApiResponse<string>`

### `POST /api/enrollment/{id}/decline`

- Auth: Yes
- Role: Admin
- Description: Decline pending enrollment (returns seat).
- Body:

```json
{ "reason": "Missing prerequisite" }
```

- Response: `ApiResponse<string>`

---

## GPA & Transcript

### `GET /api/gpa/my`

- Auth: Yes
- Role: Student
- Description: Returns student GPA summary and enrollment list. `CurrentGPA` is calculated; `StoredGPA` is the persisted value.
- Response: `ApiResponse<object>`

### `GET /api/gpa/student/{studentId}`

- Auth: Yes
- Role: Admin
- Description: Returns GPA summary for a student.
- Response: `ApiResponse<object>`

### `PUT /api/gpa/enrollment/{enrollmentId}/grade`

- Auth: Yes
- Role: Admin, Instructor (instructors limited to their own sections)
- Description: Update a grade for an enrollment and recalculate GPA.
- Body:

```json
{ "grade": "B+" }
```

- Response: `ApiResponse<object>`

### `POST /api/gpa/simulate`

- Auth: Yes
- Role: Student (self) or Admin/Instructor (must provide `studentId`)
- Description: What-if GPA calculator.
- Body:

```json
{
  "studentId": 123,
  "simulatedCourses": [
    { "transcriptId": 10, "grade": "A" },
    { "creditHours": 3, "grade": "B+" }
  ]
}
```

- Response: `ApiResponse<SimulateGpaResponseDTO>`

### `GET /api/transcript/my-transcript`

- Auth: Yes
- Role: Student
- Description: Returns student transcript summary.
- Response: `ApiResponse<StudentTranscriptSummaryDTO>`

### `GET /api/transcript/student/{studentUserId}`

- Auth: Yes
- Role: Admin, Instructor
- Description: Full transcript for a student by identity user id.
- Response: `ApiResponse<StudentTranscriptSummaryDTO>`

### `GET /api/transcript/{id}`

- Auth: Yes
- Role: Admin, Instructor
- Description: Transcript entry by id.
- Response: `ApiResponse<ViewTranscriptDTO>`

### `GET /api/transcript/by-student/{studentId}`

- Auth: Yes
- Role: Admin, Instructor
- Description: Transcript entries by student numeric id.
- Response: `ApiResponse<IEnumerable<ViewTranscriptDTO>>`

### `GET /api/transcript/by-semester?semester={semester}&year={year}`

- Auth: Yes
- Role: Admin, Instructor
- Description: Transcript entries for a term.
- Response: `ApiResponse<IEnumerable<ViewTranscriptDTO>>`

### `GET /api/transcript`

- Auth: Yes
- Role: Admin
- Description: Filter transcripts.
- Query: optional `studentId`, `courseId`, `semester`, `year`, `grade`
- Response: `ApiResponse<IEnumerable<ViewTranscriptDTO>>`

### `GET /api/transcript/students/search?query={query}&take={take}`

- Auth: Yes
- Role: Admin
- Description: Search students for transcript operations.
- Response: `ApiResponse<IEnumerable<StudentLookupDTO>>`

### `POST /api/transcript`

- Auth: Yes
- Role: Admin, Instructor
- Description: Create a transcript entry.
- Body:

```json
{
  "studentId": 123,
  "courseId": 12,
  "sectionId": 5,
  "grade": "A-",
  "semester": "Fall",
  "year": 2025
}
```

- Response: `ApiResponse<ViewTranscriptDTO>`

### `PUT /api/transcript`

- Auth: Yes
- Role: Admin, Instructor
- Description: Update grade for a transcript entry.
- Body:

```json
{ "transcriptId": 10, "grade": "B" }
```

- Response: `ApiResponse<ViewTranscriptDTO>`

### `DELETE /api/transcript/{id}`

- Auth: Yes
- Role: Admin
- Description: Delete transcript entry.
- Response: `ApiResponse<string>`

### `GET /api/transcript/gpa/{studentId}`

- Auth: Yes
- Role: Admin, Instructor
- Description: Calculate cumulative GPA.
- Response: `ApiResponse<double>`

### `GET /api/transcript/semester-gpa/{studentId}?semester={semester}&year={year}`

- Auth: Yes
- Role: Admin, Instructor
- Description: Calculate term GPA.
- Response: `ApiResponse<double>`

### `POST /api/transcript/recalculate-gpa/{studentId}`

- Auth: Yes
- Role: Admin
- Description: Forces GPA recalculation and persisted updates.
- Response: `ApiResponse<string>`

---

## Calendar

### `GET /api/calendar/registration-withdraw-dates`

- Auth: No
- Role: Public
- Description: Returns configured registration and withdraw window dates + computed status.
- Response: `ApiResponse<object>`

### `GET /api/calendar/timeline`

- Auth: No
- Role: Public
- Description: Timeline window view (read-only).
- Response: `ApiResponse<object>`

### `GET /api/calendar/events?startDate={date}&endDate={date}`

- Auth: Yes
- Role: Any authenticated user
- Description:
  - Students: global academic events + enrolled class meetings + office hour bookings
  - Instructors: global events + teaching schedule + office hours
  - Admins: global academic events only
- Response: `ApiResponse<object>`

### `GET /api/calendar/today`

- Auth: Yes
- Role: Any authenticated user
- Description: Convenience wrapper for today’s events.
- Response: `ApiResponse<object>`

### `GET /api/calendar/upcoming`

- Auth: Yes
- Role: Any authenticated user
- Description: Convenience wrapper for next 7 days.
- Response: `ApiResponse<object>`

### `GET /api/calendar/view?startDate={date}&endDate={date}`

- Auth: Yes
- Role: Any authenticated user
- Description: Unified role-aware calendar view (recommended endpoint for the calendar page). Returns events + conflict information.
- Notes:
  - Also accepts legacy query names `fromDate`/`toDate`.
- Response: `ApiResponse<object>` (data includes `{ viewRole, dateRange, events, conflicts }`)

### `GET /api/calendar/preferences`

- Auth: Yes
- Role: Any authenticated user
- Description: Get the current user's calendar UI/preferences.
- Response: `ApiResponse<object>`

### `PUT /api/calendar/preferences`

- Auth: Yes
- Role: Any authenticated user
- Description: Upsert the current user's calendar UI/preferences.
- Response: `ApiResponse<object>`

### `GET /api/notifications/reminder-rules`

- Auth: Yes
- Role: Any authenticated user
- Description: Get in-app reminder rules (used by the scheduled reminder dispatcher).
- Response: `ApiResponse<object>`

### `PUT /api/notifications/reminder-rules`

- Auth: Yes
- Role: Any authenticated user
- Description: Replace in-app reminder rules.
- Response: `ApiResponse<object>`

---

## Google Calendar Integration

All endpoints in this section require:

- Auth: Yes (except the OAuth callback)
- Role: Any authenticated user

### `GET /api/integrations/google-calendar/status`

- Description: Returns whether the current user is connected. Never returns tokens.
- Response: `ApiResponse<object>` (data: `{ connected, email }`)

### `GET /api/integrations/google-calendar/connect-url?returnUrl={relativePath}`

- Description: Returns an authorization URL. Frontend should navigate the browser to the returned URL.
- Security: `returnUrl` must be a local-relative path starting with `/`.
- Response: `ApiResponse<object>` (data: `{ url }`)

### `POST /api/integrations/google-calendar/disconnect`

- Description: Disconnect current user (removes stored tokens and event mappings). Best-effort.
- Response: `ApiResponse<string>`

### `GET /api/integrations/google-calendar/callback`

- Auth: No (Google redirects the browser)
- Description: OAuth callback endpoint configured in Google Cloud Console.
- Response: `text/plain` or redirect

---

## Chat

### `GET /api/chat`

- Auth: Yes
- Role: Any authenticated user
- Description: List user conversations.
- Response: `ApiResponse<ViewConversationsDTO>`

### `GET /api/chat/users/search?query={query}&limit={limit}`

- Auth: Yes
- Role: Any authenticated user
- Description: Search chat users.
- Response: `ApiResponse<List<ChatUserSearchResultDTO>>`

### `POST /api/chat/conversations/direct`

- Auth: Yes
- Role: Any authenticated user
- Description: Get or create a direct conversation and optionally return first page of messages.
- Body (example):

```json
{
  "otherUserId": "<identity-user-id>",
  "page": 1,
  "pageSize": 20
}
```

- Response: `ApiResponse<ViewConversationDTO>`

### `GET /api/chat/{conversationId}?page={page}&pageSize={pageSize}`

- Auth: Yes
- Role: Any authenticated user
- Description: Conversation messages.
- Notes:
  - Supports page-based paging (`page`/`pageSize`) and cursor paging (`beforeMessageId` + `pageSize`).
- Response: `ApiResponse<ViewConversationDTO>`

### `POST /api/chat/conversations/{conversationId}/messages/{messageId}/delete-for-me`

- Auth: Yes
- Role: Any authenticated user
- Description: Delete a message for the current user only.
- Response: `ApiResponse<object>`

### `POST /api/chat/conversations/{conversationId}/messages/{messageId}/delete-for-everyone`

- Auth: Yes
- Role: Any authenticated user
- Description: Delete a message for everyone in the conversation (redacts content).
- Response: `ApiResponse<object>`

### `POST /api/chat/conversations/{conversationId}/read`

- Auth: Yes
- Role: Any authenticated user
- Description: Marks conversation messages as read and notifies senders via SignalR.
- Response: `ApiResponse<object>`

### `POST /api/chat?receiverId={id}&textMessage={text}`

- Auth: Yes
- Role: Any authenticated user
- Description: Send a message. Requires either `receiverId` (new conversation) or `conversationId` (existing).
- Request body: none (query params)
- Response: `ApiResponse<ViewConversationDTO>`

SignalR hubs:

- `GET/WS /hubs/chat`
- Client events used in API flows:
  - `ReceiveMessage`
  - `ConversationCreated`
  - `MessageRead`
  - `UserTyping`
  - `UserPresenceChanged`
  - `MessageDeletedForMe`
  - `MessageDeletedForEveryone`

---

## Notifications

All endpoints in this section require:

- Auth: Yes
- Role: Any authenticated user

### `GET /api/notification?unreadOnly={bool}&page={page}&pageSize={pageSize}`

- Description: Get notifications for the current user.
- Query:
  - `unreadOnly` (optional)
  - `page` (default 1)
  - `pageSize` (default 20)
- Response: `ApiResponse<object>`
- Data shape (example):

```json
{
  "notifications": [
    {
      "notificationId": 1,
      "type": "Enrollment",
      "title": "Enrollment approved",
      "message": "Your enrollment was approved",
      "entityType": "Enrollment",
      "entityId": 123,
      "isRead": false,
      "readAt": null,
      "createdAt": "2026-01-01T10:00:00Z"
    }
  ],
  "totalCount": 10,
  "unreadCount": 3,
  "page": 1,
  "pageSize": 20,
  "totalPages": 1
}
```

### `GET /api/notification/unread-count`

- Description: Get unread notifications count.
- Response: `ApiResponse<object>` (data: `{ count }`)

### `POST /api/notification/{id}/read`

- Description: Mark a notification as read.
- Body: None
- Response: `ApiResponse<string>`

### `POST /api/notification/read-all`

- Description: Mark all notifications as read.
- Body: None
- Response: `ApiResponse<string>`

### `DELETE /api/notification/{id}`

- Description: Delete a notification.
- Body: None
- Response: `ApiResponse<string>`

### `DELETE /api/notification/clear-read`

- Description: Delete all read notifications.
- Body: None
- Response: `ApiResponse<string>`

---

## Office Hours

### Instructor

#### `GET /api/officehour/my-office-hours?fromDate={date}&toDate={date}`

- Auth: Yes
- Role: Instructor
- Description: List office hour slots for the current instructor.
- Response: `ApiResponse<object>`

#### `POST /api/officehour`

- Auth: Yes
- Role: Instructor
- Description: Create an office hour.
- Body (example):

```json
{
  "date": "2025-12-31T00:00:00Z",
  "startTime": "10:00",
  "endTime": "10:30",
  "roomId": 1,
  "isRecurring": false,
  "notes": "Bring your draft"
}
```

- Response: `ApiResponse<object>` (contains `officeHourId`)

#### `POST /api/officehour/batch`

- Auth: Yes
- Role: Instructor
- Description: Batch create office hours.
- Body: array of the create DTO
- Response: `ApiResponse<object>` (contains `createdIds`, `errors`)

#### `PUT /api/officehour/{id}`

- Auth: Yes
- Role: Instructor
- Description: Update office hour.
- Response: `ApiResponse<string>`

#### `DELETE /api/officehour/{id}`

- Auth: Yes
- Role: Instructor
- Description: Delete office hour.
- Response: `ApiResponse<string>`

#### `POST /api/officehour/bookings/{bookingId}/confirm`

- Auth: Yes
- Role: Instructor
- Description: Confirm booking.
- Response: `ApiResponse<string>`

#### `PUT /api/officehour/bookings/{bookingId}/notes`

- Auth: Yes
- Role: Instructor
- Description: Add instructor notes.
- Body:

```json
{ "notes": "Reviewed syllabus" }
```

- Response: `ApiResponse<string>`

#### `POST /api/officehour/bookings/{bookingId}/complete`

- Auth: Yes
- Role: Instructor
- Description: Mark booking completed.
- Response: `ApiResponse<string>`

#### `POST /api/officehour/bookings/{bookingId}/no-show`

- Auth: Yes
- Role: Instructor
- Description: Mark booking no-show.
- Response: `ApiResponse<string>`

### Student

#### `GET /api/officehour/available?instructorId={id}&fromDate={date}&toDate={date}`

- Auth: Yes
- Role: Student
- Description: List available office hours.
- Response: `ApiResponse<object>`

#### `GET /api/officehour/instructors`

- Auth: Yes
- Role: Student
- Description: Instructors + availability counts.
- Response: `ApiResponse<object>`

#### `POST /api/officehour/{id}/book`

- Auth: Yes
- Role: Student
- Description: Book an office hour.
- Body:

```json
{ "purpose": "Exam review", "studentNotes": "Need help with Q3" }
```

- Response: `ApiResponse<object>` (contains `bookingId`)

#### `GET /api/officehour/my-bookings?status={status}`

- Auth: Yes
- Role: Student
- Description: Student bookings.
- Response: `ApiResponse<object>`

#### `POST /api/officehour/bookings/{bookingId}/cancel`

- Auth: Yes
- Role: Student, Instructor
- Description: Cancel a booking.
- Body:

```json
{ "reason": "Schedule conflict" }
```

- Response: `ApiResponse<string>`

### Admin

#### `GET /api/officehour/all?instructorId={id}&fromDate={date}&toDate={date}&status={status}`

- Auth: Yes
- Role: Admin
- Description: Admin query of all office hours.
- Response: `ApiResponse<object>`

---

## Integrations (Google Calendar)

### `GET /api/integrations/google-calendar/connect-url?returnUrl=/settings`

- Auth: Yes
- Role: Any authenticated user
- Description: Returns a Google authorization URL. The frontend should navigate to the returned `url`.
- Response (example):

```json
{
  "success": true,
  "statusCode": 200,
  "message": "Success",
  "data": { "url": "https://accounts.google.com/o/oauth2/v2/auth?..." },
  "errors": null
}
```

### `GET /api/integrations/google-calendar/connect?returnUrl=/settings`

- Auth: Yes
- Role: Any authenticated user
- Description: Legacy redirect endpoint. Prefer `connect-url` because normal browser navigation won’t attach JWT.
- Response: `302 Redirect`

### `GET /api/integrations/google-calendar/status`

- Auth: Yes
- Role: Any authenticated user
- Description: Returns `{ connected, email }` and never returns tokens.
- Response: `ApiResponse<object>`

### `GET /api/integrations/google-calendar/callback?code=...&state=...`

- Auth: No
- Role: Public (OAuth callback)
- Description: OAuth callback configured in Google Console; stores tokens and redirects to safe return URL.
- Response: `302 Redirect` or `text/plain`

---

## Admin

All endpoints in this section require:

- Auth: Yes
- Role: Admin

### `GET /api/admin/stats`

- Description: Admin dashboard stats summary (users and enrollments).
- Body: None
- Response: `ApiResponse<object>` (see Response envelope example above)

### `GET /api/admin/users`

- Description: Paginated list of users.
- Query:
  - `email` (optional)
  - `role` (optional)
  - `pageNumber` (default 1)
  - `pageSize` (default 10)
- Body: None
- Response: `ApiResponse<object>` (items + paging fields)

### `GET /api/admin/users/{id}`

- Description: Get a user by id.
- Body: None
- Response: `ApiResponse<object>`

### `PUT /api/admin/users/{id}`

- Description: Update basic user fields.
- Body (example):

```json
{
  "fullName": "Jane Doe",
  "email": "jane.doe@example.com",
  "address": "Cairo"
}
```

- Response: `ApiResponse<object>`

### `DELETE /api/admin/users/{id}`

- Description: Delete a user.
- Body: None
- Response: `ApiResponse<string>`

### `PUT /api/admin/users/{id}/role`

- Description: Change a user role (Admin/Student/Instructor).
- Body (example):

```json
{ "newRole": "Instructor" }
```

- Response: `ApiResponse<string>`

### `POST /api/admin/create-user`

- Description: Create a user with an explicit role (Admin/Student/Instructor). Also creates the role-specific profile.
- Body: `CreateUserDTO` (shape depends on role; includes `fullName`, `email`, `password`, `role`, and optional profile fields)
- Response: `ApiResponse<object>`

### `GET /api/admin/students`

- Description: Paginated list of student users.
- Query:
  - `search` (optional)
  - `page` (default 1)
  - `pageSize` (default 10)
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/admin/students/{id}`

- Description: Get a student user by id.
- Body: None
- Response: `ApiResponse<object>`

### `PUT /api/admin/students/{id}`

- Description: Update a student user and selected student profile fields.
- Body: `UpdateStudentDTO`
- Response: `ApiResponse<object>`

### `DELETE /api/admin/students/{id}`

- Description: Delete a student user.
- Body: None
- Response: `ApiResponse<string>`

### `GET /api/admin/enrollments`

- Description: Paginated enrollments list for admin review.
- Query:
  - `search` (optional)
  - `status` (optional)
  - `page` (default 1)
  - `pageSize` (default 10)
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/admin/students/{studentId}/enrollments`

- Description: Get enrollments for a specific student user id.
- Body: None
- Response: `ApiResponse<IEnumerable<ViewEnrollmentDTO>>`

### `GET /api/admin/carts/{studentId}`

- Description: Get a student cart by student user id.
- Body: None
- Response: `ApiResponse<ViewCartDTO>`

### `GET /api/admin/students/{studentId}/cart`

- Description: Alias for viewing a student cart.
- Body: None
- Response: `ApiResponse<ViewCartDTO>`

### `POST /api/admin/students/{studentId}/force-enroll`

- Description: Force-enroll a student in a section (admin override).
- Body (example):

```json
{ "sectionId": 123 }
```

- Response: `ApiResponse<string>`

### `GET /api/admin/academic-calendar-settings`

- Description: Get the configured academic timeline dates.
- Body: None
- Response: `ApiResponse<object>`

### `PUT /api/admin/academic-calendar-settings`

- Description: Set the academic timeline dates (registration/withdraw windows).
- Body (example):

```json
{
  "registrationStartDate": "2026-01-05",
  "registrationEndDate": "2026-01-20",
  "withdrawStartDate": "2026-01-21",
  "withdrawEndDate": "2026-02-05"
}
```

- Response: `ApiResponse<string>`

### `POST /api/admin/registration-end-date`

- Description: Convenience endpoint to set registration end + withdraw end (withdraw start is set to registration end).
- Body (example):

```json
{
  "registrationEndDate": "2026-01-20",
  "withdrawEndDate": "2026-02-05"
}
```

- Response: `ApiResponse<string>`

---

## Analytics

All endpoints in this section require:

- Auth: Yes
- Role: Admin

### `GET /api/analytics/dashboard`

- Description: High-level dashboard overview (counts and status breakdowns).
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/analytics/enrollment-trends`

- Description: Enrollment trend series for the last 30 days (chart-ready).
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/analytics/course-stats`

- Description: Top course stats by enrollments.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/analytics/gpa-distribution`

- Description: GPA distribution summary + chart data.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/analytics/credits-distribution`

- Description: Completed credits distribution summary + chart data.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/analytics/instructor-stats`

- Description: Instructor-level stats (sections and student counts).
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/analytics/recent-activity`

- Description: Recent activity payload for admin dashboard.
- Query: `limit` (default 20)
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/analytics/section-capacity`

- Description: Capacity/utilization stats for sections.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/analytics/system-summary`

- Description: Aggregated system summary for admin.
- Body: None
- Response: `ApiResponse<object>`

---

## Advising

All endpoints in this section require:

- Auth: Yes
- Role: Instructor or Admin

### `GET /api/advising/pending`

- Description: Paginated list of pending enrollment requests.
- Query: `search` (optional), `page` (default 1), `pageSize` (default 10)
- Body: None
- Response: `ApiResponse<object>`

### `POST /api/advising/{enrollmentId}/approve`

- Description: Approve a pending enrollment request.
- Body: None
- Response: `ApiResponse<string>`

### `POST /api/advising/{enrollmentId}/decline`

- Description: Decline a pending enrollment request.
- Body (example):

```json
{ "reason": "Prerequisite not met" }
```

- Response: `ApiResponse<string>`

### `GET /api/advising/enrollments`

- Description: Paginated enrollments list for advisors.
- Query: `status` (optional), `search` (optional), `page` (default 1), `pageSize` (default 10)
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/advising/stats`

- Description: Summary counts (pending/approved/declined/today).
- Body: None
- Response: `ApiResponse<object>`

---

## Academic Plans

### `GET /api/academicplan/my-progress`

- Auth: Yes
- Role: Student
- Description: Get the current student's academic progress.
- Body: None
- Response: `ApiResponse<StudentAcademicProgressDTO>`

### `GET /api/academicplan/student-progress/{studentUserId}`

- Auth: Yes
- Role: Admin, Instructor
- Description: Get academic progress for a student (by Identity user id).
- Body: None
- Response: `ApiResponse<StudentAcademicProgressDTO>`

### `GET /api/academicplan`

- Auth: Yes
- Role: Admin, Instructor, Student
- Description: List academic plans.
- Body: None
- Response: `ApiResponse<IEnumerable<ViewAcademicPlanSummaryDTO>>`

### `GET /api/academicplan/{academicPlanId}`

- Auth: Yes
- Role: Admin, Instructor, Student
- Description: Get an academic plan by id.
- Body: None
- Response: `ApiResponse<ViewAcademicPlanDTO>`

### `GET /api/academicplan/{academicPlanId}/courses`

- Auth: Yes
- Role: Admin, Instructor, Student
- Description: List courses in an academic plan.
- Body: None
- Response: `ApiResponse<IEnumerable<AcademicPlanCourseDTO>>`

### `POST /api/academicplan`

- Auth: Yes
- Role: Admin
- Description: Create an academic plan.
- Body: `CreateAcademicPlanDTO`
- Response: `ApiResponse<ViewAcademicPlanDTO>`

### `PUT /api/academicplan`

- Auth: Yes
- Role: Admin
- Description: Update an academic plan.
- Body: `UpdateAcademicPlanDTO`
- Response: `ApiResponse<ViewAcademicPlanDTO>`

### `DELETE /api/academicplan/{academicPlanId}`

- Auth: Yes
- Role: Admin
- Description: Delete an academic plan.
- Body: None
- Response: `ApiResponse<string>`

### `POST /api/academicplan/add-course`

- Auth: Yes
- Role: Admin
- Description: Add a course to an academic plan.
- Body: `AddCourseToAcademicPlanDTO`
- Response: `ApiResponse<AcademicPlanCourseDTO>`

### `DELETE /api/academicplan/{academicPlanId}/courses/{courseId}`

- Auth: Yes
- Role: Admin
- Description: Remove a course from an academic plan.
- Body: None
- Response: `ApiResponse<string>`

### `POST /api/academicplan/assign-student`

- Auth: Yes
- Role: Admin
- Description: Assign a student to an academic plan. Supports JSON body or query-string payload.
- Body (example):

```json
{ "studentId": 123, "academicPlanId": "default" }
```

- Response: `ApiResponse<string>`

---

## Students

All endpoints require Auth: Yes.

### `POST /api/student`

- Role: Admin
- Description: Create a student profile.
- Body: `CreateStudentDTO`
- Response: `ApiResponse<ViewStudentProfileDTO>`

### `GET /api/student?id={id}`

- Role: Any authenticated user
- Description: Get a student profile by numeric student id.
- Body: None
- Response: `ApiResponse<ViewStudentProfileDTO>`

### `GET /api/student/me`

- Role: Any authenticated user
- Description: Get the current student's profile.
- Body: None
- Response: `ApiResponse<ViewStudentProfileDTO>`

### `GET /api/student/students`

- Role: Any authenticated user
- Description: Filtered list of students.
- Query: `GPA` (optional), `CompletedCredits` (optional), `AcademicPlanId` (optional)
- Body: None
- Response: `ApiResponse<List<ViewStudentProfileDTO>>`

### `PUT /api/student/update-student`

- Role: Admin, Student
- Description: Update a student profile (admin path).
- Body: `UpdateStudentProfileDTO`
- Response: `ApiResponse<ViewStudentProfileDTO>`

### `PUT /api/student`

- Role: Student
- Description: Change the current student's password (email is enforced from JWT).
- Body: `ChangePasswordDTO`
- Response: `ApiResponse<string>`

---

## Instructors

All endpoints require Auth: Yes.

### `POST /api/instructor`

- Role: Admin
- Description: Create an instructor profile.
- Body: `CreateInstructorDTO`
- Response: `ApiResponse<object>`

### `GET /api/instructor`

- Role: Admin, Student, Instructor
- Description: List instructors.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/instructor/{id}`

- Role: Admin, Student, Instructor
- Description: Get instructor by id.
- Body: None
- Response: `ApiResponse<object>`

### `PUT /api/instructor/{id}`

- Role: Admin
- Description: Update instructor fields.
- Body: `UpdateInstructorDTO`
- Response: `ApiResponse<object>`

### `DELETE /api/instructor/{id}`

- Role: Admin
- Description: Delete instructor.
- Body: None
- Response: `ApiResponse<string>`

### `GET /api/instructor/{id}/schedule`

- Role: Admin, Instructor
- Description: Get instructor schedule.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/instructor/my-schedule`

- Role: Instructor
- Description: Get schedule for the logged-in instructor.
- Body: None
- Response: `ApiResponse<object>`

---

## Rooms

All endpoints require Auth: Yes.

### `GET /api/room`

- Role: Admin, Instructor, Student
- Description: List rooms.
- Body: None
- Response: `ApiResponse<IEnumerable<ViewRoomDTO>>`

### `GET /api/room/{id}`

- Role: Admin, Instructor, Student
- Description: Get room by id.
- Body: None
- Response: `ApiResponse<ViewRoomDTO>`

### `POST /api/room`

- Role: Admin
- Description: Create a room (also auto-creates standard time slots for the week).
- Body: `CreateRoomDTO`
- Response: `ApiResponse<ViewRoomDTO>`

### `PUT /api/room`

- Role: Admin
- Description: Update a room.
- Body: `UpdateRoomDTO`
- Response: `ApiResponse<ViewRoomDTO>`

### `DELETE /api/room/{id}`

- Role: Admin
- Description: Delete a room.
- Body: None
- Response: `ApiResponse<string>`

---

## Time Slots

All endpoints require Auth: Yes.

### `GET /api/timeslot`

- Role: Admin, Instructor, Student
- Description: List time slots.
- Body: None
- Response: `ApiResponse<IEnumerable<ViewTimeSlotDTO>>`

### `GET /api/timeslot/room/{roomId}`

- Role: Admin, Instructor, Student
- Description: List time slots for a room.
- Body: None
- Response: `ApiResponse<IEnumerable<ViewTimeSlotDTO>>`

### `POST /api/timeslot`

- Role: Admin
- Description: Create a time slot.
- Body: `CreateTimeSlotDTO`
- Response: `ApiResponse<ViewTimeSlotDTO>`

### `PUT /api/timeslot/{id}`

- Role: Admin
- Description: Update a time slot.
- Body: `UpdateTimeSlotDTO` (must match URL id)
- Response: `ApiResponse<ViewTimeSlotDTO>`

### `DELETE /api/timeslot/{id}`

- Role: Admin
- Description: Delete a time slot.
- Body: None
- Response: `ApiResponse<string>`

---

## Schedule Slots

All endpoints require Auth: Yes.

### `POST /api/scheduleslot`

- Role: Admin
- Description: Create a schedule slot.
- Body: `CreateScheduleSlotDTO`
- Response: `ApiResponse<object>`

### `GET /api/scheduleslot`

- Role: Admin, Instructor, Student
- Description: List schedule slots.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/scheduleslot/section/{sectionId}`

- Role: Admin, Instructor, Student
- Description: List schedule slots for a section.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/scheduleslot/instructor/{instructorId}`

- Role: Admin, Instructor
- Description: List schedule slots for an instructor.
- Body: None
- Response: `ApiResponse<object>`

### `GET /api/scheduleslot/room/{roomId}`

- Role: Admin, Instructor, Student
- Description: List schedule slots for a room.
- Body: None
- Response: `ApiResponse<object>`

### `DELETE /api/scheduleslot/{id}`

- Role: Admin
- Description: Delete a schedule slot.
- Body: None
- Response: `ApiResponse<string>`

---

## Smart Schedule

### `POST /api/smartschedule/recommend`

- Auth: Yes
- Role: Student, Admin
- Description: Recommend a non-conflicting set of sections for the selected course ids.
- Body: `SmartScheduleRequestDTO`
- Response: `ApiResponse<object>` (returns recommended sections, unscheduled courses, and explanation)

---

## Developer Tools

These endpoints are intended for local development. They only work when the backend environment is `Development`.

### `POST /api/devtools/seed`

- Auth: No (but only available in Development)
- Role: Development only
- Description: Seed demo data (idempotent).
- Body: None
- Response: `ApiResponse<SeedResultDto>`

### `POST /api/devtools/reset`

- Auth: No (but only available in Development)
- Role: Development only
- Description: Reset DB (ensure deleted + migrate) then seed demo data.
- Body: None
- Response: `ApiResponse<SeedResultDto>`

### `GET /api/devtools/users`

- Auth: No (but only available in Development)
- Role: Development only
- Description: Get demo users list.
- Body: None
- Response: `ApiResponse<List<DemoUserInfoDto>>`

### `POST /api/devtools/login-as`

- Auth: No (but only available in Development)
- Role: Development only
- Description: Issue tokens for a demo user by email (dev helper).
- Body (example):

```json
{ "email": "student@demo.local" }
```

- Response: `ApiResponse<LoginResponseDTO>`

---

## Withdraw Requests

### Student

#### `POST /api/withdraw-requests`

- Auth: Yes
- Role: Student
- Description: Submit a withdraw request for an enrollment (only during withdraw window).
- Body:

```json
{ "enrollmentId": 123, "reason": "Medical" }
```

- Response: `ApiResponse<string>`

#### `GET /api/withdraw-requests/my`

- Auth: Yes
- Role: Student
- Description: List current student’s withdraw requests.
- Response: `ApiResponse<object>`

### Admin

#### `GET /api/admin/withdraw-requests`

- Auth: Yes
- Role: Admin
- Description: List all withdraw requests.
- Response: `ApiResponse<List<WithdrawRequestDTO>>`

#### `POST /api/admin/withdraw-requests/{requestId}/approve`

- Auth: Yes
- Role: Admin
- Description: Approve withdraw request and drop enrollment.
- Response: `ApiResponse<string>`

#### `POST /api/admin/withdraw-requests/{requestId}/deny`

- Auth: Yes
- Role: Admin
- Description: Deny withdraw request.
- Response: `ApiResponse<string>`
