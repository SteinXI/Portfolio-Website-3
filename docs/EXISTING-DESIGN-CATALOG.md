# Existing Design Catalogue — Portfolio Website 3

**Document ID:** WEB-DESIGN-CAT-001  
**Catalogue version:** 1.0 (source-based baseline; visual confirmation pending)  
**Reviewed:** 18 October 2026  
**Repository audited:** `SteinXI/Portfolio-Website-3`  
**Relationship:** Companion to [Website Architecture Specification](WEBSITE-ARCHITECTURE.md), especially §7G (consistency and preservation).

## 1. Scope, evidence and reading rules

This catalogue records the **design choices currently expressed in source**, to protect them when moving toward flexible sections, Supabase-managed defaults, and an Astro renderer. It is not a redesign brief.

Inspected: `index.html`, `experience.html`, `css/style.css`, `css/editorial-v2.css`, and `js/script.js`; legacy section names and slot counts were cross-checked against `docs/CONTENT-STRUCTURE.md`. Page HTML currently contains neutral placeholders, **not the final user's content**.

**Critical evidence limit:** This is a **source inspection**, not a browser computed-style or screenshot comparison. CSS precedence depends on specificity, `!important`, media conditions, source order, element state, and the viewport. Statements below describe source-identified design intentions and **candidate preserved presets**; exact rendered visual details must be confirmed against an approved screenshot and the source/reference repository (`Portfolio-Website-2`). If a class has conflicting historical declarations, preserve the **effective current rendered appearance**, not an earlier overridden rule.

**Do not do during catalogue stage:** Do not rewrite classes, normalise font sizes, replace existing padding with generic tokens, change colours, remove `!important`, alter the reference repository, or assume every section should use the same layout.

## 2. Shared website design — cross-cutting catalogue

| ID / proposed preserved preset | Source evidence | Existing intended exception to generic defaults | Migration instruction |
| --- | --- | --- | --- |
| `GLOBAL-EDITORIAL-LIGHT` | Both stylesheets; editorial file explicitly describes a white five-colour design | White/near-white surfaces, dark text, strong red/navy/cyan/green/gold accents, thin borders | Preserve the **effective** later `editorial-v2.css` values, not superseded earlier dark designs |
| `GLOBAL-TYPE-EDITORIAL` | Base `body` Arial/Helvetica, 16px, 1.7; editorial type overrides; Bodoni Moda Google Fonts | Body sans-serif plus distinctive Bodoni Moda editorial headings; responsive type; role-specific label sizes | Preserve font-family roles, weight, responsive clamp, letter spacing; proposed generic typography is for **new** default components |
| `GLOBAL-SECTION-RHYTHM` | `body:not(.experience-standalone) .content-section`; late `@media screen` rules | Deliberate internal vertical padding; final screen rules set `padding-top: clamp(49px,5.1vw,76px)` and bottom `clamp(51px,5.5vw,80px)`; mobile `45px 52px` | Keep **zero gap between section surfaces** separate from their nonzero interior padding; preserve current rhythm in migrated sections |
| `GLOBAL-DIVIDERS` | `::after` rules for multiple sections; current editorial colour `#D5D8DB` | Thin editorial dividers, in some cases stretched across the content/nav offset | Preserve border/divider behaviour and alignment independently from inter-section gap |
| `GLOBAL-RESPONSIVE` | Multiple `@media` rules at 900/768/520 etc. and final screen overrides | Some sections have bespoke breakpoints; full site is **not** presently governed by one uniform mobile switch | Record/compare all effective breakpoints before replacing them with candidate global tokens |
| `GLOBAL-PRINT` | `@media print` in `editorial-v2.css` | Navigation/tooltips hidden **only when printing**; narrower text, simpler section and page layout; controlled page breaks | Preserve print media behaviour; do not misdiagnose print-only `display:none!important` as screen navigation defect |

