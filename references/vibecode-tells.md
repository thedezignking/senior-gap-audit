# Vibe-code tells

Signals that make a senior reviewer conclude "an AI wrote this and nobody owned it". Some are harmless on their own; together they destroy trust in the codebase, because they suggest nobody read the code before shipping it. Report each one found, with evidence, so the user can see their code the way a reviewer would.

The underlying pattern behind almost all of them: **code that was generated per prompt and never reconciled into one design.**

## In the code

- **Several solutions to one problem.** Three ways to fetch data, two form libraries, two date libraries, mixed `fetch`/`axios`, mixed error-handling styles. Each prompt started fresh.
- **Narrating comments.** `// Import React`, `// State for the user`, `// Handle the click`, `// Return the result`. Comments that repeat the code rather than explain a decision.
- **Placeholder residue.** `// TODO: implement`, `// Add your logic here`, `// In a real app you would…`, `// For now…`, `throw new Error("Not implemented")` in reachable code, `example.com`, `John Doe`, `lorem ipsum` in real paths.
- **Defensive noise.** Optional chaining and null checks on values that can't be null; `try/catch` around code that can't throw; `if (!data) return null` everywhere hiding real bugs.
- **Swallowed errors.** `catch (e) { console.error(e) }` and carry on as if nothing happened.
- **Type escapes.** `any`, `as any`, `as unknown as X`, `@ts-ignore`, `eslint-disable` sprinkled to make errors go away.
- **Debug leftovers.** `console.log` everywhere, `debugger`, commented-out blocks of old attempts.
- **Version-suffixed files.** `Header2.tsx`, `utils-new.ts`, `page-old.tsx`, `final-final`, plus the unused original still in the repo.
- **Over-built abstractions.** A generic factory, strategy pattern or config-driven engine with exactly one use; wrappers around wrappers.
- **Giant files.** 600–1500 line components or route files where everything was appended in the same place.
- **Inconsistent naming.** `getUser`, `fetchUserData`, `loadUserInfo` and `userGetter` doing the same thing in different files; mixed casing conventions.
- **Hallucinated or stale APIs.** Calls to functions or options that don't exist in the installed library version, deprecated APIs, or a pattern from a different framework version, often patched around rather than fixed.
- **Magic values.** Hardcoded IDs, URLs, prices, limits and emails repeated across files.
- **Unused everything.** Exports, files, dependencies, env vars and DB columns nothing uses.

## In the repo and process

- **Commit history.** "update", "fix", "changes", "wip" or giant commits touching 80 files; or a history with no commits for weeks then one enormous one.
- **No CI, or CI that's always red.**
- **Default framework README**, or agent instruction files as the only docs.
- **Committed secrets** or `.env` files, even if later deleted (they stay in history).
- **Tests that don't run**, or were written once to satisfy a prompt and then left failing.

## In the product

The full list of design tells is in `design-ux.md` ("AI design tells"); these are the ones that also show up in an engineering review.

- **Happy path only.** No loading, empty or error states; the app breaks on the first slow network or empty account.
- **Template look.** Unmodified starter-kit UI, inconsistent spacing and type scale between pages, generic gradient-and-emoji landing copy.
- **Dead ends.** Buttons that do nothing, settings that aren't saved, "coming soon" pages in the main navigation.
- **Inconsistent copy.** Different words for the same thing across screens; mixed tone.

## How to report them

Group tells by kind, give 1–3 concrete examples each with file paths, and estimate how widespread it is ("~40 `console.log` across 22 files"). Make clear which tells are only perception (narrating comments) and which also carry risk (swallowed errors, type escapes, committed secrets), so the user fixes the risky ones first.
