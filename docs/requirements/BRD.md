# Business Requirements Document (BRD)

| Field       | Value                                              |
|-------------|----------------------------------------------------|
| **Title**   | AI-Powered Learning Platform MVP                   |
| **Version** | 1.0                                                |
| **Date**    | 2026-04-22                                         |
| **Author**  | 1-requirement-agent                                |
| **Status**  | Draft                                              |

---

## 1. Executive Summary

### 1.1 Project Overview

The AI-Powered Learning Platform is a minimum viable product (MVP) that enables Learners to enroll in and complete structured courses while Admins create and manage course content with AI-assisted generation via the GitHub Models API (GPT-4o). The platform is built with Python 3.11+, FastAPI, SQLite, and Jinja2 templates. It ships with three starter courses: **GitHub Foundations**, **GitHub Advanced Security**, and **GitHub Actions**. Course content follows a hierarchical structure (Course → Module → Lesson + Quiz) and all AI-generated content undergoes an admin draft-review-publish governance workflow before becoming visible to learners.

### 1.2 Business Objectives

- Provide a self-service learning platform where learners can enroll in, progress through, and complete structured courses on GitHub technologies.
- Enable admins to rapidly create and iterate on course content using AI-assisted generation, reducing manual authoring effort.
- Enforce a content governance workflow (draft → review → publish) so that all AI-generated material is human-reviewed before learner access.
- Track learner progress at lesson, module, and course level with quiz-based assessment.
- Deliver a responsive, minimal-dependency web experience accessible on desktop and tablet.

### 1.3 Success Metrics / KPIs

| Metric ID | Metric                                      | Target                          | Measurement Method                          |
|-----------|---------------------------------------------|---------------------------------|---------------------------------------------|
| KPI-001   | Learner course enrollment rate              | ≥ 80% of registered learners enroll in at least one course | Database enrollment count vs. user count |
| KPI-002   | Course completion rate                      | ≥ 50% of enrolled learners complete a course | Enrollment status tracking                 |
| KPI-003   | Non-AI API response time                    | < 2 seconds (p95)              | Server-side latency logging                 |
| KPI-004   | AI content generation success rate          | ≥ 95% of generation requests succeed | Generation request status tracking        |
| KPI-005   | Learner navigation efficiency               | Reach next lesson within 2 clicks from dashboard | UX audit / manual verification        |

---

## 2. Background & Context

Organizations need effective, scalable training on GitHub technologies — including GitHub Foundations, GitHub Advanced Security, and GitHub Actions. Traditional course authoring is slow and expensive. By integrating AI-powered content generation (via GitHub Models API), admins can rapidly produce draft course material, review and refine it, and publish polished courses to learners.

This MVP addresses the need for a lightweight, self-hosted learning platform that:
- Eliminates dependency on heavyweight LMS products for internal training.
- Leverages AI to accelerate content creation while maintaining human oversight.
- Provides structured progress tracking so learners and admins can monitor completion.
- Uses a zero-config SQLite database and minimal tech stack for rapid deployment.

---

## 3. Stakeholders

| Name                  | Role                      | Interest                                         | Influence |
|-----------------------|---------------------------|--------------------------------------------------|-----------|
| Platform Admin        | Course Author / Manager   | Create courses, manage content, view reports     | High      |
| Learner               | End User / Student        | Enroll in courses, complete lessons, track progress | High    |
| Engineering Team      | Development & Operations  | Build, deploy, and maintain the platform         | High      |
| Product Owner         | Business Sponsor          | Ensure platform meets learning objectives        | Medium    |

---

## 4. Scope

### 4.1 In-Scope

- User authentication and role-based access control (Admin, Learner)
- Course management CRUD (courses, modules, lessons, quiz questions)
- Course catalog browsing and search for learners
- Learner enrollment in courses
- Lesson viewing with Markdown content rendering (XSS-sanitized)
- Quiz functionality with scoring and feedback
- Progress tracking at lesson, module, and course level
- AI-assisted content generation via GitHub Models API (GPT-4o)
- Content governance workflow: draft → admin review → publish
- Admin reporting dashboard (enrollment stats, completion rates)
- Three starter courses: GitHub Foundations, GitHub Advanced Security, GitHub Actions
- Responsive web frontend using Jinja2 templates (desktop and tablet)
- REST API under `/api/v1/`

