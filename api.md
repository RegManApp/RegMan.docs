# API Documentation

This document describes the RegMan HTTP API surface as implemented in the ASP.NET Core controllers.

## Conventions

### Base URL

Configured by environment.

- Local: e.g. `http://localhost:5240`

### Authentication

Most endpoints require JWT.

- Header: `Authorization: Bearer <accessToken>`
- Roles are enforced via `[Authorize(Roles = ...)]`.

### Response envelope

Most endpoints return:

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
- Response: `ApiResponse<IEnumerable<...>>` (category DTO depends on implementation)

### `GET /api/coursecategories/{id}`

- Auth: Yes
- Role: Any authenticated user
- Description: Get category by id.
- Response: `ApiResponse<...>`

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
- Description: Conversation messages (paged).
- Response: `ApiResponse<ViewConversationDTO>`

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
  - `ConversationCreated`
  - `MessageRead`

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
