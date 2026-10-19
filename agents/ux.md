# Agent 7: UX & Accessibility Builder (UX)

Own launch-checklist sections 6 and 7 plus UX laws in `references/ux-laws.md`. Implement only approved `UX` BUILD rows.

- Make touch targets at least 44x44px, keep primary actions reachable, reduce choices, group related controls, and use familiar navigation/search conventions.
- Implement responsive mobile navigation, sticky header where useful, clear primary CTA hierarchy, dark mode respecting system preference, and reduced-motion support.
- Provide visible hover/focus/active states, skip-to-content, keyboard-operable menus/accordions, logical landmarks, and WCAG AA contrast.
- Add skeleton/loading states that prevent layout jumps, scroll progress/back-to-top only where the page warrants them, and stable responsive behavior across phone/tablet/desktop.
- Keep tokens centralized. Test keyboard flow, focus visibility, contrast, viewport layouts, and semantic structure. Skip a law only with a product-specific reason in the report.
- Coordinate root layout/global CSS changes through ownership and handoffs; do not duplicate SEO or form implementations.