### 4.2 Out-of-Scope

- Mobile native applications
- Multi-tenant / SaaS deployment
- Payment or subscription billing
- Video or multimedia content hosting
- Real-time collaboration or chat features
- Advanced analytics or ML-based recommendations
- Email notifications or push notifications
- SSO / OAuth integration (simple credential-based auth for MVP)
- Internationalization / localization
- Certificate or badge generation

### 4.3 Assumptions

- Users have network access to the deployed platform.
- The GitHub Models API is available and accessible with a valid API key.
- Admins have sufficient domain knowledge to review and approve AI-generated content.
- SQLite is adequate for MVP-scale concurrency (single-server deployment).
- Python 3.11+ runtime is available in the deployment environment.

### 4.4 Constraints

- MVP must use GitHub Models API (GPT-4o) for all AI content generation.
- Database must be SQLite (zero-config, file-based) — no external database servers.
- Frontend must use vanilla HTML/CSS/JS with Jinja2 templates — no frontend frameworks.
- API keys and secrets must be loaded from environment variables only.
- AI-generated content must never be auto-published; admin review is mandatory.

### 4.5 Dependencies

- GitHub Models API availability and rate limits
- Python 3.11+ runtime
- FastAPI framework and Pydantic v2
- SQLite database engine (bundled with Python)

---

## 5. Use Cases

| Use Case ID | Name                              | Description                                                                                                  | Priority    | Actors   |
|-------------|-----------------------------------|--------------------------------------------------------------------------------------------------------------|-------------|----------|
| UC-001      | Learner Sign-In                   | A learner signs in with credentials and is directed to their dashboard showing enrolled courses and progress. | Must-Have   | Learner  |
| UC-002      | Browse Course Catalog             | A learner browses the published course catalog, filtering by difficulty or topic, and views course details.   | Must-Have   | Learner  |
| UC-003      | Enroll in a Course                | A learner enrolls in a published course, which creates an enrollment record with status "not_started".        | Must-Have   | Learner  |
| UC-004      | Complete a Lesson                 | A learner views a lesson's Markdown content, marks it complete, and progress is recorded.                    | Must-Have   | Learner  |
| UC-005      | Take a Quiz                       | A learner answers quiz questions for a module, receives immediate scoring and explanations.                   | Must-Have   | Learner  |
| UC-006      | View Progress Dashboard           | A learner views their progress across all enrolled courses, including module and lesson completion status.    | Must-Have   | Learner  |
| UC-007      | Admin Sign-In                     | An admin signs in with credentials and accesses the admin dashboard with course management and reporting.     | Must-Have   | Admin    |
| UC-008      | Create a Course                   | An admin creates a new course with title, description, difficulty, estimated duration, and tags.              | Must-Have   | Admin    |
| UC-009      | Add Modules and Lessons           | An admin adds modules to a course and lessons to modules, specifying sort order and content.                  | Must-Have   | Admin    |
| UC-010      | AI-Generate Course Content        | An admin provides topic, audience, objectives, and difficulty; the system generates draft content via GitHub Models. | Must-Have | Admin |
| UC-011      | Review and Publish Content        | An admin reviews AI-generated draft content, edits it, and publishes it to make it visible to learners.      | Must-Have   | Admin    |
| UC-012      | Manage Quiz Questions             | An admin creates, edits, or deletes quiz questions for a module, including options and correct answers.       | Must-Have   | Admin    |
| UC-013      | View Enrollment and Completion Reports | An admin views dashboard reports showing enrollment counts, completion rates, and learner progress.      | Should-Have | Admin    |
| UC-014      | Regenerate Content Section        | An admin regenerates a specific lesson or section without regenerating the entire course.                     | Should-Have | Admin    |
| UC-015      | Manage Enrollments                | An admin manually enrolls or removes learners from courses.                                                  | Should-Have | Admin    |
| UC-016      | Export Reports                    | An admin exports enrollment and completion data for offline analysis.                                        | Nice-to-Have | Admin   |

---

## 6. Functional Requirements

### 6.1 Authentication & Authorization

