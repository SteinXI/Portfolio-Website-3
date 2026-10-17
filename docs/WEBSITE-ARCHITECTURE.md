# Portfolio Website — Infrastructure & Structural Design Specification

**Document ID:** WEB-ARCH-001  
**Document version:** 1.4 (default-first design rule)  
**Date:** 17 October 2026  
**Canonical location:** `docs/WEBSITE-ARCHITECTURE.md` in `SteinXI/Portfolio-Website-3`  
**Scope:** Site architecture, data and structural conventions, identity, hierarchy, planned design capabilities, security, publishing and technical roadmap.  
**Excluded:** Website modification activity log, chronological deployment notes and individual update records. Maintain those separately if needed.

> **Important distinction.** This document captures established architectural decisions and clearly labelled proposals. It is **not** a statement that the proposed database, CMS, Astro renderer, or adaptive block engine is already implemented.

## 1. Purpose and objectives

The Portfolio Website is a personal academic and professional portfolio website. The long-term design must separate **content**, **presentation**, and **publishing** so that content can be edited independently of HTML and CSS.

Primary goals:

- Preserve the design and textual reference in `SteinXI/Portfolio-Website-2`; develop safely in `SteinXI/Portfolio-Website-3`.
- Support new pages, subpages, sections, optional groups, nested groups, and flexible elements without hard-coded quantity limits.
- Support mixed ordering of elements and groups; neither a section nor a group must contain another group.
- Reuse content records across pages while controlling how those records appear.
- Permit draft, preview, approval, publishing, archiving, restoration and revision history.
- Allow responsive and content-length-aware layouts, avoiding clipping or unreadably shrunken text.
- Prioritise GitHub Free and other free tiers while documenting their limits rather than promising permanently free operation.
- Keep public visitors independent of live database availability.

## 2. Established architecture and status

### 2.1 Selected publishing approach

**Selected:** database-backed **static publishing**, not a live database query for each public page view.

Proposed logical flow:

```text
Private administrator
       |
       v
Secure editor / authenticated authoring service [future]
       |
       v
Supabase PostgreSQL + authentication [proposed]
  drafts, structured content, published revisions
       |
       v
Restricted build-time reader [future]
       |
       v
Astro static generator + GitHub Actions [proposed]
       |
       v
GitHub Pages [target hosting]
       |
       v
Public website (HTML/CSS/JS/assets; no live database dependency)
```

A successful publication takes a **consistent published snapshot** of content; a build must not mix draft and published states. A broken build must not replace a successfully deployed site.

### 2.2 Selected versus proposed

| Decision | Status |
| --- | --- |
| Option A — static publishing | **Selected** |
| Separate public presentation from editable content | **Selected** |
| Flexible, container-based structural framework (Approach 2) | **Selected direction** |
| Optional reusable templates built from primitives, not compulsory fixed section templates | **Selected direction** |
| Public hosting on GitHub Pages (GitHub Free) | **Target** |
| Supabase PostgreSQL and Supabase Auth | **Proposed, not provisioned** |
| Astro static site generator | **Proposed, not installed** |
| GitHub Actions publish workflow | **Proposed, not configured** |
| Private content editor and preview interface | **Proposed, not built** |
| Individual building-block catalog and visual variants | **Not finalised** |

## 3. Repository baseline and separation of responsibilities

### 3.1 Repositories

- **Source/reference:** `SteinXI/Portfolio-Website-2`. Original content and design reference; do not modify for this infrastructure project.
- **Development:** `SteinXI/Portfolio-Website-3`. Public repository holding the content-free template and structural scaffolding.
- Source was copied and initially Git-blob-verified. In repo 3, personal text and direct personal contact links were replaced with neutral placeholders. CSS/JavaScript were preserved at that stage.
- Repo 3 has structural `data-area-code` and `data-entry-code` annotations. These describe current presentation slots, **not** the final content model.

### 3.2 What belongs where

| Location | Keep here | Do not keep here |
| --- | --- | --- |
| Public GitHub repository | Layout components, code, styles, schema/migration definitions, non-secret config, documentation | Credentials, sensitive records, private drafts |
| Supabase (planned) | Editable source content, relationships, admin permissions, revisions, draft/published state | Privileged secrets embedded in public page output |
| GitHub Pages output | Intentionally published content embedded in generated static pages | Drafts or data expected to remain confidential |
| Protected backup location | Recoverable database exports, documented restoration material | Publicly accessible secrets or private backup dumps |

**Security implication:** A public repository can be cloned by anyone. Current `content/collections/*.json` files are empty templates, not confidential data. Any published static output is public even if its source was held in a private database.

### 3.3 Existing files versus future layout

Current: `index.html`, `experience.html`, `css/style.css`, `css/editorial-v2.css`, `js/script.js`, `content/areas.json`, `content/schema.json`, six empty JSON collections, `docs/CONTENT-STRUCTURE.md`, `INFRASTRUCTURE-PLAN.md` and `README.md`.

The content JSON files are **interim scaffolding**. They do not drive page rendering. A future Astro project will likely use `src/pages/`, `src/components/`, `src/lib/`, `public/`, `supabase/`, and `.github/workflows/`, but these locations are **not yet committed as the final directory tree**.

## 4. Structural hierarchy and terminology

### 4.1 Five principal concepts

