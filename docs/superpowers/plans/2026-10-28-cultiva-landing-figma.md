# Cultiva Landing Figma Design Plan

> **For agentic workers:** Follow the approved design spec and validate each Figma stage before moving on.

**Goal:** Create an editable, modern Cultiva marketplace landing in the supplied Figma file, preserving its copy and documenting responsive behavior without a separate mobile frame.

**Architecture:** Organize the Figma file into five named Pages. Add concise product/brand/type/color reference areas and one long landing frame. Compose the screen from editable text, vectors, and frames, reusing the existing logo and using a Codex-generated produce-market photo.

**Tech Stack:** Figma Design, Plugin API via `use_figma`; Vue landing files as copy and asset source.

**Spec:** `docs/superpowers/specs/2026-10-28-cultiva-landing-design.md`

## Global Constraints

- Preserve Portuguese landing copy and section order; omit the `Sou Entregador` hero CTA per user direction.
- Keep Cultiva name, logo, and `#2CA961` recognizable; use `#287A4E`, `#D6E8A7`, and warm cream for stronger brand color.
- Use one primary landing frame; document narrow-screen adaptations without a separate mobile frame.
- Keep `Inspirações` empty.
- Make the design editable; do not flatten the landing into a screenshot.

## Review Focus

- Copy omissions or edits — compare all visible copy with `frontend/src/components/landingPage/`.
- CTA label/destination mismatch — verify producer/retailer routes against `Hero.vue` and `Header.vue`; `Sou Entregador` is intentionally omitted.
- Responsive behavior missing — check documented stacking, compact nav, CTA flow, and gutters.
- Clipped or overlapping layers — inspect the Figma screenshot and frame bounds.
- Blank inspiration area populated accidentally — verify the five-page hierarchy.

---

### Task 1: Discover Figma conventions and confirm source content

**Files:** Figma file `F1e6GD5nbO8rgzB1ietIYd`; source `frontend/src/components/landingPage/*.vue`.

- [ ] Inspect existing page and available Figma libraries/design-system assets; select a library only if a relevant published system exists.
- [ ] Extract all visible copy and CTA destinations from the current landing sections.
- [ ] Record the existing brand colors and logo reference; use the generated hero image saved at `frontend/public/cultiva-marketplace-hero.png`.

**Check:** Source copy and destinations are accounted for; Figma page conventions and reusable assets are known.

### Task 2: Create the five-page Figma organization and visual foundation

**Files:** Figma file `F1e6GD5nbO8rgzB1ietIYd`.

- [ ] Create named Pages: `Contexto`, `Identidade Visual`, `Cores, Tipografias e etc`, `Telas`, `Inspirações`.
- [ ] Add editable, concise product context and brand notes grounded in the source landing.
- [ ] Show palette and typography specimens; preserve Cultiva green and use leaf green, lime soft green, and warm cream.
- [ ] Leave `Inspirações` empty.

**Check:** Page names and order match the requested organization; reference content remains editable.

### Task 3: Build the landing page and responsive notes

**Files:** Figma file `F1e6GD5nbO8rgzB1ietIYd`; sources `Header.vue`, `Hero.vue`, `ProblemSection.vue`, `SolutionSection.vue`, `FeaturesSection.vue`, `TargetAudienceSection.vue`, `Footer.vue`.

- [ ] Create one long landing frame using the editorial contemporary produce-market direction.
- [ ] Preserve all copy and section order; keep the producer and retailer CTAs and omit `Sou Entregador` as requested.
- [ ] Reuse the Cultiva logo and use the Codex-generated produce-market hero image; add no invented metrics or claims.
- [ ] Label the hero image as Codex-generated in an annotation outside the screen frame.
- [ ] Add a visible responsive-behavior note: narrow-screen content stacking, compact navigation, CTA flow, and reduced gutters.

**Check:** Every retained copy block and CTA appears once; `Sou Entregador` is absent; all design layers are editable.

### Task 4: Validate and refine the Figma result

**Files:** Figma file `F1e6GD5nbO8rgzB1ietIYd`.

- [ ] Inspect section/frame metadata and take a screenshot of the completed landing.
- [ ] Check copy, responsive note, frame bounds, text clipping, overlaps, and section hierarchy.
- [ ] Fix visual or structural defects and recheck the affected area.

**Check:** Screenshot and hierarchy confirm the approved brief; `Inspirações` remains empty.