| Req ID      | Description                                                                                       | Priority    | Acceptance Criteria                                                                                              |
|-------------|---------------------------------------------------------------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------|
| BRD-FR-001  | The system shall provide a sign-in endpoint that authenticates users by email and password.        | Must-Have   | POST `/api/v1/auth/login` returns a session/token on valid credentials; returns 401 on invalid credentials.       |
| BRD-FR-002  | The system shall enforce role-based access control (RBAC) with two roles: Admin and Learner.      | Must-Have   | Admin-only endpoints return 403 when accessed by a Learner. Learner endpoints restrict data to the authenticated user. |
| BRD-FR-003  | The system shall provide a user registration endpoint for creating new accounts with a role.      | Must-Have   | POST `/api/v1/auth/register` creates a User with id, name, email, and role; returns 409 if email already exists. |

### 6.2 Course Management

| Req ID      | Description                                                                                       | Priority    | Acceptance Criteria                                                                                              |
|-------------|---------------------------------------------------------------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------|
| BRD-FR-004  | The system shall allow admins to create a course with title, description, status, difficulty, estimatedDuration, and tags. | Must-Have | POST `/api/v1/courses` creates a course in "draft" status; returns the created course object with generated id.  |
| BRD-FR-005  | The system shall allow admins to update course metadata (title, description, difficulty, tags, estimatedDuration). | Must-Have | PUT `/api/v1/courses/{id}` updates specified fields and returns the updated course.                              |
| BRD-FR-006  | The system shall allow admins to delete a course and its associated modules, lessons, and quiz questions. | Must-Have | DELETE `/api/v1/courses/{id}` removes the course and all child entities; returns 404 if course does not exist.   |
| BRD-FR-007  | The system shall allow admins to publish a course, changing its status from "draft" to "published". | Must-Have | POST `/api/v1/courses/{id}/publish` sets status to "published"; returns 400 if course has no modules or lessons. |
| BRD-FR-008  | The system shall allow admins to unpublish a course, reverting its status to "draft".              | Must-Have   | POST `/api/v1/courses/{id}/unpublish` sets status to "draft"; existing enrollments are preserved.                |
| BRD-FR-009  | The system shall provide a course catalog endpoint that returns all published courses.             | Must-Have   | GET `/api/v1/courses` returns only courses with status "published" for Learner role; Admins see all courses.     |
| BRD-FR-010  | The system shall provide a course detail endpoint returning course info with its modules and lesson summaries. | Must-Have | GET `/api/v1/courses/{id}` returns course metadata, ordered modules, and lesson titles.                         |

### 6.3 Module & Lesson Management

| Req ID      | Description                                                                                       | Priority    | Acceptance Criteria                                                                                              |
|-------------|---------------------------------------------------------------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------|
| BRD-FR-011  | The system shall allow admins to create modules within a course, specifying title, summary, and sortOrder. | Must-Have | POST `/api/v1/courses/{courseId}/modules` creates a module; returns the module with generated id.                |
| BRD-FR-012  | The system shall allow admins to update and delete modules.                                        | Must-Have   | PUT and DELETE on `/api/v1/modules/{id}` update or remove the module and its child lessons/quizzes.              |
| BRD-FR-013  | The system shall allow admins to create lessons within a module, specifying title, markdownContent, estimatedMinutes, and sortOrder. | Must-Have | POST `/api/v1/modules/{moduleId}/lessons` creates a lesson; content is stored as Markdown.                 |
| BRD-FR-014  | The system shall allow admins to update and delete lessons.                                        | Must-Have   | PUT and DELETE on `/api/v1/lessons/{id}` update or remove the lesson.                                            |
| BRD-FR-015  | The system shall render lesson Markdown content with XSS sanitization when displayed to learners.  | Must-Have   | Lesson content endpoint returns sanitized HTML; script tags and event handlers are stripped.                      |

### 6.4 Quiz Management

| Req ID      | Description                                                                                       | Priority    | Acceptance Criteria                                                                                              |
|-------------|---------------------------------------------------------------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------|
| BRD-FR-016  | The system shall allow admins to create quiz questions for a module with question text, options, correctAnswer, and explanation. | Must-Have | POST `/api/v1/modules/{moduleId}/quizzes` creates a quiz question; options is a JSON array of strings.       |
| BRD-FR-017  | The system shall allow admins to update and delete quiz questions.                                 | Must-Have   | PUT and DELETE on `/api/v1/quizzes/{id}` update or remove the quiz question.                                     |
| BRD-FR-018  | The system shall allow learners to submit quiz answers and receive immediate scoring with explanations. | Must-Have | POST `/api/v1/quizzes/{id}/attempt` records the attempt, returns isCorrect and explanation.                      |

