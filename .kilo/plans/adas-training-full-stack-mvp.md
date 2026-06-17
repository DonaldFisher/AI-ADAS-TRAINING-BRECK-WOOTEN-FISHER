# ADAS Training Full-Stack MVP Plan

## Goal

Build a usable full-stack MVP for the FTA ADAS Training application described in `plans/FTA ADAS Training Web Application __ Planning document only __ 6 12 26.docx`, using the `UI_Mockup` HTML/screens as the visual baseline.

The MVP should support three roles:

- `FTA_ADMIN`: view/manage all agencies, users, trainings, manuals, and analytics.
- `AGENCY_ADMIN`: upload manuals, generate/manage trainings, assign users, view agency reports.
- `OPERATOR`: complete assigned pre-tests, training slides with embedded quizzes, post-tests, view progress/certificates.

## Current Repository Context

- The repository currently contains only `README.md`, `LICENSE`, the planning `.docx`, and static HTML/image mockups.
- There is no existing application framework, package manager setup, backend, database, or test setup.
- The mockups define six key screens: unified login, agency dashboard/manual upload, admin analytics, operator portal, interactive training module, and post-test assessment quiz.

## Recommended Stack

- Next.js App Router with TypeScript for a single full-stack application.
- Tailwind CSS for styling, aligning with the mockups’ Tailwind/Lexend/Material Symbols visual language.
- Prisma ORM with SQLite for the local MVP database.
- NextAuth/Auth.js Credentials provider for MVP role-based login.
- Local filesystem storage under an ignored `uploads/` directory for PDF manuals.
- `pdf-parse` or equivalent PDF text extraction package for manual ingestion.
- LiteLLM-compatible OpenAI client wrapper for generated extraction/training content, with deterministic mock generation as the default fallback when no API key/base URL is configured.
- `exceljs` or SheetJS for XLSX export of pre-test, embedded quiz, and post-test answers.
- Vitest/React Testing Library for unit/component coverage and Playwright for a small end-to-end smoke suite.

## Architecture

### Application Structure

Create a Next.js project at the repository root:

- `src/app`: App Router pages, layouts, route handlers.
- `src/components`: reusable UI primitives and feature components.
- `src/lib`: auth, database, storage, AI, PDF parsing, exports, sample data.
- `src/server`: server-only workflows such as manual processing and training generation.
- `prisma`: schema, migrations, seed script.
- `public`: static assets and any local placeholder imagery derived from mockup needs.
- `uploads`: runtime manual storage, ignored by git.

### Page Routes

Implement these role-aware routes:

- `/login`: unified login screen based on `UI_Mockup/unified_login_screen`.
- `/dashboard`: role redirector to the correct dashboard.
- `/admin`: FTA admin analytics dashboard based on `UI_Mockup/admin_analytics_dashboard`.
- `/agency`: agency admin dashboard/manual upload/training library based on `UI_Mockup/transit_agency_dashboard`.
- `/operator`: operator learning portal based on `UI_Mockup/end_user_learning_portal`.
- `/training/[assignmentId]`: interactive training player based on `UI_Mockup/interactive_training_module`.
- `/assessment/[assignmentId]/pretest`: pre-test quiz runner.
- `/assessment/[assignmentId]/posttest`: post-test quiz runner based on `UI_Mockup/post-test_assessment_quiz`.

### API Route Handlers

Implement server endpoints or server actions for:

- Auth login/logout/session.
- Manual upload and processing.
- Training generation from extracted manual content.
- Training publish/draft/archive transitions.
- Assignment creation for operators.
- Slide progress save.
- Quiz answer submission with attempt tracking and feedback rules.
- Pre-test/post-test answer submission without feedback.
- Analytics summaries.
- XLSX export for answer matrices and reports.

## Data Model

Use Prisma models equivalent to the following domain objects:

