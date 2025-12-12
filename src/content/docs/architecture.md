---
title: "Architecture Overview"
description: "A simple high-level explanation for technical readers."
pubDate: 'Dec 06 2025'
heroImage: '../../../public/hero2.png'
---

> **Status:** Living architecture notebook
>
> **Purpose:** Capture the conceptual, structural, and operational architecture
> of Astro-TURF, describing a model where **Astro-TURF is the engine** and
> **themes are integrations/plugins** with declarative self‑registration.

---

## 1. Executive Summary

Astro-TURF is a **contract-driven static content engine** built on top of Astro.

Its core responsibility is **orchestration**, not rendering:

- federating content from multiple repositories
- normalizing and enriching content data
- enforcing contracts between data, behavior, and layouts
- resolving chrome and capabilities
- delegating rendering to **theme integrations** via dependency injection

Astro-TURF is intentionally **domain-agnostic**.

Blogs, documentation, marketing pages, and future domains are implemented via
**themes**, which are treated as **Astro-style integrations/plugins**.

---

## 2. Core Model

> **Astro-TURF is the engine. Themes are integrations.**

Astro-TURF owns orchestration, federation, contracts, lifecycle, and data
resolution.

Themes extend Astro-TURF via declarative registration:

- themes declare which layouts they provide
- themes declare which capabilities they support
- themes provide rendering implementations only

Themes do not own data, defaults, or the build pipeline.

This model aligns Astro-TURF with established ecosystem semantics (Astro
integrations, Vite plugins).

---

## 3. Architectural Stack

```
Astro (build + runtime)
└── Astro-TURF (engine)
    ├── Contracts (shapes, provider interfaces)
    ├── Federation (repos → collections)
    ├── Core orchestration
    ├── Chrome & capability resolution
    │
    └── Theme integrations (plugins)
        ├── theme-blog (starter-blog-based)
        ├── theme-docs (Starlight-based)
        └── future themes
```

Themes are loaded by the engine and **register themselves**.

---

## 4. Core Principles

### 4.1 Contracts First

All boundaries are explicit:

- data shapes
- provider interfaces
- layout roles
- capability surfaces

Contracts define *what is allowed*, not how it is implemented.

---

### 4.2 Engine vs Theme Responsibilities

| Layer  | Responsibility                                   |
| ------ | ------------------------------------------------ |
| Engine | Federation, orchestration, lifecycle, resolution |
| Theme  | Rendering and domain-specific presentation       |

Themes never orchestrate the system; they extend it.

---

### 4.3 Federation Is First-Class

Astro-TURF assumes:

- content lives in multiple repos
- authorship is decentralized
- aggregation is intended to be deterministic
- build-time federation is the norm

---

## 5. Federation Workflow

### 5.1 Source Repositories

Each federated repository:

- owns its content
- owns its git history
- does **not** render a site

Workflow:

1. Author edits content
2. Commits + pushes
3. GitHub Action triggers engine build

---

### 5.2 Engine Repository

The engine repository:

- maintains a federation manifest
- checks out federated repos
- extracts git-based timestamps
- copies content into collections
- triggers Astro build

---

### 5.3 One Repo → One Collection

Design rule:

> **Each federated repository maps to exactly one content collection.**

This model applies to both single-package repositories and monorepos.

---

### 5.4 Monorepo Handling

Monorepos are treated as:

- one federated source
- with multiple declared content roots

Example:

```yaml
federation:
  - name: astro-turf
    repo: github.com/astro-but-better/astro-turf
    collection: docs/astro-turf
    roots:
      - path: packages/contracts/docs
        namespace: contracts
      - path: packages/core/docs
        namespace: core
```

Namespaces preserve semantic ownership without breaking the
one-repo-one-collection invariant.

---

## 6. Content Enrichment

During federation, Astro-TURF enriches content with derived metadata:

- created (first git commit)
- updated (latest git commit)
- source\_repo
- source\_path
- collection
- namespace
- canonical\_url (optional)

Timestamps and provenance are **derived by convention**, not manually authored.

---

## 7. Theme Integrations

### 7.1 Theme as a Black Box

A theme is an **Astro-TURF integration/plugin** that is intentionally treated as
a **black box renderer**.

A theme:

- declares **which layouts** it provides
- declares **which capabilities** it supports
- receives data exclusively via contracts
- does **not** declare what data it needs
- does **not** own data, paths, or resolution rules

Themes make no claims about data requirements beyond what is implied by their
layouts and capabilities.

Whether and how a theme *uses* available data is an implementation detail
entirely up to the theme author.

