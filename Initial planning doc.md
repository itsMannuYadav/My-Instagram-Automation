# PROJECT: My Library App Studio — Internal Instagram Content & Reel Publishing Platform

## 1. Your role

Act as a senior full-stack engineer, software architect, DevOps engineer, security reviewer, and UI/UX designer.

Build a real, secure, maintainable, production-minded internal web application for **My Library App (MLA)**. The application will be used exclusively by our internal content team to prepare, review, schedule, and publish Instagram content.

Do not create a generic social media dashboard with fake functionality. Build a working application that integrates with Meta's official Instagram APIs wherever the required functionality is officially supported.

I care about correctness, reliability, security, simplicity, and low operating costs more than flashy features.

**Most important rule: Never pretend a feature works when it has not been implemented, tested, or confirmed to be supported by the official API.**

If a feature is unsupported, explain the limitation and provide a practical manual fallback.

## 2. The actual problem we are solving

My Library App has an internal content creation team. Team members prepare videos and creative assets, but I currently have to handle the final publishing process manually.

For every Reel, I often need to:

- Upload the video.
- Choose and upload the cover image.
- Write or review the caption.
- Add hashtags and account mentions.
- Identify people who need to be tagged.
- Add collaborator invitations when applicable.
- Select the publishing date and time.
- Publish the Reel manually at the correct time.
- Verify that it was published successfully.

This is becoming a bottleneck because I may be attending meetings, travelling, or handling other responsibilities when a post is scheduled to go live.

For example, we might have four finished videos waiting to be published at different times. The platform must let us prepare all four in advance and automate the supported publishing steps.

### Our desired workflow

1. A team member uploads a video and its cover.
2. They enter the caption, hashtags, mentions, and other relevant details.
3. They choose the intended publishing date, time, and timezone.
4. The post enters the approval queue.
5. An authorised reviewer approves or rejects it.
6. Approved posts are published automatically at their scheduled times.
7. The system records the outcome and alerts the team if something goes wrong.

The goal is to remove repetitive manual work, not to recreate the entire Instagram application.

## 3. Start by inspecting the existing repository

Before writing code, inspect the existing Git repository and understand its structure.

The existing MLA web project has previously used a Next.js application, TypeScript, Tailwind CSS, Vercel, and Supabase. The repository may have a `web/` application directory, along with `supabase/` and `docs/` directories.

Do not assume this structure is still unchanged. Inspect the actual repository.

First:

- Identify the framework and installed versions.
- Inspect `package.json`, existing routes, authentication, UI components, and configuration.
- Determine whether Supabase is already configured.
- Inspect existing database migrations and reusable components.
- Identify deployment settings and environment variables.
- Check whether any existing functionality would be affected by your changes.

Then create a short implementation plan.

**Do not replace the existing project, delete working features, overwrite configuration blindly, or introduce unnecessary dependencies.** Preserve compatible existing conventions.

If the repository is empty or the existing project is unrelated, explain that and propose the appropriate structure before proceeding.

## 4. Required technology stack

Use the following stack unless the repository or verified technical constraints make a change necessary.

### Frontend

- Next.js with App Router, using the repository's compatible version.
- TypeScript in strict mode.
- Tailwind CSS.
- Reusable, accessible UI components.
- Lucide icons if an icon library is needed.

### Hosting

- Vercel for the web application.
- Prefer the Vercel Hobby plan during development, provided the usage and terms of the plan permit this use case.

Do not rely on local filesystem persistence. Vercel functions are not a permanent file-storage solution.

### Backend and database

Use Supabase for:

- Authentication.
- PostgreSQL database.
- Row Level Security (RLS).
- Video and image storage.
- Secure background processing and scheduled job coordination.
- Publishing logs and audit history.

Use Supabase Cron and Edge Functions for the publishing scheduler where appropriate.

### Instagram integration

Use Meta's official Instagram API and the currently supported authentication and content-publishing flows.

Do not use:

- Instagram password-based login.
- Browser automation or headless browsers to operate Instagram.
- Unofficial Instagram APIs or scraping.
- Cookie extraction or session hijacking.
- Artificial delays intended to evade platform restrictions.
- A third-party scheduling service unless explicitly approved later.

