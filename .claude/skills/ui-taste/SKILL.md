---
name: "ui-taste"
description: "Use when building, designing, restyling or reviewing any web page, landing page, UI component, or app screen/dashboard, so it doesn't look generic or AI-made."
---

# UI taste (anti-slop frontend rules)

Distilled from Leonxlnx/taste-skill v2 (MIT): its house rules and laws only. The original's scripts (direction drawing, browser inspector, image search) are not included, to keep this lightweight.

A model-built page usually looks like the average of every page: centred headline, little label above it, a row of icon cards, a soft gradient, thin rules, monospace captions. Replace defaults with decisions.

## Before building
1. **Page or app screen?** Pages are read and scrolled (landing, marketing, portfolio). App screens are worked in (dashboard, settings, admin) — there, quiet borders, small labels and repeated panels are fine; gradients, glows, icons in tinted squares, rainbow charts and invented vanity numbers are not.
2. **Inventory the subject:** 8–12 things that belong to this client and no competitor (name and meaning, place, numbers, constraints, materials, rituals, printed artefacts).
3. **Write the argument:** one claim + proof per section. Delete sections with no claim. Avoid the stock hero → logos → 3 features → testimonial → pricing → CTA.
4. **Find the idea:** "On this page, [something only this subject has] becomes [the visual device]." Competitor test: if it still works with a competitor's name, it's a category idea — try again.
5. **Write the copy first.** Headlines ≤ 2 lines at every width (~32 characters). One supporting sentence directly under. One primary action. Write full content (whole menu, every price). Never invent proof: no fake customers, quotes, logos, stats.

## House rules
| Don't | Instead |
|---|---|
| Headlines over two lines | Cut words, widen measure, size last |
| Missing title / unlabeled sections | One `h1` on first screen; a heading on every section |
| Sections huddled on one side, blank stretches > half a screen | Equal outer margins; close voids with a picture, column, list |
| Eyebrow labels, pill badges, "New ·" chips | Put it in the heading or drop it |
| Monospace outside code | The text face; tabular figures for numbers |
| 01/02/03 index numbering | Order by position, size, words, or a real `<ol>` |
| Hairline dividers, 1px outlined boxes, grid lines | Space, a tone step, a change of scale |
| Gradients (bg, buttons, text) | Flat colour or a real photo |
| Shadows on resting things, glows | Tone step; shadows only on menus/dialogs |
| Cream + terracotta, violet/indigo "assistant" palettes | A deliberate palette |
| Tiny uppercase letter-spaced labels | Sentence case at readable size |
| Icon-topped feature cards, rows of identical cards | Varied section compositions |
| Glass / backdrop blur, floating pill nav | Plain bar: wordmark, 3–5 links, one action |
| Emoji, blobs, decorative shapes, fake grey-bar dashboards | A real picture, real interface fragment, or nothing |
| Styled accent word in headline | One face, one colour |
| More than two typefaces | Usually one |

Removing ornament leaves space. Don't refill it with ornament — fill it with scale, full content, real pictures, or a field of flat colour. One deliberate empty rest per page at most.

## Laws
**Type:** 2–3 sizes do the work: one very large, one small. Big headline-to-body ratio (~4×+; largest type ~8vw on strong sites). Display tracking −0.02 to −0.05em, leading 0.85–1.05. Body 16–20px, leading 1.45–1.6, 45–75 chars/line. ≤ 5 sizes total. Left-aligned; centre at most one moment.

**Space:** One grid, tight equal margins (16–32px). Asymmetry inside the grid, never in the frame. Section gaps 120–240px desktop, clearly larger than inner gaps. Neighbouring sections never share a layout.

**Colour:** One ground, one ink, one accent (accent on the action and maybe one other thing). Running text in full ink, not grey. Separate surfaces by tone, not lines or shadows.

**Imagery:** One dominant image beats six small ones; hero media ≥ half the first screen. Consistent crops; square corners by default. Use real pictures (project's own, or credited open-licence archives) — never placeholder icons. Never present found images as the client's own. Nothing overlaps a headline.

**Components:** Flat buttons, one fill, square or fully round (not the 9–16px soft radius). Footer is a designed close, not a sitemap. Controls explain themselves by form and label.

**Motion (after the static page works):** One signature move; staggered section entrances (60–90ms, 16–32px rise); hover/press/focus on every control. Transform and opacity only; always a `prefers-reduced-motion` fallback; content never hidden if JS fails.

## Accessibility & states
Real links/buttons, visible focus, alt text, keyboard-operable controls with accessible names, a phone layout composed on its own. Cover hover, focus, loading, empty and error states.

## Redesigns
Screenshot/record the before. Keep the stack, content and what works — no framework migration. Order: type → colour → frame (margins, rhythm, headings) → states → replace stock components. Compare before/after at desktop, tablet, phone widths.

## Check before reporting
Look at screenshots at ~1440, 768 and 390px if you can. A stranger reading the first screen for 5 seconds must know what this is and what to do. Fix the top three visible problems, then look again.

