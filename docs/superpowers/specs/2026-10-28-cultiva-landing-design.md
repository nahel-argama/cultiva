# Cultiva landing page — design

## Intent

Redesign the Cultiva marketplace landing page in the supplied Figma file. Help small producers and retailers understand the marketplace and choose their registration path. Keep the landing page copy unchanged while replacing its visual presentation; remove the `Sou Entregador` hero CTA as directed.

## Goals

- Create a polished, minimal marketplace landing with an editorial fresh-market character.
- Preserve the existing Cultiva name, logo, Portuguese copy, section order, and producer/retailer calls to action. Omit `Sou Entregador` from the hero per user direction.
- Create one primary landing layout and document how it adapts responsively; do not create a separate mobile frame.
- Organize the Figma file into five pages named `Contexto`, `Identidade Visual`, `Cores, Tipografias e etc`, `Telas`, and `Inspirações`; leave `Inspirações` empty.

## Non-goals

- Redesign other application screens or change frontend code in this task.
- Rewrite claims, labels, navigation, registration routes, or page copy.
- Add unrequested marketplace functions, new user flows, or new brand assets.

## Constraints and current state

- Source copy lives in `frontend/src/components/landingPage/` and is composed by `frontend/src/views/LandingPage.vue`.
- Current landing includes header, hero, market challenge, how it works, platform intelligence, audience benefits, and footer.
- Keep the existing Cultiva logo and `#2CA961`. The approved palette now uses brighter leaf green `#287A4E`, lime soft green `#D6E8A7`, and warm cream instead of the previous dark forest green.
- The supplied Figma file started blank (`Page 1`); there were no existing local components or styles to inherit.
- Deliver editable Figma text and vector/layout layers, not a flattened page screenshot.

## Approaches

1. **Editorial contemporary produce market — selected.** Warm cream surfaces, stronger Cultiva and leaf greens, lime soft-green accents, fresh-produce photography, and an expressive editorial display face paired with a readable sans serif. Gives the marketplace a human food-market character while preserving the existing brand.
2. **Digital market catalog.** Lead with product tiles, price details, and offer cards. Communicates shopping quickly, but makes the institutional landing resemble an in-product catalog.

The user selected the editorial direction. Exact font families and any supporting accent colors are implementation choices, with accessibility and Figma availability as constraints.

## Design outline

- **Contexto page:** concise product and audience summary grounded in the current landing copy.
- **Identidade Visual page:** Cultiva name/logo and brand-use notes.
- **Cores, Tipografias e etc page:** existing green anchors, supporting warm-neutral palette, type samples, and basic spacing/radius guidance.
- **Telas page:** one long landing page, containing all current sections in their existing order and preserving every copy string. Keep producer and retailer CTAs; omit the `Sou Entregador` hero CTA. Add responsive behavior notes to the frame; no separate mobile frame.
- **Inspirações page:** empty for now.

The layout may regroup content visually inside each current section, but must not change copy or the destinations of the retained CTAs. Document narrow-screen stacking, compact navigation, CTA flow, and reduced gutters as responsive behavior without adding a second frame. Reuse the existing Cultiva logo and use one Codex-generated produce-market photo for the hero, labeled as generated in the Figma notes. Add no invented marketplace metrics or claims.

## Left to the builder

Choose exact typefaces, supporting neutral/accent values, image crops, spacing, and component styling. Check screenshot and layer hierarchy before delivery.
