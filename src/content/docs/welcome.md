---
title: "What is Astro-TURF?"
description: "A quick introduction to the Astro theming architecture."
pubDate: 'Dec 07 2025'
heroImage: '../../../public/hero.png'
---

Astro-TURF is a minimal but powerful architecture for building **swappable, theme-driven Astro sites**.
Its goal is simple: **separate the content, the engine, and the theme**, so each one can evolve independently.

Instead of locking your site to one starter template forever, Astro-TURF gives you a stable structure that lets you:

- use themes without rewriting content,
- build themes without worrying about routing or content setup,
- mix multiple themes across different content collections,
- and publish federated content from multiple repositories.

At its core, Astro-TURF defines a **Theme Contract**: a small set of shapes (providers, capabilities, chrome) that every theme follows.
If a theme implements the contract, the engine can plug it in — no glue code required.

Want to use a theme? Start here:
👉 **[Using Themes](../users/)**

Want to build a theme? Jump to:
👉 **[Building Themes](../themes/)**

Curious about how the system works internally?
👉 **[Architecture Overview](../architecture/)**