### Cost constraints

Our initial objective is to keep recurring infrastructure costs at zero or as low as realistically possible.

Do not add Firebase, Redis, a separate backend server, a paid scheduler, or other infrastructure without demonstrating why it is necessary.

Supabase Storage is the initial candidate for video and image assets. Verify the current Free plan limits, including storage, egress, database size, Edge Function invocations, and any relevant restrictions.

Do not assume free quotas are unlimited or permanent.

If the expected video storage exceeds the free quota, explain the options before adding paid services.

## 5. Mandatory official documentation research

Before implementing Instagram integration, verify the current documentation available on the implementation date.

Use official documentation as the source of truth. Community discussions can help identify practical problems, but must not override official API specifications.

Start with these resources:

1. Instagram API documentation:
   https://www.postman.com/meta/instagram/documentation/6yqw8pt/instagram-api

2. Meta's Instagram developer documentation:
   https://developers.facebook.com/docs/instagram-platform/

3. Supabase documentation:
   https://supabase.com/docs/

4. Supabase scheduled Edge Functions:
   https://supabase.com/docs/guides/functions/schedule-functions

5. Supabase Cron:
   https://supabase.com/docs/guides/cron

6. Supabase Storage:
   https://supabase.com/docs/guides/storage

7. Vercel documentation:
   https://vercel.com/docs

8. Vercel Cron Jobs:
   https://vercel.com/docs/cron-jobs

Check the actual current pages, not just the URL or a search-result snippet.

Verify the following Instagram integration details:

- Whether the Instagram account must be a Business or Creator account.
- Whether the chosen authentication flow requires a linked Facebook Page.
- The currently supported permissions and OAuth flow.
- Requirements for app configuration, testing, access levels, and App Review.
- The current media-container creation and publishing workflow for Reels.
- Supported video formats, file limits, dimensions, duration, and codecs.
- Requirements for hosting video URLs so Meta can retrieve the media.
- How to set a custom Reel cover.
- Whether the required tagging and collaborator functionality is supported by the selected API.
- Current publishing limits and applicable restrictions.
- Token lifetime, refresh or renewal behaviour, and revocation handling.

For each important claim, provide a direct official documentation link and identify any unresolved uncertainty.

If you cannot access or verify a document, state that explicitly. Do not invent documentation, permissions, endpoints, request parameters, or API capabilities.

## 6. Authentication and account connection

Build an appropriate authentication flow for the internal application.

Requirements:

- Only authorised team members can access the dashboard.
- Use Supabase Auth.
- Provide secure login and logout.
- Include appropriate session handling and protected routes.
- Restrict administrative actions to authorised users.
- Provide a secure workflow for connecting the Instagram professional account through Meta OAuth.

Never ask users to enter their Instagram password into our website.

Do not hardcode access tokens or client secrets.

Store Meta credentials and long-lived tokens securely on the server side. Prefer an appropriate secret-management mechanism and encryption at rest when storing account-specific credentials in the database.

If using Supabase Vault or another secure secret store, verify its availability and suitability for the selected plan and deployment.

Never expose tokens through:

- Client-side JavaScript.
- `NEXT_PUBLIC_` environment variables.
- API responses.
- Browser logs.
- Error messages.
- Git commits.

Handle expired, revoked, or invalid tokens gracefully. Provide an authorised reconnect workflow.

The interface must clearly display the connected Instagram account and its connection status.

## 7. User roles and approval workflow

Implement a simple role-based workflow.

### Admin / owner

The owner can:

- Connect or reconnect the Instagram account.
- Invite or authorise internal users.
- Review all drafts.
- Approve or reject content.
- Schedule approved posts.
- Edit scheduled posts before publishing.
- Cancel scheduled posts.
- View publishing history and failures.
- Manage application settings.

### Content team

Team members can:

- Upload video files and cover images.
- Create and edit drafts.
- Write captions and hashtags.
- Add account mentions.
- Suggest tagging or collaborator details.
- Submit drafts for approval.
- View the status of their submissions.