Themes assume only that the engine will inject a complete, contract-valid data
context.

---

### 7.2 Chrome Is User-Domain Data

All chrome data (identity, navigation, footer, metadata defaults) is
**user-domain data**.

The engine **always provides chrome data**.

This means:

- chrome belongs to the site author, not the theme
- chrome exists independently of any particular theme
- themes may consume all, some, or none of the provided chrome data
- swapping themes never requires rewriting chrome data

Themes are required only to *accept* provided chrome data. How much of it they
use is entirely up to the theme author.

---

### 7.3 Why Providers Are Derived

Providers are **not a primary abstraction** in Astro-TURF.

They are a **derived implementation detail** produced by the engine as part of
wiring, in order to satisfy the demands implied by:

- layout declarations
- capability declarations

#### Rationale

1. **Themes declare structure, not dependencies**\
   Themes never declare required data or providers. They declare only layouts
   and capabilities.

2. **Layouts and capabilities imply potential data usage**\
   The engine infers which data *may* be relevant based on what layouts and
   capabilities are active.

3. **Providers exist to move data, not define intent**\
   Providers are mechanisms used by the engine to supply data. Treating them as
   first-class declarations would invert control and couple themes to data
   topology.

By deriving providers:

- themes remain pure black boxes
- data ownership stays with the user
- wiring authority stays with the engine
- unused providers are never instantiated

Providers are therefore **engine-internal artifacts**, resolved as part of
orchestration.

---

### 7.4 Engine-Owned Wiring

Astro-TURF is responsible for wiring:

- resolves which providers are required
- determines data sources and paths
- builds and validates the provider graph
- injects resolved data into layouts

Themes do not participate in wiring by default.

This is intended to support:

- a single authority for resolution
- consistent behavior across themes
- predictable override semantics

---

### 7.5 Custom Theme Bootstrap (Advanced)

A theme **may** provide custom DI or a bootstrap function.

This is:

- optional
- explicitly opt-in
- intended for narrow, advanced use cases

Examples include:

- integrating non-file-based data sources
- bridging to legacy systems
- experimental capabilities

The existence of this escape hatch does not affect the zero-config happy path.

---

### 7.6 Zero-Config and Overrides

#### Zero-Config

In the common case, the end user provides chrome data once and installs themes:

```ts
export default defineSite({
  themes: [blog(), docs()]
});
```

The engine derives providers and wiring automatically by design.

#### Overrides

Users may override engine defaults when necessary:

- alternate chrome data for a specific domain
- custom data sources for a capability

Overrides are:

- explicit
- user-scoped
- opt-in

Chrome resolution precedence:

1. user overrides
2. engine defaults

Themes are **not** part of the precedence chain.

---

## 9. Documentation Theme (Starlight)

`theme-docs` is an Astro-TURF integration built on top of Starlight.

- Starlight is an internal implementation detail
- Sidebar, TOC, and splash are handled by Starlight
- Global chrome is handled by Astro-TURF

No provider injection into Starlight internals is required.

---

## 10. Documentation Generation Workflow

### 10.1 Package-Local Docs

Each package may generate:

- API docs
- narrative docs

Output:

```
packages/<pkg>/docs/generated/
```

---

### 10.2 Federation-Aware Docs Build

Root workflow:

```
pnpm build:docs
```

Docs are **not intended to be aggregated into a detached tree**.

Instead:

1. docs are generated in-place per package
2. the federation module reads directly from declared roots
3. git history is preserved and consulted at source
4. timestamps (created / updated) are derived reliably

This avoids lossy aggregation steps and is intended to preserve git-based
provenance.

---

### 10.3 Engine Build

Aggregated docs are treated as a federated content source and rendered via
`theme-docs`.

---

## 11. What Astro-TURF Is *Not*

Astro-TURF is intentionally **not**:

- a CMS
- a page builder
- a plugin zoo
- a runtime content loader
- a framework replacement

It is an **orchestration layer**.

---

## 12. Design Guarantees

Astro-TURF aims to provide:

- a deterministic-oriented build model
- explicit contracts (by design intent)
- stable data shapes (subject to iteration)
- separation of concerns (as an architectural goal)
- extensibility without implicit coupling (as a guiding principle)

---

## 13. Open Areas

Explicitly deferred decisions:

- versioned documentation
- cross-collection search
- theme discovery
- multi-theme pages
- author attribution across repos

---

## 14. Closing Note

Astro-TURF prioritizes **structure over features**.

Once contracts, defaults, and responsibilities are correct, complexity remains
tractable.

This notebook is a map, not a manifesto.