### 6.5 Enrollment & Progress Tracking

| Req ID      | Description                                                                                       | Priority    | Acceptance Criteria                                                                                              |
|-------------|---------------------------------------------------------------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------|
| BRD-FR-019  | The system shall allow learners to enroll in a published course.                                   | Must-Have   | POST `/api/v1/courses/{id}/enroll` creates an Enrollment with status "not_started"; returns 400 if already enrolled or course not published. |
| BRD-FR-020  | The system shall track lesson completion for each learner.                                         | Must-Have   | POST `/api/v1/lessons/{id}/complete` creates/updates a ProgressRecord with completed=true and completedAt timestamp. |
| BRD-FR-021  | The system shall automatically update module progress when all lessons in a module are completed.  | Must-Have   | Module is marked complete when all its lessons have completed ProgressRecords for the learner.                    |
| BRD-FR-022  | The system shall automatically update course enrollment status to "completed" when all modules are completed. | Must-Have | Enrollment status transitions: not_started → in_progress (on first lesson complete) → completed (all modules done). |
| BRD-FR-023  | The system shall persist progress so it is never lost on page refresh.                             | Must-Have   | ProgressRecords are written to the database immediately; refreshing the page shows the same progress state.      |
| BRD-FR-024  | The system shall provide a learner dashboard endpoint showing all enrollments and per-course progress. | Must-Have | GET `/api/v1/enrollments` returns the learner's enrollments with course info, module/lesson completion counts.   |

### 6.6 Admin Reporting

| Req ID      | Description                                                                                       | Priority     | Acceptance Criteria                                                                                             |
|-------------|---------------------------------------------------------------------------------------------------|--------------|-----------------------------------------------------------------------------------------------------------------|
| BRD-FR-025  | The system shall provide an admin dashboard endpoint with enrollment counts and completion rates per course. | Should-Have | GET `/api/v1/admin/reports/dashboard` returns aggregated stats for each course.                                |
| BRD-FR-026  | The system shall allow admins to view per-learner progress for a specific course.                  | Should-Have  | GET `/api/v1/admin/reports/courses/{id}/learners` returns each learner's progress within the course.            |
| BRD-FR-027  | The system shall allow admins to export enrollment and completion data as CSV.                     | Nice-to-Have | GET `/api/v1/admin/reports/export?format=csv` returns a CSV file download.                                     |

### 6.7 Content Governance

| Req ID      | Description                                                                                       | Priority    | Acceptance Criteria                                                                                              |
|-------------|---------------------------------------------------------------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------|
| BRD-FR-028  | AI-generated content shall be saved as "draft" with metadata (prompt, model, timestamp, requester). | Must-Have  | ContentGenerationArtifact records include generatedContent, sourceRequestId, and are not visible to learners until approved. |
| BRD-FR-029  | Admins shall review, edit, and explicitly approve AI-generated content before it becomes visible to learners. | Must-Have | Content requires an approvedBy and approvedAt value before the parent lesson/course can be published.         |
| BRD-FR-030  | AI-generated content shall be labeled as "AI-generated" until explicitly approved by an admin.     | Must-Have   | Draft content includes an `isAiGenerated` flag; the flag persists until admin approval.                          |

### 6.8 Starter Courses

| Req ID      | Description                                                                                       | Priority    | Acceptance Criteria                                                                                              |
|-------------|---------------------------------------------------------------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------|
| BRD-FR-031  | The platform shall ship with a "GitHub Foundations" starter course with 3–5 modules and 1–3 lessons per module. | Must-Have | Course exists in the database on initial setup; contains structured modules and lessons with Markdown content.  |
| BRD-FR-032  | The platform shall ship with a "GitHub Advanced Security" starter course with 3–5 modules and 1–3 lessons per module. | Must-Have | Course exists in the database on initial setup with structured content.                                       |
| BRD-FR-033  | The platform shall ship with a "GitHub Actions" starter course with 3–5 modules and 1–3 lessons per module. | Must-Have | Course exists in the database on initial setup with structured content.                                        |
| BRD-FR-034  | Each starter course shall include at least one quiz per course.                                    | Must-Have   | At least one QuizQuestion record exists per starter course in the database.                                      |