By default, team members must not be able to bypass approval or modify the Instagram account connection.

Use server-side authorisation checks and RLS policies. Hiding a button in the UI is not sufficient security.

The owner should be able to configure whether approval is mandatory.

## 8. Main application screens

Build the following screens with a consistent, polished UI.

### A. Dashboard

Display:

- Total drafts.
- Posts awaiting approval.
- Scheduled posts.
- Successfully published posts.
- Failed posts requiring attention.
- The next scheduled post.
- A compact upcoming publishing calendar.
- Recent activity.

Use actual database data. Do not display invented statistics.

### B. Content library

Create a searchable list of content items.

Include filters for:

- Draft.
- Awaiting approval.
- Approved.
- Scheduled.
- Publishing.
- Published.
- Failed.
- Cancelled.

Allow the team to search by title and filter by scheduled date.

Show a video thumbnail or preview when available.

### C. Create / edit post

This is the most important screen.

Include:

**Media**
- Video upload.
- Cover-image upload.
- Video preview.
- Cover-image preview.
- File validation.
- Upload progress.
- Clear error messages.
- Replace or remove media before scheduling.

**Post details**
- Internal post title.
- Caption editor.
- Hashtag entry.
- Account mention entry.
- Character count where appropriate.
- Scheduled date.
- Scheduled time.
- Explicit timezone selection, defaulting to `Asia/Kolkata`.
- Optional internal notes.

**Tagging information**
- Separate fields for caption mentions, actual supported people tags, and Instagram Collab invitations.
- Clearly distinguish a text mention from a genuine Instagram tag or collaborator invitation.
- Only enable automatic operations verified to be supported by the chosen official API.
- If a feature is unsupported, show a manual checklist and explain the limitation.

**Actions**
- Save draft.
- Preview.
- Submit for approval.
- Approve, where authorised.
- Schedule.
- Reschedule.
- Cancel scheduling.

Never mark a post as scheduled or published merely because a button was clicked.

### D. Post preview

Display a preview resembling the intended Instagram Reel presentation.

Include:

- Cover image.
- Video preview.
- Caption.
- Hashtags.
- Mentions.
- Planned date and time.
- Tagging and collaborator checklist.
- Approval status.

Label it as an application preview, not an exact reproduction of Instagram's interface.

### E. Calendar

Provide a calendar or equivalent date-based view of upcoming posts.

Users should be able to open a scheduled item and inspect its details.

If drag-and-drop rescheduling is implemented, validate permissions, timezones, and the post's current status on the server.

### F. Publishing history

For every publishing attempt, record:

- Content item ID.
- Scheduled time.
- Actual attempt time.
- Publishing status.
- Instagram media ID, when available.
- Attempt count.
- Relevant error category.
- Safe, sanitised diagnostic details.
- Next retry time, if applicable.

Never store secrets or unnecessarily large raw API responses in logs.

### G. Settings

Include:

- Connected account information.
- Connection status.
- Timezone.
- Approval requirements.
- Storage usage where available.
- Basic team access management.
- Reconnection guidance.
- A clear account-disconnection workflow.

## 9. Instagram Reel publishing implementation

Implement the publishing flow based on the currently documented Meta API.

At a high level, the workflow should:

1. Confirm that the post is approved and due for publishing.
2. Validate the media and required metadata.
3. Confirm that the connected account has valid authorisation.
4. Ensure that Meta can access the uploaded media through a supported URL.
5. Create the appropriate media container.
6. Check the container's processing status as required by the API.
7. Publish only when the media is ready.
8. Record the resulting Instagram media ID and publication status.
9. Notify the appropriate team members of success or failure.

Do not assume that a media-container creation request publishes the Reel immediately.

Do not publish a Reel before its scheduled time.

Do not assume that Meta accepts arbitrary signed URLs, local file paths, or private Supabase URLs. Verify the exact media retrieval requirements.

If signed URLs are used, ensure their validity period is long enough for media retrieval and processing. Generate them server-side and never expose storage credentials.

The publishing worker must only publish approved posts belonging to the connected account.

### Cover images

