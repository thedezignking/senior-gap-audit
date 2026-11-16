# Design and UX audit

Eleven areas a senior product designer would review in an AI-built app, scored separately from engineering. For each: what senior work looks like, the common gaps, and how to check.

Design judgement is more subjective than engineering, so anchor every finding to a principle the user can look up (hierarchy, proximity, consistency, contrast ratios, platform conventions, WCAG), never to taste alone. "I'd prefer blue" is not a finding; "the primary and secondary buttons are the same weight, so the screen has no obvious next step" is.

Contents:
- Gathering evidence
- 1. Hierarchy and layout
- 2. Typography
- 3. Colour, contrast and theming
- 4. Component consistency
- 5. Navigation and information architecture
- 6. Core flows and feedback
- 7. Forms and input
- 8. States: loading, empty, error, success
- 9. Accessibility
- 10. Responsive and mobile
- 11. Copy and microcopy
- AI design tells
- Scoring anchors

---

## Gathering evidence

Use two kinds of evidence; the second is stronger when you can get it.

**From the code (always available).** Design consistency is measurable:
- Find the design tokens (CSS variables, Tailwind theme, a theme file) and the component library.
- Count distinct values actually used: font sizes, font weights, corner radii, shadows, colours, spacing steps. Grep for arbitrary values (`text-[13px]`, `rounded-[18px]`, raw hex like `#1a1a1a`, inline `style={{}}`) outside the token layer. A senior system has a handful of each; generated UI often has dozens. Report the numbers ("14 distinct font sizes across 60 components; 9 corner radii").
- Find how many button, card, input and modal implementations exist. More than one of each is a consistency gap unless they're deliberate variants of one component.
- Read the pages for the core flows to see the order of elements, the number of primary actions and the states handled.

**From the running app (when a dev server, preview or live URL is available).** Take screenshots of the 4–6 most important screens (landing or sign-in, the main dashboard, the core task, a form, a list, an empty account) at desktop width (about 1440px) and phone width (about 390px). Use whatever browser tooling the session has. Screenshots let you judge hierarchy, rhythm and polish, which code alone can't show. Don't sign in with the user's real credentials or submit real data; use a test account the project provides, or audit the screens reachable without one and say what you couldn't reach.

If there's no way to see the app running, say so in "Not checked" and score the visual areas from code with lower confidence.

---

## 1. Hierarchy and layout

**Senior work:** every screen has one obvious first thing to look at and one primary action. Related things sit close together and groups are clearly separated (proximity), so borders and boxes are rarely needed. Content aligns to a grid; widths and margins are consistent between pages. Dense screens use progressive disclosure rather than showing everything.

**Common gaps:** several equally loud elements competing; two or three primary buttons on one screen; everything in its own bordered card ("box soup"); dashboards of stat tiles that don't help a decision; inconsistent page widths and paddings; elements floating off the grid.

**How to check:** screenshots: squint test (what stands out when blurred?); count primary-styled buttons per screen; look at alignment edges. Code: page container widths and paddings across routes; count of bordered wrappers.

## 2. Typography

**Senior work:** a small type scale (roughly 5–7 sizes) with clear roles (display, heading, body, small, caption); two weights doing most of the work; comfortable line height and line length (about 45–80 characters for body text); numbers in tables use tabular figures; headings balanced, no orphans in key headings.

**Common gaps:** a dozen ad hoc sizes; bold used for emphasis everywhere so nothing stands out; long lines of body text across a wide screen; tight line height on paragraphs; mixed fonts with no reason.

**How to check:** count distinct font sizes and weights in the code; check font loading and families; in screenshots, check line length and how headings, body and labels relate.

## 3. Colour, contrast and theming

**Senior work:** a restrained palette defined as tokens: neutrals plus one brand accent, with semantic colours (success, warning, danger, info) used only for meaning. Text and controls meet WCAG AA contrast (4.5:1 for body text, 3:1 for large text and UI parts). Dark mode, if offered, is designed, not inverted.

**Common gaps:** raw hex values scattered through components; several near-identical greys; colour used decoratively so it no longer signals meaning; light grey text on white below AA; gradients and glows as a substitute for hierarchy; a broken or half-done dark mode.

**How to check:** grep for hex/rgb outside the token files and count distinct colours; check token pairs for contrast; screenshots in light and dark themes if both exist.

## 4. Component consistency

**Senior work:** one component per job (button, input, card, dialog, table, tabs, badge), each with deliberate variants, used everywhere. Corner radii, shadows, borders and icon sizes come from a small set. One icon family. Motion is quiet and consistent: short ease-out transitions on enter, press feedback on buttons, nothing that delays the user.

**Common gaps:** buttons built three ways; a mix of icon libraries; different radii and shadows on similar cards; one-off styled elements where a shared component exists; flashy animations on things used every minute; inconsistent hover and focus styles.

**How to check:** list button, input, card and dialog implementations; count distinct radius and shadow values; count icon packages in the manifest; look for components that bypass the shared UI folder.

## 5. Navigation and information architecture

**Senior work:** the navigation reflects how users think about their work, not how the database is shaped. Labels are short and consistent with page titles. Users always know where they are (active state, page title, breadcrumbs where depth exists). Important things are at most two clicks away; rarely used settings are out of the way.

**Common gaps:** navigation items named after tables; the same page reachable under different names; dead or "coming soon" items; deep nesting; no active state; settings mixed into the main navigation.

**How to check:** read the navigation config and route tree; compare nav labels to page titles; click through (or trace routes) for the top three tasks.

## 6. Core flows and feedback