**Source precedence caution:** `css/style.css` loads **before** `css/editorial-v2.css`. The latter contains later, more specific and often `!important` selectors. It also contains late `@media screen` blocks explicitly described as a balanced editorial spacing adjustment. Consequently, a reader looking only at the first appearance of a CSS declaration may report the wrong design. A computed-style baseline remains required.

## 3. Homepage: nine section-specific profiles

All nine homepage sections are in `index.html` and have `data-area-code` labels. These labels are **migration references only**, not permanent UUIDs. The proposed preset IDs below are names for future design records, not existing working code.

### 3.1 HOME — Introduction / hero (`#home`, `.Intro`)

**Candidate preset:** `PRESERVE-HOME-INTRO-EDITORIAL`

- **Structure:** Dedicated first section with introduction label, large H1, description, interests and links/buttons. Unlike a normal content section, it uses `.Intro-box`.
- **Visual character:** White background; five-segment red/navy/cyan/green/gold accent strip at the top; editorial black Bodoni Moda main title; compact red label; divided description, coloured interest underlines.
- **Intentional layout:** Near-full-screen hero. The final screen rules set `.Intro` and `.Intro-box` to at least `100svh`. Desktop has an asymmetrical two-column grid, and the `min-width:769px` override uses approximately `1.25fr / .85fr` with a minimum right-column width. Mobile switches to a vertical flex layout. Height is a **minimum**, not a mandate to clip extra content.
- **Typography exception:** Final desktop hero H1 uses `clamp(4.4rem,7.7vw,8rem)`, well above the generic H1 proposal. Preserve this as an **explicit editorial hero variant**.
- **Migration cautions:** Maintain grid-to-flex transition, label/description relationships, colour stripe, full-screen feel, button focus behaviour, and readable content when text grows. Do not reduce H1 to generic 36–48px automatically.
- **Visual check:** Full intro height, browser address-bar/mobile viewport, H1 line breaks at several widths, and button positioning.

### 3.2 ABOUT — Biographical editorial layout (`#about`, `.about-section`)

**Candidate preset:** `PRESERVE-ABOUT-EDITORIAL-SPLIT`

- **Structure:** Two text regions in `.about-layout`: editorial heading/label/reflection plus biography; supplementary journey-line graphic.
- **Layout exception:** Unequal two-column grid (source varies around `.88fr / 1.38fr` with a broad internal gap); stacks to one column on smaller screens.
- **Typography/colour:** Large Bodoni Moda H2; navy label/accent in the later stylesheet; narrower biography paragraph scale and intentionally managed line height/paragraph margins.
- **Spacing:** Meaningful whitespace between the heading and body columns; not the generic `16px` group gap.
- **Migration cautions:** Preserve title wrapping/line length, line illustration, paragraph rhythm, and column ratio. Model as a section with nested groups rather than a monolithic text element.
- **Visual check:** Heading/biography vertical alignment and flow at narrow widths.

### 3.3 FOCUS — Areas of Focus (`#research`, `.focus-section`)

**Candidate preset:** `PRESERVE-FOCUS-FIVE-COLOUR-GRID`

- **Structure:** Heading/intro plus six existing `.focus-item` slots in `.focus-grid`; original items have their own headings and prose.
- **Important historical override:** Base `style.css` defines a dark research section; the later `editorial-v2.css` intentionally replaces it with a **white editorial section**. Preserve the **white effective current design** unless a separate redesign is authorised.
- **Distinctive grid:** Editorial CSS uses a six-track grid: **first two items span three tracks each** (two wide cards in first row); later four span two tracks each (three items per row). Thin grid borders and compact numbered/accent treatment are part of its designed appearance; item number display may be hidden by current CSS.
- **Responsive:** Reduces to two columns at `max-width:768px`; one column at `max-width:520px`.
- **Migration cautions:** Do not use a generic equal-card grid or restore old dark colours. Introduce a flexible preset for the **first-row emphasis rule** that can degrade gracefully when entries are added/removed.
- **Visual check:** Exact border continuity, position-based accent sequence, keyboard/reading order and behaviour when there are fewer/more than six items.