- `Agency`: tenant boundary.
- `User`: role, agency, profile, login identity.
- `Manual`: uploaded PDF metadata, storage path, processing status, extracted text summary.
- `ManualExtraction`: structured ADAS extraction results, raw model response, model metadata.
- `TrainingModule`: generated course for one manual or manual section.
- `AdasFeature`: feature-level grouping, e.g. lane keep assist, emergency braking, blind spot monitoring.
- `Slide`: ordered training, quiz, explanation, pre-test, and post-test content.
- `Question`: question stem, answer choices, correct answer, explanation, source slide/manual reference.
- `Assignment`: user-to-training relationship and status.
- `ProgressEvent`: slide views, started/completed timestamps.
- `AnswerAttempt`: user answer, attempt number, correctness, phase (`PRETEST`, `EMBEDDED`, `POSTTEST`), related slide/question.
- `Certificate`: completion metadata for passed trainings.

Important modeling rules:

- Persist all answers in the database first, then export to XLSX in the row/column format requested by the planning document.
- Treat the database as the source of truth rather than writing directly to an Excel file for each answer.
- Preserve tenant isolation by filtering every agency-scoped query by `agencyId`, except for `FTA_ADMIN` views.

## Training Generation Requirements

Implement generation in a staged pipeline:

1. Upload and store a PDF manual.
2. Extract text from all pages.
3. Identify ADAS features, functionality, ODD, and HMI content.
4. Generate separate feature sections.
5. Generate training slides in this order per feature: functionality, ODD, HMI.
6. Split complex functionality/ODD/HMI material into multiple slides.
7. Generate one embedded quiz after every training slide.
8. Generate four answer options per quiz.
9. For roughly half of generated quizzes, make the fourth option `Other`; when `Other` is used, ensure the first three choices are plausible but incorrect.
10. Generate explanation content for each incorrect answer.
11. Duplicate embedded quiz questions into pre-test and post-test phases without feedback during the assessment flow.

For MVP reliability:

- Add a deterministic sample generator that produces realistic ADAS content when no LiteLLM settings are present.
- Add a LiteLLM client wrapper with a strict JSON schema prompt for future real model calls.
- Store raw extraction/generation JSON for audit/debugging.
- Add validation that rejects malformed generated content before publishing.

## Quiz And Assessment Behavior

Embedded training quizzes:

- Show a training slide, then its quiz slide.
- Record each selected answer.
- If correct on the first or second attempt, show a congratulatory/explanation state and advance to the next training slide.
- If incorrect on the first attempt, show a retry state and present the same quiz again.
- If incorrect on the second attempt, show the correct answer and explanation, then advance.

Pre-test and post-test assessments:

- Use duplicates of all embedded quiz questions.
- Do not show feedback after each answer.
- Record the selected answer and immediately move to the next question.
- Export results with participant rows and columns ordered as: pre-test answers, embedded training quiz answers, post-test answers.

## UI Implementation Plan

Use the mockups as component guidance, not as raw static HTML pasted wholesale.

Create shared UI pieces:

- App shell with role-aware navigation.
- Header/search/profile controls.
- Sidebar navigation.
- Stat cards.
- Tables with status badges.
- Upload dropzone.
- Training cards and progress bars.
- Slide player.
- Quiz choice card.
- Assessment progress rail/grid.
- Export/report controls.

Design requirements:

- Preserve the mockups’ Lexend typography, blue `#137fec` primary color, light/dark-ready palette, rounded cards, and dashboard layout style.
- Make all pages responsive; sidebars collapse or stack on mobile.
- Replace external generated image URLs from mockups with CSS gradients, local placeholders, or stable public assets to avoid broken/hotlinked images.
- Use semantic buttons/forms and accessible labels for quiz answers, upload controls, navigation, and progress states.

## Seed Data

Add a Prisma seed script that creates:

