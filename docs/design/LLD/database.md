# Low-Level Design (LLD)

| Field                    | Value                                          |
|--------------------------|------------------------------------------------|
| **Title**                | Database Layer — Low-Level Design              |
| **Component**            | Database Layer                                 |
| **Version**              | 1.0                                            |
| **Date**                 | 2026-04-22                                     |
| **Author**               | 2-plan-and-design-agent                        |
| **HLD Component Ref**    | COMP-007                                       |

---

## 1. Component Purpose & Scope

### 1.1 Purpose

The Database Layer provides the persistent data store for all platform entities using SQLite. It includes the complete schema definition, database initialization, connection management, and seed data for the three starter courses. This component ensures data integrity through foreign keys, constraints, and indexes. It satisfies BRD-FR-023, BRD-FR-031–034, and BRD-NFR-009.

### 1.2 Scope

- **Responsible for**: Database schema (all tables), connection pool/management via aiosqlite, database initialization script, starter course seed data, parameterized query utilities.
- **Not responsible for**: Business logic (COMP-001 through COMP-005), API routing, content generation.
- **Interfaces with**: All service components (COMP-001 through COMP-005) use this layer for data persistence.

---

## 2. Detailed Design

### 2.1 Module / Class Structure

```
src/
└── database/
    ├── __init__.py
    ├── connection.py      # Database connection management (aiosqlite)
    ├── schema.py          # SQL schema definitions and initialization
    ├── seed.py            # Starter course seed data
    └── init.py            # Database initialization entry point (create tables + seed)
```

### 2.2 Key Classes & Functions

| Class / Function              | File            | Description                                                | Inputs                    | Outputs                     |
|-------------------------------|------------------|------------------------------------------------------------|---------------------------|-----------------------------|
| `get_db()`                    | connection.py    | FastAPI dependency that provides a database connection      | —                         | aiosqlite connection        |
| `init_db()`                   | connection.py    | Opens a connection and creates tables if not exist          | database_path             | None                        |
| `close_db()`                  | connection.py    | Closes the database connection pool on shutdown             | —                         | None                        |
| `CREATE_TABLES_SQL`           | schema.py        | SQL string with all CREATE TABLE statements                | —                         | SQL string                  |
| `create_tables()`             | schema.py        | Executes the schema SQL against a connection                | db connection             | None                        |
| `seed_starter_courses()`      | seed.py          | Inserts the 3 starter courses with modules, lessons, quizzes | db connection            | None                        |
| `GITHUB_FOUNDATIONS_DATA`     | seed.py          | Dict with GitHub Foundations course structure and content   | —                         | Dict                        |
| `GITHUB_ADV_SECURITY_DATA`    | seed.py          | Dict with GitHub Advanced Security course structure        | —                         | Dict                        |
| `GITHUB_ACTIONS_DATA`         | seed.py          | Dict with GitHub Actions course structure and content      | —                         | Dict                        |

### 2.3 Design Patterns Used

- **Repository / Data Access Layer**: Centralized database access through a connection dependency.
- **FastAPI Dependency Injection**: `get_db()` is a FastAPI `Depends()` callable that provides a connection to route handlers and services.
- **Idempotent Initialization**: `CREATE TABLE IF NOT EXISTS` and seed data checks ensure the database can be safely re-initialized.

---

## 3. Complete Database Schema

