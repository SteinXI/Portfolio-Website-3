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
