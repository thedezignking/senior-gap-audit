# senior-gap-audit

A Claude skill that reviews an app you built with AI the way a senior engineer would. It lists every place the app falls short of what a senior engineer would have built, and the "tells" that make code read as vibe-coded, so you can close those gaps one by one.

> **Not an assurance audit.** This is a gap-finding self-check: an experienced reader's opinion backed by evidence from your code. Closing the gaps brings your app close to senior work; it doesn't certify that the app is secure, correct or production-ready.

## What you get

A Markdown report saved in your project (by default `docs/senior-gap-audit.md`) with:

- **Verdict**: what a senior engineer would say after an hour with your code, plus a 1–10 score for each area and overall.
- **Fix first**: the 3–7 gaps that matter most, in order, each with evidence and the fix.
- **All findings**: every gap, with severity (Critical, High, Medium, Low), effort (S, M, L) and a file path and line for each one.
- **What reads as vibe-coded**: the patterns that make a reviewer think nobody owned the code.
- **Already at senior level**: what you did well, so you know what not to change.
- **Not checked**: what the audit couldn't verify, and how to verify it.
- **Re-audit when**: when to run it again.

## What it checks

Twelve areas, judged against what a senior engineer would ship at your app's stage (a prototype isn't held to the bar of an app taking payments):

1. Security and access control
2. Data model and integrity
3. Architecture and boundaries
4. Error handling and resilience
5. Testing and verification
6. Code hygiene and consistency
7. Performance
8. Operations and delivery
9. Dependencies and configuration
10. Loading, empty and error states, and accessibility
11. Docs and developer experience
12. Product and scope judgement

It runs your project's own type check, linter, tests and dependency audit and reports the real results. It never runs anything that writes to a hosted database, sends email, charges money or deploys, and it doesn't change your code during the audit.

## Install

Install with the [Skills CLI](https://skills.sh/), which works with Claude Code and the other agents that support skills.

### Everywhere (recommended)

Installs globally, so the skill is available in every project:

```bash
npx skills add thedezignking/senior-gap-audit -g
```

### One project only

Run from the project's root folder. The skill is added to that project and can be committed with it, so your team gets it too:

```bash
npx skills add thedezignking/senior-gap-audit
```

### Manual install (Claude Code)

If you'd rather not use the CLI, clone the repository into your skills folder.

macOS or Linux:

```bash
git clone https://github.com/thedezignking/senior-gap-audit.git ~/.claude/skills/senior-gap-audit
```

Windows (PowerShell):

```powershell
git clone https://github.com/thedezignking/senior-gap-audit.git "$env:USERPROFILE\.claude\skills\senior-gap-audit"
```

### Claude apps (web and desktop)

Download this repository as a ZIP (green **Code** button → **Download ZIP**) and upload it as a custom skill in Claude's settings.

### Update

```bash
npx skills update
```

Start a new session after installing or updating so the agent picks up the skill.

## Use

Open your project with Claude and ask in your own words, for example:

- "How would a senior engineer rate my app?"
- "Does my codebase look vibe-coded?"
- "Audit my app before I launch."
- "Run the senior gap audit."

After the report, Claude offers to work through the Fix first list with you. When you've fixed things, ask for a re-audit: it checks the earlier findings first and marks each one closed, partly closed or still open.

## Files

```
senior-gap-audit/
├── SKILL.md                     # the method and report format
└── references/
    ├── dimensions.md            # the 12 areas, how to check each, scoring anchors
    └── vibecode-tells.md        # signs that code was generated and never owned
```
