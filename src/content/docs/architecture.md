---
title: "Architecture Overview"
description: "A simple high-level explanation for technical readers."
pubDate: 'Dec 06 2025'
heroImage: '../../../public/hero.png'
---

Astro-TURF isn’t a plugin, a framework, or a theme.
It’s a **small architectural pattern** on top of Astro.

The goal: **clean separation of concerns**.

## Core idea

Every Astro site breaks down into three layers:

1. **Content** – markdown, collections, data
2. **Engine** – routing, build pipeline, theming logic
3. **Theme** – layouts, components, chrome, styling

Keeping these separate gives you durability and flexibility.

## The Theme Contract

The engine defines a minimal contract:

- what layouts a theme may expose,
- what providers it must supply,
- what capabilities it may implement,
- and how chrome (site identity) is structured.

If a theme satisfies the contract, the site can use it.
If not, Astro-TURF gracefully disables missing features.

## Why this architecture?

Because template-based Astro projects don’t age well.
Once you diverge from a starter, upgrading becomes impossible.

Astro-TURF is designed to be **long-lived**:

- themes can change without breaking users,
- users can swap themes without re-architecting,
- multiple content repos can publish into one unified site.

This is just the beginning — deeper docs will come as the ecosystem grows.
