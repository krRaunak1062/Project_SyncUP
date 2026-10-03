# SyncUp: Project Brief

> Paste this whole document at the start of a new AI chat, then ask your question.
> Last updated: 3 Oct 2026. Treat the "Open decisions" section as unresolved.

## 0. How to use this brief (instructions for the AI reading it)

You are acting as a senior engineer and mentor for a 5-person student team that runs this project as a self-organized internship. They want production-level work and real industry practice, not tutorial-style shortcuts. When you answer:

- Respect the decisions in this brief; if you disagree with one, say why instead of silently changing it.
- Prefer simple solutions suited to 5 people. Avoid over-engineering (no microservices, no heavy infrastructure before it is needed).
- Name industry practices where they apply (PR review, tests, CI, migrations, ADRs) and explain briefly why.
- Point out risks and trade-offs. Ask one clarifying question if a decision depends on missing information.
- Do not suggest a code editor, code execution, or specialized group types. These are out of scope for now.

## 1. Product summary

SyncUp is one place where any group (study squads, project teams, friends) can chat, share files, assign and track tasks, plan their days, and meet. It replaces the juggling of Discord/Slack, Zoom/Meet, Notion/Drive and Trello/Jira.

Path: website first, later a dedicated mobile app. Far-future idea (not now): LLM/RAG features, only after the core product is complete.

Primary goals: help groups coordinate efficiently and save time, while the team gains real industry experience in web development and system design.

## 2. Team and working style

- 5 people. Self-run internship; agile throughout.
- Learning goal: web development, system design and low-level engineering as industry practices them.
- Assumption (confirm): each person has roughly 8 to 10 hours per week for the project.

## 3. Feature scope by release

| Release | Features |
|---|---|
| R1: Core | Sign up and log in, profile, create/join groups (invite code), roles, announcements, group chat, calendar with daily notes |
| R2: Collaboration | Resource vault (folders, upload, download), Kanban task board (assignees, deadlines, progress bars), read/unread status in chat |
| R3: Meetings | Join meeting by ID and password, small-group audio/video (WebRTC), scheduled meetings shown in calendar |
| R4: Scale and polish | Meeting recording and transcoding, caching, event queue, monitoring dashboards, mobile app |

### Main screens (from the team's plan)

1. **Auth:** sign up and log in pages first.
2. **Main page** with three sections:
   - **Calendar:** per day, what is done, what is to do, what remains; notes for revision; meeting join by ID and password (Zoom/Teams style).
   - **Groups:** all groups the user joined. Each group has chat, resources, announcements, group meet/call.
   - **Profile:** the user's information.

### Product gaps to cover over time

Email verification and password reset, notifications (in-app first, then email), search within a group, onboarding for new users (empty states), leave group/remove member/report/delete account, accessibility (keyboard, contrast, labels).

## 4. Roles and permissions (proposed)

- **Owner:** everything, including deleting the group.
- **Admin:** manage members, post announcements, assign tasks, manage folders.
- **Member:** chat, upload files, update own tasks, write own calendar notes.

Groups are joined with an invite code.

## 5. Technical decisions (proposed)

- **Architecture:** modular monolith. One backend, split into modules: auth, groups, chat, resources, tasks, calendar. Split into services later only for a concrete reason.
- **Frontend:** Next.js, TypeScript, Tailwind CSS. Global state with Zustand or Redux Toolkit. Optimistic updates, list virtualization for long chat/file lists.
- **Backend:** Node.js with TypeScript (recommended so the whole team shares one language).
- **Data:** PostgreSQL (source of truth), Redis (cache, real-time support), S3-compatible object storage (files, later recordings) using pre-signed URLs.
- **Real-time:** Socket.io for chat and live updates, with auto-reconnect handling. WebRTC with STUN/TURN for calls in R3 (peer-to-peer works for roughly 4 to 6 participants; larger calls would need a media server).
- **Concurrency:** version column on tasks (optimistic locking) or row locks, so two admins editing the same task or card do not overwrite each other.
- **Tooling:** Docker, GitHub Actions CI/CD.
- **Add in R4 or when needed:** Kafka or RabbitMQ (async progress calculation, notifications), Terraform, Prometheus and Grafana, FFmpeg workers for transcoding.

## 6. Data model (first draft)

Entities: `User`, `Squad` (group), `Membership` (user, group, role, joinedAt), `Channel`, `Message` (markdown body), `ReadReceipt`, `Folder` (self-referencing for subfolders), `Resource` (storageKey, mimeType), `Task` (status, deadline, version), `TaskAssignment` (progressPercent), `Meeting` (meeting ID, password, startsAt, status), `Recording` (storageKey, transcode status).

Still to add because of the calendar feature: `CalendarEntry`/`DayNote` (per user per day: done, to do, remaining, notes), `Announcement` (pinned flag), `Notification`.

## 7. Work split (proposed: by feature, not by layer)

1. Foundation, auth and DevOps: repo, CI/CD, environments, login, profile
2. Groups and announcements: groups, roles, invites, announcements
3. Real-time chat: sockets, message storage, later WebRTC
4. Calendar and notes: daily view, notes, scheduled meetings
5. Resources and task board: file vault, Kanban, progress