```text
SITE
  +-- GLOBAL (shared navigation/footer and possibly other shared structures)
  +-- PAGE (root page or subpage)
        +-- SECTION
              +-- ELEMENT
              +-- GROUP (optional)
                    +-- ELEMENT
                    +-- GROUP (nested, optional)
                          +-- ELEMENT
```

- **Site:** root configuration and overall page collection.
- **Global:** components intended to appear consistently across pages (e.g. navigation, footer). Reusable but non-global structures are a separate concept.
- **Page:** standalone destination, including subpages; subpages are simply pages with a `parent_page_id`.
- **Section:** major container owned by a page; may contain elements and groups in any sequence.
- **Group:** optional organisational/layout container; may contain elements and other groups in any sequence, subject to safe nesting limits.
- **Element:** individual content/behaviour unit; its actual category is defined later.

A section **does not have to** contain groups. Groups **do not have to** contain elements immediately; nested groups are allowed.

### 4.2 Page and subpage relationship

Pages form a hierarchy through `parent_page_id`, but their content still follows `page → section → group/element`. A page route might be `/research/` and its subpage `/research/project-a/`. URL slugs are separate from permanent identifiers. Route collisions and circular parent relations must be prohibited.

### 4.3 Reuse and ownership

- Every structural item has one **canonical parent** for purposes of hierarchy and numbering.
- Reuse content in multiple places via a **reference/instance relationship**, not by giving one item multiple contradictory parents.
- Shared content and visual placement are different concerns: one record can be rendered as a brief item on a homepage and in full on its own page.
- Global components and reusable components are not synonymous: globals are site-wide placements; reusable components are selectable instances.

### 4.4 Structural operations

Future editor should permit **create, rename, insert, reorder, move, duplicate, enable/hide, archive, restore**, and optionally copy or reference a reusable item. Deleting a group must explicitly distinguish (a) archive its subtree from (b) relocate children to parent.

Avoid parent cycles, orphan records, duplicate routes, and excessive nested depth. Exact UI safeguards and maximum supported depth will be chosen at implementation.

## 5. Identity, numbering and naming

### 5.1 Two different IDs

**Permanent internal ID**: UUID version 4, generated automatically and never used to express page location or display order.

Format: `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`, where `x` is a hexadecimal digit and `y` is 8, 9, a, or b. Example: `550e8400-e29b-41d4-a716-446655440000`.

- UUIDs consist of 128 bits; UUIDv4 has 122 random bits, allowing approximately 5.3 × 10^36 possible random identifiers.
- They do **not** encode a name, date, website position, document text, or private facts.
- Use database uniqueness/primary-key constraints even though collision probability is negligible at portfolio scale.
- The practical number of items is constrained by database quotas, application performance, and editor usability, **not by ID exhaustion**.
- UUID remains constant across moves, edits, visibility changes, style changes, and archival/restoration.
- New copies normally receive new UUIDs; an existing item restored from archive retains its UUID.

**Hierarchical display code**: generated from the **current** ancestor chain, component kinds, and sibling positions. It changes when the item moves or its sibling ordering changes.

### 5.2 Structural type codes

| Prefix | Meaning |
| --- | --- |
| `P` | Page, including subpages |
| `S` | Section |
| `G` | Group |
| `E-X` | Element; `X` is its provisional one-letter category |

Full codes join segments with periods, e.g. `P01.S02.G03.E-H01`.

**Important:** A code identifies the item **type** and its **structural location**, but not necessarily the number of visual columns or whether siblings appear side-by-side.

### 5.3 Numbering rules

1. Pages use `P01`, `P02`, etc. at their own sibling level. Subpages can extend their parent's path, e.g. `P02.P01`; actual routing is independent.
2. Sections restart with `S01` for each page.
3. All **mixed children of one parent share one ordinal sequence**, even if some are groups and others are elements.
4. Any new parent restarts its own child sequence at `01`.
5. Omitting a group does **not** create a missing numbering level.
6. Use at least two digits (`01`–`99`), expanding naturally to `100` and beyond.
7. Display codes are **derived**, not primary keys and not stable external links.
8. The category prefix reflects the current element type; if type changes, the display code can change but UUID does not.

```text
P01                          Home page
  P01.S01                    Section 1
    P01.S01.E-H01            Heading directly in section (position 1)
    P01.S01.G02              Group alongside heading (position 2)
      P01.S01.G02.E-T01      Text directly in group (position 1)
      P01.S01.G02.G02        Nested group (position 2)
        P01.S01.G02.G02.E-I01    Image inside nested group
      P01.S01.G02.G03        Another nested subgroup, sibling of G02
        P01.S01.G02.G03.E-T01    Text inside that sibling subgroup
    P01.S01.E-T03            Text directly in section, not inside G02
  P01.S02                    Next section: numbering restarts
    P01.S02.E-H01            First child of section 2
P02                          Another top-level page
  P02.S01                    Its first section
```

**How to read ancestry:** `P01.S01.G02.G02.E-I01` belongs inside subgroup `G02`, itself inside main group `P01.S01.G02`. `P01.S01.G02.G03` is a sibling of the nested `G02`, not inside it. `P01.S01.E-T03` is alongside the main group at section level.

The full code shows **parentage and structural depth**; CSS layout settings, not the numbering, determine physical left/right/stacked placement.

### 5.4 Separate order, name and route fields