### 3.4 EXPERIENCE — Featured professional experience (`#experience`, `.experience-section`)

**Candidate preset:** `PRESERVE-EXPERIENCE-EDITORIAL-LIST`

- **Structure:** Label, large editorial H2/intro and three existing featured `.experience-entry` cards, plus route to detailed experience page.
- **Layout exception:** Metadata/details arranged in unequal two-column rows with horizontal dividing lines; the final screen rules tighten row padding to `22px 0 25px`. Narrow widths stack rows into one column.
- **Typography:** Large serif section title, comparatively compact role/entry headings, restrained body copy, dates and tags.
- **Migration cautions:** Preserve featured versus full listing distinction. Reuse shared experience content records with separate compact/detailed presentation presets. Allow more/less than three records in the future database without fixed HTML slots.
- **Visual check:** Row alignment, links, long role names, bullets/tags and no unintended blank padding.

### 3.5 EDUCATION — Academic timeline (`#education`, `.education-section`)

**Candidate preset:** `PRESERVE-EDUCATION-TIMELINE`

- **Structure:** Editorial intro plus five existing `.education-entry` slots.
- **Layout exception:** Custom three-track row (`minmax(110px,150px) 30px minmax(0,1fr)`) for metadata, timeline/gutter and details, with mobile alterations. Not a generic card stack.
- **Colour/detail:** Individual entries use `--school-accent` inline variables; later stylesheet adjusts borders to pale grey, and heading/prose sizes remain role-specific.
- **Migration cautions:** Keep timeline line/marker geometry and per-entry accent as **validated local data/tokens**. Do not silently flatten into generic text cards.
- **Visual check:** Vertical marker alignment, long institution names and mobile stacking.

### 3.6 LANGUAGES — Language proficiency (`#languages`, `.languages-section`)

**Candidate preset:** `PRESERVE-LANGUAGES-RATING-GRID`

- **Structure:** Heading/intro plus four `.language-cell` entries in `.language-grid`; individual glyphs, level labels and star icons.
- **Historical override:** Earlier base CSS defines dark backgrounds and light text; later editorial CSS replaces those with **white panels, dark text and light-grey borders**. The gold/neutral star colours are specific to the editorial treatment.
- **Responsive:** Base four-column layout with responsive transitions (including two-column and eventually one-column behaviours). Later editorial code also has two-column rules in its responsive cascade: verify exact final per-breakpoint computed result.
- **Migration cautions:** Keep meaningful proficiency text visible even if decorative stars are removed or misrendered; new items should not assume a hard limit of four.
- **Visual check:** Cell dividers, star contrast, 4/2/1-column transitions and screen-reader meaning of proficiency.

### 3.7 CREDENTIALS — Tabbed credential directory (`#credentials`, `.credentials-section`)

**Candidate preset:** `PRESERVE-CREDENTIALS-TABBED-DIRECTORY`

- **Structure:** Section heading, **three tabs/panels**, and **14 existing `.credential-item` article slots** divided by category. Content uses item body, issuer, date and decorative award icon.
- **Interaction exception:** `js/script.js` implements click plus ArrowLeft/ArrowRight/Home/End keyboard operation, `aria-selected`, focus and hidden panels. This is a behavioural pattern to preserve, not merely a colour style.
- **Design:** White bordered directory; later stylesheet changes the selected tab from older gold style to **navy**, reduces internal gaps and tightens item padding (final `15px 12px`).
- **Migration cautions:** Dynamic content schema must represent categories/panels and relationships, not just 14 fixed slots. Preserve accessible focus and panel visibility; test panel `aria` relations after component conversion.
- **Visual check:** Tab alignment, hidden-panel logic, keyboard navigation, item density on mobile, focus indication.

### 3.8 HOBBIES — Beyond Work (`#hobbies`, `.hobbies-section`)

**Candidate preset:** `PRESERVE-HOBBIES-ACCENTED-ROWS`

