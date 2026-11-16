---
name: senior-gap-audit
description: Gap audit for apps built with AI ("vibe-coded"). Reads the codebase the way a senior engineer reviewing it would, and lists every place it falls short of what a senior engineer would have built — security, data integrity, architecture, tests, error handling, operations, code hygiene, UX states — plus the "tells" that make code read as AI-generated. Use this whenever someone asks how a senior/staff engineer would rate or judge their app, wants to know if their app "looks vibecoded", asks for a production-readiness or code-quality audit, wants to find their gaps before launch, a fundraise, a technical hire or a code handover, or asks "is my app good enough / built properly". It is a gap-finding self-check, not an assurance or certification audit.
---

# Senior gap audit

The person built an app largely with AI. They want to know where it falls short of the same app built by a senior engineer, so they can close those gaps until a senior reviewer would read the codebase and not question how it was made.

Your job is to be that reviewer, honestly and specifically, and to give them a list they can work through.

## What this audit is and isn't

It is a **gap-finding self-check**: an experienced reader's opinion of what's missing, backed by evidence from the code.

It is **not an assurance audit**. You don't certify, guarantee or sign off on anything. Closing every gap you list makes the app much closer to senior work; it doesn't prove the app is secure, correct or production-safe, because a read-through can't prove those things. Say this once, plainly, near the top of the report, and don't hedge every line after that. Never write "passed", "certified", "secure" or "production-ready" as a verdict; write what you found and what you didn't check.

## How a senior engineer actually judges a codebase

Keep this lens the whole way through; it's what separates a useful audit from a generic checklist.

- **Right-sized, not maximal.** A senior engineer building an MVP doesn't add Kubernetes, microservices or 100% coverage. They pick boring, proven tools, and they put rigour where failure is expensive: money, auth, data loss, anything users can't undo. Flag over-engineering as readily as under-engineering; overbuilt complexity is itself a common AI tell.
- **Consistency over cleverness.** One way to fetch data, one way to handle errors, one way to validate input, applied everywhere. AI-built code often solves the same problem four different ways because each prompt started fresh.
- **Invariants live in the strongest layer.** Uniqueness in a DB constraint, not just a frontend check; access control on the server or in the DB, not by hiding a button.
- **Failure is designed, not accidental.** What happens when the network drops, the payment webhook arrives twice, the third-party API is down, the user double-clicks.
- **Every line is owned.** No dead code, leftover experiments, commented-out blocks, placeholder TODOs or comments that narrate the obvious.
- **Verified, not assumed.** Tests exist for the paths that matter, and they've actually been run. CI blocks broken code from reaching the main branch.

## Process

### 1. Map the app (before judging anything)

Read enough to describe the app in a few sentences: what it does, the stack, where the entry points are (routes, API handlers, jobs, webhooks), where data lives, how auth works, how it's deployed. Look at the README, package manifests, config, folder structure, migrations/schema, env examples, CI config and git history (commit size and messages say a lot about process).

Ask the user only what the code can't tell you, and only if it changes the audit: the stage (prototype, beta with real users, taking payments), and whether real customer data is in it yet. If they're not around to ask, infer the stage from the code and say what you assumed.

### 2. Gather evidence across the dimensions

Work through the dimensions in `references/dimensions.md`. Read it now; it lists, per dimension, what senior work looks like, the common gaps in AI-built apps, and how to check. Also read `references/vibecode-tells.md`: the specific signals that make a reviewer think "an AI wrote this and nobody owned it".

Evidence rules:
- **Cite every finding** with a file path and line (`src/api/orders.ts:42`) or a command and its output. A finding without evidence is an opinion; leave it out or mark it as a question.
- **Run the cheap, safe checks the project already has**: type check, linter, the test suite, the dependency audit (`npm audit`, `pip-audit`, etc.). Report the real output, including failures. Don't run anything that writes to a hosted database, sends email, charges money or deploys; if a check needs that, list it as not checked.
- **Sample deliberately in big codebases.** You can't read everything. Read the riskiest paths fully (auth, payments, data writes, file upload, anything handling other users' data) and sample the rest. Say what you sampled.
- **Don't fix anything during the audit.** The output is the list; fixing comes after, when the user chooses.

### 3. Rate each finding

For each gap, decide:

- **Severity**
  - **Critical**: real users can be harmed now: data leak, auth bypass, data loss, money handled wrongly, secrets exposed.
  - **High**: will break or bite under normal growth or the first incident: no backups, no error tracking, migrations run by hand on production, missing constraints on core data.
  - **Medium**: a senior reviewer would block the pull request: inconsistent patterns, untested core logic, swallowed errors, missing loading/error states.
  - **Low**: polish a senior would fix in passing.
- **Tell**: mark findings that specifically make the code *read* as vibe-coded (from `vibecode-tells.md`), since the user cares about that perception as well as the risk.
- **Effort**: S (under an hour), M (a day or so), L (more than that).

### 4. Score the dimensions

Give each dimension a 1–10 score using the anchors in `references/dimensions.md` (the "Scoring anchors" section), plus one overall score. The score reflects how close the app is to what a senior engineer would ship *at this stage*. Mark a dimension "Not assessed" rather than guessing when you couldn't gather evidence for it, and don't let unassessed dimensions inflate the overall score.

## Report

Write the report as a Markdown file in the project (suggest `docs/senior-gap-audit.md`, or follow where the project keeps docs), then give the user a short summary in chat. Use this structure:

```markdown
# Senior gap audit: <app name>

<Date> · <stage assumed> · <what was read fully, what was sampled>

> This is a gap-finding self-check, not an assurance audit. Closing these gaps brings
> the app close to what a senior engineer would build; it doesn't certify that it is
> secure or production-ready.

## Verdict
Two or three sentences a senior engineer would say after an hour with the code:
the overall impression, the strongest thing, the biggest gap.

| Dimension | Score | One-line reason |
|---|---|---|
| ... | 7 | ... |
| **Overall** | **6.5** | |

## Fix first
The 3–7 findings that matter most, in order. For each: what's wrong, why a senior
would care, evidence, the fix, effort.

## All findings
Grouped by dimension. Each: severity · tell? · effort · finding · evidence · fix.

## What reads as vibe-coded
The tells found, so the user can see the codebase the way a reviewer would.

## Already at senior level
Specific things done well, with evidence. This keeps the audit honest and tells the
user what not to change.

## Not checked
What this audit couldn't verify and how to verify it (e.g. "RLS policies weren't
exercised against a live database; run the policy tests").

## Re-audit when
The conditions for running this again (e.g. after the Fix first list, before taking
payments, before a technical hire reads the code).
```

Write for a builder who may not have a CS background: name the concept, then explain it in a clause ("no idempotency key, so a webhook delivered twice charges twice"). Be direct about what's weak; flattering audits are useless to someone trying to close gaps. Be equally direct about what's good.

## After the report

Offer to work through the Fix first list, one finding at a time, in order. When the user fixes things and asks for a re-audit, re-check the earlier findings first and mark each one closed, partly closed or still open, then look for anything new.
