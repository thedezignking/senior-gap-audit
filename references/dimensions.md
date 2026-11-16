# Audit dimensions

Twelve dimensions. For each: what senior work looks like, the gaps AI-built apps usually have, and how to check. Adapt to the stack; the questions transfer even when the tools differ.

Contents:
1. Security and access control
2. Data model and integrity
3. Architecture and boundaries
4. Error handling and resilience
5. Testing and verification
6. Code hygiene and consistency
7. Performance
8. Operations and delivery
9. Dependencies and configuration
10. UX states and accessibility
11. Docs and developer experience
12. Product and scope judgement
Scoring anchors (at the end)

---

## 1. Security and access control

**Senior work:** every server entry point checks who the caller is and whether they may touch *this* record (not just "is logged in"). Authorisation is enforced on the server or in the database, never only in the UI. Secrets live in env vars and never reach the client bundle. Input is validated at the boundary with a schema. Rate limits on auth, signup, password reset and anything expensive (AI calls, email sends). Security headers set. Webhooks verify signatures.

**Common gaps:**
- IDOR: `GET /api/invoices/:id` returns any invoice to any logged-in user.
- Auth checked in middleware or the page, but the API route or server action behind it is open.
- Service-role / admin DB keys used in request handlers "because RLS was in the way".
- Supabase/Firebase rules disabled or `using (true)`.
- Secrets in the repo, in `NEXT_PUBLIC_*`/`VITE_*` vars, or logged.
- No rate limiting on login, signup, OTP, AI endpoints.
- Webhook handlers that trust the payload without verifying the signature.
- File uploads with no type/size limits, served from the same origin.
- `dangerouslySetInnerHTML` / raw HTML from user content; string-built SQL.

**How to check:** list every route handler, server action, API endpoint and job; for each, find the auth check and the ownership check. Grep for admin/service keys and where they're imported. Read DB policies/rules. Grep the repo and git history for keys (`sk_`, `-----BEGIN`, `api_key`, `.env` committed). Check client-exposed env prefixes. Check webhook handlers for signature verification.

## 2. Data model and integrity

**Senior work:** schema changes go through versioned migrations in the repo. Invariants are DB constraints: `not null`, `unique`, foreign keys with deliberate `on delete`, `check`s for enums and ranges. Multi-step writes that must succeed together use a transaction. Money is integer minor units or decimal, never float. Timestamps are timezone-aware. Indexes exist for the queries the app runs. Deletion is a decision (soft delete, cascade, or retention), not an accident.

**Common gaps:** schema edited by hand in a dashboard with no migration history; uniqueness enforced only in app code (race → duplicates); money as float; orphaned rows because FKs are missing; partial writes when step 2 of 3 fails; `text` columns holding JSON blobs that should be tables; no indexes beyond primary keys.

**How to check:** read the migrations folder (exists? ordered? matches the live schema per generated types?). Scan core tables for constraints and FKs. Find multi-write operations (create order + line items + payment) and check for a transaction or a single RPC. Grep for money fields and their types.

## 3. Architecture and boundaries

**Senior work:** a clear, boring structure someone can learn in an hour: where reads live, where writes live, where validation lives, where UI lives. Business logic isn't inside UI components. Server-only code can't be imported into the client. Third-party services sit behind a small module, not called ad hoc from twenty files. Files and functions are sized for reading.

**Common gaps:** the same fetch/validation logic copy-pasted per page; 800+ line components mixing data fetching, business rules and markup; three state libraries or three styling approaches; direct DB/API calls from client components; a `utils.ts` that's become a junk drawer; abstractions with one caller (over-engineering) next to duplication with ten (under-engineering).

**How to check:** read the folder structure and 3–4 representative features end to end (UI → handler → DB). Note how many different patterns solve the same problem. List the largest files. Check for server-only guards.

## 4. Error handling and resilience