### 6.9 Frontend Pages

| Req ID      | Description                                                                                       | Priority    | Acceptance Criteria                                                                                              |
|-------------|---------------------------------------------------------------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------|
| BRD-FR-035  | The system shall provide a sign-in page rendered via Jinja2 template.                              | Must-Have   | GET `/login` returns an HTML form; form submits to the auth API endpoint.                                        |
| BRD-FR-036  | The system shall provide a learner dashboard page showing enrolled courses and progress.           | Must-Have   | GET `/dashboard` renders enrolled courses with progress bars and links to resume.                                 |
| BRD-FR-037  | The system shall provide a course catalog page listing all published courses.                      | Must-Have   | GET `/courses` renders course cards with title, description, difficulty, and an enroll button.                    |
| BRD-FR-038  | The system shall provide a course detail page showing modules and lessons.                         | Must-Have   | GET `/courses/{id}` renders the course with expandable modules and lesson links.                                 |
| BRD-FR-039  | The system shall provide a lesson viewer page rendering sanitized Markdown content.                | Must-Have   | GET `/lessons/{id}` renders lesson content as sanitized HTML with a "Mark Complete" button.                       |
| BRD-FR-040  | The system shall provide an admin course management page for CRUD operations.                     | Must-Have   | GET `/admin/courses` renders a table of courses with create/edit/delete/publish actions.                         |
| BRD-FR-041  | The system shall provide an admin AI content generation page.                                      | Must-Have   | GET `/admin/generate` renders a form for topic, audience, objectives, difficulty; submits to the AI generation API. |
| BRD-FR-042  | The system shall provide an admin reporting dashboard page.                                        | Should-Have | GET `/admin/reports` renders charts/tables with enrollment and completion stats.                                  |

---

## 7. Non-Functional Requirements

| Req ID       | Category       | Description                                                                                   | Target                                             |
|--------------|----------------|-----------------------------------------------------------------------------------------------|----------------------------------------------------|
| BRD-NFR-001  | Performance    | Non-AI API endpoints shall respond within 2 seconds at the 95th percentile.                   | < 2s (p95)                                         |
| BRD-NFR-002  | Performance    | AI content generation requests may be handled asynchronously if execution exceeds 5 seconds.  | Async response with status polling                 |
| BRD-NFR-003  | Security       | RBAC shall be enforced on all API endpoints; unauthorized access returns 401/403.             | 100% of endpoints have role checks                 |
| BRD-NFR-004  | Security       | All secrets (API keys, database credentials) shall be loaded from environment variables only. | Zero hardcoded secrets in source code              |
| BRD-NFR-005  | Security       | Rendered Markdown lesson content shall be sanitized to prevent XSS attacks.                   | No executable scripts in rendered HTML output      |
| BRD-NFR-006  | Security       | CORS shall be scoped to allowed origins only.                                                 | CORS policy configured via environment variable    |
| BRD-NFR-007  | Usability      | A learner shall reach the next lesson within 2 clicks from the dashboard.                     | ≤ 2 clicks from dashboard to next lesson           |
| BRD-NFR-008  | Usability      | The frontend shall be responsive for desktop and tablet screen sizes.                         | Functional on viewports ≥ 768px                    |
| BRD-NFR-009  | Reliability    | Learner progress shall never be lost on page refresh.                                         | Progress persisted to database before response     |
| BRD-NFR-010  | Reliability    | AI generation failures shall return retryable error states with meaningful error messages.    | Error response includes retry guidance             |
| BRD-NFR-011  | Reliability    | Published courses shall remain accessible even when the AI service is unavailable.            | Course serving has no runtime dependency on AI API |
| BRD-NFR-012  | Observability  | The system shall log authentication events (sign-in, failed attempts).                        | Auth events appear in application logs             |
| BRD-NFR-013  | Observability  | The system shall log publishing actions and AI generation calls with latency metrics.         | Logs include timestamp, action, duration           |
| BRD-NFR-014  | Observability  | The system shall capture and log AI generation errors with request context.                   | Error logs include prompt summary and error detail |
| BRD-NFR-015  | Maintainability| The codebase shall use Python type hints on all function signatures.                          | All public functions have type annotations         |
| BRD-NFR-016  | Maintainability| Request/response schemas shall be defined as Pydantic v2 models.                             | No raw dict responses in API endpoints             |
| BRD-NFR-017  | Testability    | Every API endpoint and service function shall have corresponding test coverage.               | pytest test exists for each endpoint               |

