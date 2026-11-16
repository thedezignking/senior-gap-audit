---
name: senior-gap-audit
description: Gap audit for apps built with AI ("vibe-coded"). Reviews the codebase the way a senior engineer would and the product the way a senior product designer would, and lists every place it falls short — security, data integrity, architecture, tests, error handling, operations, code hygiene, plus visual design, UX flows, states, accessibility and mobile — along with the "tells" that make code and UI read as AI-generated. Two scores, Engineering and Design. Use this whenever someone asks how a senior engineer or designer would rate their app, wants to know if their app or UI "looks vibecoded" or AI-generated, asks for a production-readiness, code-quality, UI/UX or design audit of their own app, or wants to find their gaps before launch, a fundraise, a hire or a handover. It is a gap-finding self-check, not an assurance or certification audit.
---

# Senior gap audit

The person built an app largely with AI. They want to know where it falls short of the same app built by a senior engineer and designed by a senior product designer, so they can close those gaps until a senior reviewer would look at the code and the screens and not question how it was made.

Your job is to be those reviewers, honestly and specifically, and to give them a list they can work through.

## What this audit is and isn't

It is a **gap-finding self-check**: an experienced reviewer's opinion of what's missing, backed by evidence from the code and, where possible, the running app.

It is **not an assurance audit**. You don't certify, guarantee or sign off on anything. Closing every gap you list makes the app much closer to senior work; it doesn't prove the app is secure, correct, production-ready or well designed, because a review can't prove those things. Say this once, plainly, near the top of the report, and don't hedge every line after that. Never write "passed", "certified", "secure" or "production-ready" as a verdict; write what you found and what you didn't check.

## Scope

By default the audit has two parts, scored separately:

- **Engineering**: the 11 areas in `references/dimensions.md`.
- **Design and UX**: the 11 areas in `references/design-ux.md`.

If the user asks for only one ("just the code", "design only", "UX audit"), run that part and say the other wasn't covered. If the app has no user interface (an API, a CLI, a library), skip Design and say why.

## How senior reviewers actually judge

Keep this lens the whole way through; it's what separates a useful audit from a generic checklist.

- **Right-sized, not maximal.** A senior engineer building an MVP doesn't add Kubernetes, microservices or 100% coverage; a senior designer doesn't demand a bespoke design system for a beta. They put rigour where failure is expensive: money, auth, data loss, anything users can't undo, and the core task the product exists for. Flag over-engineering and over-decoration as readily as gaps; both are common AI tells.
- **Consistency over cleverness.** One way to fetch data, handle errors and validate input; one button, one type scale, one spacing system. AI-built apps often solve the same problem four different ways because each prompt started fresh.
- **Invariants live in the strongest layer.** Uniqueness in a DB constraint, not just a frontend check; access control on the server or in the DB, not by hiding a button.
- **Failure is designed, not accidental.** What happens when the network drops, the webhook arrives twice, the list is empty, the request fails, the user double-clicks.
- **Every line and pixel is owned.** No dead code, placeholder TODOs, narrating comments, leftover demo content or decoration that serves no purpose.
- **Verified, not assumed.** Tests exist for the paths that matter and actually run; CI blocks broken code from reaching the main branch; the UI has been looked at on a phone.

## Process

### 1. Map the app (before judging anything)

Read enough to describe the app in a few sentences: what it does, who uses it, the two or three core tasks, the stack, the entry points (routes, API handlers, jobs, webhooks), where data lives, how auth works, how it's deployed, and where the UI components and design tokens live. Look at the README, package manifests, config, folder structure, migrations/schema, env examples, CI config and git history (commit size and messages say a lot about process).

Ask the user only what the code can't tell you, and only if it changes the audit: the stage (prototype, beta with real users, taking payments), whether real customer data is in it yet, and whether there's a running app you can look at (a dev server command, preview or live URL, and a test account). If they're not around to ask, infer the stage from the code and say what you assumed.

### 2. Gather evidence

Read `references/dimensions.md` and `references/vibecode-tells.md` for Engineering, and `references/design-ux.md` for Design and UX. Each lists, per area, what senior work looks like, the common gaps in AI-built apps, and how to check.

