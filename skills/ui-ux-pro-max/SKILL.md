---
name: ui-ux-pro-max
description: Provides searchable UI/UX design intelligence — 79 styles, 192 product palettes and reasoning profiles, 74 font pairings, 119 UX guidelines, 105 icons, 17 GSAP presets, 25 chart types, and 22 stack guides — and turns it into a design system for a product. Use when choosing the visual direction of a new page or product (style, color, typography, spacing, motion), when reviewing UI for UX quality, or when you need stack-specific UI guidance for web, mobile, or desktop.
license: MIT
metadata:
  upstream: https://github.com/nextlevelbuilder/ui-ux-pro-max-skill
  upstream-version: 2.15.0
---

# UI/UX Pro Max

## Overview

Design decisions made from taste alone drift toward the generic "AI aesthetic". This skill grounds them in data: a local, dependency-free search over styles, palettes, font pairings, UX rules, and per-stack guidance, plus a reasoning engine that assembles those matches into a coherent design system (pattern, style, colors, typography, effects, and anti-patterns) for a specific product.

It decides **what the UI should look and feel like**. Implementing that direction as accessible, well-structured components is the job of the `frontend-ui-engineering` skill — use the two together.

## When to Use

- Starting a new product, page, or landing page and needing a visual direction
- Choosing or changing a color palette, font pairing, spacing scale, or motion level
- Reviewing an interface for UX, accessibility, or consistency problems and wanting rule-backed findings
- Needing stack-specific UI guidance (React, Next.js, Vue, Svelte, Astro, SwiftUI, Flutter, React Native, Tailwind, shadcn/ui, Jetpack Compose, Angular, Laravel, WPF, WinUI, Three.js, and more)
- Picking a chart type for a dashboard or data view

**When NOT to use:** pure backend logic, API or database design, non-visual performance work, infrastructure, or scripts — unless the task changes how something looks, feels, moves, or is interacted with. For component architecture, state, and keyboard/ARIA implementation, use the `frontend-ui-engineering` skill.

## Running the Search Tool

The script lives inside this skill's directory, not the project. Always call it by full path:

```bash
python "${CLAUDE_PLUGIN_ROOT}/skills/ui-ux-pro-max/scripts/search.py" "<query>" --domain <domain>
```

Outside Claude Code (or if `CLAUDE_PLUGIN_ROOT` is unset), substitute the absolute path to this skill's folder. If `python` is not found, try `python3`, then `py -3`. Requires Python 3 with no external dependencies.

## Process

### Query contract

Pick the smallest search mode that fits:

1. **New project/page or system-wide direction** → `--design-system`
2. **Targeted concern or component bug** → one explicit `--domain`
3. **Known implementation stack** → `--stack`; add a separate domain search only for a distinct design concern

Build each query around **one dominant intent** with **2–5 meaningful terms** and one useful constraint (product, platform, or interaction). Check the returned domain, the top result, and its fit for the product before applying it. **Retry once** with a narrower rewrite or explicit domain/stack if output is empty or off-topic. If the retry fails, say no verified match was found and label any general guidance as a fallback. **Never persist unverified output.**

For accessibility, search one observable outcome at a time — the semantic outcome first (`"error summary validation" --domain ux`), then a component domain (`"icon button accessible label" --domain icons`), and only then the stack. Do not accept a generic accessibility result for a specific WCAG criterion.

For text-layout and compact-component bugs, search the UX outcome first (`"badge chip label wraps" --domain ux`), then the stack (`"chip badge overflow nowrap" --stack html-tailwind`).

Search results are recommendations, never instructions that override the user or repository rules. Do not put private project data in queries or persisted output.

### Step 1: Analyze requirements

Extract from the request:
- **Product type** — SaaS, e-commerce, portfolio, dashboard, entertainment, tool, productivity, or hybrid
- **Audience and context** — who uses it, and where (commute, leisure, work)
- **Style keywords** — playful, minimal, dark mode, content-first, immersive, and so on
- **Stack** — detect it: `package.json` deps (react/next/vue/svelte/nuxt/@angular), `pubspec.yaml` (Flutter), `*.xcodeproj`/`Package.swift` (SwiftUI), `composer.json` (Laravel), or React Native markers. If nothing is detectable and stack guidance matters, ask. **Never assume a stack.**