---

## 8. GitHub Models Integration Requirements

This section captures requirements specific to the platform's use of the GitHub Models API for AI-driven content generation.

| Req ID       | Description                                                                                          | Priority    | Notes                                                        |
|--------------|------------------------------------------------------------------------------------------------------|-------------|--------------------------------------------------------------|
| BRD-INT-001  | The system shall integrate with the GitHub Models API to generate course content (lessons, quiz questions) from admin-provided prompts. | Must-Have | Uses GPT-4o model; endpoint and API key from environment variables. |
| BRD-INT-002  | The system shall store each generation request with prompt, model, requester ID, status, and timestamp. | Must-Have | ContentGenerationRequest entity; status: pending/completed/failed. |
| BRD-INT-003  | The system shall store generated content as draft artifacts linked to their source request.           | Must-Have   | ContentGenerationArtifact entity with generatedContent and sourceRequestId. |
| BRD-INT-004  | The system shall handle GitHub Models API rate limits with exponential backoff and retry logic.       | Must-Have   | Maximum 3 retries with exponential backoff; log each retry attempt. |
| BRD-INT-005  | The system shall return meaningful error responses when AI generation fails, without exposing API keys. | Must-Have | Error response includes user-friendly message; API key never in logs or responses. |
| BRD-INT-006  | The system shall allow admins to regenerate content for a specific lesson or section without regenerating the entire course. | Should-Have | Regeneration creates a new ContentGenerationRequest and ContentGenerationArtifact. |
| BRD-INT-007  | The AI generation service shall be designed as a pluggable module for future MCP integration.        | Should-Have | Service interface abstracted behind a protocol/ABC; implementation is swappable. |

### Integration Considerations

- **Model Selection**: GPT-4o via GitHub Models API is the primary model for all content generation tasks. The model identifier is configured via environment variable `GITHUB_MODELS_ENDPOINT`.
- **Rate Limits & Quotas**: The system expects moderate usage (admin-triggered generation only, not learner-facing). Exponential backoff with a maximum of 3 retries handles transient rate limit responses (HTTP 429).
- **Prompt Management**: Prompts are constructed from admin inputs (topic, audience, objectives, difficulty) combined with system-level prompt templates. Prompts are stored with each ContentGenerationRequest for auditability.
- **Fallback Strategy**: When the GitHub Models API is unavailable, generation requests are marked as "failed" with a retryable status. Admins can retry later. Published courses are served entirely from the database with no AI API dependency at read time.
- **Authentication**: API key is loaded from the `GITHUB_MODELS_API_KEY` environment variable. The key is never logged, displayed in error messages, or stored in source code.

---

## 9. Data Model

### 9.1 Core Entities

| Entity                       | Key Attributes                                                                                     |
|------------------------------|----------------------------------------------------------------------------------------------------|
| **User**                     | id, name, email, role (admin/learner), passwordHash, createdAt                                     |
| **Course**                   | id, title, description, status (draft/published), difficulty, estimatedDuration, tags, createdAt   |
| **Module**                   | id, courseId (FK→Course), title, summary, sortOrder                                                |
| **Lesson**                   | id, moduleId (FK→Module), title, markdownContent, estimatedMinutes, sortOrder                      |
| **QuizQuestion**             | id, moduleId (FK→Module), question, options (JSON array), correctAnswer, explanation               |
| **Enrollment**               | id, userId (FK→User), courseId (FK→Course), enrolledAt, status (not_started/in_progress/completed) |
| **ProgressRecord**           | id, userId (FK→User), lessonId (FK→Lesson), moduleId (FK→Module), completed, completedAt, lastViewedAt |
| **QuizAttempt**              | id, userId (FK→User), quizQuestionId (FK→QuizQuestion), selectedAnswer, isCorrect, attemptedAt    |
| **ContentGenerationRequest** | id, prompt, model, requesterId (FK→User), status (pending/completed/failed), createdAt            |
| **ContentGenerationArtifact**| id, generatedContent, sourceRequestId (FK→ContentGenerationRequest), approvedBy (FK→User), approvedAt |

### 9.2 Relationships

