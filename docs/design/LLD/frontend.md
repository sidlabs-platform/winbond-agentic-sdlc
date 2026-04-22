# Low-Level Design (LLD)

| Field                    | Value                                          |
|--------------------------|------------------------------------------------|
| **Title**                | Frontend (UI Layer) — Low-Level Design         |
| **Component**            | Frontend (UI Layer)                            |
| **Version**              | 1.0                                            |
| **Date**                 | 2026-04-22                                     |
| **Author**               | 2-plan-and-design-agent                        |
| **HLD Component Ref**    | COMP-006                                       |

---

## 1. Component Purpose & Scope

### 1.1 Purpose

The Frontend component delivers server-rendered HTML pages via Jinja2 templates with vanilla CSS and JavaScript. It provides all user-facing pages for both Learner and Admin roles, including sign-in, dashboard, course catalog, lesson viewer, course management, AI content generation, and admin reporting. The UI is responsive for desktop and tablet viewports (≥ 768px). This component satisfies BRD-FR-035 through BRD-FR-042.

### 1.2 Scope

- **Responsible for**: HTML page routes (non-API), Jinja2 template rendering, static CSS/JS assets, responsive layout, form submissions to API endpoints, navigation flow (2-click learner access).
- **Not responsible for**: API logic (COMP-001 through COMP-005), database access (COMP-007), content sanitization (COMP-002 sanitizer — sanitized HTML is passed to templates).
- **Interfaces with**: All API services via REST calls (client-side JS `fetch()` or form submissions), COMP-002 sanitizer (receives pre-sanitized HTML for lesson content).

---

## 2. Detailed Design

### 2.1 Module / Class Structure

```
src/
├── templates/                     # Jinja2 HTML templates
│   ├── base.html                  # Base layout with nav, header, footer
│   ├── login.html                 # Sign-in page (BRD-FR-035)
│   ├── register.html              # Registration page
│   ├── dashboard.html             # Learner dashboard (BRD-FR-036)
│   ├── catalog.html               # Course catalog (BRD-FR-037)
│   ├── course_detail.html         # Course detail with modules/lessons (BRD-FR-038)
│   ├── lesson.html                # Lesson viewer (BRD-FR-039)
│   ├── admin/
│   │   ├── courses.html           # Admin course management (BRD-FR-040)
│   │   ├── course_edit.html       # Course/module/lesson editor
│   │   ├── generate.html          # AI content generation form (BRD-FR-041)
│   │   └── reports.html           # Admin reporting dashboard (BRD-FR-042)
│   └── components/
│       ├── nav.html               # Navigation bar partial
│       ├── course_card.html       # Reusable course card partial
│       └── progress_bar.html      # Progress bar partial
├── static/
│   ├── css/
│   │   └── style.css              # Main stylesheet (responsive)
│   └── js/
│       └── app.js                 # Client-side interactivity (fetch API calls, form handling)
└── frontend/
    ├── __init__.py
    └── router.py                  # FastAPI routes for HTML page endpoints
```

### 2.2 Key Page Routes

| Route                    | Template              | Description                                        | Auth            | BRD Req     |
|--------------------------|-----------------------|----------------------------------------------------|-----------------|-------------|
| GET /login               | login.html            | Sign-in form                                       | Public          | BRD-FR-035  |
| GET /register            | register.html         | Registration form                                  | Public          | BRD-FR-003  |
| GET /dashboard           | dashboard.html        | Learner dashboard with enrolled courses & progress | Learner         | BRD-FR-036  |
| GET /courses             | catalog.html          | Course catalog with published courses              | Learner/Admin   | BRD-FR-037  |
| GET /courses/{id}        | course_detail.html    | Course detail with modules and lessons             | Learner/Admin   | BRD-FR-038  |
| GET /lessons/{id}        | lesson.html           | Lesson viewer with sanitized content               | Learner         | BRD-FR-039  |
| GET /admin/courses       | admin/courses.html    | Admin course management table                      | Admin           | BRD-FR-040  |
| GET /admin/generate      | admin/generate.html   | AI content generation form                         | Admin           | BRD-FR-041  |
| GET /admin/reports       | admin/reports.html    | Admin reporting dashboard                          | Admin           | BRD-FR-042  |