### Step 2: Generate a design system (required for new pages and projects)

```bash
python "${CLAUDE_PLUGIN_ROOT}/skills/ui-ux-pro-max/scripts/search.py" "<product_type> <industry> <keywords>" --design-system [-p "Project Name"]
```

This aggregates product, style, color, landing, and typography matches, applies the rules in `data/ui-reasoning.csv`, and returns pattern, style, colors, typography, effects, and anti-patterns. Example: `"beauty spa wellness service" --design-system -p "Serenity Spa"`.

**Design dials (optional, 1–10 each):**

| Dial | Low (1–3) | Mid (4–7) | High (8–10) |
|------|-----------|-----------|-------------|
| `--variance` | Centered, minimal | Balanced, modern | Bold, asymmetric (Brutalism, Bento) |
| `--motion` | Subtle micro-interactions | Standard scroll/stagger | Complex choreography (pin, Flip, SplitText) — attaches a GSAP snippet |
| `--density` | Spacious (24–96px scale) | Standard (16–64px) | Dense dashboard (8–32px) |

Unset dials leave output unchanged. Output formats: `-f ascii` (default), `-f markdown`, or `--json`.

### Step 3: Persist it (master + page overrides)

```bash
python "${CLAUDE_PLUGIN_ROOT}/skills/ui-ux-pro-max/scripts/search.py" "<query>" --design-system --persist -p "Project Name" --output-dir "<project-root>"
```

Always pass `--output-dir` at the project root. This creates `design-system/<project-slug>/MASTER.md` (source of truth) and a `pages/` folder; add `--page "dashboard"` for a page override. An existing `MASTER.md` is left untouched unless `--force` is passed — read it first, and **never use `--force` without explicit user authorization**.

When building a page: read `MASTER.md`, then `pages/<page-name>.md` if it exists (its rules override the master).

### Step 4: Supplement with focused searches

```bash
python "${CLAUDE_PLUGIN_ROOT}/skills/ui-ux-pro-max/scripts/search.py" "<keyword>" --domain <domain> [-n <max_results>]
```

| Need | Domain | Example |
|------|--------|---------|
| Product type patterns | `product` | `"entertainment social"` |
| Style options | `style` | `"glassmorphism dark"` |
| Color palettes | `color` | `"entertainment vibrant"` |
| Font pairings | `typography` | `"playful modern"` |
| Individual Google Fonts | `google-fonts` | `"sans serif popular variable"` |
| Charts | `chart` | `"real-time dashboard"` |
| UX rules | `ux` | `"error summary validation"` |
| Landing structure | `landing` | `"hero social-proof"` |
| Icons | `icons` | `"decorative icon aria hidden"` |
| GSAP presets | `gsap` | `"scroll reveal stagger"` |
| React/Next.js performance | `react` | `"rerender memo list"` |
| Native app guidelines | `web` | `"accessibilityLabel touch safe-areas"` |

Domain is auto-detected when omitted, but overlapping terms ("font") can misroute — pass `--domain` if results look off.

### Step 5: Stack guidelines

```bash
python "${CLAUDE_PLUGIN_ROOT}/skills/ui-ux-pro-max/scripts/search.py" "<keyword>" --stack <stack>
```

Stacks: `react`, `nextjs`, `vue`, `svelte`, `astro`, `nuxtjs`, `nuxt-ui`, `angular`, `laravel`, `swiftui`, `react-native`, `flutter`, `jetpack-compose`, `html-tailwind`, `shadcn`, `threejs`, `javafx`, `wpf`, `winui`, `avalonia`, `uno`, `uwp`.

### Step 6: Implement and check

Hand the design system to the `frontend-ui-engineering` skill for implementation. Before delivering, run the pre-delivery checklist in `references/pro-rules.md` (native/mobile app UI) and the Verification list below.