Evidence rules:
- **Cite every finding** with a file path and line (`src/api/orders.ts:42`), a command and its output, a measured count ("11 distinct font sizes"), or a screenshot. A finding without evidence is an opinion; leave it out or mark it as a question.
- **Run the cheap, safe checks the project already has**: type check, linter, the test suite, the dependency audit (`npm audit`, `pip-audit`, etc.), formatter check. Report the real output, including failures. If CI exists, look at its recent runs too. Don't run anything that writes to a hosted database, sends email, charges money or deploys; if a check needs that, list it as not checked.
- **Look at the running app when you can.** If a dev server, preview or URL is available and you have browser tools, screenshot the main screens at desktop and phone width (see `design-ux.md`, "Gathering evidence"). Use only test accounts and test data; never the user's real credentials.
- **Sample deliberately in big codebases.** Read the riskiest paths fully (auth, payments, data writes, file upload, anything handling other users' data, and the core user flows) and sample the rest. Say what you sampled.
- **Don't fix anything during the audit.** The output is the list; fixing comes after, when the user chooses.

### 3. Rate each finding

For each gap, decide:

- **Severity**
  - **Critical**: real users can be harmed now: data leak, auth bypass, data loss, money handled wrongly, secrets exposed, a core task impossible for some users (for example, unusable by keyboard or on phones).
  - **High**: will break or bite under normal growth or the first incident (no backups, no error tracking, production data used for development, missing constraints on core data), or a core flow that confuses most users.
  - **Medium**: a senior reviewer would block the pull request or the design review: inconsistent patterns, untested core logic, swallowed errors, missing loading or error states, inconsistent components.
  - **Low**: polish a senior would fix in passing.
- **Tell**: mark findings that specifically make the code or UI *read* as AI-generated, since the user cares about that perception as well as the risk.
- **Effort**: S (under an hour), M (a day or so), L (more than that).

### 4. Score

Score each area 1–10 with the anchors in its reference file, then give two overall scores: **Engineering** and **Design**. Keep them separate and never average them into one number, so strong visuals can't hide weak security, or the reverse. The scores reflect how close the app is to senior work *at this stage*. Mark an area "Not assessed" rather than guessing when you couldn't gather evidence, and don't let unassessed areas inflate the overall score.

## Report

Write the report as a Markdown file in the project (suggest `docs/senior-gap-audit.md`, or follow where the project keeps docs; save screenshots next to it, e.g. `docs/senior-gap-audit/`), then give the user a short summary in chat. Use this structure:

```markdown
# Senior gap audit: <app name>

<Date> · <stage assumed> · <what was read fully, what was sampled, which screens were seen>

> This is a gap-finding self-check, not an assurance audit. Closing these gaps brings
> the app close to what a senior engineer and designer would build; it doesn't certify
> that it is secure, production-ready or well designed.

## Verdict
Two or three sentences each from the senior engineer and the senior designer after an
hour with the app: the overall impression, the strongest thing, the biggest gap.

**Engineering: 7/10 · Design: 6/10**

| Engineering area | Score | One-line reason |
|---|---|---|
| ... | 7 | ... |

| Design and UX area | Score | One-line reason |
|---|---|---|
| ... | 6 | ... |

## Fix first
The 3–7 findings that matter most across both parts, in order, each labelled
Engineering or Design: what's wrong, why a senior would care, evidence, the fix, effort.

## Engineering findings
Grouped by area. Each: severity · tell? · effort · finding · evidence · fix.

## Design and UX findings
Same format; link screenshots where they show the problem.

## What reads as AI-generated
The code tells and design tells found, so the user can see the app the way a reviewer would.

## Already at senior level
Specific things done well in both parts, with evidence. This keeps the audit honest and
tells the user what not to change.

## Not checked
What this audit couldn't verify and how to verify it (e.g. "screens behind sign-in weren't
seen; provide a test account").

## Re-audit when
The conditions for running this again.
```

Write for a builder who may not have a CS or design background: name the concept, then explain it in a clause ("no idempotency key, so a webhook delivered twice charges twice"; "two primary buttons, so the screen has no obvious next step"). Be direct about what's weak; flattering audits are useless to someone trying to close gaps. Be equally direct about what's good.

## After the report

Offer to work through the Fix first list, one finding at a time, in order. When the user fixes things and asks for a re-audit, re-check the earlier findings first and mark each one closed, partly closed or still open, then look for anything new.
