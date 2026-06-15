<!-- xkit package spec. Specs = Docs: this is xkit's own standing spec, carried in its own repo.
     Public + self-contained — references only public peers by URL, never private system docs.
     Per C-41 this describes the standalone component (the box + its interface), never the system wiring. -->

# xkit — the shared design system, render component, and API client

- **Package** `@xresearch/xkit`
- **Layer** 3 shared
- **Visibility** public
- **License** Apache-2.0
- **Language** TS
- **Status** active
- **Depends on** xcontract
- **Style** [xresearch-std/CODING-STYLE.md](https://github.com/xresearch-it/xresearch-std)

This document is xkit's own spec. Per **Specs = Docs**, it lives in this repo and is self-contained: an external contributor reads it without any private repo. It references only public peers — [`xcontract`](https://github.com/xresearch-it/xcontract) (the normative DTOs/Zod/enums) and [`xresearch-std`](https://github.com/xresearch-it/xresearch-std) (the coding constraints `C-n` cited throughout) — by URL.

## Responsibility

xkit is the **shared design-system + render-component + API-client library** — the box a UI is built *on*, never a UI itself. It owns three things, and ships no product screen of its own:

- **The design tokens** — the SSOT for the visual language (Institutional Technical Minimalism, below).
- **The one render component + its render protocol** — a single "subgraph-in → view-out" component, with the honesty-rendering conventions (the ◇-agent-glyph, the AI-wash tint, the confidence marks) pinned in it.
- **The typed API client** — a thin, typed wrapper over the shared contract for calling an edge.

What xkit does **not** own: the honesty *scoring* logic (a separate engine package computes scores; xkit only renders them); any application layout or screen composition; any edge runtime, route, persistence, or deployment. A consumer takes these three primitives and *wires* them — where that consumer lives, what data it feeds in, and which screens it builds are out of xkit's scope. *Litmus (C-41): extract xkit to its own service tomorrow → this spec does not change a word.*

## Interface

What crosses xkit's boundary, in and out:

- **Design tokens** (out): the typed token set — colors, type scale, spacing, the technical-monospace family, the KaTeX hookup. A consumer imports the tokens; it never forks them.
- **The render component** (in → out): takes **a subgraph + a selection** as opaque input, renders the right-hand view; emits selection/link events. It is parameterised by *which* subgraph and *which* entity type — it never knows where the subgraph came from.
- **The card component** (in → out): takes a paper / claim / answer entity, renders it at a requested density (detail / list / inline-in-graph / node-glyph), emits the same selection/link events.
- **The typed API client** (in → out): takes a contract-shaped request, returns `Result<T, E>` over a contract-shaped response. It targets the contract's routes; *where* those routes run is the caller's concern, injected, never baked in.
- **The server-side citation-render endpoint contract** (out): the copy-as-cited affordance is a *caller* of one read-class render endpoint — xkit ships no client-side citation logic; it issues the request and renders the result.

Every shape on this boundary is a normative DTO from [`xcontract`](https://github.com/xresearch-it/xcontract); xkit never invents a shape (C-2).

## Behaviour

### The design tokens — Institutional Technical Minimalism

xkit's tokens encode one aesthetic: a **scientific-publishing surface**, not a SaaS dashboard. **Typeset, not decorated; printed, not floating.** The aesthetic signals a notation system, not a startup.

- **Restrained palette.** A small, disciplined set of values. The palette reads as ink on a typeset page, not as brand chrome.
- **Technical monospace for the technical layer.** IDs, citation keys, equations, and source locators render in a technical monospace (JetBrains Mono / IBM Plex Mono). The monospace *is* the signal that a token is a machine-precise identifier, not prose.
- **KaTeX for math.** Equations render as real typeset math, not images or inline text.
- **Forbidden, by token construction:** gradients, glassmorphism, glow, floating orbs, and any "dopamine" AI palette. These are not style preferences — they are excluded from the token set so a render *cannot* reach for them.
- **Dark mode only at MVP** — no light-mode toggle ships in the first milestone; the token set is single-mode by decision, not by omission.

The tokens are xkit's SSOT. Any mirror of the visual language mirrors these tokens; it never forks them. A **token-drift check** keeps a mirror aligned with this source — a mirror that diverges is a drift bug, caught mechanically, not a local override (C-3: one canonical way per concern).

### Honesty rendering — the conventions xkit pins

The honesty *model* (what a score means, how an axis is computed) belongs to a separate engine package. xkit owns the **rendering** of it — how honesty becomes visible at a glance. These are visual conventions pinned in the render component, so honesty looks the same everywhere it is rendered:

- **The ◇ glyph is reserved for agents.** Every agent-authored card or avatar carries the ◇ glyph. A human-authored artifact never does. Agent provenance is structural, not decorative.
- **The AI-wash tint marks unaccepted agent output.** Any agent-authored block not yet accepted by a human carries a wash tint — AI provenance is visible *at a glance*, never discovered only on hover.
- **Two confidence marks, never merged.** A claim card shows its machine-coherence signal and its human-validation signal as **two distinct marks**. When human validation is empty it renders **empty** — never back-filled from the coherence signal. A net-non-positive validated state renders as its own **"disputed"** mark — never collapsed into the coherence mark, never shown as empty.
- **Auto-accepted is visibly auto-accepted.** A reference a machine gate accepted (rather than a human) carries a persistent "machine-reviewed, not validated" mark; it never renders like a human-reviewed claim.
- **Verb discipline on AI copy.** Component copy about agents and panels uses retrieval verbs (*retrieved, surfaced, suggested, flagged*), never comprehension claims (*understands, interprets, knows*). The audience is philosophically literate; an epistemically cavalier verb is a credibility-ender. The forbidden verbs are pinned in a copy-snapshot fixture, not left to drafting judgment.
- **Refusal and gap are first-class states.** "Nothing here clears the bar for this question" and "near, but short → an answerable gap" render as **designed outcomes**, not error toasts or empty results. Honest copy is claim-level, never an omniscient "the corpus is silent."
- **Tombstones render.** A superseded or retracted claim stays visible with a badge and its supersede pointer; a citation to it carries the badge too. The honesty survives wherever the claim travels — including the clipboard (a copied claim keeps its hedges, its disputed mark, its tombstone note).

These conventions are not optional render choices. They are part of xkit because the product's credibility rests on honesty being **uniform and unforgeable** — the only render path enforces the ◇ glyph and the wash tint, so an AI claim cannot be quietly rendered as a human one.

### The one-render-component principle

A paper, a claim, an answer, and a selection are all **subgraphs over the same claims**. So the right-hand view is **one component** that renders "a subgraph", parameterised by *which* subgraph:

- a paper → its subgraph · papers → the joined subgraph · an answer → the answering subgraph · a claim → a one-node (degenerate) subgraph.

New views — source-reading, compose, a divergence panel — are **view-states registered against an entity type**: **view-switching, not modal-stacking.** Compose is an *expand-mode* of the same component, not a new screen.

To be precise about what "one component" binds: it is **one render protocol** — one selection model, one link model, one subgraph-in / view-out interface. Specialized view implementations behind that protocol are fine and expected; what is forbidden is a **second selection or link model** (C-3). The protocol and the render component are xkit's; a caller wires them to data, it never introduces a parallel selection mechanism. The discipline keeps a UI small without mandating a god component.

**Typed, linkable entities — one link model.** Every entity — paper, claim, answer, question, subgraph, source — is a **typed, linkable resource**. A link to one (including inside a chat message) switches the view to that entity. **One link model, one switch behaviour** — defined once, here.

**Standardised cards.** A card (paper / claim / answer) is one component: a header + small avatar (who or what authored it — a person, or an agent carrying the ◇ glyph), the body, and an **expanded referenced-papers footer** separated by a thin line. **Density adapts to context** — detail / list / inline-in-graph / node-glyph — the same data in four sizes. A claim card shows both confidence marks distinctly (per Honesty rendering).

**Cross-context behaviours the component realises.** Behaviours that live in the render component because they must look identical wherever a claim is rendered:

- **Source-highlighting.** Click a claim → its grounding highlighted **in the original source**: a span in a PDF, or the image region for a claim extracted from a figure or standalone image. For *shared* claims the affordance is state-honest (quote-only / full-on-own-ingest / open-access one-click). In-text `[N]` citations are clickable to their reference entity. Provenance you can *see*, not trust.
- **Add-to-cart.** "I buy that claim" toggles the reference's composition flag; the working set is the compose input.
- **Copy-as-cited.** Every claim card and every answer carries a one-click copy affordance — the highest-frequency way a researcher takes value *out* (a deliverable lives outside the product: a LaTeX draft, an email, a slide). Formats: **plain** (claim + hedges + "Author (Year), locator") · **Markdown** (with quote + reference) · **LaTeX** (`\cite{key}` + the sentence, hedges bracketed, with the matching BibTeX entry alongside). The component is a **caller of one server-side render endpoint** (read-class, deterministic, instant) — never client-side citation logic, which would be a second implementation diverging silently (C-3). Multi-select is by construction: the selection *is* the request, and the collision scope; a multi-claim copy block is deliberately **not** a workspace export (copy carries no gaps, so it must never pose as "the state of the topic"). Every honesty badge survives the clipboard.

### The typed API client

xkit ships the **one typed client** for calling an edge through the contract. The client is a thin, typed wrapper:

- **Conforms to `xcontract`.** Request and response shapes are the normative DTOs from [`xcontract`](https://github.com/xresearch-it/xcontract) — the client never invents a shape, and CI diffs the client against the contract (C-2: the contract is the only source of shapes; never reference a field or route you have not read in it).
- **Parse at the boundary (C-10).** Every response is validated at the seam with the contract's Zod schema; past the seam the static type is the guarantee. No defensive re-checking scattered through callers.
- **Errors are values (C-11, C-13).** The client returns `Result<T, E>` across the boundary, with failures drawn from the contract's closed `ErrorCode` taxonomy — never free-text strings or HTTP numbers smeared through call sites. **No silent fallback** (C-12): a degraded path is an explicit, named, returned state, never a swallowed catch with a best-guess default.
- **Environment-agnostic.** The client is pure logic — it carries no edge/Workers specifics. It targets the contract's routes; *where* those routes run is injected, not xkit's concern.

One client, one set of shapes, one error taxonomy.

## Invariants

The `C-n` constraints that bind xkit hardest, written to [`xresearch-std/CODING-STYLE.md`](https://github.com/xresearch-it/xresearch-std):

- **C-2** — the contract is the only source of shapes: read it, never imagine it. Tokens, client, and component all speak `xcontract` DTOs.
- **C-3** — one canonical way per concern: the tokens, the render protocol, the link model, and the citation-render endpoint are each *the* one way; a second selection model or a client-side citation path is a bug.
- **C-10 / C-11 / C-12 / C-13** — parse at the boundary; errors as typed values; no silent fallback — the honesty model applied to the client.
- **C-15 / C-18** — pure core; compose only over the contract, never a peer's internals. xkit is environment-agnostic pure logic + UI components; it never imports edge/Workers/Pages specifics and never reaches into a peer package's internals. That single rule is what keeps it independently extractable.
- **C-34** — claims true at write-time: this doc asserts what xkit does *now*, not what it will do.

## UI

Like every package in the monolith, this repo ships **its own UI**: a dev / inspector / demo surface, built on xkit itself, that exercises the render component, the tokens, and the client **in isolation** — you see xkit's behaviour visually, standalone, without any consumer. Because xkit *is* the design system and render component, its dev UI is also the **reference rendering** — the canonical place the tokens, the card densities, the honesty marks, and the view-states are seen rendered correctly. A rendering that diverges from this reference has drifted.

All UI work in this repo runs through the `superpowers` `brainstorming/visual-companion` convention before a line of UI is written — a standing rule, no exceptions.

## Build & identity

The package's identity is its workspace name, `@xresearch/xkit`, stable regardless of where this repo physically lives — imports never change when the repo moves, which is what makes the structure viable without future restructuring.

Dependencies are minimal, pinned, audited (C-40); the lockfile is committed and supply-chain + license are checked in CI.

**Related (public peers):**

- [`xcontract`](https://github.com/xresearch-it/xcontract) — the normative request/response shapes the API client and the render boundary conform to and CI diffs against
- [`xresearch-std`](https://github.com/xresearch-it/xresearch-std) — the shared coding constraints (`C-n`) cited throughout, submodule'd into every repo
