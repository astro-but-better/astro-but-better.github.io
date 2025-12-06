---
title: "Building Themes"
description: "A high-level overview for theme developers."
pubDate: 'Dec 06 2025'
heroImage: '../../../public/hero.png'
---

If you want to design reusable Astro layouts, Astro-TURF gives you a stable, predictable structure to build on.

## Why build a theme?

Because themes become **portable packages**:

- they don’t depend on the user’s project structure,
- they don’t need custom setup scripts,
- they work anywhere the Theme Contract is supported,
- and upgrading the theme doesn’t break the user’s content.

## Theme anatomy (high-level)

A theme provides:

- **Layouts** – page templates (`Page`, `BlogIndex`, `DocPage`, etc.)
- **Providers** – functions that supply navigation, authors, footer, etc.
- **Chrome** – the site identity structure shared across themes
- **Capabilities** (optional) – Search, Related Content, Taxonomy, Outline, etc.

That’s it. A theme is just an npm package exporting these.

## How DI works (in one sentence)

The site imports the theme, calls `createTheme()`, and receives a bundle of **layouts + providers** that plug directly into pages.

No custom wiring. No boilerplate.

## What’s next?

A deeper specification will come later, but this is enough to start building themes today.

Curious how the engine actually works?
👉 **[Architecture Overview](../architecture/)**