- One FTA admin.
- Two transit agencies.
- One agency admin per agency.
- Several operators.
- One uploaded/manual-like sample record.
- One generated ADAS training with features such as Lane Keep Assist, Automatic Emergency Braking, Blind Spot Monitoring, Adaptive Cruise Control, and HMI Dashboard Interpretation.
- Assignments in not-started, in-progress, and completed states.

Provide visible demo credentials on the login page for local MVP use only.

## Security And Validation

- Enforce role access in middleware and server-side checks.
- Validate form inputs with Zod.
- Restrict uploads to PDF MIME/type and size limit, e.g. 50 MB as shown in mockups.
- Never trust client-side role or agency data.
- Keep uploaded files outside `public` and serve only through authorized handlers if download/review is needed.
- Store password hashes for seeded/demo users, not plaintext passwords in the database.
- Add `.env.example` documenting database/auth/LiteLLM settings.

## Reporting And XLSX Export

Implement agency and admin exports:

- Per-training answer matrix XLSX.
- Agency completion report XLSX/CSV.
- System-level summary CSV for FTA admin.

Answer matrix format:

- Rows: participants/operators.
- Columns: identifying fields, pre-test answers in question order, embedded quiz answers in slide/question order, post-test answers in question order.
- Include correctness and attempt count either as adjacent columns or a second worksheet for scoring details.

## Implementation Phases

### Phase 1: Project Bootstrap

- Initialize Next.js, TypeScript, Tailwind, ESLint, and test tooling.
- Configure app theme tokens to match the mockups.
- Add Prisma/SQLite and initial schema.
- Add seed script and demo credentials.

### Phase 2: Auth And Layouts

- Implement NextAuth/Auth.js credential login.
- Add role-based middleware and dashboard redirect.
- Build shared app shell, sidebar, header, and card/table primitives.
- Implement `/login`, `/admin`, `/agency`, and `/operator` with seeded data.

### Phase 3: Manual Upload And Generation Workflow

- Add agency manual upload UI and API.
- Store PDFs locally and persist manual metadata.
- Add PDF text extraction.
- Add deterministic sample generation and LiteLLM adapter behind one interface.
- Save generated features, slides, questions, and training module records.
- Show processing/draft/active statuses on the agency dashboard.

### Phase 4: Training Player And Assessments

- Implement assignment model and operator assignment listing.
- Build training slide player and embedded quiz flow.
- Implement two-attempt feedback logic.
- Implement pre-test and post-test quiz flows without per-question feedback.
- Persist all answer attempts and progress events.

### Phase 5: Analytics And Export

- Add agency/admin summary queries.
- Implement training completion, average score, and compliance metrics.
- Add XLSX export for answer matrix.
- Add CSV/XLSX export buttons to dashboards.

### Phase 6: Quality Pass

- Add unit tests for quiz attempt rules, role scoping, and export column ordering.
- Add component tests for quiz and upload flows.
- Add Playwright smoke tests for login, dashboard access, training completion path, and export availability.
- Run lint, typecheck, tests, and build.

## Acceptance Criteria

- A local user can run the app, log in as each role, and see role-appropriate screens.
- An agency admin can upload a PDF or use seeded/manual-like sample data to generate a draft training.
- A generated training contains feature-specific training slides, embedded quiz slides, duplicated pre-test/post-test questions, and explanation content.
- An operator can complete pre-test, training slides with embedded quizzes, and post-test.
- Embedded quiz attempts follow the requested retry/correct-answer behavior.
- All answers are stored and exportable to XLSX with participant rows and ordered answer columns.
- FTA admin can view cross-agency analytics; agency admin can view only their agency.
- The UI visually aligns with the supplied mockups and works on desktop and mobile.

## Open Decisions For Implementation

- Whether the MVP must use real LiteLLM calls immediately or can default to deterministic generation until credentials are provided.
- Whether deployment target should be Vercel, a federal/cloud VM, Docker, or local-only for the MVP.
- Whether certificates need downloadable PDFs in the MVP or can be represented as completion records first.