**Senior work:** errors are handled where something useful can be done, and otherwise propagate to one place that logs them and shows the user a sane message. No raw DB/stack errors reach the client. External calls have timeouts; retries only where safe (idempotent). Webhooks and jobs are idempotent (a duplicate delivery doesn't double-charge or double-email). Background work runs in a queue/job system, not fire-and-forget promises in a request.

**Common gaps:** `try { ... } catch (e) { console.log(e) }` that swallows failures; `catch` returning `null` so callers can't tell "not found" from "broken"; error messages leaking SQL; no error boundary/500 page; emails sent inline in a request handler; webhook handlers that aren't idempotent; unawaited promises.

**How to check:** grep `catch` blocks and read what they do. Find every external call (payments, email, AI, storage) and check timeout/retry/idempotency. Check how a failed server action surfaces to the user. Look for an error-tracking SDK (Sentry etc.) and whether it's initialised.

## 5. Testing and verification

**Senior work:** tests where bugs would be costly: auth/permissions, money, core business rules, data transformations. A few end-to-end tests for the main journeys. Tests actually pass right now. CI runs type check, lint and tests on every push and blocks merges on failure. Permission rules are tested (policy tests), not just assumed.

**Common gaps:** no tests; tests generated once and never run (they fail); only snapshot tests or tests that assert mocks were called; no CI, or CI that doesn't block; type errors suppressed with `any`/`@ts-ignore`; tests that hit the production database.

**How to check:** find the test setup and count tests by area. Run the suite and the type check; report real results. Read 2–3 tests: do they assert behaviour or just exercise code? Read the CI config (exists? what runs? required?). Grep `any`, `@ts-ignore`, `@ts-expect-error`, `eslint-disable`.

## 6. Code hygiene and consistency

**Senior work:** code reads like one careful person wrote it. Consistent naming, formatting enforced by a formatter, lint clean. No dead code, unused exports, commented-out blocks or debug logs. Comments explain *why*, not what. Small, purposeful commits with clear messages.

**Common gaps:** see `vibecode-tells.md`; most tells live here. Also: no formatter/linter config; lint errors ignored; giant commits like "updates" or "fix"; files named `new-`, `-v2`, `-old`, `-final`, `-copy`.

**How to check:** run the linter and formatter check. Grep for `console.log`, `TODO`, `FIXME`, `debugger`, commented-out code. Look for unused files/exports (tools like `knip`, `ts-prune`, `vulture` if available). Read `git log --oneline` for the last 50 commits.

## 7. Performance

**Senior work:** lists are paginated; queries select what they need; no N+1 (a query per row in a loop); obvious indexes exist; images are sized and optimised; client bundles don't ship server libraries; expensive work is cached or moved to jobs. No premature optimisation either.

**Common gaps:** `select *` of whole tables into the client; a query inside `.map()`; fetching all rows to count them; waterfalls of sequential awaits that could be parallel; huge client bundles (whole icon libraries, moment.js); unoptimised images; no loading states masking slow pages.

**How to check:** read list pages and their queries. Grep for awaits inside loops. Check for `Promise.all` where independent. Check bundle analysis if available, and image handling.

## 8. Operations and delivery

**Senior work:** separate environments (local/dev, staging or preview, production) with separate databases and keys. Deploys are automated from the main branch after CI passes. Migrations are applied by a repeatable process, reviewed before production. Error tracking and basic logging are live. Backups exist and someone knows how to restore. Secrets rotate without code changes. There's a short runbook for "the site is down" and "we leaked a key".

**Common gaps:** one database for dev and production; deploying from a laptop; editing production data by hand; no error tracking, so bugs are found by users; no backups (or never tested); everything committed straight to main with no review or CI.

**How to check:** read deploy config (Vercel/Netlify/Docker/CI files), env examples, README deploy notes. Ask the user how they deploy and whether dev and prod share a database if the code can't tell you. Check for an error-tracking setup and backup notes.

## 9. Dependencies and configuration

**Senior work:** a lockfile is committed; dependencies are few, maintained and used; no known critical vulnerabilities left unexplained; versions pinned sensibly; config is validated at startup (missing env var fails fast with a clear message).

**Common gaps:** two libraries for the same job (axios + fetch wrapper, two date libs, two UI kits); abandoned or typo-squat packages; unused dependencies; no lockfile; `npm audit` criticals ignored; env vars read raw everywhere with `!` and no validation.

**How to check:** read the manifest; look for duplicates and unused packages; run the dependency audit; check for an env validation module.

## 10. UX states and accessibility

**Senior work:** every data view has loading, empty and error states. Forms validate inline, disable while submitting, prevent double-submit and preserve input on error. Destructive actions confirm. Keyboard navigation and focus are handled; labels, contrast and alt text meet WCAG AA. Works at phone width.

**Common gaps:** happy path only (blank screen while loading, crash on empty); spinners forever on error; buttons that can be clicked twice; `div` with `onClick` instead of `button`; inputs without labels; placeholder-as-label; modals that trap or lose focus; layout breaks on mobile.

**How to check:** read the main pages and forms for loading/empty/error handling. Grep for `onClick` on non-interactive elements, inputs without labels, images without alt. Run axe/Lighthouse if a dev server is available.

## 11. Docs and developer experience

**Senior work:** a README that gets a new engineer running in under 15 minutes: prerequisites, env setup (with an `.env.example`), commands, how to run tests, how to deploy. Architecture notes or ADRs for non-obvious decisions. Scripts for common tasks.

**Common gaps:** the default framework README; no `.env.example`; setup steps that only exist in the builder's head; AI-agent instruction files that are the only documentation and contradict the code.

**How to check:** read the README and follow it mentally. Check `.env.example` against env vars used in code.

## 12. Product and scope judgement

Light touch: this is an engineering audit, but seniors do judge scope.

**Senior work:** the core workflow is excellent before secondary features are added. Features that exist are finished (no dead buttons, half-built pages, "coming soon" in the main navigation).

**Common gaps:** many shallow features rather than a few deep ones; settings pages for features that don't work; demo/placeholder data left in production paths.

**How to check:** walk the navigation and routes; count features versus finished features; grep for placeholder/demo/lorem.

---

## Scoring anchors

Score each dimension 1–10 relative to what a senior engineer would ship **at the app's stage** (a prototype isn't penalised for missing on-call runbooks; an app taking payments is).

| Score | Meaning |
|---|---|
| 9–10 | A senior reviewer would approve as is; would point to it as a good example. |
| 7–8 | Senior-level with a few gaps a reviewer would note but not block on. |
| 5–6 | Works, but a senior reviewer would block on several issues; reads as unowned in places. |
| 3–4 | Significant gaps; would need rework before real users or a technical handover. |
| 1–2 | Missing or actively unsafe (e.g. any user can read others' data). |

Overall score: weight Security, Data integrity and Error handling highest when the app holds real user data or money. Any open Critical finding caps the overall score at 5, because a senior engineer wouldn't ship with it regardless of how polished the rest is.
