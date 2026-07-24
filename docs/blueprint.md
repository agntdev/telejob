# Job Finder Bot — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A moderated job board for Telegram communities and online roles. Job posters submit listings for moderation; job seekers browse, apply with resume and message; posters review applicants and message via the bot. Moderators approve/reject posts in an admin chat.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- job posters
- job seekers
- community moderators

## Success criteria

- 100 active job listings in public feed
- 100 applications submitted in first month
- 90% moderation queue processed within 24 hours

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with Post job, Browse jobs, My posts, My applications
- **Post job** (button, actor: user, callback: post:start) — Begin job posting form
  - inputs: role type, salary/range, description, tags, location, employment type
  - outputs: moderation queue item
- **Browse jobs** (button, actor: user, callback: browse:start) — View public job feed with filters
  - inputs: role type filter, tag filter
  - outputs: paginated job list
- **My posts** (button, actor: user, callback: my_posts:list) — View user's active job postings
  - inputs: none
  - outputs: job list
- **My applications** (button, actor: user, callback: my_apps:list) — View user's submitted applications
  - inputs: none
  - outputs: application list
- **Report job** (button, actor: user, callback: report:start) — Submit a report for a job listing
  - inputs: report reason, job id
  - outputs: moderation queue item
- **Approve** (button, actor: moderator, callback: moderate:approve) — Approve a moderation queue item
  - inputs: queue item id
  - outputs: approved job listing
- **Reject** (button, actor: moderator, callback: moderate:reject) — Reject a moderation queue item
  - inputs: queue item id, optional feedback
  - outputs: rejected job listing

## Flows

### Job Posting
_Trigger:_ post:start

1. Select role type from pick-list
2. Enter salary/range
3. Add optional description
4. Add tags
5. Add optional location and employment type
6. Preview and submit for moderation

_Data touched:_ job listing, moderation queue item

### Job Moderation
_Trigger:_ moderate:queue

1. Moderator sees new post in admin chat
2. Approve or reject with optional feedback
3. Post appears in feed if approved

_Data touched:_ moderation queue item, job listing

### Job Application
_Trigger:_ apply:start

1. User selects Apply on job listing
2. Write application message
3. Upload resume (PDF/DOC/DOCX)
4. Submit application
5. Poster receives notification with message and resume link

_Data touched:_ application, job listing

### Application Review
_Trigger:_ review:start

1. Poster views applications for a job
2. Message applicant via bot (proxied DM)
3. Mark application status (Accepted/Rejected/Interview)

_Data touched:_ application, job listing

### Reporting
_Trigger:_ report:start

1. User selects Report on job listing
2. Enter report reason
3. Submit to moderation queue
4. Moderator reviews and takes action

_Data touched:_ moderation queue item, job listing

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — User account with poster/seeker roles
  - fields: user id, telegram username, role (poster/seeker)
- **Job Listing** _(retention: persistent)_ — Moderated job posting with application tracking
  - fields: job id, title/role type, salary/range, description, tags, location, employment type, poster id, moderation status, applications
- **Application** _(retention: persistent)_ — Candidate application with message and resume
  - fields: application id, applicant id, message, resume file, timestamp, job id, status
- **Moderation Queue Item** _(retention: persistent)_ — Pending moderation decision with context
  - fields: queue item id, job id, poster id, moderation status, timestamp, moderator feedback

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Admin chat ID for moderation decisions
- Role type pick-list options
- Resume file retention period (default 90 days)

## Notifications

- Posters receive DM when application is submitted
- Applicants receive DM with confirmation of application
- Moderators receive chat alerts for new posts and reports
- Moderators receive feedback when rejecting a post

## Permissions & privacy

- Posters' Telegram usernames are visible in job listings
- Applicants' resumes are stored securely and auto-deleted after 90 days
- Moderators can only access moderation queue items
- All user data is stored only for job posting lifecycle

## Edge cases

- User tries to post without required fields (role type, salary)
- Moderator tries to approve a post that's already approved
- User applies to a job that's been rejected
- User tries to upload unsupported resume file type
- Moderator chat is inactive for extended period

## Required tests

- End-to-end job posting flow from form to moderation approval
- Application submission with resume upload and poster notification
- Moderation queue processing with approve/reject buttons
- User reporting flow to moderation queue
- Resume file retention and auto-deletion after 90 days

## Assumptions

- Moderation will be handled by a single admin chat initially
- Role type pick-list will include 8 common roles plus 'Other' option
- Resume retention is 90 days by default
- Notifications will be sent via Telegram DM and admin chat