**Senior work:** the two or three tasks the product exists for take few steps and no dead ends. Every action gives feedback (pressed state, progress, success confirmation, or the result appearing in place). Destructive actions are clearly marked and confirmed; reversible actions offer undo instead of a confirmation. Create and edit happen in context (dialogs or side panels for short tasks) so users don't lose their place.

**Common gaps:** a core task spread across several pages with no progress indicator; buttons that do something silently; full-page redirects after small edits; confirmations for everything (so people stop reading them) or for nothing destructive; no way back after a mistake.

**How to check:** walk each core flow end to end (screens or code path), counting steps, decisions and moments without feedback; check destructive actions for styling and confirmation.

## 7. Forms and input

**Senior work:** visible labels (not placeholders as labels), sensible input types and autocomplete attributes, inline validation with specific messages next to the field, required and optional clearly marked, submit disabled or guarded while sending, input kept on error, sensible defaults, and the minimum number of fields.

**Common gaps:** placeholder-only labels; a generic "Something went wrong" at the top of the form; validation only after submit; lost input on error; double submission; long forms asking for data the product doesn't need yet.

**How to check:** read the main forms (sign-up, onboarding, the core create form); check labels, types, autocomplete, error placement and submitting state.

## 8. States: loading, empty, error, success

**Senior work:** every data view has a loading state that matches the final layout (skeletons rather than a spinner in an empty page), an empty state that explains what goes here and offers the first action, an error state with a way to retry, and a clear success state. First-run experience is designed: a new account isn't a wall of empty widgets.

**Common gaps:** blank screens during loading; spinners forever when a request fails; empty tables with headers and nothing else; no first-run guidance; success with no visible change.

**How to check:** for each main page, find its loading, empty and error handling (framework loading/error files, Suspense boundaries, explicit branches); look at a brand-new account if one can be made with test data.

## 9. Accessibility

**Senior work:** WCAG 2.2 AA as a baseline: semantic HTML (buttons are buttons, links are links), every input labelled, visible focus styles, full keyboard use including dialogs and menus (focus trapped and restored), alt text on meaningful images, sufficient contrast, target sizes of at least 24×24px, respects reduced-motion settings, and automated checks (axe) in tests.

**Common gaps:** `div` with `onClick`; focus outlines removed with nothing in their place; icon-only buttons with no accessible name; modals that lose focus; colour as the only signal of state; animations that ignore reduced motion.

**How to check:** grep for click handlers on non-interactive elements, `outline-none` without a replacement focus style, icon buttons without `aria-label`, images without `alt`; run axe if a running app is available.

## 10. Responsive and mobile

**Senior work:** layouts are designed for phone width, not just squeezed: navigation collapses sensibly, tables become cards or scroll within their container, touch targets are large enough, nothing scrolls sideways, and text stays readable without zooming.

**Common gaps:** horizontal page scroll on phones; tiny tap targets; desktop sidebars that cover the content; tables cut off; fixed heights that clip content; modals taller than the screen.

**How to check:** phone-width screenshots of the main screens; check that the document width equals the viewport; grep for fixed pixel widths and heights on layout containers.

## 11. Copy and microcopy

**Senior work:** plain, specific language in the product's own voice; the same word for the same thing everywhere; buttons say what they do ("Send invoice", not "Submit"); error messages say what happened and what to do next; no filler.

**Common gaps:** generic button labels; inconsistent terms for one concept (client, customer, account); marketing superlatives inside the app; lorem ipsum or "John Doe" in real screens; robotic or over-excited AI-written copy ("Unlock the power of…", emoji in headings).

**How to check:** read button labels, headings, empty states and error messages across the main screens; grep for placeholder names and filler phrases.

---

## AI design tells

The signals that make a designer think the UI was generated and never art-directed. Report each one found with evidence, the same way as the code tells.

- **Template sections**: the stock landing page (hero with gradient blob, three feature cards with emoji or generic icons, logo strip, testimonial carousel, pricing table) with no product-specific idea.
- **Generic decoration**: purple-blue gradients, glowing orbs, glassmorphism and noise used everywhere instead of hierarchy.
- **Box soup**: every element wrapped in a bordered, shadowed, rounded card.
- **Random values**: many slightly different radii, shadows, greys and font sizes, because each screen was generated separately.
- **Dashboard filler**: stat tiles and charts that don't answer a question the user has.
- **Emoji headings and sparkles** in a professional product.
- **Default kit look**: an unmodified component library with no brand decisions.
- **AI copy voice**: "Seamlessly", "Unlock", "Elevate", "Supercharge", "Effortlessly"; every heading a slogan.
- **Placeholder residue**: stock avatars, "Lorem ipsum", "Jane Doe", fake metrics presented as real.
- **Inconsistent screens**: two pages that look like different products.

---

## Scoring anchors

Score each area 1–10 against what a senior product designer would ship **at the app's stage** (an MVP can be plain; it shouldn't be inconsistent or confusing). Give one overall Design score, reported next to the Engineering score, never averaged into it.

| Score | Meaning |
|---|---|
| 9–10 | Distinctive and consistent; a senior designer would use it as a reference. |
| 7–8 | Clean, consistent and usable, with a few rough edges a reviewer would note. |
| 5–6 | Usable but generic or inconsistent; reads as a template in places. |
| 3–4 | Confusing flows or obvious inconsistency; users will struggle or distrust it. |
| 1–2 | Broken layouts, inaccessible core paths, or unusable on common devices. |

An accessibility gap that blocks a core task (for example, a sign-up form that can't be completed by keyboard) caps the Design score at 5.