### 3.1 Entity-Relationship Diagram

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌────────────────┐
│  users   │     │ courses  │     │ modules  │     │   lessons      │
│──────────│     │──────────│     │──────────│     │────────────────│
│ id (PK)  │     │ id (PK)  │◄────│ course_id│◄────│ module_id (FK) │
│ name     │     │ title    │     │ id (PK)  │     │ id (PK)        │
│ email    │     │ desc     │     │ title    │     │ title          │
│ pass_hash│     │ status   │     │ summary  │     │ markdown_content│
│ role     │     │ difficulty│    │ sort_order│    │ est_minutes    │
│ created  │     │ est_dur  │     └──────────┘     │ sort_order     │
└──────┬───┘     │ tags     │                      └────────────────┘
       │         │ created  │     ┌──────────────┐
       │         └──────────┘     │quiz_questions │
       │                          │──────────────│
       │              ┌───────────│ module_id(FK)│
       │              │           │ id (PK)      │
       │              │           │ question     │
       │              │           │ options      │
       │              │           │ correct_ans  │
       │              │           │ explanation  │
       │              │           └──────────────┘
       │
       │    ┌──────────────┐    ┌──────────────────┐    ┌───────────────┐
       ├────│ enrollments  │    │ progress_records │    │ quiz_attempts │
       │    │──────────────│    │──────────────────│    │───────────────│
       │    │ id (PK)      │    │ id (PK)          │    │ id (PK)       │
       ├────│ user_id (FK) │    │ user_id (FK)     │────│ user_id (FK)  │
       │    │ course_id(FK)│    │ lesson_id (FK)   │    │ quiz_id (FK)  │
       │    │ enrolled_at  │    │ module_id (FK)   │    │ selected_ans  │
       │    │ status       │    │ completed        │    │ is_correct    │
       │    └──────────────┘    │ completed_at     │    │ attempted_at  │
       │                        │ last_viewed_at   │    └───────────────┘
       │                        └──────────────────┘
       │
       │    ┌───────────────────────────┐    ┌────────────────────────────────┐
       └────│content_generation_requests│    │content_generation_artifacts    │
            │───────────────────────────│    │────────────────────────────────│
            │ id (PK)                   │◄───│ source_request_id (FK)         │
            │ prompt                    │    │ id (PK)                        │
            │ model                    │    │ generated_content              │
            │ requester_id (FK→users)  │    │ is_ai_generated               │
            │ status                    │    │ approved_by (FK→users)         │
            │ error_message             │    │ approved_at                    │
            │ created_at                │    │ created_at                     │
            └───────────────────────────┘    └────────────────────────────────┘
```

### 3.2 Complete SQL Schema

```sql
-- ============================================
-- Users
-- ============================================
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL CHECK(role IN ('admin', 'learner')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE UNIQUE INDEX IF NOT EXISTS idx_users_email ON users(email);

-- ============================================
-- Courses
-- ============================================
CREATE TABLE IF NOT EXISTS courses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'draft' CHECK(status IN ('draft', 'published')),
    difficulty TEXT NOT NULL CHECK(difficulty IN ('beginner', 'intermediate', 'advanced')),
    estimated_duration INTEGER NOT NULL,
    tags TEXT DEFAULT '[]',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ============================================
-- Modules
-- ============================================
CREATE TABLE IF NOT EXISTS modules (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    course_id INTEGER NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    summary TEXT NOT NULL,
    sort_order INTEGER NOT NULL DEFAULT 0
);
CREATE INDEX IF NOT EXISTS idx_modules_course_id ON modules(course_id);

-- ============================================
-- Lessons
-- ============================================
CREATE TABLE IF NOT EXISTS lessons (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    module_id INTEGER NOT NULL REFERENCES modules(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    markdown_content TEXT NOT NULL DEFAULT '',
    estimated_minutes INTEGER NOT NULL DEFAULT 5,
    sort_order INTEGER NOT NULL DEFAULT 0
);
CREATE INDEX IF NOT EXISTS idx_lessons_module_id ON lessons(module_id);

-- ============================================
-- Quiz Questions
-- ============================================
CREATE TABLE IF NOT EXISTS quiz_questions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    module_id INTEGER NOT NULL REFERENCES modules(id) ON DELETE CASCADE,
    question TEXT NOT NULL,
    options TEXT NOT NULL DEFAULT '[]',
    correct_answer TEXT NOT NULL,
    explanation TEXT NOT NULL DEFAULT ''
);
CREATE INDEX IF NOT EXISTS idx_quiz_questions_module_id ON quiz_questions(module_id);

-- ============================================
-- Enrollments
-- ============================================
CREATE TABLE IF NOT EXISTS enrollments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL REFERENCES users(id),
    course_id INTEGER NOT NULL REFERENCES courses(id),
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status TEXT NOT NULL DEFAULT 'not_started' CHECK(status IN ('not_started', 'in_progress', 'completed')),
    UNIQUE(user_id, course_id)
);
CREATE INDEX IF NOT EXISTS idx_enrollments_user ON enrollments(user_id);
CREATE INDEX IF NOT EXISTS idx_enrollments_course ON enrollments(course_id);