| Field | Responsibility |
| --- | --- |
| `id` | Permanent UUID |
| `parent_id` / `parent_page_id` | Canonical parent UUID |
| `kind` | `page`, `section`, `group`, `element` |
| `element_type` | Category such as `H` or `I`, when kind is element |
| `name` | Editable administrative display name |
| `sort_order` | Order among siblings; e.g. 10, 20, 30 |
| `slug` | Human-readable page URL fragment, for pages |
| `status` | Draft/published/archived state (publishing model needs refinement) |
| `visible` | Whether an eligible item is displayed |
| `created_at`, `updated_at` | Creation and latest update timestamps |
| `revision` | Current revision number (history stored separately) |

`sort_order` can use gaps such as 10/20/30 to simplify insertions; numbering displays consecutive 01/02/03 calculated from the sorted siblings. Reordering need not rewrite UUIDs.

Use descriptive, `snake_case` property names. The human-facing short code can be displayed in an editor, but the permanent UUID should be the actual key used by APIs and database relationships.

## 6. Element categories, levels and variants — provisional

The **framework** is decided before the building-block catalogue. Codes below are candidates, **not final implementation requirements**.

| Code | Category | Intended scope / boundary |
| --- | --- | --- |
| `E-H` | Heading | Headings and subheadings, with semantic heading level |
| `E-T` | Text | Paragraphs, statements and longer prose |
| `E-I` | Image | Images, photos, illustrations, logos; captions/alt text as properties or associated blocks |
| `E-L` | List | Ordered/unordered/nested lists |
| `E-M` | Media | Audio, video, animation or supported embeds |
| `E-C` | Card | **Provisional**; may instead be a styled group containing other elements |
| `E-B` | Button/link | Navigational and action controls |
| `E-D` | Divider | Visual dividers, with accessibility-appropriate semantics |
| `E-F` | Form | Form area or controls; needs secure processing if interactive |
| `E-N` | Number | Statistics, percentages and key figures |

Because one-letter codes are limited and can conflict, maintain a registry that prevents duplicate assignments. Do **not** choose detailed element types/variants until the building-block stage.

### 6.1 Three different concepts

- **Element category** (`E-H`, `E-T`, `E-I`): what the component is.
- **Semantic level** (e.g. heading `H1`, `H2`, `H3`): document meaning, where applicable.
- **Design variant** (e.g. `V1`, `V2`, `V3`): visual styling; may be specific to category.

Thus `P01.S02.G01.E-H02` = second sibling in a group and of type heading. The trailing `02` **does not mean H2** and does not mean Variant 2. Those are separate properties.

### 6.2 Which elements require semantic hierarchy?

- Heading elements need meaningful HTML heading levels (`h1` to `h6`) based on the actual document outline. Default convention: one main H1 per page; H2 for major sections; H3 for nested headings, as appropriate.
- A visually large text label is not necessarily semantically H1; semantic and visual size must remain independent.
- Images do **not** have H1/H2/H3 levels. `Image V1/V2/V3` would mean design variants (for example full-width, portrait-framed or square), to be specified later.
- Text, lists, media, buttons and numbers generally have **presentation choices**, not independent heading hierarchies.
- Nested list structure and form label relationships have their own accessibility semantics; do not force them into heading levels.

### 6.3 Composite content

A card with heading, image and paragraph may be better modelled as a **group with a card appearance**, rather than a monolithic element. Decide this at building-block design. Background is generally a style property, not a separate structural element.

## 7. Presentation capabilities to design later

These are requirements discussed, **not configured or implemented**.

- Reusable primitives and optional saved templates composed from primitives; never force a whole section into one template.
- Content-aware flow: text wraps naturally; containers grow with content; prevent overflow/clipping; do not blindly shrink fonts until unreadable.
- Responsive arrangements: columns may stack, redistribute, or use constrained widths on smaller screens.
- Configurable placement and width (e.g. left/centre/right; full-width/fractional widths), with valid responsive rules.
- Predetermined style tokens for heading levels and other text, plus carefully controlled overrides.
- Backgrounds at **site**, **page**, **section**, and where appropriate **group/element** levels. Options may include colour, transparency, image, GIF, and gradients, with overlays, positioning and fallbacks.
- Inheritance: lower containers inherit upper-level defaults until explicitly overridden.
- **Default gap between adjacent sections: 0**; separate internal padding and spacing between child blocks.
- Allow page/subpage background choice independently, with consistent theme defaults.
- Maintain accessibility: legible typography, colour contrast, image alt text, reduced-motion support for animation/GIF, keyboard order, heading outline and mobile readability.
- Avoid promising free-form arbitrary CSS editing inside the CMS; prefer safe validated settings and reusable tokens.

**Layout versus hierarchy:** Two sibling groups may be side-by-side or vertically stacked. Their position in the document tree indicates ownership/order; the visual layout configuration controls placement.

## 7A. Design-layer hierarchy and inheritance contract

**Scope:** Design-layer framework only. This section defines where presentation rules live and how they are resolved; individual element looks and variant catalogues remain **to be decided**.

### 7A.1 Structural ownership and visual design are independent

A node's **structure** answers "what is its parent and in what reading order?". Its **design configuration** answers "how does it display?". A group can contain two sibling groups without implying they appear in two columns; the group's layout setting decides whether they stack, split, or align.

**Proposed design cascade:**