### 2.3 Design Patterns Used

- **Template Inheritance**: All pages extend `base.html` which provides the common layout (nav, footer, CSS/JS includes).
- **Partials / Includes**: Reusable UI components (course cards, progress bars, navigation) are Jinja2 includes.
- **Progressive Enhancement**: Pages render server-side; JavaScript enhances with dynamic fetch calls for actions like enrollment, lesson completion, and quiz submission.
- **Responsive Design**: CSS media queries for ≥ 768px tablet and desktop layouts. Mobile is out of scope but layout degrades gracefully.

---

## 3. Page Designs

### 3.1 Navigation Structure

```
Public Pages:
  /login → Sign-in form
  /register → Registration form

Learner Pages (after sign-in):
  /dashboard → Enrolled courses with progress → [Click course] → /courses/{id}
  /courses → Course catalog → [Click course] → /courses/{id}
  /courses/{id} → Modules & lessons → [Click lesson] → /lessons/{id}
  /lessons/{id} → Lesson content + "Mark Complete" button

Admin Pages (after sign-in):
  /admin/courses → Course management table (CRUD, publish/unpublish)
  /admin/generate → AI content generation form
  /admin/reports → Enrollment/completion stats

Navigation Flow (2-click requirement — BRD-NFR-007):
  Dashboard → Course Detail → Lesson Viewer
  (Click 1: course link, Click 2: lesson link)
```

### 3.2 Base Layout (`base.html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Learning Platform{% endblock %}</title>
    <link rel="stylesheet" href="/static/css/style.css">
</head>
<body>
    {% include "components/nav.html" %}
    <main class="container">
        {% block content %}{% endblock %}
    </main>
    <footer>
        <p>&copy; 2026 AI-Powered Learning Platform</p>
    </footer>
    <script src="/static/js/app.js"></script>
</body>
</html>
```

### 3.3 Key Page Elements

| Page              | Key Elements                                                                                  |
|-------------------|-----------------------------------------------------------------------------------------------|
| login.html        | Email input, password input, sign-in button, link to register                                |
| dashboard.html    | List of enrolled courses with progress bars, "Resume" links, course completion status         |
| catalog.html      | Grid of course cards (title, description, difficulty badge, "Enroll" button)                  |
| course_detail.html | Course title/description, accordion of modules, lesson links with completion checkmarks      |
| lesson.html       | Lesson title, sanitized HTML content area, "Mark Complete" button, next/previous navigation  |
| admin/courses.html | Table of all courses (draft/published), create/edit/delete/publish action buttons            |
| admin/generate.html | Form: topic, audience, objectives, difficulty dropdown, output type, "Generate" button      |
| admin/reports.html | Stats cards (total learners, enrollments), per-course table with completion rates             |

---

## 4. API Integration (Client-Side)

### 4.1 JavaScript API Calls

The frontend uses vanilla JavaScript `fetch()` to interact with REST API endpoints. The JWT token is stored in `localStorage` and sent as a `Bearer` token in the `Authorization` header.

| Action              | API Call                                  | Trigger                        |
|---------------------|-------------------------------------------|--------------------------------|
| Sign in             | POST /api/v1/auth/login                   | Login form submit              |
| Register            | POST /api/v1/auth/register                | Register form submit           |
| Enroll in course    | POST /api/v1/courses/{id}/enroll          | "Enroll" button click          |
| Mark lesson complete | POST /api/v1/lessons/{id}/complete       | "Mark Complete" button click   |
| Submit quiz answer  | POST /api/v1/quizzes/{id}/attempt         | Quiz form submit               |
| Generate content    | POST /api/v1/ai/generate                  | Generation form submit         |
| Create course       | POST /api/v1/courses                      | Course creation form submit    |
| Publish course      | POST /api/v1/courses/{id}/publish         | "Publish" button click         |

### 4.2 Token Management

```javascript
// Store token after login
localStorage.setItem('access_token', response.access_token);