Each person owns their feature from UI to database. Everyone still builds the frontend for their own feature.

## 8. Agile process

- **Sprint length:** 2 weeks. Sprint 0 is 1 week (setup).
- **Ceremonies:** sprint planning (about 1 hour), short daily standup (15 min, or async in chat on busy days), sprint review plus retrospective (about 1.5 hours).
- **Backlog:** user stories with acceptance criteria in GitHub Projects (or Jira).
- **Definition of Done:** code reviewed, tests passing, merged to `main`, deployed to staging, docs updated.
- **Rotating roles each sprint:** Scrum Master, sprint tech lead, QA owner.
- **Reserve 15 to 20 percent** of each sprint for tech debt and bug fixes.
- Run a time-boxed spike (about 2 days) before estimating anything unknown, such as WebRTC.

## 9. Engineering standards

**Code and review**
- Short-lived feature branches; pull requests with at least one reviewer; no direct pushes to `main`.
- Conventional commit messages. CI on every PR: lint, type-check, tests, build.
- Preview deployments per PR when possible.

**Testing**
- Unit tests for logic, integration tests for API endpoints against a test database, a few end-to-end tests (Playwright) for critical flows. A feature is not done without tests.

**Environments and data**
- Local, staging, production. Secrets never in the repo. Database changes only through versioned migrations. Automated backups and one practice restore.

**Security and reliability**
- Hashed passwords, access/refresh tokens, input validation on every endpoint (e.g., Zod), authorization checks on every group action with explicit tests, rate limiting on login and messaging, dependency and secret scanning.

**Observability**
- Structured logs with request IDs, error tracking (Sentry), health-check endpoint, uptime monitoring. Dashboards later.

**Design and documentation**
- API-first: write the OpenAPI contract before building endpoints. Architecture Decision Records (ADRs) for major choices. UML/schema diagrams stored in `docs/`. README that gets a new person running in under 15 minutes. `CONTRIBUTING.md`. Runbook. Changelog and version tags. One-page design doc before any big feature. Blameless postmortems after incidents. Feature flags so unfinished work can be merged safely.

## 10. Adoption timeline

| When | Adopt |
|---|---|
| Sprint 0 and 1 | Git workflow, PR reviews, CI, README, ADRs, environments, migrations |
| R1 | Unit and integration tests, validation, auth security, staging deploys, Sentry |
| R2 | E2E tests, feature flags, structured logging, notifications, search |
| R3 | WebRTC spike, load tests, runbook |
| R4 | Queue, caching strategy, Prometheus/Grafana, Terraform, mobile app |

Principle: do not adopt everything at once. Each sprint should deliver something usable plus one new professional habit.

## 11. Sprint 0 and Sprint 1 plan (draft)

**Sprint 0 (1 week): setup**
- Create the repo, branch protection, PR template, CI pipeline (lint, types, tests, build).
- Write README, CONTRIBUTING, first ADRs (modular monolith, Node/TypeScript, Postgres).
- Choose design tokens and component approach; create wireframes for auth, main page, groups, profile.
- Set up local Docker environment and a staging environment.

**Sprint 1 (2 weeks): frontend shell with mock data, plus backend groundwork in parallel**

Frontend user stories:
1. As a visitor, I can sign up and log in (forms with validation, error states).
2. As a user, I land on a main page with navigation: Calendar, Groups, Profile.
3. As a user, I see a calendar where I pick a day and view done / to do / remaining items and a notes field.
4. As a user, I can enter a meeting ID and password to join a meeting (form and validation only; no live call yet).
5. As a user, I see a list of my groups; opening one shows tabs for Chat, Resources, Announcements, Meet (placeholders).
6. As a user, I can view my profile page.
7. As a user on a phone or laptop, the layout is responsive and keyboard-accessible.

Backend groundwork (parallel):
- Draft the database schema and first migration.
- Draft the OpenAPI contract for auth and groups.
- Implement register/login endpoints with tests, if time allows.

Sprint 1 goal: a clickable, well-designed shell that the whole team and a few friends can use to give feedback, with the contracts ready so Sprint 2 can connect real data.

## 12. Open decisions

1. Final backend language (recommended: Node.js with TypeScript).
2. ORM/migration tool (Prisma or Drizzle).
3. Hosting for staging and production.
4. Confirm the sprint length (2 weeks), the start date and a target date for R1.
5. Confirm weekly hours per person.
6. Assign owners to the five areas in section 7.
7. Confirm roles and permissions in section 4.
8. Who owns the media pipeline (pre-signed uploads, FFmpeg) in R3/R4; the original charter does not assign it.

## 13. Out of scope for now

Code editor and code execution, specialized group types (DSA, gaming, etc.), LLM/RAG features, native mobile app. Revisit after R4.

## 14. Original charter (summary)

The original charter described five modules: Hub (dashboard with feed, pinned announcements, upcoming meets), Live Chat (multi-channel, markdown, read/unread sync), Resource Vault (folders, upload/stream/download), Sprint Board (Kanban with live progress), Meet Chamber (WebRTC calls plus async recording and transcoding). Constraints: concurrency control on edits, sub-second real-time updates without refresh, caching of frequently read data. It proposed a 7 to 8 engineer team split by layer; this brief adapts it for 5 engineers split by feature.