-- ============================================
-- Progress Records
-- ============================================
CREATE TABLE IF NOT EXISTS progress_records (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL REFERENCES users(id),
    lesson_id INTEGER NOT NULL REFERENCES lessons(id),
    module_id INTEGER NOT NULL REFERENCES modules(id),
    completed BOOLEAN NOT NULL DEFAULT 0,
    completed_at TIMESTAMP,
    last_viewed_at TIMESTAMP,
    UNIQUE(user_id, lesson_id)
);
CREATE INDEX IF NOT EXISTS idx_progress_user ON progress_records(user_id);
CREATE INDEX IF NOT EXISTS idx_progress_lesson ON progress_records(lesson_id);
CREATE INDEX IF NOT EXISTS idx_progress_module ON progress_records(module_id);

-- ============================================
-- Quiz Attempts
-- ============================================
CREATE TABLE IF NOT EXISTS quiz_attempts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL REFERENCES users(id),
    quiz_question_id INTEGER NOT NULL REFERENCES quiz_questions(id),
    selected_answer TEXT NOT NULL,
    is_correct BOOLEAN NOT NULL,
    attempted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX IF NOT EXISTS idx_quiz_attempts_user ON quiz_attempts(user_id);
CREATE INDEX IF NOT EXISTS idx_quiz_attempts_question ON quiz_attempts(quiz_question_id);