- **Structure:** Intro plus four existing `.hobby-card` entries in `.hobbies-grid`.
- **Layout/design exception:** Two-column grid with tight/no row gap; individual hobby cards are transparent, horizontally organised icon + text rows with bottom borders. Per-position accent colours: red/gold/cyan/green.
- **Responsive:** Reduces to one column at narrower viewports; later screen rule reduces card padding to `17px 0`.
- **Migration cautions:** Preserve the light, non-card-box visual treatment, distinct icons/accent colours, and orderly reading sequence. New hobby items should not rely on `:nth-child()` for permanent semantic colour identity unless that is intentional.
- **Visual check:** Last-item border rules, column order and mobile icon/text alignment.

### 3.9 CONTACT — Let's Connect (`#contact`, `.contact-section`)

**Candidate preset:** `PRESERVE-CONTACT-EDITORIAL-ACTIONS`

- **Structure:** Intro label, large editorial heading with decorative globe/earth icon, multilined introductory message, one primary contact action and secondary contact methods.
- **Historical override:** Earlier CSS contains a dark section and cyan typography. The later stylesheet converts this to the **white editorial language**, dark text, navy/coloured hover accents and borders; source footer also becomes light.
- **Layout/design exception:** Specific primary-versus-secondary action arrangement, link-arrow motion, email wrapping and controlled custom spacings. Later desktop/mobile rules adjust the heading, intro text and contact gaps.
- **Migration cautions:** Contact destinations become editable content records but must be explicitly validated; do not reintroduce dark styles or alter live addresses/links during structural refactoring.
- **Visual check:** Links, hover/focus, long email addresses, responsive icon positioning and colour contrast.

## 4. Detailed professional experience page and its sections

The separate `experience.html` page uses `body.experience-standalone` and has **its own page shell**; it is **not** just the homepage Experience section repeated verbatim.

| Candidate preset | Present structure | Distinctive styling / intentional exception | Preserve/test |
| --- | --- | --- | --- |
| `PRESERVE-EXPERIENCE-DETAIL-SHELL` | Top bar, brand/back link, page H1 and intro | White stand-alone page; width constrained to about `1180px`; separate back/navigation bar; serif page title | Route from/to homepage, keyboard focus, heading hierarchy, responsive page padding |
| `PRESERVE-EXPERIENCE-DETAIL-HISTORY` | Employment history section; **9** existing experience articles | Two-column meta/details row (`.86fr / 1.6fr`), border-separated, stacked on narrow screens | Full detail vs featured summary; list bullets, dates, long content; dynamic record count |
| `PRESERVE-EXPERIENCE-DETAIL-SKILLS` | Skills section with two `.experience-skill-column` groups | Serif H2; two-column skills grid, responsive single column; deliberately separate section pacing | Multi-list hierarchy and column reading order |
| `PRESERVE-EXPERIENCE-DETAIL-LANGUAGES` | Language section, four value cells | Compact four-column language list with horizontal boundaries; adaptive two-column behaviour | Consistent data reuse and accessible labels |
| `PRESERVE-EXPERIENCE-DETAIL-CONTACT` | Contact section and links | Serif heading, compact contact copy, explicit links | Link destinations and appropriate text wrapping |

**Legacy annotation warning:** `EXPERIENCE_DETAIL` appears as `data-area-code` on both the page `main` and its nested history `section`. This is **not** a unique identifier. During conversion create separate UUIDs for the page, history section and other sections; `data-area-code` remains only a temporary mapping label.

## 5. Shared interactive interface and printed output