```text
SITE default tokens and theme
  └── PAGE (including subpage; can inherit or override)
        └── SECTION (can inherit or override)
              └── GROUP (optional; repeatable nesting)
                    └── ELEMENT (category + semantic role + visual variant)
```

Global UI (navigation, footer) may use site tokens and its own component defaults. Reusable instances should retain a stable definition with separately specified allowed overrides.

### 7A.2 Resolution and override rules

For each independently configurable style property:

1. Start with a **system/default design token** (safe fallback).
2. Apply the **site theme** token, where provided.
3. Apply a **parent's inheritable value** where the property is defined as inheritable.
4. Apply an **explicit local override** on the page, section, group or element.
5. Apply valid **responsive rules** for the current viewport.
6. Enforce accessibility and technical constraints (for example minimum legible text size, no unsafe overflow).

**Explicit local override wins over inherited values**, unless a defined safety rule prohibits it. Not every property inherits automatically: text colour and typography may inherit, while margins, positioning, padding, fixed height, image crop and display modes require explicit behaviour. The exact per-property inheritance registry is a later design task.

Distinguish these states:
- `inherit` / **not set**: use the effective parent/default setting.
- `override`: use an intentionally chosen value.
- `none` / `transparent` / `0`: meaningful explicit values, *not* the same as "inherit".
- `reset`: revert to site/system default, if the editor offers that operation.

Design properties should live in validated configuration data and design tokens, not be concatenated into unrestricted user-entered CSS.

### 7A.3 Scope of configurable properties (requirements, not final defaults)

| Property family | Site | Page/subpage | Section | Group | Element |
| --- | --- | --- | --- | --- | --- |
| Typography and colour tokens | Theme defaults | Inherit/override | Inherit/override | Inherit/override | Inherit/override |
| Background colour, transparent, image/GIF, gradient | Default | Inherit/override | Inherit/override | Inherit/override | Where supported |
| Spacing outside/inside containers | Defaults | Override | Override | Override | Override where meaningful |
| Layout direction, width/column behaviour | Base rules | Page shell | Section children | Group children | Element sizing |
| Content-related design variant | Category definitions | N/A | Optional preset | Optional preset | Individual `variant` |

An image/GIF background needs defined cover/contain, anchor/crop, contrast overlay, loading and reduced-motion or static fallback. Page-level backgrounds may extend over the whole page; section backgrounds sit in their own section bounds. Transparency reveals whichever background is *actually painted behind the element*; it does not automatically copy the parent's fill.

**Default gap between adjacent sections: `0`.** This is separate from internal section padding, gaps between children, and the parent page's content gutter. Background and spacing overrides must not accidentally produce unreadable text.

### 7A.4 Content-aware and responsive behaviour

- Prefer natural line-wrapping and height growth for paragraphs, headings, cards and sections; **do not globally auto-shrink font sizes to fit arbitrary word counts**.
- Use responsive font ranges, maximum readable line lengths and fluid containers.
- A multi-column layout can stack on narrow screens. The ordered content tree supplies a sensible reading/keyboard order.
- Long words, translated content, missing images and empty optional content must not break the layout.
- Avoid hard fixed-heights where content can change; if clipping/truncation is deliberate, it must be an explicit component setting with accessible access to the full content.
- User-controlled positioning (left/centre/right/width) should map to validated layout modes rather than unconstrained absolute positioning.
- Use logical spacing and size values where possible to support localisation.

### 7A.5 Semantics are not visual variants

- Heading **H1–H6** represents semantic outline. A heading's font appearance is independently selected via a style token or `variant`.
- Image `V1`, `V2`, etc. represents **visual treatment**, not semantic depth or structural numbering.
- Structural item codes such as `P01.S01.G02.E-H01` do **not** encode H1 or V1.
- Browser navigation, metadata and SEO depend on valid page routes, headings and link semantics, not on visual font size alone.

### 7A.6 Example override sequence (illustrative, not deployed)

```text
Site: background = white; text colour = dark; section gap = 0
Page: background = subtle image; other values = inherit
Section A: background = transparent; padding = default
  Group 1: background = solid pale colour
    Element: heading H2; visual variant V1
Section B: background = white; section gap override = 16px
```

Here Section A shows the page image through transparent areas, while Group 1 paints its own surface. The second section's gap is a controlled override, not the site default.

### 7A.7 Open design choices deliberately deferred

Exact spacing scales, type sizes, breakpoints, columns, accessible palettes, background media storage, CSS token names, allowed per-node overrides, element variant catalogues and page-template arrangements are **not yet approved**. The next stage is to specify these one family at a time without replacing the fundamental tree.

### 7A.8 Design property inheritance matrix — proposed framework standard

This clarifies the earlier high-level cascade. It is a **framework proposal**, not executable CSS or implemented editor behaviour.