- A Course contains many Modules (one-to-many).
- A Module contains many Lessons and many QuizQuestions (one-to-many).
- A User can have many Enrollments (one-to-many).
- A User can have many ProgressRecords and QuizAttempts (one-to-many).
- A ContentGenerationRequest produces one or more ContentGenerationArtifacts (one-to-many).

### 9.3 Course Hierarchy

```
Course
 └── Module (sortOrder)
      ├── Lesson (sortOrder)
      └── QuizQuestion
```

---

## 10. Risks & Mitigations

| Risk ID | Description                                                              | Likelihood | Impact | Mitigation Strategy                                                                             |
|---------|--------------------------------------------------------------------------|------------|--------|-------------------------------------------------------------------------------------------------|
| R-001   | GitHub Models API becomes unavailable or changes its interface.          | Medium     | High   | Abstract the AI service behind a pluggable interface; implement retry with exponential backoff; published content has no runtime AI dependency. |
| R-002   | AI-generated content is inaccurate or low quality.                       | Medium     | High   | Mandatory admin review before publishing; content labeled as AI-generated; admins can regenerate or manually edit. |
| R-003   | SQLite concurrency limitations under higher-than-expected load.          | Low        | Medium | SQLite is adequate for MVP single-server use; document migration path to PostgreSQL for scale-up. |
| R-004   | XSS vulnerabilities in rendered Markdown lesson content.                 | Medium     | High   | Sanitize all Markdown-to-HTML rendering; strip script tags and event handlers; use established sanitization library. |
| R-005   | API key exposure in logs, error messages, or client responses.           | Low        | High   | Load keys from environment variables; never log or include keys in error responses; code review checks. |
| R-006   | Scope creep beyond MVP boundaries delays delivery.                       | Medium     | Medium | Clearly defined in-scope/out-of-scope boundaries; prioritize Must-Have requirements first.      |
| R-007   | Learner progress data loss due to application errors.                    | Low        | High   | Write progress to database before returning response; implement database transaction handling.   |

---

## 11. Appendix

### 11.1 Glossary

| Term                     | Definition                                                                                              |
|--------------------------|---------------------------------------------------------------------------------------------------------|
| GitHub Models API        | GitHub's hosted AI model inference service providing access to models like GPT-4o for content generation. |
| GitHub Foundations       | A starter course covering fundamental GitHub concepts including repositories, branches, and pull requests. |
| GitHub Actions           | GitHub's CI/CD platform; also a starter course topic covering workflow automation and pipeline configuration. |
| GitHub Advanced Security | GitHub's security feature set including code scanning, secret scanning, and dependency review; also a starter course topic. |
| FastAPI                  | A modern, high-performance Python web framework for building APIs, based on standard Python type hints.  |
| Pydantic                 | A Python data validation library using type annotations; used for request/response schema definitions.   |
| SQLite                   | A self-contained, serverless, file-based SQL database engine bundled with Python.                        |
| Jinja2                   | A Python templating engine used to render HTML pages on the server side.                                 |
| RBAC                     | Role-Based Access Control — restricting system access based on the roles assigned to individual users.   |
| XSS                      | Cross-Site Scripting — a security vulnerability where malicious scripts are injected into web content.   |
| BRD                      | Business Requirements Document — this document defining the project's business and functional requirements. |
| MVP                      | Minimum Viable Product — the version of the product with the minimum feature set to be usable.           |
| MCP                      | Model Context Protocol — a future integration pattern for pluggable AI services.                         |

### 11.2 References

- `.github/copilot-instructions.md` — Project conventions, tech stack, domain model, and coding standards.
- `templates/BRD.md` — BRD document template used as the structural foundation for this document.
- GitHub Models API documentation: https://docs.github.com/en/github-models
- FastAPI documentation: https://fastapi.tiangolo.com/
- Pydantic v2 documentation: https://docs.pydantic.dev/latest/

### 11.3 Requirement Traceability Summary

| ID Range              | Category                  | Count |
|-----------------------|---------------------------|-------|
| BRD-FR-001 – BRD-FR-042  | Functional Requirements   | 42    |
| BRD-NFR-001 – BRD-NFR-017 | Non-Functional Requirements | 17  |
| BRD-INT-001 – BRD-INT-007 | Integration Requirements  | 7     |
| **Total**             |                           | **66** |