Support a custom Reel cover only if the selected API flow currently supports it.

Verify the accepted cover parameter and image requirements.

If cover upload or selection cannot be automated in the chosen flow, explain the limitation and provide a manual fallback. Do not silently publish using an unintended cover.

### Captions, hashtags, and mentions

Store caption text and hashtags as editable content.

Build the caption according to the supported API format.

Do not invent API parameters for account mentions, people tags, location tags, product tags, or collaborator invitations.

Before implementing any of these operations, verify the current official API documentation.

### Collaborator invitations and people tagging

Treat the following as different capabilities:

- Writing `@username` in a caption.
- Adding an actual clickable people tag.
- Inviting another account as a Reel collaborator.
- Adding a location tag.

Create a capability matrix that records what is supported, what permissions are required, and what needs to be done manually.

If a feature is not available through the chosen API, do not use browser automation to bypass that limitation.

The rest of the publishing workflow must remain usable even if some tags need manual completion.

## 10. Scheduling and background processing

The scheduler is a core requirement.

Do not rely on an open browser tab, client-side JavaScript timer, laptop, or the owner being logged in.

Do not use a Vercel Hobby cron job as the sole minute-level scheduling mechanism. Verify the current limitations of the chosen hosting plan.

Prefer Supabase Cron invoking a secure Edge Function or another suitable server-side scheduler.

Implement the scheduler with these principles:

### A. Polling

Use an appropriate recurring interval to find posts whose scheduled time has arrived.

A one-minute polling interval is a reasonable starting point if supported by the chosen plan and within its quotas.

The application should aim to start publishing near the requested minute, while acknowledging that network delays, media processing, rate limits, and third-party service availability can delay actual publication.

### B. Atomic job claiming

Prevent two scheduler executions from claiming and publishing the same post.

Use a database transaction, conditional update, row lock, or another appropriate atomic claiming mechanism.

Use a unique job or attempt identifier where appropriate.

### C. Idempotency and duplicate prevention

Design the system to prevent accidental duplicate publishing.

Do not assume the Meta publishing endpoint provides an idempotency guarantee unless that behaviour is documented.

If a publishing request times out after being submitted, the system must not blindly submit another publish request without reconciling the uncertain outcome.

Implement a safe recovery procedure for ambiguous publishing results.

### D. Retries

Retry only failures that are safe to retry.

Use bounded exponential backoff for appropriate transient failures.

Do not retry indefinitely or bypass Meta rate limits.

Treat authentication errors, invalid media, unsupported operations, and permission errors differently from transient network errors.

### E. Timezones

Store scheduled timestamps in UTC using an appropriate PostgreSQL timestamp type.

Store the intended timezone separately when needed.

Display the scheduled time in the selected timezone, initially `Asia/Kolkata`.

Handle daylight-saving transitions correctly for other timezones.

Do not rely on the browser's timezone alone.

### F. Publishing states

Implement an explicit state machine. At minimum, support:

- `draft`
- `pending_approval`
- `approved`
- `scheduled`
- `publishing`
- `published`
- `failed`
- `cancelled`

Define valid transitions and enforce them server-side.

Do not allow a failed or published post to be accidentally processed as an ordinary scheduled post.

### G. Failure notifications

Show clear alerts in the dashboard when publishing fails.

Include enough information to diagnose the issue without exposing secrets.

Where practical, offer an email notification mechanism that can be configured later.

## 11. Database design

Create database migrations rather than relying on undocumented manual database changes.

Propose and implement a suitable schema covering:

- Internal users and roles.
- Instagram account connections.
- Content items.
- Media assets.
- Scheduled publishing jobs.
- Publishing attempts and logs.
- Approval history.
- Application settings.
- Audit events.

You may use fewer or additional tables if justified by the actual implementation.

Use foreign keys, indexes, constraints, and sensible deletion rules.

For the publishing queue, index the fields used to identify due jobs and filter by status.

Enforce valid state transitions and ownership rules where appropriate.

Prevent unauthorised users from reading or modifying another user's restricted data.

Use RLS policies that match the intended role model.