| Preset / behaviour | Source | Intentional exception | Migration safeguard |
| --- | --- | --- | --- |
| `PRESERVE-FLOATING-NAV` | `.floating-nav`, mobile menu, `js/script.js` | Navigation becomes visible after the introduction and is suppressed near the footer. Current editorial sidebar has a white bordered look; mobile button/menu have different layouts. | Preserve active-state logic, anchor targets, click-outside/Escape actions, reduced obstruction and keyboard access. Test mobile opening and hiding. |
| `PRESERVE-FOOTER` | `.site-footer` and editorial overrides | White footer with dark text, navy links, red top-accent line and deliberate spacing | Preserve visual boundary, back-to-top link, text and responsive behaviour |
| `PRESERVE-PRINT` | `@media print` | Hide floating navigation, mobile button, skip link and tooltips **only in print**; simplify sizing/page breaks | Test browser print/PDF separately; the print-only hide rule is not a normal-screen defect |
| `PRESERVE-KEYBOARD-ACCESS` | Skip-link, focus-visible styles, tab JS | Functional accessibility not fully expressible as a colour/layout preset | Retain skip-to-main, tab focus, valid controls, headings, and ability to operate with keyboard |

## 6. Design categories: preserved presets vs new generic defaults

**A. New generic section / element** — uses the approved default-first tokens: site typography, `auto` height, valid responsive sizing, section-to-section gap `0`, and modest adjustable inner padding.

**B. Existing migrated section** — uses a **named preserved preset** containing section-specific typography, group layout, accent, spacing, dividers, responsive policy and interaction associations where applicable. A preserved preset may reference shared tokens for allowed properties but does **not** automatically inherit every later global size change.

**C. New variant based on existing design** — explicit clone/reuse of a named preset. It receives its own reference/variant and is previewable and reversible. Structural UUIDs stay independent of preset names.

**D. Local exception** — validated custom value for a particular node/element, including unusual type scale, timeline accent or section-specific gap. Must not affect siblings and must pass publication checks.

**Preset ≠ structure:** A preset can be attached to a section/group/component but must not be encoded into permanent UUID or hierarchical path. No fixed content slot counts.

## 7. Open issues and remaining work — not classified as source-code errors

| Priority | Task | Reason / exit criterion |
| --- | --- | --- |
| High | Establish approved reference screenshots (desktop, tablet, mobile, print) | The source audit cannot prove computed/rendered visual parity or pixel-equivalence |
| High | Resolve intentional component presets and effective CSS properties through browser inspection | Current style sheets contain repeated rules, media overrides and `!important`; document final rendered values before refactoring |
| High | Decide how legacy slots map to flexible node UUIDs / shared content records | Fixed `FOCUS_01`, etc. labels are not permanent database entities |
| High | Test existing behaviours: navigation, tabs, links, keyboard, screen-reader semantics | CSS/HTML copying is not equivalent to behavioural verification |
| Medium | Clarify first-row focus grid algorithm when count differs from 6 | Current `:nth-child` layout is position dependent |
| Medium | Decide accent colour policy when new education/hobby entries are added | Existing entries have either inline `--school-accent` or position-dependent accents |
| Medium | Confirm legacy desired typography and section padding at specific viewport widths | Generic recommended scales differ intentionally from existing editorial design |
| Medium | Validate real content lengths and media behaviour after replacing placeholders | Placeholder lengths do not fully exercise overflow/layout |
| Later | Replace hardcoded colour/spacing/CSS overrides with manageable tokens and versioned presets | Must happen after proof of visual parity, not as an automatic cleanup |

### Proposed checklist before any CSS migration

- [ ] Capture and approve visual screenshots for each section at representative desktop and mobile widths.
- [ ] Record computed font/colour/padding/grid/border values for each legacy section.
- [ ] Confirm final list of preserved design preset names, allowed property overrides, and content relationships.
- [ ] Verify responsive behaviours, print layout, keyboard navigation, credentials tab interactions and footer visibility.
- [ ] Record a migration table mapping each legacy area/entry label to a new permanent UUID.
- [ ] Write regression checks comparing source/reference and newly generated website.
- [ ] Do not alter `Portfolio-Website-2`, and avoid private details or secrets in this public repository.

---

**Classification:** Source-backed catalogue of current designs; **not** a visual approval, feature implementation or website update log. This document describes how to preserve design intent while introducing the selected flexible database-backed static-publishing architecture.