| Property | Resolution policy | Why |
| --- | --- | --- |
| Font family | Inherit by default; local override allowed | Visual consistency |
| Text colour | Inherit by default; contrast validation required | Readability |
| Base text size | Site token as baseline; semantic/component presets for individual roles; controlled override | Prevent accidental font-size compounding |
| Heading level | Independent semantic field, **not** inherited | Correct document hierarchy |
| Text weight/line height | Component/type defaults; optionally inherit appropriate values | Legible text |
| Background | Each surface has its own default; can be explicitly transparent or custom; visual underlying surface shows through transparency | CSS backgrounds are not normally inherited |
| Gap between sections | **0 by default**; owning page/section-stack controls gap; per-section override where defined | Deterministic adjacent spacing |
| Padding inside section/group | Local component default; custom override possible, not inherited as the same number | Avoid multiplied spacing |
| Margins | Controlled by owning layout; not automatically inherited | Avoid collapsed/doubled gaps |
| Width, columns, placement, alignment | Controlled by immediate parent layout plus local size constraints | Layout is contextual, not a universal inherited value |
| Height | Content-based by default; min/max only where appropriate | Avoid clipping variable text |
| Images / background media | Local media settings and fallback; no implicit inheritance of crop | Preserve predictable framing |
| Responsive rules | Site-defined breakpoint policy; scoped overrides with validation | Consistent cross-device behaviour |
| Animations | Respect reduced-motion preference and accessibility fallback | Safety and usability |

**Resolution order for configurable properties:** system fallback → theme token → any applicable inherited value → component preset → local override → responsive rule → accessibility/validation constraints. The exact priority between preset and inherited values must be encoded explicitly per property during implementation; no accidental reliance on CSS source order.

**Setting states:** `default` uses the component/site specified fallback; `inherit` follows an inheritable parent value; `custom` supplies a validated value; `reset` removes the local setting and restores its documented default mode. Explicit `0`, `none`, and `transparent` are legitimate values, never interpreted as unset.

**Important example:** A page with an image background can contain a section with a transparent background so the page image remains visible, while a nested group may paint an opaque colour. Its children do not silently inherit the group’s padding or background-image coordinates.

**Separate structure from visual stacking:** Hierarchical depth and codes (e.g. `P01.S01.G02.E-T01`) identify ownership, not CSS layering or left/right position. If overlapping surfaces are supported later, their z-order is a separate explicit layout rule; the DOM and keyboard reading order must remain logical.

**Decisions postponed to building-block design:** precise token values; which individual variants permit which overrides; breakpoint values; allowed minimum/maximum dimensions; controls for decorative media and accessibility fallbacks.

### 7A.9 Default-first styling and explicit exceptions — selected design principle

**Principle:** Every new page, section, group and element should be usable with **no custom design configuration**. Website-wide defaults supply consistent typography, heading appearance, paragraph style, text colour, background behaviour, spacing, gaps and layout. Custom settings are opt-in, never required for ordinary content.

**Proposed resolution:**

1. **Site defaults:** Named tokens and standard rules for headings (by semantic role), paragraphs, font family, font sizes, line height, font weight, colours, section gap (default **0**), internal spacing, responsive behaviour and layout defaults.
2. **Component defaults:** An H2 or paragraph uses its role-specific default style rather than treating every property as an unqualified inherited value. Normal paragraph and heading styles remain consistent throughout the site.
3. **Section override (optional):** A section can change an eligible property for itself and its descendants, e.g. a local text-colour theme or spacing scale. This must not silently override semantic heading levels or force every child to the same font size.
4. **Nested group override (optional):** A group can override applicable inherited settings only within its own subtree.
5. **Element override (optional):** A particular paragraph/heading/image can depart from defaults where its element type permits it. Otherwise it uses the normal effective style.
6. **Responsive and accessibility safeguards:** Readability, keyboard order, contrast and responsive rules still apply. Exact CSS precedence will be specified alongside component tokens.