// Include token in API requests
fetch('/api/v1/courses', {
    headers: {
        'Authorization': `Bearer ${localStorage.getItem('access_token')}`
    }
});
```

---

## 5. Styling Approach

### 5.1 CSS Structure

- **Reset / Normalize**: Minimal CSS reset for consistent cross-browser rendering.
- **Layout**: CSS Grid for page layout; Flexbox for component alignment.
- **Responsive**: Media queries at 768px breakpoint for tablet/desktop.
- **Color Scheme**: Neutral palette with accent colors for actions (enroll, publish, complete).
- **Typography**: System font stack for performance; clear hierarchy with headings.

### 5.2 Responsive Breakpoints

| Breakpoint | Layout                                          |
|------------|--------------------------------------------------|
| < 768px    | Single column (out of scope but graceful degrade) |
| ≥ 768px    | Two-column layout for catalogs, side navigation  |
| ≥ 1024px   | Three-column course catalog grid, wider content  |

---

## 6. Error Handling Strategy

### 6.1 Client-Side Errors

| Error Type               | User Experience                                              |
|--------------------------|--------------------------------------------------------------|
| API error (4xx/5xx)      | Display error message banner at top of page                  |
| Network error            | Display "Unable to connect. Please check your connection."   |
| Auth token expired       | Redirect to /login with "Session expired" message            |
| Validation error         | Inline field-level error messages                            |

### 6.2 Server-Side Errors

| Error Type               | Behavior                                                     |
|--------------------------|--------------------------------------------------------------|
| Template render error    | Return 500 with a generic error page                         |
| Resource not found       | Return 404 with a "Page not found" template                  |

---

## 7. Configuration & Environment Variables

No additional configuration. The frontend uses the same FastAPI application server and inherits its settings.

---

## 8. Dependencies

### 8.1 Internal Dependencies

| Component              | Purpose                                              | Interface                     |
|------------------------|------------------------------------------------------|-------------------------------|
| COMP-001 (Auth)        | Token validation for protected pages                 | JWT token in page route deps  |
| COMP-002 (Courses)     | Course/module/lesson data for page rendering         | REST API calls                |
| COMP-003 (AI)          | AI generation form submission and results            | REST API calls                |
| COMP-004 (Progress)    | Enrollment and progress data for dashboard           | REST API calls                |
| COMP-005 (Reporting)   | Admin report data for reporting dashboard            | REST API calls                |

### 8.2 External Dependencies

| Package / Service       | Version           | Purpose                                           |
|-------------------------|-------------------|---------------------------------------------------|
| jinja2                  | 3.x               | Server-side HTML template rendering                |
| fastapi                 | 0.115+            | Static file serving and template response          |

---

## 9. Traceability

| LLD Element                  | HLD Component  | BRD Requirement(s)              |
|------------------------------|----------------|---------------------------------|
| GET /login (login.html)      | COMP-006       | BRD-FR-035                      |
| GET /dashboard               | COMP-006       | BRD-FR-036, BRD-NFR-007        |
| GET /courses (catalog)       | COMP-006       | BRD-FR-037                      |
| GET /courses/{id} (detail)   | COMP-006       | BRD-FR-038                      |
| GET /lessons/{id} (viewer)   | COMP-006       | BRD-FR-039, BRD-NFR-005        |
| GET /admin/courses           | COMP-006       | BRD-FR-040                      |
| GET /admin/generate          | COMP-006       | BRD-FR-041                      |
| GET /admin/reports           | COMP-006       | BRD-FR-042                      |
| Responsive CSS               | COMP-006       | BRD-NFR-008                     |
| 2-click navigation           | COMP-006       | BRD-NFR-007                     |
