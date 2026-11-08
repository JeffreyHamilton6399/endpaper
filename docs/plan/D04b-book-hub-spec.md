# D4b: Book Hub Specification

Status: Draft for PO review (Phase 1) | Author: frontend-lead | Date: 2026-11-07
Companion to D4 (screen inventory) and D19 (toolchain). Words in *italics* are jargon explained once.

## 1. What this screen is, in one paragraph

The book hub is the main screen. It looks and behaves like a real book: a closed cover the first time, then a book that opens to today's page, with a page for every day you recorded. You turn pages by swiping, tapping the page edge, pressing arrow keys, or using buttons. Pages are grouped into chapters, one per month. The page turn is the one signature animation in the app (brief 8.2). Everything else stays quiet and fast.

## 2. The metaphor, mapped to data

| Book part | What it is in the app |
|---|---|
| Cover | Title ("Your name's book" or the user-chosen title) on a leather or cloth texture. Style depends on the design direction the PO picks in Phase 2. |
| Endpapers | The decorative inside covers (our product name). Shown when the book opens from the cover and on the very first page. Also the home of the "Chapters" and "Jump to date" controls. |
| Chapter | One calendar month of one year, for example "March 2026". Opens with a chapter-title page showing the month name, number of entries, and a small tally (see 2.2). |
| Page | One calendar day that has an entry. Shows the date as a heading and the rewritten entry text. |
| Page back ("verso") / front ("recto") | On desktop spread, the left and right sides. |

### 2.1 Multiple entries on the same day

Recommendation: **one page per day, with sections for each recording**, not one page per recording.

- The day is the unit the PO described ("a page per day"). People also record twice (a morning thought and a night entry).
- If a day has one entry: the page shows it as normal.
- If a day has two or more: the page shows them in time order, each with a small time label ("9:40 pm") and a thin rule between them. If the text is longer than one screen, the page scrolls inside itself (see 5.6) and never spills onto invented extra pages.
- A very long single entry also scrolls within its page. We do not paginate text across multiple physical pages in v1, because that needs text-measuring and breaks editing and screen readers. Revisit in a later phase if the PO wants true typeset pages.
- The "which day does this belong to" rule is owned by D6 (data model). The book only displays the `entry_date` it is given. Open point for backend-lead: recordings after midnight.

### 2.2 Days with no entry

Empty days get **no page** in the book. Turning from 3 March to the next page lands on the next day that has an entry, so the book never feels padded.
- Chapter-title page shows a small month grid of dots: filled for days with entries, empty ring for days without. It is decorative and also a quick jump (tap a day dot). Dots are real buttons with labels ("March 4, has entry").
- Jumping by calendar to a day without an entry lands on the nearest following entry and shows a short note: "No page for March 5. Showing March 7."
- Today without an entry yet: the book opens to a "Today" page with a gentle prompt and the Record button (see 7).

## 3. Opening the book

1. First ever visit (no entries): cover, then a single page: "Your first page is waiting." plus Record. No empty-book clutter.
2. Returning user: the app opens **already open to the latest page** (today if it exists, otherwise the most recent entry). Showing the cover every night would add friction to a nightly habit.
3. The cover is reachable: "Back to cover" in the chapters sheet. A short, one-off "cover opens" animation plays only on first open per session if motion is allowed, and never twice in a row.
4. State is in the URL: `/book/2026-03-14` is the page for that date; `/book` redirects to the latest. This makes back/forward, sharing within your own account, bookmarks and the PWA all work, and keeps the screen reader announcement simple.

## 4. Date navigation

### 4.1 Chapter list ("Contents")
- A "Contents" button (bookmark glyph) opens a bottom sheet on phones and a side panel on desktop.
- Lists years (collapsed, newest first), each expanding to its months, each showing entry count. Tapping a month goes to that chapter's title page.
- Current location is marked with the current-page state and `aria-current`.

