# Content Area Codes and Naming — Portfolio Website 3

This repository is a **content-free layout template** derived from Portfolio-Website-2. All code attributes are invisible in the visual layout.

## Area codes
| Code | Existing HTML target | Intended content |
| --- | --- | --- |
| HOME | index.html #home | Intro headline, summary and interests |
| ABOUT | index.html #about | Biography, figures and introductory material |
| FOCUS | index.html #research | Research and professional areas of focus |
| EXPERIENCE | index.html #experience | Featured professional positions |
| EDUCATION | index.html #education | Education history |
| LANGUAGES | index.html #languages | Language proficiency |
| CREDENTIALS | index.html #credentials | Credentials, activities and recognition |
| HOBBIES | index.html #hobbies | Personal interests |
| CONTACT | index.html #contact | Contact links |
| EXPERIENCE_DETAIL | experience.html main | Full professional experience view |

## Naming conventions
- **Area:** uppercase stable code, e.g. `FOCUS`; added to existing sections as `data-area-code="FOCUS"`.
- **Layout slot:** `AREA_01`, `AREA_02`, etc. Currently attached to existing `article` elements with `data-entry-code`. These are **presentation slots**, not permanent database record IDs. On the detailed page use `EXPERIENCE_DETAIL_01`.
- **Record ID (future):** a permanent database ID independent of display order and page; the same experience record can appear on both pages.
- **Fields:** lowercase snake_case, e.g. `title`, `description`, `start_date`, `sort_order`, `visible`.
- **Conceptual field reference:** `FOCUS_01.title`. The eventual database will separate record ID and field name.

## Layout slot counts
- FOCUS: 6
- EXPERIENCE: 3 featured positions
- EDUCATION: 5
- LANGUAGES: 4
- CREDENTIALS: 14
- HOBBIES: 4
- EXPERIENCE_DETAIL: 9 full experience slots

Counts describe the copied **template**, not fixed database capacity; the database may contain any number of entries.

## Files
- `content/areas.json`: section registry, page mapping, area metadata.
- `content/schema.json`: provisional content data shape (not an executable SQL schema).
- `content/collections/*.json`: intentionally empty collections.
- HTML `data-area-code` and `data-entry-code` attributes: labels for eventual template rendering; they **do not connect to JSON or Supabase yet**.

## Next technical steps
1. Identify precise field slots including all text, lists and links. Keep a source-to-slot mapping; verify against Portfolio-Website-2.
2. Convert template HTML into reusable Astro components and render from data, preserving visual appearance.
3. Test desktop/mobile behavior, links and content parity.
4. Define Supabase tables and RLS, build read-only publishing, backups and secure editor.

## Preservation rules
- Do not change Portfolio-Website-2.
- Do not commit private personal data or database secrets to this public repository.
- Leave existing CSS and JavaScript unchanged until rendering parity can be checked.
- Placeholders are visual stand-ins; they are not actual records or published content.

## Migration interpretation and preservation rules (architecture alignment)

This document describes **legacy presentation slots** in the copied HTML template, **not a permanent, fixed-card authoring model**. The authoritative future design rules are documented in [WEBSITE-ARCHITECTURE.md](WEBSITE-ARCHITECTURE.md), especially §7G.

- Preserve existing intentionally chosen visual designs, layout behaviours and interaction patterns through **named migrated presets**; do not automatically force new generic typography, section padding, colours or grid defaults onto current sections.
- A legacy `data-area-code` label can occur on more than one HTML container (e.g. `EXPERIENCE_DETAIL` on both the main container and its nested list). It is not a unique database key; assign distinct UUIDs to real structural nodes during migration.
- Legacy `data-entry-code` values such as `FOCUS_01` identify existing **layout slots**, not permanent item IDs or final hierarchical codes. Do not impose their fixed counts on dynamic content.
- Maintain a verified mapping of each existing layout slot/section to the future UUID-backed model, and compare desktop/mobile visual parity and interactions before editing or removing legacy selectors.
- The existing `content/schema.json` is provisional and **will be superseded** by the flexible site/page/section/group/element schema; neither JSON file currently drives rendering.