Provide a safe development seed process that creates example data without inserting fake Instagram credentials or production tokens.

Do not put service-role keys in the browser.

## 12. Video and image storage

Use Supabase Storage initially.

Implement:

- Separate, appropriately secured storage paths or buckets for original media and cover images.
- File type and size validation.
- Upload progress.
- Clear storage errors.
- Safe file naming.
- Appropriate access policies.
- A strategy for deleting unused temporary assets.
- Cleanup rules that never delete assets referenced by active scheduled jobs.
- A configurable upload limit based on verified plan limits and API requirements.

Do not assume the Free plan can store unlimited videos.

Do not assume a video is small enough for every upload mechanism. Verify the limits of the selected SDK, browser upload method, storage service, and Meta API.

For large uploads, implement an appropriate resumable or multipart upload approach if supported and necessary.

Use short-lived signed URLs when appropriate for media retrieval. Never expose the Supabase service-role key.

Avoid unnecessary duplication of original videos.

Where possible, clean up obsolete assets after publication according to a configurable retention policy. Do not delete the only copy of a video while it may still be needed for retry or recovery.

Show storage usage and warn the owner before approaching configured limits.

## 13. Security requirements

Treat this as a private internal application, not a public social media service.

Implement:

- Server-side session validation.
- Role-based authorisation.
- Supabase RLS policies.
- Secure OAuth state validation and redirect handling.
- CSRF protection where applicable.
- Input validation and output sanitisation.
- Secure secret management.
- Safe error messages.
- Rate limiting for sensitive application endpoints where appropriate.
- Audit logging for important actions.
- File validation and upload restrictions.
- Secure logout and account disconnection.

Never commit `.env` files, access tokens, client secrets, or private credentials.

Add an appropriate `.gitignore` if one does not exist.

Create `.env.example` containing only placeholder values.

Document which environment variables belong in Vercel and which belong in Supabase. Do not expose server-only secrets through `NEXT_PUBLIC_` variables.

## 14. User interface and design direction

The interface should feel like a polished internal product used by a modern startup team.

Brand: My Library App.

Use MLA's orange branding as the primary accent. Keep the overall layout clean, professional, and easy to navigate.

Prioritise:

- A clear sidebar or compact navigation.
- A dashboard that surfaces actionable information.
- An intuitive Reel creation workflow.
- A visual content calendar.
- Clear approval and publishing states.
- Useful loading, empty, and error states.
- Responsive layouts for laptop, tablet, and mobile.
- Accessible form labels and keyboard navigation.
- Consistent spacing, typography, buttons, and form controls.

Do not overdesign the application with excessive gradients, animations, or unnecessary dashboard cards.

Avoid fake live metrics, decorative features that do nothing, and placeholder buttons that appear functional.

Every interactive element must have a working action or be clearly marked as unavailable.

## 15. Testing requirements

Do not declare the project complete merely because it compiles.

Create meaningful tests for:

- Authentication and authorisation.
- Role restrictions.
- Database validation and state transitions.
- RLS policies.
- Media upload validation.
- Scheduled-time calculations.
- Timezone conversion.
- Atomic job claiming.
- Duplicate prevention.
- Retry behaviour.
- Failure recovery.
- OAuth state validation.
- Token-expiration handling.
- API error handling.

Use mocks for external Meta API calls in automated tests.

Do not publish real Instagram content from an automated test.

Provide a documented manual integration test that can be performed using an authorised test or production account after the necessary permissions are configured.

Test the following scenario:

1. Create a draft Reel.
2. Upload a video and cover.
3. Enter a caption and hashtags.
4. Submit for approval.
5. Approve the Reel.
6. Schedule it a few minutes in the future.
7. Confirm that the scheduler detects it.
8. Confirm that media processing is checked.
9. Confirm that the Reel is published only once.
10. Confirm that the resulting media ID and status are recorded.
11. Test a simulated API failure.
12. Confirm that an ambiguous publishing outcome does not trigger a duplicate post.

If any step cannot be tested because Meta credentials or account permissions are unavailable, document the exact missing prerequisite. Do not fabricate a successful test result.