### 4.2 Calendar jump
- A "Jump to date" button opens a month calendar (a native-feeling grid built on an accessible date-picker pattern, not the browser's native date input, which cannot show which days have entries).
- Days with entries are marked and focusable first; days without are visibly muted but still selectable (see 2.2).
- A year/month selector at the top for far jumps; keyboard: arrow keys move by day, PageUp/PageDown by month, Home/End to week start/end.
- Fast path: "Today" and "Latest" shortcut buttons.

### 4.3 Edge scrubber (desktop and tablet, optional in v1)
A thin timeline along the book edge showing months as ticks. Good for years of data; marked "nice to have", cut first if time is short.

### 4.4 Search
Out of scope for v1 (not in brief). The URL scheme and data fetching are designed so a later search can jump to a page.

## 5. Page-turn interaction

### 5.1 Layout by screen size
| Width | Layout |
|---|---|
| Below 768 px (phones) | **Single page** fills the screen. Turn reveals the next/previous page as if folding one leaf. |
| 768 to 1023 px (tablets, small laptops) | Single page, larger margins, book frame visible. |
| 1024 px and up | **Two-page spread**: left and right pages with a visible spine. Turning flips one leaf, revealing two new pages. |

Spread rule: because pages are days, the spread shows two consecutive entry days. The spread never splits a day across pages. When the user lands on a date through a link, that date is on the left or right as needed, so the target page is always visible.

### 5.2 Input methods (all required, brief 8.1 item 19)
| Input | Behaviour |
|---|---|
| Swipe (phones) | Horizontal drag follows the finger; the leaf curls with it. Release past about 35 percent of the width or a quick flick commits the turn, otherwise it springs back. Vertical movement scrolls the page text; we lock the direction after about 10 px so scrolling and turning do not fight. |
| Drag (desktop mouse) | Same as swipe, grabbing the outer page corner or edge. Cursor changes to a grab hand. |
| Tap zones | Tapping the outer 20 percent of the page edge turns the page (right edge = next, left edge = previous, matching reading direction). Middle area is reserved for reading and text selection. Zones are at least 44 by 44 px. |
| Buttons | Visible "Previous page" and "Next page" buttons below (phone) or beside (desktop) the book. Always present, never hover-only. They are the primary accessible path. |
| Keyboard | Left/Right arrows turn when the book region is focused. Home/End jump to first/last page. PageUp/PageDown move by chapter. Esc closes sheets. Arrow keys do not turn pages while a text field is focused (editing an entry). |
| Entry edit | Editing happens on the entry view reached from the page (D3); turning is disabled while editing to prevent lost changes. |

Direction note: the book follows reading direction (left-to-right) for v1. Right-to-left languages are a later concern [noted, not scoped].

### 5.3 Animation approach: recommendation

Options compared (versions verified against the npm registry on 2026-11-07):

| | Hand-rolled CSS 3D + small pointer-gesture code | `react-pageflip` 2.0.3 / `page-flip` 2.0.7 | `motion` 14.10.1 (formerly Framer Motion) |
|---|---|---|---|
| What it is | We write the leaf as stacked layers with CSS `transform: rotateY` and a gradient shadow, and drive the angle from the finger. | Ready-made flipbook with realistic curl, canvas or HTML mode. | General animation library; no page curl built in. We would still hand-build the leaf. |
| Bundle size | Near zero (a few KB of our own code). | Small library but its engine is bundled in the client; exact gzip size [VERIFY by bundle analysis in Phase 3]. | Larger. `motion` offers a lazy-loaded minimal feature set; still meaningful KB [VERIFY]. |
| Maintenance | Ours; we know every line. | **Last published April 2021** (both packages). Unmaintained for 5+ years. | Very active (published 2026-11-06). |
| Accessibility | Full control: real DOM, real headings, real focus order, easy to disable. | Pages are wrapped by its engine; its own keyboard and screen-reader behaviour is minimal and hard to change. [VERIFY by prototype] | Good for transitions; a11y still ours. |
| Fit with our needs | Full control of one page per day, scrolling pages, dynamic heights, virtual pages. | Assumes a fixed set of equal-size pages loaded up front, which fights virtualization and variable text length. | Helps with spring physics but adds weight for one effect. |
| 60 fps on mid-range Android | Achievable if we animate only `transform` and `opacity`, use `will-change` sparingly, and keep layers few. | Canvas mode can be heavy; HTML mode similar to CSS. [VERIFY on device] | Same constraints; no magic. |

**Recommendation: hand-rolled CSS 3D transforms with a small pointer-event gesture controller, no animation library.**

Reasons, in plain language:
1. The page turn is the signature of the product, so we should own it. A dependency untouched since 2021 is a risk for something this central (React 19, new browsers).
2. Accessibility and reduced motion are legal requirements here (brief 8.1). Owning the DOM makes them straightforward.
3. Our content is variable-length text with years of pages. Library flipbooks assume a fixed, fully loaded set of pages, which conflicts with virtualization (section 9).
4. Smallest bundle, which helps the Lighthouse target (the hub is logged-in, but the cover preview on the landing page must stay light).

How it works (short):
- At any moment only **three leaves exist in the DOM**: the current one, and one on each side (previous/next), plus the spread's partner on desktop.
- The turning leaf has a front and a back face (`backface-visibility: hidden`). Its angle goes from 0 to -180 degrees. A moving gradient overlay fakes shading; a light shadow on the page underneath fakes depth. Only `transform` and `opacity` change per frame.
- Gesture code uses Pointer Events (one code path for touch, mouse and pen), `touch-action: pan-y` on the page so vertical scroll stays native, and `requestAnimationFrame` with a spring-style ease on release.
- A "snapshot" fallback: if a device reports low performance (see 5.5), the turn becomes a quick slide plus cross-fade.

Risk and exit: if the Phase 3 prototype cannot hold 60 fps on the test phone, the fallback is the simpler slide/fade, not a library. Prototype checkpoint is task one of the book work (D16).

### 5.4 Reduced motion
- Respect `prefers-reduced-motion: reduce` at all times, and also provide an in-app "Reduce motion" switch in Settings (some people have not set the OS flag).
- With reduced motion: no curl, no 3D. The page changes with an instant swap or a 150 ms cross-fade; swipe and tap zones still work but commit on release with no drag-follow animation. The cover-opening animation is skipped.
- Reduced motion is a first-class design, tested in Playwright with emulated media.

### 5.5 Performance target for the turn
- 60 fps (frames under 16.7 ms) on a mid-range Android reference, chosen with the PO (suggest: a roughly three-year-old Moto G or Galaxy A-series) [VERIFY device choice].
- Interaction to first visual response under 100 ms.
- Use of `content-visibility` and `contain: layout paint` on non-active pages.
- Optional runtime guard: measure frame times for the first turns; if a majority exceed 24 ms, switch that device to the slide/fade fallback and remember the choice locally.

### 5.6 Page content and scrolling
- A page is a fixed-height frame; the entry text scrolls inside it with a soft fade at the bottom edge when more text follows. A small "more below" cue for screen magnifier and low vision users is a visible text/glyph, not colour alone.
- Paper texture is a light, tiled, compressed image (target under 30 KB, modern format) applied with CSS; text contrast against the texture must meet WCAG 2.2 AA (4.5:1) at its worst point [test with contrast tooling].

## 6. Keyboard and screen-reader model

- The book is a landmark `region` labelled "Your book".
- **Each page is a `section` (region) labelled by its date**, via `aria-labelledby` pointing at the page's date heading ("Friday, March 14, 2026"). Entries inside use real headings and paragraphs, so reading is natural.
- Only the current page (or current spread) is exposed. Neighbouring leaves are `inert`/`aria-hidden` and not focusable, so a screen reader never reads 30 pages.
- On every page change, a **polite live region** announces "Page for Friday, March 14, 2026. Page 12 of 31 in March." (not the entry text, which is private and long). It is debounced so rapid turns announce only the final page.
- After a turn, focus stays on the Next/Previous button that was used (or the book region if keyboard arrows were used) so users can keep turning. Focus is moved to the page heading only after a jump (calendar or chapter), where context changed.
- Previous/Next buttons have names with the destination: "Next page, March 15" (when known). Disabled at the ends with `aria-disabled` and a visible reason, not just greyed out.
- Visible focus rings (at least 3:1 contrast, 2 px) on every control; target sizes at least 24 by 24 CSS px per WCAG 2.2 and 44 px for primary controls.
- Drag has a single-pointer alternative everywhere (buttons, tap zones, keys), as WCAG 2.2 requires.
- Calendar and Contents sheets: focus-trapped dialogs that return focus to the opener (provided by the headless component kit, see D19).
- Testing: automated axe checks on every hub state, plus manual runs with VoiceOver (iOS), TalkBack (Android) and NVDA (Windows) before launch (launch checklist item).

## 7. Handoff from recording

The recording screen and the hub are separate routes, but they must feel like one flow.

1. User taps **End**. The raw transcript is saved first (locally and to the server). The recording screen shows the "writing your page..." state (crafted, matched to page layout).
2. When the rewrite finishes, the app navigates to the hub at today's date: `/book/<today>`.
3. The **new page appears** with a gentle arrival: the book opens to the previous page and the new page turns in once (if motion is allowed), so the user *sees* their page join the book. With reduced motion, the new page simply appears with a short "Added to your book" confirmation (visible text, announced politely).
4. If the user already has a page for today, the new recording becomes a new section on that page (2.1) and the page scrolls to it.
5. Failure paths: if the rewrite fails or is slow, the page appears right away showing the original transcript with a clear "Rewrite in progress / failed, retry" strip, so nothing is ever lost or blank. Offline: page marked "Saved on this device, will sync" until synced.
6. Data refresh: after save, the client refreshes the affected month's data (Next.js revalidation or client cache update, see D19) so no manual reload is needed.

## 8. Loading, skeletons and empty states

- **Skeletons match the real layout**: a page-shaped leaf with the date heading line and 6 to 8 text-line bars at realistic widths, same paper colour, same frame, so there is no layout jump. Shimmer is disabled under reduced motion (static bars).
- Cover and frame render from the server immediately (static); only the entry text streams in. The hub shell is a server component; the book itself is a client component (see D19 boundary rules).
- Turning to a page whose data has not arrived: the leaf still turns and shows the skeleton on its face; text fades in on arrival. Prefetch makes this rare (section 9).
- Empty states: first day (6.1), today not recorded (prompt + Record), no entries in a month (month is not listed in Contents), offline with no cached pages ("You are offline. Pages you opened recently are still here."), network error with Retry.
- Error states are plain language with an action (brief 8.3 item 13).

## 9. Virtualization for years of entries

Goal: a user with 5+ years (about 1,800 pages) must see the same speed as a user on day 3.

- **DOM**: at most three leaves (single) or five (spread) are mounted, as in 5.3. Pages never "exist" in the DOM beyond that window.
- **Data**: never load the whole book. Load by **month chapter**:
  - A lightweight index (dates, entry ids, a one-line snippet length, chapter counts) for the whole book: tiny even for years, loaded once and cached. It powers Contents, calendar dots and prev/next resolution.
  - Full entry text for the current page plus a window of about 3 pages each side, prefetched on idle, evicted beyond about 15 pages.
- **Cache**: an in-memory cache keyed by entry date, plus IndexedDB for recently opened pages so the book opens instantly and works offline (read-only offline in v1; edits queue per D5).
- **Jumps** do not animate through every page between. A jump across more than 2 pages shows a short fade or a fast "riffle" (3 fast leaf flicks at most, skipped under reduced motion), then lands.
- **Server**: queries are by date range with an index on (user, entry_date); pagination by chapter. Owned by backend-lead (D6) [dependency noted].
- Images: none on pages in v1 (photos are deferred), which keeps this simple.

## 10. Performance budget

| Metric | Target |
|---|---|
| Hub first load, mobile (mid-range Android, 4G, logged in) | LCP under 2.5 s; the open page text is visible without waiting on animation code |
| JS shipped for the hub route | Book client code under about 40 KB gzip, excluding framework; whole route under about 170 KB gzip [VERIFY with bundle analyzer; set as CI budget in Phase 3] |
| Page turn | 60 fps, response to first input under 100 ms |
| INP (interaction delay) | under 200 ms (Core Web Vitals "good") |
| Layout shift (CLS) | under 0.1; skeletons prevent jumps |
| Texture and cover images | Cover under 60 KB, paper texture under 30 KB, modern formats, sized for display |
| Memory | No growth over 200 consecutive turns (leaf elements are reused, not recreated) |
| Public pages | Lighthouse 90+ in all four categories (brief 8.3, item 18); the hub is behind login so Lighthouse CI also audits it with a logged-in test account [VERIFY in D19] |

## 11. Phone behaviour summary

- 360 px minimum, no horizontal scroll; safe-area insets respected (notches, home indicator).
- Thumb zone: Previous/Next, Contents, Jump, and the Record button sit in a bottom bar within thumb reach. Record is the most prominent control.
- Pull-to-refresh and browser back-swipe on edges must not collide with page-turn: the outer 16 px of the screen edge is left to the OS, so the turning drag starts inside that.
- Orientation: portrait is primary. In landscape on a phone, still single page, text frame scrolls.
- PWA: standalone display; the hub is the start URL.

## 12. Open questions for the PO

1. **Open to the cover or to today?** Recommendation: open straight to the latest page; cover on first run and via "Back to cover".
2. **Multiple entries per day**: one page with time-labelled sections (recommended) vs. a separate page per recording.
3. **Long entries**: scrolling inside a page (recommended for v1) vs. true typeset multi-page layout (later).
4. **Which reference phone** counts as "mid-range" for the 60 fps test?
5. **Chapter titles**: just the month name, or can users add a chapter title/note (deferred)?
6. **Which of the 2 to 3 book directions** (leather, minimal, scrapbook) from Phase 2: affects cover, texture and turn shading but not structure.
7. **Edge scrubber** (4.3): worth including in v1 or later?

## 13. Acceptance criteria (for D16 task briefs)

- All five input methods turn the page; vertical scroll within a page never triggers a turn.
- With `prefers-reduced-motion`, no 3D or curl, and all functions still work.
- Each page is a region labelled by date; the live region announces page changes once, politely; Playwright + axe pass on all hub states.
- Book with 2,000 seeded entries: DOM leaf count constant, turn stays 60 fps on the reference device, initial open under budget.
- New recording lands on its page within one navigation, with no manual refresh.
- No horizontal scroll at 360 px; targets at least 44 px for primary controls.
- Journal text is never sent to analytics, error tracking or logs from the hub.