-- ============================================
-- Content Generation Requests
-- ============================================
CREATE TABLE IF NOT EXISTS content_generation_requests (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    prompt TEXT NOT NULL,
    model TEXT NOT NULL DEFAULT 'gpt-4o',
    requester_id INTEGER NOT NULL REFERENCES users(id),
    status TEXT NOT NULL DEFAULT 'pending' CHECK(status IN ('pending', 'completed', 'failed')),
    error_message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX IF NOT EXISTS idx_cgr_requester ON content_generation_requests(requester_id);
CREATE INDEX IF NOT EXISTS idx_cgr_status ON content_generation_requests(status);

-- ============================================
-- Content Generation Artifacts
-- ============================================
CREATE TABLE IF NOT EXISTS content_generation_artifacts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    generated_content TEXT NOT NULL,
    source_request_id INTEGER NOT NULL REFERENCES content_generation_requests(id),
    is_ai_generated BOOLEAN NOT NULL DEFAULT 1,
    approved_by INTEGER REFERENCES users(id),
    approved_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX IF NOT EXISTS idx_cga_source ON content_generation_artifacts(source_request_id);
```

---

## 4. Starter Course Seed Data

### 4.1 Seed Data Structure

Each starter course is seeded with the following structure per BRD-FR-031 through BRD-FR-034:

| Course                      | Modules | Lessons per Module | Quiz Questions |
|-----------------------------|---------|---------------------|----------------|
| GitHub Foundations          | 4       | 2–3                 | 3              |
| GitHub Advanced Security    | 3       | 2–3                 | 3              |
| GitHub Actions              | 4       | 2–3                 | 3              |

### 4.2 GitHub Foundations Course

| Module | Title                              | Lessons                                                              |
|--------|------------------------------------|----------------------------------------------------------------------|
| 1      | Introduction to GitHub             | What is GitHub?, Creating Your First Repository                      |
| 2      | Branching and Collaboration        | Understanding Branches, Pull Requests and Code Review, Merge Strategies |
| 3      | Issues and Project Management      | Working with Issues, GitHub Projects and Boards                      |
| 4      | GitHub Settings and Administration | Repository Settings, Managing Access and Permissions                 |

### 4.3 GitHub Advanced Security Course

| Module | Title                              | Lessons                                                              |
|--------|------------------------------------|----------------------------------------------------------------------|
| 1      | Code Scanning                      | Introduction to Code Scanning, Configuring CodeQL, Interpreting Results |
| 2      | Secret Scanning                    | How Secret Scanning Works, Managing Alerts                           |
| 3      | Dependency Review                  | Understanding Dependency Graphs, Dependabot Alerts and Updates       |

### 4.4 GitHub Actions Course

| Module | Title                              | Lessons                                                              |
|--------|------------------------------------|----------------------------------------------------------------------|
| 1      | Introduction to GitHub Actions     | What are GitHub Actions?, Workflow Syntax Basics                     |
| 2      | Building CI Pipelines              | Creating a CI Workflow, Testing and Linting in CI, Build Artifacts   |
| 3      | Advanced Workflow Features         | Matrix Builds and Parallelism, Reusable Workflows                   |
| 4      | Deployment and Environments        | Deploying with GitHub Actions, Environment Secrets and Protection Rules |

---

## 5. Connection Management

### 5.1 Connection Lifecycle

```python
import aiosqlite
from contextlib import asynccontextmanager

DATABASE_PATH = "learning.db"


async def get_db():
    """FastAPI dependency that provides a database connection."""
    db = await aiosqlite.connect(DATABASE_PATH)
    db.row_factory = aiosqlite.Row
    await db.execute("PRAGMA foreign_keys = ON")
    try:
        yield db
    finally:
        await db.close()


@asynccontextmanager
async def lifespan(app):
    """Application lifespan: initialize DB on startup."""
    await init_db()
    yield


async def init_db():
    """Create tables and seed starter courses if database is empty."""
    db = await aiosqlite.connect(DATABASE_PATH)
    await db.execute("PRAGMA foreign_keys = ON")
    await create_tables(db)
    await seed_starter_courses(db)
    await db.commit()
    await db.close()
```

### 5.2 SQLite Configuration

| Pragma                    | Value | Purpose                                          |
|---------------------------|-------|--------------------------------------------------|
| foreign_keys              | ON    | Enforce foreign key constraints                  |
| journal_mode              | WAL   | Write-ahead logging for better concurrent reads  |
| busy_timeout              | 5000  | Wait up to 5 seconds on locked database          |

---

## 6. Error Handling Strategy

### 6.1 Database Errors

| Error Type                  | Handling                                              |
|-----------------------------|-------------------------------------------------------|
| UNIQUE constraint violation | Catch `IntegrityError`, return appropriate 409/400    |
| Foreign key violation       | Catch `IntegrityError`, return 400 with message       |
| Database locked (timeout)   | Catch `OperationalError`, return 503 with retry hint  |
| Connection failure          | Catch on startup; log and exit if database unavailable |

### 6.2 Logging

- **INFO**: Database initialized, tables created, seed data loaded.
- **ERROR**: Schema creation failures, seed data errors, connection failures.

---

## 7. Configuration & Environment Variables

| Variable                  | Description                                    | Required | Default              |
|---------------------------|------------------------------------------------|----------|----------------------|
| DATABASE_URL              | Path to SQLite database file                   | No       | sqlite:///learning.db |

---

## 8. Dependencies

### 8.1 Internal Dependencies

None — this is the foundational layer that other components depend on.

### 8.2 External Dependencies

| Package / Service       | Version           | Purpose                                           |
|-------------------------|-------------------|---------------------------------------------------|
| aiosqlite               | 0.20+             | Async SQLite database access                       |

---

## 9. Traceability

| LLD Element                       | HLD Component  | BRD Requirement(s)                     |
|-----------------------------------|----------------|----------------------------------------|
| users table                       | COMP-007       | BRD-FR-001, BRD-FR-003                |
| courses table                     | COMP-007       | BRD-FR-004, BRD-FR-009                |
| modules table                     | COMP-007       | BRD-FR-011                             |
| lessons table                     | COMP-007       | BRD-FR-013                             |
| quiz_questions table              | COMP-007       | BRD-FR-016                             |
| enrollments table                 | COMP-007       | BRD-FR-019                             |
| progress_records table            | COMP-007       | BRD-FR-020, BRD-FR-023, BRD-NFR-009   |
| quiz_attempts table               | COMP-007       | BRD-FR-018                             |
| content_generation_requests table | COMP-007       | BRD-INT-002                            |
| content_generation_artifacts table | COMP-007      | BRD-INT-003                            |
| Starter course seed data          | COMP-007       | BRD-FR-031, BRD-FR-032, BRD-FR-033, BRD-FR-034 |
| PRAGMA foreign_keys = ON          | COMP-007       | BRD-NFR-009 (data integrity)           |