**Three meaningful editor choices per setting:**
- `default`: use the standard type/component value (such as the site's predefined H2 or paragraph text size).
- `inherit`: follow the effective inheritable value from the parent, if that property supports inheritance.
- `custom`: explicitly specify a validated override.
- **Reset action:** clear a custom setting and return to its default behaviour.

These should not be confused: site-standard heading typography is often **default**, while text colour may **inherit** from a section. The editor should initially show `default` without requiring the user to choose every property manually.

**Illustration:**

```text
SITE: standard H1/H2/H3, paragraph style, colour, spacing; section gap = 0
  PAGE A: no custom values -> site defaults
    SECTION 1: no custom values -> standard heading and paragraph appearance
      H2: default heading style
      Paragraph: default paragraph style
    SECTION 2: custom text colour and custom internal gap
      H2: default H2 size/weight, section text colour where permitted
      Paragraph A: standard paragraph size, section text colour
      Paragraph B: custom font size; other style properties default/inherit
    SECTION 3: no custom values -> standard appearance resumes
```

**Scope rule:** An override affects only the relevant node/subtree and properties designated as inheritable; neighbouring or later sections are not affected. A section-wide spacing override does not automatically replace every element's local margin and padding. To prevent cumulative spacing and CSS conflicts, the owning parent controls gaps between its direct children, while the child controls its own internal padding.

**Performance/maintenance intent:** Fewer stored overrides, consistent appearance, easier global updates, and less manual configuration. Updating a global token will update all components still using the relevant default; specifically customised elements keep their overrides. Optional saved themes/variants remain later design tasks.

## 7B. Layout and visual layer rules — framework specification

**Status:** Proposed framework defaults for review. This is a rule set, not implementation. Layout types and element design variants are distinct. No final CSS values or individual building-block designs have been approved.

### 7B.1 Separate three independent axes

1. **Document structure** (`site → page → section → group → element`): parentage, content ownership, stable UUIDs, and structural path codes.
2. **Layout mode** (`L1`–`L4` initially): how one container arranges its **immediate children**.
3. **Visual stacking** (background/content/overlay/interface): which painted objects cover others when overlapping is intentionally enabled.

**Never infer left/right position, column count, or z-index from** `P01.S01.G02.E-T01`. This code specifies hierarchy and element kind, not visual placement. H1/H2 heading semantics and V1/V2 visual variants are independent of L1/L2 layout modes.

### 7B.2 Proposed layout mode registry

| Label | Internal mode | Purpose | Defaults and cautions |
| --- | --- | --- | --- |
| `L1` | `stack` | Vertical stacking (top to bottom) | Default page/section mode; natural item height; logical reading order |
| `L2` | `row` | Horizontal items in one row or wrapping row | Permit responsive wrapping/stacking; no forced horizontal overflow |
| `L3` | `grid` | Repeating items in rows and columns | Configurable minimum item width; automatic column reduction on small screens |
| `L4` | `overlay` | Intentional background/foreground composition | Opt-in; must enforce legibility, accessible reading/interaction order, and safe fallback |

**Registry extensibility:** Future `L5`, `L6` modes may be added without changing the tree or UUIDs. A flexible two-column split could be implemented as L2 or L3 configuration rather than a separate hard-coded section template. The mode is a property of a **container**; an element's separate `V1` variant controls that element's appearance.

### 7B.3 Container scope and nested layouts

- **Site:** supplies global width constraints, responsive standards, and design tokens; not a single layout mode for every descendant.
- **Page:** owns page shell and the default arrangement of its sections. Page sections default to vertical layout with **zero outer section-to-section gap**.
- **Section:** chooses how its direct group and element children appear. Default is `stack`; can override to row/grid/overlay.
- **Group:** independently chooses a layout mode for its own children; nested groups may choose another mode.
- **Element:** normally participates in its parent container's layout. It may specify width/span, self-alignment or sizing constraints where allowed, not take control of siblings' ordering.
- **Global UI:** navigation and footer have dedicated semantic and stacking rules, separate from normal page content.
- A page/subpage uses the same structural rules; a subpage may inherit theme settings but not blindly copy every ancestor page's layout.

**Mixed child example:**

```text
P01.S01 (L1: stack)
  E-H01            heading
  G02 (L3: grid)   grid container
    E-I01          image, first grid cell
    G02 (L1: stack)
      E-H01        heading in second grid cell
      E-T02        paragraph below heading
  E-T03            text below the grid, direct child of section
```

The full structural path identifies the correct parent and sibling level. The layout mode decides visual position; grouping and order remain stable when styles change.

### 7B.4 Position, order and alignment

- **Reading order** is the structural sibling order, shared across groups and elements; use it to generate accessible HTML and default keyboard/tab sequence.
- **Visual order** should normally match reading order; avoid desktop/mobile CSS reordering that makes reading or keyboard order confusing.
- Child settings may include `align_self`, `justify_self`, `span`, `width`, `min_width`, `max_width`, and validated position presets; availability depends on parent layout mode.
- Container settings may include direction, wrapping, column count or minimum column width, alignment, justification, and gaps.
- Parent sets the layout; children control their own sizing and placement only within allowed constraints. This prevents two conflicting sources of truth.
- Use normal document flow by default. Avoid unrestricted `position:absolute` for content-heavy or editable sections.
- Do not confuse **same structural depth** with **side-by-side placement**: siblings can be stacked or adjacent, based on the parent layout.

### 7B.5 Size and content-adaptive layout

- Default width: constrained by parent and responsive page gutters; default height: `auto`.
- Content length should cause natural wrapping/height growth, not arbitrary font shrinking.
- Use `min-width: 0`-equivalent constraints for shrinking flex/grid children and handle long unbroken words safely.
- Grid count may shrink at narrower widths; rows may wrap or stack with logical source order preserved.
- Empty or hidden elements should not leave accidental blank cards/gaps. The editor's visibility controls should be distinct from preserving draft data.
- Fixed dimensions, clipping, absolute overlays and scroll regions are explicit advanced choices with mobile/accessibility validation.
- Plan for long multilingual text, zoomed text, slow-loading images, missing content, image aspect ratios, and browser font changes.

### 7B.6 Spacing and section adjacency

- **Between page sections:** default gap is `0`, per user's selected convention.
- **Inside a section/group:** padding and child gaps are independently configured; a zero section gap does **not** imply zero internal breathing room.
- Prefer one responsible owner for each gap (the parent layout) to avoid double margins.
- Permitting a per-section spacing override requires a clear rule: a section's `space_before` or `space_after` contributes to the relevant boundary; avoid independent adjacent margins adding unpredictably. Exact conflict resolution must be specified in the spacing stage.
- Page margins/gutters, container padding, element margins and adjacent-section gaps are different settings.

### 7B.7 Visual layering and z-order

The layers below are **conceptual purposes**, not literal universal CSS z-index numbers. Browser stacking contexts may prevent a child's z-index from escaping its parent's context.

| Priority concept | Purpose | Rule |
| --- | --- | --- |
| Background | Site/page/section/group colour, image, GIF or gradient | Painted behind relevant content; transparency reveals underlying surfaces |
| Content | Ordinary text, images, cards and links | Default foreground; no overlapping unless enabled |
| Overlay | Approved decorative or compositional overlap | Opt-in, constrained; noninteractive decoration must not intercept clicks |
| Interface | Menus, dialogs and other utility controls | Above ordinary content when open, with keyboard and focus management |

A visual overlay never changes its UUID, structural ancestry or reading order. Decorative overlays should not obscure controls or essential text. Where media animations are used, support reduced motion and a static fallback. Modal/dialog stacking and focus-trapping are separate UI rules to define later.

**Terminology caution:** A CSS `z-index` property is not a substitute for a sound stacking-context model. Design-layer inheritance (section 7A) and visual paint order are different concepts.

### 7B.8 Proposed configuration example (not yet executable)

```json
{
  "kind": "section",
  "layout": {
    "mode": "stack",
    "gap": 0,
    "align": "stretch"
  },
  "children": [
    {"kind": "element", "element_type": "H", "variant": "V1"},
    {"kind": "group", "layout": {"mode": "grid", "min_column_width": "18rem"}},
    {"kind": "element", "element_type": "T", "variant": "V1"}
  ]
}
```

This demonstrates configuration scope only. The final storage schema must use UUID relationships rather than treating embedded arrays as permanent database identity.

### 7B.9 Validation and deferred decisions

**Validation requirements:** every node kind must be permitted under its parent; no parent loops; no invalid stacking over interface controls; responsive layouts must fit supported viewports; semantic heading order and keyboard interaction must remain logical; media needs fallbacks; editor must reject incompatible layout-property combinations.

**Deferred:** exact breakpoint sizes, supported width fractions, precise row/grid column constraints, element-specific layout compatibility, numeric z-index policy, advanced overlay placements, spacing conflict resolution and final editor controls. These belong in the sizing/spacing/background/element stages, not in the structural framework.

## 8. Drafts, revisions, history, archive and recovery

Each item receives one permanent UUID and can have multiple revisions. Recommended conceptual storage:

- `items`: current canonical item, hierarchy and metadata.
- `item_revisions`: immutable historical entries with distinct UUID `revision_id`, item UUID, revision integer, date/time, actor, action, and snapshot of content plus relevant structure/settings.
- `publication_revisions`: coherent published snapshots/manifests for static builds.
- `archived_at` or equivalent lifecycle marker on items; **archiving is not hard deletion**.
- Independent, tested backups of schema and editable content, protected outside public GitHub.

Track at least create, edit, move, reorder, type change, hide/unhide, publish, archive and restore. Example: an item may go from revision 1 to revision 3 while always retaining the same UUID. Its **display path can change with each revision**.

Revision history records what changed **to the content and structure**. This is not the website update/activity log (which this document explicitly excludes).

Restoring a historical revision should ordinarily create a **new** revision documenting the restore, not erase newer history. The exact retention policy, rollback and backup schedule will be set during implementation.

## 9. Security, cost and operational boundaries

### 9.1 Security

- GitHub Pages serves static files; it cannot itself securely execute administrator-only server-side operations.
- Build-time read credentials belong in protected CI secrets; never in browser JavaScript, public commits or generated pages.
- Enforce least-privilege database access and Supabase Row Level Security where appropriate.
- Separate admin authorisation from read-only build access. Prefer a scoped build role/view over a broadly privileged service-role key.
- Protect administrator login with MFA where supported, and do not permit unsolicited editor registration.
- Sanitize or strictly control rich content and embedded HTML to avoid unsafe generated output.
- Use secure server-side service/endpoint for a future dashboard Publish action; do not embed GitHub tokens in the browser.
- GitHub Free Pages source repository is public in this project; therefore source files and history must not contain confidential details.
- Public website content is public, regardless of database access controls.

### 9.2 Free-tier and resilience

The architecture is intended to run within free-tier limits for a small portfolio, **not promised permanently free**. GitHub Actions quotas, GitHub Pages conditions, hosting-provider changes and Supabase quota/pausing policies must be checked at the time of provisioning. A static deployed site can continue serving published pages even if a future database project is paused; authoring/publishing may be unavailable until service resumes. Independent exports remain important because managed backup capabilities differ by plan.

## 10. Data and publishing flow

1. Author edits an item in private interface, by UUID.
2. Store as a draft/revision without changing the live site.
3. Preview a coherent draft snapshot using existing design components.
4. Author approves/publishes a complete revision.
5. Build process retrieves **published** data using restricted credentials.
6. Validate schema, page hierarchy, required fields, routes, content safety, assets and links.
7. Astro generates static pages from page → section → group/element structure.
8. GitHub Actions deploys to GitHub Pages.
9. Keep previous successful publish available for rollback on failure.

The final admin UI, preview mechanics, authentication provider details and build trigger are design topics for subsequent stages.

## 11. Data model — conceptual, not yet SQL

Core candidates: `sites`, `pages`, `nodes` (sections/groups/elements), `content_records`, `content_references`, `media`, `item_revisions`, `publication_revisions`, and `shared_components`.

A single `nodes` table with `kind`, `parent_id`, `sort_order`, and optional `element_type` may simplify mixed group/element ordering; pages can remain separate for route constraints. Alternatively, dedicated tables may be preferable depending on validation and CMS ergonomics. **This is an open technical decision**, not a selected schema.

Do not confuse earlier interim JSON `areas.json` and fixed numbered `data-entry-code` slots with the future dynamic, unlimited-entry model. The final renderer must support adding/removing content without requiring an HTML card slot to already exist.

## 12. Decision register (not a change log)

| Topic | Decision / current position |
| --- | --- |
| Source repository | Portfolio-Website-2 is the immutable reference for this migration |
| Development repository | Portfolio-Website-3 |
| Public site | GitHub Pages, static output |
| Content source | Supabase proposed; not provisioned |
| Generator | Astro proposed; not installed |
| Structural philosophy | Flexible empty sections containing arbitrary ordered elements/groups |
| Hierarchy | Site → Page/subpage → Section → optional nested Group → Element |
| Mixed children | Allowed; single sibling sequence |
| IDs | UUIDv4 permanent; full hierarchical code derived |
| Code style | `P01.S01.G02.E-T01` with one-letter element category |
| Child numbering | Restart for each parent; skip unused levels |
| Semantic headings | H1–H6 independent of element code and style variant |
| Design variants | Independent from identity and hierarchy; exact catalog pending |
| Section-to-section gap | Default 0; override planned |
| Design cascade | Site → Page → Section → Group → Element, with explicit override and safe fallback; detailed tokens pending |
| Archiving/history | Persistent UUID, recorded revisions, soft archive proposed |
| Building-block visual system | To be designed step-by-step in a future stage |

## 13. Risks and matters to decide later

**Architecture review (17 October 2026):** The following are design gaps or implementation constraints, not evidence of broken deployed functionality.


1. Exact one-letter element registry, including whether cards are elements or styled groups.
2. Acceptable nesting depth and UI controls to keep editing manageable.
3. Database schema trade-off: unified node table versus specialised tables.
4. Reusable content references versus cloned page-specific components.
5. Route redirects when changing a page slug and canonical SEO addresses.
6. Accessibility rules for editable heading hierarchy and visual ordering.
7. Full background/media storage strategy, reduced motion, optimisation and licensing.
8. Rendering and visual regression tests against the original site.
9. Hosting of the authenticated CMS, secure triggering and preview environment.
10. Revision retention, restore UX, backup frequency and export portability.
11. Publication snapshot consistency and rollback mechanism.
12. How admin-level design overrides are constrained to protect consistency and responsive behaviour.
13. **Schema mismatch:** Existing `content/areas.json`, `content/schema.json` and `data-entry-code` annotations describe fixed slots; they must not be treated as the final flexible node/element model.
14. **Numbering implementation:** Define whether sibling numbering is computed on all nodes or only visible nodes, and how draft/archived nodes affect codes; document and test this before building the editor.
15. **Page and subpage numbering:** Confirm how child pages participate in the visible code sequence and handle changed parent pages without breaking URLs.
16. **Design rule precedence:** Finalise which properties inherit, reset, or default at each level, and specify narrow/mobile screen overrides.
17. **Content safety:** Validate embeds, external assets and rich text; define image rights, alt-text rules, metadata, and accessibility checks.
18. **Operational ownership:** Define who can publish, publish approvals, secret rotation, monitoring, domain/URL strategy and backup restoration tests.
19. **Performance:** Set targets for build time, large content volumes, media optimisation and page size so unlimited logical entries do not imply unlimited rendering performance.
20. **Reusable instances:** Define whether an instance can override source content, and whether changing a shared source updates all appearances automatically.

## 14. Planned implementation stages

1. **Framework contract:** finalise the hierarchy, UUIDv4 IDs, derived structural codes, constraints and semantics.
2. **Building-block design:** select element categories, variants, style tokens and container layout settings *step by step*.
3. **Renderer proof of concept:** demonstrate dynamic addition, removal, nesting, mixed ordering, long text, and adaptive layout in Astro using non-sensitive fixture data.
4. **Visual parity:** map original content and layout from repo 2; compare desktop/mobile against reference without unintentionally redesigning the website.
5. **Supabase implementation:** create validated schema, policies, revision/archive storage and restricted publishing view.
6. **Build/publish pipeline:** GitHub Actions, project base-path handling, previews, validation and rollback.
7. **Private administration:** secure editor, flexible hierarchy controls, preview, publication and restore.
8. **Review and launch:** check privacy, accessibility, cost, backup restore, performance, and maintenance.

## 15. Document governance and recommended storage

**Canonical architecture record:** This file (`docs/WEBSITE-ARCHITECTURE.md`) in the **development repository**. Why: it stays next to the actual implementation, supports Markdown review, Git history, pull requests and changes to the architecture without scattering design decisions across chat messages.

**Status of public storage:** This file describes only public-safe architecture and conventions. Do not put credentials, unreleased private content, administrator account details or confidential backups in this public repository.

**Activity/update record (separate):** If maintained, use a separate `CHANGELOG.md`, issue tracker, release notes or deployment log. Do not mix operational change entries into this architecture specification.

**Optional portable copy:** Export a DOCX/PDF for reading or offline reference, but treat the GitHub Markdown as the authoritative maintained version. Do not maintain divergent versions manually.

**Maintenance rule:** Update this document only when an architectural decision or design contract changes; label open proposals, update the document version deliberately, and review alignment between document and code. This specification is *not* a website update log. `CR` was the ChatGPT project label; **Portfolio Website** is the neutral official document title. Git already records when the document itself was modified.

---
*End of infrastructure and format specification. Website update activity history intentionally excluded.*
