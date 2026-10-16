# Portfolio Website 3 — Infrastructure Migration Plan

## Baseline
Source: `SteinXI/Portfolio-Website-2`, default branch `main`.
Development: `SteinXI/Portfolio-Website-3`, default branch `main`.
The six source files were copied without edits and verified by matching Git blob SHAs.
The source repository must remain unchanged.

## Objectives
- Keep existing content, design, navigation, spacing, and responsive behavior unchanged.
- Separate content from page markup and styling.
- Eventually manage content in Supabase PostgreSQL using a private editor.
- Statically generate the published site (proposed: Astro) via GitHub Actions and deploy to GitHub Pages.
- Keep public visits independent of live database availability.

## Source structure
- `index.html`: home and portfolio sections with embedded content
- `experience.html`: professional experience details
- `css/style.css`: original layout and styles
- `css/editorial-v2.css`: styling overrides
- `js/script.js`: client-side interactions
- `README.md`: source readme

## Planned milestones (not implemented)
1. **Baseline audit**: enumerate all website content fields, external assets, navigation, links, breakpoints and interactive behavior; record screenshots for desktop/mobile.
2. **Static content extraction**: move copy into structured content files while keeping a pixel-equivalent output; test content and link parity.
3. **Astro adoption**: convert page shells and components without visual redesign; configure GitHub Pages project base path and deployment; regression-test.
4. **Supabase setup**: design schema, published revision snapshots and RLS; implement restricted read-only build access and separate credentials, backup exports.
5. **Publishing workflow**: authenticated publish trigger, validation, static build, deployment, rollback and failure handling.
6. **Private editor**: secure hosting, individual administrator login and MFA, edit/preview/publish flow.

## Safety rules
- No database credentials, elevated access tokens, or unpublished information in public frontend code or Git history.
- Do not expose service-role keys in browser scripts.
- GitHub Pages does not provide a private server runtime; the admin/editor backend must use a separate secure service.
- Deploy only validated published content revisions, not drafts.
- Keep restoration/export paths independent of the free database service.

## Current status
- [x] Original source copied to destination and hash-verified
- [ ] Content extraction
- [ ] Astro migration
- [ ] Supabase provisioning
- [ ] Admin interface
- [ ] GitHub Pages deployment
