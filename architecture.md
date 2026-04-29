# Architecture

A high-level look at how Studio Player is designed — the patterns that shape the codebase, the engineering decisions worth talking about, and the principles the project optimizes for.

> Source code is private. This page covers the *thinking*, not the *recipe* — implementation specifics live in the private repo.

---

## Single source of truth across four editors

Studio Player ships native integrations for Bricks, Gutenberg, Divi, and Elementor — plus a shortcode that works in any other context. Five surfaces, one rendering pipeline.

The pattern: **every editor integration delegates server-side rendering to the shortcode.** The Bricks element doesn't have its own render path. Neither does the Gutenberg block, the Divi module, or the Elementor widget. They each have their own attribute schema, their own UI controls, their own visual conventions — but when it's time to produce HTML, they all map their attributes onto the canonical shortcode shape and call into one function.

This means:

- Adding a new option once propagates to every editor automatically
- Bug fixes in the render layer land everywhere simultaneously
- The output is byte-identical regardless of which editor placed the player

The cost is a small attribute-mapping translation layer in each editor integration (camelCase for Gutenberg, snake_case for shortcode, on/off booleans for Divi, etc.). The benefit is that the rendering surface stays small, well-tested, and impossible to drift out of sync.

This is the kind of decision that's easy to get wrong. The common alternative — each editor has its own renderer — leads to four implementations that slowly diverge until "the same player" produces three slightly different outputs. The single-source pattern prevents that drift architecturally.

---

## True cross-page playback without a SPA theme

The hardest problem the project solves: **audio that survives page navigation** on standard, server-rendered WordPress sites.

Most "continuous playback" features in the WordPress plugin ecosystem rely on the active theme already implementing SPA-style navigation — barba.js, jquery.pjax, or similar. If the theme provides that, audio survives. If it doesn't, audio dies on every link click. Almost no Bricks or Divi sites have an SPA layer, which means almost no Bricks or Divi sites get the feature.

Studio Player ships its own AJAX navigation layer. Internal `<a>` clicks are intercepted, the destination is fetched, and a configurable content-area selector is replaced in-place. The player wrapper lives in a footer-injected persistent host that lives outside that swap region — so it's never destroyed, and the audio element never reloads.

The layer is intentionally conservative. External origins, fragments, modifier-key clicks, downloads, forms, and a configurable URL-exclusion list all defer to native browser navigation. Failures fall back to a full page reload — never silent breakage. The customer enables it via a single admin toggle and ships.

---

## A vanilla-JavaScript engine

The frontend engine is a single self-contained vanilla-JavaScript file. No bundler. No transpiler. No Node dependency. No build step.

The reasoning is opinionated:

- A WordPress plugin shouldn't require Node.js to clone and edit
- Browser support for everything used is broad (Web Audio, MediaElement, IntersectionObserver, MutationObserver, Web Share, FileReader, `fetch`)
- A build step would be dead weight at this size
- The engine works as a standalone drop-in script outside WordPress — useful for static-site demos and testing

Internationalization is handled the same way: the engine reads from `window.StudioPlayerI18n` with English fallbacks baked into every call. WordPress fills the dictionary via `wp_localize_script`; without WordPress, English strings render. Seventy-plus strings localized.

---

## In-house file-format parsing where it matters

Two surfaces in the engine parse file formats directly rather than reaching for a library:

**ID3 metadata reader** — visitor-uploaded MP3s get title, artist, album, and cover art automatically extracted on upload. Roughly 100 lines of ID3v2.3 / v2.4 parsing in the engine, defensive about text encoding (Latin-1, UTF-16 with BOM, UTF-8) and about malformed files. No `jsmediatags` dependency, no fingerprintable third-party CDN request.

**Adaptive accent color** — the dominant color in a track's cover art is extracted via canvas pixel sampling and applied to the player's accent variable. The math is straightforward (skip near-white, near-black, low-alpha pixels; average what's left); the value is in shipping it inline rather than as a library.

Each of these is the kind of feature that gets *added* to a player rather than *built into* one. Doing it inline keeps the dependency surface small.

---

## Lazy-loaded heavy dependencies

The project has exactly one optional external dependency: `wavesurfer.js` for waveform visualization (~70 KB).

It loads only when the toggle is on, only on the page where it's needed. Players without waveform mode pay zero cost. Self-hosters can override the URL to keep the dependency same-origin.

This is the model for any future heavy dependency — opt-in, lazy, with a self-host option.

---

## A persistence model designed around per-player state

Listener-side state lives in the browser, scoped per player via a configurable persistence key. Different players on the same site get different keys and don't interfere with each other.

State persisted:

- Collapsed / expanded UI state
- Volume level
- Playback speed
- Shuffle on/off
- Repeat mode (none / all / one)
- Per-track resume position
- Cross-page resume snapshot (for continuous playback)

Most of this lives in `localStorage` (persistent across sessions). The cross-page resume snapshot uses `sessionStorage` instead — exactly the lifecycle we want for "audio keeps playing within this browsing session, but a fresh tab starts fresh."

---

## Performance, where it actually matters

Most frontend performance work is premature optimization. The engine treats performance as a discipline applied to specific hot paths, not a blanket rule.

Three places get explicit attention:

- **Engine boot on page load** — the engine's auto-boot scanner runs once on `DOMContentLoaded` and once via `MutationObserver` for late-inserted mounts (the Bricks builder canvas re-inserts elements via Vue, which the observer catches). Subsequent renders don't re-scan unless something changes.
- **Cover-art adaptive color** — the canvas sampling happens at low resolution (12×12 pixels) once per track change. The math runs in microseconds; the cost is the image fetch, which the browser caches.
- **AJAX-nav content swap** — the swap reads only the configured selector from the response, not the full DOM, and re-runs only inline scripts inside that region.

Other code paths take the straightforward approach. Optimization happens in response to measurement, not preemptively.

---

## Design principles

Five constraints the architecture is consistently optimized for:

1. **One renderer, four editors** — single source of truth at the render layer means no drift between editor surfaces.
2. **Audio that survives navigation** — the differentiating feature, supported by the persistent-host pattern and a conservative AJAX nav layer.
3. **No build step** — vanilla JavaScript, single file, edit-and-reload workflow. The plugin should be readable and modifiable without a tooling chain.
4. **Lazy-loaded everything heavy** — the default install pays only for what it uses. Waveform off → no wavesurfer. ID3 disabled → no parser ran. Adaptive color off → no canvas work.
5. **Engine usable outside WordPress** — the JavaScript engine works as a standalone drop-in. WordPress integration is a layer on top, not a requirement.

---

## What this document doesn't cover

Implementation specifics — file paths, class signatures, exact persistence keys, internal data structures, license validation behavior — are intentionally omitted. They live in the private repo.

For licensing inquiries, evaluation copies, or technical conversations with prospective collaborators: contact via the [Studio Grid GitHub](https://github.com/studiogrid).
