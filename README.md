# senior-gap-audit

A Claude skill that reviews an app you built with AI the way a senior engineer and a senior product designer would. It lists every place the app falls short of what they would have built, and the "tells" that make code and UI read as vibe-coded, so you can close those gaps one by one.

> **Not an assurance audit.** This is a gap-finding self-check: an experienced reviewer's opinion backed by evidence from your code and, where possible, your running app. Closing the gaps brings your app close to senior work; it doesn't certify that the app is secure, correct, production-ready or well designed.

## What you get

A Markdown report saved in your project (by default `docs/senior-gap-audit.md`) with:

- **Verdict**: what a senior engineer and a senior designer would say after an hour with your app, with two separate scores out of 10, **Engineering** and **Design**, plus a score for each area. They're never averaged, so good looks can't hide weak security, or the reverse.
- **Fix first**: the 3–7 gaps that matter most across both, in order, each with evidence and the fix.
- **Engineering findings** and **Design and UX findings**: every gap, with severity (Critical, High, Medium, Low), effort (S, M, L) and evidence: a file and line, a command's output, a measured count or a screenshot.
- **What reads as AI-generated**: the code and design patterns that make a reviewer think nobody owned the product.
- **Already at senior level**: what you did well, so you know what not to change.
- **Not checked**: what the audit couldn't verify, and how to verify it.
- **Re-audit when**: when to run it again.

## What it checks

Everything is judged against what seniors would ship at your app's stage: a prototype isn't held to the bar of an app taking payments.

**Engineering**

1. Security and access control
2. Data model and integrity
3. Architecture and boundaries
4. Error handling and resilience
5. Testing and verification
6. Code hygiene and consistency
7. Performance
8. Operations and delivery
9. Dependencies and configuration
10. Docs and developer experience
11. Product and scope judgement

**Design and UX**

1. Hierarchy and layout
2. Typography
3. Colour, contrast and theming
4. Component consistency
5. Navigation and information architecture
6. Core flows and feedback
7. Forms and input
8. Loading, empty, error and success states
9. Accessibility (WCAG 2.2 AA)
10. Responsive and mobile
11. Copy and microcopy

It runs your project's own type check, linter, tests and dependency audit and reports the real results. For design, it measures consistency in the code (how many font sizes, colours, radii and button styles you actually use) and, when your app is running and browser tools are available, screenshots the main screens at desktop and phone width.

It never runs anything that writes to a hosted database, sends email, charges money or deploys, never signs in with your real credentials, and doesn't change your code during the audit.

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
- "Does my UI look AI-generated? Do a design and UX audit."
- "Just the engineering part, skip design."
- "Audit my app before I launch."
- "Run the senior gap audit."

After the report, Claude offers to work through the Fix first list with you. When you've fixed things, ask for a re-audit: it checks the earlier findings first and marks each one closed, partly closed or still open.

## Files

```
senior-gap-audit/
├── SKILL.md                     # the method and report format
└── references/
    ├── dimensions.md            # the 11 engineering areas, how to check each, scoring
    ├── design-ux.md             # the 11 design and UX areas, AI design tells, scoring
    └── vibecode-tells.md        # signs that code was generated and never owned
```