## 16. Deployment and setup documentation

Prepare a complete setup guide for a developer with limited time.

Include:

- Required software and compatible versions.
- Repository setup.
- Dependency installation.
- Supabase project configuration.
- SQL migration execution.
- Storage bucket setup.
- RLS policies.
- Authentication setup.
- Meta developer app configuration.
- OAuth redirect URI configuration.
- Required permissions and access requirements.
- Environment variables.
- Local development instructions.
- Vercel deployment instructions.
- Supabase Cron configuration.
- Edge Function deployment.
- Secret configuration.
- Test procedures.
- Troubleshooting.
- Cost and quota considerations.

If the existing application is deployed from a monorepo, preserve the appropriate Vercel Root Directory rather than changing it blindly.

Explain any manual dashboard configuration that cannot be automated safely.

Do not assume that a deployment is complete just because the build succeeds.

## 17. Cost and operational limits

Create a short cost analysis based on the actual current official pricing documentation.

Estimate the expected requirements for a small internal team publishing a few Reels per week.

Consider:

- Storage consumed by original videos and covers.
- Bandwidth and egress.
- Database size.
- Edge Function invocations.
- Cron execution.
- Email notifications, if added.
- Vercel usage.
- Meta developer requirements and any potential costs.

Clearly separate:

1. Features that can operate within current free allowances.
2. Features that require manual setup but no additional paid service.
3. Features that may require a paid plan or additional service.
4. Usage limits that could interrupt publishing.

Do not claim that the whole system will remain free indefinitely.

Prefer a simple architecture that can be upgraded later without a rewrite.

## 18. Implementation strategy

Work incrementally and keep the project deployable.

### Phase 1: Repository audit

Inspect the existing code and report the findings, constraints, and implementation plan.

### Phase 2: Foundation

Implement authentication, protected routes, database migrations, RLS, storage configuration, and the core application layout.

### Phase 3: Content management

Implement drafts, video and cover uploads, captions, metadata, previews, and the approval workflow.

### Phase 4: Meta integration

Implement the documented OAuth flow, connection status, media-container creation, processing checks, and publishing.

### Phase 5: Scheduler

Implement secure scheduled processing, atomic job claiming, state transitions, duplicate prevention, retries, and failure reporting.

### Phase 6: Testing and deployment

Run checks, fix errors, document setup, and perform the available integration tests.

After each phase:

- Explain what was implemented.
- List the important files changed.
- Run relevant tests and type checks.
- Report any failures honestly.
- Identify any manual setup still required.

Do not dump an enormous amount of untested code into the repository in one step.

Do not leave essential functionality as TODO comments while claiming it is finished.

If you cannot complete a feature, clearly distinguish working code from scaffolding.

## 19. Required deliverables

At the end, provide:

1. A working Next.js application integrated with Supabase.
2. Database migrations and appropriate RLS policies.
3. Secure Instagram account connection and server-side publishing integration, subject to required Meta access.
4. A usable content creation and approval workflow.
5. A scheduled publishing queue.
6. Publishing history and failure handling.
7. Automated tests and documented manual integration tests.
8. `.env.example` with placeholder values only.
9. A setup and deployment README.
10. A list of current limitations, outstanding tasks, and verified API restrictions.
11. A realistic cost and quota estimate.
12. Evidence of the checks and tests actually performed.

## 20. Final engineering rules

These rules override any temptation to make the project look more complete than it is.

- Use official, currently supported APIs.
- Verify API capabilities before implementing them.
- Never invent endpoints, permissions, or request parameters.
- Never expose credentials.
- Never rely on the browser being open to execute a scheduled job.
- Never blindly retry an uncertain publishing request.
- Never claim an external API operation succeeded without verifying its result.
- Never promise exact-second publishing.
- Never assume unsupported Instagram tags or collaborator invitations can be automated.
- Never add paid dependencies without explaining the need.
- Never break existing MLA code unnecessarily.
- Never claim tests passed unless they were actually run.

**Start now by auditing the existing repository and current official documentation. Then implement the project incrementally, prioritising a reliable end-to-end Reel publishing workflow over optional features.**