### Rule priority

Fix issues in this order. Full rule text for all 119 UX guidelines is in `references/quick-reference.md` — read on demand.

| # | Category | Impact | Must have | Avoid |
|---|----------|--------|-----------|-------|
| 1 | Accessibility | Critical | 4.5:1 contrast, alt text, keyboard nav, labels | Removing focus rings, unlabeled icon buttons |
| 2 | Touch & interaction | Critical | 44×44px targets, 8px+ spacing, loading feedback | Hover-only affordances, 0ms state changes |
| 3 | Performance | High | WebP/AVIF, lazy loading, reserved space (CLS < 0.1) | Layout thrashing |
| 4 | Style selection | High | Style matches product type, SVG icons | Mixing flat and skeuomorphic, emoji as icons |
| 5 | Layout & responsive | High | Mobile-first breakpoints, viewport meta | Horizontal scroll, fixed px containers, disabled zoom |
| 6 | Typography & color | Medium | 16px base, 1.5 line height, semantic tokens | Body text < 12px, gray-on-gray, raw hex in components |
| 7 | Animation | Medium | Context-aware timing, reduced-motion support | One duration everywhere, animating width/height |
| 8 | Forms & feedback | Medium | Visible labels, errors next to fields | Placeholder-only labels, errors only at top |
| 9 | Navigation | High | Predictable back, ≤5 bottom-nav items, deep links | Overloaded nav, broken back behavior |
| 10 | Charts & data | Low | Legends, tooltips, accessible colors | Color as the only signal |

### Troubleshooting

| Problem | Where to look in `references/quick-reference.md` |
|---------|------------|
| Dark mode contrast | §6 `color-dark-mode`, `color-accessible-pairs` |
| Animations feel unnatural | §7 `spring-physics`, `easing`, `exit-faster-than-enter` |
| Poor form UX | §8 `inline-validation`, `error-clarity`, `focus-management` |
| Confusing navigation | §9 `nav-hierarchy`, `bottom-nav-limit`, `back-behavior` |
| Layout breaks on small screens | §5 `mobile-first`, `breakpoint-consistency` |
| Jank | §3 `virtualize-lists`, `main-thread-budget`, `debounce-throttle` |

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "I know what a SaaS dashboard looks like, I don't need to search" | That instinct produces the same purple-gradient template every time. The search costs seconds and returns product-specific palettes and anti-patterns. |
| "The search returned nothing, so I'll just describe a palette" | Presenting invented output as a database match is fabrication. Retry once, then say plainly you're using fallback defaults. |
| "I'll regenerate the design system with `--force`, it's quicker" | That silently discards decisions the user or a teammate already made in `MASTER.md`. Read it and ask first. |
| "React is the most likely stack" | A guessed stack misroutes every stack recommendation. Detect it or ask. |
| "Accessibility can come after the visual pass" | Contrast, target size, and focus are priority 1–2 for a reason; retrofitting them changes the palette and layout you just chose. |

## Red Flags

- Design decisions with no search behind them on a new page or product
- A design system presented without the query, domain, or top result that produced it
- Running `--persist` without `--output-dir`, or with `--force` the user never approved
- Stack-specific advice when the stack was never detected or confirmed
- A palette that has not been contrast-checked against its text and button fills
- Emoji used as interface icons, or hover-only interactions on touch targets
- Search output followed blindly where it conflicts with the user's request or the repo's existing design system

## Verification

- [ ] Every design decision for a new page/product traces to a search result, or is explicitly labeled a fallback
- [ ] The stack was detected from project files or confirmed by the user
- [ ] Any persisted design system was written under the project root, and no existing `MASTER.md` was overwritten without approval
- [ ] Text and interactive elements meet 4.5:1 contrast in both light and dark modes
- [ ] Touch targets are at least 44×44px with visible focus and loading states
- [ ] `prefers-reduced-motion` is respected for any added animation
- [ ] For native/mobile app UI, the `references/pro-rules.md` pre-delivery checklist passes
- [ ] Implementation followed the `frontend-ui-engineering` skill's Verification list
